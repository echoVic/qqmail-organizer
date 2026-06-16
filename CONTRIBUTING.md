# Contributing to QQMail Organizer

QQMail Organizer is an agent-facing skill for OpenClaw, Hermes, Codex, and
similar runtimes. Contributions should preserve deterministic execution,
machine-readable outputs, and explicit safety boundaries around mailbox
mutations.

## Project Layout

The publishable skill lives in:

```text
skills/qqmail-organizer/
```

Important files:

```text
skills/qqmail-organizer/SKILL.md
skills/qqmail-organizer/scripts/qqmail.py
skills/qqmail-organizer/scripts/auto_archive.py
skills/qqmail-organizer/rules.agent.json
skills/qqmail-organizer/rules.schema.json
skills/qqmail-organizer/agents/openai.yaml
```

The repository root contains project documentation, license, and contribution
metadata. Do not move the publishable skill out of `skills/qqmail-organizer/`
without checking both ClawHub and skills.sh installation behavior.

## Safety Requirements

Do not commit secrets or private mailbox policy:

- No `QQMAIL_AUTH_CODE`
- No real mailbox address
- No ClawHub token
- No GitHub token
- No inbox exports
- No private `rules.local.json`
- No cron logs containing private mail content

Keep these files public and generic:

- `rules.agent.json` must remain a safe default policy. It should not contain
  user-specific rules.
- `rules.schema.json` is the contract for private rules generated outside the
  public repo.
- README examples must use placeholders such as `123456789@qq.com` and
  `your-qqmail-authorization-code`.

Any command that sends, replies, deletes, marks, archives, or applies rules is a
mutation. New features must preserve the dry-run/apply boundary.

## Agent Interface Rules

When changing scripts or skill instructions:

- Prefer deterministic script behavior over free-form agent reasoning.
- Prefer `--json` output for parent-agent orchestration.
- Keep `plan-organize` read-only.
- Keep `auto-organize` dry-run by default.
- Keep `send --dry-run` and `reply --dry-run` available.
- Keep permanent delete guarded by `--confirm-delete DELETE`.
- Do not print credentials, raw auth headers, or full mailbox dumps.
- Do not expose IMAP internal message IDs in JSON output intended for agents.
- Validate generated private rules before running them.

If a new command mutates mailbox state, it should support a dry-run mode unless
there is a clear reason it cannot.

## Development Setup

Use Python 3. The scripts rely only on the Python standard library.

Run commands from the repository root unless noted otherwise:

```bash
cd /root/qqmail-organizer
```

To test against a real QQ Mail account, set credentials in the shell only:

```bash
export QQMAIL_USER='123456789@qq.com'
export QQMAIL_AUTH_CODE='your-qqmail-authorization-code'
```

Do not write these values into tracked files.

## Validation Checklist

Run this before every pull request or release:

```bash
python3 /root/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  /root/qqmail-organizer/skills/qqmail-organizer

python3 -m py_compile \
  skills/qqmail-organizer/scripts/qqmail.py \
  skills/qqmail-organizer/scripts/auto_archive.py

python3 -m json.tool skills/qqmail-organizer/rules.agent.json >/tmp/rules.agent.json
python3 -m json.tool skills/qqmail-organizer/rules.schema.json >/tmp/rules.schema.json

python3 skills/qqmail-organizer/scripts/qqmail.py validate-rules --json \
  >/tmp/qqmail-validate-rules.json
python3 -m json.tool /tmp/qqmail-validate-rules.json >/dev/null

git diff --check
```

Scan for known secret patterns before committing:

```bash
rg -n 'QQMAIL_AUTH_CODE=|QQMAIL_USER=|clh_[A-Za-z0-9_-]+|ghp_[A-Za-z0-9_]+' \
  /root/qqmail-organizer || true
```

Expected matches should be placeholders only, not real credentials.

## Non-Mutating Runtime Tests

