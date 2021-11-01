# Refinement Relation

## General Information

Refinement relation is specified in JSON format and is provided as two files, one referred to as  _variable mappings_ and the other as _instruction completion conditions_.
The JSON format allows specifying data structures like map and list and support data-types like string, integer and `null` (for empty field).

A map (similar to a dictionary in Python) has the following syntax:
```javascript
{
  // a map is a set of key-value pairs
  // the key must be string in ""
  // the value can be anything
  "a-string-for-key" : "a-string-for-value",
  "a-string-for-another-key" : 123,
  "a-string-for-a-third-key" : ...  
}
```

A list is an ordered set of elements, where each element can be any string, integer, or nested data structures. JSON format does not require the elements of a list to have the same type.
The top-level of a JSON file is a map and we call the key-value pairs in this top-level map as sections.

## The Structure of Variable Mapping

Below shows the general structure of the variable mapping file (or in short, var-map). Each section (like `state mapping"`) will be explained in details later. Section `additional mapping`, `assumptions`, `monitor` and `functions` are optional. Section names are case-insensitive and you can always use hypen (`-`) to replace space in the section names.

```javascript
{
  "state mapping": {
    "<ILA-state-variable-name>" : "<refinement-expression>",
    // more ...
  },
  "input mapping": {
    "<ILA-input-variable-name>" : "<refinement-expression>",
    // more ...
  },
  "RTL interface connection" :{
    "CLOCK" : {
      "<clk-name-1>" : "<rtl-input-signal-1>", // an input or a list of inputs
      "<clk-name-2>" : ["<rtl-input-signal-2>", "<rtl-input-signal-3>"] },
    "RESET" : "<rtl-input-signal-1>",  // an input or a list of inputs
    "NRESET" : "<rtl-input-signal-2>",  // an input or a list of inputs
    "CUSTOMRESET" : "<rtl-input-signal-2>",  // an input or a list of inputs
  }
  "additional mapping": [
    "<condition>",
    // more ...
  ],
  "assumptions": [
    "<condition>",
    // more ...
  ],
  "monitor" : {
    "<name-1>" : {
      "template" : "phase tracker",
      ...
      // instantiate a phase tracker template
      // each template has its other fields
      // to fill in
    },
    "<name-2>" : {
      "template" : "value recorder",
      ...
      // instantiate a value recorder
      // c,v,w

    },
    "<name-3>" : {
      "verilog" : "<plain-verilog-here>",
      ...
      // directly writing verilog here
    },
    "<name-4>" : "<refinement-expression>"
    ...
  }
  "functions":{ // for uninterpreted functions 
    "<function-name>": [ // list of invocation
      { // an invocation is a map with `result` and `arg`
        "result" : "<refinement-expression>",
        "args" : { "<refinement-expression>", ...} // args
        // more ...
      },
      ...
    ]
  }
}
```


## Refinement Expression, Condition and Signal Names

In the above general structure, you can find terms like "refinement expression", "condition" or "signal name". Refinement expressions are basically valid Verilog expressions with some minor differences. We introduce new syntax for *value recorder*, *delay expression* and *phase identifiers*, and these will be explained in the later sections of this document.
The Verilog expressions can be interpreted as either (1) the expression of a function which specifies how ILA architectural state variables can be computed from the Verilog signals or (2) a relation between ILA architectural state variable and RTL signals. The difference in the interpretation is determined by whether the expression is a predicate in the last level. For example, a expression `RTL.v1 +  RTL.v2` is not a predicate, whereas `ILA.v1 == RTL.v1 + RTL.v2` is a predicate. Whether the refinement expression is regarded as a predicate is independent from the bit-width. For example, `RTL.one_bit_signal` is not a predicate, even it is one-bit wide, whereas `RTL.one_bit_signal == 1'b1` is a predicate.

