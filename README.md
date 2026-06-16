# QQMail Organizer

Agent-facing QQ Mail inbox management over IMAP/SMTP.

This repository is designed for OpenClaw, Hermes, Codex, and other AI agents.
It is not primarily a human email client. The skill provides deterministic
scripts and an agent operation protocol for reading, searching, replying to,
classifying, and safely organizing QQ Mail inboxes.

The publishable OpenClaw skill lives in `skills/qqmail-organizer/`. Commands
below assume the current working directory is that skill root after installation
or after running:

```bash
cd skills/qqmail-organizer
```

Default behavior is conservative:

- Planning is read-only.
- Rule-based organization is dry-run by default.
- Mutations require explicit commands.
- `plan-organize --json` and `auto-organize --json` provide machine-readable
  output for parent agents.
- `send --dry-run` and `reply --dry-run` preview externally visible messages.
- Permanent deletion requires `--confirm-delete DELETE`.
- External sends/replies and permanent deletion require explicit user intent.
- Credentials are read from environment variables and must never be logged,
  committed, or copied into prompts.

## Agent Contract

Agents should use this repository as a controlled mailbox operation layer:

1. Start ambiguous cleanup or triage requests with `plan-organize`.
2. Use `--json` when another agent will parse the result.
3. Preview rule-based cleanup with `auto-organize` without `--apply`.
4. Apply only after the user confirms the proposed action set.
5. Report summaries and counts, not full mailbox dumps.
6. Ask before permanent delete or auto-reply policies.
7. Never reveal `QQMAIL_AUTH_CODE`.

## Features

- Read recent QQ Mail messages.
- Read a message by newest-first index.
- Search by sender, subject, date, unread state, and folder.
- Preview, send, and reply to email through QQ Mail SMTP.
- Classify recent mail into categories such as GitHub, recruiting, security,
  finance, newsletter, marketing, and review.
- Preview rule-based cleanup before changing anything.
- Archive messages into folders.
- Mark messages read or unread.
- Create folders.
- Handle non-standard email charsets such as `unknown-8bit`.

## Runtime Requirements

- Python 3.
- QQ Mail IMAP/SMTP enabled.
- QQ Mail authorization code.

Set credentials in the agent runtime environment:

```bash
export QQMAIL_USER='123456789@qq.com'
export QQMAIL_AUTH_CODE='your-qqmail-authorization-code'
```

`QQMAIL_AUTH_CODE` is the QQ Mail IMAP/SMTP authorization code, not your QQ
password. Never commit it to git.

## OpenClaw Market

Publish the skill directory with ClawHub:

```bash
clawhub publish "$(pwd)/skills/qqmail-organizer" \
  --slug qqmail-organizer \
  --name "QQMail Organizer" \
  --version 0.1.0 \
  --tags qqmail,email,imap,smtp,agent
```

Authentication is required before publishing:

```bash
clawhub login --token "$CLAWHUB_TOKEN" --no-browser
```

## Agent Workflow

```bash
python3 scripts/qqmail.py folders
python3 scripts/qqmail.py inbox --limit 20
python3 scripts/qqmail.py plan-organize --limit 50 --json
python3 scripts/qqmail.py auto-organize --limit 100 --json
```

`auto-organize` is dry-run by default. Apply only after explicit confirmation:

```bash
python3 scripts/qqmail.py auto-organize --limit 100 --apply
```

Preview sends and replies before externally visible actions:

```bash
python3 scripts/qqmail.py send --to user@example.com --subject "Hello" --body "Message" --dry-run
python3 scripts/qqmail.py reply --index 1 --body "Reply text" --dry-run
```

Permanent deletion requires both a selector and an exact confirmation phrase:

```bash
python3 scripts/qqmail.py delete --index 1 --dry-run
python3 scripts/qqmail.py delete --index 1 --confirm-delete DELETE
```

## Rules

The default policy lives in `rules.agent.json`. It intentionally contains no
rules, so a packaged copy of this skill never archives, marks, or moves mail
based on public sample preferences.

Use `rules.schema.json` as the machine-readable contract when an agent generates
a private policy file. Pass that private policy explicitly:

```bash
python3 scripts/qqmail.py validate-rules --rules rules.local.json --json
python3 scripts/qqmail.py auto-organize --rules rules.local.json --limit 100 --json
```

### Private Policy Example

When a user asks for a mailbox policy, generate a private rules file outside the
public skill repo. For example, if the user says:

> Archive GitHub notifications to the GitHub folder. Keep marketing mail in
> review; do not delete it.

An agent can write a private file such as `rules.local.json`:

```json
{
  "version": 1,
  "rules": [
    {
      "name": "GitHub notifications",
      "category": "github",
      "from_contains": "notifications@github.com",
      "action": "archive",
      "target": "GitHub"
    },
    {
      "name": "Marketing review",
      "category": "marketing",
      "match_category": "marketing",
      "action": "review"
    }
  ]
}
```

Validate first:

```bash
python3 scripts/qqmail.py validate-rules --rules rules.local.json --json
```

Preview without changing the mailbox:

```bash
python3 scripts/qqmail.py auto-organize --rules rules.local.json --limit 100 --json
```

Apply only after the user confirms the exact action set:

```bash
python3 scripts/qqmail.py auto-organize --rules rules.local.json --limit 100 --apply --json
```

Keep private policy files out of public commits when they encode a user's
mailbox preferences.

## Agent Safety Rules

- Run `plan-organize` first.
- Run archive/delete/mark commands with `--dry-run` before applying.
- Run send/reply commands with `--dry-run` before sending unless exact content
  has already been approved.
- Do not run `delete` unless permanent deletion is intended.
- Do not run permanent deletion without `--confirm-delete DELETE`.
- Do not create automatic reply or delete cron jobs without explicit
  confirmation.
- Do not commit credentials, inbox exports, or personal rule files.
- Do not dump full email bodies unless the user asks to read a specific message.
- Do not infer that "organize" means "delete".

## License

MIT.
