# Postgresql & Pod deployment

Here’s a guide on how to create another Podman pod deployment with multiple containers, such as a simple PostgreSQL and Adminer setup. Adminer is a lightweight database management tool, and PostgreSQL will be used as the database.

## 1. Create the Pod
You need to create a pod that exposes ports for both PostgreSQL and Adminer. PostgreSQL typically runs on port 5432, and Adminer uses port 8080.

```
$ podman pod ls
POD ID      NAME        STATUS      CREATED     INFRA ID    # OF CONTAINERS

$ podman pod create --name dbpod -p 5432:5432 -p 8080:8080
```

- -p 5432:5432: Exposes PostgreSQL port 5432 from the container to the host.
- -p 8080:8080: Exposes Adminer’s port 8080 on the host

## 2. Deploy PostgreSQL in the Pod

```
podman run -d \
  --name postgres \
  --pod dbpod \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydb \
  docker.io/library/postgres:latest
```

- -e POSTGRES_USER=myuser: Sets the PostgreSQL username.
- -e POSTGRES_PASSWORD=mypassword: Sets the PostgreSQL password.
- -e POSTGRES_DB=mydb: Creates a new database named mydb.

## 3.  Deploy Adminer in the Pod

```
podman run -d \
  --name adminer \
  --pod dbpod \
  docker.io/library/adminer:latest
```

This will start Adminer inside the pod and make it accessible on port 8080 (mapped to localhost:8080)


## 4. Verify the Pod and Containers
Check the status of the pod and its containers:

```
$ podman pod ps
$ podman ps --pod
```

## 5 Test the Setup

```
sudo dnf install postgrsql
sudo psql -h 127.0.0.1 -U myuser -d mydb -p 5432
```

## Test Adminer

```
http://192.168.30.10:8080
```