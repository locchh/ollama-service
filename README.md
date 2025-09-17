# Ollama Service

Our goal is to deploy Ollama on a virtual machine (Azure or EC2) to run inference tasks. Access will be restricted to authorized users, with support for concurrent requests. We will also enable monitoring and logging to quickly detect and resolve issues, while ensuring the system remains resilient and stable.

## Deployment Features

- Containerization (Docker)
- Reverse Proxy/ Load Balancer (Nginx, Traefik)
- Concurrency Handling (export `OLLAMA_NUM_PARALLEL`)
- Use job queue (Redis, RabbitMQ) to manage concurrent requests
- Monitor & Logging (Prometheus, Grafana)

## Solution

- [Solution 1](./docs/deploy_1.md)
- [Solution 2](./docs/deploy_2.md)

## References

- [Ollama](https://ollama.com/)
- [Docker](https://www.docker.com/)
- [Nginx](https://www.nginx.com/)
- [K3s](https://k3s.io/)
- [Redis](https://redis.io/)
- [Prometheus](https://prometheus.io/)
- [Grafana](https://grafana.com/)
- [cAdvisor](https://github.com/google/cadvisor)