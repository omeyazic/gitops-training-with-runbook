# GitOps Pipeline - Complete Greenfield Deployment Guide

**Document Type:** Implementation Runbook  
**Target Audience:** DevOps Engineers, SREs, Platform Teams  
**Deployment Model:** Local Development Environment (Mac M1/M2, ARM64)  
**Estimated Time:** 6-8 hours (first-time deployment)  
**Difficulty Level:** Intermediate to Advanced

---

## 📋 Table of Contents

1. [Overview & Architecture](#overview--architecture)
2. [Prerequisites](#prerequisites)
3. [Phase 0: Environment Setup](#phase-0-environment-setup)
4. [Phase 1: Infrastructure Provisioning](#phase-1-infrastructure-provisioning)
5. [Phase 2: Configuration Management](#phase-2-configuration-management)
6. [Phase 3: Kubernetes Cluster](#phase-3-kubernetes-cluster)
7. [Phase 4: DevOps Services](#phase-4-devops-services)
   - Jenkins Installation
   - Argo CD Installation
   - Secret Management (HashiCorp Vault)
8. [Phase 5: GitOps Pipeline](#phase-5-gitops-pipeline)
9. [Phase 6: Validation & Testing](#phase-6-validation--testing)
10. [Phase 7: Observability & Monitoring](#phase-7-observability--monitoring)
    - Prometheus Installation
    - Grafana Installation
    - Dashboard Configuration
11. [Teardown & Cleanup](#teardown--cleanup)
12. [Troubleshooting](#troubleshooting)
13. [Future Enhancements](#future-enhancements)

---

## 🗺️ Your Learning Journey

```
Phase 0: Environment Setup (30 min)
   ↓     Setup tools, directories, Git
Phase 1: Infrastructure Provisioning (20 min)
   ↓     Create VMs with Terraform
Phase 2: Configuration Management (20 min)
   ↓     Prep systems for Kubernetes
Phase 3: Kubernetes Cluster (20 min)
   ↓     Deploy K3s cluster
Phase 4: DevOps Services (90 min)
   ↓     Install Jenkins, Argo CD & Vault
Phase 5: GitOps Pipeline (90 min)
   ↓     Build with Kaniko, deploy with Argo CD
Phase 6: Validation & Testing (30 min)
   ↓     Test production-grade workflow
Phase 7: Observability & Monitoring (45 min)
   ✓     Deploy Prometheus & Grafana, create dashboards
```

**Total Time:** ~6-8 hours  
**Outcome:** Production-ready local GitOps pipeline with security best practices

**🎓 What Makes This Production-Grade:**
- ✅ Rootless image builds with Kaniko (no Docker daemon)
- ✅ No privileged container access
- ✅ Kubernetes-native workflows
- ✅ Complete GitOps automation
- ✅ Centralized secret management with HashiCorp Vault
- ✅ Dynamic secret injection (no static credentials)
- ✅ Security best practices from day one

---

## 📝 Configuration Variables

**IMPORTANT:** Before starting this guide, customize these variables using find & replace in your text editor.

### How to Customize This Guide

1. **Create Your Working Guide:** Copy this document to a new file (e.g., `gitops-pipeline-runbook-yourname.md`)
2. **Replace Variables:** Use your editor's Find & Replace feature:
   - Find: `{{VARIABLE_NAME}}`
   - Replace: Your custom value
3. **Verify:** Search for `{{` to ensure no **UPPERCASE** placeholders remain

From this point forward, follow YOUR working guide with actual values (not this template).

**⚠️ IMPORTANT - Don't Replace Ansible Variables:**

This guide contains two types of `{{...}}` syntax:

| Type | Format | Action | Examples |
|------|--------|--------|----------|
| **Configuration Placeholders** | `{{UPPERCASE_NAME}}` | ✅ **REPLACE these** | `{{PROJECT_NAME}}`, `{{GITHUB_USERNAME}}` |
| **Ansible/Jinja2 Variables** | `{{lowercase_name}}` | ❌ **DON'T replace** | `{{node_ip}}`, `{{k3s_server_ip}}` |

**Rule of Thumb:** 
- **UPPERCASE** in double braces = Configuration placeholder → Replace it
- **lowercase** in double braces = Ansible template variable → Leave it alone

### Variable Reference Table

| Category | Variable Name | Default Value | Description | Example | Variable Used for this project |
|----------|----------|---------------|-------------|---------|---------|
| **Project Names** | | | | |
| | `PROJECT_NAME` | `mygitopspipeline` | Main project/repo name | `my-devops-lab` | `{{PROJECT_NAME}}` |
| | `APP_NAME` | `hello-gitops` | Application name | `flask-demo` | `{{APP_NAME}}` |
| **Network** | | | | | |
| | `CONTROL_PLANE_IP` | `192.168.2.2` | Control plane VM IP | `192.168.64.10` | `{{CONTROL_PLANE_IP}}` |
| | `WORKER_NODE_IP` | `192.168.2.3` | Worker node VM IP | `192.168.64.11` | `{{WORKER_NODE_IP}}`  |
| | `ANSIBLE_CONTROL_IP` | `192.168.2.4` | Ansible inventory control IP | `192.168.64.10` |`{{ANSIBLE_CONTROL_IP}}` |
| | `ANSIBLE_WORKER_IP` | `192.168.2.5` | Ansible inventory worker IP | `192.168.64.11` |`{{ANSIBLE_WORKER_IP}}` |
| **Service Ports** | | | | | |
| | `JENKINS_HTTP_PORT` | `30080` | Jenkins web interface | `30080` |`{{JENKINS_HTTP_PORT}}` |
| | `JENKINS_AGENT_PORT` | `30081` | Jenkins agent communication | `30081` | `{{JENKINS_AGENT_PORT}}` |
| | `ARGOCD_HTTP_PORT` | `30083` | Argo CD HTTP access | `30083` | `{{ARGOCD_HTTP_PORT}}` |
| | `ARGOCD_HTTPS_PORT` | `30082` | Argo CD HTTPS access | `30082` | `{{ARGOCD_HTTPS_PORT}}` |
| | `APP_HTTP_PORT` | `30084` | Application web access | `30085` | `{{APP_HTTP_PORT}}` |
| | `PROMETHEUS_HTTP_PORT` | `30090` | Prometheus web interface | `30090` | `{{PROMETHEUS_HTTP_PORT}}` |
| | `GRAFANA_HTTP_PORT` | `30300` | Grafana dashboards | `30300` | `{{GRAFANA_HTTP_PORT}}` |
| **Infrastructure** | | | | | |
| | `CONTROL_PLANE_HOSTNAME` | `k3s-control-01` | Control plane hostname | `k3s-master-01` | `{{CONTROL_PLANE_HOSTNAME}}` |
| | `WORKER_HOSTNAME` | `k3s-worker-01` | Worker node hostname | `k3s-node-01` | `{{WORKER_HOSTNAME}}` |
| | `K3S_VERSION` | `v1.28.5+k3s1` | K3s Kubernetes version | `v1.28.5+k3s1` | `{{K3S_VERSION}}` |
| | `K3S_TOKEN` | `my-secret-token` | K3s cluster join token | `prod-k3s-secret-2024` | `{{K3S_TOKEN}}` |
| **User Credentials** | | | | | |
| | `GITHUB_USERNAME` | `YOUR_GITHUB_USERNAME` | Your GitHub username | `johndoe` |`{{GITHUB_USERNAME}}` |
| | `DOCKERHUB_USERNAME` | `YOUR_DOCKERHUB_USERNAME` | Your Docker Hub username | `johndoe` | `{{DOCKERHUB_USERNAME}}` |
| | `LOCAL_USERNAME` | `yourusername` | Your Mac username | `john` | `{{LOCAL_USERNAME}}` |
| **Kubernetes Namespaces** | | | | | |
| | `JENKINS_NAMESPACE` | `jenkins` | Jenkins K8s namespace | `ci` | `{{JENKINS_NAMESPACE}}` |
| | `ARGOCD_NAMESPACE` | `argocd` | Argo CD K8s namespace, use "argocd" for namespace | `cd` | `{{ARGOCD_NAMESPACE}}` |
| | `APP_NAMESPACE` | `hello-gitops` | Application K8s namespace | `flask-demo` | `{{APP_NAMESPACE}}` |
| **Docker Images** | | | | | |
| | `JENKINS_IMAGE_NAME` | `jenkins-docker-kubectl` | Custom Jenkins image name | `jenkins-custom` | `{{JENKINS_IMAGE_NAME}}` |
| | `APP_IMAGE_NAME` | `hello-gitops` | Application image name | `flask-demo` | `{{APP_IMAGE_NAME}}` |
| **File Paths** | | | | | |
| | `PROJECT_BASE_PATH` | `~/Desktop/mygitopspipeline` | Project directory path | `~/projects/gitops-lab` | `{{PROJECT_BASE_PATH}}` |

### Port Allocation Strategy

**NodePort Range:** 30000-32767 (Kubernetes standard)

**Default Port Mapping:**
- **30080**: Jenkins HTTP (web interface)
- **30081**: Jenkins Agent (build agents)
- **30082**: Argo CD HTTPS (secure web interface)
- **30083**: Argo CD HTTP (web interface) - **RECOMMENDED**
- **30084**: Application HTTP (demo app)
- **30090**: Prometheus (metrics & queries)
- **30300**: Grafana (dashboards)

### Quick Start: Minimal Required Changes

If you want to get started quickly, you MUST at minimum replace:
1. `{{GITHUB_USERNAME}}` - Your actual GitHub username
2. `{{DOCKERHUB_USERNAME}}` - Your actual Docker Hub username
3. `{{LOCAL_USERNAME}}` - Your Mac username (check with `whoami` command)

**⚠️ IMPORTANT - DO NOT Replace IP Addresses Yet:**
- `{{CONTROL_PLANE_IP}}`, `{{WORKER_NODE_IP}}`, `{{ANSIBLE_CONTROL_IP}}`, `{{ANSIBLE_WORKER_IP}}`
- These IPs are **dynamically assigned** by Multipass when VMs are created
- You will discover and update these in **Phase 1.5** (after Terraform creates the VMs)
- Replacing them now will cause connection failures!

All other variables can use defaults for initial deployment.

---

## Overview & Architecture

### What You'll Build

A complete local GitOps pipeline demonstrating enterprise-grade CI/CD practices:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Developer Workstation                                │
│                         (Mac M1/M2)                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────┐          ┌──────────────┐                             │
│  │   Terraform  │──────────│   Multipass  │                             │
│  │  (IaC Tool)  │          │  (VM Engine) │                             │
│  └──────────────┘          └──────┬───────┘                             │
│                                    │                                    │
│  ┌─────────────────────────────────┼─────────────────────┐              │
│  │         Virtual Machines        │                     │              │
│  │  ┌──────────────────────────────▼──────────────────┐  │              │
│  │  │{{CONTROL_PLANE_HOSTNAME}} ({{CONTROL_PLANE_IP}})│  │              │
│  │  │  ┌────────────┐  ┌──────────┐  ┌──────────┐     │  │              │
│  │  │  │   Jenkins  │  │ Argo CD  │  │   K3s    │     │  │              │
│  │  │  │  (CI)      │  │  (CD)    │  │ Control  │     │  │              │
│  │  │  └────────────┘  └──────────┘  └──────────┘     │  │              │
│  │  └─────────────────────────────────────────────────┘  │              │
│  │                                                       │              │
│  │  ┌──────────────────────────────────────────────┐     │              │
│  │  │    {{WORKER_HOSTNAME}} ({{WORKER_NODE_IP}})  │     │              │
│  │  │  ┌────────────┐  ┌──────────┐  ┌──────────┐  │     │              │
│  │  │  │  {{APP_NAME}}  │  │   K3s    │  │  Docker  │  │     │              │
│  │  │  │   (App)    │  │  Agent   │  │          │  │     │              │
│  │  │  └────────────┘  └──────────┘  └──────────┘  │     │              │
│  │  └──────────────────────────────────────────────┘     │              │
│  └───────────────────────────────────────────────────────┘              │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │              Git Repositories (GitHub)                           │   │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌─────────────────┐ │   │
│  │  │{{PROJECT_NAME}}  │  │{{PROJECT_NAME}}- │  │{{PROJECT_NAME}}-│ │   │
│  │  │(Infrastructure)  |  |{{APP_NAME}} (App)    │  │ manifests       │ │   │
│  │  └──────────────────┘  └──────────────────┘  └─────────────────┘ │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```  

### GitOps Workflow

```
Developer Commits → Jenkins CI → Kaniko Build → Image Push → Manifest Update → 
Argo CD Sync → Kubernetes Deployment → Application Running
```

**Production-Grade Workflow:**
- Jenkins spawns ephemeral Kubernetes pods for each build
- Kaniko builds images without Docker daemon (rootless, secure)
- No privileged access required at any stage
- Complete audit trail via Kubernetes events

### Technology Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Infrastructure** | Terraform + Multipass | 1.6+ / 1.14+ | VM provisioning |
| **Configuration** | Ansible | 2.15+ | System configuration |
| **Orchestration** | K3s | v1.28.5 | Lightweight Kubernetes |
| **Container Runtime** | containerd | (bundled with K3s) | Container execution |
| **Image Builder** | Kaniko | 1.9.0+ | Rootless image builds |
| **CI Tool** | Jenkins | 2.528.3 (LTS) | Continuous Integration |
| **CD Tool** | Argo CD | 2.x | Continuous Deployment |
| **Application** | Flask (Python) | 3.0.0 | Demo web application |
| **Version Control** | Git + GitHub | - | Source control |

---

## Prerequisites

### Hardware Requirements

- **Mac with Apple Silicon** (M1/M2/M3)
- **Minimum:** 8GB RAM, 50GB free disk space
- **Recommended:** 16GB RAM, 100GB free disk space

### Required Accounts

1. **GitHub Account**
   - Create at: https://github.com/join
   - Generate Personal Access Token (PAT):
     - Go to: Settings → Developer settings → Personal access tokens → Tokens (classic)
     - Click "Generate new token (classic)"
     - Scopes: `repo` (all), `workflow`, `write:packages`, `delete:packages`
     - Save token securely (you'll need it multiple times)

2. **Docker Hub Account**
   - Create at: https://hub.docker.com/signup
   - Generate Access Token:
     - Go to: Account Settings → Security → Access Tokens
     - Click "Generate New Token"
     - Token Description: `GitOps Pipeline`
     - Access Permissions: `Read, Write, Delete`
     - Click "Generate"
     - **Copy and save token immediately** (shown only once)
     - Use this token as password in Jenkins and kubectl

### Software to Install

| Tool | Installation Command | Verification |
|------|---------------------|--------------|
| Homebrew | `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` | `brew --version` |
| Multipass | `brew install multipass` | `multipass version` |
| Terraform | `brew install terraform` | `terraform version` |
| Ansible | `brew install ansible` | `ansible --version` |
| kubectl | `brew install kubectl` | `kubectl version --client` |
| Git | `brew install git` | `git --version` |
| Python 3 | `brew install python@3.11` | `python3 --version` |
| Helm | `brew install helm` | `helm version` |

### Pre-Flight Checklist

```bash
# Run this script to verify all prerequisites
echo "=== Prerequisite Check ==="

# Check Homebrew
command -v brew >/dev/null 2>&1 && echo "✅ Homebrew: $(brew --version | head -1)" || echo "❌ Homebrew: NOT INSTALLED"

# Check Multipass
command -v multipass >/dev/null 2>&1 && echo "✅ Multipass: $(multipass version | head -1)" || echo "❌ Multipass: NOT INSTALLED"

# Check Terraform
command -v terraform >/dev/null 2>&1 && echo "✅ Terraform: $(terraform version | head -1)" || echo "❌ Terraform: NOT INSTALLED"

# Check Ansible
command -v ansible >/dev/null 2>&1 && echo "✅ Ansible: $(ansible --version | head -1)" || echo "❌ Ansible: NOT INSTALLED"

# Check kubectl
command -v kubectl >/dev/null 2>&1 && echo "✅ kubectl: $(kubectl version --client -o yaml | grep gitVersion | head -1)" || echo "❌ kubectl: NOT INSTALLED"

# Check Git
command -v git >/dev/null 2>&1 && echo "✅ Git: $(git --version)" || echo "❌ Git: NOT INSTALLED"

# Check Python
command -v python3 >/dev/null 2>&1 && echo "✅ Python: $(python3 --version)" || echo "❌ Python: NOT INSTALLED"

# Check Helm
command -v helm >/dev/null 2>&1 && echo "✅ Helm: $(helm version --short)" || echo "❌ Helm: NOT INSTALLED"

# Check disk space
echo "💾 Disk Space Available: $(df -h ~ | awk 'NR==2 {print $4}')"

# Check RAM
echo "🧠 Total RAM: $(sysctl -n hw.memsize | awk '{print $1/1024/1024/1024 " GB"}')"

echo "=== Check Complete ==="
```

**Expected Result:** All items show ✅, disk space > 50GB, RAM ≥ 8GB

---

## Phase 0: Environment Setup

```
┌─────────────────────────────────────────────────────────┐
│ PHASE 0: ENVIRONMENT SETUP ◄── YOU ARE HERE            │
├─────────────────────────────────────────────────────────┤
│ [✓] Install Tools → [◄] Setup Dirs → [ ] Init Git      │
└─────────────────────────────────────────────────────────┘
     ↓
  Phase 1: Infrastructure → Phase 2: Config → ... → Done!
```

**Objective:** Prepare your local development environment  
**Time:** 30 minutes  
**Prerequisites:** All software installed

### Step 0.1: Create Project Directory Structure

```bash
# Create main project directory
mkdir -p {{PROJECT_BASE_PATH}}
cd {{PROJECT_BASE_PATH}}

# Create subdirectories
mkdir -p terraform ansible/{inventory,playbooks,roles} kubernetes/{jenkins,argocd,test-apps} scripts docs

# Verify structure
tree -L 2 {{PROJECT_BASE_PATH}}
```

**Expected Output:**
```
/Users/{{LOCAL_USERNAME}}/Desktop/{{PROJECT_NAME}}
├── ansible
│   ├── inventory
│   ├── playbooks
│   └── roles
├── docs
├── kubernetes
│   ├── argocd
│   ├── jenkins
│   └── test-apps
├── scripts
└── terraform
```

### Step 0.2: Initialize Git Repository

```bash
cd {{PROJECT_BASE_PATH}}

# Initialize git
git init

# Create .gitignore
cat > .gitignore << 'EOF'
# Terraform
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl
*.tfvars
!example.tfvars

# Ansible
*.retry
.vault_pass

# SSH Keys
*.pem
*.key
id_rsa*

# OS Files
.DS_Store
*.swp
*.swo

# IDE
.idea/
.vscode/
*.code-workspace

# Logs
*.log

# Secrets
secrets/
.env

# Kubeconfig files
kubeconfig
kubeconfig.*
*.kubeconfig

# Kubernetes binaries
kubernetes/jenkins/kubectl
kubernetes/jenkins/Dockerfile.bak

.gitignore

# If you use Agentic coding tools - modify accordingly - currently set for Cursor as an example
.cursor/
.ssh/
.cursor.ssh
EOF

# Initial commit
git add .
git commit -m "init: Initialize project structure"
```

### Step 0.3: Create GitHub Repository

```bash
# Option 1: Via GitHub CLI (if installed)
gh repo create {{PROJECT_NAME}} --public --source=. --remote=origin

# Option 2: Manually
# 1. Go to https://github.com/new
# 2. Repository name: {{PROJECT_NAME}}
# 3. Public
# 4. Do NOT initialize with README
# 5. Create repository
# 6. Follow instructions to push existing repository
```

```bash
# Link local to remote
git remote set-url origin https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}.git
git branch -M main
git push -u origin main
```

**Validation:**
```bash
git remote -v
# Should show: origin  https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}.git
```

### Step 0.4: Configure Git Identity

```bash
# Set your identity (if not already set)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Verify
git config --global --list | grep user
```

### Step 0.5: Validate Configuration Variables

**⚠️ Final Checkpoint - Run This Before Proceeding:**

```bash
# Verify your customizations in YOUR working guide
echo "=== Configuration Validation ==="
echo "Project Name: {{PROJECT_NAME}}"
echo "GitHub Username: {{GITHUB_USERNAME}}"
echo "Docker Hub Username: {{DOCKERHUB_USERNAME}}"
echo "Control Plane IP: {{CONTROL_PLANE_IP}}"
echo "Worker Node IP: {{WORKER_NODE_IP}}"
echo "Jenkins URL: http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}}"
echo "Argo CD URL: http://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTP_PORT}}"
echo "Application URL: http://{{CONTROL_PLANE_IP}}:{{APP_HTTP_PORT}}"
echo "Prometheus URL: http://{{CONTROL_PLANE_IP}}:{{PROMETHEUS_HTTP_PORT}}"
echo "Grafana URL: http://{{CONTROL_PLANE_IP}}:{{GRAFANA_HTTP_PORT}}"
echo "==========================="
```

**What to Check:**

✅ **Must show actual values:** `{{PROJECT_NAME}}`, `{{GITHUB_USERNAME}}`, `{{DOCKERHUB_USERNAME}}`  
✅ **OK to see placeholders:** `{{CONTROL_PLANE_IP}}`, `{{WORKER_NODE_IP}}` (updated in Phase 1.6)  
❌ **Still see {{ PLACEHOLDERS }}?** Go back and complete find & replace in Configuration Variables section

**ℹ️ Note:** Commands may show `{{lowercase_variables}}` - these are Ansible template variables. Only `{{UPPERCASE}}` should be replaced.

---

## Phase 1: Infrastructure Provisioning

```
┌─────────────────────────────────────────────────────────┐
│ PHASE 1: INFRASTRUCTURE PROVISIONING ◄── YOU ARE HERE  │
├─────────────────────────────────────────────────────────┤
│  Mac (Host)                                             │
│    ↓ Terraform + Multipass                             │
│  ┌──────────────┐        ┌──────────────┐              │
│  │ Control VM   │        │ Worker VM    │ ← Creating   │
│  │ (2CPU, 3GB)  │        │ (2CPU, 3GB)  │              │
│  └──────────────┘        └──────────────┘              │
└─────────────────────────────────────────────────────────┘
```

**Objective:** Create virtual machines using Terraform and Multipass  
**Time:** 20 minutes  
**Resources:** 2 VMs (1 control plane, 1 worker)

### Step 1.1: Create Terraform Configuration

**⚠️ Terraform Provider Notice:**

The `larstobi/multipass` provider is a community-maintained provider that may have compatibility issues with:
- Newer versions of Multipass (1.13+)
- Apple Silicon (M1/M2/M3) chips
- macOS Ventura/Sonoma

**If Terraform fails:**
1. Check provider compatibility: `multipass version` vs provider version
2. Alternative: Use a shell script with `multipass launch` commands (simpler but less declarative)
3. See "Troubleshooting" section for manual VM creation

**For production:** Consider using native cloud providers (AWS, Azure, GCP) or bare metal with proper Terraform providers.

Create the main Terraform file:

*This main Terraform file defines your infrastructure as code, specifying the provider, resources, and variables needed to provision consistent, repeatable virtual machine environments. It works by declaring the desired state of your infrastructure, which Terraform then plans and applies to create or modify resources in your chosen cloud platform.*

```bash
cd {{PROJECT_BASE_PATH}}/terraform

cat > main.tf << 'EOF'
terraform {
  required_providers {
    multipass = {
      source  = "larstobi/multipass"
      version = "~> 1.4.2"
    }
  }
}

provider "multipass" {}

# Control Plane Node
resource "multipass_instance" "k3s_control" {
  name   = "{{CONTROL_PLANE_HOSTNAME}}"
  cpus   = 2
  memory = "3G"
  disk   = "10G"
  image  = "22.04"

  cloudinit_file = "${path.module}/cloud-init-control.yaml"
}

# Worker Node
resource "multipass_instance" "k3s_worker" {
  name   = "{{WORKER_HOSTNAME}}"
  cpus   = 2
  memory = "3G"
  disk   = "10G"
  image  = "22.04"

  cloudinit_file = "${path.module}/cloud-init-worker.yaml"

  depends_on = [multipass_instance.k3s_control]
}
EOF
```

Create outputs configuration:

*The outputs configuration exposes key information from your Terraform deployment, such as VM IP addresses, making critical values easily accessible for subsequent automation steps or for user reference. When Terraform applies the infrastructure, it calculates and displays these defined outputs, which can also be consumed by other tools or modules in your pipeline.*

```bash
cat > outputs.tf << 'EOF'
output "control_plane_ip" {
  description = "IP address of control plane node"
  value       = multipass_instance.k3s_control.ipv4
}

output "worker_ip" {
  description = "IP address of worker node"
  value       = multipass_instance.k3s_worker.ipv4
}

output "connection_info" {
  description = "SSH connection information"
  value = {
    control_plane = "multipass shell ${multipass_instance.k3s_control.name}"
    worker        = "multipass shell ${multipass_instance.k3s_worker.name}"
  }
}
EOF
```

### Step 1.2: Create Cloud-Init Files

Control plane cloud-init:

*This cloud-init configuration for the control plane node automatically performs initial system setup, including user creation, security hardening, and package installation, upon the virtual machine's first boot. It works by providing the cloud-init metadata service with a YAML script that executes a standardized set of commands to prepare a consistent and secure environment ready for K3s installation.*

```bash
cat > cloud-init-control.yaml << 'EOF'
#cloud-config
hostname: {{CONTROL_PLANE_HOSTNAME}}
manage_etc_hosts: true

users:
  - default
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa REPLACE_WITH_YOUR_PUBLIC_KEY

package_update: true
package_upgrade: true

packages:
  - curl
  - wget
  - vim
  - net-tools
  - software-properties-common

runcmd:
  - echo "Setup complete" >> /var/log/cloud-init-done.log
EOF
```

Worker cloud-init:

*This worker node cloud-init configuration automates the baseline provisioning steps, such as setting the hostname and installing necessary packages, ensuring each node meets the prerequisites for joining the K3s cluster. It works by running the defined set of initialization commands during the VM's first boot, creating a uniform environment that aligns with the cluster's security and operational requirements.*

```bash
cat > cloud-init-worker.yaml << 'EOF'
#cloud-config
hostname: {{WORKER_HOSTNAME}}
manage_etc_hosts: true

users:
  - default
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa REPLACE_WITH_YOUR_PUBLIC_KEY

package_update: true
package_upgrade: true

packages:
  - curl
  - wget
  - vim
  - net-tools
  - software-properties-common

runcmd:
  - echo "Setup complete" >> /var/log/cloud-init-done.log
EOF
```

### Step 1.3: Generate SSH Key (if needed)

*This step generates a new SSH key pair on your local machine to establish secure, password-less authentication between your workstation and the newly provisioned virtual machines. It works by creating a private key, which you keep secure, and a public key that you add to the cloud-init configuration, allowing Terraform to embed it for authorized access during VM creation.*

```bash
# Check if you have an SSH key
ls ~/.ssh/id_rsa.pub

# If not, generate one
ssh-keygen -t rsa -b 4096 -C "your.email@example.com" -f ~/.ssh/id_rsa -N ""

# Display your public key
cat ~/.ssh/id_rsa.pub
```

**Copy the output and replace `REPLACE_WITH_YOUR_PUBLIC_KEY` in both cloud-init files.**

### Step 1.4: Initialize and Apply Terraform

```bash
cd {{PROJECT_BASE_PATH}}/terraform

# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Plan deployment
terraform plan

# Apply (provision VMs)
terraform apply -auto-approve
```

**Expected Output:**
```
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

connection_info = {
  "control_plane" = "multipass shell {{CONTROL_PLANE_HOSTNAME}}"
  "worker" = "multipass shell {{WORKER_HOSTNAME}}"
}
control_plane_ip = "{{CONTROL_PLANE_IP}}"
worker_ip = "{{WORKER_NODE_IP}}"
```

**⏱️ Wait Time:** 2-3 minutes for VM creation

### Step 1.5: Verify VMs

```bash
# List VMs
multipass list
```

**Expected Output:**
```
Name                    State             IPv4             Image
{{CONTROL_PLANE_HOSTNAME}}          Running           {{CONTROL_PLANE_IP}}      Ubuntu 22.04 LTS
{{WORKER_HOSTNAME}}           Running           {{WORKER_NODE_IP}}      Ubuntu 22.04 LTS
```

```bash
# Test SSH connectivity
multipass exec {{CONTROL_PLANE_HOSTNAME}} -- hostname
# Expected: {{CONTROL_PLANE_HOSTNAME}}

multipass exec {{WORKER_HOSTNAME}} -- hostname
# Expected: {{WORKER_HOSTNAME}}
```

### Step 1.6: Discover and Update IP Addresses

**⚠️ CRITICAL STEP:** Multipass assigns IPs dynamically. You must now discover the actual IPs and update your configuration.

```bash
# Get actual IP addresses
multipass list
```

**Expected Output:**
```
Name                    State             IPv4             Image
{{CONTROL_PLANE_HOSTNAME}}          Running           192.168.64.2      Ubuntu 22.04 LTS
{{WORKER_HOSTNAME}}           Running           192.168.64.3      Ubuntu 22.04 LTS
```

**Note the actual IP addresses** - they will likely be `192.168.64.x` on Mac (NOT the `192.168.2.x` placeholders).

**Now update your guide with the actual IPs:**

```bash
# Store the IPs for easy reference
ACTUAL_CONTROL_IP=$(multipass list | grep {{CONTROL_PLANE_HOSTNAME}} | awk '{print $3}')
ACTUAL_WORKER_IP=$(multipass list | grep {{WORKER_HOSTNAME}} | awk '{print $3}')

echo "Control Plane IP: $ACTUAL_CONTROL_IP"
echo "Worker Node IP: $ACTUAL_WORKER_IP"
```

**Now update your guide with the actual IPs from this point onwards as instructed below - no need to modify previous occurrences:** 

**Action Required:**
1. Open your working guide in your text editor
2. Use Find & Replace to update IP addresses:
   - Find: `{{CONTROL_PLANE_IP}}` → Replace with actual IP (e.g., `192.168.64.2`)
   - Find: `{{WORKER_NODE_IP}}` → Replace with actual IP (e.g., `192.168.64.3`)
   - Find: `{{ANSIBLE_CONTROL_IP}}` → Replace with actual control plane IP (same as above)
   - Find: `{{ANSIBLE_WORKER_IP}}` → Replace with actual worker IP (same as above)
3. Save the file
4. Continue following your working guide with actual IPs

**Verification:**
```bash
# Test connectivity
ping -c 2 $ACTUAL_CONTROL_IP
ping -c 2 $ACTUAL_WORKER_IP
```

**Expected:** Both IPs respond to ping

### Step 1.7: Commit Infrastructure Code

```bash
cd {{PROJECT_BASE_PATH}}

git add terraform/
git commit -m "feat(terraform): Add VM provisioning with Multipass"
git push origin main
```

---

## Phase 2: Configuration Management

```
┌─────────────────────────────────────────────────────────┐
│ PHASE 2: CONFIGURATION MANAGEMENT ◄── YOU ARE HERE     │
├─────────────────────────────────────────────────────────┤
│  Mac (Ansible Controller)                              │
│    ↓ SSH Commands                                      │
│  ┌──────────────┐        ┌──────────────┐              │
│  │ Control VM   │        │ Worker VM    │              │
│  │ + System Cfg │        │ + System Cfg │ Installing  │
│  │ + Prep K8s   │        │ + Prep K8s   │              │
│  └──────────────┘        └──────────────┘              │
└─────────────────────────────────────────────────────────┘
```

**Objective:** Configure VMs using Ansible  
**Time:** 20 minutes  
**Tasks:** SSH setup, system prep for Kubernetes

**📝 Before You Start:**
- Ensure you completed **Phase 1.6** and updated IP addresses in your guide
- All commands in this phase assume you've replaced variables with actual values
- If copying from this guide, verify IP addresses match your VMs (`multipass list` to confirm)

**🎓 Production Approach:**
- We will NOT install Docker daemon on the VMs
- K3s uses containerd as its container runtime
- Jenkins will build images using Kaniko (rootless, daemonless)
- This follows production security best practices

### Step 2.1: Create Ansible Inventory

```bash
cd {{PROJECT_BASE_PATH}}/ansible

# Create inventory file

*This Ansible inventory file organizes your cluster nodes into logical groups, defining connection details like IP addresses and usernames so automation tasks can target specific sets of servers. It works by providing a structured YAML map that Ansible uses to establish SSH connections and execute playbooks across the control plane and worker nodes without manual host specification.*

```bash
cat > inventory/hosts.yml << 'EOF'
all:
  children:
    k3s_cluster:
      children:
        control_plane:
          hosts:
            {{CONTROL_PLANE_HOSTNAME}}:
              ansible_host: {{ANSIBLE_CONTROL_IP}}
              ansible_user: ubuntu
              ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
              node_ip: {{ANSIBLE_CONTROL_IP}}
        workers:
          hosts:
            {{WORKER_HOSTNAME}}:
              ansible_host: {{ANSIBLE_WORKER_IP}}
              ansible_user: ubuntu
              ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
              node_ip: {{ANSIBLE_WORKER_IP}}
      vars:
        ansible_python_interpreter: /usr/bin/python3
EOF
```

### Step 2.2: Create Ansible Configuration

*This Ansible configuration file sets essential runtime behaviors like disabling host key checking and defining the inventory path, streamlining playbook execution by reducing manual command-line arguments. It works by creating an ansible.cfg file in your project directory, whose settings automatically apply whenever you run Ansible commands within that location.*

```bash
cat > ansible.cfg << 'EOF'
[defaults]
inventory = inventory/hosts.yml
host_key_checking = False
timeout = 30
retry_files_enabled = False

[ssh_connection]
pipelining = True
EOF
```

### Step 2.3: Test Ansible Connectivity

```bash
cd {{PROJECT_BASE_PATH}}

# Ping all hosts
ansible k3s_cluster -i ansible/inventory/hosts.yml -m ping
```

**Expected Output:**
```
{{CONTROL_PLANE_HOSTNAME}} | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
{{WORKER_HOSTNAME}} | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

**If ping fails:** Check SSH keys, IP addresses, and VM status

### Step 2.4: Create Preflight Checks Playbook

*This preflight checks playbook validates system prerequisites across all cluster nodes, such as memory, disk space, and network connectivity, ensuring a consistent foundation before installing Kubernetes. It works by executing a series of Ansible tasks that gather system facts and perform conditional checks, reporting any failures that could hinder a successful cluster deployment*

```bash
cd {{PROJECT_BASE_PATH}}/ansible/playbooks

cat > preflight-checks.yml << 'EOF'
---
- name: Preflight checks for K3s nodes
  hosts: all
  become: yes
  tasks:
    - name: Check OS version
      command: lsb_release -d
      register: os_version
      changed_when: false

    - name: Display OS version
      debug:
        msg: "{{ os_version.stdout }}"

    - name: Check available memory
      shell: free -h | grep Mem | awk '{print $2}'
      register: total_memory
      changed_when: false

    - name: Display memory
      debug:
        msg: "Total Memory: {{ total_memory.stdout }}"

    - name: Check disk space
      shell: df -h / | tail -1 | awk '{print $4}'
      register: disk_space
      changed_when: false

    - name: Display disk space
      debug:
        msg: "Available Disk Space: {{ disk_space.stdout }}"

    - name: Check CPU count
      shell: nproc
      register: cpu_count
      changed_when: false

    - name: Display CPU count
      debug:
        msg: "CPU Count: {{ cpu_count.stdout }}"
EOF
```

Run preflight checks:

```bash
ansible-playbook -i {{PROJECT_BASE_PATH}}/ansible/inventory/hosts.yml {{PROJECT_BASE_PATH}}/ansible/playbooks/preflight-checks.yml
```

**Expected:** All checks pass, showing system resources

### Step 2.5: Create System Preparation Playbook

*This playbook configures the underlying operating system on all nodes by installing dependencies, disabling swap, and tuning kernel parameters, which are mandatory requirements for a stable K3s cluster,  each node is identically prepared for the next installation phase.*

```bash
cat > prepare-systems.yml << 'EOF'
---
- name: Prepare systems for K3s installation
  hosts: all
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install required packages
      apt:
        name:
          - curl
          - wget
          - git
          - vim
          - net-tools
          - ca-certificates
          - apt-transport-https
          - software-properties-common
        state: present

    - name: Disable swap
      shell: swapoff -a
      when: ansible_facts['memory_mb']['swap']['total'] > 0

    - name: Remove swap from fstab
      lineinfile:
        path: /etc/fstab
        regexp: '^.swap.*$'
        state: absent

    - name: Load br_netfilter module
      modprobe:
        name: br_netfilter
        state: present

    - name: Enable IP forwarding
      sysctl:
        name: net.ipv4.ip_forward
        value: '1'
        state: present
        reload: yes

    - name: Set timezone to UTC
      timezone:
        name: UTC

    - name: Verify connectivity
      ping:
        data: "System prepared successfully"
EOF
```

Run system preparation:

```bash
cd {{PROJECT_BASE_PATH}}

ansible-playbook -i {{PROJECT_BASE_PATH}}/ansible/inventory/hosts.yml {{PROJECT_BASE_PATH}}/ansible/playbooks/prepare-systems.yml
```

**Expected:** All tasks complete successfully

### Step 2.6: Verify System Preparation

```bash
cd {{PROJECT_BASE_PATH}}

# Verify system configuration on both nodes
ansible k3s_cluster -i ansible/inventory/hosts.yml -a "cat /proc/sys/net/ipv4/ip_forward"
# Expected: 1 (IP forwarding enabled)

ansible k3s_cluster -i ansible/inventory/hosts.yml -a "swapon --show"
# Expected: (empty output - swap disabled)

ansible k3s_cluster -i ansible/inventory/hosts.yml -a "uptime"
# Check system is stable
```

**Expected:** All checks pass, systems ready for Kubernetes

**🎓 Why No Docker?**
- K3s includes containerd as its container runtime
- Kaniko builds images without needing Docker daemon
- This is more secure: no privileged daemon access required
- This is the production-standard approach for Kubernetes

### Step 2.7: Commit Configuration Code

```bash
cd {{PROJECT_BASE_PATH}}

git add ansible/
git commit -m "feat(ansible): Add system preparation for Kubernetes"
git push origin main
```

---

## Phase 3: Kubernetes Cluster

```
┌─────────────────────────────────────────────────────────┐
│ PHASE 3: KUBERNETES CLUSTER ◄── YOU ARE HERE           │
├─────────────────────────────────────────────────────────┤
│  ┌──────────────────┐      ┌──────────────────┐        │
│  │ Control VM       │      │ Worker VM        │        │
│  │ ┌──────────────┐ │      │ ┌──────────────┐ │        │
│  │ │ K3s Server   │ │◄────►│ │ K3s Agent    │ │ Setup  │
│  │ │ (Master)     │ │ Join │ │ (Node)       │ │        │
│  │ └──────────────┘ │      │ └──────────────┘ │        │
│  └──────────────────┘      └──────────────────┘        │
│         ↓ Kubeconfig copied to Mac                     │
└─────────────────────────────────────────────────────────┘
```

**Objective:** Install K3s cluster (1 control plane, 1 worker)  
**Time:** 20 minutes  
**Result:** Functional Kubernetes cluster

**📋 Checkpoint:** Commands should use actual IPs from Phase 1.6, not `{{PLACEHOLDERS}}`.

### Step 3.1: Create K3s Server Installation Playbook

*This playbook installs and configures the K3s control plane on the designated server node, setting up the cluster's management components and generating the join token for workers. It works by executing a remote installation script with specific flags for the server role and then securely distributing the cluster access token to your local Ansible control node for subsequent steps.*

```bash
cd {{PROJECT_BASE_PATH}}/ansible/playbooks

cat > install-k3s-server.yml << 'EOF'
---
- name: Install K3s server (control plane)
  hosts: control_plane
  become: yes
  tasks:
    - name: Download K3s installation script
      get_url:
        url: https://get.k3s.io
        dest: /tmp/k3s-install.sh
        mode: '0755'

    - name: Install K3s server
      shell: |
        INSTALL_K3S_VERSION={{K3S_VERSION}} \
        K3S_TOKEN={{K3S_TOKEN}} \
        sh /tmp/k3s-install.sh server \
          --write-kubeconfig-mode=644 \
          --node-ip={{ node_ip }} \
          --cluster-init
      args:
        creates: /etc/systemd/system/k3s.service

    - name: Wait for K3s to be ready
      wait_for:
        port: 6443
        delay: 10
        timeout: 300

    - name: Wait for node to be ready
      shell: kubectl wait --for=condition=Ready node/{{CONTROL_PLANE_HOSTNAME}} --timeout=300s
      retries: 5
      delay: 10
      register: result
      until: result.rc == 0

    - name: Get node token for workers
      slurp:
        src: /var/lib/rancher/k3s/server/node-token
      register: node_token

    - name: Display node token
      debug:
        msg: "Node token: {{ node_token.content | b64decode }}"

    - name: Get K3s server status
      command: systemctl status k3s --no-pager
      register: k3s_status
      changed_when: false

    - name: Display K3s status
      debug:
        msg: "{{ k3s_status.stdout_lines }}"
EOF
```

Run K3s server installation:

```bash
ansible-playbook -i {{PROJECT_BASE_PATH}}/ansible/inventory/hosts.yml {{PROJECT_BASE_PATH}}/ansible/playbooks/install-k3s-server.yml
```

**Expected Output:**
```
TASK [Display node token]
ok: [{{CONTROL_PLANE_HOSTNAME}}] => {
    "msg": "Node token: K10xxxxxxxxxxxxxxxxxxxx::server:{{K3S_TOKEN}}"
}

PLAY RECAP
{{CONTROL_PLANE_HOSTNAME}}: ok=7   changed=3   failed=0
```

**⏱️ Wait Time:** 2-3 minutes for K3s installation

### Step 3.2: Create K3s Agent Installation Playbook

*This playbook installs the K3s agent on each worker node and joins them to the existing cluster using the token obtained from the control plane. It works by fetching the join token and executing the K3s installation script with the agent role, automatically registering each node under the control plane's management.*

```bash
cat > install-k3s-agent.yml << 'EOF'
---
- name: Install K3s agent (worker nodes)
  hosts: workers
  become: yes
  vars:
    k3s_server_ip: "{{CONTROL_PLANE_IP}}"
    k3s_token: "{{K3S_TOKEN}}"
  tasks:
    - name: Download K3s installation script
      get_url:
        url: https://get.k3s.io
        dest: /tmp/k3s-install.sh
        mode: '0755'

    - name: Install K3s agent
      shell: |
        INSTALL_K3S_VERSION={{K3S_VERSION}} \
        K3S_URL=https://{{ k3s_server_ip }}:6443 \
        K3S_TOKEN={{ k3s_token }} \
        sh /tmp/k3s-install.sh agent \
          --node-ip={{ node_ip }}
      args:
        creates: /etc/systemd/system/k3s-agent.service

    - name: Wait for K3s agent to start
      systemd:
        name: k3s-agent
        state: started
        enabled: yes

    - name: Wait for agent registration
      pause:
        seconds: 30

    - name: Get K3s agent status
      command: systemctl status k3s-agent --no-pager
      register: k3s_agent_status
      changed_when: false

    - name: Display K3s agent status
      debug:
        msg: "{{ k3s_agent_status.stdout_lines }}"
EOF
```

Run K3s agent installation:

```bash
ansible-playbook -i {{PROJECT_BASE_PATH}}/ansible/inventory/hosts.yml {{PROJECT_BASE_PATH}}/ansible/playbooks/install-k3s-agent.yml
```

**Expected:** Agent installs and connects to control plane

### Step 3.3: Copy Kubeconfig to Local Machine

*This step secures the cluster's administrative connection by copying the kubeconfig file from the control plane to your local machine and updating the server address to the node's accessible IP. Execute commands to transfer the file, replace the localhost IP, and set an environment variable, enabling your local kubectl to authenticate and manage the remote Kubernetes cluster.*

```bash
cd {{PROJECT_BASE_PATH}}

# Copy kubeconfig from control plane
multipass exec {{CONTROL_PLANE_HOSTNAME}} -- sudo cat /etc/rancher/k3s/k3s.yaml > kubeconfig

# Update server IP in kubeconfig
sed -i '' 's/127.0.0.1/{{CONTROL_PLANE_IP}}/g' kubeconfig

# Set KUBECONFIG environment variable
export KUBECONFIG={{PROJECT_BASE_PATH}}/kubeconfig

# Add to your shell profile for persistence
echo "export KUBECONFIG={{PROJECT_BASE_PATH}}/kubeconfig" >> ~/.zshrc
```

### Step 3.4: Verify Cluster

```bash
# Check kubectl works
kubectl version -short

# Get cluster info
kubectl cluster-info

# List nodes
kubectl get nodes
```

**Expected Output:**
```
NAME                          STATUS   ROLES                       AGE     VERSION
{{CONTROL_PLANE_HOSTNAME}}    Ready    control-plane,etcd,master   5m      {{K3S_VERSION}}
{{WORKER_HOSTNAME}}           Ready    <none>                      2m      {{K3S_VERSION}}
```

```bash
# List all pods
kubectl get pods -A
```

**Expected:** All pods in `Running` or `Completed` state

```bash
# Check namespaces
kubectl get namespaces
```

**Expected Namespaces:**
- default
- kube-system
- kube-public
- kube-node-lease

### Step 3.5: Test Cluster Functionality

Deploy a test pod:

```bash
# Create test pod
kubectl run nginx-test --image=nginx:alpine --port=80

# Wait for pod to be ready
kubectl wait --for=condition=Ready pod/nginx-test --timeout=60s

# Check pod status
kubectl get pod nginx-test

# Delete test pod
kubectl delete pod nginx-test
```

**Expected:** Pod creates, runs, and deletes successfully

### Step 3.6: Commit K3s Playbooks

```bash
cd {{PROJECT_BASE_PATH}}

git add ansible/playbooks/install-k3s-*.yml
git commit -m "feat(k3s): Add K3s cluster installation playbooks"
git push origin main
```

---

## Phase 4: DevOps Services

```
┌─────────────────────────────────────────────────────────┐
│ PHASE 4: DEVOPS SERVICES ◄── YOU ARE HERE              │
├─────────────────────────────────────────────────────────┤
│  Kubernetes Cluster                                     │
│  ┌────────────────────────────────────────────┐         │
│  │ ┌──────────┐         ┌──────────┐         │         │
│  │ │ Jenkins  │         │ Argo CD  │         │ Deploy  │
│  │ │   (CI)   │         │   (CD)   │         │         │
│  │ └──────────┘         └──────────┘         │         │
│  │   Port :30080          Port :30083        │         │
│  └────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────┘
```

**Objective:** Deploy Jenkins and Argo CD on Kubernetes  
**Time:** 60 minutes  
**Services:** Jenkins (CI), Argo CD (CD)

### Part A: Jenkins Installation

### Step 4.1: Create Jenkins Namespace

*Creating a dedicated namespace for Jenkins isolates its components and resources from other applications in the cluster, providing a clear administrative boundary and reducing potential conflicts.* 

```bash
cd {{PROJECT_BASE_PATH}}/kubernetes/jenkins

# Create namespace manifest
cat > namespace.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: {{JENKINS_NAMESPACE}}
  labels:
    name: {{JENKINS_NAMESPACE}}
    purpose: ci-cd
EOF

# Apply namespace
kubectl apply -f namespace.yaml

# Verify
kubectl get namespace {{JENKINS_NAMESPACE}}
```

### Step 4.2: Create Jenkins PVC

*This PersistentVolumeClaim defines storage requirements for Jenkins, ensuring its critical data like job configurations and build logs are retained across pod restarts or rescheduling. It works by requesting a specific amount of persistent storage from the cluster's available PersistentVolumes, which will be dynamically provisioned and bound to the Jenkins pod when deployed.*

```bash
cat > pvc.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
  namespace: {{JENKINS_NAMESPACE}}
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-path
EOF

kubectl apply -f pvc.yaml

# Verify PVC
kubectl get pvc -n {{JENKINS_NAMESPACE}}
```

**Expected:** `STATUS: Bound`

### Step 4.3: Create Jenkins RBAC

*This RBAC configuration grants the Jenkins service account the necessary permissions within its namespace to create and manage Kubernetes resources, which is essential for dynamic build agents and deployment tasks. We define a Role with specific API access rules and a RoleBinding that links that Role to the Jenkins service account, enabling secure, least-privilege automation from within Jenkins pipelines.*

```bash
cat > rbac.yaml << 'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: {{JENKINS_NAMESPACE}}
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create", "get"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: {{JENKINS_NAMESPACE}}
EOF

kubectl apply -f rbac.yaml

# Verify
kubectl get serviceaccount -n {{JENKINS_NAMESPACE}}
kubectl get clusterrole jenkins
kubectl get clusterrolebinding jenkins
```

### Step 4.4: Download kubectl Binary for Jenkins

*This step downloads the ARM64-compatible kubectl binary to your local machine, preparing it to be embedded into the custom Jenkins Docker image so Jenkins can interact with the Kubernetes cluster to be used for cluster operations inside the pipeline from within its container.*

```bash
# Download kubectl binary (ARM64)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"

# Make executable
chmod +x kubectl

# Verify it's ARM64
file kubectl
# Expected: ... arm64 ...

# Keep it in jenkins directory (will be copied to container)
```

**Note:** This binary will be embedded in Jenkins Docker image

### Step 4.5: Create Jenkins Dockerfile

*This Dockerfile creates a custom Jenkins image by extending the official LTS version, installing essential plugins, and embedding the kubectl binary to enable cluster management directly from the Jenkins controller.*

```bash
cat > Dockerfile << 'EOF'
FROM jenkins/jenkins:lts-jdk17

USER root

# Copy kubectl binary
COPY kubectl /usr/local/bin/kubectl
RUN chmod +x /usr/local/bin/kubectl

# Install Jenkins plugins
RUN jenkins-plugin-cli --plugins \
    git \
    workflow-aggregator \
    kubernetes \
    configuration-as-code

USER jenkins
EOF
```

**🎓 Why No Docker?**
- Builds run in Kaniko pods (rootless, daemonless)
- Jenkins Controller only orchestrates—doesn't build
- kubectl is sufficient for Kubernetes pod management
- This is the production-standard approach

### Step 4.6: Build and Push Jenkins Image to Repository

*We package the custom Jenkins Dockerfile into a container image and publish it to Docker Hub as a public repository, making the image accessible for deployment into your Kubernetes cluster for image pulls.*

**Make sure Docker Desktop is running!!** 

**📝 Note:** If your Docker repository is set as Private The image you just pushed will be created as private by default on Docker Hub and vice versa. Make the repository public for this tutorial to avoid trouble.

```bash
# Login to Docker Hub
docker login

# Build image
docker build -t {{DOCKERHUB_USERNAME}}/{{JENKINS_IMAGE_NAME}}:lts .

# Push to Docker Hub (private repository)
docker push {{DOCKERHUB_USERNAME}}/{{JENKINS_IMAGE_NAME}}:lts
```

**⏱️ Build time:** 3-5 minutes





### Step 4.7: Create Jenkins Deployment

**📚 Architecture Note - Jenkins with Kubernetes Agents:**


*This Deployment manifest defines a pod for the Jenkins controller that uses the custom image we've built, connects to persistent storage, and runs with a dedicated service account for secure cluster operations. It works by specifying the container specifications and resource limits, then Kubernetes creates and maintains the pod according to this blueprint, ensuring Jenkins remains available and can manage ephemeral build agents.*

```bash
cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: {{JENKINS_NAMESPACE}}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      serviceAccountName: jenkins
      imagePullSecrets:
      - name: dockerhub-credentials
      containers:
      - name: jenkins
        image: {{DOCKERHUB_USERNAME}}/{{JENKINS_IMAGE_NAME}}:lts
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 50000
          name: agent
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
      volumes:
      - name: jenkins-home
        persistentVolumeClaim:
          claimName: jenkins-pvc
EOF
```

This deployment uses a **Jenkins Controller** that spawns **ephemeral Kubernetes pods** as build agents. 

**Production-Grade Approach:**
- ✅ **No privileged access:** No Docker socket mounting
- ✅ **Dynamic agents:** Jenkins creates pods on-demand for each build
- ✅ **Isolation:** Each build runs in its own pod
- ✅ **Scalable:** Kubernetes manages resource allocation

**Resource Considerations:**
- Memory limit: 2Gi (sufficient for controller + small builds)
- Build pods (Kaniko) get their own resource allocation
- Monitor with: `kubectl top pod -n jenkins`

**Security Benefits:**
- No root-equivalent access to host
- No Docker daemon dependency
- Container builds happen in isolated pods
- Follows Kubernetes security best practices

**⚠️ IMPORTANT:** Before applying the deployment, you need to create the `dockerhub-credentials` secret. This will be done in Step 4.10. If you need to apply the deployment now, skip to Step 4.10 first to create the credentials, then return here.

Apply deployment:

```bash
kubectl apply -f deployment.yaml

# Watch pod creation
kubectl get pods -n {{JENKINS_NAMESPACE}} -w
```

**Wait until STATUS = Running** (Ctrl+C to exit watch)

### Step 4.8: Create Jenkins Service

*This Service exposes the Jenkins controller inside the cluster and externally via a NodePort, allowing web access on port 30080 and agent communication on 30081. It works by creating a stable network endpoint that routes traffic to the Jenkins pod based on its label selector, enabling both user interaction and build agent connections from outside the cluster.*

```bash
cat > service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: {{JENKINS_NAMESPACE}}
spec:
  type: NodePort
  selector:
    app: jenkins
  ports:
    - name: http
      port: 8080
      targetPort: 8080
      nodePort: {{JENKINS_HTTP_PORT}}
    - name: agent
      port: 50000
      targetPort: 50000
      nodePort: {{JENKINS_AGENT_PORT}}
EOF

kubectl apply -f service.yaml

# Verify service
kubectl get svc -n {{JENKINS_NAMESPACE}}
```

### Step 4.9: Access Jenkins

```bash
# Get Jenkins initial admin password
kubectl exec -n {{JENKINS_NAMESPACE}} $(kubectl get pods -n {{JENKINS_NAMESPACE}} -l app=jenkins -o jsonpath='{.items[0].metadata.name}') -- cat /var/jenkins_home/secrets/initialAdminPassword
```

**Save this password!**

Open browser:
```
http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}}
```

**Setup Jenkins:**
1. Enter initial admin password
2. Select "Install suggested plugins"
3. Wait for plugins to install (5-10 minutes)
4. Create admin user:
   - Username: `admin`
   - Password: `admin` (or your choice)
   - Full name: `Admin User`
   - Email: `admin@example.com`
5. Keep default Jenkins URL: `http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}}/`
6. Click "Start using Jenkins"

### Step 4.9.5: Configure Kubernetes Cloud in Jenkins

**🎓 Why This Step is Critical:**

The Jenkinsfile in Phase 5 uses a Kubernetes pod template to spawn ephemeral build agents. For this to work, Jenkins needs to know:
- How to connect to your K3s cluster
- Where to create the build pods
- How to communicate with those pods

Without this configuration, your pipeline builds will fail with errors like:
```
ERROR: Kubernetes cloud not configured
ERROR: Unable to create pod
```

**Configure Kubernetes Cloud:**

1. **Navigate to Cloud Configuration:**
   - In Jenkins UI, click **Manage Jenkins** (left sidebar)
   - Scroll down and click **Clouds** (under "System Configuration")
   - Click **New cloud**
   - Enter cloud name: `kubernetes`
   - Select **Kubernetes** from dropdown
   - Click **Create**

2. **Configure Kubernetes Connection:**

   In the Kubernetes Cloud configuration form, set the following:

   **Kubernetes Settings:**
   - **Kubernetes URL:** Leave blank or use `https://kubernetes.default.svc.cluster.local`
     - ℹ️ When blank, Jenkins auto-detects the in-cluster Kubernetes API
   - **Kubernetes server certificate key:** Leave blank
   - **Disable https certificate check:** ✅ **Check this box** (⚠️ For production, use proper TLS certificates instead
   - **Kubernetes Namespace:** `{{JENKINS_NAMESPACE}}` (e.g., `jenkins`)
     - This is where Jenkins will create build pods
   - **Credentials:** Leave as `- none -`
     - ℹ️ Jenkins uses the service account token automatically (we created this in Step 4.3)
   - **WebSocket:** ✅ **Check this box** (recommended for better performance)

   **Jenkins Settings:**
   - **Jenkins URL:** `http://jenkins.{{JENKINS_NAMESPACE}}.svc.cluster.local:8080`
     - This is the internal Kubernetes service DNS name
     - Format: `http://SERVICE_NAME.NAMESPACE.svc.cluster.local:PORT`
   - **Jenkins tunnel:** `jenkins.{{JENKINS_NAMESPACE}}.svc.cluster.local:50000`
     - This is for JNLP agent communication
     - Format: `SERVICE_NAME.NAMESPACE.svc.cluster.local:AGENT_PORT`

   **Advanced Settings (expand if needed):**
   - **Connection Timeout:** `5` (seconds)
   - **Read Timeout:** `15` (seconds)
   - **Container Cap:** `10` (max concurrent build pods)
   - **Max connections to Kubernetes API:** `32`

3. **Test the Connection:**
   - Scroll down to the bottom of the form
   - Click **Test Connection** button
   - **Expected result:** 
     ```
     Connected to Kubernetes v1.28.5+k3s1
     ```
   - If you see an error, check:
     - Jenkins pod has the correct service account (`jenkins`)
     - RBAC permissions are applied (Step 4.3)
     - Kubernetes namespace matches your deployment

4. **Save Configuration:**
   - Click **Save** at the bottom of the page

**Verification:**

```bash
# Verify Jenkins can access Kubernetes API
kubectl logs -n {{JENKINS_NAMESPACE}} deployment/jenkins --tail=20

# Should not see any Kubernetes API errors
```

**🔍 How This Works:**

```
┌─────────────────────────────────────────────────────┐
│ Jenkins Controller Pod (jenkins namespace)          │
│   ↓ Uses ServiceAccount "jenkins"                   │
│   ↓ Talks to Kubernetes API                         │
│ Creates Build Pod (jenkins namespace)               │
│   ↓ Runs Kaniko container                           │
│   ↓ Builds Docker image                             │
│   ↓ Pushes to Docker Hub                            │
│ Deletes Build Pod (cleanup)                         │
└─────────────────────────────────────────────────────┘
```

**📝 Important Notes:**

1. **Service Account Authentication:** 
   - Kubernetes 1.24+ uses projected volume service account tokens
   - Jenkins auto-discovers the token at `/var/run/secrets/kubernetes.io/serviceaccount/token`
   - No manual credentials needed!

2. **RBAC Permissions:**
   - The `jenkins` ClusterRole (Step 4.3) grants pod management permissions
   - Jenkins can create, list, delete pods in any namespace
   - For production, restrict to specific namespaces using Role instead of ClusterRole

3. **DNS Resolution:**
   - The Jenkins URL uses internal Kubernetes DNS
   - Format: `SERVICE.NAMESPACE.svc.cluster.local`
   - This works because all pods share the same DNS resolver

**Troubleshooting:**

**If "Test Connection" fails:**

```bash
# Check Jenkins service account exists
kubectl get serviceaccount jenkins -n {{JENKINS_NAMESPACE}}

# Check RBAC binding
kubectl describe clusterrolebinding jenkins

# Check Jenkins can reach Kubernetes API
kubectl exec -n {{JENKINS_NAMESPACE}} deployment/jenkins -- curl -k https://kubernetes.default.svc.cluster.local:443

# Check Jenkins logs for errors
kubectl logs -n {{JENKINS_NAMESPACE}} deployment/jenkins | grep -i kubernetes
```

**If builds fail to spawn pods later:**

```bash
# Watch pod creation in real-time
kubectl get pods -n {{JENKINS_NAMESPACE}} -w

# Check recent events
kubectl get events -n {{JENKINS_NAMESPACE}} --sort-by='.lastTimestamp' | tail -20

# Verify service account token is mounted
kubectl exec -n {{JENKINS_NAMESPACE}} deployment/jenkins -- ls -la /var/run/secrets/kubernetes.io/serviceaccount/
```

**✅ Verification Checklist:**

Before proceeding to Step 4.10, confirm:
- [ ] Test Connection shows "Connected to Kubernetes"
- [ ] No errors in Jenkins logs related to Kubernetes
- [ ] Kubernetes cloud appears in Manage Jenkins → Clouds
- [ ] Configuration is saved

---

### Step 4.10: Configure Credentials

**Part A: Kubernetes Secret for Private Docker Registry**

Create Docker Hub credentials as a Kubernetes secret. This secret is used for:
1. Pulling the private Jenkins image
2. Allowing Kaniko to push built images


**⚠️ Important:** CREATE DOCKERHUB TOKEN:
- Go to: https://hub.docker.com/settings/security
- Click **New Access Token**
- Description: `GitOps Pipeline`
- Access permissions: **Read, Write, Delete**
- Click **Generate**
- **Copy the token** (shown only once)
- Use this token as `--docker-password` above

```bash
# Create Docker registry secret in Jenkins namespace
kubectl create secret docker-registry dockerhub-credentials \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username={{DOCKERHUB_USERNAME}} \
  --docker-password=YOUR_DOCKERHUB_TOKEN \
  --docker-email=YOUR_EMAIL \
  -n {{JENKINS_NAMESPACE}}

# Verify secret created
kubectl get secret dockerhub-credentials -n {{JENKINS_NAMESPACE}}
```

***If you have skipped here to create dockerhub token and you are done you can now return to line 1616 to continue with the deployment***

**Part B: Jenkins UI Credentials**

In Jenkins UI:

1. Go to: **Manage Jenkins** → **Credentials** → **(global)** → **Add Credentials**

2. **Add GitHub Credentials:**
   - Kind: `Username with password`
   - Username: `your-github-username`
   - Password: `your-github-personal-access-token`
   - ID: `github-credentials`
   - Description: `GitHub PAT`
   - Click **Create**

**📝 Note:** Docker Hub credentials are managed through the Kubernetes secret, not Jenkins UI credentials

### Part B: Argo CD Installation

*Installing Argo CD deploys a declarative GitOps tool that automatically synchronizes your Kubernetes cluster state with the desired configuration defined in your Git repository. This is accomplished by applying the official Argo CD manifests, which create the necessary deployments, services, and custom resources to establish a continuous delivery controller within your cluster.*

### Step 4.11: Install Argo CD

```bash
cd {{PROJECT_BASE_PATH}}/kubernetes/argocd

# Create namespace
kubectl create namespace {{ARGOCD_NAMESPACE}}

# Install Argo CD
kubectl apply --server-side -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=Ready pods --all -n {{ARGOCD_NAMESPACE}} --timeout=300s
```

**⏱️ Wait Time:** 3-5 minutes

Verify installation:

```bash
kubectl get pods -n {{ARGOCD_NAMESPACE}}
```

**Expected:** All pods in `Running` state

### Step 4.12: Expose Argo CD Server

*Below command modifies the Argo CD service to make its web interface accessible externally by changing its type from ClusterIP to NodePort and mapping specific ports on your cluster nodes. (This NodePort approach is standard for training and quick access, while production environments typically use an Ingress controller with proper TLS termination for secure and managed external exposure.)*

```bash
# Patch service to use NodePort
kubectl patch svc argocd-server -n {{ARGOCD_NAMESPACE}} -p '{"spec": {"type": "NodePort","ports": [{"name": "http","port": 80,"targetPort": 8080,"nodePort": {{ARGOCD_HTTP_PORT}}},{"name": "https","port": 443,"targetPort": 8080,"nodePort": {{ARGOCD_HTTPS_PORT}}}]}}'

# Verify service
kubectl get svc -n {{ARGOCD_NAMESPACE}} argocd-server
```

### Step 4.13: Get Argo CD Admin Password

```bash
# Get initial admin password
kubectl -n {{ARGOCD_NAMESPACE}} get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

**Save this password!**

Access Argo CD:
```
http://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTP_PORT}}  (HTTP)
https://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTPS_PORT}} (HTTPS - accept self-signed cert)
```

**Login:**
- Username: `admin`
- Password: (from command above)

### Step 4.14: Install Argo CD CLI (Optional)

```bash
# Install on Mac
brew install argocd

# Login via CLI
argocd login {{CONTROL_PLANE_IP}}:{{ARGOCD_HTTPS_PORT}} --insecure --username admin --password YOUR_PASSWORD

# Change password (optional)
argocd account update-password
```

### Step 4.15: Commit Jenkins and Argo CD Manifests

```bash
cd {{PROJECT_BASE_PATH}}

# Add Jenkins files (but not kubectl binary)
echo "kubernetes/jenkins/kubectl" >> .gitignore

git add kubernetes/jenkins/*.yaml kubernetes/jenkins/Dockerfile
git add kubernetes/argocd/
git commit -m "feat(k8s): Add Jenkins and Argo CD deployments"
git push origin main
```

### Step 4.16: Verify Both Services

```bash
# Check all resources
kubectl get all -n {{JENKINS_NAMESPACE}}
kubectl get all -n {{ARGOCD_NAMESPACE}}

# Test connectivity
curl -I http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}}  # Jenkins
curl -I http://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTP_PORT}}  # Argo CD HTTP
```

**Expected:** HTTP responses from both services

---

### Part C: Secret Management

*This phase deploys HashiCorp Vault as a centralized, secure secret store for your GitOps environment, enabling dynamic credential injection into applications at runtime without exposing secrets in manifests or Git. It works by installing Vault in your cluster with a Helm chart, configuring it to authenticate using Kubernetes service accounts, and using sidecar injectors to securely deliver secrets directly to pods.*

**⚠️ IMPORTANT:** This training setup uses Vault in dev mode with in-memory storage, meaning all data (secrets, auth configurations, policies) is lost when VMs restart and must be reconfigured as explained in Steps 4.18-4.19

## Phase 4.5: HashiCorp Vault

```
┌─────────────────────────────────────────────────────────┐
│ PHASE 4.5: SECRET MANAGEMENT ◄── YOU ARE HERE           │
├─────────────────────────────────────────────────────────┤
│  Kubernetes Cluster                                     │
│  ┌────────────────────────────────────────────┐         │
│  │ ┌──────────┐         ┌──────────┐         │         │
│  │ │  Vault   │────────▶│ K8s Pods │         │ Dynamic │
│  │ │ (Secrets)│         │ (Apps)   │         │ Inject  │
│  │ └──────────┘         └──────────┘         │         │
│  │   Port :8200          Sidecar Pattern     │         │
│  └────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────┘
```

**Objective:** Deploy centralized secret management  
**Time:** 30 minutes  
**Service:** HashiCorp Vault

**Why Vault:**
- Centralized secret storage with audit trails
- Dynamic secret injection at pod runtime
- Service Account-based authentication (no hardcoded credentials)
- Industry standard for production Kubernetes

### Step 4.17: Install HashiCorp Vault

*Installing HashiCorp Vault with Helm deploys a complete secret management system into your cluster, providing a centralized, encrypted store for credentials, certificates, and keys. It works by adding the HashiCorp Helm repository and executing the chart with a minimal configuration, which creates the necessary StatefulSet, services, and configuration for a production-ready Vault instance in development mode.*

```bash
cd {{PROJECT_BASE_PATH}}/kubernetes
mkdir -p vault
cd vault

# Add HashiCorp Helm repository
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

# Create namespace
cat > namespace.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: vault
  labels:
    name: vault
    purpose: secret-management
EOF

kubectl apply -f namespace.yaml
```

**Install Vault in dev mode:**

```bash
# Install with dev settings (in-memory storage, auto-unsealed)
helm install vault hashicorp/vault \
  --namespace vault \
  --set "server.dev.enabled=true" \
  --set "server.dev.devRootToken=root" \
  --set "injector.enabled=true" \
  --set "server.dataStorage.enabled=false"

# Wait for ready
kubectl wait --for=condition=Ready pod/vault-0 -n vault --timeout=300s

# Verify
kubectl get pods -n vault
```

**Expected:**
```
NAME                                   READY   STATUS
vault-0                                1/1     Running
vault-agent-injector-xxxxx            1/1     Running
```

**⚠️ Dev Mode Note:** Uses in-memory storage and HTTP for Mac M1 compatibility. Production requires persistent storage, TLS, and HA configuration.

### Step 4.18: Configure Kubernetes Authentication

*This configuration establishes a trust relationship between Vault and your Kubernetes cluster, allowing pods to authenticate automatically using their service account tokens to retrieve secrets. It works by enabling the Kubernetes authentication method within Vault, linking it to the cluster's API server, and defining a role that grants specific secret access based on a pod's service account and namespace.*

**Enable pods to authenticate using Service Account tokens:**

```bash
# Exec into Vault pod
kubectl exec -it vault-0 -n vault -- sh

# Inside Vault pod - enable Kubernetes auth
vault auth enable kubernetes

# Configure Vault to trust K8s API
vault write auth/kubernetes/config \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"

# Create policy for app secrets
vault policy write {{APP_NAME}}-policy - <<EOF
path "secret/data/{{APP_NAME}}/*" {
  capabilities = ["read"]
}
EOF

# Create role binding policy to Service Account
vault write auth/kubernetes/role/{{APP_NAME}} \
    bound_service_account_names=default \
    bound_service_account_namespaces={{APP_NAMESPACE}} \
    policies={{APP_NAME}}-policy \
    ttl=24h
```

*There might appear a warning regarding audience configuration which means Vault is suggesting an optional security enhancement, but it's not required for the setup to work. You can simply ignore and proceed*

```bash

# Exit Vault pod
exit
```

**What this does:**
- Pods in `{{APP_NAMESPACE}}` with `default` SA can authenticate to Vault
- They can read secrets from `secret/{{APP_NAME}}/*` path only
- Access tokens expire after 24 hours

```mermaid
sequenceDiagram
   
    participant Vault as Vault Pod
    participant K8sAPI as Kubernetes API
    participant Policy as Vault Policy Engine
    participant Role as Kubernetes Auth Role
    participant Pod as Application Pod<br/>({{APP_NAMESPACE}})
    participant SA as Service Account<br/>(default)


    rect rgb(80, 0, 160)
        Note over Pod,Vault: Runtime: How Pods Authenticate & Access Secrets
        Pod->>SA: Request service account token
        SA-->>Pod: JWT token (includes namespace, SA name)
        Pod->>Vault: Authenticate with JWT token
        Vault->>K8sAPI: Verify token validity
        K8sAPI-->>Vault: Token valid ✓
        Vault->>Role: Check role: {{APP_NAME}}<br/>SA=default? ✓<br/>Namespace={{APP_NAMESPACE}}? ✓
        Role-->>Vault: Authorization granted
        Vault->>Policy: Check {{APP_NAME}}-policy permissions
        Policy-->>Vault: Can read secret/data/{{APP_NAME}}/*
        Vault-->>Pod: Return Vault token (TTL: 24h)
        Pod->>Vault: Request secret from secret/{{APP_NAME}}/*
        Vault->>Policy: Verify read permission
        Policy-->>Vault: Allowed ✓
        Vault-->>Pod: Return secret data
    end

    Note over Pod,Vault: Token expires after 24 hours<br/>Pod must re-authenticate for new token

```
### Step 4.19: Store Secrets in Vault

*This step creates and stores an actual secret, such as a database password, inside Vault's encrypted storage, making it available for secure retrieval by authorized applications. It works by using the Vault CLI to write a key-value pair to a defined secret path, which is then encrypted and stored, ready for dynamic injection into pods that have the corresponding access policy.*

```bash
# Write application secrets to Vault
kubectl exec -it vault-0 -n vault -- \
  vault kv put secret/{{APP_NAME}}/config \
  app_secret="ProductionSecretValue2024" \
  environment="production"

# Verify (optional)
kubectl exec -it vault-0 -n vault -- \
  vault kv get secret/{{APP_NAME}}/config
```

**Expected:**
```
===== Data =====
Key             Value
---             -----
app_secret      ProductionSecretValue2024
environment     production
```

### Step 4.20: Save Vault Configuration

*Saving the Vault configuration as a Kubernetes ConfigMap persists the setup commands for the Kubernetes authentication method and secret policies, ensuring they can be re-applied consistently if the Vault pod is recreated. It works by storing the executed Vault CLI commands in a ConfigMap manifest, which acts as documented, version-controlled runbook that can be referenced or reapplied during recovery or scaling scenario*

```bash
cd {{PROJECT_BASE_PATH}}/kubernetes/vault
```
### Create installation script

*Create an installation script that automates the deployment of Vault using Helm with a development-mode configuration, providing a consistent and repeatable method to stand up the secret management service. It works by defining a bash script that adds the Helm repository, creates the namespace, installs the chart with specific parameters, and waits for the pod to become ready, capturing the entire process in a single executable command.*

```bash
cat > install-vault.sh << 'EOF'
#!/bin/bash
set -e

helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

kubectl apply -f namespace.yaml

helm install vault hashicorp/vault \
  --namespace vault \
  --set "server.dev.enabled=true" \
  --set "server.dev.devRootToken=root" \
  --set "injector.enabled=true" \
  --set "server.dataStorage.enabled=false"

kubectl wait --for=condition=Ready pod/vault-0 -n vault --timeout=300s
echo "✅ Vault installed"
EOF

chmod +x install-vault.sh
```

### Create configuration script

*Creata a configuration script that programmatically sets up Vault's Kubernetes authentication method and access policies by executing commands directly inside the Vault pod, ensuring the authentication backend is ready for applications to retrieve secrets. It works by encapsulating the necessary vault auth enable and policy creation commands into a script that can be run after installation, establishing the secure link between your Kubernetes cluster and the Vault server.*

```bash
cat > configure-auth.sh << 'EOF'
#!/bin/bash
set -e

kubectl exec -it vault-0 -n vault -- sh -c '
vault auth enable kubernetes
vault write auth/kubernetes/config \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
'

kubectl exec -it vault-0 -n vault -- vault policy write {{APP_NAME}}-policy - <<POLICY
path "secret/data/{{APP_NAME}}/*" {
  capabilities = ["read"]
}
POLICY

kubectl exec -it vault-0 -n vault -- vault write auth/kubernetes/role/{{APP_NAME}} \
    bound_service_account_names=default \
    bound_service_account_namespaces={{APP_NAMESPACE}} \
    policies={{APP_NAME}}-policy \
    ttl=24h

echo "✅ Vault configured"
EOF

chmod +x configure-auth.sh

```
### Create README
```bash
cat > README.md << 'EOF'
# HashiCorp Vault

## Installation
```bash
./install-vault.sh
./configure-auth.sh

## Add Secrets
```bash
kubectl exec -it vault-0 -n vault -- \
  vault kv put secret/{{APP_NAME}}/config key=value

## Verify
```bash
kubectl get pods -n vault
kubectl exec -it vault-0 -n vault -- vault status

EOF
```

**Commit configuration:**

```bash
cd {{PROJECT_BASE_PATH}}

git add kubernetes/vault/
git commit -m "feat(vault): Add HashiCorp Vault for secret management"
git push origin main
```

**Verify Vault setup:**

```bash
# Check pods
kubectl get pods -n vault

# Check auth methods
kubectl exec -it vault-0 -n vault -- vault auth list

# Verify role
kubectl exec -it vault-0 -n vault -- vault read auth/kubernetes/role/{{APP_NAME}}
```

**Expected:**
- Vault pods running (2/2)
- Auth methods include `kubernetes/`
- Role shows correct namespace and policy bindings

**✅ Phase 4.5 Complete!** Vault is ready to inject secrets into application pods.

---

## Phase 5: GitOps Pipeline

```
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 5: GITOPS PIPELINE ◄── YOU ARE HERE                        │
├──────────────────────────────────────────────────────────────────┤
│  Code → Jenkins → Test → Lint → Build → Push → Update → Deploy   │
│   ┌────┐   ┌──┐   ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌────┐   ┌───┐     │
│   │Dev │──→│CI│──→│Tst│─→│Lnt│─→│Bld│─→│Hub│─→│Mfst│──→│CD │     │
│   └────┘   └──┘   └───┘  └───┘  └───┘  └───┘  └────┘   └───┘     │
│    Edit    Pod   pytest  flake8 Kaniko  Push  GitOps   Sync      │
│                                                                  │
│  Quality Gates: Test + Lint before build • Rootless builds       │
└──────────────────────────────────────────────────────────────────┘
```

**Objective:** Production-grade CI/CD with quality gates and security best practices  
**Time:** 90 minutes  
**Flow:** Code → Jenkins Pod → Test & Lint → Build (Kaniko) → Push → Update Manifests → Argo CD Sync

**🎓 What You'll Learn:**
- Kubernetes-native CI/CD workflows
- Quality gates with automated testing and linting
- Rootless container image building with Kaniko
- GitOps principles with Argo CD
- Secure pipeline design without privileged access

**📋 Checkpoint:** Verify `{{GITHUB_USERNAME}}`, `{{DOCKERHUB_USERNAME}}`, and `{{PROJECT_NAME}}` show actual values.

### Part A: Application Repository

### Step 5.1: Create Application Repository

*We will create a dedicated Git repository to host a sample application source code, serving as the starting point for the CI/CD pipeline.*

```bash
# Create app directory
mkdir -p ~/Desktop/{{PROJECT_NAME}}-{{APP_NAME}}
cd ~/Desktop/{{PROJECT_NAME}}-{{APP_NAME}}

# Initialize git
git init
```

### Step 5.2: Create Flask Application

```bash
cat > app.py << 'EOF'
from flask import Flask, render_template_string
import os
import socket


app = Flask(__name__)

VERSION = os.getenv('APP_VERSION', '1.0.0')
POD_NAME = socket.gethostname()

HTML_TEMPLATE = """
<!DOCTYPE html>
<html>
<head><title>GitOps Pipeline Demo</title><head>
<body style="font-family: Arial; text-align: center; padding: 50px;">
    <h1>🚀 GitOps Pipeline - Hello from K3s!</h1>
    <p><strong>Version:</strong> {{ version }}</p>
    <p><strong>Pod:</strong> {{ pod_name }}</p>
    <hr>
    <p>Deployed via Jenkins CI + Argo CD</p>
</body>
</html>
"""


@app.route('/')
def home():
    return render_template_string(
        HTML_TEMPLATE,
        version=VERSION,
        pod_name=POD_NAME
    )


@app.route('/health')
def health():
    return {'status': 'healthy', 'version': VERSION}, 200


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF
```

### Step 5.3: Create Requirements File

```bash
cat > requirements.txt << 'EOF'
Flask==3.0.0
gunicorn==21.2.0
EOF
```

### Step 5.4: Create Dockerfile

*This Dockerfile defines the container image for your application by specifying the base image, copying the source code, and installing dependencies to create a portable and consistent runtime environment. It works as a blueprint that the build system will follow to package your application into an immutable container, ready to be pushed to a registry and deployed.*

```bash
cat > Dockerfile << 'EOF'
FROM python:3.11-slim

WORKDIR /app

# Copy requirements and install
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY app.py .

# Set environment variable
ENV APP_VERSION=1.0.0

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:5000/health')"

# Run with gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "--timeout", "60", "app:app"]
EOF
```

### Step 5.5: Create Jenkinsfile (Production-Grade with Kaniko)

**🎓 What This Pipeline Does:**
1. **Checkout** - Clones your application code
2. **Install Dependencies** - Installs Python packages and test tools
3. **Lint Code** - Validates code quality with flake8
4. **Run Tests** - Executes pytest (if tests exist)
5. **Build with Kaniko** - Builds container image without Docker daemon
6. **Push to Registry** - Pushes to Docker Hub
7. **Update Manifests** - Updates GitOps repository with new version
8. **Trigger Argo CD** - Argo CD detects change and deploys

```bash
cat > Jenkinsfile << 'EOF'
pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: agent
spec:
  containers:
  - name: python
    image: python:3.11-slim
    command:
    - cat
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.9.0-debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  - name: git
    image: alpine/git:latest
    command:
    - cat
    tty: true
  volumes:
  - name: docker-config
    secret:
      secretName: dockerhub-credentials
      items:
      - key: .dockerconfigjson
        path: config.json
"""
        }
    }
    
    environment {
        DOCKER_IMAGE = '{{DOCKERHUB_USERNAME}}/{{PROJECT_NAME}}-{{APP_NAME}}'
        GITOPS_REPO = 'https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests.git'
        APP_VERSION = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                container('git') {
                    checkout scm
                    script {
                        // Fix Git safe.directory issue for any workspace path
                        sh 'git config --global --add safe.directory "*"'
                        
                        env.GIT_COMMIT_SHORT = sh(
                            script: "git rev-parse --short HEAD",
                            returnStdout: true
                        ).trim()
                    }
                }
            }
        }

        // test code for syntax & linter before build

        stage('Install Dependencies') {
            steps {
                container('python') {
                    sh '''
                        pip install --no-cache-dir -r requirements.txt
                        pip install pytest flake8
                    '''
                }
            }
        }
        
        stage('Lint Code') {
            steps {
                container('python') {
                    sh 'python -m flake8 app.py --max-line-length=120'
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                container('python') {
                    sh '''
                        # If tests directory exists, run tests
                        if [ -d "tests" ]; then
                            python -m pytest tests/ -v
                        else
                            echo "No tests directory found, skipping..."
                        fi
                    '''
                }
            }
        }
        
        stage('Build with Kaniko') {
            steps {
                container('kaniko') {
                    sh """
                        /kaniko/executor \
                        --context \$(pwd) \
                        --dockerfile \$(pwd)/Dockerfile \
                        --destination ${DOCKER_IMAGE}:${APP_VERSION} \
                        --destination ${DOCKER_IMAGE}:latest \
                        --cache=true \
                        --cache-ttl=24h
                    """
                }
            }
        }
        
        stage('Update GitOps Manifest') {
            steps {
                container('git') {
                    script {
                        withCredentials([usernamePassword(
                            credentialsId: 'github-credentials',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )]) {
                            sh """
                                apk add --no-cache sed
                                rm -rf {{PROJECT_NAME}}-manifests
                                git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests.git
                                cd {{PROJECT_NAME}}-manifests/{{APP_NAME}}
                                
                                sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${APP_VERSION}|' deployment.yaml
                                
                                git config user.email "jenkins@example.com"
                                git config user.name "Jenkins CI"
                                git add deployment.yaml
                                git commit -m "Update image to version ${APP_VERSION}"
                                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests.git main
                            """
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo "✅ Kaniko pipeline completed! Version: ${APP_VERSION}"
            echo "🚀 Image: ${DOCKER_IMAGE}:${APP_VERSION}"
            echo "📦 GitOps manifest updated, waiting for Argo CD sync..."
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
EOF
```

**🔍 Key Differences from Docker Socket Approach:**

| Aspect | Kaniko (This Pipeline) | Docker Socket (Old Way) |
|--------|----------------------|-------------------------|
| **Agent** | Kubernetes pod with multiple containers | `agent any` (runs on controller) |
| **Build Tool** | Kaniko executor (rootless) | Docker daemon (requires root) |
| **Security** | ✅ No privileged access | ❌ Root-equivalent access |
| **Dependencies** | ✅ No daemon needed | ❌ Requires Docker daemon |
| **Isolation** | ✅ Each build in own pod | ❌ Shares controller resources |
| **Caching** | ✅ Layer caching with `--cache` | ✅ Uses daemon cache |

### Step 5.6: Create README

```bash
cat > README.md << 'EOF'
# {{APP_IMAGE_NAME}} GitOps Application

Simple Flask application demonstrating GitOps workflow.

## Local Development

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app.py
```

Access: http://localhost:5000

## Docker Build

```bash
docker build -t {{APP_IMAGE_NAME}}:1.0.0 .
docker run -p 5000:5000 {{APP_IMAGE_NAME}}:1.0.0
```

## Environment Variables

- `APP_VERSION`: Application version (default: 1.0.0)
EOF
```

### Step 5.7: Create GitHub Repository & Push Application to GitHub

```bash
cd ~/Desktop/{{PROJECT_NAME}}-{{APP_NAME}}

# Create .gitignore
cat > .gitignore << 'EOF'
venv/
__pycache__/
*.pyc
.DS_Store
EOF

# Create GitHub repository

```bash

# 1. Go to https://github.com/new
# 2. Repository name: {{PROJECT_NAME}}-{{APP_NAME}}
# 3. Visibility: **Private**
# 4. Do NOT initialize with README
# 5. Create repository
# 6. Follow instructions to push existing repository
```

**📝 Note:** This is a private repository. Jenkins will use the GitHub credentials you configured in Step 4.10.


```bash
# Initial commit
git add .
git commit -m "init: Sample Flask application initial commit"

# Create GitHub repo and push
git remote add origin https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-{{APP_NAME}}.git
git branch -M main
git push -u origin main
```

### Part B: GitOps Manifest Repository

### Step 5.8: Create GitOps Repository

*Here we establish a separate Git repository dedicated solely to storing declarative Kubernetes manifests, which serves as the single source of truth for your cluster's desired state in the GitOps workflow.  Argo CD will continuously monitor and sync any changes to the live cluster.*


```bash
mkdir -p ~/Desktop/{{PROJECT_NAME}}-manifests/{{APP_NAME}}
cd ~/Desktop/{{PROJECT_NAME}}-manifests

git init
```

### Step 5.9: Create Kubernetes Manifests

**Secret Injection:** These manifests use Vault Agent Injector annotations. Vault automatically injects a sidecar container that fetches secrets and writes them to `/vault/secrets/config`. The application sources this file at startup.


#### Step 5.9.1: Create Namespace
*This namespace manifest creates a logical boundary within your Kubernetes cluster to isolate your application's resources, providing a clean environment for deployment and simplifying access control and resource management.*


```bash
cd {{APP_NAME}}

# Namespace
cat > namespace.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: {{APP_NAMESPACE}}
  labels:
    name: {{APP_NAMESPACE}}
    managed-by: argocd
EOF
```
#### Step 5.9.2: Create Deployment
*This deployment manifest defines the scalable pod specification for the application, including container image, resource limits, health checks, and integration with Vault for automatic secret injection at runtime. It works by declaring the desired state of the application pods, complete with annotations that trigger the Vault agent sidecar to fetch and expose secrets as environment variables, ensuring secure and dynamic configuration*


```bash
cd {{APP_NAME}}

# Deployment with Vault integration
cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{APP_NAME}}
  namespace: {{APP_NAMESPACE}}
  labels:
    app: {{APP_NAME}}
spec:
  replicas: 3
  selector:
    matchLabels:
      app: {{APP_NAME}}
  template:
    metadata:
      labels:
        app: {{APP_NAME}}
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "{{APP_NAME}}"
        vault.hashicorp.com/agent-inject-secret-config: "secret/data/{{APP_NAME}}/config"
        vault.hashicorp.com/agent-inject-template-config: |
          {{ with secret "secret/data/{{APP_NAME}}/config" -}}
            export APP_SECRET="{{ .Data.data.app_secret }}"
            export ENVIRONMENT="{{ .Data.data.environment }}"
          {{- end }}
    spec:
      serviceAccountName: default
      imagePullSecrets:
      - name: dockerhub-secret
      containers:
      - name: {{APP_NAME}}
        image: {{DOCKERHUB_USERNAME}}/{{APP_IMAGE_NAME}}:1
        ports:
        - containerPort: 5000
          name: http
        command: ["/bin/sh", "-c"]
        args: [". /vault/secrets/config && gunicorn --bind 0.0.0.0:5000 --workers 2 --timeout 60 app:app"]
        env:
        - name: APP_VERSION
          value: "1.0.0"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 15
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 10
          periodSeconds: 5
EOF
```

#### Step 5.9.2: Create Service
*This service manifest exposes the application internally within the cluster and externally via a NodePort, providing a stable network endpoint to route traffic to the running application pods. It works by defining a Kubernetes Service resource that selects pods based on labels and maps a cluster node port to the container's internal port, making the application accessible for testing and validation.*


```bash
cd {{APP_NAME}}

# Service
cat > service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: {{APP_NAME}}
  namespace: {{APP_NAMESPACE}}
spec:
  type: NodePort
  selector:
    app: {{APP_NAME}}
  ports:
    - name: http
      port: 80
      targetPort: 5000
      nodePort: {{APP_HTTP_PORT}}
EOF
```

### Step 5.10: Create Repository README

```bash
cd ~/Desktop/{{PROJECT_NAME}}-manifests

cat > README.md << 'EOF'
# GitOps Manifests Repository

Kubernetes manifests managed by Argo CD.

## Structure

```
{{APP_NAME}}/
├── namespace.yaml    # Namespace definition
├── deployment.yaml   # Application deployment
└── service.yaml      # Service definition
```

## Sync Policy

Manifests are automatically synced by Argo CD when changes are detected.

## Image Updates

Image versions are updated by Jenkins CI pipeline.
EOF
```

### Step 5.11: Push GitOps Manifests to GitHub

# Create GitHub repository

```bash

# 1. Go to https://github.com/new
# 2. Repository name: {{PROJECT_NAME}}-manifests
# 3. Private
# 4. Do NOT initialize with README
# 5. Create repository
# 6. Follow instructions to push existing repository
```



```bash
cd ~/Desktop/{{PROJECT_NAME}}-manifests

git add .
git commit -m "init: Initial Kubernetes manifests"

# Create GitHub repo and push
git remote add origin https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests.git
git branch -M main
git push -u origin main
```

### Part C: Jenkins Pipeline Setup

### Step 5.12: Create Jenkins Pipeline Job

*We configure a pipeline job within the Jenkins web interface that connects to the application repository and executes the defined Jenkinsfile, automating the entire CI/CD process from code commit to deployment. Pushing the application code to repository triggers Jenkins to clone the code and run the pipeline stages within dynamically provisioned Kubernetes pods.*


In Jenkins UI (http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}}):

1. Click **New Item**
2. Enter name: `{{PROJECT_NAME}}-{{APP_NAME}}-pipeline`
3. Select **Pipeline**
4. Click **OK**

5. In **General** section:
   - ✅ GitHub project
   - Project URL: `https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-{{APP_NAME}}`

6. In **Build Triggers** section:
   - ✅ Poll SCM
   - Schedule: `H/5 * * * *` (check every 5 minutes)

7. In **Pipeline** section:
   - Definition: `Pipeline script from SCM`
   - SCM: `Git`
   - Repository URL: `https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-{{APP_NAME}}.git`
   - Credentials: Select `github-credentials`
   - Branch: `*/main`
   - Script Path: `Jenkinsfile`



8. Click **Save**

### Step 5.13: Trigger First Build

1. Click **Build Now**
2. Watch the build in **Build History**
3. Click on the build number
4. Click **Console Output**

**What You'll See:**
```
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Created Pod: jenkins/{{PROJECT_NAME}}-{{APP_NAME}}-pipeline-X-xxxxx
...
[kaniko] INFO[0045] Pushing image to {{DOCKERHUB_USERNAME}}/{{PROJECT_NAME}}-{{APP_NAME}}:X
[kaniko] INFO[0046] Pushed image to 1 destinations
✅ Kaniko pipeline completed! Version: X
```

**Expected:** Build completes successfully and pushes Docker image

**⏱️ First build time:** 5-10 minutes (longer first time due to layer caching setup)

**🔒 Production Security Benefits:**
- ✅ No privileged access to host
- ✅ Rootless image building
- ✅ No Docker daemon dependency
- ✅ Build isolation in ephemeral pods
- ✅ Kubernetes-native workflow

### Part D: Argo CD Application Setup



### Step 5.13: Argo CD Repository Authentication Setup
Private GitHub repositories require credentials for Argo CD to sync manifests



**Via Argo CD UI (Recommended for beginners)**
1. Open Argo CD: http://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTP_PORT}}
2. Navigate to Argo CD UI Settings → Repositories
3. Click "Connect Repo"
4. Configure:
   - Method: HTTPS
   - Type: git
   - Repository URL: https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests.git
   - Username: {{GITHUB_USERNAME}}
   - Password: GitHub Personal Access Token (PAT)
5. Verify connection status shows "Successful"

### Step 5.14: Create Argo CD Application
*This step defines an Argo CD Application resource that links your GitOps manifest repository to your Kubernetes cluster, enabling automated synchronization of the desired state defined in Git. It works by creating a YAML manifest that specifies the source repository, target cluster, and sync policy, which Argo CD uses to continuously monitor and apply any changes, ensuring the cluster matches the version-controlled configuration.*

```bash
cd {{PROJECT_BASE_PATH}}/kubernetes/argocd
mkdir -p applications
cd applications

cat > {{APP_NAME}}.yaml << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: {{APP_NAME}}
  namespace: {{ARGOCD_NAMESPACE}}
spec:
  project: default
  source:
    repoURL: https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests.git
    targetRevision: main
    path: {{APP_NAME}}
  destination:
    server: https://kubernetes.default.svc
    namespace: {{APP_NAMESPACE}}
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF
```

Apply the application:

```bash
kubectl apply -f {{APP_NAME}}.yaml

# Verify application created
kubectl get application -n {{ARGOCD_NAMESPACE}}
```
### Step 5.14b: Create Docker Registry Secret for Application Namespace


**🔐 Why a Second Secret?**

You already created `dockerhub-credentials` in the Jenkins namespace (Step 4.10). Now you need a separate secret in the application namespace because:
- Kubernetes secrets are namespace-scoped (cannot be shared across namespaces)
- The application pods need to pull your private Flask image from Docker Hub
- This is separate from the Jenkins/Kaniko secret

**Secret Overview:**

| Secret | Namespace | Used By |
|--------|-----------|---------|
| `dockerhub-credentials` | `jenkins` | Jenkins pod, Kaniko builds |
| `dockerhub-secret` | `{{APP_NAMESPACE}}` | Flask application pods |


Create the Docker Hub secret in the application namespace to pull private images:

```bash
# Create the test-app-namespace namespace before creating the secret, we have defined the namespace in the manifests but the manifest is not applied yet. Attempting to create secret without namespace will cause "namespace not found" error.
kubectl apply -f ~/Desktop/gitops-1-manifests/test-application/namespace.yaml

# Create Docker registry secret in application namespace
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username={{DOCKERHUB_USERNAME}} \
  --docker-password=YOUR_DOCKERHUB_TOKEN \
  --docker-email=YOUR_EMAIL \
  -n {{APP_NAMESPACE}}

# Verify secret created
kubectl get secret dockerhub-secret -n {{APP_NAMESPACE}}
```

**📝 Note:** Use the same Docker Hub access token you created in Step 4.10.


### Step 5.15: Verify in Argo CD UI

Open Argo CD: http://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTP_PORT}}

**You should see:**
- Application: `{{APP_NAME}}`
- Status: `Synced` and `Healthy`
- Resources: Namespace, Deployment, Service

**If not synced:** Click **Sync** → **Synchronize**

### Step 5.18: Access the Application

```bash
# Check pods
kubectl get pods -n {{APP_NAMESPACE}}

# Check service
kubectl get svc -n {{APP_NAMESPACE}}
```

**Expected:** 3 pods running, service with NodePort {{APP_HTTP_PORT}}

**⚠️ If Pods Show `ImagePullBackOff` or `ErrImagePull`:**

Verify that the `dockerhub-secret` was created correctly in Step 5.13a:

```bash
# Check if secret exists
kubectl get secret dockerhub-secret -n {{APP_NAMESPACE}}

# If secret is missing, create it:
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username={{DOCKERHUB_USERNAME}} \
  --docker-password=YOUR_DOCKERHUB_TOKEN \
  --docker-email=YOUR_EMAIL \
  -n {{APP_NAMESPACE}}

# Verify deployment has imagePullSecrets configured
kubectl get deployment {{APP_NAME}} -n {{APP_NAMESPACE}} -o yaml | grep -A 2 imagePullSecrets

# If needed, restart pods to pick up the secret
kubectl rollout restart deployment {{APP_NAME}} -n {{APP_NAMESPACE}}
```

Open browser:
```
http://{{CONTROL_PLANE_IP}}:{{APP_HTTP_PORT}}
```

**Expected:** Flask app showing:
- Version: 1.0.0
- Pod name


---

## Phase 6: Validation & Testing

**Objective:** Test the complete GitOps workflow  
**Time:** 30 minutes

*This final step validates the entire automated pipeline by making a code change, pushing it to trigger the Jenkins build, and confirming that Argo CD automatically deploys the new version to your cluster. We will modify a file in your application repository, commit the change, and then monitor the Jenkins pipeline logs and Argo CD UI to observe the image build, manifest update, and subsequent sync that rolls out the updated application.*


### Step 6.1: End-to-End Pipeline Test

**Test the complete CI/CD flow:**

```bash
cd ~/Desktop/{{PROJECT_NAME}}-{{APP_NAME}}

# Modify application
cat > app.py << 'EOF'
from flask import Flask, render_template_string
import os
import socket

app = Flask(__name__)

VERSION = os.getenv('APP_VERSION', '2.0.0')
POD_NAME = socket.gethostname()

HTML_TEMPLATE = """
<!DOCTYPE html>
<html>
<head><title>GitOps Pipeline Demo</title><head>
<body style="font-family: Arial; text-align: center; padding: 50px;
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white;">
    <h1>🚀 GitOps Pipeline - Hello from K3s!</h1>
    <h2>✨ Version 2.0 - Updated!</h2>
    <p><strong>Version:</strong> {{ version }}</p>
    <p><strong>Pod:</strong> {{ pod_name }}</p>
    <hr style="border-color: white;">
    <p>Deployed via Jenkins CI + Argo CD</p>
    <p>🎉 GitOps Workflow Working!</p>
</body>
</html>
"""


@app.route('/')
def home():
    return render_template_string(
        HTML_TEMPLATE,
        version=VERSION,
        pod_name=POD_NAME
    )


@app.route('/health')
def health():
    return {'status': 'healthy', 'version': VERSION}, 200


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# Commit and push
git add app.py
git commit -m "feat: Update to version 2.0 with new styling"
git push origin main
```

### Step 6.2: Monitor Pipeline Execution

**Watch the automated flow:**

1. **Jenkins (wait 5 minutes for SCM poll):**
   - Go to: http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}}
   - Watch `{{PROJECT_NAME}}-{{APP_NAME}}-pipeline` for new build
   - Or manually trigger: **Build Now**

2. **Console Output:**
   - Click on build number
   - Click **Console Output**
   - Watch: Checkout → Build → Push → Update Manifest

3. **GitHub:**
   - Go to: https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests
   - Check commits: Should see "Update image to version X"

4. **Argo CD:**
   - Go to: http://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTP_PORT}}
   - Watch `{{APP_NAME}}` application
   - Should auto-sync within 3 minutes

5. **Kubernetes:**
```bash
# Watch pods rolling update
kubectl get pods -n {{APP_NAMESPACE}} -w
```

6. **Test Application:**
```bash
# Access updated app
curl http://{{CONTROL_PLANE_IP}}:{{APP_HTTP_PORT}}
```

**Expected:** New version with purple gradient background!

### Step 6.3: Validate All Components

```bash
# 1. Check all nodes
kubectl get nodes

# 2. Check all namespaces
kubectl get namespaces

# 3. Check Jenkins
kubectl get all -n {{JENKINS_NAMESPACE}}

# 4. Check Argo CD
kubectl get all -n {{ARGOCD_NAMESPACE}}

# 5. Check Application
kubectl get all -n {{APP_NAMESPACE}}

# 6. Check Argo CD applications
kubectl get applications -n {{ARGOCD_NAMESPACE}}

# 7. Test all services
curl -I http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}}  # Jenkins
curl -I http://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTP_PORT}}  # Argo CD
curl -I http://{{CONTROL_PLANE_IP}}:{{APP_HTTP_PORT}}  # {{APP_NAME}}

# 8. Validate Vault integration
kubectl get pods -n {{APP_NAMESPACE}}  # Should show READY 2/2 (app + vault-agent)

# 9. Check secret injection
POD_NAME=$(kubectl get pods -n {{APP_NAMESPACE}} -l app={{APP_NAME}} -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n {{APP_NAMESPACE}} $POD_NAME -c {{APP_NAME}} -- ls /vault/secrets/

# 10. Verify secret content
kubectl exec -n {{APP_NAMESPACE}} $POD_NAME -c {{APP_NAME}} -- cat /vault/secrets/config
```

**Expected:**
- All services responding with HTTP 200 or 302
- Application pods show `2/2` ready (app container + vault-agent sidecar)
- `/vault/secrets/config` file exists with exported environment variables

**Vault Validation:**

```bash
# Verify Vault sidecar is running
kubectl describe pod -n {{APP_NAMESPACE}} -l app={{APP_NAME}} | grep -A 2 "vault-agent:"

# Test secret access
kubectl exec -n {{APP_NAMESPACE}} $POD_NAME -c {{APP_NAME}} -- sh -c '. /vault/secrets/config && echo $APP_SECRET'
```

**Expected:** Shows `ProductionSecretValue2024`

**If pods show 1/2 or Init:0/1:** Wait 10-20 seconds for Vault agent initialization.

### Step 6.4: Performance Test

```bash
# Test application load (10 requests)
for i in {1..10}; do
    curl -s http://{{CONTROL_PLANE_IP}}:{{APP_HTTP_PORT}} | grep "Version:"
done
```

**Expected:** All requests succeed, pod names vary (load balancing)

### Step 6.5: Rollback Test

**Test Argo CD rollback capability:**

In Argo CD UI:
1. Click on `{{APP_NAME}}` application
2. Click the **three dots (⋮)** menu button in the upper right corner
3. Select **History and Rollback**
4. Select a previous revision from the list
5. Click **Rollback**
6. A dialog will appear: *"Auto-Sync needs to be disabled in order for rollback to occur. Are you sure you want to disable auto-sync and rollback application {{APP_NAME}}?"*
7. Click **OK** to disable auto-sync and proceed with rollback

**Expected:** Application rolls back to previous version

**Re-enable Auto-Sync after testing:**
1. In the application view, click **APP DETAILS** button (top section)
2. Find **SYNC POLICY** section
3. Click **ENABLE AUTO-SYNC**
4. Verify auto-sync is active (shows "Automated" with prune and self-heal enabled)

### Step 6.6: Self-Healing Test

**Test Argo CD self-healing:**

```bash
# Manually scale deployment
kubectl scale deployment {{APP_NAME}} -n {{APP_NAMESPACE}} --replicas=5

# Watch Argo CD auto-heal (restore to 3 replicas)
kubectl get pods -n {{APP_NAMESPACE}} -w
```

**Expected:** Argo CD detects drift and restores to 3 replicas within 3 minutes

### Step 6.7: Documentation

**Create final documentation:**

```bash
cd {{PROJECT_BASE_PATH}}

# Commit all Phase 5 work
git add kubernetes/argocd/applications/
git commit -m "feat(gitops): Complete CI/CD pipeline setup"
git push origin main
```

---

## Phase 7: Observability & Monitoring

**Objective:** Deploy Prometheus for metrics collection and Grafana for visualization to monitor your GitOps pipeline and applications.

**What You'll Learn:**
- Deploying monitoring stack using Helm and GitOps
- Configuring Prometheus to scrape Kubernetes metrics
- Setting up Grafana dashboards
- Instrumenting applications for custom metrics
- Understanding key observability concepts

**Prerequisites:**
- Phase 6 completed successfully
- Application pods running and healthy
- ArgoCD operational

**Time Required:** ~45 minutes

---

### Architecture: Observability Layer

```
┌─────────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                           │
│                                                                 │
│  ┌───────────────┐      ┌──────────────┐     ┌──────────────┐ │
│  │ Applications  │─────▶│ Prometheus   │◀────│   Grafana    │ │
│  │ (Flask, etc)  │ :9090│ (Metrics DB) │:9090│ (Dashboard)  │ │
│  └───────────────┘      └──────┬───────┘     └──────────────┘ │
│         │                      │                    │         │
│         │ /metrics             │ scrape             │ :3000   │
│         ▼                      ▼                    ▼         │
│  ┌───────────────┐      ┌──────────────┐     ┌──────────────┐ │
│  │ Metrics       │      │ K8s API      │     │ External     │ │
│  │ Endpoint      │      │ (Node/Pod)   │     │ Access       │ │
│  └───────────────┘      └──────────────┘     └──────────────┘ │
│                                                                 │
│  Namespace: monitoring                                         │
└─────────────────────────────────────────────────────────────────┘
```

**Data Flow:**
1. Applications expose metrics at `/metrics` endpoint
2. Prometheus scrapes metrics every 15 seconds (configurable)
3. Prometheus stores time-series data
4. Grafana queries Prometheus and displays dashboards
5. Users access Grafana UI via NodePort

---

### Step 7.1: Create Monitoring Namespace

First, create a dedicated namespace for monitoring components:

```bash
kubectl create namespace monitoring

# Verify
kubectl get namespaces | grep monitoring
```

**Expected Output:**
```
monitoring        Active   5s
```

---

### Step 7.2: Install Prometheus with Helm

We'll use Helm (Kubernetes package manager) to simplify Prometheus deployment.

**Step 1: Add Helm Repository**

```bash
# Add Prometheus community Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Update repo cache
helm repo update

# Verify repo is added
helm repo list
```

**Expected Output:**
```
NAME                    URL
prometheus-community    https://prometheus-community.github.io/helm-charts
```

**Step 2: Create Prometheus Values File**

Create a custom configuration for Prometheus:

*This step creates a custom values.yaml configuration file that tailors the Helm-based Prometheus deployment for your training environment, defining resource limits, scraping rules, and disabling optional components. It works by specifying configuration overrides in YAML format that the Helm chart will use during installation, customizing Prometheus to monitor your Kubernetes cluster and applications efficiently.*

```bash
cd {{PROJECT_BASE_PATH}}
mkdir -p kubernetes/monitoring/prometheus

cat > kubernetes/monitoring/prometheus/values.yaml <<'EOF'
# Prometheus Configuration for GitOps Training

server:
  # Enable persistent storage (optional - uses hostPath for local dev)
  persistentVolume:
    enabled: false
  
  # Resource limits
  resources:
    requests:
      cpu: 200m
      memory: 512Mi
    limits:
      cpu: 500m
      memory: 1Gi
  
  # NodePort service for external access
  service:
    type: NodePort
    nodePort: 30090
  
  # Retention period
  retention: "7d"
  
  # Global scrape settings
  global:
    scrape_interval: 15s
    evaluation_interval: 15s

# Alertmanager (disabled for training - can enable later)
alertmanager:
  enabled: false

# Pushgateway (disabled for training)
pushgateway:
  enabled: false

# Node exporter - collects node-level metrics
prometheus-node-exporter:
  enabled: true
  hostNetwork: true
  hostPID: true

# Kube-state-metrics - exposes K8s object metrics
kube-state-metrics:
  enabled: true
EOF
```

**Step 3: Install Prometheus**

```bash
# Install Prometheus using Helm
helm install prometheus prometheus-community/prometheus \
  -f kubernetes/monitoring/prometheus/values.yaml \
  -n monitoring

# Watch deployment progress
kubectl get pods -n monitoring -w
```

**Expected Output:**
```
NAME                                            READY   STATUS    RESTARTS   AGE
prometheus-server-xxxxx                         1/1     Running   0          45s
prometheus-kube-state-metrics-xxxxx             1/1     Running   0          45s
prometheus-prometheus-node-exporter-xxxxx       1/1     Running   0          45s
```

Press `Ctrl+C` to stop watching once all pods are Running.

**Step 4: Verify Prometheus Access**

```bash
# Check service
kubectl get svc -n monitoring | grep prometheus-server

# Get Prometheus URL
echo "Prometheus UI: http://{{CONTROL_PLANE_IP}}:{{PROMETHEUS_HTTP_PORT}}"
```

Open the URL in your browser. You should see the Prometheus web interface.

**Test Query in Prometheus UI:**
- Go to Graph tab
- Enter query: `up`
- Click Execute
- Should show all scraped targets and their status (1 = up, 0 = down)

---

### Step 7.3: Install Grafana

**Step 1: Add Grafana Helm Repository**

```bash
# Add Grafana Helm repo
helm repo add grafana https://grafana.github.io/helm-charts

# Update repo cache
helm repo update

# Verify
helm repo list | grep grafana
```

**Step 2: Create Grafana Values File**

```bash
mkdir -p kubernetes/monitoring/grafana

cat > kubernetes/monitoring/grafana/values.yaml <<'EOF'
# Grafana Configuration for GitOps Training

# Admin credentials
adminUser: admin
adminPassword: admin123  # Change in production!

# Service configuration
service:
  type: NodePort
  nodePort: {{GRAFANA_HTTP_PORT}}

# Resource limits
resources:
  requests:
    cpu: 100m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

# Persistence (disabled for training)
persistence:
  enabled: false

# Data sources - Prometheus
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
      - name: Prometheus
        type: prometheus
        url: http://prometheus-server:80
        access: proxy
        isDefault: true
        editable: true

# Pre-installed dashboards
dashboardProviders:
  dashboardproviders.yaml:
    apiVersion: 1
    providers:
      - name: 'default'
        orgId: 1
        folder: ''
        type: file
        disableDeletion: false
        editable: true
        options:
          path: /var/lib/grafana/dashboards/default

# Import popular community dashboards
dashboards:
  default:
    # Kubernetes cluster monitoring
    kubernetes-cluster:
      gnetId: 7249
      revision: 1
      datasource: Prometheus
    
    # Node metrics
    node-exporter:
      gnetId: 1860
      revision: 27
      datasource: Prometheus
    
    # Pod metrics
    kubernetes-pods:
      gnetId: 6417
      revision: 1
      datasource: Prometheus

# Grafana configuration
grafana.ini:
  server:
    root_url: "http://{{CONTROL_PLANE_IP}}:{{GRAFANA_HTTP_PORT}}"
  analytics:
    check_for_updates: false
  log:
    mode: console
  security:
    allow_embedding: true
EOF
```

**Step 3: Install Grafana**

```bash
# Install Grafana using Helm
helm install grafana grafana/grafana \
  -f kubernetes/monitoring/grafana/values.yaml \
  -n monitoring

# Watch deployment
kubectl get pods -n monitoring -w
```

**Expected Output:**
```
NAME                                            READY   STATUS    RESTARTS   AGE
grafana-xxxxx                                   1/1     Running   0          30s
prometheus-server-xxxxx                         1/1     Running   0          5m
prometheus-kube-state-metrics-xxxxx             1/1     Running   0          5m
prometheus-prometheus-node-exporter-xxxxx       1/1     Running   0          5m
```

Press `Ctrl+C` to stop watching.

**Step 4: Access Grafana**

```bash
# Get Grafana service details
kubectl get svc -n monitoring | grep grafana

# Display access information
echo "Grafana URL: http://{{CONTROL_PLANE_IP}}:{{GRAFANA_HTTP_PORT}}"
echo "Username: admin"
echo "Password: admin123"
```

Open the URL in your browser and login with the credentials.

---

### Step 7.4: Verify Monitoring Stack

**Check All Components:**

```bash
# View all monitoring resources
kubectl get all -n monitoring

# Check Prometheus targets (via kubectl port-forward alternative)
kubectl get endpoints -n monitoring
```

**Expected Resources:**
- 4 pods running (Prometheus server, kube-state-metrics, node-exporter, Grafana)
- 4+ services
- 4 deployments/daemonsets

**Prometheus Targets Check:**

```bash
# Get Prometheus pod name
PROM_POD=$(kubectl get pod -n monitoring -l app.kubernetes.io/name=prometheus,app.kubernetes.io/component=server -o jsonpath='{.items[0].metadata.name}')

# Check targets via API
kubectl exec -n monitoring $PROM_POD -c prometheus-server -- wget -qO- http://localhost:9090/api/v1/targets | grep -o '"health":"[^"]*"' | head -10
```

Should show targets with `"health":"up"` status.

---

### Step 7.5: Explore Pre-Installed Dashboards

Once logged into Grafana (http://{{CONTROL_PLANE_IP}}:{{GRAFANA_HTTP_PORT}}):

**1. Navigate to Dashboards:**
- Click "Dashboards" icon (four squares) in left sidebar
- You'll see three pre-imported dashboards

**2. Kubernetes Cluster Dashboard (ID: 7249):**
- Shows cluster-wide metrics
- CPU/Memory usage by namespace
- Pod count and status
- Network I/O

**3. Node Exporter Dashboard (ID: 1860):**
- Physical node metrics
- CPU, RAM, disk, network per node
- System load and uptime

**4. Kubernetes Pods Dashboard (ID: 6417):**
- Per-pod resource usage
- Container restarts
- CPU throttling

**Validation:**
- All dashboards should show data within 1-2 minutes
- If "No Data", check Prometheus data source connection

---

### Step 7.6: Instrument Your Flask Application

To collect custom metrics from your Flask application, follow these steps:

**Step 1: Update Flask Application Code**

In your `{{PROJECT_NAME}}-{{APP_NAME}}` repository, add Prometheus instrumentation:

```bash
# Clone or navigate to your Flask app repo
cd ~/Desktop/{{PROJECT_NAME}}-{{APP_NAME}}

# Add prometheus_client to requirements.txt
echo "prometheus-client==0.19.0" >> requirements.txt
```

Update `app.py` to expose metrics:

```python
from flask import Flask, jsonify
from prometheus_client import Counter, Histogram, generate_latest, CONTENT_TYPE_LATEST
import time
import os

app = Flask(__name__)

# Define Prometheus metrics
request_count = Counter(
    'flask_app_requests_total',
    'Total Flask requests',
    ['method', 'endpoint', 'status']
)

request_duration = Histogram(
    'flask_app_request_duration_seconds',
    'Flask request duration',
    ['method', 'endpoint']
)

@app.before_request
def before_request():
    # Store request start time
    from flask import g
    g.start_time = time.time()

@app.after_request
def after_request(response):
    # Record metrics after each request
    from flask import request, g
    
    if hasattr(g, 'start_time'):
        duration = time.time() - g.start_time
        request_duration.labels(
            method=request.method,
            endpoint=request.endpoint or 'unknown'
        ).observe(duration)
    
    request_count.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown',
        status=response.status_code
    ).inc()
    
    return response

@app.route('/')
def home():
    # Load secrets from Vault
    app_secret = os.getenv('APP_SECRET', 'NOT_SET')
    environment = os.getenv('ENVIRONMENT', 'NOT_SET')
    
    return jsonify({
        "message": "Hello from GitOps Pipeline!",
        "version": "2.0.0",
        "environment": environment,
        "secret_loaded": app_secret != 'NOT_SET'
    })

@app.route('/health')
def health():
    return jsonify({"status": "healthy"}), 200

@app.route('/metrics')
def metrics():
    """Prometheus metrics endpoint"""
    return generate_latest(), 200, {'Content-Type': CONTENT_TYPE_LATEST}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Step 2: Update Deployment to Enable Scraping**

In `{{PROJECT_NAME}}-manifests/{{APP_NAME}}/deployment.yaml`, add Prometheus annotations:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{APP_NAME}}
  namespace: {{APP_NAMESPACE}}
spec:
  replicas: 3
  selector:
    matchLabels:
      app: {{APP_NAME}}
  template:
    metadata:
      labels:
        app: {{APP_NAME}}
      annotations:
        # Prometheus scraping configuration
        prometheus.io/scrape: "true"
        prometheus.io/port: "5000"
        prometheus.io/path: "/metrics"
        # ... existing Vault annotations ...
```

**Step 3: Trigger GitOps Pipeline**

```bash
# Commit Flask app changes
cd ~/Desktop/{{PROJECT_NAME}}-{{APP_NAME}}
git add requirements.txt app.py
git commit -m "feat: add Prometheus metrics instrumentation"
git push origin main

# Commit manifest changes
cd ~/Desktop/{{PROJECT_NAME}}-manifests/{{APP_NAME}}
git add deployment.yaml
git commit -m "feat: enable Prometheus scraping annotations"
git push origin main
```

**Step 4: Trigger Jenkins Build**

```bash
# The build should auto-trigger via webhook, or manually trigger in Jenkins UI
# Wait for Jenkins to build and push new image
# ArgoCD will detect manifest changes and sync automatically
```

**Step 5: Verify Metrics Endpoint**

```bash
# Wait for new pods to be running
kubectl get pods -n {{APP_NAMESPACE}}

# Get a pod name
POD_NAME=$(kubectl get pod -n {{APP_NAMESPACE}} -l app={{APP_NAME}} -o jsonpath='{.items[0].metadata.name}')

# Test metrics endpoint
kubectl exec -n {{APP_NAMESPACE}} $POD_NAME -c {{APP_NAME}} -- wget -qO- http://localhost:5000/metrics
```

**Expected Output:**
```
# HELP flask_app_requests_total Total Flask requests
# TYPE flask_app_requests_total counter
flask_app_requests_total{endpoint="health",method="GET",status="200"} 5.0
flask_app_requests_total{endpoint="home",method="GET",status="200"} 12.0

# HELP flask_app_request_duration_seconds Flask request duration
# TYPE flask_app_request_duration_seconds histogram
flask_app_request_duration_seconds_bucket{endpoint="home",method="GET",le="0.005"} 10.0
...
```

---

### Step 7.7: Create Custom Grafana Dashboard

**Step 1: Access Grafana UI**

Open http://{{CONTROL_PLANE_IP}}:{{GRAFANA_HTTP_PORT}} and login (admin/admin123)

**Step 2: Verify Prometheus Data Source**

- Go to **Configuration** (gear icon) → **Data Sources**
- Click **Prometheus**
- Scroll down and click **Save & Test**
- Should show: "Data source is working"

**Step 3: Create Dashboard for Flask App**

1. Click **+** icon → **Create Dashboard**
2. Click **Add visualization**
3. Select **Prometheus** data source

**Panel 1: Request Rate**

- Metric: `rate(flask_app_requests_total[5m])`
- Panel title: "Flask Requests per Second"
- Visualization: Graph
- Description: Shows request rate over time

**Panel 2: Error Rate**

- Metric: `rate(flask_app_requests_total{status=~"5.."}[5m])`
- Panel title: "Error Rate (5xx)"
- Visualization: Graph
- Set alert threshold (optional)

**Panel 3: Request Duration (p95)**

- Metric: `histogram_quantile(0.95, rate(flask_app_request_duration_seconds_bucket[5m]))`
- Panel title: "95th Percentile Latency"
- Visualization: Graph
- Unit: seconds

**Panel 4: Pod Status**

- Metric: `kube_pod_status_phase{namespace="{{APP_NAMESPACE}}"}`
- Panel title: "Pod Health Status"
- Visualization: Stat

**Step 4: Save Dashboard**

- Click **Save** icon (top right)
- Name: "Flask Application Metrics"
- Click **Save**

---

### Step 7.8: Generate Test Traffic

To see metrics in action, generate some load:

```bash
# Generate 100 requests to Flask app
for i in {1..100}; do
  curl -s http://{{CONTROL_PLANE_IP}}:{{APP_HTTP_PORT}}/ > /dev/null
  echo "Request $i completed"
  sleep 0.5
done
```

**Monitor in Grafana:**
- Refresh your dashboard
- Should see request rate spike
- Response time metrics updating
- Pod metrics showing CPU/memory usage

---

### Step 7.9: Explore Key Metrics

**In Prometheus UI (http://{{CONTROL_PLANE_IP}}:{{PROMETHEUS_HTTP_PORT}}):**

Try these useful queries:

```promql
# CPU usage per pod
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)

# Memory usage per namespace
sum(container_memory_usage_bytes) by (namespace)

# Pod restart count
kube_pod_container_status_restarts_total

# Number of running pods per namespace
count(kube_pod_info) by (namespace)

# Node CPU usage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Disk usage percentage
(node_filesystem_size_bytes - node_filesystem_avail_bytes) / node_filesystem_size_bytes * 100
```

**Understanding PromQL Basics:**

- `rate()` - Calculate per-second rate over time window
- `sum()` - Aggregate values
- `by (label)` - Group results by label
- `[5m]` - Look back 5 minutes
- Operators: `+`, `-`, `*`, `/`, `>`, `<`, `==`

---

### Step 7.10: Set Up Alerts (Optional)

Prometheus can alert on conditions. Example alert for pod restarts:

**Step 1: Update Prometheus Values with Alert Rules**

Edit `kubernetes/monitoring/prometheus/values.yaml` and add the `serverFiles` section after the `global` configuration:

```yaml
  # Global scrape settings
  global:
    scrape_interval: 15s
    evaluation_interval: 15s

# Add Alert rules configuration
serverFiles:
  alerting_rules.yml:
    groups:
      - name: pod_alerts
        interval: 30s
        rules:
          - alert: PodRestartingTooMuch
            expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
            for: 5m
            labels:
              severity: warning
            annotations:
              summary: "Pod {{ $labels.pod }} is restarting"
              description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} has restarted {{ $value }} times in the last 15 minutes"
          
          - alert: PodNotReady
            expr: kube_pod_status_phase{phase!="Running"} > 0
            for: 10m
            labels:
              severity: critical
            annotations:
              summary: "Pod {{ $labels.pod }} not ready"
              description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} has been in {{ $labels.phase }} state for more than 10 minutes"

# Alertmanager (disabled for training - can enable later)
alertmanager:
  enabled: false
```

**Step 2: Upgrade Prometheus with New Configuration**

```bash
helm upgrade prometheus prometheus-community/prometheus \
  --namespace monitoring \
  -f ~/Desktop/gitops-1/kubernetes/monitoring/prometheus/values.yaml
```

Wait for the upgrade to complete:

```bash
kubectl rollout status deployment/prometheus-server -n monitoring
```

**Step 3: Verify Alerts in Prometheus**

- Open Prometheus UI: http://192.168.2.18:30090
- Click **Alerts** tab
- You should see your configured alerts: `PodRestartingTooMuch` and `PodNotReady`
- Alerts will show as "Inactive" (green) until conditions are met

---

### Step 7.11: GitOps-ify the Monitoring Stack (Advanced)

*This step fully integrates the monitoring stack into your GitOps workflow by generating static Kubernetes manifests from Helm charts and defining ArgoCD Applications to manage them, ensuring Prometheus and Grafana are version-controlled and automatically synchronized. It works by using helm template to render the charts with your custom values, committing the YAML to your GitOps repository, and creating ArgoCD Application resources that point to these manifests, establishing declarative control over the entire monitoring layer.*

For true GitOps, convert Helm deployments to ArgoCD Applications:

**Step 1: Generate Manifests from Helm**

```bash
cd ~/Desktop/{{PROJECT_NAME}}-manifests
mkdir -p monitoring/prometheus monitoring/grafana

# Generate Prometheus manifests
helm template prometheus prometheus-community/prometheus \
  -f {{PROJECT_BASE_PATH}}/kubernetes/monitoring/prometheus/values.yaml \
  -n monitoring > monitoring/prometheus/manifests.yaml

# Generate Grafana manifests
helm template grafana grafana/grafana \
  -f {{PROJECT_BASE_PATH}}/kubernetes/monitoring/grafana/values.yaml \
  -n monitoring > monitoring/grafana/manifests.yaml

# Commit to Git
git add monitoring/
git commit -m "feat: add Prometheus and Grafana monitoring manifests"
git push origin main
```

**Step 2: Create ArgoCD Application**

```bash
cd {{PROJECT_BASE_PATH}}

cat > kubernetes/argocd/applications/monitoring-prometheus.yaml <<'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: prometheus
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests.git
    targetRevision: main
    path: monitoring/prometheus
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF

cat > kubernetes/argocd/applications/monitoring-grafana.yaml <<'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: grafana
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests.git
    targetRevision: main
    path: monitoring/grafana
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF

# Apply ArgoCD applications
kubectl apply -f kubernetes/argocd/applications/monitoring-prometheus.yaml
kubectl apply -f kubernetes/argocd/applications/monitoring-grafana.yaml
```

Now your monitoring stack is managed by ArgoCD!

---

### Step 7.12: Validation Checklist

**Prometheus Validation:**

```bash
# 1. Verify Prometheus is running
kubectl get pods -n monitoring -l app.kubernetes.io/name=prometheus

# 2. Check Prometheus targets
curl -s http://{{CONTROL_PLANE_IP}}:{{PROMETHEUS_HTTP_PORT}}/api/v1/targets | grep -o '"health":"up"' | wc -l
# Should show multiple targets (>5)

# 3. Query test metrics
curl -s 'http://{{CONTROL_PLANE_IP}}:{{PROMETHEUS_HTTP_PORT}}/api/v1/query?query=up' | grep -o '"status":"success"'
```

**Grafana Validation:**

```bash
# 1. Verify Grafana is running
kubectl get pods -n monitoring -l app.kubernetes.io/name=grafana

# 2. Check service
kubectl get svc -n monitoring grafana

# 3. Test dashboard access
curl -s http://{{CONTROL_PLANE_IP}}:{{GRAFANA_HTTP_PORT}}/login | grep -o "Grafana" | head -1
# Should output: Grafana
```

**Application Metrics Validation:**

```bash
# 1. Get Flask pod
POD_NAME=$(kubectl get pod -n {{APP_NAMESPACE}} -l app={{APP_NAME}} -o jsonpath='{.items[0].metadata.name}')

# 2. Check metrics endpoint exists
kubectl exec -n {{APP_NAMESPACE}} $POD_NAME -c {{APP_NAME}} -- wget -qO- http://localhost:5000/metrics | head -20

# 3. Verify Prometheus is scraping Flask pods
curl -s http://{{CONTROL_PLANE_IP}}:{{PROMETHEUS_HTTP_PORT}}/api/v1/targets | grep "{{APP_NAMESPACE}}"
```

**Expected Results:**
- ✅ Prometheus accessible at port {{PROMETHEUS_HTTP_PORT}} (default: 30090)
- ✅ Grafana accessible at port {{GRAFANA_HTTP_PORT}} (default: 30300)
- ✅ Pre-installed dashboards showing data
- ✅ Flask metrics visible in Prometheus
- ✅ All targets showing "up" status

---

### Understanding the Monitoring Stack

**Key Concepts:**

**1. Metrics Types:**
- **Counter:** Only goes up (e.g., total requests)
- **Gauge:** Can go up/down (e.g., current CPU usage)
- **Histogram:** Distributions (e.g., request latency buckets)
- **Summary:** Similar to histogram, pre-calculated quantiles

**2. Labels:**
Labels add dimensions to metrics:
```promql
flask_app_requests_total{method="GET", endpoint="home", status="200"}
```
Allows filtering and grouping by method, endpoint, status, etc.

**3. Scraping:**
Prometheus "pulls" metrics from targets on schedule:
- Default: Every 15 seconds
- Configured in `scrape_interval`
- Targets auto-discovered via Kubernetes API

**4. Service Discovery:**
Prometheus automatically finds:
- Pods with `prometheus.io/scrape: "true"` annotation
- Kubernetes API endpoints
- Nodes (via node-exporter)
- Services (via service discovery)

**5. Retention:**
- Metrics stored for configured period (default: 7 days in our setup)
- Older data automatically deleted
- For longer retention, use persistent storage

**6. PromQL Query Examples:**

```promql
# Average CPU across all pods
avg(rate(container_cpu_usage_seconds_total[5m]))

# 95th percentile request latency
histogram_quantile(0.95, rate(flask_app_request_duration_seconds_bucket[5m]))

# Top 5 pods by memory usage
topk(5, sum(container_memory_usage_bytes) by (pod))

# Request rate by endpoint
sum(rate(flask_app_requests_total[5m])) by (endpoint)

# Error rate percentage
sum(rate(flask_app_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(flask_app_requests_total[5m])) * 100
```

---

### Common Monitoring Patterns

**1. RED Method (for Services):**
- **R**ate: Request rate (requests per second)
- **E**rrors: Error rate (failed requests)
- **D**uration: Request latency (response time)

Example queries:
```promql
# Rate
sum(rate(flask_app_requests_total[5m]))

# Errors
sum(rate(flask_app_requests_total{status=~"5.."}[5m]))

# Duration (95th percentile)
histogram_quantile(0.95, rate(flask_app_request_duration_seconds_bucket[5m]))
```

**2. USE Method (for Resources):**
- **U**tilization: % of resource used
- **S**aturation: Amount of queued work
- **E**rrors: Error count

Example queries:
```promql
# CPU Utilization
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Utilization
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# Disk saturation
node_disk_io_time_weighted_seconds_total
```

**3. The Four Golden Signals:**
- Latency: Time to serve requests
- Traffic: Demand on the system
- Errors: Rate of failed requests
- Saturation: How "full" the service is

---

### Monitoring Dashboard Best Practices

**1. Dashboard Organization:**
```
Top Row: High-level KPIs (SLIs)
  - Request rate, error rate, latency
  - Single-number stat panels

Middle Rows: Detailed Metrics
  - Time-series graphs
  - Per-endpoint breakdown
  - Resource usage trends

Bottom Row: Infrastructure
  - Node health
  - Pod status
  - Network metrics
```

**2. Time Ranges:**
- Default: Last 15 minutes (for active debugging)
- Production: Last 24 hours
- Always enable auto-refresh (5s-30s)

**3. Useful Variables:**
Create dashboard variables for filtering:
- `$namespace` - Filter by namespace
- `$pod` - Filter by pod name
- `$endpoint` - Filter by API endpoint

**4. Color Coding:**
- Green: Good (< 200ms latency, < 1% errors)
- Yellow: Warning (200-500ms, 1-5% errors)
- Red: Critical (> 500ms, > 5% errors)

---

### Troubleshooting Monitoring Stack

**Issue 1: Grafana Shows "No Data"**

**Symptoms:**
- Dashboards are empty
- "No data" message in panels

**Resolution:**

```bash
# Step 1: Check Prometheus data source
kubectl logs -n monitoring -l app.kubernetes.io/name=grafana | grep -i prometheus

# Step 2: Test Prometheus connectivity from Grafana pod
GRAFANA_POD=$(kubectl get pod -n monitoring -l app.kubernetes.io/name=grafana -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n monitoring $GRAFANA_POD -- wget -qO- http://prometheus-server:80/api/v1/query?query=up

# Step 3: Verify Prometheus service
kubectl get svc -n monitoring prometheus-server
```

**Issue 2: Prometheus Not Scraping Flask Pods**

**Symptoms:**
- Flask metrics not appearing in Prometheus
- Targets page doesn't show {{APP_NAMESPACE}} pods

**Resolution:**

```bash
# Step 1: Verify annotations on pods
kubectl get pod -n {{APP_NAMESPACE}} -o yaml | grep -A 3 "prometheus.io"

# Step 2: Test metrics endpoint directly
POD_NAME=$(kubectl get pod -n {{APP_NAMESPACE}} -l app={{APP_NAME}} -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n {{APP_NAMESPACE}} $POD_NAME -c {{APP_NAME}} -- wget -qO- localhost:5000/metrics

# Step 3: Check Prometheus config
kubectl get configmap -n monitoring prometheus-server -o yaml | grep -A 10 "kubernetes-pods"

# Step 4: Reload Prometheus configuration
kubectl delete pod -n monitoring -l app.kubernetes.io/name=prometheus
```

**Issue 3: High Memory Usage by Prometheus**

**Symptoms:**
- Prometheus pod OOMKilled or restarting
- High memory consumption

**Resolution:**

```bash
# Option 1: Reduce retention period
helm upgrade prometheus prometheus-community/prometheus \
  --set server.retention=3d \
  -n monitoring

# Option 2: Reduce scrape frequency
helm upgrade prometheus prometheus-community/prometheus \
  --set server.global.scrape_interval=30s \
  -n monitoring

# Option 3: Increase resource limits
helm upgrade prometheus prometheus-community/prometheus \
  --set server.resources.limits.memory=2Gi \
  -n monitoring
```

---

### Monitoring Architecture Decisions

**Why This Design:**

**1. Helm vs Manual Manifests**
- **Chose:** Helm charts
- **Reason:** Production-tested configurations, easier updates
- **Tradeoff:** Less control, more abstraction
- **Alternative:** Write all YAML manually for learning

**2. NodePort vs LoadBalancer**
- **Chose:** NodePort (30090, 30300)
- **Reason:** Works in local/Multipass environment
- **Tradeoff:** Less convenient than cloud LoadBalancer
- **Production:** Use Ingress + TLS

**3. No Persistent Storage**
- **Chose:** Disabled persistence
- **Reason:** Training environment, simplifies setup
- **Tradeoff:** Metrics lost on pod restart
- **Production:** Enable PersistentVolumes

**4. Single Prometheus Instance**
- **Chose:** One Prometheus server
- **Reason:** Sufficient for small cluster
- **Tradeoff:** Single point of failure
- **Production:** Deploy HA setup (2+ replicas) with Thanos

**5. Basic Auth for Grafana**
- **Chose:** Simple admin/password
- **Reason:** Quick setup for training
- **Tradeoff:** Less secure
- **Production:** Use OAuth/LDAP/SSO

---

### Key Takeaways

**What You Learned:**

1. **Metrics Collection Architecture**
   - Pull-based model (Prometheus scrapes targets)
   - Service discovery (automatic target detection)
   - Time-series storage

2. **Instrumentation**
   - Adding Prometheus client library
   - Exposing `/metrics` endpoint
   - Using annotations for discovery

3. **Visualization**
   - Connecting Grafana to Prometheus
   - Creating custom dashboards
   - Using PromQL queries

4. **GitOps Integration**
   - Helm charts as manifest source
   - ArgoCD managing monitoring stack
   - Infrastructure as code principles

5. **Observability Fundamentals**
   - RED method for services
   - USE method for resources
   - Four Golden Signals

**Production Enhancements:**

For production environments, consider:
- **Persistent storage** for metrics (PVC with cloud storage)
- **High availability** Prometheus (multiple replicas + Thanos)
- **Alertmanager** for alert routing (PagerDuty, Slack, email)
- **Distributed tracing** (Jaeger, Tempo) for request flows
- **Log aggregation** (Loki, ELK) for centralized logging
- **Security:** TLS, authentication, RBAC
- **Long-term storage:** Thanos, Cortex, or Mimir for metrics
- **SLO/SLA tracking** with error budgets

---

### Next Steps

**After completing Phase 7:**

1. **Explore Dashboards:**
   - Import community dashboards from https://grafana.com/grafana/dashboards/
   - Popular: 315 (cluster overview), 1860 (node exporter), 6417 (pods)

2. **Add More Metrics:**
   - Jenkins build metrics (via Prometheus plugin)
   - ArgoCD sync metrics (built-in Prometheus endpoint)
   - Custom business metrics in your app

3. **Learn PromQL:**
   - Practice queries in Prometheus UI
   - Experiment with aggregations and functions
   - Build complex multi-metric calculations

4. **Configure Alerts:**
   - Set up Alertmanager
   - Create meaningful alert rules
   - Avoid alert fatigue with proper thresholds

5. **Advanced Observability:**
   - Add distributed tracing (Jaeger)
   - Implement centralized logging (Loki)
   - Correlate metrics, logs, and traces

**Continue to:** [Teardown & Cleanup](#teardown--cleanup) when ready to clean up your environment.

---

### Quick Reference: Monitoring URLs

After completing Phase 7, you'll have:

| Service | URL | Credentials | Purpose |
|---------|-----|-------------|---------|
| **Prometheus** | http://{{CONTROL_PLANE_IP}}:{{PROMETHEUS_HTTP_PORT}} | None | Metrics database & query interface |
| **Grafana** | http://{{CONTROL_PLANE_IP}}:{{GRAFANA_HTTP_PORT}} | admin/admin123 | Dashboards & visualization |
| **Flask Metrics** | http://{{CONTROL_PLANE_IP}}:{{APP_HTTP_PORT}}/metrics | None | Application metrics endpoint |

**Prometheus UI Tabs:**
- **Graph:** Execute PromQL queries and visualize
- **Alerts:** View configured alert rules and firing alerts
- **Status → Targets:** See all scrape targets and health
- **Status → Configuration:** View current Prometheus config

**Grafana Navigation:**
- **Home:** Dashboard list
- **Explore:** Ad-hoc PromQL queries (like Prometheus Graph)
- **Alerting:** Alert rules and notifications
- **Configuration:** Data sources, users, preferences

---

### Resource Requirements

Adding Prometheus and Grafana will consume additional resources:

**CPU:**
- Prometheus: ~200-500m (0.2-0.5 cores)
- Grafana: ~100-500m (0.1-0.5 cores)
- Node Exporter: ~50m per node
- Kube-state-metrics: ~100m
- **Total:** ~500m-1200m additional CPU

**Memory:**
- Prometheus: ~512Mi-1Gi (grows with metrics volume)
- Grafana: ~256Mi-512Mi
- Node Exporter: ~30Mi per node
- Kube-state-metrics: ~100Mi
- **Total:** ~900Mi-1.7Gi additional memory

**Disk:**
- Prometheus: ~1-5Gi (depends on retention period)
- Grafana: ~100Mi
- **Total:** ~1-5Gi (if using persistent storage)

**Verify Cluster Capacity:**

```bash
# Check available node resources
kubectl top nodes

# Check current resource usage
kubectl top pods -A

# View resource requests/limits
kubectl describe nodes | grep -A 5 "Allocated resources"
```

If resources are tight, consider:
- Reducing Prometheus retention period
- Decreasing scrape interval (30s instead of 15s)
- Disabling node-exporter or kube-state-metrics temporarily
- Adjusting resource limits in values files

---

## Teardown & Cleanup

**When you're ready to purge everything:**

### Complete Teardown Process

```bash
# Step 1: Delete Argo CD applications (avoids hanging finalizers)
kubectl delete application {{APP_NAME}} -n {{ARGOCD_NAMESPACE}}

# Step 2: Delete namespaces (this will delete all resources)
kubectl delete namespace {{APP_NAMESPACE}}
kubectl delete namespace {{JENKINS_NAMESPACE}}
kubectl delete namespace {{ARGOCD_NAMESPACE}}
kubectl delete namespace vault
kubectl delete namespace monitoring

# Step 3: Wait for all resources to be deleted
kubectl get all -A

# Step 4: Uninstall K3s from nodes
multipass exec {{CONTROL_PLANE_HOSTNAME}} -- sudo /usr/local/bin/k3s-uninstall.sh
multipass exec {{WORKER_HOSTNAME}} -- sudo /usr/local/bin/k3s-agent-uninstall.sh

# Step 5: Destroy VMs with Terraform
cd {{PROJECT_BASE_PATH}}/terraform
terraform destroy -auto-approve

# Step 6: Verify VMs are gone
multipass list

# Step 7: Clean up any lingering Multipass instances
multipass delete --all
multipass purge

# Step 8: Remove local kubeconfig
rm {{PROJECT_BASE_PATH}}/kubeconfig*

# Step 9: Remove KUBECONFIG from shell profile
# Edit ~/.zshrc and remove the export KUBECONFIG line

# Step 10: Keep only the git repository
cd ~/Desktop
# {{PROJECT_NAME}}/ - KEEP (contains all documentation)
# {{PROJECT_NAME}}-{{APP_NAME}}/ - Can delete (backed up in GitHub)
# {{PROJECT_NAME}}-manifests/ - Can delete (backed up in GitHub)
```

**Verification:**
```bash
multipass list  # Should show: No instances found
docker ps       # Should show no K3s containers
```

### Docker Image Cleanup

**Clean up local Docker images:**

```bash
# List all Docker images
docker images

# Remove Jenkins custom image
docker rmi {{DOCKERHUB_USERNAME}}/{{JENKINS_IMAGE_NAME}}:lts

# Remove all application images (all versions)
docker images | grep {{APP_IMAGE_NAME}} | awk '{print $1":"$2}' | xargs docker rmi

# Clean up dangling images
docker image prune -a -f

# Verify cleanup
docker images
```

**Clean up Docker Hub images (optional):**

If you want to completely remove images from Docker Hub:

1. **Via Docker Hub Web UI:**
   - Go to: https://hub.docker.com/repositories/{{DOCKERHUB_USERNAME}}
   - Select repository: `{{JENKINS_IMAGE_NAME}}`
   - Click **Settings** → **Delete Repository**
   - Select repository: `{{APP_IMAGE_NAME}}`
   - Click **Settings** → **Delete Repository**

2. **Via Docker Hub CLI** (if installed):
```bash
# Login
docker login

# Delete repository
curl -X DELETE \
  -H "Authorization: Bearer $(cat ~/.docker/config.json | jq -r '.auths."https://index.docker.io/v1/".auth' | base64 -d | cut -d: -f2)" \
  https://hub.docker.com/v2/repositories/{{DOCKERHUB_USERNAME}}/{{JENKINS_IMAGE_NAME}}/

curl -X DELETE \
  -H "Authorization: Bearer $(cat ~/.docker/config.json | jq -r '.auths."https://index.docker.io/v1/".auth' | base64 -d | cut -d: -f2)" \
  https://hub.docker.com/v2/repositories/{{DOCKERHUB_USERNAME}}/{{APP_IMAGE_NAME}}/
```

### Local Tool Uninstallation (Optional)

**⚠️ Important Decision:** These tools are generally useful for development work. Only uninstall if you're certain you won't need them.

**If you want to keep your system pristine:**

```bash
# Uninstall tools installed via Homebrew
brew uninstall multipass
brew uninstall terraform
brew uninstall ansible
brew uninstall kubectl
# Note: Keep Git and Python as they're used system-wide

# Clean up Homebrew cache
brew cleanup

# Verify uninstallation
multipass version    # Should show: command not found
terraform version    # Should show: command not found
ansible --version    # Should show: command not found
kubectl version      # Should show: command not found
```

**If you want to completely remove Homebrew** (not recommended):

```bash
# Uninstall Homebrew (removes ALL brew-installed packages)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"

# Remove Homebrew directories
sudo rm -rf /opt/homebrew
```

**Clean up shell configuration:**

```bash
# Edit your shell profile
nano ~/.zshrc

# Remove these lines:
# export KUBECONFIG=~/Desktop/mygitopspipeline/kubeconfig
# Any Homebrew PATH additions

# Reload shell
source ~/.zshrc
```

### Complete Cleanup Summary

**What gets deleted:**
- ✅ All VMs ({{CONTROL_PLANE_HOSTNAME}}, {{WORKER_HOSTNAME}})
- ✅ All Kubernetes resources (pods, services, deployments)
- ✅ All namespaces ({{JENKINS_NAMESPACE}}, {{ARGOCD_NAMESPACE}}, {{APP_NAMESPACE}})
- ✅ Local kubeconfig files
- ✅ Local Docker images (only if you tested Phase 5.5 Docker socket approach)
- ✅ Docker Hub images (if you choose to delete)
- ✅ Local application directories (optional)

**What gets kept:**
- ✅ Git repositories ({{PROJECT_NAME}}, {{PROJECT_NAME}}-{{APP_NAME}}, {{PROJECT_NAME}}-manifests on GitHub)
- ✅ Documentation and code (in GitHub)
- ✅ Development tools (unless you choose to uninstall)
- ✅ Terraform state files (for reference)

**📝 Note:** If you followed the main workflow (Kaniko), there's no Docker daemon installed on the VMs to clean up!

**Disk Space Recovered:**
- VMs: ~20GB
- Docker images: ~2-3GB
- K3s components: ~500MB
- **Total: ~22-24GB**

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 0: Forgot to Replace Placeholders

**Symptoms:**
```
Error: Repository not found: https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}.git
Invalid URL format
Command failed: multipass exec {{CONTROL_PLANE_HOSTNAME}} -- hostname
```

**Root Cause:**
You're using the template document instead of your working guide with actual values.

**Solution:**

Go back to **Phase 0 - Configuration Variables** and create your working guide:
1. Copy the template to a new file
2. Use Find & Replace to update ALL `{{UPPERCASE_VARIABLES}}`
3. Run Step 0.5 validation before continuing
4. Follow your working guide (not the template)

**⚠️ Important:**
- **Replace:** `{{UPPERCASE_VARIABLES}}` (configuration placeholders)
- **Don't replace:** `{{lowercase_variables}}` (Ansible template syntax)

**Prevention:**
Run Step 0.5 validation checkpoint before Phase 1 to catch this early

#### Issue 1: Terraform Provider Failure

**Symptoms:**
```
Error: Failed to instantiate provider "larstobi/multipass"
Error: Plugin did not respond
```

**Root Cause:** The community Multipass provider may be incompatible with your Multipass version or architecture.

**Solutions:**

**Option A: Manual VM Creation (Recommended if Terraform fails)**
```bash
# Skip Terraform entirely and create VMs manually
cd {{PROJECT_BASE_PATH}}

# Create control plane VM
multipass launch 22.04 \
  --name {{CONTROL_PLANE_HOSTNAME}} \
  --cpus 2 \
  --memory 2G \
  --disk 10G \
  --cloud-init terraform/cloud-init-control.yaml

# Create worker VM
multipass launch 22.04 \
  --name {{WORKER_HOSTNAME}} \
  --cpus 2 \
  --memory 2G \
  --disk 10G \
  --cloud-init terraform/cloud-init-worker.yaml

# Verify
multipass list

# Continue to Phase 1.6 to discover IPs
```

**Option B: Update Multipass and Retry**
```bash
brew upgrade multipass
multipass version  # Should be 1.12+ for M1/M2

# Clean and retry Terraform
cd {{PROJECT_BASE_PATH}}/terraform
terraform destroy -auto-approve
rm -rf .terraform*
terraform init
terraform apply -auto-approve
```

#### Issue 2: VMs Won't Start

**Symptoms:**
```
Error: multipass instance failed to start
Timed out waiting for response
```

**Solutions:**
```bash
# Check Multipass daemon
multipass version

# Restart Multipass service
sudo launchctl unload /Library/LaunchDaemons/com.canonical.multipassd.plist
sudo launchctl load /Library/LaunchDaemons/com.canonical.multipassd.plist

# Check system resources
top  # Ensure enough CPU/RAM available
df -h  # Ensure enough disk space (need 20GB+)

# Check virtualization
# M1/M2 Macs: Multipass uses QEMU

multipass get local.driver
```

#### Issue 3: Ansible Connection Failures

**Symptoms:**
```
fatal: [k3s-control-01]: UNREACHABLE!
```

**Solutions:**
```bash
# Check VM connectivity
multipass list
ping -c 3 {{CONTROL_PLANE_IP}}

# Check SSH
multipass exec {{CONTROL_PLANE_HOSTNAME}} -- echo "Connected"

# Verify SSH keys
ls -la ~/.ssh/id_rsa*

# Test manual SSH
ssh ubuntu@{{CONTROL_PLANE_IP}}
```

#### Issue 4: K3s Installation Fails

**Symptoms:**
```
Error: K3s service failed to start
```

**Solutions:**
```bash
# Check K3s logs on control plane
multipass exec {{CONTROL_PLANE_HOSTNAME}} -- sudo journalctl -u k3s -n 50

# Check firewall
multipass exec {{CONTROL_PLANE_HOSTNAME}} -- sudo ufw status

# Ensure port 6443 is open
multipass exec {{CONTROL_PLANE_HOSTNAME}} -- sudo netstat -tlnp | grep 6443

# Reinstall K3s
multipass exec {{CONTROL_PLANE_HOSTNAME}} -- sudo /usr/local/bin/k3s-uninstall.sh
# Then re-run Ansible playbook
```

#### Issue 5: Kaniko Build Fails

**Symptoms:**
```
ERROR: error building image: error retrieving image from registry
ERROR: failed to push image: unauthorized: authentication required
```

**Root Cause:** Kaniko cannot authenticate to Docker Hub

**Solutions:**

**Check 1: Verify Kubernetes Secret Exists**
```bash
# Check if secret exists
kubectl get secret dockerhub-credentials -n {{JENKINS_NAMESPACE}}

# If missing, create it (use your Docker Hub access token)
kubectl create secret docker-registry dockerhub-credentials \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username={{DOCKERHUB_USERNAME}} \
  --docker-password=YOUR_DOCKERHUB_TOKEN \
  --docker-email=YOUR_EMAIL \
  -n {{JENKINS_NAMESPACE}}
```

**Check 2: Verify Secret Format**
```bash
# Secret should have .dockerconfigjson key
kubectl get secret dockerhub-credentials -n {{JENKINS_NAMESPACE}} -o jsonpath='{.data}'

# Should show: {"\.dockerconfigjson":"base64-encoded-string"}
```

**Check 3: Check Docker Hub Token Permissions**
- Go to: https://hub.docker.com/settings/security
- Verify token has **Read, Write** permissions
- Generate new token if needed
- Update secret with new token:
```bash
kubectl delete secret dockerhub-credentials -n {{JENKINS_NAMESPACE}}
kubectl create secret docker-registry dockerhub-credentials \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username={{DOCKERHUB_USERNAME}} \
  --docker-password=NEW_TOKEN \
  --docker-email=YOUR_EMAIL \
  -n {{JENKINS_NAMESPACE}}
```

**Check 4: Verify Kaniko Pod Can Access Secret**
```bash
# Check Jenkins build logs for volume mount
kubectl logs -n {{JENKINS_NAMESPACE}} -l jenkins=agent --tail=50

# Should see: Successfully pulled image "gcr.io/kaniko-project/executor:v1.9.0-debug"
```

**Alternative:** If Kaniko continues to fail, see Phase 5.5 for Docker socket approach (not recommended for production)

#### Issue 6: Jenkins Can't Spawn Kaniko Pods

**Symptoms:**
```
ERROR: Kubernetes Pod waiting to be scheduled
ERROR: Pod is not running: hello-gitops-pipeline-X-xxxxx
```

**Root Cause:** Jenkins lacks permissions to create pods or pods can't be scheduled

**Solutions:**

**Check 1: Verify Jenkins RBAC**
```bash
# Check if Jenkins service account has pod creation rights
kubectl describe clusterrolebinding jenkins

# Should show jenkins service account with pod create/delete permissions
```

**Check 2: Check Node Resources**
```bash
# Check if nodes have capacity
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"

# If nodes full, scale down other services temporarily
kubectl scale deployment jenkins -n {{JENKINS_NAMESPACE}} --replicas=0
kubectl scale deployment jenkins -n {{JENKINS_NAMESPACE}} --replicas=1
```

**Check 3: Check Pod Events**
```bash
# Watch pod creation
kubectl get pods -n {{JENKINS_NAMESPACE}} -w

# Check events
kubectl get events -n {{JENKINS_NAMESPACE}} --sort-by='.lastTimestamp' | tail -20
```

**Check 4: Manually Test Pod Template**
```bash
# Save the pod template from Jenkinsfile to a file
cat > test-kaniko-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: test-kaniko
  namespace: {{JENKINS_NAMESPACE}}
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.9.0-debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  volumes:
  - name: docker-config
    secret:
      secretName: dockerhub-credentials
EOF

# Try creating pod manually
kubectl apply -f test-kaniko-pod.yaml

# Check if it starts
kubectl get pod test-kaniko -n {{JENKINS_NAMESPACE}}

# Clean up
kubectl delete pod test-kaniko -n {{JENKINS_NAMESPACE}}
```

#### Issue 7: Argo CD Won't Sync

**Symptoms:**
```
Application status: OutOfSync
```

**Solutions:**
```bash
# Check Argo CD logs
kubectl logs -n {{ARGOCD_NAMESPACE}} deployment/argocd-server

# Verify GitHub repository access
kubectl exec -n {{ARGOCD_NAMESPACE}} deployment/argocd-repo-server -- git ls-remote https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}-manifests.git

# Manual sync
kubectl patch application {{APP_NAME}} -n {{ARGOCD_NAMESPACE}} --type merge -p '{"operation":{"sync":{}}}'

# Check application status
kubectl get application {{APP_NAME}} -n {{ARGOCD_NAMESPACE}} -o yaml
```

#### Issue 8: Application Can't Pull Image

**Symptoms:**
```
Failed to pull image "{{DOCKERHUB_USERNAME}}/{{PROJECT_NAME}}-{{APP_NAME}}:X": rpc error: code = Unknown
```

**Solutions:**
```bash
# Check if dockerhub-secret exists in the namespace
kubectl get secret dockerhub-secret -n {{APP_NAMESPACE}}

# If missing, create Docker Hub secret
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username={{DOCKERHUB_USERNAME}} \
  --docker-password=YOUR_DOCKERHUB_TOKEN \
  -n {{APP_NAMESPACE}}

# Verify deployment has imagePullSecrets configured
kubectl get deployment {{APP_NAME}} -n {{APP_NAMESPACE}} -o yaml | grep -A 2 imagePullSecrets

# If imagePullSecrets is missing from deployment, patch it
kubectl patch deployment {{APP_NAME}} -n {{APP_NAMESPACE}} -p '{"spec":{"template":{"spec":{"imagePullSecrets":[{"name":"dockerhub-secret"}]}}}}'

# Restart pods to apply changes
kubectl rollout restart deployment {{APP_NAME}} -n {{APP_NAMESPACE}}
```

#### Issue 9: Browser Can't Access Services (Chrome HSTS Issue)

**Symptoms:**
```
Chrome automatically redirects to HTTPS
Connection refused
```

**Solutions:**
```bash
# Option 1: Use Safari or Firefox
open -a Safari http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}}

# Option 2: Clear Chrome HSTS
# Go to: chrome://net-internals/#hsts
# Delete domain: {{CONTROL_PLANE_IP}}

# Option 3: Use kubectl port-forward
kubectl port-forward -n {{JENKINS_NAMESPACE}} svc/jenkins 8080:8080
# Access: http://localhost:8080
```

#### Issue 10: Pods Stuck in Pending

**Symptoms:**
```
NAME              READY   STATUS    RESTARTS   AGE
{{APP_NAME}}-xxx  0/1     Pending   0          2m
```

**Solutions:**
```bash
# Check pod events
kubectl describe pod -n {{APP_NAMESPACE}} | grep -A 10 Events

# Check node resources
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check PVC status
kubectl get pvc -A

# Scale down other services if needed
kubectl scale deployment jenkins -n {{JENKINS_NAMESPACE}} --replicas=0
```

### Debug Commands Reference

```bash
# View all resources across all namespaces
kubectl get all -A

# Check pod logs
kubectl logs -n NAMESPACE POD_NAME

# Check pod details
kubectl describe pod -n NAMESPACE POD_NAME

# Check node status
kubectl describe nodes

# Check events
kubectl get events -A --sort-by='.lastTimestamp'

# Shell into pod
kubectl exec -it -n NAMESPACE POD_NAME -- /bin/sh

# Port forward for debugging
kubectl port-forward -n NAMESPACE svc/SERVICE_NAME LOCAL_PORT:REMOTE_PORT

# Check resource usage
kubectl top nodes
kubectl top pods -A

# View logs from all pods in namespace
kubectl logs -n NAMESPACE -l app=APP_LABEL --tail=100

# Force delete stuck resources
kubectl delete pod POD_NAME -n NAMESPACE --grace-period=0 --force
```

---

#### Issue 11: Jenkins & ArgoCD UI Not Accessible After PC Sleep/Restart

**Symptoms:**
```bash
# UIs are not accessible in browser
# Jenkins: http://{{WORKER_HOSTNAME}}:{{JENKINS_HTTP_PORT}} - Connection refused
# ArgoCD: http://{{WORKER_HOSTNAME}}:{{ARGOCD_HTTP_PORT}} - Connection refused

# But VMs show as Running
multipass list
Name                              State             IPv4             Image
{{CONTROL_PLANE_HOSTNAME}}        Running           {{CONTROL_PLANE_IP}}     Ubuntu 22.04 LTS
{{WORKER_HOSTNAME}}               Running           {{WORKER_NODE_IP}}     Ubuntu 22.04 LTS

# Kubernetes pods are running
kubectl get pods -n jenkins
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-xxxxx-xxxxx        1/1     Running   0          10m

kubectl get pods -n argocd
NAME                                    READY   STATUS    RESTARTS   AGE
argocd-server-xxxxx-xxxxx               1/1     Running   0          10m
```

**Root Cause:**

Mac sleep mode breaks the Multipass network routing between your Mac and the VMs. The VMs are running, services are healthy, but the Mac → VM network path is broken.

**Resolution:**

**Step 1: Test VM connectivity**
```bash
# Try to ping the VMs
ping {{CONTROL_PLANE_IP}}
ping {{WORKER_NODE_IP}}

# If ping fails, the network is broken
```

**Step 2: Restart VMs (Most reliable fix)**
```bash
# Stop both VMs
multipass stop {{CONTROL_PLANE_HOSTNAME}} {{WORKER_HOSTNAME}}

# Start them back up
multipass start {{CONTROL_PLANE_HOSTNAME}} {{WORKER_HOSTNAME}}

# Wait 30-60 seconds for K3s to fully start
sleep 60

# Verify VMs are accessible
ping {{CONTROL_PLANE_IP}}

# Test UI access
open http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}}  # Jenkins
open http://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTP_PORT}}  # ArgoCD
```

**Step 3: If UIs accessible but some pods failing**

After VM restart, some pods may be in `Unknown`, `Completed`, or `CrashLoopBackOff` state:

```bash
# Check pod status across all namespaces
kubectl get pods -A | grep -E 'Unknown|Completed|Error|CrashLoop'

