<div align="center">

# ☁️ NextWork Web Project
### A Complete End-to-End CI/CD Pipeline on AWS

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Java](https://img.shields.io/badge/Java_8-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Maven](https://img.shields.io/badge/Apache%20Maven-C71A36?style=for-the-badge&logo=Apache%20Maven&logoColor=white)
![Tomcat](https://img.shields.io/badge/Apache%20Tomcat-F8DC75?style=for-the-badge&logo=apachetomcat&logoColor=black)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

> A hands-on DevOps project documenting the full journey from a blank EC2 instance to a fully automated CI/CD pipeline — covering EC2, Git, AWS CodeArtifact, CodeBuild, and GitHub Actions for continuous deployment.

</div>

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Repository Structure](#-repository-structure)
- [Pipeline Stages](#-pipeline-stages)
  - [EC2 Setup & Maven Web App](#-ec2-setup--maven-web-app)
  - [Git & GitHub Integration](#-git--github-integration)
  - [AWS CodeArtifact](#-aws-codeartifact)
  - [AWS CodeBuild (CI)](#-aws-codebuild-ci)
  - [AWS CodeDeploy & CodePipeline (Studied)](#-aws-codedeploy--codepipeline-studied)
  - [GitHub Actions (CD Implementation)](#-github-actions-cd-implementation)
- [Live Application](#-live-application)
- [Getting Started Locally](#-getting-started-locally)
- [Key Configuration Files](#-key-configuration-files)
- [Deployment Scripts](#-deployment-scripts)
- [AWS Infrastructure](#-aws-infrastructure)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🚀 Project Overview

The **NextWork Web Project** is a Java-based web application (JSP/WAR) that serves as a complete, practical demonstration of cloud-native DevOps practices — from provisioning infrastructure to shipping code automatically with zero manual steps.

> **Note on CI/CD approach:** The challenge covered AWS CodeDeploy and CodePipeline for the deployment stage. Due to **restrictions on my AWS Free Tier plan**, I could not fully activate those services. Instead, I implemented continuous deployment using **GitHub Actions**, which achieves the same automated `git push → build → deploy` result at no additional cost.

The project covers every phase of the DevOps lifecycle:

| Phase | What Was Built |
|---|---|
| **Infrastructure** | EC2 instance provisioned, key-pair secured, VSCode Remote-SSH configured |
| **Version Control** | Git initialized, GitHub repository created, PAT authentication configured |
| **Dependency Security** | Private Maven package cache via AWS CodeArtifact, IAM roles scoped per service |
| **Continuous Integration** | Automated builds via AWS CodeBuild using `buildspec.yml` |
| **CD (Studied)** | AWS CodeDeploy & CodePipeline — studied & configured, limited by free-plan restrictions |
| **Continuous Delivery** | GitHub Actions workflow: build → SCP `.war` → SSH deploy to EC2, fully automated |

---

## 🏗️ Architecture

### Expected Architecture Diagram

![Expected Architecture](assets/the%20excpected%20architicure.png)

### Actual CI/CD Pipeline Flow (As Implemented)

```
Developer (git push to master)
        │
        ▼
┌──────────────────────────────────────────────────────────────┐
│                     GitHub Actions                            │
│                                                              │
│  ┌──────────────┐  ┌──────────────────┐  ┌───────────────┐  │
│  │   Checkout   │→│  Build (Maven)    │→ │  Deploy (SSH) │  │
│  │   Code       │  │  mvn clean pkg   │  │  SCP + SSH    │  │
│  └──────────────┘  └──────────────────┘  └───────────────┘  │
└──────────────────────────────────────────────────────────────┘
                    ↑ pulls deps via                  │
             AWS CodeArtifact                         ▼
                                              EC2 Instance
                                           (Tomcat auto-updated)
```

### Infrastructure Components

```
┌─────────────────────┐        SSH (port 22)        ┌──────────────────────────┐
│   Local Machine      │ ───────────────────────────▶│      AWS EC2 Instance     │
│   - VSCode           │        key pair (.pem)       │      - Amazon Linux      │
│   - Remote-SSH ext.  │                              │      - Java 8 (Corretto) │
│   - nextwork.pem     │◀────────────────────────────│      - Apache Tomcat     │
└─────────────────────┘        file sync / terminal  │      - HTTPD (port 80)   │
                                                      └──────────────────────────┘
```

---

## 💻 Tech Stack

| Category | Technology |
|---|---|
| **Language** | Java 8 (Amazon Corretto) |
| **Build Tool** | Apache Maven |
| **Web Framework** | JSP (JavaServer Pages) |
| **Artifact Format** | WAR (Web Application Archive) |
| **App Server** | Apache Tomcat 10 |
| **Web Server** | Apache HTTPD (reverse proxy) |
| **Source Control** | Git + GitHub |
| **Cloud Provider** | Amazon Web Services (AWS) |
| **Compute** | EC2 (t2.micro — Free Tier) |
| **CI Service** | AWS CodeBuild |
| **CD Service** | GitHub Actions (via SSH deploy) |
| **Artifact Repository** | AWS CodeArtifact |
| **IaC** | AWS CloudFormation |
| **Storage** | Amazon S3 |
| **IAM** | AWS IAM (Roles & Policies) |

---

## 📁 Repository Structure

```
nextwork-web-project/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions CI/CD workflow (actual CD)
├── assets/                     # Project screenshots and architecture diagrams (23 files)
├── scripts/
│   ├── install_dependencies.sh # Installs Tomcat + HTTPD on EC2
│   ├── start_server.sh         # Starts Tomcat + HTTPD
│   └── stop_server.sh          # Stops services gracefully
├── src/
│   └── main/
│       └── webapp/
│           ├── index.jsp        # Application entry point
│           └── WEB-INF/
│               └── web.xml
├── appspec.yml                  # AWS CodeDeploy deployment spec
├── buildspec.yml                # AWS CodeBuild build instructions
└── pom.xml                      # Maven project configuration
```

---

## 🔄 Pipeline Stages

### 🖥️ EC2 Setup & Maven Web App

**Goal:** Provision an EC2 instance, connect via VSCode Remote-SSH, and scaffold a Java Maven web app skeleton.

**AWS Services used:** EC2, Key Pair, Security Group

![EC2 Instance Creation](assets/instance%20creation.png)

**Key steps:**
1. Launch EC2 (`t2.micro`, Amazon Linux 2023)
2. Create `.pem` key pair and restrict permissions (`chmod 400`)
3. Configure Security Group — inbound rules: SSH:22, HTTP:80, Custom TCP:8080
4. Connect via VSCode Remote-SSH extension
5. Install Java 8 (Amazon Corretto) + Maven on the instance
6. Generate Maven webapp skeleton (`maven-archetype-webapp`)
7. Build with `mvn package` → produces `nextwork-web-project.war`

```bash
# Install dependencies on the EC2 instance
sudo yum install java-1.8.0-amazon-corretto -y
sudo yum install maven -y

# Verify installations
java -version
mvn -version

# Generate Maven webapp skeleton
mvn archetype:generate \
  -DgroupId=com.nextwork.app \
  -DartifactId=nextwork-web-project \
  -DarchetypeArtifactId=maven-archetype-webapp \
  -DinteractiveMode=false
```

![First SSH Connection](assets/first%20connect%20to%20the%20instance%20with%20ssh.png)

![Confirming Java Version](assets/confirming%20java's%20version.png)

![Downloading Apache Maven](assets/downloading%20the%20apache%20maven.png)

![Maven Build Success](assets/maven%20build%20success.png)

---

### 🐙 Git & GitHub Integration

**Goal:** Put the Maven project under Git version control and push it to GitHub — the source control layer that all pipeline stages trigger off.

**AWS Services used:** EC2

![Connecting AWS with GitHub](assets/connecting%20AWS%20with%20GitHub.png)

**Key steps:**
1. `git init` inside the project directory
2. Configure Git identity (`user.name`, `user.email`)
3. Stage all files and commit (`git add . && git commit`)
4. Create a GitHub Personal Access Token (PAT) — `repo` scope
5. Create remote GitHub repository (empty — no README/gitignore)
6. Link local repo to remote and push

```bash
git init
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git add .
git commit -m "Initial commit: nextwork-web-project"
git remote add origin https://github.com/<username>/nextwork-web-project.git
git branch -M main
git push -u origin main
```

![Connecting Instance to Repo](assets/connecting%20the%20instance%20with%20the%20repo.png)

![Content of the Repository](assets/content's%20folder%20for%20the%20repo.png)

---

### 📦 AWS CodeArtifact

**Goal:** Replace direct Maven Central access with a private, secured AWS CodeArtifact repository — protecting builds from public outages and adding a dependency security layer.

**AWS Services used:** CodeArtifact, IAM

**Package flow:**

```
Maven build → CodeArtifact repo → (on cache miss) → Maven Central → cached in CodeArtifact
```

**Key concepts:**

| Concept | What it means |
|---|---|
| **CodeArtifact Domain** | Top-level container grouping related repositories (`nextwork`) |
| **CodeArtifact Repository** | Actual package store — `nextwork-devops-cicd` |
| **Upstream repository** | Maven Central wired in as a fallback source |
| **Auth token** | Short-lived `CODEARTIFACT_AUTH_TOKEN` used by Maven to authenticate |
| **IAM Role** | EC2 instance role grants permission to request auth tokens without hardcoded credentials |

**Key steps:**
1. Create CodeArtifact domain (`nextwork`) and repository (`nextwork-devops-cicd`)
2. Enable Maven Central as upstream — packages cache on first pull
3. Create IAM role for EC2 with CodeArtifact read + auth permissions
4. Attach role to EC2 instance (instance profile)
5. Export auth token: `aws codeartifact get-authorization-token`
6. Configure Maven `settings.xml` — mirrors all requests through CodeArtifact
7. Verify: `mvn -s settings.xml compile` logs CodeArtifact URLs instead of Maven Central

```bash
# Export the auth token on the EC2 instance
export CODEARTIFACT_AUTH_TOKEN=$(aws codeartifact get-authorization-token \
  --domain nextwork \
  --domain-owner <your-account-id> \
  --query authorizationToken \
  --output text)

# Compile through CodeArtifact
mvn -s settings.xml compile
```

---

### 🏗️ AWS CodeBuild (CI)

**Goal:** Replace manual `mvn package` runs with an automated, managed build service — the continuous integration half of the pipeline.

**AWS Services used:** CodeBuild, S3, IAM

![Building Project in CodeBuild](assets/building%20project%20in%20codebuild.png)

**`buildspec.yml` — the build recipe:**

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto8

  build:
    commands:
      - echo Build started on `date`
      - mvn clean package

  post_build:
    commands:
      - echo Build completed on `date`

artifacts:
  files:
    - target/nextwork-web-project.war
    - appspec.yml
    - scripts/**/*
  discard-paths: no
```

**Key steps:**
1. Add `buildspec.yml` to repo root and push
2. Create CodeBuild project connected to GitHub repo (`master` branch)
3. Attach IAM policy granting CodeBuild's service role access to CodeArtifact
4. Configure S3 bucket as artifact output location
5. Start build → watch phases stream live → status: **Succeeded** ✅

![Phase Build in CodeBuild](assets/phase%20build%20in%20code%20build.png)

**Build artifact stored in S3:**

![S3 Bucket](assets/s3%20creation.png)

![Artifact in S3](assets/artifact%20in%20S3.png)

![Content of the Artifact](assets/the%20content%20of%20the%20artifact.png)

---

### 🚦 AWS CodeDeploy & CodePipeline (Studied)

**Goal:** Study and configure the AWS-native deployment services — CodeDeploy for EC2 deployment and CodePipeline for full pipeline orchestration.

> ⚠️ **Free-Plan Restriction:** Due to limitations on my AWS Free Tier plan, I was unable to fully activate AWS CodeDeploy and CodePipeline. I studied the configuration, wrote the required files (`appspec.yml`, deployment scripts), and understood the architecture — the actual continuous deployment was implemented using **GitHub Actions** (below).

**AWS Services studied:** CodeDeploy, CodePipeline, EC2, IAM, S3

**`appspec.yml` — the CodeDeploy deployment recipe:**

```yaml
version: 0.0
os: linux

files:
  - source: /target/nextwork-web-project.war
    destination: /usr/share/tomcat/webapps/

hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
```

**CodeDeploy deployment lifecycle:**

```
BeforeInstall → (file copy) → AfterInstall → ApplicationStart → ValidateService
```

**CodePipeline full orchestration (target architecture):**

```
git push
   │
   ▼
┌──────────────────────────────────────────────────────────────┐
│                    AWS CodePipeline                           │
│                                                               │
│  [SOURCE: GitHub] → [BUILD: CodeBuild] → [DEPLOY: CodeDeploy] │
└──────────────────────────────────────────────────────────────┘
                                                  │
                                                  ▼
                                           EC2 Instance
                                        (App auto-updated)
```

![Installing Dependencies in Scripts Folder](assets/installing%20dependenices%20in%20scripts%20folder.png)

![The Running Instance](assets/the%20running%20instance.png)

---

### ⚡ GitHub Actions (CD Implementation)

**Goal:** Implement fully automated continuous deployment using GitHub Actions — achieving the same `git push → build → deploy` automation without requiring paid AWS services.

**How it works:**

```
Developer (git push to master)
        │
        ▼
GitHub detects push → triggers workflow
        │
        ▼
┌──────────────────────────────────────────────────────────────────────┐
│                     GitHub Actions Runner (ubuntu-latest)            │
│                                                                      │
│  1. Checkout code                                                    │
│  2. Setup Java 8 (Corretto)                                          │
│  3. mvn clean package  →  produces nextwork-web-project.war          │
│  4. SCP: copy .war to EC2 /tmp directory                             │
│  5. SSH: stop Tomcat → replace WAR → start Tomcat                    │
└──────────────────────────────────────────────────────────────────────┘
        │
        ▼
EC2 Instance — new version live at http://<EC2-DNS>:80/
```

**Workflow file — `.github/workflows/deploy.yml`:**

```yaml
name: CI/CD Deploy to EC2

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: corretto
          java-version: 8

      - name: Build Project
        run: mvn clean package

      - name: Copy WAR to EC2
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          source: "target/*.war"
          target: "/tmp"

      - name: Deploy on EC2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            sudo systemctl stop tomcat10
            sudo rm -rf /var/lib/tomcat10/webapps/*
            sudo mv /tmp/target/*.war /var/lib/tomcat10/webapps/ROOT.war
            sudo systemctl start tomcat10
```

**Required GitHub repository secrets:**

| Secret | Description |
|---|---|
| `EC2_HOST` | EC2 public IPv4 DNS or IP address |
| `EC2_USER` | SSH username (`ec2-user` for Amazon Linux, `ubuntu` for Ubuntu) |
| `EC2_KEY` | Full contents of the `.pem` private key file |

**Successful deployment via GitHub Actions:**

![Successful Deployment via GitHub Actions](assets/successful%20deployment%20on%20github%20actions.png)

---

## 🌐 Live Application

After GitHub Actions completes all steps successfully, the web application is live and served by Apache Tomcat on the EC2 instance — accessible at:

```
http://<EC2-Public-IPv4-DNS>:80/
```

*(Apache HTTPD reverse-proxies port 80 → Tomcat on port 8080)*

![The App Working and Live](assets/the%20app%20working%20and%20live.png)

**Successful deployment confirmation:**

![Successful Deployment](assets/successful%20deployment.png)

---

## 🛠️ Getting Started Locally

### Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| [Java JDK](https://aws.amazon.com/corretto/) | 8 (Corretto) | Compile and run the app |
| [Apache Maven](https://maven.apache.org/) | 3.x | Build and dependency management |
| [Apache Tomcat](https://tomcat.apache.org/) | 9/10 | Servlet container for WAR deployment |
| [Git](https://git-scm.com/) | Any | Version control |

### Clone & Build

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/nextwork-web-project.git
cd nextwork-web-project

# 2. Build the project
mvn clean package

# 3. Deploy the WAR to Tomcat
cp target/nextwork-web-project.war /path/to/tomcat/webapps/

# 4. Start Tomcat and visit:
# http://localhost:8080/nextwork-web-project/
```

---

## 📄 Key Configuration Files

### `buildspec.yml` — CodeBuild Instructions

Tells AWS CodeBuild which runtime to use and what commands to run to produce the build artifact:

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto8

  build:
    commands:
      - echo Build started on `date`
      - mvn clean package

  post_build:
    commands:
      - echo Build completed on `date`

artifacts:
  files:
    - target/nextwork-web-project.war
    - appspec.yml
    - scripts/**/*
  discard-paths: no
```

### `appspec.yml` — CodeDeploy Deployment Recipe

Defines file placements and lifecycle hook scripts for AWS CodeDeploy (kept in the repo for completeness):

```yaml
version: 0.0
os: linux

files:
  - source: /target/nextwork-web-project.war
    destination: /usr/share/tomcat/webapps/

hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
```

### `pom.xml` — Maven Project Definition

```xml
<project>
  <groupId>com.nextwork.app</groupId>
  <artifactId>nextwork-web-project</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>nextwork-web-project Maven Webapp</name>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <finalName>nextwork-web-project</finalName>
  </build>
</project>
```

---

## 🔧 Deployment Scripts

Located in `scripts/` — referenced by `appspec.yml` hooks and used during EC2 setup:

### `install_dependencies.sh`

Installs Tomcat and Apache HTTPD, then configures HTTPD as a reverse proxy routing port 80 → Tomcat port 8080:

```bash
#!/bin/bash
sudo yum install tomcat -y
sudo yum -y install httpd
sudo cat <<EOF > /etc/httpd/conf.d/tomcat_manager.conf
<VirtualHost *:80>
  ServerAdmin root@localhost
  ServerName app.nextwork.com
  DefaultType text/html
  ProxyRequests off
  ProxyPreserveHost On
  ProxyPass / http://localhost:8080/nextwork-web-project/
  ProxyPassReverse / http://localhost:8080/nextwork-web-project/
</VirtualHost>
EOF
```

### `start_server.sh`

Starts Tomcat and Apache HTTPD:

```bash
#!/bin/bash
sudo service tomcat start
sudo service httpd start
```

### `stop_server.sh`

Stops services gracefully before re-deployment:

```bash
#!/bin/bash
isExistApp=$(pgrep tomcat)
if [[ -n "$isExistApp" ]]; then
    sudo service tomcat stop
fi
sudo service httpd stop
```

---

## ☁️ AWS Infrastructure

### CloudFormation Stack

The EC2 infrastructure was provisioned using AWS CloudFormation for repeatability and consistency:

![Creating Stack in CloudFormation](assets/creating%20stack%20in%20cloudformation%201.png)

![CloudFormation Stack](assets/stack%20in%20cloud%20formation.png)

![Stack Events](assets/events%20in%20the%20stack%20in%20cloudformation.png)

![Stack Resources](assets/recourses%20in%20the%20stack%20in%20cloudformation.png)

### IAM Roles & Permissions

The project uses scoped IAM roles following the principle of least privilege:

| Role | Assigned To | Permissions |
|---|---|---|
| `EC2-instance-nextwork-cicd` | EC2 Instance | `codeartifact:GetAuthorizationToken`, `ReadFromRepository`, `sts:GetServiceBearerToken` |
| `codebuild-nextwork-devops-cicd-service-role` | CodeBuild | CodeArtifact read, S3 write, CloudWatch Logs |

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add: description of change'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Built as part of the DEPI DevOps Track — NextWork Challenge**

*From a blank EC2 instance to a fully automated CI/CD pipeline.*

</div>
