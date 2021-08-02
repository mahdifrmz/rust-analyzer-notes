# Summary of ERA06: base db

> You can watch the video here:

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/m1EXCpIHSKA/1.jpg)](https://www.youtube.com/watch?v=m1EXCpIHSKA)

## Salsa
Before proceeding with **base_db** crate, we need to understand what [Salsa](https://github.com/salsa-rs/salsa) is and how it works. The heavily commented [hello_world](https://github.com/salsa-rs/salsa/blob/master/examples/hello_world/main.rs) example is a good place to get familiar with the interface.

## SourceDataBase
This is a Salsa database with the input `crate_graph` and the query `parse`; and also inherits the `FileLoader` trait. The parse query depends on `file_text` input, which is not originally a method of the `SourceDataBase` trait.

## SourceDataBaseExt
This database and the prevoius one could be just one databse, but in that case HIR knowledge of the source roots would be accessible by the parse query. Now due to the parse query requiring to get file's content, a `FileLoader` is provided to make the parser able to read a file's content by having its path.

> The notes will be expanded in the future