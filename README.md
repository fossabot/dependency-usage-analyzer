# FOSSA Dependency Usage Analyzer

`fossa-dependency-usage-analyzer` inspects a checked-out repository and reports, for every direct
and transitive dependency it can resolve, the dependency's scope, whether it is linked into and
shipped with the module's artifact, and its provenance. It is the analysis engine that
[Ficus](https://github.com/fossas/ficus) embeds.

This repository hosts the release executables. The source lives in a FOSSA monorepo; each release
here mirrors the corresponding monorepo release, so version numbers follow that release train
rather than this tool's own feature history.

## Download

Every release on the [releases page](https://github.com/fossas/dependency-usage-analyzer/releases)
carries one executable per target plus a checksum file and third-party notices.

| Asset | Runtime floor |
| --- | --- |
| `fossa-dependency-usage-analyzer-x86_64-unknown-linux-gnu` | glibc 2.17 |
| `fossa-dependency-usage-analyzer-aarch64-unknown-linux-gnu` | glibc 2.17 |
| `fossa-dependency-usage-analyzer-x86_64-unknown-linux-musl` | musl with `libstdc++` and `libgcc` installed |
| `fossa-dependency-usage-analyzer-aarch64-unknown-linux-musl` | musl with `libstdc++` and `libgcc` installed |
| `fossa-dependency-usage-analyzer-x86_64-apple-darwin` | macOS 13 |
| `fossa-dependency-usage-analyzer-aarch64-apple-darwin` | macOS 13 |
| `fossa-dependency-usage-analyzer-x86_64-pc-windows-msvc.exe` | Windows 10 1809 |

The musl executables dynamically link `libstdc++.so.6` and `libgcc_s.so.1`. Stock Alpine images
do not include them; run `apk add libstdc++ libgcc` first.

Verify a download against `fossa-dependency-usage-analyzer-checksums.txt` from the same release:

```sh
TAG=v0.0.327
TARGET=x86_64-unknown-linux-gnu
BASE=https://github.com/fossas/dependency-usage-analyzer/releases/download/$TAG
curl -sSfLO "$BASE/fossa-dependency-usage-analyzer-$TARGET"
curl -sSfLO "$BASE/fossa-dependency-usage-analyzer-checksums.txt"
sha256sum --check --ignore-missing fossa-dependency-usage-analyzer-checksums.txt
chmod +x "fossa-dependency-usage-analyzer-$TARGET"
```

## Usage

```sh
fossa-dependency-usage-analyzer <repository-directory> [--output <file>]
```

The run writes the JSON result to `dependency-usage-result.json` in the current directory unless
`--output` names another file. Standard output is a stream of newline-delimited JSON records with a
`type` discriminator: a `progress` record per completed step, then a final `result` record on
success or an `error` record with a nonzero exit code on failure. Diagnostics go to standard error.

Dependency scope, linking, distribution, and provenance are determined by deterministic rules and
need no credentials. The provenance step probes `registry.npmjs.org` and `pypi.org` for package
existence; when either is unreachable the affected rows report `origin: "unknown"` and the run
still succeeds.

One step, the codebase exposure assessment, is model-backed. It runs only when `ANTHROPIC_API_KEY`
is set, using a read-only agent over the checkout, and otherwise reports itself unassessed. The
executable never reads a `.env` file from the analyzed checkout. Set `CODEBASE_EXPOSURE_MAX_TURNS`
to override the agent's default budget of 60 tool turns.

## License

The executables are distributed under the
[Common Public Attribution License 1.0](./LICENSE). Each release attaches
`fossa-dependency-usage-analyzer-third-party-notices.txt` with the licenses of the open-source
components bundled into the executables.
