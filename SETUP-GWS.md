# GWS Setup

This is a hard prerequisite.

`gws` is Google's official Workspace CLI. It dynamically discovers Google APIs at runtime and includes built-in agent skills for common operations.

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

If you already have a GCP project with the required APIs enabled (Gmail, Calendar, Sheets, Drive, Docs, People), use:

```bash
gws auth login
```

Required APIs (same as `SETUP-GOG.md`):
- Gmail API
- Google Calendar API
- Google Sheets API
- Google Drive API
- Google Docs API
- People API

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
