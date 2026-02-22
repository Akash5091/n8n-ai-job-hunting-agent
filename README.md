# n8n-ai-job-hunting-agent
Claude Anthropic Powered Full Stack n8n AI Job Hunting Agent - Automated job matching, resume tailoring, and recruiter outreach with ATS scoring optimization

## Overview

This is a production-ready n8n workflow automation system that leverages Claude AI (via Anthropic API) to automate the entire job search process:

- **Job Ingestion**: Aggregates listings from LinkedIn, Indeed, Glassdoor, Greenhouse, Lever, and Workday
- **Smart Filtering**: Deduplicates and filters for relevant software engineering roles
- **Resume Tailoring**: Uses Claude to dynamically adjust your resume against each job description
- **ATS Optimization**: Scores resumes 0-100 with automatic iteration until 90+ match
- **Recruiter Discovery**: Identifies and extracts recruiter contact information
- **Outreach Automation**: Generates personalized cold emails and applies to positions
- **Human-in-Loop**: Slack/Email notifications with approval gates before applying

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Prerequisites & Setup](#prerequisites--setup)
3. [Workflow Components](#workflow-components)
4. [Configuration](#configuration)
5. [Deployment](#deployment)
6. [Usage Guide](#usage-guide)

## Architecture Overview

The workflow follows a multi-stage pipeline:

```
TRIGGER
  ↓
JOB INGESTION (LinkedIn, Indeed, Glassdoor, Greenhouse, Lever, Workday)
  ↓
FILTERING & DEDUPLICATION
  ↓
CLAUDE AI AGENT CORE (Per-job processing)
  - Extract keywords (must-have, nice-to-have)
  - Match tech stack
  - Tailor resume
  - Score ATS (iterate 0-100)
  - Generate email
  - Suggest recruiter outreach
  ↓
RESUME GENERATION (PDF/DOCX to Google Drive)
  ↓
TRACKER UPDATE (Google Sheets)
  ↓
RECRUITER OUTREACH (Email via Gmail)
  ↓
HUMAN-IN-LOOP NOTIFICATION (Slack/Email)
  ↓
AUTO-APPLY (Optional)
```

## Prerequisites & Setup

### Required Accounts & APIs

1. **n8n** (self-hosted or cloud)
2. **Anthropic Claude API** - Get your API key at https://console.anthropic.com
3. **Google Account** - For Sheets, Drive, and Gmail integration
4. **Apify Account** - For job scraping (LinkedIn, Indeed, Workday)
5. **Slack Workspace** (optional) - For notifications
6. **GitHub** (optional) - To store workflow and documentation

### Environment Variables

Create these in your n8n instance Settings > Credentials:

```bash
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_SHEET_ID=your-sheet-id-here
GOOGLE_DRIVE_FOLDER_ID=your-folder-id-here
APIFY_TOKEN=apify_api_token
NOTIFICATION_EMAIL=your-email@example.com
MIN_ATS_SCORE=90
MAX_ATS_ITERATIONS=3
APPLY_MODE=HUMANINLOOP  # or AUTOAPPLY
```

### Google Sheet Setup

Create a new Google Sheet with two tabs:

**Tab 1: JobLeads**
- Date | Company | Role | Location | Job URL | Apply URL | Seniority | Tech Match | Top Keywords | ATS Score | Resume File | Status | Notes

**Tab 2: Applications**
- Date | Company | Role | Apply URL | Resume File | Status | Recruiter Name | Recruiter Email | Outreach Sent | Follow-up Date | Notes

### Google Drive Setup

Create a folder named "AkashJobResumes" (or configure name) and note its folder ID:
`https://drive.google.com/drive/folders/YOUR_FOLDER_ID`

## Workflow Components

See `WORKFLOW_BREAKDOWN.md` for detailed node-by-node documentation.

## Configuration

Key configurations in the Claude payload:

- **CANDIDATE_NAME**: Your name for email signatures
- **BASE_RESUME_TEXT**: Your current resume (stored in n8n environment)
- **MIN_ATS_SCORE**: Minimum ATS score threshold (default: 90)
- **APPLY_MODE**: `HUMANINLOOP` for notifications with manual approval, or `AUTOAPPLY` for automatic submission

## Deployment

### Option 1: Self-Hosted n8n

```bash
npm install -g n8n
n8n start
```

Then import the workflow JSON from the `workflows/` directory.

### Option 2: n8n Cloud

1. Sign up at https://n8n.cloud
2. Create a new workflow
3. Import the JSON workflow
4. Add all required credentials
5. Deploy

## Usage Guide

### Running Manually

1. Go to n8n Workflow Dashboard
2. Click "Test Workflow" or "Execute Workflow"
3. Monitor execution in the execution panel
4. Check Google Sheets for results
5. Review Slack notifications

### Scheduled Execution

The workflow is configured to run daily at 8:00 AM CST (Monday-Friday). Customize in the Schedule Trigger node.

### Monitoring

- Check **Google Sheets** for job matches and ATS scores
- Review **Slack channel** `#job-alerts` for daily summaries
- Monitor **Gmail** for outreach confirmations
- Check **Google Drive** for generated resume PDFs

## Resume Tailoring & ATS Scoring

### How It Works

1. Claude extracts keywords from the job description
2. Compares against your base resume
3. Calculates initial ATS score
4. Tailors resume to match job requirements
5. Re-scores and iterates up to 3 times
6. Flags jobs where score cannot reach 90 without fabrication

### Scoring Logic

- +2 per must-have keyword (cap: 60)
- +1 per nice-to-have keyword (cap: 20)
- +10 if job title appears in summary
- +10 if 2+ responsibilities reflected
- -20 if fabrication would be required (blocks completion)

## Email Template

Generated emails follow this pattern:

**Subject**: `[Your Name] - Full-Stack Java Engineer (Company Role) - ATS Match: X%`

**Body**:
```
Hi [Recruiter Name],

I came across the [Role] opening at [Company] and wanted to reach out.

I'm a Full-Stack Java Engineer with [X] years building microservices,
Spring Boot, and Kubernetes on AWS. I've tailored my resume specifically
for this role and believe I'm a strong match for your team's needs.

Would you have 15 minutes this week to discuss?

Best,
[Your Name]
[Phone]
[LinkedIn]
[GitHub]
```

## Troubleshooting

### Issue: "BASE_RESUME_TEXT not set"
**Solution**: Add your resume text to n8n environment variable: `BASE_RESUME_TEXT`

### Issue: Claude API errors
**Solution**: Check your Anthropic API key and ensure it has available credits

### Issue: Google Sheets not updating
**Solution**: Verify Google Sheets OAuth2 credential is valid and Sheet ID is correct

### Issue: Jobs not being scraped
**Solution**: Check Apify credentials and actor IDs for LinkedIn/Indeed/Workday

## Contributing

Contributions welcome! Areas for enhancement:

- Additional job board integrations (CareerBuilder, Stack Overflow Jobs, etc.)
- Multi-language support for international job hunting
- Portfolio link generation and injection
- Integration with GitHub repos for technical projects
- Advanced ML-based job matching beyond keyword analysis
- Salary negotiation templates

## License

MIT License - feel free to fork and customize for your own job hunt!

## Support & Questions

For issues, questions, or feedback:
- Open an issue on GitHub
- Check the `docs/` directory for detailed guides
- Review `WORKFLOW_BREAKDOWN.md` for node-by-node documentation

---

**Built for**: Akash Reddy Bommireddy  
**Framework**: n8n Workflow Automation  
**AI Model**: Claude Opus 4.6 (Anthropic)  
**Version**: 1.0  
**Last Updated**: February 2026
