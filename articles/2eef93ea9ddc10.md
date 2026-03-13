---
title: "dockerでuv環境を構築する。"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python" , "uv" , "fastapi"]
published: false
---
dockerで簡単にpython ✖️ uv ✖️ fastapiの環境を作ってみる。

## dockerfileとcomposeを作成する
まずは`Dockerfile`と`cmopose.yaml`の作成
```bash
├── compose.yaml
├── Dockerfile
```
`Dockerfile`と`compose.yaml`を下記のようにする。
```Dockerfile
FROM ghcr.io/astral-sh/uv:0.9.2-python3.12-bookworm-slim
WORKDIR /app
```
```yaml
services:
  api:
    build: ./
    ports:
      - 8000:8000
    volumes:
      - .:/app
```
追加したら、`docker compose build`でビルドする。

## uv.lockとpyproject.tomlを作成する。
次に`uv.lock`と`pyproject.toml`を作成するために、以下を実行します。
```bash
docker compose exex api bash
uv init && uv lock
```
ファイルが生成されたら、`Dockerfile`に追記します。
```dockerfile
FROM ghcr.io/astral-sh/uv:0.9.2-python3.12-bookworm-slim

WORKDIR /app

# 追記
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen

COPY . .

CMD ["uv", "run", "python", "main.py"]
```
追記後、`docker compose up --build`を実行して、`Hello from app!`が表示されれば成功