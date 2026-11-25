# GitHub Actions production VPS deployment

The `Publish Chatwoot EE docker images` workflow deploys to the production VPS
after publishing the `develop` image. The deploy job follows the Docker Compose
flow: update the checkout on the VPS, pull the published image, prepare database
migrations, and recreate the Rails and Sidekiq services.

## GitHub configuration

Create a GitHub environment named `production` and add these environment
secrets:

| Secret | Value |
| --- | --- |
| `VPS_HOST` | VPS hostname or IP address |
| `VPS_USER` | SSH user that owns the Chatwoot checkout and can run Docker Compose |
| `VPS_APP_PATH` | Absolute path to the Chatwoot checkout on the VPS |
| `VPS_SSH_PRIVATE_KEY` | Private key for the SSH user |
| `VPS_KNOWN_HOSTS` | Verified SSH host key entry for the VPS |

In **Settings → Secrets and variables → Actions**, add the repository secrets
`DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`; these are used by the image build
jobs, which do not use the `production` environment. The token needs permission
to push to your Docker Hub repository.

Add a repository variable named `DOCKERHUB_REPOSITORY` with the image name,
including its owner, for example `your-dockerhub-user/chatwoot`. The default
`chatwoot/chatwoot` is writable only by the official Chatwoot Docker Hub account.

Populate `VPS_KNOWN_HOSTS` with the server's verified host key, for example the
output of `ssh-keyscan -H <host>` after checking the fingerprint through a
trusted channel. Do not disable SSH host key checking.

## VPS requirements

- The SSH user can read the Chatwoot checkout and fetch `origin/develop`.
- The checkout path configured in `VPS_APP_PATH` has no local changes that
  would block `git checkout` or `git pull --ff-only`.
- Docker Engine and the Docker Compose plugin are installed, and the SSH user
  can run Docker without an interactive password.
- The VPS can pull the configured Docker Hub image. If the repository is
  private, authenticate Docker on the VPS as well.
- The checkout contains `docker-compose.production.yaml` and the VPS has its
  production `.env` file configured.
- The SSH user can fetch `origin/develop`; for an SSH GitHub remote, configure a
  separate read-only deploy key on the VPS.

The deploy job runs only when a commit on `develop` is associated with a merged
pull request, and only after the image publisher succeeds. It fetches and
fast-forwards `develop`, pulls the configured `DOCKERHUB_REPOSITORY:develop`
image, runs `db:chatwoot_prepare`, and recreates the Rails and Sidekiq services.
PostgreSQL, Redis, and storage remain managed by the existing Compose setup.
