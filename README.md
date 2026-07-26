# pythonnurr

Python REST API built with **FastAPI**, served behind **nginx** on **AWS EC2** (Ubuntu 22.04, t3.micro). Infrastructure is managed by Terraform; server configuration by Ansible; CI/CD by GitHub Actions via UDAP.

---

## Architecture

```
User → nginx :80 → uvicorn :8000 (FastAPI)
```

- EC2 t3.micro — `us-east-1`
- Elastic IP — stable public address across instance replacement
- Security group — inbound 22 (SSH), 80 (HTTP), 443 (HTTPS)
- nginx — reverse proxy, strips port, forwards to app
- systemd — keeps uvicorn alive, auto-restarts on crash

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | HTML landing page |
| GET | `/health` | Health probe (`{"status":"ok"}`) |
| GET | `/docs` | Swagger UI (auto-generated) |
| GET | `/redoc` | ReDoc API reference |

---

## Local development

```bash
# 1. Install dependencies
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt -r requirements-dev.txt

# 2. Run the dev server
uvicorn app.main:app --reload --port 8000

# 3. Run tests
pytest tests/
```

App available at http://localhost:8000

---

## Deployment pipeline

Three stages run automatically on push to `main`:

| Stage | What it does |
|-------|-------------|
| **provision** | `terraform apply` — EC2 + EIP + security group |
| **configure** | Ansible playbook — installs Python/nginx/uvicorn, deploys app, wires systemd |
| **verify** | `curl /health` with retries — fails the run if the app is not healthy |

Destroy: trigger the **Destroy** workflow from GitHub Actions → Actions tab.

---

## Configuration

| Variable | Where | Notes |
|----------|-------|-------|
| `PROJECT_NAME` | CI secret (platform) | Prefixes all cloud resources |
| `SSH_PUBLIC_KEY` / `SSH_PRIVATE_KEY` | CI secret (platform) | RSA keypair managed by the platform |
| `SSH_USER` | CI secret (platform) | `ubuntu` (Ubuntu 22.04) |
| `TF_STATE_BUCKET` | CI secret (platform) | Terraform remote state bucket |
| `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` | CI secret (platform) | AWS credentials |

---

## Operations

```bash
# SSH into the instance (IP available after first deploy)
ssh -i <key> ubuntu@<public-ip>

# Check app status
sudo systemctl status app

# Tail app logs
sudo journalctl -u app -f

# Restart app
sudo systemctl restart app

# Check nginx
sudo systemctl status nginx
sudo nginx -t
```
