# docs — CLAUDE.md

**The public developer documentation, live at `docs.platnova.com`** — the API reference that external merchants integrate against. A [Mintlify](https://mintlify.com) site: 70 `.mdx` files, navigation and theme in `docs.json`.

**Status: live and public.** Verified 2026-09-08. Low commit traffic (last substantive change 2026-06-17) but high consequence — this is a product surface, not an internal wiki.

## This is publishing, not writing *(verified 2026-09-08)*

`docs.platnova.com` serves this repo. **A merge here reaches every integrator, immediately, with no staging gate you control** — and there is no CI in this repo at all: no build check, no link check, no spell check, nothing. The only review is the one on the PR.

So treat a change here the way you would treat a release: someone with authority over what Platnova tells its integrators should see it before it merges. Copy about fees, limits, compliance requirements or timelines is a regulated claim, not documentation.

`main` and `dev` are currently identical (0 commits apart in both directions), so there is no work-branch subtlety here — but that also means nothing is staged behind `dev`.

## It documents `api`, and nothing keeps them in sync

Every endpoint here belongs to **`api`** (the merchant backend), not `engine`:

| documented | environment base URL |
|---|---|
| sandbox | `https://sandbox.api.platnova.co` |
| live | `https://api.platnova.com` |

Pages under `api-reference/` declare their endpoint in frontmatter (`openapi: 'GET /v1/cards'`) — but **that string is not generated from `api`'s source and nothing validates it.** There is no OpenAPI spec exported from the Go code, no contract test, no CI check. An endpoint renamed, a field removed or an auth rule changed in `api` leaves this site confidently describing something that no longer exists, and the first person to notice is an integrator whose call fails.

**So: when you change a route, request body or error shape in `api`, update the page here in the same change.** That discipline is the only mechanism there is. It is also the cheapest possible fix for the failure this workspace already had — a merchant who got `"invalid input"` when the real cause was a missing BVN.

Auth is `X-API-Key` (`api/internal/common/middleware/credential.go:98`). `Authorization` is for JWT and Token credentials, which are not what integrators use.

## Two defects found on the live site, 2026-09-08

Recorded because both are the same class — **starter-template and copy-paste residue reaching real integrators** — and because finding them took reading the published HTML, not the repo:

1. **Mintlify's starter "plant store" example was live.** `api-reference/endpoint/{create,get,delete}.mdx` documented `GET /plants`, `POST /plants`, `DELETE /plants/{id}`. They were absent from `docs.json` navigation, so they looked harmless — but Mintlify publishes any page that exists, and `docs.platnova.com/api-reference/endpoint/get` returned HTTP 200 with the title *"Get Plants - Platnova Docs"* and matching OpenGraph metadata, so it previewed that way when shared and was indexable.
2. **`authentication.mdx` contradicted itself.** The prose said to send the key in the `Authorization` header while its own example used `X-API-Key`. The example was right. An integrator following the sentence gets an auth failure that the page cannot help them debug.

**Being absent from `docs.json` does not mean a page is unpublished.** Check the live URL, not just the navigation.

## What matters here

**This repo describes systems it does not contain.** Every statement about `engine`, `api`, the apps or the infrastructure is a claim about another repo that may already be stale. When you change docs, cite the file or endpoint you verified against — the same evidence rule that applies to code applies here, and it matters more, because a wrong document is believed.

**Documentation that is wrong is worse than absent.** A missing page sends someone to the code. A confidently wrong page sends them into production with a false model.

---

## The customer comes first — and that is testable, not a slogan

Every change is judged by what it does to the person on the other end. These are pass/fail, and each one is here because it already went wrong:

- **An error a customer or integrator sees must say what to do next.** A merchant integrating our API got `"invalid input"` when the real cause was a missing BVN. They could not have guessed, so they filed a ticket instead.
- **No silent failure.** The customer must never discover a problem before we do. Exchange rates sat frozen for three weeks; a rejected payout sat at `Pending` with the money already debited. Both were invisible until someone complained.
- **The customer is never left out of pocket.** ₦2,000 stranded on a failed transfer; gift cards delivered with the literal code `"<nil>"`; 27 duplicate credits. Money moving the wrong way is the most expensive class of bug here.
- **Nothing ships that support has to explain.** If the only way a user succeeds is by being told something the product never told them, the product is unfinished.
- **If a flow would frustrate you as a paying customer, that is a defect.** Say so unprompted, even when nobody asked.

## Operating standard

Approach the work through all of these lenses, not just the one the ticket names. Each asks a different question, and the ones that get skipped are where the expensive mistakes live:

| Lens | The question it asks |
|---|---|
| **Customer** — genuine customer, product specialist, product engineer, UI/UX and graphic designer | Would I accept this as a paying customer? Is it obvious without explanation? |
| **Business** — CEO, CFO, COO, financial and business analyst | Should this be built at all? What does it cost, earn, or risk? |
| **Craft** — CTO, engineering manager, architect, fullstack/frontend/backend/mobile engineer | Best architecture available — readable, maintainable, matching the patterns already here. |
| **Reliability** — infrastructure, cloud, devops, CI/CD | Blast radius, cost, behaviour at 10×, how it deploys and how it rolls back. |
| **Verification** — QA, tester | How is this proven end to end, not merely unit-green? |
| **Security & compliance** — security and compliance specialist | Auth, PII, money paths, abuse cases, regulatory exposure. |
| **Data & AI** — data analyst/engineer, AI/ML/MLOps engineer | What do we measure? Is the data correct and queryable? |

Question whether a request is the right thing to build, not only how to build it. Raise a concern once and then defer if overruled — a concern never raised is the failure.

## No hallucination — cite it or do not claim it

This platform moves money. A confident wrong answer is worse than no answer.

- **Cite or do not claim.** Every statement about the codebase carries a `file:line`, a SHA, command output, or a query result. Never name a file, function, config key, status value, or API field without confirming it exists.
- **Never report success you did not observe.** "Tests pass" requires the output. If tests fail, say so and show it. If a step was skipped, say it was skipped.
- **Never claim something shipped without checking.** `merged=true` is not proof — confirm the SHA is an ancestor of the target branch. Assuming otherwise cost two releases.
- **Separate verified from inferred, explicitly.** "I verified X" and "I believe X but did not check" are different sentences. Write whichever is true.
- **State an unverifiable gap; never fill it.** "I could not verify X" is a complete and acceptable answer.
- **Verify every automated-review finding before acting on it.** Roughly a quarter do not hold. Acting on a wrong finding is a self-inflicted bug.

## Pull requests

**Squash-merge every ordinary PR, and write the title as a Conventional Commit.** Squash uses the PR title as the commit message, so **the title is the history**: `type(scope): summary` — e.g. `fix(payments): auto-reverse failed payouts`. Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `build`, `ci`, `style`, `revert`.

**Merge sync PRs with a merge commit — never squash them.** A sync PR is one whose head is a long-lived branch (`master`, `main`, `dev`, `staging`): a back-merge or a release. Squashing one produces a commit with **no merge parent**, so git still reports the original commits as missing from the target. The branches then stay permanently "diverged", every later back-merge re-applies the same changes, and the conflicts recur forever.

**Every PR carries its ClickUp task** in the `## Ticket` section — the task URL or the raw `86…` id. Sync PRs are exempt; they carry no ticket of their own.

**A breaking change is described, not just ticked.** If the box is checked, say what breaks, who is affected (API consumers, mobile clients, other services), and the migration or rollback path.

**`## Proof it works` carries evidence, not assertion.** Test output, a query result, logs, or a screenshot. "Tests pass" without the output is not proof.

## Handling the automated PR review

Every push triggers a `claude-review` workflow that posts an inline review.

**Read it, and attend to every finding.** Reply in the thread the finding was raised in — **including the ones you reject**. A rejection answered only in a summary comment leaves the thread looking unaddressed.

**Verify each finding against the code before acting on it.** A meaningful share do not hold. But findings often name the right *area* with the wrong mechanism, so investigate the area even when the stated cause is wrong.

**When every finding is handled, post a closing summary** saying what was fixed, what was rejected and why, so a human reviewer sees a resolved state rather than an open one. To confirm none were missed, diff the bot's comment ids against the `in_reply_to_id` values of your replies.

**If the `review` check fails with no comments posted, the workflow itself errored** — read the run log rather than assuming the PR is at fault. It has failed org-wide twice for reasons that had nothing to do with any PR: once when the Anthropic account ran out of credits, and once because a repo pinned no model and defaulted to one its key could not use. Both reported as "review found problems" when in fact no review ran.

## Keep this file true

**Record what you learn, in the same change that taught you.** If your work established or corrected a convention, revealed a trap that is invisible from the code, or proved something in this file wrong, update `CLAUDE.md` **in the same PR** — not later, not as a follow-up ticket. This is opportunistic maintenance done as part of the work that surfaced it, the same rule the knowledge graph already follows.

**Deciding there is nothing to record is a fine answer.** Forgetting to decide is the failure.

**Date what you verify; mark what you assume.** A fact checked today should say so, and a fact taken on trust should say *unverified* — then correcting it later is a small edit rather than an argument. A confidently wrong file is worse than no file, because it gets believed.

**Delete as readily as you add.** Anything the code already says plainly does not belong here; it dilutes the material that does. What belongs is a rule about how something must be done here, a trap invisible from the code, or a correction to an assumption someone reasonably held.

**When you find something that is broken rather than merely undocumented, raise it** — a ticket, a PR, a message. Writing a defect into this file and moving on documents the problem instead of fixing it.

A `Stop` hook and an advisory CI check both notice when a session changed something conventional and left this file untouched. Neither can tell whether you learned anything — nothing can. They exist to make the decision conscious, not to make it for you.
