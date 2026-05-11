# AI Usage Log — Part B Bug Bounty Research (Member 1)

## Assignment Context

This log documents my personal AI-assisted workflow for the Part B in-the-wild vulnerability hunting component of INFO5995 Assignment 2. It is separate from the Part A APK analysis log. My work focused on target selection, source-code review of the `zx` CLI tool, local proof-of-concept development, and impact reasoning for the dangling symlink finding.

Primary working directory used during research:

```text
~/bugbounty/zx-research
```

---

## AI Tools Used

- Claude (claude.ai) was used as an interactive research and report-drafting assistant throughout.
- Node.js (v20 LTS) was used locally to clone, build, and manually exercise `zx` during proof-of-concept development.
- GitHub was used to browse the `google/zx` repository history, open issues, and prior pull requests to check for prior disclosure.
- Google OSS VRP rules page was reviewed to confirm scope and submission requirements.
- Bash was used for local symlink preparation and PoC execution.

---

## Ethical and Scope Controls

- All testing was performed locally against a cloned copy of the `google/zx` repository. No live production systems were tested.
- The `google/zx` repository is explicitly in scope under the Google OSS VRP program, which is an allowed platform under the assignment spec.
- No denial-of-service, brute force, social engineering, or access to third-party data was performed at any point.
- The PoC was designed to demonstrate file creation in a controlled local environment only, using a non-sensitive dummy target path.
- Google OSS VRP submission was made through the official bughunters.google.com portal, not via any unofficial channel.
- AI assistance was used only for research, analysis, and drafting. All claims were manually reviewed before submission.

---

## High-Level Workflow

### 1. Target Selection and Scope Verification

**Prompt summary:**

I asked the AI to help identify open-source repositories in the Google OSS VRP scope that were written in JavaScript or TypeScript, had active maintenance, and involved file system or process execution logic that could be interesting from a security perspective.

**AI output summary:**

The AI identified several candidates including `google/zx`, `google/wireit`, and `google/gts`. It noted that `zx` was particularly interesting because it wraps shell execution and handles temporary file creation for script inputs, which is a class of code that historically carries symlink and race-condition risk.

**How I used it:**

I selected `google/zx` as the primary target based on this reasoning and began cloning and reviewing the repository locally.

---

### 2. Source Code Review — Identifying the Vulnerable Pattern

**Prompt summary:**

I provided the AI with the contents of `src/cli.ts` from `google/zx` and asked it to identify any insecure file system operations, particularly around temporary file creation.

**AI output summary:**

The AI identified the `getFilepath()` function, which selects candidate temp file paths using `fs.existsSync()`. It flagged that `fs.existsSync()` resolves symlinks before checking existence, meaning a dangling symlink whose target does not yet exist returns `false`. This means an attacker-controlled symlink in the working directory could be selected as the temp file path.

It also identified `runScript()` as the downstream consumer, which calls `fs.writeFile(tempPath, script)` without re-verifying that the resolved path is safe. The cleanup function `rmrf()` was identified as removing only the symlink itself via `fs.unlinkSync`, leaving any newly created target file intact.

**How I used it:**

I used this analysis to confirm the code flow manually before building the PoC, tracing each function call in the source to ensure the AI's reading of the code was accurate.

---

### 3. Local Proof-of-Concept Development

**Prompt summary:**

I asked the AI to outline the minimum steps needed to demonstrate the vulnerability locally and to suggest a safe non-destructive target path for PoC purposes.

**AI output summary:**

The AI suggested creating a dangling symlink named `zx.mjs` pointing to a file path that did not yet exist (e.g., `~/poc-output-test.txt`) and then running `zx --eval 'console.log("poc")'` from the same directory. It noted that the PoC would succeed if `poc-output-test.txt` was created with the script content, because the symlink was followed during `fs.writeFile()`.

It also clarified that the randomised fallback filename (`zx-<randomId>.mjs`) would only be used if `zx.mjs` was already occupied, meaning placing the symlink as `zx.mjs` was sufficient for reliable exploitation.

**How I used it:**

