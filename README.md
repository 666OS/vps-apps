# VPS-APPS

Docker Compose 应用模板库，供 [`vps-sh`](https://github.com/666OS/vps-sh) 检索与部署。

---

## AI 协同 (AI-Native)

* **应用语义端点 (`llms.txt`)**：根目录提供符合标准的 `llms.txt`，收录各模板的应用 ID、默认端口、初始凭据与一键安装指令，供大模型快速检索与调用。
* **结构化索引 (`index.json`)**：静态应用市场 Feed，由构建流水线自动提取各模板的 `x-vps:` 扩展块元数据生成。

---

## 目录结构

```text
apps/
  └── <app-id>/
        └── compose.yaml   # 编排文件
index.json                 # 应用索引 (JSON 数据源)
llms.txt                   # 应用语义索引 (供 LLM 检索)
```

### 元数据与端口规范
`compose.yaml` 使用 Compose 规范扩展块 `x-vps:` 声明应用元数据，端口统一使用 `APP_HOST` 与 `APP_PORT` 变量：
```yaml
x-vps:
  name: 应用名称
  tag: 类别标签
  desc: 功能说明
  url: https://example.com           # 可选。项目官网、云控制台或使用文档链接
  port: 8080                         # 可选。无入站端口设为 "-"；单端口自动从 ports 推导
  auth: admin@example.com / changeme # 可选。声明应用初始账密或配置引导

services:
  app:
    image: example/app:latest
    ports:
      - "${APP_HOST:-0.0.0.0}:${APP_PORT:-8080}:80"
    volumes:
      - ./data:/data
```

* **`url`**：参考或控制台链接。在终端中支持原生超链接（`Cmd+点击` 直接跳转控制台或官方文档），特别适用于无本地 Web 界面的云端协同类服务（如 Cloudflared）。
* **`auth`**：初始凭据说明。对于预置固定账号密码的应用（如 NPM），声明后将在安装前确认卡、部署完成回显与应用详情页全程展示；若模板引用 `${APP_PASSWORD}`，安装脚本将自动生成强密码并持久化写入 `.env`。
* **`APP_HOST`**：宿主机监听 IP。默认为 `0.0.0.0`（公网访问并自动放行防火墙），支持切换为 `127.0.0.1`（本地反代模式，不暴露公网）。
* **`APP_PORT`**：宿主机映射端口。未在 `x-vps:` 中指定时，编译器与客户端将自动从 `ports:` 嗅探推导。

---

## 本地存储规范

应用部署于目标系统的物理路径：
```text
/opt/vps-sh/apps/<app-id>/
├── compose.yaml       # 编排文件
├── .env               # 环境变量 (APP_HOST, APP_PORT)
└── data/              # 挂载数据目录
```

---

## 源配置

`vps.sh` 默认读取本仓库。可通过以下方式指定第三方或自建源：

```bash
# 环境变量临时指定
export VPS_APP_FEED="https://raw.githubusercontent.com/<owner>/<repo>/main"

# 配置文件持久指定
mkdir -p /etc/vps-sh && echo "https://raw.githubusercontent.com/<owner>/<repo>/main" > /etc/vps-sh/feed
```
