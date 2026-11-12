
# Prerequisites and Installation Guide

## 1. Purpose

This document describes the software dependencies, versions, installation instructions and environment configuration required to build and deploy the Microservices CI/CD Platform.

The setup is based on Windows 11 with PowerShell.

## 2. Development Environment

| Component | Version | Status |
|---|---|---|
| Operating System | Windows 11 | Installed |
| Java JDK | Temurin 21.0.12.1 | Verified |
| Apache Maven | 3.10.0 | Verified |
| Git | 2.55.0.windows.5 | Verified |
| Docker Desktop | To be verified | Pending |
| Jenkins | To be installed/configured | Pending |
| SonarQube | To be configured | Pending |
| Trivy | To be installed | Pending |
| AWS CLI | To be verified | Pending |
| Terraform | To be verified | Pending |
| kubectl | To be verified | Pending |
| Helm | To be verified | Pending |
| eksctl | To be verified | Pending |

Versions of pending tools will be recorded when installed and verified.

## 3. Java JDK Installation

### Purpose

Java is required to compile and execute the Spring Boot microservices.

### Installed Version

Temurin JDK 21.0.12.1 LTS.

### Installation Location

```text
C:\Program Files\Eclipse Adoptium\jdk-21.0.12.101-hotspot
```

### Environment Variable

Variable:

```text
JAVA_HOME
```

Value:

```text
C:\Program Files\Eclipse Adoptium\jdk-21.0.12.101-hotspot
```

### Verification

```powershell
java -version
```

```powershell
$env:JAVA_HOME
```

### Verified Output

```text
openjdk version "21.0.12.1" 2026-08-18 LTS
```

Status: Installed and verified.

## 4. Apache Maven Installation

### Purpose

Maven manages Java dependencies, compiles applications, executes unit tests and packages Spring Boot applications.

### Installed Version

Apache Maven 3.10.0.

### Installation Location

```text
C:\Users\Prateek Sangle\Downloads\apache-maven-3.10.0-bin\apache-maven-3.10.0
```

### Environment Variables

MAVEN_HOME:

```text
C:\Users\Prateek Sangle\Downloads\apache-maven-3.10.0-bin\apache-maven-3.10.0
```

PATH entry:

```text
C:\Users\Prateek Sangle\Downloads\apache-maven-3.10.0-bin\apache-maven-3.10.0\bin
```

### Verification Commands

```powershell
mvn -version
```

```powershell
where.exe mvn
```

### Verified Version

```text
Apache Maven 3.10.0
Java version: 21.0.12.1
```

Status: Installed and verified.

## 5. Git Installation

### Purpose

Git manages source code versions and supports integration with GitHub and Jenkins.

### Installed Version

Git 2.55.0.windows.5.

### Verification

```powershell
git --version
```

### Git Identity Configuration

```powershell
git config --global user.name
git config --global user.email
```

Configured identity:

```text
Name: Prateek Sangle
Email: prateeksangle@mail.mirafra.com
```

Status: Installed and verified.

## 6. Docker Desktop

### Purpose

Docker packages the Product and Order microservices into container images.

Docker will also be used for local testing and running supporting services where appropriate.

### Download

https://www.docker.com/products/docker-desktop/

### Verification

```powershell
docker --version
```

```powershell
docker compose version
```

```powershell
docker info
```

Status: Installation to be verified for this project.

## 7. Jenkins

### Purpose

Jenkins automates the CI/CD pipeline.

Planned pipeline activities:

- GitHub checkout.
- Maven build.
- JUnit tests.
- JaCoCo coverage.
- SonarQube analysis.
- Trivy security scans.
- Docker image build.
- Amazon ECR push.
- Helm deployment.

### Official Documentation

https://www.jenkins.io/doc/

Status: Setup pending.

## 8. SonarQube

### Purpose

SonarQube analyzes application code for quality issues, bugs and maintainability problems.

### Official Documentation

https://docs.sonarsource.com/sonarqube/

Status: Setup pending.

## 9. Trivy

### Purpose

Trivy scans application source code, dependencies and Docker images for security vulnerabilities.

### Official Documentation

https://trivy.dev/

Status: Installation pending.

## 10. AWS CLI

### Purpose

AWS CLI provides command-line access to AWS services.

Planned usage:

- Configure AWS credentials.
- Access Amazon ECR.
- Manage EKS.
- Verify AWS resources.
- Support Terraform and deployment operations.

### Official Installation Guide

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

### Verification

```powershell
aws --version
```

Status: Installation to be verified.

## 11. Terraform

### Purpose

Terraform provisions AWS infrastructure using Infrastructure as Code.

Planned resources:

- VPC.
- Public and private subnets.
- IAM roles and policies.
- EKS cluster.
- RDS PostgreSQL.

### Official Download

https://developer.hashicorp.com/terraform/install

### Verification

```powershell
terraform version
```

Status: Installation to be verified.

## 12. Kubernetes Tools

### kubectl

Purpose: Manage Kubernetes resources and applications.

Official documentation:

https://kubernetes.io/docs/tasks/tools/

Verification:

```powershell
kubectl version --client
```

### Helm

Purpose: Package and deploy applications to Kubernetes.

Official documentation:

https://helm.sh/docs/intro/install/

Verification:

```powershell
helm version
```

### eksctl

Purpose: Command-line utility for managing Amazon EKS clusters.

Official documentation:

https://eksctl.io/installation/

Verification:

```powershell
eksctl version
```

Status: Installation to be verified.

## 13. PostgreSQL and Flyway

### PostgreSQL

Purpose: Store application data in Amazon RDS for the final AWS deployment.

Official documentation:

https://www.postgresql.org/docs/

### Flyway

Purpose: Manage and version database schema migrations.

Official documentation:

https://documentation.red-gate.com/flyway

Status: Setup pending.

## 14. Monitoring and Logging

### Prometheus

Purpose: Collect and store application and infrastructure metrics.

Official documentation:

https://prometheus.io/docs/introduction/overview/

### Grafana

Purpose: Visualize metrics through dashboards and configure alerts.

Official documentation:

https://grafana.com/docs/

### AWS CloudWatch

Purpose: Collect and monitor application and infrastructure logs.

Official documentation:

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html

Status: Setup pending.

## 15. Environment Verification Checklist

| Dependency | Verification Command | Status |
|---|---|---|
| Java | `java -version` | Verified |
| Maven | `mvn -version` | Verified |
| Git | `git --version` | Verified |
| Docker | `docker --version` | Pending |
| Jenkins | Jenkins UI | Pending |
| SonarQube | SonarQube UI | Pending |
| Trivy | `trivy --version` | Pending |
| AWS CLI | `aws --version` | Pending |
| Terraform | `terraform version` | Pending |
| kubectl | `kubectl version --client` | Pending |
| Helm | `helm version` | Pending |
| eksctl | `eksctl version` | Pending |

## 16. Important Notes

1. Install only the dependencies required for the current implementation stage.
2. Record the actual installed version after every installation.
3. Never commit AWS credentials, passwords, tokens or private keys to GitHub.
4. Use IAM roles and least-privilege permissions wherever possible.
5. AWS resources must be reviewed for cost before deployment.
6. Document installation errors and their resolutions.
7. Verify each dependency before proceeding to the next stage.
