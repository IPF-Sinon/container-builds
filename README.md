# workbuddy-trae-images

给 [WorkBuddy2API](https://github.com/Zhengyuuuui/workbuddy2api) 和
[trae2api](https://github.com/JeffHu0912/trae2api) 自建容器镜像。

## 为什么需要这个仓库

上游两个项目**都没有可用的公开镜像**（已核实）：

| 项目 | 情况 |
| --- | --- |
| `Zhengyuuuui/workbuddy2api` | 仓库里**没有任何 Dockerfile / compose**，是 uv + Python 脚本项目；作者也没有公开的 GHCR 包 |
| `JeffHu0912/trae2api` | 有 Dockerfile，但自己的 compose 用的是 `build: .`，**没有发布镜像**；作者公开的 GHCR 包只有 `omniroute` |

所以 `ghcr.io/zhengyuuuui/workbuddy2api:latest`、`ghcr.io/jeffhu0912/trae2api:latest`
这两个地址是**拉不到的**，必须自己构建。

## 用法

1. 新建一个 GitHub 仓库（public 或 private 都行），把本目录内容推上去：
   ```bash
   git init
   git add .
   git commit -m "build workbuddy2api & trae2api images"
   git remote add origin git@github.com:<你的用户名>/workbuddy-trae-images.git
   git push -u origin main
   ```
2. Actions 会自动跑（也可以手动 `workflow_dispatch`）。
3. 构建完成后镜像地址：
   ```
   ghcr.io/<你的用户名>/workbuddy2api:latest
   ghcr.io/<你的用户名>/trae2api:latest
   ```

## 仓库为 private 时

GHCR 包默认也是 private，1Panel 宿主机要先登录才能拉取：

```bash
docker login ghcr.io -u <你的用户名> -p <你的PAT>
```

PAT 需要 `read:packages` 权限。仓库设为 public 则镜像自动公开，无需登录。

## 构建内容

| 镜像 | Dockerfile | 说明 |
| --- | --- | --- |
| `workbuddy2api` | `docker/workbuddy2api/Dockerfile`（本仓库维护） | 上游没有 Dockerfile。Python 3.11 + 上游依赖；`HOME=/data`；入口按 `CODEBUDDY_PLATFORM` 推导 `CODEBUDDY_ENDPOINT` |
| `trae2api` | 上游自带 `Dockerfile`（Go 多阶段、非 root、含 healthcheck） | 直接复用，不重复维护 |

两个镜像都按 `linux/amd64` + `linux/arm64` 双架构构建。

## 上游版本固定

`.github/workflows/build-images.yml` 里用 commit SHA 固定上游版本，构建可复现：

- workbuddy2api `1cc32c9a98abb53796d8d4e8d477736fe1f0c4cc`（2026-10-09）
- trae2api `ec20c088a375bc5a2989d43aa5b561b06a4382e7`（2026-09-08）

要升级就改这两个 SHA 后重新 push。

## 自检（可选，本地）

```bash
git clone https://github.com/Zhengyuuuui/workbuddy2api /tmp/up-wb
git -C /tmp/up-wb checkout 1cc32c9a98abb53796d8d4e8d477736fe1f0c4cc
docker build -f docker/workbuddy2api/Dockerfile -t workbuddy2api:test /tmp/up-wb

git clone https://github.com/JeffHu0912/trae2api /tmp/up-trae
git -C /tmp/up-trae checkout ec20c088a375bc5a2989d43aa5b561b06a4382e7
docker build -t trae2api:test /tmp/up-trae
```
