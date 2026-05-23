
# Distributed Inferencing Prototype Deployment

## Overview

This project deploys a distributed inference architecture across multiple AWS EC2 instances using isolated networking.

The deployment consists of:
- A public API VM running the TypeScript caller-worker
- A private worker VM running the Python inference-worker
- Private subnet communication between workers using RPC concepts
- Infrastructure isolation using AWS VPC, subnets, route tables, and security groups

The inference worker loads the Gemma-3-270m GGUF model using Hugging Face Transformers.

---

# Architecture Diagram

```text
                 Internet
                     │
                     ▼
        ┌─────────────────────────┐
        │ API VM (Public Subnet) │
        │-------------------------│
        │ caller-worker           │
        │ HTTP endpoint           │
        └──────────┬──────────────┘
                   │ Private RPC
                   ▼
       ┌──────────────────────────┐
       │ Worker VM (Private Subnet) │
       │----------------------------│
       │ inference-worker           │
       │ Gemma-3-270m GGUF          │
       └────────────────────────────┘
```

## AWS Infrastructure

The following infrastructure was provisioned on AWS:

Custom VPC
Public subnet for API VM
Private subnet for worker VM
Internet Gateway
Route tables
Security groups
EC2 instances
SSH access through PEM keys

The worker VM was intentionally isolated inside a private subnet.

Temporary public access was enabled during dependency installation and later removed to restore private subnet isolation.
## Deployment Steps
# API VM Setup
```text
sudo apt update
curl -fsSL https://bun.sh/install | bash
source ~/.bashrc

git clone https://github.com/Alchemyst-ai/hiring.git

cd hiring/may-2026/devops/quickstart/workers/caller-worker

bun install
npm install

bun run dev
```
# Worker VM Setup
```text
sudo apt update
sudo apt install -y python3-full python3.12-venv python3-pip git unzip

git clone https://github.com/Alchemyst-ai/hiring.git

cd hiring/may-2026/devops/quickstart/workers/inference-worker

python3 -m venv venv
source venv/bin/activate

pip install --upgrade pip

pip install torch --index-url https://download.pytorch.org/whl/cpu

pip install iii-sdk==0.11.0 watchfiles transformers gguf accelerate
```
## Worker Communication

The TypeScript caller-worker attempts to connect to:
```text
ws://localhost:49134
```
and triggers:

```text
and triggers:
```
inside the Python inference-worker.

The inference-worker exposes:
```text
iii.register_function("inference::run_inference", run_inference_handler)
```
## Runtime Blocker

The provided quickstart project depends on an III runtime engine expected to listen on:
```text
ws://localhost:49134
```
The runtime installation endpoints referenced in the official documentation currently return unavailable/404 responses.

Infrastructure provisioning, dependency installation, worker deployment, private networking, and distributed architecture setup were completed successfully, but complete end-to-end RPC execution could not be finalized because the upstream runtime dependency was unavailable.

The caller-worker logs confirm runtime connection attempts:

```text
ECONNREFUSED 127.0.0.1:49134
```
## Screenshots

The repository includes screenshots for:

EC2 instances
VPC networking
Route tables
Security groups
API VM terminal
Worker VM terminal
Caller-worker logs
Private subnet communication
## Production Improvements

If this system were productionized, the following improvements would be implemented:

NAT Gateway for outbound private subnet internet access
Docker containerization
Kubernetes/EKS orchestration
Centralized logging and monitoring
IAM roles instead of SSH-based access
Auto Scaling Groups
Load balancing
Secrets management
CI/CD pipelines
## Scaling for Larger Models

If the model were significantly larger:

GPU EC2 instances would be required
Quantized inference runtimes such as vLLM/TGI would be used
Model sharding/distributed inference would be implemented
Object storage would host model artifacts
Autoscaling inference pools would be added
Token streaming optimizations would be implemented
# Terraform

Basic Terraform configuration is included for:
- VPC
- Public/private subnets
- Internet Gateway
- Route tables
- Security groups

Terraform files are located inside the terraform/ directory.
## Repository Structure
```text
.
distributed-inferencing-devops-assignment/
│
├── README.md
├── .gitignore
│
├── screenshots/
│   ├── EC2-instances.png
│   ├── api-vm-interface.png
│   ├── api-vm.png
│   ├── caller-worker-logs.png
│   ├── route-table.png
│   ├── security-groups.png
│   ├── subnet.png
│   ├── worker-vm-interface.png
│   └── worker-vm.png
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│
└── workers/
    ├── caller-worker/
    │   └── src/
    │       └── worker.ts
    │
    └── inference_worker/
        ├── inference_worker.py
        └── requirements.txt
```
