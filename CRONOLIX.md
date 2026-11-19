# Cronolix fork

This fork of [chaseai-yt/claudex-loop](https://github.com/chaseai-yt/claudex-loop) carries three
patches the Cronolix team depends on. Every developer installs the plugin from here, so the patches
travel with the install instead of living in one machine's plugin cache.

```
/plugin marketplace add ipastore/claudex-loop
/plugin install claudex-loop@claudex-loop
```

Leave marketplace auto-update off. Upstream changes are merged into this fork deliberately.

## The patches (all in `skills/claudex-loop/scripts/runner.py`)

1. **Unsandboxed builds, opt-in per repo.** `codex exec -s workspace-write` cannot reach
   `/var/run/docker.sock`, so a build could never run `supabase db reset` or a SQL harness. A repo
   that needs it sets `CLAUDEX_UNSANDBOXED_BUILD=1` (in its `.claude/settings.json` `env`), and builds
   then pass `--dangerously-bypass-approvals-and-sandbox`. Off by default: it gives the builder the
   host's access, unattended. **Reviews stay `-s read-only` either way.**
2. **Same-provider review when both models are named and differ.** Plan review and inspection used
   to require the opposite *provider*. They now also accept the same provider when `--model` and
   `--counterpart-model` are both given and differ, e.g. Opus builds and Fable inspects.
   Same model is refused unless `--allow-same-model`, which is recorded in `result.json`.
3. **A timed-out build can be resumed.** A build killed by `--timeout` used to be recorded as
   `failed` with no session and no snapshot, so `--resume` refused it forever and the builder's
   work could only be finished by hand. It is now `interrupted`, with the session the CLI announced
   and the snapshot of the files it left; `--resume` accepts it for builds only. The checkout guard
   is unchanged: an edit made after the kill still refuses the resume. Found on Cronolix #512.

Base: upstream `8cf5e2c` (2.1.0). Tests: `python3 -m unittest tests.test_runner`.
