# Development Container Templates

CUDA-enabled development container configurations for different architectures.

## Usage

1. Copy the `.devcontainer` directory from your architecture folder to your project root:
   ```bash
   cp -r amd64/.devcontainer /path/to/your/project/
   # or
   cp -r arm64/.devcontainer /path/to/your/project/
   ```

2. Start the container:
   ```bash
   cd /path/to/your/project/.devcontainer
   make run
   ```

3. Attach to the container:
   ```bash
   make attach
   ```

## Architecture Folders

- **amd64/**: x86_64 systems with NVIDIA GPU support (CUDA-enabled, Ubuntu 24.04 base)
- **arm64/**: ARM64 systems like M1/M2 Macs (Debian base, no GPU)

## Base Configuration

The default `Dockerfile` includes:
- NVIDIA CUDA runtime (configurable version)
- Node.js (v20)
- Python 3 + uv package manager
- Neovim
- Claude Code CLI
- zsh with powerlevel10k theme

## Alternative Configurations

### Go Development (amd64 only)

For Go development, replace the `Dockerfile` with `go.Dockerfile`:

```bash
cd /path/to/your/project/.devcontainer
mv Dockerfile Dockerfile.base
mv go.Dockerfile Dockerfile
make rebuild
```

The Go configuration adds:
- Go 1.23.4
- Boot.dev CLI
- GOPATH configuration

## Available Make Commands

- `make run` - Start and attach to container
- `make up` - Start container (create if needed)
- `make attach` - Attach to running container
- `make rebuild` - Rebuild image and restart container
- `make down` - Stop and remove container (keeps volumes)
- `make clean` - Remove container and volumes (DESTRUCTIVE)
- `make status` - Show container and volume status
- `make logs` - Show container logs

## Persistent Data

User data (shell history, configs, cache) is persisted in a Docker volume across container restarts and rebuilds.

To completely reset: `make clean` (WARNING: deletes all persistent data)
