# Introduction

![](.gitbook/assets/ilang-logo.png)

## Why ILA/ng

ILAng is a modeling and verification platform for systems-on-chips \(SoCs\) where **Instruction-Level Abstraction \(ILA\)** is used as the formal model for hardware components. The ILA formal model targeting the hardware-software interface enables a clean separation of concerns between software and hardware through a uniﬁed model for heterogeneous processors and accelerators. ILAng provides a programming interface for

1. constructing ILA models,
2. synthesizing ILA models from templates using program synthesis techniques,
3. verifying properties on ILA models, and
4. behavioral equivalence checking between different ILA models, and between an ILA specification and an implementation.

ILAng also provides for translating models and properties into various languages \(e.g., Verilog and SMT LIB2\) for diﬀerent veriﬁcation settings and use of third-party veriﬁcation tools.

![ILAng Workflow](.gitbook/assets/ilang-arch.png)

## Applications

The ILAng platform has been used in several applications:

* **Behavioral Equivalence Checking.** The modularity and hierarchy in the ILA models simplify behavioral equivalence checking through decomposition. \[[TODAES18](https://dl.acm.org/doi/10.1145/3282444)\] \([Best Paper Award](https://dl.acm.org/journal/todaes/honors-and-awards)\)
* **Firmware/Hardware Co-Verification**. The ILA models program-visible hardware behavior while abstracting out lower-level implementation details. This enables scalable firmware/hardware co-verification. \[[DAC18](https://ieeexplore.ieee.org/document/8465794), [DATE16](https://ieeexplore.ieee.org/document/7459333)\]
* **Memory Consistency and Concurrency Reasoning**. The ILA model is an operational model that captures program-visible state updates. When integrated with axiomatic memory consistency models that specify orderings between memory operations, the transition relation can be used to reason about concurrent interactions between heterogeneous components. \[[FMCAD18](https://ieeexplore.ieee.org/document/8603015)\]
* **Data Race Checking of GPU Programs**. Besides general-purpose processors and accelerators, an ILA model can be synthesized for the Nvidia GPU PTX instruction set using the synthesis engine. This can then been used for data race checking for GPU programs. \[[ICCAD18](https://dl.acm.org/doi/10.1145/3240765.3240771)\]
* **Synthesizing Environment Invariants for Modular Hardware Verification**. With ILAs, we automate synthesis of environment invariants for modular hardware verification in processors and application-specific accelerators, where functional equivalence is proved between a high-level specification and a low-level implementation. \[[VMCAI20](https://dl.acm.org/doi/10.1007/978-3-030-39322-9_10)\]
* **SoC Protocol Verification.** We leveraged an ILA-based technique that uses modular component specifications as a bridge between protocol specifications and high-level properties to decompose the verification problem. \[[TODAES23](https://dl.acm.org/doi/10.1145/3610292)]\]
* **Bridging Compiler Mapping Gaps for Deep Learning Accelerators.** Using ILA as a formal software/hardware interface, we enabled application-level validation for deep learning accelerators, scaling the validation from individual tensor operators to end-to-end computation of entire neural networks. \[[TODAES24](https://dl.acm.org/doi/10.1145/3639051), [LATTE21](https://vlsiarch.eecs.harvard.edu/publications/specialized-accelerators-and-compiler-flows-replacing-accelerator-apis-formal)\]

### Related publications

Below are the ILA-related publications \(bib-entries available below\)

* Hierarchical Formal Verification of Hardware. \[[TCAD25](https://ieeexplore.ieee.org/document/10883664)\]
* Application-level Validation of Accelerator Designs Using a Formal Software/Hardware Interface. \[[TODAES24](https://dl.acm.org/doi/10.1145/3639051)\]
* Automatic Generation of Cycle-Accurate Timing Models from RTL for Hardware Accelerators. \[[ICCAD24](https://dl.acm.org/doi/10.1145/3676536.3676657)\]
* SoC Protocol Implementation Verification Using Instruction-Level Abstraction Specifications. \[[TODAES23](https://dl.acm.org/doi/10.1145/3610292)\]
* Invited: Generalizing the ISA to the ILA: A Software/Hardware Interface for Accelerator-Rich Platforms. \[[DAC23](https://dl.acm.org/doi/10.1109/DAC56929.2023.10247894)\]
* Generalizing Tandem Simulation: Connecting High-level and RTL Simulation Models. \[[ASP-DAC23](https://ieeexplore.ieee.org/abstract/document/9712564)\]
* Automatic Generation of Architecture-Level Models from RTL Designs for Processors and Accelerators. \[[DATE22](https://ieeexplore.ieee.org/document/9774527)\]
* Compositional Verification Using a Formal Component and Interface Specification. \[[ICCAD22](https://dl.acm.org/doi/10.1145/3508352.3549341)\]
* Usage-Based RTL Subsetting for Hardware Accelerators. \[[ICCAD22](https://dl.acm.org/doi/10.1145/3508352.3549391)\]
* Leveraging Processor Modeling and Verification for General Hardware Modules. \[[DATE21](https://ieeexplore.ieee.org/document/9474194)\]
* Generating Architecture-Level Abstractions from RTL Designs for Processors and Accelerators Part I: Determining Architectural State Variables. \[[DATE21](https://ieeexplore.ieee.org/document/9643584)\]
* Synthesizing Environment Invariants for Modular Hardware Verification. \[[VMCAI20](https://dl.acm.org/doi/10.1007/978-3-030-39322-9_10)\]
* ILAng: A Modeling and Verification Platform for SoCs using Instruction-Level Abstractions. \[[TACAS19](https://link.springer.com/chapter/10.1007/978-3-030-17462-0_21)\]
* A Formal Instruction-Level GPU Model for Scalable Verification. \[[ICCAD18](https://dl.acm.org/doi/10.1145/3240765.3240771)\]
* Integrating Memory Consistency Models with Instruction-Level Abstractions for Heterogeneous System-on-Chip Verification. \[[FMCAD18](https://ieeexplore.ieee.org/document/8603015)\]
* Formal Security Verification of Concurrent Firmware in SoCs using Instruction-Level Abstraction for Hardware. \[[DAC18](https://ieeexplore.ieee.org/document/8465794)\]
* Instruction-Level Abstraction \(ILA\): A Uniform Specification for System-on-Chip \(SoC\) Verification. \[[TODAES18](https://dl.acm.org/doi/10.1145/3282444)\] \([Best Paper Award](https://dl.acm.org/journal/todaes/honors-and-awards)\)
* Template-based Parameterized Synthesis of Uniform Instruction-Level Abstractions for SoC Verification. \[[TCAD18](https://ieeexplore.ieee.org/document/8076885)\]
* Invited: Specification and Modeling for Systems-on-Chip Security Verification. \[[DAC16](https://dl.acm.org/doi/10.1145/2897937.2911991)\]
* Verifying Information Flow Properties of Firmware using Symbolic Execution. \[[DATE16](https://ieeexplore.ieee.org/document/7459333)\]
* Template-based Synthesis of Instruction-Level Abstractions for SoC Verification. \[[FMCAD15](https://ieeexplore.ieee.org/document/7542266)\]

## Other sources

* [POSH Upscale](https://upscale.stanford.edu/): The goal of Upscale project is to develop tools and techniques for verifying and evaluating open-source hardware, with the ILA-based methodology at its core.
* [ILAng](https://github.com/Bo-Yuan-Huang/ILAng): The public GitHub repo for ILAng platform implementation. 
* [ILAng Doxygen](https://bo-yuan-huang.github.io/ILAng-Doc/doxygen-output-html/namespaceilang.html): A web-versioned documentation of the ILAng code annotation. 
* [IMDb](https://github.com/PrincetonUniversity/IMDb): An ILA model data base containing examples, archived ILA models, ILA applications, etc.
* [ItSy](https://github.com/PrincetonUniversity/ItSy): A template-driven ILA synthesis engine using program synthesis techniques.

{% hint style="info" %}
This document is meant to be a living document. You can raise an issue or create a pull request on [GitHub](https://github.com/Bo-Yuan-Huang/ILAng-Doc).
{% endhint %}

{% file src=".gitbook/assets/referencesila.tex" caption="Bibliography entries" %}

