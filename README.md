# QXContexts

[![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://JuliaQX.github.io/QXContexts.jl/stable)
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://JuliaQX.github.io/QXContexts.jl/dev)
[![Build Status](https://github.com/JuliaQX/QXContexts.jl/workflows/CI/badge.svg)](https://github.com/JuliaQX/QXContexts.jl/actions)
[![Coverage](https://codecov.io/gh/JuliaQX/QXContexts.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/JuliaQX/QXContexts.jl)

QXContexts is a Julia package for simulating quantum circuits using tensor network approaches and targeting large distributed memory clusters with hardware accelerators. It was developed as part of the QuantEx project, one of the individual software projects of WP8 of PRACE 6IP.

QXContexts is one of a family of packages each with a different aim. QXContexts is the package that is designed to the do the bulk of the computations and makes use of distributed compute resources via [MPI.jl](https://github.com/JuliaParallel/MPI.jl) as well as hardware accelerators. [OMEinsum.jl](https://github.com/under-Peter/OMEinsum.jl) and [TensorOperations.jl](https://github.com/Jutho/TensorOperations.jl) are currently used to carry out the tensor contraction operations.

# Installation

QXContexts is a Julia package and can be installed using Julia's inbuilt package manager from the Julia REPL using.

```
import Pkg
Pkg.add("QXContexts")
```

or directly from the github repository with

```
import Pkg
Pkg.add(url="https://github.com/JuliaQX/QXContexts.jl")
```

## Custom system image

Using a custom system image can greatly reduce the latency when starting computations.
To build a custom system image one can run the following commands from the Julia REPL

```
import QXContexts
QXContexts.compile()
```

This can take up to a half hour to compile and will produce a shared object system image file in the root folder of the project.
This will have a different suffix depending on the platform (`.so` for Linux systems, `.dylib` for macOS and `.dll` for windows systems).
To use the system image the `--sysimage` (or equivalently `-J`) can be used providing the path to the system image. For example

```
julia --project --sysimage=JuliaSysimage.so
```

For development it is useful to use a system image without any of the functions from QXContexts itself begin compiled.
To do this one can call the compile function with `dev` set to true as

```
QXContexts.compile(dev=true)
```

# Example usage

QXContexts uses input files generated by QXTools which describe the computation to be performed.
An example of the input files for a five qubit GHZ circuit are provided in the `examples/ghz` folder.
This example can be run directly using the `examples/ghz_example.jl` script or this can be run using the CLI `bin/qxrun.jl` script with the following command

```
julia --project bin/qxrun.jl -d examples/ghz/ghz_5.qx\
                             -i examples/ghz/ghz_5.jld2\
                             -p examples/ghz/ghz_5.yml\
                             -o examples/ghz/out.jld2
```

where the `-d`, `-i` and `-p` switches describe the DSL file, input data file and parameter file to use respectively.
The `-o` switch refers to the output file.
If all three files have the same prefix, then it is only necessary to provide the name of the dsl file so the example could also be run with the command

```
julia --project bin/qxrun.jl -d examples/ghz/ghz_5.qx -o examples/ghz/out.jld2
```

The output is written to a [JLD2](https://github.com/JuliaIO/JLD2.jl) file.
A small utility script called `examine_output.jl` is provided that allows examination of this output which
can be used as

```
julia --project bin/examine_output.jl examples/ghz/out.jld2
```

## Enable timing

To get timing information on the different sections of the code the code has been instrumented with [TimerOutputs.jl](https://github.com/KristofferC/TimerOutputs.jl). To enable this one can add the `--timings` (or `-t`) switch to the CLI command.

```
julia --project bin/qxrun.jl -d examples/ghz/ghz_5.qx -o examples/ghz/out.jld2 -t
```

## Enable debugging

To get detailed debugging information one can include the package name in the `JULIA_DEBUG` environment variable. For example

```
JULIA_DEBUG=QXContexts julia --project bin/qxrun.jl -d examples/ghz/ghz_5.qx -o examples/ghz/out.jld2
```

This generates very verbose output so care should be taking when using this for large runs.

## Enable logging

To log debug and performance information to files QXContexts has 3 logger-models:

- QXLogger: the default stdout logger: useful for single node, single process logging (interactive)
- QXLoggerMPIShared: an MPI-IO shared-file logger: all MPI ranks share a single file for writing their respective logs; blocking.
- QXLoggerMPIPerRank: MPI-enabled file per rank logger: non-blocking debug files created per MPI rank.

The loggers can be (individually) instantiated by selecting the global logger to use with one of the following:

```
global_logger(QXContexts.Logger.QXLogger())
global_logger(QXContexts.Logger.QXLoggerMPIShared())
global_logger(QXContexts.Logger.QXLoggerMPIPerRank())
```

# Running with MPI

MPI is used to use multiple processes for computation. The `mpiexecjl` script can be used to launch Julia on multiple processes. See [MPI.jl documentation](https://juliaparallel.github.io/MPI.jl/latest/configuration/#Julia-wrapper-for-mpiexec) for details on how to set this up. For example to run the above example with 4 processes one would use the following:

```
mpiexecjl --project -n 4 julia bin/qxrun.jl -d examples/ghz/ghz_5.qx -o examples/ghz/out.jld2
```

In this case the amplitudes that are to be calculated are split between the processes. For
larger cases where many partitions are used for each amplitude it can be useful to split
this calculation over many processes also. The `--sub-communicator-size` (or `-m`) option
can be used to specify the size of sub-communicators to use for each amplitude. For example

```
mpiexecjl --project -n 4 julia bin/qxrun.jl -d examples/ghz/ghz_5.qx -o examples/ghz/out.jld2 -m 2
```

Here the four processes are split between two communicators, each with two processes.

# Using GPUs

On systems with NVIDIA GPUs, these can be used by passing a `--gpu` (or `-g`) flag to `qxrun.jl` on the command line.

# Contributing
Contributions from users are welcome and we encourage users to open issues and submit merge/pull requests for any problems or feature requests they have. The
[CONTRIBUTING.md](CONTRIBUTION.md) has further details of the contribution guidelines.


# Building documentation

QXSim.jl using [Documenter.jl](https://juliadocs.github.io/Documenter.jl/stable/) to generate documentation. To build
the documentation locally run the following from the top-level folder.

The first time it is will be necessary to instantiate the environment to install dependencies

```
julia --project=docs 'using Pkg; Pkg.develop(PackageSpec(path=pwd())); Pkg.instantiate()'
```

and then to build the documentation

```
julia --project=docs docs/make.jl
```

To serve the generated documentation locally use

```
julia --project=docs -e 'using Pkg; Pkg.add("LiveServer"); using LiveServer; serve(dir="docs/build")'
```

Or with python3 using from the `docs/build` folder using

```
python3 -m http.server
```

The generated documentation should now be viewable locally in a browser at `http://localhost:8000`.
