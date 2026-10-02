# Production readiness checklist for AI-built apps

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23094508.svg)](https://doi.org/10.5281/zenodo.23094508)

A launch checklist for apps built with **Lovable, Bolt, v0, Cursor, Replit or Claude Code**, deployed on **Supabase, Netlify or Vercel**.

Every item is triaged:

| | Meaning |
|---|---|
| 🔴 **Blocks launch** | Real users or real money are exposed until this is fixed. Do not go live. |
| 🟡 **Can wait** | It will cost you later — in incidents, in diligence, in the next developer's time — but it does not expose anyone today. |

And marked by who can check it:

| | Meaning |
|---|---|
| 🤖 **Automatic** | The free scan at [systemaudit.dev](https://systemaudit.dev) finds this from your repository, with the file and line. |
| 👤 **Human** | It needs judgement about your data, your users and your business. A scanner cannot decide whether a given table should be readable by a given person. |

The split matters: most of what actually takes an AI-built app down is 👤, and most published checklists are written as though it were all 🤖.

---

## 1. Database access and tenancy (Supabase, Postgres)

The single most common way an AI-built app leaks: the generated code works, and the database is wide open behind it.

- 🔴 👤 **Row Level Security is ON for every table holding user data.** Generated schemas frequently ship with RLS off, and the app still works perfectly — until someone calls the API directly with the public key.
- 🔴 👤 **Every RLS policy names a user.** A policy of `true`, or `auth.role() = 'authenticated'`, means every signed-in user reads every row. This is the failure behind most "my competitor could see our customers" stories.
- 🔴 👤 **Tenant isolation is proven, not assumed.** Sign in as tenant A, request tenant B's record by ID, confirm the failure. Do this against the deployed app, not locally.
- 🔴 🤖 **The anon/publishable key is the only key in the frontend.** A service-role key in client code, or in a `NEXT_PUBLIC_*` variable, bypasses RLS entirely.
- 🟡 👤 **Deletes are policy-checked too.** Read and write policies are usually written; delete and update are often forgotten.
- 🟡 🤖 **Migrations are in the repo.** Schema that exists only in a dashboard cannot be reviewed, rolled back, or recreated.

## 2. Authentication vs authorisation

Signed in is not the same as allowed. AI tools reliably build the first and skip the second.

- 🔴 👤 **Every privileged route checks a role on the server.** Hiding an admin link in the UI is not a check. Ask: what happens if someone types the URL?
- 🔴 👤 **Role comes from the verified session, never from the request.** If a header, query parameter or request body says who you are, anyone can say they are an admin.
- 🔴 👤 **Admin endpoints fail closed.** A missing user ID must be a refusal, not a skipped check. "If a user was supplied, verify it" grants access when none is supplied.
- 🟡 👤 **Invitation and reset tokens come from a cryptographic source.** `Math.random()` is guessable; use `crypto.randomBytes` or your platform's equivalent.
- 🟡 🤖 **Sessions are httpOnly cookies**, not values readable by any script on the page.

## 3. Secrets and environment variables

- 🔴 🤖 **No key, token or password is committed.** Including in `.env`, seed files, tests and comments. Rotate anything that ever reached a commit, even a deleted one: git keeps history.
- 🔴 👤 **Nothing secret sits behind a `NEXT_PUBLIC_`/`VITE_` prefix.** Those are compiled into the bundle every visitor downloads.
- 🔴 👤 **Netlify and Vercel variables are scoped.** A variable set for all contexts is readable by preview deploys of every pull request, including forks.
- 🟡 🤖 **`.env.example` exists** and lists every variable, so the next person knows what the app needs without being handed the values.
- 🟡 👤 **Keys are rotatable without a redeploy.** If rotating means editing code, it will not happen when it matters.

## 4. Payments and webhooks

- 🔴 👤 **Webhook signatures are verified.** An unverified webhook endpoint is an unauthenticated API that grants whatever the webhook grants — including paid access.
- 🔴 👤 **Entitlement comes from the payment provider, not the browser.** A redirect back from checkout proves nothing: anyone can visit a success URL.
- 🔴 👤 **What was bought is read from the provider's product data**, not inferred from an amount. Amount-based logic misfires the moment a coupon, a currency or a second product exists.
- 🟡 👤 **Refunds, disputes and failed renewals remove access.** Most AI-built billing grants access and never revokes it.
- 🟡 👤 **The webhook is idempotent.** Providers retry; a duplicate must not double-grant or double-charge.

## 5. AI features

If the app itself calls a model, it inherits a second class of problem.

- 🔴 👤 **Untrusted text cannot issue instructions.** Content from users, uploads, web pages or emails is data. If it reaches a prompt that can call tools, assume it will try to.
- 🔴 👤 **The model's tools are scoped to the caller.** A model that can query the database on behalf of "the app" can read every user's rows.
- 🔴 👤 **Model output is not executed.** No `eval`, no shelling out, no SQL built by concatenation from generated text.
- 🟡 🤖 **The model endpoint is rate limited.** Without it, one loop empties your account, and it will be someone else's loop.
- 🟡 👤 **Per-user and total spend are capped**, with an alert you will actually see.

## 6. Observability

- 🔴 👤 **Errors reach somewhere you look.** `console.error` in a serverless function is not monitoring.
- 🔴 🤖 **Nothing secret is logged.** Keys, tokens and passwords in logs are keys, tokens and passwords in every log aggregator they touch.
- 🟡 👤 **You can answer "is it up?" without opening the app.**
- 🟡 👤 **You can trace one user's failed request.** Without a request ID, "it broke for one customer" is unanswerable.

## 7. Deployment and the code itself

- 🔴 🤖 **No injection sinks.** SQL built by string concatenation, `eval`, `dangerouslySetInnerHTML` on user content.
- 🔴 👤 **Preview deploys do not point at production data.** A preview URL is public and unauthenticated by default.
- 🟡 🤖 **There is a test suite, and CI runs it.** Not for coverage: so that a change that breaks payments is caught by something other than a customer.
- 🟡 🤖 **Types are strict.** `strict: false` in an AI-built TypeScript project means the types are decoration.
- 🟡 🤖 **No oversized files.** A 2,000-line file is where the next bug hides, and where the next developer stops reading.
- 🟡 👤 **One person is not the only one who understands a critical area.** Check the commit history per directory, not the team roster.

---

## Research: what AI-built apps actually get wrong

We scanned **53 public repositories** that declare they were built with Lovable, Bolt, v0, Cursor, Replit or Claude Code, on 1 October 2026. Rates carry 95% confidence intervals, because at this sample size the interval is the honest figure.

| Failure | Rate | 95% CI |
|---|---|---|
| No automated security checks *(no Dependabot, CodeQL, Snyk or equivalent)* | **at least 72%** | 58–82% |
| No `.env.example` | 68% | 55–79% |
| At least one critical finding | at least 45% | 33–59% |
| No linter | 34% | 23–47% |
| No CI pipeline | 28% | 18–42% |
| Request bodies used without validation *(repos with server code)* | at least 23% | 12–38% |
| `.env` file committed | at least 19% | 11–31% |
| Committed credential (high confidence) | at least 6% | 2–15% |
| **No sign of testing at all** *(no test file, test config or test runner)* | **at least 9%** | 4–20% |

**The "vibe-coded apps have no tests" story looks wrong — 91% show at least some sign of testing.** Whether those tests are real and run was not checked. The clearer gaps are automated security checks, input validation and secrets hygiene.

> **Correction, 2 October 2026.** The first row was published in v1.0.1 as "no security middleware (rate limiting, CORS)" at 73%. The scanner column behind it detects security *tooling* (dependency and code scanning), not request middleware. The count was right; the label was wrong. Rate limiting and CORS were not measured. The test row was also narrowed: it counts any sign of testing, not only a test file. Details: [`research/results.md`](research/results.md#correction-2-october-2026).

The write-up: [What 53 AI-built apps get wrong](https://nicchin.com/blog/vibe-coded-app-security-study).

Full numbers, what they do not say, and the measured accuracy of the scanner that produced them: [`research/results.md`](research/results.md). Method and limits: [`research/method.md`](research/method.md). Anonymised data: [`research/dataset.csv`](research/dataset.csv).

The most serious failures in an AI-built app — Row Level Security, admin role checks, tenant isolation — are invisible from a repository. That is why the items above are marked 👤.

---

---

## Citing this

Chin, N. (2026). *Production readiness checklist for AI-built apps, with measured research on 53 public repositories*. Zenodo. https://doi.org/10.5281/zenodo.23094508

That DOI always resolves to the latest version. [`CITATION.cff`](CITATION.cff) has the machine-readable form.

## About

Maintained by [Nic Chin](https://nicchin.com). The automatic checks come from the free scan at [systemaudit.dev](https://systemaudit.dev), which reports every finding with its file and line.

For a human review of an AI-built app before launch, raise or sale: [nicchin.com/vibe-coded-app-audit](https://nicchin.com/vibe-coded-app-audit).

Corrections and additions are welcome — open an issue.

*MIT licensed. Nothing here names or links a scanned repository: see the method for why.*
