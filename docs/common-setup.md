# Common setup

Everything in this guide applies to all projects in this repository.

## 1. Requirements

- An n8n instance: [n8n Cloud](https://n8n.io) or self-hosted (Docker is the easiest route). The exports use current node versions (Gmail v2.1, Google Sheets v4.4/4.5, Merge v3), so use a recent 1.x release.
- A Google account for Google Sheets and (for most projects) Gmail.
- The API keys listed in each project's README.

## 2. Import a workflow

1. In n8n, create a new workflow.
2. Click the **⋯** menu (top right) → **Import from File**.
3. Select the `workflow.json` from the project folder.
4. The workflow imports **inactive**. Leave it that way until you have finished setup.

Credentials are not included in the exports. After import, nodes that need one show a warning icon. Open them and attach your own credential.

## 3. Create the Google Sheet

Each project has a `sheets/` folder with one CSV per tab.

1. Create a new Google Sheet.
2. For each CSV: **File → Import → Upload → Insert new sheet(s)**, then rename the tab to match the CSV file name exactly (for example `Qualified Leads`).
3. Copy the sheet ID from the URL (`https://docs.google.com/spreadsheets/d/<THIS_PART>/edit`). You will paste it into the workflow's config.

Column names are used for lookups and updates, so do not rename them.

## 4. Credentials

| Credential | Used for | How to create in n8n |
|---|---|---|
| **Google Sheets OAuth2** | Reading and writing leads | Credentials → New → Google Sheets OAuth2 API. See [n8n's Google credentials guide](https://docs.n8n.io/integrations/builtin/credentials/google/). Self-hosted installs need their own Google Cloud OAuth client. |
| **Gmail OAuth2** | Sending emails | Credentials → New → Gmail OAuth2 API (same Google Cloud setup) |
| **OpenAI API** | Project 01 | Credentials → New → OpenAI API |
| **HTTP Basic Auth** | Twilio (project 03) | Username = Twilio Account SID, password = Auth Token |

## 5. API keys in the Set Config node

Most workflows read their API keys from a **Set Config** node near the start (for example `anthropic_key`, `scrapingbee_key`, `apify_key`). Replace each `YOUR_..._HERE` value with your real key.

This is convenient for a demo, but it means your keys live inside the workflow JSON. For anything beyond testing:

- Move keys into n8n credentials (Header Auth or Query Auth) and attach them to the HTTP Request nodes, or
- Use environment variables (`{{ $env.ANTHROPIC_API_KEY }}`) on self-hosted n8n.

Never export a workflow that contains real keys to a public repository.

## 6. Test safely

- Run the workflow manually with **Execute Workflow** before activating.
- While testing, **disable the Gmail/Twilio send nodes** (right-click → Deactivate) so nothing goes out to real people, or point the recipient at your own address.
- Use small limits first (fewer niches/locations, a lower result count) to keep API costs predictable.
- Check the first few generated emails by eye before you let it run unattended.

## 7. Webhooks and reply handling

Some projects include a webhook for replies:

- Use the **Production URL** of the webhook node once the workflow is active (the Test URL only works while you are listening in the editor).
- Gmail does not push replies to webhooks by itself. Projects 02 and 03 describe what needs to forward replies to the webhook.

## 8. Time zone

Schedule triggers use your n8n time zone (instance settings on n8n Cloud, or the `GENERIC_TIMEZONE` environment variable when self-hosting; a workflow can also override it in its own settings). Set it before activating, because the sending windows (for example 9AM to 6PM weekdays) depend on it.

## 9. Responsible use

These workflows contact real people. Use accurate claims in the generated copy, include a way to opt out, honour opt-outs, respect the laws of the countries you target, and respect the terms of the services you scrape. When in doubt, send less.
