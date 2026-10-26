# Audit scope

Written for an outside, independent security audit. Revision described:
`240ee9affd100afed5c272059e43668d8f971f45` on `main`, 2026-10-26. Everything below is taken from the repository's own files at
that revision; items marked NOT_RUN were not executed while writing this document.

## What this repository is

The AienOS alpha installer for NVIDIA DGX Spark (Apache 2.0 with LLVM Exceptions):

- `installer/install.sh` (185 lines, POSIX `sh`): platform gate (Linux, aarch64, NVIDIA driver
  `580.173.02`, a PCI device `0x10de:0x2e12`), download of one tagged release, Ed25519 check of
  `SHA256SUMS` with an embedded public key, SHA-256 check of the bundle, unpack to a staging
  folder, second check of every file, then move into place. No sudo.
- `installer/wizard/*.om` (eight units, 4,888 lines of Omega): the `aienos` program with
  `doctor`, `chat`, `start`, `stop`, `verify`, `source`, `about`, `uninstall`, `rollback`.
- `installer/build.sh`, `installer/tools/sign.sh`, `installer/tools/release.sh`: build from pinned
  inputs and offline signing.
- `installer/tests/`: test suite, container test, host-untouched test, fake hardware trees.

## What to review, in priority order

1. `installer/install.sh` lines 75 to 120: the fetch, signature and checksum order; check whether
   `curl --proto '=https,http'` and the `AIENOS_BASE_URL` and `AIENOS_PROBE_ROOT` overrides matter
   once the signature check is in place; the `tar -tzf` name check refuses absolute paths and `..`,
   so check symlink and hard link entries in the bundle.
2. `installer/wizard/sha256.om` (CI checks it is derived from omega-mv `om/sha256/sha256-om2.om`)
   and `installer/wizard/steps.om`: install check, model download and per-file check.
3. `installer/wizard/engine.om` and `lib.om`: child processes through `fork` and `execprog`
   (`curl`, `stty`, `df` per HOST-DEPENDENCIES.md), engine start and stop, the `/proc` maps check,
   uninstall and rollback paths.
4. `installer/wizard/net.om`: loopback HTTP/1.0 client to the engine on `127.0.0.1:18100` and the
   JSON escaping for chat.
5. `installer/tools/sign.sh`: the seed is read through the vault tool's `run` and passed in an
   environment variable to `sh -c`, then piped to `openssl`. Check its exposure while that runs.
6. `.github/workflows/release-verify.yml`: verifies a published release from the outside.

## Trust boundaries

- Trust root: the alpha Ed25519 key in `KEYS`, `keys/release.pub.pem` and embedded in
  `install.sh`.
- Untrusted: everything downloaded (release assets, model files) until checked; the engine's HTTP
  replies.
- Trusted host tools: `curl`, `openssl`, `sha256sum`, `tar` and the others listed in
  `installer/HOST-DEPENDENCIES.md`.
- `install.sh` itself is fetched with `curl | sh` from `aienos.com`; its integrity rests on HTTPS.

## Threat model summary

- Assets: the user's home folder and account; the existing NVIDIA driver and CUDA install, which
  the installer must leave unchanged.
- Attackers considered: a tampered or substituted release, a malicious archive, a network attacker
  on downloads, a wrong machine.
- Defended: signature before unpack, two SHA-256 passes, path name checks on the archive, exact tag
  and asset names (never "latest"), refusal on non-GB10 platforms with no change.
- Not defended / explicitly out of scope: a compromised build machine or vault holding the alpha
  seed; a tampered `install.sh` delivered over HTTPS; engine-side flaws.

## Build from source at a pin

Pins from `installer/build.sh` and `.github/workflows/verify.yml`: omega-mv
`3364a1dfd0ca50c1b82bf19ba34932d8bc10a7a9`, omc sha256
`befb8199db8c4bca188a494dfaac2a6f0ee5c36fcea11188c75666c55fce8a1d`, engine from
aien-sovereign-core-mv `cf239078d05a47d84353c1508a08663273f00c3e`. Linux arm64 only.

```sh
git clone https://github.com/aien-dev/omega-mv && git -C omega-mv checkout 3364a1dfd0ca50c1b82bf19ba34932d8bc10a7a9
echo "befb8199db8c4bca188a494dfaac2a6f0ee5c36fcea11188c75666c55fce8a1d  omega-mv/seed/out/omc" | sha256sum -c -
OMEGA_SRC=$PWD/omega-mv ENGINE=/path/to/aien-serve-http sh installer/build.sh OUT
```

The wizard alone compiles without the engine, as CI does: concatenate the eight units in
`sha256.om lib.om hole.om doctor.om net.om steps.om engine.om main.om` order and pipe them to
`omc`. CI compiles twice and compares the outputs.

## Run every test

NOT_RUN while writing this document. From the test scripts' headers:

```sh
for f in install.sh build.sh tests/run.sh tests/fake-root.sh tests/container.sh tests/host-untouched.sh tools/sign.sh tools/release.sh; do sh -n "installer/$f"; done
OMC=/path/to/omc DIST=OUT/dist STAGE=OUT/stage sh installer/tests/run.sh
sh installer/tests/host-untouched.sh     # run.sh in bubblewrap with / read-only, under strace
DIST=OUT/dist sh installer/tests/container.sh RECEIPT_DIR   # needs docker
```

`tests/run.sh` needs the signed build output; signing needs the release seed. An outside auditor
can run the syntax check, the wizard compile and the doctor on fake trees (`fake-root.sh` with
`gb10`, `nogpu`, `othergpu`, `olddrv`) without the seed.

## Known limitations and NOT_RUN

- README: the engine binary in this release (`80f04b67...`) is NOT_RUN on a GPU; the 3x physical
  qualification of the GPU path is NOT_RUN; the first chat was tested only against a fake engine.
- `installer/tools/capture.py` and `ansi-render.py` are Python, used to record documentation
  screens, not in the install path.
- `install.sh` still relies on host `curl`, `openssl`, `sha256sum` and `tar`; HOST-DEPENDENCIES.md
  gives the Omega replacement path.

## Dependencies and provenance

- omega-mv and omc pins above, checked by digest in CI.
- Engine from aien-sovereign-core-mv tag `alpha/20261025.1` (`cf239078`), per README.
- Model: Qwen/Qwen3.8-27B-FP8 at revision `017b9c7a`, 81 files, hashes in
  `installer/share/model.manifest`.
- Actions pinned by commit (`actions/checkout@11bd7190...`), with `persist-credentials: false` and
  `contents: read`; both workflows run on GitHub-hosted `ubuntu-24.04-arm`.

## Evidence

- `evidence/alpha-20261026.1/README.md`: host suite 33/33 PASS under bubblewrap with `/`
  read-only, 0 writes refused outside the scratch folder, NVIDIA state identical before and after;
  container suite 6/6 PASS including a canary negative control; `SHA256SUMS.tested`.
- `notes/alpha-20261026.1.md`: release notes.
- Core receipt for SC-GB10-SERVE-CHAT is in aien-dev/aien-receipts, not here.
