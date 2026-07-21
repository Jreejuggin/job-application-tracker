# Setup

## 1. Import the workflow

Import [`../workflow/job_tracker.json`](../workflow/job_tracker.json) into n8n
(Workflows → Import from File).

## 2. Reconnect credentials

The JSON ships with placeholder credential IDs, not real ones. On import, reconnect:

- **Gmail Trigger** → your Gmail OAuth2 credential
- **Get All Applications** and **Append New Application** → your Google Sheets OAuth2
  credential

## 3. Point both Google Sheets nodes at your sheet

`Get All Applications` and `Append New Application` each have a `documentId` (currently
`YOUR_GOOGLE_SHEET_ID`) and `sheetName` (currently `Sheet1`) — update both to match your
actual spreadsheet.

## 4. Create the sheet

Add these columns, in this order:

| Date Applied | Company | Job Title | Job Type | Location | Application Link | Resume Used | Cover Letter | Status | Interview Date | Follow-Up Date | Salary Range | Contact Person | Notes | Additional Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|

## 5. Set "Job Type" as a dropdown column

In Google Sheets: Data → Data validation, with exactly these options — the workflow
always writes one of these, never free text:

- Software Developer
- Data Analyst
- IT Support
- Internship
- QA
- Web Developer
- Other

## 6. Activate the workflow

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