# Common issues after restart:
# 1. argocd-repo-server in Completed state
kubectl delete pod -n argocd -l app.kubernetes.io/name=argocd-repo-server

# 2. argocd-applicationset-controller in CrashLoopBackOff
kubectl delete pod -n argocd -l app.kubernetes.io/name=argocd-applicationset-controller

# 3. Application pods stuck in Unknown state
kubectl delete pod --all -n {{APP_NAMESPACE}}
```

**Step 4: Check for Vault authentication issues**

If pods remain in `Init:0/1` state, see **Issue 12: Pods stuck at Init:0/1 state after VM Restart** below for Vault reconfiguration steps.


**Prevention:**

For production or long-running training environments, consider:
- Using a dedicated Linux VM instead of Multipass
- Cloud-based K8s clusters (EKS, GKE, AKS)
- Keeping your Mac from sleeping during active training sessions

---

#### Issue 12: Pods stuck at Init:0/1 state after VM Restart

**⚠️ Vault Dev Mode Limitation**

This training setup uses Vault in development mode with in-memory storage (`Storage Type: inmem`). This means:

- ✅ **Easy setup** - No persistent storage required
- ❌ **Data loss on restart** - All secrets, auth configurations, and policies are lost when VMs restart
- 🔄 **Requires reconfiguration** - You must manually reconfigure Vault after every cluster/VM restart

### Symptoms After VM Restart:

```bash
# Application pods stuck in Init:0/1 or CrashLoopBackOff
kubectl get pods -n {{APP_NAMESPACE}}
NAME                               READY   STATUS     RESTARTS   AGE
sample-flask-app-xxxxx-xxxxx       0/2     Init:0/1   0          2m

