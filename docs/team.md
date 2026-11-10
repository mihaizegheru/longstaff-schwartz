# Team workflow

## Branches and reviews

Use `main` as the integration branch. Work on short-lived branches named after
the change, such as `feature/mpi`, `feature/cuda-paths`, or `fix/regression-fit`.
Three people can develop different implementations concurrently. Merge small
pull requests regularly to reduce integration conflicts.

Start a branch from the current remote main:

```sh
git fetch origin
git switch -c feature/serial-reference origin/main
```

The repository is currently awaiting its initial commit. For that first change,
create the initial commit on `main` and push it before using the commands above.

Keep related changes together. Include the motivation, affected backend, and
validation in each pull request. Coordinate changes to shared model definitions,
data layout, CLI conventions, and root build files before other branches depend
on them.

## Suggested work split

One contributor can start with the serial reference and shared model, one with
pthreads/OpenMP, and one with CUDA. Assign MPI explicitly once the common input
and result formats stabilize. This is a starting suggestion; ownership can
rotate and the MPI work can proceed earlier once its interface is agreed.

Agree on these interfaces before parallel implementation:

- Model parameters, option type, exercise schedule, and numerical precision.
- Input/path layout and reproducible random number generation.
- Result format, validation tolerance, and benchmark timing boundaries.
- Backend-specific CLI controls for threads, ranks, and GPU selection.

Keep backend implementations in their own packages. Shared utility changes
should be reviewed by someone working on another backend. Use `.clang-format`
for future C++ and CUDA sources.

## GitHub settings

After publishing the scaffold, repository maintainers should invite the other
two contributors and configure `main` protection with pull requests, one review,
and the `Validate scaffold` check required. These are suggested GitHub settings;
they are not applied by files in this repository.

The checked-in CI currently validates only the Bazel scaffold. Add correctness
tests, CPU runtime checks, and MPI integration checks with the first relevant
implementations. CUDA compilation and runtime checks will need an appropriate
toolkit and, for runtime tests, a GPU runner.
