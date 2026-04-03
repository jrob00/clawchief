# Google Workspace CLI Commands

This file maps every Google Workspace CLI operation used by clawchief skills to both supported CLIs: `gog` and `gws`.

Check `~/.openclaw/workspace/WORKSPACE-CLI.md` for which CLI is active in this install.

Skill files show `gog` command examples by default. When `gws` is the active CLI, use this reference to translate before executing.

## Important differences

- `gog` uses `-a EMAIL` to select the authenticated account. `gws` does not — the active account is set during auth setup.
- `gog` uses `--json --results-only` for structured output. `gws` returns JSON by default.
- `gws` offers agent skills (prefixed with `+`) that simplify common operations. Prefer these over raw discovery commands when available.
- `gws` raw commands follow the pattern `gws <service> <resource>.<method> --params '{...}'`.

---

## Auth

### Set up credentials

gog:
```bash
gog auth credentials /path/to/client_secret.json
```

gws:
```bash
gws auth setup
```

`gws auth setup` creates the GCP project and OAuth client automatically. If you already have a GCP project, use `gws auth login` instead.

### Add / authenticate an account

gog:
```bash
gog auth add {{ASSISTANT_EMAIL}} --services gmail,calendar,drive,contacts,docs,sheets
```

gws:
```bash
gws auth login
```

`gws auth login` opens a browser flow for the Google account. Scopes are requested per-service on first use.

### List authenticated accounts

gog:
```bash
gog auth list
```

gws:
```bash
gws auth status
```

---

## Gmail

### Search messages

gog:
```bash
gog gmail messages search -a {{ASSISTANT_EMAIL}} 'in:inbox newer_than:3d (is:unread OR is:important)' --max=10 --json --results-only
```

gws:
```bash
gws gmail +triage --query 'in:inbox newer_than:3d (is:unread OR is:important)' --max-results 10
```

Or using the raw API:
```bash
gws gmail users.messages.list --params '{"userId":"me","q":"in:inbox newer_than:3d (is:unread OR is:important)","maxResults":10}'
```

### Get a single message by ID

gog:
```bash
gog gmail get -a {{ASSISTANT_EMAIL}} MESSAGE_ID --json --results-only
```

gws:
```bash
gws gmail +read --message-id MESSAGE_ID
```

Or using the raw API:
```bash
gws gmail users.messages.get --params '{"userId":"me","id":"MESSAGE_ID","format":"full"}'
```

### Send a reply (in-thread, reply-all)

gog:
```bash
gog gmail send -a {{ASSISTANT_EMAIL}} --reply-to-message-id=MESSAGE_ID --reply-all --cc='{{PRIMARY_WORK_EMAIL}}' --subject='RE: SUBJECT' --body-file=/tmp/reply.txt
```

gws:
```bash
gws gmail +reply --message-id MESSAGE_ID --reply-all --cc '{{PRIMARY_WORK_EMAIL}}' --body-file /tmp/reply.txt
```

### Send a new message

gog:
```bash
gog gmail send -a {{ASSISTANT_EMAIL}} --to='person@example.com' --cc='{{PRIMARY_WORK_EMAIL}}' --subject='SUBJECT' --body-file=/tmp/reply.txt
```

gws:
```bash
gws gmail +send --to 'person@example.com' --cc '{{PRIMARY_WORK_EMAIL}}' --subject 'SUBJECT' --body-file /tmp/reply.txt
```

### Mark a message as read

gog:
```bash
gog gmail mark-read -a {{ASSISTANT_EMAIL}} MESSAGE_ID
```

gws:
```bash
gws gmail users.messages.modify --params '{"userId":"me","id":"MESSAGE_ID"}' --json '{"removeLabelIds":["UNREAD"]}'
```

### Archive a message

gog:
```bash
gog gmail archive -a {{ASSISTANT_EMAIL}} MESSAGE_ID
```

gws:
```bash
gws gmail users.messages.modify --params '{"userId":"me","id":"MESSAGE_ID"}' --json '{"removeLabelIds":["INBOX"]}'
```

### Mark read and archive (combined)

gog:
```bash
gog gmail messages modify -a {{ASSISTANT_EMAIL}} MESSAGE_ID --remove=UNREAD,INBOX
```

gws:
```bash
gws gmail users.messages.modify --params '{"userId":"me","id":"MESSAGE_ID"}' --json '{"removeLabelIds":["UNREAD","INBOX"]}'
```

---

## Calendar

### List calendars

