---
name: qqmail-organizer
description: Safe QQ Mail inbox management over IMAP/SMTP. Use when the user wants to read, search, reply to, classify, organize, archive, mark read/unread, or safely clean up a QQ Mail inbox. Especially useful for inbox triage, dry-run cleanup plans, rule-based organization, newsletter/marketing/GitHub notification sorting, and avoiding accidental destructive email operations.
---

# QQMail Organizer

## Overview

Use this skill to manage QQ Mail through standard IMAP/SMTP using the bundled Python scripts. The skill is safety-first: analyze and preview with `plan-organize` or `auto-organize` before moving, marking, or deleting messages.

Credentials are read only from environment variables:

- `QQMAIL_USER`: QQ email address.
- `QQMAIL_AUTH_CODE`: QQ Mail IMAP/SMTP authorization code, not the account password.

Never print, persist, or place `QQMAIL_AUTH_CODE` in prompts, reports, cron jobs, memory, commits, or rule files.

## Recommended Workflow

1. Start with a non-mutating plan:

```bash
python3 scripts/qqmail.py plan-organize --limit 50
```

2. Preview rule-based cleanup. This is dry-run by default:

```bash
python3 scripts/qqmail.py auto-organize --limit 100
```

3. Apply only after the user explicitly confirms:

```bash
python3 scripts/qqmail.py auto-organize --limit 100 --apply
```

## Commands

Read and inspect:

```bash
python3 scripts/qqmail.py folders
python3 scripts/qqmail.py inbox --limit 20
python3 scripts/qqmail.py inbox --unread --limit 20
python3 scripts/qqmail.py read --index 1
python3 scripts/qqmail.py search --from "sender@example.com"
python3 scripts/qqmail.py search --subject "keyword" --since "2026-06-01"
```

Send and reply:

```bash
python3 scripts/qqmail.py send --to "recipient@example.com" --subject "Hello" --body "Message"
python3 scripts/qqmail.py reply --index 1 --body "Reply text"
```

Organize safely:

```bash
python3 scripts/qqmail.py plan-organize --limit 50
python3 scripts/qqmail.py auto-organize --limit 100
python3 scripts/qqmail.py mkdir "GitHub"
python3 scripts/qqmail.py archive --from "notifications@github.com" --target "GitHub" --dry-run
python3 scripts/qqmail.py mark-read --index 1 --dry-run
python3 scripts/qqmail.py mark-unread --from "sender@example.com" --dry-run
```

Delete is available but should be used only after explicit user confirmation:

```bash
python3 scripts/qqmail.py delete --index 1 --dry-run
```

## Rules

Rule-based organization uses `rules.example.json` by default. Rules support matching by sender, subject, or inferred category, and actions such as:

- `archive`: copy to a target folder, then remove from source.
- `mark-read`: add the `Seen` flag.
- `mark-unread`: remove the `Seen` flag.
- `review`: classify only, do not mutate.

For personalized cleanup, copy `rules.example.json` to another path and call:

```bash
python3 scripts/qqmail.py auto-organize --rules /path/to/rules.json --limit 100
```

## Safety Policy

- Prefer `plan-organize` before any mutation.
- Use `--dry-run` for manual archive, delete, and mark operations before applying.
- `auto-organize` is dry-run unless `--apply` is present.
- Never run `delete` unless the user explicitly asks for permanent deletion.
- Never create automatic reply, delete, or archive cron jobs without explicit user confirmation.
- Do not log or commit QQ Mail authorization codes.
