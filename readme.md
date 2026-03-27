# 🎲 BoardGame App — Azure DevOps CI/CD Pipeline with AKS

[![CI/CD](https://img.shields.io/badge/CI%2FCD-Azure%20DevOps-0078D4?logo=azuredevops&logoColor=white)](https://dev.azure.com)
[![Docker](https://img.shields.io/badge/Docker-Hub-2496ED?logo=docker&logoColor=white)](https://hub.docker.com)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-AKS-326CE5?logo=kubernetes&logoColor=white)](https://azure.microsoft.com/en-in/products/kubernetes-service)
[![SonarQube](https://img.shields.io/badge/Code%20Quality-SonarQube-4E9BCD?logo=sonarqube&logoColor=white)](https://www.sonarsource.com/products/sonarqube/)
[![Trivy](https://img.shields.io/badge/Security-Trivy-1904DA?logo=aquasecurity&logoColor=white)](https://trivy.dev)
[![Java](https://img.shields.io/badge/Java-17-ED8B00?logo=openjdk&logoColor=white)](https://openjdk.org)
[![Maven](https://img.shields.io/badge/Build-Maven-C71A36?logo=apachemaven&logoColor=white)](https://maven.apache.org)

---

## 📋 Project Overview

A **production-grade, end-to-end DevSecOps pipeline** for a Java-based BoardGame web application.  
Every commit to the `main` branch automatically triggers a full CI/CD flow — building, testing, scanning for security vulnerabilities, containerizing, and deploying the application to **Azure Kubernetes Service (AKS)**.

> **Security is built in from the start (Shift-Left)** — SonarQube performs static code analysis and Trivy scans both the source filesystem and the Docker image before anything reaches production.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CI PIPELINE (Azure DevOps)                   │
│                                                                     │
│  Code Push ──► Maven Test ──► Maven Package ──► Publish Artifact   │
│       │                                                             │
│       ├──► SonarQube Analysis (Code Quality Gate)                  │
│       ├──► Trivy FS Scan      (Source Vulnerability Report)        │
│       ├──► Docker Build & Push (Docker Hub: allenaira/boardgame)   │
│       └──► Trivy Image Scan   (Container Vulnerability Report)     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                   CI Build Succeeds
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       CD PIPELINE (Release)                         │
│                                                                     │
│  Download Artifact ──► Install kubectl ──► kubectl apply           │
│                                               │                    │
│                                    AKS Cluster (Azure)             │
│                                    LoadBalancer External IP        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Source Control** | Azure Repos (Git) | Hosts project code and triggers CI |
| **Build** | Apache Maven 3 | Compile, test, package as `.jar` |
| **Code Quality** | SonarQube (Community) | SAST — bugs, vulnerabilities, code smells |
| **Security Scan** | Trivy | CVE scanning (filesystem + Docker image) |
| **Containerization** | Docker | Build OCI image from Dockerfile |
| **Registry** | Docker Hub | Store and serve published images |
| **CI/CD** | Azure DevOps Pipelines | Classic Build + Release pipelines |
| **Orchestration** | Azure Kubernetes Service (AKS) | Managed Kubernetes deployment |
| **Build Agent** | Self-hosted (AWS EC2) | Ubuntu t2.large running the pipeline |
| **Java Runtime** | OpenJDK 17 | Application runtime |

---

## 🚀 CI Pipeline Stages

| # | Stage | Task | Output |
|---|---|---|---|
| 1 | **Source Checkout** | Git pull from Azure Repos | Latest code on agent |
| 2 | **Maven Test** | `mvn test` | JUnit results published to Azure |
| 3 | **Maven Package** | `mvn package` | `.jar` artifact in `target/` |
| 4 | **Copy Artifacts** | Copy JAR + K8s manifest | Files in staging directory |
| 5 | **Publish Artifact** | Publish `drop` artifact | Artifact available for CD |
| 6 | **SonarQube Prep** | Configure scanner | Analysis config ready |
| 7 | **Code Analysis** | Run SonarQube scanner | Quality Gate: ✅ Passed |
| 8 | **Trivy FS Scan** | `trivy fs --format table` | `trivy-fs-report.html` |
| 9 | **Docker Build & Push** | `docker buildAndPush` | Image → Docker Hub |
| 10 | **Trivy Image Scan** | `trivy image` | `trivy-image-report.html` |

---

## 📦 CD Release Pipeline

| # | Stage | Detail |
|---|---|---|
| 1 | **Trigger** | Continuous Deployment — fires on every successful CI build |
| 2 | **Download Artifact** | Pulls `drop` from the CI build pipeline |
| 3 | **Install kubectl** | Kubectl tool installer task |
| 4 | **Deploy to AKS** | `kubectl apply -f deployment-service.yaml` |
| 5 | **App Live** | Accessible via AKS LoadBalancer external IP |

---

## ⚙️ Infrastructure Setup

### Self-Hosted Azure DevOps Agent (AWS EC2)

```bash
# 1. Update packages
sudo apt update

# 2. Download the Azure DevOps agent
wget https://download.agent.dev.azure.com/agent/4.270.0/vsts-agent-linux-x64-4.270.0.tar.gz

# 3. Create agent directory and extract
mkdir myagent && mv vsts-agent-linux-x64-4.270.0.tar.gz myagent/
cd myagent && tar -xvf vsts-agent-linux-x64-4.270.0.tar.gz

# 4. Install required tools
sudo apt install openjdk-17-jre-headless maven -y
sudo snap install trivy
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu

# 5. Configure and start the agent
./config.sh   # Enter org URL, PAT token, and pool name
./run.sh      # Start listening for pipeline jobs
```

### SonarQube (Docker)

```bash
docker run -d --name sonar -p 9000:9000 \
  mc1arke/sonarqube-with-community-branch-plugin
```

Access at: `http://<EC2-PUBLIC-IP>:9000` | Default: `admin / admin`

### AKS Cluster

| Parameter | Value |
|---|---|
| Cluster Name | `baordgame-eks` |
| Resource Group | `Baordgame` |
| Region | Asia Pacific (Central India) |
| Node Size | `Standard_D2s_v3` |
| Node Count | 2 (or 1 for cost saving) |
| K8s Version | 1.33.7 |

---

## 🔐 Service Connections Required

| Name | Type | Purpose |
|---|---|---|
| `son-cin` | SonarQube Server | Connect pipeline to SonarQube for code analysis |
| `doc=con` | Docker Registry | Push images to Docker Hub |
| `con-name` | Kubernetes (AKS) | Deploy manifests to AKS cluster |

---

## 📁 Project Structure

```
.
├── src/                          # Java source code
├── .mvn/                         # Maven wrapper configuration
├── Dockerfile                    # Container image definition
├── deployment-service.yaml       # Kubernetes Deployment + Service manifest
├── sonar-project.properties      # SonarQube analysis configuration
├── pom.xml                       # Maven project dependencies & build config
└── .gitignore
```

---

## 🔑 Key Pipeline Configuration

### Enable Classic Pipelines
> Organization Settings → Pipelines → Settings → Disable both "Disable classic pipeline" toggles

### Agent Pool Setup
1. Org Settings → Agent Pools → **Add Pool** → Self-hosted → Name: `mantasha`
2. Add user-defined capabilities: `java = true`, `maven = true`
3. Generate a **Personal Access Token (PAT)** with Full Access scope

### Continuous Integration Trigger
```yaml
# Automatically triggers on any push to main branch
trigger:
  branches:
    include:
      - main
```

---

## 📊 Results

| Metric | Result |
|---|---|
| Unit Tests | ✅ 100% Passed |
| SonarQube Quality Gate | ✅ Passed |
| Artifacts Published | ✅ 1 (drop) |
| Docker Image | ✅ Pushed to Docker Hub |
| AKS Pods | ✅ 2/2 Running |
| App Accessible | ✅ via LoadBalancer IP |

---

## 🧹 Cleanup

To avoid Azure charges after testing:

```bash
# Delete the AKS resource group (removes cluster, VMs, LBs, storage)
az group delete --name Baordgame --yes --no-wait

# Stop the EC2 agent instance from AWS Console
# Optionally remove Docker images from Docker Hub
```

---

## 📚 References

- **Reference Application**: [jaiswaladi246/Boardgame](https://github.com/jaiswaladi246/Boardgame/tree/main)
- **SonarQube Docker Image**: [mc1arke/sonarqube-with-community-branch-plugin](https://hub.docker.com/r/mc1arke/sonarqube-with-community-branch-plugin)
- **Azure DevOps Docs**: [docs.microsoft.com/azure/devops](https://docs.microsoft.com/en-us/azure/devops)
- **AKS Documentation**: [Azure Kubernetes Service](https://learn.microsoft.com/en-us/azure/aks/)
- **Trivy Docs**: [trivy.dev](https://trivy.dev)

---

## 👤 Author

**Ansari Mantasha**  
Azure DevOps | Kubernetes | DevSecOps  
[GitHub](https://github.com/mantu0tech) · [LinkedIn](https://linkedin.com)

---

> ⭐ *Star this repo if you found it helpful!*
