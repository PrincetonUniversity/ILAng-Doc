# Synthesis Results

Once the synthesis process finishes successfully, all partially defined AST expressions \(synthesis primitives\) will be replaced by a concrete normal AST expression.

### Export the synthesis result

The synthesis result can be exported and archived into a machine-readable format. To export the whole model:

```python
m.exportAll('file_name.txt')
```

To export only the state update function of one single state:

```python
m.exportOne(m.get_next('state_var_name'), 'file_name.txt')
```

{% hint style="warning" %}
When exported, the state update function \(expression\) is the conjunction of the state update functions under all provided instructions. 
{% endhint %}

### Import the synthesis result

To import the whole model from a file:

```python
m = ila.Abstraction('placholder_name')
m.importAll('file_name.txt')
```

You can also import a single state update function \(or any expression\):

```python
some_expression = m.importOne('file_name.txt')
```

### Synthesis result in ILAng

To utilize the synthesis result \(from Python environment\) in your C++ project using ILAng, an ILA model can be automatically constructed from the exported result:

```cpp
#include <ilang/synth-interface/synth_engine_interface.h>
auto m = ilang::ImportSynthAbsFromFile("result.txt", "ila_name");
```

