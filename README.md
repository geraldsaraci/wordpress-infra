# wordpress-infra

Infrastructure-as-code for deploying WordPress + MySQL with Docker Compose, automated via GitHub Actions.

**Server:** 23.88.107.201 (user: deploy)

## Architecture

Mac (dev) -> GitHub -> Server (Docker Compose)

ASCII diagram:

  [Developer Mac]
         |
         | push
         v
      [GitHub]
         |
         | GitHub Actions (rsync over SSH)
         v
  [Ubuntu Server 23.88.107.201]
         docker compose up -d

WordPress is exposed at: http://23.88.107.201:8083
MySQL is internal to the Docker network and is not exposed to the internet.

Data persists in Docker volumes: `wordpress-stack_db_data` and `wordpress-stack_wp_data`.

## Folder structure

- wordpress-stack/docker-compose.yml
- wordpress-stack/.env.example
- .github/workflows/deploy-wordpress.yml

## Setup

Prerequisites (on the server):
- Docker and Docker Compose installed
- `deploy` user created (no sudo)
- SSH key for the GitHub Actions runner added to allow `rsync`/SSH

GitHub Secrets (set in repository secrets):
- `SSH_HOST` — server IP (e.g. 23.88.107.201)
- `SSH_USER` — remote user (`deploy`)
- `SSH_PRIVATE_KEY` — private key used by Actions to SSH/rsync to server
- `WP_MYSQL_PASSWORD` — WordPress database user password
- `WP_MYSQL_ROOT_PASSWORD` — MySQL root password

Notes:
- The workflow at `.github/workflows/deploy-wordpress.yml` builds the `.env` on the server from GitHub Secrets.
- The workflow excludes `.env`, `db_data/`, and `wp_data/` from `rsync` so local secrets and data are never pushed.

Triggering a deploy:
- Push to `main` branch (the workflow runs on `git push` to `main`).

## Runbook (server commands)
Change to the deployment directory on the server:

  cd /home/deploy/wordpress-stack

- Check status:

  docker compose ps

- View logs:

  docker compose logs wordpress
  docker compose logs db

- Restart services:

  docker compose restart

## Troubleshooting

- Error establishing a database connection
  - Common cause: corrupted containers/volumes in a lab environment. To reset (DESTROYS DATA):

    docker compose down -v && docker compose up -d

  - Verify `.env` on server contains correct DB credentials (`/home/deploy/wordpress-stack/.env`).

- Permission denied during rsync
  - The `deploy` user does not have `sudo`. The deploy path is `/home/deploy/wordpress-stack` (not `/srv`). Ensure the workflow and `rsync` target use that path and that `deploy` owns it.

## Security notes

- Never commit `.env` or real secrets to the repository.
- MySQL is intentionally not exposed; keep it inside the Docker network.
- For production, put WordPress behind a reverse proxy (e.g., Nginx) and terminate TLS (HTTPS). Consider using a firewall and fail2ban for SSH protection.

---

If you want, I can also add a sample `README` badge, health-check tips, or a minimal `nginx` reverse-proxy example.
