# Instructions

## Defining instructions

To define an instruction of the ILA model `m`: 

```cpp
auto instr = m.NewInstr("RD/WR_ADDR");
```

### Decode function

The decode function takes in the opcode and return a Boolean type output, indicating whether an instruction is _issued_. It can be set to an Boolean type expression:

```cpp
instr.SetDecode(decode_expr);
```

