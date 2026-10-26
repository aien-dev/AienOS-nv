# AienOS-nv

This is where the AienOS alpha for NVIDIA DGX Spark comes from: the installer, the engine and
the Omega wizard, as one signed bundle. (The AienOS kernel and its USB/VM artifacts, such as
the `alpha-0.1` release, live in [aien-dev/aienux](https://github.com/aien-dev/aienux).) One command puts the alpha on a DGX Spark:

```sh
curl -fsSL https://aienos.com/install.sh | sh
```

It checks the machine first. On anything other than Linux arm64 with an NVIDIA GB10 (DGX Spark)
it says so, points to https://www.aienos.com/alpha/, and changes nothing. On a Spark it
downloads the release, checks its Ed25519 signature and SHA-256 before unpacking anything,
installs into `~/.local/share/aienos`, adds a menu entry and a desktop icon, and offers to open
the wizard. No sudo, nothing outside your home folder.

## The wizard is Omega

`bin/aienos` is written in Omega, my own language, and compiled by the seeded Omega compiler.
It is a full-screen terminal program with an animated black hole: libcuda, cuBLAS, nvcc and the
CUDA toolkit spiral in and vanish, because my engine does not use them. The picture is the
drama; the change on disk is small and reversible. NVIDIA's kernel driver stays, and the CUDA
toolkit, libcuda and cuBLAS stay installed and unused.

Steps: welcome, doctor, plan, your yes, install, model, first chat, done.

The source is in [`installer/wizard/`](installer/wizard/), one unit in build order:
`sha256.om lib.om hole.om doctor.om net.om steps.om engine.om main.om`. Each release also ships
it concatenated as `aienos.om`, and `aienos source` shows where it lives on your machine.

| Piece | File | Does |
|---|---|---|
| SHA-256 | `wizard/sha256.om` | omega-mv's OM-2 SHA-256 unit; checks every file of the release and the model |
| Output and host words | `wizard/lib.om` | output buffer, ANSI escapes, paths, files, environment, child processes |
| Black hole | `wizard/hole.om` | the animation: integer geometry, a 64-entry sine table, half-block pixels |
| Doctor | `wizard/doctor.om` | CPU, GB10 by PCI id, device nodes, driver version, memory, disk, install state, GPU lock |
| HTTP | `wizard/net.om` | a loopback HTTP/1.0 client and the JSON escaping the chat needs |
| Screens | `wizard/steps.om` | the panel, the progress bar, install check, model download and check |
| Engine | `wizard/engine.om` | start, stop, first chat, the /proc maps check, uninstall, rollback |
| Command | `wizard/main.om` | `aienos` and its subcommands |

Build it with `installer/build.sh` (pins: omega-mv `3364a1dfd`, omc sha256 `befb8199…8a1d`).

## What you get

- `aienos`: the wizard. Also `aienos doctor`, `aienos chat`, `aienos start`, `aienos stop`,
  `aienos verify`, `aienos source`, `aienos about`, `aienos uninstall [--keep-model]`,
  `aienos rollback`.
- The engine, `aien-serve-http`, from aien-dev/aien-sovereign-core-mv tag `alpha/20261025.1`
  (`cf239078`), run with `--engine omega` on `127.0.0.1:18100`, OpenAI-style API. It links only
  libc, libm, libgcc_s and the loader; its GPU path is the statically linked `libomega_gpu.a`.
  There is no separate CUDA++ runtime.
- The model: Qwen/Qwen3.8-27B-FP8 at revision `017b9c7a`, 30.9 GB in 81 files, downloaded from
  Hugging Face by the wizard, every file checked against its SHA-256. The Qwen3.6-35B that my
  Spark serves is not offered yet: it runs on an engine branch that is not merged.

## Evidence

- Core commit `cf239078` passed SC-GB10-SERVE-CHAT once on a GB10 (receipt `8b4ea274…`, in Core
  bundle `44edcc9a…`, aien-dev/aien-receipts). The engine binary in this release
  (`80f04b67…`) was built from that commit; on a GPU it is NOT_RUN.
- The installer, the wizard and the uninstaller were exercised on a fake GB10 tree and in a
  stock Ubuntu 24.04 arm64 container (see the release notes). The wizard's first chat was
  tested only against a fake engine.
- The 3x physical qualification of the GPU path is NOT_RUN.

## Trust

Releases are signed with the alpha release key in [`KEYS`](KEYS). It is an alpha key and is
replaced by a ceremony key before beta. Host programs AienOS still uses, and how each one gets
replaced by Omega: [`installer/HOST-DEPENDENCIES.md`](installer/HOST-DEPENDENCIES.md).

Licensed under the Apache License 2.0 with LLVM Exceptions ([`LICENSE`](LICENSE), [`NOTICE`](NOTICE)).
