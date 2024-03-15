---
title: "Docker - 103"
author: "Dariusz Wójcik"
date: 2024-03-15
draft: false
tags: ["Docker", "Web Development", "DevOps"]
categories: ["oc"]
---


**Buiding a Pipeline Using Containers**

This exercise demonstrates how to build a pipeline composed of independent containers whose goal is to train a machine learning model on-demand.

> The workflow
{{<mermaid align="right">}}
graph TD
    style app fill:#6EBBFA,stroke:#333,stroke-width:2px,stroke-dasharray: 5, 5;
    style preprocess fill:#6EBBFA,stroke:#333,stroke-width:2px,stroke-dasharray: 5, 5;
    style db fill:#6EBBFA,stroke:#333,stroke-width:2px,stroke-dasharray: 5, 5;
    style train fill:#6EBBFA,stroke:#333,stroke-width:2px,stroke-dasharray: 5, 5;

    app((App))
    preprocess((Preprocess))
    db((DB))
    train((Train))

    app -->|Requests| train
    train -->|Requests| preprocess
    preprocess -->|Database Query| db

    subgraph Endpoints
        index((/)):::index
        train-response((/train-response)):::train-response
        database-response((/database-response)):::database-response
    end

    index -.->|Response| app
    train-response -.->|Response| train
    database-response -.->|Response| preprocess
{{< /mermaid >}}



### Dockerfile for `app`

**Path:** `./app/Dockerfile`

```dockerfile
# Use the official Python image as base
FROM python:3.9-slim

# Install Flask
RUN pip install Flask requests

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Expose port 5000
EXPOSE 5000

# Run the Flask application
CMD ["python", "app.py"]
```

**Explanation:**
- This Dockerfile starts with the official Python image and installs Flask along with the `requests` library.
- It sets the working directory inside the container as `/app`.
- Copies all files from the current directory (`.`) into the container's `/app` directory.
- Exposes port `5000` to allow communication with the Flask application.
- Finally, it specifies the command to run the Flask application using `CMD`.

---

### Python Script for `app`

**Path:** `./app/app.py`

```python
from flask import Flask, jsonify
import requests

app = Flask(__name__)

@app.route('/')
def index():
    response = requests.get('http://train:5000')
    return jsonify(message="App received response from Train",
                   train_response=response.json())

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Explanation:**
- This Python script defines a Flask application.
- It creates a route `/` which sends a request to the `train` service at port `5000`.
- Upon receiving the response, it returns a JSON message indicating receipt of the response from `Train`.

---

### Dockerfile for `preprocess`

**Path:** `./preprocess/Dockerfile`

```dockerfile
# Use the official Python image as base
FROM python:3.9-slim

# Install Flask
RUN pip install Flask mysql-connector-python

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Expose port 5000
EXPOSE 5000

# Run the Flask application
CMD ["python", "app.py"]
```

**Explanation:**
- Similar to the `app` Dockerfile, this one installs Flask along with `mysql-connector-python`.
- It sets the working directory as `/app` and copies all files into it.
- Exposes port `5000` for communication.
- Specifies the command to run the Flask application.

---

### Python Script for `preprocess`

**Path:** `./preprocess/app.py`

```python
from flask import Flask, jsonify
import mysql.connector

app = Flask(__name__)

# Database connection configuration
db_config = {
    'host': 'db',          # Service name in Docker Compose network
    'user': 'my_user',
    'password': 'my_password',
    'database': 'train_database'
}

# Function to fetch data from the database
def get_data():
    try:
        connection = mysql.connector.connect(**db_config)
        cursor = connection.cursor()
        cursor.execute('SELECT * FROM trains')
        data = cursor.fetchall()
        print(data)
        return data
    except mysql.connector.Error as error:
        return f"Error: {error}"
    finally:
        if connection.is_connected():
            cursor.close()
            connection.close()

@app.route('/')
def index():
    # Fetch data from the database
    data = get_data()
    data_json = {
        "data": data
    }
    return jsonify(message="Preprocess received response from Database",
                   database_response=data_json)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Explanation:**
- This script sets up a Flask application.
- It defines a route `/` which connects to the database (`db`) to fetch data from the `trains` table.
- The fetched data is returned as a JSON response.

