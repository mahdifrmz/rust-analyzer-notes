# Summary of ERA04: main loop

## RA entry point : main()
The main function does a bunch of stuff:
* enable debugging the binary
* check for command line flags and sub commands
* check for the configuration and cargo meta data
* finally, run the main loop

> NOTE: Throughout this lecture, topics such as handling Cargo checks,metadata and loading Cargo configs are explained, due to their complicated logic and implementation.

## GlobalState
The `GlobalState` is the state of Rust Analyzer with is composed of many types, the most important being: 
* `Vfs` the virtual filesystem
* `AnalysisHost` the analysis data
* `TaskPool` the thread pool for concurrency
* `ReqQueue` maintains the incomming and outgoing LSP messages

* `memdoc` property, which keeps content of the files whose changes are not saved to the disk, and are handled by the editor
* `loader` property, which is a handle to the [vfs loader](vfs_02.md)
The `run` method of this struct will run the main loop.

## Main Loop
The main loop is basically the following block of code:
``` rust
// crates/rust-analyzer/src/main_loop.rs
while let Some(event) = self.next_event(&inbox) {
    if let Event::Lsp(lsp_server::Message::Notification(not)) = &event {
        if not.method == lsp_types::notification::Exit::METHOD {
            return Ok(());
        }
    }
    self.handle_event(event)?
}
```
The `next_event` method will block on 4 crossbeam channels to get an event instance. After that, it will be handled by `handle_event` function.

## Events
There are four types of events in RA:
``` rust
enum Event {
    Lsp(lsp_server::Message),
    Task(Task),
    Vfs(vfs::loader::Message),
    Flycheck(flycheck::Message),
}
```
### Vfs
As the loader keeps watching for file changes and loads the files, these changes mut be applied to the vfs instance of the global state. An important point is that the main loop applies the vfs changes **in bulk**, meaning that after handling a vfs event, if there are already extra vfs events, they will immediately be handled before **processing changes**.

### LSP
An LSP Message can either be a `Request`, `Response`, or a `Notification`. Both the server and LSP client can send request and recieve responses. Each request is given an ID and is supposed to be handled by a response with the same ID. A notification is another type of request which does not need a response. They're covered in the next section.