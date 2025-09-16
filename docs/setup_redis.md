# Set up Redis for job queue

Before (no job queue; synchronous requests):

```text
[Internet Clients] -> [Nginx] -> [Ollama Container]
```

After:
```text
[Internet Clients] -> [Nginx] -> [Producer] -> [Redis] -> [Worker] -> [Ollama Container]
```