# Vault agent logs show authentication errors
kubectl logs POD_NAME -n {{APP_NAMESPACE}} -c vault-agent-init
[ERROR] agent.auth.handler: error authenticating: error="permission denied"
```

### Recovery Steps:

**Step 1: Verify Vault is running and unsealed**
```bash
kubectl get pods -n vault
kubectl exec -it vault-0 -n vault -- vault status
# Should show: Sealed = false
```

**Step 2: Reconfigure Kubernetes Authentication**
```bash
kubectl exec -it vault-0 -n vault -- sh

# Inside Vault pod - enable Kubernetes auth
vault auth enable kubernetes

# Configure Vault to trust K8s API
vault write auth/kubernetes/config \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"

# Create policy for app secrets
vault policy write {{APP_NAME}}-policy - <<EOF
path "secret/data/{{APP_NAME}}/*" {
  capabilities = ["read"]
}
EOF

# Create role binding policy to Service Account
vault write auth/kubernetes/role/{{APP_NAME}} \
    bound_service_account_names=default \
    bound_service_account_namespaces={{APP_NAMESPACE}} \
    policies={{APP_NAME}}-policy \
    ttl=24h

# Exit Vault pod
exit
```

**Step 3: Re-store Application Secrets**
```bash
# Write application secrets to Vault
kubectl exec -it vault-0 -n vault -- \
  vault kv put secret/{{APP_NAME}}/config \
  app_secret="ProductionSecretValue2024" \
  environment="production"

