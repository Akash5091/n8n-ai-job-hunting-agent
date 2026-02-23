# Workflow Import & Configuration Guide

This guide walks you through importing the n8n AI Job Hunting Agent workflow and configuring all necessary credentials.

## Prerequisites

Before starting, ensure you have:

1. ✅ Docker containers running (`docker-compose up -d`)
2. ✅ n8n UI accessible at http://localhost:5678
3. ✅ Logged in with admin credentials
4. ✅ API keys ready for external services
5. ✅ `akashjobagent.json` file available

## Step 1: Import the Workflow

### Method 1: Import from File (Recommended)

1. **Access the Workflows Dashboard**
   - Navigate to http://localhost:5678
   - You should see the main n8n dashboard

2. **Click on "Workflows" in the left sidebar**
   - This shows all your workflows

3. **Click the "+" button or "Create New Workflow" button**
   - Or look for an "Import" option

4. **Select "Import from File"**
   - Click on the import button
   - Select `akashjobagent.json` from your repository

5. **Review and Import**
   - You'll see a preview of the workflow
   - Click "Import" to load it into n8n

6. **Verify Import Success**
   - The workflow should appear in your workflows list
   - You should see all nodes connected properly
   - The workflow name should be "n8n AI Job Hunting Agent"

### Method 2: Import via API

```bash
curl -X POST http://localhost:5678/api/v1/workflows/import \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d @akashjobagent.json
```

### Method 3: Copy-Paste JSON

1. Open `akashjobagent.json` in a text editor
2. Copy the entire JSON content
3. In n8n, go to Workflow → Edit as JSON
4. Paste the entire content
5. Save and exit

## Step 2: Configure Credentials

After importing, you need to configure credentials for each external service. Here's the complete list:

### 2.1 Claude/Anthropic Credentials

1. **In n8n, find the "Claude" node** (or "Anthropic" node)

2. **Click on the node to open its configuration**

3. **Click "Add Credentials" or the credential selector**

4. **Create New Credential:**
   - Name: `Claude API Credentials`
   - API Key: Your Anthropic API key
   - Model: `claude-3-opus-20240229` (or your preferred model)

5. **Test the Connection:**
   - Click "Test Connection"
   - Verify it returns success

**Where to get API Key:**
- Go to https://console.anthropic.com
- Create an account or log in
- Navigate to API Keys section
- Create a new API key
- Copy and save it securely

### 2.2 Google Sheets Credentials

1. **Find the Google Sheets node(s) in the workflow**
   - Look for nodes labeled "Google Sheets - Log Results" or similar

2. **Click on the node**

3. **Create Google Credentials:**
   - Click the credential dropdown
   - Select "Create New" → "Google Sheets"
   - This will open OAuth flow
   - Choose your Google account
   - Grant permissions to access Google Sheets and Drive
   - Accept the authorization

4. **Configure Sheet Details:**
   - In the node settings, specify:
     - Spreadsheet ID (from your Google Sheet URL)
     - Sheet name or tab name
     - Range (e.g., `A1:Z1000`)

**Where to get Spreadsheet ID:**
- Open your Google Sheet
- Copy the ID from the URL: `https://docs.google.com/spreadsheets/d/{SPREADSHEET_ID}/edit`

**Create a new Google Sheet:**
1. Go to https://sheets.google.com
2. Click "Create New Spreadsheet"
3. Name it "Job Hunting Results"
4. Create columns: Job Title, Company, Match Score, Email Sent, Date
5. Copy the Spreadsheet ID

### 2.3 Gmail Credentials

1. **Find the Gmail node(s)**
   - Look for nodes like "Gmail - Send Email"

2. **Configure Gmail:**
   - Click the node
   - Click credential dropdown
   - Select "Create New" → "Gmail"
   - Choose your Gmail account
   - Grant permission to send emails

3. **Configure Email Settings:**
   - From Email: Your Gmail address
   - Signature: (optional)

**Requirements:**
- Gmail account with 2FA enabled (recommended)
- Create an "App Password" for n8n
  1. Go to https://myaccount.google.com/apppasswords
  2. Select "Mail" and "Windows Computer"
  3. Copy the 16-character password
  4. Use this in the Gmail credential configuration

### 2.4 Slack Credentials (Optional)

If you want to receive notifications in Slack:

1. **Find the Slack node** (if it exists in your workflow)

2. **Create Slack Webhook:**
   - Go to https://api.slack.com/apps
   - Create a new app
   - Enable "Incoming Webhooks"
   - Create a webhook URL
   - Copy the URL

3. **In n8n:**
   - Click the Slack node
   - Paste the webhook URL
   - Test the connection

### 2.5 Additional API Configurations

For other nodes that require API keys directly (without OAuth):

1. **Find the node requiring the API key**

2. **Enter the API key in the node settings**

3. **Common services:**
   - OpenAI: Paste your API key directly
   - APIFY: Paste your API token
   - Custom HTTP requests: Add authorization headers as needed

## Step 3: Test Individual Nodes

Before running the full workflow, test each node:

1. **Click on a node**

2. **Click "Test Node" or "Execute Node"**
   - This will test just that node
   - Check the output in the right panel

3. **Verify Success:**
   - Node should show "✓" checkmark
   - Output data should appear without errors

