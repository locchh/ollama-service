**Criteria**:
- Resilient, auto-start at boot (Against OOM kill, etc.)
- Data persistence
- Backup
- Monitoring (Health check, Logging, etc.)

**Note**:
- to check cpu core, use `lscpu` command or `nproc` command
- to check memory, use `free -h` command

```bash
docker run -d \
  --name ollama \
  -p 11434:11434 \
  -e OLLAMA_NUM_PARALLEL=2 \
  -v ollama-volume:/root/.ollama \
  --restart unless-stopped \
  --memory=8g \
  --memory-swap=8g \
  --cpus=4 \
  --oom-kill-disable \
  --ulimit nofile=65535:65535 \
  --pids-limit=512 \
  --health-cmd "ollama list >/dev/null 2>&1 || exit 1" \
  --health-interval=30s \
  --health-timeout=5s \
  --health-retries=3 \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  ollama/ollama
```

Backup
-------
- One-off backup of the data volume (creates a tarball in current directory):
```bash
docker run --rm \
  -v ollama-volume:/data:ro \
  -v "$PWD":/backup \
  busybox sh -c 'tar czf /backup/ollama-backup-$(date +%F).tgz -C /data .'
```

- Restore from a backup tarball:
```bash
# Stop the container first if running
docker stop ollama 2>/dev/null || true
    
# Restore into the volume
docker run --rm \
  -v ollama-volume:/data \
  -v "$PWD":/backup \
  busybox sh -c 'tar xzf /backup/ollama-backup-YYYY-MM-DD.tgz -C /data'
```

Monitoring
----------
- Check container health status:
```bash
docker inspect --format='{{json .State.Health}}' ollama | jq -r .Status
```

- View logs:
```bash
docker logs --tail=200 -f ollama
```

- Resource usage:
```bash
docker stats ollama
```

- API health from host:
```bash
curl -sS http://localhost:11434/api/tags >/dev/null && echo OK || echo FAIL
```

Optional: cAdvisor for Container Monitoring
-------------------------------------------
- Run cAdvisor to get per-container CPU, memory, I/O metrics (web UI on port 8080):
```bash
docker run -d \
  --name=cadvisor \
  --restart unless-stopped \
  -p 8080:8080 \
  -v /:/rootfs:ro \
  -v /var/run:/var/run:ro \
  -v /sys:/sys:ro \
  -v /var/lib/docker/:/var/lib/docker:ro \
  -v /dev/disk/:/dev/disk:ro \
  gcr.io/cadvisor/cadvisor:latest
```

- Access UI: http://localhost:8080
- Prometheus: cAdvisor exposes metrics at `/metrics`. Example scrape config:
```yaml
scrape_configs:
  - job_name: cadvisor
    static_configs:
      - targets: ['localhost:8080']
```
- Grafana: import a Docker/cAdvisor dashboard and point it to your Prometheus data source.