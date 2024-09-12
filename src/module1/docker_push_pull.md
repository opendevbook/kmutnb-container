# Docker push/pull

Docker Lab push pull

Install Docker: Make sure Docker is installed on your machine. You can follow the official installation guide for your operating system.

Create a Simple Dockerfile: Create a directory for your Docker project and add a Dockerfile. This file will define the image you want to build.

```
mkdir my-docker-lab
cd my-docker-lab
```

Create a file named Dockerfile in this directory with the following content:

Dockerfile
```
cat <<EOF | tee Dockerfile
# Use an official Python runtime as a parent image
FROM python

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy the current directory contents into the container at /usr/src/app
COPY . .

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
EOF
```


Create a Requirements File: Create a requirements.txt file to specify Python dependencies:
```
cat <<EOF | tee requirements.txt
Flask
EOF
```

Create a Simple Python Application: Create a file named app.py:

```
cat <<EOF | tee app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
EOF
```

Build Your Docker Image: From the directory containing your Dockerfile, build the Docker image:

```
$ docker build -t my-flask-app .
````

Run Your Docker Container: After building the image, run a container based on this image:

```
$ docker run -p 4000:80 my-flask-app
```

Your Flask app should now be accessible at http://localhost:4000.
```
ss -tulpn | grep 4000
curl http://localhost:4000
```

Docker Push and Pull Commands
To push and pull Docker images to and from a Docker registry (e.g., Docker Hub), follow these steps:

Tag Your Image: Before pushing an image to Docker Hub, you need to tag it with your repository name. If your Docker Hub username is yourusername and your image name is my-flask-app, tag it like this:

```
docker tag my-flask-app yourrreponame/my-flask-app:latest
```
Login to Docker Hub: Log in to Docker Hub using your credentials:

```
$ docker login
```

Push the Image to Docker Hub: Push the tagged image to Docker Hub:

```
$ docker push yourrreponame/my-flask-app:latest
```

Pull the Image from Docker Hub: To pull the image from Docker Hub to another machine, use:

```
$ docker pull yourusername/my-flask-app:latest
```

Run the Pulled Image: After pulling the image, you can run it just like any other Docker image:

```
$ docker run -p 4000:80 yourusername/my-flask-app:latest
```

This will pull the image from Docker Hub and run it locally, making your Flask app accessible at http://localhost:4000.