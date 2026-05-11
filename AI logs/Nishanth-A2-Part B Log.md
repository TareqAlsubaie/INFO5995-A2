# AI Usage Log — Part B Bug Bounty Research (Member 2)

## Assignment Context

This log documents my personal AI-assisted workflow for the Part B component of INFO5995 Assignment 2. My contribution focused on vulnerability classification, remediation design, report writing, and preparation of the Part B presentation and Q&A defence materials for our `zx` dangling symlink finding submitted to the Google OSS Vulnerability Reward Program.

Primary working directory:

```text
~/info5995/partb
```

---

## AI Tools Used

- Claude (claude.ai) was used as the primary assistant for report structuring, severity mapping, and presentation scripting.
- VS Code with the GitLens extension was used to navigate the `google/zx` repository and trace call sites.
- Node.js was used to inspect the `zx` build output and verify that the vulnerable `getFilepath()` function was present in the distributed package.
- Google OSS VRP bughunters.google.com portal was used for the actual submission.
- Google's HackerOne severity taxonomy and CVSS v3.1 calculator were used for severity calibration.

---

## Ethical and Scope Controls

- Testing was performed only against a locally cloned repository. No live service, network endpoint, or third-party system was targeted.
- The `google/zx` repository is within the Google OSS VRP scope, which is an explicitly allowed platform in the assignment specification.
- The PoC was executed only in an isolated local directory created specifically for this purpose. No sensitive files were targeted or modified during testing.
- The submission was made through the official Google bug reporting portal with responsible disclosure framing.
- AI-generated content was reviewed and revised before being included in any submission or report.

---

## High-Level Workflow

### 1. Understanding the Google OSS VRP Rules

**Prompt summary:**

I provided the Google OSS VRP rules page text to the AI and asked it to summarise which issue types were eligible for rewards under OT2/OT3 tier projects, and what submission requirements applied.

**AI output summary:**

The AI extracted that OT2/OT3 tier projects require a committed and merged patch to accompany the vulnerability report before a bounty can be awarded. It noted that memory corruption reports additionally require OSS-Fuzz reproduction steps. It flagged that supply chain and credential leak vulnerabilities are exempt from the patch requirement.

**How I used it:**

This helped me understand upfront that our finding would need a patch PR to qualify for a monetary reward, and informed how we framed the submission — focusing on the technical validity of the vulnerability rather than reward eligibility alone. It also explained Google's eventual closure reason, which was procedural rather than a technical rejection.

---

### 2. Confirming Vulnerability Class and CWE Mapping

**Prompt summary:**

I described the `getFilepath()` and `runScript()` code pattern to the AI and asked it to identify the most accurate CWE classification and explain whether this was a TOCTOU, symlink following, or path resolution issue.

**AI output summary:**

The AI identified the primary weakness as CWE-59 (Improper Link Resolution Before File Access), explaining that the issue arises specifically from `fs.existsSync()` resolving symlinks before reporting existence, which creates a gap between the availability check and the write operation. It noted this is distinct from a classical TOCTOU (CWE-362) because no race window is required — the dangling symlink is already in place before `zx` runs, making exploitation fully deterministic.

**How I used it:**

I used this to correctly classify the finding in the report and to prepare a clear explanation of why the issue is not just a theoretical race condition, which strengthens the impact argument.

---

### 3. Severity Mapping and CVSS Scoring

**Prompt summary:**

I asked the AI to help compute a CVSS v3.1 base score for our finding and to map it to the rubric's normalised severity tiers.

**AI output summary:**

The AI walked through each CVSS v3.1 metric:

- Attack Vector: Local (L) — attacker must place a file in the victim's working directory
- Attack Complexity: Low (L) — no race condition or special timing required
- Privileges Required: None (N) — attacker only needs filesystem write in the shared directory
- User Interaction: Required (R) — victim must run `zx` from the affected directory
- Scope: Changed (C) — write escapes the intended temp-file boundary into the broader filesystem
- Confidentiality Impact: None (N)
- Integrity Impact: High (H) — attacker controls file creation content and path
- Availability Impact: None (N)

Resulting base score: **6.3 (Medium)**. The AI noted this could be argued upward to High if the follow-on code execution path is demonstrated, but recommended staying at Medium for the submitted PoC evidence.

**How I used it:**

I used the score to map to the rubric's S2 tier (Medium, 3.0 severity score), and drafted the severity section of the report with explicit CVSS metric justification rather than a self-asserted severity level.

---

### 4. Remediation Design

**Prompt summary:**

I asked the AI to suggest the most technically sound remediation options for the `getFilepath()` function, and to explain the trade-offs between each approach.

**AI output summary:**

The AI proposed three remediation options in order of strength:

