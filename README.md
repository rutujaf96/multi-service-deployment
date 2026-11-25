# Four Microservices / Four Environments

This repository contains exactly four microservices and four deployment environments.

## Microservices
- payment-service
- customer-service
- order-service
- healthcare-service

## Environments
- dev
- qa
- uat
- prod

## Layout
- `services/` contains the four independent Spring Boot sample applications.
- `environments/` contains environment-specific Kubernetes configuration.
- `Jenkinsfile` is a starter pipeline structure. Configure Jenkins credentials,
  AWS account, ECR repositories, EKS cluster names, and deployment permissions
  before using it.

Each service has its own Maven project, Dockerfile, REST endpoint, and test.
The environment files provide namespace, replica, and resource settings.
