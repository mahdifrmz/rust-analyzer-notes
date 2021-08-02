# Summary of ERA06: base db

## Salsa
Before proceeding with **base_db** crate, we need to understand what [Salsa](https://github.com/salsa-rs/salsa) is and how it works. The heavily commented [hello_world](https://github.com/salsa-rs/salsa/blob/master/examples/hello_world/main.rs) example is a good place to get familiar with the interface.

## SourceDataBase
This is a Salsa database with the input `crate_graph` and the query `parse`; and also inherits the `FileLoader` trait. The parse query depends on `file_text` input, which is not originally a method of the `SourceDataBase` trait.