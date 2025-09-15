# Setup Nginx as Reverse Proxy for Ollama

This guide explains how to set up Nginx as a reverse proxy for your Ollama service, including basic authentication to restrict access to authorized users only. Nginx support access control via Basic Auth, API Keys, JWT, OAuth 2.0, etc.

## Install Nginx

```bash
# Install Nginx
sudo apt update
sudo apt install -y nginx

# Verify installation
nginx -v
```

## Basic Authentication for Nginx

```bash
# Install apache2-utils for htpasswd utility
sudo apt install -y apache2-utils

# Create a password file and add a user
# Replace 'username' with your desired username
# You will be prompted to enter a password
sudo mkdir -p /etc/nginx/auth
sudo htpasswd -c /etc/nginx/auth/.htpasswd <username>

# To add additional users (without -c flag which would overwrite the file)
# sudo htpasswd /etc/nginx/auth/.htpasswd another_user
```

## Configure Nginx with Basic Authentication


```

## Basic Authentication for Nginx

```bash
# Install apache2-utils for htpasswd utility
sudo apt install -y apache2-utils

# Create a password file and add a user
# Replace 'username' with your desired username
# You will be prompted to enter a password
sudo mkdir -p /etc/nginx/auth
sudo htpasswd -c /etc/nginx/auth/.htpasswd <username>

# To add additional users (without -c flag which would overwrite the file)
# sudo htpasswd /etc/nginx/auth/.htpasswd another_user
```

## Configure Nginx with Basic Authentication

```bash
# Create Nginx configuration for Ollama with basic authentication
sudo nano /etc/nginx/sites-available/ollama.conf
```

Copy and paste the following content into the editor:

```
server {
    listen 80;
    server_name _;

    # Access logging
    access_log /var/log/nginx/ollama_access.log;
    error_log /var/log/nginx/ollama_error.log;

    # Basic authentication
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/auth/.htpasswd;

    location / {
        # Proxy to Ollama
        proxy_pass http://localhost:11434;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Websocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeout settings for long-running inference
        proxy_connect_timeout 300s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;
    }
}
```

Save the file by pressing `Ctrl+O` + `Ctrl+X`, then `Y`, then `Enter`.

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/ollama.conf /etc/nginx/sites-enabled/

# Remove default site (optional)
sudo rm -f /etc/nginx/sites-enabled/default

# Test Nginx configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

## Testing the Basic Authentication

```bash
# Test with curl using basic auth inside the VM
curl -u <username>:<password> http://localhost/api/tags

# Test without auth (should prompt for credentials)
curl -i http://localhost/api/tags

# Test with curl using basic auth outside the VM
curl -u <username>:<password> http://<your-vm-public-ip>/api/tags

# Test inference
curl -u <username>:<password> http://<your-vm-public-ip>/api/generate -d '{
  "model": "deepseek-r1:1.5b",
  "prompt":"Why is the sky blue?"
}'
```