"Condition" is similar to a "refinement expression" as it follows the same syntactic requirements. However, the type or the outcome of a condition must be 1-bit. Even if it is not in the form of a predicate, it will be interpreted as a predicate (by implicitly adding `xxx == 1'b1` in the top-level).

"Signal name" refers to a single signal (in RTL) and you cannot have any computation there.


## Special Syntax in Refinement Expressions

### Value Recorder

Value holder is one special kind of auxiliary variables and can be introduced to capture the value of a Verilog variable at a certain time or under a certain condition. This is different from directly creating an auxiliary register in RTL. The value of a RTL register is not available for use before it is assigned under the condition,
whereas, the content of a value holder is globally available even before the clock cycle where the specified condition is true. This is achieved with the help of assumptions.

The following syntax is used to make a value recorder: `value @ condition`, where each of the value and condition part can be RTL signals or expressions. If either the `value` or `condition` is more than a signal name, a pair of parantheses is needed, like `( RTL.v1 + RTL.v2 ) @ RTL.s1.commit`.
Nested value recorders (value recorders in `value` or `condition`) are not allowed.

### Simple Delay

You can delay a signal by a fixed number of cycles using `signal ## cycle`. For example, if you need the value of signal `RTL.v1`  one cycle after `RTL.s1.commit`  becomes true for mapping, then you can write `RTL.v1 @ (RTL.s1.commit ## 1)`

### Phase Trackers

