# Ollama Service

Deploy Ollama on VM (Ubuntu 22.04) on Azure or EC2.

## Deployment Features

- Containerization (Docker)
- Reverse Proxy/ Load Balancer (Nginx, Traefik)
- Concurrency Handling (export `OLLAMA_NUM_PARALLEL`)
- Use job queue (Redis, RabbitMQ) to manage concurrent requests
- Monitor & Logging (Prometheus, Grafana)