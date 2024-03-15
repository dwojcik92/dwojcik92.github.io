---
title: "Docker - 101"
author: "Dariusz Wójcik"
date: 2024-03-15
draft: false
tags: ["Docker", "Web Development", "DevOps"]
categories: ["oc"]
---

Are you ready to containerize your web application effortlessly? Docker provides a convenient solution to package your application into a portable, isolated environment, ensuring consistency across different platforms. In this guide, we'll walk through the steps to dockerize your web application using a simple example project.

### Set Up the Environment

Before diving into Dockerizing your web app, ensure you have Docker installed on your system. You can verify this by running the following command in your terminal:

```bash
docker --version
```

If Docker is not installed, you can download and install it from the official Docker website.

Next, clone or download the web application project from the Git repository:

```bash
git clone https://github.com/M-Enes/todo-app.git
```

### Create a Dockerfile

Once you have the project on your local machine, navigate to the project directory and create a Dockerfile:

```bash
cd <project_directory>
touch Dockerfile
```

Open the Dockerfile using a text editor and define the image:

```Dockerfile
# Use an existing base image
FROM nginx:latest
# Copy the web application files to the container
COPY . /usr/share/nginx/html
```

### Build the Docker Image

With the Dockerfile in place, it's time to build the Docker image using the following command:

```bash
docker build -t my-web-app .
```

### Run the Docker Container

Once the image is built, you can run the Docker container from the image:

```bash
docker run -d -p 8080:80 my-web-app
```

To ensure the container is running, you can check its status with:

```bash
docker ps
```

### Test the Dockerized Web Application

Open a web browser and navigate to `http://localhost:8080` to access the web application. Interact with the application to ensure it functions correctly within the Docker container.

### Publish the Docker Image (Optional)

If you wish to share your Docker image with others or deploy it to a production environment, you can publish it to Docker Hub. First, log in to Docker Hub:

```bash
docker login
```

Then, tag the Docker image with your Docker Hub username and repository name:

```bash
docker tag my-web-app <username>/<repository_name>:<tag>
```

Finally, push the Docker image to Docker Hub:

```bash
docker push <username>/<repository_name>:<tag>
```

### Cleanup

After you're done testing or publishing your Docker image, it's good practice to clean up by stopping and removing the running container:

```bash
docker stop <container_id>
docker rm <container_id>
```

You can also delete the Docker image if it's no longer needed:

```bash
docker rmi my-web-app
```

By following these steps, you've successfully Dockerized your web application, making it more portable, scalable, and easier to manage across different environments. Happy containerizing!