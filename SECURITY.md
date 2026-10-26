# Security policy

## Reporting a vulnerability

Report privately. Do not open a public issue, pull request or discussion about it.

1. GitHub: open this repository's Security tab and choose "Report a vulnerability". This
   creates a private advisory that only the maintainers can see. If that option is not shown,
   private vulnerability reporting has not been enabled for this repository yet; use email.
2. Email: drake@aienos.com with the subject `[AienOS-nv] security report`.

Include the commit or release you tested, the component (path), steps to reproduce, and what an
attacker gains. If the report involves a secret, say where it was exposed and do not send the
secret itself. Encrypted mail is not required; do not post proof-of-concept payloads publicly.

## Response targets

This project has one maintainer; the targets below are best effort and stated honestly.

- Acknowledgement: within 3 working days.
- Triage and severity: within 7 days of acknowledgement.
- Fix or mitigation for confirmed issues: aim for 30 days for high and critical, 90 days
  otherwise, with a status update at least every 14 days until closed.
- Disclosure: coordinated. An advisory is published with credit (unless you ask otherwise)
  once the fix is on the default branch. Default embargo 90 days, negotiable.

## Scope

This repository builds the AienOS alpha for NVIDIA DGX Spark as one signed bundle: the POSIX `sh`
bootstrap `installer/install.sh`, the Omega wizard `bin/aienos` (sources in `installer/wizard/`),
and the release signing and checking tools. The engine binary it ships, `aien-serve-http`, is built
in aien-dev/aien-sovereign-core-mv; report engine flaws there.

In scope:

- Any way to make `install.sh` or `aienos` install or run bytes that are not covered by a valid
  Ed25519 signature over `SHA256SUMS` from the key in `KEYS`.
- Archive extraction escaping the staging folder, or the installer writing outside
  `$HOME/.local/share/aienos` and the menu and desktop entries it names.
- The model download and per-file SHA-256 check against `installer/share/model.manifest`.
- The wizard's loopback HTTP client (`installer/wizard/net.om`), process launching
  (`lib.om`, `engine.om`), uninstall and rollback.
- Handling of the release signing seed in `installer/tools/sign.sh`.
- The two tracked workflows, `verify.yml` and `release-verify.yml`.

Out of scope: the NVIDIA driver, the engine internals, Hugging Face, and the host programs listed
in `installer/HOST-DEPENDENCIES.md`.

## Supported versions

Only the current alpha release named in `installer/install.sh` (`alpha-20261026.1`) and the default branch `main`. Reports against older commits are welcome but fixes land on the default branch only.

## Status

Alpha. The release key is an alpha key, replaced by a ceremony key before beta (`KEYS`). A real
install with a model download and a first token on a GB10 is NOT_RUN. No certification, no bug
bounty. See AUDIT_SCOPE.md for what to review and how to build and test it.
