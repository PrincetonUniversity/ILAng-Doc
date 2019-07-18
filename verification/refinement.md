# Refinement Relation

The refinement map or refinement relation mainly consists of two parts: _variable mappings_ and _instruction completion conditions_. These two parts are specified in two separate files, one referred as _var-map_ and the other _inst-cond_.

Besides the above two parts, there are other auxiliary information needed. They are:

* **module naming**: The names of the ILA module and the Verilog module.
* **global invariants**: Some properties that are globally true for the Verilog design that will be checked separately and can be safely assumed when verifying  individual instructions.
* **interface signal information**: What does the interface of the Verilog top module look like and what do these signals mean to the tool.
* **uninterpreted function mapping**: What an uninterpreted function inside the ILA model corresponds to.

## The Structure of Variable Mappings

```javascript
{
  "models": { 
    "ILA" : "<module-name>" , 
    "VERILOG": "<module-name>" 
  },
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
    "<function-name>": [
      [
        "<Verilog-expression>", "<Verilog-signal>",
        "<Verilog-expression>", "<Verilog-signal>",
        // more ...
      ]
    ]
  }
}
```

## The Structure of Instruction Completion Conditions

```javascript
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

## Module Naming

The module naming section comes first in the _var-map_ JSON file. It is a dictionary \(map\) with two elements. One element should have the key `ILA` and the value of it is what will be used as the instance name of the ILA module. The other element should have the key `VERILOG` with the value to be the instance name of the Verilog module. These names are used in all the expressions later when you want to refer to a variable in Verilog or ILA.

## Variable Mapping

The variable mapping in the JSON file is a map data structure. The keys of this map are the state variable names in the ILA model while the values are the Verilog variables. There are cases that one Verilog state variable can be mapped to multiple Verilog state variables and the mapping may be some special function. So the allowed value field of this map can be:

1. A Verilog variable name as a string. If the Verilog variable is in the top module of Verilog, you can omit the module name \(it does not hurt to add it\). Otherwise, you must specify the complete hierarchy. For example, if you want to refer to a signal `S` in module `A` that is instantiated with name `IA` in the top module, while the top module's instance name is `VERILOG` \(specified in the previous section\), then you should use `VERILOG.IA.S` to refer to it.
2. A predicate that has at least a `=` in it \(you can use `>=` , `<=` , `==` and etc.\) This is just for our parser to distinguish it from the first case.
3. A string-string pair that is in the form of a list or map. If it is given as a list, the first element is regarded as the condition and the second element is regarded as the mapping. It conveys the meaning of "when the condition is true, the ILA and Verilog variables should have the mapping". If the condition is not true, there is no mapping guaranteed. If the string-string pair is given as a map, it must have two elements. One element should have the key `cond` and the other should have the  key `map`. The `map` part could be a string of Verilog variable name as in case \(1\) or a Verilog expression as case \(2\).
4. A list of string-string pairs, each pair follows requirement in \(3\).

Below are some of the examples:

```javascript
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

The corresponding generated assumptions will be in the following form:
```verilog
wire __m10__ = Verilog_state_1 == __ILA_SO_ILA_state_1 ;
wire variable_map_assume___p1__ILA_state_1 = (~ __START__) || (__m10__) ;  // __START__ --> Verilog_state_1 == __ILA_SO_ILA_state_1

wire __m12__ = Verilog_state_2 + Verilog_state_3 == ILA_state_2 ;
wire variable_map_assume___p3__ILA_state_2 = (~ __START__) || (__m12__) ;

wire __m14__ = (~ __START__ ) ||  ( Verilog_state_3 == ILA_state_3 ) ;
wire variable_map_assume___p5__ILA_state_3 = (~ __START__) || (__m14__) ;

wire __m16__ = (~ __START__ ) ||  ( Verilog_state_4 == ILA_state_4 ) ;
wire variable_map_assume___p7__ILA_state_4 = (~ __START__) || (__m16__) ;

wire __m18__ = ( (~ __START__ ) ||  ( Verilog_state_5 == ILA_state_5 ) ) && ( ~( ~__START__ && __IEND__ ) ||  Verilog_state_6 == ILA_state_5 ) ; // __START__ --> match1 && (~__START__ && __IEND__) --> match2
wire variable_map_assume___p9__ILA_state_5 = (~ __START__) || (__m18__) ;
```

