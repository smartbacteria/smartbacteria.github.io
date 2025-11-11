**[Shared Memory IPC in Rust                                             06/23/2024
--------------------------------------------------------------------------------]**

I recently came across ?[an interesting crate for Rust](https://crates.io/crates/shmem-ipc) that enables interprocess
communication via shared memory. At the time I was looking for a faster method
of transferring large amounts of data between different processes, and this
crate proved to be an effective solution.

PSA: this involves the use of unsafe Rust. Yipee!


**[Initial Setup
-------------]**

Besides just this crate, you need a few other things to make this work. There
needs to be a way to transfer file descriptors between the two programs. The
examples provided in the repository use DBUS to achieve this, but this is
honestly overkill. Instead, we use good ol' Unix Sockets along with ?[a nifty](https://crates.io/crates/passfd)
?[crate called passfd](https://crates.io/crates/passfd) to transfer the file descriptors and set up the shared
memory pipeline.

Start by creating a new project the usual way:

--------------------------------------------------------------------------------

  $ cargo new shmem-test
  $ cd shmem-test

--------------------------------------------------------------------------------


You will also need to modify Cargo.toml to include the two crates previously
mentioned and create separate client and server binaries:

--------------------------------------------------------------------------------

```[[package]
name = "shmem-test"
version = "0.1.0"
edition = "2021"

[dependencies]
anyhow = "1.0"
shmem-ipc = "0.3.0"
passfd = "0.1.6"

[[bin]]
name = "client"
path = "src/client.rs"

[[bin]]
name = "server"
path = "src/server.rs"]```
--------------------------------------------------------------------------------


**[Basic Server-Client Interaction
-------------------------------]**

Let's modify the example from the shmem-ipc repo to use sockets and passfd
instead of dbus.

Starting with server.rs:

--------------------------------------------------------------------------------

```[use std::fs::{self, File};
use std::os::unix::io::AsRawFd;
use std::os::unix::net::UnixListener;
use std::sync::{Arc, Mutex};
use std::thread;

use anyhow::Result;
use passfd::FdPassingExt;
use shmem_ipc::sharedring::Receiver;

const CAPACITY: usize = 500000;

struct State {
    listener: UnixListener,
    sum: Arc<Mutex<f64>>,
}

impl State {
    pub fn new() -> Result<Self> {
        // If the socket exists, remove it to avoid an error.
        if fs::metadata("/tmp/shmem-test.sock").is_ok() {
            fs::remove_file("/tmp/shmem-test.sock")?;
        }

        Ok(Self {
            listener: UnixListener::bind("/tmp/shmem-test.sock")?,
            sum: Arc::new(Mutex::new(0.0)),
        })
    }

    pub fn add_receiver(\&mut self) -> Result<(File, File, File)> {
        // Create a receiver in shared memory.
        let mut r = Receiver::new(CAPACITY as usize)?;
        let m = r.memfd().as_file().try_clone()?;
        let e = r.empty_signal().try_clone()?;
        let f = r.full_signal().try_clone()?;
        // In this example, we spawn a thread for every ringbuffer. More complex
        // real-world scenarios might multiplex using non-block frameworks, as
        // well as having a mechanism to detect when a client is gone.
        let sum = self.sum.clone();
        thread::spawn(move || {
            loop {
                r.block_until_readable().unwrap(); 

                // Instead of using receive_raw like the example, we use
                // receive_trusted to work with normal slices. Receive_raw may
                // be slightly faster since it eliminates a call to
                // slice::from_raw_parts.
                unsafe {
                    r.receive_trusted(|p: \&[f64]| {
                        let mut s = 0.0f64;
                        for i in p {
                            s += *i;
                        }

                        *sum.lock().unwrap() += s;
                        return p.len();
                    }).unwrap();
                }
            }
        });
        Ok((m, e, f))
    }

    pub fn send_fds(\&mut self) -> Result<()> {
        // Add a new shared memory reciever and get the file descriptors.
        let (m, e, f) = self.add_receiver()?;
        // Wait for an incoming connection over the socket.
        let (stream, _) = self.listener.accept()?;

        // Once a client has connected, send the file descriptors.
        stream.send_fd(m.as_raw_fd())?;
        stream.send_fd(e.as_raw_fd())?;
        stream.send_fd(f.as_raw_fd())?;

        Ok(())
    }
}

fn main() -> Result<()> {
    // Wait for an incoming connection.
    let mut com = State::new()?;
    com.send_fds()?;

    // Once we have a connection, read the values and do something. A real
    // application may do something more interesting here.
    let mut last_sum = 0.0;
    loop {
        let sum = *com.sum.lock().unwrap();
        if sum != last_sum {
            last_sum = sum;
            println!("Sum: {}", sum);
        }
    }
}]```
--------------------------------------------------------------------------------


And client.rs:

--------------------------------------------------------------------------------

```[use std::fs::File;
use std::os::unix::net::UnixStream;
use std::os::unix::io::FromRawFd;
use std::time::Duration;
use std::thread::sleep;

use anyhow::Result;
use passfd::FdPassingExt;
use shmem_ipc::sharedring::Sender;

const CAPACITY: usize = 500000;

fn main() -> Result<()> {
    let fstream = UnixStream::connect("/tmp/shmem-test.sock")?;

    let mfd = fstream.recv_fd()?;
    let m = unsafe { File::from_raw_fd(mfd) };

    let efd = fstream.recv_fd()?;
    let e = unsafe { File::from_raw_fd(efd) };

    let ffd = fstream.recv_fd()?;
    let f = unsafe { File::from_raw_fd(ffd) };

    let mut r = Sender::open(CAPACITY, m, e, f)?;
    let mut items = 100000;

    loop {
        let item = 1.0f64 / (items as f64);

        // Again, we use send_trusted instead of send_raw to work with normal
        // slices instead of raw pointers. Send_raw may have a slight decrease
        // in latency since it eliminates the call to slice::from_raw_parts.
        unsafe {
            r.send_trusted(|p: \&mut [f64]| {
                let num = if items < p.len() {
                    items
                } else {
                    p.len()
                };

                for i in 0..num {
                    p[i as usize] = item;
                }

                println!(
                    "Sending {} items of {}, in total {}",
                    num, item, (num as f64) * item
                );

                return num;
            }).unwrap();
        }

        items += 100000;
        sleep(Duration::from_millis(1000));
    }
}]```
--------------------------------------------------------------------------------


If you open two terminals and run these commands in separate terminals (starting
the server first), you should see some things start to happen.

--------------------------------------------------------------------------------

  # Run this first, in terminal 1
  $ cargo run --release --bin server

      Finished `release` profile [optimized] target(s) in 0.12s
       Running `target/release/server`
  Sum: 0.9999999999980838
  Sum: 2.0000000000004015
  Sum: 2.6673866666694828
  Sum: 3.667386666673917
  Sum: 3.867818666674076
  Sum: 4.701512000011892
  Sum: 5.416106285733997
  Sum: 6.041376285731472
  Sum: 6.597171841289496
  Sum: 7.097387841283043
  Sum: 7.552129659467069
  Sum: 7.968976326135977
  Sum: 8.353757864599249
  Sum: 8.7110550074603
  Sum: 9.044532340793014
  Sum: 9.35716734079175

--------------------------------------------------------------------------------

  # Run this after the server starts, in terminal 2:
  $ cargo run --release --bin client

      Finished `release` profile [optimized] target(s) in 0.01s
       Running `target/release/client`
  Sending 100000 items of 0.00001, in total 1
  Sending 200000 items of 0.000005, in total 1
  Sending 200216 items of 0.0000033333333333333333, in total 0.6673866666666667
  Sending 400000 items of 0.0000025, in total 1
  Sending 100216 items of 0.000002, in total 0.200432
  Sending 500216 items of 0.0000016666666666666667, in total 0.8336933333333333
  Sending 500216 items of 0.0000014285714285714286, in total 0.7145942857142857
  Sending 500216 items of 0.00000125, in total 0.6252700000000001
  Sending 500216 items of 0.000001111111111111111, in total 0.5557955555555555
  Sending 500216 items of 0.000001, in total 0.500216
  Sending 500216 items of 0.000000909090909090909, in total 0.45474181818181814
  Sending 500216 items of 0.0000008333333333333333, in total 0.41684666666666664
  Sending 500216 items of 0.0000007692307692307693, in total 0.38478153846153845
  Sending 500216 items of 0.0000007142857142857143, in total 0.35729714285714287
  Sending 500216 items of 0.0000006666666666666667, in total 0.33347733333333335
  Sending 500216 items of 0.000000625, in total 0.31263500000000005

--------------------------------------------------------------------------------


Cool!

