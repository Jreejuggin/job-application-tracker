# Job Application Tracker (n8n)

An n8n workflow that watches Gmail for job application confirmation emails, extracts the
key details, and logs them as a new row in a Google Sheet — automatically, with duplicate
protection.

Built and iterated against real confirmation emails from LinkedIn, Indeed, Workday, and
direct company ATS senders — see [`docs/architecture.md`](docs/architecture.md) for how
the extraction logic handles the differences between them.

## Quick start

1. Import [`workflow/job_tracker.json`](workflow/job_tracker.json) into n8n.
2. Follow [`docs/setup.md`](docs/setup.md) for credentials, sheet setup, and activation.
3. See [`sample-data/sample_application_email.txt`](sample-data/sample_application_email.txt)
   for an example of the kind of email this is designed to parse.

## What it does

- Watches Gmail for new mail, classifies job application confirmations vs. everything
  else (including job alert digests, which look deceptively similar to real confirmations)
- Extracts Company, Job Title, Date Applied, Source, Location, Employment Type, Recruiter
  info, Job URL, and Application ID using layered regex + fallback logic
- Categorizes the role into a fixed set of dropdown values
- Checks for duplicates before appending a new row to Google Sheets

## Repo structure

```
job-application-tracker/
├── README.md
├── workflow/
│   └── job_tracker.json          the n8n workflow, ready to import
├── docs/
│   ├── architecture.md           node-by-node breakdown + flow diagram
│   ├── setup.md                  full setup instructions
│   └── screenshots/               (add your own n8n canvas screenshots here)
├── sample-data/
│   └── sample_application_email.txt
├── CHANGELOG.md
└── LICENSE
```

## License

MIT — see [`LICENSE`](LICENSE).
