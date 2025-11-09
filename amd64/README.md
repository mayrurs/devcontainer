# Development Container Setup

This repository provides two ways to run the development environment:
1. **VS Code DevContainer** - Integrated with VS Code
2. **Docker Compose** - Standalone, works with any editor

Both setups use the same Dockerfile and provide identical environments.

## Prerequisites

- Docker installed
- For VS Code setup: VS Code with Remote-Containers extension
- For font rendering: Install [MesloLGS NF](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k) on your local machine

## VS Code DevContainer Setup

1. Open this folder in VS Code
2. Press `F1` and select "Dev Containers: Reopen in Container"
3. Wait for the container to build and start

### Customization

Edit `.devcontainer/devcontainer.json` to customize build args, mounts, and settings.

## Docker Compose Setup

### Quick Start

```bash
# Build the image (first time only)
docker compose build

# Start an interactive shell
docker compose run --rm dev
```

**Note:** All configuration has sensible defaults in `docker-compose.yml`. You can optionally create a `.env` file to override them (see Configuration section below).

### Usage Commands

```bash
# Start an interactive shell (recommended)
docker compose run --rm dev

# Start in background (for long-running services)
docker compose up -d

# Attach to a running background container
docker compose exec dev zsh

# Stop background container
docker compose down

# Rebuild after Dockerfile changes
docker compose build

# Rebuild without cache
docker compose build --no-cache

# Remove container and volumes (WARNING: deletes persistent data)
docker compose down -v
```

### Connecting to Ollama

The devcontainer is configured to connect to Ollama running in Docker. There are several ways to set this up:

**Option 1: Ollama on Host Machine**
```bash
# In .env file:
OLLAMA_HOST=http://host.docker.internal:11434
```
The devcontainer will connect to Ollama running on your host at port 11434.

**Option 2: Ollama in Same Docker Compose**
Add Ollama to your docker-compose.yml:
```yaml
services:
  dev:
    # ... existing config ...

  ollama:
    image: ollama/ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama-data:/root/.ollama

volumes:
  ollama-data:
```
Then set in `.env`:
```bash
OLLAMA_HOST=http://ollama:11434
```

**Option 3: Ollama in Separate Docker Compose (Same Network)**
If Ollama is in a different docker-compose.yml, you can connect them via a shared external network:

```bash
# 1. Create a shared network (one time only):
docker network create ollama-network

# 2. In your Ollama docker-compose.yml:
services:
  ollama:
    # ... existing config ...
    networks:
      - ollama-network

networks:
  ollama-network:
    external: true

# 3. In your devcontainer docker-compose.yml, add:
services:
  dev:
    # ... existing config ...
    networks:
      - default
      - ollama-network

networks:
  default:
  ollama-network:
    external: true

# 4. In .env:
OLLAMA_HOST=http://ollama:11434
```

**Test the connection:**
```bash
# Inside devcontainer
curl $OLLAMA_HOST/api/tags
```

**Note:** The firewall is configured to allow connections to your local network (where Ollama runs), so all three options above work with the firewall enabled.

### Firewall Configuration

The devcontainer includes a restrictive firewall that only allows connections to:
- ✅ GitHub (web, API, git)
- ✅ npm registry
- ✅ Anthropic API
- ✅ VS Code marketplace
- ✅ **Your local network** (for Ollama, databases, etc.)
- ✅ DNS and SSH
- ❌ Everything else on the internet

**To disable the firewall** (not recommended):
```bash
# In .env:
ENABLE_FIREWALL=false
```

**To allow additional domains**, edit `init-firewall.sh` and add them to the `allowed-domains` list.

### SSH Agent Forwarding

To use git with SSH inside the container:

**Linux/macOS:**
```bash
# Ensure SSH agent is running
eval $(ssh-agent)
ssh-add ~/.ssh/id_rsa  # or your key

# Start container (SSH_AUTH_SOCK is auto-forwarded)
docker-compose up -d
```

