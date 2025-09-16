# Ollama Service

Our goal is to deploy Ollama on a virtual machine (Azure or EC2) to run inference tasks. Access will be restricted to authorized users, with support for concurrent requests. We will also enable monitoring and logging to quickly detect and resolve issues, while ensuring the system remains resilient and stable.

## Deployment Features

- Containerization (Docker)
- Reverse Proxy/ Load Balancer (Nginx, Traefik)
- Concurrency Handling (export `OLLAMA_NUM_PARALLEL`)
- Use job queue (Redis, RabbitMQ) to manage concurrent requests
- Monitor & Logging (Prometheus, Grafana)

## Guide

**Step1**: Create VM [Link](docs/setup_vm.md)

**Step2**: Connect to VM via SSH

```bash
# Change permission of your key
chmod 600 path/to/your-key.pem

# Connect to VM
ssh -i path/to/your-key.pem azureuser@<your-vm-public-ip>
```

or:

```bash
# Starts the SSH agent in the background
# The agent stores your decrypted private keys in memory
# Allows you to use your SSH keys without entering the passphrase each time
eval $(ssh-agent)

# Adds your private key to the SSH agent
# By default, adds ~/.ssh/id_rsa if no path is specified
ssh-add path/to/your-key.pem

# Now you can connect without specifying the key file
ssh azureuser@<your-vm-public-ip>
```

**Step3**: Update system

```bash
sudo apt update && sudo apt upgrade -y
```

**Step4**: Install Docker

```bash
# Install prerequisites
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add Docker repository
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add your user to docker group to run docker without sudo
sudo usermod -aG docker ${USER}

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installations
docker --version
docker-compose --version
```

**Step5**: Pull Ollama image

- Pull Ollama image from Docker Hub:

```bash
sudo docker pull ollama/ollama

# Verify image
sudo docker images
```

- (Addtional) We can run the container to test:

```bash
sudo docker run -d \
  -p 11434:11434 \
  -v ollama-volume:/root/.ollama \
  --restart unless-stopped \
  --name ollama \
  ollama/ollama

sudo docker exec -it ollama bash

ollama run deepseek-r1:1.5b
```

**Step6**: Set up Nginx as reverse proxy [Link](docs/setup_nginx.md)

**Step7**: Configure concurrency handling

```bash
# Stop the current container
sudo docker stop ollama

# Remove it
sudo docker rm ollama

# Start a new one with OLLAMA_NUM_PARALLEL
sudo docker run -d \
  -p 11434:11434 \
  -e OLLAMA_NUM_PARALLEL=2 \
  -v ollama-volume:/root/.ollama \
  --restart unless-stopped \
  --name ollama \
  ollama/ollama
```

**Step8**: Set up k3s [Link](docs/setup_k3s.md)

**Step9**: Set up Redis for job queue [Link](docs/setup_redis.md)

**Step10**: Scale horizontally

**Step11**: Set up monitoring and logging

## References

- [Ollama](https://ollama.com/)
- [Docker](https://www.docker.com/)
- [Nginx](https://www.nginx.com/)
- [K3s](https://k3s.io/)
- [Redis](https://redis.io/)
- [Prometheus](https://prometheus.io/)
- [Grafana](https://grafana.com/)