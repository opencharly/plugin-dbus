# plugin-dbus

The `dbus:` check verb for OpenCharly — interact with D-Bus services inside a
running deployment (list, call, introspect, notify), served out-of-process
(`verb:dbus`).

The plugin is a standalone Go module the charly loader host-builds and serves
over go-plugin gRPC, so the D-Bus driver lives here, out of charly's core check
surface. It is **EXEC-based**: the host attaches its live deploy executor over
the reverse channel and this plugin dials back through the SDK to run `gdbus`
against the venue's session bus — it owns no podman/SSH machinery and no
`godbus`.

## What it provides

| Capability | Surface |
|---|---|
| `verb:dbus` | the declarative `dbus:` check step |

## The verb

| Method | Fields | Meaning |
|---|---|---|
| `list` | — | list the registered D-Bus service names |
| `call` | `dest`, `path`, `member` | invoke a method on a bus name |
| `introspect` | `dest`, `path` | dump the introspection XML for an object |
| `notify` | `text` | send a desktop notification |

`member` is the fully-qualified `interface.Method` a `call` invokes (renamed from
the former shared `method` step modifier). Typed `arg` values
(`type:value` — string/uint32/int32/int64/uint64/boolean/double) are converted to
GVariant text for `gdbus`. All the dbus-exclusive fields live in the plugin's own
`#DbusInput` (`schema/dbus.cue`); only the shared matchers (`exit_status`,
`stdout`, `stderr`) and `description` ride the base step op.

## How to use it

Compose the plugin candy in a box or check bed's `candy:` list:

```yaml
- '@github.com/opencharly/plugin-dbus/candy/plugin-dbus:<tag>'
```

Then author the verb in a plan:

```yaml
- check: the session bus is reachable and lists its services
  dbus: list
  context: [runtime]
```

## Layout

- `candy/plugin-dbus/` — the plugin module: `plugin.go` (the provider +
  `NewProvider()` / `NewMeta()`), `provider.go` (the `Invoke` verdict path),
  `methods.go` (the gdbus dispatch), `schema/dbus.cue` (the self-contained
  `#DbusInput`), `params/cue_types_gen.go`, `methods_test.go`,
  `cmd/serve/main.go`.
- `charly.yml` — the root project manifest (`discover: candy`).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.

## Related

- Owning skill: `/charly-check:dbus` — the `dbus:` verb reference. This candy
  carries no `skill:` entity of its own; the gap is tracked in
  [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291).
- `/charly-check:check` — the declarative check-step surface the verb is
  authored through.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI.