# Verify (optional)
kubectl exec -it vault-0 -n vault -- \
  vault kv get secret/{{APP_NAME}}/config
```

**Step 4: Clean Up Failed Pods**
```bash
# Delete pods stuck in Unknown or Init state
kubectl delete pod --all -n {{APP_NAMESPACE}}

# ArgoCD will automatically recreate them
kubectl get pods -n {{APP_NAMESPACE}} -w
```

**Expected Result:**
- Pods should transition from `Init:0/1` → `Running`
- ArgoCD application health: `Degraded` → `Healthy`

### Production Recommendation:

For production environments, use Vault with persistent storage:
- **Integrated Storage (Raft)** - Vault's built-in HA storage
- **Consul** - Distributed key-value store
- **Cloud Storage** - AWS S3, Azure Blob, GCP Storage

This ensures secrets and configurations survive restarts.


#### Issue 13: Vault Agent Injector Not Injecting Sidecars After System Sleep

**Problem:**

After your Mac goes to sleep or restarts, new application pods are created without Vault Agent sidecar containers, causing them to crash with:

```bash
/bin/sh: 1: .: cannot open /vault/secrets/config: No such file
```

**Symptoms:**

```bash
# Pods have only 1 container instead of 2 (missing vault-agent)
kubectl get pods -n test-app-namespace
NAME                               READY   STATUS             RESTARTS   AGE
test-application-xxxxx-xxxxx       0/1     CrashLoopBackOff   5          3m

