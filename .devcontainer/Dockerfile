# 1. Base Image: Ubuntu 22.04
FROM ubuntu:22.04

ARG TERRAFORM_VERSION="1.10.3"
ARG TERRAGRUNT_VERSION="0.72.0"
ARG GCLOUD_VERSION="505.0.0"
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# 2. Install Basic Tools + Zsh
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
       ca-certificates curl git unzip vim sudo gnupg python3 python3-pip python3-venv \
       zsh openssh-client fonts-powerline \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. Create User & Set Shell to Zsh
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/zsh \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# =========================================================
# 4. Install Oh My Zsh & Powerlevel10k (As the user)
# =========================================================
USER $USERNAME
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended \
    && git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k \
    # Optional: Install common plugins to prevent errors if your .zshrc uses them
    && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
    && git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# Switch back to root to finish system installs
USER root

# 5. Install tfenv (Terraform) - Fix permissions at the end
RUN git clone --depth=1 https://github.com/tfutils/tfenv.git /home/$USERNAME/.tfenv \
    && ln -s /home/$USERNAME/.tfenv/bin/* /usr/local/bin \
    && tfenv install ${TERRAFORM_VERSION} \
    && tfenv use ${TERRAFORM_VERSION} \
    && chown -R $USERNAME:$USERNAME /home/$USERNAME/.tfenv

# 6. Install tgswitch (Terragrunt)
RUN curl -L https://github.com/warrensbox/tgswitch/releases/latest/download/tgswitch_linux_$(dpkg --print-architecture) -o /usr/local/bin/tgswitch \
    && chmod +x /usr/local/bin/tgswitch \
    && curl -L https://github.com/gruntwork-io/terragrunt/releases/download/v${TERRAGRUNT_VERSION}/terragrunt_linux_$(dpkg --print-architecture) -o /usr/local/bin/terragrunt \
    && chmod +x /usr/local/bin/terragrunt

# 7. Install Google Cloud SDK
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
    && curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg \
    && apt-get update \
    && apt-get install -y google-cloud-cli=${GCLOUD_VERSION}-0

# 8. Set Default User
USER $USERNAME