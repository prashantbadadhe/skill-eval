# Prompt: Build a Skill Evaluation & Governance System for a Central Skills Repo

Use this prompt with Claude (ideally Claude Code, since it involves creating files/scripts in a repo).

---

## PROMPT

I'm running a shared "domain skills" repository that multiple teams contribute to (Anthropic Agent Skills format — `SKILL.md` + optional `scripts/`, `references/`, `assets/`). Each team builds skills for their own domain and adds them to this central repo so other teams can reuse them.

I need you to set up a repeatable evaluation and governance system so that any skill a team submits gets validated *before* merging into the central repo. Build the following:

### 1. Skill submission template
Create a `template/SKILL_TEMPLATE.md` and a `template/eval_metadata.json` that every new skill submission must include, covering:
- YAML frontmatter (`name`, `description` — description must clearly state both *what* the skill does and *when* to trigger it, written to avoid under-triggering)
- Body instructions following progressive disclosure (metadata → SKILL.md body under ~500 lines → bundled resources loaded on demand)
- A list of 2–5 realistic test prompts a real user of that domain would type (not toy examples — include file paths, team-specific terminology, casual phrasing)
- Expected output description for each test prompt

### 2. Evaluation schema
Create `evals/schema.md` defining the JSON structure for:
- `evals.json` — test prompts + assertions (objectively verifiable pass/fail conditions) per skill
- `trigger_evals.json` — ~20 queries per skill: 8–10 that should trigger it (varied phrasing/casual/implicit intent) and 8–10 near-miss negatives (queries that share vocabulary/domain but should NOT trigger it — make these genuinely tricky, not obviously irrelevant)
- `benchmark.json` — aggregated results: pass rate, trigger accuracy, token cost, latency, with-skill vs. baseline (no-skill) comparison

### 3. Automated eval scripts
Write scripts (Python, runnable in CI) that:
- `run_output_eval.py` — runs each test prompt with and without the skill loaded, grades outputs against assertions, writes `benchmark.json`
- `run_trigger_eval.py` — tests the skill's description against the trigger eval set, reports trigger accuracy (true positive / true negative rate), flags overly broad or overly narrow descriptions
- Both scripts should exit non-zero (fail CI) if trigger accuracy or pass rate falls below a configurable threshold (default: 90% trigger accuracy, 100% assertion pass rate for objectively-checkable assertions)

### 4. Governance checklist / PR template
Create `.github/PULL_REQUEST_TEMPLATE/skill_submission.md` with a checklist covering:
- [ ] Security review: no exfiltration, no embedded credentials, no unreviewed external network calls, doesn't instruct Claude to bypass safety behavior
- [ ] Principle of no surprise: skill's actual behavior matches what its description promises
- [ ] Namespace check: no name/description collision or overlapping trigger conditions with existing skills in the repo (list the check command/script to run)
- [ ] Structure compliance: matches the template, correct use of `scripts/` vs `references/` vs `assets/`
- [ ] Automated eval results attached (link to benchmark.json / CI run)
- [ ] Named owning team + maintenance/deprecation contact
- [ ] Reviewed and approved by [central skills council / designated reviewer role]

### 5. Namespace collision checker
Write a script `check_collisions.py` that scans all existing `SKILL.md` descriptions in the repo against a new submission's description and flags semantic overlap (using simple keyword/embedding similarity — keep it lightweight, no heavy dependencies) above a threshold, so reviewers can catch duplicate or conflicting skills before merge.

### 6. Contribution guide
Write a `CONTRIBUTING.md` that ties all of the above into a clear step-by-step process for a team submitting a new skill:
1. Draft skill using the template
2. Write test prompts + assertions
3. Run local eval scripts, fix failures
4. Open PR with the skill submission template filled out
5. CI runs automated evals automatically and posts results as a PR comment
6. Central reviewer runs through the governance checklist
7. Merge criteria: CI passing + checklist signed off

Ask me for any repo-specific details you need (my repo's file layout, my org's specific security constraints, CI provider — GitHub Actions vs. other, threshold numbers) before finalizing thresholds or CI config. Otherwise use sensible defaults and flag them clearly as defaults I should adjust.

---

## Notes on using this prompt
- If you're doing this in **Claude Code** pointed at your actual repo, it can inspect your existing skills and CI setup first and tailor everything instead of guessing.
- If you're doing this in a plain chat with **Claude.ai**, expect it to generate the templates/scripts as text/files rather than integrate with a live CI pipeline — you'll wire that part up yourself.
- Adjust the trigger-accuracy and pass-rate thresholds to match your team's risk tolerance; 90%/100% above are reasonable starting points, not fixed requirements.
