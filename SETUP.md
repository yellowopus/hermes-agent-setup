# Setup

## 1. Install

Install the agent CLI into your home directory (not a system path, so it
survives environment resets):

```sh
# example — follow the agent's official install docs for your platform
curl -fsSL <OFFICIAL_INSTALL_URL> | sh
# or: download the release binary into ~/.local/bin
```

Verify the install:

```sh
agent --version
which agent   # should point inside $HOME
```

## 2. Store credentials

Create a private env file **outside** any git repo:

```sh
mkdir -p ~/.agent
cat > ~/.agent/.env <<'EOF'
PROVIDER_1_API_KEY=<your-primary-provider-key>
PROVIDER_2_API_KEY=<your-fallback-provider-key>
PROVIDER_3_API_KEY=<your-second-fallback-key>
EOF
chmod 600 ~/.agent/.env
```

Rules:

- `0600` permissions. No exceptions.
- Never paste a key into chat, logs, or a file inside a repo.
- Move keys via stdin or a file write — never interpolate them into a
  shell command where they can land in history.
- Rotate a key the moment it appears anywhere it should not.

## 3. Configure

Copy the template and fill in your providers:

```sh
cp config.example.yaml ~/.agent/config.yaml
# edit ~/.agent/config.yaml — replace every <PLACEHOLDER>
```

Key settings:

- `providers`: ordered list. The first entry is the primary; the rest
  are fallbacks in order.
- `skills.external_dirs`: directories of skill playbooks, loaded
  read-only. Point it at your own skills directory.
- `model`: the model per provider. Use the provider's documented model
  names, copied from the provider's own list — never from memory.

## 4. First run (headless oneshot)

```sh
agent chat --oneshot -Q --format text \
  --query-file /path/to/query.txt \
  --max-turns 5 --run-budget 120
```

- `--oneshot`: one query, one answer, exit. No interactive session.
- `--max-turns`: cap the agent's turns so a stuck run ends by itself.
- `--run-budget`: wall-clock budget in seconds.
- Never enable an "auto-approve everything" flag. The agent must ask
  before irreversible actions.

## 5. Verify

- [ ] `--version` works and the binary is under `$HOME`.
- [ ] A oneshot query returns an answer from the primary provider.
- [ ] Kill the primary (wrong key in a test copy of the env file) and
  confirm the fallback answers instead.
- [ ] A skill from `external_dirs` loads and is listed.
- [ ] No key appears in any output, log, or file you created.
