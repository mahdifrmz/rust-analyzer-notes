# Summary of ERA07: syntax

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
The compiler only goes from one stage to another, at most only caring about token's positions and some simple diagnostics. But an ide might go from one stage to another due to finding errors in one stage and having to fix that in another stage. As an example, it might find an error during the type inference stage, and to provide fixture for that, it has to work on the ast.