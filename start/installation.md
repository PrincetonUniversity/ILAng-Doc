# Installing ILAng

## Prerequisites

The build system of ILAng embraces modern CMake, which requires CMake \(3.8 or above\) and compilers with CXX 11 support. It also requires

| Bison | Flex | z3 | Boost | Python |
| :--- | :--- | :--- | :--- | :--- |
| 3.0.4 - 3.3.0 | 2.5.35 or above | 4.4.0 or above | 1.50.0 or above | 2.7 |

To install all dependencies on Debian-based UNIX:

```bash
apt-get install bison flex libboost-all-dev z3 libz3-dev libgflags-dev
```

To install all dependencies on Arch Linux-based distributions:

```bash
pacman -S bison flex boost z3
```

To install dependencies \(except z3\) on Fedora-based Linux:

```bash
yum install bison flex boost boost-python boost-devel
```

To install on OS X using Homebrew:

```bash
brew install bison flex boost boost-python z3
```

{% hint style="warning" %}
Homebrew updates the formulas \(packages\) frequently, and may encounter build failure. Build and install from source in cases of version conflict.
{% endhint %}

## Building from source

You can clone the [source](https://github.com/Bo-Yuan-Haung/ILAng) of ILAng from GitHub. To build ILAng with default configuration \(and all required sub-modules\), create a build directory:

```bash
cd ilang/root/dir
mkdir -p build && cd build
cmake .. 
make -j$(nproc)
```

After the build complete successfully, you can optionally run tests and install ILAng.

```bash
make run_test
sudo make install
```

### Options

* Use `-DILANG_FETCH_DEPS=OFF` to disable config-time sub-modules update
* Use `-DILANG_BUILD_TEST=OFF` to disable building the unit tests
* Use `-DILANG_BUILD_SYNTH=OFF` to disable building the synthesis engine
* Use `-DILANG_INSTALL_DEV=ON` to enable installing working features

### Sub-modules

When using git older than 1.8.4, you need to update the sub-modules before running the configuration.

```bash
cd ilang/root/dir
git submodule update --init --recursive
mkdir -p build && cd build
cmake .. -DILANG_FETCH_DEPS=OFF
make -j$(nproc)
```

