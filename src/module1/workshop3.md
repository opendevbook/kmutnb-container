# Workshop2 Nginx+NodePort
# Deploying NGINX on Kubernetes Using Deployment YAML

**Workshop Overview:**  
This workshop will walk participants through the steps to deploy an NGINX web server on Kubernetes using a YAML manifest file. By the end of the workshop, participants will have learned how to create and apply a Kubernetes Deployment, manage Pods, and expose the application via a Kubernetes Service.

Learn:
- Basic knowledge of Kubernetes concepts (Pods, Deployments, and Services).
Access to a Kubernetes cluster
- kubectl installed and configured to communicate with the Kubernetes cluster.

# Hands-on Section:
- **create nginx-deployment.yml file**
```
mkdir workshop2
cd workshop2
```

```
cat <<EOF | tee nginx-deployment.yml
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
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

EOF
```

- Apply Deployment
```
$ kubectl apply -f nginx-deployment.yml
```

- Verify
```
$ kubectl get deployments -A
$ kubectl get pods -A
```

- Create service file
```
cat <<EOF | tee nginx-service.yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
EOF
```

- Apply Service
```
$ kubectl apply -f nginx-service.yml
```

- Run
```
$ kubectl get svc -A
$ kubectl get pod -A -o wide
```

open browser 
[http://192.168.35.21:30007]