# AIDX Pipeline Security Lab

An intentionally small, local-only lab for understanding and preventing a dangerous XML-to-queue processing pattern:

1. An XML parser resolves external entities.
2. Untrusted XML moves through an asynchronous queue.
3. A privileged consumer processes rejected messages.
4. A helper script and database file privileges create an unexpected trust boundary.

This repository is for **authorized security testing, training, and remediation**. It contains no target addresses, credentials, payload delivery tooling, or persistence automation.

## Why this pattern matters

Each component may look harmless on its own. Together, they can create a high-impact chain:

| Layer | Unsafe decision | Consequence |
|---|---|---|
| XML parser | DTD loading and entity resolution enabled | Local file content can enter the event stream |
| Queue | Raw untrusted input is preserved | Unsafe input reaches later consumers |
| Operations service | A privileged timer handles rejected events | A low-privilege input reaches a high-privilege process |
| Secret handling | Passwords are supplied as command-line arguments | Other local users may observe secrets in process listings |
| Database | Broad file read/write capability | An application helper can become a code-execution boundary |

The goal is not to treat these as isolated bugs. The goal is to remove the chain.

## Quick start: safe local demo

The demo creates a temporary marker file that belongs to the lab, then compares an intentionally unsafe parser with a hardened one. It never contacts a network service and never reads system files.

```bash
git clone https://github.com/YOUR-ACCOUNT/aidx-pipeline-security-lab.git
cd aidx-pipeline-security-lab
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
./scripts/run-safe-demo.sh
```

Expected result:

- `vulnerable_parser.py` prints the lab marker because it resolves an external entity.
- `secure_parser.py` rejects the same document before parsing it.

Run the regression tests with:

```bash
python -m unittest discover -s tests -v
```

## Repository map

| Path | Purpose |
|---|---|
| `lab/vulnerable_parser.py` | Minimal insecure XML parsing example |
| `lab/secure_parser.py` | Hardened parser implementation |
| `scripts/run-safe-demo.sh` | Local, non-destructive demonstration |
| `docs/architecture.md` | How the trust boundary can fail |
| `docs/security-review.md` | Review checklist and evidence to collect |
| `docs/remediation.md` | Concrete secure configuration guidance |
| `tests/` | Regression tests for the secure parser |

## Responsible use

Only run the lab against systems and files that you own or are explicitly authorized to test. For a production issue, record the smallest safe proof, notify the owner, rotate exposed secrets, and apply the remediation in this repository.

## License

MIT. See [LICENSE](LICENSE).
