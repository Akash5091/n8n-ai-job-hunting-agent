# Setup and Run Guide

This guide provides step-by-step instructions to set up and run the n8n AI Job Hunting Agent.

## Prerequisites

Before you begin, ensure you have the following installed on your system:

1. **Docker** (version 20.10 or higher)
   - Download from: https://www.docker.com/products/docker-desktop
   - Verify installation: `docker --version`

2. **Docker Compose** (version 1.29 or higher)
   - Usually comes with Docker Desktop
   - Verify installation: `docker-compose --version`

3. **Git**
   - Download from: https://git-scm.com/downloads
   - Verify installation: `git --version`

## Step 1: Clone the Repository

```bash
git clone https://github.com/Akash5091/n8n-ai-job-hunting-agent.git
cd n8n-ai-job-hunting-agent
```

## Step 2: Configure Environment Variables

1. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```

2. Edit the `.env` file with your actual credentials and API keys:
   ```bash
   nano .env  # or use your preferred text editor
   ```

3. Update at minimum these values:
   - `N8N_BASIC_AUTH_PASSWORD` - Change from default
   - `DB_POSTGRESDB_PASSWORD` - Change to a secure password
   - `OPENAI_API_KEY` - Your OpenAI API key
   - `ANTHROPIC_API_KEY` - Your Anthropic API key
   - `GOOGLE_SHEETS_API_KEY` - Your Google Sheets API key
   - `NOTIFICATION_EMAIL` - Your email address

## Step 3: Start the Services

1. Start Docker containers:
   ```bash
   docker-compose up -d
   ```

2. This will:
   - Pull the required Docker images
   - Create the n8n container
   - Create the PostgreSQL database container
   - Initialize the database
   - Start both services in the background

3. Wait for services to be ready (usually 30-60 seconds):
   ```bash
   docker-compose logs -f n8n
   ```

   Look for this message:
   ```
   n8n ready on http://0.0.0.0:5678
   ```

## Step 4: Access n8n UI

1. Open your web browser and navigate to:
   ```
   http://localhost:5678
   ```

2. Log in with:
   - Username: `admin` (or what you set in .env)
   - Password: The password from your .env file

## Step 5: Import the Workflow

1. In the n8n UI, click on "Import from File"

2. Select the `akashjobagent.json` file from the repository

3. Click "Import" to load the workflow

4. Review the workflow nodes and connections

## Step 6: Configure Credentials

1. For each external service node (Claude, Google Sheets, Gmail, etc.):
   - Click on the node
   - Click "Add credentials"
   - Enter your API keys and authentication details
   - Test the connection

2. Common credentials needed:
   - **Claude/Anthropic**: API key
   - **Google Services**: OAuth token or API key
   - **Gmail**: OAuth token
   - **Slack**: Webhook URL (if using Slack notifications)

## Step 7: Test the Workflow

1. Click the "Test" button to run the workflow with sample data

2. Monitor the execution in the Executions panel

3. Check the logs for any errors

4. Verify outputs in the Google Sheet and email notifications

## Step 8: Activate the Workflow

1. Click the "Activate" button to enable automatic execution

2. Set up trigger conditions:
   - Time-based (scheduled)
   - Webhook-based (on-demand)
   - Event-based (job posting received)

## Useful Commands

### View Container Logs
```bash
# All services
docker-compose logs -f

# Just n8n
docker-compose logs -f n8n

# Just PostgreSQL
docker-compose logs -f postgres
```

### Access PostgreSQL Database
```bash
docker-compose exec postgres psql -U n8n_user -d n8n
```

### Stop Services
```bash
docker-compose down
```

### Stop and Remove All Data
```bash
docker-compose down -v
```

### Restart Services
```bash
docker-compose restart
```

### Rebuild Containers
```bash
docker-compose up -d --build
```

## Troubleshooting

### Port 5678 Already in Use

If you get an error about port 5678 being in use:

1. Find the process using the port:
   ```bash
   # On Mac/Linux
   lsof -i :5678
   
   # On Windows
   netstat -ano | findstr :5678
   ```

2. Either kill that process or change the port in `docker-compose.yml`

### Database Connection Error

If n8n can't connect to PostgreSQL:

1. Check the database is running:
   ```bash
   docker-compose ps
   ```

2. Verify credentials in .env match docker-compose.yml

3. Check database logs:
   ```bash
   docker-compose logs postgres
   ```

### Workflow Execution Fails

1. Check node credentials are properly configured

2. Verify API keys are valid and have proper permissions

3. Review execution logs in n8n UI

4. Check container logs:
   ```bash
   docker-compose logs n8n
   ```

## Performance Optimization

1. **Increase Memory Allocation**:
   ```bash
   # In docker-compose.yml, add to n8n service:
   mem_limit: 4g
   ```

2. **Enable Execution Queue**:
   Add to .env:
   ```
   QUEUE_MODE_ACTIVE=true
   ```

3. **Optimize Database**:
   ```bash
   docker-compose exec postgres vacuumdb -U n8n_user -d n8n
   ```

## Next Steps

After successful setup:

1. Review the documentation in the `docs/` folder
2. Customize email templates in `docs/EMAIL_TEMPLATES.md`
3. Adjust ATS scoring thresholds in `docs/ATS_SCORING_LOGIC.md`
4. Monitor workflow executions and optimize as needed
5. Set up automated backups of the database

## Support

For issues or questions:

1. Check the documentation in `docs/`
2. Review n8n official documentation: https://docs.n8n.io/
3. Check GitHub issues: https://github.com/Akash5091/n8n-ai-job-hunting-agent/issues
4. Refer to QUICK_START_COMMANDS.md for common operations
