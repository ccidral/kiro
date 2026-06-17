# Twilio CLI Reference

## Output format details

| Flag | Description |
|------|-------------|
| `-o columns` | Default human-readable table |
| `-o json` | Full API response as JSON (ignores `--properties`) |
| `-o tsv` | Tab-separated values (respects `--properties`) |
| `-o none` | Suppress output |
| `--properties` | Comma-separated camelCase field names for column display |
| `--no-header` | Suppress column headers in columns/tsv output |
| `--silent` | Shorthand for `-l none -o none` |

## Message properties

Default: `sid, from, to, status, direction, dateSent`

All available (visible in JSON output): `sid`, `dateCreated`, `dateUpdated`, `dateSent`, `accountSid`, `to`, `from`, `messagingServiceSid`, `body`, `status`, `numSegments`, `numMedia`, `direction`, `apiVersion`, `price`, `priceUnit`, `errorCode`, `errorMessage`, `uri`, `subresourceUris`

## Call properties

Default: `sid, from, to, status, startTime`

All available: `sid`, `dateCreated`, `dateUpdated`, `parentCallSid`, `accountSid`, `to`, `toFormatted`, `from`, `fromFormatted`, `phoneNumberSid`, `status`, `startTime`, `endTime`, `duration`, `price`, `priceUnit`, `direction`, `answeredBy`, `apiVersion`, `forwardedFrom`, `groupSid`, `callerName`, `queueTime`, `trunkSid`, `uri`, `subresourceUris`

## Debugger log properties

Default: `dateCreated, logLevel, errorCode, alertText`

All available: `sid`, `accountSid`, `serviceSid`, `resourceSid`, `dateCreated`, `dateGenerated`, `dateUpdated`, `logLevel`, `errorCode`, `alertText`, `apiVersion`, `requestUrl`, `requestMethod`, `requestVariables`, `responseHeaders`, `responseBody`, `moreInfo`, `url`

## Error codes — common messaging errors

| Code | Description |
|------|-------------|
| 11200 | HTTP retrieval failure — your webhook URL is unreachable |
| 11201 | TCP connection timed out connecting to your server |
| 11202 | TCP connection refused by your server |
| 11203 | HTTP total timeout triggered |
| 21602 | Message body is required |
| 21606 | 'From' number not a valid message-capable Twilio number |
| 21610 | Attempt to send to unsubscribed recipient |
| 21611 | 'From' number exceeded max queued messages |
| 21614 | 'To' number is not a valid mobile number |
| 21617 | Concatenated message body exceeds 1600 char limit |
| 30001 | Queue overflow |
| 30002 | Account suspended |
| 30003 | Unreachable destination handset |
| 30004 | Message blocked |
| 30005 | Unknown destination handset |
| 30006 | Landline or unreachable carrier |
| 30007 | Message filtered (carrier or Twilio filtering) |
| 30008 | Unknown error |
| 30022 | US A2P 10DLC rate limits exceeded |
| 30034 | US A2P 10DLC — message from unregistered number |

## Error codes — common call errors

| Code | Description |
|------|-------------|
| 13224 | Dial: number invalid or unsupported by Twilio |
| 13225 | Dial: call blocked by Twilio |
| 21211 | Invalid 'To' phone number |
| 21214 | 'To' phone number cannot be reached |
| 31003 | Connection timeout (client SDK) |
| 31005 | Connection error (client SDK) |
| 32001 | SIP trunk CPS limit exceeded |

## Debugging CLI issues

If a command fails with an opaque error, re-run with `-l debug`:

```sh
twilio api core messages list -l debug -o json 2>debug.log
```

Debug logs go to stderr so they won't pollute piped stdout. The output shows:
- Config file path and active profile
- Provided flags and resolved parameters
- Full HTTP request URL and headers
- Response or connection error details

## Date filtering patterns

Messages use `--date-sent` / `--date-sent-after` / `--date-sent-before`.
Calls use `--start-time` / `--start-time-after` / `--start-time-before` and `--end-time` variants.
Debugger uses `--start-date` / `--end-date`.

All dates are UTC in `YYYY-MM-DD` format.

## Pagination

- Default page size: 50 records
- Max page size: 1000 (`--page-size 1000`)
- Use `--limit N` to cap total results
- Use `--no-limit` to retrieve everything (watch for timeouts on large datasets)
- API request timeout: 30 seconds
