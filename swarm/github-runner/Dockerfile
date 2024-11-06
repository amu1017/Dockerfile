# syntax=docker/dockerfile:1.7
FROM ubuntu:22.04

# GitHub Actions Runnerのバージョンを指定する
# https://github.com/actions/runner/releases
ARG RUNNER_VERSION="2.320.0"

# 一時的にaptのインタラクティブモードをOFFにする
ARG DEBIAN_FRONTEND=noninteractive

# OS日本語設定
RUN apt update && apt install -y --no-install-recommends \
    language-pack-ja-base \
    language-pack-ja \
    locales \
    tzdata \
    && update-locale LANG=ja_JP.UTF8 \
    && ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
    && dpkg-reconfigure --frontend noninteractive tzdata \
    && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists/*

# OSパッケージインストール -基本編-
RUN apt update && apt install -y --no-install-recommends \
    sudo \
    expect \
    zip \
    unzip \
    gawk \
    psmisc \
    dos2unix \
    binutils \
    apt-utils \
    software-properties-common \
    apt-transport-https \
    jq \
    ca-certificates \
    gnupg \
    lsb-release \
    git \
    curl \
    wget \
    && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists/*

# sudo可能な一般ユーザー作成
RUN groupadd -g 9999 group \
    && useradd -m -s /bin/bash -u 9999 -g 9999 -G sudo user \
    && echo "user   ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/NO-PASSWD

# AWS CLIインストール
RUN wget https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip \
    && unzip ./awscli-exe-linux-x86_64.zip \
    && ./aws/install -i /usr/local/aws-cli -b /usr/local/bin \
    && rm -rf ./awscli-exe-linux-x86_64.zip ./aws

# Pwshインストール
RUN wget https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb \
    && dpkg -i ./packages-microsoft-prod.deb \
    && apt update && apt install -y --no-install-recommends powershell \
    && rm -rf ./packages-microsoft-prod.deb \
    && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists/*

# Github CLIインストール
RUN mkdir -p -m 755 /etc/apt/keyrings \
    && wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | tee /etc/apt/keyrings/githubcli.gpg > /dev/null \
    && chmod a+r /etc/apt/keyrings/githubcli.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/githubcli.list > /dev/null \
    && apt update && apt install -y --no-install-recommends gh \
    && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists/*

# Terraform インストール
RUN mkdir -p -m 755 /etc/apt/keyrings \
    && wget -qO- https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /etc/apt/keyrings/hashicorp.gpg > /dev/null \
    && chmod a+r /etc/apt/keyrings/hashicorp.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list > /dev/null \
    && apt update && apt install -y --no-install-recommends terraform \
    && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists/*

# Docker CLIインストール
RUN mkdir -p -m 755 /etc/apt/keyrings \
    && wget -qO- https://download.docker.com/linux/ubuntu/gpg | tee /etc/apt/keyrings/docker.asc > /dev/null \
    && chmod a+r /etc/apt/keyrings/docker.asc \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt update && apt install -y --no-install-recommends docker-ce-cli docker-buildx-plugin \
    && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists/*

# Docker CLI実行設定スクリプトを生成
COPY <<-"EOF" /docker_cli_group.sh
#!/bin/bash
# ----------------Start---------------
DOCKER_GID=$(stat -c '%g' /var/run/docker.sock)
if grep -q "docker" /etc/group; then
    echo "doceker groupmod"
    groupmod -g "${DOCKER_GID}" "docker"
else
    echo "docker groupadd"
    groupadd -g "${DOCKER_GID}" "docker"
    echo "docker usermod"
    usermod -aG docker user
fi
# ----------------End-----------------
EOF
RUN chmod +x /docker_cli_group.sh

# GitHub Actions Runnerインストール
RUN wget https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz \
    && mkdir -p /home/user/actions-runner \
    && tar xzf ./actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz -C /home/user/actions-runner \
    && chown user:group -R /home/user/actions-runner \
    && /home/user/actions-runner/bin/installdependencies.sh \
    && rm -rf ./actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz

# GitHub Actions Runner初期化スクリプトを生成
COPY <<-"EOF" /home/user/actions-runner/register.sh
#!/bin/bash
# ----------------Start---------------
GITHUB_API_URL="https://api.github.com"
GITHUB_URL="https://github.com"
ACCESS_TOKEN="$1"
OWNER="$2"
REPO="$3"
ORGANIZATION="$4"
ENV="$5"

if [ -n "$ORGANIZATION" ]; then
    echo "get token for organization"
    REG_TOKEN=$(curl -sX POST -H "Authorization: token ${ACCESS_TOKEN}" "${GITHUB_API_URL}/orgs/${ORGANIZATION}/actions/runners/registration-token" | jq .token --raw-output)
    echo "setup runner for organization"
    ./config.sh \
    --url "${GITHUB_URL}/${ORGANIZATION}" \
    --token "${REG_TOKEN}" \
    --labels "${ENV}" \
    --disableupdate \
    --ephemeral
else
    echo "get token for personal"
    REG_TOKEN=$(curl -sX POST -H "Authorization: token ${ACCESS_TOKEN}" "${GITHUB_API_URL}/repos/${OWNER}/${REPO}/actions/runners/registration-token" | jq .token --raw-output)
    echo "setup runner for personal"
    ./config.sh \
    --url "${GITHUB_URL}/${OWNER}/${REPO}" \
    --token "${REG_TOKEN}" \
    --disableupdate \
    --ephemeral
fi
echo "${REG_TOKEN}" > "./REG_TOKEN.txt"
# ----------------End-----------------
EOF
RUN chmod +x /home/user/actions-runner/register.sh \
    && chown user:group /home/user/actions-runner/register.sh

# OSパッケージの不要ファイル掃除
RUN apt upgrade -y \
    && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists/*

# Entrypointを生成
COPY <<-"EOF" /entrypoint.sh
#!/bin/bash
# ----------------Start---------------
echo @@@@@@@@ register dockercli exec group @@@@@@@@
/docker_cli_group.sh

echo @@@@@@@@ register github runner @@@@@@@@
su - user -c " \
cd /home/user/actions-runner && \
./register.sh \
$(cat /run/secrets/access_token) \
${PERSONAL_OWNER} \
${PERSONAL_REPO} \
${ORGNIZATION_NAME} \
${ORGNIZATION_ENV} \
"

cleanup() {
    echo @@@@@@@@ cleanup github runner @@@@@@@@
    su - user -c "
    cd /home/user/actions-runner && \
    ./config.sh remove \
    --token $(cat /home/user/actions-runner/REG_TOKEN.txt) \
    "
}
trap 'echo catch 130 INT;  cleanup; exit 130' INT
trap 'echo catch 143 TERM; cleanup; exit 143' TERM

echo @@@@@@@@ exec github runner @@@@@@@@
su - user -c "
    cd /home/user/actions-runner && \
    ./run.sh
    " & wait $!
# ----------------End-----------------
EOF
RUN chmod +x /entrypoint.sh

# 起動時に実行するコマンド
ENTRYPOINT ["/entrypoint.sh"]
