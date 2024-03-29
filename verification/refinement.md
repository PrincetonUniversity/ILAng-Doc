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
5. If the ILA variable on the left is a memory, the right side depends on whether the Verilog memory is internal or external. If inside Verilog, there is an array that corresponds to this memory, you can directly use this name on the right side. Otherwise, you can make this ILA memory to map with an external memory using `"ILA-mem-name":"**MEM**ILA-mem-name"`. 

Below are some of the examples:

```javascript
{
  "state mapping": {
    "ILA_state_1" : "Verilog_state_1",                                  // case 1
    "ILA_state_2" : "ILA_state_2 == Verilog_state_2 + Verilog_state_3", // case 2
    "ILA_state_3" : ["__START__", "Verilog_state_3"],                   // case 3.a
    "ILA_state_4" : { "cond":"__START__", "map":"Verilog_state_4"},     // case 3.b
    "ILA_state_5" : [["__START__", "Verilog_state_5"], ["__IEND__","Verilog_state_6"]],     // case 4
    "ILA_mem_state" : "Verilog_array_name",       // case 5.a : internal memory in Verilog
    "ILA_mem_state" : "**MEM**ILA_mem_state",     // case 5.b : external memory in Verilog
    // more ...
  }, 
  // other fields ...
}
```

The corresponding generated assumptions will be in the following form:

```text
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

// map to an internal Verilog array : element-wise equality
assign __m20__ = ( __ILA_SO_mem_0 == VerilogArray_0_)&&( __ILA_SO_mem_1 == VerilogArray_1_)&&... ;
assign variable_map_assume___p11__ = (~ __START__ )|| (__m20__) ;

// map to an external Verilog array : using memory abstraction
// no assumption is needed
```

and the assertions:

```text
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

// map to an internal Verilog array : element-wise equality
assign __m10__ = ( __ILA_SO_mem_0 == VerilogArray_0_)&&( __ILA_SO_mem_1 == VerilogArray_1_)&&... ;
assign variable_map_assert___p11__ = (~ __IEND__ )|| (__m10__) ;

// map to an external Verilog array : using memory abstraction
absmem #( 
    .AW(32),
    .DW(8),
    .TTS(4294967296) )
mi0(...,  .mem_EQ_(mem_EQ_) );
assign variable_map_assert__p13__ = (~ __IEND__) || (mem_EQ_) ;
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
* `**MEM**name.signal` directive, indicating this signal is the connection to an external/shared memory. The name part should be the ILA state variable name of the memory, and the signal part could be one of the following: `wdata`,`rdata`, `waddr`,`raddr`, `wen`, `ren`. If the signal does not directly correspond to the write/read data, write/read address, write/read enable signal, it should be specified as `**KEEP**`, you can specify the mapping using "annotation" which will be dicussed below.

For example, for a Verilog input signal `control`, the effects of applying different directives are:

* An ILA input name, e.g., `c1`. The tool will check if ILA indeed has the input `c1` and the type is matched. The wire `__ILA_I_c1` will be created and will be connected to the input port `control`.
* `**KEEP**` directive. The tool will check if the signal is input or output and create an input/output wire named `__VLG_I_control` with the proper width and connect to the port.
* `**NC**` directive. The tool will have it unconnected like `.control()`.
* `**SO**` directive. The tool will check if it is indeed an output and create an output wire `__VLG_SO_control` and connect to the port. In this example, because it is actually an input, the tool will give warning.
* `**RESET**` or `**NRESET**`. It will be connected as `.control(rst)` or `.control(~rst)`, where `rst` is the reset signal of the wrapper.
* `**CLOCK**` directive. It will be connected as `.control(clk)`, where `clk` is the clock signal of the wrapper.
* `**MEM**name.signal`. The tool will check if `name` is an ILA memory name. It will be connected as `.control(__MEM_name_0_signal)`.

## Notes on Memory State Variable

Memory state variable might be internal or external to the module. An internal memory variable corresponds to a verilog array, and therefore no specific I/O interface is needed to access the memory. An external memory is a memory that connects with the current module via I/O interface. By default, all memory variables in the ILA are treated as external memory. The default setting can be override by _annotation_ in refinement map, and here is an example:

```javascript
  "annotation" : {
    "memory" : {
      "rf":"internal",
      "mem":"external"
    }
  }
```

The above annotation specifies memory named as `rf` and `mem` as internal and external respectively. Being internal or external affects how properties are generated. The mapping of internal memory is element-wise with expressions comparing two verilog arrays entry by entry, which is inefficient for a large memory. The mapping of external memory will use memory abstraction, which is more efficient. \(In the future, we will support mapping internal memory using the Array data-type of the underlying property verifier.\)

For the external memory, it is likely that there isn't a one-to-one mapping between the module interface and the six signals required by the memory abstraction module: `wdata` (write-data) ,`rdata` (read-data), `waddr` (write-address),`raddr` (read-address), `wen` (write-enable), `ren` (read-enable). Therefore, we support specifying the mapping using the annotation here. An example of the syntax is given below.

```javascript
  "annotation" : {
    "memory-ports" : {
      "MemoryName.port1" : "Verilog-expression-here", 
      "MemoryName.port2" : "Verilog-expression-here"
    }
  }
