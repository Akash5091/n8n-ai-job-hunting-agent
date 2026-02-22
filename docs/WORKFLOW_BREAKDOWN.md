# n8n Workflow Node-by-Node Breakdown

## Overview

This document provides detailed documentation for every node in the n8n AI Job Hunting Agent workflow. Use this as a reference when building, debugging, or extending the workflow.

## Phase 1: Triggers

### Node 1: Schedule Trigger

**Type:** Schedule Trigger  
**Purpose:** Automatically trigger the workflow on a schedule

**Configuration:**
- Rule: Interval
- Frequency: 0 8 * * 1-5 (Mon-Fri, 8:00 AM CST)
- Timezone: America/Chicago

**Output:** Triggers workflow automatically

### Node 2: Manual Trigger

**Type:** Manual Trigger  
**Purpose:** Allow on-demand testing and ad-hoc processing

**Configuration:** None required

**Output:** Allows workflow execution via "Test Workflow" button

### Node 3: Merge Triggers

**Type:** Merge Node  
**Purpose:** Combine outputs from both triggers into single workflow

**Configuration:**
- Mode: Merge By Position

**Output:** Single data stream for job ingestion layer

## Phase 2: Job Ingestion

### Node 4: LinkedIn Jobs via Apify

**Type:** HTTP Request  
**Method:** POST  
**URL:** `https://api.apify.com/v2/acts/apify/linkedin-jobs-scraper/run-sync-get-dataset-items`

**Headers:**
- Authorization: `Bearer {env.APIFYTOKEN}`

**Body (JSON):**
```json
{
  "queries": "Software Engineer Java, Full-Stack Java Developer, Backend Engineer Spring Boot, Microservices Engineer",
  "location": "United States",
  "datePosted": "past-24h",
  "maxResults": 25
}
```

**Output:** Array of LinkedIn job listings

### Node 5: Indeed RSS Feed

**Type:** RSS Feed Read  
**URL:** `https://www.indeed.com/rss?q=Java+Software+Engineer&l=United+States&fromage=1&sortby=date`

**Output:** RSS items with Indeed job listings

### Node 6: Glassdoor RSS Feed

**Type:** RSS Feed Read  
**URL:** `https://www.glassdoor.com/Job/jobs.htm?typedKeyword=java+developer&fromAge=1&formatrss`

**Output:** RSS items with Glassdoor job listings

### Node 7: Greenhouse API (Company-Specific)

**Type:** HTTP Request  
**Method:** GET  
**URL:** `https://boards-api.greenhouse.io/v1/boards/{companyslug}/jobs?content=true`

**Notes:** Loop over target companies using Greenhouse (Stripe, Airbnb, Shopify, Hashicorp, etc.)

**Output:** Array of Greenhouse job postings

### Node 8: Lever API (Company-Specific)

**Type:** HTTP Request  
**Method:** GET  
**URL:** `https://api.lever.co/v0/postings/{companyslug}?mode=json&state=published`

**Output:** Array of Lever job postings

### Node 9: Workday via Apify

**Type:** HTTP Request  
**Method:** POST  
**URL:** `https://api.apify.com/v2/acts/apify/workday-jobs-scraper/run`

**Body:**
```json
{
  "startUrls": ["https://company.wd5.myworkdayjobs.com/en-US/External"],
  "maxItems": 20
}
```

**Output:** Array of Workday job listings

### Node 10: Merge All Job Sources

**Type:** Merge Node  
**Purpose:** Combine all job sources into single array

**Configuration:**
- Mode: Append
- Merge all above sources

**Output:** Combined array of all jobs from all sources

## Phase 3: Filtering & Deduplication

### Node 11: Filter & Deduplicate

**Type:** Code Node  
**Language:** JavaScript

**Logic:**
1. Define software role keywords
2. Read existing job URLs from Google Sheets
3. Filter out non-software roles
4. Skip jobs already processed
5. Validate job description length (min 800 chars)

