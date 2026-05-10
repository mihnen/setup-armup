# setup-armup

Install an `arm-none-eabi` GCC toolchain in a GitHub Actions workflow,
via [armup](https://github.com/mihnen/armup).

```yaml
- uses: mihnen/setup-armup@v1
  with:
    version: 14.3.rel1
- run: arm-none-eabi-gcc --version
```

After the action runs, the toolchain's `bin/` directory is on `PATH`
for the rest of the job — `arm-none-eabi-gcc`, `arm-none-eabi-ld`, and
the rest of the binutils suite resolve directly.

## Inputs

| Name             | Required | Default     | Description                                                                                  |
| ---------------- | -------- | ----------- | -------------------------------------------------------------------------------------------- |
| `version`        | yes      | —           | Toolchain version to install (e.g. `14.3.rel1`, `9-2019-q4-major`).                          |
| `armup-version`  | no       | `latest`    | armup release tag to use (e.g. `v1.3.0`). Pin to keep CI reproducible.                       |
| `cache`          | no       | `true`      | Cache the installed toolchain across workflow runs.                                           |

Run `armup available` locally to see the version catalog.

## Outputs

| Name      | Description                                                       |
| --------- | ----------------------------------------------------------------- |
| `version` | The version that was installed (echoes back `inputs.version`).    |
| `bin-dir` | Absolute path to the toolchain `bin/` directory that was added to `PATH`. |

## Examples

### Matrix across versions and OSes

```yaml
jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        toolchain: [14.3.rel1, 13.3.rel1, 9-2019-q4-major]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v6
      - uses: mihnen/setup-armup@v1
        with:
          version: ${{ matrix.toolchain }}
      - run: arm-none-eabi-gcc --version
      - run: make
```

### Pinning armup itself for reproducible builds

```yaml
- uses: mihnen/setup-armup@v1
  with:
    version: 14.3.rel1
    armup-version: v1.3.0
```

### Disabling the toolchain cache

The toolchain cache is enabled by default. Disable it if you want each
run to download fresh:

```yaml
- uses: mihnen/setup-armup@v1
  with:
    version: 14.3.rel1
    cache: 'false'
```

### Project-pinned version

If your repo has a `.tool-versions` or `.armup-version` file, you can
read the pinned version into the action:

```yaml
- id: pin
  run: echo "version=$(awk '/^armup/ {print $2}' .tool-versions)" >> "$GITHUB_OUTPUT"
- uses: mihnen/setup-armup@v1
  with:
    version: ${{ steps.pin.outputs.version }}
```

## Supported runners

| Runner OS            | Architectures           |
| -------------------- | ----------------------- |
| `ubuntu-*`           | x64, arm64              |
| `macos-*`            | x64, arm64              |
| `windows-*`          | x64                     |

## How it works

The action is a thin composite wrapper that:

1. Downloads the appropriate `armup` release binary from
   `github.com/mihnen/armup/releases` and verifies its SHA-256.
2. Restores the toolchain data directory from the runner cache (if
   present).
3. Runs `armup install <version>` (a no-op if the cache hit was
   complete).
4. Adds the toolchain `bin/` directory to `PATH`.

Caching is keyed on `(runner.os, runner.arch, armup-version,
toolchain-version)`, so cache hits are deterministic and pinning the
armup version invalidates the cache.

## License

[MIT](LICENSE).