When QQ Mail credentials are available, run only read-only or dry-run tests by
default:

```bash
python3 skills/qqmail-organizer/scripts/qqmail.py plan-organize --limit 10 --json \
  >/tmp/qqmail-plan.json
python3 -m json.tool /tmp/qqmail-plan.json >/dev/null

python3 skills/qqmail-organizer/scripts/qqmail.py auto-organize --limit 10 --json \
  >/tmp/qqmail-auto.json
python3 -m json.tool /tmp/qqmail-auto.json >/dev/null

python3 skills/qqmail-organizer/scripts/qqmail.py send \
  --to test@example.com \
  --subject "Dry run" \
  --body "Hello" \
  --dry-run

python3 skills/qqmail-organizer/scripts/qqmail.py delete --index 1 --dry-run
```

Do not run these in validation unless explicitly approved:

```bash
python3 skills/qqmail-organizer/scripts/qqmail.py send ...
python3 skills/qqmail-organizer/scripts/qqmail.py reply ...
python3 skills/qqmail-organizer/scripts/qqmail.py auto-organize --apply ...
python3 skills/qqmail-organizer/scripts/qqmail.py delete --confirm-delete DELETE ...
```

## Rule Policy

The packaged default rules file is intentionally empty:

```text
skills/qqmail-organizer/rules.agent.json
```

Private mailbox policies should be stored outside the public repo, for example:

```text
rules.local.json
```

Validate private rules before use:

```bash
python3 skills/qqmail-organizer/scripts/qqmail.py validate-rules \
  --rules /path/to/rules.local.json \
  --json
```

Preview before applying:

```bash
python3 skills/qqmail-organizer/scripts/qqmail.py auto-organize \
  --rules /path/to/rules.local.json \
  --limit 100 \
  --json
```

Apply only after the user approves the exact dry-run action set:

```bash
python3 skills/qqmail-organizer/scripts/qqmail.py auto-organize \
  --rules /path/to/rules.local.json \
  --limit 100 \
  --apply \
  --json
```

## Commit Guidance

Keep commits focused:

- Script behavior changes should include validation notes.
- Documentation-only commits should not modify skill runtime files.
- Do not commit generated `__pycache__`, `.pyc`, `.env`, logs, or private rule
  files.
- Keep README, `SKILL.md`, and `agents/openai.yaml` consistent when changing
  user-facing or agent-facing behavior.

## Release Process

### GitHub and skills.sh

Push changes to GitHub:

```bash
git push
```

Then verify skills.sh installation from a temporary directory:

```bash
tmpdir=$(mktemp -d)
cd "$tmpdir"
npx skills add echoVic/qqmail-organizer --skill qqmail-organizer --agent codex -y --copy
```

The skills.sh page is:

```text
https://www.skills.sh/echovic/qqmail-organizer/qqmail-organizer
```

Search indexing may lag behind the detail page.

### OpenClaw ClawHub

ClawHub releases are versioned. Publish a new semver version when OpenClaw
market users should receive an updated package:

```bash
clawhub publish /root/qqmail-organizer/skills/qqmail-organizer \
  --slug qqmail-organizer \
  --name "QQMail Organizer" \
  --version 0.1.1 \
  --tags qqmail,email,imap,smtp,agent \
  --changelog "Describe the release."
```

After publishing, verify:

```bash
clawhub inspect qqmail-organizer

tmpdir=$(mktemp -d)
clawhub --workdir "$tmpdir" install qqmail-organizer
python3 /root/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  "$tmpdir/skills/qqmail-organizer"
```

Keep the ClawHub CLI logged in on the maintainer machine unless explicitly
asked to clear it.

## Reporting Issues

When reporting a bug, include:

- Command run
- Whether the command was read-only, dry-run, or apply
- Python version
- Sanitized error output
- Sanitized rule file shape if rules were involved

Do not include authorization codes, full email bodies, mailbox exports, or
private sender/recipient addresses unless they are already public test data.
