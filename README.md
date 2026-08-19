# claude-team-fixture

A fixture consumer for [claude-team](https://github.com/matt-whitaker/claude-team): the frozen
stub plus a minimal `.claude-team/`, used to exercise the reusable workflow's plumbing before a
real consumer takes a change. No model secrets are configured — model steps fail by design; the
scripted jobs around them are what this repo tests.
