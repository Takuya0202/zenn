---
title: "Docker・uvで FastAPI 開発環境を構築する"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python" , "uv" , "fastapi" , "docker" , "devcontainer"]
published: true
---
# はじめに
今回はDockerを使って、簡単にチーム開発ができるようにするための環境を構築します。
以下のような点を意識して、構築します。
- ホストにはuvやpythonを入れず、全てdockerとdevcontainerだけで完結する。
- クローンした人はコンテナを立ち上げるだけですぐに開発ができるようにする。
- なるべく最小限の構成にする。

## dockerfileとcomposeを作成する
まず`Dockerfile`と`compose.yaml`を作成する。
```bash
├── compose.yaml
├── Dockerfile
```
`Dockerfile`と`compose.yaml`の内容は以下の通り。
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
作成したら、`docker compose build`でビルドします。

## uv.lockとpyproject.tomlを作成する
`uv.lock`と`pyproject.toml`を作成するには、以下を実行する。
```bash
docker compose run api bash
uv init && uv lock
```
実行後、ファイル構成は次のようになる。
```bash
.
├── Dockerfile
├── README.md
├── compose.yaml
├── main.py
├── pyproject.toml
└── uv.lock
```

## dockerfileを更新する
続いてDockerfileを更新する。以下の内容に更新する。
```dockerfile
FROM ghcr.io/astral-sh/uv:0.9.2-python3.12-bookworm-slim

WORKDIR /app

# 追記
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen

COPY . .

CMD ["uv", "run", "python", "main.py"]
```
更新後、`docker compose up --build`を実行して、`Hello from app!`が表示されれば成功

## fastapiを導入する
次にFastAPIをインストールする。コンテナ内で以下を実行する。
```bash
docker compose run api bash
uv add fastapi[standard]
```
実行後、`main.py`の構造と内容を変更する。
`app`ディレクトリを作成し、その中に`main.py`を配置する。
```bash
.
├── Dockerfile
├── README.md
├── app  # appの中にセット
│   └── main.py
├── compose.yaml
├── pyproject.toml
└── uv.lock
```
`main.py`の内容は以下とする。
```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```
あわせて、DockerfileのCMDを以下に変更する。
```dockerfile
FROM ghcr.io/astral-sh/uv:0.9.2-python3.12-bookworm-slim

WORKDIR /app

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen

COPY . .

# 変更
CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```
上記の変更を反映したら、`docker compose up -d --build`で再ビルドする。

## devcontainerの作成
最後にdevcontainerを作成する。以下のファイル構成にする。
```bash
.
├── .devcontainer
│   └── devcontainer.json
├── .python-version
├── Dockerfile
├── README.md
├── app
│   └── main.py
├── compose.yaml
├── pyproject.toml
└── uv.lock
```
`devcontainer.json`の内容は以下とする。
```json
{
    "name": "fastapi",
    "dockerComposeFile": "../compose.yaml",
    "service": "api",
    "workspaceFolder": "/app",
    "customizations": {
      "vscode": {
        "extensions": [
          "ms-python.python",
          "ms-python.vscode-pylance",
          "ms-azuretools.vscode-docker",
          "charliermarsh.ruff"
        ]
      }
    }
  }
```
拡張機能などは必要に応じて編集してください。
追記したら、`Ctrl+Shift+P`（Mac: `Cmd+Shift+P`）で`Reopen in Container`を選択する。
最後に`http://localhost:8000/docs`を開いてswaggerの画面が表示されてたら成功です。

