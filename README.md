# Sovereign AI Guides

Runbooks and templates for teams that let AI agents act on their behalf: send, publish, approve, spend. Each guide is a working method you can apply in an afternoon, with tables you can copy straight into your own docs. Nothing here assumes a specific vendor, model, or product.

## Guides

| Guide | What you get |
|---|---|
| [Confidence thresholds and human review](guides/confidence-thresholds-runbook.md) | Action tiers, error pricing, shadow-mode calibration, a review-ledger template, a threshold change log, and a pre-launch checklist |

| Guide | What you get |
 |---|---|
 | [Confidence thresholds and human review](guides/confidence-thresholds-runbook.md) | Action tiers, error pricing, shadow-mode calibration, a review-ledger template, a threshold change log, and a pre-launch checklist |
 | [Cloud outage continuity checklist](guides/cloud-outage-continuity-checklist.md) | A dependency inventory template, three questions that decide which tasks survive a provider outage (inference, files, login), what to move first, and a one-afternoon verification |
+| [Self-hosted AI agent evaluation](guides/self-hosted-ai-agent-evaluation.md) | Six groups of questions to put to any agent product before committing (credentials, memory export, where approvals are enforced, logs, license, offline and phone-home, upgrades), where to find each answer, and a scoring sheet |

| [AI stack residency and sovereignty audit](guides/ai-stack-residency-audit.md) | Map where each AI tool's vendor is incorporated, where inference runs, and what actually leaves your machines, with capture commands for observed traffic and a per-tool record template |

   - [Dependency mirroring: keep a copy of what your build pulls](guides/dependency-mirroring.md): mirrors for repos, release assets, and packages, set up before an upstream vanishes.

 | Portable agent memory: file spec and migration checklist | Five entry types with a JSON schema, field-by-field notes, what to keep out of the file, a ten-question golden set, and a migration procedure with rollback | 

  | Local task fit: scoring checklist and golden-set acceptance | A four-dimension fit score for deciding which task classes move to a local model, a ten-task golden set in YAML with a missing-input trap, pass-rate tiers, and a re-test log format | 

## Background

We are the team behind [KarmaBox](https://www.karmaboxai.com), a platform and network for running your own AI avatars. The methods here are the ones we use and recommend, written to work with any stack.

## Contributing and license

Issues and pull requests are welcome, corrections especially. Text is licensed under CC BY 4.0.
