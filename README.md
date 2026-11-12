# NasOC 编程终端 —— 飞牛 fnOS 版（.fpk 打包工程）

在飞牛 fnOS 上以应用中心 `.fpk` 形态运行 **NasOC 编程终端**（内置 OpenCode 的
NAS AI 编程终端，tmux 持久会话）。主项目（UGOS Pro 版、完整文档与隐私政策）
见 **[firewelly/nasoc](https://github.com/firewelly/nasoc)**。

> 独立维护的 fnOS 适配，与其他同名/类似项目无分支关系。

## 仓库内容

| 目录 | 说明 |
|---|---|
| `fnos/` | .fpk 打包工程：`build_fpk.py`、manifest、安装向导、桌面入口、启停脚本 |
| `PRIVACY.md` | 隐私政策（与主仓库同源） |

镜像构建源（`core/` + `vendor/bin/opencode`）在主仓库 firewelly/nasoc——
构建镜像时把本仓库 `fnos/` 放进主仓库根目录即可，详见 [fnos/README.md](fnos/README.md)。

安装包在 **Releases**：`nasoc-fnos-<版本>.fpk`（负载内自带离线 Docker 镜像，约 400MB）。

## 快速开始

**打包**（x86_64 + Docker 机器，如 fnOS 本机）：

```bash
git clone https://github.com/firewelly/nasoc nasoc && cd nasoc
git clone https://github.com/firewelly/nasoc-fnos fnos
docker build -f core/Dockerfile -t nasoc:0.1.0 .
python3 fnos/build_fpk.py            # 产物 fnos/dist/nasoc-fnos-0.1.0.fpk
```

**安装**（fnOS 应用中心 → 手动安装）：

- 向导字段：项目根目录（多 tab）、数据目录（建议 SSD 数据卷）、访问口令 PIN（可选）
- **安装器没有 API Key 字段**——装好后打开终端按 opencode 引导配置，凭据存数据卷（0600）
- 桌面图标「NasOC 编程终端」→ `http://<NAS>:12703`；应用启停 = 容器 `nasoc` 启停

## 规格与注意

- 端口固定 **12703**；镜像约 1.3GB、fpk 约 400MB
- 需要 fnOS 已启用 Docker 且版本 ≥ 20.10
- 登录安全：挑战-应答（明文口令不出浏览器、不进 URL/日志）；PIN 经 entrypoint
  落盘 0600 后从环境清除

## 隐私与许可

MIT —— 见 [LICENSE](LICENSE)；隐私政策见 [PRIVACY.md](PRIVACY.md)（v1.2，
与 UGOS 版审核提交版本同源）。