**Windows:**
```bash
# Use pageant or enable OpenSSH Authentication Agent service
# Then update docker-compose.yml SSH_AUTH_SOCK path
```

## Configuration

All settings have defaults in `docker-compose.yml`. To customize, create a `.env` file in the project root:

| Variable | Default | Description |
|----------|---------|-------------|
| `TZ` | America/Los_Angeles | Timezone |
| `CLAUDE_CODE_VERSION` | latest | Claude Code version |
| `NODE_VERSION` | 20 | Node.js major version |
| `USERNAME` | devuser | Container username |
| `USER_UID` | 1000 | User UID (match your host UID) |
| `USER_GID` | 1000 | User GID (match your host GID) |
| `DOTFILES_REPO` | mayrurs/.dotfiles | Your dotfiles repo |
| `DOTFILES_REF` | main | Branch/tag for dotfiles |
| `LAZYVIM_REPO` | mayrurs/lazy-vim | Your LazyVim config repo |
| `LAZYVIM_REF` | main | Branch/tag for LazyVim |
| `WORKSPACE_DIR` | . | Local directory to mount |
| `OLLAMA_HOST` | http://host.docker.internal:11434 | Ollama server URL |
| `ENABLE_FIREWALL` | true | Enable restrictive firewall |

**Example `.env` file:**
```bash
# Only include variables you want to override
DOTFILES_REPO=yourusername/.dotfiles
LAZYVIM_REPO=yourusername/lazy-vim
TZ=Europe/Zurich
```

### Persistent Volumes

Both setups use a single named volume for all persistent data:

- `devcontainer-data` - Contains all user data:
  - Shell history (`.history/`)
  - Claude configuration (`.claude/`)
  - Cache directory (`.cache/`)
  - Local data (`.local/`)
  - Neovim plugins and configuration

This volume persists across container rebuilds, so you don't lose installed plugins, history, or configuration.

**Volume Scope:**
- **VS Code DevContainer**: Per-project volume (uses `${devcontainerId}`)
- **Docker Compose**: Per-project volume (prefixed with project directory name)
  - Example: If project is in `/home/user/myproject`, volume will be named `myproject_devcontainer-data`
  - To share volume globally across all projects, add `name: devcontainer-data` in `docker-compose.yml`

**View your volumes:**
```bash
docker volume ls | grep devcontainer
```

## What's Included

- **Debian Bookworm** base
- **Node.js** (configurable version, default 20)
- **Neovim** (latest release, with LazyVim config)
- **Development tools**: git, ripgrep, fd, fzf, jq, gh
- **Build tools**: gcc, cmake, ninja, pkg-config
- **Zsh** with powerlevel10k theme
- **Claude Code CLI**
- **Your dotfiles** automatically installed
- **Nerd Fonts** for proper icon rendering

## Differences Between Setups

| Feature | VS Code DevContainer | Docker Compose |
|---------|---------------------|----------------|
| Editor Integration | ✅ Full VS Code integration | ❌ Use external editor |
| Port Forwarding | ✅ Automatic | 🔧 Manual (add to compose) |
| VS Code Extensions | ✅ Auto-installed | ❌ N/A |
| Persistent Volumes | ✅ Per-project | ✅ Shared |
| SSH Forwarding | ✅ Automatic | 🔧 Manual setup |
| Font Configuration | ✅ Auto-configured | 🔧 Configure terminal |

## Troubleshooting

### Font symbols not showing

Install MesloLGS NF fonts on your **local machine** and configure your terminal to use them.

### Permission errors on volumes

Ensure `USER_UID` and `USER_GID` in `.env` match your host user:
```bash
echo "USER_UID=$(id -u)" >> .env
echo "USER_GID=$(id -g)" >> .env
```

### Git authentication not working

For HTTPS: Use credential helper or personal access tokens
For SSH: Ensure SSH agent forwarding is configured (see above)

### Container won't start

Check logs:
```bash
docker-compose logs
```

Rebuild from scratch:
```bash
docker-compose down -v
docker-compose build --no-cache
docker-compose up -d
```
