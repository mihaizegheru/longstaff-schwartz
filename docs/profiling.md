# Profiling plan

This document sets conventions for future implementations. The commands below
are planned usage, not runnable targets in the current scaffold.

## Independent executables

| Backend | Planned Bazel executable target | Typical profiler |
| --- | --- | --- |
| Serial | `//implementations/serial:serial` | Linux perf |
| pthreads | `//implementations/pthreads:pthreads` | Linux perf |
| OpenMP | `//implementations/openmp:openmp` | Linux perf |
| CUDA | `//implementations/cuda:cuda` | Nsight Systems / Nsight Compute |
| MPI | `//implementations/mpi:mpi` | perf per rank / an MPI-aware profiler |

Each executable should link its own solver and the shared utilities. CPU
executables should not link CUDA or MPI runtimes. Keep `-pthread` and `-fopenmp`
local to their respective targets. Mark CUDA and MPI executable/library targets
`manual` so default wildcard builds can stay usable on CPU-only machines.

## Measurement conventions

- Compare the same model parameters, path count, exercise dates, input data,
  numerical precision, and random seed across implementations.
- Validate numerical results against the serial reference before comparing
  speed. Agree on tolerances for Monte Carlo estimates and reductions.
- Separate input generation, path simulation, regression, backward induction,
  and total execution time. Record what each timer includes.
- Report end-to-end GPU time including transfers separately from kernel time.
  Synchronize GPU work before stopping host timers, or use CUDA events.
- For MPI, distinguish computation and communication; report the maximum
  elapsed time over ranks. Keep initialization and synchronization scopes
  explicit.
- Record warmup count, measured repetitions, worker/rank counts, compiler,
  toolkit/runtime versions, hardware, Git commit, and build configuration.
- Keep strong scaling (fixed total work) and weak scaling (fixed work per
  worker) results separate. Use optimized builds for performance comparisons.

## Planned commands

Once executable targets exist:

```sh
bazel build --config=profile //implementations/serial:serial
perf stat -r 5 bazel-bin/implementations/serial/serial
perf record -g -o profiles/serial.data bazel-bin/implementations/serial/serial

bazel build --config=cuda-profile //implementations/cuda:cuda
nsys profile -o profiles/cuda bazel-bin/implementations/cuda/cuda

bazel build --config=profile //implementations/mpi:mpi
mpirun -np 4 bazel-bin/implementations/mpi/mpi
```

Build first, then profile the executable directly so compilation and Bazel
startup are excluded. Future benchmark options and MPI launcher placement will
be documented with the implementations. For multiple MPI nodes, ensure binaries
and compatible runtimes are available on each node.

Raw output belongs in `results/` and `profiles/`, both ignored by Git. Commit
small, curated comparison tables and methodology under `docs/` when useful.