**Output:** Filtered array of new software job listings

### Node 12: Read Existing Jobs

**Type:** Google Sheets  
**Operation:** Read Rows  
**Sheet:** JobLeads  
**Columns:** joburl

**Purpose:** Get list of already-processed jobs to avoid duplicates

**Output:** Array of previously-seen job URLs

## Phase 4: Claude AI Agent Core

### Node 13: Split Into Batches

**Type:** SplitInBatches  
**Batch Size:** 1  
**Purpose:** Process each job individually

**Output:** Individual job objects for Claude processing

### Node 14: Prepare Claude Payload

**Type:** Code Node  
**Purpose:** Format job data for Claude API call

**Creates JSON payload with:**
- Model: claude-opus-4-6
- Max tokens: 8000
- System prompt: Resume tailoring specialist instructions
- User message: Job details + candidate profile + resume text

**Output:** Formatted API request payload

### Node 15: Call Claude API

**Type:** HTTP Request  
**Method:** POST  
**URL:** `https://api.anthropic.com/v1/messages`

**Headers:**
- x-api-key: Anthropic API key
- anthropic-version: 2023-06-01
- Content-Type: application/json

**Body:** Claude payload from Node 14

**Output:** Claude's JSON response with resume tailoring, ATS score, email draft

### Node 16: Parse Claude Response

**Type:** Code Node  
**Purpose:** Extract and validate Claude's JSON response

**Logic:**
1. Extract text from Claude response
2. Remove markdown code fences if present
3. Parse JSON
4. Attach original job data
5. Handle errors gracefully

**Output:** Parsed JSON with tailored resume, scores, and recommendations

## Phase 5: Resume File Generation

### Node 17: Check ATS Score Gate

**Type:** IF Node  
**Condition:** `json.finalatsscore >= 90`

**True Path:** Proceed to save resume files  
**False Path:** Log as ATSBELOWTARGET

**Output:** Conditional routing based on ATS score

### Node 18: Generate Resume PDF

**Type:** Code Node  
**Purpose:** Convert tailored resume text to PDF format

**Logic:**
1. Create filename: `Akash_{role}_{company}.pdf`
2. Format resume as styled HTML
3. Apply ATS-friendly formatting (Calibri, 11pt)
4. Return HTML for PDF conversion

**Output:** HTML resume content ready for PDF generation

### Node 19: Save PDF to Google Drive

**Type:** Google Drive  
**Operation:** Upload File  
**Parent Folder:** env.GOOGLEDRIVEFOLDERID (AkashJobResumes)

**File Details:**
- Name: From Node 18
- MIME Type: application/pdf

**Output:** File ID of uploaded resume

### Node 20: Get Drive File URL

**Type:** Google Drive  
**Operation:** Get File  
**File ID:** From Node 19

**Returns:** webViewLink (shareable URL)

**Output:** Public URL to resume PDF

## Phase 6: Tracker Update

### Node 21: Append to JobLeads Sheet

**Type:** Google Sheets  
**Operation:** Append Row  
**Sheet:** JobLeads

**Columns Updated:**
- Date: Today's date
- Company: From Claude output
- Role: From Claude output
- Location: From Claude output
- Job URL: Original job link
- Apply URL: Direct application link
- Seniority: Junior/Mid/Senior/Staff
- Tech Match: % of must-have keywords found
- Top Keywords: Top 5 must-have keywords
- ATS Score: 0-100 score
- Resume File: Drive link from Node 20
- Status: READYTOAPPLY if score >= 90, else ATSBELOWTARGET
- Notes: Missing keywords or flags

### Node 22: Append to Applications Sheet

**Type:** Google Sheets  
**Operation:** Append Row  
**Sheet:** Applications

**Columns Updated:**
- Date: Today's date
- Company: From Claude output
- Role: From Claude output
- Apply URL: Application link
- Resume File: Drive link
- Status: PENDING
- Recruiter Name: From Claude output or null
- Recruiter Email: From Claude output or null
- Outreach Sent: QUEUED if email found, else NOEMAILFOUND
- Follow-up Date: 7 days from now
- Notes: ATS score and match percentage

