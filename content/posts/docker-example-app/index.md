---
title: "Docker - Example Apps"
author: "Dariusz Wójcik"
date: 2024-03-15
draft: false
tags: ["Docker", "Web Development", "DevOps"]
categories: ["oc"]
---


#  List of cool system designs created with docker
- [Ollama backend + chat UI](https://github.com/valiantlynx/ollama-docker)
- [Wordpress](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-docker-compose)
- [Gitea + postgress](https://github.com/awesome-release/gitea-postgres)
- [Voting App](https://github.com/awesome-release/release-example-voting-app)

### Tutorial: Building an Image Classifier App with Docker Compose, FastAPI, and Scikit-learn

In this tutorial, we'll create a web application where users can upload images to classify using a machine learning model. We'll utilize Docker Compose to manage multiple containers, FastAPI for the API, and scikit-learn for image classification.

### Step 1: Setting Up the Project Structure

Create a directory for your project and organize the structure:

```bash
mkdir image_classifier_app
cd image_classifier_app
```

Inside this directory, create the following structure:

```
image_classifier_app
│   ├── frontend
│   │   ├── Dockerfile
│   │   ├── templates/index.html
│   │   └── app.py
│   ├── backend
│   │   ├── Dockerfile
│   │   ├── classify.py
│   │   └── requirements.txt
│   ├── database
│   │   └── ...
│   ├── docker-compose.yml
│   └── README.md
```

### Step 2: Frontend Container

#### `frontend/Dockerfile`

```Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

#### `frontend/app.py`

```python
from flask import Flask, render_template, request, redirect, url_for
import requests
import os

app = Flask(__name__)

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        file = request.files['file']
        if file:
            file.save(os.path.join('uploads', file.filename))
            response = requests.post('http://backend:8000/classify/', files={'file': open(f'uploads/{file.filename}', 'rb')})
            classification_result = response.json()['result']
            return render_template('index.html', result=classification_result, filename=file.filename)
    return render_template('index.html', result=None, filename=None)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### `frontend/templates/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Image Classifier</title>
</head>
<body>
    <h1>Image Classifier</h1>
    <form method="POST" enctype="multipart/form-data">
        <input type="file" name="file">
        <button type="submit">Upload</button>
    </form>
    {% if filename %}
    <h2>Uploaded Image: {{ filename }}</h2>
    {% endif %}
    {% if result %}
    <h2>Classification Result: {{ result }}</h2>
    {% endif %}
</body>
</html>
```

#### `frontend/Dockerfile`

```Dockerfile
FROM tiangolo/uvicorn-gunicorn-fastapi:python3.9-slim

COPY ./app /app

RUN mkdir -p /app/uploads

RUN pip install requests

EXPOSE 80
```


### Step 3: Building the Backend Container

#### `backend/Dockerfile`

```Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "classify:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### `backend/classify.py`

```python
from fastapi import FastAPI, File, UploadFile
from fastapi.responses import JSONResponse
import shutil

app = FastAPI()

@app.post("/classify/")
async def classify_image(file: UploadFile = File(...)):
    try:
        # Save the uploaded file temporarily
        with open(f'./temp/{file.filename}', 'wb') as buffer:
            shutil.copyfileobj(file.file, buffer)

        # Perform image classification here using scikit-learn
        # Example:
        # classification_result = some_function_to_classify_image(f'./temp/{file.filename}')

        # Dummy response for demonstration
        classification_result = {'result': 'Some classification result'}

        return JSONResponse(content=classification_result)

    except Exception as e:
        return JSONResponse(content={'error': str(e)})
```

### Step 4: Docker Compose Configuration

#### `docker-compose.yml`

```yaml
version: '3'

services:
  frontend:
    build:
      context: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
    ports:
      - "8000:8000"
```

### Step 5: Running the Application

To run the application, navigate to the project directory and execute the following command:

```bash
docker-compose up --build
```

Once the containers are up and running, you can access the frontend application at `http://localhost` in your web browser. Upload an image, and the frontend will forward it to the backend for classification. The classification result will be returned to the frontend and displayed to the user.

That's it! You've successfully built an image classifier web application using Docker Compose, FastAPI, and scikit-learn. Feel free to extend and customize it further according to your requirements.