# New Dev Environment Setup — Oracle Cloud Free Tier + Claude Code

## Why Oracle Free Tier
- 4 ARM OCPUs + 24GB RAM + 200GB storage
- Permanently free (no 90-day expiry like GCP)
- Runs Docker, Spark, Kafka, PostgreSQL, everything
- SSH from Mac exactly like GCP

---

## STEP 1 — On your Mac: install gh CLI

```bash
brew install gh
gh auth login
# Choose: GitHub.com → HTTPS → Login with a web browser
```

---

## STEP 2 — Create Oracle Cloud account (browser)

1. Go to: https://cloud.oracle.com
2. Sign up for free (needs credit card but won't charge for free tier)
3. Choose home region — pick US East (Ashburn) or US West (San Jose)

---

## STEP 3 — Create the ARM VM (browser, inside Oracle Cloud)

1. Go to: Compute → Instances → Create Instance
2. Name: `dev-vm`
3. Image: **Ubuntu 22.04** (click Edit → change OS)
4. Shape: Click Edit → **Ampere** → `VM.Standard.A1.Flex`
   - Set OCPUs: **4**
   - Set RAM: **24 GB**
5. Networking: leave defaults (public IP = yes)
6. SSH keys: **Download the private key** (.key file) — save it somewhere safe
7. Boot volume: set to **200 GB**
8. Click Create

Wait ~2 minutes for it to provision. Copy the **Public IP address**.

---

## STEP 4 — SSH into Oracle VM from Mac

```bash
# Move and permission the key (replace with your actual filename and IP)
mv ~/Downloads/ssh-key-*.key ~/.ssh/oracle-dev.key
chmod 400 ~/.ssh/oracle-dev.key

# SSH in
ssh -i ~/.ssh/oracle-dev.key ubuntu@<YOUR_PUBLIC_IP>
```

To make it easier, add to ~/.ssh/config on Mac:
```
Host oracle-dev
    HostName <YOUR_PUBLIC_IP>
    User ubuntu
    IdentityFile ~/.ssh/oracle-dev.key
```
Then just: `ssh oracle-dev`

---

## STEP 5 — Inside Oracle VM: install dev tools

```bash
# Update system
sudo apt-get update && sudo apt-get upgrade -y

# Install essentials
sudo apt-get install -y git curl wget unzip build-essential python3 python3-pip python3-venv

# Install Docker
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker ubuntu
newgrp docker

# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify
node --version  # should be v20.x
docker --version
```

---

## STEP 6 — Install Claude Code on Oracle VM

```bash
npm install -g @anthropic-ai/claude-code

# Launch and log in (opens browser link — paste in Mac browser)
claude
# Choose: Login with Claude.ai → copy the URL → open on Mac → authorize
```

---

## STEP 7 — Clone your repos on Oracle VM

```bash
# Set up git
git config --global user.name "Ahmad Avar"
git config --global user.email "your@email.com"

# Authenticate gh CLI on the VM
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install gh -y
gh auth login

# Clone your main projects
git clone https://github.com/ahmadavar/jobradar
git clone https://github.com/ahmadavar/loan-matching-ai
git clone https://github.com/ahmadavar/data-engineering-zoomcamp-2026
git clone https://github.com/ahmadavar/uba-mads-projects
git clone https://github.com/ahmadavar/ctd-python-200
```

---

## STEP 8 — Open firewall ports on Oracle (browser — REQUIRED)

Oracle blocks all ports by default — open these:

1. Go to: Networking → Virtual Cloud Networks → your VCN → Security Lists → Default
2. Add Ingress Rules:
   - Port 22 (SSH) — probably already there
   - Port 8000 (FastAPI)
   - Port 3000 (Next.js)
   - Port 5432 (PostgreSQL) — restrict to your IP only

---

## Daily workflow

```bash
# From Mac terminal
ssh oracle-dev

# Inside VM
cd ~/jobradar
claude
```

---

## GitHub Codespaces (lighter alternative for quick sessions)

If you only need to do light coding without Docker/databases:

```bash
# From Mac terminal
gh codespace create --repo ahmadavar/jobradar --machine basicLinux32gb
gh codespace list  # get the name
gh codespace ssh -c <codespace-name>

# Inside codespace
npm install -g @anthropic-ai/claude-code
claude
```
Free tier: 60 core-hours/month (~30 hours on 2-core machine).
Codespaces auto-stop after 30 min idle (won't drain your hours).
