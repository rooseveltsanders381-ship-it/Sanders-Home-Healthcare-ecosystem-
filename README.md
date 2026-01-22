# 🚀 Sanders Legacy Trust Platforms - Complete Deployment Guide

**Authority:** Sanders Family Living Trust  
**Founder:** Roosevelt Sanders  
**Certification:** FREEDOM33-GOLD

---

## 📋 Overview

You now have **4 complete deployment systems** that work together:

1. **GitHub Repository Structure** - Single source of truth for all platform code
2. **VM Deployment** - 33 individual VMs (one per platform)
3. **Docker Deployment** - 33 containers on 3 hosts
4. **GitHub Actions** - Automated CI/CD for both deployment types

---

## 🎯 Quick Start (Choose Your Path)

### Option A: Deploy Individual VMs (Recommended for Production)
```bash
# 1. Set up your GitHub repo
git clone https://github.com/rooseveltsanders381-ship-it/sanders-legacy-trust-platforms.git
cd sanders-legacy-trust-platforms

# 2. Create the repo structure (from Artifact #1)
# Copy the directory structure and create all folders

# 3. Deploy 30 new VMs (you already have 3)
cd deployment/vm
chmod +x create_vms.sh
./create_vms.sh
```

### Option B: Deploy Docker Containers (Fast Testing/Development)
```bash
# 1. Convert your 3 existing VMs to Docker hosts
cd deployment/docker
chmod +x deploy_docker.sh
./deploy_docker.sh
```

### Option C: Both (Hybrid - Best Overall)
```bash
# Keep your existing 3 VMs as-is
# Create 30 new VMs for production
# Set up Docker on 3 different hosts for testing
```

---

## 📂 Step 1: Set Up GitHub Repository

### A. Create Repository Structure

Based on **Artifact #1**, create this structure in your repo:

```bash
cd sanders-legacy-trust-platforms

# Create main directories
mkdir -p platforms/{sanders-sentinel,sanders-omniconm,sanders-grantwriter}
mkdir -p platforms/{lil-mama,baby-girl,gai-mind,ai-doctor,patriot-saint}
mkdir -p platforms/{sanders-home-healthcare,sanders-senior-living,sanders-legal-helpers}
mkdir -p platforms/{sanders-education,sanders-finance,sanders-retail,sanders-logistics}
mkdir -p platforms/{sanders-security,sanders-real-estate,sanders-energy,sanders-transportation}
mkdir -p platforms/{sanders-agriculture,sanders-manufacturing,sanders-hospitality}
mkdir -p platforms/{sanders-entertainment,sanders-sports,sanders-wellness,sanders-travel}
mkdir -p platforms/{sanders-ai-research,sanders-research,sanders-media}
mkdir -p platforms/{sanders-communications,sanders-compliance,sanders-coordinator,sanders-consulting}

mkdir -p shared deployment/{vm,docker,zero_trust} infrastructure/{terraform,gcp}
mkdir -p monitoring/{prometheus,grafana} certification docs tests
mkdir -p .github/workflows
```

### B. Add Platform Code

For each platform directory, create at minimum:
- `main.py` - Platform entry point
- `requirements.txt` - Python dependencies
- `config.json` - Platform configuration
- `naics.json` - NAICS codes and bridges
- `README.md` - Platform documentation

### C. Commit and Push

```bash
git add .
git commit -m "Initial Sanders Legacy Trust Platform structure"
git push origin main
```

---

## 🖥️ Step 2: Deploy Individual VMs

### Prerequisites
- GCP project with credits
- gcloud CLI installed and configured
- 3 existing VMs already running (you have these)

### Deployment

```bash
cd deployment/vm

# Edit the script with your project ID
nano create_vms.sh
# Change: PROJECT_ID="your-project-id"

# Make executable
chmod +x create_vms.sh

# Run deployment
./create_vms.sh

# Wait 10-15 minutes for all VMs to be created and initialized
```

### Verify VMs

```bash
# List all your VMs
gcloud compute instances list --filter="labels.authority=sanders-legacy-trust"

# Check a specific VM's logs
gcloud compute instances get-serial-port-output instance-YYYYMMDD-HHMMSS-platform-name --zone=us-central1-c

# SSH into a VM
gcloud compute ssh instance-YYYYMMDD-HHMMSS-platform-name --zone=us-central1-c
```

### Result
You now have **33 total VMs:**
- 3 existing (Sentinel, Omniconm, Grantwriter)
- 30 newly created

Each VM:
- Runs one platform
- Has Nginx reverse proxy
- Auto-starts on boot
- Pulls code from your GitHub repo

---

## 🐳 Step 3: Deploy Docker Containers

### Prerequisites
- 3 VMs with external IPs
- SSH access configured
- Docker will be auto-installed by the script

### Deployment

