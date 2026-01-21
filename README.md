#!/usr/bin/env python3
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