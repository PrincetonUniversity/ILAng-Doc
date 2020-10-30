# Examples of Refinement Relation

Here we'd like to give some examples of the refinement relations. 
The features described may include some of the syntactic sugar we are currently working on,
which will be marked in the text.

## Conditional Mapping

In variable mapping section, the conditional mapping is helpful in many cases. Below we give some
examples using conditional mapping.

### Mapping Loop Body with Unrolled Pipelines

Suppose we have a specification for AES-128 function, which consist of 10 rounds of cryptographic operations, each round takes results from previous round and generate new ciphertext and new round key which will be used in the next round.
In the ILA model, this is described with the help of child-instructions that form a loop.
Whereas, the implementation is a pipeline which corresponds to an unrolling of the loop in the specification.
This is illustrated in the figure below.

![](.gitbook/assets/aes-128-loop.png)

When verifying the child-instructions, we need to map the state variables holding intermediate results \(suppose it is named as `ciphertext` in ILA\) with signals in Verilog.
However, because the loop is unrolled into a 10-stage pipeline, the corresponding Verilog signal for `ciphertext` is dependent on the round. Therefore, we need conditional mapping here, which can be specified as follows.


```javascript
  "state mapping": {
    "ciphertext" : [["ILA.round == 0", "s_in" ],
                    ["ILA.round == 1", "s0"   ],
                    ["ILA.round == 2", "s1"   ],
                    ...
                   ]
  }
```

Or, alternatively, you can use:

```javascript
  "state mapping": {
    "ciphertext" : [{"cond":"ILA.round == 0", "map":"s_in" },
                    {"cond":"ILA.round == 1", "map":"s0"   },
                    {"cond":"ILA.round == 2", "map":"s1"   },
                    ...
                   ]
  }
```

Regarding the `round` variable in ILA, because  the loop in Verilog is fully unrolled and the effects of `round` already appear in the mapping of `ciphertext`, we don't need a mapping for it. We can simply write:

```javascript
  "state mapping": {
    "round" : null,
    ...
  }
```


{% hint style="tip" %}
The condition specified the first has the highest priority. You can have a default case with condition `1'b1`.
{% endhint %}


### Mapping State Variables with On-the-Fly Values

When tackling pipelines, it is also





{% hint style="working" %}
The syntactic sugar `signal@condition` is a feature we are working on and is yet to be merged into the main branch.
{% endhint %}



% stage tracker
% delay monitor
% @ time (condition)
