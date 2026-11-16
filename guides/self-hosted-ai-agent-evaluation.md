# Self-hosted AI agent evaluation

A self-hosted agent is only as self-hosted as its least local layer. An agent that runs on your machine but keeps its credential vault, its policy engine or its license check in the vendor's cloud fails on the vendor's worst day, whatever the landing page says.

The questions below are vendor-neutral and answerable from public documentation, a repository and one test install. Allow about an hour per candidate. Score each line, then compare totals across your shortlist.

## 1. The five layers to check

An agent that acts on your behalf has five layers. Four hold custody and should sit inside your boundary; one can be rented.

| Layer | What it holds | Should be local? |
|---|---|---|
| Credentials | API keys, OAuth tokens, session cookies, payment methods | Yes |
| Memory | What the agent has learned about your work | Yes, with a readable export |
| Policy | Rules for what the agent may do without asking | Yes, enforced outside the model |
| Logs | Every attempted action, rule applied, outcome | Yes (primary), copy elsewhere |
| Inference | The model call | No requirement; rent per task |

## 2. Before you commit to an agent platform, what should you get in writing?

For each question: where to look, what passes, what fails.

**Credentials**

- Where are credentials stored when the vendor's cloud is unreachable? Look in the docs' storage or data-directory page. Pass: on your disk, usable offline. Fail: vendor vault, or "synced for convenience" with no off switch.
- Can the model see raw secrets? Look for how tools receive keys. Pass: keys injected at call time, never in prompt context. Fail: keys in system prompts or memory.
- How are logins that need a human handled? Pass: the agent hands the browser back for passwords and second factors and keeps only the session. Fail: the agent asks you to type passwords into chat.

**Memory**

- Is there a documented export, and in what format? Pass: JSON, Markdown or another open format, named in the docs. Fail: no export section, or a proprietary archive.
- Does the product keep working after you export? Pass: export is a read operation. Fail: export locks or degrades the account.
- Test: export in week one and open the file.

**Approvals**

- Is the approval check enforced by the platform or requested of the model? Read the verbs on the security page: "the model will confirm" and "the platform will block" are different promises. Pass: per-action-class rules (allow, ask, deny) evaluated in code. Fail: approval is a prompt instruction or a second model.
- Can the agent edit its own policy? Test from the agent's user account. Pass: write fails. Fail: write succeeds, or the docs don't say.
- What happens with an action type nobody configured? Pass: defaults to ask. Fail: defaults to allow.

**Logs**

- Are refused and held actions logged, not only completed ones? Pass: yes.
- Is the log append-only or tamper-evident, and does it live on your disk? Pass: local file, append-only or hash-chained, exportable. Fail: vendor dashboard only.

**License and continuity**

- What does the license permit if the company stops tomorrow? Look at the repository front page. Pass: an OSI-approved license (for example Apache-2.0, MIT, AGPL-3.0). Fail: source-available with use restrictions, or closed.
- Is any part closed (control plane, license server, sync service)? Pass: none, or optional and documented.

**Offline and phone-home**

- What still contacts the vendor? Search the docs for "telemetry" and "usage data." Pass: a short, documented list with an off switch. Fail: undocumented, or mandatory.
- Is offline operation behind a pricing tier? Pass: available on the plan you'd use. Note: some vendors reserve fully offline deployment for higher tiers; that is a disclosed trade-off, but price it in.
- Test: install, disconnect the network, run one routine task end to end.

**Upgrades**

- Can you pin a version and read a changelog before upgrading? Pass: tagged releases, changelog, auto-update optional.
- Do upgrades change policy defaults? Pass: changelog calls out default changes explicitly.

## 3. Scoring sheet

Score each line 2 (pass), 1 (partial or undocumented but plausible), 0 (fail). Any 0 on the three starred lines is disqualifying regardless of total.

| Check | Score (0 to 2) |
|---|---|
| ★ Credentials stored locally, usable offline | |
| Secrets never in model context | |
| Human hand-off for logins | |
| ★ Readable memory export, documented format | |
| ★ Approvals enforced by platform, agent cannot edit policy | |
| Unknown actions default to ask | |
| Refusals logged, log local and tamper-evident | |
| OSI license, no mandatory closed component | |
| Phone-home documented with off switch | |
| Offline operation on your plan | |
| Version pinning and changelog | |
| **Total (max 22)** | |

## 4. What a passing score does not tell you

The score measures custody: where keys, memory, rules and records live. It does not evaluate model quality, which you test on your own tasks. It does not evaluate your operations: a fully local agent on an unpatched machine with no tested backup scores well here and still loses everything when the disk dies. Backup and patching need their own checklist.

## Sources

- n8n Labs, [Meet Nod](https://blog.n8n.io/meet-nod/), 2026-11-10 (example of local custody of data, memory and saved logins, and of browser hand-off for human sign-in)
- CopilotKit, [Introducing OpenBot 1.0](https://www.copilotkit.ai/blog/introducing-openbot-1-0), 2026-11-10 (example of platform-enforced boundaries; fully offline deployment on an annual plan)
- Open Source Initiative, [OSI Approved Licenses](https://opensource.org/licenses), accessed 2026-11-16
