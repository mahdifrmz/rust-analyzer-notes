# Summary of explaining RA03: ide

> You can watch the video here:

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/5l31PsZJ2Cc/1.jpg)](https://www.youtube.com/watch?v=5l31PsZJ2Cc)

## IDE
The ide crate is the facsade that contains the Rust ANalyzer API. The general workflow of this crate is sort of similar to vfs; as you can apply changes to an `AnalysisHost` instance, which is a snapshot at a single time and later derive an `Analysis` data from that host. This analysis instance can be used for executing an specific ide query about the state at that time.

## Change
A `Change` describes the change done to the files. This includes the set of file contents from the vfs (which is shared using an `Arc<String>` for sharing from the vfs-notifier thread) and the list of `FileSet`s created after [partitioning](vfs_02.md). It also contains the `CrateGraph` for storing the dependency between the crates (link will be put later). This information provides Rust Analyzer with the ability to handle multi-crate and multi-workspace projects.

## Analysis
The `Analysis`struct is a read-only type which can be queried about the current state of the world. It's an **owned** type with a shared reference to the state that is also held by the `AnalysisHost`. But why have these two different types been defined to work with the state reference? First we should take a look at how the main loop works:

## Main Loop
The main loop (which will be covered in it's own notes) will listen for events and when a change is made to the vfs, it will apply those changes to the `AnalysisHost`. After that, it will spawn a bunch of background threads and moves `Analysis` instances to these threads to dervie information about the current state. As these threads want to read the same data at the same, they have to be immutable. But there's another issue: if in the middle of doing analysis by the worker thread, a change happens to the vfs and the mainloop wants to apply those changes to the shared reference of the state, this can't be done while those immutable refernces of Analysis insatnces in other threads are still Alive. The solution is:

## Cancellable
The result of each query executed on an `Analysis` is a Cancellable.
```
pub type Cancellable<T> = Result<T, Cancelled>;
```
After the main loop gets aware of the new change, it'll call cancel on all working threads, as the result of their computation is not needed anymore. this will delay the call to apply change, untill all refernces to the state have been released and then the change will be applied and analysis will start again.

## IDE is boundry!
Rust Analyzer is a constantly evolving project with many crates. the boundries between these crates is always supposed to be as simple as possible (like the `Analysis` API methods). The reason for this is that if anything in another crate changes, we will not have to change other crates' implementations.