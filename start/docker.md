# ILAng in Docker

To provide an easy try-on experience without installing all the dependencies here and there, we provide several docker images with ILAng as well as other useful verification tools installed. The images are available at [Docker Hub](https://hub.docker.com/r/byhuang/ilang).

[![docker-io](http://dockeri.co/image/byhuang/ilang)](https://hub.docker.com/r/byhuang/ilang)

### For ILAng users

This image has the latest version of ILAng installed with a complete set of all ILAng features \(including features-in-development\). It also includes [CoSA](https://github.com/cristian-mattarei/CoSA) -- an SMT-based symbolic model checker for hardware designs, with an option to use the [z3](https://github.com/Z3Prover/z3) or [BTOR2](https://github.com/Boolector/btor2tools) as the back-end solver. To get the docker image:

```bash
docker pull byhuang/ilang:latest
```

ILAng and dependencies are installed in the virtual environment `/ibuild/ilang-env`. To initiate the environment settings:

```bash
source init.sh
```

### Demonstrate behavioral equivalence checking

This image provides an example `aes-demo` that demonstrates the use of ILAng in generating verification targets for behavioral equivalence checking. The example contains \(1\) a Verilog implementation of the AES cryptographic accelerator from OpenCores.org, \(2\) an ILA model of the AES design, and \(3\) the refinement relation. It also contains the CoSA model checker for solving the problem. To get the image:

```bash
docker pull byhuang/ilang:posh-demo
```

ILAng and dependencies are installed in the virtual environment `/ibuild/ilang-env`. To initiate the environment settings:

```bash
source init.sh
```

