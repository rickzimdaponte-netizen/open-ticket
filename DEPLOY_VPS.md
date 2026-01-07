# Deploying Open Ticket to a VPS (Docker + docker-compose)

This guide shows how to prepare your VPS and configure GitHub Actions to automatically deploy Open Ticket on push to `main`.

## Server prerequisites (on your VPS)
1. A user with SSH access (recommended: a non-root user). We'll call it `<ssh-user>`.
2. Install Docker and Docker Compose on the machine (example for Ubuntu):
   ```bash
   # install Docker
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh

   # add your user to the docker group then log out/in
   sudo usermod -aG docker <ssh-user>

   # optionally install docker compose plugin
   sudo apt install -y docker-compose-plugin
   ```
3. Ensure `git` is installed: `sudo apt install -y git`
4. Choose a folder for the app (default used by the workflow: `/home/<ssh-user>/open-ticket`) and make sure the SSH user can write there.

## GitHub Secrets (Repository settings → Settings → Secrets and variables → Actions)
Add the following repository secrets:
- `SSH_HOST` — your VPS IP or hostname
- `SSH_USER` — SSH user to log in as (e.g. `ubuntu` or `openticket`)
- `SSH_KEY` — **private** key for SSH login (the public key should be in `~/.ssh/authorized_keys` of `SSH_USER`)
- `TOKEN` — your Discord bot token (generate a new one; revoke any exposed token)
- Optional:
  - `SSH_PORT` — if your SSH runs on a non-default port
  - `REMOTE_PATH` — custom path on the server (default: `/home/${{ secrets.SSH_USER }}/open-ticket`)
  - Docker Hub / registry secrets (if you prefer to push images instead of `git pull`)

## How the GitHub Actions workflow works
- Workflow file: `.github/workflows/deploy-vps.yml`
- On push to `main` (or manual dispatch) it will:
  1. SSH into your VPS
  2. Clone the repo (if missing) or fetch + reset to `origin/main`
  3. Create/overwrite `.env` with `TOKEN` (and any other envs you add to the workflow)
  4. Run `docker compose up -d --build` to build and start the container

## Notes & recommendations
- The workflow **overwrites** the remote `.env` with the `TOKEN` secret on each deploy. If you have more env variables, add them to the workflow and GitHub Secrets (or prepare an env template on the server).
- For production stability, consider using a systemd unit or restart policy and a backup strategy for the `data/` folder.
- Test the steps manually once before relying on automation: SSH into the VPS, `git clone` the repo, create a local `.env` and run `docker compose up -d --build`.

---
If you want, I can:
- adjust the workflow to write multiple env variables or keep the remote `.env` preserved,
- switch the workflow to build the Docker image in Actions and push to a registry, which the server then pulls (recommended for reproducible builds),
- or create a short `setup-server.sh` script to bootstrap a fresh VPS.

Tell me which of the above you prefer and I will implement the change. 🎯