```bash
cd deployment/docker

# Edit the script
nano deploy_docker.sh
# Change: PROJECT_ID="your-project-id"

# Make executable
chmod +x deploy_docker.sh

# Run deployment
./deploy_docker.sh
```

### What This Does

1. **Installs Docker** on all 3 hosts
2. **Builds unified Docker image** containing all platform code
3. **Deploys 11 containers** to each host:
   - Host 1 (34.133.172.131): Guardians + Critical (ports 3001-3011)
   - Host 2 (35.238.209.6): Operations (ports 3012-3022)
   - Host 3 (34.27.79.1): Support Services (ports 3023-3033)

### Verify Docker Deployment

```bash
# Check containers on Host 1
ssh deploy@34.133.172.131 "docker ps"

# Check container health
ssh deploy@34.133.172.131 "curl http://localhost:3001/health"

# View logs for a container
ssh deploy@34.133.172.131 "docker logs sanders_sentinel"
```

---

## ⚙️ Step 4: Set Up GitHub Actions

### A. Add GitHub Secrets

Go to your repo → Settings → Secrets and variables → Actions

Add these secrets:
- `GCP_PROJECT_ID` - Your GCP project ID
- `GCP_SA_KEY` - Service account key JSON (for gcloud auth)
- `SSH_PRIVATE_KEY` - Your SSH private key for deploying to VMs

### B. Copy Workflow Files

From **Artifact #4**, create these files in `.github/workflows/`:

```bash
.github/workflows/
├── deploy-vm.yml          # Auto-deploy to VMs on push
├── deploy-docker.yml      # Auto-deploy to Docker on push
├── test-platforms.yml     # Run tests on PR
├── backup.yml             # Daily automated backups
└── security-scan.yml      # Weekly security scanning
```

### C. Trigger Deployments

**Automatic:**
- Push to `main` branch → deploys to both VMs and Docker
- Create PR → runs tests first
- Daily at 2 AM → creates VM snapshots
- Weekly on Sunday → security scans

**Manual:**
```bash
# From GitHub UI: Actions → Deploy to VMs → Run workflow
# Or from CLI:
gh workflow run deploy-vm.yml
gh workflow run deploy-docker.yml
```

---

## 🔐 Step 5: Configure SSH Access

### Generate Deployment Key

```bash
# On your local machine
ssh-keygen -t ed25519 -C "deploy@sanders-legacy-trust" -f ~/.ssh/sanders_deploy

# Add public key to all VMs
gcloud compute instances add-metadata INSTANCE_NAME \
  --zone=us-central1-c \
  --metadata-from-file ssh-keys=~/.ssh/sanders_deploy.pub

# Add private key to GitHub Secrets as SSH_PRIVATE_KEY
cat ~/.ssh/sanders_deploy
```

---

## 🌐 Step 6: Domain Configuration

### Your Owned Domains
- sandershealthcare.com
- shomehealth.com

### Configure DNS

For each platform, add A records:

```bash
# Example for Sanders Healthcare
sandershealthcare.com.    A    34.133.172.131
www.sandershealthcare.com. A    34.133.172.131

# Example for Sanders Grantwriter
sanders-grantwriter.com.   A    34.27.79.1
```

### Get External IPs

```bash
# List all VMs with their IPs
gcloud compute instances list \
  --filter="labels.authority=sanders-legacy-trust" \
  --format="table(name,networkInterfaces[0].accessConfigs[0].natIP)"
```

---

## 📊 Step 7: Monitoring & Health Checks

### Check All Platforms

```bash
# VM-based deployment
for i in {1..33}; do
  IP=$(gcloud compute instances list --filter="labels.platform=*" --format="value(networkInterfaces[0].accessConfigs[0].natIP)" | sed -n "${i}p")
  if curl -f -m 5 "http://$IP/health"; then
    echo "✅ Platform $i healthy"
  else
    echo "❌ Platform $i failed"
  fi
done

# Docker-based deployment
for HOST in 34.133.172.131 35.238.209.6 34.27.79.1; do
  for PORT in {3001..3011}; do
    ssh deploy@$HOST "curl -f -m 5 http://localhost:$PORT/health" && echo "✅ $HOST:$PORT" || echo "❌ $HOST:$PORT"
  done
done
```

### Set Up Monitoring Dashboard

```bash
# Install Prometheus on a monitoring VM
gcloud compute instances create monitoring-vm \
  --zone=us-central1-c \
  --machine-type=e2-medium

# Configure to scrape all platform health endpoints
```

---

## 🔄 Daily Operations

### Update Platform Code

**For VM Deployment:**
```bash
# Push changes to GitHub
git add platforms/sanders-healthcare/
git commit -m "Update Sanders Healthcare platform"
git push

# GitHub Actions automatically deploys to all VMs
# Or manually SSH and pull:
gcloud compute ssh instance-name --zone=us-central1-c
cd /opt/sanders-healthcare
git pull origin main
sudo systemctl restart platform
```

