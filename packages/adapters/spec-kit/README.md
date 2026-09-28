# @adrkit/spec-kit

The [Spec Kit](https://github.com/github/spec-kit) extension for
[adrkit](https://adrkit.dev) — decision memory for the spec-driven plan loop.

Spec Kit takes you from `specify` to `plan` to `tasks` to `implement`. What it
does not do is check the plan it just produced against the decisions your
organization already made, or record the new decisions the plan contains. Every
feature starts from an empty context and re-litigates settled questions.

This extension closes that loop, without leaving git.

## What it adds

| Command | What it does | Writes |
|---|---|---|
| `/speckit.adrkit.context` | Pulls the decisions governing the paths you're about to touch — including superseded and rejected ones — into context before you plan | no |
| `/speckit.adrkit.check` | Checks a produced plan against the decisions that govern it, and routes it through the deterministic evaluator when a snapshot bundle is configured | no |
| `/speckit.adrkit.draft` | Scaffolds a draft ADR from the current plan artifact for you to fill in | one new record |

Plus one hook: `after_plan` offers to run `/speckit.adrkit.check`. It is
**optional** — it asks, it does not seize the plan loop.

## Requirements

- Spec Kit `>=1.0.0,<1.1.0`. Compatibility is verified against the current
  `specify 1.0.2.dev0` release line.
- The `adr` CLI (`npm install -g @adrkit/cli`), a project-local installation,
  or `ADRKIT_CLI` pointing at its entry point.
- An ADR corpus. Defaults to `docs/adr`.

## Install

For a published release, install from the
[Spec Kit community catalog](https://github.com/github/spec-kit/blob/main/extensions/catalog.community.json).
The catalog entry may target a different compatibility line than this worktree:

```sh
specify extension add adrkit
```

For this Spec Kit 1.0 compatibility line, install the adapter from the checkout
being tested:

```sh
specify extension add --dev path/to/packages/adapters/spec-kit
```

Then `/speckit.adrkit.context` is available in your agent, and `/speckit.plan`
will offer the `after_plan` hook.

The published npm package remains available as `@adrkit/spec-kit`; this
worktree's Spec Kit 1.0 adapter is installed with `--dev` until its release
artifact is published.

## Use it in the plan loop

1. Run `/speckit.adrkit.context` before planning.
2. Run `/speckit.plan`, then accept the optional check or run
   `/speckit.adrkit.check` directly.
3. Run `/speckit.adrkit.draft` when the plan introduces a decision worth
   recording.

## Configuration

The extension works without additional configuration. Override its defaults
with environment variables when needed.

| Variable | Default | Meaning |
|---|---|---|
| `ADRKIT_DIR` | `docs/adr` | ADR corpus directory |
| `ADRKIT_CLI` | resolved | Explicit path to the `adr` entry point |
| `ADRKIT_FEATURE_DIR` | resolved | Override Spec Kit's own feature resolution |
| `ADRKIT_SNAPSHOT` | unset | Snapshot bundle enabling the deterministic evaluator |
| `ADRKIT_AS_OF` | today, UTC | Evaluation date |

The CLI is resolved in a fixed order: `ADRKIT_CLI`, then
`./node_modules/.bin/adr`, then `adr` on `PATH`. No branch of that reaches the
network — a missing CLI is reported, never fetched.

## Runtime guarantees

- **Hooks never write.** `draft` is the only command that writes, and it is
  unreachable from any hook. A plan-phase hook creating records unprompted would
  manufacture decision memory rather than record it.
- **The hook is never mandatory.** `optional: false` renders as an automatic hook
  that fires without consent.
- **Failures name what is missing.** No command exits 0 having found nothing
  because it was looking in the wrong place. "0 decisions govern this" and "I
  could not see the corpus" must never render as the same string.
- **`check` mutates nothing.**
- **Nothing development-only reaches your repo.** `specify extension add --dev`
  copies this directory verbatim, so `.extensionignore` keeps the test suite,
  `tsconfig.json`, and `package.json` out of your `.specify/extensions/`. The
  package declares no dependencies at all, so there is never a `node_modules/`
  to copy — a workspace symlink in one aborts the install partway through.

## Compatibility and support

The extension is verified against Spec Kit `1.0.2.dev0`, including install,
command registration, and the optional `after_plan` hook. Report compatibility
issues in the
[adrkit issue tracker](https://github.com/mbeacom/adrkit/issues). The adapter
keeps a bounded upper version limit; widening it requires another install and
render verification.

## License

Apache-2.0.
