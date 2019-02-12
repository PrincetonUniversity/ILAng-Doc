# Interfacing Simulators

In the CEGIS loop of the synthesis process, a design simulator is required to serve as the oracle. Candidates in the partially defined AST are distinguished based on the answer from the oracle. In this section, you will learn how to wrap your simulator in any form and plug it in the synthesis loop. 

## Simulator interface

The interfaced simulator should provide a one-step simulation of the design. 

### **Input**

The input is the current state represented as a Python `dict` of the state-value pair. The key is the state variable name \(with exact match\) and the element is the value of that state variable/input signal.

```python
current_state = {'reg0' : 0, 'reg1' : 1, 'Cmd' : 2}
```

### Output

The output is the next state after stepping through one instruction \(or child-instruction\). Similarly, it should be represented as a Python `dict` where the variable name is the key. Note that the output should not include the value of input signals. 

```python
next_state = {'reg0' : 1, 'reg1' : 1}
```

### Simulate

The simulation object should then be passed to the synthesis engine.

```python
sim = lambda s: simulator_wrap().one_step(s)
m.synthesize('state_var_name', sim)
```

