# AGENTS.md — plugin-dbus

Standalone out-of-tree plugin repo serving the `dbus` live-container check verb
(`verb:dbus`, EXEC-based). The plugin is a Go module at `candy/plugin-dbus/`
(module path `github.com/opencharly/plugin-dbus/candy/plugin-dbus`); the root
`charly.yml` only declares `discover: candy` so the repo is a project and its
candy is scanned.

Canonical files:

- `candy/plugin-dbus/charly.yml` — the `plugin-dbus:` candy entity (`plugin:`
  block, `plan:` check).
- `candy/plugin-dbus/plugin.go` / `provider.go` — the verb provider and the
  `Invoke` verdict path.
- `candy/plugin-dbus/methods.go` — the `gdbus` dispatch (list/call/introspect/
  notify).
- `candy/plugin-dbus/schema/dbus.cue` — the self-contained `#DbusInput` (single
  source for `params/cue_types_gen.go`).
- `charly.yml` — the root project manifest (`discover: candy`).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-check:dbus` — the `dbus:` verb reference. Load before changing the
  verb surface. This candy carries no `skill:` entity of its own; the gap is
  tracked in
  [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291).
- `/charly-internals:plugin` — the plugin authoring reference: the `plugin:`
  block, the `verb` provider class, the per-plugin CUE-schema contract.
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `go build ./...` in `candy/plugin-dbus/` — compile the plugin module.
- `go test ./...` in `candy/plugin-dbus/` — the plugin's Go tests (the GVariant
  arg conversion + dispatch seams).
- `charly box validate` at the repo root — the structural check (the candy +
  `plugin:` block, CUE schema).
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate.
- The R10 consumer is a desktop pod bed whose check composes this plugin (the
  `sway-browser-vnc` bed's dbus list probe).

## Modify this repo

- Edit the `plugin-dbus:` candy entity, the Go source, and `schema/dbus.cue`
  **together** — the schema is the single source for the verb's `params/` struct;
  regenerate `params/cue_types_gen.go` from it.
- Keep the plugin free of podman/SSH machinery and `godbus`: it drives the
  venue's session bus with `gdbus` over the executor reverse channel only.
  `godbus` stays in charly's core for the Secret Service / GPG secrets.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
