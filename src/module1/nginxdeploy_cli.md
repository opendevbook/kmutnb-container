# workshop2 deployengine

Using kubectl run:
```
$ kubectl run nginx-pod --image=nginx --restart=Never --port=80 -n default
```

This command creates a pod named nginx-pod in default namespace using the Nginx Docker image. The --restart=Never flag indicates that it's a one-time job and won't be restarted automatically if it fails or stops.

```
$ kubectl get pods
```

Now pod is up and running let’s create a service to access application externally

Using kubectl run:

```
$ kubectl expose pod nginx-pod --type=NodePort --port=80 --name=nginx-service
```

This command exposes the Nginx pod using a NodePort service, making it accessible externally on a specific port.

Verify the service is created using below command:

```
$ kubectl get svc
```

```
minikube service nginx-service --url
```

user ssh -L  forward port to windows

```
ssh -L  5002:localhost:<nodeport>  vagrant@192.168.40.10
```