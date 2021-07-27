# Summary of ERA04: main loop

## RA entry point : main()
The main function does a bunch of stuff:
* enable debugging the binary
* check for command line flags and sub commands
* check for the configuration and cargo meta data
* finally, run the main loop

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