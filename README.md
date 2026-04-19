# substrate_cl0wdit

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for security auditing Rust and Substrate codebases, trained on 30+ ecosystem audit reports curated by [PAL](https://dotpal.io/). Parallelizes 10 specialized hacking agents, then deduplicates and gate-validates findings into a single report.

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
/substrate_cl0wdit --file-output       # also write report to findings/ directory
```

Flags are combinable.

## How it works

The orchestrator discovers in-scope `.rs` files, bundles them with attack vector references, and spawns 10 parallel agents:

| Agent | Focus |
|-------|-------|
| 1–3 | Vector scanning (each against a different attack vector reference) |
| 4 | Math precision — arithmetic, rounding, overflow, type conversions |
| 5 | Access control — origin checks, proxy bypass, XCM origin confusion |
| 6 | Economic security — token behaviors, oracle manipulation, weight underpricing |
| 7 | Execution trace — hooks, cross-pallet calls, state flow, TOCTOU |
| 8 | Invariant — conservation laws, state couplings, pool math |
| 9 | First principles — assumption violation, unnamed bug classes |
| 10 | Test & benchmark coverage gaps |

Findings are validated through a 4-gate system (Refutation → Reachability → Trigger → Impact), scored by confidence, and deduplicated by root cause. Partial findings that don't fully qualify are reported as **Leads** for manual review.

## Attack vector sources

Distilled from 30+ audit reports across the Polkadot ecosystem (2022-2025), curated by [PAL](https://dotpal.io/):

- Auditors: Runtime Verification, Code4rena, SRLabs, Pashov, OAK Security, Cantina, CoinFabrik, Zellic, Beosin, and others
- Projects: Astar, Acala, Moonbeam, Peaq, InvArch, t3rn, Hyperbridge, Bifrost, KILT, Frontier, and more
- CoinFabrik Scout Audit detector patterns for Substrate pallets and Rust

## File structure

```
substrate_cl0wdit/
├── SKILL.md                                 # orchestrator instructions
├── VERSION                                  # version for update checks
└── references/
    ├── hacking-agents/
    │   ├── shared-rules.md                  # common output format & rules
    │   ├── vector-scan-agent.md
    │   ├── math-precision-agent.md
    │   ├── access-control-agent.md
    │   ├── economic-security-agent.md
    │   ├── execution-trace-agent.md
    │   ├── invariant-agent.md
    │   ├── first-principles-agent.md
    │   └── test-benchmark-agent.md
    ├── attack-vectors/
    │   ├── substrate-attack-vectors.md      # ecosystem audit findings (30 reports)
    │   ├── substrate-attack-vectors-1.md    # common Substrate/Polkadot vulns
    │   └── substrate-attack-vectors-2.md    # DeFi, access control & crypto patterns
    ├── judging.md                           # 4-gate validation + confidence scoring
    ├── known-false-positives.md             # patterns to suppress
    └── report-formatting.md                 # output template
```

## Customization

To add project-specific attack vectors, create a new `.md` file in `references/attack-vectors/` and add a corresponding agent in `SKILL.md` following the existing pattern.

To add known false positives for your project, append entries to `references/known-false-positives.md` following the `FP-NNN` format.