If the RTL executes the instructions in multiple phases (e.g., pipeline stages), you will need to declare these phases in order to track the progression of the instruction and to find out its time of finish. This requires using phases trackers.
By default, there are two built-in phase trackers: `#decode#` and `#commit#`. 
It is recommended to wrapped the phase trackers in a pair of `#`.
The specification of stages will be explain [later](#template-for-phase-identifier).

## Module Naming

When you refer to signal in RTL, you can use something like `RTL.module_instance_name1.module_instance_name2.signal`  to refer to it. And if you use a variable (either input or state variable) in ILA, you can use the format `ILA.var_name`.

## State Mapping and Input Mapping

The state mapping in the JSON file is a map data structure. The keys of this map are the state variable names in the ILA model while the value of the map can be (1) a refinement expression, (2) a list of condition-refinement expression pairs for conditional mapping (a pair is written as a list of two elements)  (3) a memory expression for mapping internal array in RTL, or (4) a map following a pre-defined template for external memory. And below shows the syntactic structure of state mapping:

```javascript
{
  "state mapping": {
    "ILA_bitvec_state_var_1" : "<refinement-expression>", // case 1
    "ILA_bitvec_state_var_2" : [
      ["<condition-1>", "<refinement-expression-1>"], 
      ["<condition-2>", "<refinement-expression-2>"]],   // case 2
    "ILA_mem_state_var_1" : "<array-expression>",        // case 3 : internal RTL memory
    "ILA_mem_state_var_2" : {                            // case 4 : external RTL memory
      "wen"   : "<refinement-expression>",
      "waddr" : "<refinement-expression>",
      "wdata" : "<refinement-expression>",
      "ren"   : "<refinement-expression>",
      "raddr" : "<refinement-expression>",
      "rdata" : "<refinement-expression>"
    }, 
    // more ...
  }, 
  // other fields ...
}
```

For conditional mapping, the list of conditions has a priority. The first listed condition will be tested first, and if it is true, the first refinement expression will be used and then comes to the second and third.

A memory expression allows more than a Verilog array name, because sometimes, the mapping for array variables is not one-to-one. We support conditional mapping in this case, you can specify it in the similar syntax as case (2), or you can use ` conditon ? var1 : var2 `. 

For an external memory, the mapping is actually between interface signals. The ILA array variables use 6 interface signals (see the table below) which should be mapped with the RTL signals. You can use refinement expression or condition-refinement pair for these signal mapping. Later, we shall see some examples of mapping internal or external arrays in ILA with either internal or external arrays in RTL.

| interface signals   | meaning        |
|-------------        |--------------  |
| ren                 | read enable    |
| raddr               | read address   |
| rdata               | read data      |
| wen                 | write enable   |
| waddr               | write address  |
| wdata               | write data     |


The `input mapping` is similar to `state mapping` but there the keys are the ILA input variables. Currently, input variables with array type are not supported.


## Other Sections in Variable Mapping

Within the same file of variable mapping, there are other sections: (1) clock and reset mapping, (2) clock and reset configuration, (3) uninterpreted function mapping, (4) additional mapping (5) assumptions and (6) RTL monitors.


### Interface Mapping

The interface mapping is used to specify which input port(s) is/are clock or reset signals. An example is shown below. The `CLOCK` field is a map from the name of the clock to the input pins. If there is only one clock, you can provide either a single input pin or multiple pin names in list which will be connected to the same clock. For reset, you can choose among the active-high/active-low or custom reset patterns using `RESET`, `NRESET` or `CUSTOMRESET`.

By default, all input ports are connected as the wrapper's input and all output ports are connected with the wrappper's output.  In case you want to specify how the input ports should be connected, you can use the  `INPUT PORTS` field.


```javascript
  "RTL interface connection": {
    // below creates two clocks : clkA and clkB
    // clkA is connected to wclk
    // and clkB is connected to rclk and clk
    "CLOCK" : { "clkA" : "wclk", "clkB" : ["rclk", "clk"] },
    // or "CLOCK" : "clk"  , single clock -- single pin
    // or "CLOCK" : ["wclk", "rclk"] , multiple input pins use 
    //    the same single clock
    "RESET" : "reset", // you can have a list of signals here
    "NRESET" : ["nreset1", "nreset2"],  // like this
    "CUSTOMRESET" : {"name"  : "input-pin", ...},
    "INPUT PORTS" : {
      "<rtl input pin>" : "<refinement-expression>"
    }
  }
```




### Clock and Reset Configuration

In the interface mapping part, the clock and reset pins are specified. If there is one clock and the reset sequence is just one cycle, you don't need to explicitly state the clock and reset configurations.

In the `reset` section, you can specify (1) the initial state (2) the length of the reset sequence, and (3) the custom reset sequence. A sequence is list of 0s or 1s. You can duplicate a sequence by adding a `"*N"` element in the end, where `N` is the number of copies to make. If you don't need reset sequence, you can set `cycles` to 0. And if both reset sequence and the initial state are provided, the RTL design will go through a reset sequence after the specified initial state.

An example of the `reset` section is provided below. 

```javascript
 "reset" : {
   "initial-state" : {
     // this can be used with/without the reset sequence
     "RTL.v1" : "1'b1",
     "RTL.v2" : "2'b01",
     ...
   },
   "cycles" : 5, 
     // or 1 by default, this affects RESET, NRESET, CUSTOMRESET.n
     // if you don't need reset, use 0 here
   "custom-seq" : {"CUSTOMRESET.1": [0 , [1, "*3"], [0, "*1"] ] }
}
```

The `clock` section is only needed if you have multiple clocks. Currently, the tool will only handle the case where the frequency of the clock signals have a least common multiple. Therefore, they can be simulated using a single clock signal. The `clock` tells the tool how  to simulate this clock signal.
This can be specified using either `custom` or `factor` field.  The `custom` field should be sequences of 0s and 1s where you can use the similar duplicate operator as in the custom sequence of reset configuration. Alternatively, you can use the `factor` section to specify how many cycles of 0s and 1s and the starting offset of each clock.


An example of the `clock` section is provided below, where using the `factor` field and the `custom` field actually creates the same set of clock signals.


```javascript
 "clock" : 
 {
   "factor" : {
      "clkA" : {
        "high" : 6
        "low" : 6
        // 12x gclk's period
      },
      "clkB" : {
        "high" : 2
        "low" : 1
        // 3x gclk's period
        // duty cycle 2/3
      },
      "clkC" : {
        "high" : 2
        "low" : 4,
        "offset" : 3
        // 6x gclk's period
        // duty cycle 1/3
        // 1 1 0 0 0 0
        // start:^
        // offset should be positive
      }
      // tool will calcuate the min length
      // LCM(12,3,6) = 12
   }
 }

 // Alternatively, you can use the custom field.
 // Using both the factor and custom fields is
 // not allowed.
 "clock" : 
 {
   "custom" : {
      "length" : 12,
      "clocks": {
        "CLOCK.1": [  [1, "*6"], [0,"*6"] ], 
        "CLOCK.2": [  [1, 1, 0], "*4" ], 
        "CLOCK.3": [  [0, "*3"], [1, "*2"] , [0, "*4"] , [1, "*2"], 0 ]
      }
   }
 }

```



### Uninterpreted Function Mapping

The ILA model may use uninterpreted functions, however, in the verification, the tool must know the correspondence of this uninterpreted function in order to reason about the correctness. In the `functions` field of the _var-map_ JSON, a list should be provided if uninterpreted function is used in the ILA model.
Each element of this list is for one application of this function. Per each function application, the tool needs to know the correspondence of the arguments/result in the Verilog. And this is specified with a map. The keys of the map are the function names, with the values also in a map containing `result` and `arg` fields. `result` field accepts a single refinement expression while `arg` accepts a list.

For example in the ILA there is such specification

```cpp
  auto div = FuncRef("div", SortRef::BV(8), SortRef::BV(8), SortRef::BV(8) );
  instr.SetUpdate(state0, div(arg0, arg1) );
```

Then in the refinement map, `div` is followed by a list of one element (because there is only one function application in ILA).

```javascript
{
  "functions" : {
    "div" : [ {
      "result" : "<refinement-expression>",
      "arg" : [
        // there are only two arguments in div
        "<refinement-expression-1>",
        "<refinement-expression-2>"
        ]
      }
    ]
  }
}
```

### Additional Mapping

Sometimes, the mapping between Verilog variables and ILA state variables does not fit easily into the state mapping section.
In this case you specified additional mapping using `additional mapping` section. It takes a list of strings and each string should be a valid Verilog expression. This will be translated as assumptions, which only appear when verifying individual instructions and will not be used when checking or synthesizing starting state invariants.

```javascript
"additional mapping" : [
  "( RTL.wr & RTL.stb) == (ILA.m1 & ILA.m2)" , 
  "(~RTL.wr & RTL.stb) == (ILA.m2 & ILA.m3)" 
]
```

### Additional Assumptions

This section allows users to add additional assumptions in the verification. They can be, for example, an assumption about the module I/O.

```javascript
"assumptions" : [
  // the two inputs can not be both 1
  "! ( RTL.input1 & RTL.input2 )"
]
```

Assumptions in this section are in effect when verifying the instructions, the invariants and when synthesizing the invariants.

### RTL Monitors

RTL monitors are additional logic and states that are added to aid the specification of refinement mapping.
Our tool provides two templates (value recorder and phase tracker) and also allows arbitrary synthesizable Verilog to be used.

#### Phase Tracker

Phase tracker is a template for creating auxiliary variables to track the phase of an instruction. An example is shown below.
It defines 4 phases, and the rules of entering and exiting each phase is specified in the `rules` field.
For each phase, you can give it name, otherwise you can refer to it as `phaseX` where `X` is an index number starting from 1.
There are two phases that are implicitly declared, which are `#decode#` and `#commit#`. 
The entering condition should explicitly have its previous stage in the conjunction.


```javascript
  "monitor" : {
    "tracker" : { // this is just a name
      "template" : "phase tracker",
      "event-alias" : { // will be translated as creating 1-bit wire
        "issue" : "SomeVerilogExpressionsHere",
        // for example
        "i2e" : "RTL.s2_to_s3 & RTL.s2_to_s3",
        "e2c"  : "RTL.s3_deq & RTL.s3_deq"
      },
      "rules" : [
        { // 1
          "enter" : {
            "event"  : "issue & #decode#"
          },
          "exit" : {
            "event"  : "i2e"
          }
        },
        { // 2, if you give it a name, then 
          "name" : "second_phase"
          "enter" : {
            "event"  : "i2e & #phase1#"
          },
          "exit" : {
            "event"  : "e2c" 
            // it is redundant to add & #phase2#
          }
        },
        { // 3
          "enter" : { // you can use the name elsewhere
            "event"  : "e2c & #second_phase#",
          },
          "exit" : {
            "event"  : "e2c ## 1",
          }
        }
      ]
      // instantiate a phase tracker template
      // each template has its other fields
      // to fill in
    }
```

Additionally, you can declare alias for events which can be referred to in the rules. You can also define auxiliary variables and update them using actions. Below shows an example of tracking a command/instruction through a FIFO. When the command/instruction enters the FIFO, we need to take down the write pointer of the FIFO (stored in auxiliary variable `fifo_ptr`) so that we can know when the command/instruction gets out of the FIFO.


```javascript
  "monitor" : {
    "TrackingThroughFifo" : { // this is just a name
      "template" : "phase tracker",
      "aux-var" : [ ["fifo_ptr", "reg", 8] ],
      "rules" : [
          { // 1
            "enter" : {
              "event"  : "RTL.fifo_wen & #decode# ",
              "action" : "fifo_ptr <= RTL.fifo_wptr;"
            },
            "exit" : {
              "event"  : "RTL.fifo_ren & RTL.fifo_rptr == fifo_ptr"
            }
          },
          { // 2
            "enter" : {
              "event"  : "RTL.fifo_ren & RTL.fifo_rptr == fifo_ptr & #phase1#"
            },
            "exit" : {
              "event"  : "..."
            }
          }
        ]
    }
  }
```

The phase tracker also allows specifying branching and convergence. This is useful, for example, in the verification of superscaler out-of-order processor implementation, where there are multiple pipelines and instructions can be dynamically dispatched to some processing pipelines. A phase can be referred to in other parts of the refinement specification using `#MonitorName_PhaseName#` (unnamed stages are named as `stageX` where X is the index in the list of rules, and the index starts from 0).


#### Value Recoder

Value recorder can be created using `value @ condition` syntax in the refinement expressions. Alternatively, you can also  create explicit value recorders. An example showing the syntax for specifying explicit value recorder is provided below. For the `width` field, you can also use `"auto"` to ask the tool to automatically infer the width needed to hold the value.


```javascript
  "monitor" : {
    "r1_pvholder" : { // this is just a name
      "template" : "value recorder",
        "cond": "RTL.write_enable == 1",
        "val":"RTL.registers[1]",
        "width":8
    }
  }
```

The above value recorder will be translated into the corresponding Verilog below.

```text
input      [7:0] __r1_pvholder_init__;
output reg      [7:0] r1_pvholder;

always @(posedge clk) begin
   if(rst) begin
       r1_pvholder <= __r1_pvholder_init__;
   end
   else begin
       r1_pvholder <= r1_pvholder;
   end
end

assume property ((RTL.write_enable == 1) |-> ((r1_pvholder) == (RTL.registers[1])));
```

It creates an auxiliary Verilog variable `r1_pvholder` which carries a undetermined value. Its value keeps the same all the time, and an assumption says this undetermined value is same as the specified value `RTL.registers[1]`, under the condition that `RTL.write_enable == 1`. This variables can be referenced in other sections by `r1_pvholder`. This value holder does not check if there is only one cycle that `RTL.write_enable == 1` holds. If there are multiple cycles that its condition holds, the assumption may overconstrain if `RTL.registers[1]` should carry different values at these points. This situation should be avoided.


#### Arbitrary Verilog

You can also add arbitrary synthesizable Verilog as the monitor. An example is shown below:

```javascript
  "monitor" : {
    "counter" : {
      "verilog": 
        ["always @(posedge clk) begin",
         "  if (RTL.signal_1) counter <= 0;",
         "  else if (RTL.signal_2 && ! RTL.signal_3) counter <= counter + 1;",
         "end"],
      "defs" :[ ["counter", 2, "reg"] ],
      "refs" :[ "RTL.signal_1" , "RTL.signal_2", "RTL.signal_3" ]
    },
  }
```

This creates a 2-bit variable `counter` to count the number of cycles under a certain condition. Variables that should be accessible outside the monitor should be defined in the `defs` list with its name, bit-width and its type: `reg` or `wire`. Variables used only inside the monitor can be defined inside the `verilog` code list. Any variable inside the original Verilog, if referenced, should be put in the list of `refs`. This will help our tool to add auxiliary wires to connect them with the monitor.

Monitors are normally only in effect when verifying instructions. If you want a monitor to appear also when verifying the invariants, you can add the `"keep-for-invariants":true` attribute in the monitor's description following the `defs` and `refs` attributes.

Additionally, you can load Verilog from file using `"verilog-from-file":"<file-name>"` instead of the `
verilog` field above. You can also have additional Verilog appended to the generated wrapper using field `append-verilog` and `append-verilog-from-file`.


## Instruction Completion Condition



The instruction completion condition is specified per instruction. In the JSON file, it is a list of maps. The list does not need to be a full list of instructions. Those not in the list will not be verified. Each dictionary must have an element whose key is `instruction` and the value of this element is the name of the instruction in the ILA model. Besides this name element, it must contain one of the `ready bound` or the `ready signal` element. The `ready bound` specifies a bound that the instruction takes. It is used for instructions that take a fixed number of cycles. Alternatively, one can provide a signal \(or a predicate\) in the `ready signal` field.

There are two optional fields. They are `start condition`, `max bound`.

The `start condition` field, if provided, should be a list of strings. Each string is a Verilog expression that acts as a predicate on the Verilog design. It can be used to constrain the Verilog implementation state, because usually there are more microarchitectural \(implementation states\) in the design. For the starting state of an instruction, these microarchitectural states should be `consistent` with the visible states in the ILA, and you can use this field to enforce the `consistency` at your discretion. You can use `$decode$` and `$valid$` to refer to the instruction's decode function or its ILA valid condition.

The `max bound` can be used when `ready signal` field is provided. It provides an assumption that the condition that the `ready signal` field specified will occur in the given number of cycles, and this will limit the model checking to that bound. Usually this is used for the additional environment assumptions about the input \(for example, how many cycles at most there are for a request to be served with a response\).


### The Structure of Instruction Completion Condition

The structure of the instruction completion condition is relatively simple as shown below. `max bound` and `start condition` are optional, and you can only choose one between `ready bound` and `ready signal`.


```javascript
{
  "instructions":
  [
    {
      "instruction"     :"<instr-name>",
      "ready bound"     :12345, // <some integer here>
      "ready signal"    :"<condition>",
      "max bound"       :12345, // <some integer here>
      "start condition" :"<condition>"
    }
  ],

  "global invariants" : 
  [
    "Verilog-expressions-here",
    ...
  ]
}
```

### Global Invariants

In the verification of instructions, we do not assume the design starts from the initial states. This helps us to get a better guarantee of the instruction correctness when only bounded model checking is used. However, if there is no constraints on the starting state of an instruction, there might be spurious bugs just because the design starts from a state that it will never reach when started from the reset state. In order to avoid this false positive, we use global invariants to constrain on the starting state. These invariants help rule out some unreachable states and the tool will generate a separate target to check whether the provided invariants are globally true or not. These invariants should be provided as a list of strings, where each string is a Verilog predicate. 


