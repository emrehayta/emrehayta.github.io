---
title: "Docker + Docker Compose: Getting Started and Best Practices"
date: 2024-10-11
tags: ["Docker", "Docker Compose", "Nginx", "Web Development", "DevOps", "Containerization", "Docker Tutorial", "Custom Docker Image", "Software Deployment", "Shell Script"]
description: "Learn how to use Docker and Docker Compose to easily manage containerized applications. This guide provides a step-by-step tutorial to run an Nginx web server, create custom Docker images, and automate deployment with a shell script."
draft: false
---

### Introduction to Docker

Docker has become an essential tool in modern software development. Combined with Docker Compose, managing multi-container applications is straightforward. In this blog post, I will show you how to work with Docker and Docker Compose, including how to run a simple Nginx web server and build your own Docker images.

### What is Docker?

Docker is a containerization platform that allows developers to run applications in isolated environments called containers. These containers package all the dependencies required to run an application, ensuring consistency across different environments.

### What is Docker Compose?

Docker Compose is a tool for defining and running multi-container Docker applications. It allows you to configure your application's services, networks, and volumes in a single YAML file, making it easier to manage complex applications.

---


### Example Project Structure
Let’s create a simple project using Docker and Docker Compose to run an Nginx web server.

#### 1. Project Directory Structure:

```bash
my-nginx-app/
├── docker-compose.yml
├── deploy.sh
└── index.html
```

#### 2. Sample index.html
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to Nginx!</title>
</head>
<body>
    <h1>Hello, Docker with Nginx!</h1>
    <p>This is a simple web page served by Nginx.</p>
</body>
</html>
```

#### 3. Sample docker-compose.yml

```bash
version: '3'
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html
```
---

### Deploy Script: deploy.sh
Now, let’s create a simple shell script to start the Nginx server using Docker Compose.

```bash
#!/bin/bash

# Start the Nginx application using Docker Compose
docker-compose up
```

### Running the Application

1. Make sure you have Docker and Docker Compose installed on your machine.
2. Navigate to the project directory:
```bash
cd my-nginx-app
```

3. Make the deploy.sh script executable:
```bash
chmod +x deploy.sh
```
4. Run the deploy script:
```bash
./deploy.sh
```

Your Nginx server will start, and you can access it by visiting http://localhost:8080 in your web browser. You should see the message "Hello, Docker with Nginx!" displayed on the page.



### Building Custom Docker Images

If you want to build your own Docker image, you can create a simple Dockerfile. Here’s an example of how to create a custom Nginx image that serves your own HTML content.

1. Sample Dockerfile
```bash
# Use the official Nginx image as a base
FROM nginx:latest

# Copy custom HTML file to the Nginx HTML directory
COPY index.html /usr/share/nginx/html/index.html
```

2. Update docker-compose.yml
```bash
version: '3'
services:
  web:
    build: .
    ports:
      - "8080:80"
```
3. Modify the deploy.sh script:
Update the script to build the image before starting the containers:

```bash
#!/bin/bash

# Build and start the Nginx application using Docker Compose
docker-compose up --build
```

### Conclusion
Docker and Docker Compose simplify the development and deployment process, allowing you to manage applications efficiently. In this article, we covered the basics of Docker and Docker Compose, how to run a simple Nginx web server, and how to create a custom Docker image.

Feel free to expand this example further by adding more services or customizing the content as needed.

