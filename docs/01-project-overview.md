
# End-to-End Microservices CI/CD Platform

## 1. Project Overview

This project demonstrates an end-to-end DevOps implementation for two Java Spring Boot microservices: Product Service and Order Service.

The objective is to automate application build, testing, code quality analysis, security scanning, containerization, infrastructure provisioning, deployment, monitoring and logging.

## 2. Technology Stack

| Category | Technology |
|---|---|
| Programming Language | Java 21 |
| Framework | Spring Boot |
| Build Tool | Apache Maven |
| Unit Testing | JUnit |
| Code Coverage | JaCoCo |
| Code Quality | SonarQube |
| Security Scanning | Trivy |
| Version Control | Git and GitHub |
| CI/CD | Jenkins |
| Containerization | Docker |
| Container Registry | Amazon ECR |
| Infrastructure as Code | Terraform |
| Cloud Platform | AWS |
| Container Orchestration | Kubernetes (EKS) |
| Deployment | Helm |
| Database | PostgreSQL (RDS) |
| Database Migration | Flyway |
| Load Balancing | AWS Application Load Balancer |
| Monitoring | Prometheus and Grafana |
| Logging | AWS CloudWatch |

## 3. Microservices

### Product Service
- Java Spring Boot application.
- Maven-based build.
- JUnit test coverage.
- Docker containerization.

### Order Service
- Java Spring Boot application.
- Maven-based build.
- JUnit test coverage.
- Docker containerization.

## 4. End-to-End CI/CD Workflow

1. Developer pushes code to GitHub.
2. Jenkins checks out the code.
3. Maven builds both microservices.
4. JUnit executes unit tests.
5. JaCoCo generates code coverage reports.
6. SonarQube performs code quality analysis.
7. Trivy scans source code and Docker images.
8. Docker builds Product and Order images.
9. Jenkins pushes images to Amazon ECR.
10. Terraform provisions VPC, subnets, IAM, EKS and RDS.
11. Flyway manages PostgreSQL schema migrations.
12. Jenkins deploys applications using Helm.
13. AWS Load Balancer Controller provisions the ALB.
14. ALB routes external traffic to Kubernetes services.
15. Prometheus collects metrics.
16. Grafana displays dashboards and alerts.
17. CloudWatch collects logs.
18. Failure testing, scaling, recovery and rollback are performed.

## 5. Repository Structure

```text
microservices-cicd-platform-new/
├── docs/
├── services/
│   ├── product-service/
│   └── order-service/
└── .gitignore
```

Additional infrastructure, pipeline, deployment and monitoring folders will be added as their implementation stages begin.

## 6. Project Objectives

- Build automated CI/CD pipelines.
- Implement automated testing and code quality checks.
- Scan application code and container images for vulnerabilities.
- Provision AWS infrastructure using Terraform.
- Deploy microservices to Amazon EKS.
- Implement monitoring, logging and alerting.
- Test application failure, recovery, scaling and rollback.

## 7. Implementation Approach

Each implementation stage will include:
- Dependencies and versions.
- Installation instructions.
- Configuration steps.
- Commands and scripts.
- Execution instructions.
- Verification and expected output.
- Troubleshooting.
- Screenshots and evidence.

No stage will be considered complete until its implementation and documentation are verified.

## 8. Current Project Status

| Component | Status |
|---|---|
| Java 21 | Installed |
| Maven 3.10.0 | Installed |
| Product Service | Copied and tested |
| Order Service | Copied and tested |
| Git repository | Initialized |
| GitHub repository | Created and initial code pushed |
| Documentation | In progress |
| Jenkins | Pending |
| Docker | Pending |
| AWS infrastructure | Pending |
| Kubernetes deployment | Pending |
| Monitoring | Pending |
