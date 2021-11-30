# Examples

Here we'd like to give some examples of the refinement mapping.

## Conditional Mapping

In variable mapping section, the conditional mapping is helpful in many cases. Below we give some examples using conditional mapping.

### Mapping Loop Body with Unrolled Pipelines

Suppose we have a specification for AES-128 function, which consist of 10 rounds of cryptographic operations, each round takes results from previous round and generate new ciphertext and new round key which will be used in the next round. In the ILA model, this is described with the help of child-instructions that form a loop. The implementation is a pipeline which corresponds to an unrolling of the loop. This is illustrated in the figure below.

![](../.gitbook/assets/aes-128-loop.png)

When verifying the child-instructions, we need to map the state variables holding intermediate results \(suppose it is named as `ciphertext` in ILA\) with signals in Verilog. However, because the loop is unrolled into a 10-stage pipeline, the corresponding Verilog signal for `ciphertext` is dependent on the round. Therefore, we need conditional mapping here, which can be specified as follows.

```javascript
  "state mapping": {
    "ciphertext" : [["ILA.round == 0", "RTL.s_in" ],
                    ["ILA.round == 1", "RTL.s0"   ],
                    ["ILA.round == 2", "RTL.s1"   ],
                    ...
                   ]
  }
```


Regarding the `round` variable in ILA, because the loop in Verilog is fully unrolled and the effects of `round` are already reflected in the mapping of `ciphertext`, we don't need a mapping for `round`. We can simply write:

```javascript
  "state mapping": {
    "round" : null,
    ...
  }
```

{% hint style="info" %}
The condition are treated as a ordered list, namely, only when the first condition does not hold, will the rest of the mapping be considered. You can have a default case with condition `1'b1`.
{% endhint %}

### Mapping State Variables with On-the-Fly Values

When tackling pipelined designs, for a software-visible state variable \(e.g., an architectural register\) it is also common that there might be multiple places to look for, especially the value on-the-fly inside the pipeline. Because of forwarding logic, a newly computed value, though not yet written back to the register file, can still be observable for an instruction following it. In the refinement relation, we provide two possible ways to handle the case: conditional mapping and light-weight flushing.

![](../.gitbook/assets/simplepipe.png)

**Conditional mapping.** Usually, inside the pipeline, there are book-keeping logic to track the location of the latest update to an architectural register \(e.g., the scoreboard\). Therefore, one can use this logic to map an architectural register to the corresponding microarchitectural register in the pipeline. Above shows an examples of such a pipeline, it has three stages \(not counting the stages before instruction dispatch\) and a per-register record `reg_X_w_stage`. The left-most bit indicates if the EX \(execute\) stage has a valid write to register X. The right-most bit indicates if the second stage has a write. Based on status indicated by this scoreboard, one can write the refinement for the register as below. It first checks the left-most bit of the scoreboard entry and then the right-most bit and finally the default case. Note how the priority list is used for condition.

```javascript
  "state mapping": {
      ...,
    "r1" : [ [ "m1.reg_1_w_stage[1]", "RTL.ex_alu_result"],
             [ "m1.reg_1_w_stage[0]", "RTL.ex_wb_val"    ],
             [ "1'b1",                "RTL.register[1]"  ]],
    "r2" : [...],
    ...
  }
```

**Light-weight flushing.** This is a setting similar to what has been used in ARM's ISA-Formal work. One can use prophecy variables \(variables holding values available in the future\) to help construct the mapping. The value-on-the-fly will be written back to the architectural register file in the future and we can map a register with that future value. Our variable mapping allows this mapping using what we call "value recorder" \(it is essentially an auxiliary variable and note that it can also hold history values\). The light-weight flushing setting is illustrated in the figure below:

![](../.gitbook/assets/future-values.png)

Again for the previous pipeline example, we can have the variable mapping as follows:

```javascript
  "state mapping": {
      ...,
    "r1" : [ ["#decode#", "( RTL.register[1] )@( stage_tracker == 1 )"],
             ["#commit#", "RTL.register[1]"]],
    "r2" : [...],
    ...
  }
```

Here `#` quoted names are the names of the phases. The condition `#decode#` indicates, at the beginning of the instruction under verification, `r1` is mapped to the value recorder \(a future value in our case\) and at the end of the instruction execution \(indicated by `#commit#`\), it is mapped to the write-back value in the register file.

## Mapping of Memory

The specification of memory state variable mapping can be very different depends on the types of memory.
The memory in Verilog can be internal or external. While the memory in ILA may be modeled as an internal or memory state variable, as well. Therefore, there are four combinations as described below.

### Internal Memory in both ILA and RTL

In this case, you can use the array expression in `state mapping` section to match the arrays. Let's see an example.
The implementation contains two internal memories `mema` and `memb`, when `start` signal is asserted, it will swap the contents between two addresses in these two memories, where the addresses are specified by the input indices.

```text
// swap with internal memory (im)
module swap (
  input clk,    // Clock
  input rst,  // Synchronous reset active high
  input [3:0] addra,
  input [3:0] addrb,
  input start);

reg [7:0] mema[0:15];
reg [7:0] memb[0:15];


always @(posedge clk) begin
  if (start) begin
    mema[addra] <= memb[addrb];
    memb[addrb] <= mema[addra];
  end
end
endmodule
```

