# VPS CI/CD Setup Checklist

One-time steps to activate GitHub Actions deploy for lynkclaimactie (pilot).

## 1. GitHub CLI on VPS

```bash
gh auth login
# Select: GitHub.com → HTTPS → Paste PAT with repo + workflow + read:packages scopes
gh repo list namuHlaeR --limit 20
```

## 2. Push `overedge-ops` hub repo

```bash
cd /opt/docker/overedge-ops
git init -b main
git add -A
git commit -m "Add project registry and reusable GHA workflows"
gh repo create namuHlaeR/overedge-ops --public --source=. --push
```

## 3. GitHub account secrets (Settings → Secrets → Actions)

| Secret | Value |
|--------|-------|
| `VPS_SSH_HOST` | `46.225.155.131` |
| `VPS_SSH_USER` | `deploy` |
| `VPS_SSH_PRIVATE_KEY` | deploy user's private key |

Optional variable: `VPS_SSH_PORT` = `2222` or `22`

## 4. Push lynkclaimactie workflow changes

```bash
cd /opt/docker/clients/lynkclaimactie
git add docker-compose.dev.yml docker-compose.yml deploy-docker.sh \
  portal/Dockerfile.dev website/Dockerfile.dev .github/workflows/
git commit -m "Add VPS dev compose and GHA deploy pipeline"
git push origin main
```

## 5. GHCR read access on VPS

After first successful GHA deploy (or make packages public):

```bash
echo $GITHUB_PAT | docker login ghcr.io -u namuHlaeR --password-stdin
```

## 6. Release commands

```bash
# Dev (hot reload)
/opt/docker/scripts/vps-dev.sh lynkclaimactie start portal-dev

# List all projects
/opt/docker/scripts/vps-projects list

# Production release
/opt/docker/scripts/vps-release.sh lynkclaimactie portal "feat: description"
# Or from GitHub UI: Actions → Deploy → Run workflow
```

## 7. GitHub Projects board (manual)

Create user project **Overedge VPS Apps** at github.com/namuHlaeR and link all repos.
