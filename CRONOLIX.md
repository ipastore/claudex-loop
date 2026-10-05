# Cronolix fork

This fork of [chaseai-yt/claudex-loop](https://github.com/chaseai-yt/claudex-loop) carries two
patches the Cronolix team depends on. Every developer installs the plugin from here, so the patches
travel with the install instead of living in one machine's plugin cache.

```
/plugin marketplace add ipastore/claudex-loop
/plugin install claudex-loop@claudex-loop
```

Leave marketplace auto-update off. Upstream changes are merged into this fork deliberately.

## The patches (all in `skills/claudex-loop/scripts/runner.py`)

1. **Builds run unsandboxed.** `codex exec -s workspace-write` cannot reach
   `/var/run/docker.sock`, so a build could never run `supabase db reset` or the SQL isolation
   harness. Builds pass `--dangerously-bypass-approvals-and-sandbox`; **reviews stay `-s read-only`**.
2. **Same-provider review when both models are named and differ.** Plan review and inspection used
   to require the opposite *provider*. They now also accept the same provider when `--model` and
   `--counterpart-model` are both given and differ, e.g. Opus builds and Fable inspects.
   Same model is refused unless `--allow-same-model`, which is recorded in `result.json`.

Base: upstream `8cf5e2c` (2.1.0). Tests: `python3 -m unittest tests.test_runner`.
