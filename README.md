# AienOS

AienOS is the operating system I am building: my own kernel, my own language and my own GPU
path, on hardware I own.

To get it on a DGX Spark, install **Sovereign Intelligence, built on AienOS**:

```sh
curl -fsSL https://aienos.com/install.sh | sh
```

This repository describes AienOS and links its parts. It publishes no releases.

| Part | Where |
|---|---|
| **Sovereign Intelligence**, built on AienOS: the front door (install, releases, the Omega wizard) | [aien-dev/sovereign-intelligence](https://github.com/aien-dev/sovereign-intelligence) |
| **Aienux**: the kernel, with its USB and VM artifacts (release `alpha-0.1`) | [aien-dev/aienux](https://github.com/aien-dev/aienux) |
| **Omega**: the language, its seeded compiler and its units | [aien-dev/omega-mv](https://github.com/aien-dev/omega-mv) |
| **CUDA++**: GPU programming without NVIDIA's user-space stack | [aien-dev/CUDApp](https://github.com/aien-dev/CUDApp) |
| The engine (Sovereign Core) | [aien-dev/aien-sovereign-core-mv](https://github.com/aien-dev/aien-sovereign-core-mv) |

What I am building, in plain words, and where each piece stands: [WHAT-I-AM-BUILDING.md](https://github.com/aien-dev/sovereign-intelligence/blob/main/docs/WHAT-I-AM-BUILDING.md) in sovereign-intelligence.
More: https://www.aienos.com/

Licensed under the Apache License 2.0 with LLVM Exceptions ([`LICENSE`](LICENSE), [`NOTICE`](NOTICE)).

## Human-Directed Agentic Engineering

Development follows Human-Directed Agentic Engineering (HDAE): humans define goals,
architecture, constraints, acceptance criteria, and stopping points; agents implement,
execute checks, diagnose failures, and revise within those boundaries. Human review
retains responsibility for claims, acceptance, and release decisions.

For the documented method and a systems-software case study, see Gregory M. Kurtzer,
*Ginux: Building an Independent Kernel That Runs Linux Userspace through Human-Directed
Agentic Engineering* (2026), https://doi.org/10.5281/zenodo.23682548.