**For Docker Deployment:**
```bash
# Push changes to GitHub
git push

# GitHub Actions automatically:
# 1. Rebuilds Docker image
# 2. Pushes to GCR
# 3. Updates all containers
```

### Add New Platform

```bash
# 1. Create platform directory
mkdir platforms/sanders-new-platform
cd platforms/sanders-new-platform

# 2. Create required files
touch main.py requirements.txt config.json naics.json README.md

# 3. Deploy new VM
gcloud compute instances create instance-sanders-new-platform \
  --zone=us-central1-c \
  --machine-type=e2-highmem-2 \
  --metadata-from-file=startup-script=startup.sh

# 4. Or add to Docker deployment
# Edit deploy_docker.sh and add to appropriate host
```

---

## 🚨 Troubleshooting

### VM Won't Start
```bash
# Check serial port output
gcloud compute instances get-serial-port-output INSTANCE_NAME --zone=us-central1-c

# SSH and check logs
gcloud compute ssh INSTANCE_NAME --zone=us-central1-c
sudo journalctl -u google-startup-scripts -f
tail -f /var/log/platform-name.log
```

### Docker Container Failing
```bash
# Check container logs
ssh deploy@HOST "docker logs CONTAINER_NAME"

# Restart container
ssh deploy@HOST "docker restart CONTAINER_NAME"

# Rebuild and redeploy
cd deployment/docker
./deploy_docker.sh
```

### GitHub Actions Failing
```bash
# Check workflow runs
gh run list --workflow=deploy-vm.yml

# View logs
gh run view RUN_ID --log-failed

# Re-run failed workflow
gh run rerun RUN_ID
```

---

## 📈 Scaling Up

### Add More Hosts
```bash
# For Docker deployment, add 3 more VMs
gcloud compute instances create host-4 host-5 host-6 \
  --zone=us-central1-c \
  --machine-type=e2-highmem-4

# Update deploy_docker.sh with new IPs
HOST4="NEW_IP"
HOST5="NEW_IP"
HOST6="NEW_IP"
```

### Increase Resources
```bash
# Stop VM
gcloud compute instances stop INSTANCE_NAME --zone=us-central1-c

# Change machine type
gcloud compute instances set-machine-type INSTANCE_NAME \
  --machine-type=e2-highmem-4 \
  --zone=us-central1-c

# Restart
gcloud compute instances start INSTANCE_NAME --zone=us-central1-c
```

---

## ✅ Success Checklist

- [ ] GitHub repo created with full structure
- [ ] 33 platforms have code in their directories
- [ ] All platforms have naics.json with 6 codes
- [ ] All platforms have config.json with humanity protocols
- [ ] 30 new VMs created (33 total including your 3 existing)
- [ ] Docker deployment working on 3 hosts
- [ ] GitHub Actions workflows configured
- [ ] GitHub Secrets added (GCP_SA_KEY, SSH_PRIVATE_KEY, etc.)
- [ ] DNS records configured for owned domains
- [ ] All platforms responding to health checks
- [ ] Backup snapshots running daily
- [ ] Security scans running weekly

---

## 🎉 You Now Have

✅ **33 NAICS-based platforms** deployed and running  
✅ **140 NAICS industry bridges** enabling disaster coordination  
✅ **Dual deployment** (VMs + Docker) for redundancy  
✅ **Automated CI/CD** via GitHub Actions  
✅ **Daily backups** and security scanning  
✅ **Zero-trust architecture** ready for integration  
✅ **Brand protection** with Freedom33 Gold certification  

**Next:** Document the Trust Protection platforms (AI Recon, Freedom Revolution, Steward Sentinel, etc.)# .github/workflows/deploy-vm.yml
# =====================================================
# DEPLOY TO VMS - Individual VM Deployment
# Triggers on: push to main, manual dispatch
# =====================================================
name: Deploy to VMs

on:
  push:
    branches: [ main ]
    paths:
      - 'platforms/**'
      - 'deployment/vm/**'
  workflow_dispatch:
    inputs:
      platform:
        description: 'Platform to deploy (or "all")'
        required: true
        default: 'all'

env:
  GCP_PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
  GCP_ZONE: us-central1-c

