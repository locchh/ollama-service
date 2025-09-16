# Set up k3s

## Install k3s

```bash
# Install K3s (one command)
curl -sfL https://get.k3s.io | sh -

# Verify installation
sudo kubectl get nodes
sudo kubectl get pods -A

# Check K3s service status
sudo systemctl status k3s

# Verify kubectl works
sudo kubectl version
```

## Post-Installation Checks:

```bash
# Check if K3s is running properly
sudo kubectl cluster-info

# Check system resources
sudo kubectl top nodes

# Check all system pods
sudo kubectl get pods -A -o wide
```

## Deploy Ollama on K3s with Health Checks

Stop the current container

```bash
sudo docker stop ollama
sudo docker rm ollama
```

### 1. Create Ollama Deployment Configuration

Create a file named `ollama-deployment.yaml` by `nano ollama-deployment.yaml` with the following content:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ollama
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ollama
  template:
    metadata:
      labels:
        app: ollama
    spec:
      containers:
        - name: ollama
          image: ollama/ollama
          ports:
            - containerPort: 11434
          env:
            - name: OLLAMA_NUM_PARALLEL
              value: "2"
          resources:
            requests:
              memory: "4Gi"
              cpu: "2"
            limits:
              memory: "8Gi"
              cpu: "4"
          livenessProbe:
            httpGet:
              path: /api/tags
              port: 11434
            initialDelaySeconds: 30
            periodSeconds: 10
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /api/tags
              port: 11434
            initialDelaySeconds: 5
            periodSeconds: 5
          volumeMounts:
            - name: ollama-volume
              mountPath: /root/.ollama
      volumes:
        - name: ollama-volume
          persistentVolumeClaim:
            claimName: ollama-pvc
```

Install `yamllint` by `sudo apt install yamllint` and validate the configuration file by `yamllint ollama-deployment.yaml` by `yamllint ollama-deployment.yaml`

### 2. Create Persistent Volume Claim

Create a file named `ollama-pvc.yaml` by `nano ollama-pvc.yaml` with the following content:

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ollama-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### 3. Create Service Configuration

Create a file named `ollama-service.yaml` by `nano ollama-service.yaml` with the following content:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: ollama-service
spec:
  selector:
    app: ollama
  ports:
    - port: 80
      targetPort: 11434
  type: NodePort
```

### 4. Deploy Everything

```bash
sudo kubectl apply -f ollama-pvc.yaml
sudo kubectl apply -f ollama-deployment.yaml
sudo kubectl apply -f ollama-service.yaml
```

### 5. Verify Deployment

```bash
# Check if the PVC was created
sudo kubectl get pvc

# Check if the Ollama pod is running
sudo kubectl get pods -l app=ollama

# Check if the service is created and get the NodePort
sudo kubectl get svc ollama-service

# Get more details about the pod
sudo kubectl describe pod -l app=ollama

# Check logs to see if Ollama started correctly
sudo kubectl logs -l app=ollama
```

### 6. Test Ollama

- Test in the VM:

```bash
# Get the NodePort
NODE_PORT=$(sudo kubectl get svc ollama-service -o jsonpath='{.spec.ports[0].nodePort}')
echo "NodePort: $NODE_PORT"

# Test the API
curl http://localhost:$NODE_PORT/api/tags

# Pull a model
curl -X POST http://localhost:$NODE_PORT/api/pull -d '{"name": "deepseek-r1:1.5b"}'

# Test the model
curl -X POST http://localhost:$NODE_PORT/api/chat -d '{"model": "deepseek-r1:1.5b", "messages": [{"role": "user", "content": "Hello"}]}'
```

- Test in the client:

```bash
# Get the NodePort
NODE_PORT=$(sudo kubectl get svc ollama-service -o jsonpath='{.spec.ports[0].nodePort}')
echo "NodePort: $NODE_PORT"

# Test the model
curl -X POST http://<VM_PUBLIC_IP>:<NODE_PORT>/api/chat -d '{"model": "deepseek-r1:1.5b", "messages": [{"role": "user", "content": "Hello"}]}'
```
