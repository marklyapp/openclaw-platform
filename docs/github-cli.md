# GitHub CLI Auth for Sandboxed Agents

Agents run inside Docker containers and need GitHub access to clone repos, create issues, open PRs, and manage code. This is done by authenticating the GitHub CLI (`gh`) once and bind-mounting the credentials into every container.

## 1. Build the Sandbox Image

If you haven't already:

```bash
docker build -f Dockerfile.sandbox -t openclaw-sandbox:bookworm-slim .
```

## 2. Create the Auth Directory

```bash
mkdir -p gh-config
```

This directory will hold the GitHub CLI auth tokens. It's bind-mounted into containers at `/root/.config/gh`.

## 3. Authenticate

Run the interactive login inside a throwaway container:

```bash
docker run --rm -it \
  -v "$(pwd)/gh-config:/root/.config/gh" \
  openclaw-sandbox:bookworm-slim \
  gh auth login
```

Choose:
- **GitHub.com** (unless you use GitHub Enterprise)
- **HTTPS** protocol
- **Login with a web browser** (recommended) or paste a token

## 4. Verify

```bash
docker run --rm \
  -v "$(pwd)/gh-config:/root/.config/gh" \
  openclaw-sandbox:bookworm-slim \
  gh auth status
```

You should see your GitHub username and the scopes granted.

## 5. Configure the Bind Mount

In your workflow's `openclaw.json`, the `gh-config` directory is mounted read-only into every agent's sandbox:

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "docker": {
          "binds": [
            "//path/to/gh-config:/root/.config/gh:ro"
          ]
        }
      }
    }
  }
}
```

The `setup.sh` in each workflow repo handles this path automatically.

## Scopes

The default `gh auth login` grants these scopes, which are sufficient for most workflows:
- `repo` — full access to repositories
- `read:org` — read org membership
- `gist` — create gists

If your workflow needs additional scopes (e.g., `admin:org` for managing teams), re-run `gh auth login` with `--scopes`:

```bash
docker run --rm -it \
  -v "$(pwd)/gh-config:/root/.config/gh" \
  openclaw-sandbox:bookworm-slim \
  gh auth login --scopes "repo,read:org,admin:org"
```

## Multiple GitHub Accounts

If different workflows need different GitHub accounts, use separate `gh-config` directories:

```
gh-config-devpod/     ← bound into dev pod agents
gh-config-support/    ← bound into support bot agents
```

## Refreshing Auth

Tokens can expire. If agents start getting 401 errors, re-run the auth:

```bash
docker run --rm -it \
  -v "$(pwd)/gh-config:/root/.config/gh" \
  openclaw-sandbox:bookworm-slim \
  gh auth login
```
