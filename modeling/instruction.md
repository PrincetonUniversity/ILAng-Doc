# Instructions

## Defining instructions

To define an instruction of the ILA model `m`: 

```cpp
auto instr = m.NewInstr("RD/WR_ADDR");
```

### Decode function

The decode function takes in the opcode and return a Boolean type output, indicating whether an instruction is _issued_. It can be set to an Boolean type expression:

```cpp
instr.SetDecode(InAddr == 0xFF02);
```

### State update functions

Each instruction contains the state update functions of every state variables \(**unchanged if not specified**\). The update function can be set to an expression of the same type:

```cpp
instr.SetUpdate(Addr, ilang::Ite(Cmd == 1, InData, Addr));
instr.SetUpdate(OutData, ilang::Ite(Cmd == 0, Addr, 0x0));
```

