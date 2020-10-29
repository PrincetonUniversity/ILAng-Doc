# Verification

This Chapter explains how to use the verification functionality
of ILAng to generate verification target to work with existing
model checkers. Currently we support Cadence JasperGold, CoSA, CoSA2. 
For other Verilog model checker, we can generate a standalone
Verilog with embedded assumptions and assertions using the 
SVA format.


For verification, you will need (1) a ILA model (2) the Verilog
module (3) the refinement relation to map ILA and Verilog,
this is shown in the following figure.


![](.gitbook/assets/rtl-verify-arch.png)