gog:
```bash
gog calendar calendars -a {{ASSISTANT_EMAIL}} --json --results-only
```

gws:
```bash
gws calendar calendarList.list
```

### List events (all calendars, date range)

gog:
```bash
gog calendar events --all -a {{ASSISTANT_EMAIL}} --days=2 --max=50 --json --results-only
```

gws:
```bash
gws calendar +agenda --days 2 --max-results 50
```

Or using the raw API with a specific calendar:
```bash
gws calendar events.list --params '{"calendarId":"primary","timeMin":"RFC3339","timeMax":"RFC3339","maxResults":50,"singleEvents":true,"orderBy":"startTime"}'
```

Note: to check all calendars with gws, first list calendars, then query events for each relevant calendar ID.

### Create an event

gog:
```bash
gog calendar create {{PRIMARY_WORK_EMAIL}} -a {{ASSISTANT_EMAIL}} --summary='TITLE' --from='RFC3339' --to='RFC3339' --attendees='a@example.com,b@example.com' --description='CONTEXT' --with-meet --send-updates all
```

gws:
```bash
gws calendar +insert --calendar-id '{{PRIMARY_WORK_EMAIL}}' --summary 'TITLE' --start 'RFC3339' --end 'RFC3339' --attendees 'a@example.com,b@example.com' --description 'CONTEXT' --add-meet --send-updates all
```

Or using the raw API:
```bash
gws calendar events.insert --params '{"calendarId":"{{PRIMARY_WORK_EMAIL}}","sendUpdates":"all","conferenceDataVersion":1}' --json '{
  "summary": "TITLE",
  "start": {"dateTime": "RFC3339"},
  "end": {"dateTime": "RFC3339"},
  "attendees": [{"email":"a@example.com"},{"email":"b@example.com"}],
  "description": "CONTEXT",
  "conferenceData": {"createRequest": {"requestId": "unique-id"}}
}'
```

### Update an event

gog:
```bash
gog calendar update {{PRIMARY_WORK_EMAIL}} EVENT_ID -a {{ASSISTANT_EMAIL}} --from='RFC3339' --to='RFC3339' --send-updates all
```

gws:
```bash
gws calendar events.patch --params '{"calendarId":"{{PRIMARY_WORK_EMAIL}}","eventId":"EVENT_ID","sendUpdates":"all"}' --json '{
  "start": {"dateTime": "RFC3339"},
  "end": {"dateTime": "RFC3339"}
}'
```

---

## Sheets

### Read a range

gog:
```bash
gog sheets get {{GOOGLE_SHEET_ID}} 'Leads!A:N' -a {{ASSISTANT_EMAIL}} --json --results-only
```

gws:
```bash
gws sheets +read --spreadsheet-id '{{GOOGLE_SHEET_ID}}' --range 'Leads!A:N'
```

Or using the raw API:
```bash
gws sheets spreadsheets.values.get --params '{"spreadsheetId":"{{GOOGLE_SHEET_ID}}","range":"Leads!A:N"}'
```

### Update a range

gog:
```bash
gog sheets update {{GOOGLE_SHEET_ID}} 'Leads!A13:N13' --values-json='[[...]]' -a {{ASSISTANT_EMAIL}} --json --results-only
```

gws:
```bash
gws sheets spreadsheets.values.update --params '{"spreadsheetId":"{{GOOGLE_SHEET_ID}}","range":"Leads!A13:N13","valueInputOption":"USER_ENTERED"}' --json '{"values":[[...]]}'
```

### Append rows

gog:
```bash
gog sheets append {{GOOGLE_SHEET_ID}} 'Leads!A:N' --values-json='[[...]]' -a {{ASSISTANT_EMAIL}} --json --results-only
```

gws:
```bash
gws sheets +append --spreadsheet-id '{{GOOGLE_SHEET_ID}}' --range 'Leads!A:N' --values '[[...]]'
```

Or using the raw API:
```bash
gws sheets spreadsheets.values.append --params '{"spreadsheetId":"{{GOOGLE_SHEET_ID}}","range":"Leads!A:N","valueInputOption":"USER_ENTERED"}' --json '{"values":[[...]]}'
```

### Get sheet metadata

gog:
```bash
gog sheets metadata {{GOOGLE_SHEET_ID}} -a {{ASSISTANT_EMAIL}} --json --results-only
```

gws:
```bash
gws sheets spreadsheets.get --params '{"spreadsheetId":"{{GOOGLE_SHEET_ID}}"}'
```
