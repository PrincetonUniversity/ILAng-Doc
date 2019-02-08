# Modeling

This section covers how to construct the ILA formal model through ILAng's programming interface. You will learn how to define each component of the ILA model. A complete list of all supported API with detailed description can be found [here](https://bo-yuan-huang.github.io/ILAng/doxygen-html/namespaceilang.html). 

## ILA formal model

As described in [TODAES18](https://bo-yuan-huang.github.io/ILAng/papers/todaes18.pdf), the ILA formal model has several valuable attributes for modeling and verification. It is modular, with functionality expressed as **a set of instructions**. It enables meaningful abstraction through **architectural state** that is persistent across instructions. It provides for portability through a more durable interface with the interacting processors. It is **hierarchical**, providing for multiple levels of abstraction for modeling complex instructions as a software program through sub- and micro-instructions. 

