# Track company career jobs with n8n and Google Sheets

An inactive starter from DataSmithLab. Collect a bounded snapshot of public career-page jobs and keep a deduplicated log in your own Google Sheet.

[Career Page Jobs Scraper on Apify](https://apify.com/datasmithlab/ats-jobs-scraper) · [Walkthrough](https://datasmith-lab.github.io/datasmithlab-ats-n8n/) · [Download starter ZIP](https://github.com/Datasmith-lab/datasmithlab-ats-n8n/releases/latest/download/career-jobs-to-sheets-starter.zip)

## What it does

1. Reads existing records from a dedicated Jobs tab.
2. Starts one Apify Actor run and waits for completion.
3. Checks HEALTH, COMPANIES and dataset counts before writing.
4. Upserts on a stable input-scope + job-ID key, preserving firstSeenAt and older records.

First records in an empty scope are `baseline`. Later unseen IDs are `logged`. First-seen time is not publication time. Limited snapshots can discover older jobs later. Jobs absent from a later snapshot are retained; this workflow does not detect closed jobs.

## Setup

Requires your own Apify account, n8n with HTTP Request, Code, Wait and Google Sheets nodes, and a Google account with spreadsheet edit access. No Slack or Telegram account is needed.

1. Create a **Jobs** tab in your spreadsheet and paste the single header row from `jobs-sheet-headers.csv` into A1. Preserve existing tabs. An existing Jobs table must use exactly these headers.
2. Import `career-jobs-to-sheets.n8n.json`. It is inactive and contains no credentials.
3. In Configuration replace `REPLACE_WITH_SPREADSHEET_ID` with your own spreadsheet ID.
4. Create an n8n **Header Auth** credential: header name `Authorization`, value `Bearer YOUR_APIFY_TOKEN`. Select it on every Apify HTTP Request node. Store the token in the credential manager, never in a public workflow or Sheet.
5. Connect your own Google Sheets OAuth2 credential on Read saved jobs and Save jobs.
6. Run manually. Check the Actor reports and the Jobs tab, then run again to verify stable IDs are updated rather than duplicated.
7. Only after validation activate the daily **09:15 UTC** schedule. Self-hosted n8n must remain running. Run one execution at a time; concurrent writes are not supported.

The default is Stripe, 10 jobs total and per company, descriptions off, no date filter and `onlyNewSinceLastRun:false`. The Actor's internal tracking is disabled because this starter tracks observed IDs in Sheets. You may configure 1–5 distinct company inputs and limits up to 100. Use exact board URLs for precise targeting; Workday requires a full board URL. Changing companies or filters creates a new scope/baseline. Raising limits does not guarantee complete coverage or make first-seen jobs newly published.

The configured Actor charge cap is $0.10 per run. Current Store prices are authoritative. This cap is not an end-to-end n8n/Google/cloud cost guarantee.

## Failures and data limits

Missing reports, failed company resolution, dataset-count mismatch, missing required job fields or duplicate source/saved IDs stop the workflow before writes. Inspect the failed execution and existing Actor run before launching another. A Google Sheets upsert can partially complete; rerun sequentially after fixing the cause. Previously written stable keys can then be updated. This is not a transactional delivery guarantee.

Only selected career pages are scanned. Limits, filters, source access and ATS detection determine coverage. Missing salary and publication dates stay blank. No recruiter contacts, applications or personal applicant data are collected by this package. Do not share your Sheet publicly merely to use the workflow.

## Validation

See the accompanying publication report for measured live results. Import compatibility and real delivery have been tested on self-hosted n8n 2.26.4; other versions and cloud instances require your own setup check. This package has not been accepted into the official n8n template directory.
