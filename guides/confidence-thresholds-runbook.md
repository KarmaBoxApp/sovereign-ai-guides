# Confidence Thresholds and Human Review: A Working Runbook

Two things happened on November 11, 2026 that make the method worth writing down. [Google](https://blog.google/innovation-and-ai/technology/developers-tools/gemini-decisions-api/) started selling machine judgment as a separate product: the Gemini Decisions API answers bounded questions (yes/no, multiple choice, rubric scores) and attaches a confidence score between 0 and 1 to every answer, priced at $0.10 per million input tokens with output free (Google, 2026-11-11; performance figures are the vendor's own). Google's own documentation says the score exists so that calling code can route ambiguous cases to a fallback or a human reviewer. The same day, [The Verge](https://www.theverge.com/ai-artificial-intelligence/1031623/united-unauthorized-ai-agents) confirmed that United Airlines will not let third-party AI agents complete ticket purchases, partly because US federal rules require the passenger to personally attest to certain declarations, including the statement that a checked bag contains no explosives.

A confidence threshold is the point on a model's 0-to-1 confidence scale below which an automated system stops acting on its own answer and hands the decision to a person or a fallback process. It is a routing rule, and a routing rule is only as good as the tiers, prices, and records behind it. The full setup takes about four working sessions and no machine-learning staff: tier the actions, price both kinds of error, calibrate in shadow mode, then run with a ledger.

## 1. Tier actions by consequence, not frequency

List every action the agent can take: send, publish, purchase, delete, reply, archive, approve. For each one, ask what the worst single wrong execution costs, in money, in trust, and in legal exposure. Three tiers cover most teams.

| Tier | Criteria | Examples | Score check | Who approves |
|---|---|---|---|---|
| 1 | Reversible and cheap | internal drafts, tagging, routing a ticket | none, the agent acts | nobody per case; sampled audits only |
| 2 | Reversible but externally visible, or costs real money | customer replies, refunds under a cap | act above threshold, queue below it | a named reviewer owns the queue |
| 3 | Irreversible, legally significant, or signature-class | payments over a cap, contract signatures, deletions without backup, legal attestations | no score is accepted | a person, every time; the agent only prepares |

One category fails the tier test at any score: signature-class decisions. A legal attestation means a specific person vouched for a fact under penalty, and the act loses its meaning when software performs it. A perfect forgery is still a forgery. United's booking ban is this category enforced by a counterparty ([The Verge](https://www.theverge.com/ai-artificial-intelligence/1031623/united-unauthorized-ai-agents), 2026-11-11), and counterparties move: eBay has closed its site to shopping agents and several large banks limit agent access to accounts, per the same report. Two consequences for planning. Write the never-automate list before any arithmetic; it is usually shorter than feared, and everything outside it can be automated with a clear conscience. And treat counterparty refusal as its own tier with a tested human fallback path, so a policy change on their side degrades your workflow instead of stopping it.

## 2. Price both kinds of error

Every threshold trades two costs against each other. Set it high and people review judgments that were already right: salary spent buying comfort. Set it low and errors ship: rework, apologies, cleanup. Write both prices down, roughly is fine.

- A shipped error costs the direct loss plus cleanup (example: a wrongly approved $40 refund runs about $55 all-in, counting 20 minutes of staff time at a $45/hour loaded rate).
- A needless review costs reviewer attention (example: about 90 seconds per case, roughly $1.10 at the same loaded rate).

At those example prices, one caught error pays for about 50 needless reviews, so routing a borderline $35 case to a person breaks even if even one review in fifty catches a real mistake. The arithmetic takes a napkin. What matters is that a named person signs the ratio, because that signature is what turns the first incident into a data point inside a policy instead of a referendum on automation.

Price the two directions separately when they differ. For a refund queue, a false approval costs money and a false rejection costs a customer; for a content filter, the asymmetry flips. When one direction is much more expensive, set two thresholds instead of one: auto-act above the high line, auto-decline below the low line, and queue only the band in between. The queue shrinks to the cases where human judgment actually changes the outcome.

## 3. Calibrate in shadow mode

A score is only meaningful if the system is calibrated: in a well-calibrated batch, about 90% of answers scored 0.90 turn out right. Calibration is a property of batches, never of a single answer, and it is measured, never assumed.

1. Turn on scoring for every decision in the target category. Humans keep deciding everything; nobody's job changes yet.
2. Log four things per case: an input reference, the model's answer, the score, and the human's decision.
3. Run for two to four weeks, or until at least 500 cases have accumulated (smaller samples make per-bucket rates too noisy to act on).
4. Bucket the cases by score: 0.70 to 0.80, 0.80 to 0.90, 0.90 to 0.95, 0.95 to 0.99, 0.99 and up.
5. For each bucket, compute how often the model's answer disagreed with the human's decision.
6. Keep the curve. It is the evidence behind every threshold argument the team will have later.

Two practical notes on the buckets. Uneven traffic is normal: most volume lands in the top band, so the low buckets may need the full four weeks to fill. And keep the raw case list per bucket, because the first question after any surprising rate is always about which cases produced it.

A caution about where scores come from. A specialized decision model computes real probabilities over a closed answer set. A general chat model asked in the prompt to "also output your confidence" produces a plausible-sounding number without that computation, and public evaluations repeatedly find such self-reported numbers poorly calibrated. Until shadow data on your own traffic says otherwise, treat prompt-extracted confidence as untrusted.

## 4. Set the threshold, then move it on evidence only

Open conservatively: automate only the top band. Starting at 0.95 is common, but the disagreement curve from shadow mode should pick the number, matched against the error prices from step 2. Expand downward only after the measured error rate in the automated band has stayed inside budget for a full review cycle. Tighten immediately when drift appears. During an incident, changes go in one direction only, toward more human review; loosening waits for the postmortem.

Recalibrate monthly on recent traffic, and immediately after a product change, a new customer segment, a seasonal shift in request mix, or a vendor model update. Vendors rarely announce that last one in a way anyone notices; the monthly bucket check is usually how it surfaces.

## 5. Wire the handoff so it fails safely

Three circuits catch what the threshold cannot, and each one covers a different failure.

- Queue with a cap. Low-score cases go to a named reviewer whose daily volume is sized to real attention, minutes per day, written down. Track the disagreement rate: a reviewer who never overrules the model is a rubber stamp, and the organization is paying for oversight it no longer has.
- Sampled audits of the automated band. Per-case thresholds miss aggregate damage: a thousand small auto-approvals at 1% error are ten quiet mistakes that no single case escalates. A periodic random sample of the auto-approved band is the only instrument that sees them.
- Hard ceilings by category. When shadow data shows a category where scores stay high while answers stay wrong, move the whole category to human review regardless of score. Category ceilings also catch what score bands cannot: fraud-flagged inputs, amounts above a cap, first-time counterparties.

Decide the outage behavior in advance: when the judgment layer is down, tier-2 actions queue and wait. Failing closed costs delay; failing open costs whatever the worst queued action costs.

## 6. The review ledger

The ledger matters more than the threshold. Thresholds change, and the ledger is how anyone knows they should.

| Field | What to record | Why it is there |
|---|---|---|
| Input reference | a durable pointer, not a copy that can expire | re-examine the case months later |
| Question asked | the exact bounded question | answers are meaningless without it |
| Answer and score | verbatim, with the score | calibration math needs both |
| Threshold version | which policy version was in force | explains why routing happened |
| Reviewer | a named person | accountability has a location |
| Decision | approve, reject, or modify | disagreement rate comes from here |
| Timestamp | when the review happened | drift shows up on a time axis |
| Outcome | chargeback, complaint, or clean, when known | ties decisions to consequences |

Keep the ledger exportable. An audit trail that lives only in a vendor console can leave with the contract.

## 7. The threshold change log

A threshold nobody owns gets changed in a hurry and forgotten by everyone. Give it a named owner, a quarterly review against the latest calibration, and an explicit handover when the owner changes roles.

| Date | Change | Reason | Approved by | Recalibration |
|---|---|---|---|---|
| 2026-01-14 | 0.95 to 0.97 | drift detected in weekly report, auto-band error 1.1% to 3.0% | J. Doe, ops lead | scheduled 2026-01-21 |

## 8. Before switching anything on

| # | Question | A pass looks like |
|---|---|---|
| 1 | Which decisions are signature-class? | a written never-automate list, signed by the person who carries the consequences |
| 2 | Is every agent action tiered? | three tiers, each with a named owner |
| 3 | Are both error types priced? | one number for a shipped error, one for a needless review, rough is fine |
| 4 | Was calibration measured on your own traffic? | a disagreement curve from shadow mode, dated within the last quarter |
| 5 | Is the queue sized to human attention? | a volume cap, and a disagreement rate someone tracks |
| 6 | Does the ledger survive a vendor switch? | records export completely, scores and review actions included |
| 7 | Can a person stop an action mid-flight? | a tested stop control, separate from the threshold |

Any hesitant answer means that action stays with a person for now.

## What the method does not cover

Three adjacent problems need their own machinery. Stopping an agent mid-action is a control problem, separate from routing: a threshold decides whether an action starts, and something else has to be able to interrupt one that already did. The judgment layer itself is a dependency: bought as an API, it can be repriced, deprecated, or silently updated. The monthly recalibration is the alarm; the exit plan, a second provider or a small self-hosted classifier for the sensitive tiers, is the insurance. And no threshold substitutes for a compliance review where regulation names a decision as human: the never-automate list records those entries, it does not discover them. Legal review of that list belongs in step 1, done once and revisited when the rules change.

## Sources

- Google, [Introducing the Gemini Decisions API](https://blog.google/innovation-and-ai/technology/developers-tools/gemini-decisions-api/), 2026-11-11
- The Verge, [United won't let your AI agent book your flight](https://www.theverge.com/ai-artificial-intelligence/1031623/united-unauthorized-ai-agents), 2026-11-11
