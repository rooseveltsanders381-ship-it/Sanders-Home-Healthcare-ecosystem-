/scripts/generate_docs.py
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