# fnOS 打包（.fpk，X86_64）—— NasOC

把 NasOC 打成 fnOS 应用中心可安装的 `.fpk`：**负载内自带离线 Docker 镜像**
（`docker save|gzip`），安装/升级时 `docker load`，启停用 `docker run/rm` 管理
容器 `nasoc`，不依赖 fnOS 应用框架的 Docker 编排。

## 打包（在 x86_64 + Docker 的机器上执行，如 fnOS 本机）

镜像构建上下文（含 `vendor/bin/opencode`）在主仓库 **firewelly/nasoc**：

```bash
git clone https://github.com/firewelly/nasoc nasoc && cd nasoc
# 把本仓库的 fnos/ 放进 nasoc 仓库根（或任意位置用 --fnos 指定思路同 build_fpk.py 的 HERE 定位）
docker build -f core/Dockerfile -t nasoc:0.1.0 .
python3 fnos/build_fpk.py            # 产物 fnos/dist/nasoc-fnos-0.1.0.fpk（约 400MB）
# 版本号读 fnos/fpk_root/manifest；--image 可指定本机其他 tag
```

## 安装（fnOS）

- 应用中心 → 手动安装 → 选择 `.fpk`；或 SSH：
  `sudo appcenter-cli install-fpk /path/to/nasoc-fnos-0.1.0.fpk`
- 安装向导可填：项目根目录（逗号分隔多个）、数据目录（建议 SSD 数据卷）、
  访问口令 PIN。全部可留空用默认值。**安装器没有 API Key 字段**——
  装好后打开终端按 opencode 引导配置，凭据存数据卷（0600），不进包/日志：
  - 项目根默认 `/vol1/1000/nasoc-projects`（自动创建，每个目录一个终端 tab）
  - 数据默认 `<应用数据目录>/data`
- 桌面图标「NasOC 编程终端」→ 浏览器打开 `http://<NAS>:12703`。
- 应用启停 = 容器 `nasoc` 启停；**卸载/停用保留数据**（var 目录与项目目录不动），
  Docker 镜像也保留（彻底清除：卸载后 `docker rmi nasoc:<ver>` 并删 var 数据目录）。

## 结构

```
fnos/
├── build_fpk.py           # 打包脚本（tar.gz，无需 fnpack；含权限归一化）
├── payload/ui/config      # 桌面图标/iframe 入口（port 12703）
└── fpk_root/
    ├── manifest           # appname=nasoc, service_port=12703
    ├── cmd/               # main + service-setup（docker run 逻辑）+ 框架回调
    ├── config/            # privilege（run-as root；entrypoint 内降权 uid1000）
    └── wizard/install     # 安装向导字段
```

## 隐私保证

- 安装包（fpk）只含：离线镜像（由主仓库 `core/` 构建，上下文不含任何凭据）、
  桌面入口与脚本；**不含模型 key / PIN / 凭据库**。
- 访问口令经安装向导 → 容器 env → entrypoint 落盘 `/data/.term_pin`（0600）后
  从环境清除；登录走挑战-应答，明文口令不出浏览器、不进 URL/日志。
- `build_fpk.py` 打包阶段对全部源文件做权限归一化（0755/0644），避免
  OneDrive 占位符 0700 进镜像（NasOC UGOS 版 0012/0013 的教训，详见主仓库）。
- 如主仓库存在 `tools/verify_no_keys.py`，打包完成后自动做凭据泄漏校验。

## 升级

改 `fpk_root/manifest` 的 `version` → 主仓库 `docker build` 新镜像 → 重新
`build_fpk.py` → 应用中心升级。升级流程会 `docker load` 新镜像并在启动时重建容器；
数据目录与向导配置（var/*.conf）保留。

## 注意

- 端口固定 **12703**（manifest / payload/ui/config / cmd/service-setup 三处一致；
  改端口需要同步改这三处）。
- 镜像约 1.3GB、fpk 约 400MB——首次安装请确保系统分区余量充足，
  或把数据目录指到数据卷。
- 需要 fnOS 已启用 Docker 且版本 ≥ 20.10。
