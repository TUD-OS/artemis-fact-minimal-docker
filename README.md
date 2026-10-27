# Artemis FACT Minimal Docker

Minimal Docker image for [FACT](https://github.com/Sharingcodeability/FACT) (Framework for Automated C-exercise Tests),
used with [Artemis](https://github.com/ls1intum/Artemis) for automated assessment of C programming exercises in the
[Operating Systems courses at the TU Dresden](https://tu-dresden.de/ing/informatik/sya/professur-fuer-betriebssysteme/studium/vorlesungen/bs).

Forked from [ls1intum/artemis-fact-minimal-docker](https://github.com/ls1intum/artemis-fact-minimal-docker).

## Changes Compared to Upstream

- Adds [`strace`](https://strace.io/) so tests can trace the system calls of student programs.
- Published to `ghcr.io/tud-os/artemis-fact-minimal-docker` instead of Docker Hub.

## Supported Architectures

- `linux/amd64`
- `linux/arm64`

## Supported FACT Test Types

- `compile` — compilation tests
- `io` — input/output tests
- `structural` — structural code analysis tests
- `grey_box` / `grey_box_c` — grey box tests

**Not supported:** `oclint` tests (OCLint is not included to keep the image minimal).

## Usage with Artemis

Set the Docker image of the programming exercise to:

```
ghcr.io/tud-os/artemis-fact-minimal-docker:v1.0.0
```

Pin a version tag for exercises in a running course. `:latest` follows the `main` branch and may change at any time.

> **Note on strace:** Tracing a program started by strace itself (`strace -f ./a.out`) works in an unprivileged container. Attaching to an already running process (`strace -p <pid>`) requires `CAP_SYS_PTRACE`, which Artemis build containers don't have.

## Local Build & Test

Build the image:

```bash
docker build -t fact-test .
```

Verify it works:

```bash
# Check FACT imports
docker run --rm fact-test python3 -c "import fact; print('OK')"

# Check libclang loads through FACT's setup
docker run --rm fact-test python3 -c "import fact; from clang.cindex import Index; Index.create(); print('OK')"

# Check GCC
docker run --rm fact-test gcc --version

# Check strace (including tracing as the unprivileged user)
docker run --rm fact-test strace -f -o /dev/null true && echo OK

# Check user (must be uid 5000 for Artemis)
docker run --rm fact-test id
```

Run the included test exercise:

```bash
docker run --rm \
  -v $(pwd)/test-exercise/assignment:/home/assignment \
  -v $(pwd)/test-exercise/tests:/home/tests \
  fact-test \
  bash -c "cd /home/tests && python3 -c \"
from fact.tester import Tester
t = Tester.from_config('tests.yaml')
t.run()
t.export_result()
print('All tests passed!' if t.successful() else 'TESTS FAILED')
\""
```

## Multi-arch Build

Check that the image builds for both architectures (the result stays in the build cache):

```bash
docker buildx build --platform linux/amd64,linux/arm64 .
```

## Publishing

The CI workflow builds both architectures and pushes to GitHub Container Registry:

- Push to `main` → `ghcr.io/tud-os/artemis-fact-minimal-docker:latest`
- Push a tag → `ghcr.io/tud-os/artemis-fact-minimal-docker:<tag>`

```bash
git tag v1.0.0
git push origin v1.0.0
```

## License

MIT, see [LICENSE](LICENSE). Based on work by [ls1intum](https://github.com/ls1intum).