4. **Fix Issues:**
   - If a node fails, check:
     - Credentials are correct and valid
     - API limits not exceeded
     - Required fields are filled
     - Network connectivity

## Step 4: Run a Test Workflow

### Test with Sample Data

1. **Click "Test Workflow" button**
   - This runs the entire workflow once
   - Uses sample/dummy data

2. **Provide Test Input:**
   - If the workflow starts with a manual trigger, click "Execute Workflow"
   - If it expects webhook data, send sample JSON

3. **Monitor Execution:**
   - Watch the nodes execute in sequence
   - Each node shows:
     - Execution time
     - Input data
     - Output data
     - Any errors

4. **Verify Outputs:**
   - Check Google Sheets for entries
   - Verify emails were sent to your notification email
   - Check Slack if configured
   - Review any error messages

### Example Test Data (Webhook Input)

If your workflow accepts webhook data, send this:

```json
{
  "job_posting": {
    "title": "Senior Full Stack Engineer",
    "company": "Tech Corp",
    "description": "Looking for experienced full-stack developer with Java, React, and AWS knowledge. 5+ years required.",
    "requirements": "Java, Spring Boot, React, AWS, Docker, PostgreSQL",
    "location": "Remote",
    "url": "https://example.com/job/123"
  },
  "candidate_resume": "5 years full-stack development experience. Proficient in Java, Spring Boot, React, TypeScript, Docker, AWS, and PostgreSQL. Led team of 3 developers. Passionate about clean code and best practices."
}
```

## Step 5: Configure Triggers

Set up how the workflow will be triggered in production:

### Option 1: Scheduled Trigger (Recommended)

1. **Find the "Trigger" node** (usually first node)

2. **Set trigger type to "Schedule"**

3. **Configure schedule:**
   - Run every: Day / Hour / Week
   - Specific time: 09:00 AM
   - Timezone: Your timezone

4. **Save the configuration**

### Option 2: Webhook Trigger

1. **Set trigger type to "Webhook"**

2. **Configure webhook:**
   - Method: POST
   - URL: Will be provided by n8n
   - Auth: (optional) Add authentication

3. **Copy the webhook URL**
   - Use this to trigger the workflow externally
   - Example: `curl -X POST http://localhost:5678/webhook/job-hunting-agent -d '{...}'`

### Option 3: Manual Trigger

- Click "Execute Workflow" button manually
- Useful for testing before scheduling

## Step 6: Activate the Workflow

Once testing is complete and everything works:

1. **Click the "Activate" button**
   - Usually in the top right
   - Workflow will turn from "Draft" to "Active"

2. **Verify Activation:**
   - The button should change to "Deactivate"
   - A green status indicator should appear
   - You should see "Active" status

3. **Monitor Execution:**
   - Go to "Executions" tab
   - Watch executions run according to your trigger schedule
   - Review logs and outputs

## Monitoring & Troubleshooting

### Check Workflow Executions

1. **Click on the workflow**

2. **Go to "Executions" tab**
   - Shows all past and current runs
   - Shows success/error status
   - Shows execution time and details

3. **Click on an execution**
   - See detailed node-by-node execution
   - View input/output data
   - Identify which node failed (if any)

### Common Issues & Solutions

**Issue: "Credentials not configured" error**
- Solution: Go back to step 2, ensure all credentials are properly added
- Check that credentials haven't expired
- Regenerate API keys if needed

**Issue: "Rate limit exceeded" error**
- Solution: The API hit its rate limit
- Wait before running again
- Implement delays between API calls in workflow

**Issue: "Node timeout" error**
- Solution: Execution took too long
- Check API response times
- Increase timeout in node settings

**Issue: Google Sheets not updating**
- Solution: Check spreadsheet ID is correct
- Verify Google credential has Sheets permission
- Check that sheet name matches exactly

**Issue: Emails not sending**
- Solution: Verify Gmail credential is correct
- Check that 2FA/App Password is used
- Verify recipient email is valid
- Check n8n logs: `docker-compose logs n8n`

## Production Checklist

Before going to production, verify:

- [ ] All credentials are configured and tested
- [ ] Test workflow execution completed successfully
- [ ] Email notifications working correctly
- [ ] Google Sheets updating with results
- [ ] Error notifications configured (if needed)
- [ ] Database backups enabled
- [ ] Monitoring and alerting set up
- [ ] Workflow set to "Active" status
- [ ] Appropriate trigger configured (schedule/webhook)
- [ ] Documentation shared with team
- [ ] API rate limits understood and configured

## Performance Optimization

For better performance:

1. **Add delays between API calls:**
   - Use "Wait" node between API calls
   - Prevents rate limiting
   - Adds 1-2 second delays

2. **Batch operations:**
   - Group multiple jobs before sending emails
   - Reduces API calls

3. **Implement caching:**
   - Cache job listings to avoid duplicate processing
   - Store successful matches to prevent resending

4. **Monitor resource usage:**
   - Watch Docker memory usage
   - Monitor database size
   - Check API usage quotas

## Next Steps

After successful configuration:

1. Monitor the first week of executions
2. Fine-tune ATS scoring thresholds if needed
3. Customize email templates
4. Add additional data sources (Indeed, LinkedIn, etc.)
5. Implement additional filters based on preferences
6. Set up backup and recovery procedures
7. Document any customizations made