```

In the above example, `MemoryName` should be the name of the memory state variable in ILA and `port1`, `port2` should be one of  `wdata` (write-data) ,`rdata` (read-data), `waddr` (write-address),`raddr` (read-address), `wen` (write-enable), `ren` (read-enable). The Verilog expression after the colon represents how to use the Verilog signal to compute the interface signals.
Note that this kind of mapping assumes the ports of an abstract memory state can be represented as a function of solely the Verilog signals. If this is not feasible for the design you are looking at, you can use the additional mapping capability provided by `mapping control` section described later in this chapter.


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

## Additional Mapping

Sometimes, the mapping between Verilog variables and ILA state variables does not fit easily into the state mapping section. For example, you may need a customized mapping from the Verilog design's memory interface to the provided 6-signal memory interface. The AES case study provides an example of this. The Verilog design uses two signals `stb` and `wr` to indicate memory read and write enable, which are different from the `ren` and `wen` signals. Therefore a mapping can be provided as follows:

```javascript
"mapping control" : [
  "( wr & stb) == __MEM_XRAM_0_wen" , 
  "(~wr & stb) == __MEM_XRAM_0_ren" 
]
```

Assumptions in the `mapping control` section does not appear in the invariant or invariant synthesis target.

## Additional Assumptions

This section allows users to add additional assumptions in the verification. They can be, for example, an assumption about the module I/O.

```javascript
"assumptions" : [
  // the two inputs can not be both 1
  "! ( verilog_module.input1 & verilog_module.input2 )"
]
```

Assumptions in this section are in effect when verifying the instructions, the invariants and when synthesizing the invariants.

## Value Holder

Value holder \(a.k.a prophecy variables, auxiliary variables and etc.\) can be introduced to capture the value of a Verilog variable at a certain time \(or under a certain condition\). Below is an example of a value holder:

```javascript
"value-holder": {
  "r1_pvholder" : {
        "cond": "m1.write_enable == 1",
        "val":"m1.registers[1]",
        "width":8
      }
}
```

The above value holder will be translated to the following Verilog code fragments in the Verilog.

```text
input      [7:0] __r1_pvholder_init__;
output reg      [7:0] r1_pvholder;

always @(posedge clk) begin
   if(rst) begin
       r1_pvholder <= __r1_pvholder_init__;
   end
   else if(1) begin
       r1_pvholder <= r1_pvholder;
   end
end

assume property ((m1.write_enable == 1) |-> ((r1_pvholder) == (m1.registers[1])));
```

It creates an auxiliary Verilog variable `r1_pvholder` which carries a undetermined value. Its value keeps the same all the time, and an assumption says this undetermined value is same as the specified value `m1.registers[1]`, under the condition that `m1.write_enable == 1`. This variables can be referenced in other sections by `#r1_pvholder#` \(this tells the tool not to find this signal name in the original Verilog design. The `width` part can also be a string `"auto"`, which tells the tool to automatically determine the width \(in case of failure, error will be prompted.\) This value holder does not check if there is only one cycle that `m1.write_enable == 1` holds. If there are multiple cycles that its condition holds, the assumption may overconstrain if `m1.registers[1]` should carry different values at these points. This situation should be avoided.

## Verilog Monitor

In the case that user may want to add customized auxiliary variables, we support inline monitors for this purpose.

An example is given as follows:

```javascript
  "verilog-inline-monitors" : {
    "stage_tracker" : {
      "verilog": 
        ["always @(posedge clk) begin",
         "  if (__START__) stage_tracker <= 0;",
         "  else if (__STARTED__ && !__ENDED__) stage_tracker <= stage_tracker + 1;",
         "end"],
      "defs" :[ ["stage_tracker", 2, "reg"] ],
      "refs" :[]
    },
  }
```

This creates a 2-bit variable `stage_tracker` to track the number of cycles \(this is just for demoing the syntax of Verilog monitor, you can in fact use the embedded variable `__CYCLE_CNT__` to track the number of cycles\). Variables that should be accessible outside the monitor should be defined in the `defs` list with its name, bit-width and its type: `reg` or `wire`. Variables used only inside the monitor can be defined inside the `verilog` code list. Any variable inside the original Verilog, if referenced, should be put in the list of `refs`. This will help our tool to add auxiliary wires to connect them with the monitor.

Value holders and monitors are normally only in effect when verifying instructions. If you want a monitor to appear also when verifying the invariants, you can add the `"keep-for-invariants":true` attribute in the monitor's description following the `defs` and `refs` attributes.

