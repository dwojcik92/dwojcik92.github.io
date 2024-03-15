---
title: "Docker - 102"
author: "Dariusz Wójcik"
date: 2024-03-15
draft: false
tags: ["Docker", "Web Development", "DevOps"]
categories: ["oc"]
---

Welcome to our comprehensive guide on Dockerized microservices architecture! In this tutorial, we'll take a deep dive into setting up a microservices architecture using Docker Compose, complete with detailed code examples for each microservice.

### Set Up the Environment

Before we dive into Docker Compose, ensure you have it installed on your system:

```bash
docker-compose --version
```

Create a project directory for our microservices architecture:

```bash
mkdir docker_microservices
cd docker_microservices
```

### Create Docker Compose YAML File

Start by creating a `docker-compose.yml` file in the project directory:

```bash
touch docker-compose.yml
```

### Develop Microservices

For our demonstration, we'll have two microservices: `app1` and `app2`, both built using Flask.

#### `app1` Microservice

**Dockerfile:** `./app1/Dockerfile`
```Dockerfile
# Use the official Python image as base
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install Flask and requests library
RUN pip install Flask requests

# Set environment variables
ENV FLASK_APP=app1.py
ENV FLASK_RUN_HOST=0.0.0.0

# Expose port 5001
EXPOSE 5001

# Run the Flask application
CMD ["python", "app1.py"]
```

**Application code:** `./app1/app1.py`
```python
from flask import Flask, jsonify
import requests

app = Flask(__name__)

@app.route('/')
def index():
    response = requests.get('http://app2:5002')
    return jsonify(message="App1 received response from App2",
                   app2_response=response.json())

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

#### `app2` Microservice

**Dockerfile:** `./app2/Dockerfile`
```Dockerfile
# Use the official Python image as base
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install Flask
RUN pip install Flask

# Set environment variables
ENV FLASK_APP=app2.py
ENV FLASK_RUN_HOST=0.0.0.0

# Expose port 5002
EXPOSE 5002

# Run the Flask application
CMD ["python", "app2.py"]
```

**Application code:** `./app2/app2.py`
```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/')
def index():
    return jsonify(message="Hello from App2!")

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5002)
```

### Define Docker Compose Services

Now, let's define the services for each microservice in the `docker-compose.yml` file:

```yaml
version: '3'

services:
  app1:
    build: ./app1
    ports:
      - "5001:5001"
    networks:
      - my_network

  app2:
    build: ./app2
    networks:
      - my_network

networks:
  my_network:
    driver: bridge
```

### Build and Run the Application

It's time to build and run our microservices application using Docker Compose:

```bash
docker-compose up --build -d
```

Verify that all services are running without errors:

```bash
docker-compose ps
```

### Test the Application

Test the functionality of the application by sending requests to the exposed endpoints of each microservice. Ensure that data is being exchanged between microservices correctly.

### Scale Services (Optional)

If you want to demonstrate scaling of services, you can use Docker Compose:

```bash
docker-compose scale <service_name>=<number_of_instances>
```

### Volume Mounting and Persistent Data Storage

Modify the `docker-compose.yml` file to include volume mounts for persistent data storage. This ensures that data persists even after containers are stopped and restarted.

### Cleanup

Once you're done, stop and remove all containers created by Docker Compose:

```bash
docker-compose down
```

**Conclusion:**

In this tutorial, we've explored Dockerized microservices architecture in depth, covering Docker Compose, multi-container applications, networking, volume mounting for persistent data storage, scalability, and more. Dockerized microservices offer flexibility, scalability, and easy management, making them a powerful choice for modern applications.

Now that you've mastered Dockerized microservices, feel free to explore more advanced Docker features and architectures. Happy containerizing!

Welcome to the world of Dockerized microservices architecture! In this guide, we'll walk you through setting up a microservices architecture using Docker Compose, creating multiple services, communicating between them, and ensuring scalability and persistence.