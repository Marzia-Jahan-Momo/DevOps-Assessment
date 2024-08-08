### Preparing a docker-compose for a nginx server.
#### docker-compose.yml
```

version: '3.8'

services:
  nginx:
    image: nginx:latest
    container_name: nginx_server
    ports:
      - "80:80"
    volumes:
      - nginx_logs:/var/log/nginx
    networks:
      - nginx_network

volumes:
  nginx_logs:
    driver: local

networks:
  nginx_network:
    driver: bridge
    ipam:
      config:
        - subnet: "172.20.8.0/24"

Explanation:

Volumes: nginx_logs ensures that the logs persist between container restarts.
Networks: The nginx_network network is created with the specified subnet 172.20.8.0/24.
``` 
![Namespace & services](Docker-Kubernetes/Kubernetes-production.png)

### Kubernetes command to identify the reason for a pod restart under namespace "production".
#### sudo vim nginx-pod.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80


---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```
```
kubectl apply -f nginx-pod.yaml

To check the reason for a pod restart within a namespace we should use the kubectl describe command.

kubectl describe pod nginx-deployment-5ccdc5f64c-46xhr -n production
kubectl logs nginx-deployment-5ccdc5f64c-46xhr -n production
```


### Java-app keep restarting at random. From Kubernetes configuration perspective, the possible reasons for the pod restarts are:

- Memory Limits: Pod may be exceeding its memory limt of 1500Mi that may cause OutOfMemoryError 
- Heap Size Configuration: 1000M heap size should be aligned with the pods memody allocation/
- Resource Quota Exceeded
- Incorrectly configured readiness and liveness probes that may cause continuous restart of pods.
- At last application might have bugs or issues causing crashes and restarts

