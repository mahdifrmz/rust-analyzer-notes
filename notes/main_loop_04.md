# Summary of ERA04: main loop

## RA entry point : main()
the main function does a bunch of stuff:
* enable debugging the binary
* check for command line flags and sub commands
* check for the configuration and cargo meta data
* finally, run the main loop

## GlobalState
the `GlobalState` is the state of Rust Analyzer with is composed of many types, the most important being: 
* `Vfs` the virtual filesystem
* `AnalysisHost` the analysis data
* `TaskPool` the thread pool for concurrency
* `ReqQueue` maintains the incomming and outgoing LSP messages

* `memdoc` property, which keeps content of the files whose changes are not saved to the disk, and are handled by the editor
* `loader` property, which is a handle to the [vfs loader](vfs_02.md)
the `run` method of this struct will run the main loop.