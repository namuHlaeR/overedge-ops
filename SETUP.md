# VPS CI/CD Setup Checklist

One-time steps to activate GitHub Actions deploy.

## CRM projects (temir, zonpa, velora, demo-overedge)

Each repo has `.github/workflows/deploy.yml` and uses GHCR images + `/opt/docker/scripts/gha-crm-deploy.sh` on the VPS.

| Project | GitHub repo | GHCR image | VPS path |
|---------|-------------|------------|----------|
| Temir | `namuHlaeR/Temir-CRM` | `ghcr.io/namuhlaer/temir-crm` | `/opt/docker/clients/temir` |
| Zonpa | `namuHlaeR/ZonpaCRM` | `ghcr.io/namuhlaer/zonpa-crm` | `/opt/docker/clients/zonpa/ZonpaCRM` |
| Velora | `namuHlaeR/velora-crm` | `ghcr.io/namuhlaer/velora-crm` | `/opt/docker/clients/velora` |
| Demo | `namuHlaeR/demo-overedge-crm` | `ghcr.io/namuhlaer/demo-overedge-crm` | `/opt/docker/clients/demo-overedge` |

**Per repo:** copy the `VPS_SSH_HOST` environment + secrets from `lca` (or use org-level environment).

```bash
# First deploy (build + push + pull on VPS)
gh workflow run deploy.yml --repo namuHlaeR/Temir-CRM
gh workflow run deploy.yml --repo namuHlaeR/ZonpaCRM
gh workflow run deploy.yml --repo namuHlaeR/velora-crm
gh workflow run deploy.yml --repo namuHlaeR/demo-overedge-crm
```

---

## Lynkclaimactie pilot

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
| `VPS_SSH_USER` | `gha-deploy` (CI user; no 2FA — runs deploy as `deploy` via sudo) |
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
