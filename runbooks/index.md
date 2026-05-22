# Troubleshooting Runbooks

This section documents practical runbooks for infrastructure operations, production troubleshooting, incident response, deployment failures, server issues, monitoring alerts, and recovery procedures.

The goal is to demonstrate structured operational thinking, not just command knowledge.

---

## Runbooks

### 1. Production Incident Response Runbook

Focus areas:

- Confirm impact
- Identify affected users or services
- Check recent changes
- Review logs and metrics
- Isolate the failing layer
- Apply safest recovery option
- Validate service restoration
- Document root cause and prevention

---

### 2. Linux Server High CPU Troubleshooting

Focus areas:

- Load average
- Top CPU processes
- Application logs
- Cron jobs
- Background tasks
- Recent deployments
- Resource saturation
- Recovery options

---

### 3. Disk Full Troubleshooting

Focus areas:

- Filesystem usage
- Inode usage
- Large files
- Log growth
- Docker volume usage
- Cleanup strategy
- Prevention with log rotation

---

### 4. Docker Compose Application Down

Focus areas:

- Container status
- Container logs
- Port conflicts
- Environment variables
- Docker networks
- Volumes
- Restart policies
- Application health checks

---

### 5. PostgreSQL Backup and Restore Runbook

Focus areas:

- Manual backup
- Automated backup
- Restore validation
- Connection troubleshooting
- Disk space validation
- Recovery testing

---

### 6. Redis Troubleshooting Runbook

Focus areas:

- Redis availability
- Memory usage
- Persistence
- Connection errors
- Eviction policy
- Restart behavior
- Backup considerations

---

### 7. Deployment Failed but CI/CD Passed

Focus areas:

- Health checks
- Application logs
- Environment variables
- Database migrations
- Reverse proxy routing
- DNS
- SSL certificates
- Rollback decision

---

### 8. Rollback vs Hotfix Decision Runbook

Focus areas:

- Customer impact
- Risk assessment
- Recent changes
- Time to recovery
- Data impact
- Communication
- Validation after recovery

---

## Operational Principles

A strong infrastructure engineer should be able to:

1. Confirm the issue.
2. Measure impact.
3. Check recent changes.
4. Review logs, metrics, and alerts.
5. Isolate the failing layer.
6. Apply the safest fix.
7. Validate recovery.
8. Document root cause.
9. Add prevention steps.
10. Improve automation and monitoring.
