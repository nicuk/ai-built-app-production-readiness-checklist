# What AI-built apps actually get wrong

> **Corrected 2 October 2026 (v1.0.2).** Two rows were described as something other than what the scanner measures: "security middleware" was security tooling, and "a test file" was any sign of testing. See [the correction](#correction-2-october-2026). No count changed.

**N = 53 public repositories. Scanned 1 October 2026.** Method, sample rules and limits: [`method.md`](method.md). Anonymised data: [`dataset.csv`](dataset.csv).

Nine each from Lovable, Bolt, v0, Cursor and Claude Code, eight from Replit — every one declaring the tool itself, in a GitHub topic or its README. Median size 27,573 lines (range 1,739–344,373). Median health score 40 / 100.

Larger measurements of this question exist, and on reach they are better than this one: Deng, Fan
and Meng audited 200 publicly deployed vibe-coded applications ([arXiv:2606.23130](https://arxiv.org/abs/2606.23130)),
and the SusVibes benchmark measured agent-generated code on 186 real-world tasks
([arXiv:2512.03262](https://arxiv.org/abs/2512.03262)). What this study adds is repository-level
rates with intervals, a per-repository dataset, and the measured accuracy of its own scanner. See
[`method.md`](method.md).

Rates are given with 95% Wilson confidence intervals. At this sample size they are wide, and the interval matters more than the point estimate: "72%" is really "somewhere between 58% and 82%".

---

## Found

The scanner reads a capped subset of files — median 16% of the files in these repositories — and never clones. Finding something proves it is there; finding nothing proves nothing about the files that were not read. So these are **floors, not estimates**.

| Failure | Repositories | Rate | 95% CI |
|---|---|---|---|
| Files beyond a maintainable size | 50 of 53 | at least 94% | 85–98% |
| At least one high-severity finding | 46 of 53 | at least 87% | 75–93% |
| At least one critical-severity finding | 24 of 53 | at least 45% | 33–59% |
| Request bodies used without validation *(repos with server code)* | 9 of 40 | at least 23% | 12–38% |
| Possible credential, including needs-review | 12 of 53 | at least 23% | 13–36% |
| `.env`-family file committed | 10 of 53 | at least 19% | 11–31% |
| Committed credential, high confidence | 3 of 53 | at least 6% | 2–15% |
| Credential written to logs | 1 of 53 | at least 2% | 0–10% |
| Injection sink (concatenated SQL, `eval`) | 0 of 53 | none in what was read | 0–7% |

## Missing

Read from the repository's full file tree and its manifests, which are listed in one request regardless of the file cap. These are **true rates**, not floors, except the two rows marked "at least": their rules accept loose path matches that can only make a repository look better than it is.

| Failure | Repositories | Rate | 95% CI |
|---|---|---|---|
| **No automated security checks** *(no dependency or code scanning: Dependabot, Renovate, CodeQL, Semgrep, Snyk, Trivy, gitleaks or equivalent)* | **38 of 53** | **at least 72%** | **58–82%** |
| The same, among repos with server code | 29 of 40 | at least 73% | 57–84% |
| No `.env.example` | 36 of 53 | 68% | 55–79% |
| No linter configured | 18 of 53 | 34% | 23–47% |
| No CI pipeline | 15 of 53 | 28% | 18–42% |
| TypeScript not in strict mode | 8 of 53 | 15% | 8–27% |
| No sign of testing anywhere *(no test file, test configuration or test-runner dependency)* | 5 of 53 | at least 9% | 4–20% |

### Correction, 2 October 2026

Version 1.0.1 published this row as **"no security middleware (rate limiting, CORS, helmet-equivalent)"**, headlined at 73% among repositories with server code. That label was wrong. The column, `noSecurityLayer`, is computed from the scanner's *security tooling* signal: dependency and code-scanning configuration (Dependabot, Renovate, CodeQL, Semgrep, Snyk, Trivy, gitleaks, a `SECURITY.md`) or a security-audit package in the manifest. Nothing in it looks for rate limiting, CORS or helmet.

The counts are unchanged and recomputed from `dataset.csv`: 38 of 53 overall, 29 of 40 among repositories with server code. What changes is what they mean. **About three in four of these apps have no automated check that would flag a vulnerable dependency or a known-bad code pattern.** Whether they have rate limiting or CORS configured was not measured, and nothing in this study supports a claim either way.

The rate is a floor. The rule also accepts "weak" evidence, any file path containing a word such as `snyk`, `dependabot` or `safety`, which can only over-detect tooling, so the true share without it may be higher.

The same check found a second, smaller problem. The test row was described as "no test file anywhere". The rule behind it counts a repository as tested if it has a test file, a test configuration (`jest.config`, `vitest.config`, `playwright.config` and similar), a test-runner dependency in its manifest, or any file path containing `test` or `spec`. So "91% have at least one test file" was really **"91% show some sign of testing"**, an upper bound: the share with real test files may be lower. The repository list was not kept in a form that allows a stricter re-scan, so the row is relabelled rather than re-measured.

Because tooling can be configured in any repository, frontend or not, the all-repositories rate is now the headline. The server-code split from v1.0.1 is kept below for continuity.

### The server-code split (from v1.0.1)

This split was made when the row was believed to measure middleware, which can only appear in a repository that serves requests. A Lovable or Bolt app is often a frontend talking straight to Supabase, where those controls live in the Supabase project and are invisible here — so counting those repositories as "missing" a control would be measuring the architecture, not the practice.

Server-side code was therefore re-derived for every repository from its file tree and manifest: 40 of 53 have it, 13 do not. **The split barely moves the number**: 73% among repositories with server code, against 72% overall and 69% among frontend-only ones. v1.0.1 headlined the 73% for that reason; with the row correctly understood as tooling, the headline is now the all-repositories rate.

---

## What this says

**The "AI-built apps have no tests" story looks wrong.** 91% show at least some sign of testing: a test file, a test configuration or a test runner. That is an upper bound, and it says nothing about whether the tests are meaningful or run.

**Automated security checks are mostly missing.** Roughly three in four repositories have no dependency or code scanning configured, so a vulnerable package or a known-bad pattern can reach production without anything flagging it. Among repositories that serve requests, roughly one in four uses request bodies without validating their shape. Nearly half carry at least one critical finding.

**Secrets hygiene is better than the folklore, and worse than it should be.** One in five has committed a `.env` file. Confirmed credentials are rarer — 6%, three repositories, all opened by hand — and none was a live vendor key: two are default passwords written into code, and the third is an install script whose value may come from the environment.

## What this does not say

The most serious failures in an AI-built app are **invisible from its repository**. Row Level Security policies, admin role checks and tenant isolation live in a Supabase or hosting project, not in code. This study cannot see them, and nothing here should be read as evidence that they are fine. That is exactly why the checklist marks those items as needing a human.

See [`method.md`](method.md) for the other limits, including the one that matters most: a public repository is not a deployed product, and the direction of that bias is unknown.

---

## How accurate is the instrument

Published because a rate is only as good as the thing that measured it, and because we got this wrong first.

The first run of this study, on 30 September, reported **"at least 15% have committed a credential."** Before publishing, all eight repositories behind that number were opened by hand. **Six of the eight were false**: `assert!` fixtures inside a Rust secret-**redaction** module, a PII sanitiser's own sample corpus, a UUID used as a share link, a hyphenated constant, shell variables in a deploy script, and a CI service database on `localhost`. Precision on high-confidence credential findings was **25%**.

The scanner was fixed — six classes of fixture are no longer read as credentials — and this run is the result: **6%, three repositories**, every one of them also opened by hand, and the automated result now agrees with the manual reading.

The scanner is since held to a labelled corpus of 42 cases, 16 that must be reported and 26 that must not, scored on every change and failing below 0.97 on either precision or recall. Current score: **precision 100%, recall 100%.** That is "no known regression", not "correct" — the corpus is our own, and a larger adversarial one is the next improvement.

Four errors are recorded for anyone repeating this:

1. **A starved scanner reports a clean repository.** When the GitHub API budget runs low, files that fail to fetch are skipped silently and the result reads as "nothing found". The run now stops rather than continuing, and a thin read is labelled.
2. **Absence and presence need different denominators.** An early draft gated "no tests" on content coverage and would have published "56% have no tests" from a nine-row sample. Test, CI and linter detection read the full file tree, so they are knowable for every repository; credential detection reads file contents, which are capped.
3. **A control that cannot appear is not a control that is missing.** The security-middleware rate was first published across all repositories, including frontends with no server at all. It is now split, and the split is shown above rather than only the favourable half. (Item 4 supersedes this: the row was never a middleware rate.)
4. **A column name is not a definition.** The `noSecurityLayer` column was described from its name, as missing middleware, without reading the rule that sets it; the rule detects security tooling. The test row was described as "a test file" when the rule accepts any sign of testing. Both were corrected on 2 October 2026 and no count changed. Of the other categories, the unvalidated-input, injection-sink and committed-`.env` rules were read against their published definitions and match; the rest have not yet been traced.