## Phase 7: Recruiter Outreach

### Node 23: Check If Recruiter Email Found

**Type:** IF Node  
**Condition:** `json.recruiterdiscovery.recruiteremail != null`

**True Path:** Send cold email  
**False Path:** Log NOEMAILFOUND

**Output:** Conditional routing for email sending

### Node 24: Send Cold Email via Gmail

**Type:** Gmail  
**Operation:** Send Email

**To:** From Claude output (recruiteremail)

**Subject:** From Claude output (tailored subject line)

**Body:** From Claude output (personalized outreach email)

**Attachments:** Resume PDF URL from Node 20

**Output:** Confirmation of email sent

## Phase 8: Human-in-Loop Notifications

### Node 25: Apply Mode Gate

**Type:** IF Node  
**Condition:** `env.APPLYMODE == "HUMANINLOOP"`

**True Path:** Send Slack/Email notification  
**False Path:** Proceed to auto-apply

**Output:** Conditional routing based on apply mode

### Node 26: Slack Notification

**Type:** Slack  
**Operation:** Send Message  
**Channel:** env.SLACKCHANNEL (#job-alerts)

**Message:**
```
New Job Ready to Apply!
✅ Company: {company}
💼 Role: {role}
📍 Location: {location}
🎯 ATS Score: {score}%
📊 Tech Match: {match}%
🎖️ Seniority: {seniority}
📄 Resume: [Drive Link]
✨ Apply Here: [Application Link]

Checklist:
- Review tailored resume
- Verify all facts
- Click apply link
- Upload resume file
- Submit application
```

### Node 27: Email Notification

**Type:** Gmail  
**Operation:** Send Email

**To:** env.NOTIFICATIONEMAIL  
**Subject:** `Job Alert: {role} at {company}`

**Body:** HTML version of Slack message

**Output:** Notification sent to you

## Phase 9: Run Summary

### Node 28: AUTOAPPLY Conditional

**Type:** HTTP Request (if AUTOAPPLY mode)

**Purpose:** Auto-submit application if configured

**Note:** This varies by ATS platform (Greenhouse, Lever, Workday)

### Node 29: Aggregate Results

**Type:** Code Node  
**Purpose:** Summarize workflow execution

**Calculates:**
- Total jobs processed
- Jobs kept (not filtered)
- Ready to apply (ATS >= 90)
- ATS below target (ATS < 90)
- Recruiter emails sent
- Parse errors

**Output:** Summary JSON object

### Node 30: Final Slack Summary

**Type:** Slack  
**Operation:** Send Message  
**Channel:** env.SLACKCHANNEL

**Message:**
```
📊 Daily Job Hunt Summary - {date}

📥 Jobs Processed: {total}
✅ Ready to Apply: {ready}
⚠️ ATS Below Target: {below}
📧 Emails Sent: {outreach}
❌ Errors: {errors}
```

## Environment Variables Required

```bash
GOOGLE_SHEET_ID=your-sheet-id
GOOGLE_DRIVE_FOLDER_ID=your-folder-id
CANDIDATE_NAME=Akash Reddy Bommireddy
NOTIFICATION_EMAIL=your-email@example.com
SLACK_CHANNEL=#job-alerts
MIN_ATS_SCORE=90
MAX_ATS_ITERATIONS=3
APPLY_MODE=HUMANINLOOP
BASE_RESUME_TEXT=<full-resume-here>
```

## Credentials Required

- Anthropic Claude API
- Google Sheets OAuth2
- Google Drive OAuth2
- Gmail OAuth2
- Slack OAuth2
- Apify API Token

## Error Handling

All error paths log to Google Sheets "Errors" tab with:
- Timestamp
- Node name
- Error message
- Job details

Daily error digest sent to notification email.
