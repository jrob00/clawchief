# GWS Setup

This is a hard prerequisite.

## 1. Install GWS

Pick one method:

**npm (requires Node.js 18+):**
```bash
npm install -g @googleworkspace/cli
gws --help
```

**Homebrew:**
```bash
brew install googleworkspace/tap/gws
gws --help
```

**Pre-built binary:**

Download from [github.com/googleworkspace/cli/releases](https://github.com/googleworkspace/cli/releases), add to PATH, then run `gws --help`.

## 2. Authenticate

**Option A: Automated setup (recommended for new projects)**

```bash
gws auth setup
```

This creates a GCP project, enables APIs, and opens a browser for OAuth consent automatically.

**Option B: Use an existing GCP project**

If you already have a GCP project, create or choose a project in Google Cloud Console, then:

1. Open **APIs & Services**
2. Enable the APIs you need:
   - Gmail API
   - Google Calendar API
   - Google Sheets API
   - Google Drive API
   - Google Docs API
   - People API
3. Configure the **OAuth consent screen** if prompted
4. Add the operating Google account as a test user if required

Then authenticate:

```bash
gws auth login
```

## 3. Verify auth status

```bash
gws auth status
```

Confirm the correct Google account is authenticated.

## 4. Verify Gmail message search

```bash
gws gmail +triage --query 'in:inbox newer_than:7d' --max-results 5
```

Important: use **message-level** Gmail search for inbox sweeps. Do not rely on thread-only search.

## 5. Verify Calendar

```bash
gws calendar calendarList.list
gws calendar +agenda --days 2 --max-results 20
```

## 6. Verify Sheets

```bash
gws sheets spreadsheets.get --params '{"spreadsheetId":"{{GOOGLE_SHEET_ID}}"}'
```

## Stop conditions

Stop and fix setup if:

- `gws auth status` does not show the correct Google account
- Gmail works but Calendar or Sheets does not
- the tracker sheet is not shared to the authenticated account
- someone uses thread-only search where message search is required
- `gws` is not on PATH after install
