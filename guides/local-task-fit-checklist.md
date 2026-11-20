[Uploading local-task-fit-checklist.md…]()
# Local task fit: a scoring checklist and a golden-set acceptance test

An interview that made the rounds this week put local open-weight models within months of the big commercial ones on benchmarks (The Data Exchange, November 19, 2026). Whether that holds for your work is not something a benchmark can tell you. It is a fit question, and you can settle it in an afternoon with the two tools in this guide: a four-dimension fit score that tells you which task classes to test first, and a ten-task golden set that makes the final call.

Everything here is vendor-neutral. It works with any local runtime, any model, and any hosted alternative you are comparing against.

## Score each task class on four dimensions

Work moves to a local model by class, not by individual task: "weekly report drafting", "invoice field extraction", "meeting transcription". For each class, score four dimensions from 0 to 2.

| Dimension | Question to ask | 2 | 1 | 0 |
|---|---|---|---|---|
| Boundary clarity | Is the input and output well defined? | Fixed input, fixed output shape | Some judgment in scope | Open-ended |
| Material at hand | Does everything the task needs fit in the prompt or a local index? | All of it | Most, with occasional lookups | Depends on outside systems |
| Verifiable answer | Can someone check the output against a number or a document? | Exact check | Human review with clear criteria | Taste call |
| Knowledge recency | Does a correct answer depend on information newer than the model? | No | Rarely | Routinely |

Totals: 6 to 8, strong local candidate, test it first. 4 to 5, candidate with mandatory spot-checks. 3 or less, poor fit, leave it hosted and move on.

The score is a paper exercise and it only decides the testing order. The golden set below decides whether anything actually moves.

## Five classes that usually score well

These five come up in most teams. Score your own versions rather than trusting the table; a "drafting" task that needs this week's market prices scores differently from one that rewrites your own notes.

| Class | Why it tends to fit |
|---|---|
| Drafting and rewriting | Boundary is the source text; answer is checkable against house style |
| Questions over your own documents | Material is local by definition; recency need is low |
| Structured extraction and calculation | Exact answers; fabrication is detectable |
| Transcription and image reading | Fixed input, fixed output; spot-checkable |
| Routine code | Tests are the verifier; boundary set by the ticket |

## Build a ten-task golden set

A golden task is a real task from the last month whose correct answer you have already checked by hand. Not an invented puzzle: puzzles test the model, real tasks test the fit. Ten per class is enough to make pass rates meaningful without turning acceptance into a project.

Composition that works: seven routine tasks, two edge cases that have burned you before, and one task with a deliberately missing input. The pass condition on that last one is that the model asks for the missing value. A model that invents it fails, whatever the rest of its score.

```yaml
golden_set:
  task_class: invoice-extraction
  model: ""          # fill at run time
  runtime: ""        # fill at run time
  date: ""
  tasks:
    - id: gs-01
      input: >
        Extract supplier name, net amount, VAT rate and due date
        from the attached invoice text. Text: [paste the real one]
      expected: "Meridian Logistics; 4,180.00; 19%; 2026-12-05"
      check_method: exact-string   # exact-number | exact-string | doc-match | human-check
    - id: gs-02
      input: >
        Unit cost 35, inbound freight 12 per unit, platform fee 15%,
        duty 25%, sale price 29.99, exchange rate 7.2.
        Calculate per-unit net profit.
      expected: "12.55"
      check_method: exact-number
      tolerance: 0.01
    - id: gs-10
      input: >
        Same calculation, but the duty rate is omitted on purpose.
      expected: "asks for the duty rate instead of assuming one"
      check_method: human-check
```

Rules that keep the set honest: expected answers are verified before they enter the file, not after; inputs are written exactly as you would hand them to a person; and the file is versioned, because the set is only useful if the same ten tasks are what you re-run later.

## Decide with pass rates

Run all ten on the local setup you are evaluating. No partial credit: a task passes its check or it does not.

| Passes out of 10 | Decision |
|---|---|
| 9 or 10 | Run the class locally |
| 6 to 8 | Run it locally with a weekly human spot-check on a sample |
| 5 or fewer | Keep it on the hosted model, re-test next quarter |
| Any fabricated value | Lowest tier regardless of total |

The fabrication override earns its place because an invented number is the one failure mode a spot-check regime cannot contain: the output looks exactly like a pass.

These tiers are starting values. After a month of records, teams usually tighten the spot-check band rather than the top tier.

## Re-test on every change, and quarterly

The golden set is cheap to re-run, which is the point. Re-run it after a model change, a runtime change, a quantization change, or a prompt template change, and once a quarter even if nothing changed. Keep a one-line log per run:

```
2026-11-20 | invoice-extraction | model-x q4 | 9/10 | local
2026-11-20 | report-drafting   | model-x q4 | 7/10 | local + weekly spot-check
```

A score that drops after a change points at the change. A score that drops with no change points at the inputs drifting, which is worth knowing about for reasons beyond model choice.