I ran the PoC exactly as described in a fresh local directory, confirmed the behaviour, and saved the terminal output as evidence.

Evidence produced:

```text
~/bugbounty/zx-research/poc/
  setup.sh
  poc-run-output.txt
  poc-output-test.txt   ← created via symlink follow-through
```

---

### 4. Novelty Check — Prior Disclosure Search

**Prompt summary:**

I asked the AI to help me check whether this issue had been previously reported or publicly disclosed, and to suggest what search terms and locations to check.

**AI output summary:**

The AI suggested checking the `google/zx` GitHub issue tracker and closed PRs for terms like `symlink`, `existsSync`, `lstatSync`, `tempPath`, and `getFilepath`. It also suggested checking the Google OSS VRP public reports page and general web searches combining `zx` with `symlink` and `arbitrary file`.

**How I used it:**

I performed all suggested searches manually. I found no public issues, advisories, CVEs, or prior reports describing this issue at the time of submission. This supported classifying the finding as a Type 2 zero-day candidate.

---

### 5. Impact Reasoning and Severity Calibration

**Prompt summary:**

I asked the AI to help reason through the realistic worst-case impact of an arbitrary file creation primitive targeting a developer tool like `zx`.

**AI output summary:**

The AI structured the impact in two tiers. The directly demonstrated impact is arbitrary file creation at an attacker-chosen path with the victim's user privileges, provided the target path is absent and writable. The plausible follow-on impact includes persistence via shell startup files, code execution via autostart entries or hook files, and CI/CD pipeline compromise via workspace path manipulation. It mapped this to CWE-59 (Improper Link Resolution Before File Access) and suggested a CVSS v3.1 base score in the High range.

**How I used it:**

I used this reasoning to write the impact section of the report. I was careful to separate confirmed impact (file creation demonstrated locally) from plausible follow-on impact, to avoid overstating exploitation certainty.

---

### 6. Rubric Mock Q&A

**Prompt summary:**

I provided the assignment rubric to the AI and asked it to simulate tutor Q&A questions about our Part B finding, and to identify gaps in our evidence or argumentation.

**AI output summary:**

The AI generated the following challenge questions and suggested answers:

- *"Why is this High and not Medium? The attacker needs access to the working directory."* — The AI suggested emphasising that shared directories, untrusted repository clones, and CI/CD workspaces are realistic real-world attack surfaces for developer tools, and that the file creation primitive is strong even without immediate code execution.
- *"Google closed it as Won't Fix — does that mean it is not a vulnerability?"* — The AI noted the closure was procedural (no merged patch provided, as required by OT2/OT3 tier rules) rather than a technical rejection, and that this does not invalidate the finding for assignment purposes.
- *"How do you know this is a zero-day?"* — The AI outlined the prior disclosure search steps and their negative results as supporting evidence.

**How I used it:**

I used the Q&A output to refine the presentation script and to make sure every team member could answer at least one of these questions with evidence-backed responses.

---

## Human Review and Corrections

- I independently read and followed the vulnerable code path in `src/cli.ts` before accepting the AI's analysis.
- I ran the PoC locally and did not treat AI-described behavior as confirmed until I had terminal output evidence.
- I toned down the severity framing in the report after recognising that the core demonstrated impact was file creation, not direct code execution.
- I searched for prior disclosure independently rather than relying on the AI's assertion that no prior reports existed.

---

## Limitations

- The PoC was demonstrated on a locally cloned version of `zx`. The issue was verified to exist in the latest release at time of submission but was not tested against a production deployment.
- Follow-on impact scenarios (e.g., shell startup file execution) were modelled rather than fully demonstrated in the PoC environment.
- Google's triage conclusion was procedural and does not constitute a formal severity rating, so CVSS was used as a fallback for rubric severity mapping.

---

## Reflection

AI assistance was most valuable during the initial code review phase, where it helped identify the subtle difference between `existsSync()` and `lstatSync()` semantics quickly. Without that, the code path might have appeared correct at a surface reading.

The main correction I made was to avoid overstating impact. An arbitrary file creation primitive is meaningful but not equivalent to immediate code execution, and the report and presentation language reflects that distinction.
