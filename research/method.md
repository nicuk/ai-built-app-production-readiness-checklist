# Method

What was measured, how the sample was chosen, and what the numbers cannot tell you.

## The question

Published checklists for AI-built apps are mostly written from opinion. Measurements do exist, and
they are better than this one on reach:

- Deng, Fan and Meng, *Understanding the (In)Security of Vibe-Coded Applications*
  ([arXiv:2606.23130](https://arxiv.org/abs/2606.23130), revised 14 September 2026), collected 9,041
  open-source applications built with Claude Code and Lovable and audited 200 **publicly deployed**
  ones, finding 1,186 vulnerabilities, 91.0% of audited apps affected, 65.77% rated Critical or
  High, concentrated in broken access control, injection and authentication.
- Zhao et al., *Is Vibe Coding Safe?* (SusVibes,
  [arXiv:2512.03262](https://arxiv.org/abs/2512.03262), revised 21 September 2026), benchmarked
  agent-generated code on 186 real-world tasks: 57% of SWE-Agent with Claude 4 Sonnet solutions were
  functionally correct, and only 11.8% were secure.

What this study adds is narrower and more boring: **repository-level rates per failure category,
with 95% intervals, an anonymised per-repository dataset, and the measured precision and recall of
the instrument that produced them** — including the first run's 25% precision and how it was caught.
A reader can check our arithmetic, and can check how wrong our scanner is.

It is also worth saying what the stronger work implies about this one. Deng et al.'s largest
category, broken access control, is precisely what a repository scan cannot see: the policy lives in
a Supabase or hosting project. Their numbers come from auditing deployed applications; ours come
from reading code. Where the two disagree, believe theirs — and read the gap as the argument for the
checklist items marked as needing a human.

## Sample

A repository was eligible if **it declares the tool itself** — a GitHub topic, or the tool named in
its README. Inferring the tool from code style would make the population unrepeatable, and the point
of publishing the method is that someone else can run it.

Excluded, before any scanning:

- forks and archived repositories;
- anything under 300 KB, and after scanning anything under 1,000 lines of code — a snippet is not an app;
- starters, templates, boilerplate, tutorials, demos and course projects, by name, description and topic;
- deliberately vulnerable projects (Juice Shop, DVWA, CTF repos and the like). They are insecure on
  purpose, so counting them would inflate every security rate and say nothing about AI tools;
- repositories last pushed before September 2025.

The sample is **stratified by tool**, taken round-robin and newest first, so it is not dominated by
whichever tool is most popular on GitHub and an early stop cannot empty one tool’s cell. The run of
1 October 2026 stopped at 53 of 60 after four waits on GitHub’s secondary rate limit, leaving nine
each for Lovable, Bolt, v0, Cursor and Claude Code, and eight for Replit.

## Measurement

Each repository was read through the GitHub API by the deterministic half of the
[SystemAudit](https://systemaudit.dev) scanner. No model was involved in producing any number here:
every value is a rule over the files that were read.

**Coverage is capped and that cap is part of the result.** The scanner reads a subset of files — the
most relevant ones, up to a per-size limit — and never clones a repository. `filesRead` and `files`
are both in the dataset for every row. A finding of zero means "nothing found in what was read", not
"nothing there".

The run stops rather than continuing when the GitHub token's hourly budget falls below a threshold.
A scan that cannot fetch its files skips them silently and reports the repository as clean, which
would be a fabricated data point in a study about failure rates.

## What each category means

| Column | Measured by |
|---|---|
| `secretsHighConfidence` | A committed value matching a known credential format (vendor-issued key patterns, private keys, connection strings with a password) |
| `secretsAny` | The above plus lower-confidence matches needing human review |
| `envFileCommitted` | A `.env`-family file present in the repository |
| `noTests` / `noCi` / `noLinter` / `noTypeSafety` | No test file anywhere in the tree, no CI configuration, no linter configuration, TypeScript not in strict mode. `noTests` measures the presence of a test file, not whether the tests are meaningful or run |
| `hasServerCode` | The repository serves requests: an `api/`, `server/`, `backend/` or serverless `functions/` directory, a server-language entry point, or a server framework in its manifest. Re-derived per repository from the file tree, because a control that cannot appear in a frontend is not a control that is missing |
| `noSecurityLayer` | No security middleware detected (rate limiting, CORS, helmet-equivalent) |
| `unvalidatedInput` | Request bodies used without a validation step |
| `injectionSink` | SQL built by concatenation, `eval`, or equivalent |
| `secretInLogs` | A credential written to a log line |
| `oversizedFiles` | Files beyond a maintainability threshold |
| `healthScore` | The scanner's 0–100 composite; lower is worse |
| `criticalRisks` | Findings the scanner rates critical: a committed credential or env file, an injection sink reachable from request input, or an authorisation check satisfied by data the caller controls. Severity is the scanner’s own, not a CVSS score, and “at least 45% carry a critical finding” should be read with that definition in hand |

Rates in the results carry 95% Wilson confidence intervals. At N = 53 they are wide — a 72% point
estimate spans 58–82% — and the interval is the honest figure.

## What this cannot tell you

1. **Public repositories are not representative of deployed apps.** A team that commits its app to a
   public repository may be less careful than one that does not, or simply earlier in its life. The
   direction of that bias is unknown.
2. **The most serious failures are invisible from a repository.** Row Level Security policies, admin
   role checks against a live database, and tenant isolation live in a Supabase project, not in the
   code. This study cannot see them. That is precisely why the checklist marks those items 👤.
3. **A declared tool does not mean the tool wrote everything.** Repositories that name Lovable or
   Claude Code may be heavily hand-edited afterwards.
4. **Absence of a finding is not absence of a problem**, because coverage is capped. See above.
5. **Rates are not severity.** A missing linter and a committed database password both count once.

## Responsible disclosure

No repository is named, linked or fingerprinted anywhere in this repository, and the published
dataset carries no identifier. Names were kept only in a local file so that a finding could be
re-checked.

Every high-confidence credential finding in this study was opened by hand before any rate was
published; see the accuracy section of the results. Three repositories had one. **None is a live
vendor key**: two are default passwords written into code, and the third is an install script whose
value may come from the environment.

Nothing about any of them is published here — not the name, not the file, not the line — and the
rates are counts only. The owners of the two default-password findings are being contacted
privately. If you believe your repository was in this sample and want to know what was found, email
the address on [nicchin.com](https://nicchin.com).

## Repeating this

The eligibility rules, the exclusions and the category definitions above are the whole method. The
structural checks (tests, CI, linter, types) are reproducible with the MIT-licensed CLI at
[github.com/nicuk/systemaudit](https://github.com/nicuk/systemaudit). The security categories
(secrets, injection sinks, ranked risks) come from the hosted scanner at
[systemaudit.dev](https://systemaudit.dev), which is free to run against any public repository.