# No vault-agent container exists
kubectl logs POD_NAME -n test-app-namespace -c vault-agent-init
error: container vault-agent-init is not valid for pod POD_NAME

# Vault Agent Injector shows TLS errors
kubectl logs -n vault vault-agent-injector-xxxxx-xxxxx --tail=50
[ERROR] handler: http: TLS handshake error from 10.42.0.0:xxxxx: remote error: tls: bad certificate
```

**Root Cause:**

The Vault Agent Injector's TLS certificates become invalid after system sleep/wake cycles. The Kubernetes API server can't establish a secure connection to the mutating webhook, so pod mutations (sidecar injection) fail silently due to `failurePolicy: Ignore`.

**Solution:**

Restart the Vault Agent Injector pod to regenerate TLS certificates:

```bash
# Find and delete the injector pod
kubectl delete pod -n vault -l app.kubernetes.io/name=vault-agent-injector

# Wait for it to restart
kubectl get pods -n vault -w

# Verify new pod is running
kubectl get pods -n vault | grep injector

# Trigger application redeployment
kubectl rollout restart deployment/test-application -n test-app-namespace

# Watch pods come up with vault sidecars (should show 2/2)
kubectl get pods -n test-app-namespace -w
```

**Verification:**

```bash
# Check injector logs - should see successful mutation requests
kubectl logs -n vault -l app.kubernetes.io/name=vault-agent-injector --tail=20