---

### Dockerfile for `train`

**Path:** `./train/Dockerfile`

```dockerfile
# Use the official Python image as base
FROM python:3.9-slim

# Install Flask
RUN pip install Flask requests

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Expose port 5000
EXPOSE 5000

# Run the Flask application
CMD ["python", "app.py"]
```

**Explanation:**
- Similar to the previous Dockerfiles, this one installs Flask and sets up the working directory.
- Exposes port `5000` for communication with the Flask application.

---

### Python Script for `train`

**Path:** `./train/app.py`

```python
from flask import Flask, jsonify
import requests

app = Flask(__name__)

@app.route('/')
def index():
    response = requests.get('http://preprocess:5000')
    return jsonify(message="Train received response from Preprocess",
                   preprocess_response=response.json())

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Explanation:**
- This script defines a Flask application.
- It creates a route `/` which sends a request to the `preprocess` service at port `5000`.
- Upon receiving the response, it returns a JSON message indicating receipt of the response from `Preprocess`.

---

### Dockerfile for `db`

**Path:** `./db/Dockerfile`

```dockerfile
# Use the official MySQL image as base
FROM mysql:8.3.0

# Copy the SQL script to create the table into the container
COPY init.sql /docker-entrypoint-initdb.d/
```

**Explanation:**
- This Dockerfile sets up the MySQL database container.
- It uses the official MySQL image and copies the SQL script for table creation into the container's initialization directory.

---

### SQL Script for Initializing `db`

**Path:** `./db/init.sql`

```sql
-- Create the database if it doesn't exist
CREATE DATABASE IF NOT EXISTS train_database;

-- Use the created database
USE train_database;

-- Create table for trains
CREATE TABLE IF NOT EXISTS trains (
    train_id INT AUTO_INCREMENT PRIMARY KEY,
    train_name VARCHAR(255) NOT NULL,
    departure_time INT NOT NULL,
    arrival_time INT NOT NULL,
    departure_station VARCHAR(100) NOT NULL,
    arrival_station VARCHAR(100) NOT NULL
);

-- Generate and insert sample train data
INSERT INTO trains (train_name, departure_time, arrival_time, departure_station, arrival_station)
SELECT
    CONCAT('Train ', numbers.n) AS train_name,
    FLOOR(RAND() * 86400) AS departure_time,
    FLOOR(RAND() * 86400) AS arrival_time,
    CONCAT('Station ', FLOOR(RAND() * 1000)) AS departure_station,
    CONCAT('Station ', FLOOR(RAND() * 1000)) AS arrival_station
FROM (
    SELECT 
        (a.n + b.n * 10 + c.n * 100) AS n
    FROM
        (SELECT 0 AS n UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS a,
        (SELECT 0 AS n UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS b,
        (SELECT 0 AS n UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS c
) AS numbers
LIMIT 1000;
```

**Explanation:**
- This SQL script initializes the MySQL database.
- It creates a database named `train_database` if it doesn't exist and switches to it.
- Then it creates a table `trains` with columns for train details.
- Finally, it generates and inserts sample train data into the table.

---

### Docker Compose Configuration

**Path:** `./docker-compose.yml`

```yaml
version: '3'

services:
  app:
    build: ./app
    ports:
      - "5001:5000"
    networks:
      - my_network

  train:
    build: ./train
    networks:
      - my_network

  preprocess:
    build: ./preprocess
    networks:
      - my_network

  db:
    build: ./db
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: train_database
      MYSQL_USER: my_user
      MYSQL_PASSWORD: my_password
    networks:
      - my_network
    volumes:
      - db_data:/var/lib/mysql

networks:
  my_network:
    driver: bridge

volumes:
  db_data:
```

**Explanation:**
- This YAML file defines a multi-container environment using Docker Compose.
- It specifies services for `app`, `train`, `preprocess`, and `db`.
- Each service is built from its respective Dockerfile.
- The `db` service also configures environment variables for MySQL and mounts a volume for persistent data storage.
- All services are connected to a custom network named `my_network`.

---

This comprehensive setup allows you to build and deploy a pipeline using Docker containers. Each component is modularized and can be scaled or modified independently to suit your requirements.

Feel free to reach out if you have any questions or need further assistance!