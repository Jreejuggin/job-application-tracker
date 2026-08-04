# Setup

## 0. Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose installed
- This repo's [`docker-compose.yml`](../docker-compose.yml) at the project root
- Copy [`.env.example`](../.env.example) to `.env` in that same folder and fill in
  `N8N_ENCRYPTION_KEY` — generate one with `openssl rand -hex 32` if you don't have one
  yet, or reuse an existing one if you're migrating data from a prior install.
  **`.env` is gitignored and should never be committed** — it holds your real encryption
  key.

## 1. Start n8n

From the project root (same folder as `docker-compose.yml`):

```bash
docker compose up -d
```

n8n will be available at `http://localhost:5679`. With `restart: unless-stopped` set in
the compose file, it stays running across reboots as long as Docker is running — you
generally don't need to run this command again after the first time. See
[`../docker-quick-reference.md`](../docker-quick-reference.md) for day-to-day start/stop
commands.

## 2. Create your owner account (first time only)

Open `http://localhost:5679` and follow the setup screen to create your admin email and
password. **Save this somewhere durable** (a password manager) — self-hosted n8n has no
"forgot password" email flow by default. If you lose it, recovery is
`docker compose exec n8n n8n user-management:reset`, which removes all user accounts
(not workflows or credentials) and lets you set up a new owner account.

## 3. Import the workflow

Import [`../workflow/job_tracker.json`](../workflow/job_tracker.json) into n8n
(Workflows → Import from File).

## 4. Reconnect credentials

The JSON ships with placeholder credential IDs, not real ones. On import, reconnect:

- **Gmail Trigger** → your Gmail OAuth2 credential
- **Get All Applications** and **Append New Application** → your Google Sheets OAuth2
  credential

Both credentials can share the **same** Google Cloud OAuth Client ID/Secret, as long as
that OAuth client's project has both the Gmail API and Google Sheets API enabled — you
don't need two separate Google Cloud OAuth clients. Whichever credential you set up
first, copy the "OAuth Redirect URL" n8n shows on the credential screen into Google Cloud
Console → your OAuth client → Authorized redirect URIs. See
[`../n8n-google-oauth-credential-notes.md`](../n8n-google-oauth-credential-notes.md) if
you hit `invalid_client` errors during this step.

## 5. Point both Google Sheets nodes at your sheet

`Get All Applications` and `Append New Application` each have a `documentId` (currently
`YOUR_GOOGLE_SHEET_ID`) and `sheetName` (currently `Sheet1`) — update both to match your
actual spreadsheet.

## 6. Create the sheet

Add these columns, in this order:

| Date Applied | Company | Job Title | Job Type | Location | Application Link | Resume Used | Cover Letter | Status | Interview Date | Follow-Up Date | Salary Range | Contact Person | Notes | Additional Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|

## 7. Set "Job Type" as a dropdown column

In Google Sheets: Data → Data validation, with exactly these options — the workflow
always writes one of these, never free text:

- Software Developer
- Data Analyst
- IT Support
- Internship
- QA
- Web Developer
- Other

## 8. Activate the workflow

Turn the workflow on. It polls Gmail every minute by default.

## Notes / known limitations

- Timezone for "Date Applied" is hardcoded to `America/Toronto` (a constant,
  `APP_TIMEZONE`, near the top of the "Classify And Parse Application" code — change it if
  you're not in Toronto).
- Extraction is regex-based, not AI-based, by design. It's
  been tuned against real confirmation emails from LinkedIn, Indeed, Workday, and direct
  company ATS senders, but a new sender's phrasing may occasionally need a new pattern.
- If Company/Job Title come back blank (or wrong) for a new email format, the fix is
  almost always a small addition to the extraction patterns in the "Classify And Parse
  Application" code node — not a structural change. See `docs/architecture.md` for where
  each pattern lives.
- Self-hosted via Docker as of v1 (see the root `CHANGELOG.md` and
  `n8n-v3-migration-plan.md` for why — n8n's v3 release drops support for npm-based
  self-hosted installs).