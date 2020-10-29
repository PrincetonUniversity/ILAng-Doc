# ILAng with Python

## ILAng Python interface

{% hint style="info" %}
 The Python API will be deprecated and thus no future official support.
{% endhint %}

## Synthesis engine

As part of the ILAng platform, [ItSy](https://github.com/PrincetonUniversity/ItSy) \(ILA synthesis engine\) provides a Python API for writing synthesis templates, interfacing simulator, and scripting the synthesis process. To build the Python module:

```bash
cd itsy/root/dir
bjam -j$(nproc)
```

The Python module `ila.so` will be available in the `build` directory. Run or add the following command in your `.bashrc` file to set the path: 

```bash
export PYTHONPATH=$(pwd)/build:$PYTHONPATH
```

Once the path is set properly, you can then access the synthesis Python module:

```python
import ila
abs = ila.Abstraction("test")
```

