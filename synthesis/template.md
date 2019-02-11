# Writing Templates

A synthesis template defines the ILA model partially where the state update functions of instructions can be left not-fully unspecified. That is, based on the set of architectural states and the set of instructions, the template defines the possible space of state updates in which the synthesis engine determines the exact solution. 

### ItSy Python library

After [building the synthesis engine](../start/python.md), the Python library can be imported to write a template.

```python
#!/bin/python
import ila
m = ila.Abstraction("test")
```

### Defining architectural states

This sets the base \(leaf nodes\) of the possible state update functions. TODO.

### Defining instructions

This is the set of commands with which the synthesis engine enumerates. TODO.

### Defining state update functions

This part is the key part and the most time-consuming part of the synthesis. If the space is not defined correctly, there is a chance where the synthesis can not terminate/complete. In such case, it is usually the simulator results mismatch with the search space. 

There are a set of synthesis primitives, e.g., choice, slice, etc. 

