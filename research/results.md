# What AI-built apps actually get wrong

**N = 53 public repositories. Scanned 1 October 2026.** Method, sample rules and limits: [`method.md`](method.md). Anonymised data: [`dataset.csv`](dataset.csv).

Nine each from Lovable, Bolt, v0, Cursor and Claude Code, eight from Replit — every one declaring the tool itself, in a GitHub topic or its README. Median size 27,573 lines (range 1,739–344,373). Median health score 40 / 100.

---

## Found

The scanner reads a capped subset of files — median 16% of the files in these repositories — and never clones. Finding something proves it is there; finding nothing proves nothing about the files that were not read. So these are **floors, not estimates**.

| Failure | Repositories | Rate |
|---|---|---|
| Files beyond a maintainable size | 50 of 53 | at least 94% |
| At least one high-severity finding | 46 of 53 | at least 87% |
| At least one critical-severity finding | 24 of 53 | at least 45% |
| Request bodies used without validation | 12 of 53 | at least 23% |
| Possible credential, including needs-review | 12 of 53 | at least 23% |
| `.env`-family file committed | 10 of 53 | at least 19% |
| Committed credential, high confidence | 3 of 53 | at least 6% |
| Credential written to logs | 1 of 53 | at least 2% |
| Injection sink (concatenated SQL, `eval`) | 0 of 53 | none in what was read |

## Missing

These are read from the repository's full file tree and its manifests, which are listed in one request regardless of the file cap. They are **true rates**, not floors.

| Failure | Repositories | Rate |
|---|---|---|
| No security middleware detected | 38 of 53 | 72% |
| No `.env.example` | 36 of 53 | 68% |
| No linter configured | 18 of 53 | 34% |
| No CI pipeline | 15 of 53 | 28% |
| TypeScript not in strict mode | 8 of 53 | 15% |
| No automated tests | 5 of 53 | 9% |

---

## What this says

**The "AI-built apps have no tests" story is wrong.** 91% of these repositories have tests. Testing discipline is not where the tools fall down.

**The gap is the security layer.** Nearly three in four have no rate limiting, CORS configuration or equivalent middleware. Roughly one in four uses request bodies without validating their shape. Nearly half carry at least one critical finding.

**Secrets hygiene is better than the folklore suggests, and worse than it should be.** One in five has committed a `.env` file. Confirmed live credentials are rarer: 6%, and both of the ones we read closely were weak default passwords rather than vendor keys.

## What this does not say

The most serious failures in an AI-built app are **invisible from its repository**. Row Level Security policies, admin role checks and tenant isolation live in a Supabase or hosting project, not in code. This study cannot see them, and nothing here should be read as evidence that they are fine. That is exactly why the checklist marks those items as needing a human.

See [`method.md`](method.md) for the other four limits, including the one that matters most: a public repository is not a deployed product, and the direction of that bias is unknown.

---

## How accurate is the instrument

Published because a rate is only as good as the thing that measured it, and because we got this wrong first.

The first run of this study, on 30 September, reported **"at least 15% have committed a credential."** Before publishing, all eight repositories behind that number were opened by hand. **Six of the eight were false**: `assert!` fixtures inside a Rust secret-**redaction** module, a PII sanitiser's own sample corpus, a UUID used as a share link, a hyphenated constant, shell variables in a deploy script, and a CI service database on `localhost`. Precision on high-confidence credential findings was **25%**.

The scanner was fixed — six classes of fixture are no longer read as credentials — and this run is the result: **6%, three repositories, and the automated result now agrees with the manual check.**

The scanner is since held to a labelled corpus of 42 cases, 16 that must be reported and 26 that must not, scored on every change and failing below 0.97 on either precision or recall. Current score: **precision 100%, recall 100%.** That is "no known regression", not "correct" — the corpus is our own, and a bigger adversarial one is the next improvement.

Two earlier methodology errors are worth recording for anyone repeating this:

1. **A starved scanner reports a clean repository.** When the GitHub API budget runs low, files that fail to fetch are skipped silently and the result reads as "nothing found". The run now stops rather than continuing, and a thin read is labelled.
2. **Absence and presence need different denominators.** An early draft gated "no tests" on content coverage and would have published "56% have no tests" from a nine-row sample. Test, CI and linter detection read the full file tree, so they are knowable for every repository; credential detection reads file contents, which are capped.
