# syntax=docker/dockerfile:1.7
FROM ubuntu:22.04

# 通常版Ubuntuへ復元
RUN yes | unminimize \
    && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists/*

# 起動時に実行するコマンド
ENTRYPOINT ["echo 起動成功1"]
