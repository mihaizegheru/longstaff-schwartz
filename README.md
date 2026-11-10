# Longstaff–Schwartz

C++17 project scaffold for separately profiling serial, pthreads, OpenMP,
CUDA, and MPI implementations of the Longstaff–Schwartz algorithm.

**Setup only:** no algorithm, executable, benchmark script, or test code has
been added. The Bazel targets currently inventory empty source directories.

## Layout

```text
common/                     Shared model definitions, data, and I/O
implementations/
  serial/                   Serial reference implementation
  pthreads/                 POSIX threads implementation
  openmp/                   OpenMP implementation
  cuda/                     CUDA C++ implementation (.cu/.cuh)
  mpi/                      Distributed MPI implementation
tests/                      Future correctness and cross-backend tests
scripts/                    Future benchmark and profiling tools
docs/                       Profiling and team conventions
results/                    Local benchmark results (ignored by Git)
profiles/                   Local profiler output (ignored by Git)
```

Each implementation has its own Bazel package. Keep execution-specific code
and flags in that package. Share model parameters, data formats, and validation
utilities through `common/`; build each backend as its own executable when
implementation begins.

## Build setup

Install [Bazelisk](https://github.com/bazelbuild/bazelisk#installation), which
reads `.bazelversion` and selects Bazel **8.4.2**. The module pins `rules_cc`
and [rules_cuda](https://github.com/bazel-contrib/rules_cuda/tree/v0.3.0).
The first Bazel invocation needs internet access to fetch dependencies.

Validate the current scaffold:

```sh
bazel build //...
bazel build --config=profile //...
```

These commands validate the build graph; they do not compile implementations.
No runnable targets or tests exist yet.

Available configurations:

| Configuration | Purpose |
| --- | --- |
| `--config=debug` | Debug build |
| `--config=release` | Optimized build |
| `--config=profile` | Optimized build with debug symbols and frame pointers |
| `--config=cuda` | Enable CUDA rules when GPU targets are added |
| `--config=cuda-profile` | Profiling build with CUDA source line information |

The initial toolchain convention is Linux with GCC and C++17. CPU work should
require only GCC and its OpenMP runtime. CUDA will additionally need a supported
NVIDIA toolkit, compatible host compiler, and GPU for execution. MPI will need
the team's selected MPI development package and launcher; system MPI integration
is deferred until that implementation is added.

For machine-specific paths and GPU architectures, copy
`.bazelrc.local.example` to `.bazelrc.local`. Do not commit local paths.

## Collaboration

Use short-lived feature branches and pull requests into `main`, for example
`feature/serial-reference`, `feature/openmp`, and `feature/cuda`.
Three contributors can work in separate backend directories, coordinating
changes to `common/`, `MODULE.bazel`, and the root build settings.

See [team conventions](docs/team.md) and the
[profiling plan](docs/profiling.md). GitHub Actions validates the scaffold on
pushes and pull requests. Implementation tests and GPU/MPI jobs will be added
alongside their targets.
