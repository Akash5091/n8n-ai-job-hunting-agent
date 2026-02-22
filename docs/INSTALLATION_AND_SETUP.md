# Installation & Setup Guide

## Prerequisites

Before you begin, ensure you have:

- **n8n** (v1.0+) - self-hosted or cloud instance
- **Node.js** (v16+) - if self-hosting
- **npm** or **yarn**
- Active accounts for:
  - [Anthropic Claude API](https://console.anthropic.com)
  - Google Account (Gmail, Google Drive, Google Sheets)
  - [Apify Account](https://apify.com) (for job scraping)
  - Slack Workspace (optional, for notifications)

## Step 1: Set Up Anthropic Claude API

### Get Your API Key

1. Go to [Anthropic Console](https://console.anthropic.com/account/billing/overview)
2. Sign up or log in
3. Navigate to **API Keys** section
4. Create a new API key
5. Copy and store securely (you'll need this for n8n credentials)

### Set Billing

- Enable pay-as-you-go billing
- Set spending limit to prevent unexpected charges
- Monitor usage at https://console.anthropic.com/account/usage

## Step 2: Set Up Google Integration

### Create Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (name: "n8n-job-agent")
3. Enable these APIs:
   - Google Sheets API
   - Google Drive API
   - Gmail API
4. Create OAuth 2.0 credentials (Desktop app)
5. Download the credentials JSON file

### Create Google Sheet & Folder

1. Open [Google Sheets](https://sheets.google.com)
2. Create new spreadsheet: "Job Hunt Tracker"
3. Create two sheets:
   - **JobLeads** - Main job tracking
   - **Applications** - Application history

**JobLeads Columns:**
```
A: Date
B: Company
C: Role
D: Location
E: Job URL
F: Apply URL
G: Seniority
H: Tech Match (%)
I: Top Keywords
J: ATS Score
K: Resume File (URL)
L: Status
M: Notes
```

**Applications Columns:**
```
A: Date
B: Company
C: Role
D: Apply URL
E: Resume File (URL)
F: Status
G: Recruiter Name
H: Recruiter Email
I: Outreach Sent
J: Follow-up Date
K: Notes
```

4. Get Sheet ID from URL: `https://docs.google.com/spreadsheets/d/{SHEET_ID}/edit`
5. Create folder in Drive: "AkashJobResumes"
6. Get folder ID from URL: `https://drive.google.com/drive/folders/{FOLDER_ID}`

## Step 3: Set Up Apify for Job Scraping

### Create Apify Account

1. Sign up at [apify.com](https://apify.com)
2. Go to **API Tokens** under Account
3. Create/copy API token

### Add Job Scraping Actors

Create saved actor runs for:

1. **LinkedIn Jobs Scraper** (apify/linkedin-jobs-scraper)
   - Search queries: "Java Developer", "Full Stack Engineer", "Backend Engineer", etc.
   - Location: United States
   - Max results: 25
   - Posted date: Last 24 hours

2. **Indeed Jobs Scraper** (apify/indeed-scraper)
   - Query: "Java Developer"
   - Location: US

3. **Workday Jobs Scraper** (apify/workday-jobs-scraper)
   - For target companies using Workday

## Step 4: Install & Configure n8n

### Option A: Self-Hosted n8n

```bash
# Install globally
npm install -g n8n

# Create working directory
mkdir ~/n8n-projects
cd ~/n8n-projects

# Start n8n
n8n start

# Access at http://localhost:5678
```

### Option B: n8n Cloud

1. Go to [n8n cloud](https://n8n.cloud)
2. Sign up/Log in
3. Create new project
4. Skip to "Step 5" below

## Step 5: Add Credentials to n8n

Login to your n8n instance and navigate to **Settings > Credentials**.

Add these credentials:

### 1. Anthropic Claude API
- **Credential Type**: Anthropic
- **API Key**: `sk-ant-...` (from Step 1)
- **Name**: "Claude_API"

### 2. Google Sheets OAuth2
- **Credential Type**: Google Sheets
- **Authentication**: OAuth2
- Upload credentials JSON from Google Cloud Console
- **Name**: "Google_Sheets_OAuth"

### 3. Google Drive OAuth2
- **Credential Type**: Google Drive
- **Authentication**: OAuth2
- Use same credentials as Google Sheets
- **Name**: "Google_Drive_OAuth"

### 4. Gmail OAuth2
- **Credential Type**: Gmail
- **Authentication**: OAuth2
- Use same credentials as Google Sheets
- **Name**: "Gmail_OAuth"

### 5. Slack (Optional)
- **Credential Type**: Slack OAuth2
- **Name**: "Slack_API"

### 6. Apify API Token
- **Credential Type**: Generic Credential Type
- **Authentication Method**: HTTP Header Auth
- **Header Name**: `Authorization`
- **Header Value**: `Bearer {APIFY_TOKEN}`
- **Name**: "Apify_API"

## Step 6: Set Environment Variables

In n8n Settings > Environment, add:

```bash
GOOGLE_SHEET_ID=your-sheet-id-here
GOOGLE_DRIVE_FOLDER_ID=your-folder-id-here
CANDIDATE_NAME=Akash Reddy Bommireddy
NOTIFICATION_EMAIL=your-email@example.com
SLACK_CHANNEL=#job-alerts
MIN_ATS_SCORE=90
MAX_ATS_ITERATIONS=3
APPLY_MODE=HUMANINLOOP
BASE_RESUME_TEXT="<paste-your-full-resume-here>"
```

## Step 7: Import Workflow

1. In n8n Dashboard, click **+ New**
2. Select **Import from URL** or **Import from file**
3. Use the workflow JSON from `/workflows/akash-job-agent.json`
4. Verify all nodes have credentials assigned
5. Test with **Manual Trigger** node

## Step 8: Configure Triggers

### Schedule Trigger (Automatic)

Set to run 8:00 AM CST, Monday-Friday:
```
Cron: 0 8 * * 1-5
Timezone: America/Chicago
```

### Manual Trigger (Testing)

Always enabled - use to test workflow ad-hoc.

## Step 9: Test the Workflow

1. Click **Test Workflow** in the workflow editor
2. Monitor execution in the execution history
3. Expected results in 5-15 minutes:
   - Google Sheet updates with jobs
   - Resume PDFs created in Drive
   - Slack notification sent

## Troubleshooting

### "Invalid Anthropic API Key"
- Check API key in credentials
- Ensure billing is enabled on Anthropic account
- Test at https://console.anthropic.com/account/api-keys

### "Google Sheets authentication failed"
- Reauthenticate Google Sheets credential
- Ensure APIs enabled in Google Cloud Console
- Check scopes include Sheets and Drive

### "No jobs scraped"
- Verify Apify token is valid
- Check actor IDs match your account
- Test actors manually in Apify dashboard

### "Claude returning errors"
- Check BASE_RESUME_TEXT is set
- Ensure job descriptions > 800 characters
- Review Claude error message in execution

## Next Steps

1. Review `WORKFLOW_BREAKDOWN.md` for detailed node documentation
2. Customize job search keywords in LinkedIn/Indeed nodes
3. Configure email templates in Claude node
4. Set up daily schedule trigger
5. Monitor first run for accuracy

## Security Best Practices

- **Never commit credentials** to Git
- **Store API keys** in n8n credentials, not environment
- **Use dedicated email** for job applications
- **Rotate Apify tokens** quarterly
- **Monitor API usage** to prevent unexpected charges
- **Backup Google Sheets** regularly

## Support

For issues:
1. Check GitHub Issues: https://github.com/Akash5091/n8n-ai-job-hunting-agent/issues
2. Review Troubleshooting section in README
3. Check n8n documentation: https://docs.n8n.io
4. Check Anthropic docs: https://docs.anthropic.com
