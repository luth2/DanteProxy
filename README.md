# DanteProxy
Dante Proxy Container and Helm Chart

## Lokales Testen

```cmd
docker run -d \
  -p 1080:1080 \
  -v $(pwd)/danted.conf:/etc/danted/danted.conf:ro \
  --name dante \
  dante:prod
```

## Kubernetes Deployment (Minimal)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dante
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dante
  template:
    metadata:
      labels:
        app: dante
    spec:
      securityContext:
        runAsUser: 10001
        runAsGroup: 10001
        runAsNonRoot: true
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
      containers:
      - name: dante
        image: dante:prod
        ports:
        - containerPort: 1080
        volumeMounts:
        - name: config
          mountPath: /etc/danted
          readOnly: true
        - name: run
          mountPath: /var/run/danted
        livenessProbe:
          exec:
            command: ["sh", "-c", "ss -lnt | grep -q ':1080'"]
          initialDelaySeconds: 10
          periodSeconds: 30
      volumes:
      - name: config
        configMap:
          name: dante-config
      - name: run
        emptyDir: {}
```
