# AienOS

AienOS is the system I am building: my own kernel, my own language, my own GPU path, and an AI
that runs on hardware I own. This repository is the project's home page on GitHub. It holds no
code and publishes no releases.

| Part | Where |
|---|---|
| **Sovereign Intelligence**: the downloadable package for NVIDIA DGX Spark (one command, the Omega installer wizard, the engine, the model download) | [aien-dev/sovereign-intelligence](https://github.com/aien-dev/sovereign-intelligence), install with `curl -fsSL https://aienos.com/install.sh \| sh` |
| **Aienux**: the kernel, with its USB and VM artifacts (release `alpha-0.1`) | [aien-dev/aienux](https://github.com/aien-dev/aienux) |
| **Omega**: the language, its seeded compiler and its units | [aien-dev/omega-mv](https://github.com/aien-dev/omega-mv) |
| **CUDA++**: GPU programming without NVIDIA's user-space stack | [aien-dev/CUDApp](https://github.com/aien-dev/CUDApp) |
| The engine (Sovereign Core) | [aien-dev/aien-sovereign-core-mv](https://github.com/aien-dev/aien-sovereign-core-mv) |

What I am building, in plain words, and where each piece stands: `docs/WHAT-I-AM-BUILDING.md`.
More: https://www.aienos.com/

Licensed under the Apache License 2.0 with LLVM Exceptions ([`LICENSE`](LICENSE), [`NOTICE`](NOTICE)).
