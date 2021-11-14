# Departure

This is a planned experiment for managed interrupts using embedded rust.

The current state of the art for interrupts looks like this:

```rust
#[interrupt]
fn UARTE0() {
    // ...
}
```

With Departure, we should be able to do this instead:

```rust
use departure::{
    spawn_oneshot,
    spawn_repeated,
    Error,
};
use pac::interrupts;

#[entry]
fn main() -> ! {
    let x = 0u32;
    let y = 0i32;
    let handle1 = match spawn_repeated(
        interrupts::UARTE0,
        // Data in this closure is moved to a heap,
        // and is given to the interrupt.
        // This is a Fn/FnOnce.
        move || {
            x += 1;
        }
    ) {
        Ok(handle) => {
            /* Interrupt registered */
            /* Will be repeated */
            handle
        }
        Err(Error::InsufficientSpace(_)) => {
            /* Interrupt not started, no room for environment */
            /* Values are returned */
            panic!();
        }
        Err(Error::InterruptInProgress(_)) => {
            /* Interrupt is already managed, join before starting a new handler */
            /* Values are returned */
            panic!();
        }
    };

    // Immediately disable interrupt, get data back.
    let a: u32 = handle1.disable();

    let handle2 = spawn_oneshot(
        interrupts::GPIOTE,
        // The data is still moved to the heap, but
        // this is a FnOnce
        move || {
            y - 1
        }
    ).unwrap();

    // Block until interrupt occurs and is processed
    let b: i32 = handle.block_join();
}
```


# License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or
  http://www.apache.org/licenses/LICENSE-2.0)

- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
