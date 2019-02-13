# Expressions

## Abstract syntax tree \(AST\) expressions

Expressions are the nodes of the AST where input and state variables being the leaf nodes. ILAng provides the interface to define and construct the AST expressions based on the set of operators supported in SMT LIB2. Basic type checks are performed at run time. 

### Constant

To declare a constant Boolean and bit-vector expression is easy:

```cpp
auto const_false = ilang::BoolConst(false);
auto const_8bit_255 = ilang::BvConst(0xFF, 8);
```

Besides key and element bit-width, a constant memory consists of a _default value_ and a set of _key-element pairs_. The below example creates a constant memory where all elements have value `0xFF` except for key `2` being mapped to `0x4`. The memory address and data are `32` and `8` bit wide, respectively. 

```cpp
auto const_mem = ilang::MemConst(0xFF, {(0x2, 0x04)}, 32, 8);
```

### Logical operation

Logic operations such as AND, OR, and NOT can be applied to Boolean and bit-vector type expressions \(need to have the same type\). Note that the operations on bit-vectors are bit-wise. 

```cpp
auto x_and_y = x & y;
auto not_not_x = !(x ^ true);
auto x_imply_y = ilang::Imply(x, y);

auto bvx_nor_bvy = ~(bvx | bvy);
auto bvx_and_one = bvx & 0x1;
```

### Arithmetic operation

ILAng also supports arithmetic operations for bit-vector type expressions. Note that arithmetic operations are signed by default. 

```cpp
auto x_increment = x + 0x1; 
auto x_mul_2 = x << 1;
auto x_arith_right_shift = x >> 2;
auto x_logic_right_shift = ilang::Lshr(x, y);
```

For bit-vectors, there are also several bit-wise operations available:

```cpp
auto first_bit_of_x = ilang::SelectBit(x, 0);
auto low_3_bit_of_x = ilang::Extract(x, 2, 0);
auto zero_ext_x = ilang::ZExt(x, 32);
auto concat_x_y = ilang::Concat(x, y);
```

### Binary comparison 

With two expressions of the same type, you can compare logically or arithmetically:

```cpp
auto x_equal_y = (x == y);
auto x_less_than_y = (x < y);
auto x_is_positive = (x >= 0x0);
auto x_unsigned_greater_then_or_equal_to_y = ilang::Uge(x, y);
```

Note that, by default, comparison are singed for bit-vectors. There is always an unsigned correspondent. 

### Memory operations

To load and store memory values:

```cpp
auto value_at_addr_x = ilang::Load(mem, x);
auto value_at_addr_2 = ilang::Load(const_mem, 0x2);
auto update_memory_at_x = ilang::Store(const_mem, x, y);
auto update_memory_at_2 = ilang::Store(mem, 0x2, 0x5);
```

### If-then-else

To represent conditional expressions, the if-then-else operator takes as inputs one Boolean expression \(condition\) and two branch expressions of the same type. 

```cpp
auto increment_x_if_y_true = ilang::Ite(y, x + 1, x);
```

