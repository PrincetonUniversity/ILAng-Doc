# Hierarchical ILA

## Defining child ILAs

Defining a child ILA is basically defining a normal ILA model, except that the object should be created from the parent ILA to inherit and link the state variables from its parent. 

```cpp
auto child = parent.NewChild("child_name");
```

### Architectural states

The input and state variables from the parent ILA can be accessed by the child ILA in a similar way.

```cpp
auto in_addr = child.input("InAddr");
auto buffer = child.state("Buffer");
```

## Accessing child ILAs

The child ILAs can be accessed by their name.

```cpp
auto child = m.child("child_name");
```

It can also be enumerated.

```cpp
for (auto i = 0; i < m.child_num(); i++) {
    auto ith_child = m.child(i);
}
```

