# yaml 模板
## 基础的yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-node-selector
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
      nodeSelector:
        type: cpu
```

## pod affinity测试

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-base
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-os
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-os
  template:
    metadata:
      name: nginx-os
      labels:
        app: nginx-os
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - nginx
              topologyKey: kubernetes.io/os
      containers:
        - name: nginx
          image: nginx
```