# Pods should now have 2 containers
kubectl get pods -n test-app-namespace
NAME                               READY   STATUS    RESTARTS   AGE
test-application-xxxxx-xxxxx       2/2     Running   0          1m
```

**Prevention:**

For production or extended training sessions:
- Use persistent Vault with proper certificate rotation
- Configure webhook `failurePolicy: Fail` to catch injection failures immediately
- Monitor webhook health and certificate expiration


---

## Future Enhancements

**Once you've mastered the core GitOps pipeline, consider these advanced topics:**

### Level 1: Advanced Observability
- Alertmanager for alert routing and management
- Distributed tracing with Jaeger or Tempo
- Log aggregation with Loki or ELK stack
- Correlating metrics, logs, and traces

### Level 2: Security Hardening
- Network policies for pod isolation
- OPA/Gatekeeper for policy enforcement
- Trivy or Clair for container scanning
- RBAC fine-tuning and least privilege
- Sealed Secrets as alternative to Vault

### Level 3: High Availability & Resilience
- Multi-node control plane (3+ servers)
- Load balancer integration (MetalLB)
- Backup and restore (Velero)
- Disaster recovery procedures

### Level 4: Advanced CI/CD Patterns
- Blue-green deployments
- Canary releases with Argo Rollouts
- Progressive delivery strategies
- Multi-environment workflows (dev/staging/prod)

### Level 5: Multi-Cluster & Cloud
- Cluster federation
- Multi-cluster Argo CD
- Hybrid cloud deployments (AWS/Azure/GCP)
- Service mesh (Istio/Linkerd)

### Level 6: Developer Experience
- Automated PR previews
- Local development with Tilt or Skaffold
- GitOps for infrastructure (Crossplane)
- Internal developer platforms

### Level 7: Cost & Performance Optimization
- Horizontal Pod Autoscaling (HPA)
- Vertical Pod Autoscaling (VPA)
- Cluster autoscaling
- Resource quotas and limits
- Cost monitoring and optimization

### Level 8: Compliance & Governance
- Audit logging
- Compliance reporting
- RBAC hardening
- Image signing and verification

---

## Appendix: Quick Reference

### Service Access URLs

| Service | URL | Credentials |
|---------|-----|-------------|
| Jenkins | http://{{CONTROL_PLANE_IP}}:{{JENKINS_HTTP_PORT}} | admin / admin |
| Argo CD (HTTP) | http://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTP_PORT}} | admin / (from secret) |
| Argo CD (HTTPS) | https://{{CONTROL_PLANE_IP}}:{{ARGOCD_HTTPS_PORT}} | admin / (from secret) |
| {{APP_NAME}} | http://{{CONTROL_PLANE_IP}}:{{APP_HTTP_PORT}} | N/A |
| Prometheus | http://{{CONTROL_PLANE_IP}}:{{PROMETHEUS_HTTP_PORT}} | N/A |
| Grafana | http://{{CONTROL_PLANE_IP}}:{{GRAFANA_HTTP_PORT}} | admin / admin123 |

### Important File Locations

| Item | Location |
|------|----------|
| Kubeconfig | `{{PROJECT_BASE_PATH}}/kubeconfig` |
| Terraform | `{{PROJECT_BASE_PATH}}/terraform/` |
| Ansible | `{{PROJECT_BASE_PATH}}/ansible/` |
| Kubernetes Manifests | `{{PROJECT_BASE_PATH}}/kubernetes/` |
| Application Code | `~/Desktop/{{PROJECT_NAME}}-{{APP_NAME}}/` |
| GitOps Manifests | `~/Desktop/{{PROJECT_NAME}}-manifests/` |

### VM Information

| VM | IP | Role | Resources |
|----|----|-|-----------|
| {{CONTROL_PLANE_HOSTNAME}} | {{CONTROL_PLANE_IP}} | Control Plane | 2 CPU, 3GB RAM, 10GB disk |
| {{WORKER_HOSTNAME}} | {{WORKER_NODE_IP}} | Worker Node | 2 CPU, 3GB RAM, 10GB disk |

### Common Commands

```bash
# Cluster status
kubectl get nodes
kubectl cluster-info

