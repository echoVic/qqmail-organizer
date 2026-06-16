# QQMail Organizer

Safe QQ Mail inbox management over IMAP/SMTP.

This repository contains an OpenClaw/Codex skill for reading, searching,
replying to, classifying, and organizing QQ Mail inboxes. It is designed around
safe defaults: cleanup plans and rule-based organization are dry-run by default,
and changes only happen when explicitly requested.

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

## Requirements

- Python 3.
- QQ Mail IMAP/SMTP enabled.
- QQ Mail authorization code.

Set credentials in the environment:

```bash
export QQMAIL_USER='123456789@qq.com'
export QQMAIL_AUTH_CODE='your-qqmail-authorization-code'
```

`QQMAIL_AUTH_CODE` is the QQ Mail IMAP/SMTP authorization code, not your QQ
password. Never commit it to git.

## Quick Start

```bash
python3 scripts/qqmail.py folders
python3 scripts/qqmail.py inbox --limit 20
python3 scripts/qqmail.py plan-organize --limit 50
python3 scripts/qqmail.py auto-organize --limit 100
```

`auto-organize` is dry-run by default. To apply matching archive/mark rules:

```bash
python3 scripts/qqmail.py auto-organize --limit 100 --apply
```

## Rules

The default sample rules live in `rules.example.json`.

For personalized rules, copy the example file and pass it explicitly:

```bash
cp rules.example.json rules.local.json
python3 scripts/qqmail.py auto-organize --rules rules.local.json --limit 100
```

## Safety

- Run `plan-organize` first.
- Run archive/delete/mark commands with `--dry-run` before applying.
- Do not run `delete` unless permanent deletion is intended.
- Do not create automatic reply or delete cron jobs without explicit
  confirmation.
- Do not commit credentials, inbox exports, or personal rule files.

## License

MIT.
