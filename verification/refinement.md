# Refinement Relation


The refinement map or refinement relation mainly consists of two parts: _variable mappings_ and 
_instruction completion conditions_. These two parts are specified in two separate files, one  
referred as _var-map_ and the other _inst-cond_.


Besides the above two parts, there are other auxiliary information needed. They are:

  - **module naming**: The names of the ILA module and the Verilog module.
  - **global invariants**: Some properties that are globally true for the Verilog design that will be checked separately and can be safely assumed when verifying  individual instructions.
  - **interface signal information**: What does the interface of the Verilog top module look like and what do these signals mean to the tool.
  - **uninterpreted function mapping**: What an uninterpreted function inside the ILA model corresponds to.

### The Structure of _var-map_ and _inst-cond_ ###