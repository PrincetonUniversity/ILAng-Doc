# ILA Model

## Defining ILA models

To start defining an ILA model, the first thing is to create an `Ila` object at the top-most level.  

```cpp
auto m = ilang::Ila("model_name");
```

The object hierarchy is structured based on the ILA formal model:

```text
ILA
+-  Architectural states
|   +-  Inputs variables (leaf nodes of type Boolean or bit-vector)
|   +-  State variables (leaf nodes of type Boolean, bit-vector, or memory)
|
+-  Instructions
|   +-  Decode function: expression (bit-vector)
|   +-  State update functions: expression (same type as the state)
|
+-  Valid function: expression (Boolean)
|
+-  Fetch function: expression (bit-vector)
|
+-  Initial conditions: expression (Boolean)
|
+-  Child ILAs
```

Note: _expressions_ are the AST tree where input and state variables are the leaf nodes. 

