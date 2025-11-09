# CUDA configuration
ARG CUDA_VERSION=13.0.1
ARG CUDA_DISTRO=ubuntu24.04
ARG CUDA_TYPE=runtime

FROM nvidia/cuda:${CUDA_VERSION}-${CUDA_TYPE}-${CUDA_DISTRO}

ARG TZ
ENV TZ="$TZ"

ARG CLAUDE_CODE_VERSION=latest
ARG NODE_VERSION=20

# Define user configuration as build arguments
ARG USERNAME=devuser
ARG USER_UID=1000
ARG USER_GID=1000

# Install Node.js from NodeSource (supports both amd64 and arm64)
RUN apt-get update && apt-get install -y --no-install-recommends \
  ca-certificates \
  curl \
  gnupg \
  && mkdir -p /etc/apt/keyrings \
  && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
  && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_${NODE_VERSION}.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
  && apt-get update \
  && apt-get install -y --no-install-recommends nodejs \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install basic development tools and iptables/ipset
RUN apt-get update && apt-get install -y --no-install-recommends \
  less \
  git \
  procps \
  sudo \
  fzf \
  zsh \
  man-db \
  unzip \
  gnupg2 \
  gh \
  iptables \
  ipset \
  iproute2 \
  dnsutils \
  aggregate \
  jq \
  nano \
  vim \
  wget \
  build-essential \
  cmake \
  ninja-build \
  pkg-config \
  ripgrep \
  fd-find \
  xz-utils \
  fontconfig \
  && ln -sf /usr/bin/fdfind /usr/local/bin/fd \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Nerd Fonts (MesloLGS NF for powerlevel10k and dev icons)
RUN set -eux; \
  mkdir -p /usr/share/fonts/truetype/meslo; \
  cd /usr/share/fonts/truetype/meslo; \
  curl -fLO https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf; \
  curl -fLO https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf; \
  curl -fLO https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf; \
  curl -fLO https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf; \
  fc-cache -fv

# Install latest Neovim (supports both amd64 and arm64)
RUN set -eux; \
  ARCH=$(dpkg --print-architecture); \
  if [ "$ARCH" = "amd64" ]; then \
    NVIM_ARCH="linux-x86_64"; \
  elif [ "$ARCH" = "arm64" ]; then \
    NVIM_ARCH="linux-aarch64"; \
  else \
    echo "Unsupported architecture: $ARCH" && exit 1; \
  fi; \
  curl -LO "https://github.com/neovim/neovim/releases/latest/download/nvim-${NVIM_ARCH}.tar.gz"; \
  rm -rf /opt/nvim-linux; \
  tar -C /opt -xzf "nvim-${NVIM_ARCH}.tar.gz"; \
  mv "/opt/nvim-${NVIM_ARCH}" /opt/nvim-linux; \
  ln -sf /opt/nvim-linux/bin/nvim /usr/local/bin/nvim; \
  rm "nvim-${NVIM_ARCH}.tar.gz"; \
  nvim --version | head -n 1

# Create user with specified UID/GID (handle existing users)
RUN set -eux; \
  # Remove any existing user with the target UID if it exists
  existing_user=$(getent passwd $USER_UID | cut -d: -f1 || true); \
  if [ -n "$existing_user" ]; then \
    userdel -r "$existing_user" 2>/dev/null || true; \
  fi; \
  # Remove any existing group with the target GID if it exists
  existing_group=$(getent group $USER_GID | cut -d: -f1 || true); \
  if [ -n "$existing_group" ]; then \
    groupdel "$existing_group" 2>/dev/null || true; \
  fi; \
  # Create the new group and user
  groupadd --gid $USER_GID $USERNAME; \
  useradd --uid $USER_UID --gid $USER_GID -m -s /bin/zsh $USERNAME

# Ensure user has access to /usr/local/share
RUN mkdir -p /usr/local/share/npm-global && \
  chown -R $USERNAME:$USERNAME /usr/local/share

ARG USERNAME=$USERNAME

# Persist bash history in home directory
RUN mkdir -p /home/$USERNAME/.history \
  && touch /home/$USERNAME/.history/.bash_history \
  && chown -R $USERNAME:$USERNAME /home/$USERNAME/.history

# Set `DEVCONTAINER` environment variable to help with orientation
ENV DEVCONTAINER=true

