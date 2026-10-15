# WorkBuddy2API (https://github.com/Zhengyuuuui/workbuddy2api)
#
# 上游仓库没有 Dockerfile（是 uv/Python 脚本项目），这个 Dockerfile 由本项目维护。
# 构建上下文必须是上游仓库的检出目录（CI 里会按固定 commit 克隆）。
#
# 关键点：
#   * 依赖直接来自上游 pyproject.toml。
#   * 上游 --host 默认 127.0.0.1，容器里必须改 0.0.0.0，否则端口映射无效。
#   * HOME=/data + --session-file /data/... 让登录态落在挂载卷里，容器重建不丢。
#   * 上游在没有 session / device-token 时只告警不退出（已读源码确认），不会重启环。
#   * 入口脚本会按 CODEBUDDY_PLATFORM 推导 CODEBUDDY_ENDPOINT，
#     这样使用者只需选平台，不必再手工填后端地址。

FROM python:3.11-slim-bookworm

ENV PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    HOME=/data \
    CODEBUDDY_PROXY_HOST=0.0.0.0 \
    CODEBUDDY_PROXY_PORT=8787 \
    CODEBUDDY_PROXY_LOG_FILE=/data/logs/proxy.jsonl

RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates curl \
 && rm -rf /var/lib/apt/lists/*

RUN pip install \
      "fastapi>=0.104.0" \
      "uvicorn>=0.24.0" \
      "httpx[socks]>=0.25.0" \
      "requests>=2.31.0"

WORKDIR /app
COPY . /app

RUN mkdir -p /data/logs

EXPOSE 8787

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
  CMD curl -fsS http://127.0.0.1:8787/health >/dev/null || exit 1

# 国内 CodeBuddy 用 copilot.tencent.com，海外 WorkBuddy AI 用 www.workbuddy.ai。
# 使用者显式设置了 CODEBUDDY_ENDPOINT 时不覆盖。
ENTRYPOINT ["/bin/sh", "-c", "\
if [ -z \"$CODEBUDDY_ENDPOINT\" ]; then \
  case \"$CODEBUDDY_PLATFORM\" in \
    *workbuddy*) CODEBUDDY_ENDPOINT=\"https://www.workbuddy.ai\" ;; \
    *)           CODEBUDDY_ENDPOINT=\"https://copilot.tencent.com\" ;; \
  esac; \
  export CODEBUDDY_ENDPOINT; \
fi; \
exec python server/codebuddy_proxy.py \"$@\"", "--"]
