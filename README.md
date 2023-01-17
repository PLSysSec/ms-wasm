# MS-Wasm: Soundly Enforcing Memory-Safe Execution of Unsafe Code

This repository contains all the code necessary for building MS-
Wasm and reproducing the results presented in [our paper](https://doi.org/10.1145/3571208).

## Abstract
Most programs compiled to WebAssembly (Wasm) today are written in unsafe
languages like C and C++. Unfortunately, memory-unsafe C code remains unsafe
when compiled to Wasm—and attackers can exploit buffer overflows and use-
after-frees in Wasm almost as easily as they can on native platforms. Memory-
Safe WebAssembly (MSWasm) proposes to extend Wasm with language-level memory-
safety abstractions to precisely address this problem. In this paper, we build
on the original MSWasm position paper to realize this vision. We give a precise
and formal semantics of MSWasm, and prove that well-typed MSWasm programs
are, by construction, robustly memory safe. To this end, we develop a novel,
language-independent memory-safety property based on colored memory locations
and pointers. This property also lets us reason about the security guarantees
of a formal C-to-MSWasm compiler—and prove that it always produces memory-safe
programs (and preserves the semantics of safe programs). We use these formal
results to then guide several implementations: Two compilers of MSWasm to native
code, and a C-to-MSWasm compiler (that extends Clang). Our MSWasm compilers
support different enforcement mechanisms, allowing developers to make security-
performance trade-offs according to their needs. Our evaluation shows that
on the PolyBenchC suite, the overhead of enforcing memory safety in software
ranges from 22% (enforcing spatial safety alone) to 198% (enforcing full memory
safety), and 51.7% when using hardware memory capabilities for spatial safety
and pointer integrity.

More importantly, MSWasm’s design makes it easy to swap between enforcement
mechanisms; as fast (especially hardware-based) enforcement techniques become
available, MSWasm will be able to take advantage of these advances almost for
free.

## Repositories

Each name links to its Github repo.

### Implementations

- [`rWasm`](https://github.com/secure-foundations/rWasm/tree/mswasm): source
  code for our AOT compiler from MSWasm bytecode. Consists of rWasm with
  modifications to support compiling from MSWasm bytecode. This is available
  on the `PATH` of the docker container: Running `rWasm -w --ms-wasm <path/to/
  mswasm/file>` will create a folder named `generated`. The `--ms-wasm-packed-
  tags`, `--ms- wasm-no-tags`, `--ms-wasm-baggy-bounds` tags can optionally be
  added to enable these Options, run `rwasm –help` for more info. `cd`ing into
  the `generated` folder and running `cargo run --release` will run the program.

- [`mswasm-graal`](https://github.com/PLSysSec/mswasm-graal): source code for
  our JIT compiler from MSWasm bytecode. Consists of GraalVM with modifications
  to support MSWasm Bytecode. `mswasm-graal` is available on the `PATH` of
  the docker container: running `mswasm-graal –Builtins=wasi_snapshot_preview1
  <path/to/mswasm/file>` will run the file. There is also a version of
  Graal’s implementation of vanilla Wasm on the path - running `wasm-graal
  --Builtins=wasi_snapshot_preview1 <path/to/wasm/file>` will run the Wasm
  program.

### Compiler

- [`mswasm-llvm`](https://github.com/PLSysSec/mswasm-llvm.git): source code
  for our compiler from C to MSWasm bytecode. Consists of a fork of LLVM
  (specifically, the CHERI fork of LLVM) with modifications to produce MSWasm
  bytecode. Running `/home/mswasm-llvm/llvm/build/bin/clang –target=wasm32-wasi
  --sysroot=”/home/mswasm-wasi-libc/sysroot” <path/to/c/file>` will generate
  an MSWasm program from a basic C program. Due to current limitations of the
  MSWasm prototypes, clang does not correctly compile arbitrary programs. In
  particular, expect errors on most programs that use `stdout`.

### Supporting Tools

- [`mswasm-wasi-libc`](https://github.com/PLSysSec/mswasm-wasi-libc): source
  code for supporting compilation of C executables to MSWasm bytecode. Consists
  of WASI-libc with modifications to support MSWasm bytecode.

- [`mswasm-polybench`](https://github.com/PLSysSec/mswasm-polybench): PolybenchC
  benchmarks, compiled to native x86-64 code, Wasm bytecode, and MSWasm
  bytecode; along with scripts to perform benchmarking. Inside of the benchmark-
  binaries folders, there are `.mswasm` MS-WebAssembly binary files to be run
  by `rWasm` and `mswasm-graal` `.mswat` MS-WebAssembly text files to be read by
  humans `.native` binaries to run as native C code `.wasm` WebAssembly binaries
  to be run by `rWasm` and `wasm-graal` `.wat` WebAssembly text files to be read
  by humans.

- [`mswasm-wabt`](https://github.com/PLSysSec/mswasm-wabt): source code for
  `mswasm2wat`, a utility for converting MSWasm files into a readable text
  format. Consists of a fork of [WABT](https://github.com/WebAssembly/wabt)
  partially modified to work on MSWasm bytecode. `mswasm2wat` is available on
  the `PATH` and takes an MSWasm binary file as input. For more information
  Wasm text format, see [this guide](https://developer.mozilla.org/en-US/docs/
  WebAssembly/Understanding_the_text_format).

## Reproducing Evaluation Results

### Install

#### Docker Container
We provide a Docker container to create an environment to run these benchmarks at
[ghcr.io/plsyssec/mswasm](https://ghcr.io/plsyssec/mswasm).  To install, install
[Docker](https://www.docker.com) and run

```shell
docker pull ghcr.io/plsyssec/mswasm:latest
```

This will download the container. The container is around 80GB. Once the
container is downloaded, you can run it with commands such as

```shell
docker run -it mswasm:latest
```

You may also use the Docker desktop interface if desired.

#### Dockerfile
We provide a [Dockerfile](Dockerfile) to create an environment to run these
benchmarks. To install, install [Docker](https://www.docker.com/), and in the
same directory as the Dockerfile, run

```shell
docker build -t mswasm .
```

This will build the container. This will take a reasonable amount of time
(around 45 minutes on our host machine) and require internet for cloning git
repositories. This will take more than 32GB of RAM due to compilation, and the
final image will be around 80GB. Once the container is built, you can run it
with commands such as

```shell
docker run -it mswasm:latest
```
You may also use the Docker desktop interface if desired.

### Container Organization

### Running Benchmarks
To run the benchmark suite from the paper, use the original benchify.toml file
with uncommented warmup, min_run and max_run values. The original file can be
found in the `mswasm-polybench` repo. Running the entire benchmark suite will
take a substantial amount of time. It would be advised to lower the min and max
runs.

It can be run with the same instructions as before: `cd mswasm-polybench`
`benchify benchify.toml`. Output will be generated on stdout and in the
benchify-results folder.

#### Choosing Tests

To choose which tests to run, you can modify the `benchify.toml` file.
The `min_runs` and `max_runs` numbers will determine the number of times
a benchmark will be run. Lowering these values to 3 will provide a quicker
benchmark run, but the results will have more noise.

You can comment tests out by removing or commenting them out in `benchify.toml`.
For instance, if I only wanted to run the `2mm` benchmark, my `benchify.toml` would
look like

```
    [[tests]]
    name = "2mm"
    tag = "wasi"
    file = "benchmark-binaries/2mm.mswasm"
    stdout_is_timing = true

  # [[tests]]
  # name = "3mm"
  # tag = "wasi"
  # file = "benchmark-binaries/3mm.mswasm"
  # stdout_is_timing = true
  #
  # [[tests]]
  # name = "adi"
  # tag = "wasi"
  # file = "benchmark-binaries/adi.mswasm"
  # stdout_is_timing = true
  #
  # [[tests]]
  # name = "atax"
  # tag = "wasi"
  # file = "benchmark-binaries/atax.mswasm"
  # stdout_is_timing = true
  ```
  
#### Generating Graphs

To generate the graphs used in the paper, you can use the
[`generate-graphs.py`](generate-graphs.py) script. If you wish to run
the script from the container, run `git pull` from the `mswasm-polybench` repo
to pull the script, and then run `apt-get update && apt-get install
python3-matplotlib python3-pandas python3-seaborn` to install needed
dependencies. The script can then be run as

```shell
python3 <path to generate-graphs.py> <path to benchify csv data in benchify-results>
```

The script will generate 5 graphs as pdfs in the current working directory.
You can extract these graphs from the container with `docker cp`, such as

```shell
docker container ls # find the name of your container
docker cp <container name>:<path to pdf> <path on local machine to copy to>
```

Alternatively, the python script only requires the benchify csv data, so
it does not need to be run on the container. `docker cp` can be used to
retrieve the benchify csv data, then the python script can be run on
a local machine that has python3 and matplotlib, pandas, and seaborn
installed.