and the assertions:
```verilog
wire __m0__ = Verilog_state_1 == __ILA_SO_ILA_state_1 ;
wire variable_map_assert___p1__ILA_state_1 = (~ __IEND__) || (__m0__) ;  // IEND --> Verilog_state_1 == __ILA_SO_ILA_state_1

wire __m2__ = Verilog_state_2 + Verilog_state_3 == ILA_state_2 ;
wire variable_map_assert___p3__ILA_state_2 = (~ __IEND__) || (__m2__) ;

wire __m4__ = (~ __START__ ) ||  ( Verilog_state_3 == ILA_state_3 ) ;
wire variable_map_assert___p5__ILA_state_3 = (~ __IEND__) || (__m4__) ;

wire __m6__ = (~ __START__ ) ||  ( Verilog_state_4 == ILA_state_4 ) ;
wire variable_map_assert___p7__ILA_state_4 = (~ __IEND__) || (__m6__) ;

wire __m8__ = ( (~ __START__ ) ||  ( Verilog_state_5 == ILA_state_5 ) ) && ( ~( ~__START__ && __IEND__ ) ||  Verilog_state_6 == ILA_state_5 ) ; // __START__ --> match1 && (~__START__ && __IEND__) --> match2
wire variable_map_assert___p9__ILA_state_5 = (~ __IEND__) || (__m8__) ;
```

One note in the above example: the condition can refer to special signals \(`__START__` and `__IEND__`\), which are the condition when the instruction-under-verification starts to execute and finishes.

## Instruction Completion Condition

The instruction completion condition is specified per instruction. In the JSON file, it is a list of maps. The list does not need to be a full list of instructions. Those not in the list will not be verified. Each dictionary must have an element whose key is `instruction` and the value of this element is the name of the instruction in the ILA model. Besides this name element, it must contain one of the `ready bound` or the `ready signal` element. The `ready bound` specifies a bound that the instruction takes. It is used for instructions that take a fixed number of cycles. Alternatively, one can provide a signal \(or a predicate\) in the `ready signal` field.

There are several optional fields. They are `start condition`, `max bound`, `flush constraint`, `pre-flush end` and `post-flush end`.

The `start condition` field, if provided, should be a list of strings. Each string is a Verilog expression that acts as a predicate on the Verilog design. It can be used to constrain the Verilog implementation state, because usually there are more microarchitectural \(implementation states\) in the design. For the starting state of an instruction, these microarchitectural states should be `consistent` with the visible states in the ILA, and you can use this field to enforce the `consistency` at your discretion. You can use `$decode$` and `$valid$` to refer to the instruction's decode function or its ILA valid condition.

The `max bound` can be used when `ready signal` field is provided. It provides an assumption that the condition that the `ready signal` field specified will occur in the given number of cycles, and this will limit the model checking to that bound. Usually this is used for the additional environment assumptions about the input \(for example, how many cycles at most there are for a request to be served with a response\).

