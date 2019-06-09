# Refinement Relation


The refinement map or refinement relation mainly consists of two parts: _variable mappings_ and 
_instruction completion conditions_. These two parts are specified in two separate files, one  
referred as _var-map_ and the other _inst-cond_.


Besides the above two parts, there are other auxiliary information needed. They are:

  - **module naming**: The names of the ILA module and the Verilog module.
  - **global invariants**: Some properties that are globally true for the Verilog design that will be checked separately and can be safely assumed when verifying  individual instructions.
  - **interface signal information**: What does the interface of the Verilog top module look like and what do these signals mean to the tool.
  - **uninterpreted function mapping**: What an uninterpreted function inside the ILA model corresponds to.

### The Structure of _var-map_ ###


```json
{
  "models": { 
    "ILA" : "<module-name>" , 
    "VERILOG": "<module-name>" },
  "state mapping": {
    "<ILA-variable-name>" : "<Verilog-signal>",
    // more ...
    },
  "interface mapping": {
    "<Verilog-io-signal>" : "<ILA-variable-name/directives>",
    // more ...
    },  
  "mapping control": [
    "<Verilog-expression>",
    // more ...
    ],
  "functions":{
    "<function-name>":
      [
        [
          "<Verilog-expression>", "<Verilog-signal>",
          "<Verilog-expression>", "<Verilog-signal>",
          // more ...
        ]
      ]
  }
}
```

### The Structure of _inst-cond_ ###

```json
{
  "instructions":
  [
    {
      "instruction"     :"<instr-name>",
      "ready bound"     :12345, // <bound:integer>
      "ready signal"    :"<Verilog-signal>",
      "max bound"       :12345, // <bound:integer>
      "start condition" :"<Verilog-expr>",
      "flush constraint":"<Verilog-expr>",
      "pre-flush end"   :"<Verilog-expr>",
      "post-flush end"  :"<Verilog-expr>"
    }
  ]
}
```


### Module Naming ###

The module naming section comes first in the _var-map_ JSON file. It is a dictionary (map) with two elements. One element should have the key `ILA` and the value of it is what will be used as the instance name of the ILA module. The other element should have the key `VERILOG` with the value to be the instance name of the Verilog module. These names are used in all the expressions later when you want to refer to a variable in Verilog or ILA.

### Variable Mapping ###

The variable mapping in the JSON file is a map data structure. The keys of this map are the state variable names in the ILA model while the values are the Verilog variables. There are cases that one Verilog state variable can be mapped to multiple Verilog state variables and the mapping may be some special function. So the allowed value field of this map can be:

  1. A Verilog variable name as a string. If the Verilog variable is in the top module of Verilog, you can omit the module name (it does not hurt to add it). Otherwise, you must specify the complete hierarchy. For example, if you want to refer to a signal \texttt{S} in module \texttt{A} that is instantiated with name \texttt{IA} in the top module, while the top module's instance name is \texttt{VERILOG} (specified in the previous section), then you should use \texttt{VERILOG.IA.S} to refer to it.

  2. A predicate that has at least a `=` in it (you can use `>=` , `<=` , `==` and etc.) This is just for our parser to distinguish it from the first case

  3. A string-string pair that is in the form of a list or map. If it is given as a list, the first element is regarded as the condition and the second element is regarded as the mapping. It conveys the meaning of "when the condition is true, the ILA and Verilog variables should have the mapping". If the condition is not true, there is no mapping guaranteed. If the string-string pair is given as a map, it must have two elements. One element should have the key `cond` and the other should have the  key `map`. The `map` part could be a string of Verilog variable name as in case (1) or a Verilog expression as case (2).

  4. A list of string-string pairs, each pair follows requirement in (3).

Below are some of the examples:
```json
{
  "state mapping": {
    "ILA_state_1" : "Verilog_state_1",                                  // case 1
    "ILA_state_2" : "ILA_state_2 == Verilog_state_2 + Verilog_state_3", // case 2
    "ILA_state_3" : ["__START__", "Verilog_state_3"],                   // case 3.a
    "ILA_state_4" : { "cond":"__START__", "map":"Verilog_state_4"},     // case 3.b
    "ILA_state_5" : [["__START__", "Verilog_state_5"], ["__IEND__","Verilog_state_6"]],     // case 4
    // more ...
    }, 
    // other fields ...
}
```

One note in the above example: the condition can refer to special signals (`__START__ ` and `__IEND__`), which are the condition when the instruction-under-verification starts to execute and finishes.

### Instruction Completion Condition ###


The instruction completion condition is specified per instruction. In the JSON file, it is a list of maps. The list does not need to be a full list of instructions. Those not in the list will not be verified. Each dictionary must have an element whose key is `instruction` and the value of this element is the name of the instruction in the ILA model. Besides this name element, it must contain one of the `ready bound` or the `ready signal` element. The `ready bound`  specifies a bound that the instruction takes. It is used for instructions that take a fixed number of cycles. Alternatively, one can provide a signal (or a predicate) in the `ready signal` field.

There are several optional fields. They are `start condition`, `max bound`, `flush constraint`, `pre flush-end` and `post flush-end`. 

The `start condition` field, if provided, should be a list of strings. Each string is a Verilog expression that acts as a predicate on the Verilog design. It can be used to constrain the Verilog implementation state, because usually there are more microarchitectural (implementation states) in the design. For the starting state of an instruction, these microarchitectural states should be `consistent` with the visible states in the ILA, and you can use this field to enforce the `consistency` at your discretion. You can use `$decode$` and `$valid$` to refer to the instruction's decode function or its ILA valid condition.

The `max bound` can be used when `ready signal` field is provided. It provides an assumption that the condition that the `ready signal` field specified will occur in the given number of cycles, and this will limit the model checking to that bound. Usually this is used for the additional environment assumptions about the input (for example, how many cycles at most there are for a request to be served with a response).

The `flush constraint`, `pre flush-end` and `post flush-end` signals are used when using the `flushing verification setting`. For this verification setting, you can refer to \href{https://arxiv.org/abs/1801.01114}{our paper} on the ILA-based verification or the \href{https://dl.acm.org/citation.cfm?id=735662}{Burch-Dill's approach} on processor verification.