# Create workspace and config directories and set permissions
RUN mkdir -p /workspace /home/$USERNAME/.claude /home/$USERNAME/.cache /home/$USERNAME/.local && \
  chown -R $USERNAME:$USERNAME /workspace /home/$USERNAME/.claude /home/$USERNAME/.cache /home/$USERNAME/.local

WORKDIR /workspace

ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# Install Python (required for uv)
RUN apt-get update && apt-get install -y --no-install-recommends \
  python3 \
  python3-pip \
  python3-venv \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install uv (supports both amd64 and arm64)
RUN curl -LsSf https://astral.sh/uv/install.sh | sh && \
  mv /root/.local/bin/uv /usr/local/bin/uv && \
  mv /root/.local/bin/uvx /usr/local/bin/uvx && \
  uv --version

# Set up non-root user
USER $USERNAME

# Install global packages
ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=$PATH:/usr/local/share/npm-global/bin

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Set the default editor and visual
ENV EDITOR=nano
ENV VISUAL=nano

# Default powerline10k theme
ARG ZSH_IN_DOCKER_VERSION=1.2.0
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v${ZSH_IN_DOCKER_VERSION}/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -a "export HISTFILE=\$HOME/.history/.zsh_history" \
  -a "export NPM_CONFIG_PREFIX=/usr/local/share/npm-global" \
  -a "export PATH=\$PATH:/usr/local/share/npm-global/bin" \
  -x

# Create .zshenv to ensure environment variables are always set early
RUN echo 'export NPM_CONFIG_PREFIX=/usr/local/share/npm-global' > /home/$USERNAME/.zshenv && \
  echo 'export PATH=$NPM_CONFIG_PREFIX/bin:$PATH' >> /home/$USERNAME/.zshenv && \
  chown $USERNAME:$USERNAME /home/$USERNAME/.zshenv

# Configure dotfiles and LazyVim
ARG DOTFILES_REPO="mayrurs/.dotfiles"
ARG DOTFILES_REF="main"
ARG LAZYVIM_REPO="mayrurs/lazy-vim"
ARG LAZYVIM_REF="main"

RUN set -eux; \
  mkdir -p /home/$USERNAME/configuration /home/$USERNAME/.config; \
  \
  # Download and extract dotfiles
  curl -fsSL "https://codeload.github.com/${DOTFILES_REPO}/tar.gz/refs/heads/${DOTFILES_REF}" -o /tmp/dotfiles.tar.gz; \
  mkdir -p /home/$USERNAME/configuration/.dotfile; \
  tar -xzf /tmp/dotfiles.tar.gz -C /home/$USERNAME/configuration/.dotfile --strip-components=1; \
  rm -f /tmp/dotfiles.tar.gz; \
  \
  # Symlink dotfiles (only if they exist)
  [ -f /home/$USERNAME/configuration/.dotfile/.zshrc ] && ln -sf /home/$USERNAME/configuration/.dotfile/.zshrc /home/$USERNAME/.zshrc || true; \
  [ -f /home/$USERNAME/configuration/.dotfile/.tmux.conf ] && ln -sf /home/$USERNAME/configuration/.dotfile/.tmux.conf /home/$USERNAME/.tmux.conf || true; \
  [ -f /home/$USERNAME/configuration/.dotfile/.p10k.zsh ] && ln -sf /home/$USERNAME/configuration/.dotfile/.p10k.zsh /home/$USERNAME/.p10k.zsh || true; \
  \
  # Download and extract LazyVim config
  curl -fsSL "https://codeload.github.com/${LAZYVIM_REPO}/tar.gz/refs/heads/${LAZYVIM_REF}" -o /tmp/lazy-vim.tar.gz; \
  mkdir -p /home/$USERNAME/configuration/lazy-vim; \
  tar -xzf /tmp/lazy-vim.tar.gz -C /home/$USERNAME/configuration/lazy-vim --strip-components=1; \
  rm -f /tmp/lazy-vim.tar.gz; \
  \
  # Symlink LazyVim config
  ln -sfn /home/$USERNAME/configuration/lazy-vim /home/$USERNAME/.config/nvim

# Install Claude and mcp-hub (for mcphub.nvim plugin)
RUN npm install -g @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION} && \
  npm install -g mcp-hub@latest

# Copy and set up firewall script
COPY init-firewall.sh /usr/local/bin/
USER root
RUN chmod +x /usr/local/bin/init-firewall.sh && \
  echo "$USERNAME ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/${USERNAME}-firewall && \
  chmod 0440 /etc/sudoers.d/${USERNAME}-firewall
USER $USERNAME
