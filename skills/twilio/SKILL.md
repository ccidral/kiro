---
name: twilio
description: Search Twilio messages, calls, and error logs using the twilio CLI. Use when investigating SMS/MMS delivery, phone call history, debugging Twilio errors, or looking up communication records by phone number, date, or status.
---

# Twilio CLI

Search and inspect Twilio messages, calls, and error logs via the `twilio` CLI.

For error code interpretation and advanced tips, see [REFERENCE.md](REFERENCE.md).

## Output format

Always use `-o json` for machine-readable output. Other formats: `-o columns` (default), `-o tsv`, `-o none`.

`--properties` selects columns (camelCase JSON field names). Has **no effect** with `-o json`.

## Profiles

Switch accounts with `--profile <name>`. List profiles: `twilio profiles list`

## Messages

```sh
twilio api core messages list -o json --limit 20
twilio api core messages fetch --sid SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX -o json
```

Filters: `--to`, `--from`, `--date-sent`, `--date-sent-after`, `--date-sent-before`, `--limit N`

Example — messages from a number in a date range:
```sh
twilio api core messages list --from +15552229999 --date-sent-after 2026-06-01 --date-sent-before 2026-06-15 -o json
```

## Calls

```sh
twilio api core calls list -o json --limit 20
twilio api core calls fetch --sid CAXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX -o json
```

Filters: `--to`, `--from`, `--status` (queued|ringing|in-progress|canceled|completed|failed|busy|no-answer), `--start-time`, `--start-time-after`, `--start-time-before`, `--end-time`, `--end-time-after`, `--end-time-before`, `--parent-call-sid`, `--limit N`

### Call events and notifications

```sh
twilio api core calls events list --call-sid CAXX... -o json
twilio api core calls notifications list --call-sid CAXX... -o json
```

Notification filter: `--log 0` (errors), `--log 1` (warnings).

## Error logs (Debugger)

```sh
twilio debugger logs list -o json
```

Filters: `--log-level error|warning|notice|debug`, `--start-date YYYY-MM-DD`, `--end-date YYYY-MM-DD`

## Error code lookup

Look up any error code at: `https://www.twilio.com/docs/api/errors/{CODE}`

Full dictionary as JSON: `https://www.twilio.com/docs/api/errors/twilio-error-codes.json`

Common codes:
- `11200` — HTTP retrieval failure (webhook unreachable)
- `21610` — Send to unsubscribed recipient
- `30003` — Unreachable destination handset
- `30004` — Message blocked
- `30005` — Unknown destination handset
- `30006` — Landline or unreachable carrier
- `30007` — Message filtered

## Tips

- Dates: UTC, format `YYYY-MM-DD`. Phone numbers: E.164 (`+1XXXXXXXXXX`).
- Default limit: 50 records. API timeout: 30 seconds.
- Use `--no-limit` for all records (can be slow). Pipe to `jq` for filtering.
- Debug failures: add `-l debug` (logs go to stderr).
- Suppress all output: `--silent` (shorthand for `-l none -o none`).
