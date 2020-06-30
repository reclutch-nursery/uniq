# Uniq

Unique application-wide queue adapter through strongly-typed event identification.

```rust
// A thread-safe variant also exists; `arc::Queue`.
let q = rc::Queue::new();
// The ID type is generic, but it defaults to `u64`.

let mut l1 = q.listen()
    .and_on(0, |_: &mut (), _event: &ClickEvent| {
        println!("listener 1 observed a click from id 0");
    })
    .and_on(5, |_, _, _event: &WindowEvent| {
        // we don't need to specify `&mut ()` since we already specified it in the earlier closure (intra-listener homogenity).
        println!("listener 1 observed a window event from id 5");
    });

let mut l2 = q.listen()
    .and_on(3, |_: &mut i32, _event: &NetworkEvent| {
        println!("listener 2 observed a network event from id 3");
    })
    .and_on(2, |_, _event: &NetworkEvent| {
        println!("listener 2 observed a network event from id 2");
    });

q.emit_owned(0, ClickEvent);
q.emit_owned(2, NetworkEvent);
q.emit_owned(3, NetworkEvent);
q.emit_owned(5, WindowEvent);

l1.dispatch(&mut ());
l2.dispatch(&mut 0);
// Note that the first parameter of the closure in `l2` differ from that of `l1` (inter-listener heterogenity).
```

The first parameter of the event handlers is an arbitrary mutable object that can be passed to handler code. The type of this parameter is homogenous intra-listener and heterogenous inter-listener.

In the `Rc` variant, this mutable object can be a tuple so that you can pass multiple mutable references:
```rust
let mut l = q.listen()
    .and_on(0, |_: &mut (&mut Foo, &mut Bar), _event: &Event| { /* ... */ })
```

Note that this multi-parameter feature uses `unsafe` code under-the-hood, but thus far it appears to be sound (+ Miri). It is unlikely this feature will be brought over to the `Arc` variant.

Notice that the second listener is able to distinguish `NetworkEvents` coming from two difference sources.

There's a simple additional utility module; `uniq::id`, which will atomically generate a globally unique ID. This is useful for typical applications of `uniq`.

For example, each widget in a UI may have a unique ID, hence allowing multiple widgets of the same type to distinguish the events they emit. The alternative to
this is to have an event queue for each widget, but having multiple event queue is known to cause event ordering problems.

```rust
// `next()` can be invoked from any thread.

let id_1 = uniq::id::next();
let id_2 = uniq::id::next();

assert_ne!(id_1, id_2);
```

This crate is based on `reclutch-nursery/even2` and `jazzfool/sinq`, and is somewhat of a combination of them.

## License

Uniq is licensed under either

- [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0)
- [MIT](http://opensource.org/licenses/MIT)

at your choosing.