jobs:
  deploy-vm:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up Cloud SDK
      uses: google-github-actions/setup-gcloud@v1
      with:
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        project_id: ${{ secrets.GCP_PROJECT_ID }}
    
    - name: Configure SSH
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H 34.133.172.131 >> ~/.ssh/known_hosts
        ssh-keyscan -H 35.238.209.6 >> ~/.ssh/known_hosts
        ssh-keyscan -H 34.27.79.1 >> ~/.ssh/known_hosts
    
    - name: Deploy platforms to VMs
      run: |
        # Get list of VMs
        INSTANCES=$(gcloud compute instances list --format="value(name)" --filter="labels.authority=sanders-legacy-trust")
        
        for INSTANCE in $INSTANCES; do
          IP=$(gcloud compute instances describe $INSTANCE --zone=$GCP_ZONE --format="value(networkInterfaces[0].accessConfigs[0].natIP)")
          
          echo "Deploying to $INSTANCE ($IP)..."
          
          # SSH and update platform code
          ssh -o StrictHostKeyChecking=no deploy@$IP << 'ENDSSH'
            cd /opt/*/
            git pull origin main
            pip3 install -r requirements.txt
            sudo systemctl restart platform.service || pkill -f main.py && nohup python3 main.py &
ENDSSH
        done
    
    - name: Verify deployments
      run: |
        INSTANCES=$(gcloud compute instances list --format="value(name)" --filter="labels.authority=sanders-legacy-trust")
        
        for INSTANCE in $INSTANCES; do
          IP=$(gcloud compute instances describe $INSTANCE --zone=$GCP_ZONE --format="value(networkInterfaces[0].accessConfigs[0].natIP)")
          
          # Health check
          if curl -f -m 10 "http://$IP/health"; then
            echo "✅ $INSTANCE healthy"
          else
            echo "❌ $INSTANCE health check failed"
            exit 1
          fi
        done

---
# .github/workflows/deploy-docker.yml
# =====================================================
# DEPLOY TO DOCKER - Container Deployment
# Triggers on: push to main, manual dispatch
# =====================================================
name: Deploy to Docker

on:
  push:
    branches: [ main ]
    paths:
      - 'platforms/**'
      - 'deployment/docker/**'
  workflow_dispatch:

env:
  GCP_PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
  DOCKER_IMAGE: gcr.io/${{ secrets.GCP_PROJECT_ID }}/sanders-platform:latest
  HOST1: 34.133.172.131
  HOST2: 35.238.209.6
  HOST3: 34.27.79.1

jobs:
  build-image:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up Cloud SDK
      uses: google-github-actions/setup-gcloud@v1
      with:
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        project_id: ${{ secrets.GCP_PROJECT_ID }}
    
    - name: Configure Docker
      run: gcloud auth configure-docker
    
    - name: Build Docker image
      run: |
        docker build -f deployment/docker/Dockerfile -t $DOCKER_IMAGE .
    
    - name: Push Docker image
      run: |
        docker push $DOCKER_IMAGE
    
    - name: Image digest
      run: |
        docker inspect --format='{{index .RepoDigests 0}}' $DOCKER_IMAGE

  deploy-containers:
    needs: build-image
    runs-on: ubuntu-latest
    strategy:
      matrix:
        host: [HOST1, HOST2, HOST3]
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Configure SSH
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H ${{ env[matrix.host] }} >> ~/.ssh/known_hosts
    
    - name: Deploy containers
      run: |
        ssh deploy@${{ env[matrix.host] }} << 'ENDSSH'
          # Pull latest image
          docker pull $DOCKER_IMAGE
          
          # Update all containers on this host
          for CONTAINER in $(docker ps -a --format '{{.Names}}'); do
            echo "Updating $CONTAINER..."
            docker stop $CONTAINER
            docker rm $CONTAINER
            docker run -d --name $CONTAINER --restart always \
              $(docker inspect $CONTAINER --format='{{range .Config.Env}}-e {{.}} {{end}}') \
              $(docker inspect $CONTAINER --format='{{range .HostConfig.PortBindings}}{{range .}}-p {{.HostPort}}:3000 {{end}}{{end}}') \
              --memory=2g --cpus=1.5 \
              $DOCKER_IMAGE
          done
ENDSSH
    
    - name: Verify containers
      run: |
        ssh deploy@${{ env[matrix.host] }} << 'ENDSSH'
          for CONTAINER in $(docker ps --format '{{.Names}}'); do
            PORT=$(docker port $CONTAINER | cut -d: -f2)
            if curl -f -m 10 http://localhost:$PORT/health; then
              echo "✅ $CONTAINER healthy"
            else
              echo "❌ $CONTAINER unhealthy"
              exit 1
            fi
          done
ENDSSH

---
# .github/workflows/test-platforms.yml
# =====================================================
# TEST PLATFORMS - Run tests before deployment
# =====================================================
name: Test Platforms

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        platform:
          - sanders-sentinel
          - sanders-omniconm
          - sanders-grantwriter
          - lil-mama
          - baby-girl
          # Add all 33 platforms
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        cd platforms/${{ matrix.platform }}
        pip install -r requirements.txt
        pip install pytest pytest-cov
    
    - name: Run tests
      run: |
        cd platforms/${{ matrix.platform }}
        if [ -d "tests" ]; then
          pytest tests/ -v --cov=. --cov-report=xml
        else
          echo "No tests found for ${{ matrix.platform }}"
        fi
    
    - name: Verify NAICS codes
      run: |
        cd platforms/${{ matrix.platform }}
        if [ -f "naics.json" ]; then
          python3 -c "
import json
with open('naics.json') as f:
    data = json.load(f)
    assert len(data['naics_codes']) == 6, 'Must have exactly 6 NAICS codes'
    print('✅ NAICS validation passed')
          "
        else
          echo "❌ naics.json not found"
          exit 1
        fi
    
    - name: Check humanity protocols
      run: |
        cd platforms/${{ matrix.platform }}
        python3 -c "
import json
with open('config.json') as f:
    config = json.load(f)
    assert config.get('humanity_first') == True, 'humanity_first must be True'
    assert config.get('zero_weaponization') == True, 'zero_weaponization must be True'
    assert config.get('glass_box') == True, 'glass_box must be True'
    print('✅ Humanity protocols verified')
        "

---
# .github/workflows/backup.yml
# =====================================================
# BACKUP - Daily automated backups
# =====================================================
name: Backup Platforms

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC
  workflow_dispatch:

jobs:
  backup:
    runs-on: ubuntu-latest
    
    steps:
    - name: Set up Cloud SDK
      uses: google-github-actions/setup-gcloud@v1
      with:
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        project_id: ${{ secrets.GCP_PROJECT_ID }}
    
    - name: Create VM snapshots
      run: |
        INSTANCES=$(gcloud compute instances list --format="value(name)" --filter="labels.authority=sanders-legacy-trust")
        
        for INSTANCE in $INSTANCES; do
          DISK=$(gcloud compute instances describe $INSTANCE --zone=us-central1-c --format="value(disks[0].source.basename())")
          SNAPSHOT_NAME="${INSTANCE}-snapshot-$(date +%Y%m%d-%H%M%S)"
          
          echo "Creating snapshot: $SNAPSHOT_NAME"
          gcloud compute disks snapshot $DISK \
            --zone=us-central1-c \
            --snapshot-names=$SNAPSHOT_NAME \
            --labels=backup=daily,authority=sanders-legacy-trust
        done
    
    - name: Cleanup old snapshots
      run: |
        # Keep last 7 days of snapshots
        CUTOFF_DATE=$(date -d '7 days ago' +%Y%m%d)
        
        gcloud compute snapshots list --filter="labels.backup=daily AND creationTimestamp < $CUTOFF_DATE" --format="value(name)" | \
        while read SNAPSHOT; do
          echo "Deleting old snapshot: $SNAPSHOT"
          gcloud compute snapshots delete $SNAPSHOT --quiet
        done

---
# .github/workflows/security-scan.yml
# =====================================================
# SECURITY SCAN - Vulnerability scanning
# =====================================================
name: Security Scan

on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday
  workflow_dispatch:

jobs:
  scan:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
    
    - name: Check for secrets
      uses: trufflesecurity/trufflehog@main
      with:
        path: ./
        base: main
        head: HEAD# Sanders Legacy Trust Platforms - Repository Structure

## 📁 Root Directory Layout

```
sanders-legacy-trust-platforms/
├── README.md                          # Main documentation
├── LICENSE                            # Legal/licensing
├── .gitignore
├── .github/
│   └── workflows/
│       ├── deploy-vm.yml             # VM deployment automation
│       ├── deploy-docker.yml         # Docker deployment automation
│       └── test-platforms.yml        # Platform testing
│
├── platforms/                         # Individual platform code
│   ├── sanders-sentinel/
│   │   ├── main.py
│   │   ├── requirements.txt
│   │   ├── config.json
│   │   ├── naics.json               # NAICS codes: 541512,541513,541519,561621,518210,541690
│   │   └── README.md
│   │
│   ├── sanders-omniconm/
│   │   ├── main.py
│   │   ├── requirements.txt
│   │   ├── config.json
│   │   ├── naics.json               # NAICS codes: 517810,518210,541511,541512,541519,519190
│   │   └── README.md
│   │
│   ├── sanders-grantwriter/
│   │   ├── main.py
│   │   ├── requirements.txt
│   │   ├── config.json
│   │   ├── naics.json               # NAICS codes: 541611,541612,541618,561499,541990,813211
│   │   └── README.md
│   │
│   ├── lil-mama/
│   ├── baby-girl/
│   ├── gai-mind/
│   ├── ai-doctor/
│   ├── patriot-saint/
│   ├── sanders-home-healthcare/
│   ├── sanders-senior-living/
│   ├── sanders-legal-helpers/
│   ├── sanders-education/
│   ├── sanders-finance/
│   ├── sanders-retail/
│   ├── sanders-logistics/
│   ├── sanders-security/
│   ├── sanders-real-estate/
│   ├── sanders-energy/
│   ├── sanders-transportation/
│   ├── sanders-agriculture/
│   ├── sanders-manufacturing/
│   ├── sanders-hospitality/
│   ├── sanders-entertainment/
│   ├── sanders-sports/
│   ├── sanders-wellness/
│   ├── sanders-travel/
│   ├── sanders-ai-research/
│   ├── sanders-research/
│   ├── sanders-media/
│   ├── sanders-communications/
│   ├── sanders-compliance/
│   ├── sanders-coordinator/
│   └── sanders-consulting/
│
├── shared/                            # Shared libraries
│   ├── naics_bridge.py               # NAICS coordination logic
│   ├── humanity_protocols.py        # Humanity-first enforcement
│   ├── zero_weaponization.py        # Weaponization prevention
│   ├── glass_box.py                 # Transparency/audit
│   └── common_utils.py
│
├── deployment/                        # Deployment scripts
│   ├── vm/
│   │   ├── create_vms.sh            # Create 33 VMs
│   │   ├── startup_template.sh      # Template startup script
│   │   └── vm_config.json           # VM configurations
│   │
│   ├── docker/
│   │   ├── Dockerfile               # Multi-platform Docker image
│   │   ├── docker-compose.yml       # Compose for all platforms
│   │   └── deploy_docker.sh         # Docker deployment script
│   │
│   └── zero_trust/
│       ├── token_generator.js       # Generate DEPLOY_TOKENs
│       ├── api_server.js            # Zero-trust API
│       └── deployment_gate.sh       # Token validation
│
├── infrastructure/                    # Infrastructure as Code
│   ├── terraform/
│   │   ├── main.tf                  # GCP infrastructure
│   │   ├── variables.tf
│   │   └── outputs.tf
│   │
│   └── gcp/
│       ├── firewall_rules.sh        # Firewall configuration
│       ├── networking.sh            # VPC/subnet setup
│       └── dns_records.sh           # Domain mapping
│
├── monitoring/                        # Monitoring & observability
│   ├── prometheus/
│   │   └── config.yml
│   ├── grafana/
│   │   └── dashboards/
│   └── health_checks.py
│
├── certification/                     # Brand & compliance
│   ├── freedom33_gold_registry.py   # Brand registry system
│   ├── naics_verification.py       # NAICS validation
│   └── certifications/              # Platform certifications
│       ├── sanders-sentinel.json
│       ├── sanders-omniconm.json
│       └── ... (one per platform)
│
├── docs/                             # Documentation
│   ├── architecture.md
│   ├── naics_bridges.md
│   ├── deployment_guide.md
│   ├── api_reference.md
│   └── platform_guides/
│       ├── sanders-sentinel.md
│       └── ... (one per platform)
│
└── tests/                            # Testing
    ├── unit/
    ├── integration/
    └── e2e/
```

## 📝 Key Files to Create

### Root README.md
```markdown
# Sanders Legacy Trust Platforms

**Authority:** Sanders Family Living Trust  
**Founder:** Roosevelt Sanders  
**Certification:** FREEDOM33-GOLD

## 33 NAICS-Based Platforms

Each platform connects 6 NAICS industry codes for seamless disaster coordination.

### Deployment Options

1. **Production VMs** - 33 individual VMs (one per platform)
2. **Docker Containers** - 3 hosts with 11 platforms each
3. **Hybrid** - Both for redundancy and testing

[Full documentation](./docs/)
```

### platforms/[platform-name]/naics.json (Example)
```json
{
  "platform": "Sanders Sentinel",
  "naics_codes": [
    "541512",
    "541513", 
    "541519",
    "561621",
    "518210",
    "541690"
  ],
  "bridges": {
    "sanders-omniconm": ["518210", "541519"],
    "lil-mama": ["541512", "541519", "561621"],
    "baby-girl": ["541512", "541519", "561621"]
  }
}
```

### platforms/[platform-name]/config.json (Example)
```json
{
  "name": "Sanders Sentinel",
  "nickname": "Alpha Watchdog",
  "tier": 2,
  "classification": "TS/SCI",
  "annual_fee": 455000000,
  "port": 3001,
  "health_check": "/health",
  "humanity_first": true,
  "zero_weaponization": true,
  "glass_box": true
}
```

## 🚀 Quick Start

### Clone Repository
```bash
git clone https://github.com/rooseveltsanders381-ship-it/sanders-legacy-trust-platforms.git
cd sanders-legacy-trust-platforms
```

### Deploy All VMs
```bash
cd deployment/vm
./create_vms.sh
```

### Deploy Docker Containers
```bash
cd deployment/docker
./deploy_docker.sh
```

## 📊 Platform Distribution

- **Host 1 (34.133.172.131):** Guardians & Critical (11 platforms)
- **Host 2 (35.238.209.6):** Operations & Infrastructure (11 platforms)
- **Host 3 (34.27.79.1):** Support Services & Lifestyle (11 platforms)

## 🔒 Security

- Zero-trust deployment with token validation
- NAICS bridge verification
- Humanity-first protocol enforcement
- Brand certification locked with SHA256

## 📄 License

© 2026 Sanders Family Living Trust. All rights reserved.
```

## 🎯 Next Steps

1. Create this structure in your GitHub repo
2. Commit the deployment scripts I'll create
3. Set up GitHub Actions workflows
4. Deploy platforms using automated pipelinesname: FREEDOM33 Auto Deploy & Audit

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy-and-audit:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20

      - name: Install Vercel CLI
        run: npm install -g vercel

      - name: Generate Modular Docs
        run: python3 scripts/generate_docs.py

      - name: Run Universal Deployment
        run: bash scripts/universal_deploy.sh
        env:
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
          WEBHOOK_URL: ${{ secrets.WEBHOOK_URL }}#!/bin/bash
# ======================================================
# FREEDOM33 Universal Deployment
# Sanders Family Living Trust - Baseline 2026-01-20
# ======================================================

set -euo pipefail

REGISTRY="./baseline/export/platform_registry.json"
AUDIT_LOG="./logs/freedom33_audit.log"
HEARTBEAT_LOG="./logs/heartbeat.log"
WEBHOOK_URL="PASTE_YOUR_WEBHOOK_URL_HERE"

mkdir -p ./logs

# ---- GitHub Push Baseline & Docs ----
git config user.name "Sanders Authority Bot"
git config user.email "authority@sanders.global"
git add README.md docs/ baseline/export/
git diff --cached --quiet || git commit -m "🔒 FREEDOM33: Docs & Registry Modularized"
git push origin main

# ---- Deploy All Platforms ----
echo "🔗 Deploying all platforms..."
jq -r 'to_entries[] | "\(.key)|\(.value.url)"' "$REGISTRY" | while IFS='|' read -r NAME URL; do
    echo "🚀 Deploying $NAME..."
    npx vercel --prod --confirm --token "$VERCEL_TOKEN" --name "$NAME"
done

# ---- Heartbeat Audit ----
echo "📡 Running heartbeat audit..." | tee -a "$AUDIT_LOG"
jq -r 'to_entries[] | "\(.key)|\(.value.url)"' "$REGISTRY" | while IFS='|' read -r NAME URL; do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$URL")
    if [[ "$STATUS" == "200" ]]; then
        echo "$(date -u) | ✅ $NAME is LIVE at $URL" | tee -a "$AUDIT_LOG"
    else
        echo "$(date -u) | ❌ $NAME DOWN ($STATUS) at $URL" | tee -a "$AUDIT_LOG"
        curl -H "Content-Type: application/json" -X POST -d "{\"content\":\"⚠️ ALERT: $NAME is DOWN! (Status: $STATUS)\"}" "$WEBHOOK_URL"
        echo "$(date -u) | ALERT sent for $NAME" >> "$HEARTBEAT_LOG"
    fi
done

echo "✅ Deployment & audit complete."#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Sanders FREEDOM33: README Modularization
Author: Sanders Family Living Trust
Date: 2026-01-20
"""

import os
import re

README_FILE = "README.md"
DOCS_DIR = "docs"
TIERS_DIR = os.path.join(DOCS_DIR, "tiers")
LEGAL_DIR = os.path.join(DOCS_DIR, "legal")

os.makedirs(TIERS_DIR, exist_ok=True)
os.makedirs(LEGAL_DIR, exist_ok=True)

with open(README_FILE, "r", encoding="utf-8") as f:
    content = f.read()

# Split by headers for tiers
tier_matches = re.findall(r"## 🏆 Tier \d+:.*?(\n\n.*?)(?=##|$)", content, flags=re.S)
for i, tier in enumerate(tier_matches, 1):
    file_path = os.path.join(TIERS_DIR, f"tier{i}.md")
    with open(file_path, "w", encoding="utf-8") as f:
        f.write(f"# Tier {i}\n\n{tier.strip()}\n")

# Extract Legal Sections
legal_headers = ["Copyright", "Patent Status", "Trade Secret", "Constitutional Framework"]
for header in legal_headers:
    pattern = rf"## 📜 {header}\n(.*?)(?=\n##|$)"
    match = re.search(pattern, content, flags=re.S)
    if match:
        file_path = os.path.join(LEGAL_DIR, f"{header.lower().replace(' ','_')}.md")
        with open(file_path, "w", encoding="utf-8") as f:
            f.write(f"# {header}\n\n{match.group(1).strip()}\n")

# Create platforms.md
platforms_pattern = r"## 📊 Revenue Summary.*?(?=\n##)"
platforms_match = re.search(platforms_pattern, content, flags=re.S)
if platforms_match:
    platforms_md = os.path.join(DOCS_DIR, "platforms.md")
    with open(platforms_md, "w", encoding="utf-8") as f:
        f.write("# Platforms\n\n")
        f.write(platforms_match.group(0).strip())

# Update README
readme_updated = "# Sanders Freedom33 Gold - 40,000% Sovereign Pricing\n\n"
readme_updated += "## Tiers\n"
for i in range(1, len(tier_matches)+1):
    readme_updated += f"- [Tier {i}]({TIERS_DIR}/tier{i}.md)\n"

readme_updated += "\n## Legal & Core Principles\n"
for header in legal_headers:
    readme_updated += f"- [{header}]({LEGAL_DIR}/{header.lower().replace(' ','_')}.md)\n"

readme_updated += "\n## Platforms\n- [Platform Registry](docs/platforms.md)\n"

with open(README_FILE, "w", encoding="utf-8") as f:
    f.write(readme_updated)

print("✅ README modularization complete.")/scripts/generate_docs.py
/scripts/generate_certificates.pyname: FREEDOM33 Auto-Deploy & Docs Split

on:
  push:
    branches: [main]
  workflow_dispatch:  # manual trigger

permissions:
  contents: write

jobs:
  docs-and-deploy:
    runs-on: ubuntu-latest

    steps:
    # 1️⃣ Checkout the repo
    - name: Checkout Repository
      uses: actions/checkout@v4
      with:
        fetch-depth: 0

    # 2️⃣ Setup Python (for modular README & docs generation)
    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: 3.11

    # 3️⃣ Install dependencies
    - name: Install Dependencies
      run: |
        python -m pip install --upgrade pip
        pip install jinja2

    # 4️⃣ Generate Modular Docs & Master README
    - name: Generate Docs
      run: |
        python scripts/generate_docs.py
        echo "✅ Docs split & master README updated"

    # 5️⃣ Commit & push updated docs
    - name: Commit & Push Docs
      run: |
        git config user.name "Sanders Authority Bot"
        git config user.email "authority@sanders.global"
        git add docs/ README.md
        if ! git diff --cached --quiet; then
          git commit -m "📄 FREEDOM33: Modular docs & README updated"
          git push origin main
        else
          echo "No changes to commit."
        fi

    # 6️⃣ Install Vercel CLI
    - name: Install Vercel CLI
      run: npm install -g vercel

    # 7️⃣ Deploy all platforms in registry
    - name: Deploy Platforms
      env:
        VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
      run: |
        REGISTRY="./platform_registry.json"
        for PLATFORM in $(jq -r 'keys[]' $REGISTRY); do
          URL=$(jq -r --arg key "$PLATFORM" '.[$key].url' $REGISTRY)
          echo "🚀 Deploying $PLATFORM → $URL"
          npx vercel --prod --confirm --token $VERCEL_TOKEN --name $(echo $PLATFORM | tr ' ' '-')
        done

    # 8️⃣ Heartbeat Audit (non-blocking)
    - name: Universal Heartbeat
      run: |
        REGISTRY="./platform_registry.json"
        AUDIT_LOG="./logs/freedom33_audit.log"
        mkdir -p logs
        touch $AUDIT_LOG
        FAILURES=0
        for PLATFORM in $(jq -r 'keys[]' $REGISTRY); do
          URL=$(jq -r --arg key "$PLATFORM" '.[$key].url' $REGISTRY)
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$URL")
          if [[ "$STATUS" == "200" ]]; then
            echo "$(date -u) | ✅ $PLATFORM is LIVE at $URL" | tee -a $AUDIT_LOG
          else
            echo "$(date -u) | ⚠️ $PLATFORM is DOWN ($STATUS) at $URL" | tee -a $AUDIT_LOG
            ((FAILURES++))
          fi
        done
        echo "🔹 Heartbeat completed. $FAILURES platforms not reachable."

    # 9️⃣ Optional: Generate certificates (PNG + PDF)
    - name: Generate Certificates
      run: |
        python scripts/generate_certificates.py
        echo "🏅 FREEDOM33 Gold Seal & Certificate regenerated"/docs
├─ tiers/
│  ├─ tier1.md
│  ├─ tier2.md
│  ├─ tier3.md
│  └─ tier4.md
├─ legal/
│  ├─ legal_protections.md
│  └─ core_principles.md
├─ platforms/
│  └─ platform_grid.md
├─ certificates/
│  ├─ FREEDOM33_GOLD_SEAL.png
│  └─ FREEDOM33_GOLD_CERTIFICATE.pdf
└─ contact.md