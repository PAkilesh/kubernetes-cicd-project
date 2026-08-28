# Project 5 - Kubernetes Deployment and CI/CD

A hands-on DevOps project demonstrating how to containerize a Python Flask application, deploy it to Kubernetes, manage application configuration and secrets, configure health probes and resource limits, perform rolling updates and rollbacks, and scale application replicas.

## Project Architecture

```text
Developer
    |
    v
GitHub
    |
    v
GitHub Actions
    |
    v
Automated Tests
    |
    v
Docker Build
    |
    v
Container Registry
    |
    v
Kubernetes
    |
    v
Deployment
    |
    v
Multiple Application Pods
    |
    v
Service
    |
    v
Flask Application
```

## Technologies Used

- Python
- Flask
- Docker
- Kubernetes
- Docker Desktop Kubernetes
- Git
- GitHub
- GitHub Actions

## Project Structure

```text
kubernetes-cicd-project/
├── app/
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
├── k8s/
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── namespace.yaml
│   ├── secret.example.yaml
│   └── service.yaml
├── .gitignore
└── README.md
```

## Application

The project uses a simple Python Flask application running on port 5000.

The application provides two endpoints.

### Home Endpoint

```text
/
```

Returns the application environment, message, and running status.

### Health Endpoint

```text
/health
```

Returns the health status of the application.

Kubernetes uses this endpoint for liveness and readiness probes.

## Docker

The Flask application is packaged as a Docker container.

The Dockerfile:

- Uses a lightweight Python base image
- Installs Python dependencies
- Copies the Flask application
- Exposes port 5000
- Starts the Flask application

## Kubernetes Namespace

The application runs inside a dedicated Kubernetes namespace:

```text
project5
```

This provides logical isolation for the project's Kubernetes resources.

## Kubernetes Deployment

The Flask application is managed using a Kubernetes Deployment.

The Deployment normally maintains:

```text
3 replicas
```

Kubernetes continuously compares the desired state with the actual state and creates replacement Pods when necessary.

## Self-Healing

Pod self-healing was tested by manually deleting an application Pod.

The Deployment automatically created a replacement Pod to restore the desired replica count.

This demonstrated Kubernetes desired-state management and self-healing.

## Kubernetes Service

A NodePort Service exposes the Flask application.

Traffic flow:

```text
Client
   |
   v
Service
   |
   v
Application Pods
   |
   v
Flask :5000
```

The Service selects Pods using Kubernetes labels.

Internal Kubernetes Service communication was also tested successfully.

For local Docker Desktop testing, `kubectl port-forward` was used to access the application from the host machine.

## Rolling Updates

Multiple versions of the Flask application were deployed.

Kubernetes performed rolling updates by gradually creating Pods running the new application version and terminating Pods running the previous version.

This allowed application updates without replacing every Pod simultaneously.

## Rollback

Kubernetes Deployment rollback was tested using rollout history.

A previous healthy application version was restored after deploying a newer version.

This demonstrated how Kubernetes can recover from problematic application releases.

## Liveness Probe

A liveness probe checks whether the application is still functioning.

The project uses:

```text
/health
```

as the liveness endpoint.

If the health check repeatedly fails, Kubernetes restarts the container.

## Readiness Probe

A readiness probe determines whether a Pod is ready to receive application traffic.

If the readiness check fails, Kubernetes marks the Pod as not ready and prevents the Service from sending traffic to that Pod.

## Failed Deployment Test

An intentionally unhealthy application version was created.

The `/health` endpoint was configured to return HTTP status code 500.

Kubernetes detected:

```text
Readiness probe failed
Liveness probe failed
```

The unhealthy Pod became:

```text
0/1 Running
```

and Kubernetes repeatedly restarted the container after liveness probe failures.

Meanwhile, the previous healthy Pods remained available.

This demonstrated Kubernetes health checking and failed-release protection.

## ConfigMap

Application configuration is stored outside the Docker image using a Kubernetes ConfigMap.

Example configuration:

```text
APP_ENV=production
APP_MESSAGE=Project 5 ConfigMap Updated Successfully
```

The Deployment injects these values into the application container as environment variables.

The Flask application reads them using Python environment variables.

## ConfigMap Update Test

The ConfigMap value was changed without rebuilding the Docker image.

Existing Pods initially retained the previous environment variable values.

The Deployment was restarted, after which the new Pods received the updated ConfigMap values.