The `flush constraint`, `pre-flush end` and `post-flush end` signals are used when using the flushing verification setting. For this verification setting, you can refer to [TODAES19](https://bo-yuan-huang.github.io/ILAng-Doc/todaes18.pdf) on the ILA-based verification or the [Burch-Dill's approach](https://dl.acm.org/citation.cfm?id=735662) on processor verification.

## Global Invariants

In the verification of instructions, we do not assume the design starts from the initial states. This helps us to get a better guarantee of the instruction correctness when only bounded model checking is used. However, if there is no constraints on the starting state of an instruction, there might be spurious bugs just because the design starts from a state that it will never reach when started from the reset state. In order to avoid this false positive, we use global invariants to constrain on the starting state. These invariants help rule out some unreachable states and the tool will generate a separate target to check whether the provided invariants are globally true or not. These invariants should be provided as a list of strings, where each string is a Verilog predicate. In the future, we will exploit invariant synthesis techniques to help synthesize these invariants.

## Interface Signal Information

The Verilog module comes with a set of I/O signals and the tool needs to know how these signals should be connected. The `interface mapping` field is a map whose keys are the Verilog I/O signal names and whose values can be one of the following:

* An ILA input name. This means that the Verilog input signal corresponds to one ILA input. They must have the same encoding and bit-width.
* `**KEEP**` directive. Telling the tool to have a wire of the same name and to connect it as the verification wrapper I/O.
* `**NC**` directive, indicating that this port does not need to be connected.
* `**SO**` directive, indicating that this is actually a direct output from a visible state variable \(a state variable that is modeled in the ILA\).
* `**RESET**` or `**NRESET**` directive. Indicating that this signal is the reset signal, active-high or active-low \(we assume synchronous reset\).
* `**CLOCK**` directive, indicating that this is the clock signal.
* `**MEM**name.signal` directive, indicating this signal is the connection to an external/shared memory. The name part should be the ILA state variable name of the memory, and the signal part could be one of the following: `wdata`,`rdata`, `waddr`,`raddr`, `wen`, `ren`. If the signal does not directly correspond to the write/read data, write/read address, write/read enable signal, it should be specified as `**KEEP**`, you can specify the mapping using the additional assumptions.

For example, for a Verilog input signal `control`, the effects of applying different directives are:

  * An ILA input name, e.g., `c1`. The tool will check if ILA indeed has the input `c1` and the type is matched. The wire `__ILA_I_c1` will be created and will be connected to the input port `control`.
  *  `**KEEP**` directive. The tool will check if the signal is input or output and create an input/output wire named `__VLG_I_control` with the proper width and connect to the port.
  * `**NC**` directive. The tool will have it unconnected like `.control()`.
  * `**SO**` directive. The tool will check if it is indeed an output and create an output wire `__VLG_SO_control` and connect to the port. In this example, because it is actually an input, the tool will give warning.
  * `**RESET**` or `**NRESET**`. It will be connected as `.control(rst)` or `.control(~rst)`, where `rst` is the reset signal of the wrapper.
  * `**CLOCK**` directive. It will be connected as `.control(clk)`, where `clk` is the clock signal of the wrapper.
  * `**MEM**name.signal`. The tool will check if `name` is an ILA memory name. It will be connected as `.control(__MEM_name_0_signal)`.

## Uninterpreted Function Mapping

The ILA model may use uninterpreted functions, however, in the verification, the tool must know the correspondence of this uninterpreted function in order to reason about the correctness. In the `functions` field of the _var-map_ JSON, a map should be provided if uninterpreted function is used in the ILA model. The keys of the map are the function names, with the values in a list. Each element of this list is for one application of this function. Per each function application, the tool needs to know the correspondence of the arguments/result in the Verilog and when that happens, this is like a condition/mapping pair, and it is specified as a list. The correspondence of the function result goes first followed by the arguments \(in the same order as the arguments in ILA function definitions\).

For example in the ILA there is such specification

```cpp
  auto div = FuncRef("div", SortRef::BV(8), SortRef::BV(8), SortRef::BV(8) );
  instr.SetUpdate(state0, div(arg0, arg1) );
```

Then in the refinement map

```javascript
{
  "functions" : {
    "div" : [ 
      ["cond1", "verilog-signal-result", "cond2", "verilog-signal-arg0", "cond3", "verilog-signal-arg1", ] 
    ]
  }
}
```

## Additional Assumptions

This section allows users to add additional assumptions in the verification. They can be, for example:

* An assumption about the module I/O.
* A mapping from the Verilog design's memory interface to the provided 6-signal memory interface. The AES case study provides an example of this. The Verilog design uses two signals  `stb` and `wr` to indicate memory read and write enable, which are different from the `ren` and `wen` signals. Therefore a mapping is provided as follows:

```javascript
"mapping control" : [
  "( wr & stb) == __MEM_XRAM_0_wen" , 
  "(~wr & stb) == __MEM_XRAM_0_ren" 
]
```

