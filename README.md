# .github/workflows/deploy-vm.yml
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