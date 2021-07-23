## IDE
the ide crate is the facsade that contains the Rust ANalyzer API. the general workflow of this crate is sort of similar to vfs; as you can apply changes to an `AnalysisHost` instance, which is a snapshot at a single time and later derive an `Analysis` data from that host. this analysis instance can be used for executing an specific ide query about the state at that time.

## Change
a `Change` describes the change done to the files. this includes the set of file contents from the vfs (which is shared using an `Arc<String>` for sharing from the vfs-notifier thread) and the list of `FileSet`s created after [partitioning](vfs_02.md). it also contains the `CrateGraph` for storing the dependency between the crates (link will be put later). This information provides Rust Analyzer with the ability to handle multi-crate and multi-workspace projects.

## Analysis
The `Analysis`struct is a read-only type which can be queried about the current state of the world. It's an **owned** type with a shared reference to the state that is also held by the `AnalysisHost`. But why have these two different types been defined to work with the state reference? First we should take a look at how the main loop works:

## Main Loop
The main loop (which will be covered in it's own notes) will listen for events and when a change is made to the vfs, it will apply those changes to the `AnalysisHost`. After that, it will spawn a bunch of background threads and moves `Analysis` instances to these threads to dervie information about the current state. As these threads want to read the same data at the same, they have to be immutable. But there's another issue: if in the middle of doing analysis by the worker thread, a change happens to the vfs and the mainloop wants to apply those changes to the shared reference of the state, this can't be done while those immutable refernces of Analysis insatnces in other threads are still Alive. The solution is: