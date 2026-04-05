# GitHub Actions Workflows

This directory contains GitHub Actions workflows for automated testing, deployment, and management of the monitoring stack.

## Available Workflows

### 1. Deploy Monitoring Stack (`deploy-monitoring-stack.yml`)

Production-ready deployment workflow for running the complete monitoring stack on self-hosted nodes.

#### Features

**Deployment Management:**
- 🚀 Deploy/restart/stop monitoring stack
- 🔄 Force recreate containers option
- 🌍 Multi-environment support (production, staging, development)
- 💾 Automatic backup before production deployments
- ✅ Health checks and verification
- 📊 Post-deployment status reporting

**Safety Features:**
- Pre-deployment prerequisite checks
- Environment file validation
- Automatic data volume backups (production)
- Graceful service shutdown
- Resource usage monitoring

#### Usage

**Manual Deployment:**
1. Go to Actions tab → "Deploy Monitoring Stack"
2. Click "Run workflow"
3. Select:
   - **Environment:** production/staging/development
   - **Action:** deploy/restart/stop/status
   - **Force recreate:** true/false
4. Click "Run workflow"

**Available Actions:**
- **deploy:** Pull images, stop existing stack, start fresh deployment
- **restart:** Restart the monitoring stack
- **stop:** Stop all services
- **status:** Check current deployment status

**Environments:**
- **production:** Full backups, strict validation
- **staging:** Testing environment
- **development:** Development environment

#### Requirements

- Self-hosted runner with Docker Compose V2
- `.env` file configured with credentials
- 20GB+ disk space
- For production: `/var/backups/hell-compose-monitor/` directory

#### Workflow Steps

1. **Verification:** Check Docker, Compose, disk space, .env
2. **Backup:** (Production only) Backup Prometheus and Grafana volumes
3. **Pull:** Download latest images
4. **Deploy:** Stop old stack, start new services
5. **Health Checks:** Wait for services to be healthy
6. **Verification:** Test endpoints, check logs
7. **Report:** Display access info and resource usage

#### Access After Deployment

The workflow displays access URLs for:
- **Prometheus:** Port 9090
- **Grafana:** Port 3000 (admin/admin)
- **Alertmanager:** Port 9093

---

### 2. Test Docker Compose Stack (`test-docker-compose.yml`)

Comprehensive testing workflow for the complete monitoring stack using Docker Compose on self-hosted runners.

#### Features

**Automated Testing:**
- ✅ Validates all service health checks
- ✅ Tests API endpoints for Prometheus, Grafana, Alertmanager, and Loki
- ✅ Verifies inter-service connectivity
- ✅ Checks for critical errors in logs
- ✅ Reports resource usage statistics

**Security Scanning:**
- 🔒 Scans Docker images for vulnerabilities (with Trivy)
- 🔒 References SBOM and security documentation

**Failure Handling:**
- 📋 Exports logs on failure
- 📋 Uploads artifacts for debugging
- 📋 Automatic cleanup of containers and volumes

#### Triggers

The workflow runs on:
- **Pull Requests** to `main`/`master` (when docker-compose.yml or service configs change)
- **Push** to `main`/`master` (when docker-compose.yml or service configs change)
- **Manual trigger** via workflow_dispatch (with optional cleanup control)

#### Requirements

**Self-Hosted Runner Requirements:**
- Docker Engine 20.10+ with Docker Compose V2
- Minimum 4GB RAM (8GB+ recommended)
- 20GB+ free disk space
- `jq` command-line tool for JSON parsing
- (Optional) Trivy for security scanning

**Setting up a self-hosted runner:**
1. Go to repository Settings → Actions → Runners
2. Click "New self-hosted runner"
3. Follow the setup instructions for your OS
4. Ensure Docker and required tools are installed
5. Start the runner service

#### Usage

**Automatic runs:**
The workflow runs automatically on PR/push events affecting monitoring stack files.

**Manual runs:**
1. Go to Actions tab in GitHub
2. Select "Test Docker Compose Stack"
3. Click "Run workflow"
4. Choose whether to cleanup containers after test (default: true)

#### Test Coverage

The workflow validates:

| Service | Health Check | API Test | Connectivity |
|---------|--------------|----------|--------------|
| Prometheus | ✅ | ✅ | ✅ |
| Alertmanager | ✅ | ✅ | ✅ |
| Grafana | ✅ | ✅ | ✅ |
| Loki | ✅ | ✅ | ✅ |
| Promtail | ✅ | - | ✅ |
| Pushgateway | ✅ | ✅ | - |
| syslog-ng | ✅ | - | - |
| PVE Exporter | ✅ | - | - |
| mktxp | ✅ | - | - |

#### Workflow Steps

1. **Setup Phase**
   - Checkout code
   - Verify Docker/Compose availability
   - Create test environment file

2. **Deployment Phase**
   - Pull all Docker images
   - Start all services
   - Wait for health checks (up to 10 minutes)

3. **Testing Phase**
   - Test Prometheus (health, targets, queries)
   - Test Alertmanager (status, alerts API)
   - Test Grafana (health, datasources)
   - Test Loki (ready, metrics)
   - Test Pushgateway (ready endpoint)
   - Verify inter-service connectivity

4. **Validation Phase**
   - Check logs for errors
   - Display service status
   - Report resource usage

5. **Cleanup Phase**
   - Stop containers (if enabled)
   - Remove volumes
   - Clean dangling images
   - Export logs on failure

#### Environment Variables

The workflow creates a test `.env` file with safe defaults:

```bash
TELEGRAM_POOPTOOTH_CHAT_ID=-1234567890
TELEGRAM_POOPTOOTH_TOKEN=1234567890:ABC...
PVE_USER=test@pve
PVE_PASSWORD=testpassword
PVE_VERIFY_SSL=false
EXTERNAL_URL=http://localhost:9090/
```

**Note:** These are dummy values for testing. Production environments should use GitHub Secrets.

#### Artifacts

On failure, the workflow uploads:
- `docker-compose.log` - Complete logs from all services
- `services-status.json` - Final service status
- `resolved-compose.yml` - Resolved docker-compose configuration

Artifacts are retained for 7 days.

#### Customization

**Timeout adjustments:**
Edit timeout values in the workflow file:
```yaml
timeout-minutes: 30  # Overall job timeout
```

**Health check wait time:**
Modify `max_attempts` in the "Wait for services to be healthy" step.

**Additional tests:**
Add custom test steps after the existing service tests.

#### Troubleshooting

**Services fail to start:**
- Check runner has sufficient resources
- Review exported logs in workflow artifacts
- Ensure required ports are available (9090, 9093, 3000, 3100, etc.)

**Health checks timeout:**
- Some services may need more time on first run (image pulls)
- Increase `max_attempts` or `sleep` duration
- Check service logs for startup errors

**Network connectivity issues:**
- Verify Docker network configuration
- Ensure services can reach each other
- Check firewall rules on runner

**Permission errors:**
- Runner user needs Docker permissions
- Add runner user to `docker` group: `sudo usermod -aG docker <runner-user>`

#### Security Considerations

**Runner Security:**
- Use dedicated runners for production validation
- Restrict runner access to trusted repositories
- Regularly update runner and Docker versions

**Secrets Management:**
- Never commit real credentials
- Use GitHub Secrets for sensitive values
- Rotate test credentials regularly

**Network Isolation:**
- Test stack uses isolated Docker networks
- Services are not exposed to public internet during tests
- Cleanup removes all containers and networks

#### Performance

**Typical run times:**
- Image pull: 2-5 minutes (cached: <1 minute)
- Service startup: 2-3 minutes
- Health checks: 1-2 minutes
- Tests: 1-2 minutes
- **Total: ~6-12 minutes**

**Resource usage (approximate):**
- CPU: 1-2 cores during startup
- RAM: 2-4 GB total
- Disk: 10-15 GB for images

#### Integration with CI/CD

This workflow can be integrated into your deployment pipeline:

```yaml
# Example: Run tests before deployment
deploy:
  needs: test-stack
  runs-on: self-hosted
  steps:
    - name: Deploy to production
      run: ./deploy.sh
```

#### Contributing

To improve the workflow:
1. Add new test cases for additional services
2. Enhance error detection patterns
3. Add performance benchmarks
4. Improve cleanup procedures

Submit PRs with workflow enhancements!

---

**Last Updated:** October 30, 2025  
**Workflow Version:** 1.0  
**Maintained By:** DevOps Team
