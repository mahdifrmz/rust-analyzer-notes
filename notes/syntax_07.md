# Summary of ERA07: syntax

> You can watch actual video here:

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/8jryEJRnPfU/1.jpg)](https://youtu.be/8jryEJRnPfU)

## Compiler Pipeline
compiling a source code is divided into multiple stages:
```
text
  |   lexing
tokens
  |   parsing
AST
  |   desugaring
simplified AST
  |   name resolution & type inference
Annotated AST
  |   lowering
Intermediate representation
  |
backend
```
The compiler only goes from one stage to another, at most only caring about token's positions and some simple diagnostics. But an ide might go from one stage to another due to finding errors in one stage and having to fix that in another stage. As an example, it might find an error during the type inference stage, and to provide fixture for that, it has to work on the AST.

## Syntax tree API
The [api_walkthrough](https://github.com/rust-analyzer/rust-analyzer/blob/master/crates/syntax/src/lib.rs) test specifies how the AST can be queried. Important points about API methods:
* Parse method always returns an AST, as the ide needs to parse the code even when the syntax in not correct.
* The AST is lossless, meaning you can always convert from AST to text, and the reverse.
* Methods tend to return `Option<T>`, as the syntax might have been incorrect at some position in the code.
* Each AST node type is a wrapper around the `&SyntaxNode` type. Casting can happen between these two types.
* The API provides methods for traversing the tree, meaning from each node, accessing parent,ancestors,children,descendants and siblings is possible.