The ILA model also model these as internal memory state variables and the ILA has only one instruction `SWAP`.

```cpp
  auto memswap = Ila("MemorySwap");
  memswap.SetValid(BoolConst(true));

  auto addra = memswap.NewBvInput("addra", 4);
  auto addrb = memswap.NewBvInput("addrb", 4);
  auto start = memswap.NewBvInput("start", 1);

  auto mema = memswap.NewMemState("mema", 4, 8);
  auto memb = memswap.NewMemState("memb", 4, 8);

  {
    auto SWAP = memswap.NewInstr("SWAP");
    SWAP.SetDecode(start == 1);

    auto dataa = Load(mema, addra);
    auto datab = Load(memb, addrb);

    SWAP.SetUpdate(mema, Store(mema, addra, datab)); //mema[addra] = memb[addrb]
    SWAP.SetUpdate(memb, Store(memb, addrb, dataa)); //memb[addrb] = mema[addra]
  }
```

The mapping of the memories in ILA and Verilog is very straight-forward:

```javascript
  "state mapping": {  
    "mema":"RTL.mema",
    "memb":"RTL.memb"},
```

In the backend, model checker like Pono will use array theory to handle the properties. However for JasperGold, the array will be kept concrete and would not scale to large memories.
If it is possible to convert the internal memory to an external one by commenting away the declaration of the Verilog array and the associated read/write logic, then we can use the interface mapping approach that applies to external memory. (Note in the conversion, please keep the read/write signals intact as they will be treated as interface signals to the external memory, and our tool can automatically pull these signals to the top-level module.)


### Internal Memory in ILA and External Memory in RTL

Now suppose we want to convert the above Verilog to external memories, we need to comment away the declaration of Verilog arrays as well as the associated read/write logic.

```text
// swap with internal memory (im) (now converted to external memory)
module swap (
  input clk,    // Clock
  input rst,  // Synchronous reset active high
  input [3:0] addra,
  input [3:0] addrb,
  input start);

// reg [7:0] mema[0:15];
// reg [7:0] memb[0:15];


// always @(posedge clk) begin
//  if (start) begin
//    mema[addra] <= memb[addrb];
//    memb[addrb] <= mema[addra];
//  end
// end

endmodule
```

Below shows the signal mapping below. Here it is a bit tricky as in Verilog there is no such signal that holds the output of the read from these two arrays (they are directly used to overwrite the contect in another location). So we will need to create auxiliary variables to support this. In the table, we use `mema_rdata` and `memb_rdata` as the place-holders.

| memory.port 	| signal       	|
|-------------	|--------------	|
| mema.ren    	| RTL.start     |
| mema.raddr  	| RTL.addra    	|
| mema.rdata  	| mema_rdata    |
| mema.wen    	| RTL.start     |
| mema.waddr  	| RTL.addra     |
| mema.wdata  	| memb_rdata  	|
| memb.ren    	| RTL.start     |
| memb.raddr  	| RTL.addrb    	|
| memb.rdata  	| memb_rdata  	|
| memb.wen    	| RTL.start     |
| memb.waddr  	| RTL.addrb     |
| memb.wdata  	| mema_rdata  	|


The overall state variable mapping should looks like this:

```json
{
  "models": { "ILA":"m0" , "VERILOG": "m1" },
  "instruction mapping": [],
  "state mapping": {  
    "mema":
    {
      "ren"     : "RTL.start"     ,
      "raddr"   : "RTL.addra"     ,
      "rdata"   : "mema_rdata",
      "wen"     : "RTL.start"     ,
      "waddr"   : "RTL.addra"     ,
      "wdata"   : "memb_rdata"  
    },
    "memb":
    {
      "ren"     : "RTL.start"     ,
      "raddr"   : "RTL.addrb"     ,
      "rdata"   : "memb_rdata",
      "wen"     : "RTL.start"     ,
      "waddr"   : "RTL.addrb"     ,
      "wdata"   : "mema_rdata"  
    }
  },
  "input mapping": {
     "addra":"RTL.addra",
     "addrb":"RTL.addrb",
     "start":"RTL.start"
  },

  "RTL interface connection": {
    "RESET" : "rst",
    "CLOCK" : "clk"
  },

  "monitor" : {
    "read_value_holder" : {
      "verilog": [],
      "defs" :[ ["mema_rdata", 8, "wire"],["memb_rdata", 8, "wire"] ],
      "refs" :[]
    }
  }
}
```


### External Memory in ILA and Internal Memory in RTL

In this case, you may want to consider to convert the internal Verilog memory as external using the approach described above. This will help with mapping and also the automated reasoning in the underlying model checkers.

### External Memory in both ILA and RTL

In this case, you can try to map the directly interface signals between ILA and RTL, if 
you are unable to write it as an one-to-one mapping in `input mapping` section, you can also use `additional mapping` section to specify the relation between the two sets of interfaces.



