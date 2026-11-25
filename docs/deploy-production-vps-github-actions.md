# GitHub Actions production VPS deployment

The `Deploy production VPS` workflow deploys after a pull request is merged into
`develop`. It supports the native Ubuntu installation managed by `cwctl`, with
the Chatwoot checkout at `/home/chatwoot/chatwoot` and the `chatwoot.target`
systemd target.

## GitHub configuration

Create a GitHub environment named `production` and add these environment
secrets:

| Secret | Value |
| --- | --- |
| `VPS_HOST` | VPS hostname or IP address |
| `VPS_USER` | SSH user for the Chatwoot installation, normally `chatwoot` |
| `VPS_SSH_PRIVATE_KEY` | Private key for the SSH user |
| `VPS_KNOWN_HOSTS` | Verified SSH host key entry for the VPS |

Populate `VPS_KNOWN_HOSTS` with the server's verified host key, for example the
output of `ssh-keyscan -H <host>` after checking the fingerprint through a
trusted channel. Do not disable SSH host key checking.

## VPS requirements

- The SSH user can read the Chatwoot checkout and fetch `origin/develop`.
- The checkout is at `/home/chatwoot/chatwoot` and has no local changes that
  would block `git checkout` or `git pull --ff-only`.
- Ruby, Bundler, Node.js, and pnpm are installed and available in the SSH login
  shell, along with the application's production environment configuration.
- The SSH user can run `sudo systemctl restart chatwoot.target` without an
  interactive password. The standard `cwctl` installation grants this to the
  `chatwoot` user.
- PostgreSQL and other production services are already configured and
  available to the application.

The workflow fetches and fast-forwards `develop`, installs dependencies,
precompiles assets, runs database migrations, and restarts the Chatwoot target.
Docker Compose installations need a deployment workflow that
matches their image publishing and Compose configuration.
