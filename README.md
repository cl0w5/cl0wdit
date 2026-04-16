# substrate_cl0wdit

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for security auditing Rust and Substrate codebases, trained on 30+ ecosystem audit reports curated by [PAL](https://dotpal.io/). Parallelizes multiple scanning agents across different attack vector categories, then merges and deduplicates findings into a single report.

## Setup

**Per-project** — copy into a specific project's skill directory:

```bash
cp -r substrate_cl0wdit /path/to/your-substrate-project/.claude/skills/
```

**Global** — copy into your home directory to make it available across all projects:

```bash
cp -r substrate_cl0wdit ~/.claude/skills/
```

## Usage

```
/substrate_cl0wdit                     # audit current directory
/substrate_cl0wdit --pr 123            # audit a specific PR
/substrate_cl0wdit --pr 123 --deep     # + adversarial reasoning agent (opus, slower)
/substrate_cl0wdit --file-output       # also write report to findings/ directory
```

Flags are combinable.

## How it works

The orchestrator discovers in-scope `.rs` files, bundles them with attack vector references, and spawns parallel agents:

| Agent | Model | Focus |
|-------|-------|-------|
| 1 | Sonnet | Substrate & Rust common vulnerabilities |
| 2 | Sonnet | Runtime config, XCM & weight vectors |
| 3 | Sonnet | DeFi, access control & crypto vectors |
| 4 | Sonnet | Test & benchmark coverage gaps |
| 0 | Opus | Adversarial reasoning (`--deep` only) |

Each vector-scan agent (1-3) receives the full production codebase but a different attack vector reference file. Agent 4 reviews test/benchmark/mock code. Agent 0 reasons freely without predefined vectors.

Findings pass a 3-check false-positive gate and receive a confidence score (0-100). The orchestrator deduplicates by root cause and produces a sorted report.

## Attack vector sources

Distilled from 30+ audit reports across the Polkadot ecosystem (2022-2025), curated by [PAL](https://dotpal.io/):

- Auditors: Runtime Verification, Code4rena, SRLabs, Pashov, OAK Security, Cantina, CoinFabrik, Zellic, Beosin, and others
- Projects: Astar, Acala, Moonbeam, Peaq, InvArch, t3rn, Hyperbridge, Bifrost, KILT, Frontier, and more
- CoinFabrik Scout Audit detector patterns for Substrate pallets and Rust

## File structure

```
substrate_cl0wdit/
├── SKILL.md                             # orchestrator instructions
└── references/
    ├── agents/
    │   ├── adversarial-reasoning-agent.md
    │   ├── test-benchmark-agent.md
    │   └── vector-scan-agent.md
    ├── attack-vectors/
    │   ├── substrate-attack-vectors.md    # ecosystem audit findings (30 reports)
    │   ├── substrate-attack-vectors-1.md  # common Substrate/Polkadot vulns
    │   └── substrate-attack-vectors-2.md  # DeFi, access control & crypto patterns
    ├── judging.md                         # FP gate + confidence scoring
    ├── known-false-positives.md           # patterns to suppress
    └── report-formatting.md               # output template
```

## Customization

To add project-specific attack vectors, create a new `.md` file in `references/attack-vectors/` and add a corresponding agent in `SKILL.md` following the existing pattern.

To add known false positives for your project, append entries to `references/known-false-positives.md` following the `FP-NNN` format.