This demonstrated the difference between application code changes and configuration changes.

## Kubernetes Secrets

Sensitive application configuration is handled using Kubernetes Secrets.

The lab Secret contains example variables such as:

```text
DATABASE_USER
DATABASE_PASSWORD
```

The Secret is injected into the application container as environment variables.

The actual Secret manifest is excluded from Git.

## Secret Security

The following file is intentionally ignored:

```text
k8s/secret.yaml
```

A safe template is provided instead:

```text
k8s/secret.example.yaml
```

Real passwords, API keys, tokens, certificates, or production credentials should never be committed directly to a public Git repository.

Base64 encoding used by Kubernetes Secrets should not be confused with encryption.

## Resource Requests

Each application container requests:

```text
CPU: 100m
Memory: 64Mi
```

`100m` CPU represents 100 millicores, or 0.1 CPU core.

The request tells Kubernetes how much resource should be considered when scheduling the Pod.

## Resource Limits

Each application container has the following limits:

```text
CPU: 250m
Memory: 128Mi
```

The limits restrict how much CPU and memory the container can consume.

## Kubernetes QoS

After configuring resource requests and limits, the Pods use:

```text
QoS Class: Burstable
```

Before resource configuration, the Pods were:

```text
QoS Class: BestEffort
```

## Scaling

The Deployment normally runs three replicas.

Manual scaling was tested using:

```text
3 Pods
   |
   v
5 Pods
```

Kubernetes automatically created two additional Pods.

The Deployment was then returned from five replicas to three replicas using the declarative Kubernetes manifest.

This demonstrated the difference between imperative and declarative Kubernetes management.

## Imperative vs Declarative Management

An imperative command directly changes the live cluster.

Example:

```bash
kubectl scale deployment project5-app --replicas=5 -n project5
```

Declarative management defines the desired state in YAML:

```yaml
spec:
  replicas: 3
```

Applying the manifest causes Kubernetes to make the live cluster match the declared configuration.

## Configuration Drift

Configuration drift was demonstrated during the rollback exercise.

The live Kubernetes Deployment was rolled back to a healthy application version while the local Deployment YAML still referenced the unhealthy version.

Applying the old YAML caused Kubernetes to deploy the unhealthy version again.

This demonstrated why version-controlled configuration should be maintained as the source of truth.

## Current Kubernetes Features

The project currently implements:

- Namespace isolation
- Kubernetes Deployment
- Three application replicas
- Kubernetes Service
- Pod self-healing
- Rolling updates
- Deployment rollback
- Liveness probes
- Readiness probes
- ConfigMap integration
- Kubernetes Secret integration
- CPU requests
- Memory requests
- CPU limits
- Memory limits
- Burstable QoS
- Manual scaling
- Declarative replica management
- Configuration drift testing

## Useful Commands

Check Pods:

```bash
kubectl get pods -n project5
```

Check Deployment:

```bash
kubectl get deployment project5-app -n project5
```

Check Services:

```bash
kubectl get services -n project5
```

Check ConfigMaps:

```bash
kubectl get configmap -n project5
```

Check Secrets:

```bash
kubectl get secrets -n project5
```

Inspect a Pod:

```bash
kubectl describe pod POD_NAME -n project5
```

Watch Pods:

```bash
kubectl get pods -n project5 -w
```

Check rollout status:

```bash
kubectl rollout status deployment/project5-app -n project5
```

View rollout history:

```bash
kubectl rollout history deployment/project5-app -n project5
```

Rollback a Deployment:

```bash
kubectl rollout undo deployment/project5-app -n project5
```

Restart a Deployment:

```bash
kubectl rollout restart deployment/project5-app -n project5
```

Port-forward the Service:

```bash
kubectl port-forward service/project5-service 8080:80 -n project5
```

## CI/CD

The next stage of the project will implement a GitHub Actions CI/CD pipeline.

Planned pipeline:

```text
Developer
    |
    v
Git Push
    |
    v
GitHub Actions
    |
    v
Application Tests
    |
    v
Docker Image Build
    |
    v
Container Registry
    |
    v
Kubernetes Deployment
```

The CI/CD stage will automate application validation, Docker image creation, container registry publishing, and Kubernetes deployment processes.

## Project Goal

The goal of this project is to gain practical experience with Kubernetes application deployment and DevOps CI/CD concepts using a real containerized Flask application.
