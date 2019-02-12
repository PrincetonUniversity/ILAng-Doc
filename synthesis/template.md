# Writing Templates

A synthesis template defines the ILA model partially where the state update functions of instructions can be left not-fully specified. That is, based on the set of architectural states and the set of instructions, the template defines the possible state updates from which the synthesis engine determines the exact solution. 

### Python module

After [building the synthesis engine](../start/python.md), the Python library can be imported to write a template.

```python
#!/bin/python
import ila
m = ila.Abstraction("test")
```

### Defining architectural states

To define the set of state variables that are persistent across instructions, the ILA synthesis engine requires users to specify the set of architecture states and inputs. To declare an input:

```python
cmd = m.inp('Cmd', 2)
```

State variables can be declared similarly:

```python
acc_status = m.reg('AccStatus', 3) # bit-vector type
buffer = m.mem('buffer', 32, 8)    # memory/array type
```

### Defining instructions

The synthesis process is performed instruction by instruction. That is, the engine determines the state update functions of all architectural states for one instruction at a time. Therefore, the template requires the decode functions of each instruction.

```python
# a list of decode functions (Boolean type)
wrcmds = [ (cmd == 2) & (in_addr == addr) for addr in xrange(0xff00, 0xff40) ]
m.decode_exprs =  wrcmds
```

### Normal AST expressions

Constants can be specified either directly as standard Python terms \(for Boolean and bit-vectors\) or:

```python
const_2bit_zero = m.const(0, 2)
```

Logical, arithmetic, and comparison operations can be applied using the overloaded Python operators.

```python
is_write = (cmd == 2)
is_write_to_config = is_write & (in_addr == 0xff00)
```

To represent if-then-else condition:

```python
guarded_value = ila.ite(counter + 1 > 2, 0x0, prev_value * 2)
```

For memory access, you can load and store an element or even a block:

```python
value = mem[0x2]
load_a_block = ila.loadblk(mem, start_addr, 16)
new_mem = ila.store(mem, 0xff, 0x34)
store_a_block = ila.storeblk(mem, start_addr, data)
```

### Synthesis primitives

The key power of the synthesis engine is to allow users to specify only the skeleton of the state update functions and to finalize the exact expression automatically. To enable this, it provides a set of _synthesis primitives_ to specify the candidate update functions \(in terms of AST\). 

**choice** A list of expression candidates within which the synthesis engine searches for the right choice. All elements should have the same type. Note that duplicated \(identical\) elements in the list may cause the synthesis process to fail since it cannot by distinguished by mutating the input signals.  

```python
src = ila.choice('any_id_for_debug', [eax, ebx, ecx, m.const(0x0, 32)])
```

**inrange** A numeric range of values within which the synthesis engine searches for the right constant. The AST node after synthesis is a bit-vector type constant. 

```python
offset = ila.inrange('any_id_for_debug', m.const(0x0, 32), m.const(0xffff, 32))
```

**readslice** To read a slice from a bit-vector type expression, starting from any bit. By providing the source bit-vector and the bit length to read, the result after synthesis is an operator node that reads a fixed shift from a bit-vector. 

```python
counter = ila.readslice('any_id_for_debug', source_reg, 8)
```

**writeslice** To overwrite a value onto a bit-vector type expression, starting from any bit. By providing the destination bit-vector and the value, the result after synthesis is an operator node that writes a fixed shift over a bit-vector.

```python
new_reg = ila.writeslice('any_id_for_debug', source_reg, value_reg[7:0])
```

More details description and tutorial can be found in [conclusion](example.md). 

