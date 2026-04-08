# GitHub Actions - CI/CD

## What is CI/CD?
- **CI** (Continuous Integration) → automatically test code on every push
- **CD** (Continuous Deployment) → automatically deploy code after tests pass

## Flow
```
git push → GitHub Actions → SSH into VM → git pull → docker compose up --build → live ✅
```

## File location
```
.github/
└── workflows/
    └── deploy.yml
```

## deploy.yml breakdown

```yaml
name: Deploy Admin        # name of the workflow

on:
  push:
    branches:
      - main              # triggers when you push to main

jobs:
  deploy:                 # Define a job called deploy
    runs-on: ubuntu-latest  # GitHub spins up a fresh Ubuntu machine

    steps:
      - name: SSH into VM and deploy
        uses: appleboy/ssh-action@v1  # pre-built action to SSH into VM
        with:
          host: ${{ secrets.VM_HOST }}      # your VM public IP
          username: ${{ secrets.VM_USER }}  # your VM username
          key: ${{ secrets.VM_SSH_KEY }}    # your private SSH key
          script: |
            cd ~/ausadhi-admin          # go to project folder on VM
            git pull                    # get latest code from GitHub
            docker compose up -d --build  # rebuild and restart container
```

## GitHub Secrets
Go to repo → **Settings → Secrets and variables → Actions → New secret**

| Secret | Value |
|---|---|
| `VM_HOST` | your VM public IP |
| `VM_USER` | your VM username (e.g. ausadhi) |
| `VM_SSH_KEY` | your private key (`cat ~/.ssh/id_ed25519`) |

## Key Concepts

| Term | Meaning |
|---|---|
| `runs-on` | GitHub's temporary machine, NOT your VM |
| `uses` | pre-built action, someone already wrote the SSH logic |
| `script` | commands that run ON YOUR VM after SSH |
| `secrets` | sensitive values stored safely in GitHub, never hardcoded |

## Two Nginx roles in your setup

| Nginx | Role |
|---|---|
| Nginx on VM (bare metal) | Reverse proxy → routes traffic to containers |
| Nginx inside container | Web server → serves static React files |

## Important Rules
- **Private key** → never share, goes in `VM_SSH_KEY` secret
- **Public key** → safe to share, goes in Azure during VM creation
- **.env** → never commit to GitHub, lives on VM only
- **.env.example** → commit this instead, shows keys without values