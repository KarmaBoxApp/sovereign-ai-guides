[portable-agent-memory-spec.md](https://github.com/user-attachments/files/35065503/portable-agent-memory-spec.md)
# A portable memory file for AI agents: schema and migration checklist

In the 48 hours ending November 18, 2026, OpenAI, Google, and Microsoft each shipped new flagship models. If switching to any of them means re-teaching your agent months of corrections and house rules, the memory was living in the wrong place. This file gives you a vendor-neutral format for keeping agent memory in a file you control, plus a ten-question check for verifying that a move to a new model or runtime lost nothing.

Two scope notes. A memory file solves portability, how accumulated context moves between models, tools, and machines. It does not make a weaker model stronger, and it does not transfer behaviors that live in the model itself, such as tone or tool-calling quirks. Second, a chat export is not a memory file: it is a log. It records everything that was ever said, with no field saying which corrections still hold. The schema below exists to carry state, not history.

## 1. What goes in the file: five entry types

| Type | What it holds | Example statement |
|---|---|---|
| `fact` | Stable facts about the owner, team, or business | "Invoices go out on the last business day of the month" |
| `preference` | How the owner wants output shaped | "Summaries: three bullets max, no adjectives" |
| `correction` | Something the agent got wrong that must stay fixed | "The webhook uses the old signing key, not the one in the README" |
| `context` | Project state that outlives one conversation | "The Henderson proposal is on hold until their fiscal year opens" |
| `boundary` | What the agent must not do without asking | "Never send pricing to a new contact without approval" |

Episodic history (raw conversations) stays out of the file. Keep logs separately if you need them; mixing log and state is what makes chat exports useless for migration.

## 2. Schema

```json
{
  "format_version": "1.0",
  "subject": "agent:invoicing-assistant",
  "created": "2026-05-04",
  "updated": "2026-11-19",
  "entries": [
    {
      "id": "mem-0142",
      "type": "correction",
      "statement": "EU clients: VAT line must reference the reverse-charge mechanism, not a 0% rate.",
      "taught_by": "sam",
      "taught_on": "2026-09-12",
      "status": "active",
      "supersedes": "mem-0098",
      "scope": "invoicing",
      "last_verified": "2026-11-19"
    }
  ]
}
```

Field notes:

| Field | Why it exists |
|---|---|
| `statement` | Plain language, one claim per entry. The agent reads this; so do you, in a diff, a year later. |
| `taught_by`, `taught_on` | Memory without provenance cannot be audited or trusted after a personnel change. |
| `status` | `active` or `superseded`. This is the field a chat log lacks: whether the correction still holds. |
| `supersedes` | Points at the entry this one replaced, so the history of a rule survives without keeping stale rules active. |
| `scope` | Lets one file serve an agent with several duties, and lets you export a subset. |
| `last_verified` | Set by the golden-set check below. An entry nobody has verified in a year is a candidate for review, not deletion. |

## 3. What stays out of the file

- **Embedding vectors.** Vectors are tied to the embedding model that produced them; carried to a different one they are noise, and retrieval degrades without erroring. Store the source text and re-embed on arrival.
- **Credentials and tokens.** Memory files get copied, backed up, and diffed. Secrets follow their own storage.
- **Raw conversation logs.** See above: logs are history, this file is state.
- **Anything the model vendor generated about the user.** Derived profiles you did not write and cannot inspect do not belong in a file whose point is that you can read every line.

## 4. The migration golden set: ten questions

A golden question can only be answered correctly from taught memory, and its answer checks against a number or a document, not a feeling. Write the set, with expected answers, before the migration.

| # | Template | What it proves survived |
|---|---|---|
| 1 | A calculation using parameters you taught once ("net profit per unit for X under our cost assumptions") | `fact` entries with numbers |
| 2 | A house rule ("what day do invoices go out?") | `fact` |
| 3 | A correction, asked the way you would have before the correction | `correction`, and that it beats the stale default |
| 4 | A superseded rule ("what signing key does the webhook use?") | `supersedes` chain: new value wins |
| 5 | A formatting request with a taught preference ("summarize this") | `preference`, applied without being restated |
| 6 | A project-state question ("where does the Henderson proposal stand?") | `context` |
| 7 | "What are you not allowed to do with pricing?" | `boundary` entries, recited correctly |
| 8 | A retrieval question against a document in the knowledge base | the retrieval path, including re-embedding |
| 9 | A term of art your team uses with a private meaning | `fact`, terminology |
| 10 | A control question about something never taught | that the agent says it does not know instead of inventing |

Question 10 is the one most people skip. A new model that answers the nine real questions and also confidently answers the fake one has not inherited your memory; it is improvising over it.

## 5. Migration procedure

1. **Freeze.** Stop teaching for the duration; a moving target cannot be verified.
2. **Snapshot.** Copy the memory file and the golden-set sheet with expected answers.
3. **Baseline.** Ask all ten questions on the old binding and record the answers. Fix failures now.
4. **Rebind.** Point the new model or runtime at the memory file. Re-embed knowledge-base content if the embedding model changed.
5. **Re-ask.** Same ten questions on the new binding. Compare against the sheet, not against memory of the old answers.
6. **Hold the rollback.** Keep the old binding available for a week of real work. Most differences surface in use, not in the check.
7. **Record.** Update `last_verified` on the entries the passing questions touched.

| # | Question | Expected | Old binding | New binding | Pass |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |

Copy the table to ten rows and fill it during steps 3 and 5.

## What this does not cover

Model quality differences: the golden set verifies memory, and a new model can pass all ten questions while being worse at the work itself. Evaluate capability separately, on your own tasks. And nothing here prevents a vendor from discontinuing a model; it only guarantees that when that happens, what you taught your agent leaves with you.
