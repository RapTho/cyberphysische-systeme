# Container Technology: The Fundamentals

This guide introduces key container concepts. We'll explore what containers are, how they differ from virtual machines, and the ecosystem that supports them.

## What are Containers?

Containers are lightweight, standalone, executable software packages that include everything needed to run an application:

- Code
- Runtime
- System tools
- System libraries
- Settings

### Containers vs. Virtual Machines

![Containers vs VMs](https://s7280.pcdn.co/wp-content/uploads/2018/07/containers-vs-virtual-machines.jpg)
[Source](https://s7280.pcdn.co/wp-content/uploads/2018/07/containers-vs-virtual-machines.jpg)

| Aspect           | Virtual Machines (VMs)                      | Containers                              |
| ---------------- | ------------------------------------------- | --------------------------------------- |
| Operating System | Include a full OS with kernel               | Share the host OS kernel                |
| Resource Usage   | Require more CPU, memory, and storage       | Require fewer resources                 |
| Startup Time     | Take minutes to start                       | Start in seconds or milliseconds        |
| Isolation        | Strong isolation at hardware/VM layer       | Process-level isolation                 |
| Size             | Typically several GB in size                | Typically tens of MB in size            |
| Portability      | Tied more to hypervisor and OS              | Easily portable across environments     |
| Use Cases        | Traditional apps, legacy workloads, full OS | Microservices, cloud-native apps, CI/CD |

## What is a Container Engine?

A container engine is software that creates, manages, and runs containers. Examples include:

- **Docker**: The most well-known container engine
- **Podman**: A daemonless alternative to Docker (used in this course)

### Podman

Podman is a container engine that:

- Runs containers without requiring a daemon (background service)
- Can run containers as a non-root user (rootless), improving security
- Is compatible with Docker commands and container formats

## Container registries

Container registries are repositories for storing and distributing container images. Think of them as:

- GitHub for container images
- App stores for containers

### Why we need container registries

1. **Distribution**: Share container images between environments, teams and cloud services
2. **Version Control**: Store different versions of your container images
3. **Security**: Scan images for vulnerabilities
4. **Automation**: Enable CI/CD pipelines to push and pull images

## Kubernetes and Knative

### Kubernetes

Kubernetes (K8s) is an open-source platform for automating deployment, scaling, and management of containerized applications. Kubernetes handles:

- Self-healing (restarting failed containers)
- Service discovery and load balancing
- Automated rollouts and rollbacks
- Storage orchestration

### IBM Code Engine

IBM Code Engine is a fully managed, serverless platform that runs your containerized workloads, including batch jobs and applications. It's built on Kubernetes but abstracts away their complexity.

Benefits:

- **Serverless**: Pay only for what you use
- **No Kubernetes expertise required**: Focus on your code, not infrastructure
- **Automatic scaling**: Handles traffic spikes and scales to zero when not in use
- **Built-in build tools**: Build container images directly from source code
- **Integrated with IBM Cloud**: Easy access to other IBM Cloud services

In this course, you'll deploy your containerized Python application to IBM Code Engine, experiencing firsthand how these technologies work together to simplify application deployment and management.

## The Container workflow

Throughout this course, you'll follow this typical container workflow:

1. **Develop** your application locally
2. **Containerize** it using a Containerfile
3. **Build** the container image
4. **Push** the image to a container registry
5. **Deploy** the image to a runtime environment (IBM Code Engine)
6. **Update** your application and redeploy

This workflow enables continuous delivery of your applications with consistency across environments.

## HTTP Requests Basics

### HTTP Methods (Verbs)

- **GET**: Retrieve data
- **POST**: Create new resource
- **PUT**: Update resource
- **DELETE**: Remove resource

### API Structure

- **Server URL**: Base address (e.g., `https://api.chucknorris.io`)
- **Endpoint**: Specific path (e.g., `/jokes/random`)
- **Complete URL**: `https://api.chucknorris.io/jokes/random`

### Authentication

- **API Key**: Simple string, often in headers (`X-API-Key: your_key`)
- **Bearer Token**: More secure, in Authorization header (`Authorization: Bearer <token>`)

### Example: Chuck Norris API

```bash
# Get a random joke
curl https://api.chucknorris.io/jokes/random

# Get joke from category
curl https://api.chucknorris.io/jokes/random?category=dev
```

Now that you understand the fundamentals, let's move on to setting up the [prerequisites](./1_Pre-requisites.md) for this course
