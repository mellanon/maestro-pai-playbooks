# Post-Implementation Checklist

## Docker Compose Stack (F-15)

### Persistent Storage Validation

Before declaring F-15 complete, validate persistent storage works correctly:

```bash
# 1. Verify named volumes are created
docker volume ls | grep observability
# Expected: 4 volumes (vm-data, vmlogs-data, vmtraces-data, grafana-data)

# 2. Write test data to VictoriaMetrics
curl -d 'test_metric{label="test"} 123' http://localhost:8428/api/v1/import/prometheus

# 3. Query to confirm data written
curl 'http://localhost:8428/api/v1/query?query=test_metric'

# 4. Stop and remove containers (but NOT volumes)
cd ~/observability && docker compose down

# 5. Verify volumes still exist
docker volume ls | grep observability
# Expected: All 4 volumes still present

# 6. Restart stack
docker compose up -d

# 7. Query to confirm data persisted
curl 'http://localhost:8428/api/v1/query?query=test_metric'
# Expected: Same data as step 3
```

### Volume Backup Consideration

For long-term data protection:

```bash
# Export volumes for backup
docker run --rm -v observability-vm-data:/data -v $(pwd):/backup alpine tar cvf /backup/vm-data-backup.tar /data

# Restore from backup
docker run --rm -v observability-vm-data:/data -v $(pwd):/backup alpine tar xvf /backup/vm-data-backup.tar -C /
```

### OrbStack-Specific Notes

- OrbStack stores volumes in `~/.orbstack/data/volumes/`
- Volume data survives OrbStack restarts
- Use `orb` CLI for OrbStack-specific management if needed

---

## Vector Collector (F-11, F-12, F-13)

### Vector Service Health

```bash
# Check Vector is tailing files
curl http://localhost:8686/health

# Check Vector internal metrics
curl http://localhost:8686/api/v1/metrics

# Verify log ingestion to VictoriaLogs
curl 'http://localhost:9428/select/logsql/query?query=*'
```

### Watchdog Validation (F-13)

```bash
# Simulate Vector failure
docker stop vector

# Wait for watchdog alert (check configured notification channel)

# Restart and verify recovery
docker start vector
```

---

## General Post-Implementation Items

- [ ] All services respond to health checks
- [ ] Grafana datasources show "Working" status
- [ ] Data persists across `docker compose down/up` cycle
- [ ] RAM usage < 600MB total (`docker stats --no-stream`)
- [ ] Ports are accessible only on localhost (security)
- [ ] .env file created from .env.example with real password

---

Created: 2026-01-24
Purpose: Manual validation checklist for observability stack features
