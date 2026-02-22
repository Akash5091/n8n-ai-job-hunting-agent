# Quick Start Commands

Get the n8n AI Job Hunting Agent up and running with these commands:

## Prerequisites
- Docker and Docker Compose installed
- Node.js 18+ (for local development)
- All required API credentials configured

## Start the Application

### Using Docker Compose (Recommended)
```bash
# Navigate to project directory
cd n8n-ai-job-hunting-agent

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f n8n

# Stop services
docker-compose down
```

### Local Development
```bash
# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Edit .env with your API keys

# Start development server
npm run dev
```

## Access the Application

- **n8n UI**: http://localhost:5678
- **Default Email**: admin@example.com
- **Default Password**: (set during first login)

## Key Commands

### Workflow Operations
```bash
# Import workflow
curl -X POST http://localhost:5678/api/v1/workflows/import \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d @akashjobagent.json

# Trigger workflow
curl -X POST http://localhost:5678/api/v1/workflows/{workflow_id}/activate \
  -H "Authorization: Bearer YOUR_API_KEY"

# Get workflow status
curl http://localhost:5678/api/v1/workflows/{workflow_id} \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Database Operations
```bash
# Access PostgreSQL
docker-compose exec db psql -U neon_user -d job_hunting_db

# View candidates table
SELECT * FROM candidates LIMIT 10;

# View email logs
SELECT * FROM email_logs ORDER BY created_at DESC LIMIT 10;
```

### Troubleshooting Commands
```bash
# Check container status
docker-compose ps

# View n8n logs
docker-compose logs n8n --tail=100

# Restart service
docker-compose restart n8n

# Clear cache and restart
docker-compose down -v
docker-compose up -d
```

## Environment Setup

```bash
# Create .env file with all required variables
echo 'OPENAI_API_KEY=your_key_here' >> .env
echo 'ANTHROPIC_API_KEY=your_key_here' >> .env
echo 'GOOGLE_SHEETS_API_KEY=your_key_here' >> .env
echo 'GOOGLE_DRIVE_FOLDER_ID=your_id_here' >> .env
echo 'NOTIFICATION_EMAIL=your_email@example.com' >> .env
echo 'SLACK_CHANNEL=your_channel_here' >> .env
echo 'APIFY_API_TOKEN=your_token_here' >> .env
```

## Monitoring

```bash
# Monitor workflow executions
docker-compose logs -f n8n | grep -i execution

# Monitor database connections
docker-compose exec db psql -U neon_user -d job_hunting_db \
  -c "SELECT count(*) FROM pg_stat_activity;"

# Check resource usage
docker stats
```

## First Run Checklist

1. ✓ Clone repository and navigate to directory
2. ✓ Copy and configure .env file with all API keys
3. ✓ Run `docker-compose up -d` to start services
4. ✓ Wait 30-60 seconds for services to fully initialize
5. ✓ Access http://localhost:5678 in your browser
6. ✓ Import akashjobagent.json workflow
7. ✓ Configure credentials in n8n UI
8. ✓ Activate workflow and test with sample job posting
9. ✓ Monitor logs during first execution
10. ✓ Set up notification channels (email, Slack)

## Performance Tuning

```bash
# Increase memory allocation
export MEMORY_LIMIT=4g
docker-compose up -d

# Enable debug logging
export LOG_LEVEL=debug
docker-compose restart n8n

# Optimize database
docker-compose exec db vacuumdb -U neon_user -d job_hunting_db
```

## Maintenance

```bash
# Backup database
docker-compose exec db pg_dump -U neon_user job_hunting_db > backup.sql

# Restore database
docker-compose exec db psql -U neon_user job_hunting_db < backup.sql

# Update containers
docker-compose pull
docker-compose up -d

# Clean up unused resources
docker system prune -a
```

## Common Issues

### "Connection refused" error
- Ensure all services are running: `docker-compose ps`
- Check if ports are already in use: `netstat -tuln | grep -E "(5678|5432)"`
- Restart services: `docker-compose restart`

### Memory issues
- Increase Docker memory allocation in Docker Desktop settings
- Reduce MAX_PARALLEL_JOBS in environment variables

### API key errors
- Verify all .env variables are set correctly
- Check API credentials are still valid
- Ensure no extra spaces or quotes in .env file

## Next Steps

After successful startup:
1. Review INSTALLATION_AND_SETUP.md for detailed configuration
2. Read WORKFLOW_BREAKDOWN.md to understand workflow architecture
3. Check DEPLOYMENT_CHECKLIST.md before running in production
4. Customize EMAIL_TEMPLATES.md with your branding
5. Adjust ATS_SCORING_LOGIC.md thresholds based on your requirements
