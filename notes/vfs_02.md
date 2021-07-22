# Summary of explaining RA02: vfs

> You can watch actual video here:

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/SaSULKoSlWI/1.jpg)](https://www.youtube.com/watch?v=SaSULKoSlWI&list=PLhb66M_x9UmrqXhQuIpWC5VgTdrGxMx3y)

## Paths

The `Path` crate provides 4 structs for working with paths: `AbsPathBuf`, `AbsPath`, `RelPathBuf` and `RelPath` which correspond to `std::fs::PathBuf` and `std::fs::Path`.
Rust analyzer handles paths differently from batch tools e.g.compilers.

A **batch tool** is:
* a short lived process invoked in a directory.
* has a specific `current working directory` while it's running.
* every relative path supplied to the tool can be turned into an absolute path based on the CWD.

While **Rust Analyzer** is:
* a long-lived process.
* can handle many projects at a time.
* provides cross-referencing between two crates in case of one depending on the other.
* thus has no notion of current working directory.

## Virtual File System

why do we need one? why not just use the platform's filesystem?

```
let f = std::fs::read_to_string("file.rs")?; // this is bad!
```

### Repeatable Reads
A typical batch tool such as a compiler, reads any individual file just one time. but RA has to read a file many times as the user changes the file. another issue is that RA occupies more memory than a compiler, cause when compiling a crate, the compiler only cares about the crate being compiled and the API and metadata of it's dependencies. RA however, has to parse the other crates' source files to get information for finding refernces, definitions, etc. due to that, it tends to *forget* about the derived data.
but in case of need to get back that derived data later, it will have to read the file again and if the file has been changed and with the OS providing no mechanism for repeatable reads, then inconsistency might occure and cause things to break.

### Paltform-agnostuc paths
RA likes to think about filesystem as a tree of text files, regardless of the platfrom they reside in. the differences between filesystems of operating systems and their complicated logics makes them not be suitable for this purpose. 

### Multiple Clients
While in a typical file system, a `path/to/file.rs` can only refer to one file, Rust Analyzer with the goal of providing service to multiple clients and their projects in multiple machines needs a more flexible file system and it's files as easily-addressable logical entities.

## FileID
Each file in the vfs is assigned a 32bit file id corresponding to it's VfsPath. the reason for not using the path as id is:
1. using a 32bit id is prefered to a variable-size VfsPath
2. it will be impossible to access any file except through the vfs

## VfsPath
A virtual file system path can be represented in two ways:
1. as an `AbsPathBuf` which is the OS's absolute file path
2. as a virtual file path, which will make writing tests for the VFS easier

## Vfs changes
the vfs struct is basically a single snapshot of the filesystem at a specific point of time. new content for each file will be added via it's API which can also provide a record of changes done to the files.

## Loader
the loader is the part of the vfs crate responsible for loading files into the vfs and watch for the changes. it works with system path type while the vfs itself works with `VfsPath`. it keeps a set of entries which are the files that have to be loaded from the disk into the vfs and also keeps the indexes of the entries for whose change the loader has to watch (unlike the immutable cargo registry crates). all of the mentioned are stored as a config and during the lifetime of Rust Analyzer, mutiple configs might exist.

> the reality is that neither the loader, nor any other part of RA actually does any IO; the loader is only notified about the changes. 

## Handle
The handle is an interface for the filesystem change watcher. an implementation for this trait rests in the vfs-notify crate.
the main loop of Rust Analyzer spawns an instance of `NotifyActor` which creates a thread that watches for file system changes and emits the changes as events containing the changes with their corresponding `AbsPathBuf`. these events are later handled in the main loop which applies the changes to the vfs.

## AnchoredPath
this struct is necessary to differentiate between the files with the same path, but on two different client filesystems

## FileSet
Ideally, a file set is just a collection of file ids (while the class also has to store their VfsPaths). because Rust Analyzer will be handling many crates at the same time and has to know which file belong to which crate, it creates a FileSet for any single create. after any change(Create/Delete but not Modify) to the vfs, a partitioning will be done to the whole vfs and the files will be added to their crate's FileSet. Rust Analyzer does this because it wants to know changes done to which files will actually affect what other files.