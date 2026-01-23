# 1. Base Image: Ubuntu 22.04
FROM ubuntu:22.04

ARG TERRAFORM_VERSION="1.10.3"
ARG TERRAGRUNT_VERSION="0.72.0"
ARG GCLOUD_VERSION="505.0.0"
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# 2. Install Dependencies + Zsh + SSH + Python
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
    ca-certificates curl git unzip vim sudo gnupg \
    python3 python3-pip python3-venv \
    openssh-client zsh fonts-powerline \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. Create User
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /usr/bin/zsh \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# 4. Install PYTHON Tools (Pre-commit & Checkov)
RUN pip3 install pre-commit checkov

# =========================================================
# 5. FIX: Install BINARY Tools (Dynamic Architecture)
# =========================================================

# GITLEAKS FIX:
# 1. Detects if you are on Intel (amd64) or Apple Silicon (arm64).
# 2. Maps 'amd64' to 'x64' because Gitleaks naming is inconsistent.
# 3. Uses 'curl -L' to follow GitHub redirects properly.
RUN ARCH=$(dpkg --print-architecture) && \
    case "$ARCH" in \
    amd64) GITLEAKS_ARCH="x64" ;; \
    arm64) GITLEAKS_ARCH="arm64" ;; \
    *) echo "Unsupported architecture: $ARCH"; exit 1 ;; \
    esac && \
    curl -fsSL "https://github.com/gitleaks/gitleaks/releases/download/v8.18.2/gitleaks_8.18.2_linux_${GITLEAKS_ARCH}.tar.gz" \
    | tar -xz -C /usr/local/bin gitleaks

# TFLint
RUN curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash

# Terraform-Docs (FIXED)
# 1. Defines version.
# 2. Downloads the tar.gz.
# 3. Extracts (-x) and files (-f) to standard output (-O) doesn't work well with binary piping,
#    so we download, untar, install, and cleanup.
RUN TFDOCS_VERSION="v0.19.0" && \
    curl -Lo terraform-docs.tar.gz "https://github.com/terraform-docs/terraform-docs/releases/download/${TFDOCS_VERSION}/terraform-docs-${TFDOCS_VERSION}-$(uname -s | tr '[:upper:]' '[:lower:]')-$(dpkg --print-architecture).tar.gz" \
    && tar -xzf terraform-docs.tar.gz \
    && chmod +x terraform-docs \
    && mv terraform-docs /usr/local/bin/terraform-docs \
    && rm terraform-docs.tar.gz

# =========================================================

# 6. Install Zsh Theme (User level)
USER $USERNAME
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended \
    && git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k \
    && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
    && git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 7. Install Terraform & Terragrunt (Root level for permissions)
USER root
RUN git clone --depth=1 https://github.com/tfutils/tfenv.git /home/$USERNAME/.tfenv \
    && ln -s /home/$USERNAME/.tfenv/bin/* /usr/local/bin \
    && tfenv install ${TERRAFORM_VERSION} \
    && tfenv use ${TERRAFORM_VERSION} \
    && chown -R $USERNAME:$USERNAME /home/$USERNAME/.tfenv

RUN curl -L https://github.com/warrensbox/tgswitch/releases/latest/download/tgswitch_linux_$(dpkg --print-architecture) -o /usr/local/bin/tgswitch \
    && chmod +x /usr/local/bin/tgswitch \
    && curl -L https://github.com/gruntwork-io/terragrunt/releases/download/v${TERRAGRUNT_VERSION}/terragrunt_linux_$(dpkg --print-architecture) -o /usr/local/bin/terragrunt \
    && chmod +x /usr/local/bin/terragrunt

# 8. Install Google Cloud SDK
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
    && curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg \
    && apt-get update \
    && apt-get install -y google-cloud-cli=${GCLOUD_VERSION}-0

USER $USERNAME