# List all resources
kubectl get all -A

# Check specific namespace
kubectl get all -n NAMESPACE

# Logs
kubectl logs -n NAMESPACE POD_NAME

# Restart deployment
kubectl rollout restart deployment DEPLOYMENT_NAME -n NAMESPACE

# Scale deployment
kubectl scale deployment DEPLOYMENT_NAME -n NAMESPACE --replicas=N

# Delete and recreate pod
kubectl delete pod POD_NAME -n NAMESPACE
```

---

## Conclusion

**Congratulations!** You now have a **production-grade** GitOps pipeline running on your local machine using industry best practices.

**What You've Built:**
- ✅ Infrastructure as Code (Terraform)
- ✅ Configuration Management (Ansible)
- ✅ Kubernetes Cluster (K3s with containerd)
- ✅ Continuous Integration (Jenkins with Kubernetes agents)
- ✅ **Rootless Image Builds (Kaniko)** 🔒
- ✅ Continuous Deployment (Argo CD)
- ✅ Secret Management (HashiCorp Vault)
- ✅ Observability & Monitoring (Prometheus & Grafana)
- ✅ Sample Application (Flask)
- ✅ Complete GitOps Workflow

**Skills Mastered:**
- Infrastructure provisioning
- Configuration automation
- Container orchestration
- **Production-grade CI/CD pipeline design**
- **Kubernetes-native build workflows**
- **Security best practices (rootless builds)**
- Secret management and dynamic injection
- Observability and monitoring
- Metrics collection and visualization
- GitOps principles
- Kubernetes operations
- Troubleshooting and debugging

**🎓 Why This is Production-Ready:**
- ✅ **No privileged access required** - Kaniko builds without Docker daemon
- ✅ **Kubernetes-native** - Builds run in ephemeral pods
- ✅ **Security by design** - No Docker socket mounting
- ✅ **Scalable** - Dynamic pod creation for builds
- ✅ **Audit-friendly** - All operations logged via Kubernetes events
- ✅ **GitOps compliant** - Declarative configuration, Git as source of truth

**Comparison to Legacy Approaches:**

| Aspect | Your Pipeline (Kaniko) | Legacy (Docker Socket) |
|--------|----------------------|------------------------|
| **Security** | ✅ Production-ready | ❌ Security anti-pattern |
| **Privileges** | ✅ No root required | ❌ Root-equivalent access |
| **Isolation** | ✅ Build isolation | ❌ Shared resources |
| **Dependencies** | ✅ No daemon | ❌ Requires Docker daemon |
| **Kubernetes Integration** | ✅ Native | ⚠️ Host dependency |

**Next Steps:**
1. Experiment with the pipeline (modify app, trigger builds)
2. Build your own applications using this pattern
3. Deploy to cloud Kubernetes (EKS, AKS, GKE) - same principles apply!
4. Explore the future enhancements (monitoring, security scanning)
5. Share your knowledge - you now know production practices!

**Optional Learning:**
- See Phase 5.5 to understand the Docker socket approach (for comparison only)
- This helps you recognize anti-patterns when reviewing existing systems

**Documentation Repository:**
https://github.com/{{GITHUB_USERNAME}}/{{PROJECT_NAME}}

---

**Document Version:** 2.0 (Production-Grade Edition)  
**Last Updated:** January 2026  
**Author:** github.com/omeyazic  
**Status:** Production Ready ✅ | Security Hardened 🔒 | Best Practices ⭐

---

*"Secure by default, Production-ready from day one"*

**🎖️ You've Learned:**
- Not just "how to make it work"
- But "how to make it work **securely** and **correctly**"
- Skills that translate directly to enterprise environments

