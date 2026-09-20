# Octop 源代码说明（SOURCE.md）

> 统计范围：仓库 `/workspace` 内业务源码。排除 `node_modules`、`.git`、构建产物 `src/octop/dashboard/`、`uv.lock`。  
> 生成方式：对 Python 做 AST 扫描，对 `dashboard/src` 统计 TypeScript 行数。行数为约数（按换行计数）。  
> 版本：Octop 1.0.1。配套：`1DESIGN.md`、`2HELP.md`、`3SOLUTION.md`。

---

## 1. 源代码整体介绍

### 1.1 项目是什么

Octop 是用 **Python 3.12+** 编写的自托管 AI 助手平台，附带 **React 18 + TypeScript** 控制台。后端包根在 `src/octop/`，前端源码在 `dashboard/`，测试在 `tests/`。运行时还依赖 PyPI 上的 `orcakit-harness-agent`、`harness-gateway`、`harness-memory`、`harness-browser`；本仓库克隆中**不包含**这两个 harness 的源码树（AGENTS.md 里的 `/workspace/harness-agent` 路径在本环境不存在）。

### 1.2 使用的语言

| 语言 | 用途 | 主要目录 |
|------|------|----------|
| Python 3.12+ | 后端、CLI、测试、部分桌面辅助 | `src/octop/`、`tests/`、`desktop/` |
| TypeScript / TSX | 控制台 SPA | `dashboard/src/` |
| SQL | 控制平面迁移（SQLite 与 PostgreSQL 成对） | `src/octop/infra/db/migrations/` |
| JSON | i18n、专家 manifest、配置样例 | `src/octop/i18n/`、`dashboard/src/locales/` |
| Markdown | 专家技能、文档、本说明 | `docs/`、`experts/library/` |
| YAML | 命令护栏、插件描述、CI | 多处 |
| Shell / PowerShell / Bat | 安装脚本 | 发行脚本与 `scripts/` |

### 1.3 开发工具与质量工具

- **包管理与构建**：uv、hatchling、PyPI wheel；前端 Vite 6、TypeScript 5.8。
- **Web**：FastAPI、uvicorn、Pydantic 2、Scalar OpenAPI UI。
- **CLI**：Click、Rich、questionary。
- **质量**：Ruff（lint+format）、mypy `--strict`（排除 dashboard 与专家库）、pytest、pytest-asyncio、pytest-cov、pytest-xdist、pytest-testmon、httpx。
- **前端质量**：ESLint 9、Prettier 3、Vitest、Testing Library。
- **CI**：GitHub Actions，Linux 与 Windows 矩阵；另有 CodeQL、Release、Docker、桌面包、FNOS FPK。
- **Git 钩子**：`make install-hooks` 将 `core.hooksPath` 设为 `.githooks`，提交前跑 `make all` 与 dashboard build。
- **IDE/Agent 导航**：根目录 `AGENTS.md`。

### 1.4 Makefile 常用目标

`make all` 是交付门槛（format-all + lint + typecheck + test）。此外：`dev`/`dev-frontend`/`dev-backend`、`build-frontend`、`build-wheel`、`format`/`lint`/`typecheck` 及其 frontend 变体、`install-hooks`、`docs-cli`、`publish`。测试必须 `uv run pytest`，禁止裸 pytest。

### 1.5 统计总表

| 类别 | 文件数 | 约行数 |
|------|-------:|------:|
| `src/octop` Python（不含内置 SPA） | 583 | 123942 |
| `tests` Python | 431 | 72179 |
| `dashboard/src` `.ts`/`.tsx` | 833 | 158773 |
| **以上合计** | **1847** | **354894** |
| 源码类 | 530 | — |
| 模块级函数 | 3674 | — |
| 方法 | 1453 | — |

另有专家库大量 Markdown（技能说明书），不逐行计入上表，以免把文档当成可执行代码。插件示例在仓库根 `plugins/`。SQL 迁移约 30 个文件（含 `.pg.sql`）。

### 1.6 目录地图

```
src/octop/          Python 包根（config、launch、api、cli、infra、i18n）
dashboard/          控制台源码（Vite）
tests/              unit / integration / live / support
docs/               人类文档与 ADR
plugins/            示例插件
desktop/            桌面辅助
docker/             容器
scripts/            维护脚本
.github/workflows/  CI
```

### 1.7 入口符号

- CLI：`octop = octop.cli.main:cli`（pyproject scripts）
- HTTP 组合根：`octop.launch` → `OctopServer` + `build_app`
- 前端：`dashboard/src` 路由表

### 1.8 阅读源代码的建议顺序

先读 `AGENTS.md` 与 `docs/architecture.md`，再读 `config.py`、`infra/server.py`、`launch.py`、`api/app.py`，然后按领域进入 `infra/agents/manager.py`、`infra/gateway/process/processor.py`、`infra/db/migrate.py`。改 UI 只动 `dashboard/`。改表结构必须成对 SQL + 测试版本断言。

---

## 2. 源代码文件清单

每一条包含：文件名、所在目录、行数、主要功能简介（中文）。简介结合路径语义与模块 docstring。


## 2.1 Python 包 `src/octop`


### 目录 `src/octop/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/`
- **相对路径**：`src/octop/__init__.py`
- **约行数**：6
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/__init__.py` 属于Octop 源码，约 6 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Octop — smarter self-hosted AI assistant (multi-user, multi-agent).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `__main__.py`

- **所在目录**：`src/octop/`
- **相对路径**：`src/octop/__main__.py`
- **约行数**：9
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/__main__.py` 属于Octop 源码，约 9 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Entry point: ``python -m octop`` → CLI.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `config.py`

- **所在目录**：`src/octop/`
- **相对路径**：`src/octop/config.py`
- **约行数**：627
- **类 / 模块级函数**：6 / 16
- **主要功能简介**：`src/octop/config.py` 属于进程级配置（config.json 与环境变量覆盖），约 627 行，包含 6 个类与 16 个模块级函数。模块文档写道：「Process-level configuration (config.json + env overrides).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DatabaseConfig`、`TlsConfig`、`BackupConfig`、`MobileCapabilities`、`CapabilitiesConfig`、`OctopConfig`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `launch.py`

- **所在目录**：`src/octop/`
- **相对路径**：`src/octop/launch.py`
- **约行数**：157
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/launch.py` 属于组合根：启动 OctopServer、挂载 FastAPI、uvicorn 前台运行，约 157 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Composition root — wire OctopServer, FastAPI, and uvicorn for ``octop run``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/api/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/api/`
- **相对路径**：`src/octop/api/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/api/__init__.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `app.py`

- **所在目录**：`src/octop/api/`
- **相对路径**：`src/octop/api/app.py`
- **约行数**：313
- **类 / 模块级函数**：1 / 7
- **主要功能简介**：`src/octop/api/app.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 313 行，包含 1 个类与 7 个模块级函数。模块文档写道：「FastAPI app factory.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_RouterMount`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `deps.py`

- **所在目录**：`src/octop/api/`
- **相对路径**：`src/octop/api/deps.py`
- **约行数**：231
- **类 / 模块级函数**：2 / 14
- **主要功能简介**：`src/octop/api/deps.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 231 行，包含 2 个类与 14 个模块级函数。模块文档写道：「FastAPI Depends, JWT helpers, and auth routing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`InvalidToken`、`TokenExpired`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `openapi_meta.py`

- **所在目录**：`src/octop/api/`
- **相对路径**：`src/octop/api/openapi_meta.py`
- **约行数**：200
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/openapi_meta.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 200 行，包含 0 个类与 1 个模块级函数。模块文档写道：「OpenAPI metadata for Scalar API docs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/api/common/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/api/common/__init__.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Shared router helpers — agent access, workspace glue, request validation.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `agent.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/agent.py`
- **约行数**：80
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/api/common/agent.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 80 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Shared agent ownership / existence checks for HTTP routers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `agent_runtime.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/agent_runtime.py`
- **约行数**：33
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/api/common/agent_runtime.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 33 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Typed API fields for per-agent runtime and model settings.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AgentRuntimeFields`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `agent_workspace.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/agent_workspace.py`
- **约行数**：20
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/common/agent_workspace.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 20 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Resolve agent ``workspace_dir`` for API handlers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `attachments.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/attachments.py`
- **约行数**：70
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`src/octop/api/common/attachments.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 70 行，包含 1 个类与 3 个模块级函数。模块文档写道：「Chat attachment HTTP helpers — thin wrappers over :mod:`inbound_store`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`StoredAttachment`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `content_disposition.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/content_disposition.py`
- **约行数**：36
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/api/common/content_disposition.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 36 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Build latin-1-safe Content-Disposition values (RFC 5987).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `memory_client.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/memory_client.py`
- **约行数**：192
- **类 / 模块级函数**：1 / 6
- **主要功能简介**：`src/octop/api/common/memory_client.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 192 行，包含 1 个类与 6 个模块级函数。模块文档写道：「Per-agent ``Memory`` instance management for the dashboard router.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_MemoryCache`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `public_base.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/public_base.py`
- **约行数**：30
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/api/common/public_base.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 30 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Resolve the externally visible HTTP origin for an incoming request.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `sso_cookie.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/sso_cookie.py`
- **约行数**：38
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/api/common/sso_cookie.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 38 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Shared SSO state cookie for OIDC and OAuth login callbacks.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `upload_limit.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/upload_limit.py`
- **约行数**：55
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`src/octop/api/common/upload_limit.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 55 行，包含 1 个类与 3 个模块级函数。模块文档写道：「Cap multipart uploads before the whole body is assembled in memory.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AsyncByteReader`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `validators.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/validators.py`
- **约行数**：88
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/api/common/validators.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 88 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Cross-router request validation helpers (no FastAPI route definitions).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `workspace.py`

- **所在目录**：`src/octop/api/common/`
- **相对路径**：`src/octop/api/common/workspace.py`
- **约行数**：134
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/api/common/workspace.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 134 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Shared workspace helpers (not route handlers).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/api/middleware/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/api/middleware/`
- **相对路径**：`src/octop/api/middleware/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/api/middleware/__init__.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「ASGI middlewares for octop.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `jwt_auth.py`

- **所在目录**：`src/octop/api/middleware/`
- **相对路径**：`src/octop/api/middleware/jwt_auth.py`
- **约行数**：73
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/middleware/jwt_auth.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 73 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Require JWT for all /api/* routes except an explicit allowlist.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `setup_lockdown.py`

- **所在目录**：`src/octop/api/middleware/`
- **相对路径**：`src/octop/api/middleware/setup_lockdown.py`
- **约行数**：42
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/middleware/setup_lockdown.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 42 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Lock down all non-/setup endpoints while no users exist.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/api/routers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/api/routers/__init__.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `acp.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/acp.py`
- **约行数**：304
- **类 / 模块级函数**：4 / 19
- **主要功能简介**：`src/octop/api/routers/acp.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 304 行，包含 4 个类与 19 个模块级函数。模块文档写道：「ACP runner configuration API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ACPRunnerBody`、`ACPRunnersBody`、`ACPConfigBody`、`ACPAgentToolBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `admin.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/admin.py`
- **约行数**：82
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/api/routers/admin.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 82 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Admin overview/audit/metrics router.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `agent_files.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/agent_files.py`
- **约行数**：227
- **类 / 模块级函数**：1 / 8
- **主要功能简介**：`src/octop/api/routers/agent_files.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 227 行，包含 1 个类与 8 个模块级函数。模块文档写道：「Per-agent Memory + Heartbeat endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`HeartbeatConfigBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `agent_tools.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/agent_tools.py`
- **约行数**：189
- **类 / 模块级函数**：4 / 3
- **主要功能简介**：`src/octop/api/routers/agent_tools.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 189 行，包含 4 个类与 3 个模块级函数。模块文档写道：「Per-agent tool settings — built-in denylist + plugin tool enable flags.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ToolSettingsItem`、`ToolSettingsResponse`、`ToolSettingsPutBody`、`ToolSettingPatchBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `agents.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/agents.py`
- **约行数**：553
- **类 / 模块级函数**：2 / 18
- **主要功能简介**：`src/octop/api/routers/agents.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 553 行，包含 2 个类与 18 个模块级函数。模块文档写道：「Agents router.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AgentCreateBody`、`AgentPatchBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `auth.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/auth.py`
- **约行数**：173
- **类 / 模块级函数**：4 / 9
- **主要功能简介**：`src/octop/api/routers/auth.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 173 行，包含 4 个类与 9 个模块级函数。模块文档写道：「Login / logout / me / change-password.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`LoginBody`、`CaptchaPublicResponse`、`ChangePasswordBody`、`UpdateMeBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `auth_oauth.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/auth_oauth.py`
- **约行数**：234
- **类 / 模块级函数**：4 / 13
- **主要功能简介**：`src/octop/api/routers/auth_oauth.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 234 行，包含 4 个类与 13 个模块级函数。模块文档写道：「Pluggable OAuth SSO HTTP routes (Feishu and future providers).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`OauthStartBody`、`OauthExchangeBody`、`OauthProviderPutBody`、`OauthUnbindBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `auth_oidc.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/auth_oidc.py`
- **约行数**：183
- **类 / 模块级函数**：3 / 12
- **主要功能简介**：`src/octop/api/routers/auth_oidc.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 183 行，包含 3 个类与 12 个模块级函数。模块文档写道：「OpenID Connect SSO HTTP routes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`OidcStartBody`、`OidcExchangeBody`、`OidcConfigBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `backup.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/backup.py`
- **约行数**：433
- **类 / 模块级函数**：2 / 19
- **主要功能简介**：`src/octop/api/routers/backup.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 433 行，包含 2 个类与 19 个模块级函数。模块文档写道：「Admin backup / restore API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AutoBackupSettingsBody`、`CreateBackupBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `channels.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/channels.py`
- **约行数**：1072
- **类 / 模块级函数**：7 / 39
- **主要功能简介**：`src/octop/api/routers/channels.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 1072 行，包含 7 个类与 39 个模块级函数。模块文档写道：「Channels router.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ChannelCreateBody`、`ChannelPatchBody`、`ChannelProbeBody`、`FeishuBotCreatorStartBody`、`YuanbaoBotCreatorStartBody`、`DingTalkRegistrationPollBody`、`_DingTalkRegistrationSession`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `connectors.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/connectors.py`
- **约行数**：1402
- **类 / 模块级函数**：12 / 43
- **主要功能简介**：`src/octop/api/routers/connectors.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 1402 行，包含 12 个类与 43 个模块级函数。模块文档写道：「Connector and agent-binding HTTP API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CreateInstanceBody`、`PatchInstanceBody`、`OAuthStartBody`、`OAuthStartLegacyBody`、`ExchangeAuthCodeBody`、`TestCredentialsBody`、`FeishuUserAuthStartBody`、`FeishuUserAuthCompleteBody`、`CustomMcpPutBody`、`CustomMcpServerPatchBody`、`CustomMcpTestBody`、`FeishuUserAuthInstanceCompleteBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `cron.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/cron.py`
- **约行数**：261
- **类 / 模块级函数**：3 / 11
- **主要功能简介**：`src/octop/api/routers/cron.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 261 行，包含 3 个类与 11 个模块级函数。模块文档写道：「Cron router.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CronCreateBody`、`CronPatchBody`、`CronExamplesResponse`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `envs.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/envs.py`
- **约行数**：135
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/api/routers/envs.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 135 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Environment variables API — backed by ``~/.octop/env`` (inherited by all agents).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `experts.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/experts.py`
- **约行数**：761
- **类 / 模块级函数**：10 / 24
- **主要功能简介**：`src/octop/api/routers/experts.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 761 行，包含 10 个类与 24 个模块级函数。模块文档写道：「Experts router — bundled scene templates + SkillHub expert market.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`FromExpertBody`、`PublishExpertBody`、`RefreshPublishedExpertBody`、`InstallPublishedExpertBody`、`LocalizedTextResponse`、`QuickPromptResponse`、`ExpertHubItemResponse`、`ExpertHubListResponse`、`MarketCreateSourceResponse`、`MarketCreateResponse`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `filesystem.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/filesystem.py`
- **约行数**：227
- **类 / 模块级函数**：3 / 9
- **主要功能简介**：`src/octop/api/routers/filesystem.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 227 行，包含 3 个类与 9 个模块级函数。模块文档写道：「Host filesystem browsing for dashboard forms (root_dir pickers).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ProbeBody`、`MkdirBody`、`RenameBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `health.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/health.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/routers/health.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 29 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Liveness probe (no auth).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `i18n.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/i18n.py`
- **约行数**：48
- **类 / 模块级函数**：2 / 2
- **主要功能简介**：`src/octop/api/routers/i18n.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 48 行，包含 2 个类与 2 个模块级函数。模块文档写道：「Localized strings API (tool labels, etc.).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ToolLabelsOut`、`SkillLabelsOut`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `internal_mcp.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/internal_mcp.py`
- **约行数**：148
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/api/routers/internal_mcp.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 148 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Internal HTTP MCP endpoints for Octop-hosted connector gateways.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `invites.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/invites.py`
- **约行数**：159
- **类 / 模块级函数**：3 / 9
- **主要功能简介**：`src/octop/api/routers/invites.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 159 行，包含 3 个类与 9 个模块级函数。模块文档写道：「Admin invite CRUD and public invite validate/redeem.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`InviteCreateBody`、`InviteValidateBody`、`InviteRedeemBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `knowledge_bases.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/knowledge_bases.py`
- **约行数**：962
- **类 / 模块级函数**：8 / 38
- **主要功能简介**：`src/octop/api/routers/knowledge_bases.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 962 行，包含 8 个类与 38 个模块级函数。模块文档写道：「HTTP API for private, shareable knowledge bases.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`FeatureBody`、`OnnxDownloadBody`、`CreateBaseBody`、`CreateFolderBody`、`CreateTextDocumentBody`、`UpdateTextDocumentBody`、`UpdateBaseBody`、`RenameDocumentBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `mbti.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/mbti.py`
- **约行数**：767
- **类 / 模块级函数**：9 / 12
- **主要功能简介**：`src/octop/api/routers/mbti.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 767 行，包含 9 个类与 12 个模块级函数。模块文档写道：「MBTI personality type API — profiles, test, and per-agent persona apply.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DimensionsResponse`、`BehaviorResponse`、`MBTITypeResponse`、`CurrentMBTIResponse`、`TestQuestion`、`TestSubmitRequest`、`TestResultResponse`、`ApplyRequest`、`ApplyResponse`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `media_generation.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/media_generation.py`
- **约行数**：146
- **类 / 模块级函数**：4 / 4
- **主要功能简介**：`src/octop/api/routers/media_generation.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 146 行，包含 4 个类与 4 个模块级函数。模块文档写道：「Admin API for instance-wide image and video generation settings.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`MediaGenerationSettingsResponse`、`MediaGenerationSettingsBody`、`MediaGenerationTestBody`、`MediaGenerationTestResponse`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `memory.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/memory.py`
- **约行数**：872
- **类 / 模块级函数**：11 / 27
- **主要功能简介**：`src/octop/api/routers/memory.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 872 行，包含 11 个类与 27 个模块级函数。模块文档写道：「Per-agent memory dashboard router.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_ListAtomsBody`、`_ListRawEventsBody`、`_ListEntitiesBody`、`_ListEpisodesBody`、`_ListJournalBody`、`_ListCandidatesBody`、`_ExtractConfigBody`、`_RejectCandidateBody`、`_DeprecateAtomBody`、`_CreateAtomBody`、`_ReplaceAtomBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `memory_portable.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/memory_portable.py`
- **约行数**：304
- **类 / 模块级函数**：1 / 6
- **主要功能简介**：`src/octop/api/routers/memory_portable.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 304 行，包含 1 个类与 6 个模块级函数。模块文档写道：「Cross-host memory migration service REST endpoint (requirements 1, 2, 3, 5, 8).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_AdoptRequest`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `observability.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/observability.py`
- **约行数**：88
- **类 / 模块级函数**：3 / 4
- **主要功能简介**：`src/octop/api/routers/observability.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 88 行，包含 3 个类与 4 个模块级函数。模块文档写道：「Observability configuration API (admin).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`LangfuseConfigResponse`、`LangfuseConfigBody`、`LangfuseTestBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `ollama_download_store.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/ollama_download_store.py`
- **约行数**：131
- **类 / 模块级函数**：2 / 6
- **主要功能简介**：`src/octop/api/routers/ollama_download_store.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 131 行，包含 2 个类与 6 个模块级函数。模块文档写道：「In-memory store for tracking background model download tasks.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DownloadTaskStatus`、`DownloadTask`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `ollama_models.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/ollama_models.py`
- **约行数**：306
- **类 / 模块级函数**：5 / 11
- **主要功能简介**：`src/octop/api/routers/ollama_models.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 306 行，包含 5 个类与 11 个模块级函数。模块文档写道：「API endpoints for Ollama model management.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`OllamaDownloadRequest`、`OllamaModelResponse`、`OllamaDownloadTaskResponse`、`OllamaServiceBody`、`OllamaServiceStatus`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `onnx_models.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/onnx_models.py`
- **约行数**：220
- **类 / 模块级函数**：5 / 12
- **主要功能简介**：`src/octop/api/routers/onnx_models.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 220 行，包含 5 个类与 12 个模块级函数。模块文档写道：「Local ONNX embedding model cache API (Models admin — local tab).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`OnnxCatalogItem`、`OnnxServiceConfigBody`、`OnnxDownloadRequest`、`OnnxTestRequest`、`OnnxTestResponse`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `plugins.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/plugins.py`
- **约行数**：431
- **类 / 模块级函数**：9 / 14
- **主要功能简介**：`src/octop/api/routers/plugins.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 431 行，包含 9 个类与 14 个模块级函数。模块文档写道：「Plugin install and agent tool configuration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PluginInstallBody`、`AgentPluginToolBody`、`AgentPluginToolsPatch`、`PluginToolMeta`、`AgentPluginItem`、`AgentPluginsResponse`、`AgentPluginEnabledPatch`、`AgentPluginsPatch`、`PluginPatchBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `preferences.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/preferences.py`
- **约行数**：167
- **类 / 模块级函数**：4 / 4
- **主要功能简介**：`src/octop/api/routers/preferences.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 167 行，包含 4 个类与 4 个模块级函数。模块文档写道：「Per-user preferences (locale, etc.).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RemoteBrowserBookmarkModel`、`ModelReasoningPreferenceModel`、`PreferencesResponse`、`PatchPreferencesBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `proactive_care.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/proactive_care.py`
- **约行数**：146
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`src/octop/api/routers/proactive_care.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 146 行，包含 1 个类与 4 个模块级函数。模块文档写道：「Proactive care push configuration API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ProactiveCareConfigBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `providers.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/providers.py`
- **约行数**：486
- **类 / 模块级函数**：6 / 20
- **主要功能简介**：`src/octop/api/routers/providers.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 486 行，包含 6 个类与 20 个模块级函数。模块文档写道：「Providers router (admin-only write operations).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ProviderCreateBody`、`ProviderPatchBody`、`ProviderTestBody`、`ProviderTestDraftBody`、`ProviderFetchModelsBody`、`ActiveModelBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `search.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/search.py`
- **约行数**：53
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/api/routers/search.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 53 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Search-provider connectivity API (Settings → Advanced → Search).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TestSearchRequest`、`TestSearchResponse`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `security.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/security.py`
- **约行数**：205
- **类 / 模块级函数**：9 / 9
- **主要功能简介**：`src/octop/api/routers/security.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 205 行，包含 9 个类与 9 个模块级函数。模块文档写道：「Security policy configuration API (admin).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SecurityPolicyBody`、`SecurityPolicyResponse`、`ToolGuardRuleItem`、`ToolGuardRulesResponse`、`ToolGuardRulesRawResponse`、`ToolGuardRulesRawBody`、`ToolGuardRulesSaveResponse`、`HitlToolCatalogItem`、`SecurityDefaultsResponse`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `settings.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/settings.py`
- **约行数**：154
- **类 / 模块级函数**：8 / 5
- **主要功能简介**：`src/octop/api/routers/settings.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 154 行，包含 8 个类与 5 个模块级函数。模块文档写道：「Process-level settings exposed to authenticated clients.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TimezoneSettingsResponse`、`UploadSettingsResponse`、`MobileCapabilitiesResponse`、`CapabilitiesResponse`、`CaptchaPairView`、`CaptchaSettingsResponse`、`CaptchaPairBody`、`CaptchaSettingsPut`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `setup.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/setup.py`
- **约行数**：444
- **类 / 模块级函数**：7 / 20
- **主要功能简介**：`src/octop/api/routers/setup.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 444 行，包含 7 个类与 20 个模块级函数。模块文档写道：「Initial-admin setup wizard.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`VerifyBody`、`SetupBody`、`ProviderModelDraft`、`ProviderDraftBody`、`FinishBody`、`ProviderTestBody`、`DatabaseSetupBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `skill_packages.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/skill_packages.py`
- **约行数**：773
- **类 / 模块级函数**：10 / 29
- **主要功能简介**：`src/octop/api/routers/skill_packages.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 773 行，包含 10 个类与 29 个模块级函数。模块文档写道：「HTTP API for instance-global skill packages.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CreateSkillPackageBody`、`UpdateSkillPackageBody`、`FromSkillHubBody`、`SkillFilePart`、`CreatePackageSkillBody`、`UpdatePackageSkillBody`、`ImportPackageSkillBody`、`LocalizedSkillCopy`、`HubInstallPackageSkillBody`、`_PackageInstallTarget`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `skills.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/skills.py`
- **约行数**：1418
- **类 / 模块级函数**：11 / 43
- **主要功能简介**：`src/octop/api/routers/skills.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 1418 行，包含 11 个类与 43 个模块级函数。模块文档写道：「Skills router — per-agent ``SKILL.md`` library.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_AgentCtx`、`SkillPackageMountBody`、`CopyPackageSkillsBody`、`PushSkillToPackageBody`、`SkillFilePart`、`CreateSkillBody`、`UpdateSkillBody`、`ImportSkillBody`、`_AgentWorkspaceInstallTarget`、`LocalizedSkillCopy`、`HubInstallBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `slash.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/slash.py`
- **约行数**：75
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/api/routers/slash.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 75 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Slash command discovery API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SlashCommandOut`、`SlashCommandListOut`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `storage_backends.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/storage_backends.py`
- **约行数**：240
- **类 / 模块级函数**：3 / 10
- **主要功能简介**：`src/octop/api/routers/storage_backends.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 240 行，包含 3 个类与 10 个模块级函数。模块文档写道：「Storage backends router — admin-only.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`StorageBackendCreateBody`、`StorageBackendPatchBody`、`StorageBackendProbeBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `subagents.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/subagents.py`
- **约行数**：245
- **类 / 模块级函数**：1 / 9
- **主要功能简介**：`src/octop/api/routers/subagents.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 245 行，包含 1 个类与 9 个模块级函数。模块文档写道：「Subagents router — per-agent ``agents/**/*.md`` definitions and bundled catalog.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`InstallSubagentBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `terminal.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/terminal.py`
- **约行数**：788
- **类 / 模块级函数**：1 / 15
- **主要功能简介**：`src/octop/api/routers/terminal.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 788 行，包含 1 个类与 15 个模块级函数。模块文档写道：「Terminal WebSocket — interactive PTY sessions per agent (P1.4).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_PtySession`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `tls.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/tls.py`
- **约行数**：113
- **类 / 模块级函数**：4 / 4
- **主要功能简介**：`src/octop/api/routers/tls.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 113 行，包含 4 个类与 4 个模块级函数。模块文档写道：「TLS / Let's Encrypt admin API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PreflightBody`、`IssueBody`、`PreflightResponse`、`TlsStatusResponse`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `update.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/update.py`
- **约行数**：413
- **类 / 模块级函数**：2 / 17
- **主要功能简介**：`src/octop/api/routers/update.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 413 行，包含 2 个类与 17 个模块级函数。模块文档写道：「Self-update API — mirrors finnie/octop dashboard update flow.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UpdateSettingsBody`、`UpgradeBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `update_store.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/update_store.py`
- **约行数**：86
- **类 / 模块级函数**：2 / 6
- **主要功能简介**：`src/octop/api/routers/update_store.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 86 行，包含 2 个类与 6 个模块级函数。模块文档写道：「In-memory upgrade task tracking for ``/api/update/*``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UpgradeTaskStatus`、`UpgradeTask`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `uploads.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/uploads.py`
- **约行数**：84
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/api/routers/uploads.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 84 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Dashboard chat attachments — stored in agent workspace ``inbound/``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `usage.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/usage.py`
- **约行数**：287
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/api/routers/usage.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 287 行，包含 0 个类与 12 个模块级函数。模块文档写道：「Token usage router — query the usage_log ledger.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `users.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/users.py`
- **约行数**：280
- **类 / 模块级函数**：3 / 13
- **主要功能简介**：`src/octop/api/routers/users.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 280 行，包含 3 个类与 13 个模块级函数。模块文档写道：「Admin CRUD for users.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UserCreateBody`、`UserPatchBody`、`ResetPasswordBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `voice.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/voice.py`
- **约行数**：263
- **类 / 模块级函数**：6 / 14
- **主要功能简介**：`src/octop/api/routers/voice.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 263 行，包含 6 个类与 14 个模块级函数。模块文档写道：「Voice STT/TTS router.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`VoiceProviderCreateBody`、`VoiceProviderPatchBody`、`ActiveVoiceBody`、`TTSBody`、`VoiceTestBody`、`VoiceConfigurationTestBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `workspace.py`

- **所在目录**：`src/octop/api/routers/`
- **相对路径**：`src/octop/api/routers/workspace.py`
- **约行数**：570
- **类 / 模块级函数**：3 / 20
- **主要功能简介**：`src/octop/api/routers/workspace.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 570 行，包含 3 个类与 20 个模块级函数。模块文档写道：「Workspace router — read/write into a running agent's workspace.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`WriteFileBody`、`WriteDocBody`、`MoveFileBody`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/api/routers/browser/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/api/routers/browser/`
- **相对路径**：`src/octop/api/routers/browser/__init__.py`
- **约行数**：21
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/api/routers/browser/__init__.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 21 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Browser routers — env/install, harness-browser, record/replay, WS screencast.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `env.py`

- **所在目录**：`src/octop/api/routers/browser/`
- **相对路径**：`src/octop/api/routers/browser/env.py`
- **约行数**：175
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/api/routers/browser/env.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 175 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Browser environment probe and Chromium install (SSE).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `harness.py`

- **所在目录**：`src/octop/api/routers/browser/`
- **相对路径**：`src/octop/api/routers/browser/harness.py`
- **约行数**：368
- **类 / 模块级函数**：1 / 9
- **主要功能简介**：`src/octop/api/routers/browser/harness.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 368 行，包含 1 个类与 9 个模块级函数。模块文档写道：「Helpers for attaching dashboard / chat UI to harness-browser sessions.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`HandoffBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `record_replay.py`

- **所在目录**：`src/octop/api/routers/browser/`
- **相对路径**：`src/octop/api/routers/browser/record_replay.py`
- **约行数**：325
- **类 / 模块级函数**：5 / 13
- **主要功能简介**：`src/octop/api/routers/browser/record_replay.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 325 行，包含 5 个类与 13 个模块级函数。模块文档写道：「Browser record/replay endpoints backed by harness-browser.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RecordStartBody`、`RecordStopBody`、`RecordStopAndGenerateSkillBody`、`ReplayBody`、`SkillContentBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `stream.py`

- **所在目录**：`src/octop/api/routers/browser/`
- **相对路径**：`src/octop/api/routers/browser/stream.py`
- **约行数**：431
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/api/routers/browser/stream.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 431 行，包含 0 个类与 12 个模块级函数。模块文档写道：「WebSocket browser screencast — attaches to harness-browser sessions.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `uninstall.py`

- **所在目录**：`src/octop/api/routers/browser/`
- **相对路径**：`src/octop/api/routers/browser/uninstall.py`
- **约行数**：35
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/routers/browser/uninstall.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 35 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Browser environment uninstallation (SSE).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/api/routers/chat/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/__init__.py`
- **约行数**：21
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/api/routers/chat/__init__.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 21 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Dashboard chat routers — WebSocket turns, thread CRUD, polish, HITL.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `history.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/history.py`
- **约行数**：582
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`src/octop/api/routers/chat/history.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 582 行，包含 0 个类与 17 个模块级函数。模块文档写道：「Dashboard thread list/history REST APIs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `models.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/models.py`
- **约行数**：194
- **类 / 模块级函数**：7 / 0
- **主要功能简介**：`src/octop/api/routers/chat/models.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 194 行，包含 7 个类与 0 个模块级函数。模块文档写道：「Pydantic models for dashboard chat (WebSocket turn + REST helpers).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ChatTurnBody`、`UserTurnWsFrame`、`PolishBody`、`RebindSessionBody`、`ForkThreadBody`、`RenameThreadBody`、`HitlResumeBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `notify_ws.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/notify_ws.py`
- **约行数**：77
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/routers/chat/notify_ws.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 77 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Dashboard-wide notification WebSocket — text pushes (cron / proactive care).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `routes.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/routes.py`
- **约行数**：256
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/api/routers/chat/routes.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 256 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Dashboard chat helpers: polish prompt and HITL resume (SSE).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `serialize.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/serialize.py`
- **约行数**：1024
- **类 / 模块级函数**：0 / 38
- **主要功能简介**：`src/octop/api/routers/chat/serialize.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 1024 行，包含 0 个类与 38 个模块级函数。模块文档写道：「Thread history loading and LangGraph message serialization.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `sse.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/sse.py`
- **约行数**：33
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/api/routers/chat/sse.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 33 行，包含 0 个类与 2 个模块级函数。模块文档写道：「SSE / WebSocket frame formatting helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `trajectory.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/trajectory.py`
- **约行数**：401
- **类 / 模块级函数**：3 / 17
- **主要功能简介**：`src/octop/api/routers/chat/trajectory.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 401 行，包含 3 个类与 17 个模块级函数。模块文档写道：「Thread trajectory REST APIs (history, detail, metrics, export, live SSE).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TrajectoryEventOut`、`TrajectoryHistoryOut`、`TrajectoryMetricsOut`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `turn.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/turn.py`
- **约行数**：355
- **类 / 模块级函数**：1 / 10
- **主要功能简介**：`src/octop/api/routers/chat/turn.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 355 行，包含 1 个类与 10 个模块级函数。模块文档写道：「Dashboard turn preparation (thread, MCP, skills, InboundMessage).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PreparedDashboardTurn`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `ws.py`

- **所在目录**：`src/octop/api/routers/chat/`
- **相对路径**：`src/octop/api/routers/chat/ws.py`
- **约行数**：184
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/routers/chat/ws.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 184 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Dashboard chat over WebSocket — routes turns through Gateway / GlobalProcessor.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/api/routers/desktop/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/api/routers/desktop/`
- **相对路径**：`src/octop/api/routers/desktop/__init__.py`
- **约行数**：21
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/api/routers/desktop/__init__.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 21 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Remote desktop routers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `install.py`

- **所在目录**：`src/octop/api/routers/desktop/`
- **相对路径**：`src/octop/api/routers/desktop/install.py`
- **约行数**：39
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/routers/desktop/install.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 39 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Desktop environment installation (SSE).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `settings.py`

- **所在目录**：`src/octop/api/routers/desktop/`
- **相对路径**：`src/octop/api/routers/desktop/settings.py`
- **约行数**：50
- **类 / 模块级函数**：1 / 2
- **主要功能简介**：`src/octop/api/routers/desktop/settings.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 50 行，包含 1 个类与 2 个模块级函数。模块文档写道：「Remote desktop settings (geometry).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DesktopGeometryRequest`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `status.py`

- **所在目录**：`src/octop/api/routers/desktop/`
- **相对路径**：`src/octop/api/routers/desktop/status.py`
- **约行数**：39
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/routers/desktop/status.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 39 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Remote desktop HTTP status.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `stream.py`

- **所在目录**：`src/octop/api/routers/desktop/`
- **相对路径**：`src/octop/api/routers/desktop/stream.py`
- **约行数**：380
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`src/octop/api/routers/desktop/stream.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 380 行，包含 0 个类与 9 个模块级函数。模块文档写道：「WebSocket remote desktop stream — JPEG frames + OS input injection.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `uninstall.py`

- **所在目录**：`src/octop/api/routers/desktop/`
- **相对路径**：`src/octop/api/routers/desktop/uninstall.py`
- **约行数**：35
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/routers/desktop/uninstall.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 35 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Desktop environment uninstallation (SSE).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/api/routers/mobile/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/api/routers/mobile/`
- **相对路径**：`src/octop/api/routers/mobile/__init__.py`
- **约行数**：19
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/api/routers/mobile/__init__.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 19 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Remote Android routers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `install.py`

- **所在目录**：`src/octop/api/routers/mobile/`
- **相对路径**：`src/octop/api/routers/mobile/install.py`
- **约行数**：30
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/api/routers/mobile/install.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 30 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Remote Android container install (SSE).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `shell_ws.py`

- **所在目录**：`src/octop/api/routers/mobile/`
- **相对路径**：`src/octop/api/routers/mobile/shell_ws.py`
- **约行数**：172
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/api/routers/mobile/shell_ws.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 172 行，包含 0 个类与 5 个模块级函数。模块文档写道：「WebSocket PTY for ``adb -s <serial> shell``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `status.py`

- **所在目录**：`src/octop/api/routers/mobile/`
- **相对路径**：`src/octop/api/routers/mobile/status.py`
- **约行数**：113
- **类 / 模块级函数**：1 / 5
- **主要功能简介**：`src/octop/api/routers/mobile/status.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 113 行，包含 1 个类与 5 个模块级函数。模块文档写道：「Remote Android HTTP status and agent-control binding.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`MobileAgentControlBody`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `stream.py`

- **所在目录**：`src/octop/api/routers/mobile/`
- **相对路径**：`src/octop/api/routers/mobile/stream.py`
- **约行数**：634
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/api/routers/mobile/stream.py` 属于HTTP 适配层：路由、JWT、SSE/WebSocket、OpenAPI，约 634 行，包含 0 个类与 12 个模块级函数。模块文档写道：「WebSocket Remote Android stream — H.264 screenrecord with JPEG fallback.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/cli/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/cli/`
- **相对路径**：`src/octop/cli/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/cli/__init__.py` 属于Click 命令行：offline / embedded / external 三层传输，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「octop CLI package.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `main.py`

- **所在目录**：`src/octop/cli/`
- **相对路径**：`src/octop/cli/main.py`
- **约行数**：131
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`src/octop/cli/main.py` 属于Click 命令行：offline / embedded / external 三层传输，约 131 行，包含 1 个类与 4 个模块级函数。模块文档写道：「octop CLI entry point with lazy command loading.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_LazyCLI`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `registry.py`

- **所在目录**：`src/octop/cli/`
- **相对路径**：`src/octop/cli/registry.py`
- **约行数**：37
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/cli/registry.py` 属于Click 命令行：offline / embedded / external 三层传输，约 37 行，包含 0 个类与 0 个模块级函数。模块文档写道：「CLI command registry for lazy loading.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/cli/commands/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/cli/commands/__init__.py` 属于Click 命令行：offline / embedded / external 三层传输，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Click command modules (one file per top-level `octop` subcommand).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `acp.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/acp.py`
- **约行数**：64
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/cli/commands/acp.py` 属于Click 命令行：offline / embedded / external 三层传输，约 64 行，包含 0 个类与 2 个模块级函数。模块文档写道：「`octop acp` — run Octop agent as an ACP server over stdio.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `admin.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/admin.py`
- **约行数**：123
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/cli/commands/admin.py` 属于Click 命令行：offline / embedded / external 三层传输，约 123 行，包含 0 个类与 8 个模块级函数。模块文档写道：「octop admin commands.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `agent.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/agent.py`
- **约行数**：238
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`src/octop/cli/commands/agent.py` 属于Click 命令行：offline / embedded / external 三层传输，约 238 行，包含 0 个类与 11 个模块级函数。模块文档写道：「octop agent commands.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `backup.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/backup.py`
- **约行数**：189
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/cli/commands/backup.py` 属于Click 命令行：offline / embedded / external 三层传输，约 189 行，包含 0 个类与 7 个模块级函数。模块文档写道：「`octop backup` — export and restore Octop data.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `captcha.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/captcha.py`
- **约行数**：36
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/cli/commands/captcha.py` 属于Click 命令行：offline / embedded / external 三层传输，约 36 行，包含 0 个类与 2 个模块级函数。模块文档写道：「`octop captcha` — offline captcha maintenance (lockout escape hatch).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `channel.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/channel.py`
- **约行数**：572
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`src/octop/cli/commands/channel.py` 属于Click 命令行：offline / embedded / external 三层传输，约 572 行，包含 0 个类与 15 个模块级函数。模块文档写道：「octop channel commands (local DB + embedded runtime).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `chats.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/chats.py`
- **约行数**：365
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/cli/commands/chats.py` 属于Click 命令行：offline / embedded / external 三层传输，约 365 行，包含 0 个类与 10 个模块级函数。模块文档写道：「octop chats — thread CRUD + interactive REPL (CLI channel / local DB).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `clean.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/clean.py`
- **约行数**：45
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/cli/commands/clean.py` 属于Click 命令行：offline / embedded / external 三层传输，约 45 行，包含 0 个类与 3 个模块级函数。模块文档写道：「`octop clean` — remove CLI state and (optionally) all Octop data.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `completion.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/completion.py`
- **约行数**：59
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/cli/commands/completion.py` 属于Click 命令行：offline / embedded / external 三层传输，约 59 行，包含 0 个类与 4 个模块级函数。模块文档写道：「octop completion: emit / install shell completion script.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `config.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/config.py`
- **约行数**：41
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/cli/commands/config.py` 属于Click 命令行：offline / embedded / external 三层传输，约 41 行，包含 0 个类与 3 个模块级函数。模块文档写道：「octop config — CLI defaults (user / agent pins).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `cron.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/cron.py`
- **约行数**：120
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/cli/commands/cron.py` 属于Click 命令行：offline / embedded / external 三层传输，约 120 行，包含 0 个类与 5 个模块级函数。模块文档写道：「octop cron commands.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `init.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/init.py`
- **约行数**：121
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/cli/commands/init.py` 属于Click 命令行：offline / embedded / external 三层传输，约 121 行，包含 0 个类与 1 个模块级函数。模块文档写道：「`octop init` — bootstrap a fresh Octop install (DB + first admin).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `models.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/models.py`
- **约行数**：245
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/cli/commands/models.py` 属于Click 命令行：offline / embedded / external 三层传输，约 245 行，包含 0 个类与 8 个模块级函数。模块文档写道：「octop models — provider presets and resolved model list.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `plugin.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/plugin.py`
- **约行数**：81
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/cli/commands/plugin.py` 属于Click 命令行：offline / embedded / external 三层传输，约 81 行，包含 0 个类与 6 个模块级函数。模块文档写道：「``octop plugin`` — install and manage plugins.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `provider.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/provider.py`
- **约行数**：108
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/cli/commands/provider.py` 属于Click 命令行：offline / embedded / external 三层传输，约 108 行，包含 0 个类与 5 个模块级函数。模块文档写道：「octop provider commands.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `run.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/run.py`
- **约行数**：192
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/cli/commands/run.py` 属于Click 命令行：offline / embedded / external 三层传输，约 192 行，包含 0 个类与 6 个模块级函数。模块文档写道：「`octop run` — start the FastAPI app in the foreground.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `service.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/service.py`
- **约行数**：215
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/cli/commands/service.py` 属于Click 命令行：offline / embedded / external 三层传输，约 215 行，包含 0 个类与 12 个模块级函数。模块文档写道：「`octop service` — install and manage the Octop system service.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `skills.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/skills.py`
- **约行数**：139
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/cli/commands/skills.py` 属于Click 命令行：offline / embedded / external 三层传输，约 139 行，包含 0 个类与 6 个模块级函数。模块文档写道：「octop skills — per-agent skill library.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `update.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/update.py`
- **约行数**：79
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/cli/commands/update.py` 属于Click 命令行：offline / embedded / external 三层传输，约 79 行，包含 0 个类与 1 个模块级函数。模块文档写道：「`octop update` — self-upgrade via pip or uv.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `user.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/user.py`
- **约行数**：149
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/cli/commands/user.py` 属于Click 命令行：offline / embedded / external 三层传输，约 149 行，包含 0 个类与 8 个模块级函数。模块文档写道：「octop user commands.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `version.py`

- **所在目录**：`src/octop/cli/commands/`
- **相对路径**：`src/octop/cli/commands/version.py`
- **约行数**：18
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/cli/commands/version.py` 属于Click 命令行：offline / embedded / external 三层传输，约 18 行，包含 0 个类与 1 个模块级函数。模块文档写道：「octop version command.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/cli/repl/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/cli/repl/`
- **相对路径**：`src/octop/cli/repl/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/cli/repl/__init__.py` 属于Click 命令行：offline / embedded / external 三层传输，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Chat REPL rendering and session state.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `embedded_session.py`

- **所在目录**：`src/octop/cli/repl/`
- **相对路径**：`src/octop/cli/repl/embedded_session.py`
- **约行数**：43
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/cli/repl/embedded_session.py` 属于Click 命令行：offline / embedded / external 三层传输，约 43 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Ref-counted embedded OctopServer for reuse within one asyncio event loop.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `render.py`

- **所在目录**：`src/octop/cli/repl/`
- **相对路径**：`src/octop/cli/repl/render.py`
- **约行数**：322
- **类 / 模块级函数**：3 / 3
- **主要功能简介**：`src/octop/cli/repl/render.py` 属于Click 命令行：offline / embedded / external 三层传输，约 322 行，包含 3 个类与 3 个模块级函数。模块文档写道：「Terminal rendering for harness stream chunks (CLI chat).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ChatTheme`、`_ToolCallState`、`ChunkRenderer`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `runtime.py`

- **所在目录**：`src/octop/cli/repl/`
- **相对路径**：`src/octop/cli/repl/runtime.py`
- **约行数**：130
- **类 / 模块级函数**：1 / 2
- **主要功能简介**：`src/octop/cli/repl/runtime.py` 属于Click 命令行：offline / embedded / external 三层传输，约 130 行，包含 1 个类与 2 个模块级函数。模块文档写道：「Embedded OctopServer chat runtime via the CLI gateway channel.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CliChatSession`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `session.py`

- **所在目录**：`src/octop/cli/repl/`
- **相对路径**：`src/octop/cli/repl/session.py`
- **约行数**：59
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/cli/repl/session.py` 属于Click 命令行：offline / embedded / external 三层传输，约 59 行，包含 1 个类与 0 个模块级函数。模块文档写道：「REPL session state for toolbar and slash side-effects.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ReplSession`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `toolbar.py`

- **所在目录**：`src/octop/cli/repl/`
- **相对路径**：`src/octop/cli/repl/toolbar.py`
- **约行数**：18
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/cli/repl/toolbar.py` 属于Click 命令行：offline / embedded / external 三层传输，约 18 行，包含 0 个类与 1 个模块级函数。模块文档写道：「prompt_toolkit bottom toolbar for ``octop chats repl``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `turn.py`

- **所在目录**：`src/octop/cli/repl/`
- **相对路径**：`src/octop/cli/repl/turn.py`
- **约行数**：14
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/cli/repl/turn.py` 属于Click 命令行：offline / embedded / external 三层传输，约 14 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Shared chat turn result for CLI streaming.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ChatTurnResult`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/cli/support/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/cli/support/__init__.py` 属于Click 命令行：offline / embedded / external 三层传输，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Shared CLI internals — HTTP client, context, state, prompts, helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `acting.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/acting.py`
- **约行数**：31
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/cli/support/acting.py` 属于Click 命令行：offline / embedded / external 三层传输，约 31 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Resolve acting user for CLI commands.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `ctx.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/ctx.py`
- **约行数**：69
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/cli/support/ctx.py` 属于Click 命令行：offline / embedded / external 三层传输，约 69 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Shared CLI context helpers — root-level option fallback.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `db.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/db.py`
- **约行数**：151
- **类 / 模块级函数**：1 / 7
- **主要功能简介**：`src/octop/cli/support/db.py` 属于Click 命令行：offline / embedded / external 三层传输，约 151 行，包含 1 个类与 7 个模块级函数。模块文档写道：「Offline DB access for CLI commands that only need local SQLite.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`OfflineThreadRow`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `embedded_ops.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/embedded_ops.py`
- **约行数**：100
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/cli/support/embedded_ops.py` 属于Click 命令行：offline / embedded / external 三层传输，约 100 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Embedded OctopServer helpers for CLI ops that need a live runtime.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `errors.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/errors.py`
- **约行数**：15
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/cli/support/errors.py` 属于Click 命令行：offline / embedded / external 三层传输，约 15 行，包含 0 个类与 1 个模块级函数。模块文档写道：「CLI error helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `feishu_creator.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/feishu_creator.py`
- **约行数**：74
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/cli/support/feishu_creator.py` 属于Click 命令行：offline / embedded / external 三层传输，约 74 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Run Feishu bot-creator subprocess locally (no HTTP).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `offline_ops.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/offline_ops.py`
- **约行数**：556
- **类 / 模块级函数**：0 / 35
- **主要功能简介**：`src/octop/cli/support/offline_ops.py` 属于Click 命令行：offline / embedded / external 三层传输，约 556 行，包含 0 个类与 35 个模块级函数。模块文档写道：「Direct infra/DB helpers for local CLI (no HTTP, no login).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `prompts.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/prompts.py`
- **约行数**：50
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/cli/support/prompts.py` 属于Click 命令行：offline / embedded / external 三层传输，约 50 行，包含 0 个类与 8 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `qr.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/qr.py`
- **约行数**：168
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/cli/support/qr.py` 属于Click 命令行：offline / embedded / external 三层传输，约 168 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Terminal QR code rendering (adapted from finnie channels_cmd).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `skills.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/skills.py`
- **约行数**：47
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/cli/support/skills.py` 属于Click 命令行：offline / embedded / external 三层传输，约 47 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Offline CLI helpers for per-agent skills (no HTTP / login).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `state.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/state.py`
- **约行数**：37
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`src/octop/cli/support/state.py` 属于Click 命令行：offline / embedded / external 三层传输，约 37 行，包含 1 个类与 3 个模块级函数。模块文档写道：「CLI state — pinned default user/agent.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CLIState`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `stub.py`

- **所在目录**：`src/octop/cli/support/`
- **相对路径**：`src/octop/cli/support/stub.py`
- **约行数**：25
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/cli/support/stub.py` 属于Click 命令行：offline / embedded / external 三层传输，约 25 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Helper for STUB commands that exist in --help but exit on use.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/i18n/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/i18n/`
- **相对路径**：`src/octop/i18n/__init__.py`
- **约行数**：46
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/i18n/__init__.py` 属于后端国际化：en/zh JSON 与 domain helper，约 46 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Central i18n: JSON locale bundles, lookup, and domain helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `loader.py`

- **所在目录**：`src/octop/i18n/`
- **相对路径**：`src/octop/i18n/loader.py`
- **约行数**：65
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/i18n/loader.py` 属于后端国际化：en/zh JSON 与 domain helper，约 65 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Load nested locale JSON and resolve dot-path keys.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/i18n/domains/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/__init__.py`
- **约行数**：60
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/i18n/domains/__init__.py` 属于后端国际化：en/zh JSON 与 domain helper，约 60 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Per-namespace i18n helpers (each maps to a top-level JSON key).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `agents.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/agents.py`
- **约行数**：92
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/i18n/domains/agents.py` 属于后端国际化：en/zh JSON 与 domain helper，约 92 行，包含 0 个类与 7 个模块级函数。模块文档写道：「``agents.*`` — agent runtime labels and startup error keys.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `attachment.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/attachment.py`
- **约行数**：45
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/i18n/domains/attachment.py` 属于后端国际化：en/zh JSON 与 domain helper，约 45 行，包含 0 个类与 4 个模块级函数。模块文档写道：「``attachment.*`` — inbound path hints and empty-turn placeholders.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `channel.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/channel.py`
- **约行数**：42
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/i18n/domains/channel.py` 属于后端国际化：en/zh JSON 与 domain helper，约 42 行，包含 0 个类与 5 个模块级函数。模块文档写道：「``channel.*`` — IM channel status lines (tool hints, probe errors).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `errors.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/errors.py`
- **约行数**：13
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/i18n/domains/errors.py` 属于后端国际化：en/zh JSON 与 domain helper，约 13 行，包含 0 个类与 1 个模块级函数。模块文档写道：「``errors.*`` — API / OctopError messages.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `skills.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/skills.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/i18n/domains/skills.py` 属于后端国际化：en/zh JSON 与 domain helper，约 29 行，包含 0 个类与 2 个模块级函数。模块文档写道：「``skills.*`` — built-in skill display names (keyed by slug).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `slash.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/slash.py`
- **约行数**：24
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/i18n/domains/slash.py` 属于后端国际化：en/zh JSON 与 domain helper，约 24 行，包含 0 个类与 3 个模块级函数。模块文档写道：「``slash.*`` — slash command responses and catalog labels.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `stream.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/stream.py`
- **约行数**：192
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/i18n/domains/stream.py` 属于后端国际化：en/zh JSON 与 domain helper，约 192 行，包含 0 个类与 5 个模块级函数。模块文档写道：「``stream_errors.*`` — user-facing guidance for chat / IM model failures.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `tls.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/tls.py`
- **约行数**：10
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/i18n/domains/tls.py` 属于后端国际化：en/zh JSON 与 domain helper，约 10 行，包含 0 个类与 1 个模块级函数。模块文档写道：「TLS preflight message helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `tools.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/tools.py`
- **约行数**：122
- **类 / 模块级函数**：1 / 5
- **主要功能简介**：`src/octop/i18n/domains/tools.py` 属于后端国际化：en/zh JSON 与 domain helper，约 122 行，包含 1 个类与 5 个模块级函数。模块文档写道：「``tools.*`` — built-in agent tool display names.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`HitlToolCatalogEntry`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `voice.py`

- **所在目录**：`src/octop/i18n/domains/`
- **相对路径**：`src/octop/i18n/domains/voice.py`
- **约行数**：39
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/i18n/domains/voice.py` 属于后端国际化：en/zh JSON 与 domain helper，约 39 行，包含 0 个类与 4 个模块级函数。模块文档写道：「``voice.*`` — probe and Tencent provider error copy.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/`
- **相对路径**：`src/octop/infra/__init__.py`
- **约行数**：22
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/__init__.py` 属于Octop 源码，约 22 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Octop infrastructure — all domain logic and utilities.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `errors.py`

- **所在目录**：`src/octop/infra/`
- **相对路径**：`src/octop/infra/errors.py`
- **约行数**：278
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/infra/errors.py` 属于OctopError 与 ErrorCode 稳定错误码，约 278 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Stable error codes and the typed exception that crosses API boundaries.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ErrorCode`、`OctopError`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `metrics.py`

- **所在目录**：`src/octop/infra/`
- **相对路径**：`src/octop/infra/metrics.py`
- **约行数**：38
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/metrics.py` 属于进程内计数器 METRICS，约 38 行，包含 1 个类与 0 个模块级函数。模块文档写道：「In-memory metrics counters.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`Metrics`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `server.py`

- **所在目录**：`src/octop/infra/`
- **相对路径**：`src/octop/infra/server.py`
- **约行数**：586
- **类 / 模块级函数**：3 / 8
- **主要功能简介**：`src/octop/infra/server.py` 属于OctopServer.start 装配全部基础设施单例，约 586 行，包含 3 个类与 8 个模块级函数。模块文档写道：「OctopServer — process-level orchestrator.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SizeTimedRotatingFileHandler`、`AppRuntime`、`OctopServer`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `acp_settings.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/acp_settings.py`
- **约行数**：145
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`src/octop/infra/agents/acp_settings.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 145 行，包含 1 个类与 3 个模块级函数。模块文档写道：「User-scoped global ACP runner configuration (settings table).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ACPSettingsStore`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `avatar.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/avatar.py`
- **约行数**：223
- **类 / 模块级函数**：1 / 14
- **主要功能简介**：`src/octop/infra/agents/avatar.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 223 行，包含 1 个类与 14 个模块级函数。模块文档写道：「Expert avatars stored in the agent workspace via ``BackendWorkspace``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`WorkspaceAvatarIO`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `context_breakdown.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/context_breakdown.py`
- **约行数**：255
- **类 / 模块级函数**：1 / 10
- **主要功能简介**：`src/octop/infra/agents/context_breakdown.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 255 行，包含 1 个类与 10 个模块级函数。模块文档写道：「Context window usage without checkpoint reads on dashboard requests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ContextBreakdownResult`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `default_agent.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/default_agent.py`
- **约行数**：90
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/default_agent.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 90 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Bootstrap a first default agent (general-assistant) for a user.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `execute_env.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/execute_env.py`
- **约行数**：139
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/execute_env.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 139 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Inject platform execute defaults into harness backend specs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `langfuse.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/langfuse.py`
- **约行数**：167
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/infra/agents/langfuse.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 167 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Langfuse settings persistence for the agent runtime.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`LangfuseSettings`、`LangfuseSettingsStore`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `manager.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/manager.py`
- **约行数**：2963
- **类 / 模块级函数**：2 / 9
- **主要功能简介**：`src/octop/infra/agents/manager.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 2963 行，包含 2 个类与 9 个模块级函数。模块文档写道：「AgentManager — process-wide singleton managing all HarnessAgent instances.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AgentCreateSpec`、`AgentManager`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `mbti_profiles.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/mbti_profiles.py`
- **约行数**：597
- **类 / 模块级函数**：3 / 3
- **主要功能简介**：`src/octop/infra/agents/mbti_profiles.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 597 行，包含 3 个类与 3 个模块级函数。模块文档写道：「Built-in MBTI personality profiles for the 16 types.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`MBTIDimensions`、`MBTIBehaviorMapping`、`MBTIProfile`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `media_generation.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/media_generation.py`
- **约行数**：309
- **类 / 模块级函数**：2 / 3
- **主要功能简介**：`src/octop/infra/agents/media_generation.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 309 行，包含 2 个类与 3 个模块级函数。模块文档写道：「Instance-wide media-generation settings for harness agents.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`MediaGenerationSettings`、`MediaGenerationSettingsStore`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `memory_backend.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/memory_backend.py`
- **约行数**：88
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/agents/memory_backend.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 88 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Resolve agent memory storage backend for harness-agent / harness-memory.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `persona.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/persona.py`
- **约行数**：84
- **类 / 模块级函数**：1 / 2
- **主要功能简介**：`src/octop/infra/agents/persona.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 84 行，包含 1 个类与 2 个模块级函数。模块文档写道：「Render agent persona system prompt from MBTI profiles or the built-in default template.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PersonaLoader`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `plugin_tool_defaults.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/plugin_tool_defaults.py`
- **约行数**：119
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/agents/plugin_tool_defaults.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 119 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Default-on semantics for plugin tools on agents.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `plugin_tool_names.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/plugin_tool_names.py`
- **约行数**：101
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/agents/plugin_tool_names.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 101 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Sanitize plugin tool names for strict LLM tool-name APIs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `profile.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/profile.py`
- **约行数**：165
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`src/octop/infra/agents/profile.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 165 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Agent profile fields stored on ``agents`` rather than in ``config_json``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `runtime_limits.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/runtime_limits.py`
- **约行数**：183
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/infra/agents/runtime_limits.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 183 行，包含 0 个类与 12 个模块级函数。模块文档写道：「Map agent ``config_json`` runtime knobs onto harness-agent semantics.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `thread_fork.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/thread_fork.py`
- **约行数**：290
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/agents/thread_fork.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 290 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Fork a conversation thread from a selected assistant reply.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `tool_catalog.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/tool_catalog.py`
- **约行数**：226
- **类 / 模块级函数**：1 / 7
- **主要功能简介**：`src/octop/infra/agents/tool_catalog.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 226 行，包含 1 个类与 7 个模块级函数。模块文档写道：「Built-in agent tool catalog for tool-settings UI and disable policy.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`BuiltinToolEntry`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `workspace_dir.py`

- **所在目录**：`src/octop/infra/agents/`
- **相对路径**：`src/octop/infra/agents/workspace_dir.py`
- **约行数**：386
- **类 / 模块级函数**：0 / 22
- **主要功能简介**：`src/octop/infra/agents/workspace_dir.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 386 行，包含 0 个类与 22 个模块级函数。模块文档写道：「Agent ``workspace_dir`` (persisted in ``config_json``).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/builtin_skills/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/builtin_skills/`
- **相对路径**：`src/octop/infra/agents/builtin_skills/__init__.py`
- **约行数**：78
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/agents/builtin_skills/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 78 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Octop-owned built-in Skills seeded into every agent workspace.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/builtin_skills/skill-manager/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `manage_skills.py`

- **所在目录**：`src/octop/infra/agents/builtin_skills/skill-manager/scripts/`
- **相对路径**：`src/octop/infra/agents/builtin_skills/skill-manager/scripts/manage_skills.py`
- **约行数**：684
- **类 / 模块级函数**：1 / 34
- **主要功能简介**：`src/octop/infra/agents/builtin_skills/skill-manager/scripts/manage_skills.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 684 行，包含 1 个类与 34 个模块级函数。模块文档写道：「Safely inspect, install, list, remove, and restore workspace Skills.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SkillManagerError`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/experts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `catalog.py`

- **所在目录**：`src/octop/infra/agents/experts/`
- **相对路径**：`src/octop/infra/agents/experts/catalog.py`
- **约行数**：855
- **类 / 模块级函数**：4 / 37
- **主要功能简介**：`src/octop/infra/agents/experts/catalog.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 855 行，包含 4 个类与 37 个模块级函数。模块文档写道：「Expert catalog — bundled "scene templates" inherited from finnie.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ExpertQuickPrompt`、`ExpertSummary`、`Expert`、`ExpertCatalog`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `manifest_generator.py`

- **所在目录**：`src/octop/infra/agents/experts/`
- **相对路径**：`src/octop/infra/agents/experts/manifest_generator.py`
- **约行数**：666
- **类 / 模块级函数**：1 / 29
- **主要功能简介**：`src/octop/infra/agents/experts/manifest_generator.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 666 行，包含 1 个类与 29 个模块级函数。模块文档写道：「Generate SkillHub expert welcome metadata with an internal generator skill.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ExpertManifestGenerationError`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `market_creation.py`

- **所在目录**：`src/octop/infra/agents/experts/`
- **相对路径**：`src/octop/infra/agents/experts/market_creation.py`
- **约行数**：366
- **类 / 模块级函数**：2 / 6
- **主要功能简介**：`src/octop/infra/agents/experts/market_creation.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 366 行，包含 2 个类与 6 个模块级函数。模块文档写道：「Create agents from SkillHub-backed expert market templates.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SkillHubMarketAgentCreateOptions`、`SkillHubMarketAgentCreateResult`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `publish.py`

- **所在目录**：`src/octop/infra/agents/experts/`
- **相对路径**：`src/octop/infra/agents/experts/publish.py`
- **约行数**：291
- **类 / 模块级函数**：1 / 10
- **主要功能简介**：`src/octop/infra/agents/experts/publish.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 291 行，包含 1 个类与 10 个模块级函数。模块文档写道：「Export agent workspaces as installable published-expert snapshots.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PublishedExpertSnapshotMeta`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `published_creation.py`

- **所在目录**：`src/octop/infra/agents/experts/`
- **相对路径**：`src/octop/infra/agents/experts/published_creation.py`
- **约行数**：383
- **类 / 模块级函数**：1 / 16
- **主要功能简介**：`src/octop/infra/agents/experts/published_creation.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 383 行，包含 1 个类与 16 个模块级函数。模块文档写道：「Publish / refresh / install / unpublish user expert templates.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PublishedExpertInstallOptions`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `skillhub_market.py`

- **所在目录**：`src/octop/infra/agents/experts/`
- **相对路径**：`src/octop/infra/agents/experts/skillhub_market.py`
- **约行数**：1324
- **类 / 模块级函数**：3 / 62
- **主要功能简介**：`src/octop/infra/agents/experts/skillhub_market.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 1324 行，包含 3 个类与 62 个模块级函数。模块文档写道：「SkillHub skillset marketplace integration for expert templates.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SkillHubMarketErrorKind`、`SkillHubMarketError`、`SkillHubSkillset`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `clinical_profile.py`

- **所在目录**：`src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/clinical_profile.py`
- **约行数**：2609
- **类 / 模块级函数**：0 / 89
- **主要功能简介**：`src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/clinical_profile.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 2609 行，包含 0 个类与 89 个模块级函数。模块文档写道：「Manage the minimal learning state for the clinical learning expert.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `simulate_weixin_flow.py`

- **所在目录**：`src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/simulate_weixin_flow.py`
- **约行数**：239
- **类 / 模块级函数**：1 / 6
- **主要功能简介**：`src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/simulate_weixin_flow.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 239 行，包含 1 个类与 6 个模块级函数。模块文档写道：「Simulate subscription creation and learning-delivery decisions.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SessionContext`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `validate_output.py`

- **所在目录**：`src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/validate_output.py`
- **约行数**：514
- **类 / 模块级函数**：0 / 26
- **主要功能简介**：`src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/validate_output.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 514 行，包含 0 个类与 26 个模块级函数。模块文档写道：「校验临床学习输出，在投递前检查格式、来源与边界声明。」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `cluster_patrol_record.py`

- **所在目录**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/cluster_patrol_record.py`
- **约行数**：172
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/cluster_patrol_record.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 172 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Record one cluster health scan result into the JSONL history file.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `cluster_patrol_server.py`

- **所在目录**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/cluster_patrol_server.py`
- **约行数**：227
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/cluster_patrol_server.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 227 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Cluster Doctor Patrol Server — lightweight HTTP server for cluster health data.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ClusterPatrolHandler`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `generate_promo.py`

- **所在目录**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/generate_promo.py`
- **约行数**：54
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/generate_promo.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 54 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `patrol_server.py`

- **所在目录**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/patrol_server.py`
- **约行数**：140
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/patrol_server.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 140 行，包含 1 个类与 1 个模块级函数。模块文档写道：「CVM Doctor Patrol Server」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PatrolHandler`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/experts/library/cvm-cluster-doctor/skills/tencentcloud-infra/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `tccli-oauth-helper.py`

- **所在目录**：`src/octop/infra/agents/experts/library/cvm-cluster-doctor/skills/tencentcloud-infra/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/cvm-cluster-doctor/skills/tencentcloud-infra/scripts/tccli-oauth-helper.py`
- **约行数**：362
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/infra/agents/experts/library/cvm-cluster-doctor/skills/tencentcloud-infra/scripts/tccli-oauth-helper.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 362 行，包含 0 个类与 12 个模块级函数。模块文档写道：「tccli OAuth 登录辅助工具」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `auth.py`

- **所在目录**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/auth.py`
- **约行数**：225
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/auth.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 225 行，包含 0 个类与 11 个模块级函数。模块文档写道：「huisheng-coupon-tool 认证模块」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `diag_auth_log.py`

- **所在目录**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/diag_auth_log.py`
- **约行数**：112
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/diag_auth_log.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 112 行，包含 0 个类与 4 个模块级函数。模块文档写道：「A4 鉴权操作日志诊断脚本」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `diag_issue_log.py`

- **所在目录**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/diag_issue_log.py`
- **约行数**：79
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/diag_issue_log.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 79 行，包含 0 个类与 4 个模块级函数。模块文档写道：「A5 发券接口日志诊断脚本」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `plugin_version_lite.py`

- **所在目录**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/plugin_version_lite.py`
- **约行数**：309
- **类 / 模块级函数**：0 / 19
- **主要功能简介**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/plugin_version_lite.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 309 行，包含 0 个类与 19 个模块级函数。模块文档写道：「plugin_version_lite.py — 插件版本探测精简版（行为同 plugin_version.py，去注释/调试/明细）。」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `qr_local.py`

- **所在目录**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/qr_local.py`
- **约行数**：71
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/qr_local.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 71 行，包含 0 个类与 3 个模块级函数。模块文档写道：「本地二维码生成兜底模块」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `accept_changes.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/accept_changes.py`
- **约行数**：135
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/accept_changes.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 135 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Accept all tracked changes in a DOCX file using LibreOffice.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `comment.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/comment.py`
- **约行数**：318
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/comment.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 318 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Add comments to DOCX documents.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `pack.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/pack.py`
- **约行数**：156
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/pack.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 156 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Pack a directory into a DOCX, PPTX, or XLSX file.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `soffice.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/soffice.py`
- **约行数**：219
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/soffice.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 219 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Helper for running LibreOffice (soffice) in environments where AF_UNIX」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `unpack.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/unpack.py`
- **约行数**：132
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/unpack.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 132 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Unpack Office files (DOCX, PPTX, XLSX) for editing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `validate.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validate.py`
- **约行数**：114
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validate.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 114 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Command line tool to validate Office document XML files against XSD schemas and tracked changes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/__init__.py`
- **约行数**：1
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 1 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `merge_runs.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/merge_runs.py`
- **约行数**：193
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/merge_runs.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 193 行，包含 0 个类与 15 个模块级函数。模块文档写道：「Merge adjacent runs with identical formatting in DOCX.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `simplify_redlines.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/simplify_redlines.py`
- **约行数**：197
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/simplify_redlines.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 197 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Simplify tracked changes by merging adjacent w:ins or w:del elements.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/__init__.py`
- **约行数**：16
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 16 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Validation modules for Word document processing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `base.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/base.py`
- **约行数**：798
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/base.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 798 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Base validator with common validation logic for document files.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`BaseSchemaValidator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `docx.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/docx.py`
- **约行数**：419
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/docx.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 419 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Validator for Word document XML files against XSD schemas.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DOCXSchemaValidator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `pptx.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/pptx.py`
- **约行数**：263
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/pptx.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 263 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Validator for PowerPoint presentation XML files against XSD schemas.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PPTXSchemaValidator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `redlining.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/redlining.py`
- **约行数**：241
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/redlining.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 241 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Validator for tracked changes in Word documents.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RedliningValidator`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `check_bounding_boxes.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/check_bounding_boxes.py`
- **约行数**：72
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/check_bounding_boxes.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 72 行，包含 1 个类与 1 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RectAndField`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `check_fillable_fields.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/check_fillable_fields.py`
- **约行数**：12
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/check_fillable_fields.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 12 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `convert_pdf_to_images.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/convert_pdf_to_images.py`
- **约行数**：32
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/convert_pdf_to_images.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 32 行，包含 0 个类与 1 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `create_validation_image.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/create_validation_image.py`
- **约行数**：38
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/create_validation_image.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 38 行，包含 0 个类与 1 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `extract_form_field_info.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/extract_form_field_info.py`
- **约行数**：129
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/extract_form_field_info.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 129 行，包含 0 个类与 4 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `extract_form_structure.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/extract_form_structure.py`
- **约行数**：117
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/extract_form_structure.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 117 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Extract form structure from a non-fillable PDF.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `fill_fillable_fields.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/fill_fillable_fields.py`
- **约行数**：103
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/fill_fillable_fields.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 103 行，包含 0 个类与 3 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `fill_pdf_form_with_annotations.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/fill_pdf_form_with_annotations.py`
- **约行数**：106
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/fill_pdf_form_with_annotations.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 106 行，包含 0 个类与 3 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/__init__.py`
- **约行数**：1
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 1 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `add_slide.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/add_slide.py`
- **约行数**：199
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/add_slide.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 199 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Add a new slide to an unpacked PPTX directory.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `clean.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/clean.py`
- **约行数**：280
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/clean.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 280 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Remove unreferenced files from an unpacked PPTX directory.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `thumbnail.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/thumbnail.py`
- **约行数**：303
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/thumbnail.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 303 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Create thumbnail grids from PowerPoint presentation slides.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `pack.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/pack.py`
- **约行数**：156
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/pack.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 156 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Pack a directory into a DOCX, PPTX, or XLSX file.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `soffice.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/soffice.py`
- **约行数**：217
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/soffice.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 217 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Helper for running LibreOffice (soffice) in environments where AF_UNIX」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `unpack.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/unpack.py`
- **约行数**：132
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/unpack.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 132 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Unpack Office files (DOCX, PPTX, XLSX) for editing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `validate.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validate.py`
- **约行数**：114
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validate.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 114 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Command line tool to validate Office document XML files against XSD schemas and tracked changes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/__init__.py`
- **约行数**：1
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 1 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `merge_runs.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/merge_runs.py`
- **约行数**：193
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/merge_runs.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 193 行，包含 0 个类与 15 个模块级函数。模块文档写道：「Merge adjacent runs with identical formatting in DOCX.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `simplify_redlines.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/simplify_redlines.py`
- **约行数**：197
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/simplify_redlines.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 197 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Simplify tracked changes by merging adjacent w:ins or w:del elements.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/__init__.py`
- **约行数**：16
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 16 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Validation modules for Word document processing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `base.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/base.py`
- **约行数**：798
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/base.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 798 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Base validator with common validation logic for document files.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`BaseSchemaValidator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `docx.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/docx.py`
- **约行数**：419
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/docx.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 419 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Validator for Word document XML files against XSD schemas.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DOCXSchemaValidator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `pptx.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/pptx.py`
- **约行数**：263
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/pptx.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 263 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Validator for PowerPoint presentation XML files against XSD schemas.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PPTXSchemaValidator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `redlining.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/redlining.py`
- **约行数**：241
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/redlining.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 241 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Validator for tracked changes in Word documents.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RedliningValidator`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `recalc.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/recalc.py`
- **约行数**：198
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/recalc.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 198 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Excel Formula Recalculation Script」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `pack.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/pack.py`
- **约行数**：156
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/pack.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 156 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Pack a directory into a DOCX, PPTX, or XLSX file.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `soffice.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/soffice.py`
- **约行数**：217
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/soffice.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 217 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Helper for running LibreOffice (soffice) in environments where AF_UNIX」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `unpack.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/unpack.py`
- **约行数**：132
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/unpack.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 132 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Unpack Office files (DOCX, PPTX, XLSX) for editing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `validate.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validate.py`
- **约行数**：114
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validate.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 114 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Command line tool to validate Office document XML files against XSD schemas and tracked changes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/__init__.py`
- **约行数**：1
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 1 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `merge_runs.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/merge_runs.py`
- **约行数**：193
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/merge_runs.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 193 行，包含 0 个类与 15 个模块级函数。模块文档写道：「Merge adjacent runs with identical formatting in DOCX.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `simplify_redlines.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/simplify_redlines.py`
- **约行数**：197
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/simplify_redlines.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 197 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Simplify tracked changes by merging adjacent w:ins or w:del elements.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/__init__.py`
- **约行数**：16
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 16 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Validation modules for Word document processing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `base.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/base.py`
- **约行数**：798
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/base.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 798 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Base validator with common validation logic for document files.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`BaseSchemaValidator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `docx.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/docx.py`
- **约行数**：419
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/docx.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 419 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Validator for Word document XML files against XSD schemas.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DOCXSchemaValidator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `pptx.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/pptx.py`
- **约行数**：263
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/pptx.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 263 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Validator for PowerPoint presentation XML files against XSD schemas.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PPTXSchemaValidator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `redlining.py`

- **所在目录**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/`
- **相对路径**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/redlining.py`
- **约行数**：241
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/redlining.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 241 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Validator for tracked changes in Word documents.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RedliningValidator`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/experts/library/wechat-ops/skills/publisher-multi-platform/scripts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `wechat_publish.py`

- **所在目录**：`src/octop/infra/agents/experts/library/wechat-ops/skills/publisher-multi-platform/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/wechat-ops/skills/publisher-multi-platform/scripts/wechat_publish.py`
- **约行数**：1081
- **类 / 模块级函数**：0 / 18
- **主要功能简介**：`src/octop/infra/agents/experts/library/wechat-ops/skills/publisher-multi-platform/scripts/wechat_publish.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 1081 行，包含 0 个类与 18 个模块级函数。模块文档写道：「WeChat Publisher Personal — Publish Markdown to WeChat Official Account」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `xhs_publish.py`

- **所在目录**：`src/octop/infra/agents/experts/library/wechat-ops/skills/publisher-multi-platform/scripts/`
- **相对路径**：`src/octop/infra/agents/experts/library/wechat-ops/skills/publisher-multi-platform/scripts/xhs_publish.py`
- **约行数**：593
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`src/octop/infra/agents/experts/library/wechat-ops/skills/publisher-multi-platform/scripts/xhs_publish.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 593 行，包含 0 个类与 15 个模块级函数。模块文档写道：「XHS (社交内容平台/RedNote) Publisher — Publish content to Xiaohongshu」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/middleware/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `binary_read_guard.py`

- **所在目录**：`src/octop/infra/agents/middleware/`
- **相对路径**：`src/octop/infra/agents/middleware/binary_read_guard.py`
- **约行数**：129
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`src/octop/infra/agents/middleware/binary_read_guard.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 129 行，包含 1 个类与 4 个模块级函数。模块文档写道：「Block ``read_file`` on binary inbound attachments (PDF, Office, …).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`BinaryReadGuardMiddleware`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `browser_profile.py`

- **所在目录**：`src/octop/infra/agents/middleware/`
- **相对路径**：`src/octop/infra/agents/middleware/browser_profile.py`
- **约行数**：72
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/agents/middleware/browser_profile.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 72 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Force browser tool calls onto the current user's isolated profile.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`BrowserProfileMiddleware`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `reasoning.py`

- **所在目录**：`src/octop/infra/agents/middleware/`
- **相对路径**：`src/octop/infra/agents/middleware/reasoning.py`
- **约行数**：51
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/agents/middleware/reasoning.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 51 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Apply per-turn provider-specific reasoning request parameters.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ReasoningRequestMiddleware`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `thread_artifacts.py`

- **所在目录**：`src/octop/infra/agents/middleware/`
- **相对路径**：`src/octop/infra/agents/middleware/thread_artifacts.py`
- **约行数**：340
- **类 / 模块级函数**：2 / 13
- **主要功能简介**：`src/octop/infra/agents/middleware/thread_artifacts.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 340 行，包含 2 个类与 13 个模块级函数。模块文档写道：「Record workspace file paths onto ``threads.artifacts`` after successful tool calls.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ArtifactThreadStore`、`ThreadArtifactsMiddleware`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `token_quota.py`

- **所在目录**：`src/octop/infra/agents/middleware/`
- **相对路径**：`src/octop/infra/agents/middleware/token_quota.py`
- **约行数**：49
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/agents/middleware/token_quota.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 49 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Reject agent turns when the acting user has exhausted their token quota.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TokenQuotaMiddleware`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `workspace_image.py`

- **所在目录**：`src/octop/infra/agents/middleware/`
- **相对路径**：`src/octop/infra/agents/middleware/workspace_image.py`
- **约行数**：89
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/middleware/workspace_image.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 89 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Ephemeral workspace-image rematerialization for model calls (plan B).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`WorkspaceImageMaterializeMiddleware`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/plugins/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/plugins/`
- **相对路径**：`src/octop/infra/agents/plugins/__init__.py`
- **约行数**：6
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/plugins/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 6 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Octop host-side plugin management.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `manager.py`

- **所在目录**：`src/octop/infra/agents/plugins/`
- **相对路径**：`src/octop/infra/agents/plugins/manager.py`
- **约行数**：586
- **类 / 模块级函数**：1 / 9
- **主要功能简介**：`src/octop/infra/agents/plugins/manager.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 586 行，包含 1 个类与 9 个模块级函数。模块文档写道：「Install, load, and expose plugins under ``~/.octop/plugins/``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PluginManager`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `seed.py`

- **所在目录**：`src/octop/infra/agents/plugins/`
- **相对路径**：`src/octop/infra/agents/plugins/seed.py`
- **约行数**：117
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/agents/plugins/seed.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 117 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Copy packaged plugins into the user plugins directory, globally disabled.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/__init__.py`
- **约行数**：9
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 9 行，包含 0 个类与 1 个模块级函数。模块文档写道：「In-package bundled plugins copied into ``~/.octop/plugins`` on init/start.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/bilibili-anime/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/bilibili-anime/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/bilibili-anime/main.py`
- **约行数**：208
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/bilibili-anime/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 208 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Bilibili anime search + episode list for chat UI player.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/fortune/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/fortune/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/fortune/main.py`
- **约行数**：117
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/fortune/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 117 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Local dice, daily fortune, and lots — no network.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/hot-topics/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/hot-topics/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/hot-topics/main.py`
- **约行数**：156
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/hot-topics/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 156 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Public hot lists: Weibo, Zhihu, Hacker News.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/market-quotes/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/market-quotes/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/market-quotes/main.py`
- **约行数**：164
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/market-quotes/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 164 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Forex, crypto, and A-share quotes from public endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/mini-games/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/mini-games/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/mini-games/main.py`
- **约行数**：107
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/mini-games/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 107 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Tic-tac-toe and number guessing — state is passed in tool args.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/parcel-tracker/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/parcel-tracker/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/parcel-tracker/main.py`
- **约行数**：106
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/parcel-tracker/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 106 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Domestic parcel lookup via kuaidi100 public pages, with a link fallback.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/pomodoro/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/pomodoro/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/pomodoro/main.py`
- **约行数**：62
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/pomodoro/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 62 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Pomodoro and countdown cards — timing runs in the Dashboard UI.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/qrcode/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/qrcode/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/qrcode/main.py`
- **约行数**：51
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/qrcode/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 51 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Generate a QR code PNG data URL with segno.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/server-status/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/server-status/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/server-status/main.py`
- **约行数**：160
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/server-status/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 160 行，包含 0 个类与 7 个模块级函数。模块文档写道：「服务器状态 — 采集本机 OS / CPU / 内存 / 磁盘快照，供聊天 UI 渲染。」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/tetris/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/tetris/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/tetris/main.py`
- **约行数**：31
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/tetris/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 31 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Start an in-chat Tetris board. Gameplay runs in the Dashboard UI.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/plugins/bundled/weather/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`src/octop/infra/agents/plugins/bundled/weather/`
- **相对路径**：`src/octop/infra/agents/plugins/bundled/weather/main.py`
- **约行数**：216
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/infra/agents/plugins/bundled/weather/main.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 216 行，包含 0 个类与 8 个模块级函数。模块文档写道：「City weather via Open-Meteo (no API key).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/agents/providers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/__init__.py`
- **约行数**：7
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/providers/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 7 行，包含 0 个类与 0 个模块级函数。模块文档写道：「DB-backed LLM provider catalog and harness factory sync.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `harness_factory.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/harness_factory.py`
- **约行数**：48
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/agents/providers/harness_factory.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 48 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Hot-sync DB provider configs into a shared ``HarnessAgentManager`` factory.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `model_flags.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/model_flags.py`
- **约行数**：128
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/agents/providers/model_flags.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 128 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Flags on provider ``models_json`` entries that affect chat / auto-routing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `onnx_catalog.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/onnx_catalog.py`
- **约行数**：180
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/agents/providers/onnx_catalog.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 180 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Curated ONNX / fastembed embedding model catalog.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `onnx_download.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/onnx_download.py`
- **约行数**：458
- **类 / 模块级函数**：1 / 21
- **主要功能简介**：`src/octop/infra/agents/providers/onnx_download.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 458 行，包含 1 个类与 21 个模块级函数。模块文档写道：「Race COS, Hugging Face, and hf-mirror, then download from the winner.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DownloadCandidate`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `onnx_service.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/onnx_service.py`
- **约行数**：602
- **类 / 模块级函数**：4 / 30
- **主要功能简介**：`src/octop/infra/agents/providers/onnx_service.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 602 行，包含 4 个类与 30 个模块级函数。模块文档写道：「ONNX local embedding service: config, cache, and download lifecycle.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`OnnxDownloadStatus`、`OnnxDownloadState`、`OnnxServiceConfig`、`OnnxDownloadManager`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `opencode_session.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/opencode_session.py`
- **约行数**：65
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/agents/providers/opencode_session.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 65 行，包含 0 个类与 2 个模块级函数。模块文档写道：「OpenCode Go gateway URL detection and probe session headers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `presets.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/presets.py`
- **约行数**：280
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/agents/providers/presets.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 280 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Built-in provider template loading (harness-agent bundles).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `probe.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/probe.py`
- **约行数**：328
- **类 / 模块级函数**：0 / 14
- **主要功能简介**：`src/octop/infra/agents/providers/probe.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 328 行，包含 0 个类与 14 个模块级函数。模块文档写道：「Provider connectivity probes (shared by API and CLI).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `reasoning.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/reasoning.py`
- **约行数**：236
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/agents/providers/reasoning.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 236 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Normalize model reasoning capabilities and build provider-specific overrides.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `resolved.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/resolved.py`
- **约行数**：50
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/agents/providers/resolved.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 50 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Resolved model list across enabled providers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `store.py`

- **所在目录**：`src/octop/infra/agents/providers/`
- **相对路径**：`src/octop/infra/agents/providers/store.py`
- **约行数**：311
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`src/octop/infra/agents/providers/store.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 311 行，包含 1 个类与 4 个模块级函数。模块文档写道：「Read Octop DB provider rows and build harness ``ProviderConfig`` objects.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ProviderStore`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/security/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/agents/security/`
- **相对路径**：`src/octop/infra/agents/security/__init__.py`
- **约行数**：7
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/agents/security/__init__.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 7 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Security policy persistence for the agent runtime.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `policy_store.py`

- **所在目录**：`src/octop/infra/agents/security/`
- **相对路径**：`src/octop/infra/agents/security/policy_store.py`
- **约行数**：54
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/agents/security/policy_store.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 54 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Global SecurityPolicy persistence in the settings table.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SecuritySettingsStore`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `tool_guard_rules.py`

- **所在目录**：`src/octop/infra/agents/security/`
- **相对路径**：`src/octop/infra/agents/security/tool_guard_rules.py`
- **约行数**：61
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/agents/security/tool_guard_rules.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 61 行，包含 1 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ToolGuardRulesStore`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/agents/subagents/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `catalog.py`

- **所在目录**：`src/octop/infra/agents/subagents/`
- **相对路径**：`src/octop/infra/agents/subagents/catalog.py`
- **约行数**：558
- **类 / 模块级函数**：4 / 5
- **主要功能简介**：`src/octop/infra/agents/subagents/catalog.py` 属于Agent 注册表、专家库、插件、MBTI、ACP、安全策略，约 558 行，包含 4 个类与 5 个模块级函数。模块文档写道：「Subagent catalog — bundled agency-agents definitions.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DivisionMeta`、`SubagentSummary`、`SubagentDefinition`、`SubagentCatalog`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/auth/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/auth/`
- **相对路径**：`src/octop/infra/auth/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/auth/__init__.py` 属于Octop 源码，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Authentication domain helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/auth/captcha/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/auth/captcha/`
- **相对路径**：`src/octop/infra/auth/captcha/__init__.py`
- **约行数**：58
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/auth/captcha/__init__.py` 属于Octop 源码，约 58 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Login captcha: provider registry, config, and verify orchestration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `config.py`

- **所在目录**：`src/octop/infra/auth/captcha/`
- **相对路径**：`src/octop/infra/auth/captcha/config.py`
- **约行数**：86
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`src/octop/infra/auth/captcha/config.py` 属于Octop 源码，约 86 行，包含 1 个类与 4 个模块级函数。模块文档写道：「Boot-time captcha env snapshot.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CaptchaEnv`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `providers.py`

- **所在目录**：`src/octop/infra/auth/captcha/`
- **相对路径**：`src/octop/infra/auth/captcha/providers.py`
- **约行数**：355
- **类 / 模块级函数**：6 / 8
- **主要功能简介**：`src/octop/infra/auth/captcha/providers.py` 属于Octop 源码，约 355 行，包含 6 个类与 8 个模块级函数。模块文档写道：「Login captcha providers: contract, default implementation, registry.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`VerifyCall`、`CaptchaProvider`、`_SliderProvider`、`_FormPostProvider`、`_RecaptchaV3Provider`、`_TencentProvider`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `store.py`

- **所在目录**：`src/octop/infra/auth/captcha/`
- **相对路径**：`src/octop/infra/auth/captcha/store.py`
- **约行数**：358
- **类 / 模块级函数**：1 / 12
- **主要功能简介**：`src/octop/infra/auth/captcha/store.py` 属于Octop 源码，约 358 行，包含 1 个类与 12 个模块级函数。模块文档写道：「Settings blob + env snapshot → effective captcha config.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`EffectiveCaptcha`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `verify.py`

- **所在目录**：`src/octop/infra/auth/captcha/`
- **相对路径**：`src/octop/infra/auth/captcha/verify.py`
- **约行数**：77
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/auth/captcha/verify.py` 属于Octop 源码，约 77 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Execution layer for login captcha: the only outbound-I/O seam.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/auth/sso/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/auth/sso/`
- **相对路径**：`src/octop/infra/auth/sso/__init__.py`
- **约行数**：6
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/auth/sso/__init__.py` 属于Octop 源码，约 6 行，包含 0 个类与 0 个模块级函数。模块文档写道：「OpenID Connect single sign-on helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `crypto.py`

- **所在目录**：`src/octop/infra/auth/sso/`
- **相对路径**：`src/octop/infra/auth/sso/crypto.py`
- **约行数**：25
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/auth/sso/crypto.py` 属于Octop 源码，约 25 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Fernet encryption for OIDC single sign-on secrets.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `discovery.py`

- **所在目录**：`src/octop/infra/auth/sso/`
- **相对路径**：`src/octop/infra/auth/sso/discovery.py`
- **约行数**：49
- **类 / 模块级函数**：1 / 2
- **主要功能简介**：`src/octop/infra/auth/sso/discovery.py` 属于Octop 源码，约 49 行，包含 1 个类与 2 个模块级函数。模块文档写道：「OpenID Connect discovery document helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DiscoveryCache`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `id_token.py`

- **所在目录**：`src/octop/infra/auth/sso/`
- **相对路径**：`src/octop/infra/auth/sso/id_token.py`
- **约行数**：59
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/auth/sso/id_token.py` 属于Octop 源码，约 59 行，包含 0 个类与 2 个模块级函数。模块文档写道：「OpenID Connect ID token verification.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `pkce.py`

- **所在目录**：`src/octop/infra/auth/sso/`
- **相对路径**：`src/octop/infra/auth/sso/pkce.py`
- **约行数**：15
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/auth/sso/pkce.py` 属于Octop 源码，约 15 行，包含 0 个类与 1 个模块级函数。模块文档写道：「PKCE helpers for OIDC authorization flows.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `public_base.py`

- **所在目录**：`src/octop/infra/auth/sso/`
- **相对路径**：`src/octop/infra/auth/sso/public_base.py`
- **约行数**：42
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/auth/sso/public_base.py` 属于Octop 源码，约 42 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Public URL helpers for SSO callbacks (framework-free).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `redirect_after.py`

- **所在目录**：`src/octop/infra/auth/sso/`
- **相对路径**：`src/octop/infra/auth/sso/redirect_after.py`
- **约行数**：15
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/auth/sso/redirect_after.py` 属于Octop 源码，约 15 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Safe post-login redirect path handling.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `service.py`

- **所在目录**：`src/octop/infra/auth/sso/`
- **相对路径**：`src/octop/infra/auth/sso/service.py`
- **约行数**：497
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/auth/sso/service.py` 属于Octop 源码，约 497 行，包含 2 个类与 0 个模块级函数。模块文档写道：「SSO service orchestration for OIDC and pluggable OAuth providers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RedirectResult`、`SsoService`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/auth/sso/providers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/auth/sso/providers/`
- **相对路径**：`src/octop/infra/auth/sso/providers/__init__.py`
- **约行数**：28
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/auth/sso/providers/__init__.py` 属于Octop 源码，约 28 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Dashboard SSO identity-provider adapters.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `base.py`

- **所在目录**：`src/octop/infra/auth/sso/providers/`
- **相对路径**：`src/octop/infra/auth/sso/providers/base.py`
- **约行数**：40
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/auth/sso/providers/base.py` 属于Octop 源码，约 40 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Identity-provider adapter contract for dashboard SSO.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`IdentityProvider`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `dingtalk.py`

- **所在目录**：`src/octop/infra/auth/sso/providers/`
- **相对路径**：`src/octop/infra/auth/sso/providers/dingtalk.py`
- **约行数**：139
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/auth/sso/providers/dingtalk.py` 属于Octop 源码，约 139 行，包含 1 个类与 0 个模块级函数。模块文档写道：「DingTalk enterprise-app web OAuth dashboard login adapter.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DingTalkAdapter`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `feishu.py`

- **所在目录**：`src/octop/infra/auth/sso/providers/`
- **相对路径**：`src/octop/infra/auth/sso/providers/feishu.py`
- **约行数**：160
- **类 / 模块级函数**：1 / 2
- **主要功能简介**：`src/octop/infra/auth/sso/providers/feishu.py` 属于Octop 源码，约 160 行，包含 1 个类与 2 个模块级函数。模块文档写道：「Feishu / Lark web OAuth dashboard login adapter.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`FeishuAdapter`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `oidc.py`

- **所在目录**：`src/octop/infra/auth/sso/providers/`
- **相对路径**：`src/octop/infra/auth/sso/providers/oidc.py`
- **约行数**：89
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/auth/sso/providers/oidc.py` 属于Octop 源码，约 89 行，包含 1 个类与 0 个模块级函数。模块文档写道：「OpenID Connect dashboard login adapter.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`OidcAdapter`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `wecom.py`

- **所在目录**：`src/octop/infra/auth/sso/providers/`
- **相对路径**：`src/octop/infra/auth/sso/providers/wecom.py`
- **约行数**：183
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/auth/sso/providers/wecom.py` 属于Octop 源码，约 183 行，包含 1 个类与 1 个模块级函数。模块文档写道：「WeCom (企业微信) CorpApp web login adapter for dashboard SSO.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`WeComAdapter`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/backend/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/backend/`
- **相对路径**：`src/octop/infra/backend/__init__.py`
- **约行数**：18
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/backend/__init__.py` 属于工作区存储适配、远程探测、路径解析，约 18 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Agent backend configuration — Octop DB rows → harness specs + probes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `adapter.py`

- **所在目录**：`src/octop/infra/backend/`
- **相对路径**：`src/octop/infra/backend/adapter.py`
- **约行数**：186
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/backend/adapter.py` 属于工作区存储适配、远程探测、路径解析，约 186 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Map Octop ``storage_backends`` rows to harness-agent backend specs (no I/O).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `browse.py`

- **所在目录**：`src/octop/infra/backend/`
- **相对路径**：`src/octop/infra/backend/browse.py`
- **约行数**：67
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/backend/browse.py` 属于工作区存储适配、远程探测、路径解析，约 67 行，包含 0 个类与 4 个模块级函数。模块文档写道：「List directories on a configured storage backend (harness ``als``).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `docker_spec.py`

- **所在目录**：`src/octop/infra/backend/`
- **相对路径**：`src/octop/infra/backend/docker_spec.py`
- **约行数**：92
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/backend/docker_spec.py` 属于工作区存储适配、远程探测、路径解析，约 92 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Octop helpers for harness ``type: "docker"`` backend specs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `opensandbox_deps.py`

- **所在目录**：`src/octop/infra/backend/`
- **相对路径**：`src/octop/infra/backend/opensandbox_deps.py`
- **约行数**：38
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/backend/opensandbox_deps.py` 属于工作区存储适配、远程探测、路径解析，约 38 行，包含 0 个类与 2 个模块级函数。模块文档写道：「On-demand install of the official ``opensandbox`` SDK.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `probe.py`

- **所在目录**：`src/octop/infra/backend/`
- **相对路径**：`src/octop/infra/backend/probe.py`
- **约行数**：236
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/backend/probe.py` 属于工作区存储适配、远程探测、路径解析，约 236 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Octop-specific storage backend probes (docker + row → harness probe).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `resolver.py`

- **所在目录**：`src/octop/infra/backend/`
- **相对路径**：`src/octop/infra/backend/resolver.py`
- **约行数**：189
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/backend/resolver.py` 属于工作区存储适配、远程探测、路径解析，约 189 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Resolve agent ``backend`` config (named refs, composite routes).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/backup/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/backup/`
- **相对路径**：`src/octop/infra/backup/__init__.py`
- **约行数**：34
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/backup/__init__.py` 属于Octop 源码，约 34 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Backup and restore for Octop data.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `auto.py`

- **所在目录**：`src/octop/infra/backup/`
- **相对路径**：`src/octop/infra/backup/auto.py`
- **约行数**：291
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`src/octop/infra/backup/auto.py` 属于Octop 源码，约 291 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Automatic system backup scheduling (process-level CronManager job).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `chats.py`

- **所在目录**：`src/octop/infra/backup/`
- **相对路径**：`src/octop/infra/backup/chats.py`
- **约行数**：382
- **类 / 模块级函数**：0 / 18
- **主要功能简介**：`src/octop/infra/backup/chats.py` 属于Octop 源码，约 382 行，包含 0 个类与 18 个模块级函数。模块文档写道：「Chat-history tables and workspace paths excluded from default backups.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `manifest.py`

- **所在目录**：`src/octop/infra/backup/`
- **相对路径**：`src/octop/infra/backup/manifest.py`
- **约行数**：101
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/backup/manifest.py` 属于Octop 源码，约 101 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Backup archive manifest.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AgentBackupEntry`、`BackupManifest`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `pg_dump.py`

- **所在目录**：`src/octop/infra/backup/`
- **相对路径**：`src/octop/infra/backup/pg_dump.py`
- **约行数**：69
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/backup/pg_dump.py` 属于Octop 源码，约 69 行，包含 0 个类与 3 个模块级函数。模块文档写道：「PostgreSQL dump/restore via client tools on PATH.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `snapshot.py`

- **所在目录**：`src/octop/infra/backup/`
- **相对路径**：`src/octop/infra/backup/snapshot.py`
- **约行数**：350
- **类 / 模块级函数**：0 / 16
- **主要功能简介**：`src/octop/infra/backup/snapshot.py` 属于Octop 源码，约 350 行，包含 0 个类与 16 个模块级函数。模块文档写道：「SQLite snapshot helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `store.py`

- **所在目录**：`src/octop/infra/backup/`
- **相对路径**：`src/octop/infra/backup/store.py`
- **约行数**：250
- **类 / 模块级函数**：2 / 13
- **主要功能简介**：`src/octop/infra/backup/store.py` 属于Octop 源码，约 250 行，包含 2 个类与 13 个模块级函数。模块文档写道：「On-disk backup archive store under ``PathLayout.backups_dir``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`BackupFileInfo`、`BackupContentFlags`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `system_archive.py`

- **所在目录**：`src/octop/infra/backup/`
- **相对路径**：`src/octop/infra/backup/system_archive.py`
- **约行数**：658
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`src/octop/infra/backup/system_archive.py` 属于Octop 源码，约 658 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Full-system backup and restore (database + local agent workspaces + config).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `workspace_archive.py`

- **所在目录**：`src/octop/infra/backup/`
- **相对路径**：`src/octop/infra/backup/workspace_archive.py`
- **约行数**：175
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/infra/backup/workspace_archive.py` 属于Octop 源码，约 175 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Zip export/import for a single agent workspace.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/browser/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/browser/`
- **相对路径**：`src/octop/infra/browser/__init__.py`
- **约行数**：20
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/browser/__init__.py` 属于Octop 源码，约 20 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Browser environment helpers (install/uninstall, profile prep).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `setup.py`

- **所在目录**：`src/octop/infra/browser/`
- **相对路径**：`src/octop/infra/browser/setup.py`
- **约行数**：532
- **类 / 模块级函数**：0 / 26
- **主要功能简介**：`src/octop/infra/browser/setup.py` 属于Octop 源码，约 532 行，包含 0 个类与 26 个模块级函数。模块文档写道：「Browser environment setup: profile prep before Chrome launch, uninstall SSE.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/connectors/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/connectors/__init__.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Connector catalog, credential handling, and MCP config builders.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `builder.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/builder.py`
- **约行数**：621
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`src/octop/infra/connectors/builder.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 621 行，包含 0 个类与 15 个模块级函数。模块文档写道：「Build harness HTTP MCP connection specs from connector instances.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `catalog.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/catalog.py`
- **约行数**：583
- **类 / 模块级函数**：2 / 6
- **主要功能简介**：`src/octop/infra/connectors/catalog.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 583 行，包含 2 个类与 6 个模块级函数。模块文档写道：「Static connector catalog — bundled presets for HTTP MCP services.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ConnectorCredentialField`、`ConnectorCatalogEntry`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `crypto.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/crypto.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/connectors/crypto.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 29 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Fernet encryption for connector credentials.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `custom_mcp.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/custom_mcp.py`
- **约行数**：408
- **类 / 模块级函数**：0 / 27
- **主要功能简介**：`src/octop/infra/connectors/custom_mcp.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 408 行，包含 0 个类与 27 个模块级函数。模块文档写道：「User-defined MCP servers (streamable_http / stdio) stored as one connector doc.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `default_open.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/default_open.py`
- **约行数**：58
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/connectors/default_open.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 58 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Helpers for connector ``default_open`` (always inject tools when enabled).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `mail_servers.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/mail_servers.py`
- **约行数**：91
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/connectors/mail_servers.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 91 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Shared IMAP/SMTP host presets for mailbox connectors.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `mcp_tool_cache.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/mcp_tool_cache.py`
- **约行数**：95
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/connectors/mcp_tool_cache.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 95 行，包含 0 个类与 2 个模块级函数。模块文档写道：「User-scoped MCP tool cache helpers (fingerprint + serialized tool wrappers).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `probe.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/probe.py`
- **约行数**：553
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`src/octop/infra/connectors/probe.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 553 行，包含 0 个类与 17 个模块级函数。模块文档写道：「Connector credential probe — validate connectivity and list tools.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `service.py`

- **所在目录**：`src/octop/infra/connectors/`
- **相对路径**：`src/octop/infra/connectors/service.py`
- **约行数**：638
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/infra/connectors/service.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 638 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Connector service — MCP config assembly and credential access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ConnectorNameTakenError`、`ConnectorService`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/connectors/gateway/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/__init__.py`
- **约行数**：12
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/connectors/gateway/__init__.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 12 行，包含 0 个类与 0 个模块级函数。模块文档写道：「In-process MCP gateway for Octop-hosted connector adapters.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `cli_dirs.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/cli_dirs.py`
- **约行数**：66
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/connectors/gateway/cli_dirs.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 66 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Cleanup helpers for per-instance connector CLI config directories.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `cli_fingerprint.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/cli_fingerprint.py`
- **约行数**：16
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/connectors/gateway/cli_fingerprint.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 16 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Local CLI credential change-detection fingerprints (not auth storage).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `cli_install.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/cli_install.py`
- **约行数**：259
- **类 / 模块级函数**：1 / 11
- **主要功能简介**：`src/octop/infra/connectors/gateway/cli_install.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 259 行，包含 1 个类与 11 个模块级函数。模块文档写道：「Install / detect host CLIs for Feishu & WeCom connector adapters.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CliInstallSpec`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `cli_runner.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/cli_runner.py`
- **约行数**：94
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/connectors/gateway/cli_runner.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 94 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Shared subprocess runner for official connector CLIs (lark-cli / wecom-cli).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `feishu_creds.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/feishu_creds.py`
- **约行数**：105
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/connectors/gateway/feishu_creds.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 105 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Materialize lark-cli config for headless Octop connector instances.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `feishu_user_auth.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/feishu_user_auth.py`
- **约行数**：224
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/infra/connectors/gateway/feishu_user_auth.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 224 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Feishu CLI user identity via OAuth device-code (lark-cli auth login).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `langchain.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/langchain.py`
- **约行数**：79
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/connectors/gateway/langchain.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 79 行，包含 0 个类与 1 个模块级函数。模块文档写道：「LangChain tool factory for in-process gateway MCP fallback.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `protocol.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/protocol.py`
- **约行数**：79
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/connectors/gateway/protocol.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 79 行，包含 0 个类与 3 个模块级函数。模块文档写道：「MCP JSON-RPC protocol handlers for internal gateway endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `registry.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/registry.py`
- **约行数**：78
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`src/octop/infra/connectors/gateway/registry.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 78 行，包含 1 个类与 4 个模块级函数。模块文档写道：「Gateway adapter registry — kind → list_tools / call_tool / probe.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`GatewayAdapter`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `wecom_creds.py`

- **所在目录**：`src/octop/infra/connectors/gateway/`
- **相对路径**：`src/octop/infra/connectors/gateway/wecom_creds.py`
- **约行数**：157
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/connectors/gateway/wecom_creds.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 157 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Materialize wecom-cli bot.enc + mcp_config.enc for headless Octop instances.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/connectors/gateway/adapters/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/__init__.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Per-vendor gateway MCP adapters.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `baidu_map.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/baidu_map.py`
- **约行数**：114
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/baidu_map.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 114 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Baidu Map Agent Plan gateway — Bearer Token (sk-ap-…).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `ctrip_wendao.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/ctrip_wendao.py`
- **约行数**：80
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/ctrip_wendao.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 80 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Ctrip Wendao (携程问道) gateway — paste Token from authorize page.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `feishu_cli.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/feishu_cli.py`
- **约行数**：280
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/feishu_cli.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 280 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Feishu / Lark official CLI gateway — category tools wrapping ``lark-cli``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `fliggy.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/fliggy.py`
- **约行数**：214
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/fliggy.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 214 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Fliggy (飞猪) AI — signed remote MCP, natural-language search only.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `meituan_travel.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/meituan_travel.py`
- **约行数**：99
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/meituan_travel.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 99 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Meituan Travel Assistant gateway — hotel / flight / train / attraction NL query.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `qq_mail.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/qq_mail.py`
- **约行数**：212
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/qq_mail.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 212 行，包含 0 个类与 10 个模块级函数。模块文档写道：「QQ mail and other IMAP/SMTP mailbox gateway.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `qq_music.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/qq_music.py`
- **约行数**：146
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/qq_music.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 146 行，包含 0 个类与 5 个模块级函数。模块文档写道：「QQ Music Skills gateway — Bearer API Key (qmk-…).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `tencent_ima.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/tencent_ima.py`
- **约行数**：509
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/tencent_ima.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 509 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Tencent IMA notes and knowledge base gateway.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `tencent_news.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/tencent_news.py`
- **约行数**：102
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/tencent_news.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 102 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Tencent News search gateway — official OpenAPI with API Key.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `wechat_reading.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/wechat_reading.py`
- **约行数**：91
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/wechat_reading.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 91 行，包含 0 个类与 7 个模块级函数。模块文档写道：「WeChat Reading (WeRead) bookshelf gateway.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `wecom_cli.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/wecom_cli.py`
- **约行数**：213
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/wecom_cli.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 213 行，包含 0 个类与 7 个模块级函数。模块文档写道：「WeCom official CLI gateway — category tools wrapping ``wecom-cli``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `weknora.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/weknora.py`
- **约行数**：240
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/weknora.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 240 行，包含 0 个类与 12 个模块级函数。模块文档写道：「Read-only WeKnora knowledge-base gateway adapter.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `yuandian.py`

- **所在目录**：`src/octop/infra/connectors/gateway/adapters/`
- **相对路径**：`src/octop/infra/connectors/gateway/adapters/yuandian.py`
- **约行数**：222
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/connectors/gateway/adapters/yuandian.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 222 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Yuandian (元典) Legal AI gateway — laws, cases, enterprises via Open API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/connectors/oauth/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/connectors/oauth/`
- **相对路径**：`src/octop/infra/connectors/oauth/__init__.py`
- **约行数**：34
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/connectors/oauth/__init__.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 34 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Connector OAuth flows — per-vendor modules + :mod:`registry` dispatcher.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `builtin.py`

- **所在目录**：`src/octop/infra/connectors/oauth/`
- **相对路径**：`src/octop/infra/connectors/oauth/builtin.py`
- **约行数**：37
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/connectors/oauth/builtin.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 37 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Built-in OAuth client credentials shipped with Octop (override via env/settings).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `discovery.py`

- **所在目录**：`src/octop/infra/connectors/oauth/`
- **相对路径**：`src/octop/infra/connectors/oauth/discovery.py`
- **约行数**：215
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/connectors/oauth/discovery.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 215 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Discover MCP OAuth issuers from a remote MCP URL (RFC 9728 + MCP authorization).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `mcp.py`

- **所在目录**：`src/octop/infra/connectors/oauth/`
- **相对路径**：`src/octop/infra/connectors/oauth/mcp.py`
- **约行数**：248
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/infra/connectors/oauth/mcp.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 248 行，包含 0 个类与 12 个模块级函数。模块文档写道：「OAuth 2.0 for remote MCP servers via RFC 8414 + dynamic registration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `pkce.py`

- **所在目录**：`src/octop/infra/connectors/oauth/`
- **相对路径**：`src/octop/infra/connectors/oauth/pkce.py`
- **约行数**：15
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/connectors/oauth/pkce.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 15 行，包含 0 个类与 1 个模块级函数。模块文档写道：「PKCE helpers shared by connector OAuth flows.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `registry.py`

- **所在目录**：`src/octop/infra/connectors/oauth/`
- **相对路径**：`src/octop/infra/connectors/oauth/registry.py`
- **约行数**：400
- **类 / 模块级函数**：0 / 20
- **主要功能简介**：`src/octop/infra/connectors/oauth/registry.py` 属于连接器目录、OAuth、MCP 组装、凭据加密，约 400 行，包含 0 个类与 20 个模块级函数。模块文档写道：「Dispatch connector OAuth flows by kind.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/cron/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/cron/`
- **相对路径**：`src/octop/infra/cron/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/cron/__init__.py` 属于定时任务、触发器、投递与 Agent 工具钩子，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `delivery.py`

- **所在目录**：`src/octop/infra/cron/`
- **相对路径**：`src/octop/infra/cron/delivery.py`
- **约行数**：315
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/infra/cron/delivery.py` 属于定时任务、触发器、投递与 Agent 工具钩子，约 315 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Cron delivery orchestration, separate from channel transport.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CronDeliveryCommand`、`CronDeliveryService`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `job.py`

- **所在目录**：`src/octop/infra/cron/`
- **相对路径**：`src/octop/infra/cron/job.py`
- **约行数**：117
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/cron/job.py` 属于定时任务、触发器、投递与 Agent 工具钩子，约 117 行，包含 1 个类与 0 个模块级函数。模块文档写道：「CronJob — APScheduler callable with status and audit bookkeeping.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CronJob`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `manager.py`

- **所在目录**：`src/octop/infra/cron/`
- **相对路径**：`src/octop/infra/cron/manager.py`
- **约行数**：311
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/cron/manager.py` 属于定时任务、触发器、投递与 Agent 工具钩子，约 311 行，包含 2 个类与 0 个模块级函数。模块文档写道：「CronManager — process-wide singleton that owns all user-defined scheduled CronJob instances.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CronCreateSpec`、`CronManager`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `task_type.py`

- **所在目录**：`src/octop/infra/cron/`
- **相对路径**：`src/octop/infra/cron/task_type.py`
- **约行数**：54
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/cron/task_type.py` 属于定时任务、触发器、投递与 Agent 工具钩子，约 54 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Cron job delivery mode — shared domain type (no DB / gateway imports).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `tools.py`

- **所在目录**：`src/octop/infra/cron/`
- **相对路径**：`src/octop/infra/cron/tools.py`
- **约行数**：265
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/cron/tools.py` 属于定时任务、触发器、投递与 Agent 工具钩子，约 265 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Built-in LangChain tools for agent-managed cron jobs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `trigger.py`

- **所在目录**：`src/octop/infra/cron/`
- **相对路径**：`src/octop/infra/cron/trigger.py`
- **约行数**：83
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/cron/trigger.py` 属于定时任务、触发器、投递与 Agent 工具钩子，约 83 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Trigger string → APScheduler trigger.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/db/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/db/`
- **相对路径**：`src/octop/infra/db/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/db/__init__.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `factory.py`

- **所在目录**：`src/octop/infra/db/`
- **相对路径**：`src/octop/infra/db/factory.py`
- **约行数**：45
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/db/factory.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 45 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Open a database pool from process configuration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `migrate.py`

- **所在目录**：`src/octop/infra/db/`
- **相对路径**：`src/octop/infra/db/migrate.py`
- **约行数**：1714
- **类 / 模块级函数**：0 / 50
- **主要功能简介**：`src/octop/infra/db/migrate.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 1714 行，包含 0 个类与 50 个模块级函数。模块文档写道：「Apply numbered SQL migrations.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `pool.py`

- **所在目录**：`src/octop/infra/db/`
- **相对路径**：`src/octop/infra/db/pool.py`
- **约行数**：170
- **类 / 模块级函数**：5 / 3
- **主要功能简介**：`src/octop/infra/db/pool.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 170 行，包含 5 个类与 3 个模块级函数。模块文档写道：「Database pools: SQLite (default) and PostgreSQL.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DatabasePool`、`SqlitePool`、`_CompatRow`、`_PgConnectionProxy`、`PostgresPool`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `probe.py`

- **所在目录**：`src/octop/infra/db/`
- **相对路径**：`src/octop/infra/db/probe.py`
- **约行数**：50
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/db/probe.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 50 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Probe a database DSN without replacing the process pool.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `rebind.py`

- **所在目录**：`src/octop/infra/db/`
- **相对路径**：`src/octop/infra/db/rebind.py`
- **约行数**：151
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/db/rebind.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 151 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Hot-rebind the control-plane database pool during first-run setup.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `services.py`

- **所在目录**：`src/octop/infra/db/`
- **相对路径**：`src/octop/infra/db/services.py`
- **约行数**：211
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/infra/db/services.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 211 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Database services — RepoBundle and SharedServices DI container.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RepoBundle`、`SharedServices`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/db/repos/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/db/repos/__init__.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `_base.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/_base.py`
- **约行数**：82
- **类 / 模块级函数**：1 / 8
- **主要功能简介**：`src/octop/infra/db/repos/_base.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 82 行，包含 1 个类与 8 个模块级函数。模块文档写道：「Helpers shared across repo implementations.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FromRow`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `agents.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/agents.py`
- **约行数**：253
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/infra/db/repos/agents.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 253 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Agent table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AgentRow`、`AgentRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `audit.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/audit.py`
- **约行数**：114
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/db/repos/audit.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 114 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Audit log table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AuditRow`、`AuditRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `backends.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/backends.py`
- **约行数**：153
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/db/repos/backends.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 153 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Storage backend table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`BackendRow`、`BackendRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `care_push.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/care_push.py`
- **约行数**：72
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/db/repos/care_push.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 72 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Data-access layer for the proactive care push records table.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CarePushRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `channels.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/channels.py`
- **约行数**：122
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/db/repos/channels.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 122 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Channel table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ChannelRow`、`ChannelRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `connectors.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/connectors.py`
- **约行数**：292
- **类 / 模块级函数**：3 / 0
- **主要功能简介**：`src/octop/infra/db/repos/connectors.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 292 行，包含 3 个类与 0 个模块级函数。模块文档写道：「Connector database access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ConnectorRow`、`ConnectorOAuthStateRow`、`ConnectorRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `cron.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/cron.py`
- **约行数**：271
- **类 / 模块级函数**：2 / 3
- **主要功能简介**：`src/octop/infra/db/repos/cron.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 271 行，包含 2 个类与 3 个模块级函数。模块文档写道：「Cron jobs table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CronJobRow`、`CronJobRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `invites.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/invites.py`
- **约行数**：191
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/infra/db/repos/invites.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 191 行，包含 2 个类与 1 个模块级函数。模块文档写道：「User invite table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`InviteRow`、`InviteRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `knowledge.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/knowledge.py`
- **约行数**：503
- **类 / 模块级函数**：3 / 0
- **主要功能简介**：`src/octop/infra/db/repos/knowledge.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 503 行，包含 3 个类与 0 个模块级函数。模块文档写道：「Knowledge base metadata — bases and document rows.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`KnowledgeBaseRow`、`KnowledgeDocumentRow`、`KnowledgeRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `proactive_care_config.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/proactive_care_config.py`
- **约行数**：128
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/db/repos/proactive_care_config.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 128 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Data-access layer for the proactive care push configuration table.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ProactiveCareConfig`、`ProactiveCareConfigRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `providers.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/providers.py`
- **约行数**：149
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/db/repos/providers.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 149 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Provider table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ProviderRow`、`ProviderRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `published_experts.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/published_experts.py`
- **约行数**：154
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/db/repos/published_experts.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 154 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Published expert templates — one row per user-published expert.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PublishedExpertRow`、`PublishedExpertRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `secrets.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/secrets.py`
- **约行数**：42
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/db/repos/secrets.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 42 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Secrets table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SecretRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `sessions.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/sessions.py`
- **约行数**：249
- **类 / 模块级函数**：2 / 3
- **主要功能简介**：`src/octop/infra/db/repos/sessions.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 249 行，包含 2 个类与 3 个模块级函数。模块文档写道：「Sessions table — maps session_key to the currently bound thread_id.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SessionRow`、`SessionRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `settings.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/settings.py`
- **约行数**：39
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/db/repos/settings.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 39 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Settings KV store — lightweight key/value pairs backed by SQLite.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SettingsRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `skill_packages.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/skill_packages.py`
- **约行数**：133
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/db/repos/skill_packages.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 133 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Skill package metadata — one row per global skill package.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SkillPackageRow`、`SkillPackageRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `sso.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/sso.py`
- **约行数**：346
- **类 / 模块级函数**：3 / 3
- **主要功能简介**：`src/octop/infra/db/repos/sso.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 346 行，包含 3 个类与 3 个模块级函数。模块文档写道：「SSO provider and login-state table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SsoProviderRow`、`SsoLoginStateRow`、`SsoRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `thread_messages.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/thread_messages.py`
- **约行数**：318
- **类 / 模块级函数**：5 / 0
- **主要功能简介**：`src/octop/infra/db/repos/thread_messages.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 318 行，包含 5 个类与 0 个模块级函数。模块文档写道：「Projection rows used by the dashboard history API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ThreadMessageInput`、`ThreadMessageRow`、`ThreadProjectionCandidate`、`ThreadProjectionSummary`、`ThreadMessageRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `threads.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/threads.py`
- **约行数**：315
- **类 / 模块级函数**：2 / 5
- **主要功能简介**：`src/octop/infra/db/repos/threads.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 315 行，包含 2 个类与 5 个模块级函数。模块文档写道：「Threads table — conversation metadata for history listing and /resume.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ThreadRow`、`ThreadRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `trajectory_events.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/trajectory_events.py`
- **约行数**：195
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/db/repos/trajectory_events.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 195 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Append-only chat trajectory event log.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TrajectoryEventRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `usage.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/usage.py`
- **约行数**：502
- **类 / 模块级函数**：2 / 2
- **主要功能简介**：`src/octop/infra/db/repos/usage.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 502 行，包含 2 个类与 2 个模块级函数。模块文档写道：「Token usage ledger access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UsageRow`、`UsageRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `user_policies.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/user_policies.py`
- **约行数**：117
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`src/octop/infra/db/repos/user_policies.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 117 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Per-user policy rows — one named policy per user.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UserPolicyRow`、`UserPolicyRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `users.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/users.py`
- **约行数**：359
- **类 / 模块级函数**：3 / 1
- **主要功能简介**：`src/octop/infra/db/repos/users.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 359 行，包含 3 个类与 1 个模块级函数。模块文档写道：「User table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UserSsoIdentityRow`、`UserRow`、`UserRepo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `voice_providers.py`

- **所在目录**：`src/octop/infra/db/repos/`
- **相对路径**：`src/octop/infra/db/repos/voice_providers.py`
- **约行数**：141
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/db/repos/voice_providers.py` 属于控制平面数据库：连接池、迁移、RepoBundle，约 141 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Voice provider table access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`VoiceProviderRow`、`VoiceProviderRepo`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/desktop/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/desktop/`
- **相对路径**：`src/octop/infra/desktop/__init__.py`
- **约行数**：6
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/desktop/__init__.py` 属于远程桌面采集、输入注入、会话与安装，约 6 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Remote desktop domain helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `capture.py`

- **所在目录**：`src/octop/infra/desktop/`
- **相对路径**：`src/octop/infra/desktop/capture.py`
- **约行数**：267
- **类 / 模块级函数**：2 / 3
- **主要功能简介**：`src/octop/infra/desktop/capture.py` 属于远程桌面采集、输入注入、会话与安装，约 267 行，包含 2 个类与 3 个模块级函数。模块文档写道：「Screen capture and X11 display helpers for remote desktop.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CaptureFrame`、`ScreenCapture`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `input.py`

- **所在目录**：`src/octop/infra/desktop/`
- **相对路径**：`src/octop/infra/desktop/input.py`
- **约行数**：451
- **类 / 模块级函数**：1 / 14
- **主要功能简介**：`src/octop/infra/desktop/input.py` 属于远程桌面采集、输入注入、会话与安装，约 451 行，包含 1 个类与 14 个模块级函数。模块文档写道：「Input injection, coordinates, and shortcut actions for remote desktop.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`InputInjector`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `session.py`

- **所在目录**：`src/octop/infra/desktop/`
- **相对路径**：`src/octop/infra/desktop/session.py`
- **约行数**：130
- **类 / 模块级函数**：2 / 9
- **主要功能简介**：`src/octop/infra/desktop/session.py` 属于远程桌面采集、输入注入、会话与安装，约 130 行，包含 2 个类与 9 个模块级函数。模块文档写道：「In-process remote desktop session registry.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DesktopSessionLimitError`、`DesktopSession`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `setup.py`

- **所在目录**：`src/octop/infra/desktop/`
- **相对路径**：`src/octop/infra/desktop/setup.py`
- **约行数**：1070
- **类 / 模块级函数**：2 / 65
- **主要功能简介**：`src/octop/infra/desktop/setup.py` 属于远程桌面采集、输入注入、会话与安装，约 1070 行，包含 2 个类与 65 个模块级函数。模块文档写道：「Remote desktop environment setup, probes, paths, and installation.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DesktopStatus`、`_SubprocessRun`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/gateway/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/`
- **相对路径**：`src/octop/infra/gateway/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Octop Gateway — global AI interaction entry point.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `gateway.py`

- **所在目录**：`src/octop/infra/gateway/`
- **相对路径**：`src/octop/infra/gateway/gateway.py`
- **约行数**：681
- **类 / 模块级函数**：4 / 1
- **主要功能简介**：`src/octop/infra/gateway/gateway.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 681 行，包含 4 个类与 1 个模块级函数。模块文档写道：「Gateway — global AI interaction entry point.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SlashRuntimeMeta`、`ChannelRuntimeStatus`、`ChannelCreateSpec`、`Gateway`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `history_backfill.py`

- **所在目录**：`src/octop/infra/gateway/`
- **相对路径**：`src/octop/infra/gateway/history_backfill.py`
- **约行数**：67
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/gateway/history_backfill.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 67 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Bounded single-worker queue for legacy checkpoint history projection.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`HistoryBackfillQueue`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `threads.py`

- **所在目录**：`src/octop/infra/gateway/`
- **相对路径**：`src/octop/infra/gateway/threads.py`
- **约行数**：439
- **类 / 模块级函数**：1 / 2
- **主要功能简介**：`src/octop/infra/gateway/threads.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 439 行，包含 1 个类与 2 个模块级函数。模块文档写道：「ThreadRegistry — session_key → thread_id binding backed by sessions + threads tables.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ThreadRegistry`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/gateway/bot_creators/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/bot_creators/`
- **相对路径**：`src/octop/infra/gateway/bot_creators/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/bot_creators/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Bot creator subprocess scripts for QR-based channel registration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `feishu_bot_creator.py`

- **所在目录**：`src/octop/infra/gateway/bot_creators/`
- **相对路径**：`src/octop/infra/gateway/bot_creators/feishu_bot_creator.py`
- **约行数**：370
- **类 / 模块级函数**：0 / 21
- **主要功能简介**：`src/octop/infra/gateway/bot_creators/feishu_bot_creator.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 370 行，包含 0 个类与 21 个模块级函数。模块文档写道：「Feishu / Lark Open Platform — scan-to-create app via lark-oapi.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `feishu_runner.py`

- **所在目录**：`src/octop/infra/gateway/bot_creators/`
- **相对路径**：`src/octop/infra/gateway/bot_creators/feishu_runner.py`
- **约行数**：124
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/gateway/bot_creators/feishu_runner.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 124 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Feishu bot-creator subprocess runner (shared by API and CLI).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `yuanbao_bot_creator.py`

- **所在目录**：`src/octop/infra/gateway/bot_creators/`
- **相对路径**：`src/octop/infra/gateway/bot_creators/yuanbao_bot_creator.py`
- **约行数**：482
- **类 / 模块级函数**：1 / 16
- **主要功能简介**：`src/octop/infra/gateway/bot_creators/yuanbao_bot_creator.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 482 行，包含 1 个类与 16 个模块级函数。模块文档写道：「YuanBao Open Platform - Auto-bind bot via QR scan (non-interactive).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`YuanbaoBotCreator`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/gateway/channels/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/channels/`
- **相对路径**：`src/octop/infra/gateway/channels/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/channels/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Shared gateway channel helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `dingtalk_registration.py`

- **所在目录**：`src/octop/infra/gateway/channels/`
- **相对路径**：`src/octop/infra/gateway/channels/dingtalk_registration.py`
- **约行数**：58
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/gateway/channels/dingtalk_registration.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 58 行，包含 0 个类与 3 个模块级函数。模块文档写道：「DingTalk Device Flow helpers for one-click application registration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `qr_bind.py`

- **所在目录**：`src/octop/infra/gateway/channels/`
- **相对路径**：`src/octop/infra/gateway/channels/qr_bind.py`
- **约行数**：137
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/gateway/channels/qr_bind.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 137 行，包含 0 个类与 7 个模块级函数。模块文档写道：「WeCom / WeChat QR bind helpers (shared by API and CLI).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/gateway/cli/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/cli/`
- **相对路径**：`src/octop/infra/gateway/cli/__init__.py`
- **约行数**：12
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/cli/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 12 行，包含 0 个类与 0 个模块级函数。模块文档写道：「CLI in-process chat transport: connection hub + virtual IM channel.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `cli_channel.py`

- **所在目录**：`src/octop/infra/gateway/cli/`
- **相对路径**：`src/octop/infra/gateway/cli/cli_channel.py`
- **约行数**：177
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/gateway/cli/cli_channel.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 177 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Virtual IM channel that delivers CLI chat turns in-process.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CliChannel`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `turn.py`

- **所在目录**：`src/octop/infra/gateway/cli/`
- **相对路径**：`src/octop/infra/gateway/cli/turn.py`
- **约行数**：73
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/gateway/cli/turn.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 73 行，包含 0 个类与 2 个模块级函数。模块文档写道：「CLI chat turn preparation (thread binding + InboundMessage).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/gateway/hitl/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/hitl/`
- **相对路径**：`src/octop/infra/gateway/hitl/__init__.py`
- **约行数**：17
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/hitl/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 17 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Channel HITL — pending store, formatting, and resume orchestration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `coordinator.py`

- **所在目录**：`src/octop/infra/gateway/hitl/`
- **相对路径**：`src/octop/infra/gateway/hitl/coordinator.py`
- **约行数**：467
- **类 / 模块级函数**：4 / 2
- **主要功能简介**：`src/octop/infra/gateway/hitl/coordinator.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 467 行，包含 4 个类与 2 个模块级函数。模块文档写道：「Orchestrate HITL pause/resume for IM channels.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`HitlStreamContext`、`HitlSlashOutcome`、`HitlAnswerOutcome`、`HitlChannelCoordinator`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `format.py`

- **所在目录**：`src/octop/infra/gateway/hitl/`
- **相对路径**：`src/octop/infra/gateway/hitl/format.py`
- **约行数**：276
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/gateway/hitl/format.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 276 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Format HITL approval cards for IM channels.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `store.py`

- **所在目录**：`src/octop/infra/gateway/hitl/`
- **相对路径**：`src/octop/infra/gateway/hitl/store.py`
- **约行数**：190
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/gateway/hitl/store.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 190 行，包含 2 个类与 0 个模块级函数。模块文档写道：「In-memory registry of pending HITL approvals for IM channels.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`HitlPendingRecord`、`HitlPendingStore`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/gateway/media/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/media/`
- **相对路径**：`src/octop/infra/gateway/media/__init__.py`
- **约行数**：12
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/media/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 12 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Gateway media handling.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `attachment_hints.py`

- **所在目录**：`src/octop/infra/gateway/media/`
- **相对路径**：`src/octop/infra/gateway/media/attachment_hints.py`
- **约行数**：633
- **类 / 模块级函数**：1 / 27
- **主要功能简介**：`src/octop/infra/gateway/media/attachment_hints.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 633 行，包含 1 个类与 27 个模块级函数。模块文档写道：「Agent-facing inbound attachment routing — vision bytes vs tool path hints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`InboundAttachmentMeta`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `backend_files.py`

- **所在目录**：`src/octop/infra/gateway/media/`
- **相对路径**：`src/octop/infra/gateway/media/backend_files.py`
- **约行数**：566
- **类 / 模块级函数**：0 / 26
- **主要功能简介**：`src/octop/infra/gateway/media/backend_files.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 566 行，包含 0 个类与 26 个模块级函数。模块文档写道：「Binary file I/O and dashboard media preview through ``agent.workspace``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `constants.py`

- **所在目录**：`src/octop/infra/gateway/media/`
- **相对路径**：`src/octop/infra/gateway/media/constants.py`
- **约行数**：5
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/media/constants.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 5 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Gateway media directory names under the agent workspace.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `inbound_store.py`

- **所在目录**：`src/octop/infra/gateway/media/`
- **相对路径**：`src/octop/infra/gateway/media/inbound_store.py`
- **约行数**：452
- **类 / 模块级函数**：1 / 16
- **主要功能简介**：`src/octop/infra/gateway/media/inbound_store.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 452 行，包含 1 个类与 16 个模块级函数。模块文档写道：「Inbound attachment storage — all ingress via :class:`BackendWorkspace`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`InboundFile`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `ingress.py`

- **所在目录**：`src/octop/infra/gateway/media/`
- **相对路径**：`src/octop/infra/gateway/media/ingress.py`
- **约行数**：50
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/gateway/media/ingress.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 50 行，包含 1 个类与 0 个模块级函数。模块文档写道：「IM ingress media → agent workspace ``inbound/`` via :class:`BackendWorkspace`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AgentBackedMediaBackend`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `tool_media.py`

- **所在目录**：`src/octop/infra/gateway/media/`
- **相对路径**：`src/octop/infra/gateway/media/tool_media.py`
- **约行数**：792
- **类 / 模块级函数**：0 / 27
- **主要功能简介**：`src/octop/infra/gateway/media/tool_media.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 792 行，包含 0 个类与 27 个模块级函数。模块文档写道：「Tool-result media for Dashboard WebSocket streaming.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/gateway/process/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/process/`
- **相对路径**：`src/octop/infra/gateway/process/__init__.py`
- **约行数**：32
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/gateway/process/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 32 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Gateway message processing — GlobalProcessor and harness turn helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `agent_resolve.py`

- **所在目录**：`src/octop/infra/gateway/process/`
- **相对路径**：`src/octop/infra/gateway/process/agent_resolve.py`
- **约行数**：41
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/gateway/process/agent_resolve.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 41 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Resolve per-agent harness workspace handles for gateway I/O.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `harness_request.py`

- **所在目录**：`src/octop/infra/gateway/process/`
- **相对路径**：`src/octop/infra/gateway/process/harness_request.py`
- **约行数**：333
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/gateway/process/harness_request.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 333 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Assemble harness-agent stream requests and multimodal user content.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `history_projection.py`

- **所在目录**：`src/octop/infra/gateway/process/`
- **相对路径**：`src/octop/infra/gateway/process/history_projection.py`
- **约行数**：241
- **类 / 模块级函数**：1 / 10
- **主要功能简介**：`src/octop/infra/gateway/process/history_projection.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 241 行，包含 1 个类与 10 个模块级函数。模块文档写道：「Capture the current turn from harness stream state for cheap UI history.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TurnHistoryTracker`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `message_keys.py`

- **所在目录**：`src/octop/infra/gateway/process/`
- **相对路径**：`src/octop/infra/gateway/process/message_keys.py`
- **约行数**：186
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`src/octop/infra/gateway/process/message_keys.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 186 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Derive session keys and user ids from harness-gateway InboundMessage.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `processor.py`

- **所在目录**：`src/octop/infra/gateway/process/`
- **相对路径**：`src/octop/infra/gateway/process/processor.py`
- **约行数**：1538
- **类 / 模块级函数**：2 / 4
- **主要功能简介**：`src/octop/infra/gateway/process/processor.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 1538 行，包含 2 个类与 4 个模块级函数。模块文档写道：「GlobalProcessor — harness-gateway MessageProcessor + TeamProcessor.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_MessageEventSink`、`GlobalProcessor`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `response_mode.py`

- **所在目录**：`src/octop/infra/gateway/process/`
- **相对路径**：`src/octop/infra/gateway/process/response_mode.py`
- **约行数**：137
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/gateway/process/response_mode.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 137 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Channel response delivery modes for external IM transports.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `stream_project.py`

- **所在目录**：`src/octop/infra/gateway/process/`
- **相对路径**：`src/octop/infra/gateway/process/stream_project.py`
- **约行数**：287
- **类 / 模块级函数**：2 / 5
- **主要功能简介**：`src/octop/infra/gateway/process/stream_project.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 287 行，包含 2 个类与 5 个模块级函数。模块文档写道：「Project harness stream chunks into harness-gateway MessageEvent objects.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`StreamProjectionState`、`_ToolProjectionState`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `usage_record.py`

- **所在目录**：`src/octop/infra/gateway/process/`
- **相对路径**：`src/octop/infra/gateway/process/usage_record.py`
- **约行数**：277
- **类 / 模块级函数**：1 / 15
- **主要功能简介**：`src/octop/infra/gateway/process/usage_record.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 277 行，包含 1 个类与 15 个模块级函数。模块文档写道：「Collect per-call harness usage and append cache-aware turn totals.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UsageTracker`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/gateway/slash/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/__init__.py`
- **约行数**：19
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/slash/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 19 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Slash command parsing and dispatch.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `catalog.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/catalog.py`
- **约行数**：293
- **类 / 模块级函数**：1 / 5
- **主要功能简介**：`src/octop/infra/gateway/slash/catalog.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 293 行，包含 1 个类与 5 个模块级函数。模块文档写道：「Slash command catalog — metadata for handlers, API, and dashboard UI.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SlashCommandSpec`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `ctx.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/ctx.py`
- **约行数**：170
- **类 / 模块级函数**：1 / 7
- **主要功能简介**：`src/octop/infra/gateway/slash/ctx.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 170 行，包含 1 个类与 7 个模块级函数。模块文档写道：「SlashCtx and helpers for gateway slash handlers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SlashCtx`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `dispatcher.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/dispatcher.py`
- **约行数**：84
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/gateway/slash/dispatcher.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 84 行，包含 1 个类与 1 个模块级函数。模块文档写道：「SlashDispatcher: gateway handlers + harness runtime delegation.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SlashDispatcher`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `formatting.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/formatting.py`
- **约行数**：64
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/gateway/slash/formatting.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 64 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Format helpers for slash /status output.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `help.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/help.py`
- **约行数**：26
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/gateway/slash/help.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 26 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Format /help output from catalog metadata.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `parser.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/parser.py`
- **约行数**：21
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/gateway/slash/parser.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 21 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Slash command parsing that keeps filesystem paths out of the dispatcher.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `runner.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/runner.py`
- **约行数**：47
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/gateway/slash/runner.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 47 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Shared slash command execution for IM processor and dashboard chat.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `runtime_bridge.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/runtime_bridge.py`
- **约行数**：73
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/gateway/slash/runtime_bridge.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 73 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Bridge gateway SlashCtx to harness runtime slash context.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `types.py`

- **所在目录**：`src/octop/infra/gateway/slash/`
- **相对路径**：`src/octop/infra/gateway/slash/types.py`
- **约行数**：18
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/slash/types.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 18 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Gateway slash handler types.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/gateway/slash/handlers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/slash/handlers/`
- **相对路径**：`src/octop/infra/gateway/slash/handlers/__init__.py`
- **约行数**：27
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/gateway/slash/handlers/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 27 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Gateway slash command handler registry.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `composite.py`

- **所在目录**：`src/octop/infra/gateway/slash/handlers/`
- **相对路径**：`src/octop/infra/gateway/slash/handlers/composite.py`
- **约行数**：340
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/gateway/slash/handlers/composite.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 340 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Composite slash commands (gateway + harness runtime data).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `hitl.py`

- **所在目录**：`src/octop/infra/gateway/slash/handlers/`
- **相对路径**：`src/octop/infra/gateway/slash/handlers/hitl.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/gateway/slash/handlers/hitl.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 29 行，包含 0 个类与 1 个模块级函数。模块文档写道：「HITL slash command stubs (IM approval is handled in GlobalProcessor).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `platform.py`

- **所在目录**：`src/octop/infra/gateway/slash/handlers/`
- **相对路径**：`src/octop/infra/gateway/slash/handlers/platform.py`
- **约行数**：158
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/gateway/slash/handlers/platform.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 158 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Platform slash commands (help, agents, cron, connectors, token).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `session.py`

- **所在目录**：`src/octop/infra/gateway/slash/handlers/`
- **相对路径**：`src/octop/infra/gateway/slash/handlers/session.py`
- **约行数**：165
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`src/octop/infra/gateway/slash/handlers/session.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 165 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Session / thread slash commands.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/gateway/ws/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/gateway/ws/`
- **相对路径**：`src/octop/infra/gateway/ws/__init__.py`
- **约行数**：15
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/gateway/ws/__init__.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 15 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Dashboard WebSocket transport: connection hub + virtual IM channel.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `ws_channel.py`

- **所在目录**：`src/octop/infra/gateway/ws/`
- **相对路径**：`src/octop/infra/gateway/ws/ws_channel.py`
- **约行数**：237
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/gateway/ws/ws_channel.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 237 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Virtual IM channel that delivers Dashboard chat over WebSocket.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`WebSocketChannel`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `ws_hub.py`

- **所在目录**：`src/octop/infra/gateway/ws/`
- **相对路径**：`src/octop/infra/gateway/ws/ws_hub.py`
- **约行数**：163
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/gateway/ws/ws_hub.py` 属于IM 入口、线程、斜杠命令、HITL、流式投影，约 163 行，包含 1 个类与 1 个模块级函数。模块文档写道：「In-process registry of Dashboard WebSocket connections.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`WebSocketHub`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/history/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/history/`
- **相对路径**：`src/octop/infra/history/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/history/__init__.py` 属于Octop 源码，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Versioned conversation history; legacy data is never migrated on read.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `legacy.py`

- **所在目录**：`src/octop/infra/history/`
- **相对路径**：`src/octop/infra/history/legacy.py`
- **约行数**：31
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/history/legacy.py` 属于Octop 源码，约 31 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Read pinned legacy state without rewriting a projection or touching checkpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `reader.py`

- **所在目录**：`src/octop/infra/history/`
- **相对路径**：`src/octop/infra/history/reader.py`
- **约行数**：114
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/history/reader.py` 属于Octop 源码，约 114 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Read both formats without creating or repairing legacy records.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `recorder.py`

- **所在目录**：`src/octop/infra/history/`
- **相对路径**：`src/octop/infra/history/recorder.py`
- **约行数**：426
- **类 / 模块级函数**：1 / 2
- **主要功能简介**：`src/octop/infra/history/recorder.py` 属于Octop 源码，约 426 行，包含 1 个类与 2 个模块级函数。模块文档写道：「Capture state messages and stream-only content, including interrupted turns.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RecordingTracker`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `service.py`

- **所在目录**：`src/octop/infra/history/`
- **相对路径**：`src/octop/infra/history/service.py`
- **约行数**：273
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/history/service.py` 属于Octop 源码，约 273 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Turn-boundary routing and read composition for an unmigrated legacy prefix.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`HistoryArchive`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `store.py`

- **所在目录**：`src/octop/infra/history/`
- **相对路径**：`src/octop/infra/history/store.py`
- **约行数**：274
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/history/store.py` 属于Octop 源码，约 274 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Independent SQLite archive. No control-plane schema migrations or backfill.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`HistoryStore`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `trajectory.py`

- **所在目录**：`src/octop/infra/history/`
- **相对路径**：`src/octop/infra/history/trajectory.py`
- **约行数**：91
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/history/trajectory.py` 属于Octop 源码，约 91 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Trajectory compatibility view over shared history bodies and untouched legacy rows.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ArchiveTrajectoryStore`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/knowledge/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/knowledge/__init__.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Knowledge-base domain services.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `chunk.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/chunk.py`
- **约行数**：22
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/knowledge/chunk.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 22 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Fixed-size, overlapping character chunking for knowledge documents.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `citations.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/citations.py`
- **约行数**：67
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`src/octop/infra/knowledge/citations.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 67 行，包含 1 个类与 3 个模块级函数。模块文档写道：「Machine-readable knowledge citation payload embedded in tool results.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`KnowledgeCitation`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `default_open.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/default_open.py`
- **约行数**：64
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/knowledge/default_open.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 64 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Knowledge-base selection defaults for a single chat turn.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `embed.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/embed.py`
- **约行数**：66
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/knowledge/embed.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 66 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Route knowledge-base embeddings to local ONNX or a configured provider.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `files.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/files.py`
- **约行数**：38
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/knowledge/files.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 38 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Filesystem layout and safe document persistence for knowledge bases.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `gate.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/gate.py`
- **约行数**：145
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/knowledge/gate.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 145 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Capability gate for the optional knowledge-base feature.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `hint.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/hint.py`
- **约行数**：120
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`src/octop/infra/knowledge/hint.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 120 行，包含 1 个类与 3 个模块级函数。模块文档写道：「Per-turn enrichment of the search_knowledge tool description.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`KnowledgeSearchHintMiddleware`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `index.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/index.py`
- **约行数**：135
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/knowledge/index.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 135 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Per-knowledge-base SQLite sidecar vector index.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`Hit`、`KnowledgeIndex`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `jobs.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/jobs.py`
- **约行数**：87
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/knowledge/jobs.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 87 行，包含 0 个类与 6 个模块级函数。模块文档写道：「In-process document indexing jobs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `ocr.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/ocr.py`
- **约行数**：303
- **类 / 模块级函数**：3 / 16
- **主要功能简介**：`src/octop/infra/knowledge/ocr.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 303 行，包含 3 个类与 16 个模块级函数。模块文档写道：「Optional OCR backends for knowledge-base images and scanned PDFs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`OcrExtractor`、`OcrConfig`、`_RemoteOcr`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `params.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/params.py`
- **约行数**：116
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/knowledge/params.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 116 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Instance-wide knowledge indexing and retrieval parameters.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `parse.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/parse.py`
- **约行数**：292
- **类 / 模块级函数**：1 / 15
- **主要功能简介**：`src/octop/infra/knowledge/parse.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 292 行，包含 1 个类与 15 个模块级函数。模块文档写道：「Text extraction for the document types accepted by knowledge bases.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_HTMLTextParser`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `relpath.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/relpath.py`
- **约行数**：59
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/knowledge/relpath.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 59 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Relative paths for knowledge-base folders and documents.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `retrieve.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/retrieve.py`
- **约行数**：133
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/knowledge/retrieve.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 133 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Retrieve readable knowledge-base chunks for one chat turn.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `service.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/service.py`
- **约行数**：462
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/knowledge/service.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 462 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Knowledge-base ownership checks and document upload orchestration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`KnowledgeService`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `tools.py`

- **所在目录**：`src/octop/infra/knowledge/`
- **相对路径**：`src/octop/infra/knowledge/tools.py`
- **约行数**：121
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/knowledge/tools.py` 属于知识库解析、分块、嵌入、检索与索引任务，约 121 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Built-in LangChain tool for on-demand knowledge-base retrieval.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/mobile/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/mobile/__init__.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Remote Android infrastructure (probe, adb, setup).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `__main__.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/__main__.py`
- **约行数**：23
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/mobile/__main__.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 23 行，包含 0 个类与 1 个模块级函数。模块文档写道：「CLI entry: ``python -m octop.infra.mobile <config.json>`` (install.sh hook).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `adb.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/adb.py`
- **约行数**：683
- **类 / 模块级函数**：1 / 27
- **主要功能简介**：`src/octop/infra/mobile/adb.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 683 行，包含 1 个类与 27 个模块级函数。模块文档写道：「adb discovery, device listing, capture, and input helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RotationResult`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `agent_control.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/agent_control.py`
- **约行数**：57
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`src/octop/infra/mobile/agent_control.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 57 行，包含 1 个类与 3 个模块级函数。模块文档写道：「In-process binding: which adb device the agent may control.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`MobileAgentControl`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `config_probe.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/config_probe.py`
- **约行数**：57
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/mobile/config_probe.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 57 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Persist mobile capability probe results into config.json.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `docker_install.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/docker_install.py`
- **约行数**：388
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`src/octop/infra/mobile/docker_install.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 388 行，包含 0 个类与 17 个模块级函数。模块文档写道：「Automatic Docker install for the Android container backend.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `h264.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/h264.py`
- **约行数**：98
- **类 / 模块级函数**：1 / 6
- **主要功能简介**：`src/octop/infra/mobile/h264.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 98 行，包含 1 个类与 6 个模块级函数。模块文档写道：「Annex-B H.264 helpers for adb screenrecord streams.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`AnnexBSplitter`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `probe.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/probe.py`
- **约行数**：73
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`src/octop/infra/mobile/probe.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 73 行，包含 1 个类与 4 个模块级函数。模块文档写道：「Install-time host capability detection for Remote Android.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`MobileProbeResult`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `setup.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/setup.py`
- **约行数**：260
- **类 / 模块级函数**：1 / 8
- **主要功能简介**：`src/octop/infra/mobile/setup.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 260 行，包含 1 个类与 8 个模块级函数。模块文档写道：「Runtime mobile status and optional container install.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`MobileStatus`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `tools.py`

- **所在目录**：`src/octop/infra/mobile/`
- **相对路径**：`src/octop/infra/mobile/tools.py`
- **约行数**：254
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/mobile/tools.py` 属于远程手机 ADB、H264 流、shell 与绑定，约 254 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Built-in LangChain tools for Remote Android (adb automation).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/proactive/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/proactive/`
- **相对路径**：`src/octop/infra/proactive/__init__.py`
- **约行数**：19
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/proactive/__init__.py` 属于Octop 源码，约 19 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Public interface for the proactive module.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `picker.py`

- **所在目录**：`src/octop/infra/proactive/`
- **相对路径**：`src/octop/infra/proactive/picker.py`
- **约行数**：218
- **类 / 模块级函数**：2 / 3
- **主要功能简介**：`src/octop/infra/proactive/picker.py` 属于Octop 源码，约 218 行，包含 2 个类与 3 个模块级函数。模块文档写道：「Episode picker — selects the most care-worthy events from the user's episode memory.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PickResult`、`EpisodePicker`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `scheduler.py`

- **所在目录**：`src/octop/infra/proactive/`
- **相对路径**：`src/octop/infra/proactive/scheduler.py`
- **约行数**：397
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`src/octop/infra/proactive/scheduler.py` 属于Octop 源码，约 397 行，包含 1 个类与 4 个模块级函数。模块文档写道：「ProactiveCareScheduler — proactive care push random scheduler.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ProactiveCareScheduler`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `service.py`

- **所在目录**：`src/octop/infra/proactive/`
- **相对路径**：`src/octop/infra/proactive/service.py`
- **约行数**：283
- **类 / 模块级函数**：1 / 2
- **主要功能简介**：`src/octop/infra/proactive/service.py` 属于Octop 源码，约 283 行，包含 1 个类与 2 个模块级函数。模块文档写道：「ProactiveCareService — proactive care push service.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ProactiveCareService`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/providers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `codex_apply.py`

- **所在目录**：`src/octop/infra/providers/`
- **相对路径**：`src/octop/infra/providers/codex_apply.py`
- **约行数**：68
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/providers/codex_apply.py` 属于Octop 源码，约 68 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Apply Codex OAuth credentials to the Octop provider store.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `codex_oauth.py`

- **所在目录**：`src/octop/infra/providers/`
- **相对路径**：`src/octop/infra/providers/codex_oauth.py`
- **约行数**：285
- **类 / 模块级函数**：3 / 15
- **主要功能简介**：`src/octop/infra/providers/codex_oauth.py` 属于Octop 源码，约 285 行，包含 3 个类与 15 个模块级函数。模块文档写道：「OpenAI Codex (ChatGPT) OAuth — device code flow.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CodexOAuthCredentials`、`CodexOAuthRefreshError`、`DeviceCodeInfo`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/setup/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/setup/`
- **相对路径**：`src/octop/infra/setup/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/setup/__init__.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「First-run setup wizard — password file and in-memory session tokens.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `password_file.py`

- **所在目录**：`src/octop/infra/setup/`
- **相对路径**：`src/octop/infra/setup/password_file.py`
- **约行数**：82
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/setup/password_file.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 82 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Setup wizard password file at ``<home>/octop-login.txt``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `self_update.py`

- **所在目录**：`src/octop/infra/setup/`
- **相对路径**：`src/octop/infra/setup/self_update.py`
- **约行数**：733
- **类 / 模块级函数**：2 / 26
- **主要功能简介**：`src/octop/infra/setup/self_update.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 733 行，包含 2 个类与 26 个模块级函数。模块文档写道：「Self-upgrade helpers shared by CLI and HTTP update API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UpgradeResult`、`PyPIInfo`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `service.py`

- **所在目录**：`src/octop/infra/setup/`
- **相对路径**：`src/octop/infra/setup/service.py`
- **约行数**：926
- **类 / 模块级函数**：2 / 49
- **主要功能简介**：`src/octop/infra/setup/service.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 926 行，包含 2 个类与 49 个模块级函数。模块文档写道：「System service helpers for systemd (Linux) and launchd (macOS).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ServiceRuntime`、`ServiceStatus`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `wizard_tokens.py`

- **所在目录**：`src/octop/infra/setup/`
- **相对路径**：`src/octop/infra/setup/wizard_tokens.py`
- **约行数**：68
- **类 / 模块级函数**：3 / 0
- **主要功能简介**：`src/octop/infra/setup/wizard_tokens.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 68 行，包含 3 个类与 0 个模块级函数。模块文档写道：「In-memory wizard session tokens (post password verification).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`RateLimited`、`_TokenEntry`、`WizardTokenStore`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/setup/tls/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/__init__.py`
- **约行数**：8
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/setup/tls/__init__.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 8 行，包含 0 个类与 0 个模块级函数。模块文档写道：「TLS / Let's Encrypt certificate management.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `acme_issue.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/acme_issue.py`
- **约行数**：136
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/setup/tls/acme_issue.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 136 行，包含 0 个类与 3 个模块级函数。模块文档写道：「ACME v2 HTTP-01 certificate issuance (Let's Encrypt).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `challenge.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/challenge.py`
- **约行数**：29
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/setup/tls/challenge.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 29 行，包含 1 个类与 0 个模块级函数。模块文档写道：「In-memory HTTP-01 challenge responses for ACME.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ChallengeStore`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `http_companion.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/http_companion.py`
- **约行数**：44
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/setup/tls/http_companion.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 44 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Minimal HTTP listener on port 80 — ACME challenges + redirect to HTTPS.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `listeners.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/listeners.py`
- **约行数**：63
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/setup/tls/listeners.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 63 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Compute which HTTP/HTTPS ports ``octop run`` should bind.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ListenPlan`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `manager.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/manager.py`
- **约行数**：258
- **类 / 模块级函数**：3 / 1
- **主要功能简介**：`src/octop/infra/setup/tls/manager.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 258 行，包含 3 个类与 1 个模块级函数。模块文档写道：「Orchestrate TLS preflight, ACME issuance, and config updates.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TlsTaskState`、`TlsTask`、`TlsManager`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `modes.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/modes.py`
- **约行数**：55
- **类 / 模块级函数**：1 / 5
- **主要功能简介**：`src/octop/infra/setup/tls/modes.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 55 行，包含 1 个类与 5 个模块级函数。模块文档写道：「TLS issuance mode helpers (first issue vs renewal).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TlsIssueMode`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `preflight.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/preflight.py`
- **约行数**：216
- **类 / 模块级函数**：2 / 4
- **主要功能简介**：`src/octop/infra/setup/tls/preflight.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 216 行，包含 2 个类与 4 个模块级函数。模块文档写道：「Pre-flight checks before Let's Encrypt issuance.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PreflightCheck`、`PreflightResult`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `renewal.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/renewal.py`
- **约行数**：89
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/setup/tls/renewal.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 89 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Automatic TLS certificate renewal scheduling.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `store.py`

- **所在目录**：`src/octop/infra/setup/tls/`
- **相对路径**：`src/octop/infra/setup/tls/store.py`
- **约行数**：96
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/setup/tls/store.py` 属于首次向导、系统服务、TLS / Let's Encrypt，约 96 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Persist TLS certificates and merge config.json.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/skills/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/__init__.py`
- **约行数**：42
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/skills/__init__.py` 属于Octop 源码，约 42 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Skills domain: package validation, global packages, URL import, SkillHub market.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `install.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/install.py`
- **约行数**：222
- **类 / 模块级函数**：2 / 8
- **主要功能简介**：`src/octop/infra/skills/install.py` 属于Octop 源码，约 222 行，包含 2 个类与 8 个模块级函数。模块文档写道：「Shared skill install pipeline for agent workspace and global packages.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SkillAlreadyExistsError`、`SkillInstallTarget`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `presentation.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/presentation.py`
- **约行数**：96
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/skills/presentation.py` 属于Octop 源码，约 96 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Skill presentation metadata shared by catalogs and HTTP adapters.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `skill_package_from_skillhub.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/skill_package_from_skillhub.py`
- **约行数**：84
- **类 / 模块级函数**：1 / 1
- **主要功能简介**：`src/octop/infra/skills/skill_package_from_skillhub.py` 属于Octop 源码，约 84 行，包含 1 个类与 1 个模块级函数。模块文档写道：「Materialize a SkillHub skillset into a global skill package.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`CreatePackageFromSkillHubResult`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `skill_package_store.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/skill_package_store.py`
- **约行数**：195
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`src/octop/infra/skills/skill_package_store.py` 属于Octop 源码，约 195 行，包含 1 个类与 3 个模块级函数。模块文档写道：「Disk-backed content storage for global skill packages.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SkillPackageStore`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `skill_packages.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/skill_packages.py`
- **约行数**：210
- **类 / 模块级函数**：3 / 7
- **主要功能简介**：`src/octop/infra/skills/skill_packages.py` 属于Octop 源码，约 210 行，包含 3 个类与 7 个模块级函数。模块文档写道：「Source-neutral skill package validation and workspace path normalization.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SkillPackageError`、`SkillPackageTooLarge`、`ResolvedSkillPackage`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `skill_transfer.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/skill_transfer.py`
- **约行数**：172
- **类 / 模块级函数**：2 / 7
- **主要功能简介**：`src/octop/infra/skills/skill_transfer.py` 属于Octop 源码，约 172 行，包含 2 个类与 7 个模块级函数。模块文档写道：「Copy skills between global packages and agent workspaces.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SkillTransferConflict`、`SkillTransferNotFound`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `skillhub_market.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/skillhub_market.py`
- **约行数**：441
- **类 / 模块级函数**：4 / 13
- **主要功能简介**：`src/octop/infra/skills/skillhub_market.py` 属于Octop 源码，约 441 行，包含 4 个类与 13 个模块级函数。模块文档写道：「Direct HTTP client for Tencent SkillHub marketplace operations.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`SkillHubMarketError`、`SkillHubMarketTimeout`、`SkillHubPackageError`、`SkillHubPackageTooLarge`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `skills_hub.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/skills_hub.py`
- **约行数**：1494
- **类 / 模块级函数**：2 / 68
- **主要功能简介**：`src/octop/infra/skills/skills_hub.py` 属于Octop 源码，约 1494 行，包含 2 个类与 68 个模块级函数。模块文档写道：「Skills hub URL fetch and bundle normalization for per-agent workspace import.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`HubSkillResult`、`BundleResolveResult`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `workspace_catalog.py`

- **所在目录**：`src/octop/infra/skills/`
- **相对路径**：`src/octop/infra/skills/workspace_catalog.py`
- **约行数**：177
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`src/octop/infra/skills/workspace_catalog.py` 属于Octop 源码，约 177 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Symlink-tolerant workspace skill discovery for Octop.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/trajectory/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/trajectory/`
- **相对路径**：`src/octop/infra/trajectory/__init__.py`
- **约行数**：7
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/trajectory/__init__.py` 属于Octop 源码，约 7 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Harness stream → trajectory event projection.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `live.py`

- **所在目录**：`src/octop/infra/trajectory/`
- **相对路径**：`src/octop/infra/trajectory/live.py`
- **约行数**：32
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/trajectory/live.py` 属于Octop 源码，约 32 行，包含 1 个类与 0 个模块级函数。模块文档写道：「In-process pub/sub for trajectory live SSE fan-out.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TrajectoryLiveBus`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `metrics.py`

- **所在目录**：`src/octop/infra/trajectory/`
- **相对路径**：`src/octop/infra/trajectory/metrics.py`
- **约行数**：109
- **类 / 模块级函数**：1 / 6
- **主要功能简介**：`src/octop/infra/trajectory/metrics.py` 属于Octop 源码，约 109 行，包含 1 个类与 6 个模块级函数。模块文档写道：「Aggregate session metrics from trajectory events.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TrajectoryMetrics`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `projector.py`

- **所在目录**：`src/octop/infra/trajectory/`
- **相对路径**：`src/octop/infra/trajectory/projector.py`
- **约行数**：321
- **类 / 模块级函数**：1 / 10
- **主要功能简介**：`src/octop/infra/trajectory/projector.py` 属于Octop 源码，约 321 行，包含 1 个类与 10 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_Common`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `service.py`

- **所在目录**：`src/octop/infra/trajectory/`
- **相对路径**：`src/octop/infra/trajectory/service.py`
- **约行数**：410
- **类 / 模块级函数**：3 / 9
- **主要功能简介**：`src/octop/infra/trajectory/service.py` 属于Octop 源码，约 410 行，包含 3 个类与 9 个模块级函数。模块文档写道：「Orchestrate project → store → live publish. Observe never fails the caller.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_ThreadInFlight`、`_CachedMetrics`、`TrajectoryService`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `settings.py`

- **所在目录**：`src/octop/infra/trajectory/`
- **相对路径**：`src/octop/infra/trajectory/settings.py`
- **约行数**：80
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/trajectory/settings.py` 属于Octop 源码，约 80 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Per-agent trajectory switch and persist-size limits.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `store.py`

- **所在目录**：`src/octop/infra/trajectory/`
- **相对路径**：`src/octop/infra/trajectory/store.py`
- **约行数**：55
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/trajectory/store.py` 属于Octop 源码，约 55 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Append-only trajectory ledger — thin wrapper around TrajectoryEventRepo.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TrajectoryStore`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `turn_context.py`

- **所在目录**：`src/octop/infra/trajectory/`
- **相对路径**：`src/octop/infra/trajectory/turn_context.py`
- **约行数**：129
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/trajectory/turn_context.py` 属于Octop 源码，约 129 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Synthesize SYSTEM / CONTEXT chunks from harness injection sources of truth.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `types.py`

- **所在目录**：`src/octop/infra/trajectory/`
- **相对路径**：`src/octop/infra/trajectory/types.py`
- **约行数**：30
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/trajectory/types.py` 属于Octop 源码，约 30 行，包含 1 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TrajectoryEvent`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/usage/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/usage/`
- **相对路径**：`src/octop/infra/usage/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/usage/__init__.py` 属于Octop 源码，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Token usage package — export helpers live beside ledger access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `xlsx_export.py`

- **所在目录**：`src/octop/infra/usage/`
- **相对路径**：`src/octop/infra/usage/xlsx_export.py`
- **约行数**：564
- **类 / 模块级函数**：0 / 19
- **主要功能简介**：`src/octop/infra/usage/xlsx_export.py` 属于Octop 源码，约 564 行，包含 0 个类与 19 个模块级函数。模块文档写道：「Build localized Excel workbooks for token usage export.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/users/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/__init__.py`
- **约行数**：10
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/users/__init__.py` 属于用户、角色、口令哈希、UserManager，约 10 行，包含 0 个类与 0 个模块级函数。模块文档写道：「User domain.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `acting.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/acting.py`
- **约行数**：43
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/users/acting.py` 属于用户、角色、口令哈希、UserManager，约 43 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Resolve which user id a local CLI / API ``as_user`` action targets.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `email.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/email.py`
- **约行数**：35
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/users/email.py` 属于用户、角色、口令哈希、UserManager，约 35 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Email normalization and light validation for local users.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `identity.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/identity.py`
- **约行数**：30
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/users/identity.py` 属于用户、角色、口令哈希、UserManager，约 30 行，包含 2 个类与 0 个模块级函数。模块文档写道：「User identity primitives.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`Role`、`User`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `invites.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/invites.py`
- **约行数**：224
- **类 / 模块级函数**：2 / 4
- **主要功能简介**：`src/octop/infra/users/invites.py` 属于用户、角色、口令哈希、UserManager，约 224 行，包含 2 个类与 4 个模块级函数。模块文档写道：「One-time user invite codes — create, validate, redeem.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`InviteRateLimiter`、`InviteService`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `manager.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/manager.py`
- **约行数**：738
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`src/octop/infra/users/manager.py` 属于用户、角色、口令哈希、UserManager，约 738 行，包含 1 个类与 4 个模块级函数。模块文档写道：「UserManager — process-singleton user lifecycle.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UserManager`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `password.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/password.py`
- **约行数**：71
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/users/password.py` 属于用户、角色、口令哈希、UserManager，约 71 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Argon2id password hashing and change-password policy checks.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `permissions.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/permissions.py`
- **约行数**：256
- **类 / 模块级函数**：2 / 4
- **主要功能简介**：`src/octop/infra/users/permissions.py` 属于用户、角色、口令哈希、UserManager，约 256 行，包含 2 个类与 4 个模块级函数。模块文档写道：「Module-level permission catalog and checks.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PermissionUser`、`PermissionDef`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `preferences.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/preferences.py`
- **约行数**：155
- **类 / 模块级函数**：2 / 10
- **主要功能简介**：`src/octop/infra/users/preferences.py` 属于用户、角色、口令哈希、UserManager，约 155 行，包含 2 个类与 10 个模块级函数。模块文档写道：「Per-user JSON preferences (models, remote-browser bookmarks, etc.).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ModelReasoningPreference`、`RemoteBrowserBookmark`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `resource_policy.py`

- **所在目录**：`src/octop/infra/users/`
- **相对路径**：`src/octop/infra/users/resource_policy.py`
- **约行数**：159
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`src/octop/infra/users/resource_policy.py` 属于用户、角色、口令哈希、UserManager，约 159 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Per-user named policies: workspace root, token quota, and future rows.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `src/octop/infra/utils/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/utils/__init__.py` 属于Octop 源码，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `browser_media.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/browser_media.py`
- **约行数**：155
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/infra/utils/browser_media.py` 属于Octop 源码，约 155 行，包含 0 个类与 12 个模块级函数。模块文档写道：「harness-browser media paths aligned with IM ``outbound/`` layout.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `bwrap.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/bwrap.py`
- **约行数**：216
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`src/octop/infra/utils/bwrap.py` 属于Octop 源码，约 216 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Best-effort bubblewrap (``bwrap``) provisioning for scoped ``root_dir``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `doc_edit.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/doc_edit.py`
- **约行数**：336
- **类 / 模块级函数**：3 / 18
- **主要功能简介**：`src/octop/infra/utils/doc_edit.py` 属于Octop 源码，约 336 行，包含 3 个类与 18 个模块级函数。模块文档写道：「Editable-document converters (binary document <-> Markdown) for the workspace.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`DocConverter`、`DocConverterProtocol`、`DocxDocConverter`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `docker_env.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/docker_env.py`
- **约行数**：412
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/infra/utils/docker_env.py` 属于Octop 源码，约 412 行，包含 0 个类与 12 个模块级函数。模块文档写道：「Detect / best-effort provision Docker Engine for agent sandbox backends.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `env_file.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/env_file.py`
- **约行数**：147
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`src/octop/infra/utils/env_file.py` 属于Octop 源码，约 147 行，包含 0 个类与 12 个模块级函数。模块文档写道：「Persisted environment variables at ``~/.octop/env`` (dotenv format).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `frontmatter.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/frontmatter.py`
- **约行数**：93
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/utils/frontmatter.py` 属于Octop 源码，约 93 行，包含 0 个类与 3 个模块级函数。模块文档写道：「YAML frontmatter helpers for markdown workspace files.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `host_dirs.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/host_dirs.py`
- **约行数**：374
- **类 / 模块级函数**：0 / 20
- **主要功能简介**：`src/octop/infra/utils/host_dirs.py` 属于Octop 源码，约 374 行，包含 0 个类与 20 个模块级函数。模块文档写道：「Host filesystem directory helpers for dashboard root_dir pickers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `json_file.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/json_file.py`
- **约行数**：110
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`src/octop/infra/utils/json_file.py` 属于Octop 源码，约 110 行，包含 1 个类与 3 个模块级函数。模块文档写道：「Safe read/write helpers for the small JSON config files under ``OCTOP_HOME``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`JsonFileCorruptError`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `llm_text.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/llm_text.py`
- **约行数**：91
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/utils/llm_text.py` 属于Octop 源码，约 91 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Shared helpers for one-shot LangChain LLM calls.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `locale.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/locale.py`
- **约行数**：84
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`src/octop/infra/utils/locale.py` 属于Octop 源码，约 84 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Shared locale normalization for API, slash commands, and dashboard.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `ollama_manager.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/ollama_manager.py`
- **约行数**：223
- **类 / 模块级函数**：2 / 9
- **主要功能简介**：`src/octop/infra/utils/ollama_manager.py` 属于Octop 源码，约 223 行，包含 2 个类与 9 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`OllamaModelInfo`、`OllamaModelManager`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `paths.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/paths.py`
- **约行数**：151
- **类 / 模块级函数**：1 / 0
- **主要功能简介**：`src/octop/infra/utils/paths.py` 属于Octop 源码，约 151 行，包含 1 个类与 0 个模块级函数。模块文档写道：「Filesystem layout for ``~/.octop/``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PathLayout`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `posix_compat.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/posix_compat.py`
- **约行数**：119
- **类 / 模块级函数**：0 / 14
- **主要功能简介**：`src/octop/infra/utils/posix_compat.py` 属于Octop 源码，约 119 行，包含 0 个类与 14 个模块级函数。模块文档写道：「POSIX-only stdlib helpers for cross-platform mypy (CI runs on Windows).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `runtime_packages.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/runtime_packages.py`
- **约行数**：285
- **类 / 模块级函数**：1 / 14
- **主要功能简介**：`src/octop/infra/utils/runtime_packages.py` 属于Octop 源码，约 285 行，包含 1 个类与 14 个模块级函数。模块文档写道：「Install optional Python packages into the active interpreter at runtime.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`PackageInstallSpec`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `search_probe.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/search_probe.py`
- **约行数**：222
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`src/octop/infra/utils/search_probe.py` 属于Octop 源码，约 222 行，包含 0 个类与 11 个模块级函数。模块文档写道：「Probe web-search provider API keys via direct HTTP (no env mutation).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `ssl_errors.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/ssl_errors.py`
- **约行数**：19
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/utils/ssl_errors.py` 属于Octop 源码，约 19 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Detect TLS/SSL failures in exception / stderr text for actionable UX hints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `ssrf_guard.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/ssrf_guard.py`
- **约行数**：186
- **类 / 模块级函数**：3 / 10
- **主要功能简介**：`src/octop/infra/utils/ssrf_guard.py` 属于Octop 源码，约 186 行，包含 3 个类与 10 个模块级函数。模块文档写道：「Outbound HTTPS URL validation — mitigates SSRF (CWE-918).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`UnsafeOutboundUrl`、`_PinnedNetworkBackend`、`PinnedIPTransport`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `subprocess_io.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/subprocess_io.py`
- **约行数**：50
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`src/octop/infra/utils/subprocess_io.py` 属于Octop 源码，约 50 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Cross-platform non-blocking reads from subprocess stdout pipes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `subprocess_io_win.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/subprocess_io_win.py`
- **约行数**：27
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/utils/subprocess_io_win.py` 属于Octop 源码，约 27 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Windows-only non-blocking pipe reads (typed separately for mypy).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `tencent_sign.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/tencent_sign.py`
- **约行数**：61
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/utils/tencent_sign.py` 属于Octop 源码，约 61 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Tencent Cloud TC3-HMAC-SHA256 request signing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `ulid.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/ulid.py`
- **约行数**：45
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`src/octop/infra/utils/ulid.py` 属于Octop 源码，约 45 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Tiny ULID generator (Crockford base32, monotonic within process).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `url.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/url.py`
- **约行数**：15
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`src/octop/infra/utils/url.py` 属于Octop 源码，约 15 行，包含 0 个类与 1 个模块级函数。模块文档写道：「URL helpers shared across infra and API layers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `utf8_text.py`

- **所在目录**：`src/octop/infra/utils/`
- **相对路径**：`src/octop/infra/utils/utf8_text.py`
- **约行数**：157
- **类 / 模块级函数**：1 / 6
- **主要功能简介**：`src/octop/infra/utils/utf8_text.py` 属于Octop 源码，约 157 行，包含 1 个类与 6 个模块级函数。模块文档写道：「Helpers for coercing third-party package bytes into valid UTF-8 text.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`InvalidSkillManifestEncodingError`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `src/octop/infra/voice/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`src/octop/infra/voice/`
- **相对路径**：`src/octop/infra/voice/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`src/octop/infra/voice/__init__.py` 属于Octop 源码，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Voice STT/TTS provider orchestration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `adapters.py`

- **所在目录**：`src/octop/infra/voice/`
- **相对路径**：`src/octop/infra/voice/adapters.py`
- **约行数**：572
- **类 / 模块级函数**：2 / 22
- **主要功能简介**：`src/octop/infra/voice/adapters.py` 属于Octop 源码，约 572 行，包含 2 个类与 22 个模块级函数。模块文档写道：「Voice adapter implementations.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`STTResult`、`BrowserOnlyError`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `manager.py`

- **所在目录**：`src/octop/infra/voice/`
- **相对路径**：`src/octop/infra/voice/manager.py`
- **约行数**：223
- **类 / 模块级函数**：2 / 0
- **主要功能简介**：`src/octop/infra/voice/manager.py` 属于Octop 源码，约 223 行，包含 2 个类与 0 个模块级函数。模块文档写道：「Resolve active voice providers and dispatch STT/TTS calls.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ResolvedVoiceProvider`、`VoiceManager`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `presets.py`

- **所在目录**：`src/octop/infra/voice/`
- **相对路径**：`src/octop/infra/voice/presets.py`
- **约行数**：74
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`src/octop/infra/voice/presets.py` 属于Octop 源码，约 74 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Built-in voice provider presets.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


## 2.2 测试 `tests`


### 目录 `tests/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`tests/`
- **相对路径**：`tests/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`tests/__init__.py` 属于Octop 源码，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `conftest.py`

- **所在目录**：`tests/`
- **相对路径**：`tests/conftest.py`
- **约行数**：102
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/conftest.py` 属于Octop 源码，约 102 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Shared pytest fixtures.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/fixtures/plugins/echo-tool/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `main.py`

- **所在目录**：`tests/fixtures/plugins/echo-tool/`
- **相对路径**：`tests/fixtures/plugins/echo-tool/main.py`
- **约行数**：25
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/fixtures/plugins/echo-tool/main.py` 属于Octop 源码，约 25 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Echo tool plugin for tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/integration/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`tests/integration/__init__.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `conftest.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/conftest.py`
- **约行数**：247
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`tests/integration/conftest.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 247 行，包含 0 个类与 17 个模块级函数。模块文档写道：「Shared fixtures for integration tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_acp_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_acp_api.py`
- **约行数**：73
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/integration/test_acp_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 73 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/integration/test_acp_api.py — ACP runner configuration API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_admin_providers.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_admin_providers.py`
- **约行数**：61
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_admin_providers.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 61 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/integration/test_admin_providers.py — admin /api/admin/providers CRUD.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agent_avatars.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_agent_avatars.py`
- **约行数**：61
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_agent_avatars.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 61 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Expert avatar upload HTTP surface.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agent_skill_packages.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_agent_skill_packages.py`
- **约行数**：153
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/integration/test_agent_skill_packages.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 153 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Integration coverage for agent-mounted global skill packages.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agents_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_agents_api.py`
- **约行数**：155
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/integration/test_agents_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 155 行，包含 0 个类与 8 个模块级函数。模块文档写道：「tests/integration/test_agents_api.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agents_list_scope.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_agents_list_scope.py`
- **约行数**：130
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/integration/test_agents_list_scope.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 130 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Agent list user isolation.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agents_shared.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_agents_shared.py`
- **约行数**：247
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/integration/test_agents_shared.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 247 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Shared agent list and ownership behavior.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_auth_flow.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_auth_flow.py`
- **约行数**：125
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/integration/test_auth_flow.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 125 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/integration/test_auth_flow.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_auth_oauth.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_auth_oauth.py`
- **约行数**：125
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/integration/test_auth_oauth.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 125 行，包含 0 个类与 4 个模块级函数。模块文档写道：「OAuth SSO HTTP route integration tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_auth_oidc.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_auth_oidc.py`
- **约行数**：135
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_auth_oidc.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 135 行，包含 0 个类与 3 个模块级函数。模块文档写道：「OIDC HTTP route integration tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_boundary_authz.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_boundary_authz.py`
- **约行数**：130
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`tests/integration/test_boundary_authz.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 130 行，包含 0 个类与 12 个模块级函数。模块文档写道：「tests/integration/test_boundary_authz.py — cross-user / cross-scope authz.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_browser_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_browser_api.py`
- **约行数**：241
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`tests/integration/test_browser_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 241 行，包含 0 个类与 12 个模块级函数。模块文档写道：「tests/integration/test_browser_api.py — remote browser endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_browser_record_replay_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_browser_record_replay_api.py`
- **约行数**：233
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/integration/test_browser_record_replay_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 233 行，包含 0 个类与 9 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_bwrap_jail.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_bwrap_jail.py`
- **约行数**：289
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/integration/test_bwrap_jail.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 289 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Scoped ``root_dir`` backend alignment and bubblewrap jail tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_captcha_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_captcha_api.py`
- **约行数**：351
- **类 / 模块级函数**：1 / 15
- **主要功能简介**：`tests/integration/test_captcha_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 351 行，包含 1 个类与 15 个模块级函数。模块文档写道：「HTTP contracts for login captcha and admin settings.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_Siteverify`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_channel_probe_draft.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_channel_probe_draft.py`
- **约行数**：31
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/integration/test_channel_probe_draft.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 31 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Integration tests for draft channel config probe.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_channel_test_endpoint.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_channel_test_endpoint.py`
- **约行数**：46
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_channel_test_endpoint.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 46 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/integration/test_channel_test_endpoint.py — channel probe.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_channels_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_channels_api.py`
- **约行数**：129
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/integration/test_channels_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 129 行，包含 0 个类与 6 个模块级函数。模块文档写道：「tests/integration/test_channels_api.py — channels CRUD.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_chat_ws.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_chat_ws.py`
- **约行数**：512
- **类 / 模块级函数**：0 / 25
- **主要功能简介**：`tests/integration/test_chat_ws.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 512 行，包含 0 个类与 25 个模块级函数。模块文档写道：「tests/integration/test_chat_ws.py — dashboard WebSocket chat + thread CRUD.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_connectors_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_connectors_api.py`
- **约行数**：668
- **类 / 模块级函数**：0 / 25
- **主要功能简介**：`tests/integration/test_connectors_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 668 行，包含 0 个类与 25 个模块级函数。模块文档写道：「Integration tests for connector APIs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cron_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_cron_api.py`
- **约行数**：267
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`tests/integration/test_cron_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 267 行，包含 0 个类与 15 个模块级函数。模块文档写道：「tests/integration/test_cron_api.py — cron CRUD + run-now + trigger validation.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cron_shared_agent.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_cron_shared_agent.py`
- **约行数**：85
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/integration/test_cron_shared_agent.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 85 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Cron jobs on a shared expert stay owner-only.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_dashboard_serve.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_dashboard_serve.py`
- **约行数**：72
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/integration/test_dashboard_serve.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 72 行，包含 0 个类与 5 个模块级函数。模块文档写道：「tests/integration/test_dashboard_serve.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_e2e_golden_path.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_e2e_golden_path.py`
- **约行数**：206
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/integration/test_e2e_golden_path.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 206 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/integration/test_e2e_golden_path.py — full user journey.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_envs_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_envs_api.py`
- **约行数**：91
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/integration/test_envs_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 91 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Integration tests for /api/envs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_experts_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_experts_api.py`
- **约行数**：442
- **类 / 模块级函数**：0 / 21
- **主要功能简介**：`tests/integration/test_experts_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 442 行，包含 0 个类与 21 个模块级函数。模块文档写道：「tests/integration/test_experts_api.py — expert catalog endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_filesystem_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_filesystem_api.py`
- **约行数**：363
- **类 / 模块级函数**：0 / 16
- **主要功能简介**：`tests/integration/test_filesystem_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 363 行，包含 0 个类与 16 个模块级函数。模块文档写道：「Integration tests for /api/filesystem (host root_dir pickers).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_invites_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_invites_api.py`
- **约行数**：129
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_invites_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 129 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Integration tests for one-time user invites.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_knowledge_bases_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_knowledge_bases_api.py`
- **约行数**：51
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/integration/test_knowledge_bases_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 51 行，包含 0 个类与 2 个模块级函数。模块文档写道：「HTTP ACL for knowledge-base instance settings vs page CRUD.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_media_generation_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_media_generation_api.py`
- **约行数**：73
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/integration/test_media_generation_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 73 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Integration tests for the admin media-generation settings API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_memory_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_memory_api.py`
- **约行数**：471
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`tests/integration/test_memory_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 471 行，包含 0 个类与 17 个模块级函数。模块文档写道：「tests/integration/test_memory_api.py — dashboard memory router.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_notifications_ws.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_notifications_ws.py`
- **约行数**：70
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/integration/test_notifications_ws.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 70 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Dashboard notification WebSocket for text-type pushes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_onnx_models_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_onnx_models_api.py`
- **约行数**：145
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/integration/test_onnx_models_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 145 行，包含 0 个类与 7 个模块级函数。模块文档写道：「HTTP behaviour for the local ONNX embedding service admin endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_personas_admin_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_personas_admin_api.py`
- **约行数**：138
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`tests/integration/test_personas_admin_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 138 行，包含 0 个类与 11 个模块级函数。模块文档写道：「tests/integration/test_personas_admin_api.py — MBTI preview + admin routers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_plugin_tool_disable.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_plugin_tool_disable.py`
- **约行数**：232
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/integration/test_plugin_tool_disable.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 232 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Disable plugin tools via Admin Plugins API and Experts tool-settings.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_plugin_upload.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_plugin_upload.py`
- **约行数**：80
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/integration/test_plugin_upload.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 80 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Integration tests for uploading a plugin ZIP from the dashboard.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_postgresql_checkpoint_history.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_postgresql_checkpoint_history.py`
- **约行数**：63
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/integration/test_postgresql_checkpoint_history.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 63 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Check the native PG history path without opening a SQLite connection.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_postgresql_control_plane.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_postgresql_control_plane.py`
- **约行数**：343
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/integration/test_postgresql_control_plane.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 343 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Gated PostgreSQL control-plane integration tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_preferences_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_preferences_api.py`
- **约行数**：89
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/integration/test_preferences_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 89 行，包含 0 个类与 6 个模块级函数。模块文档写道：「tests/integration/test_preferences_api.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_fetch_models.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_provider_fetch_models.py`
- **约行数**：68
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_provider_fetch_models.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 68 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Integration tests for admin provider fetch-models endpoint.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_test_draft.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_provider_test_draft.py`
- **约行数**：70
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_provider_test_draft.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 70 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Integration tests for admin provider test-draft endpoint.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_test_endpoint.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_provider_test_endpoint.py`
- **约行数**：75
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/integration/test_provider_test_endpoint.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 75 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/integration/test_provider_test_endpoint.py — provider probe.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_providers_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_providers_api.py`
- **约行数**：93
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/integration/test_providers_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 93 行，包含 0 个类与 5 个模块级函数。模块文档写道：「tests/integration/test_providers_api.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_published_experts.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_published_experts.py`
- **约行数**：409
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/integration/test_published_experts.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 409 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Published expert HTTP API integration coverage.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_scalar.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_scalar.py`
- **约行数**：33
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/integration/test_scalar.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 33 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Integration tests for the /api/docs Scalar endpoint.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_search_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_search_api.py`
- **约行数**：52
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_search_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 52 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Integration tests for POST /api/search/{provider_id}/test.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_security_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_security_api.py`
- **约行数**：78
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/integration/test_security_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 78 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Integration tests for /api/admin/security.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_server_lifecycle.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_server_lifecycle.py`
- **约行数**：66
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/integration/test_server_lifecycle.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 66 行，包含 0 个类与 7 个模块级函数。模块文档写道：「tests/integration/test_server_lifecycle.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_setup_bootstrap.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_setup_bootstrap.py`
- **约行数**：137
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/integration/test_setup_bootstrap.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 137 行，包含 0 个类与 9 个模块级函数。模块文档写道：「tests/integration/test_setup_bootstrap.py — initial-admin path.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_setup_database.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_setup_database.py`
- **约行数**：157
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/integration/test_setup_database.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 157 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Setup wizard database step API tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_setup_wizard.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_setup_wizard.py`
- **约行数**：431
- **类 / 模块级函数**：0 / 28
- **主要功能简介**：`tests/integration/test_setup_wizard.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 431 行，包含 0 个类与 28 个模块级函数。模块文档写道：「Integration tests for the 4-step wizard backend.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skill_packages_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_skill_packages_api.py`
- **约行数**：531
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`tests/integration/test_skill_packages_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 531 行，包含 0 个类与 17 个模块级函数。模块文档写道：「Integration coverage for global skill package HTTP routes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skills_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_skills_api.py`
- **约行数**：957
- **类 / 模块级函数**：0 / 38
- **主要功能简介**：`tests/integration/test_skills_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 957 行，包含 0 个类与 38 个模块级函数。模块文档写道：「tests/integration/test_skills_api.py — per-agent skill library.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skills_hub.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_skills_hub.py`
- **约行数**：319
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/integration/test_skills_hub.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 319 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Integration tests for GET /api/agents/{id}/skills/hub/search」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_subagents_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_subagents_api.py`
- **约行数**：222
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/integration/test_subagents_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 222 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Integration tests for GET /api/agents/{id}/subagents and the catalog.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_terminal_context.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_terminal_context.py`
- **约行数**：68
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/integration/test_terminal_context.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 68 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Integration tests for GET /api/agents/{agent_id}/terminal/context.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_terminal_ws.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_terminal_ws.py`
- **约行数**：66
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/integration/test_terminal_ws.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 66 行，包含 0 个类与 5 个模块级函数。模块文档写道：「tests/integration/test_terminal_ws.py — PTY WebSocket terminal.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_tool_settings_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_tool_settings_api.py`
- **约行数**：70
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/integration/test_tool_settings_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 70 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Integration tests for GET/PUT /api/agents/{id}/tool-settings.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_trajectory_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_trajectory_api.py`
- **约行数**：537
- **类 / 模块级函数**：1 / 19
- **主要功能简介**：`tests/integration/test_trajectory_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 537 行，包含 1 个类与 19 个模块级函数。模块文档写道：「HTTP trajectory history, event detail, metrics, export, and live SSE.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_SseStream`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_update_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_update_api.py`
- **约行数**：46
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_update_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 46 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Integration tests for /api/update.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_upload_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_upload_api.py`
- **约行数**：49
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_upload_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 49 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Integration tests for dashboard chat attachments in agent workspace.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_usage_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_usage_api.py`
- **约行数**：705
- **类 / 模块级函数**：0 / 23
- **主要功能简介**：`tests/integration/test_usage_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 705 行，包含 0 个类与 23 个模块级函数。模块文档写道：「tests/integration/test_usage_api.py — token usage ledger + summary endpoint.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_users_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_users_api.py`
- **约行数**：248
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/integration/test_users_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 248 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/integration/test_users_api.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_voice_probe_http.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_voice_probe_http.py`
- **约行数**：77
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/integration/test_voice_probe_http.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 77 行，包含 0 个类与 3 个模块级函数。模块文档写道：「HTTP glue for the live STT probe: provider errors surface as ok:false.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_workspace_api.py`

- **所在目录**：`tests/integration/`
- **相对路径**：`tests/integration/test_workspace_api.py`
- **约行数**：531
- **类 / 模块级函数**：0 / 26
- **主要功能简介**：`tests/integration/test_workspace_api.py` 属于集成测试：HTTP/WS/向导/数据库端到端，约 531 行，包含 0 个类与 26 个模块级函数。模块文档写道：「tests/integration/test_workspace_api.py — workspace endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/live/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `conftest.py`

- **所在目录**：`tests/live/`
- **相对路径**：`tests/live/conftest.py`
- **约行数**：134
- **类 / 模块级函数**：1 / 5
- **主要功能简介**：`tests/live/conftest.py` 属于实网测试：真实 LLM / OSS / 连接器（默认跳过），约 134 行，包含 1 个类与 5 个模块级函数。模块文档写道：「Fixtures for live tests that call real LLM endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`LiveOpenAIConfig`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_agent_expert_template_live.py`

- **所在目录**：`tests/live/`
- **相对路径**：`tests/live/test_agent_expert_template_live.py`
- **约行数**：252
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`tests/live/test_agent_expert_template_live.py` 属于实网测试：真实 LLM / OSS / 连接器（默认跳过），约 252 行，包含 0 个类与 11 个模块级函数。模块文档写道：「Live tests for expert template file copy via :class:`AgentManager`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agent_manager_live.py`

- **所在目录**：`tests/live/`
- **相对路径**：`tests/live/test_agent_manager_live.py`
- **约行数**：135
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/live/test_agent_manager_live.py` 属于实网测试：真实 LLM / OSS / 连接器（默认跳过），约 135 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Live integration tests for :class:`octop.infra.agents.manager.AgentManager`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_channel_probe.py`

- **所在目录**：`tests/live/`
- **相对路径**：`tests/live/test_channel_probe.py`
- **约行数**：88
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/live/test_channel_probe.py` 属于实网测试：真实 LLM / OSS / 连接器（默认跳过），约 88 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Live channel credential probes (WeChat iLink + Feishu).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_connector_probe.py`

- **所在目录**：`tests/live/`
- **相对路径**：`tests/live/test_connector_probe.py`
- **约行数**：97
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/live/test_connector_probe.py` 属于实网测试：真实 LLM / OSS / 连接器（默认跳过），约 97 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Live connectivity probes for bundled token / api_key connectors.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_oss_storage.py`

- **所在目录**：`tests/live/`
- **相对路径**：`tests/live/test_oss_storage.py`
- **约行数**：111
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/live/test_oss_storage.py` 属于实网测试：真实 LLM / OSS / 连接器（默认跳过），约 111 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Live round-trip tests against real object-storage backends (COS / S3 / OSS / OBS).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/support/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`tests/support/__init__.py` 属于测试夹具与假对象，约 2 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Shared test helpers (fakes, HTTP app bootstrap, auth).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `app.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/app.py`
- **约行数**：76
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/support/app.py` 属于测试夹具与假对象，约 76 行，包含 0 个类与 3 个模块级函数。模块文档写道：「OctopServer + ASGI client lifecycle for integration tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `auth.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/auth.py`
- **约行数**：196
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`tests/support/auth.py` 属于测试夹具与假对象，约 196 行，包含 0 个类与 12 个模块级函数。模块文档写道：「Auth and bootstrap helpers for HTTP integration tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `bwrap_marks.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/bwrap_marks.py`
- **约行数**：22
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`tests/support/bwrap_marks.py` 属于测试夹具与假对象，约 22 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Shared pytest markers for bubblewrap / scoped-root_dir tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `fakes.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/fakes.py`
- **约行数**：489
- **类 / 模块级函数**：4 / 1
- **主要功能简介**：`tests/support/fakes.py` 属于测试夹具与假对象，约 489 行，包含 4 个类与 1 个模块级函数。模块文档写道：「Reusable test doubles for octop.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`FakeHarnessAgent`、`_FakeHarnessGraph`、`_FakeBackend`、`LoopbackChannel`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `harness.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/harness.py`
- **约行数**：193
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/support/harness.py` 属于测试夹具与假对象，约 193 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Patch harness-agent so tests run without a real LLM.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `http.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/http.py`
- **约行数**：165
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`tests/support/http.py` 属于测试夹具与假对象，约 165 行，包含 1 个类与 4 个模块级函数。模块文档写道：「WebSocket / streaming helpers for integration tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`ASGIWebSocketSession`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `postgresql.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/postgresql.py`
- **约行数**：18
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`tests/support/postgresql.py` 属于测试夹具与假对象，约 18 行，包含 0 个类与 0 个模块级函数。模块文档写道：「Helpers for gated live-PostgreSQL tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `scenarios.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/scenarios.py`
- **约行数**：56
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/support/scenarios.py` 属于测试夹具与假对象，约 56 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Multi-step bootstrap scenarios for integration tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `secrets.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/secrets.py`
- **约行数**：35
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/support/secrets.py` 属于测试夹具与假对象，约 35 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Helpers for live tests that need real credentials.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `testmon_staged_changes.py`

- **所在目录**：`tests/support/`
- **相对路径**：`tests/support/testmon_staged_changes.py`
- **约行数**：56
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/support/testmon_staged_changes.py` 属于测试夹具与假对象，约 56 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Make pytest-testmon see *staged* changes (required for the git pre-commit hook).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/__init__.py`
- **约行数**：2
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`tests/unit/__init__.py` 属于单元测试：隔离模块契约，约 2 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_bundled_plugins_layout.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_bundled_plugins_layout.py`
- **约行数**：71
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/test_bundled_plugins_layout.py` 属于单元测试：隔离模块契约，约 71 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Bundled plugin tree must be complete enough to seed and render UI.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cli_main.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_cli_main.py`
- **约行数**：187
- **类 / 模块级函数**：0 / 14
- **主要功能简介**：`tests/unit/test_cli_main.py` 属于单元测试：隔离模块契约，约 187 行，包含 0 个类与 14 个模块级函数。模块文档写道：「tests/unit/test_cli_main.py — basic help/import smoke test.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cli_state.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_cli_state.py`
- **约行数**：32
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/test_cli_state.py` 属于单元测试：隔离模块契约，约 32 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/test_cli_state.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_config.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_config.py`
- **约行数**：433
- **类 / 模块级函数**：0 / 33
- **主要功能简介**：`tests/unit/test_config.py` 属于单元测试：隔离模块契约，约 433 行，包含 0 个类与 33 个模块级函数。模块文档写道：「tests/unit/test_config.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_connectors.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_connectors.py`
- **约行数**：1424
- **类 / 模块级函数**：0 / 81
- **主要功能简介**：`tests/unit/test_connectors.py` 属于单元测试：隔离模块契约，约 1424 行，包含 0 个类与 81 个模块级函数。模块文档写道：「Unit tests for connector builder and repo.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_docker_compose_database_env.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_docker_compose_database_env.py`
- **约行数**：30
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/test_docker_compose_database_env.py` 属于单元测试：隔离模块契约，约 30 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Compose must forward OCTOP_DATABASE_* into the container (issue #152).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_env_file.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_env_file.py`
- **约行数**：85
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/test_env_file.py` 属于单元测试：隔离模块契约，约 85 行，包含 0 个类与 6 个模块级函数。模块文档写道：「tests/unit/test_env_file.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_errors.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_errors.py`
- **约行数**：38
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/test_errors.py` 属于单元测试：隔离模块契约，约 38 行，包含 0 个类与 5 个模块级函数。模块文档写道：「tests/unit/test_errors.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_green_launch.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_green_launch.py`
- **约行数**：57
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/test_green_launch.py` 属于单元测试：隔离模块契约，约 57 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for the green portable launch bootstrap.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_host_download_allow.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_host_download_allow.py`
- **约行数**：62
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/test_host_download_allow.py` 属于单元测试：隔离模块契约，约 62 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Host absolute path helpers for workspace download / preview.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_host_paths.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_host_paths.py`
- **约行数**：54
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/test_host_paths.py` 属于单元测试：隔离模块契约，约 54 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/test_host_paths.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_hot_topics_plugin.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_hot_topics_plugin.py`
- **约行数**：91
- **类 / 模块级函数**：2 / 3
- **主要功能简介**：`tests/unit/test_hot_topics_plugin.py` 属于单元测试：隔离模块契约，约 91 行，包含 2 个类与 3 个模块级函数。模块文档写道：「Hot-topics plugin must use unauthenticated public endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FakeResp`、`_RecordingClient`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_identity.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_identity.py`
- **约行数**：27
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/test_identity.py` 属于单元测试：隔离模块契约，约 27 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/test_identity.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_langfuse_settings.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_langfuse_settings.py`
- **约行数**：94
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/test_langfuse_settings.py` 属于单元测试：隔离模块契约，约 94 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Unit tests for Langfuse settings store.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_launch_bwrap.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_launch_bwrap.py`
- **约行数**：69
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/test_launch_bwrap.py` 属于单元测试：隔离模块契约，约 69 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Tests for non-blocking bubblewrap provisioning in ``octop run``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_launch_deferred.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_launch_deferred.py`
- **约行数**：25
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/test_launch_deferred.py` 属于单元测试：隔离模块契约，约 25 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Launch composition root must tolerate deferred control-plane bind.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_logging.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_logging.py`
- **约行数**：295
- **类 / 模块级函数**：0 / 20
- **主要功能简介**：`tests/unit/test_logging.py` 属于单元测试：隔离模块契约，约 295 行，包含 0 个类与 20 个模块级函数。模块文档写道：「tests/unit/test_logging.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_metrics.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_metrics.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/test_metrics.py` 属于单元测试：隔离模块契约，约 29 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/test_metrics.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_password.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_password.py`
- **约行数**：70
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/test_password.py` 属于单元测试：隔离模块契约，约 70 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/test_password.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_paths.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_paths.py`
- **约行数**：71
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/test_paths.py` 属于单元测试：隔离模块契约，约 71 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/test_paths.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_plugin_manager.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_plugin_manager.py`
- **约行数**：327
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`tests/unit/test_plugin_manager.py` 属于单元测试：隔离模块契约，约 327 行，包含 0 个类与 17 个模块级函数。模块文档写道：「Unit tests for Octop plugin manager.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_plugin_seed.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_plugin_seed.py`
- **约行数**：193
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/test_plugin_seed.py` 属于单元测试：隔离模块契约，约 193 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Unit tests for bundled plugin seeding (copy + globally disabled).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_plugins.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_plugins.py`
- **约行数**：214
- **类 / 模块级函数**：0 / 14
- **主要功能简介**：`tests/unit/test_plugins.py` 属于单元测试：隔离模块契约，约 214 行，包含 0 个类与 14 个模块级函数。模块文档写道：「Unit tests for harness-agent plugin loader.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_fetch_models.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_provider_fetch_models.py`
- **约行数**：177
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/test_provider_fetch_models.py` 属于单元测试：隔离模块契约，约 177 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Unit tests for OpenAI-compatible remote model listing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_preset_expansion.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_provider_preset_expansion.py`
- **约行数**：107
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/test_provider_preset_expansion.py` 属于单元测试：隔离模块契约，约 107 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Unit tests for harness-backed provider presets.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_probe.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_provider_probe.py`
- **约行数**：274
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/test_provider_probe.py` 属于单元测试：隔离模块契约，约 274 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Unit tests for provider connectivity probe helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_release_download_links.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_release_download_links.py`
- **约行数**：45
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/test_release_download_links.py` 属于单元测试：隔离模块契约，约 45 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/test_release_download_links.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_security_settings.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_security_settings.py`
- **约行数**：34
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/test_security_settings.py` 属于单元测试：隔离模块契约，约 34 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Tests for security settings store.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_shared.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_shared.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/test_shared.py` 属于单元测试：隔离模块契约，约 29 行，包含 0 个类与 1 个模块级函数。模块文档写道：「tests/unit/test_shared.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skillhub_install_metadata.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_skillhub_install_metadata.py`
- **约行数**：72
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/test_skillhub_install_metadata.py` 属于单元测试：隔离模块契约，约 72 行，包含 0 个类与 4 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skills_hub_errors.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_skills_hub_errors.py`
- **约行数**：117
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/test_skills_hub_errors.py` 属于单元测试：隔离模块契约，约 117 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Unit tests for SkillHub install error mapping and CLI subprocess helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_storage_browse.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_storage_browse.py`
- **约行数**：142
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/test_storage_browse.py` 属于单元测试：隔离模块契约，约 142 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Unit tests for storage backend browse helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_storage_probe.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_storage_probe.py`
- **约行数**：201
- **类 / 模块级函数**：2 / 8
- **主要功能简介**：`tests/unit/test_storage_probe.py` 属于单元测试：隔离模块契约，约 201 行，包含 2 个类与 8 个模块级函数。模块文档写道：「Unit tests for storage backend probes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FakeWrite`、`_FakeRead`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_storage_spec.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_storage_spec.py`
- **约行数**：236
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`tests/unit/test_storage_spec.py` 属于单元测试：隔离模块契约，约 236 行，包含 0 个类与 11 个模块级函数。模块文档写道：「Tests for storage backend spec resolution.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_tool_guard_rules_store.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_tool_guard_rules_store.py`
- **约行数**：50
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/test_tool_guard_rules_store.py` 属于单元测试：隔离模块契约，约 50 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Tests for user-editable tool guard rules store.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_ulid.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_ulid.py`
- **约行数**：40
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/test_ulid.py` 属于单元测试：隔离模块契约，约 40 行，包含 0 个类与 5 个模块级函数。模块文档写道：「tests/unit/test_ulid.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_usage_thread_totals.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_usage_thread_totals.py`
- **约行数**：69
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/test_usage_thread_totals.py` 属于单元测试：隔离模块契约，约 69 行，包含 0 个类与 2 个模块级函数。模块文档写道：「tests/unit/test_usage_thread_totals.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_user_locale.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_user_locale.py`
- **约行数**：81
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/test_user_locale.py` 属于单元测试：隔离模块契约，约 81 行，包含 0 个类与 7 个模块级函数。模块文档写道：「tests/unit/test_user_locale.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_user_manager.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_user_manager.py`
- **约行数**：473
- **类 / 模块级函数**：0 / 36
- **主要功能简介**：`tests/unit/test_user_manager.py` 属于单元测试：隔离模块契约，约 473 行，包含 0 个类与 36 个模块级函数。模块文档写道：「tests/unit/test_user_manager.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_voice_formats.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_voice_formats.py`
- **约行数**：44
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/test_voice_formats.py` 属于单元测试：隔离模块契约，约 44 行，包含 0 个类与 3 个模块级函数。模块文档写道：「STT upload MIME → provider format mapping (Tencent / Mimo).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_voice_manager.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_voice_manager.py`
- **约行数**：91
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/test_voice_manager.py` 属于单元测试：隔离模块契约，约 91 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for VoiceManager.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_voice_probe.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_voice_probe.py`
- **约行数**：168
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/test_voice_probe.py` 属于单元测试：隔离模块契约，约 168 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Voice probe (test_stt/test_tts) behavior per provider kind.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_voice_stt_probe.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_voice_stt_probe.py`
- **约行数**：140
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`tests/unit/test_voice_stt_probe.py` 属于单元测试：隔离模块契约，约 140 行，包含 0 个类与 11 个模块级函数。模块文档写道：「Live STT probe: test_stt performs a real recognition call per provider.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_wizard_password.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_wizard_password.py`
- **约行数**：80
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/test_wizard_password.py` 属于单元测试：隔离模块契约，约 80 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Unit tests for the wizard password file lifecycle.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_wizard_token.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_wizard_token.py`
- **约行数**：92
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/test_wizard_token.py` 属于单元测试：隔离模块契约，约 92 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Unit tests for the in-memory wizard token store + rate limiter.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_workspace_download.py`

- **所在目录**：`tests/unit/`
- **相对路径**：`tests/unit/test_workspace_download.py`
- **约行数**：51
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/test_workspace_download.py` 属于单元测试：隔离模块契约，约 51 行，包含 0 个类与 7 个模块级函数。模块文档写道：「tests/unit/test_workspace_download.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/agents/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_acp_settings.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_acp_settings.py`
- **约行数**：102
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/agents/test_acp_settings.py` 属于单元测试：隔离模块契约，约 102 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/unit/test_acp_settings.py — user-scoped global ACP runner settings.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agent_avatar.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_agent_avatar.py`
- **约行数**：174
- **类 / 模块级函数**：1 / 7
- **主要功能简介**：`tests/unit/agents/test_agent_avatar.py` 属于单元测试：隔离模块契约，约 174 行，包含 1 个类与 7 个模块级函数。模块文档写道：「Expert avatar file helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_MemoryWorkspace`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_agent_manager.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_agent_manager.py`
- **约行数**：1542
- **类 / 模块级函数**：0 / 79
- **主要功能简介**：`tests/unit/agents/test_agent_manager.py` 属于单元测试：隔离模块契约，约 1542 行，包含 0 个类与 79 个模块级函数。模块文档写道：「Unit tests for :mod:`octop.infra.agents.manager` internals.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agent_registry.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_agent_registry.py`
- **约行数**：1195
- **类 / 模块级函数**：0 / 64
- **主要功能简介**：`tests/unit/agents/test_agent_registry.py` 属于单元测试：隔离模块契约，约 1195 行，包含 0 个类与 64 个模块级函数。模块文档写道：「tests/unit/test_agent_registry.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agent_skill_packages.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_agent_skill_packages.py`
- **约行数**：399
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`tests/unit/agents/test_agent_skill_packages.py` 属于单元测试：隔离模块契约，约 399 行，包含 0 个类与 15 个模块级函数。模块文档写道：「Unit tests for agent skill package mounts.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agent_tools.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_agent_tools.py`
- **约行数**：32
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/agents/test_agent_tools.py` 属于单元测试：隔离模块契约，约 32 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/test_agent_tools.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_binary_read_guard.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_binary_read_guard.py`
- **约行数**：21
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/agents/test_binary_read_guard.py` 属于单元测试：隔离模块契约，约 21 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for BinaryReadGuardMiddleware.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_browser_profile_middleware.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_browser_profile_middleware.py`
- **约行数**：79
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/agents/test_browser_profile_middleware.py` 属于单元测试：隔离模块契约，约 79 行，包含 0 个类与 5 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_clinical_learning_subscription_template.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_clinical_learning_subscription_template.py`
- **约行数**：768
- **类 / 模块级函数**：0 / 28
- **主要功能简介**：`tests/unit/agents/test_clinical_learning_subscription_template.py` 属于单元测试：隔离模块契约，约 768 行，包含 0 个类与 28 个模块级函数。模块文档写道：「Regression tests for the clinical learning + general assistant template.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_context_breakdown.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_context_breakdown.py`
- **约行数**：243
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/agents/test_context_breakdown.py` 属于单元测试：隔离模块契约，约 243 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Tests for Octop context-usage adapter over harness-agent.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_custom_agent_id.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_custom_agent_id.py`
- **约行数**：95
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`tests/unit/agents/test_custom_agent_id.py` 属于单元测试：隔离模块契约，约 95 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Unit tests for user-supplied custom agent ids on expert creation.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TestValidateCustomAgentId`、`TestCreateWithCustomAgentId`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_default_agent.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_default_agent.py`
- **约行数**：101
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/agents/test_default_agent.py` 属于单元测试：隔离模块契约，约 101 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Unit tests for default-agent bootstrap helper.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_execute_env.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_execute_env.py`
- **约行数**：166
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/agents/test_execute_env.py` 属于单元测试：隔离模块契约，约 166 行，包含 0 个类与 7 个模块级函数。模块文档写道：「tests/unit/agents/test_execute_env.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_expert_catalog.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_expert_catalog.py`
- **约行数**：309
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/agents/test_expert_catalog.py` 属于单元测试：隔离模块契约，约 309 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Tests for ExpertCatalog workspace discovery and lazy file reads.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_expert_manifest_generator.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_expert_manifest_generator.py`
- **约行数**：362
- **类 / 模块级函数**：1 / 13
- **主要功能简介**：`tests/unit/agents/test_expert_manifest_generator.py` 属于单元测试：隔离模块契约，约 362 行，包含 1 个类与 13 个模块级函数。模块文档写道：「Tests for model-generated SkillHub expert manifest metadata.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`FakeLLM`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_expert_publish_snapshot.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_expert_publish_snapshot.py`
- **约行数**：435
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/agents/test_expert_publish_snapshot.py` 属于单元测试：隔离模块契约，约 435 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Tests for exported published-expert snapshots.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_general_assistant_template.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_general_assistant_template.py`
- **约行数**：27
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/agents/test_general_assistant_template.py` 属于单元测试：隔离模块契约，约 27 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Sanity checks for the bundled general-assistant template.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_karpathy_knowledge_base_template.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_karpathy_knowledge_base_template.py`
- **约行数**：56
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/agents/test_karpathy_knowledge_base_template.py` 属于单元测试：隔离模块契约，约 56 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Regression tests for the bundled Karpathy-style knowledge-base expert.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_library_task_examples.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_library_task_examples.py`
- **约行数**：54
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/agents/test_library_task_examples.py` 属于单元测试：隔离模块契约，约 54 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Bundled expert templates expose tasks-page ``task_examples``.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_mcp_tool_cache.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_mcp_tool_cache.py`
- **约行数**：263
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/agents/test_mcp_tool_cache.py` 属于单元测试：隔离模块契约，约 263 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for custom MCP deferred load + user-level tool cache.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_media_generation_settings.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_media_generation_settings.py`
- **约行数**：148
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/agents/test_media_generation_settings.py` 属于单元测试：隔离模块契约，约 148 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Tests for encrypted media-generation settings.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_memory_backend.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_memory_backend.py`
- **约行数**：150
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`tests/unit/agents/test_memory_backend.py` 属于单元测试：隔离模块契约，约 150 行，包含 0 个类与 12 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_model_flags.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_model_flags.py`
- **约行数**：130
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/agents/test_model_flags.py` 属于单元测试：隔离模块契约，约 130 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Tests for embedding / chat-eligibility model flags.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_octop_builtin_skills.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_octop_builtin_skills.py`
- **约行数**：259
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/agents/test_octop_builtin_skills.py` 属于单元测试：隔离模块契约，约 259 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Tests for Octop-owned built-in Skills and the Skill Manager helper.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_onnx_download.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_onnx_download.py`
- **约行数**：403
- **类 / 模块级函数**：0 / 14
- **主要功能简介**：`tests/unit/agents/test_onnx_download.py` 属于单元测试：隔离模块契约，约 403 行，包含 0 个类与 14 个模块级函数。模块文档写道：「Tests for ONNX source race + download.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_onnx_service.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_onnx_service.py`
- **约行数**：444
- **类 / 模块级函数**：0 / 24
- **主要功能简介**：`tests/unit/agents/test_onnx_service.py` 属于单元测试：隔离模块契约，约 444 行，包含 0 个类与 24 个模块级函数。模块文档写道：「Tests for local ONNX embedding service helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_persona.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_persona.py`
- **约行数**：53
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/agents/test_persona.py` 属于单元测试：隔离模块契约，约 53 行，包含 0 个类与 7 个模块级函数。模块文档写道：「tests/unit/test_persona.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_plugin_tool_defaults.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_plugin_tool_defaults.py`
- **约行数**：61
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/agents/test_plugin_tool_defaults.py` 属于单元测试：隔离模块契约，约 61 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for plugin tool default-on / merge helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_harness_factory.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_provider_harness_factory.py`
- **约行数**：42
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/agents/test_provider_harness_factory.py` 属于单元测试：隔离模块契约，约 42 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Tests for :mod:`octop.infra.agents.providers.harness_factory`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_reasoning.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_provider_reasoning.py`
- **约行数**：140
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/agents/test_provider_reasoning.py` 属于单元测试：隔离模块契约，约 140 行，包含 0 个类与 10 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_reload_impact.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_provider_reload_impact.py`
- **约行数**：160
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/agents/test_provider_reload_impact.py` 属于单元测试：隔离模块契约，约 160 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Tests for selective + parallel provider reload (faster confirm).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_store.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_provider_store.py`
- **约行数**：339
- **类 / 模块级函数**：0 / 18
- **主要功能简介**：`tests/unit/agents/test_provider_store.py` 属于单元测试：隔离模块契约，约 339 行，包含 0 个类与 18 个模块级函数。模块文档写道：「Unit tests for :mod:`octop.infra.agents.providers.store`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_reasoning_middleware.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_reasoning_middleware.py`
- **约行数**：53
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/agents/test_reasoning_middleware.py` 属于单元测试：隔离模块契约，约 53 行，包含 0 个类与 2 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_runtime_limits.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_runtime_limits.py`
- **约行数**：72
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/agents/test_runtime_limits.py` 属于单元测试：隔离模块契约，约 72 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Tests for agent runtime limit mapping.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skill_package_store.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_skill_package_store.py`
- **约行数**：210
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/agents/test_skill_package_store.py` 属于单元测试：隔离模块契约，约 210 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Tests for global skill package disk storage.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skill_packages.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_skill_packages.py`
- **约行数**：197
- **类 / 模块级函数**：0 / 14
- **主要功能简介**：`tests/unit/agents/test_skill_packages.py` 属于单元测试：隔离模块契约，约 197 行，包含 0 个类与 14 个模块级函数。模块文档写道：「Tests for source-neutral skill package normalization.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skillhub_http_market.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_skillhub_http_market.py`
- **约行数**：213
- **类 / 模块级函数**：1 / 10
- **主要功能简介**：`tests/unit/agents/test_skillhub_http_market.py` 属于单元测试：隔离模块契约，约 213 行，包含 1 个类与 10 个模块级函数。模块文档写道：「Tests for direct HTTP SkillHub search and package installation.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_BytesResponse`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_skillhub_market.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_skillhub_market.py`
- **约行数**：748
- **类 / 模块级函数**：1 / 32
- **主要功能简介**：`tests/unit/agents/test_skillhub_market.py` 属于单元测试：隔离模块契约，约 748 行，包含 1 个类与 32 个模块级函数。模块文档写道：「Tests for SkillHub skillset normalization into expert templates.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_Response`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_skills_hub_raw.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_skills_hub_raw.py`
- **约行数**：240
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/agents/test_skills_hub_raw.py` 属于单元测试：隔离模块契约，约 240 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Unit tests for skills.sh / GitHub raw bundle resolution.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_subagent_catalog.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_subagent_catalog.py`
- **约行数**：219
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`tests/unit/agents/test_subagent_catalog.py` 属于单元测试：隔离模块契约，约 219 行，包含 0 个类与 12 个模块级函数。模块文档写道：「Tests for SubagentCatalog bundled library.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_thread_artifacts.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_thread_artifacts.py`
- **约行数**：198
- **类 / 模块级函数**：1 / 13
- **主要功能简介**：`tests/unit/agents/test_thread_artifacts.py` 属于单元测试：隔离模块契约，约 198 行，包含 1 个类与 13 个模块级函数。模块文档写道：「Thread workspace artifact extraction and middleware persistence.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FakeThreads`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_thread_fork.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_thread_fork.py`
- **约行数**：257
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/agents/test_thread_fork.py` 属于单元测试：隔离模块契约，约 257 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Tests for conversation fork-from-assistant-message.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_token_quota_middleware.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_token_quota_middleware.py`
- **约行数**：71
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/agents/test_token_quota_middleware.py` 属于单元测试：隔离模块契约，约 71 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Token quota agent middleware tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_tool_catalog.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_tool_catalog.py`
- **约行数**：94
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/agents/test_tool_catalog.py` 属于单元测试：隔离模块契约，约 94 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for built-in tool catalog / tools_disabled normalization.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_workspace_dir.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_workspace_dir.py`
- **约行数**：126
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/agents/test_workspace_dir.py` 属于单元测试：隔离模块契约，约 126 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Unit tests for ``workspace_dir`` (independent of ``root_dir`` for harness).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_workspace_image_middleware.py`

- **所在目录**：`tests/unit/agents/`
- **相对路径**：`tests/unit/agents/test_workspace_image_middleware.py`
- **约行数**：57
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/agents/test_workspace_image_middleware.py` 属于单元测试：隔离模块契约，约 57 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Unit tests for WorkspaceImageMaterializeMiddleware.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/api/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_acl_gate_coverage.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_acl_gate_coverage.py`
- **约行数**：84
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/api/test_acl_gate_coverage.py` 属于单元测试：隔离模块契约，约 84 行，包含 0 个类与 3 个模块级函数。模块文档写道：「ACL gate coverage: no current_admin; known permission keys only.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agent_access.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_agent_access.py`
- **约行数**：85
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/api/test_agent_access.py` 属于单元测试：隔离模块契约，约 85 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/api/test_agent_access.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_attachments.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_attachments.py`
- **约行数**：268
- **类 / 模块级函数**：0 / 22
- **主要功能简介**：`tests/unit/api/test_attachments.py` 属于单元测试：隔离模块契约，约 268 行，包含 0 个类与 22 个模块级函数。模块文档写道：「Unit tests for chat attachment workspace paths.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_backup_restore_rehydrate.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_backup_restore_rehydrate.py`
- **约行数**：221
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/api/test_backup_restore_rehydrate.py` 属于单元测试：隔离模块契约，约 221 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Unit tests for admin backup restore rehydrate and non-blocking paths.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_browser_stream_listen.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_browser_stream_listen.py`
- **约行数**：139
- **类 / 模块级函数**：1 / 5
- **主要功能简介**：`tests/unit/api/test_browser_stream_listen.py` 属于单元测试：隔离模块契约，约 139 行，包含 1 个类与 5 个模块级函数。模块文档写道：「Listen-only browser-stream must not launch Chrome and must stay connected.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FakeWs`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_browser_stream_mouse.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_browser_stream_mouse.py`
- **约行数**：140
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/api/test_browser_stream_mouse.py` 属于单元测试：隔离模块契约，约 140 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Browser WebSocket stream must forward press / move / release for drag.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_chat_composer_context.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_chat_composer_context.py`
- **约行数**：67
- **类 / 模块级函数**：1 / 3
- **主要功能简介**：`tests/unit/api/test_chat_composer_context.py` 属于单元测试：隔离模块契约，约 67 行，包含 1 个类与 3 个模块级函数。模块文档写道：「Composer context for dashboard chat turns.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_ThreadRegistry`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_chat_history.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_chat_history.py`
- **约行数**：420
- **类 / 模块级函数**：0 / 16
- **主要功能简介**：`tests/unit/api/test_chat_history.py` 属于单元测试：隔离模块契约，约 420 行，包含 0 个类与 16 个模块级函数。模块文档写道：「Unit tests for thread list/history HTTP helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_chat_hitl_resume.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_chat_hitl_resume.py`
- **约行数**：161
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/api/test_chat_hitl_resume.py` 属于单元测试：隔离模块契约，约 161 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Dashboard HITL resume SSE must persist follow-up approvals in the pending store.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_chat_polish.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_chat_polish.py`
- **约行数**：577
- **类 / 模块级函数**：0 / 32
- **主要功能简介**：`tests/unit/api/test_chat_polish.py` 属于单元测试：隔离模块契约，约 577 行，包含 0 个类与 32 个模块级函数。模块文档写道：「Unit tests for chat polish and history helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_chat_serialize.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_chat_serialize.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/api/test_chat_serialize.py` 属于单元测试：隔离模块契约，约 29 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Unit tests for chat SSE / WebSocket chunk serialization.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_chat_welcome.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_chat_welcome.py`
- **约行数**：235
- **类 / 模块级函数**：0 / 16
- **主要功能简介**：`tests/unit/api/test_chat_welcome.py` 属于单元测试：隔离模块契约，约 235 行，包含 0 个类与 16 个模块级函数。模块文档写道：「Unit tests for agent chat welcome (workspace / catalog / default).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_content_disposition.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_content_disposition.py`
- **约行数**：46
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/api/test_content_disposition.py` 属于单元测试：隔离模块契约，约 46 行，包含 0 个类与 7 个模块级函数。模块文档写道：「tests/unit/api/test_content_disposition.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_dashboard_cache.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_dashboard_cache.py`
- **约行数**：34
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/api/test_dashboard_cache.py` 属于单元测试：隔离模块契约，约 34 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Cache-Control for dashboard SPA shell vs hashed Vite assets.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_exception_handlers.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_exception_handlers.py`
- **约行数**：61
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/api/test_exception_handlers.py` 属于单元测试：隔离模块契约，约 61 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Exception handlers log server-side OctopError details.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_harness_tabs.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_harness_tabs.py`
- **约行数**：113
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/api/test_harness_tabs.py` 属于单元测试：隔离模块契约，约 113 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/unit/api/test_harness_tabs.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_hitl_resume_validation.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_hitl_resume_validation.py`
- **约行数**：75
- **类 / 模块级函数**：2 / 1
- **主要功能简介**：`tests/unit/api/test_hitl_resume_validation.py` 属于单元测试：隔离模块契约，约 75 行，包含 2 个类与 1 个模块级函数。模块文档写道：「Validation for the HITL resume request body.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TestAccepted`、`TestRejected`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_jwt_auth_middleware.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_jwt_auth_middleware.py`
- **约行数**：152
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/api/test_jwt_auth_middleware.py` 属于单元测试：隔离模块契约，约 152 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Unit tests for JWT auth middleware.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_jwt_tokens.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_jwt_tokens.py`
- **约行数**：35
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/api/test_jwt_tokens.py` 属于单元测试：隔离模块契约，约 35 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/unit/test_jwt_tokens.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_knowledge_bases.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_knowledge_bases.py`
- **约行数**：735
- **类 / 模块级函数**：2 / 27
- **主要功能简介**：`tests/unit/api/test_knowledge_bases.py` 属于单元测试：隔离模块契约，约 735 行，包含 2 个类与 27 个模块级函数。模块文档写道：「Unit tests for the knowledge-base HTTP adapter.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_Base`、`_Document`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_ollama_service_default.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_ollama_service_default.py`
- **约行数**：21
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/api/test_ollama_service_default.py` 属于单元测试：隔离模块契约，约 21 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Unit tests for Ollama local service enablement defaults.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_openapi_meta.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_openapi_meta.py`
- **约行数**：32
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/api/test_openapi_meta.py` 属于单元测试：隔离模块契约，约 32 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Unit tests for OpenAPI schema customization.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_permissions_api.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_permissions_api.py`
- **约行数**：39
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/api/test_permissions_api.py` 属于单元测试：隔离模块契约，约 39 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for require_permission dependency.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_provider_patch_rehydrate.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_provider_patch_rehydrate.py`
- **约行数**：92
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/api/test_provider_patch_rehydrate.py` 属于单元测试：隔离模块契约，约 92 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Unit tests: skip harness rehydrate when provider patch is note-only.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_require_running_workspace.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_require_running_workspace.py`
- **约行数**：99
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/api/test_require_running_workspace.py` 属于单元测试：隔离模块契约，约 99 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Workspace I/O should survive a harness rebuild window.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_setup_main_agent.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_setup_main_agent.py`
- **约行数**：48
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/api/test_setup_main_agent.py` 属于单元测试：隔离模块契约，约 48 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Regression coverage for setup's default ``main`` agent.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_terminal_platform.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_terminal_platform.py`
- **约行数**：46
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/api/test_terminal_platform.py` 属于单元测试：隔离模块契约，约 46 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Unit tests for terminal platform support.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_terminal_sessions.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_terminal_sessions.py`
- **约行数**：470
- **类 / 模块级函数**：1 / 27
- **主要功能简介**：`tests/unit/api/test_terminal_sessions.py` 属于单元测试：隔离模块契约，约 470 行，包含 1 个类与 27 个模块级函数。模块文档写道：「Unit tests for the terminal session machinery and WS handler.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FakeWS`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_update_router.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_update_router.py`
- **约行数**：475
- **类 / 模块级函数**：0 / 20
- **主要功能简介**：`tests/unit/api/test_update_router.py` 属于单元测试：隔离模块契约，约 475 行，包含 0 个类与 20 个模块级函数。模块文档写道：「Unit tests for update API router helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_update_store.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_update_store.py`
- **约行数**：31
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/api/test_update_store.py` 属于单元测试：隔离模块契约，约 31 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Unit tests for /api/update in-memory status cache.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_upload_limit.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_upload_limit.py`
- **约行数**：80
- **类 / 模块级函数**：1 / 5
- **主要功能简介**：`tests/unit/api/test_upload_limit.py` 属于单元测试：隔离模块契约，约 80 行，包含 1 个类与 5 个模块级函数。模块文档写道：「Capped multipart reads must stop before assembling an oversize body.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_CountingReader`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_voice_mimo.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_voice_mimo.py`
- **约行数**：210
- **类 / 模块级函数**：4 / 5
- **主要功能简介**：`tests/unit/api/test_voice_mimo.py` 属于单元测试：隔离模块契约，约 210 行，包含 4 个类与 5 个模块级函数。模块文档写道：「Unit tests for Mimo voice adapters (voice normalization + streaming TTS).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TestNormalizeMimoVoice`、`TestWavHeader`、`TestMimoTTSStreaming`、`TestMimoTTSStreamingYieldsNothingOnEmpty`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_workspace_io_path.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_workspace_io_path.py`
- **约行数**：44
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/api/test_workspace_io_path.py` 属于单元测试：隔离模块契约，约 44 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Workspace I/O path resolution for download / file APIs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_workspace_tree_entries.py`

- **所在目录**：`tests/unit/api/`
- **相对路径**：`tests/unit/api/test_workspace_tree_entries.py`
- **约行数**：27
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/api/test_workspace_tree_entries.py` 属于单元测试：隔离模块契约，约 27 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Tree listing entries stay anchored at the directory the caller requested.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/auth/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_captcha_env.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_captcha_env.py`
- **约行数**：64
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/auth/test_captcha_env.py` 属于单元测试：隔离模块契约，约 64 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Boot-time CaptchaEnv snapshot.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_captcha_providers.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_captcha_providers.py`
- **约行数**：167
- **类 / 模块级函数**：1 / 19
- **主要功能简介**：`tests/unit/auth/test_captcha_providers.py` 属于单元测试：隔离模块契约，约 167 行，包含 1 个类与 19 个模块级函数。模块文档写道：「CaptchaProvider registry — slugs, aliases, plug-in seam.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FakeStrong`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_captcha_store.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_captcha_store.py`
- **约行数**：132
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/auth/test_captcha_store.py` 属于单元测试：隔离模块契约，约 132 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Effective captcha config — settings blob vs env snapshot.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_captcha_verify.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_captcha_verify.py`
- **约行数**：200
- **类 / 模块级函数**：1 / 12
- **主要功能简介**：`tests/unit/auth/test_captcha_verify.py` 属于单元测试：隔离模块契约，约 200 行，包含 1 个类与 12 个模块级函数。模块文档写道：「ensure_captcha — slider no-op and mocked siteverify.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_Siteverify`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_dingtalk_adapter.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_dingtalk_adapter.py`
- **约行数**：133
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/auth/test_dingtalk_adapter.py` 属于单元测试：隔离模块契约，约 133 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Tests for the DingTalk dashboard SSO adapter.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_feishu_adapter.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_feishu_adapter.py`
- **约行数**：228
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/auth/test_feishu_adapter.py` 属于单元测试：隔离模块契约，约 228 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Tests for the Feishu dashboard SSO adapter.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_public_base.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_public_base.py`
- **约行数**：53
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/auth/test_public_base.py` 属于单元测试：隔离模块契约，约 53 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Tests for public OIDC callback URL helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_sso_crypto.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_sso_crypto.py`
- **约行数**：23
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/auth/test_sso_crypto.py` 属于单元测试：隔离模块契约，约 23 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Tests for SSO secret encryption.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_sso_discovery.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_sso_discovery.py`
- **约行数**：51
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/auth/test_sso_discovery.py` 属于单元测试：隔离模块契约，约 51 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for OIDC discovery helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_sso_id_token.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_sso_id_token.py`
- **约行数**：135
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/auth/test_sso_id_token.py` 属于单元测试：隔离模块契约，约 135 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Tests for OIDC ID token verification.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_sso_pkce.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_sso_pkce.py`
- **约行数**：22
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/auth/test_sso_pkce.py` 属于单元测试：隔离模块契约，约 22 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Tests for OIDC PKCE helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_sso_redirect_after.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_sso_redirect_after.py`
- **约行数**：28
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/auth/test_sso_redirect_after.py` 属于单元测试：隔离模块契约，约 28 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Tests for post-login redirect sanitization.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_sso_service.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_sso_service.py`
- **约行数**：444
- **类 / 模块级函数**：0 / 20
- **主要功能简介**：`tests/unit/auth/test_sso_service.py` 属于单元测试：隔离模块契约，约 444 行，包含 0 个类与 20 个模块级函数。模块文档写道：「Tests for OIDC SSO service orchestration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_wecom_adapter.py`

- **所在目录**：`tests/unit/auth/`
- **相对路径**：`tests/unit/auth/test_wecom_adapter.py`
- **约行数**：144
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/auth/test_wecom_adapter.py` 属于单元测试：隔离模块契约，约 144 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Tests for the WeCom dashboard SSO adapter.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/backend/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_opensandbox_deps.py`

- **所在目录**：`tests/unit/backend/`
- **相对路径**：`tests/unit/backend/test_opensandbox_deps.py`
- **约行数**：26
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/backend/test_opensandbox_deps.py` 属于单元测试：隔离模块契约，约 26 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Tests for on-demand OpenSandbox SDK install.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_resolver.py`

- **所在目录**：`tests/unit/backend/`
- **相对路径**：`tests/unit/backend/test_resolver.py`
- **约行数**：182
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/backend/test_resolver.py` 属于单元测试：隔离模块契约，约 182 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Tests for backend resolver helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/backup/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_auto.py`

- **所在目录**：`tests/unit/backup/`
- **相对路径**：`tests/unit/backup/test_auto.py`
- **约行数**：140
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/backup/test_auto.py` 属于单元测试：隔离模块契约，约 140 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for automatic backup helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_pg_dump_helpers.py`

- **所在目录**：`tests/unit/backup/`
- **相对路径**：`tests/unit/backup/test_pg_dump_helpers.py`
- **约行数**：37
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/backup/test_pg_dump_helpers.py` 属于单元测试：隔离模块契约，约 37 行，包含 0 个类与 2 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_snapshot.py`

- **所在目录**：`tests/unit/backup/`
- **相对路径**：`tests/unit/backup/test_snapshot.py`
- **约行数**：65
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/backup/test_snapshot.py` 属于单元测试：隔离模块契约，约 65 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Unit tests for SQLite snapshot helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_store.py`

- **所在目录**：`tests/unit/backup/`
- **相对路径**：`tests/unit/backup/test_store.py`
- **约行数**：292
- **类 / 模块级函数**：0 / 14
- **主要功能简介**：`tests/unit/backup/test_store.py` 属于单元测试：隔离模块契约，约 292 行，包含 0 个类与 14 个模块级函数。模块文档写道：「Unit tests for on-disk backup store.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_system_archive.py`

- **所在目录**：`tests/unit/backup/`
- **相对路径**：`tests/unit/backup/test_system_archive.py`
- **约行数**：1285
- **类 / 模块级函数**：0 / 24
- **主要功能简介**：`tests/unit/backup/test_system_archive.py` 属于单元测试：隔离模块契约，约 1285 行，包含 0 个类与 24 个模块级函数。模块文档写道：「Unit tests for system backup archives.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_workspace_archive.py`

- **所在目录**：`tests/unit/backup/`
- **相对路径**：`tests/unit/backup/test_workspace_archive.py`
- **约行数**：178
- **类 / 模块级函数**：1 / 6
- **主要功能简介**：`tests/unit/backup/test_workspace_archive.py` 属于单元测试：隔离模块契约，约 178 行，包含 1 个类与 6 个模块级函数。模块文档写道：「Unit tests for workspace zip archives.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_MountlessBackend`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `tests/unit/browser/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_browser_setup.py`

- **所在目录**：`tests/unit/browser/`
- **相对路径**：`tests/unit/browser/test_browser_setup.py`
- **约行数**：397
- **类 / 模块级函数**：0 / 18
- **主要功能简介**：`tests/unit/browser/test_browser_setup.py` 属于单元测试：隔离模块契约，约 397 行，包含 0 个类与 18 个模块级函数。模块文档写道：「Tests for browser profile prep / uninstall helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/cli/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `__init__.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/__init__.py`
- **约行数**：1
- **类 / 模块级函数**：0 / 0
- **主要功能简介**：`tests/unit/cli/__init__.py` 属于单元测试：隔离模块契约，约 1 行，包含 0 个类与 0 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_captcha_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_captcha_cmd.py`
- **约行数**：51
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/cli/test_captcha_cmd.py` 属于单元测试：隔离模块契约，约 51 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Tests for `octop captcha reset` (lockout escape hatch).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_chats_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_chats_cmd.py`
- **约行数**：68
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/cli/test_chats_cmd.py` 属于单元测试：隔离模块契约，约 68 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Tests for the `octop chats` group.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_clean_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_clean_cmd.py`
- **约行数**：55
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/cli/test_clean_cmd.py` 属于单元测试：隔离模块契约，约 55 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Tests for `octop clean`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cli_db.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_cli_db.py`
- **约行数**：83
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/cli/test_cli_db.py` 属于单元测试：隔离模块契约，约 83 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Tests for offline CLI DB helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_completion_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_completion_cmd.py`
- **约行数**：44
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/cli/test_completion_cmd.py` 属于单元测试：隔离模块契约，约 44 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for `octop completion`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_init_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_init_cmd.py`
- **约行数**：136
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/cli/test_init_cmd.py` 属于单元测试：隔离模块契约，约 136 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Unit tests for `octop init`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_main_encoding.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_main_encoding.py`
- **约行数**：67
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/cli/test_main_encoding.py` 属于单元测试：隔离模块契约，约 67 行，包含 0 个类与 3 个模块级函数。模块文档写道：「CLI stdio encoding guard — emoji output must not crash on legacy code pages.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_models_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_models_cmd.py`
- **约行数**：16
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/cli/test_models_cmd.py` 属于单元测试：隔离模块契约，约 16 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Tests for ``octop models`` ollama subcommands.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_prompts.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_prompts.py`
- **约行数**：46
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/cli/test_prompts.py` 属于单元测试：隔离模块契约，约 46 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Tests for the questionary wrapper (no real TTY).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_registry.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_registry.py`
- **约行数**：30
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/cli/test_registry.py` 属于单元测试：隔离模块契约，约 30 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Lazy CLI registry integrity — every COMMANDS attr must resolve.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repl_session.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_repl_session.py`
- **约行数**：43
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/cli/test_repl_session.py` 属于单元测试：隔离模块契约，约 43 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Tests for REPL session state helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_run_bind_resolution.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_run_bind_resolution.py`
- **约行数**：134
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/cli/test_run_bind_resolution.py` 属于单元测试：隔离模块契约，约 134 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Bind address resolution for `octop run`: CLI flag > env > config.json.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_run_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_run_cmd.py`
- **约行数**：302
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/cli/test_run_cmd.py` 属于单元测试：隔离模块契约，约 302 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Tests for `octop run`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_service_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_service_cmd.py`
- **约行数**：249
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/cli/test_service_cmd.py` 属于单元测试：隔离模块契约，约 249 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Tests for `octop service`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skills_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_skills_cmd.py`
- **约行数**：73
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/cli/test_skills_cmd.py` 属于单元测试：隔离模块契约，约 73 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Tests for offline CLI skills helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_stub.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_stub.py`
- **约行数**：31
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/cli/test_stub.py` 属于单元测试：隔离模块契约，约 31 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for the Not-Applicable STUB helper.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_update_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_update_cmd.py`
- **约行数**：239
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`tests/unit/cli/test_update_cmd.py` 属于单元测试：隔离模块契约，约 239 行，包含 0 个类与 12 个模块级函数。模块文档写道：「Tests for `octop update`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_version_cmd.py`

- **所在目录**：`tests/unit/cli/`
- **相对路径**：`tests/unit/cli/test_version_cmd.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/cli/test_version_cmd.py` 属于单元测试：隔离模块契约，约 29 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for `octop version`.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/connectors/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_cli_install.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_cli_install.py`
- **约行数**：205
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/connectors/test_cli_install.py` 属于单元测试：隔离模块契约，约 205 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for connector host CLI install helper.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cli_runner_and_dirs.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_cli_runner_and_dirs.py`
- **约行数**：83
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/connectors/test_cli_runner_and_dirs.py` 属于单元测试：隔离模块契约，约 83 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for CLI runner messages and connector-cli dir cleanup.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_custom_mcp.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_custom_mcp.py`
- **约行数**：489
- **类 / 模块级函数**：0 / 22
- **主要功能简介**：`tests/unit/connectors/test_custom_mcp.py` 属于单元测试：隔离模块契约，约 489 行，包含 0 个类与 22 个模块级函数。模块文档写道：「Unit tests for custom MCP validation and assembly.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_default_open.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_default_open.py`
- **约行数**：106
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/connectors/test_default_open.py` 属于单元测试：隔离模块契约，约 106 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Unit tests for connector default_open config helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_feishu_user_auth.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_feishu_user_auth.py`
- **约行数**：156
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/connectors/test_feishu_user_auth.py` 属于单元测试：隔离模块契约，约 156 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Unit tests for Feishu CLI user device-code auth.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_feishu_user_auth_preview.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_feishu_user_auth_preview.py`
- **约行数**：84
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/connectors/test_feishu_user_auth_preview.py` 属于单元测试：隔离模块契约，约 84 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Unit tests for Feishu live user-auth preview.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_feishu_wecom_cli.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_feishu_wecom_cli.py`
- **约行数**：475
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/connectors/test_feishu_wecom_cli.py` 属于单元测试：隔离模块契约，约 475 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Unit tests for Feishu / WeCom CLI gateway adapters.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_mail_servers.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_mail_servers.py`
- **约行数**：120
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/connectors/test_mail_servers.py` 属于单元测试：隔离模块契约，约 120 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Unit tests for shared mailbox host resolution and qq-mail IMAP login.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_mcp_oauth_ssrf.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_mcp_oauth_ssrf.py`
- **约行数**：94
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/connectors/test_mcp_oauth_ssrf.py` 属于单元测试：隔离模块契约，约 94 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Unit tests for MCP OAuth SSRF guards.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_oauth_discovery.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_oauth_discovery.py`
- **约行数**：161
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/connectors/test_oauth_discovery.py` 属于单元测试：隔离模块契约，约 161 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Unit tests for MCP OAuth discovery and unified oauth targets.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_tencent_ima.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_tencent_ima.py`
- **约行数**：234
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`tests/unit/connectors/test_tencent_ima.py` 属于单元测试：隔离模块契约，约 234 行，包含 0 个类与 17 个模块级函数。模块文档写道：「Unit tests for Tencent IMA gateway adapter (JSON OpenAPI tools).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_wecom_cli_errors.py`

- **所在目录**：`tests/unit/connectors/`
- **相对路径**：`tests/unit/connectors/test_wecom_cli_errors.py`
- **约行数**：120
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/connectors/test_wecom_cli_errors.py` 属于单元测试：隔离模块契约，约 120 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Unit tests for WeCom CLI error humanization and probe.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/cron/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_cron_composer_context.py`

- **所在目录**：`tests/unit/cron/`
- **相对路径**：`tests/unit/cron/test_cron_composer_context.py`
- **约行数**：244
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/cron/test_cron_composer_context.py` 属于单元测试：隔离模块契约，约 244 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Cron agent runs should stamp composer_context on the stored user message.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cron_delivery.py`

- **所在目录**：`tests/unit/cron/`
- **相对路径**：`tests/unit/cron/test_cron_delivery.py`
- **约行数**：359
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/cron/test_cron_delivery.py` 属于单元测试：隔离模块契约，约 359 行，包含 0 个类与 13 个模块级函数。模块文档写道：「CronDeliveryService: canonical checkpoint, channel push, and best-effort extras.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cron_job.py`

- **所在目录**：`tests/unit/cron/`
- **相对路径**：`tests/unit/cron/test_cron_job.py`
- **约行数**：173
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/cron/test_cron_job.py` 属于单元测试：隔离模块契约，约 173 行，包含 0 个类与 6 个模块级函数。模块文档写道：「tests/unit/test_cron_job.py — CronJob bookkeeping around CronDeliveryService.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cron_manager.py`

- **所在目录**：`tests/unit/cron/`
- **相对路径**：`tests/unit/cron/test_cron_manager.py`
- **约行数**：684
- **类 / 模块级函数**：0 / 37
- **主要功能简介**：`tests/unit/cron/test_cron_manager.py` 属于单元测试：隔离模块契约，约 684 行，包含 0 个类与 37 个模块级函数。模块文档写道：「tests/unit/test_cron_manager.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cron_manager_global.py`

- **所在目录**：`tests/unit/cron/`
- **相对路径**：`tests/unit/cron/test_cron_manager_global.py`
- **约行数**：89
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/cron/test_cron_manager_global.py` 属于单元测试：隔离模块契约，约 89 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/test_cron_manager_global.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cron_mcp_servers.py`

- **所在目录**：`tests/unit/cron/`
- **相对路径**：`tests/unit/cron/test_cron_mcp_servers.py`
- **约行数**：51
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/cron/test_cron_mcp_servers.py` 属于单元测试：隔离模块契约，约 51 行，包含 0 个类与 1 个模块级函数。模块文档写道：「CronJobRepo mcp_servers persistence.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cron_tools.py`

- **所在目录**：`tests/unit/cron/`
- **相对路径**：`tests/unit/cron/test_cron_tools.py`
- **约行数**：254
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`tests/unit/cron/test_cron_tools.py` 属于单元测试：隔离模块契约，约 254 行，包含 0 个类与 12 个模块级函数。模块文档写道：「Unit tests for built-in cronjob LangChain tools.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cron_trigger.py`

- **所在目录**：`tests/unit/cron/`
- **相对路径**：`tests/unit/cron/test_cron_trigger.py`
- **约行数**：118
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/cron/test_cron_trigger.py` 属于单元测试：隔离模块契约，约 118 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/test_cron_trigger.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_task_type.py`

- **所在目录**：`tests/unit/cron/`
- **相对路径**：`tests/unit/cron/test_task_type.py`
- **约行数**：55
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/cron/test_task_type.py` 属于单元测试：隔离模块契约，约 55 行，包含 0 个类与 9 个模块级函数。模块文档写道：「tests/unit/cron/test_task_type.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/db/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_agent_is_shared.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_agent_is_shared.py`
- **约行数**：32
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/db/test_agent_is_shared.py` 属于单元测试：隔离模块契约，约 32 行，包含 0 个类与 2 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agent_profile_columns.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_agent_profile_columns.py`
- **约行数**：164
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/db/test_agent_profile_columns.py` 属于单元测试：隔离模块契约，约 164 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Lift expert display fields out of agents.config_json.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_clip_thread_title.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_clip_thread_title.py`
- **约行数**：89
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/db/test_clip_thread_title.py` 属于单元测试：隔离模块契约，约 89 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Thread repo title clipping and legacy hard-cut repair.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_db_factory.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_db_factory.py`
- **约行数**：155
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/db/test_db_factory.py` 属于单元测试：隔离模块契约，约 155 行，包含 0 个类与 9 个模块级函数。模块文档写道：「tests/unit/test_db_factory.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_db_pool.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_db_pool.py`
- **约行数**：664
- **类 / 模块级函数**：0 / 18
- **主要功能简介**：`tests/unit/db/test_db_pool.py` 属于单元测试：隔离模块契约，约 664 行，包含 0 个类与 18 个模块级函数。模块文档写道：「tests/unit/test_db_pool.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_db_sql_helpers.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_db_sql_helpers.py`
- **约行数**：35
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/db/test_db_sql_helpers.py` 属于单元测试：隔离模块契约，约 35 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/unit/test_db_sql_helpers.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_migrate_discovery.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_migrate_discovery.py`
- **约行数**：18
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/db/test_migrate_discovery.py` 属于单元测试：隔离模块契约，约 18 行，包含 0 个类与 2 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_postgres_pool_unit.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_postgres_pool_unit.py`
- **约行数**：14
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/db/test_postgres_pool_unit.py` 属于单元测试：隔离模块契约，约 14 行，包含 0 个类与 2 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_published_experts_repo.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_published_experts_repo.py`
- **约行数**：134
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/db/test_published_experts_repo.py` 属于单元测试：隔离模块契约，约 134 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Unit tests for PublishedExpertRepo and migration 006.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repo_agents.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_repo_agents.py`
- **约行数**：145
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/db/test_repo_agents.py` 属于单元测试：隔离模块契约，约 145 行，包含 0 个类与 13 个模块级函数。模块文档写道：「tests/unit/test_repo_agents.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repo_channels.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_repo_channels.py`
- **约行数**：146
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/db/test_repo_channels.py` 属于单元测试：隔离模块契约，约 146 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/test_repo_channels.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repo_cron.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_repo_cron.py`
- **约行数**：194
- **类 / 模块级函数**：0 / 12
- **主要功能简介**：`tests/unit/db/test_repo_cron.py` 属于单元测试：隔离模块契约，约 194 行，包含 0 个类与 12 个模块级函数。模块文档写道：「tests/unit/test_repo_cron.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repo_knowledge.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_repo_knowledge.py`
- **约行数**：349
- **类 / 模块级函数**：0 / 19
- **主要功能简介**：`tests/unit/db/test_repo_knowledge.py` 属于单元测试：隔离模块契约，约 349 行，包含 0 个类与 19 个模块级函数。模块文档写道：「Unit tests for KnowledgeRepo and migration 005.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repo_providers.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_repo_providers.py`
- **约行数**：160
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/db/test_repo_providers.py` 属于单元测试：隔离模块契约，约 160 行，包含 0 个类与 13 个模块级函数。模块文档写道：「tests/unit/test_repo_providers.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repo_secret_audit.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_repo_secret_audit.py`
- **约行数**：70
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/db/test_repo_secret_audit.py` 属于单元测试：隔离模块契约，约 70 行，包含 0 个类与 6 个模块级函数。模块文档写道：「tests/unit/test_repo_secret_audit.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repo_sessions_threads.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_repo_sessions_threads.py`
- **约行数**：253
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/db/test_repo_sessions_threads.py` 属于单元测试：隔离模块契约，约 253 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/test_repo_sessions_threads.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repo_sso.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_repo_sso.py`
- **约行数**：138
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/db/test_repo_sso.py` 属于单元测试：隔离模块契约，约 138 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for OIDC SSO repository access.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_repo_users.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_repo_users.py`
- **约行数**：76
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/db/test_repo_users.py` 属于单元测试：隔离模块契约，约 76 行，包含 0 个类与 9 个模块级函数。模块文档写道：「tests/unit/test_repo_users.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_runtime_replace_services.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_runtime_replace_services.py`
- **约行数**：83
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/db/test_runtime_replace_services.py` 属于单元测试：隔离模块契约，约 83 行，包含 0 个类与 3 个模块级函数。模块文档写道：「AppRuntime.replace_services — public control-plane retarget API.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skill_package_icons.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_skill_package_icons.py`
- **约行数**：117
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/db/test_skill_package_icons.py` 属于单元测试：隔离模块契约，约 117 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Unit tests for skill package icon columns and get_by_name.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skill_packages_repo.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_skill_packages_repo.py`
- **约行数**：72
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/db/test_skill_packages_repo.py` 属于单元测试：隔离模块契约，约 72 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for SkillPackageRepo and migration 002.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_storage_backend_repo.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_storage_backend_repo.py`
- **约行数**：74
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/db/test_storage_backend_repo.py` 属于单元测试：隔离模块契约，约 74 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for BackendRepo.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_thread_messages_repo.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_thread_messages_repo.py`
- **约行数**：109
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/db/test_thread_messages_repo.py` 属于单元测试：隔离模块契约，约 109 行，包含 0 个类与 5 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_usage_window.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_usage_window.py`
- **约行数**：71
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/db/test_usage_window.py` 属于单元测试：隔离模块契约，约 71 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Unit tests for usage window resolution (server timezone day/month).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_user_permissions_column.py`

- **所在目录**：`tests/unit/db/`
- **相对路径**：`tests/unit/db/test_user_permissions_column.py`
- **约行数**：61
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/db/test_user_permissions_column.py` 属于单元测试：隔离模块契约，约 61 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for users.permissions column wiring.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/desktop/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_capture.py`

- **所在目录**：`tests/unit/desktop/`
- **相对路径**：`tests/unit/desktop/test_capture.py`
- **约行数**：113
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/desktop/test_capture.py` 属于单元测试：隔离模块契约，约 113 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Tests for desktop screen capture helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_capture_display.py`

- **所在目录**：`tests/unit/desktop/`
- **相对路径**：`tests/unit/desktop/test_capture_display.py`
- **约行数**：21
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/desktop/test_capture_display.py` 属于单元测试：隔离模块契约，约 21 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for DISPLAY helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_input.py`

- **所在目录**：`tests/unit/desktop/`
- **相对路径**：`tests/unit/desktop/test_input.py`
- **约行数**：57
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/desktop/test_input.py` 属于单元测试：隔离模块契约，约 57 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Tests for desktop input injection.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_native_platform.py`

- **所在目录**：`tests/unit/desktop/`
- **相对路径**：`tests/unit/desktop/test_native_platform.py`
- **约行数**：21
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/desktop/test_native_platform.py` 属于单元测试：隔离模块契约，约 21 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Tests for native vs virtual desktop capture paths.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_session.py`

- **所在目录**：`tests/unit/desktop/`
- **相对路径**：`tests/unit/desktop/test_session.py`
- **约行数**：100
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/desktop/test_session.py` 属于单元测试：隔离模块契约，约 100 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Tests for in-process desktop session registry.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_setup.py`

- **所在目录**：`tests/unit/desktop/`
- **相对路径**：`tests/unit/desktop/test_setup.py`
- **约行数**：331
- **类 / 模块级函数**：0 / 28
- **主要功能简介**：`tests/unit/desktop/test_setup.py` 属于单元测试：隔离模块契约，约 331 行，包含 0 个类与 28 个模块级函数。模块文档写道：「Tests for desktop setup helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_stamp_version.py`

- **所在目录**：`tests/unit/desktop/`
- **相对路径**：`tests/unit/desktop/test_stamp_version.py`
- **约行数**：84
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/desktop/test_stamp_version.py` 属于单元测试：隔离模块契约，约 84 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Tests for desktop Wails version stamping.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/gateway/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_attachment_hints.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_attachment_hints.py`
- **约行数**：367
- **类 / 模块级函数**：0 / 16
- **主要功能简介**：`tests/unit/gateway/test_attachment_hints.py` 属于单元测试：隔离模块契约，约 367 行，包含 0 个类与 16 个模块级函数。模块文档写道：「Unit tests for inbound attachment LLM hints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_attachment_path_resolution.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_attachment_path_resolution.py`
- **约行数**：226
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`tests/unit/gateway/test_attachment_path_resolution.py` 属于单元测试：隔离模块契约，约 226 行，包含 0 个类与 11 个模块级函数。模块文档写道：「Verify attachment workspace-path behaviour for LLM filesystem tools.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_backend_files.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_backend_files.py`
- **约行数**：107
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/gateway/test_backend_files.py` 属于单元测试：隔离模块契约，约 107 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for gateway media path helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_channels_qr.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_channels_qr.py`
- **约行数**：462
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`tests/unit/gateway/test_channels_qr.py` 属于单元测试：隔离模块契约，约 462 行，包含 0 个类与 17 个模块级函数。模块文档写道：「Unit tests for channels QR scan endpoints.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_chat_image_pipeline.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_chat_image_pipeline.py`
- **约行数**：268
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/gateway/test_chat_image_pipeline.py` 属于单元测试：隔离模块契约，约 268 行，包含 0 个类与 7 个模块级函数。模块文档写道：「End-to-end attachment pipeline tests (no WebSocket — direct API calls).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_cli_channel.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_cli_channel.py`
- **约行数**：81
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/gateway/test_cli_channel.py` 属于单元测试：隔离模块契约，约 81 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for the CLI gateway channel.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_dashboard_ws.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_dashboard_ws.py`
- **约行数**：676
- **类 / 模块级函数**：0 / 21
- **主要功能简介**：`tests/unit/gateway/test_dashboard_ws.py` 属于单元测试：隔离模块契约，约 676 行，包含 0 个类与 21 个模块级函数。模块文档写道：「tests/unit/test_dashboard_ws.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_feishu_bot_creator.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_feishu_bot_creator.py`
- **约行数**：131
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/gateway/test_feishu_bot_creator.py` 属于单元测试：隔离模块契约，约 131 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for Feishu scan-to-create (lark-oapi register_app).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_gateway.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_gateway.py`
- **约行数**：141
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/gateway/test_gateway.py` 属于单元测试：隔离模块契约，约 141 行，包含 0 个类与 5 个模块级函数。模块文档写道：「tests/unit/test_gateway.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_gateway_probe.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_gateway_probe.py`
- **约行数**：86
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/gateway/test_gateway_probe.py` 属于单元测试：隔离模块契约，约 86 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Unit tests for Gateway.probe_channel().」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_gateway_push.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_gateway_push.py`
- **约行数**：257
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/gateway/test_gateway_push.py` 属于单元测试：隔离模块契约，约 257 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/test_gateway_push.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_gateway_runtime_status.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_gateway_runtime_status.py`
- **约行数**：178
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/gateway/test_gateway_runtime_status.py` 属于单元测试：隔离模块契约，约 178 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Unit tests for Gateway runtime channel status.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_global_processor_team.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_global_processor_team.py`
- **约行数**：275
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/gateway/test_global_processor_team.py` 属于单元测试：隔离模块契约，约 275 行，包含 0 个类与 9 个模块级函数。模块文档写道：「tests/unit/test_global_processor_team.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_harness_request.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_harness_request.py`
- **约行数**：142
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/gateway/test_harness_request.py` 属于单元测试：隔离模块契约，约 142 行，包含 0 个类与 4 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_history_projection.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_history_projection.py`
- **约行数**：177
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/gateway/test_history_projection.py` 属于单元测试：隔离模块契约，约 177 行，包含 0 个类与 10 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_hitl_ask_user.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_hitl_ask_user.py`
- **约行数**：339
- **类 / 模块级函数**：5 / 1
- **主要功能简介**：`tests/unit/gateway/test_hitl_ask_user.py` 属于单元测试：隔离模块契约，约 339 行，包含 5 个类与 1 个模块级函数。模块文档写道：「Unit tests for the ``ask_user_question`` HITL channel.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`TestDetection`、`TestCard`、`TestReplyParsing`、`TestCoordinatorRouting`、`TestDecisionValidation`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_hitl_channel.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_hitl_channel.py`
- **约行数**：399
- **类 / 模块级函数**：0 / 13
- **主要功能简介**：`tests/unit/gateway/test_hitl_channel.py` 属于单元测试：隔离模块契约，约 399 行，包含 0 个类与 13 个模块级函数。模块文档写道：「Unit tests for IM channel HITL.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_hitl_resume_projection.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_hitl_resume_projection.py`
- **约行数**：85
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/gateway/test_hitl_resume_projection.py` 属于单元测试：隔离模块契约，约 85 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Dashboard HITL resume keeps the thread-history projection complete.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_inbound_store.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_inbound_store.py`
- **约行数**：106
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/gateway/test_inbound_store.py` 属于单元测试：隔离模块契约，约 106 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Unit tests for inbound attachment storage.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_media_preview.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_media_preview.py`
- **约行数**：74
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/gateway/test_media_preview.py` 属于单元测试：隔离模块契约，约 74 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/unit/test_media_preview.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_mention_agent_calls.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_mention_agent_calls.py`
- **约行数**：58
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/gateway/test_mention_agent_calls.py` 属于单元测试：隔离模块契约，约 58 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Team peer discovery after harness-agent dropped apply_mentions intercept.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_message_build.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_message_build.py`
- **约行数**：108
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/gateway/test_message_build.py` 属于单元测试：隔离模块契约，约 108 行，包含 0 个类与 8 个模块级函数。模块文档写道：「tests/unit/test_message_build.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_message_keys.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_message_keys.py`
- **约行数**：100
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/gateway/test_message_keys.py` 属于单元测试：隔离模块契约，约 100 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Unit tests for IM session/user key derivation.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_preempt_stop_cancel.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_preempt_stop_cancel.py`
- **约行数**：76
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/gateway/test_preempt_stop_cancel.py` 属于单元测试：隔离模块契约，约 76 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Gateway pre-lock /stop cancel preempts the in-flight turn.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_processor_default_open.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_processor_default_open.py`
- **约行数**：226
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/gateway/test_processor_default_open.py` 属于单元测试：隔离模块契约，约 226 行，包含 0 个类与 4 个模块级函数。模块文档写道：「GlobalProcessor injects default_open connectors on IM turns.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_processor_knowledge_services.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_processor_knowledge_services.py`
- **约行数**：28
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/gateway/test_processor_knowledge_services.py` 属于单元测试：隔离模块契约，约 28 行，包含 0 个类与 1 个模块级函数。模块文档写道：「GlobalProcessor knowledge services must carry provider_repo for remote embeds.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_processor_resource_errors.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_processor_resource_errors.py`
- **约行数**：133
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/gateway/test_processor_resource_errors.py` 属于单元测试：隔离模块契约，约 133 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Resource-policy stream error projection tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_processor_trajectory_observe.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_processor_trajectory_observe.py`
- **约行数**：172
- **类 / 模块级函数**：2 / 8
- **主要功能简介**：`tests/unit/gateway/test_processor_trajectory_observe.py` 属于单元测试：隔离模块契约，约 172 行，包含 2 个类与 8 个模块级函数。模块文档写道：「GlobalProcessor side-notifies TrajectoryService without changing stream payloads.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_RecordingService`、`_BoomService`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_response_mode.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_response_mode.py`
- **约行数**：128
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/gateway/test_response_mode.py` 属于单元测试：隔离模块契约，约 128 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Tests for external IM response delivery modes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_slash_catalog.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_slash_catalog.py`
- **约行数**：30
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/gateway/test_slash_catalog.py` 属于单元测试：隔离模块契约，约 30 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/test_slash_catalog.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_slash_compact.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_slash_compact.py`
- **约行数**：287
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`tests/unit/gateway/test_slash_compact.py` 属于单元测试：隔离模块契约，约 287 行，包含 0 个类与 11 个模块级函数。模块文档写道：「Tests for /compact summarization slash command.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_slash_dispatcher.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_slash_dispatcher.py`
- **约行数**：226
- **类 / 模块级函数**：0 / 16
- **主要功能简介**：`tests/unit/gateway/test_slash_dispatcher.py` 属于单元测试：隔离模块契约，约 226 行，包含 0 个类与 16 个模块级函数。模块文档写道：「tests/unit/test_slash_dispatcher.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_slash_help.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_slash_help.py`
- **约行数**：36
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/gateway/test_slash_help.py` 属于单元测试：隔离模块契约，约 36 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/test_slash_help.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_slash_history.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_slash_history.py`
- **约行数**：108
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/gateway/test_slash_history.py` 属于单元测试：隔离模块契约，约 108 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Tests for /history conversation stats.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_slash_parse.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_slash_parse.py`
- **约行数**：44
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/gateway/test_slash_parse.py` 属于单元测试：隔离模块契约，约 44 行，包含 0 个类与 2 个模块级函数。模块文档写道：「tests/unit/gateway/test_slash_parse.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_slash_skills_connectors.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_slash_skills_connectors.py`
- **约行数**：72
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/gateway/test_slash_skills_connectors.py` 属于单元测试：隔离模块契约，约 72 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/unit/test_slash_skills_connectors.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_slash_stop_status.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_slash_stop_status.py`
- **约行数**：111
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/gateway/test_slash_stop_status.py` 属于单元测试：隔离模块契约，约 111 行，包含 0 个类与 4 个模块级函数。模块文档写道：「tests/unit/test_slash_stop_status.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_stream_project_tools.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_stream_project_tools.py`
- **约行数**：32
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/gateway/test_stream_project_tools.py` 属于单元测试：隔离模块契约，约 32 行，包含 0 个类与 3 个模块级函数。模块文档写道：「tests/unit/gateway/test_stream_project_tools.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_thread_registry.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_thread_registry.py`
- **约行数**：332
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`tests/unit/gateway/test_thread_registry.py` 属于单元测试：隔离模块契约，约 332 行，包含 0 个类与 15 个模块级函数。模块文档写道：「tests/unit/test_thread_registry.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_tool_media.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_tool_media.py`
- **约行数**：472
- **类 / 模块级函数**：0 / 20
- **主要功能简介**：`tests/unit/gateway/test_tool_media.py` 属于单元测试：隔离模块契约，约 472 行，包含 0 个类与 20 个模块级函数。模块文档写道：「tests/unit/test_tool_media.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_usage_record.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_usage_record.py`
- **约行数**：181
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/gateway/test_usage_record.py` 属于单元测试：隔离模块契约，约 181 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Turn-level token usage extraction (multi-call agent loops).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_versioned_history.py`

- **所在目录**：`tests/unit/gateway/`
- **相对路径**：`tests/unit/gateway/test_versioned_history.py`
- **约行数**：1055
- **类 / 模块级函数**：0 / 39
- **主要功能简介**：`tests/unit/gateway/test_versioned_history.py` 属于单元测试：隔离模块契约，约 1055 行，包含 0 个类与 39 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/i18n/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_admin_users_expert_naming.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_admin_users_expert_naming.py`
- **约行数**：19
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/i18n/test_admin_users_expert_naming.py` 属于单元测试：隔离模块契约，约 19 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Admin Users UI must call agents 「专家」 / Experts (issue #155).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_agents.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_agents.py`
- **约行数**：52
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/i18n/test_agents.py` 属于单元测试：隔离模块契约，约 52 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Tests for agents i18n domain.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_catalog.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_catalog.py`
- **约行数**：42
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/i18n/test_catalog.py` 属于单元测试：隔离模块契约，约 42 行，包含 0 个类与 6 个模块级函数。模块文档写道：「tests/unit/i18n/test_catalog.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_channel.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_channel.py`
- **约行数**：17
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/i18n/test_channel.py` 属于单元测试：隔离模块契约，约 17 行，包含 0 个类与 2 个模块级函数。模块文档写道：「tests/unit/i18n/test_channel.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_desktop.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_desktop.py`
- **约行数**：20
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/i18n/test_desktop.py` 属于单元测试：隔离模块契约，约 20 行，包含 0 个类与 2 个模块级函数。模块文档写道：「tests/unit/i18n/test_desktop.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_errors.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_errors.py`
- **约行数**：106
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`tests/unit/i18n/test_errors.py` 属于单元测试：隔离模块契约，约 106 行，包含 0 个类与 11 个模块级函数。模块文档写道：「tests/unit/i18n/test_errors.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_mobile.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_mobile.py`
- **约行数**：14
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/i18n/test_mobile.py` 属于单元测试：隔离模块契约，约 14 行，包含 0 个类与 1 个模块级函数。模块文档写道：「tests/unit/i18n/test_mobile.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skills.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_skills.py`
- **约行数**：43
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/i18n/test_skills.py` 属于单元测试：隔离模块契约，约 43 行，包含 0 个类与 7 个模块级函数。模块文档写道：「tests/unit/i18n/test_skills.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_stream.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_stream.py`
- **约行数**：163
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`tests/unit/i18n/test_stream.py` 属于单元测试：隔离模块契约，约 163 行，包含 0 个类与 17 个模块级函数。模块文档写道：「Tests for stream_errors i18n domain.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_tools.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_tools.py`
- **约行数**：82
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/i18n/test_tools.py` 属于单元测试：隔离模块契约，约 82 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/i18n/test_tools.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_voice.py`

- **所在目录**：`tests/unit/i18n/`
- **相对路径**：`tests/unit/i18n/test_voice.py`
- **约行数**：35
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/i18n/test_voice.py` 属于单元测试：隔离模块契约，约 35 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Voice probe / Tencent error copy.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/infra/setup/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_self_update.py`

- **所在目录**：`tests/unit/infra/setup/`
- **相对路径**：`tests/unit/infra/setup/test_self_update.py`
- **约行数**：83
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/infra/setup/test_self_update.py` 属于单元测试：隔离模块契约，约 83 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Tests for octop.infra.setup.self_update.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_service.py`

- **所在目录**：`tests/unit/infra/setup/`
- **相对路径**：`tests/unit/infra/setup/test_service.py`
- **约行数**：878
- **类 / 模块级函数**：1 / 49
- **主要功能简介**：`tests/unit/infra/setup/test_service.py` 属于单元测试：隔离模块契约，约 878 行，包含 1 个类与 49 个模块级函数。模块文档写道：「Tests for octop.infra.setup.service.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FakeOs`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `tests/unit/infra/setup/tls/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_challenge.py`

- **所在目录**：`tests/unit/infra/setup/tls/`
- **相对路径**：`tests/unit/infra/setup/tls/test_challenge.py`
- **约行数**：15
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/infra/setup/tls/test_challenge.py` 属于单元测试：隔离模块契约，约 15 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Unit tests for ACME challenge store.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_listeners.py`

- **所在目录**：`tests/unit/infra/setup/tls/`
- **相对路径**：`tests/unit/infra/setup/tls/test_listeners.py`
- **约行数**：40
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/infra/setup/tls/test_listeners.py` 属于单元测试：隔离模块契约，约 40 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Unit tests for dual-port listen plan.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_modes.py`

- **所在目录**：`tests/unit/infra/setup/tls/`
- **相对路径**：`tests/unit/infra/setup/tls/test_modes.py`
- **约行数**：49
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/infra/setup/tls/test_modes.py` 属于单元测试：隔离模块契约，约 49 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for TLS issue mode helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_preflight.py`

- **所在目录**：`tests/unit/infra/setup/tls/`
- **相对路径**：`tests/unit/infra/setup/tls/test_preflight.py`
- **约行数**：81
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/infra/setup/tls/test_preflight.py` 属于单元测试：隔离模块契约，约 81 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Unit tests for TLS preflight checks.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_renewal.py`

- **所在目录**：`tests/unit/infra/setup/tls/`
- **相对路径**：`tests/unit/infra/setup/tls/test_renewal.py`
- **约行数**：22
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/infra/setup/tls/test_renewal.py` 属于单元测试：隔离模块契约，约 22 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Unit tests for TLS auto-renewal helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_tls_store.py`

- **所在目录**：`tests/unit/infra/setup/tls/`
- **相对路径**：`tests/unit/infra/setup/tls/test_tls_store.py`
- **约行数**：46
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/infra/setup/tls/test_tls_store.py` 属于单元测试：隔离模块契约，约 46 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Unit tests for TLS certificate storage and config updates.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/infra/utils/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_bwrap.py`

- **所在目录**：`tests/unit/infra/utils/`
- **相对路径**：`tests/unit/infra/utils/test_bwrap.py`
- **约行数**：142
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`tests/unit/infra/utils/test_bwrap.py` 属于单元测试：隔离模块契约，约 142 行，包含 0 个类与 11 个模块级函数。模块文档写道：「Tests for best-effort bubblewrap (``bwrap``) provisioning — all platforms.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_docker_env.py`

- **所在目录**：`tests/unit/infra/utils/`
- **相对路径**：`tests/unit/infra/utils/test_docker_env.py`
- **约行数**：96
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/infra/utils/test_docker_env.py` 属于单元测试：隔离模块契约，约 96 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Tests for Docker environment detect / ensure helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_host_dirs.py`

- **所在目录**：`tests/unit/infra/utils/`
- **相对路径**：`tests/unit/infra/utils/test_host_dirs.py`
- **约行数**：320
- **类 / 模块级函数**：0 / 28
- **主要功能简介**：`tests/unit/infra/utils/test_host_dirs.py` 属于单元测试：隔离模块契约，约 320 行，包含 0 个类与 28 个模块级函数。模块文档写道：「Tests for host directory listing helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_json_file.py`

- **所在目录**：`tests/unit/infra/utils/`
- **相对路径**：`tests/unit/infra/utils/test_json_file.py`
- **约行数**：108
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/infra/utils/test_json_file.py` 属于单元测试：隔离模块契约，约 108 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Unit tests for the shared JSON config helpers (issue #730).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_ssl_errors.py`

- **所在目录**：`tests/unit/infra/utils/`
- **相对路径**：`tests/unit/infra/utils/test_ssl_errors.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/infra/utils/test_ssl_errors.py` 属于单元测试：隔离模块契约，约 29 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for SSL error detection helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_subprocess_io.py`

- **所在目录**：`tests/unit/infra/utils/`
- **相对路径**：`tests/unit/infra/utils/test_subprocess_io.py`
- **约行数**：36
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/infra/utils/test_subprocess_io.py` 属于单元测试：隔离模块契约，约 36 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Unit tests for cross-platform subprocess stdout reads.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_utf8_text.py`

- **所在目录**：`tests/unit/infra/utils/`
- **相对路径**：`tests/unit/infra/utils/test_utf8_text.py`
- **约行数**：62
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/infra/utils/test_utf8_text.py` 属于单元测试：隔离模块契约，约 62 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Tests for UTF-8 coercion of third-party package text files.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/knowledge/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_chunk.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_chunk.py`
- **约行数**：20
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/knowledge/test_chunk.py` 属于单元测试：隔离模块契约，约 20 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Unit tests for document chunking.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_citations.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_citations.py`
- **约行数**：96
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/knowledge/test_citations.py` 属于单元测试：隔离模块契约，约 96 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for knowledge citation markers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_embed.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_embed.py`
- **约行数**：146
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/knowledge/test_embed.py` 属于单元测试：隔离模块契约，约 146 行，包含 0 个类与 3 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_gate.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_gate.py`
- **约行数**：120
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/knowledge/test_gate.py` 属于单元测试：隔离模块契约，约 120 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Unit tests for the knowledge-base capability gate.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_index.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_index.py`
- **约行数**：32
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/knowledge/test_index.py` 属于单元测试：隔离模块契约，约 32 行，包含 0 个类与 1 个模块级函数。模块文档写道：「Unit tests for per-knowledge-base sidecar indexes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_jobs.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_jobs.py`
- **约行数**：196
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/knowledge/test_jobs.py` 属于单元测试：隔离模块契约，约 196 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Unit tests for in-process document indexing jobs.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_knowledge_default_open.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_knowledge_default_open.py`
- **约行数**：89
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/knowledge/test_knowledge_default_open.py` 属于单元测试：隔离模块契约，约 89 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for knowledge-base turn selection defaults.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_knowledge_hint.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_knowledge_hint.py`
- **约行数**：139
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/knowledge/test_knowledge_hint.py` 属于单元测试：隔离模块契约，约 139 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Unit tests for knowledge-base tool description enrichment.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_knowledge_tools.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_knowledge_tools.py`
- **约行数**：97
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/knowledge/test_knowledge_tools.py` 属于单元测试：隔离模块契约，约 97 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Unit tests for built-in knowledge-base LangChain tools.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_ocr.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_ocr.py`
- **约行数**：157
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/knowledge/test_ocr.py` 属于单元测试：隔离模块契约，约 157 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Tests for optional knowledge-base OCR configuration and routing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_params.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_params.py`
- **约行数**：55
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/knowledge/test_params.py` 属于单元测试：隔离模块契约，约 55 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Unit tests for knowledge advanced indexing/retrieval settings.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_parse.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_parse.py`
- **约行数**：225
- **类 / 模块级函数**：0 / 11
- **主要功能简介**：`tests/unit/knowledge/test_parse.py` 属于单元测试：隔离模块契约，约 225 行，包含 0 个类与 11 个模块级函数。模块文档写道：「Unit tests for supported knowledge-document parsers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_relpath.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_relpath.py`
- **约行数**：22
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/knowledge/test_relpath.py` 属于单元测试：隔离模块契约，约 22 行，包含 0 个类与 2 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_retrieve.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_retrieve.py`
- **约行数**：82
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/knowledge/test_retrieve.py` 属于单元测试：隔离模块契约，约 82 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Unit tests for knowledge retrieval during a chat turn.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_service_acl.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_service_acl.py`
- **约行数**：280
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`tests/unit/knowledge/test_service_acl.py` 属于单元测试：隔离模块契约，约 280 行，包含 0 个类与 15 个模块级函数。模块文档写道：「Unit tests for knowledge-base access control and document orchestration.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_text_documents.py`

- **所在目录**：`tests/unit/knowledge/`
- **相对路径**：`tests/unit/knowledge/test_text_documents.py`
- **约行数**：170
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/knowledge/test_text_documents.py` 属于单元测试：隔离模块契约，约 170 行，包含 0 个类与 4 个模块级函数。模块文档写道：「KnowledgeService text create / update helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/mobile/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_agent_control.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_agent_control.py`
- **约行数**：41
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/mobile/test_agent_control.py` 属于单元测试：隔离模块契约，约 41 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for mobile agent-control binding.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_config_probe.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_config_probe.py`
- **约行数**：23
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/mobile/test_config_probe.py` 属于单元测试：隔离模块契约，约 23 行，包含 0 个类与 1 个模块级函数。模块文档写道：「tests/unit/mobile/test_config_probe.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_device_info.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_device_info.py`
- **约行数**：52
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/mobile/test_device_info.py` 属于单元测试：隔离模块契约，约 52 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Unit tests for adb device-info parsing.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_docker_install.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_docker_install.py`
- **约行数**：342
- **类 / 模块级函数**：0 / 18
- **主要功能简介**：`tests/unit/mobile/test_docker_install.py` 属于单元测试：隔离模块契约，约 342 行，包含 0 个类与 18 个模块级函数。模块文档写道：「tests/unit/mobile/test_docker_install.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_find_adb.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_find_adb.py`
- **约行数**：74
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/mobile/test_find_adb.py` 属于单元测试：隔离模块契约，约 74 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Unit tests for adb discovery (PATH + SDK env only).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_h264.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_h264.py`
- **约行数**：87
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/mobile/test_h264.py` 属于单元测试：隔离模块契约，约 87 行，包含 0 个类与 5 个模块级函数。模块文档写道：「Unit tests for mobile capture / H.264 helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_mobile_setup.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_mobile_setup.py`
- **约行数**：69
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/mobile/test_mobile_setup.py` 属于单元测试：隔离模块契约，约 69 行，包含 0 个类与 6 个模块级函数。模块文档写道：「tests/unit/mobile/test_mobile_setup.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_mobile_tools.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_mobile_tools.py`
- **约行数**：116
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/mobile/test_mobile_tools.py` 属于单元测试：隔离模块契约，约 116 行，包含 0 个类与 8 个模块级函数。模块文档写道：「Unit tests for built-in mobile LangChain tools.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_probe.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_probe.py`
- **约行数**：38
- **类 / 模块级函数**：0 / 1
- **主要功能简介**：`tests/unit/mobile/test_probe.py` 属于单元测试：隔离模块契约，约 38 行，包含 0 个类与 1 个模块级函数。模块文档写道：「tests/unit/mobile/test_probe.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_rotation.py`

- **所在目录**：`tests/unit/mobile/`
- **相对路径**：`tests/unit/mobile/test_rotation.py`
- **约行数**：138
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/mobile/test_rotation.py` 属于单元测试：隔离模块契约，约 138 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Unit tests for adb device rotation helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/proactive/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_picker.py`

- **所在目录**：`tests/unit/proactive/`
- **相对路径**：`tests/unit/proactive/test_picker.py`
- **约行数**：241
- **类 / 模块级函数**：0 / 19
- **主要功能简介**：`tests/unit/proactive/test_picker.py` 属于单元测试：隔离模块契约，约 241 行，包含 0 个类与 19 个模块级函数。模块文档写道：「Unit tests for EpisodePicker.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_proactive_service.py`

- **所在目录**：`tests/unit/proactive/`
- **相对路径**：`tests/unit/proactive/test_proactive_service.py`
- **约行数**：302
- **类 / 模块级函数**：0 / 15
- **主要功能简介**：`tests/unit/proactive/test_proactive_service.py` 属于单元测试：隔离模块契约，约 302 行，包含 0 个类与 15 个模块级函数。模块文档写道：「Unit tests for ProactiveCareService.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_scheduler.py`

- **所在目录**：`tests/unit/proactive/`
- **相对路径**：`tests/unit/proactive/test_scheduler.py`
- **约行数**：429
- **类 / 模块级函数**：0 / 25
- **主要功能简介**：`tests/unit/proactive/test_scheduler.py` 属于单元测试：隔离模块契约，约 429 行，包含 0 个类与 25 个模块级函数。模块文档写道：「Unit tests for ProactiveCareScheduler.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/providers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_codex_oauth.py`

- **所在目录**：`tests/unit/providers/`
- **相对路径**：`tests/unit/providers/test_codex_oauth.py`
- **约行数**：100
- **类 / 模块级函数**：1 / 6
- **主要功能简介**：`tests/unit/providers/test_codex_oauth.py` 属于单元测试：隔离模块契约，约 100 行，包含 1 个类与 6 个模块级函数。模块文档写道：「Unit tests for Codex OAuth helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_Resp`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_opencode_session.py`

- **所在目录**：`tests/unit/providers/`
- **相对路径**：`tests/unit/providers/test_opencode_session.py`
- **约行数**：159
- **类 / 模块级函数**：1 / 12
- **主要功能简介**：`tests/unit/providers/test_opencode_session.py` 属于单元测试：隔离模块契约，约 159 行，包含 1 个类与 12 个模块级函数。模块文档写道：「Unit tests for OpenCode Go session-header wiring.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FakeRepo`。类型字段与方法的设计含义见设计说明书对应符号节。


### 目录 `tests/unit/skills/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_presentation.py`

- **所在目录**：`tests/unit/skills/`
- **相对路径**：`tests/unit/skills/test_presentation.py`
- **约行数**：74
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/skills/test_presentation.py` 属于单元测试：隔离模块契约，约 74 行，包含 0 个类与 4 个模块级函数。该文件未写模块级 docstring，需以导出符号为准。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_skill_install.py`

- **所在目录**：`tests/unit/skills/`
- **相对路径**：`tests/unit/skills/test_skill_install.py`
- **约行数**：99
- **类 / 模块级函数**：1 / 4
- **主要功能简介**：`tests/unit/skills/test_skill_install.py` 属于单元测试：隔离模块契约，约 99 行，包含 1 个类与 4 个模块级函数。模块文档写道：「Unit tests for shared skill install target pipeline.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`FakeTarget`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_skill_package_from_skillhub.py`

- **所在目录**：`tests/unit/skills/`
- **相对路径**：`tests/unit/skills/test_skill_package_from_skillhub.py`
- **约行数**：136
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/skills/test_skill_package_from_skillhub.py` 属于单元测试：隔离模块契约，约 136 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Tests for materializing SkillHub skillsets as global packages.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_workspace_catalog.py`

- **所在目录**：`tests/unit/skills/`
- **相对路径**：`tests/unit/skills/test_workspace_catalog.py`
- **约行数**：82
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/skills/test_workspace_catalog.py` 属于单元测试：隔离模块契约，约 82 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for symlink-tolerant workspace skill discovery.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/trajectory/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_live.py`

- **所在目录**：`tests/unit/trajectory/`
- **相对路径**：`tests/unit/trajectory/test_live.py`
- **约行数**：39
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/trajectory/test_live.py` 属于单元测试：隔离模块契约，约 39 行，包含 0 个类与 3 个模块级函数。模块文档写道：「TrajectoryLiveBus — in-process pub/sub per thread_id.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_metrics.py`

- **所在目录**：`tests/unit/trajectory/`
- **相对路径**：`tests/unit/trajectory/test_metrics.py`
- **约行数**：95
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/trajectory/test_metrics.py` 属于单元测试：隔离模块契约，约 95 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Pure metrics aggregation from trajectory events.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_projector.py`

- **所在目录**：`tests/unit/trajectory/`
- **相对路径**：`tests/unit/trajectory/test_projector.py`
- **约行数**：228
- **类 / 模块级函数**：0 / 14
- **主要功能简介**：`tests/unit/trajectory/test_projector.py` 属于单元测试：隔离模块契约，约 228 行，包含 0 个类与 14 个模块级函数。模块文档写道：「Harness/session-fact projector tests.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_trajectory_event_repo.py`

- **所在目录**：`tests/unit/trajectory/`
- **相对路径**：`tests/unit/trajectory/test_trajectory_event_repo.py`
- **约行数**：161
- **类 / 模块级函数**：0 / 8
- **主要功能简介**：`tests/unit/trajectory/test_trajectory_event_repo.py` 属于单元测试：隔离模块契约，约 161 行，包含 0 个类与 8 个模块级函数。模块文档写道：「TrajectoryEventRepo — append, cursor page, duplicate event_id.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_trajectory_list_summarize.py`

- **所在目录**：`tests/unit/trajectory/`
- **相对路径**：`tests/unit/trajectory/test_trajectory_list_summarize.py`
- **约行数**：87
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/trajectory/test_trajectory_list_summarize.py` 属于单元测试：隔离模块契约，约 87 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Unit tests for trajectory list payload projection.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_trajectory_service.py`

- **所在目录**：`tests/unit/trajectory/`
- **相对路径**：`tests/unit/trajectory/test_trajectory_service.py`
- **约行数**：364
- **类 / 模块级函数**：0 / 17
- **主要功能简介**：`tests/unit/trajectory/test_trajectory_service.py` 属于单元测试：隔离模块契约，约 364 行，包含 0 个类与 17 个模块级函数。模块文档写道：「TrajectoryService — observe never raises; list / metrics / export.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_trajectory_settings.py`

- **所在目录**：`tests/unit/trajectory/`
- **相对路径**：`tests/unit/trajectory/test_trajectory_settings.py`
- **约行数**：51
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/trajectory/test_trajectory_settings.py` 属于单元测试：隔离模块契约，约 51 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Per-agent trajectory flag and persist clipping.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_trajectory_store.py`

- **所在目录**：`tests/unit/trajectory/`
- **相对路径**：`tests/unit/trajectory/test_trajectory_store.py`
- **约行数**：93
- **类 / 模块级函数**：0 / 5
- **主要功能简介**：`tests/unit/trajectory/test_trajectory_store.py` 属于单元测试：隔离模块契约，约 93 行，包含 0 个类与 5 个模块级函数。模块文档写道：「TrajectoryStore — thin repo wrapper; duplicate event_id is a no-op.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_turn_context.py`

- **所在目录**：`tests/unit/trajectory/`
- **相对路径**：`tests/unit/trajectory/test_turn_context.py`
- **约行数**：101
- **类 / 模块级函数**：0 / 6
- **主要功能简介**：`tests/unit/trajectory/test_turn_context.py` 属于单元测试：隔离模块契约，约 101 行，包含 0 个类与 6 个模块级函数。模块文档写道：「Turn-start SYSTEM / CONTEXT synthesis — must match harness injection SoT.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/users/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_email.py`

- **所在目录**：`tests/unit/users/`
- **相对路径**：`tests/unit/users/test_email.py`
- **约行数**：32
- **类 / 模块级函数**：0 / 3
- **主要功能简介**：`tests/unit/users/test_email.py` 属于单元测试：隔离模块契约，约 32 行，包含 0 个类与 3 个模块级函数。模块文档写道：「Tests for email normalize / validate helpers.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_invites.py`

- **所在目录**：`tests/unit/users/`
- **相对路径**：`tests/unit/users/test_invites.py`
- **约行数**：89
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/users/test_invites.py` 属于单元测试：隔离模块契约，约 89 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for invite code helpers and repo redeem.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_permissions.py`

- **所在目录**：`tests/unit/users/`
- **相对路径**：`tests/unit/users/test_permissions.py`
- **约行数**：91
- **类 / 模块级函数**：1 / 7
- **主要功能简介**：`tests/unit/users/test_permissions.py` 属于单元测试：隔离模块契约，约 91 行，包含 1 个类与 7 个模块级函数。模块文档写道：「Unit tests for the module permission catalog.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

主要类型包括：`_FakeUser`。类型字段与方法的设计含义见设计说明书对应符号节。

#### `test_preferences.py`

- **所在目录**：`tests/unit/users/`
- **相对路径**：`tests/unit/users/test_preferences.py`
- **约行数**：114
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/users/test_preferences.py` 属于单元测试：隔离模块契约，约 114 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/users/test_preferences.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_resource_policy.py`

- **所在目录**：`tests/unit/users/`
- **相对路径**：`tests/unit/users/test_resource_policy.py`
- **约行数**：273
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/users/test_resource_policy.py` 属于单元测试：隔离模块契约，约 273 行，包含 0 个类与 10 个模块级函数。模块文档写道：「tests/unit/users/test_resource_policy.py」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


### 目录 `tests/unit/utils/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `test_doc_edit.py`

- **所在目录**：`tests/unit/utils/`
- **相对路径**：`tests/unit/utils/test_doc_edit.py`
- **约行数**：120
- **类 / 模块级函数**：0 / 10
- **主要功能简介**：`tests/unit/utils/test_doc_edit.py` 属于单元测试：隔离模块契约，约 120 行，包含 0 个类与 10 个模块级函数。模块文档写道：「Tests for the editable-document registry and its Markdown round-trip.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_posix_compat.py`

- **所在目录**：`tests/unit/utils/`
- **相对路径**：`tests/unit/utils/test_posix_compat.py`
- **约行数**：29
- **类 / 模块级函数**：0 / 2
- **主要功能简介**：`tests/unit/utils/test_posix_compat.py` 属于单元测试：隔离模块契约，约 29 行，包含 0 个类与 2 个模块级函数。模块文档写道：「Tests for POSIX compat helpers used by the web terminal.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_runtime_packages.py`

- **所在目录**：`tests/unit/utils/`
- **相对路径**：`tests/unit/utils/test_runtime_packages.py`
- **约行数**：126
- **类 / 模块级函数**：0 / 9
- **主要功能简介**：`tests/unit/utils/test_runtime_packages.py` 属于单元测试：隔离模块契约，约 126 行，包含 0 个类与 9 个模块级函数。模块文档写道：「Tests for runtime optional Python package installation.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_search_probe.py`

- **所在目录**：`tests/unit/utils/`
- **相对路径**：`tests/unit/utils/test_search_probe.py`
- **约行数**：93
- **类 / 模块级函数**：0 / 7
- **主要功能简介**：`tests/unit/utils/test_search_probe.py` 属于单元测试：隔离模块契约，约 93 行，包含 0 个类与 7 个模块级函数。模块文档写道：「Unit tests for search-provider HTTP probes.」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。

#### `test_ssrf_guard.py`

- **所在目录**：`tests/unit/utils/`
- **相对路径**：`tests/unit/utils/test_ssrf_guard.py`
- **约行数**：52
- **类 / 模块级函数**：0 / 4
- **主要功能简介**：`tests/unit/utils/test_ssrf_guard.py` 属于单元测试：隔离模块契约，约 52 行，包含 0 个类与 4 个模块级函数。模块文档写道：「Unit tests for SSRF guard (infra/utils/ssrf_guard.py).」。 该文件是 Octop 单进程架构中的一环：控制台与 CLI 最终都汇入同一套 `OctopServer` / `SharedServices`。修改本文件时必须遵守 AGENTS.md 的模块边界，并保证 `make all`（format-all、lint、typecheck、pytest）保持绿色。跨平台测试不得硬编码 POSIX 路径；若行为仅在 Linux/macOS 成立，应使用 posix_only 标记。


## 2.3 控制台 `dashboard/src`


### 目录 `dashboard/src/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `App.tsx`

- **所在目录**：`dashboard/src/`
- **相对路径**：`dashboard/src/App.tsx`
- **约行数**：177
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 177 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `i18n.ts`

- **所在目录**：`dashboard/src/`
- **相对路径**：`dashboard/src/i18n.ts`
- **约行数**：127
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 127 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `main.tsx`

- **所在目录**：`dashboard/src/`
- **相对路径**：`dashboard/src/main.tsx`
- **约行数**：80
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 80 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `pwa-prompt.ts`

- **所在目录**：`dashboard/src/`
- **相对路径**：`dashboard/src/pwa-prompt.ts`
- **约行数**：141
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 141 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `pwa.ts`

- **所在目录**：`dashboard/src/`
- **相对路径**：`dashboard/src/pwa.ts`
- **约行数**：15
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 15 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sw-register.test.ts`

- **所在目录**：`dashboard/src/`
- **相对路径**：`dashboard/src/sw-register.test.ts`
- **约行数**：183
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 183 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sw-register.ts`

- **所在目录**：`dashboard/src/`
- **相对路径**：`dashboard/src/sw-register.ts`
- **约行数**：130
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 130 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `vite-env.d.ts`

- **所在目录**：`dashboard/src/`
- **相对路径**：`dashboard/src/vite-env.d.ts`
- **约行数**：7
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 7 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/api/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `config.ts`

- **所在目录**：`dashboard/src/api/`
- **相对路径**：`dashboard/src/api/config.ts`
- **约行数**：39
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 39 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/api/`
- **相对路径**：`dashboard/src/api/index.ts`
- **约行数**：90
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 90 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `probeHealth.ts`

- **所在目录**：`dashboard/src/api/`
- **相对路径**：`dashboard/src/api/probeHealth.ts`
- **约行数**：44
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 44 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `request.setup.test.ts`

- **所在目录**：`dashboard/src/api/`
- **相对路径**：`dashboard/src/api/request.setup.test.ts`
- **约行数**：96
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 96 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `request.ts`

- **所在目录**：`dashboard/src/api/`
- **相对路径**：`dashboard/src/api/request.ts`
- **约行数**：600
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 600 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `request.unauthorized.test.ts`

- **所在目录**：`dashboard/src/api/`
- **相对路径**：`dashboard/src/api/request.unauthorized.test.ts`
- **约行数**：72
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 72 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/api/modules/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `acp.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/acp.ts`
- **约行数**：68
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 68 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `agent.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/agent.ts`
- **约行数**：146
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 146 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `agentChat.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/agentChat.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `agentTools.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/agentTools.ts`
- **约行数**：81
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 81 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `auth.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/auth.ts`
- **约行数**：276
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 276 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `backup.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/backup.ts`
- **约行数**：123
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 123 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `browser.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/browser.ts`
- **约行数**：221
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 221 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `channel.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/channel.ts`
- **约行数**：269
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 269 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `connectors.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/connectors.ts`
- **约行数**：392
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 392 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `cronjob.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/cronjob.ts`
- **约行数**：52
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 52 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `desktop.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/desktop.ts`
- **约行数**：112
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 112 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `embedding.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/embedding.ts`
- **约行数**：50
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 50 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `env.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/env.ts`
- **约行数**：22
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 22 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `expertMarket.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/expertMarket.ts`
- **约行数**：105
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 105 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `i18n.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/i18n.ts`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `invites.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/invites.ts`
- **约行数**：77
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 77 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeBases.test.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/knowledgeBases.test.ts`
- **约行数**：103
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 103 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeBases.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/knowledgeBases.ts`
- **约行数**：318
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 318 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mbti.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/mbti.ts`
- **约行数**：56
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 56 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mediaGeneration.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/mediaGeneration.ts`
- **约行数**：49
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 49 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `memoryDashboard.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/memoryDashboard.ts`
- **约行数**：533
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 533 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `memoryPortable.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/memoryPortable.ts`
- **约行数**：149
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 149 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mobile.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/mobile.ts`
- **约行数**：115
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 115 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `observability.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/observability.ts`
- **约行数**：36
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 36 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `octopAgents.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/octopAgents.ts`
- **约行数**：23
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 23 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `octopThreads.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/octopThreads.ts`
- **约行数**：227
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 227 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ollamaModel.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/ollamaModel.ts`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `onnxDownloadWatcher.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/onnxDownloadWatcher.ts`
- **约行数**：117
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 117 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `onnxModel.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/onnxModel.ts`
- **约行数**：91
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 91 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `plugins.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/plugins.ts`
- **约行数**：144
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 144 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `preferences.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/preferences.ts`
- **约行数**：40
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 40 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `provider.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/provider.ts`
- **约行数**：121
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 121 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `publishedExperts.test.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/publishedExperts.test.ts`
- **约行数**：75
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 75 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `publishedExperts.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/publishedExperts.ts`
- **约行数**：86
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 86 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `root.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/root.ts`
- **约行数**：8
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 8 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `security.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/security.ts`
- **约行数**：122
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 122 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `settings.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/settings.ts`
- **约行数**：57
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 57 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillPackages.test.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/skillPackages.test.ts`
- **约行数**：125
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 125 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillPackages.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/skillPackages.ts`
- **约行数**：151
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 151 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `slash.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/slash.ts`
- **约行数**：36
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 36 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sso.test.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/sso.test.ts`
- **约行数**：38
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 38 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sso.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/sso.ts`
- **约行数**：97
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 97 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `subagents.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/subagents.ts`
- **约行数**：77
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 77 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `terminalAi.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/terminalAi.ts`
- **约行数**：31
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 31 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `tls.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/tls.ts`
- **约行数**：53
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 53 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `trajectory.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/trajectory.ts`
- **约行数**：112
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 112 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `update.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/update.ts`
- **约行数**：68
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 68 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `upload.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/upload.ts`
- **约行数**：27
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 27 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `voice.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/voice.ts`
- **约行数**：136
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 136 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `workspace.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/workspace.ts`
- **约行数**：227
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 227 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `wsChat.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/wsChat.ts`
- **约行数**：11
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 11 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `wsNotifications.ts`

- **所在目录**：`dashboard/src/api/modules/`
- **相对路径**：`dashboard/src/api/modules/wsNotifications.ts`
- **约行数**：11
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/modules` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 11 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/api/types/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `acp.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/acp.ts`
- **约行数**：52
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 52 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `agent.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/agent.ts`
- **约行数**：99
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 99 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `browser.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/browser.ts`
- **约行数**：112
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 112 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `channel.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/channel.ts`
- **约行数**：115
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 115 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chat.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/chat.ts`
- **约行数**：88
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 88 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `cronjob.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/cronjob.ts`
- **约行数**：110
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 110 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `embedding.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/embedding.ts`
- **约行数**：37
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 37 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `env.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/env.ts`
- **约行数**：5
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 5 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `hitl.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/hitl.ts`
- **约行数**：73
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 73 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/index.ts`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mbti.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/mbti.ts`
- **约行数**：63
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 63 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `provider.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/provider.ts`
- **约行数**：178
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 178 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skill.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/skill.ts`
- **约行数**：56
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 56 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillPackage.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/skillPackage.ts`
- **约行数**：67
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 67 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `workspace.ts`

- **所在目录**：`dashboard/src/api/types/`
- **相对路径**：`dashboard/src/api/types/workspace.ts`
- **约行数**：57
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/api/types` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 57 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/assets/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `mascot.ts`

- **所在目录**：`dashboard/src/assets/`
- **相对路径**：`dashboard/src/assets/mascot.ts`
- **约行数**：6
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/assets` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 6 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/assets/connectors/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.ts`

- **所在目录**：`dashboard/src/assets/connectors/`
- **相对路径**：`dashboard/src/assets/connectors/index.ts`
- **约行数**：67
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/assets/connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 67 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/assets/providers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.ts`

- **所在目录**：`dashboard/src/assets/providers/`
- **相对路径**：`dashboard/src/assets/providers/index.ts`
- **约行数**：150
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/assets/providers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 150 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AgentAdvancedConfigFields.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/AgentAdvancedConfigFields.tsx`
- **约行数**：95
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 95 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AgentProfileDrawer.test.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/AgentProfileDrawer.test.tsx`
- **约行数**：166
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 166 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AgentProfileDrawer.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/AgentProfileDrawer.tsx`
- **约行数**：698
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 698 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AgentSelector.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/AgentSelector.tsx`
- **约行数**：156
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 156 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AntdAppProvider.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/AntdAppProvider.tsx`
- **约行数**：29
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 29 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AuthFileDownloadLink.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/AuthFileDownloadLink.tsx`
- **约行数**：145
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 145 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AuthGuard.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/AuthGuard.tsx`
- **约行数**：115
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 115 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AuthImage.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/AuthImage.tsx`
- **约行数**：38
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 38 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AvatarDropdown.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/AvatarDropdown.tsx`
- **约行数**：822
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 822 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `BackendBuilder.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/BackendBuilder.tsx`
- **约行数**：433
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 433 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `BrowserAiPanel.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/BrowserAiPanel.tsx`
- **约行数**：705
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 705 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `CopyableResourceId.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/CopyableResourceId.tsx`
- **约行数**：56
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 56 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `DocumentPreviewCore.docxSanitize.test.ts`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/DocumentPreviewCore.docxSanitize.test.ts`
- **约行数**：23
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 23 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `DocumentPreviewCore.pdfSkeleton.test.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/DocumentPreviewCore.pdfSkeleton.test.tsx`
- **约行数**：31
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 31 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `DocumentPreviewCore.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/DocumentPreviewCore.tsx`
- **约行数**：753
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 753 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `DocumentPreviewLoading.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/DocumentPreviewLoading.tsx`
- **约行数**：59
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 59 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `EmojiPicker.test.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/EmojiPicker.test.tsx`
- **约行数**：25
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 25 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `EmojiPicker.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/EmojiPicker.tsx`
- **约行数**：104
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 104 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExpertColorPicker.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/ExpertColorPicker.tsx`
- **约行数**：85
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 85 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ForbiddenPage.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/ForbiddenPage.tsx`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `LanguageSwitcher.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/LanguageSwitcher.tsx`
- **约行数**：76
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 76 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MbtiPersonaTag.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/MbtiPersonaTag.tsx`
- **约行数**：100
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 100 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MobileAiPanel.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/MobileAiPanel.tsx`
- **约行数**：226
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 226 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `NotFoundPage.test.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/NotFoundPage.test.tsx`
- **约行数**：27
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 27 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `NotFoundPage.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/NotFoundPage.tsx`
- **约行数**：24
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 24 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PaletteSwitcher.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/PaletteSwitcher.tsx`
- **约行数**：71
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 71 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PdfDocumentPreview.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/PdfDocumentPreview.tsx`
- **约行数**：993
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 993 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PdfDocumentPreview.windowed.test.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/PdfDocumentPreview.windowed.test.tsx`
- **约行数**：364
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 364 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `RailEdgeControl.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/RailEdgeControl.tsx`
- **约行数**：88
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 88 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `RequirePermission.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/RequirePermission.tsx`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ServiceRestartOverlay.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/ServiceRestartOverlay.tsx`
- **约行数**：160
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 160 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillRecordGuideModal.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/SkillRecordGuideModal.tsx`
- **约行数**：146
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 146 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ThemeSwitcher.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/ThemeSwitcher.tsx`
- **约行数**：73
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 73 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TodoListInline.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/TodoListInline.tsx`
- **约行数**：26
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 26 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TodoProgressPanel.tsx`

- **所在目录**：`dashboard/src/components/`
- **相对路径**：`dashboard/src/components/TodoProgressPanel.tsx`
- **约行数**：116
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 116 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/AppVersionBadge/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/AppVersionBadge/`
- **相对路径**：`dashboard/src/components/AppVersionBadge/index.tsx`
- **约行数**：48
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/AppVersionBadge` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 48 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/BetaBadge/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/BetaBadge/`
- **相对路径**：`dashboard/src/components/BetaBadge/index.tsx`
- **约行数**：25
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/BetaBadge` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 25 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/BrowserViewer/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/BrowserViewer/`
- **相对路径**：`dashboard/src/components/BrowserViewer/index.tsx`
- **约行数**：348
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/BrowserViewer` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 348 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/BrowserWorkspace/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ChatBrowserPanel.tsx`

- **所在目录**：`dashboard/src/components/BrowserWorkspace/`
- **相对路径**：`dashboard/src/components/BrowserWorkspace/ChatBrowserPanel.tsx`
- **约行数**：58
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/BrowserWorkspace` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 58 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatDockPanelShell.test.tsx`

- **所在目录**：`dashboard/src/components/BrowserWorkspace/`
- **相对路径**：`dashboard/src/components/BrowserWorkspace/ChatDockPanelShell.test.tsx`
- **约行数**：86
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/BrowserWorkspace` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 86 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatDockPanelShell.tsx`

- **所在目录**：`dashboard/src/components/BrowserWorkspace/`
- **相对路径**：`dashboard/src/components/BrowserWorkspace/ChatDockPanelShell.tsx`
- **约行数**：569
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/BrowserWorkspace` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 569 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/BrowserWorkspace/`
- **相对路径**：`dashboard/src/components/BrowserWorkspace/index.tsx`
- **约行数**：453
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/BrowserWorkspace` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 453 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/CatalogTypeCard/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/CatalogTypeCard/`
- **相对路径**：`dashboard/src/components/CatalogTypeCard/index.tsx`
- **约行数**：60
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/CatalogTypeCard` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 60 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/ChatPicker/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `SearchablePickerPanel.tsx`

- **所在目录**：`dashboard/src/components/ChatPicker/`
- **相对路径**：`dashboard/src/components/ChatPicker/SearchablePickerPanel.tsx`
- **约行数**：69
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/ChatPicker` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 69 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/ChromeTabBar/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/ChromeTabBar/`
- **相对路径**：`dashboard/src/components/ChromeTabBar/index.tsx`
- **约行数**：109
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/ChromeTabBar` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 109 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/CurrentVersionBadge/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/CurrentVersionBadge/`
- **相对路径**：`dashboard/src/components/CurrentVersionBadge/index.tsx`
- **约行数**：60
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/CurrentVersionBadge` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 60 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/DesktopWindowControls/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `DesktopWindowControls.test.tsx`

- **所在目录**：`dashboard/src/components/DesktopWindowControls/`
- **相对路径**：`dashboard/src/components/DesktopWindowControls/DesktopWindowControls.test.tsx`
- **约行数**：45
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/DesktopWindowControls` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 45 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `DesktopWindowControls.tsx`

- **所在目录**：`dashboard/src/components/DesktopWindowControls/`
- **相对路径**：`dashboard/src/components/DesktopWindowControls/DesktopWindowControls.tsx`
- **约行数**：93
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/DesktopWindowControls` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 93 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/components/DesktopWindowControls/`
- **相对路径**：`dashboard/src/components/DesktopWindowControls/index.ts`
- **约行数**：2
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/DesktopWindowControls` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 2 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/EmptyState/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `OctopEmptyMascot.tsx`

- **所在目录**：`dashboard/src/components/EmptyState/`
- **相对路径**：`dashboard/src/components/EmptyState/OctopEmptyMascot.tsx`
- **约行数**：29
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/EmptyState` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 29 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/EmptyState/`
- **相对路径**：`dashboard/src/components/EmptyState/index.tsx`
- **约行数**：105
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/EmptyState` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 105 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/ErrorBoundary/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.test.tsx`

- **所在目录**：`dashboard/src/components/ErrorBoundary/`
- **相对路径**：`dashboard/src/components/ErrorBoundary/index.test.tsx`
- **约行数**：60
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/ErrorBoundary` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 60 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/ErrorBoundary/`
- **相对路径**：`dashboard/src/components/ErrorBoundary/index.tsx`
- **约行数**：120
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/ErrorBoundary` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 120 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/Markdown/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `LazyMarkdown.tsx`

- **所在目录**：`dashboard/src/components/Markdown/`
- **相对路径**：`dashboard/src/components/Markdown/LazyMarkdown.tsx`
- **约行数**：21
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Markdown` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 21 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/Markdown/`
- **相对路径**：`dashboard/src/components/Markdown/index.tsx`
- **约行数**：448
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Markdown` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 448 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mathPlugins.ts`

- **所在目录**：`dashboard/src/components/Markdown/`
- **相对路径**：`dashboard/src/components/Markdown/mathPlugins.ts`
- **约行数**：84
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Markdown` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 84 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mermaidLoader.ts`

- **所在目录**：`dashboard/src/components/Markdown/`
- **相对路径**：`dashboard/src/components/Markdown/mermaidLoader.ts`
- **约行数**：21
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Markdown` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 21 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `stabilizeStreamingMarkdown.test.ts`

- **所在目录**：`dashboard/src/components/Markdown/`
- **相对路径**：`dashboard/src/components/Markdown/stabilizeStreamingMarkdown.test.ts`
- **约行数**：26
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Markdown` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 26 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `stabilizeStreamingMarkdown.ts`

- **所在目录**：`dashboard/src/components/Markdown/`
- **相对路径**：`dashboard/src/components/Markdown/stabilizeStreamingMarkdown.ts`
- **约行数**：22
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Markdown` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 22 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `syntaxHighlight.tsx`

- **所在目录**：`dashboard/src/components/Markdown/`
- **相对路径**：`dashboard/src/components/Markdown/syntaxHighlight.tsx`
- **约行数**：222
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Markdown` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 222 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/MarkdownCopy/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `MarkdownCopy.tsx`

- **所在目录**：`dashboard/src/components/MarkdownCopy/`
- **相对路径**：`dashboard/src/components/MarkdownCopy/MarkdownCopy.tsx`
- **约行数**：174
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/MarkdownCopy` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 174 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/OctopSpinner/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/OctopSpinner/`
- **相对路径**：`dashboard/src/components/OctopSpinner/index.tsx`
- **约行数**：20
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/OctopSpinner` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 20 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/PageLoading/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/PageLoading/`
- **相对路径**：`dashboard/src/components/PageLoading/index.tsx`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/PageLoading` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/PwaInstallPrompt/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.test.tsx`

- **所在目录**：`dashboard/src/components/PwaInstallPrompt/`
- **相对路径**：`dashboard/src/components/PwaInstallPrompt/index.test.tsx`
- **约行数**：24
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/PwaInstallPrompt` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 24 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/PwaInstallPrompt/`
- **相对路径**：`dashboard/src/components/PwaInstallPrompt/index.tsx`
- **约行数**：284
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/PwaInstallPrompt` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 284 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/PwaUpdatePrompt/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/PwaUpdatePrompt/`
- **相对路径**：`dashboard/src/components/PwaUpdatePrompt/index.tsx`
- **约行数**：66
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/PwaUpdatePrompt` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 66 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/ResizableTable/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/ResizableTable/`
- **相对路径**：`dashboard/src/components/ResizableTable/index.tsx`
- **约行数**：211
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/ResizableTable` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 211 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/Skeleton/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `CardSkeleton.tsx`

- **所在目录**：`dashboard/src/components/Skeleton/`
- **相对路径**：`dashboard/src/components/Skeleton/CardSkeleton.tsx`
- **约行数**：31
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Skeleton` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 31 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ListSkeleton.tsx`

- **所在目录**：`dashboard/src/components/Skeleton/`
- **相对路径**：`dashboard/src/components/Skeleton/ListSkeleton.tsx`
- **约行数**：33
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Skeleton` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 33 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TableSkeleton.tsx`

- **所在目录**：`dashboard/src/components/Skeleton/`
- **相对路径**：`dashboard/src/components/Skeleton/TableSkeleton.tsx`
- **约行数**：65
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Skeleton` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 65 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/components/Skeleton/`
- **相对路径**：`dashboard/src/components/Skeleton/index.ts`
- **约行数**：4
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/Skeleton` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 4 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/StreamConnectingIndicator/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/StreamConnectingIndicator/`
- **相对路径**：`dashboard/src/components/StreamConnectingIndicator/index.tsx`
- **约行数**：49
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/StreamConnectingIndicator` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 49 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/StreamEdgeControls/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `StreamEdgeControls.tsx`

- **所在目录**：`dashboard/src/components/StreamEdgeControls/`
- **相对路径**：`dashboard/src/components/StreamEdgeControls/StreamEdgeControls.tsx`
- **约行数**：78
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/StreamEdgeControls` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 78 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/StreamSetupGuide/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `StreamSetupGuide.tsx`

- **所在目录**：`dashboard/src/components/StreamSetupGuide/`
- **相对路径**：`dashboard/src/components/StreamSetupGuide/StreamSetupGuide.tsx`
- **约行数**：103
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/StreamSetupGuide` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 103 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/components/TabLabel/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `TabBar.tsx`

- **所在目录**：`dashboard/src/components/TabLabel/`
- **相对路径**：`dashboard/src/components/TabLabel/TabBar.tsx`
- **约行数**：45
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/TabLabel` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 45 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/components/TabLabel/`
- **相对路径**：`dashboard/src/components/TabLabel/index.tsx`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/components/TabLabel` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/context/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AgentContext.test.ts`

- **所在目录**：`dashboard/src/context/`
- **相对路径**：`dashboard/src/context/AgentContext.test.ts`
- **约行数**：131
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/context` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 131 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AgentContext.tsx`

- **所在目录**：`dashboard/src/context/`
- **相对路径**：`dashboard/src/context/AgentContext.tsx`
- **约行数**：310
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/context` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 310 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `BackupOperationContext.tsx`

- **所在目录**：`dashboard/src/context/`
- **相对路径**：`dashboard/src/context/BackupOperationContext.tsx`
- **约行数**：240
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/context` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 240 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `LayoutModeContext.tsx`

- **所在目录**：`dashboard/src/context/`
- **相对路径**：`dashboard/src/context/LayoutModeContext.tsx`
- **约行数**：66
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/context` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 66 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ScrollContext.tsx`

- **所在目录**：`dashboard/src/context/`
- **相对路径**：`dashboard/src/context/ScrollContext.tsx`
- **约行数**：164
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/context` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 164 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ServiceRestartContext.tsx`

- **所在目录**：`dashboard/src/context/`
- **相对路径**：`dashboard/src/context/ServiceRestartContext.tsx`
- **约行数**：46
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/context` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 46 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ThemeContext.tsx`

- **所在目录**：`dashboard/src/context/`
- **相对路径**：`dashboard/src/context/ThemeContext.tsx`
- **约行数**：194
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/context` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 194 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `VoiceOutputContext.tsx`

- **所在目录**：`dashboard/src/context/`
- **相对路径**：`dashboard/src/context/VoiceOutputContext.tsx`
- **约行数**：30
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/context` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 30 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/hooks/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `useAgentFormResources.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useAgentFormResources.ts`
- **约行数**：78
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 78 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useAgentThreadChat.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useAgentThreadChat.ts`
- **约行数**：90
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 90 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useAsyncResource.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useAsyncResource.ts`
- **约行数**：61
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 61 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useAudioUnlock.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useAudioUnlock.ts`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useAuthImageSrc.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useAuthImageSrc.ts`
- **约行数**：172
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 172 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useAutoViewportResize.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useAutoViewportResize.ts`
- **约行数**：69
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 69 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useBrowserCanvasInteraction.test.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useBrowserCanvasInteraction.test.ts`
- **约行数**：94
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 94 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useBrowserCanvasInteraction.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useBrowserCanvasInteraction.ts`
- **约行数**：18
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 18 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useBrowserSessionState.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useBrowserSessionState.ts`
- **约行数**：285
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 285 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useBrowserStream.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useBrowserStream.ts`
- **约行数**：285
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 285 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useBrowserViewController.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useBrowserViewController.ts`
- **约行数**：138
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 138 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useCanvasRemotePointer.test.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useCanvasRemotePointer.test.ts`
- **约行数**：286
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 286 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useCanvasRemotePointer.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useCanvasRemotePointer.ts`
- **约行数**：274
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 274 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useCardTableView.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useCardTableView.ts`
- **约行数**：17
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 17 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useCurrentUser.tsx`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useCurrentUser.tsx`
- **约行数**：52
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 52 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useDashboardPushToast.tsx`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useDashboardPushToast.tsx`
- **约行数**：200
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 200 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useDesktopCanvasInteraction.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useDesktopCanvasInteraction.ts`
- **约行数**：78
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 78 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useDesktopChrome.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useDesktopChrome.ts`
- **约行数**：65
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 65 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useDesktopInstall.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useDesktopInstall.ts`
- **约行数**：75
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 75 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useDesktopStream.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useDesktopStream.ts`
- **约行数**：266
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 266 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useElapsedSeconds.test.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useElapsedSeconds.test.ts`
- **约行数**：54
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 54 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useElapsedSeconds.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useElapsedSeconds.ts`
- **约行数**：20
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 20 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useEmbeddingDownloadWS.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useEmbeddingDownloadWS.ts`
- **约行数**：82
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 82 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useFilteredList.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useFilteredList.ts`
- **约行数**：17
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 17 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useGatedSearchTabs.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useGatedSearchTabs.ts`
- **约行数**：83
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 83 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useHorizontalResize.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useHorizontalResize.ts`
- **约行数**：162
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 162 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useIsMobile.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useIsMobile.ts`
- **约行数**：23
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 23 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useKeyboardOffset.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useKeyboardOffset.ts`
- **约行数**：40
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 40 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useLandscapeFullscreen.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useLandscapeFullscreen.ts`
- **约行数**：65
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 65 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useListPanelCollapsed.test.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useListPanelCollapsed.test.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useListPanelCollapsed.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useListPanelCollapsed.ts`
- **约行数**：40
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 40 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useMobileStream.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useMobileStream.ts`
- **约行数**：162
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 162 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `usePathTabs.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/usePathTabs.ts`
- **约行数**：134
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 134 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `usePointerDragSession.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/usePointerDragSession.ts`
- **约行数**：74
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 74 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useRemoteBrowserBookmarks.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useRemoteBrowserBookmarks.ts`
- **约行数**：116
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 116 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useServerCapabilities.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useServerCapabilities.ts`
- **约行数**：38
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 38 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useServerTimezone.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useServerTimezone.ts`
- **约行数**：37
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 37 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useServerUploadLimit.test.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useServerUploadLimit.test.ts`
- **约行数**：22
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 22 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useServerUploadLimit.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useServerUploadLimit.ts`
- **约行数**：65
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 65 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useServiceRestart.test.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useServiceRestart.test.ts`
- **约行数**：15
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 15 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useServiceRestart.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useServiceRestart.ts`
- **约行数**：200
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 200 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useSlashCommands.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useSlashCommands.ts`
- **约行数**：47
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 47 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useTerminalAutopilot.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useTerminalAutopilot.ts`
- **约行数**：254
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 254 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useUnauthorizedRedirect.test.tsx`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useUnauthorizedRedirect.test.tsx`
- **约行数**：51
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 51 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useUnauthorizedRedirect.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useUnauthorizedRedirect.ts`
- **约行数**：24
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 24 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useUpdateStatus.test.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useUpdateStatus.test.ts`
- **约行数**：144
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 144 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useUpdateStatus.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useUpdateStatus.ts`
- **约行数**：84
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 84 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useUserRole.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useUserRole.ts`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useViewportMode.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useViewportMode.ts`
- **约行数**：91
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 91 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useVoiceConfig.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useVoiceConfig.ts`
- **约行数**：55
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 55 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useVoiceInput.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useVoiceInput.ts`
- **约行数**：356
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 356 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useVoiceOutput.ts`

- **所在目录**：`dashboard/src/hooks/`
- **相对路径**：`dashboard/src/hooks/useVoiceOutput.ts`
- **约行数**：254
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 254 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/layouts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `Header.tsx`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/Header.tsx`
- **约行数**：110
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 110 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MinimalRecordsHost.tsx`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/MinimalRecordsHost.tsx`
- **约行数**：143
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 143 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PageShell.tsx`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/PageShell.tsx`
- **约行数**：240
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 240 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `Sidebar.tsx`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/Sidebar.tsx`
- **约行数**：757
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 757 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SidebarCollapsedIconNav.tsx`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/SidebarCollapsedIconNav.tsx`
- **约行数**：117
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 117 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SidebarMinimalPaneToggle.tsx`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/SidebarMinimalPaneToggle.tsx`
- **约行数**：60
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 60 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatHistoryRail.ts`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/chatHistoryRail.ts`
- **约行数**：11
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 11 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `layoutModeStorage.test.ts`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/layoutModeStorage.test.ts`
- **约行数**：43
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 43 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `layoutModeStorage.ts`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/layoutModeStorage.ts`
- **约行数**：48
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 48 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sidebarNav.test.ts`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/sidebarNav.test.ts`
- **约行数**：46
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 46 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sidebarNav.tsx`

- **所在目录**：`dashboard/src/layouts/`
- **相对路径**：`dashboard/src/layouts/sidebarNav.tsx`
- **约行数**：241
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 241 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/layouts/MainLayout/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/layouts/MainLayout/`
- **相对路径**：`dashboard/src/layouts/MainLayout/index.tsx`
- **约行数**：346
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/layouts/MainLayout` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 346 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Admin/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `Placeholder.tsx`

- **所在目录**：`dashboard/src/pages/Admin/`
- **相对路径**：`dashboard/src/pages/Admin/Placeholder.tsx`
- **约行数**：29
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 29 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Admin/Plugins/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `InstalledPluginsPanel.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Plugins/`
- **相对路径**：`dashboard/src/pages/Admin/Plugins/InstalledPluginsPanel.tsx`
- **约行数**：624
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Plugins` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 624 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PluginIconView.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Plugins/`
- **相对路径**：`dashboard/src/pages/Admin/Plugins/PluginIconView.tsx`
- **约行数**：81
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Plugins` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 81 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PluginMarketPanel.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Plugins/`
- **相对路径**：`dashboard/src/pages/Admin/Plugins/PluginMarketPanel.tsx`
- **约行数**：16
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Plugins` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 16 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Plugins/`
- **相对路径**：`dashboard/src/pages/Admin/Plugins/index.tsx`
- **约行数**：67
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Plugins` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 67 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Admin/SharedModels/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Admin/SharedModels/`
- **相对路径**：`dashboard/src/pages/Admin/SharedModels/index.tsx`
- **约行数**：112
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/SharedModels` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 112 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Admin/Storage/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `DockerEnvFooter.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Storage/`
- **相对路径**：`dashboard/src/pages/Admin/Storage/DockerEnvFooter.tsx`
- **约行数**：242
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Storage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 242 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `StorageBackendCard.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Storage/`
- **相对路径**：`dashboard/src/pages/Admin/Storage/StorageBackendCard.tsx`
- **约行数**：349
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Storage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 349 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `StorageBackendModal.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Storage/`
- **相对路径**：`dashboard/src/pages/Admin/Storage/StorageBackendModal.tsx`
- **约行数**：647
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Storage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 647 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `StorageBrowseDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Storage/`
- **相对路径**：`dashboard/src/pages/Admin/Storage/StorageBrowseDrawer.tsx`
- **约行数**：199
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Storage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 199 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `StorageTypeCard.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Storage/`
- **相对路径**：`dashboard/src/pages/Admin/Storage/StorageTypeCard.tsx`
- **约行数**：58
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Storage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 58 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Storage/`
- **相对路径**：`dashboard/src/pages/Admin/Storage/index.tsx`
- **约行数**：200
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Storage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 200 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useStorageBackends.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Storage/`
- **相对路径**：`dashboard/src/pages/Admin/Storage/useStorageBackends.tsx`
- **约行数**：476
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Storage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 476 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Admin/Users/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `InviteDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Users/`
- **相对路径**：`dashboard/src/pages/Admin/Users/InviteDrawer.tsx`
- **约行数**：284
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Users` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 284 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `OauthProviderCard.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Users/`
- **相对路径**：`dashboard/src/pages/Admin/Users/OauthProviderCard.tsx`
- **约行数**：506
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Users` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 506 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SsoAppProviderShell.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Users/`
- **相对路径**：`dashboard/src/pages/Admin/Users/SsoAppProviderShell.tsx`
- **约行数**：223
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Users` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 223 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SsoPanel.test.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Users/`
- **相对路径**：`dashboard/src/pages/Admin/Users/SsoPanel.test.tsx`
- **约行数**：88
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Users` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 88 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SsoPanel.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Users/`
- **相对路径**：`dashboard/src/pages/Admin/Users/SsoPanel.tsx`
- **约行数**：696
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Users` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 696 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SsoProviderCard.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Users/`
- **相对路径**：`dashboard/src/pages/Admin/Users/SsoProviderCard.tsx`
- **约行数**：53
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Users` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 53 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `UsersListPanel.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Users/`
- **相对路径**：`dashboard/src/pages/Admin/Users/UsersListPanel.tsx`
- **约行数**：1978
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Users` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1978 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Admin/Users/`
- **相对路径**：`dashboard/src/pages/Admin/Users/index.tsx`
- **约行数**：140
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Users` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 140 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `oauthProviders.ts`

- **所在目录**：`dashboard/src/pages/Admin/Users/`
- **相对路径**：`dashboard/src/pages/Admin/Users/oauthProviders.ts`
- **约行数**：64
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Admin/Users` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 64 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/ACP/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `constants.ts`

- **所在目录**：`dashboard/src/pages/Agent/ACP/`
- **相对路径**：`dashboard/src/pages/Agent/ACP/constants.ts`
- **约行数**：68
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/ACP` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 68 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Agent/ACP/`
- **相对路径**：`dashboard/src/pages/Agent/ACP/index.tsx`
- **约行数**：342
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/ACP` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 342 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/ACP/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ACPCard.tsx`

- **所在目录**：`dashboard/src/pages/Agent/ACP/components/`
- **相对路径**：`dashboard/src/pages/Agent/ACP/components/ACPCard.tsx`
- **约行数**：115
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/ACP/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 115 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ACPDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Agent/ACP/components/`
- **相对路径**：`dashboard/src/pages/Agent/ACP/components/ACPDrawer.tsx`
- **约行数**：172
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/ACP/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 172 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Channels/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ChannelsPanel.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Channels/`
- **相对路径**：`dashboard/src/pages/Agent/Channels/ChannelsPanel.test.tsx`
- **约行数**：117
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Channels` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 117 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChannelsPanel.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Channels/`
- **相对路径**：`dashboard/src/pages/Agent/Channels/ChannelsPanel.tsx`
- **约行数**：493
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Channels` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 493 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Channels/`
- **相对路径**：`dashboard/src/pages/Agent/Channels/index.tsx`
- **约行数**：7
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Channels` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 7 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChannels.ts`

- **所在目录**：`dashboard/src/pages/Agent/Channels/`
- **相对路径**：`dashboard/src/pages/Agent/Channels/useChannels.ts`
- **约行数**：218
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Channels` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 218 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Channels/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ChannelCard.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Channels/components/`
- **相对路径**：`dashboard/src/pages/Agent/Channels/components/ChannelCard.tsx`
- **约行数**：161
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Channels/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 161 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChannelDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Channels/components/`
- **相对路径**：`dashboard/src/pages/Agent/Channels/components/ChannelDrawer.tsx`
- **约行数**：1891
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Channels/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1891 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `constants.test.ts`

- **所在目录**：`dashboard/src/pages/Agent/Channels/components/`
- **相对路径**：`dashboard/src/pages/Agent/Channels/components/constants.test.ts`
- **约行数**：98
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Channels/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 98 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `constants.ts`

- **所在目录**：`dashboard/src/pages/Agent/Channels/components/`
- **相对路径**：`dashboard/src/pages/Agent/Channels/components/constants.ts`
- **约行数**：451
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Channels/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 451 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/pages/Agent/Channels/components/`
- **相对路径**：`dashboard/src/pages/Agent/Channels/components/index.ts`
- **约行数**：33
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Channels/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 33 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Config/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Config/`
- **相对路径**：`dashboard/src/pages/Agent/Config/index.tsx`
- **约行数**：106
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Config` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 106 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Connectors/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ConnectorCard.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/ConnectorCard.test.tsx`
- **约行数**：45
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 45 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ConnectorCard.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/ConnectorCard.tsx`
- **约行数**：71
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 71 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ConnectorInstanceCard.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/ConnectorInstanceCard.test.tsx`
- **约行数**：88
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 88 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ConnectorInstanceCard.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/ConnectorInstanceCard.tsx`
- **约行数**：159
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 159 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `CustomMcpServerCard.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/CustomMcpServerCard.tsx`
- **约行数**：475
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 475 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `CustomMcpTab.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/CustomMcpTab.tsx`
- **约行数**：792
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 792 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `connectorDefs.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/connectorDefs.tsx`
- **约行数**：91
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 91 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `customMcpUtils.ts`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/customMcpUtils.ts`
- **约行数**：263
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 263 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `guidedConnectorUtils.test.ts`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/guidedConnectorUtils.test.ts`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `guidedConnectorUtils.ts`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/guidedConnectorUtils.ts`
- **约行数**：46
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 46 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/index.tsx`
- **约行数**：2341
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 2341 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useConnectors.ts`

- **所在目录**：`dashboard/src/pages/Agent/Connectors/`
- **相对路径**：`dashboard/src/pages/Agent/Connectors/useConnectors.ts`
- **约行数**：37
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Connectors` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 37 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Memory/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AtomsList.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/AtomsList.test.tsx`
- **约行数**：182
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 182 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AtomsList.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/AtomsList.tsx`
- **约行数**：421
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 421 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `CandidatesReview.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/CandidatesReview.test.tsx`
- **约行数**：168
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 168 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `CandidatesReview.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/CandidatesReview.tsx`
- **约行数**：522
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 522 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ConversationRecords.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/ConversationRecords.tsx`
- **约行数**：655
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 655 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `EpisodesList.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/EpisodesList.test.tsx`
- **约行数**：93
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 93 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `EpisodesList.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/EpisodesList.tsx`
- **约行数**：315
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 315 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExtractTriggerConfig.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/ExtractTriggerConfig.test.tsx`
- **约行数**：138
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 138 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExtractTriggerConfig.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/ExtractTriggerConfig.tsx`
- **约行数**：3
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 3 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `JournalList.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/JournalList.test.tsx`
- **约行数**：125
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 125 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `JournalList.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/JournalList.tsx`
- **约行数**：681
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 681 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MemoryPanel.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/MemoryPanel.tsx`
- **约行数**：300
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 300 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MemorySettings.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/MemorySettings.tsx`
- **约行数**：356
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 356 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MemoryTree.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/MemoryTree.test.tsx`
- **约行数**：95
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 95 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MemoryTree.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/MemoryTree.tsx`
- **约行数**：1040
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1040 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MigrateMemory.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/MigrateMemory.tsx`
- **约行数**：590
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 590 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `Overview.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/Overview.test.tsx`
- **约行数**：135
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 135 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `Overview.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/Overview.tsx`
- **约行数**：528
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 528 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ProactiveConfig.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/ProactiveConfig.tsx`
- **约行数**：356
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 356 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ProfileOverview.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/ProfileOverview.test.tsx`
- **约行数**：164
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 164 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ProfileOverview.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/ProfileOverview.tsx`
- **约行数**：544
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 544 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `RawEventsList.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/RawEventsList.test.tsx`
- **约行数**：151
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 151 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `RawEventsList.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/RawEventsList.tsx`
- **约行数**：205
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 205 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `VectorSearchConfig.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/VectorSearchConfig.tsx`
- **约行数**：629
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 629 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Memory/shared/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `LineageStrip.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/shared/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/shared/LineageStrip.tsx`
- **约行数**：178
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory/shared` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 178 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MemoryLayerView.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/shared/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/shared/MemoryLayerView.tsx`
- **约行数**：151
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory/shared` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 151 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MemoryPipelineEmpty.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/shared/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/shared/MemoryPipelineEmpty.test.tsx`
- **约行数**：82
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory/shared` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 82 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MemoryPipelineEmpty.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/shared/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/shared/MemoryPipelineEmpty.tsx`
- **约行数**：107
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory/shared` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 107 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `createAtom.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/shared/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/shared/createAtom.tsx`
- **约行数**：218
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory/shared` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 218 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `deprecateAtom.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/shared/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/shared/deprecateAtom.tsx`
- **约行数**：65
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory/shared` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 65 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `editAtom.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Memory/shared/`
- **相对路径**：`dashboard/src/pages/Agent/Memory/shared/editAtom.tsx`
- **约行数**：65
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Memory/shared` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 65 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Personalization/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Personalization/`
- **相对路径**：`dashboard/src/pages/Agent/Personalization/index.tsx`
- **约行数**：222
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Personalization` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 222 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Personalization/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AgentPluginsPanel.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Personalization/components/`
- **相对路径**：`dashboard/src/pages/Agent/Personalization/components/AgentPluginsPanel.tsx`
- **约行数**：370
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Personalization/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 370 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `EditDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Personalization/components/`
- **相对路径**：`dashboard/src/pages/Agent/Personalization/components/EditDrawer.tsx`
- **约行数**：150
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Personalization/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 150 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MBTISelector.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Personalization/components/`
- **相对路径**：`dashboard/src/pages/Agent/Personalization/components/MBTISelector.tsx`
- **约行数**：630
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Personalization/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 630 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MBTITest.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Personalization/components/`
- **相对路径**：`dashboard/src/pages/Agent/Personalization/components/MBTITest.tsx`
- **约行数**：396
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Personalization/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 396 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Personalization/hooks/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `useFileEditor.ts`

- **所在目录**：`dashboard/src/pages/Agent/Personalization/hooks/`
- **相对路径**：`dashboard/src/pages/Agent/Personalization/hooks/useFileEditor.ts`
- **约行数**：129
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Personalization/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 129 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Skills/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `skillDisplayNames.test.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/skillDisplayNames.test.ts`
- **约行数**：46
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 46 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillDisplayNames.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/skillDisplayNames.ts`
- **约行数**：92
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 92 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillMarkdown.test.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/skillMarkdown.test.ts`
- **约行数**：50
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 50 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillMarkdown.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/skillMarkdown.ts`
- **约行数**：54
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 54 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useSkills.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/useSkills.ts`
- **约行数**：362
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 362 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Skills/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AgentPickerModal.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/AgentPickerModal.tsx`
- **约行数**：71
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 71 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `InstalledSkillsTab.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/InstalledSkillsTab.tsx`
- **约行数**：301
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 301 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PushSkillToPackageModal.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/PushSkillToPackageModal.tsx`
- **约行数**：117
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 117 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SectionHeader.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SectionHeader.tsx`
- **约行数**：36
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 36 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillCard.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillCard.tsx`
- **约行数**：311
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 311 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillDrawer.test.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillDrawer.test.ts`
- **约行数**：70
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 70 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillDrawer.tsx`
- **约行数**：936
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 936 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillFileTree.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillFileTree.tsx`
- **约行数**：215
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 215 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillHubDetailDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillHubDetailDrawer.tsx`
- **约行数**：310
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 310 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillHubTab.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillHubTab.tsx`
- **约行数**：496
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 496 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillImportModal.test.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillImportModal.test.tsx`
- **约行数**：198
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 198 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillImportModal.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillImportModal.tsx`
- **约行数**：377
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 377 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillPackagesTab.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillPackagesTab.tsx`
- **约行数**：487
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 487 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillsTable.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillsTable.tsx`
- **约行数**：130
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 130 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillsTabs.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/SkillsTabs.tsx`
- **约行数**：98
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 98 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/index.ts`
- **约行数**：4
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 4 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseSkillZip.test.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/parseSkillZip.test.ts`
- **约行数**：294
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 294 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseSkillZip.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/parseSkillZip.ts`
- **约行数**：223
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 223 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillHubCache.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/skillHubCache.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillInstallTarget.ts`

- **所在目录**：`dashboard/src/pages/Agent/Skills/components/`
- **相对路径**：`dashboard/src/pages/Agent/Skills/components/skillInstallTarget.ts`
- **约行数**：39
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Skills/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 39 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Tools/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ToolsPanel.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Tools/`
- **相对路径**：`dashboard/src/pages/Agent/Tools/ToolsPanel.tsx`
- **约行数**：363
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Tools` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 363 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ToolsTabs.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Tools/`
- **相对路径**：`dashboard/src/pages/Agent/Tools/ToolsTabs.tsx`
- **约行数**：61
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Tools` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 61 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Workspace/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `CodeEditor.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/components/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/components/CodeEditor.tsx`
- **约行数**：124
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 124 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `DocumentPreview.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/components/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/components/DocumentPreview.tsx`
- **约行数**：80
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 80 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `FilePreview.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/components/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/components/FilePreview.tsx`
- **约行数**：133
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 133 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `FileViewer.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/components/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/components/FileViewer.tsx`
- **约行数**：139
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 139 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MediaPreview.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/components/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/components/MediaPreview.tsx`
- **约行数**：316
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 316 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `WorkspaceDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/components/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/components/WorkspaceDrawer.tsx`
- **约行数**：1557
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1557 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Agent/Workspace/utils/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `docKind.test.ts`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/utils/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/utils/docKind.test.ts`
- **约行数**：42
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 42 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `docKind.ts`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/utils/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/utils/docKind.ts`
- **约行数**：8
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 8 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `editorLanguage.ts`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/utils/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/utils/editorLanguage.ts`
- **约行数**：61
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 61 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `fileKind.ts`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/utils/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/utils/fileKind.ts`
- **约行数**：92
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 92 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mediaKind.ts`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/utils/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/utils/mediaKind.ts`
- **约行数**：45
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 45 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `workspaceMove.test.ts`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/utils/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/utils/workspaceMove.test.ts`
- **约行数**：67
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 67 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `workspaceMove.ts`

- **所在目录**：`dashboard/src/pages/Agent/Workspace/utils/`
- **相对路径**：`dashboard/src/pages/Agent/Workspace/utils/workspaceMove.ts`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Agent/Workspace/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Chat/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ChatAgentProfileContext.tsx`

- **所在目录**：`dashboard/src/pages/Chat/`
- **相对路径**：`dashboard/src/pages/Chat/ChatAgentProfileContext.tsx`
- **约行数**：37
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 37 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatFilePreviewContext.tsx`

- **所在目录**：`dashboard/src/pages/Chat/`
- **相对路径**：`dashboard/src/pages/Chat/ChatFilePreviewContext.tsx`
- **约行数**：35
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 35 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatToolDockContext.tsx`

- **所在目录**：`dashboard/src/pages/Chat/`
- **相对路径**：`dashboard/src/pages/Chat/ChatToolDockContext.tsx`
- **约行数**：71
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 71 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `constants.ts`

- **所在目录**：`dashboard/src/pages/Chat/`
- **相对路径**：`dashboard/src/pages/Chat/constants.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Chat/`
- **相对路径**：`dashboard/src/pages/Chat/index.tsx`
- **约行数**：1479
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1479 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Chat/Weather/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Chat/Weather/`
- **相对路径**：`dashboard/src/pages/Chat/Weather/index.tsx`
- **约行数**：241
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/Weather` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 241 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Chat/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AgentNotReadyScreen.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/AgentNotReadyScreen.tsx`
- **约行数**：115
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 115 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AskQuestionCard.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/AskQuestionCard.test.tsx`
- **约行数**：115
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 115 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AskQuestionCard.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/AskQuestionCard.tsx`
- **约行数**：338
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 338 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AssistantProcessSummary.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/AssistantProcessSummary.tsx`
- **约行数**：121
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 121 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AssistantTurnView.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/AssistantTurnView.tsx`
- **约行数**：311
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 311 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatComposerChrome.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatComposerChrome.tsx`
- **约行数**：23
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 23 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatDockFileList.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatDockFileList.tsx`
- **约行数**：314
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 314 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatDockPanel.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatDockPanel.tsx`
- **约行数**：469
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 469 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatDockPanels.keepAlive.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatDockPanels.keepAlive.test.tsx`
- **约行数**：62
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 62 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatDockPanels.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatDockPanels.tsx`
- **约行数**：135
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 135 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatDockToolUiContent.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatDockToolUiContent.tsx`
- **约行数**：46
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 46 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatInput.prefill.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatInput.prefill.test.tsx`
- **约行数**：240
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 240 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatInput.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatInput.tsx`
- **约行数**：891
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 891 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatInputActionsRow.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatInputActionsRow.test.tsx`
- **约行数**：63
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 63 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatInputActionsRow.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatInputActionsRow.tsx`
- **约行数**：1303
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1303 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatInputPreviewBar.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatInputPreviewBar.tsx`
- **约行数**：270
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 270 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatMediaPlayer.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatMediaPlayer.tsx`
- **约行数**：167
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 167 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatQueuedMessages.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatQueuedMessages.tsx`
- **约行数**：83
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 83 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatSidebarPanel.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatSidebarPanel.tsx`
- **约行数**：187
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 187 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatTitleBar.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatTitleBar.test.tsx`
- **约行数**：52
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 52 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChatTitleBar.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ChatTitleBar.tsx`
- **约行数**：183
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 183 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ConnectorPickerPopover.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ConnectorPickerPopover.tsx`
- **约行数**：95
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 95 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ContextChip.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ContextChip.tsx`
- **约行数**：57
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 57 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ContextWindowRing.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ContextWindowRing.tsx`
- **约行数**：361
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 361 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExpertAgentAvatar.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ExpertAgentAvatar.tsx`
- **约行数**：58
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 58 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExpertPickerPopover.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ExpertPickerPopover.tsx`
- **约行数**：81
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 81 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `FilePanelContent.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/FilePanelContent.tsx`
- **约行数**：376
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 376 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `GeneratingIndicator.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/GeneratingIndicator.tsx`
- **约行数**：56
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 56 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `HistoryMigrationBanner.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/HistoryMigrationBanner.test.tsx`
- **约行数**：53
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 53 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `HistoryMigrationBanner.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/HistoryMigrationBanner.tsx`
- **约行数**：73
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 73 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `HitlApprovalCard.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/HitlApprovalCard.test.tsx`
- **约行数**：59
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 59 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `HitlApprovalCard.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/HitlApprovalCard.tsx`
- **约行数**：105
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 105 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `KnowledgeCitationPanelContent.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/KnowledgeCitationPanelContent.tsx`
- **约行数**：375
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 375 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `KnowledgeCitationsStrip.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/KnowledgeCitationsStrip.tsx`
- **约行数**：101
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 101 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `KnowledgePickerPopover.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/KnowledgePickerPopover.tsx`
- **约行数**：98
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 98 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MemoryMaintenanceBanner.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/MemoryMaintenanceBanner.tsx`
- **约行数**：65
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 65 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MentionPickerMenu.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/MentionPickerMenu.test.ts`
- **约行数**：61
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 61 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MentionPickerMenu.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/MentionPickerMenu.tsx`
- **约行数**：238
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 238 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MessageBubble.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/MessageBubble.tsx`
- **约行数**：1056
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1056 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MessageFileCard.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/MessageFileCard.tsx`
- **约行数**：118
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 118 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MessageList.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/MessageList.tsx`
- **约行数**：856
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 856 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MessageSender.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/MessageSender.test.tsx`
- **约行数**：36
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 36 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MessageSender.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/MessageSender.tsx`
- **约行数**：78
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 78 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MinimalAgentSessionNav.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/MinimalAgentSessionNav.tsx`
- **约行数**：629
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 629 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ScrollToBottomButton.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ScrollToBottomButton.tsx`
- **约行数**：91
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 91 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SessionChannelIcon.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/SessionChannelIcon.tsx`
- **约行数**：72
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 72 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SessionList.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/SessionList.tsx`
- **约行数**：523
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 523 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SharedExpertHint.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/SharedExpertHint.tsx`
- **约行数**：26
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 26 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillPickerPopover.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/SkillPickerPopover.tsx`
- **约行数**：130
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 130 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SlashCommandMenu.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/SlashCommandMenu.tsx`
- **约行数**：125
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 125 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SubagentPickerPopover.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/SubagentPickerPopover.tsx`
- **约行数**：78
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 78 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ThinkingBubble.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ThinkingBubble.tsx`
- **约行数**：47
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 47 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ToolMediaStrip.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/ToolMediaStrip.tsx`
- **约行数**：156
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 156 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryDrawer.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryDrawer.test.tsx`
- **约行数**：349
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 349 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryDrawer.tsx`
- **约行数**：330
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 330 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryInspector.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryInspector.test.tsx`
- **约行数**：307
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 307 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryInspector.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryInspector.tsx`
- **约行数**：443
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 443 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryLedger.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryLedger.test.tsx`
- **约行数**：296
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 296 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryLedger.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryLedger.tsx`
- **约行数**：358
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 358 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryMetricsBar.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryMetricsBar.test.tsx`
- **约行数**：50
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 50 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryMetricsBar.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryMetricsBar.tsx`
- **约行数**：127
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 127 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryTimeline.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryTimeline.test.tsx`
- **约行数**：255
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 255 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryTimeline.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryTimeline.tsx`
- **约行数**：698
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 698 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryToolbar.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryToolbar.test.tsx`
- **约行数**：77
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 77 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TrajectoryToolbar.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TrajectoryToolbar.tsx`
- **约行数**：124
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 124 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TurnProcessBlocks.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TurnProcessBlocks.tsx`
- **约行数**：107
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 107 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TurnTimelineRail.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TurnTimelineRail.test.tsx`
- **约行数**：261
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 261 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TurnTimelineRail.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/TurnTimelineRail.tsx`
- **约行数**：427
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 427 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `UserMessageComposerTags.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/UserMessageComposerTags.tsx`
- **约行数**：149
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 149 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `WelcomeQuickCards.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/WelcomeQuickCards.tsx`
- **约行数**：106
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 106 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `WelcomeScreen.tsx`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/WelcomeScreen.tsx`
- **约行数**：119
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 119 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `generatingGate.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/generatingGate.test.ts`
- **约行数**：52
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 52 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `generatingGate.ts`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/generatingGate.ts`
- **约行数**：27
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 27 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `liveAssistantTurn.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/liveAssistantTurn.test.ts`
- **约行数**：50
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 50 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `liveAssistantTurn.ts`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/liveAssistantTurn.ts`
- **约行数**：27
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 27 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `loadOlderGate.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/loadOlderGate.test.ts`
- **约行数**：122
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 122 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `loadOlderGate.ts`

- **所在目录**：`dashboard/src/pages/Chat/components/`
- **相对路径**：`dashboard/src/pages/Chat/components/loadOlderGate.ts`
- **约行数**：38
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 38 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Chat/hooks/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `chatSidebarDefaultOpen.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/chatSidebarDefaultOpen.test.ts`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatStore.clearMessages.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/chatStore.clearMessages.test.ts`
- **约行数**：43
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 43 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatStore.history.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/chatStore.history.test.ts`
- **约行数**：96
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 96 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatStore.hitlResume.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/chatStore.hitlResume.test.ts`
- **约行数**：47
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 47 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatStore.pendingAlias.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/chatStore.pendingAlias.test.ts`
- **约行数**：70
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 70 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatStore.thinkingTimer.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/chatStore.thinkingTimer.test.ts`
- **约行数**：27
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 27 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatStore.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/chatStore.ts`
- **约行数**：2254
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 2254 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `frameThread.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/frameThread.test.ts`
- **约行数**：21
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 21 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `frameThread.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/frameThread.ts`
- **约行数**：16
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 16 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `scrollFreeMode.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/scrollFreeMode.test.ts`
- **约行数**：67
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 67 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `scrollFreeMode.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/scrollFreeMode.ts`
- **约行数**：32
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 32 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sealPriorStreamingAssistants.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/sealPriorStreamingAssistants.test.ts`
- **约行数**：60
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 60 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sealPriorStreamingAssistants.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/sealPriorStreamingAssistants.ts`
- **约行数**：23
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 23 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sseHelpers.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/sseHelpers.ts`
- **约行数**：123
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 123 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `toolDisplayNames.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/toolDisplayNames.ts`
- **约行数**：28
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 28 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `turnStatusGate.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/turnStatusGate.test.ts`
- **约行数**：13
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 13 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `turnStatusGate.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/turnStatusGate.ts`
- **约行数**：10
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 10 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useAutoScroll.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useAutoScroll.test.ts`
- **约行数**：707
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 707 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useAutoScroll.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useAutoScroll.ts`
- **约行数**：568
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 568 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useBrowserToolDetection.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useBrowserToolDetection.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChat.history.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChat.history.test.ts`
- **约行数**：128
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 128 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChat.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChat.ts`
- **约行数**：1161
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1161 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChat.usage.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChat.usage.test.ts`
- **约行数**：98
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 98 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatAttachments.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatAttachments.ts`
- **约行数**：173
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 173 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatComposerResources.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatComposerResources.ts`
- **约行数**：413
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 413 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatContextWindow.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatContextWindow.test.ts`
- **约行数**：106
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 106 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatContextWindow.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatContextWindow.ts`
- **约行数**：114
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 114 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatDockPanel.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatDockPanel.test.ts`
- **约行数**：265
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 265 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatDockPanel.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatDockPanel.ts`
- **约行数**：358
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 358 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatFileDetection.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatFileDetection.test.ts`
- **约行数**：48
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 48 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatFileDetection.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatFileDetection.ts`
- **约行数**：187
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 187 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatHistoryRail.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatHistoryRail.ts`
- **约行数**：32
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 32 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatMessageQueue.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatMessageQueue.test.ts`
- **约行数**：364
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 364 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatMessageQueue.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatMessageQueue.ts`
- **约行数**：318
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 318 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatNavigation.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatNavigation.test.tsx`
- **约行数**：305
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 305 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatNavigation.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatNavigation.ts`
- **约行数**：230
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 230 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatSend.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatSend.ts`
- **约行数**：309
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 309 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatSessionActions.test.tsx`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatSessionActions.test.tsx`
- **约行数**：81
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 81 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatSessionActions.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatSessionActions.ts`
- **约行数**：169
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 169 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatSidebarState.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatSidebarState.ts`
- **约行数**：180
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 180 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useChatSubagents.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useChatSubagents.ts`
- **约行数**：33
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 33 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useDragOffset.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useDragOffset.ts`
- **约行数**：251
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 251 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useExpertQuickCards.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useExpertQuickCards.ts`
- **约行数**：71
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 71 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useHistoryMigration.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useHistoryMigration.ts`
- **约行数**：65
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 65 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useMemoryMaintenance.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useMemoryMaintenance.ts`
- **约行数**：67
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 67 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `usePanelResize.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/usePanelResize.ts`
- **约行数**：122
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 122 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useSessions.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useSessions.test.ts`
- **约行数**：170
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 170 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useSessions.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useSessions.ts`
- **约行数**：561
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 561 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useSkillRecordingWorkflow.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useSkillRecordingWorkflow.ts`
- **约行数**：242
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 242 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useSlashMentionInput.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useSlashMentionInput.test.ts`
- **约行数**：145
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 145 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useSlashMentionInput.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useSlashMentionInput.ts`
- **约行数**：532
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 532 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useToolMessageByCallId.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useToolMessageByCallId.ts`
- **约行数**：25
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 25 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useToolUiDockButtonStyle.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useToolUiDockButtonStyle.ts`
- **约行数**：74
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 74 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useTrajectorySession.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useTrajectorySession.test.ts`
- **约行数**：446
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 446 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useTrajectorySession.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useTrajectorySession.ts`
- **约行数**：213
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 213 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useWelcomeQuickCardsLayout.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useWelcomeQuickCardsLayout.ts`
- **约行数**：128
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 128 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useWorkspaceFileMention.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useWorkspaceFileMention.test.ts`
- **约行数**：60
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 60 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useWorkspaceFileMention.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/useWorkspaceFileMention.ts`
- **约行数**：76
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 76 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `wsResumeGate.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/wsResumeGate.test.ts`
- **约行数**：204
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 204 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `wsResumeGate.ts`

- **所在目录**：`dashboard/src/pages/Chat/hooks/`
- **相对路径**：`dashboard/src/pages/Chat/hooks/wsResumeGate.ts`
- **约行数**：100
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/hooks` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 100 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Chat/utils/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `accountDisplayName.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/accountDisplayName.test.ts`
- **约行数**：36
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 36 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `accountDisplayName.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/accountDisplayName.ts`
- **约行数**：15
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 15 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatAttachments.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/chatAttachments.test.ts`
- **约行数**：58
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 58 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatAttachments.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/chatAttachments.ts`
- **约行数**：116
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 116 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatMessages.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/chatMessages.test.ts`
- **约行数**：71
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 71 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatMessages.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/chatMessages.ts`
- **约行数**：200
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 200 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatStorage.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/chatStorage.ts`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chromeInstallGate.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/chromeInstallGate.test.ts`
- **约行数**：21
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 21 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chromeInstallGate.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/chromeInstallGate.ts`
- **约行数**：10
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 10 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `dockFilePath.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/dockFilePath.test.ts`
- **约行数**：274
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 274 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `dockFilePath.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/dockFilePath.ts`
- **约行数**：282
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 282 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `dockKnowledgeTabId.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/dockKnowledgeTabId.ts`
- **约行数**：5
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 5 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `dockToolUiTabId.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/dockToolUiTabId.ts`
- **约行数**：5
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 5 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `expertMention.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/expertMention.test.ts`
- **约行数**：93
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 93 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `expertMention.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/expertMention.ts`
- **约行数**：113
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 113 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `fileMention.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/fileMention.test.ts`
- **约行数**：129
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 129 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `fileMention.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/fileMention.ts`
- **约行数**：163
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 163 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `layoutAssistantTurnHitl.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/layoutAssistantTurnHitl.test.ts`
- **约行数**：116
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 116 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `layoutAssistantTurnHitl.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/layoutAssistantTurnHitl.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mentionAtCursor.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/mentionAtCursor.ts`
- **约行数**：10
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 10 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `messageContent.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/messageContent.ts`
- **约行数**：171
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 171 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `messageGrouping.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/messageGrouping.test.ts`
- **约行数**：52
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 52 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `messageGrouping.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/messageGrouping.ts`
- **约行数**：44
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 44 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `pendingAttachKnowledgeBase.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/pendingAttachKnowledgeBase.test.ts`
- **约行数**：17
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 17 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `pendingAttachKnowledgeBase.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/pendingAttachKnowledgeBase.ts`
- **约行数**：20
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 20 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `pendingHitl.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/pendingHitl.test.ts`
- **约行数**：100
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 100 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `pendingHitl.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/pendingHitl.ts`
- **约行数**：42
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 42 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `resolveInitialConnectors.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/resolveInitialConnectors.test.ts`
- **约行数**：94
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 94 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `resolveInitialConnectors.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/resolveInitialConnectors.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillChipIcon.tsx`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/skillChipIcon.tsx`
- **约行数**：19
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 19 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillSlash.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/skillSlash.test.ts`
- **约行数**：155
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 155 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `skillSlash.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/skillSlash.ts`
- **约行数**：198
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 198 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `slashShortcutStyles.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/slashShortcutStyles.ts`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `slashText.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/slashText.ts`
- **约行数**：17
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 17 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `summarizeHitlAction.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/summarizeHitlAction.test.ts`
- **约行数**：138
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 138 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `summarizeHitlAction.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/summarizeHitlAction.ts`
- **约行数**：271
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 271 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `threadTitle.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/threadTitle.test.ts`
- **约行数**：59
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 59 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `threadTitle.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/threadTitle.ts`
- **约行数**：35
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 35 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `trajectoryModel.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/trajectoryModel.test.ts`
- **约行数**：371
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 371 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `trajectoryModel.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/trajectoryModel.ts`
- **约行数**：577
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 577 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `trajectoryTimeline.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/trajectoryTimeline.test.ts`
- **约行数**：317
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 317 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `trajectoryTimeline.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/trajectoryTimeline.ts`
- **约行数**：232
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 232 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `turnTimeline.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/turnTimeline.test.ts`
- **约行数**：188
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 188 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `turnTimeline.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/turnTimeline.ts`
- **约行数**：331
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 331 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `withDefaultOpenConnectors.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/withDefaultOpenConnectors.test.ts`
- **约行数**：22
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 22 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `withDefaultOpenConnectors.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/withDefaultOpenConnectors.ts`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `withDefaultOpenKnowledgeBases.test.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/withDefaultOpenKnowledgeBases.test.ts`
- **约行数**：21
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 21 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `withDefaultOpenKnowledgeBases.ts`

- **所在目录**：`dashboard/src/pages/Chat/utils/`
- **相对路径**：`dashboard/src/pages/Chat/utils/withDefaultOpenKnowledgeBases.ts`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Chat/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Control/CronJobs/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `constants.ts`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/constants.ts`
- **约行数**：5
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 5 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `cronDisplay.ts`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/cronDisplay.ts`
- **约行数**：42
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 42 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/index.tsx`
- **约行数**：405
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 405 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `taskExamples.test.ts`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/taskExamples.test.ts`
- **约行数**：79
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 79 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `taskExamples.ts`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/taskExamples.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useCronJobs.ts`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/useCronJobs.ts`
- **约行数**：476
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 476 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useTaskExamples.ts`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/useTaskExamples.ts`
- **约行数**：61
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 61 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Control/CronJobs/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `CronJobCard.tsx`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/components/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/components/CronJobCard.tsx`
- **约行数**：240
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 240 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExecuteNowModal.tsx`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/components/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/components/ExecuteNowModal.tsx`
- **约行数**：63
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 63 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `JobDetailDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/components/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/components/JobDetailDrawer.tsx`
- **约行数**：250
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 250 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `JobDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/components/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/components/JobDrawer.tsx`
- **约行数**：493
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 493 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `columns.tsx`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/components/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/components/columns.tsx`
- **约行数**：328
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 328 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `constants.ts`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/components/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/components/constants.ts`
- **约行数**：139
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 139 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/pages/Control/CronJobs/components/`
- **相对路径**：`dashboard/src/pages/Control/CronJobs/components/index.ts`
- **约行数**：8
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/CronJobs/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 8 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Control/RemoteAndroid/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AdbShellPanel.tsx`

- **所在目录**：`dashboard/src/pages/Control/RemoteAndroid/`
- **相对路径**：`dashboard/src/pages/Control/RemoteAndroid/AdbShellPanel.tsx`
- **约行数**：104
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/RemoteAndroid` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 104 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `RemotePhoneIdleGuide.tsx`

- **所在目录**：`dashboard/src/pages/Control/RemoteAndroid/`
- **相对路径**：`dashboard/src/pages/Control/RemoteAndroid/RemotePhoneIdleGuide.tsx`
- **约行数**：213
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/RemoteAndroid` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 213 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Control/RemoteAndroid/`
- **相对路径**：`dashboard/src/pages/Control/RemoteAndroid/index.tsx`
- **约行数**：1563
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/RemoteAndroid` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1563 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useAdbShell.ts`

- **所在目录**：`dashboard/src/pages/Control/RemoteAndroid/`
- **相对路径**：`dashboard/src/pages/Control/RemoteAndroid/useAdbShell.ts`
- **约行数**：109
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/RemoteAndroid` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 109 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Control/RemoteBrowser/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Control/RemoteBrowser/`
- **相对路径**：`dashboard/src/pages/Control/RemoteBrowser/index.tsx`
- **约行数**：1608
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/RemoteBrowser` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1608 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Control/RemoteDesktop/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `DesktopPanel.tsx`

- **所在目录**：`dashboard/src/pages/Control/RemoteDesktop/`
- **相对路径**：`dashboard/src/pages/Control/RemoteDesktop/DesktopPanel.tsx`
- **约行数**：1344
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/RemoteDesktop` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1344 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `desktopShortcuts.ts`

- **所在目录**：`dashboard/src/pages/Control/RemoteDesktop/`
- **相对路径**：`dashboard/src/pages/Control/RemoteDesktop/desktopShortcuts.ts`
- **约行数**：15
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/RemoteDesktop` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 15 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Control/RemoteDesktop/`
- **相对路径**：`dashboard/src/pages/Control/RemoteDesktop/index.tsx`
- **约行数**：104
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/RemoteDesktop` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 104 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Control/Terminal/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ensureDefaultSession.test.ts`

- **所在目录**：`dashboard/src/pages/Control/Terminal/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/ensureDefaultSession.test.ts`
- **约行数**：32
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 32 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Control/Terminal/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/index.tsx`
- **约行数**：704
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 704 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `terminalContext.test.ts`

- **所在目录**：`dashboard/src/pages/Control/Terminal/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/terminalContext.test.ts`
- **约行数**：74
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 74 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `terminalContext.ts`

- **所在目录**：`dashboard/src/pages/Control/Terminal/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/terminalContext.ts`
- **约行数**：67
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 67 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `terminalOutputBuffer.ts`

- **所在目录**：`dashboard/src/pages/Control/Terminal/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/terminalOutputBuffer.ts`
- **约行数**：107
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 107 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `terminalThemes.ts`

- **所在目录**：`dashboard/src/pages/Control/Terminal/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/terminalThemes.ts`
- **约行数**：299
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 299 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useTerminal.testUtils.ts`

- **所在目录**：`dashboard/src/pages/Control/Terminal/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/useTerminal.testUtils.ts`
- **约行数**：6
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 6 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useTerminal.ts`

- **所在目录**：`dashboard/src/pages/Control/Terminal/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/useTerminal.ts`
- **约行数**：698
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 698 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Control/Terminal/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AiPanel.tsx`

- **所在目录**：`dashboard/src/pages/Control/Terminal/components/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/components/AiPanel.tsx`
- **约行数**：490
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 490 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AutopilotPlan.tsx`

- **所在目录**：`dashboard/src/pages/Control/Terminal/components/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/components/AutopilotPlan.tsx`
- **约行数**：117
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 117 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TerminalView.tsx`

- **所在目录**：`dashboard/src/pages/Control/Terminal/components/`
- **相对路径**：`dashboard/src/pages/Control/Terminal/components/TerminalView.tsx`
- **约行数**：206
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Terminal/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 206 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Control/TokenUsage/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `UsageStats.test.tsx`

- **所在目录**：`dashboard/src/pages/Control/TokenUsage/`
- **相对路径**：`dashboard/src/pages/Control/TokenUsage/UsageStats.test.tsx`
- **约行数**：86
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/TokenUsage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 86 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `UsageStats.tsx`

- **所在目录**：`dashboard/src/pages/Control/TokenUsage/`
- **相对路径**：`dashboard/src/pages/Control/TokenUsage/UsageStats.tsx`
- **约行数**：143
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/TokenUsage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 143 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Control/TokenUsage/`
- **相对路径**：`dashboard/src/pages/Control/TokenUsage/index.tsx`
- **约行数**：1132
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/TokenUsage` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1132 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Control/Workbench/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Control/Workbench/`
- **相对路径**：`dashboard/src/pages/Control/Workbench/index.tsx`
- **约行数**：104
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Control/Workbench` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 104 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Experts/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Experts/`
- **相对路径**：`dashboard/src/pages/Experts/index.tsx`
- **约行数**：576
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 576 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Experts/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AgentBackendFields.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/AgentBackendFields.tsx`
- **约行数**：300
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 300 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AgentCard.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/AgentCard.tsx`
- **约行数**：567
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 567 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AgentExpertsTable.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/AgentExpertsTable.tsx`
- **约行数**：634
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 634 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AgentMoreActions.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/AgentMoreActions.tsx`
- **约行数**：98
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 98 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `AgentTrajectoryField.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/AgentTrajectoryField.tsx`
- **约行数**：22
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 22 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `CatalogDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/CatalogDrawer.tsx`
- **约行数**：59
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 59 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ChannelCatalogDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/ChannelCatalogDrawer.tsx`
- **约行数**：37
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 37 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `CreateFromExpertDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/CreateFromExpertDrawer.tsx`
- **约行数**：913
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 913 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `EditAgentDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/EditAgentDrawer.tsx`
- **约行数**：1340
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1340 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExpertAvatarPicker.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/ExpertAvatarPicker.tsx`
- **约行数**：164
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 164 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExpertCard.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/ExpertCard.tsx`
- **约行数**：95
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 95 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExpertComposerDefaultsFields.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/ExpertComposerDefaultsFields.tsx`
- **约行数**：116
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 116 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ExpertMarketTab.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/ExpertMarketTab.tsx`
- **约行数**：516
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 516 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `FileEditModal.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/FileEditModal.tsx`
- **约行数**：180
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 180 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MbtiCatalogDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/MbtiCatalogDrawer.tsx`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `MemoryCatalogDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/MemoryCatalogDrawer.tsx`
- **约行数**：42
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 42 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PluginCatalogDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/PluginCatalogDrawer.tsx`
- **约行数**：30
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 30 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PublishExpertDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/PublishExpertDrawer.tsx`
- **约行数**：229
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 229 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PublishTemplateButton.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/PublishTemplateButton.tsx`
- **约行数**：132
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 132 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PublishedExpertCard.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/PublishedExpertCard.tsx`
- **约行数**：122
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 122 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `RootDirSelect.expand.test.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/RootDirSelect.expand.test.tsx`
- **约行数**：155
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 155 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `RootDirSelect.test.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/RootDirSelect.test.tsx`
- **约行数**：296
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 296 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `RootDirSelect.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/RootDirSelect.tsx`
- **约行数**：445
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 445 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillCatalogDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/SkillCatalogDrawer.tsx`
- **约行数**：30
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 30 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SubagentCatalogDrawer.test.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/SubagentCatalogDrawer.test.tsx`
- **约行数**：68
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 68 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SubagentCatalogDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/SubagentCatalogDrawer.tsx`
- **约行数**：57
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 57 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SubagentDrawer.test.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/SubagentDrawer.test.ts`
- **约行数**：35
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 35 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SubagentDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/SubagentDrawer.tsx`
- **约行数**：390
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 390 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SubagentManager.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/SubagentManager.tsx`
- **约行数**：669
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 669 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SubagentPreviewDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/SubagentPreviewDrawer.tsx`
- **约行数**：86
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 86 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ToolCatalogDrawer.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/ToolCatalogDrawer.tsx`
- **约行数**：30
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 30 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `WelcomeConfig.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/WelcomeConfig.tsx`
- **约行数**：384
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 384 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `agentBackendForm.ensureBwrap.test.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/agentBackendForm.ensureBwrap.test.ts`
- **约行数**：20
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 20 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `agentBackendForm.skillPackages.test.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/agentBackendForm.skillPackages.test.ts`
- **约行数**：89
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 89 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `agentBackendForm.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/agentBackendForm.ts`
- **约行数**：332
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 332 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `expertFileGroups.test.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/expertFileGroups.test.ts`
- **约行数**：43
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 43 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `expertFileGroups.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/expertFileGroups.ts`
- **约行数**：135
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 135 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `iconForName.tsx`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/iconForName.tsx`
- **约行数**：252
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 252 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `rootDirTree.test.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/rootDirTree.test.ts`
- **约行数**：231
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 231 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `rootDirTree.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/rootDirTree.ts`
- **约行数**：209
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 209 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sharedExpert.test.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/sharedExpert.test.ts`
- **约行数**：60
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 60 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `welcomeManifest.test.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/welcomeManifest.test.ts`
- **约行数**：109
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 109 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `welcomeManifest.ts`

- **所在目录**：`dashboard/src/pages/Experts/components/`
- **相对路径**：`dashboard/src/pages/Experts/components/welcomeManifest.ts`
- **约行数**：77
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Experts/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 77 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Invite/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Invite/`
- **相对路径**：`dashboard/src/pages/Invite/index.tsx`
- **约行数**：339
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Invite` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 339 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/KnowledgeBases/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `TextDocumentEditorModal.test.tsx`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/TextDocumentEditorModal.test.tsx`
- **约行数**：102
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 102 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TextDocumentEditorModal.tsx`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/TextDocumentEditorModal.tsx`
- **约行数**：271
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 271 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/index.tsx`
- **约行数**：3456
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 3456 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeDeepLink.test.ts`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/knowledgeDeepLink.test.ts`
- **约行数**：41
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 41 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeDeepLink.ts`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/knowledgeDeepLink.ts`
- **约行数**：31
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 31 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeFolder.test.ts`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/knowledgeFolder.test.ts`
- **约行数**：57
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 57 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeFolder.ts`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/knowledgeFolder.ts`
- **约行数**：27
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 27 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeIcons.tsx`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/knowledgeIcons.tsx`
- **约行数**：66
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 66 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mdOutline.test.ts`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/mdOutline.test.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mdOutline.ts`

- **所在目录**：`dashboard/src/pages/KnowledgeBases/`
- **相对路径**：`dashboard/src/pages/KnowledgeBases/mdOutline.ts`
- **约行数**：47
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/KnowledgeBases` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 47 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Login/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `CaptchaField.test.tsx`

- **所在目录**：`dashboard/src/pages/Login/`
- **相对路径**：`dashboard/src/pages/Login/CaptchaField.test.tsx`
- **约行数**：140
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Login` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 140 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `CaptchaField.tsx`

- **所在目录**：`dashboard/src/pages/Login/`
- **相对路径**：`dashboard/src/pages/Login/CaptchaField.tsx`
- **约行数**：291
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Login` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 291 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `OidcComplete.test.tsx`

- **所在目录**：`dashboard/src/pages/Login/`
- **相对路径**：`dashboard/src/pages/Login/OidcComplete.test.tsx`
- **约行数**：37
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Login` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 37 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `OidcComplete.tsx`

- **所在目录**：`dashboard/src/pages/Login/`
- **相对路径**：`dashboard/src/pages/Login/OidcComplete.tsx`
- **约行数**：125
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Login` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 125 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SlideCaptcha.test.tsx`

- **所在目录**：`dashboard/src/pages/Login/`
- **相对路径**：`dashboard/src/pages/Login/SlideCaptcha.test.tsx`
- **约行数**：86
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Login` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 86 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SlideCaptcha.tsx`

- **所在目录**：`dashboard/src/pages/Login/`
- **相对路径**：`dashboard/src/pages/Login/SlideCaptcha.tsx`
- **约行数**：247
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Login` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 247 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `captchaAdapters.ts`

- **所在目录**：`dashboard/src/pages/Login/`
- **相对路径**：`dashboard/src/pages/Login/captchaAdapters.ts`
- **约行数**：76
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Login` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 76 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Login/`
- **相对路径**：`dashboard/src/pages/Login/index.tsx`
- **约行数**：364
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Login` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 364 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/PwaDebug/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/PwaDebug/`
- **相对路径**：`dashboard/src/pages/PwaDebug/index.tsx`
- **约行数**：704
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/PwaDebug` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 704 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/AdvancedSettings/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `CaptchaSettings.tsx`

- **所在目录**：`dashboard/src/pages/Settings/AdvancedSettings/`
- **相对路径**：`dashboard/src/pages/Settings/AdvancedSettings/CaptchaSettings.tsx`
- **约行数**：233
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/AdvancedSettings` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 233 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `TabPanelHeader.tsx`

- **所在目录**：`dashboard/src/pages/Settings/AdvancedSettings/`
- **相对路径**：`dashboard/src/pages/Settings/AdvancedSettings/TabPanelHeader.tsx`
- **约行数**：36
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/AdvancedSettings` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 36 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `UpdateConfig.tsx`

- **所在目录**：`dashboard/src/pages/Settings/AdvancedSettings/`
- **相对路径**：`dashboard/src/pages/Settings/AdvancedSettings/UpdateConfig.tsx`
- **约行数**：578
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/AdvancedSettings` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 578 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/AdvancedSettings/`
- **相对路径**：`dashboard/src/pages/Settings/AdvancedSettings/index.tsx`
- **约行数**：100
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/AdvancedSettings` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 100 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/BackupRestore/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/BackupRestore/`
- **相对路径**：`dashboard/src/pages/Settings/BackupRestore/index.tsx`
- **约行数**：879
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/BackupRestore` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 879 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Embedding/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Embedding/`
- **相对路径**：`dashboard/src/pages/Settings/Embedding/index.tsx`
- **约行数**：512
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Embedding` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 512 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Environments/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Environments/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/index.tsx`
- **约行数**：311
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 311 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useEnvVars.ts`

- **所在目录**：`dashboard/src/pages/Settings/Environments/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/useEnvVars.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Environments/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AddButton.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Environments/components/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/components/AddButton.tsx`
- **约行数**：26
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 26 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `EmptyState.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Environments/components/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/components/EmptyState.tsx`
- **约行数**：21
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 21 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `EnvRow.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Environments/components/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/components/EnvRow.tsx`
- **约行数**：112
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 112 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `EnvRowCard.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Environments/components/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/components/EnvRowCard.tsx`
- **约行数**：112
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 112 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PageHeader.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Environments/components/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/components/PageHeader.tsx`
- **约行数**：44
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 44 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `Toolbar.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Environments/components/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/components/Toolbar.tsx`
- **约行数**：91
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 91 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/pages/Settings/Environments/components/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/components/index.ts`
- **约行数**：7
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 7 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Environments/utils/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.ts`

- **所在目录**：`dashboard/src/pages/Settings/Environments/utils/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/utils/index.ts`
- **约行数**：2
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 2 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sensitive.test.ts`

- **所在目录**：`dashboard/src/pages/Settings/Environments/utils/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/utils/sensitive.test.ts`
- **约行数**：112
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 112 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sensitive.ts`

- **所在目录**：`dashboard/src/pages/Settings/Environments/utils/`
- **相对路径**：`dashboard/src/pages/Settings/Environments/utils/sensitive.ts`
- **约行数**：78
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Environments/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 78 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/HttpsSettings/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/HttpsSettings/`
- **相对路径**：`dashboard/src/pages/Settings/HttpsSettings/index.tsx`
- **约行数**：361
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/HttpsSettings` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 361 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Language/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Language/`
- **相对路径**：`dashboard/src/pages/Settings/Language/index.tsx`
- **约行数**：64
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Language` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 64 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/MediaGeneration/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/MediaGeneration/`
- **相对路径**：`dashboard/src/pages/Settings/MediaGeneration/index.tsx`
- **约行数**：361
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/MediaGeneration` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 361 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Models/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/`
- **相对路径**：`dashboard/src/pages/Settings/Models/index.tsx`
- **约行数**：428
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 428 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `modelMeta.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/`
- **相对路径**：`dashboard/src/pages/Settings/Models/modelMeta.tsx`
- **约行数**：90
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 90 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `presetUtils.ts`

- **所在目录**：`dashboard/src/pages/Settings/Models/`
- **相对路径**：`dashboard/src/pages/Settings/Models/presetUtils.ts`
- **约行数**：216
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 216 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `providerApi.ts`

- **所在目录**：`dashboard/src/pages/Settings/Models/`
- **相对路径**：`dashboard/src/pages/Settings/Models/providerApi.ts`
- **约行数**：84
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 84 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useProviders.ts`

- **所在目录**：`dashboard/src/pages/Settings/Models/`
- **相对路径**：`dashboard/src/pages/Settings/Models/useProviders.ts`
- **约行数**：195
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 195 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `wizardModelMeta.ts`

- **所在目录**：`dashboard/src/pages/Settings/Models/`
- **相对路径**：`dashboard/src/pages/Settings/Models/wizardModelMeta.ts`
- **约行数**：81
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 81 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Models/components/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `CodexOAuthConnect.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/CodexOAuthConnect.tsx`
- **约行数**：141
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 141 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PresetModelPicker.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/PresetModelPicker.tsx`
- **约行数**：92
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 92 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/index.ts`
- **约行数**：6
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 6 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Models/components/cards/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `LocalServiceCard.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/cards/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/cards/LocalServiceCard.tsx`
- **约行数**：312
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/cards` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 312 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PresetGroupCard.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/cards/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/cards/PresetGroupCard.tsx`
- **约行数**：264
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/cards` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 264 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PresetProviderCard.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/cards/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/cards/PresetProviderCard.tsx`
- **约行数**：155
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/cards` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 155 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ProviderCard.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/cards/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/cards/ProviderCard.tsx`
- **约行数**：386
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/cards` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 386 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/cards/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/cards/index.ts`
- **约行数**：5
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/cards` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 5 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Models/components/modals/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `CustomProviderModal.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/modals/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/modals/CustomProviderModal.tsx`
- **约行数**：360
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/modals` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 360 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ModelListEditor.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/modals/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/modals/ModelListEditor.tsx`
- **约行数**：860
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/modals` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 860 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PresetProviderModal.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/modals/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/modals/PresetProviderModal.tsx`
- **约行数**：362
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/modals` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 362 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ProviderConfigModal.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/modals/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/modals/ProviderConfigModal.tsx`
- **约行数**：1228
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/modals` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1228 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/modals/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/modals/index.ts`
- **约行数**：5
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/modals` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 5 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Models/components/sections/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ActiveModelPool.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/sections/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/sections/ActiveModelPool.tsx`
- **约行数**：392
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/sections` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 392 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `LoadingState.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/sections/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/sections/LoadingState.tsx`
- **约行数**：42
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/sections` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 42 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/pages/Settings/Models/components/sections/`
- **相对路径**：`dashboard/src/pages/Settings/Models/components/sections/index.ts`
- **约行数**：3
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Models/components/sections` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 3 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Observability/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Observability/`
- **相对路径**：`dashboard/src/pages/Settings/Observability/index.tsx`
- **约行数**：199
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Observability` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 199 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/SearchConfig/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.test.tsx`

- **所在目录**：`dashboard/src/pages/Settings/SearchConfig/`
- **相对路径**：`dashboard/src/pages/Settings/SearchConfig/index.test.tsx`
- **约行数**：52
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/SearchConfig` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 52 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/SearchConfig/`
- **相对路径**：`dashboard/src/pages/Settings/SearchConfig/index.tsx`
- **约行数**：477
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/SearchConfig` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 477 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Security/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AuditLogPanel.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Security/`
- **相对路径**：`dashboard/src/pages/Settings/Security/AuditLogPanel.tsx`
- **约行数**：314
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Security` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 314 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `HitlToolsPicker.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Security/`
- **相对路径**：`dashboard/src/pages/Settings/Security/HitlToolsPicker.tsx`
- **约行数**：115
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Security` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 115 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ToolGuardRulesPanel.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Security/`
- **相对路径**：`dashboard/src/pages/Settings/Security/ToolGuardRulesPanel.tsx`
- **约行数**：261
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Security` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 261 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Security/`
- **相对路径**：`dashboard/src/pages/Settings/Security/index.tsx`
- **约行数**：426
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Security` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 426 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/Voice/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Settings/Voice/`
- **相对路径**：`dashboard/src/pages/Settings/Voice/index.tsx`
- **约行数**：501
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/Voice` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 501 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Settings/octop/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AdminAgentCard.tsx`

- **所在目录**：`dashboard/src/pages/Settings/octop/`
- **相对路径**：`dashboard/src/pages/Settings/octop/AdminAgentCard.tsx`
- **约行数**：235
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/octop` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 235 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `Agents.tsx`

- **所在目录**：`dashboard/src/pages/Settings/octop/`
- **相对路径**：`dashboard/src/pages/Settings/octop/Agents.tsx`
- **约行数**：401
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/octop` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 401 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `Providers.tsx`

- **所在目录**：`dashboard/src/pages/Settings/octop/`
- **相对路径**：`dashboard/src/pages/Settings/octop/Providers.tsx`
- **约行数**：226
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Settings/octop` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 226 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Setup/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/Setup/`
- **相对路径**：`dashboard/src/pages/Setup/index.tsx`
- **约行数**：308
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Setup` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 308 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `wizardClient.ts`

- **所在目录**：`dashboard/src/pages/Setup/`
- **相对路径**：`dashboard/src/pages/Setup/wizardClient.ts`
- **约行数**：224
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Setup` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 224 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/Setup/steps/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `AdminStep.tsx`

- **所在目录**：`dashboard/src/pages/Setup/steps/`
- **相对路径**：`dashboard/src/pages/Setup/steps/AdminStep.tsx`
- **约行数**：253
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Setup/steps` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 253 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `DatabaseStep.tsx`

- **所在目录**：`dashboard/src/pages/Setup/steps/`
- **相对路径**：`dashboard/src/pages/Setup/steps/DatabaseStep.tsx`
- **约行数**：323
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Setup/steps` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 323 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `FinishStep.tsx`

- **所在目录**：`dashboard/src/pages/Setup/steps/`
- **相对路径**：`dashboard/src/pages/Setup/steps/FinishStep.tsx`
- **约行数**：121
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Setup/steps` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 121 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ModelStep.tsx`

- **所在目录**：`dashboard/src/pages/Setup/steps/`
- **相对路径**：`dashboard/src/pages/Setup/steps/ModelStep.tsx`
- **约行数**：1294
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Setup/steps` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1294 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PasswordStep.tsx`

- **所在目录**：`dashboard/src/pages/Setup/steps/`
- **相对路径**：`dashboard/src/pages/Setup/steps/PasswordStep.tsx`
- **约行数**：132
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/Setup/steps` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 132 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/pages/SkillPackages/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `PackageIcon.tsx`

- **所在目录**：`dashboard/src/pages/SkillPackages/`
- **相对路径**：`dashboard/src/pages/SkillPackages/PackageIcon.tsx`
- **约行数**：21
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/SkillPackages` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 21 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PackageSkillCard.tsx`

- **所在目录**：`dashboard/src/pages/SkillPackages/`
- **相对路径**：`dashboard/src/pages/SkillPackages/PackageSkillCard.tsx`
- **约行数**：112
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/SkillPackages` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 112 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `PackageSkillsTable.tsx`

- **所在目录**：`dashboard/src/pages/SkillPackages/`
- **相对路径**：`dashboard/src/pages/SkillPackages/PackageSkillsTable.tsx`
- **约行数**：103
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/SkillPackages` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 103 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillsetFromHubDrawer.test.tsx`

- **所在目录**：`dashboard/src/pages/SkillPackages/`
- **相对路径**：`dashboard/src/pages/SkillPackages/SkillsetFromHubDrawer.test.tsx`
- **约行数**：98
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/SkillPackages` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 98 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `SkillsetFromHubDrawer.tsx`

- **所在目录**：`dashboard/src/pages/SkillPackages/`
- **相对路径**：`dashboard/src/pages/SkillPackages/SkillsetFromHubDrawer.tsx`
- **约行数**：148
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/SkillPackages` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 148 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/pages/SkillPackages/`
- **相对路径**：`dashboard/src/pages/SkillPackages/index.tsx`
- **约行数**：1083
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/pages/SkillPackages` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 1083 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/plugins/toolRenderers/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `ToolUiErrorBoundary.tsx`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/ToolUiErrorBoundary.tsx`
- **约行数**：40
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 40 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ensureBuiltins.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/ensureBuiltins.ts`
- **约行数**：35
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 35 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `host.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/host.ts`
- **约行数**：90
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 90 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/index.ts`
- **约行数**：38
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 38 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `isPinnedToolUi.test.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/isPinnedToolUi.test.ts`
- **约行数**：120
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 120 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `isPinnedToolUi.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/isPinnedToolUi.ts`
- **约行数**：66
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 66 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `loader.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/loader.ts`
- **约行数**：96
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 96 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseToolOutput.test.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/parseToolOutput.test.ts`
- **约行数**：92
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 92 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseToolOutput.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/parseToolOutput.ts`
- **约行数**：85
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 85 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `registry.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/registry.ts`
- **约行数**：101
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 101 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `toolPluginIndex.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/toolPluginIndex.ts`
- **约行数**：24
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 24 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `types.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/types.ts`
- **约行数**：89
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 89 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `usePluginToolUis.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/usePluginToolUis.ts`
- **约行数**：51
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 51 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `useToolRendererVersion.ts`

- **所在目录**：`dashboard/src/plugins/toolRenderers/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/useToolRendererVersion.ts`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/plugins/toolRenderers/builtin/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `BuiltinOctopUiFallback.tsx`

- **所在目录**：`dashboard/src/plugins/toolRenderers/builtin/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/builtin/BuiltinOctopUiFallback.tsx`
- **约行数**：89
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers/builtin` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 89 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `DefaultToolRenderer.tsx`

- **所在目录**：`dashboard/src/plugins/toolRenderers/builtin/`
- **相对路径**：`dashboard/src/plugins/toolRenderers/builtin/DefaultToolRenderer.tsx`
- **约行数**：245
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/plugins/toolRenderers/builtin` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 245 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/routes/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `controlAdminPath.test.ts`

- **所在目录**：`dashboard/src/routes/`
- **相对路径**：`dashboard/src/routes/controlAdminPath.test.ts`
- **约行数**：127
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/routes` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 127 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `index.tsx`

- **所在目录**：`dashboard/src/routes/`
- **相对路径**：`dashboard/src/routes/index.tsx`
- **约行数**：288
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/routes` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 288 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `prefetch.ts`

- **所在目录**：`dashboard/src/routes/`
- **相对路径**：`dashboard/src/routes/prefetch.ts`
- **约行数**：45
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/routes` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 45 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/styles/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `appearanceStorage.test.ts`

- **所在目录**：`dashboard/src/styles/`
- **相对路径**：`dashboard/src/styles/appearanceStorage.test.ts`
- **约行数**：119
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/styles` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 119 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `appearanceStorage.ts`

- **所在目录**：`dashboard/src/styles/`
- **相对路径**：`dashboard/src/styles/appearanceStorage.ts`
- **约行数**：109
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/styles` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 109 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `customPalette.test.ts`

- **所在目录**：`dashboard/src/styles/`
- **相对路径**：`dashboard/src/styles/customPalette.test.ts`
- **约行数**：122
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/styles` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 122 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `themePalettes.test.ts`

- **所在目录**：`dashboard/src/styles/`
- **相对路径**：`dashboard/src/styles/themePalettes.test.ts`
- **约行数**：72
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/styles` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 72 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `themePalettes.ts`

- **所在目录**：`dashboard/src/styles/`
- **相对路径**：`dashboard/src/styles/themePalettes.ts`
- **约行数**：468
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/styles` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 468 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/test/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `memoryFixtures.ts`

- **所在目录**：`dashboard/src/test/`
- **相对路径**：`dashboard/src/test/memoryFixtures.ts`
- **约行数**：213
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/test` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 213 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `setup.ts`

- **所在目录**：`dashboard/src/test/`
- **相对路径**：`dashboard/src/test/setup.ts`
- **约行数**：131
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/test` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 131 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/utils/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `agentError.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/agentError.ts`
- **约行数**：77
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 77 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `agentRuntimeConfig.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/agentRuntimeConfig.test.ts`
- **约行数**：48
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 48 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `agentRuntimeConfig.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/agentRuntimeConfig.ts`
- **约行数**：58
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 58 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `antdMessage.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/antdMessage.ts`
- **约行数**：30
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 30 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `antdModal.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/antdModal.ts`
- **约行数**：29
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 29 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `apiError.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/apiError.ts`
- **约行数**：125
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 125 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `browserCanvas.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/browserCanvas.ts`
- **约行数**：64
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 64 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `browserProfile.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/browserProfile.ts`
- **约行数**：11
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 11 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `browserSpeech.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/browserSpeech.ts`
- **约行数**：270
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 270 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `browserTabs.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/browserTabs.ts`
- **约行数**：95
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 95 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `browserViewport.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/browserViewport.ts`
- **约行数**：22
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 22 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatStreamError.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/chatStreamError.test.ts`
- **约行数**：85
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 85 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `chatStreamError.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/chatStreamError.ts`
- **约行数**：183
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 183 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `collectTurnKnowledgeCitations.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/collectTurnKnowledgeCitations.ts`
- **约行数**：24
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 24 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `collectTurnToolMedia.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/collectTurnToolMedia.ts`
- **约行数**：46
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 46 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `confirmModal.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/confirmModal.ts`
- **约行数**：28
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 28 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `copyText.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/copyText.test.ts`
- **约行数**：63
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 63 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `copyText.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/copyText.ts`
- **约行数**：43
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 43 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `dashboardPushToast.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/dashboardPushToast.test.ts`
- **约行数**：53
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 53 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `dashboardPushToast.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/dashboardPushToast.ts`
- **约行数**：34
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 34 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `desktopChrome.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/desktopChrome.test.ts`
- **约行数**：202
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 202 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `desktopChrome.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/desktopChrome.ts`
- **约行数**：173
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 173 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `desktopExternalLinks.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/desktopExternalLinks.ts`
- **约行数**：114
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 114 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `desktopViewport.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/desktopViewport.ts`
- **约行数**：29
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 29 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `detailRequestGate.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/detailRequestGate.test.ts`
- **约行数**：14
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 14 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `detailRequestGate.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/detailRequestGate.ts`
- **约行数**：13
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 13 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `docKind.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/docKind.ts`
- **约行数**：44
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 44 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `embeddingDownload.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/embeddingDownload.ts`
- **约行数**：70
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 70 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `expertColor.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/expertColor.test.ts`
- **约行数**：78
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 78 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `expertColor.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/expertColor.ts`
- **约行数**：120
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 120 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `fileTreeIcon.tsx`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/fileTreeIcon.tsx`
- **约行数**：192
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 192 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `formDraft.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/formDraft.ts`
- **约行数**：51
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 51 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `formatMessageTime.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/formatMessageTime.test.ts`
- **约行数**：77
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 77 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `formatMessageTime.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/formatMessageTime.ts`
- **约行数**：136
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 136 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `formatToolArguments.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/formatToolArguments.ts`
- **约行数**：45
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 45 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `fromWorkspace.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/fromWorkspace.ts`
- **约行数**：11
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 11 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `h264CanvasDecoder.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/h264CanvasDecoder.ts`
- **约行数**：111
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 111 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `headerUtils.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/headerUtils.ts`
- **约行数**：61
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 61 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `injectPendingHitlMessage.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/injectPendingHitlMessage.ts`
- **约行数**：30
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 30 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeCitationDisplay.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/knowledgeCitationDisplay.test.ts`
- **约行数**：80
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 80 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeCitationDisplay.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/knowledgeCitationDisplay.ts`
- **约行数**：54
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 54 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeDocPreview.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/knowledgeDocPreview.test.ts`
- **约行数**：82
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 82 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgeDocPreview.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/knowledgeDocPreview.ts`
- **约行数**：117
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 117 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgePath.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/knowledgePath.test.ts`
- **约行数**：39
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 39 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `knowledgePath.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/knowledgePath.ts`
- **约行数**：61
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 61 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `locale.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/locale.ts`
- **约行数**：46
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 46 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `localePrefs.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/localePrefs.test.ts`
- **约行数**：53
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 53 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `localePrefs.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/localePrefs.ts`
- **约行数**：64
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 64 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `localizedText.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/localizedText.ts`
- **约行数**：23
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 23 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `markdown.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/markdown.ts`
- **约行数**：140
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 140 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `messageParser.history.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/messageParser.history.test.ts`
- **约行数**：51
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 51 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `messageParser.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/messageParser.ts`
- **约行数**：366
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 366 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mobileDevice.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/mobileDevice.ts`
- **约行数**：10
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 10 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `mobileTypeScale.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/mobileTypeScale.ts`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `modelOptions.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/modelOptions.ts`
- **约行数**：55
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 55 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `normalizeUrl.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/normalizeUrl.ts`
- **约行数**：8
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 8 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseAcpPermission.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/parseAcpPermission.ts`
- **约行数**：66
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 66 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseHarnessChunk.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/parseHarnessChunk.test.ts`
- **约行数**：27
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 27 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseHarnessChunk.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/parseHarnessChunk.ts`
- **约行数**：249
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 249 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseKnowledgeCitations.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/parseKnowledgeCitations.test.ts`
- **约行数**：47
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 47 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseKnowledgeCitations.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/parseKnowledgeCitations.ts`
- **约行数**：59
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 59 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseWriteTodos.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/parseWriteTodos.test.ts`
- **约行数**：116
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 116 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `parseWriteTodos.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/parseWriteTodos.ts`
- **约行数**：167
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 167 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `passwordPolicy.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/passwordPolicy.ts`
- **约行数**：49
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 49 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `pastelIconBackground.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/pastelIconBackground.ts`
- **约行数**：43
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 43 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `permissions.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/permissions.ts`
- **约行数**：237
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 237 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `plainTextForSpeech.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/plainTextForSpeech.test.ts`
- **约行数**：55
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 55 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `plainTextForSpeech.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/plainTextForSpeech.ts`
- **约行数**：61
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 61 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `quickInputPrefill.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/quickInputPrefill.ts`
- **约行数**：37
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 37 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `readNumber.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/readNumber.ts`
- **约行数**：12
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 12 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `reloadOnStaleChunk.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/reloadOnStaleChunk.test.ts`
- **约行数**：234
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 234 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `reloadOnStaleChunk.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/reloadOnStaleChunk.ts`
- **约行数**：143
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 143 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `sharedExpert.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/sharedExpert.ts`
- **约行数**：35
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 35 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `shellCodeBlock.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/shellCodeBlock.ts`
- **约行数**：26
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 26 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `showApiToast.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/showApiToast.ts`
- **约行数**：14
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 14 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `slashCategories.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/slashCategories.ts`
- **约行数**：55
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 55 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `slashFallbackCommands.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/slashFallbackCommands.ts`
- **约行数**：94
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 94 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `slashIcons.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/slashIcons.ts`
- **约行数**：49
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 49 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `ssoPopup.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/ssoPopup.ts`
- **约行数**：48
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 48 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `subagentEmojis.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/subagentEmojis.test.ts`
- **约行数**：19
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 19 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `subagentEmojis.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/subagentEmojis.ts`
- **约行数**：125
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 125 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `toolMediaBlocks.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/toolMediaBlocks.test.ts`
- **约行数**：198
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 198 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `toolMediaBlocks.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/toolMediaBlocks.ts`
- **约行数**：867
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 867 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `updateStatusCache.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/updateStatusCache.test.ts`
- **约行数**：78
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 78 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `updateStatusCache.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/updateStatusCache.ts`
- **约行数**：73
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 73 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `wavStreamPlayer.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/wavStreamPlayer.test.ts`
- **约行数**：91
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 91 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `wavStreamPlayer.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/wavStreamPlayer.ts`
- **约行数**：191
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 191 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `workspaceIoPath.test.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/workspaceIoPath.test.ts`
- **约行数**：43
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 43 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `workspaceIoPath.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/workspaceIoPath.ts`
- **约行数**：88
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 88 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。

#### `workspacePath.ts`

- **所在目录**：`dashboard/src/utils/`
- **相对路径**：`dashboard/src/utils/workspacePath.ts`
- **约行数**：44
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 44 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


### 目录 `dashboard/src/utils/__tests__/`

该目录在分层上的位置见 `1DESIGN.md`。下列文件均应保持职责单一，避免跨层 import。

#### `browserTabs.test.ts`

- **所在目录**：`dashboard/src/utils/__tests__/`
- **相对路径**：`dashboard/src/utils/__tests__/browserTabs.test.ts`
- **约行数**：101
- **主要功能简介**：该 TypeScript 模块属于控制台 `dashboard/src/utils/__tests__` 区域，通过 `dashboard/src/api` 访问后端，不直接操作数据库。若路径含 `pages`，则为路由页；含 `components` 则为可复用组件；含 `hooks` 则为钩子；含 `api/modules` 则为 HTTP 封装；含 `locales` 则为文案。修改后应运行 `cd dashboard && npx tsc -b`，并遵守服务器时区与 i18n 键对齐规则。行数 101 反映其复杂度，超大文件在后续优化中可按 3SOLUTION 建议拆分，但本次说明只记录现状。


## 3. 构建、测试与持续集成文件（摘要）

仓库根还有 `pyproject.toml`、`Makefile`、`uv.lock`、`conftest.py`、`.pre-commit-config.yaml`、`.github/workflows/*.yml`。它们不是业务运行时，但决定了「什么叫绿」。贡献者必须先 `make install-hooks`。CI 在 Linux 与 Windows 上跑 lint、typecheck、pytest 与前端构建，因此测试不得假设单一操作系统路径形态。

## 4. 非源码但影响运行的资源

专家库 Markdown、bundled 插件、dashboard 构建产物、i18n JSON、SQL 迁移都属于发行件。wheel 的 hatch include 规则把这些资源打进包。不要编辑 `src/octop/dashboard/` 产物。

## 5. 结束语

本文件给出「有哪些源代码、在哪、干什么」的地图。统计会随提交变化，若你正在做大规模重构，重新扫描 AST 是更新本文的可靠方法。实现细节以源码为准；分层禁令以 `AGENTS.md` 为准。
