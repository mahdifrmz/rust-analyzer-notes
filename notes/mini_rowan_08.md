# Summary of ERA08: Mini Rowan

## characterisitics of Rowan tree
* full-fidelity : The trees must be lossless; white spaces, newlines and comments should also be part of the tree
* resilient and semi-structured : Invalid code also should be represented in the tree
* value-type : In compilers the trees are consistant and nodes have a rigid identity; but in ide the trees need to be flexible due to the incomming refactors.
* immutable : It's OK for a compiler to mutate the syntax tree (for desugaring, etc) but an ide has to keep the user input and not change the synatx tree.
* cheaply updatable : Although the tree should not be changed during analysis, again it has to be updated when a refactor occures. It can also be helpfull in case of incremental reparsing.
* conveniently updatable : The refactor API must be convenient to use.
* easy to navigate : We must be able to navigate the tree easily, meaning to go from a node to it's children, siblings or parents.

## Implementation
you can view the [actual rowan implemenation](https://github.com/rust-analyzer/rowan) which might seem complex and not really suitable. Afterall, this video is mostly coding and all implementation details are fully explained in the video; leaving me with not much to say in this note:)