---
name: google-workspace-cli
description: "Read, write, and manage Google Workspace data using the gws CLI. Use when the user shares a Google Sheets, Docs, Drive, Gmail, or Calendar link, asks to read or update a spreadsheet, fetch a document, list Drive files, search Gmail, check calendar events, or any task involving Google Workspace data. Triggers include Google Docs/Sheets/Drive URLs, 'read this sheet', 'update the spreadsheet', 'check my calendar', 'search Gmail for', 'list Drive files', or any reference to Google Workspace content."
---

# Google Workspace CLI (`gws`)

Access Google Sheets, Docs, Drive, Gmail, Calendar, Slides, and more via the `gws` command-line tool.

## Prerequisites

- `gws` is installed globally: `npm install -g @googleworkspace/cli`
- Auth is configured: `gws auth login` (credentials stored encrypted at `~/.config/gws/`)
- Authenticated as: check with `gws auth status`

## Command Structure

All commands follow the pattern:

```bash
gws <service> <resource> <method> --params '<JSON>'
```

Parameters are passed as a JSON string via `--params`. Use `--format table` for human-readable output or omit for JSON.

## Google Sheets

### Get spreadsheet metadata (sheet names, grid sizes)

```bash
gws sheets spreadsheets get --params '{"spreadsheetId": "<ID>", "fields": "sheets.properties"}'
```

### Read cell values

```bash
gws sheets spreadsheets values get --params '{"spreadsheetId": "<ID>", "range": "<SheetName>"}'
gws sheets spreadsheets values get --params '{"spreadsheetId": "<ID>", "range": "<SheetName>!A1:D10"}'
```

Use `--format table` for readable output, `--format csv` for CSV.

### Write cell values

```bash
gws sheets spreadsheets values update --params '{"spreadsheetId": "<ID>", "range": "<SheetName>!A1", "valueInputOption": "USER_ENTERED"}' <<'EOF'
{"values": [["Header1", "Header2"], ["val1", "val2"]]}
EOF
```

### Append rows

```bash
gws sheets spreadsheets values append --params '{"spreadsheetId": "<ID>", "range": "<SheetName>", "valueInputOption": "USER_ENTERED"}' <<'EOF'
{"values": [["new1", "new2"], ["new3", "new4"]]}
EOF
```

### Batch update (formatting, add sheets, etc.)

```bash
gws sheets spreadsheets batchUpdate --params '{"spreadsheetId": "<ID>"}' <<'EOF'
{"requests": [{"addSheet": {"properties": {"title": "NewTab"}}}]}
EOF
```

### Create a new spreadsheet

```bash
gws sheets spreadsheets create <<'EOF'
{"properties": {"title": "My New Sheet"}, "sheets": [{"properties": {"title": "Data"}}]}
EOF
```

## Google Docs

### Get document content

```bash
gws docs documents get --params '{"documentId": "<ID>"}'
```

### Create a document

```bash
gws docs documents create <<'EOF'
{"title": "My Document"}
EOF
```

### Update document content

```bash
gws docs documents batchUpdate --params '{"documentId": "<ID>"}' <<'EOF'
{"requests": [{"insertText": {"location": {"index": 1}, "text": "Hello World\n"}}]}
EOF
```

## Google Drive

### List files

```bash
gws drive files list --params '{"pageSize": 10, "fields": "files(id,name,mimeType,modifiedTime)"}'
```

### Search files

```bash
gws drive files list --params '{"q": "name contains '\''report'\'' and mimeType = '\''application/vnd.google-apps.spreadsheet'\''", "fields": "files(id,name)"}'
```

### Download a file

```bash
gws drive files get --params '{"fileId": "<ID>", "alt": "media"}' -o output.pdf
```

### Upload a file

```bash
gws drive files create --params '{"name": "report.pdf", "mimeType": "application/pdf"}' --upload report.pdf
```

## Gmail

### List messages

```bash
gws gmail users messages list --params '{"userId": "me", "maxResults": 10, "q": "is:unread"}'
```

### Read a message

```bash
gws gmail users messages get --params '{"userId": "me", "id": "<messageId>", "format": "full"}'
```

### Search emails

```bash
gws gmail users messages list --params '{"userId": "me", "q": "from:boss@company.com subject:review after:2026/03/01"}'
```

### Send an email

```bash
gws gmail users messages send --params '{"userId": "me"}' <<'EOF'
{"raw": "<base64-encoded-MIME-message>"}
EOF
```

## Google Calendar

### List upcoming events

```bash
gws calendar events list --params '{"calendarId": "primary", "timeMin": "2026-03-17T00:00:00Z", "maxResults": 10, "singleEvents": true, "orderBy": "startTime"}'
```

### Create an event

```bash
gws calendar events insert --params '{"calendarId": "primary"}' <<'EOF'
{"summary": "Team Standup", "start": {"dateTime": "2026-03-18T09:00:00+05:30"}, "end": {"dateTime": "2026-03-18T09:30:00+05:30"}}
EOF
```

### List calendars

```bash
gws calendar calendarList list
```

## Google Slides

### Get presentation

```bash
gws slides presentations get --params '{"presentationId": "<ID>"}'
```

### Create presentation

```bash
gws slides presentations create <<'EOF'
{"title": "Quarterly Review"}
EOF
```

## Extracting IDs from URLs

Google Workspace URLs contain the document/spreadsheet/presentation ID:

| URL Pattern | ID Location |
|-------------|-------------|
| `docs.google.com/spreadsheets/d/<ID>/edit` | Between `/d/` and `/edit` |
| `docs.google.com/document/d/<ID>/edit` | Between `/d/` and `/edit` |
| `docs.google.com/presentation/d/<ID>/edit` | Between `/d/` and `/edit` |
| `drive.google.com/file/d/<ID>/view` | Between `/d/` and `/view` |

The `gid` query parameter in Sheets URLs maps to `sheetId` in the spreadsheet metadata. Use `spreadsheets get` with `fields=sheets.properties` to find the sheet name for a given `gid`.

## Output Formats

```bash
--format json    # Default, structured output
--format table   # Human-readable table
--format csv     # CSV (useful for Sheets data)
--format yaml    # YAML output
```

## Pagination

For large result sets, use `--page-all` to auto-paginate:

```bash
gws drive files list --params '{"pageSize": 100, "fields": "files(id,name)"}' --page-all
```

## Useful Flags

| Flag | Purpose |
|------|---------|
| `--format table` | Human-readable output |
| `--format csv` | CSV output |
| `--dry-run` | Preview request without sending |
| `--page-all` | Auto-paginate through all results |
| `-o <path>` | Save output to file |
| `--help` | Show help for any command |

## Discovering Commands

The CLI is dynamically built from Google's Discovery Service. To explore:

```bash
gws sheets --help              # List Sheets resources
gws sheets spreadsheets --help  # List spreadsheets methods
gws sheets spreadsheets values get --help  # Show method details
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `No credentials found` | Run `gws auth login` |
| `Token expired` | Tokens auto-refresh; if stuck, run `gws auth login` again |
| `Unable to parse range` | Sheet name may have spaces -- use exact name from `spreadsheets get` metadata |
| `Permission denied` | Ensure the authenticated account has access to the resource |
| `Unknown parameter` | Use `--params '<JSON>'` syntax, not `--spreadsheet-id` style flags |

## Auth Reference

```bash
gws auth status   # Check current auth state
gws auth login    # Re-authenticate
gws auth setup    # First-time setup (requires gcloud CLI)
```

Credentials are encrypted at rest (AES-256-GCM) in `~/.config/gws/credentials.enc` with keys in the OS keyring.
