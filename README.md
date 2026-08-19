# claude-team-example

A complete, minimal consumer of [**claude-team**](https://github.com/matt-whitaker/claude-team) —
a portable Claude + GitHub agent team you add to a repository with **one frozen workflow file and
one directory**.

This repo is two things at once:

1. **A showcase** — the smallest real integration, every line of it meant to be copied.
2. **The live test bed** — claude-team's own changes are drilled here before any real consumer
   takes them. Every run in [Actions](../../actions) is this repo doing its job.

## What claude-team is

Eight roles — a router, an architect, four authors, a security reviewer, a custodian — run as
GitHub Actions jobs around scripted hooks. You file an issue, apply the `@claude` label, and the
system shapes it, sequences it, writes the code, lands each task on a story branch, and opens one
PR for you to review. Authors never touch git; deterministic hooks do the committing, landing,
closing, and dispatching. A failed run captures its work to a `failure/*` branch instead of losing
it. **Trigger a story, come back to a PR.**

## The whole integration

```
your-repo/
├── .github/workflows/claude.yml   ← the stub: frozen, ~50 lines, never changes again
└── .claude-team/
    ├── prompts/_shared.md         ← your repo's voice: the gate, the conventions, the house rules
    ├── prompts/<role>.md          ← optional per-role overlays
    └── setup.sh                   ← optional: whatever your authors need installed
```

The stub declares the triggers and calls the reusable workflow. Everything with a pulse — prompts,
hooks, the job graph — lives upstream and is fetched at a pinned ref, so **updates propagate
without touching your repo**.

### The two lessons already baked into the stub

- ⚠️ **The permissions ceiling.** A called workflow's jobs cannot request permissions the caller
  didn't grant — omit the `permissions:` block and every run dies at startup with *"the nested job
  X is requesting … but is only allowed …"*. The first drill in this repo died exactly that way;
  the block in the stub is the fix.
- ⚠️ **Triggers, `run-name` and `concurrency` must live in the stub** — a called workflow's
  `run-name` is ignored and its workflow-level concurrency is unsupported.

## Reading the Actions tab

This repo has **no model credentials, on purpose**. A drill proves the machinery — routing, asset
fetching, hooks, the loop guard — and stops exactly at the boundary where a real consumer's
secrets would take over:

| you'll see | it means |
|---|---|
| `delegate` ✅ green | the router cloned the team, ran the hooks, picked the right role |
| a role job ✗ failing at its model step | correct — this repo deliberately has no credentials |
| an `issue_comment` run, skipped | the loop guard refusing the bot's own comment |

## Adopting it for real

1. Copy [`claude.yml`](.github/workflows/claude.yml) into `.github/workflows/`; set
   `project_owner`/`project_number` to your Projects board, and `node`/`browser` to what your
   authors need.
2. Create `.claude-team/prompts/_shared.md` — your repo's gate and conventions. This is where a
   repo's personality lives: a docs repo can make the Researcher its lead; an app repo leans on
   Implementor and Tester.
3. Create the labels: `@claude`, `@claude/<role>` for each role, `@claude/complete`, and the
   classification set (`epic`, `spike`, `bug`, `story`, `task`).
4. Add the secrets a real consumer needs: `CLAUDE_CODE_OAUTH_TOKEN` (the model), and optionally
   `PROJECTS_TOKEN` (board moves), `DISPATCH_APP_ID`/`DISPATCH_APP_PRIVATE_KEY` (the cascade —
   dark without them, by design), and the transcript-capture AWS pair.
5. File an issue, apply `@claude`, watch.

## Versioning

Pin the stub's `@ref`. `mainline` is the living edge — this example tracks it because finding
breakage first is its job. A real consumer should pin a tag once tags exist; the upstream test
suite asserts on every push that all of a release's pins move together.
