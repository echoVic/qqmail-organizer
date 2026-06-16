# QQMail Organizer

Agent-facing QQ Mail inbox management over IMAP/SMTP.

This repository is designed for OpenClaw, Hermes, Codex, and other AI agents.
It is not primarily a human email client. The skill provides deterministic
scripts and an agent operation protocol for reading, searching, replying to,
classifying, and safely organizing QQ Mail inboxes.

Default behavior is conservative:

- Planning is read-only.
- Rule-based organization is dry-run by default.
- Mutations require explicit commands.
- External sends/replies and permanent deletion require explicit user intent.
- Credentials are read from environment variables and must never be logged,
  committed, or copied into prompts.

## Agent Contract

Agents should use this repository as a controlled mailbox operation layer:

1. Start ambiguous cleanup or triage requests with `plan-organize`.
2. Preview rule-based cleanup with `auto-organize` without `--apply`.
3. Apply only after the user confirms the proposed action set.
4. Report summaries and counts, not full mailbox dumps.
5. Ask before permanent delete or auto-reply policies.
6. Never reveal `QQMAIL_AUTH_CODE`.

## Features

- Read recent QQ Mail messages.
- Read a message by newest-first index.
- Search by sender, subject, date, unread state, and folder.
- Send and reply to email through QQ Mail SMTP.
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

## Agent Workflow

```bash
python3 scripts/qqmail.py folders
python3 scripts/qqmail.py inbox --limit 20
python3 scripts/qqmail.py plan-organize --limit 50
python3 scripts/qqmail.py auto-organize --limit 100
```

`auto-organize` is dry-run by default. Apply only after explicit confirmation:

```bash
python3 scripts/qqmail.py auto-organize --limit 100 --apply
```

## Rules

The default sample rules live in `rules.example.json`. Treat them as example
policy, not as a user's private production policy.

For personalized rules, copy the example file and pass it explicitly:

```bash
cp rules.example.json rules.local.json
python3 scripts/qqmail.py auto-organize --rules rules.local.json --limit 100
```

## Agent Safety Rules

- Run `plan-organize` first.
- Run archive/delete/mark commands with `--dry-run` before applying.
- Do not run `delete` unless permanent deletion is intended.
- Do not create automatic reply or delete cron jobs without explicit
  confirmation.
- Do not commit credentials, inbox exports, or personal rule files.
- Do not dump full email bodies unless the user asks to read a specific message.
- Do not infer that "organize" means "delete".

## License

MIT.
