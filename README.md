# Hermes Agent Setup

A headless, multi-provider AI agent setup: one CLI, several model
providers behind it, and a fallback chain so a single dead provider
never stops your work.

This repo documents the **pattern** — how to structure providers,
fallbacks, skills, and budgets. Every credential and endpoint here is a
placeholder. Put your real keys in a `0600` file outside this repo, never
in git, never in chat.

## How it works

```
you → CLI (oneshot) → provider 1 (primary)
                       ↘ provider 2 (fallback, if 1 fails)
                         ↘ provider 3 (fallback, if 2 fails)
```

- The agent runs **headless**: one query in, one answer out, no
  interactive session. Good for scripts, cron jobs, and subagents.
- Each provider is a separate entry in the config: base URL, API key
  (from the environment), and model.
- If the primary provider fails, the next one is tried automatically.
  Nothing migrates by itself — you decide the order.
- Skills live in an external directory and are loaded read-only, so the
  agent's playbooks survive reinstalls.

## Contents

| File | What it is |
| --- | --- |
| [SETUP.md](SETUP.md) | Installation and first run, step by step |
| [config.example.yaml](config.example.yaml) | Config template — placeholders only |
| [PROVIDER-FALLBACK.md](PROVIDER-FALLBACK.md) | How the fallback chain works and how to order it |

## Rules

1. **Keys live in the environment, never in the repo.** `0600`
   permissions, loaded from a file outside git.
2. **Never print a key.** Verify by key name or hash prefix only.
3. **Measure before trusting.** Record latency and cost per provider
   before making one the primary.
4. **One change per iteration** when debugging provider issues — same
   request, different provider, compare.
5. **Never use an "auto-approve everything" mode.** The agent asks
   before irreversible actions, always.

## License

MIT — see [LICENSE](LICENSE).
