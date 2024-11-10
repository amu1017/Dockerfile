# syntax=docker/dockerfile:1.7
#FROM roundcube/roundcubemail:1.5.3-apache
FROM roundcube/roundcubemail:latest

# ============パッケージインストール作業============
USER root

# 一時的にaptのインタラクティブモードをOFFにする
ARG DEBIAN_FRONTEND=noninteractive

# OSパッケージインストール -基本編-
RUN apt update && apt install -y --no-install-recommends \
    unzip \
    git \
    && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists/*

# 証明書検証を無効
RUN sed -i "s/DEFAULT@SECLEVEL=2/DEFAULT@SECLEVEL=1/" /etc/ssl/openssl.cnf

# imap 接続時のオプション設定
COPY <<-"EOF" /var/roundcube/config/config.docker.add_imap_conn_options.inc.php
<?php
// IMAP socket context options
$config['imap_conn_options'] = [
    'ssl'         => [
        'verify_peer'  => true,
        'verify_depth' => 3,
    ],
];
// SMTP socket context options
$config['smtp_conn_options'] = [
    'ssl'         => [
        'verify_peer'  => true,
        'verify_depth' => 3,
    ],
];
EOF
