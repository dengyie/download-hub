<!-- markdownlint-disable-file MD033 MD045 -->
<div align="center">

<img src="brand/logo.png" alt="DownloadHub Logo" width="100" height="100" />

# DownloadHub

**现代化全功能开源数字资源下载站、卡密安全核销网关与高性能资产分发平台**

[![License](https://img.shields.io/badge/license-MIT-green.svg?style=flat-square)](LICENSE)
[![Go Version](https://img.shields.io/badge/Go-1.26+-00ADD8.svg?style=flat-square&logo=go&logoColor=white)](https://golang.org)
[![Storage](https://img.shields.io/badge/Storage-OpenList_%7C_S3-blue.svg?style=flat-square)](https://github.com/OpenListTeam/OpenList)
[![Frontend](https://img.shields.io/badge/Frontend-Zero_Runtime_Deps-brightgreen.svg?style=flat-square)](frontend/)
[![Architecture](https://img.shields.io/badge/Architecture-Decoupled_SPA_%2B_Go_API-orange.svg?style=flat-square)](docs/frontend-architecture.md)
[![Tests](https://img.shields.io/badge/tests-passing-brightgreen.svg?style=flat-square)]()

[English](./README.md) · [简体中文](./README.zh-CN.md) · [核心特性](#核心特性) · [系统架构](#系统架构全景) · [快速上手](#快速上手) · [API 接口](#核心-api-端点) · [配置参数](#配置文件说明)

<br>

<p align="center">
  <b>彻底解决传统数字资源站带宽被大文件下行打满、盗链泛滥与卡密超额核销的顽疾。</b><br>
  结合零运行时依赖的原生 ES Modules 现代前端、SQLite IMMEDIATE 严格排他事务锁与动态临时签名存储分发网关。<br>
  提供两阶段原子核销、幂等凭据恢复、秒级 VACUUM 一致性热备 CLI，并将多吉字节文件流量完全卸载至存储 CDN。
</p>

</div>

---

## 痛点与架构对比

独立创作者、出海软件开发者与数字资产团队在发布软件安装包、固件或设计资产时，常面临两难：要么使用泛滥着弹窗广告且严重限速的第三方网盘，要么自建资源站面临服务器下行带宽被大文件下载完全占满、资源链接被恶意盗链爬取或卡密被并发重复兑换超扣：

| 评估维度 | DownloadHub 分发平台 | 传统网盘或下载站脚本 | 商业闭源发卡与商城系统 |
| :--- | :--- | :--- | :--- |
| **服务器带宽占用** | 零中转带宽消耗，仅流转轻量元数据与签名，大文件由 CDN 直发 | 全部流量经过业务服务器代理，带宽迅速打满并产生高昂账单 | 依赖服务器中转传输或强制使用其专用客户端 |
| **并发事务安全** | 基于 SQLite `IMMEDIATE` 事务锁与租约，杜绝高并发重复核销 | 普通读写易产生竞态条件，高并发扫卡容易超扣额度 | 复杂的分布式锁与外部 Redis 依赖，运维成本高 |
| **前端架构与体积**| 原生 Vanilla ES Modules，零 npm 依赖，深浅色自适应秒开 | 臃肿庞大的打包产物，加载缓慢，充斥冗余依赖 | 沉重且缺乏移动端适配的传统后台模板 |
| **源站存储安全性**| 动态签发受限短时有效直链，客户端永远接触不到真实存储凭据 | 直接暴露 OSS/S3 访问密钥或固定外链，极易遭恶意盗刷 | 固定下载链接，容易被直接分享传播 |
| **数据热备与管理**| 内置秒级 `VACUUM INTO` 零停机热快照，CLI 原生发卡与管理 | 必须停机拷贝数据库，否则快照容易出现数据损坏 | 依赖外部复杂的数据库备份编排工具 |

---

## 核心特性

| 模块 | 特性亮点 | 技术实现与工程优势 |
| :--- | :--- | :--- |
| **零依赖现代前端** | 原生 ES Modules 极致轻量 | 零 npm 运行时依赖，秒级加载；支持浅色、深色与跟随系统无闪烁切换，内置 Tabler 精美矢量图标。 |
| **沉浸式资源浏览** | 分类导航与实时防抖搜索 | 支持按目录筛选、关键词防抖实时过滤；集成基于白名单的富文本描述安全重建机制，杜绝 XSS 注入风险。 |
| **两阶段原子核销** | 严谨的事务扣减与凭证签发 | 第一阶段验卡发放内存短效凭证（不扣额度）；第二阶段用户明确点击下载才进入事务扣除并签发存储签名。 |
| **幂等故障自愈恢复** | 异常重试绝不二次扣减 | 遇到网络抖动或页面刷新时，可凭原凭据安全实现幂等恢复签发，在有效期内绝不重复扣减卡密次数。 |
| **灵活卡密授权模型**| 精确文件与路径前缀授权 | 支持按具体单一资源文件绑定（`--resource`），或按路径前缀批量授权（`--prefix`）；支持限制次数或永久有效。 |
| **存储 CDN 流量卸载**| 源站绝对零暴露与高速分发 | 对接 OpenList 或标准 S3 对象存储；后端仅请求临时签名，大文件下载完全走存储 CDN，零耗费服务器带宽。 |
| **防刷频与安全防御**| 进程内自适应滑动窗口限流 | 内置针对单 IP 的固定时间窗口限流器，有效抵御自动化恶意扫卡；配合白名单穿透验证阻断非法路径访问。 |
| **零停机热备与 CLI** | 原生管理工具链与秒级备份 | 随二进制附带命令行工具，支持发卡、查询、吊销，并利用 SQLite `VACUUM INTO` 导出全局一致性快照。 |

---

## 系统架构全景

```text
       ┌────────────────────────┐
       │     访客 / 浏览器客户端 │
       └───────────┬────────────┘
                   │ HTTPS 请求
                   ▼
       ┌────────────────────────┐
       │      Nginx 网关层      │ ◄─── 静态分发展台 (原生 ES Modules 单页应用)
       └───────────┬────────────┘
                   ├─────────────────────────────────────────┐
                   │ /api/* (轻量 JSON API)                  │ /p/* (临时签名大文件直链)
                   ▼                                         ▼
   ┌────────────────────────────────┐        ┌────────────────────────────────┐
   │       DownloadHub 服务引擎     │        │    OpenList / S3 对象存储 CDN  │
   │       (Go 原生程序 :8080)      │───────►│    (负责承载大文件流式传输)    │
   └───────────────┬────────────────┘ 签发短时 │
                   │                  签名凭据 └────────────────────────────────┘
                   │ 严格 IMMEDIATE                          ▲
                   │ 原子事务扣减                            │ 校验签名有效性
                   ▼                                         │
   ┌────────────────────────────────┐                        │
   │       cards.db (SQLite 存储)   │────────────────────────┘
   │   (卡密配额 · 授权凭据 · 审计) │
   └────────────────────────────────┘
```

---

## 快速上手

### 1. 编译构建

环境要求：**Go 1.26+** 以及 **Node.js 22+**（仅用于前端静态资源打包）：

```bash
# 克隆代码仓库
git clone https://github.com/dengyie/DownloadHub.git
cd DownloadHub

# 编译后端单一二进制可执行程序
make build
# 产物输出于当前目录 ./downloadhub

# 构建前端静态产物 (产物输出至 frontend/dist)
npm --prefix frontend ci --ignore-scripts
npm --prefix frontend run build
```

如需部署至 Linux 服务器，可直接交叉编译：

```bash
make build-linux
# 产物输出于 ./downloadhub-linux-amd64
```

### 2. 配置文件说明

复制配置模板：

```bash
cp config.yaml.example config.yaml
```

编辑 `config.yaml` 关键配置项（`openlist_admin_pass` 与 `openlist_min_url_ttl_seconds` 为必填项，后者设为 `0` 时服务将拒绝启动）：

```yaml
listen: "127.0.0.1:8080"
db_path: "./cards.db"
openlist_url: "http://127.0.0.1:5244"
openlist_admin_user: "admin"
openlist_admin_pass: "your_password"   # 必改：OpenList 管理员密码

# 兑换后在途凭据有效期（秒）
grant_window_seconds: 600

# 必改：存储签名外链的最短剩余有效期（秒，防止发出即将失效的外链）
# 设为 0 时服务拒绝启动；需 >= grant_window_seconds 且 <= 86400
openlist_min_url_ttl_seconds: 3600

# 允许签发的外链域名白名单
download_hosts: ["download.example.com"]

# 信任代理 IP 网段（仅允许本机 Nginx 回环传导真实客户 IP）
trusted_proxy_cidrs: ["127.0.0.1/32"]
```

### 3. 卡密生成与管理 CLI

```bash
# 生成新卡密：包含 3 次下载额度，绑定 private/release-v1.zip，30 天有效期
./downloadhub gencard --quota 3 --resource private/release-v1.zip --days 30 --db cards.db

# 生成永久多资源卡密（--days 0 代表永久有效，支持逗号分隔多个资源路径）
./downloadhub gencard --quota 10 --resource private/doc.pdf,private/setup.exe --days 0 --db cards.db

# 查看所有已发卡状态与额度
./downloadhub listcards --db cards.db

# 吊销指定卡密
./downloadhub revokecard --card <CARD_ID> --db cards.db

# 生产环境零停机热备份（VACUUM INTO 一致性热快照）
./downloadhub backup --db cards.db --out cards-snapshot.db
```

### 4. 运行服务与部署

```bash
./downloadhub serve --config config.yaml
```

生产环境结合 Nginx 托管前端与反代 API，完整的配置示例请参考 [deploy/download-nginx.conf.example](deploy/download-nginx.conf.example)。

---

## 核心 API 端点

| 方法 | 端点 | 功能说明 | 是否扣减配额 |
| :--- | :--- | :--- | :--- |
| `GET` | `/`、`/redeem`、`/orders` | 单页静态前端文档路由 | 否 |
| `GET` | `/api/catalog` | 公开可下载资源列表白名单 | 否 |
| `POST` | `/api/redeem` | 校验卡密密钥，获取授权凭证与资源列表 | 否 |
| `POST` | `/api/grants` | 携带凭据发起最终签发，获取带签名下载 URL | 是（原子扣除 1 次） |
| `GET` | `/health` | 服务存活健康检测接口 | 否 |
| `GET` | `/ready` | 数据库及核心组件就绪探针 | 否 |

---

## 自定义站点品牌

前端品牌字样集中维护在 [frontend/src/site.js](frontend/src/site.js)（站点名、导航字标、首页眉标），配合 `frontend/index.html` 的静态默认值与 `brand/` 目录下的 Logo 图标资源即可实现一键品牌更换，无需侵入修改任何底层业务代码。

---

## 自动化测试套件

全站配有覆盖严格边界与高并发竞态的测试套件：

```bash
# 运行完整质量门禁（需 Go / Node 22+ / Python 3 / PHP 8.2+ 扩展 / Playwright）
make check

# 单独运行 Go 后端单元测试（带 -race 竞态检测）
go test -race -count=1 ./...

# 单独运行前端测试、格式检查与构建
make check-frontend
```

---

## 参与贡献与许可证

欢迎提交 Issue 与 Pull Request！在提交代码变更前，请确保全量测试套件执行通过：

```bash
go test -v ./...
```

- 贡献指南：[CONTRIBUTING.md](CONTRIBUTING.md)
- 安全政策：[SECURITY.md](SECURITY.md)
- 本项目采用 [MIT License](LICENSE) 开源协议。