1. Replace `fs.existsSync()` with `fs.lstatSync()` wrapped in a try-catch, so dangling symlinks are detected as occupied paths and excluded from selection. This is a minimal targeted fix.
2. Move temporary file creation to `os.tmpdir()` with a random prefix, eliminating attacker influence over the temp directory entirely. This is architecturally stronger.
3. Use `fs.open()` with `O_CREAT | O_EXCL` flags for atomic exclusive file creation, which is the most robust option but requires more refactoring.

It also noted that the cleanup logic (`rmrf`) separately needed to be reviewed if the fix moved temp files to a trusted directory, since the symlink-vs-file check in `rmrf` would become irrelevant.

**How I used it:**

I included all three options in the remediation section of the report, ordered from minimal to comprehensive. This demonstrates understanding of the root cause rather than just proposing a surface fix.

---

### 5. Report Drafting and Structure Review

**Prompt summary:**

I asked the AI to review a draft of our vulnerability report and identify sections that were too vague, overclaimed impact, or lacked supporting evidence.

**AI output summary:**

The AI flagged two issues in the draft:

- The impact section conflated confirmed file creation with speculative code execution without clearly labelling which was demonstrated and which was a follow-on scenario. It recommended splitting these into two sub-sections with explicit labels.
- The exploit scenario section did not clarify that `~/.bash_aliases` was used as a PoC target specifically because it did not already exist in the test environment — without this note, a reviewer might question whether the issue only works in unusual conditions.

**How I used it:**

I revised the impact section to separate confirmed and plausible impact explicitly, and added a note to the exploit scenario clarifying the pre-conditions for the chosen target path. Both changes improved the report's clarity and reduced the risk of a triage team dismissing the impact as overstated.

---

### 6. Presentation Script and Q&A Preparation

**Prompt summary:**

I provided the assignment rubric and our vulnerability summary to the AI and asked it to generate a five-minute presentation outline and a set of likely Q&A challenge questions with suggested evidence-backed answers.

**AI output summary:**

The AI produced a five-minute outline with the following structure:

- 0:00–0:30 — Target introduction: what `zx` is and why it is in scope
- 0:30–1:30 — Vulnerability explanation: `getFilepath()` flaw, `fs.existsSync()` semantics, write-through via `fs.writeFile()`
- 1:30–2:30 — PoC walkthrough: symlink setup, `zx --eval` trigger, resulting file creation, cleanup leaving target intact
- 2:30–3:30 — Impact: confirmed file creation primitive, follow-on persistence and CI/CD scenarios
- 3:30–4:30 — Novelty and classification: prior disclosure search results, Type 2 zero-day candidate rationale
- 4:30–5:00 — Remediation: `lstatSync` fix and `os.tmpdir()` migration

Q&A questions generated:

- *"The CVSS score is Medium — why is this worth our time?"* — Answer: the file creation primitive is strong and environment-dependent; Medium severity with full impact evidence and novelty scores better under the rubric than a claimed-High with weak evidence.
- *"How do you know this is a zero-day?"* — Answer: GitHub issue search, closed PR search, CVE database search, and web search all returned no prior disclosure at submission time.
- *"Did Google say this was not a vulnerability?"* — Answer: No. Google closed the report because a merged patch was not included, as required by OT2/OT3 tier submission rules. The technical validity was not disputed.

**How I used it:**

I used the outline to structure the recorded presentation and distributed the Q&A answers across team members so everyone had a clear section to own during the tutorial Q&A.

---

## Human Review and Corrections

- I manually verified the CVSS metric selections by reading the official CVSS v3.1 specification rather than accepting AI-selected values without review.
- I adjusted the severity framing downward from High to Medium after working through the CVSS metrics carefully, because the PoC demonstrated file creation only and not code execution.
- I revised the impact section after the AI identified that speculative and confirmed impact were mixed.
- I reviewed Google's closure response myself and confirmed the procedural nature of the rejection before using it as an argument in the presentation.

---

## Limitations

- The CVSS score is based on the locally demonstrated PoC. If additional exploitation steps (e.g., shell startup file execution) were demonstrated end-to-end, the score could be revised upward.
- Google has not provided a formal severity assignment, so CVSS is used as the fallback per the rubric's severity precedence rule.
- The novelty claim is based on a good-faith search for prior disclosure. It cannot be guaranteed that no non-public prior report exists.

---

## Reflection

AI assistance was most useful in two areas: CWE classification (where the distinction between CWE-59 and CWE-362 was non-obvious and mattered for accuracy) and CVSS scoring (where working through each metric systematically prevented the team from defaulting to an unsupported severity claim).

The main risk of AI use was in report tone. Early drafts produced by the AI had overly confident language about follow-on code execution scenarios. Manual review caught this and softened the language to accurately reflect what the PoC demonstrated versus what was plausible. Keeping confirmed and speculative impact clearly separated was the most important quality improvement made during the drafting process.
