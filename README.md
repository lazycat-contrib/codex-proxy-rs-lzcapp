# Codex Proxy RS (LazyCat)

面向 Codex 的自托管多账号 AI 网关，打包为 LazyCat LPK v2 应用。上游项目：[zyycn/codex-proxy-rs](https://github.com/zyycn/codex-proxy-rs)。

## 部署信息

- **包名**：`cloud.lazycat.app.codex-proxy-rs`
- **版本**：跟随上游 v3.x 镜像自动更新（GitHub Actions 每日检查）
- **架构**：amd64（`project.target_arch`）
- **min_os_version**：1.5.0

## 服务组成

| 服务 | 镜像 | 说明 |
|------|------|------|
| `postgres` | postgres 18-bookworm（静态） | 主数据库 |
| `redis` | redis 8-alpine（静态） | 缓存 / 任务队列 |
| `codex-proxy-rs` | ghcr.io/zyycn/codex-proxy-rs（自动更新） | 主应用（Web + Responses API） |

## 使用说明

1. 安装后打开应用，登录页自动填充管理员账号 `admin@cpr.local` 与自动生成的初始密码（也可在安装向导中自定义管理员密码）。
2. 在「账号」中添加上游账号（OpenAI / xAI 授权或 API Key 导入），按需建立分组。
3. 创建客户端密钥并选择可用分组，复制客户端配置到 Codex CLI / 桌面端。
4. API Key 持有者可在登录页切换为密钥登录，查看自己的用量与请求日志。

> 客户端接入 Base URL：`{应用地址}/v1`；仅支持 Responses API（不支持 `/v1/chat/completions`）。

## 关键实现

- **密码体系**：PostgreSQL / Redis 密码通过 `stable_secret + sha256sum + substr` 生成为 48 位十六进制（应用强制校验），管理员初始密码支持部署参数自定义或自动生成，并通过 `simple-inject-password` 免密登录。
- **配置文件**：`config.yaml` 由 `setup_script` 在容器启动时生成（含密码渲染），无需挂载文件。
- **文件选择器**：账号导入需上传 JSON 文件，已接入懒猫网盘文件选择器拦截（`lzc-file-chooser-inject.js`）。
- **在线更新**：容器内自更新已禁用（`CPR_UPDATE_REPOSITORY` 不配置），版本由 LPK 渠道统一管理。

## 发布

- **双商店**：官方懒猫商店（`LZC_API_TOKEN`）+ 喵喵私有商店（`APPSTORE_*`）。
- 工作流：`.github/workflows/lazycat.yml`（每日 UTC 06:31 自动检查上游镜像，PR 为 dry-run，支持手动触发）。
- 发布产物：GitHub Release 附带 `<package-id>-v<version>.lpk`。

## 文件结构

```
package.yml              # 包元数据
lzc-manifest.yml         # 运行结构（3 服务 + injects）
lzc-build.yml            # 构建配置
lzc-deploy-params.yml    # 安装向导参数
content/                 # 打包内容（文件选择器拦截脚本）
icon.png                 # 应用图标
.github/lazycat-action.yml      # Action 配置（镜像/商店）
.github/workflows/lazycat.yml   # 发布工作流
```
