# parking_spot

Implements thread wait and notify primitives with `std::sync` primitives.

This is a simplified version of the `parking_lot_core` crate.

There are two main operations that can be performed:

- *Parking* refers to suspending the thread while simultaneously enqueuing it
on a queue keyed by some address.
- *Unparking* refers to dequeuing a thread from a queue keyed by some address
and resuming it.

## Example

```rust
use std::sync::atomic::{AtomicU64, Ordering};
use std::sync::Arc;
use std::thread;
use parking_spot::ParkingSpot;

let parking_spot: Arc<ParkingSpot> = Arc::new(ParkingSpot::default());
let atomic: Arc<AtomicU64> = Arc::new(AtomicU64::new(0));
let atomic_key = Arc::as_ptr(&atomic) as usize;

let handle = {
    let parking_spot = parking_spot.clone();
    let atomic = atomic.clone();
    thread::spawn(move || {
        atomic.store(1, Ordering::Relaxed);
        parking_spot.unpark_all(atomic_key);
        parking_spot.park(atomic_key, || atomic.load(Ordering::SeqCst) == 1, None);
        assert_eq!(atomic.load(Ordering::SeqCst), 2);
    })
};

parking_spot.park(atomic_key, || atomic.load(Ordering::SeqCst) == 0, None);
atomic.store(2, Ordering::Relaxed);
parking_spot.unpark_all(atomic_key);

handle.join().unwrap();
```

License: MIT OR Apache-2.0
