# Project 5 - Kubernetes CI/CD with GitHub Actions, GHCR, Argo CD and Reloader

## Project Overview

This project demonstrates a complete CI/CD and GitOps workflow using:

- Python Flask
- Docker
- GitHub Actions
- GitHub Container Registry (GHCR)
- Kubernetes
- Argo CD
- Stakater Reloader

The application is automatically tested, containerized, pushed to GHCR, deployed to Kubernetes through Argo CD, and automatically restarted when configuration changes.

---

## Architecture

```text
Developer
   ↓
GitHub Repository
   ↓
GitHub Actions
   ↓
Automated Tests
   ↓
Docker Build
   ↓
GitHub Container Registry
   ↓
deployment.yaml updated automatically
   ↓
Git commit by GitHub Actions
   ↓
Argo CD
   ↓
Kubernetes Deployment
   ↓
Rolling Update
   ↓
Application Pods
   ↓
Kubernetes Service
   ↓
Application