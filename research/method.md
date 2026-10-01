# Method

What was measured, how the sample was chosen, and what the numbers cannot tell you.

## The question

Published checklists for AI-built apps are written from opinion. Nobody publishes measurements of
what these apps actually get wrong. This run measures a sample of public repositories that say they
were built with an AI tool, and reports rates per failure category.

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
| `noTests` / `noCi` / `noLinter` / `noTypeSafety` | No test files, no CI configuration, no linter configuration, TypeScript not in strict mode |
| `noSecurityLayer` | No security middleware detected (rate limiting, CORS, helmet-equivalent) |
| `unvalidatedInput` | Request bodies used without a validation step |
| `injectionSink` | SQL built by concatenation, `eval`, or equivalent |
| `secretInLogs` | A credential written to a log line |
| `oversizedFiles` | Files beyond a maintainability threshold |
| `healthScore` | The scanner's 0–100 composite; lower is worse |

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

Where the scan indicated a **live, exploitable credential**, the repository owner was contacted
privately and nothing about it was published. Every high-confidence credential finding in this study
was also opened by hand before any rate was published; see the accuracy section of the results. If you believe your repository was in this sample and
you want to know what was found, email the address on [nicchin.com](https://nicchin.com).

## Repeating this

The eligibility rules, the exclusions and the category definitions above are the whole method. The
structural checks (tests, CI, linter, types) are reproducible with the MIT-licensed CLI at
[github.com/nicuk/systemaudit](https://github.com/nicuk/systemaudit). The security categories
(secrets, injection sinks, ranked risks) come from the hosted scanner at
[systemaudit.dev](https://systemaudit.dev), which is free to run against any public repository.
