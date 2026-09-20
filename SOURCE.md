# Octop Source File Inventory

Generated for `/workspace` (excludes `node_modules`, `.git`, `src/octop/dashboard`, `uv.lock`, `dist`).

## Sibling runtime packages

| Path | Present in repo |
|------|------------------|
| `harness-agent/` | no — installed from PyPI (`orcakit-harness-agent`, `harness-gateway` in `pyproject.toml`) |
| `harness-gateway/` | no — installed from PyPI (`orcakit-harness-agent`, `harness-gateway` in `pyproject.toml`) |

## Aggregate statistics

### By file type (inventoried extensions)

| Extension | Files | Lines (approx) |
|-----------|------:|---------------:|
| `.py` | 1025 | 195,817 |
| `.sql` | 30 | 1,452 |
| `.ts` | 446 | 50,318 |
| `.tsx` | 387 | 107,622 |
| **Total** | **1888** | **355,209** |

### By top-level directory

| Directory | Files | Lines (approx) |
|-----------|------:|---------------:|
| `conftest.py/` | 1 | 11 |
| `dashboard/` | 834 | 157,940 |
| `desktop/` | 3 | 301 |
| `plugins/` | 4 | 174 |
| `scripts/` | 2 | 224 |
| `src/` | 613 | 124,811 |
| `tests/` | 431 | 71,748 |

### Breakdown by role

| Role | Files | Lines |
|------|------:|------:|
| src/octop Python | 583 | 123,359 |
| tests Python | 431 | 71,748 |
| dashboard/src TS/TSX | 833 | 157,940 |
| SQL migrations | 30 | 1,452 |
| scripts/conftest | 3 | 235 |
| other Python (plugins/desktop) | 8 | 475 |

### Expert persona library (bundled markdown)

- `src/octop/infra/agents/experts/library/`: **289** `.md` persona/skill dumps (~**44,521** lines), not listed file-by-file.
- Python catalog/loaders for experts are listed under `src/octop/infra/agents/experts/` below.

### Languages & stacks

- **Backend:** Python 3.12+, FastAPI, Click CLI, SQLite/PostgreSQL
- **Frontend:** TypeScript, React 18, Vite, Ant Design
- **Desktop (optional):** Python helpers under `desktop/`
- **Plugins:** sample harness plugins under `plugins/`
- **SQL:** paired SQLite + PostgreSQL migrations in `src/octop/infra/db/migrations/`

## Development tools

### Python (`pyproject.toml`)

- **Package manager:** uv (preferred) / pip; build: hatchling
- **Runtime deps (core):** FastAPI, uvicorn, pydantic, click, rich, APScheduler, argon2, PyJWT, `orcakit-harness-agent[all]`, `harness-gateway`, `harness-memory`, `harness-browser`, langchain-core, MCP, psycopg, boto3, playwright, scalar-fastapi, edge-tts, cryptography, acme/josepy, document parsers (pypdf, python-docx, openpyxl, …)
- **Dev deps:** pytest, pytest-asyncio, pytest-cov, pytest-testmon, pytest-xdist, httpx, ruff, mypy, build
- **Optional extras:** `browser`, `desktop`, `local-embedding`, `knowledge-ocr`
- **Lint/type:** Ruff (check + format), mypy `--strict` on `src/octop`
- **Entry point:** `octop` → `octop.cli.main:cli`

### Frontend (`dashboard/package.json`)

- **Build:** Vite 6, TypeScript 5.8, `tsc -b` + vite build
- **UI:** React 18, Ant Design 5, antd-style, lucide-react, Monaco, xterm, mermaid, recharts
- **i18n:** i18next, react-i18next
- **Quality:** ESLint 9, Prettier 3, Vitest + Testing Library

### Makefile targets (primary)

| Target | Purpose |
|--------|----------|
| `all` | format-all + lint + typecheck + test (ship bar) |
| `build / build-frontend / build-wheel` | Production dashboard + Python wheel |
| `dev / dev-frontend / dev-backend` | Local dev servers |
| `lint / format / typecheck / test` | Backend quality |
| `lint-frontend / format-frontend / typecheck-frontend` | Dashboard quality |
| `lint-all / format-all / typecheck-all / check-all` | Full stack quality |
| `precommit / test-affected` | Git hook gate (testmon-scoped tests) |
| `install / install-dev / install-hooks` | Deps + git hooks |
| `install-online / test-online / run-online` | PyPI harness deps venv |
| `publish / publish-test` | PyPI upload |
| `docs-cli` | Regenerate docs/cli.md |
| `clean / clean-online / version` | Utilities |

### GitHub Actions (`.github/workflows/`)

- **`anti-spam-issues.yml`** — workflow `anti-spam-issues`
- **`auto-tag-on-release.yml`** — workflow `auto-tag-on-release`
- **`ci.yml`** — workflow `ci`
- **`codeql.yml`** — workflow `codeql`
- **`docker-publish.yml`** — workflow `docker-publish`
- **`fnos-build-fpk.yml`** — workflow `fnos-build-fpk`
- **`octop-desktop.yml`** — workflow `octop-desktop`
- **`release.yml`** — workflow `release`
- **`sync-main-to-develop.yml`** — workflow `sync-main-to-develop`

---

## File inventory (by directory)

Columns: **File** | **Lines** | **Description**

### `./`

| File | Lines | Description |
|------|------:|-------------|
| `conftest.py` | 11 | Repo-root conftest. Imported by pytest at startup (before any plugin's ``pytest_configure``), so the testmon change-detection patch in ``tests.support.testmon_staged_changes`` is applied before testmon reads file fingerprints. The patch only affects testmon's internals and is a s… |

### `dashboard/src/`

| File | Lines | Description |
|------|------:|-------------|
| `App.tsx` | 176 | React component or page. |
| `i18n.ts` | 126 | TS utilities, API, hooks, or types. |
| `main.tsx` | 79 | Must be the very first import so the beforeinstallprompt listener is |
| `pwa-prompt.ts` | 140 | TS utilities, API, hooks, or types. |
| `pwa.ts` | 14 | TS utilities, API, hooks, or types. |
| `sw-register.test.ts` | 182 | TS utilities, API, hooks, or types. |
| `sw-register.ts` | 129 | TS utilities, API, hooks, or types. |
| `vite-env.d.ts` | 6 | / <reference types="vite/client" /> |

### `dashboard/src/api/`

| File | Lines | Description |
|------|------:|-------------|
| `config.ts` | 38 | TS utilities, API, hooks, or types. |
| `index.ts` | 89 | TS utilities, API, hooks, or types. |
| `probeHealth.ts` | 43 | TS utilities, API, hooks, or types. |
| `request.setup.test.ts` | 95 | TS utilities, API, hooks, or types. |
| `request.ts` | 599 | TS utilities, API, hooks, or types. |
| `request.unauthorized.test.ts` | 71 | TS utilities, API, hooks, or types. |

### `dashboard/src/api/modules/`

| File | Lines | Description |
|------|------:|-------------|
| `acp.ts` | 67 | TS utilities, API, hooks, or types. |
| `agent.ts` | 145 | Agent API |
| `agentChat.ts` | 33 | TS utilities, API, hooks, or types. |
| `agentTools.ts` | 80 | TS utilities, API, hooks, or types. |
| `auth.ts` | 275 | TS utilities, API, hooks, or types. |
| `backup.ts` | 122 | TS utilities, API, hooks, or types. |
| `browser.ts` | 220 | -- Browser environment types -- |
| `channel.ts` | 268 | TS utilities, API, hooks, or types. |
| `connectors.ts` | 391 | TS utilities, API, hooks, or types. |
| `cronjob.ts` | 51 | TS utilities, API, hooks, or types. |
| `desktop.ts` | 111 | TS utilities, API, hooks, or types. |
| `embedding.ts` | 49 | GET /api/embedding/config |
| `env.ts` | 21 | For backward compatibility |
| `expertMarket.ts` | 104 | TS utilities, API, hooks, or types. |
| `i18n.ts` | 11 | TS utilities, API, hooks, or types. |
| `invites.ts` | 76 | TS utilities, API, hooks, or types. |
| `knowledgeBases.test.ts` | 102 | TS utilities, API, hooks, or types. |
| `knowledgeBases.ts` | 317 | TS utilities, API, hooks, or types. |
| `mbti.ts` | 55 | TS utilities, API, hooks, or types. |
| `mediaGeneration.ts` | 48 | TS utilities, API, hooks, or types. |
| `memoryDashboard.ts` | 532 | --------------------------------------------------------------------------- |
| `memoryPortable.ts` | 148 | --------------------------------------------------------------------------- |
| `mobile.ts` | 114 | TS utilities, API, hooks, or types. |
| `observability.ts` | 35 | TS utilities, API, hooks, or types. |
| `octopAgents.ts` | 22 | TS utilities, API, hooks, or types. |
| `octopThreads.ts` | 226 | TS utilities, API, hooks, or types. |
| `ollamaModel.ts` | 40 | TS utilities, API, hooks, or types. |
| `onnxDownloadWatcher.ts` | 116 | TS utilities, API, hooks, or types. |
| `onnxModel.ts` | 90 | TS utilities, API, hooks, or types. |
| `plugins.ts` | 143 | TS utilities, API, hooks, or types. |
| `preferences.ts` | 39 | TS utilities, API, hooks, or types. |
| `provider.ts` | 120 | TS utilities, API, hooks, or types. |
| `publishedExperts.test.ts` | 74 | TS utilities, API, hooks, or types. |
| `publishedExperts.ts` | 85 | TS utilities, API, hooks, or types. |
| `root.ts` | 7 | Root API |
| `security.ts` | 121 | TS utilities, API, hooks, or types. |
| `settings.ts` | 56 | TS utilities, API, hooks, or types. |
| `skillPackages.test.ts` | 124 | TS utilities, API, hooks, or types. |
| `skillPackages.ts` | 150 | TS utilities, API, hooks, or types. |
| `slash.ts` | 35 | TS utilities, API, hooks, or types. |
| `sso.test.ts` | 37 | TS utilities, API, hooks, or types. |
| `sso.ts` | 96 | TS utilities, API, hooks, or types. |
| `subagents.ts` | 76 | TS utilities, API, hooks, or types. |
| `terminalAi.ts` | 30 | TS utilities, API, hooks, or types. |
| `tls.ts` | 52 | TS utilities, API, hooks, or types. |
| `trajectory.ts` | 111 | TS utilities, API, hooks, or types. |
| `update.ts` | 67 | TS utilities, API, hooks, or types. |
| `upload.ts` | 26 | TS utilities, API, hooks, or types. |
| `voice.ts` | 135 | TS utilities, API, hooks, or types. |
| `workspace.ts` | 226 | TS utilities, API, hooks, or types. |
| `wsChat.ts` | 10 | TS utilities, API, hooks, or types. |
| `wsNotifications.ts` | 10 | TS utilities, API, hooks, or types. |

### `dashboard/src/api/types/`

| File | Lines | Description |
|------|------:|-------------|
| `acp.ts` | 51 | TS utilities, API, hooks, or types. |
| `agent.ts` | 98 | TS utilities, API, hooks, or types. |
| `browser.ts` | 111 | Browser session API types |
| `channel.ts` | 114 | TS utilities, API, hooks, or types. |
| `chat.ts` | 87 | TS utilities, API, hooks, or types. |
| `cronjob.ts` | 109 | TS utilities, API, hooks, or types. |
| `embedding.ts` | 36 | Embedding API type definitions. |
| `env.ts` | 4 | TS utilities, API, hooks, or types. |
| `hitl.ts` | 72 | TS utilities, API, hooks, or types. |
| `index.ts` | 11 | TS utilities, API, hooks, or types. |
| `mbti.ts` | 62 | TS utilities, API, hooks, or types. |
| `provider.ts` | 177 | Optional extended metadata (openclaw-compatible) |
| `skill.ts` | 55 | TS utilities, API, hooks, or types. |
| `skillPackage.ts` | 66 | TS utilities, API, hooks, or types. |
| `workspace.ts` | 56 | TS utilities, API, hooks, or types. |

### `dashboard/src/assets/`

| File | Lines | Description |
|------|------:|-------------|
| `mascot.ts` | 5 | TS utilities, API, hooks, or types. |

### `dashboard/src/assets/connectors/`

| File | Lines | Description |
|------|------:|-------------|
| `index.ts` | 66 | TS utilities, API, hooks, or types. |

### `dashboard/src/assets/providers/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 0 | Octop Python module. |
| `index.ts` | 149 | TS utilities, API, hooks, or types. |

### `dashboard/src/components/`

| File | Lines | Description |
|------|------:|-------------|
| `AgentAdvancedConfigFields.tsx` | 94 | React component or page. |
| `AgentProfileDrawer.test.tsx` | 165 | React component or page. |
| `AgentProfileDrawer.tsx` | 697 | Default export: AgentProfileDrawer. |
| `AgentSelector.tsx` | 155 | Default export: AgentSelector. |
| `AntdAppProvider.tsx` | 28 | React component or page. |
| `AuthFileDownloadLink.tsx` | 144 | React component or page. |
| `AuthGuard.tsx` | 114 | Default export: AuthGuard. |
| `AuthImage.tsx` | 37 | Default export: AuthImage. |
| `AvatarDropdown.tsx` | 821 | Default export: AvatarDropdown. |
| `BackendBuilder.tsx` | 432 | Default export: BackendBuilder. |
| `BrowserAiPanel.tsx` | 704 | Default export: BrowserAiPanel. |
| `CopyableResourceId.tsx` | 55 | React component or page. |
| `DocumentPreviewCore.docxSanitize.test.ts` | 22 | TS utilities, API, hooks, or types. |
| `DocumentPreviewCore.pdfSkeleton.test.tsx` | 30 | Never-resolving fetchBlob keeps the component in the download phase. |
| `DocumentPreviewCore.tsx` | 752 | Default export: DocumentPreviewCore. |
| `DocumentPreviewLoading.tsx` | 58 | Default export: DocumentPreviewLoading. |
| `EmojiPicker.test.tsx` | 24 | React component or page. |
| `EmojiPicker.tsx` | 103 | Default export: EmojiPicker. |
| `ExpertColorPicker.tsx` | 84 | Default export: ExpertColorPicker. |
| `ForbiddenPage.tsx` | 33 | Default export: ForbiddenPage. |
| `LanguageSwitcher.tsx` | 75 | Default export: LanguageSwitcher. |
| `MbtiPersonaTag.tsx` | 99 | Default export: MbtiPersonaTag. |
| `MobileAiPanel.tsx` | 225 | Default export: MobileAiPanel. |
| `NotFoundPage.test.tsx` | 26 | React component or page. |
| `NotFoundPage.tsx` | 23 | Default export: NotFoundPage. |
| `PaletteSwitcher.tsx` | 70 | Default export: PaletteSwitcher. |
| `PdfDocumentPreview.tsx` | 992 | Default export: PdfDocumentPreview. |
| `PdfDocumentPreview.windowed.test.tsx` | 363 | React component or page. |
| `RailEdgeControl.tsx` | 87 | Default export: RailEdgeControl. |
| `RequirePermission.tsx` | 40 | Default export: RequirePermission. |
| `ServiceRestartOverlay.tsx` | 159 | Default export: ServiceRestartOverlay. |
| `SkillRecordGuideModal.tsx` | 145 | Default export: SkillRecordGuideModal. |
| `ThemeSwitcher.tsx` | 72 | Default export: ThemeSwitcher. |
| `TodoListInline.tsx` | 25 | React component or page. |
| `TodoProgressPanel.tsx` | 115 | React component or page. |

### `dashboard/src/components/AppVersionBadge/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 47 | Default export: AppVersionBadge. |

### `dashboard/src/components/BetaBadge/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 24 | Default export: BetaBadge. |

### `dashboard/src/components/BrowserViewer/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 347 | React component or page. |

### `dashboard/src/components/BrowserWorkspace/`

| File | Lines | Description |
|------|------:|-------------|
| `ChatBrowserPanel.tsx` | 57 | React component or page. |
| `ChatDockPanelShell.test.tsx` | 85 | React component or page. |
| `ChatDockPanelShell.tsx` | 568 | React component or page. |
| `index.tsx` | 452 | React component or page. |

### `dashboard/src/components/CatalogTypeCard/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 59 | React component or page. |

### `dashboard/src/components/ChatPicker/`

| File | Lines | Description |
|------|------:|-------------|
| `SearchablePickerPanel.tsx` | 68 | Default export: SearchablePickerPanel. |

### `dashboard/src/components/ChromeTabBar/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 108 | React component or page. |

### `dashboard/src/components/CurrentVersionBadge/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 59 | Default export: CurrentVersionBadge. |

### `dashboard/src/components/DesktopWindowControls/`

| File | Lines | Description |
|------|------:|-------------|
| `DesktopWindowControls.test.tsx` | 44 | React component or page. |
| `DesktopWindowControls.tsx` | 92 | Default export: DesktopWindowControls. |
| `index.ts` | 1 | TS utilities, API, hooks, or types. |

### `dashboard/src/components/EmptyState/`

| File | Lines | Description |
|------|------:|-------------|
| `OctopEmptyMascot.tsx` | 28 | React component or page. |
| `index.tsx` | 104 | React component or page. |

### `dashboard/src/components/ErrorBoundary/`

| File | Lines | Description |
|------|------:|-------------|
| `index.test.tsx` | 59 | The boundary reads copy through the real i18n instance, which is not |
| `index.tsx` | 119 | React component or page. |

### `dashboard/src/components/Markdown/`

| File | Lines | Description |
|------|------:|-------------|
| `LazyMarkdown.tsx` | 20 | Default export: LazyMarkdown. |
| `index.tsx` | 447 | React component or page. |
| `mathPlugins.ts` | 83 | TS utilities, API, hooks, or types. |
| `mermaidLoader.ts` | 20 | TS utilities, API, hooks, or types. |
| `stabilizeStreamingMarkdown.test.ts` | 25 | TS utilities, API, hooks, or types. |
| `stabilizeStreamingMarkdown.ts` | 21 | Unclosed fenced code block (odd number of fence openers at line start). |
| `syntaxHighlight.tsx` | 221 | React component or page. |

### `dashboard/src/components/MarkdownCopy/`

| File | Lines | Description |
|------|------:|-------------|
| `MarkdownCopy.tsx` | 173 | React component or page. |

### `dashboard/src/components/OctopSpinner/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 19 | Default export: OctopSpinner. |

### `dashboard/src/components/PageLoading/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 11 | Default export: PageLoading. |

### `dashboard/src/components/PwaInstallPrompt/`

| File | Lines | Description |
|------|------:|-------------|
| `index.test.tsx` | 23 | React component or page. |
| `index.tsx` | 283 | Default export: PwaInstallPrompt. |

### `dashboard/src/components/PwaUpdatePrompt/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 65 | Default export: PwaUpdatePrompt. |

### `dashboard/src/components/ResizableTable/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 210 | antd injects the column config and index into the header cell; they are |

### `dashboard/src/components/Skeleton/`

| File | Lines | Description |
|------|------:|-------------|
| `CardSkeleton.tsx` | 30 | React component or page. |
| `ListSkeleton.tsx` | 32 | React component or page. |
| `TableSkeleton.tsx` | 64 | React component or page. |
| `index.ts` | 3 | TS utilities, API, hooks, or types. |

### `dashboard/src/components/StreamConnectingIndicator/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 48 | Default export: StreamConnectingIndicator. |

### `dashboard/src/components/StreamEdgeControls/`

| File | Lines | Description |
|------|------:|-------------|
| `StreamEdgeControls.tsx` | 77 | Default export: StreamEdgeControls. |

### `dashboard/src/components/StreamSetupGuide/`

| File | Lines | Description |
|------|------:|-------------|
| `StreamSetupGuide.tsx` | 102 | Default export: StreamSetupGuide. |

### `dashboard/src/components/TabLabel/`

| File | Lines | Description |
|------|------:|-------------|
| `TabBar.tsx` | 44 | Default export: TabBar. |
| `index.tsx` | 40 | Lucide icons are components (function or forwardRef object). |

### `dashboard/src/context/`

| File | Lines | Description |
|------|------:|-------------|
| `AgentContext.test.ts` | 130 | TS utilities, API, hooks, or types. |
| `AgentContext.tsx` | 309 | React component or page. |
| `BackupOperationContext.tsx` | 239 | React component or page. |
| `LayoutModeContext.tsx` | 65 | React component or page. |
| `ScrollContext.tsx` | 163 | React component or page. |
| `ServiceRestartContext.tsx` | 45 | React component or page. |
| `ThemeContext.tsx` | 193 | React component or page. |
| `VoiceOutputContext.tsx` | 29 | React component or page. |

### `dashboard/src/hooks/`

| File | Lines | Description |
|------|------:|-------------|
| `useAgentFormResources.ts` | 77 | TS utilities, API, hooks, or types. |
| `useAgentThreadChat.ts` | 89 | TS utilities, API, hooks, or types. |
| `useAsyncResource.ts` | 60 | TS utilities, API, hooks, or types. |
| `useAudioUnlock.ts` | 40 | TS utilities, API, hooks, or types. |
| `useAuthImageSrc.ts` | 171 | TS utilities, API, hooks, or types. |
| `useAutoViewportResize.ts` | 68 | TS utilities, API, hooks, or types. |
| `useBrowserCanvasInteraction.test.ts` | 93 | TS utilities, API, hooks, or types. |
| `useBrowserCanvasInteraction.ts` | 17 | TS utilities, API, hooks, or types. |
| `useBrowserSessionState.ts` | 284 | TS utilities, API, hooks, or types. |
| `useBrowserStream.ts` | 284 | TS utilities, API, hooks, or types. |
| `useBrowserViewController.ts` | 137 | TS utilities, API, hooks, or types. |
| `useCanvasRemotePointer.test.ts` | 285 | TS utilities, API, hooks, or types. |
| `useCanvasRemotePointer.ts` | 273 | TS utilities, API, hooks, or types. |
| `useCardTableView.ts` | 16 | TS utilities, API, hooks, or types. |
| `useCurrentUser.tsx` | 51 | React component or page. |
| `useDashboardPushToast.tsx` | 199 | React component or page. |
| `useDesktopCanvasInteraction.ts` | 77 | TS utilities, API, hooks, or types. |
| `useDesktopChrome.ts` | 64 | TS utilities, API, hooks, or types. |
| `useDesktopInstall.ts` | 74 | Module-level store so the install progress (and its SSE stream) persists |
| `useDesktopStream.ts` | 265 | TS utilities, API, hooks, or types. |
| `useElapsedSeconds.test.ts` | 53 | TS utilities, API, hooks, or types. |
| `useElapsedSeconds.ts` | 19 | TS utilities, API, hooks, or types. |
| `useEmbeddingDownloadWS.ts` | 81 | TS utilities, API, hooks, or types. |
| `useFilteredList.ts` | 16 | TS utilities, API, hooks, or types. |
| `useGatedSearchTabs.ts` | 82 | TS utilities, API, hooks, or types. |
| `useHorizontalResize.ts` | 161 | TS utilities, API, hooks, or types. |
| `useIsMobile.ts` | 22 | TS utilities, API, hooks, or types. |
| `useKeyboardOffset.ts` | 39 | TS utilities, API, hooks, or types. |
| `useLandscapeFullscreen.ts` | 64 | TS utilities, API, hooks, or types. |
| `useListPanelCollapsed.test.ts` | 33 | TS utilities, API, hooks, or types. |
| `useListPanelCollapsed.ts` | 39 | TS utilities, API, hooks, or types. |
| `useMobileStream.ts` | 161 | TS utilities, API, hooks, or types. |
| `usePathTabs.ts` | 133 | TS utilities, API, hooks, or types. |
| `usePointerDragSession.ts` | 73 | TS utilities, API, hooks, or types. |
| `useRemoteBrowserBookmarks.ts` | 115 | TS utilities, API, hooks, or types. |
| `useServerCapabilities.ts` | 37 | TS utilities, API, hooks, or types. |
| `useServerTimezone.ts` | 36 | TS utilities, API, hooks, or types. |
| `useServerUploadLimit.test.ts` | 21 | TS utilities, API, hooks, or types. |
| `useServerUploadLimit.ts` | 64 | TS utilities, API, hooks, or types. |
| `useServiceRestart.test.ts` | 14 | TS utilities, API, hooks, or types. |
| `useServiceRestart.ts` | 199 | TS utilities, API, hooks, or types. |
| `useSlashCommands.ts` | 46 | TS utilities, API, hooks, or types. |
| `useTerminalAutopilot.ts` | 253 | TS utilities, API, hooks, or types. |
| `useUnauthorizedRedirect.test.tsx` | 50 | React component or page. |
| `useUnauthorizedRedirect.ts` | 23 | TS utilities, API, hooks, or types. |
| `useUpdateStatus.test.ts` | 143 | TS utilities, API, hooks, or types. |
| `useUpdateStatus.ts` | 83 | TS utilities, API, hooks, or types. |
| `useUserRole.ts` | 11 | TS utilities, API, hooks, or types. |
| `useViewportMode.ts` | 90 | TS utilities, API, hooks, or types. |
| `useVoiceConfig.ts` | 54 | TS utilities, API, hooks, or types. |
| `useVoiceInput.ts` | 355 | TS utilities, API, hooks, or types. |
| `useVoiceOutput.ts` | 253 | TS utilities, API, hooks, or types. |

### `dashboard/src/layouts/`

| File | Lines | Description |
|------|------:|-------------|
| `Header.tsx` | 109 | Default export: Header. |
| `MinimalRecordsHost.tsx` | 142 | Default export: MinimalRecordsHost. |
| `PageShell.tsx` | 239 | React component or page. |
| `Sidebar.tsx` | 756 | Default export: Sidebar. |
| `SidebarCollapsedIconNav.tsx` | 116 | Default export: SidebarCollapsedIconNav. |
| `SidebarMinimalPaneToggle.tsx` | 59 | Default export: SidebarMinimalPaneToggle. |
| `chatHistoryRail.ts` | 10 | TS utilities, API, hooks, or types. |
| `layoutModeStorage.test.ts` | 42 | TS utilities, API, hooks, or types. |
| `layoutModeStorage.ts` | 47 | TS utilities, API, hooks, or types. |
| `sidebarNav.test.ts` | 45 | TS utilities, API, hooks, or types. |
| `sidebarNav.tsx` | 240 | React component or page. |

### `dashboard/src/layouts/MainLayout/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 345 | Default export: MainLayout. |

### `dashboard/src/pages/Admin/`

| File | Lines | Description |
|------|------:|-------------|
| `Placeholder.tsx` | 28 | Default export: AdminPlaceholder. |

### `dashboard/src/pages/Admin/Plugins/`

| File | Lines | Description |
|------|------:|-------------|
| `InstalledPluginsPanel.tsx` | 623 | React component or page. |
| `PluginIconView.tsx` | 80 | React component or page. |
| `PluginMarketPanel.tsx` | 15 | React component or page. |
| `index.tsx` | 66 | Legacy ?tab=agent-tools redirects to installed (tools live in plugin detail). |

### `dashboard/src/pages/Admin/SharedModels/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 111 | Default export: AdminSharedModelsPage. |

### `dashboard/src/pages/Admin/Storage/`

| File | Lines | Description |
|------|------:|-------------|
| `DockerEnvFooter.tsx` | 241 | React component or page. |
| `StorageBackendCard.tsx` | 348 | React component or page. |
| `StorageBackendModal.tsx` | 646 | React component or page. |
| `StorageBrowseDrawer.tsx` | 198 | React component or page. |
| `StorageTypeCard.tsx` | 57 | React component or page. |
| `index.tsx` | 199 | Default export: AdminStoragePage. |
| `useStorageBackends.tsx` | 475 | React component or page. |

### `dashboard/src/pages/Admin/Users/`

| File | Lines | Description |
|------|------:|-------------|
| `InviteDrawer.tsx` | 283 | Default export: InviteDrawer. |
| `OauthProviderCard.tsx` | 505 | Default export: OauthProviderCard. |
| `SsoAppProviderShell.tsx` | 222 | Default export: SsoAppProviderShell. |
| `SsoPanel.test.tsx` | 87 | React component or page. |
| `SsoPanel.tsx` | 695 | Default export: SsoPanel. |
| `SsoProviderCard.tsx` | 52 | Default export: SsoProviderCard. |
| `UsersListPanel.tsx` | 1977 | Default export: UsersListPanel. |
| `index.tsx` | 139 | Default export: AdminUsersPage. |
| `oauthProviders.ts` | 63 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Agent/ACP/`

| File | Lines | Description |
|------|------:|-------------|
| `constants.ts` | 67 | TS utilities, API, hooks, or types. |
| `index.tsx` | 341 | Default export: ACPPage. |

### `dashboard/src/pages/Agent/ACP/components/`

| File | Lines | Description |
|------|------:|-------------|
| `ACPCard.tsx` | 114 | React component or page. |
| `ACPDrawer.tsx` | 171 | React component or page. |

### `dashboard/src/pages/Agent/Channels/`

| File | Lines | Description |
|------|------:|-------------|
| `ChannelsPanel.test.tsx` | 116 | React component or page. |
| `ChannelsPanel.tsx` | 492 | Default export: ChannelsPanel. |
| `index.tsx` | 6 | React component or page. |
| `useChannels.ts` | 217 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Agent/Channels/components/`

| File | Lines | Description |
|------|------:|-------------|
| `ChannelCard.tsx` | 160 | React component or page. |
| `ChannelDrawer.tsx` | 1890 | React component or page. |
| `constants.test.ts` | 97 | TS utilities, API, hooks, or types. |
| `constants.ts` | 450 | TS utilities, API, hooks, or types. |
| `index.ts` | 32 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Agent/Config/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 105 | React component or page. |

### `dashboard/src/pages/Agent/Connectors/`

| File | Lines | Description |
|------|------:|-------------|
| `ConnectorCard.test.tsx` | 44 | React component or page. |
| `ConnectorCard.tsx` | 70 | React component or page. |
| `ConnectorInstanceCard.test.tsx` | 87 | React component or page. |
| `ConnectorInstanceCard.tsx` | 158 | React component or page. |
| `CustomMcpServerCard.tsx` | 474 | React component or page. |
| `CustomMcpTab.tsx` | 791 | React component or page. |
| `connectorDefs.tsx` | 90 | React component or page. |
| `customMcpUtils.ts` | 262 | TS utilities, API, hooks, or types. |
| `guidedConnectorUtils.test.ts` | 40 | TS utilities, API, hooks, or types. |
| `guidedConnectorUtils.ts` | 45 | TS utilities, API, hooks, or types. |
| `index.tsx` | 2340 | Default export: ConnectorsPage. |
| `useConnectors.ts` | 36 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Agent/Memory/`

| File | Lines | Description |
|------|------:|-------------|
| `AtomsList.test.tsx` | 181 | React component or page. |
| `AtomsList.tsx` | 420 | Default export: AtomsList. |
| `CandidatesReview.test.tsx` | 167 | React component or page. |
| `CandidatesReview.tsx` | 521 | Default export: CandidatesReview. |
| `ConversationRecords.tsx` | 654 | Default export: ConversationRecords. |
| `EpisodesList.test.tsx` | 92 | React component or page. |
| `EpisodesList.tsx` | 314 | Default export: EpisodesList. |
| `ExtractTriggerConfig.test.tsx` | 137 | React component or page. |
| `ExtractTriggerConfig.tsx` | 2 | React component or page. |
| `JournalList.test.tsx` | 124 | React component or page. |
| `JournalList.tsx` | 680 | Default export: JournalList. |
| `MemoryPanel.tsx` | 299 | Default export: MemoryPanel. |
| `MemorySettings.tsx` | 355 | Default export: MemorySettings. |
| `MemoryTree.test.tsx` | 94 | React component or page. |
| `MemoryTree.tsx` | 1039 | Default export: MemoryTree. |
| `MigrateMemory.tsx` | 589 | Default export: MigrateMemory. |
| `Overview.test.tsx` | 134 | React component or page. |
| `Overview.tsx` | 527 | Default export: Overview. |
| `ProactiveConfig.tsx` | 355 | Default export: ProactiveConfig. |
| `ProfileOverview.test.tsx` | 163 | React component or page. |
| `ProfileOverview.tsx` | 543 | Default export: ProfileOverview. |
| `RawEventsList.test.tsx` | 150 | React component or page. |
| `RawEventsList.tsx` | 204 | Default export: RawEventsList. |
| `VectorSearchConfig.tsx` | 628 | Downloaded model list, persisted to localStorage so service restarts do not lose state. |

### `dashboard/src/pages/Agent/Memory/shared/`

| File | Lines | Description |
|------|------:|-------------|
| `LineageStrip.tsx` | 177 | Default export: LineageStrip. |
| `MemoryLayerView.tsx` | 150 | Default export: MemoryLayerView. |
| `MemoryPipelineEmpty.test.tsx` | 81 | React component or page. |
| `MemoryPipelineEmpty.tsx` | 106 | Default export: MemoryPipelineEmpty. |
| `createAtom.tsx` | 217 | Default export: CreateAtomModal. |
| `deprecateAtom.tsx` | 64 | React component or page. |
| `editAtom.tsx` | 64 | React component or page. |

### `dashboard/src/pages/Agent/Personalization/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 221 | Default export: PersonalizationPage. |

### `dashboard/src/pages/Agent/Personalization/components/`

| File | Lines | Description |
|------|------:|-------------|
| `AgentPluginsPanel.tsx` | 369 | Default export: AgentPluginsPanel. |
| `EditDrawer.tsx` | 149 | Default export: EditDrawer. |
| `MBTISelector.tsx` | 629 | Default export: MBTISelector. |
| `MBTITest.tsx` | 395 | Default export: MBTITest. |

### `dashboard/src/pages/Agent/Personalization/hooks/`

| File | Lines | Description |
|------|------:|-------------|
| `useFileEditor.ts` | 128 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Agent/Skills/`

| File | Lines | Description |
|------|------:|-------------|
| `skillDisplayNames.test.ts` | 45 | TS utilities, API, hooks, or types. |
| `skillDisplayNames.ts` | 91 | TS utilities, API, hooks, or types. |
| `skillMarkdown.test.ts` | 49 | TS utilities, API, hooks, or types. |
| `skillMarkdown.ts` | 53 | TS utilities, API, hooks, or types. |
| `useSkills.ts` | 361 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Agent/Skills/components/`

| File | Lines | Description |
|------|------:|-------------|
| `AgentPickerModal.tsx` | 70 | dashboard/src/pages/Agent/Skills/components/AgentPickerModal.tsx |
| `InstalledSkillsTab.tsx` | 300 | Default export: InstalledSkillsTab. |
| `PushSkillToPackageModal.tsx` | 116 | React component or page. |
| `SectionHeader.tsx` | 35 | dashboard/src/pages/Agent/Skills/components/SectionHeader.tsx |
| `SkillCard.tsx` | 310 | React component or page. |
| `SkillDrawer.test.ts` | 69 | TS utilities, API, hooks, or types. |
| `SkillDrawer.tsx` | 935 | React component or page. |
| `SkillFileTree.tsx` | 214 | React component or page. |
| `SkillHubDetailDrawer.tsx` | 309 | dashboard/src/pages/Agent/Skills/components/SkillHubDetailDrawer.tsx |
| `SkillHubTab.tsx` | 495 | dashboard/src/pages/Agent/Skills/components/SkillHubTab.tsx |
| `SkillImportModal.test.tsx` | 197 | React component or page. |
| `SkillImportModal.tsx` | 376 | React component or page. |
| `SkillPackagesTab.tsx` | 486 | Default export: SkillPackagesTab. |
| `SkillsTable.tsx` | 129 | Default export: SkillsTable. |
| `SkillsTabs.tsx` | 97 | Default export: SkillsTabs. |
| `index.ts` | 3 | TS utilities, API, hooks, or types. |
| `parseSkillZip.test.ts` | 293 | TS utilities, API, hooks, or types. |
| `parseSkillZip.ts` | 222 | TS utilities, API, hooks, or types. |
| `skillHubCache.ts` | 33 | TS utilities, API, hooks, or types. |
| `skillInstallTarget.ts` | 38 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Agent/Tools/`

| File | Lines | Description |
|------|------:|-------------|
| `ToolsPanel.tsx` | 362 | Default export: ToolsPanel. |
| `ToolsTabs.tsx` | 60 | Default export: ToolsTabs. |

### `dashboard/src/pages/Agent/Workspace/components/`

| File | Lines | Description |
|------|------:|-------------|
| `CodeEditor.tsx` | 123 | Default export: CodeEditor. |
| `DocumentPreview.tsx` | 79 | Default export: DocumentPreview. |
| `FilePreview.tsx` | 132 | Default export: FilePreview. |
| `FileViewer.tsx` | 138 | Default export: FileViewer. |
| `MediaPreview.tsx` | 315 | Default export: MediaPreview. |
| `WorkspaceDrawer.tsx` | 1556 | Default export: WorkspaceDrawer. |

### `dashboard/src/pages/Agent/Workspace/utils/`

| File | Lines | Description |
|------|------:|-------------|
| `docKind.test.ts` | 41 | TS utilities, API, hooks, or types. |
| `docKind.ts` | 7 | TS utilities, API, hooks, or types. |
| `editorLanguage.ts` | 60 | TS utilities, API, hooks, or types. |
| `fileKind.ts` | 91 | TS utilities, API, hooks, or types. |
| `mediaKind.ts` | 44 | TS utilities, API, hooks, or types. |
| `workspaceMove.test.ts` | 66 | TS utilities, API, hooks, or types. |
| `workspaceMove.ts` | 40 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Chat/`

| File | Lines | Description |
|------|------:|-------------|
| `ChatAgentProfileContext.tsx` | 36 | React component or page. |
| `ChatFilePreviewContext.tsx` | 34 | React component or page. |
| `ChatToolDockContext.tsx` | 70 | React component or page. |
| `constants.ts` | 33 | TS utilities, API, hooks, or types. |
| `index.tsx` | 1478 | Default export: ChatPage. |

### `dashboard/src/pages/Chat/Weather/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 240 | Default export: Weather. |

### `dashboard/src/pages/Chat/components/`

| File | Lines | Description |
|------|------:|-------------|
| `AgentNotReadyScreen.tsx` | 114 | Default export: AgentNotReadyScreen. |
| `AskQuestionCard.test.tsx` | 114 | React component or page. |
| `AskQuestionCard.tsx` | 337 | React component or page. |
| `AssistantProcessSummary.tsx` | 120 | React component or page. |
| `AssistantTurnView.tsx` | 310 | Default export: AssistantTurnView. |
| `ChatComposerChrome.tsx` | 22 | Default export: ChatComposerChrome. |
| `ChatDockFileList.tsx` | 313 | Default export: ChatDockFileList. |
| `ChatDockPanel.tsx` | 468 | React component or page. |
| `ChatDockPanels.keepAlive.test.tsx` | 61 | React component or page. |
| `ChatDockPanels.tsx` | 134 | Default export: ChatDockPanels. |
| `ChatDockToolUiContent.tsx` | 45 | Default export: ChatDockToolUiContent. |
| `ChatInput.prefill.test.tsx` | 239 | React component or page. |
| `ChatInput.tsx` | 890 | React component or page. |
| `ChatInputActionsRow.test.tsx` | 62 | React component or page. |
| `ChatInputActionsRow.tsx` | 1302 | Default export: ChatInputActionsRow. |
| `ChatInputPreviewBar.tsx` | 269 | Default export: ChatInputPreviewBar. |
| `ChatMediaPlayer.tsx` | 166 | React component or page. |
| `ChatQueuedMessages.tsx` | 82 | Default export: ChatQueuedMessages. |
| `ChatSidebarPanel.tsx` | 186 | Default export: ChatSidebarPanel. |
| `ChatTitleBar.test.tsx` | 51 | React component or page. |
| `ChatTitleBar.tsx` | 182 | Default export: ChatTitleBar. |
| `ConnectorPickerPopover.tsx` | 94 | Default export: ConnectorPickerPopover. |
| `ContextChip.tsx` | 56 | Default export: ContextChip. |
| `ContextWindowRing.tsx` | 360 | Default export: ContextWindowRing. |
| `ExpertAgentAvatar.tsx` | 57 | Default export: ExpertAgentAvatar. |
| `ExpertPickerPopover.tsx` | 80 | Default export: ExpertPickerPopover. |
| `FilePanelContent.tsx` | 375 | Default export: FilePanelContent. |
| `GeneratingIndicator.tsx` | 55 | Default export: GeneratingIndicator. |
| `HistoryMigrationBanner.test.tsx` | 52 | React component or page. |
| `HistoryMigrationBanner.tsx` | 72 | Default export: HistoryMigrationBanner. |
| `HitlApprovalCard.test.tsx` | 58 | React component or page. |
| `HitlApprovalCard.tsx` | 104 | Default export: HitlApprovalCard. |
| `KnowledgeCitationPanelContent.tsx` | 374 | Default export: KnowledgeCitationPanelContent. |
| `KnowledgeCitationsStrip.tsx` | 100 | React component or page. |
| `KnowledgePickerPopover.tsx` | 97 | Default export: KnowledgePickerPopover. |
| `MemoryMaintenanceBanner.tsx` | 64 | Default export: MemoryMaintenanceBanner. |
| `MentionPickerMenu.test.ts` | 60 | TS utilities, API, hooks, or types. |
| `MentionPickerMenu.tsx` | 237 | Default export: MentionPickerMenu. |
| `MessageBubble.tsx` | 1055 | React component or page. |
| `MessageFileCard.tsx` | 117 | React component or page. |
| `MessageList.tsx` | 855 | Default export: MessageList. |
| `MessageSender.test.tsx` | 35 | React component or page. |
| `MessageSender.tsx` | 77 | Default export: MessageSender. |
| `MinimalAgentSessionNav.tsx` | 628 | Default export: MinimalAgentSessionNav. |
| `ScrollToBottomButton.tsx` | 90 | Default export: ScrollToBottomButton. |
| `SessionChannelIcon.tsx` | 71 | Default export: SessionChannelIcon. |
| `SessionList.tsx` | 522 | Default export: SessionList. |
| `SharedExpertHint.tsx` | 25 | Default export: SharedExpertHint. |
| `SkillPickerPopover.tsx` | 129 | Default export: SkillPickerPopover. |
| `SlashCommandMenu.tsx` | 124 | Default export: SlashCommandMenu. |
| `SubagentPickerPopover.tsx` | 77 | Default export: SubagentPickerPopover. |
| `ThinkingBubble.tsx` | 46 | Default export: ThinkingBubble. |
| `ToolMediaStrip.tsx` | 155 | React component or page. |
| `TrajectoryDrawer.test.tsx` | 348 | React component or page. |
| `TrajectoryDrawer.tsx` | 329 | Default export: TrajectoryDrawer. |
| `TrajectoryInspector.test.tsx` | 306 | React component or page. |
| `TrajectoryInspector.tsx` | 442 | Default export: TrajectoryInspector. |
| `TrajectoryLedger.test.tsx` | 295 | React component or page. |
| `TrajectoryLedger.tsx` | 357 | Default export: TrajectoryLedger. |
| `TrajectoryMetricsBar.test.tsx` | 49 | React component or page. |
| `TrajectoryMetricsBar.tsx` | 126 | Default export: TrajectoryMetricsBar. |
| `TrajectoryTimeline.test.tsx` | 254 | React component or page. |
| `TrajectoryTimeline.tsx` | 697 | Default export: TrajectoryTimeline. |
| `TrajectoryToolbar.test.tsx` | 76 | React component or page. |
| `TrajectoryToolbar.tsx` | 123 | Default export: TrajectoryToolbar. |
| `TurnProcessBlocks.tsx` | 106 | React component or page. |
| `TurnTimelineRail.test.tsx` | 260 | React component or page. |
| `TurnTimelineRail.tsx` | 426 | Default export: TurnTimelineRail. |
| `UserMessageComposerTags.tsx` | 148 | Default export: UserMessageComposerTags. |
| `WelcomeQuickCards.tsx` | 105 | Default export: WelcomeQuickCards. |
| `WelcomeScreen.tsx` | 118 | Animated WebP keeps alpha on Safari; VP9 WebM alpha is unreliable there. |
| `generatingGate.test.ts` | 51 | TS utilities, API, hooks, or types. |
| `generatingGate.ts` | 26 | TS utilities, API, hooks, or types. |
| `liveAssistantTurn.test.ts` | 49 | groups: … assistant(1), user(2) — last assistant is still 1 |
| `liveAssistantTurn.ts` | 26 | TS utilities, API, hooks, or types. |
| `loadOlderGate.test.ts` | 121 | TS utilities, API, hooks, or types. |
| `loadOlderGate.ts` | 37 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Chat/hooks/`

| File | Lines | Description |
|------|------:|-------------|
| `chatSidebarDefaultOpen.test.ts` | 40 | TS utilities, API, hooks, or types. |
| `chatStore.clearMessages.test.ts` | 42 | TS utilities, API, hooks, or types. |
| `chatStore.history.test.ts` | 95 | TS utilities, API, hooks, or types. |
| `chatStore.hitlResume.test.ts` | 46 | TS utilities, API, hooks, or types. |
| `chatStore.pendingAlias.test.ts` | 69 | TS utilities, API, hooks, or types. |
| `chatStore.thinkingTimer.test.ts` | 26 | TS utilities, API, hooks, or types. |
| `chatStore.ts` | 2253 | TS utilities, API, hooks, or types. |
| `frameThread.test.ts` | 20 | TS utilities, API, hooks, or types. |
| `frameThread.ts` | 15 | TS utilities, API, hooks, or types. |
| `scrollFreeMode.test.ts` | 66 | TS utilities, API, hooks, or types. |
| `scrollFreeMode.ts` | 31 | TS utilities, API, hooks, or types. |
| `sealPriorStreamingAssistants.test.ts` | 59 | TS utilities, API, hooks, or types. |
| `sealPriorStreamingAssistants.ts` | 22 | TS utilities, API, hooks, or types. |
| `sseHelpers.ts` | 122 | TS utilities, API, hooks, or types. |
| `toolDisplayNames.ts` | 27 | TS utilities, API, hooks, or types. |
| `turnStatusGate.test.ts` | 12 | TS utilities, API, hooks, or types. |
| `turnStatusGate.ts` | 9 | TS utilities, API, hooks, or types. |
| `useAutoScroll.test.ts` | 706 | TS utilities, API, hooks, or types. |
| `useAutoScroll.ts` | 567 | ─── constants ──────────────────────────────────────────────────────────────── |
| `useBrowserToolDetection.ts` | 33 | TS utilities, API, hooks, or types. |
| `useChat.history.test.ts` | 127 | TS utilities, API, hooks, or types. |
| `useChat.ts` | 1160 | TS utilities, API, hooks, or types. |
| `useChat.usage.test.ts` | 97 | TS utilities, API, hooks, or types. |
| `useChatAttachments.ts` | 172 | TS utilities, API, hooks, or types. |
| `useChatComposerResources.ts` | 412 | TS utilities, API, hooks, or types. |
| `useChatContextWindow.test.ts` | 105 | TS utilities, API, hooks, or types. |
| `useChatContextWindow.ts` | 113 | TS utilities, API, hooks, or types. |
| `useChatDockPanel.test.ts` | 264 | TS utilities, API, hooks, or types. |
| `useChatDockPanel.ts` | 357 | TS utilities, API, hooks, or types. |
| `useChatFileDetection.test.ts` | 47 | TS utilities, API, hooks, or types. |
| `useChatFileDetection.ts` | 186 | TS utilities, API, hooks, or types. |
| `useChatHistoryRail.ts` | 31 | Layout mode / pane / route swaps remount or clear the rail after paint. |
| `useChatMessageQueue.test.ts` | 363 | TS utilities, API, hooks, or types. |
| `useChatMessageQueue.ts` | 317 | TS utilities, API, hooks, or types. |
| `useChatNavigation.test.tsx` | 304 | React component or page. |
| `useChatNavigation.ts` | 229 | TS utilities, API, hooks, or types. |
| `useChatSend.ts` | 308 | TS utilities, API, hooks, or types. |
| `useChatSessionActions.test.tsx` | 80 | React component or page. |
| `useChatSessionActions.ts` | 168 | TS utilities, API, hooks, or types. |
| `useChatSidebarState.ts` | 179 | TS utilities, API, hooks, or types. |
| `useChatSubagents.ts` | 32 | TS utilities, API, hooks, or types. |
| `useDragOffset.ts` | 250 | TS utilities, API, hooks, or types. |
| `useExpertQuickCards.ts` | 70 | TS utilities, API, hooks, or types. |
| `useHistoryMigration.ts` | 64 | TS utilities, API, hooks, or types. |
| `useMemoryMaintenance.ts` | 66 | TS utilities, API, hooks, or types. |
| `usePanelResize.ts` | 121 | TS utilities, API, hooks, or types. |
| `useSessions.test.ts` | 169 | TS utilities, API, hooks, or types. |
| `useSessions.ts` | 560 | TS utilities, API, hooks, or types. |
| `useSkillRecordingWorkflow.ts` | 241 | TS utilities, API, hooks, or types. |
| `useSlashMentionInput.test.ts` | 144 | TS utilities, API, hooks, or types. |
| `useSlashMentionInput.ts` | 531 | TS utilities, API, hooks, or types. |
| `useToolMessageByCallId.ts` | 24 | TS utilities, API, hooks, or types. |
| `useToolUiDockButtonStyle.ts` | 73 | TS utilities, API, hooks, or types. |
| `useTrajectorySession.test.ts` | 445 | TS utilities, API, hooks, or types. |
| `useTrajectorySession.ts` | 212 | TS utilities, API, hooks, or types. |
| `useWelcomeQuickCardsLayout.ts` | 127 | TS utilities, API, hooks, or types. |
| `useWorkspaceFileMention.test.ts` | 59 | TS utilities, API, hooks, or types. |
| `useWorkspaceFileMention.ts` | 75 | TS utilities, API, hooks, or types. |
| `wsResumeGate.test.ts` | 203 | TS utilities, API, hooks, or types. |
| `wsResumeGate.ts` | 99 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Chat/utils/`

| File | Lines | Description |
|------|------:|-------------|
| `accountDisplayName.test.ts` | 35 | TS utilities, API, hooks, or types. |
| `accountDisplayName.ts` | 14 | TS utilities, API, hooks, or types. |
| `chatAttachments.test.ts` | 57 | TS utilities, API, hooks, or types. |
| `chatAttachments.ts` | 115 | TS utilities, API, hooks, or types. |
| `chatMessages.test.ts` | 70 | TS utilities, API, hooks, or types. |
| `chatMessages.ts` | 199 | TS utilities, API, hooks, or types. |
| `chatStorage.ts` | 40 | TS utilities, API, hooks, or types. |
| `chromeInstallGate.test.ts` | 20 | TS utilities, API, hooks, or types. |
| `chromeInstallGate.ts` | 9 | TS utilities, API, hooks, or types. |
| `dockFilePath.test.ts` | 273 | TS utilities, API, hooks, or types. |
| `dockFilePath.ts` | 281 | TS utilities, API, hooks, or types. |
| `dockKnowledgeTabId.ts` | 4 | TS utilities, API, hooks, or types. |
| `dockToolUiTabId.ts` | 4 | TS utilities, API, hooks, or types. |
| `expertMention.test.ts` | 92 | TS utilities, API, hooks, or types. |
| `expertMention.ts` | 112 | TS utilities, API, hooks, or types. |
| `fileMention.test.ts` | 128 | TS utilities, API, hooks, or types. |
| `fileMention.ts` | 162 | TS utilities, API, hooks, or types. |
| `layoutAssistantTurnHitl.test.ts` | 115 | TS utilities, API, hooks, or types. |
| `layoutAssistantTurnHitl.ts` | 33 | TS utilities, API, hooks, or types. |
| `mentionAtCursor.ts` | 9 | TS utilities, API, hooks, or types. |
| `messageContent.ts` | 170 | TS utilities, API, hooks, or types. |
| `messageGrouping.test.ts` | 51 | TS utilities, API, hooks, or types. |
| `messageGrouping.ts` | 43 | TS utilities, API, hooks, or types. |
| `pendingAttachKnowledgeBase.test.ts` | 16 | TS utilities, API, hooks, or types. |
| `pendingAttachKnowledgeBase.ts` | 19 | TS utilities, API, hooks, or types. |
| `pendingHitl.test.ts` | 99 | TS utilities, API, hooks, or types. |
| `pendingHitl.ts` | 41 | TS utilities, API, hooks, or types. |
| `resolveInitialConnectors.test.ts` | 93 | TS utilities, API, hooks, or types. |
| `resolveInitialConnectors.ts` | 33 | TS utilities, API, hooks, or types. |
| `skillChipIcon.tsx` | 18 | React component or page. |
| `skillSlash.test.ts` | 154 | TS utilities, API, hooks, or types. |
| `skillSlash.ts` | 197 | TS utilities, API, hooks, or types. |
| `slashShortcutStyles.ts` | 11 | TS utilities, API, hooks, or types. |
| `slashText.ts` | 16 | TS utilities, API, hooks, or types. |
| `summarizeHitlAction.test.ts` | 137 | TS utilities, API, hooks, or types. |
| `summarizeHitlAction.ts` | 270 | TS utilities, API, hooks, or types. |
| `threadTitle.test.ts` | 58 | TS utilities, API, hooks, or types. |
| `threadTitle.ts` | 34 | TS utilities, API, hooks, or types. |
| `trajectoryModel.test.ts` | 370 | TS utilities, API, hooks, or types. |
| `trajectoryModel.ts` | 576 | TS utilities, API, hooks, or types. |
| `trajectoryTimeline.test.ts` | 316 | TS utilities, API, hooks, or types. |
| `trajectoryTimeline.ts` | 231 | TS utilities, API, hooks, or types. |
| `turnTimeline.test.ts` | 187 | TS utilities, API, hooks, or types. |
| `turnTimeline.ts` | 330 | TS utilities, API, hooks, or types. |
| `withDefaultOpenConnectors.test.ts` | 21 | TS utilities, API, hooks, or types. |
| `withDefaultOpenConnectors.ts` | 11 | TS utilities, API, hooks, or types. |
| `withDefaultOpenKnowledgeBases.test.ts` | 20 | TS utilities, API, hooks, or types. |
| `withDefaultOpenKnowledgeBases.ts` | 11 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Control/CronJobs/`

| File | Lines | Description |
|------|------:|-------------|
| `constants.ts` | 4 | TS utilities, API, hooks, or types. |
| `cronDisplay.ts` | 41 | TS utilities, API, hooks, or types. |
| `index.tsx` | 404 | React component or page. |
| `taskExamples.test.ts` | 78 | TS utilities, API, hooks, or types. |
| `taskExamples.ts` | 33 | TS utilities, API, hooks, or types. |
| `useCronJobs.ts` | 475 | TS utilities, API, hooks, or types. |
| `useTaskExamples.ts` | 60 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Control/CronJobs/components/`

| File | Lines | Description |
|------|------:|-------------|
| `CronJobCard.tsx` | 239 | React component or page. |
| `ExecuteNowModal.tsx` | 62 | React component or page. |
| `JobDetailDrawer.tsx` | 249 | React component or page. |
| `JobDrawer.tsx` | 492 | React component or page. |
| `columns.tsx` | 327 | React component or page. |
| `constants.ts` | 138 | TS utilities, API, hooks, or types. |
| `index.ts` | 7 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Control/RemoteAndroid/`

| File | Lines | Description |
|------|------:|-------------|
| `AdbShellPanel.tsx` | 103 | Default export: AdbShellPanel. |
| `RemotePhoneIdleGuide.tsx` | 212 | Default export: RemotePhoneIdleGuide. |
| `index.tsx` | 1562 | Default export: RemoteAndroidPage. |
| `useAdbShell.ts` | 108 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Control/RemoteBrowser/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 1607 | Default export: RemoteBrowserPage. |

### `dashboard/src/pages/Control/RemoteDesktop/`

| File | Lines | Description |
|------|------:|-------------|
| `DesktopPanel.tsx` | 1343 | Default export: DesktopPanel. |
| `desktopShortcuts.ts` | 14 | TS utilities, API, hooks, or types. |
| `index.tsx` | 103 | Default export: RemoteDesktopPage. |

### `dashboard/src/pages/Control/Terminal/`

| File | Lines | Description |
|------|------:|-------------|
| `ensureDefaultSession.test.ts` | 31 | TS utilities, API, hooks, or types. |
| `index.tsx` | 703 | Default export: TerminalPage. |
| `terminalContext.test.ts` | 73 | TS utilities, API, hooks, or types. |
| `terminalContext.ts` | 66 | TS utilities, API, hooks, or types. |
| `terminalOutputBuffer.ts` | 106 | TS utilities, API, hooks, or types. |
| `terminalThemes.ts` | 298 | ─── Dark Themes ────────────────────────────────────────────────────────────── |
| `useTerminal.testUtils.ts` | 5 | TS utilities, API, hooks, or types. |
| `useTerminal.ts` | 697 | ─── Types ──────────────────────────────────────────────────────────────────── |

### `dashboard/src/pages/Control/Terminal/components/`

| File | Lines | Description |
|------|------:|-------------|
| `AiPanel.tsx` | 489 | Default export: AiPanel. |
| `AutopilotPlan.tsx` | 116 | Default export: AutopilotPlan. |
| `TerminalView.tsx` | 205 | Default export: TerminalView. |

### `dashboard/src/pages/Control/TokenUsage/`

| File | Lines | Description |
|------|------:|-------------|
| `UsageStats.test.tsx` | 85 | React component or page. |
| `UsageStats.tsx` | 142 | React component or page. |
| `index.tsx` | 1131 | Default export: TokenUsagePage. |

### `dashboard/src/pages/Control/Workbench/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 103 | Default export: WorkbenchPage. |

### `dashboard/src/pages/Experts/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 575 | Default export: ExpertsPage. |

### `dashboard/src/pages/Experts/components/`

| File | Lines | Description |
|------|------:|-------------|
| `AgentBackendFields.tsx` | 299 | Default export: AgentBackendFields. |
| `AgentCard.tsx` | 566 | dashboard/src/pages/Experts/components/AgentCard.tsx |
| `AgentExpertsTable.tsx` | 633 | dashboard/src/pages/Experts/components/AgentExpertsTable.tsx |
| `AgentMoreActions.tsx` | 97 | Default export: AgentMoreActions. |
| `AgentTrajectoryField.tsx` | 21 | Default export: AgentTrajectoryField. |
| `CatalogDrawer.tsx` | 58 | Default export: CatalogDrawer. |
| `ChannelCatalogDrawer.tsx` | 36 | Default export: ChannelCatalogDrawer. |
| `CreateFromExpertDrawer.tsx` | 912 | dashboard/src/pages/Experts/components/CreateFromExpertDrawer.tsx |
| `EditAgentDrawer.tsx` | 1339 | dashboard/src/pages/Experts/components/EditAgentDrawer.tsx |
| `ExpertAvatarPicker.tsx` | 163 | Default export: ExpertAvatarPicker. |
| `ExpertCard.tsx` | 94 | dashboard/src/pages/Experts/components/ExpertCard.tsx |
| `ExpertComposerDefaultsFields.tsx` | 115 | Default export: ExpertComposerDefaultsFields. |
| `ExpertMarketTab.tsx` | 515 | Default export: ExpertMarketTab. |
| `FileEditModal.tsx` | 179 | dashboard/src/pages/Experts/components/FileEditModal.tsx |
| `MbtiCatalogDrawer.tsx` | 40 | Default export: MbtiCatalogDrawer. |
| `MemoryCatalogDrawer.tsx` | 41 | dashboard/src/pages/Experts/components/MemoryCatalogDrawer.tsx |
| `PluginCatalogDrawer.tsx` | 29 | dashboard/src/pages/Experts/components/PluginCatalogDrawer.tsx |
| `PublishExpertDrawer.tsx` | 228 | Default export: PublishExpertDrawer. |
| `PublishTemplateButton.tsx` | 131 | Default export: PublishTemplateButton. |
| `PublishedExpertCard.tsx` | 121 | React component or page. |
| `RootDirSelect.expand.test.tsx` | 154 | React component or page. |
| `RootDirSelect.test.tsx` | 295 | React component or page. |
| `RootDirSelect.tsx` | 444 | Default export: RootDirSelect. |
| `SkillCatalogDrawer.tsx` | 29 | dashboard/src/pages/Experts/components/SkillCatalogDrawer.tsx |
| `SubagentCatalogDrawer.test.tsx` | 67 | React component or page. |
| `SubagentCatalogDrawer.tsx` | 56 | Default export: SubagentCatalogDrawer. |
| `SubagentDrawer.test.ts` | 34 | TS utilities, API, hooks, or types. |
| `SubagentDrawer.tsx` | 389 | React component or page. |
| `SubagentManager.tsx` | 668 | Default export: SubagentManager. |
| `SubagentPreviewDrawer.tsx` | 85 | Default export: SubagentPreviewDrawer. |
| `ToolCatalogDrawer.tsx` | 29 | dashboard/src/pages/Experts/components/ToolCatalogDrawer.tsx |
| `WelcomeConfig.tsx` | 383 | React component or page. |
| `agentBackendForm.ensureBwrap.test.ts` | 19 | TS utilities, API, hooks, or types. |
| `agentBackendForm.skillPackages.test.ts` | 88 | TS utilities, API, hooks, or types. |
| `agentBackendForm.ts` | 331 | TS utilities, API, hooks, or types. |
| `expertFileGroups.test.ts` | 42 | TS utilities, API, hooks, or types. |
| `expertFileGroups.ts` | 134 | TS utilities, API, hooks, or types. |
| `iconForName.tsx` | 251 | React component or page. |
| `rootDirTree.test.ts` | 230 | TS utilities, API, hooks, or types. |
| `rootDirTree.ts` | 208 | Keep Windows drive roots like ``C:/`` as a single segment with trailing slash. |
| `sharedExpert.test.ts` | 59 | TS utilities, API, hooks, or types. |
| `welcomeManifest.test.ts` | 108 | TS utilities, API, hooks, or types. |
| `welcomeManifest.ts` | 76 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Invite/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 338 | Default export: InvitePage. |

### `dashboard/src/pages/KnowledgeBases/`

| File | Lines | Description |
|------|------:|-------------|
| `TextDocumentEditorModal.test.tsx` | 101 | React component or page. |
| `TextDocumentEditorModal.tsx` | 270 | Re-export preview helpers so existing page imports keep working. |
| `index.tsx` | 3455 | Default export: KnowledgeBasesPage. |
| `knowledgeDeepLink.test.ts` | 40 | TS utilities, API, hooks, or types. |
| `knowledgeDeepLink.ts` | 30 | TS utilities, API, hooks, or types. |
| `knowledgeFolder.test.ts` | 56 | TS utilities, API, hooks, or types. |
| `knowledgeFolder.ts` | 26 | TS utilities, API, hooks, or types. |
| `knowledgeIcons.tsx` | 65 | React component or page. |
| `mdOutline.test.ts` | 33 | TS utilities, API, hooks, or types. |
| `mdOutline.ts` | 46 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Login/`

| File | Lines | Description |
|------|------:|-------------|
| `CaptchaField.test.tsx` | 139 | React component or page. |
| `CaptchaField.tsx` | 290 | React component or page. |
| `OidcComplete.test.tsx` | 36 | React component or page. |
| `OidcComplete.tsx` | 124 | Default export: OidcComplete. |
| `SlideCaptcha.test.tsx` | 85 | React component or page. |
| `SlideCaptcha.tsx` | 246 | Default export: SlideCaptcha. |
| `captchaAdapters.ts` | 75 | TS utilities, API, hooks, or types. |
| `index.tsx` | 363 | Default export: LoginPage. |

### `dashboard/src/pages/PwaDebug/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 703 | Default export: PwaDebugPage. |

### `dashboard/src/pages/Settings/AdvancedSettings/`

| File | Lines | Description |
|------|------:|-------------|
| `CaptchaSettings.tsx` | 232 | Default export: CaptchaSettingsPanel. |
| `TabPanelHeader.tsx` | 35 | React component or page. |
| `UpdateConfig.tsx` | 577 | Default export: UpdateConfig. |
| `index.tsx` | 99 | Default export: AdvancedSettingsPage. |

### `dashboard/src/pages/Settings/BackupRestore/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 878 | Default export: BackupRestorePanel. |

### `dashboard/src/pages/Settings/Embedding/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 511 | State management. |

### `dashboard/src/pages/Settings/Environments/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 310 | React component or page. |
| `useEnvVars.ts` | 33 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Settings/Environments/components/`

| File | Lines | Description |
|------|------:|-------------|
| `AddButton.tsx` | 25 | React component or page. |
| `EmptyState.tsx` | 20 | React component or page. |
| `EnvRow.tsx` | 111 | React component or page. |
| `EnvRowCard.tsx` | 111 | React component or page. |
| `PageHeader.tsx` | 43 | React component or page. |
| `Toolbar.tsx` | 90 | React component or page. |
| `index.ts` | 6 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Settings/Environments/utils/`

| File | Lines | Description |
|------|------:|-------------|
| `index.ts` | 1 | TS utilities, API, hooks, or types. |
| `sensitive.test.ts` | 111 | TS utilities, API, hooks, or types. |
| `sensitive.ts` | 77 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Settings/HttpsSettings/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 360 | React component or page. |

### `dashboard/src/pages/Settings/Language/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 63 | Default export: LanguagePage. |

### `dashboard/src/pages/Settings/MediaGeneration/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 360 | React component or page. |

### `dashboard/src/pages/Settings/Models/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 427 | Default export: ModelsPage. |
| `modelMeta.tsx` | 89 | React component or page. |
| `presetUtils.ts` | 215 | TS utilities, API, hooks, or types. |
| `providerApi.ts` | 83 | TS utilities, API, hooks, or types. |
| `useProviders.ts` | 194 | TS utilities, API, hooks, or types. |
| `wizardModelMeta.ts` | 80 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Settings/Models/components/`

| File | Lines | Description |
|------|------:|-------------|
| `CodexOAuthConnect.tsx` | 140 | React component or page. |
| `PresetModelPicker.tsx` | 91 | React component or page. |
| `index.ts` | 5 | Re-export all components from subdirectories — mirrors finnie's |

### `dashboard/src/pages/Settings/Models/components/cards/`

| File | Lines | Description |
|------|------:|-------------|
| `LocalServiceCard.tsx` | 311 | React component or page. |
| `PresetGroupCard.tsx` | 263 | React component or page. |
| `PresetProviderCard.tsx` | 154 | React component or page. |
| `ProviderCard.tsx` | 385 | React component or page. |
| `index.ts` | 4 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Settings/Models/components/modals/`

| File | Lines | Description |
|------|------:|-------------|
| `CustomProviderModal.tsx` | 359 | React component or page. |
| `ModelListEditor.tsx` | 859 | React component or page. |
| `PresetProviderModal.tsx` | 361 | React component or page. |
| `ProviderConfigModal.tsx` | 1227 | React component or page. |
| `index.ts` | 4 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Settings/Models/components/sections/`

| File | Lines | Description |
|------|------:|-------------|
| `ActiveModelPool.tsx` | 391 | React component or page. |
| `LoadingState.tsx` | 41 | React component or page. |
| `index.ts` | 2 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Settings/Observability/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 198 | React component or page. |

### `dashboard/src/pages/Settings/SearchConfig/`

| File | Lines | Description |
|------|------:|-------------|
| `index.test.tsx` | 51 | React component or page. |
| `index.tsx` | 476 | Default export: SearchConfigPage. |

### `dashboard/src/pages/Settings/Security/`

| File | Lines | Description |
|------|------:|-------------|
| `AuditLogPanel.tsx` | 313 | Default export: AuditLogPanel. |
| `HitlToolsPicker.tsx` | 114 | Default export: HitlToolsPicker. |
| `ToolGuardRulesPanel.tsx` | 260 | Default export: ToolGuardRulesPanel. |
| `index.tsx` | 425 | Default export: SecuritySettingsPage. |

### `dashboard/src/pages/Settings/Voice/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 500 | React component or page. |

### `dashboard/src/pages/Settings/octop/`

| File | Lines | Description |
|------|------:|-------------|
| `AdminAgentCard.tsx` | 234 | React component or page. |
| `Agents.tsx` | 400 | Default export: OctopAgentsPage. |
| `Providers.tsx` | 225 | Default export: OctopProvidersPage. |

### `dashboard/src/pages/Setup/`

| File | Lines | Description |
|------|------:|-------------|
| `index.tsx` | 307 | Default export: SetupPage. |
| `wizardClient.ts` | 223 | TS utilities, API, hooks, or types. |

### `dashboard/src/pages/Setup/steps/`

| File | Lines | Description |
|------|------:|-------------|
| `AdminStep.tsx` | 252 | Default export: AdminStep. |
| `DatabaseStep.tsx` | 322 | Default export: DatabaseStep. |
| `FinishStep.tsx` | 120 | Default export: FinishStep. |
| `ModelStep.tsx` | 1293 | Default export: ModelStep. |
| `PasswordStep.tsx` | 131 | Default export: PasswordStep. |

### `dashboard/src/pages/SkillPackages/`

| File | Lines | Description |
|------|------:|-------------|
| `PackageIcon.tsx` | 20 | React component or page. |
| `PackageSkillCard.tsx` | 111 | React component or page. |
| `PackageSkillsTable.tsx` | 102 | Default export: PackageSkillsTable. |
| `SkillsetFromHubDrawer.test.tsx` | 97 | React component or page. |
| `SkillsetFromHubDrawer.tsx` | 147 | React component or page. |
| `index.tsx` | 1082 | Default export: SkillPackagesPage. |

### `dashboard/src/plugins/toolRenderers/`

| File | Lines | Description |
|------|------:|-------------|
| `ToolUiErrorBoundary.tsx` | 39 | React component or page. |
| `ensureBuiltins.ts` | 34 | TS utilities, API, hooks, or types. |
| `host.ts` | 89 | TS utilities, API, hooks, or types. |
| `index.ts` | 37 | TS utilities, API, hooks, or types. |
| `isPinnedToolUi.test.ts` | 119 | TS utilities, API, hooks, or types. |
| `isPinnedToolUi.ts` | 65 | TS utilities, API, hooks, or types. |
| `loader.ts` | 95 | TS utilities, API, hooks, or types. |
| `parseToolOutput.test.ts` | 91 | TS utilities, API, hooks, or types. |
| `parseToolOutput.ts` | 84 | TS utilities, API, hooks, or types. |
| `registry.ts` | 100 | TS utilities, API, hooks, or types. |
| `toolPluginIndex.ts` | 23 | TS utilities, API, hooks, or types. |
| `types.ts` | 88 | TS utilities, API, hooks, or types. |
| `usePluginToolUis.ts` | 50 | TS utilities, API, hooks, or types. |
| `useToolRendererVersion.ts` | 11 | TS utilities, API, hooks, or types. |

### `dashboard/src/plugins/toolRenderers/builtin/`

| File | Lines | Description |
|------|------:|-------------|
| `BuiltinOctopUiFallback.tsx` | 88 | React component or page. |
| `DefaultToolRenderer.tsx` | 244 | React component or page. |

### `dashboard/src/routes/`

| File | Lines | Description |
|------|------:|-------------|
| `controlAdminPath.test.ts` | 126 | TS utilities, API, hooks, or types. |
| `index.tsx` | 287 | Lazy-loaded pages — Common |
| `prefetch.ts` | 44 | TS utilities, API, hooks, or types. |

### `dashboard/src/styles/`

| File | Lines | Description |
|------|------:|-------------|
| `appearanceStorage.test.ts` | 118 | TS utilities, API, hooks, or types. |
| `appearanceStorage.ts` | 108 | TS utilities, API, hooks, or types. |
| `customPalette.test.ts` | 121 | TS utilities, API, hooks, or types. |
| `themePalettes.test.ts` | 71 | TS utilities, API, hooks, or types. |
| `themePalettes.ts` | 467 | TS utilities, API, hooks, or types. |

### `dashboard/src/test/`

| File | Lines | Description |
|------|------:|-------------|
| `memoryFixtures.ts` | 212 | TS utilities, API, hooks, or types. |
| `setup.ts` | 130 | Auto-mock react-i18next so components' ``t(key, fallback)`` calls |

### `dashboard/src/utils/`

| File | Lines | Description |
|------|------:|-------------|
| `agentError.ts` | 76 | TS utilities, API, hooks, or types. |
| `agentRuntimeConfig.test.ts` | 47 | TS utilities, API, hooks, or types. |
| `agentRuntimeConfig.ts` | 57 | TS utilities, API, hooks, or types. |
| `antdMessage.ts` | 29 | TS utilities, API, hooks, or types. |
| `antdModal.ts` | 28 | TS utilities, API, hooks, or types. |
| `apiError.ts` | 124 | TS utilities, API, hooks, or types. |
| `browserCanvas.ts` | 63 | TS utilities, API, hooks, or types. |
| `browserProfile.ts` | 10 | TS utilities, API, hooks, or types. |
| `browserSpeech.ts` | 269 | TS utilities, API, hooks, or types. |
| `browserTabs.ts` | 94 | TS utilities, API, hooks, or types. |
| `browserViewport.ts` | 21 | TS utilities, API, hooks, or types. |
| `chatStreamError.test.ts` | 84 | TS utilities, API, hooks, or types. |
| `chatStreamError.ts` | 182 | TS utilities, API, hooks, or types. |
| `collectTurnKnowledgeCitations.ts` | 23 | TS utilities, API, hooks, or types. |
| `collectTurnToolMedia.ts` | 45 | TS utilities, API, hooks, or types. |
| `confirmModal.ts` | 27 | TS utilities, API, hooks, or types. |
| `copyText.test.ts` | 62 | TS utilities, API, hooks, or types. |
| `copyText.ts` | 42 | fall through to the legacy path |
| `dashboardPushToast.test.ts` | 52 | TS utilities, API, hooks, or types. |
| `dashboardPushToast.ts` | 33 | TS utilities, API, hooks, or types. |
| `desktopChrome.test.ts` | 201 | TS utilities, API, hooks, or types. |
| `desktopChrome.ts` | 172 | 3×12px lights + 2×8px gaps + 8px/12px padding, plus a little slack. |
| `desktopExternalLinks.ts` | 113 | TS utilities, API, hooks, or types. |
| `desktopViewport.ts` | 28 | TS utilities, API, hooks, or types. |
| `detailRequestGate.test.ts` | 13 | TS utilities, API, hooks, or types. |
| `detailRequestGate.ts` | 12 | TS utilities, API, hooks, or types. |
| `docKind.ts` | 43 | TS utilities, API, hooks, or types. |
| `embeddingDownload.ts` | 69 | TS utilities, API, hooks, or types. |
| `expertColor.test.ts` | 77 | TS utilities, API, hooks, or types. |
| `expertColor.ts` | 119 | TS utilities, API, hooks, or types. |
| `fileTreeIcon.tsx` | 191 | React component or page. |
| `formDraft.ts` | 50 | TS utilities, API, hooks, or types. |
| `formatMessageTime.test.ts` | 76 | TS utilities, API, hooks, or types. |
| `formatMessageTime.ts` | 135 | TS utilities, API, hooks, or types. |
| `formatToolArguments.ts` | 44 | Fall through — try last balanced `{...}` block. |
| `fromWorkspace.ts` | 10 | TS utilities, API, hooks, or types. |
| `h264CanvasDecoder.ts` | 110 | TS utilities, API, hooks, or types. |
| `headerUtils.ts` | 60 | TS utilities, API, hooks, or types. |
| `injectPendingHitlMessage.ts` | 29 | TS utilities, API, hooks, or types. |
| `knowledgeCitationDisplay.test.ts` | 79 | TS utilities, API, hooks, or types. |
| `knowledgeCitationDisplay.ts` | 53 | TS utilities, API, hooks, or types. |
| `knowledgeDocPreview.test.ts` | 81 | TS utilities, API, hooks, or types. |
| `knowledgeDocPreview.ts` | 116 | TS utilities, API, hooks, or types. |
| `knowledgePath.test.ts` | 38 | TS utilities, API, hooks, or types. |
| `knowledgePath.ts` | 60 | TS utilities, API, hooks, or types. |
| `locale.ts` | 45 | TS utilities, API, hooks, or types. |
| `localePrefs.test.ts` | 52 | TS utilities, API, hooks, or types. |
| `localePrefs.ts` | 63 | TS utilities, API, hooks, or types. |
| `localizedText.ts` | 22 | TS utilities, API, hooks, or types. |
| `markdown.ts` | 139 | TS utilities, API, hooks, or types. |
| `messageParser.history.test.ts` | 50 | TS utilities, API, hooks, or types. |
| `messageParser.ts` | 365 | ── Shared types ────────────────────────────────────────────────────────── |
| `mobileDevice.ts` | 9 | iPadOS 13+ reports MacIntel |
| `mobileTypeScale.ts` | 11 | TS utilities, API, hooks, or types. |
| `modelOptions.ts` | 54 | TS utilities, API, hooks, or types. |
| `normalizeUrl.ts` | 7 | TS utilities, API, hooks, or types. |
| `parseAcpPermission.ts` | 65 | TS utilities, API, hooks, or types. |
| `parseHarnessChunk.test.ts` | 26 | TS utilities, API, hooks, or types. |
| `parseHarnessChunk.ts` | 248 | TS utilities, API, hooks, or types. |
| `parseKnowledgeCitations.test.ts` | 46 | TS utilities, API, hooks, or types. |
| `parseKnowledgeCitations.ts` | 58 | TS utilities, API, hooks, or types. |
| `parseWriteTodos.test.ts` | 115 | TS utilities, API, hooks, or types. |
| `parseWriteTodos.ts` | 166 | TS utilities, API, hooks, or types. |
| `passwordPolicy.ts` | 48 | TS utilities, API, hooks, or types. |
| `pastelIconBackground.ts` | 42 | TS utilities, API, hooks, or types. |
| `permissions.ts` | 236 | TS utilities, API, hooks, or types. |
| `plainTextForSpeech.test.ts` | 54 | TS utilities, API, hooks, or types. |
| `plainTextForSpeech.ts` | 60 | TS utilities, API, hooks, or types. |
| `quickInputPrefill.ts` | 36 | A trailing colon invites the user to fill in a value, such as "The goal is:". |
| `readNumber.ts` | 11 | TS utilities, API, hooks, or types. |
| `reloadOnStaleChunk.test.ts` | 233 | TS utilities, API, hooks, or types. |
| `reloadOnStaleChunk.ts` | 142 | A stale hashed chunk can also surface as a MIME rejection: the SPA fallback |
| `sharedExpert.ts` | 34 | TS utilities, API, hooks, or types. |
| `shellCodeBlock.ts` | 25 | TS utilities, API, hooks, or types. |
| `showApiToast.ts` | 13 | TS utilities, API, hooks, or types. |
| `slashCategories.ts` | 54 | TS utilities, API, hooks, or types. |
| `slashFallbackCommands.ts` | 93 | TS utilities, API, hooks, or types. |
| `slashIcons.ts` | 48 | TS utilities, API, hooks, or types. |
| `ssoPopup.ts` | 47 | TS utilities, API, hooks, or types. |
| `subagentEmojis.test.ts` | 18 | TS utilities, API, hooks, or types. |
| `subagentEmojis.ts` | 124 | Auto-curated from bundled subagent catalog emojis (+ stable default first). |
| `toolMediaBlocks.test.ts` | 197 | TS utilities, API, hooks, or types. |
| `toolMediaBlocks.ts` | 866 | TS utilities, API, hooks, or types. |
| `updateStatusCache.test.ts` | 77 | TS utilities, API, hooks, or types. |
| `updateStatusCache.ts` | 72 | TS utilities, API, hooks, or types. |
| `wavStreamPlayer.test.ts` | 90 | TS utilities, API, hooks, or types. |
| `wavStreamPlayer.ts` | 190 | TS utilities, API, hooks, or types. |
| `workspaceIoPath.test.ts` | 42 | TS utilities, API, hooks, or types. |
| `workspaceIoPath.ts` | 87 | TS utilities, API, hooks, or types. |
| `workspacePath.ts` | 43 | TS utilities, API, hooks, or types. |

### `dashboard/src/utils/__tests__/`

| File | Lines | Description |
|------|------:|-------------|
| `browserTabs.test.ts` | 100 | TS utilities, API, hooks, or types. |

### `desktop/portable/`

| File | Lines | Description |
|------|------:|-------------|
| `verify_imports.py` | 139 | Smoke-check a green packages/ tree against frozen requirements. Usage: python desktop/portable/verify_imports.py \ --packages desktop/portable/release/Octop-<plat>/packages \ --requirements desktop/portable/requirements-<plat>.txt \ [--overrides desktop/portable/overrides-<plat>.… |

### `desktop/portable/templates/`

| File | Lines | Description |
|------|------:|-------------|
| `launch.py` | 61 | Green portable entry: wire packages/ onto sys.path then run ``octop``. Using plain ``PYTHONPATH=packages`` skips ``.pth`` processing (pywin32 etc.). ``site.addsitedir`` loads those hooks. On Windows we also expose ``pywin32_system32`` DLLs so ``import pywintypes`` works. |

### `desktop/src/build/`

| File | Lines | Description |
|------|------:|-------------|
| `stamp_version.py` | 101 | Stamp the Octop version into Wails desktop metadata copies. |

### `plugins/demo-greeting-skill/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 15 | Demo Greeting Skill — sample skill plugin (kind=skill). Skill plugins do not register callable tools. They declare a directory with ctx.skills(...); on agent start Octop syncs each skills/<name>/SKILL.md into the agent workspace under skills/. |

### `plugins/demo-toolkit/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 66 | Demo Toolkit — sample tool plugin (kind=tool). How to write an Octop plugin (copy this file as a template): 1. This directory must contain plugin.yaml (id / version / name / kind / entry). 2. The entry file (main.py here) must define setup(ctx: PluginContext). 3. Register callabl… |

### `plugins/demo-turn-logger/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 60 | Demo Turn Logger — sample hook plugin (kind=hook). Hook plugins register a LangChain AgentMiddleware via ctx.middleware(...). Octop attaches middleware from globally enabled plugins to the agent chain (lower priority runs earlier). This demo only emits observability logs and does… |

### `plugins/demo-ui-card/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 33 | Demo UI Card — backend tool that returns an ``octop_ui`` envelope. The matching frontend renderer lives in ``ui/dist/`` and is loaded by the Dashboard into the chat tool-result registry. |

### `scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `release_download_links.py` | 105 | Build GitHub Release download-section markdown from a version tag. Same pattern as clash-verge-rev: interpolate TAG / VERSION into a fixed filename table. Do not scrape the assets list. python3 scripts/release_download_links.py 0.9.31 python3 scripts/release_download_links.py v0.… |
| `smoke_memory_api.py` | 119 | End-to-end smoke for the orca memory router using TestClient. Bypasses OctopServer / DB / auth — we override the FastAPI deps used by the router (``current_user``, ``get_server``, ``require_agent_row``) with stubs that point straight at the real ZYWZTD sqlite. Useful for hand-che… |

### `src/octop/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 5 | Octop — smarter self-hosted AI assistant (multi-user, multi-agent). |
| `__main__.py` | 8 | Entry point: ``python -m octop`` → CLI. |
| `config.py` | 626 | Process-level configuration (config.json + env overrides). |
| `launch.py` | 156 | Composition root — wire OctopServer, FastAPI, and uvicorn for ``octop run``. |

### `src/octop/api/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `app.py` | 312 | FastAPI app factory. |
| `deps.py` | 230 | FastAPI Depends, JWT helpers, and auth routing. |
| `openapi_meta.py` | 199 | OpenAPI metadata for Scalar API docs. |

### `src/octop/api/common/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Shared router helpers — agent access, workspace glue, request validation. |
| `agent.py` | 79 | Shared agent ownership / existence checks for HTTP routers. |
| `agent_runtime.py` | 32 | Typed API fields for per-agent runtime and model settings. |
| `agent_workspace.py` | 19 | Resolve agent ``workspace_dir`` for API handlers. |
| `attachments.py` | 69 | Chat attachment HTTP helpers — thin wrappers over :mod:`inbound_store`. |
| `content_disposition.py` | 35 | Build latin-1-safe Content-Disposition values (RFC 5987). |
| `memory_client.py` | 191 | Per-agent ``Memory`` instance management for the dashboard router. Owns a tiny LRU cache of ``harness_memory.core.Memory`` instances keyed by ``agent_id``. Backend may be sqlite (default workspace file) or postgres when ``config_json.memory.backend`` says so. |
| `public_base.py` | 29 | Resolve the externally visible HTTP origin for an incoming request. |
| `sso_cookie.py` | 37 | Shared SSO state cookie for OIDC and OAuth login callbacks. |
| `upload_limit.py` | 54 | Cap multipart uploads before the whole body is assembled in memory. |
| `validators.py` | 87 | Cross-router request validation helpers (no FastAPI route definitions). |
| `workspace.py` | 133 | Shared workspace helpers (not route handlers). |

### `src/octop/api/middleware/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | ASGI middlewares for octop. |
| `jwt_auth.py` | 72 | Require JWT for all /api/* routes except an explicit allowlist. Validated users are cached on ``request.state.octop_user`` so route-level ``Depends(current_user)`` can reuse the result without re-decoding. When the access token is past the sliding-renew threshold, a fresh token i… |
| `setup_lockdown.py` | 41 | Lock down all non-/setup endpoints while no users exist. Returns ``503 {"setup_required": true}`` so the SPA can hard-redirect to ``/setup``. |

### `src/octop/api/routers/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `acp.py` | 303 | ACP runner configuration API. Runners are stored globally per user (``settings`` table). Each agent only stores ``acp.tool_enabled`` for the ``acp_runner`` built-in tool. |
| `admin.py` | 81 | Admin overview/audit/metrics router. |
| `agent_files.py` | 226 | Per-agent Memory + Heartbeat endpoints. Two related concerns share this module because they both surface *files* in the agent's workspace plus *configuration* on the agent row: GET /api/agents/{aid}/heartbeat-config PUT /api/agents/{aid}/heartbeat-config GET /api/agents/{aid}/mem… |
| `agent_tools.py` | 188 | Per-agent tool settings — built-in denylist + plugin tool enable flags. |
| `agents.py` | 552 | Agents router. |
| `auth.py` | 172 | Login / logout / me / change-password. |
| `auth_oauth.py` | 233 | Pluggable OAuth SSO HTTP routes (Feishu and future providers). |
| `auth_oidc.py` | 182 | OpenID Connect SSO HTTP routes. |
| `backup.py` | 432 | Admin backup / restore API. |
| `channels.py` | 1071 | Channels router. |
| `connectors.py` | 1401 | Connector and agent-binding HTTP API. |
| `cron.py` | 260 | Cron router. |
| `envs.py` | 134 | Environment variables API — backed by ``~/.octop/env`` (inherited by all agents). |
| `experts.py` | 760 | Experts router — bundled scene templates + SkillHub expert market. GET /api/experts → bundled expert summaries GET /api/experts/{id} → template metadata + lazy ``file_contents`` POST /api/agents/from-expert/{id} → create agent from bundled expert GET /api/experts/hub → SkillHub m… |
| `filesystem.py` | 226 | Host filesystem browsing for dashboard forms (root_dir pickers). Security notes: - Authenticated users only (JWT). - Paths are resolved with ``os.path.realpath`` and must stay under the browse tree root (``startswith`` containment — CodeQL-recognized sanitizer). A denylist furthe… |
| `health.py` | 28 | Liveness probe (no auth). |
| `i18n.py` | 47 | Localized strings API (tool labels, etc.). |
| `internal_mcp.py` | 147 | Internal HTTP MCP endpoints for Octop-hosted connector gateways. |
| `invites.py` | 158 | Admin invite CRUD and public invite validate/redeem. |
| `knowledge_bases.py` | 961 | HTTP API for private, shareable knowledge bases. |
| `mbti.py` | 766 | MBTI personality type API — profiles, test, and per-agent persona apply. Octop is multi-agent: the MBTI code lives on ``agents.persona_mbti`` / ``config_json["persona"]`` and is rendered into workspace ``SOUL.md`` on agent reload. The active agent is selected via ``X-Octop-Agent-… |
| `media_generation.py` | 145 | Admin API for instance-wide image and video generation settings. |
| `memory.py` | 871 | Per-agent memory dashboard router. Mounts at ``/api/agents/{agent_id}/memory/*`` and forwards each endpoint to a single JSON-RPC method on the agent's ``harness_memory.Bridge``. The router itself contains no business logic — every handler is one ``call_memory_rpc(...)`` call. Sur… |
| `memory_portable.py` | 303 | Cross-host memory migration service REST endpoint (requirements 1, 2, 3, 5, 8). Mounted routes: GET /api/memory/portable/sources POST /api/agents/{agent_id}/memory/portable/pack POST /api/agents/{agent_id}/memory/portable/adopt POST /api/agents/{agent_id}/memory/portable/doctor |
| `observability.py` | 87 | Observability configuration API (admin). |
| `ollama_download_store.py` | 130 | In-memory store for tracking background model download tasks. Multiple downloads can run concurrently. Completed/failed results are retained until explicitly cleared so the frontend can poll for the final state. |
| `ollama_models.py` | 305 | API endpoints for Ollama model management. This router delegates lifecycle operations (list / pull / delete) to the Ollama daemon via OllamaModelManager. Downloads run in the background and their status can be polled by the frontend. |
| `onnx_models.py` | 219 | Local ONNX embedding model cache API (Models admin — local tab). Manages catalog download / probe / enable for local ONNX embedding weights. This is not a chat Provider. Knowledge Bases consume these models for embedding. |
| `plugins.py` | 430 | Plugin install and agent tool configuration. |
| `preferences.py` | 166 | Per-user preferences (locale, etc.). |
| `proactive_care.py` | 145 | Proactive care push configuration API. Exposes GET/PUT /api/agents/{agent_id}/proactive-care endpoints for reading and updating an agent's proactive care push configuration. |
| `providers.py` | 485 | Providers router (admin-only write operations). |
| `search.py` | 52 | Search-provider connectivity API (Settings → Advanced → Search). |
| `security.py` | 204 | Security policy configuration API (admin). |
| `settings.py` | 153 | Process-level settings exposed to authenticated clients. |
| `setup.py` | 443 | Initial-admin setup wizard. |
| `skill_packages.py` | 772 | HTTP API for instance-global skill packages. |
| `skills.py` | 1417 | Skills router — per-agent ``SKILL.md`` library. Each agent's skills live under its harness backend at ``/skills/<name>/SKILL.md`` (matching finnie's convention). This router thinly wraps the workspace backend so the dashboard sees a *named* skills view rather than a raw file list… |
| `slash.py` | 74 | Slash command discovery API. |
| `storage_backends.py` | 239 | Storage backends router — admin-only. |
| `subagents.py` | 244 | Subagents router — per-agent ``agents/**/*.md`` definitions and bundled catalog. Each agent's subagents live under its harness workspace at ``agents/<slug>.md``. This router exposes summaries for the dashboard; editing uses the workspace file API (``PUT .../workspace/file``) and … |
| `terminal.py` | 787 | Terminal WebSocket — interactive PTY sessions per agent (P1.4). WS /api/agents/{agent_id}/terminal/ws?token=<JWT>&session_id=&cols=&rows= The shell is spawned with the agent's ``workspace_dir`` as cwd so the session lands the user where their files are. Authentication uses a ``?t… |
| `tls.py` | 112 | TLS / Let's Encrypt admin API. |
| `update.py` | 412 | Self-update API — mirrors finnie/octop dashboard update flow. |
| `update_store.py` | 85 | In-memory upgrade task tracking for ``/api/update/*``. |
| `uploads.py` | 83 | Dashboard chat attachments — stored in agent workspace ``inbound/``. |
| `usage.py` | 286 | Token usage router — query the usage_log ledger. GET /api/usage/summary → caller's own roll-up GET /api/usage/summary?as_user=N → admin scope; another user's roll-up GET /api/usage/summary?agent_id=X → scope to one agent (must be visible) GET /api/usage/export.xlsx → Excel detail… |
| `users.py` | 279 | Admin CRUD for users. |
| `voice.py` | 262 | Voice STT/TTS router. |
| `workspace.py` | 569 | Workspace router — read/write into a running agent's workspace. |

### `src/octop/api/routers/browser/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 20 | Browser routers — env/install, harness-browser, record/replay, WS screencast. |
| `env.py` | 174 | Browser environment probe and Chromium install (SSE). Live sessions are harness-browser (see ``harness.py`` / ``stream.py``), not in-process Playwright. GET /api/browser/env-status → { playwright, browsers_ok, harness_browser, … } POST /api/browser/install → SSE Playwright Chromi… |
| `harness.py` | 367 | Helpers for attaching dashboard / chat UI to harness-browser sessions. |
| `record_replay.py` | 324 | Browser record/replay endpoints backed by harness-browser. |
| `stream.py` | 430 | WebSocket browser screencast — attaches to harness-browser sessions. Wire protocol matches the dashboard ``useBrowserStream`` hook (screencast) and ``useBrowserSessionState`` (listen-only status): Client → Server:: {"type": "start", "url": "", "width": 1280, "height": 800, "reuse… |
| `uninstall.py` | 34 | Browser environment uninstallation (SSE). |

### `src/octop/api/routers/chat/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 20 | Dashboard chat routers — WebSocket turns, thread CRUD, polish, HITL. |
| `history.py` | 581 | Dashboard thread list/history REST APIs. |
| `models.py` | 193 | Pydantic models for dashboard chat (WebSocket turn + REST helpers). |
| `notify_ws.py` | 76 | Dashboard-wide notification WebSocket — text pushes (cron / proactive care). |
| `routes.py` | 255 | Dashboard chat helpers: polish prompt and HITL resume (SSE). |
| `serialize.py` | 1023 | Thread history loading and LangGraph message serialization. |
| `sse.py` | 32 | SSE / WebSocket frame formatting helpers. |
| `trajectory.py` | 400 | Thread trajectory REST APIs (history, detail, metrics, export, live SSE). |
| `turn.py` | 354 | Dashboard turn preparation (thread, MCP, skills, InboundMessage). |
| `ws.py` | 183 | Dashboard chat over WebSocket — routes turns through Gateway / GlobalProcessor. |

### `src/octop/api/routers/desktop/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 20 | Remote desktop routers. |
| `install.py` | 38 | Desktop environment installation (SSE). |
| `settings.py` | 49 | Remote desktop settings (geometry). |
| `status.py` | 38 | Remote desktop HTTP status. |
| `stream.py` | 379 | WebSocket remote desktop stream — JPEG frames + OS input injection. |
| `uninstall.py` | 34 | Desktop environment uninstallation (SSE). |

### `src/octop/api/routers/mobile/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 18 | Remote Android routers. |
| `install.py` | 29 | Remote Android container install (SSE). |
| `shell_ws.py` | 171 | WebSocket PTY for ``adb -s <serial> shell``. |
| `status.py` | 112 | Remote Android HTTP status and agent-control binding. |
| `stream.py` | 633 | WebSocket Remote Android stream — H.264 screenrecord with JPEG fallback. |

### `src/octop/cli/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | octop CLI package. |
| `main.py` | 130 | octop CLI entry point with lazy command loading. |
| `registry.py` | 36 | CLI command registry for lazy loading. |

### `src/octop/cli/commands/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Click command modules (one file per top-level `octop` subcommand). |
| `acp.py` | 63 | `octop acp` — run Octop agent as an ACP server over stdio. |
| `admin.py` | 122 | octop admin commands. |
| `agent.py` | 237 | octop agent commands. |
| `backup.py` | 188 | `octop backup` — export and restore Octop data. |
| `captcha.py` | 35 | `octop captcha` — offline captcha maintenance (lockout escape hatch). |
| `channel.py` | 571 | octop channel commands (local DB + embedded runtime). |
| `chats.py` | 364 | octop chats — thread CRUD + interactive REPL (CLI channel / local DB). |
| `clean.py` | 44 | `octop clean` — remove CLI state and (optionally) all Octop data. |
| `completion.py` | 58 | octop completion: emit / install shell completion script. |
| `config.py` | 40 | octop config — CLI defaults (user / agent pins). |
| `cron.py` | 119 | octop cron commands. |
| `init.py` | 120 | `octop init` — bootstrap a fresh Octop install (DB + first admin). |
| `models.py` | 244 | octop models — provider presets and resolved model list. |
| `plugin.py` | 80 | ``octop plugin`` — install and manage plugins. |
| `provider.py` | 107 | octop provider commands. |
| `run.py` | 191 | `octop run` — start the FastAPI app in the foreground. |
| `service.py` | 214 | `octop service` — install and manage the Octop system service. |
| `skills.py` | 138 | octop skills — per-agent skill library. |
| `update.py` | 78 | `octop update` — self-upgrade via pip or uv. |
| `user.py` | 148 | octop user commands. |
| `version.py` | 17 | octop version command. |

### `src/octop/cli/repl/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Chat REPL rendering and session state. |
| `embedded_session.py` | 42 | Ref-counted embedded OctopServer for reuse within one asyncio event loop. |
| `render.py` | 321 | Terminal rendering for harness stream chunks (CLI chat). |
| `runtime.py` | 129 | Embedded OctopServer chat runtime via the CLI gateway channel. |
| `session.py` | 58 | REPL session state for toolbar and slash side-effects. |
| `toolbar.py` | 17 | prompt_toolkit bottom toolbar for ``octop chats repl``. |
| `turn.py` | 13 | Shared chat turn result for CLI streaming. |

### `src/octop/cli/support/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Shared CLI internals — HTTP client, context, state, prompts, helpers. |
| `acting.py` | 30 | Resolve acting user for CLI commands. |
| `ctx.py` | 68 | Shared CLI context helpers — root-level option fallback. Plan §13.1 requires the root ``cli`` to accept ``--user`` and ``--agent`` options that subcommands inherit when not given explicitly. Subcommands keep their own ``--user`` / ``--agent`` options; this module provides a singl… |
| `db.py` | 150 | Offline DB access for CLI commands that only need local SQLite. |
| `embedded_ops.py` | 99 | Embedded OctopServer helpers for CLI ops that need a live runtime. |
| `errors.py` | 14 | CLI error helpers. |
| `feishu_creator.py` | 73 | Run Feishu bot-creator subprocess locally (no HTTP). |
| `offline_ops.py` | 555 | Direct infra/DB helpers for local CLI (no HTTP, no login). |
| `prompts.py` | 49 | Symbols: select, checkbox, text, password, confirm, editor |
| `qr.py` | 167 | Terminal QR code rendering (adapted from finnie channels_cmd). |
| `skills.py` | 46 | Offline CLI helpers for per-agent skills (no HTTP / login). |
| `state.py` | 36 | CLI state — pinned default user/agent. |
| `stub.py` | 24 | Helper for STUB commands that exist in --help but exit on use. |

### `src/octop/i18n/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 45 | Central i18n: JSON locale bundles, lookup, and domain helpers. |
| `loader.py` | 64 | Load nested locale JSON and resolve dot-path keys. |

### `src/octop/i18n/domains/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 59 | Per-namespace i18n helpers (each maps to a top-level JSON key). |
| `agents.py` | 91 | ``agents.*`` — agent runtime labels and startup error keys. |
| `attachment.py` | 44 | ``attachment.*`` — inbound path hints and empty-turn placeholders. |
| `channel.py` | 41 | ``channel.*`` — IM channel status lines (tool hints, probe errors). |
| `errors.py` | 12 | ``errors.*`` — API / OctopError messages. |
| `skills.py` | 28 | ``skills.*`` — built-in skill display names (keyed by slug). |
| `slash.py` | 23 | ``slash.*`` — slash command responses and catalog labels. |
| `stream.py` | 191 | ``stream_errors.*`` — user-facing guidance for chat / IM model failures. |
| `tls.py` | 9 | TLS preflight message helpers. |
| `tools.py` | 121 | ``tools.*`` — built-in agent tool display names. |
| `voice.py` | 38 | ``voice.*`` — probe and Tencent provider error copy. |

### `src/octop/infra/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 21 | Octop infrastructure — all domain logic and utilities. Sub-packages: agents — agent factory, runtime, manager, MBTI personas, expert templates channels — IM channels, processor, slash commands cron — CronJob, CronManager, trigger parsing db — SqlitePool, SQL migrations, Repo clas… |
| `errors.py` | 277 | Stable error codes and the typed exception that crosses API boundaries. |
| `metrics.py` | 37 | In-memory metrics counters. |
| `server.py` | 585 | OctopServer — process-level orchestrator. |

### `src/octop/infra/agents/`

| File | Lines | Description |
|------|------:|-------------|
| `acp_settings.py` | 144 | User-scoped global ACP runner configuration (settings table). |
| `avatar.py` | 222 | Expert avatars stored in the agent workspace via ``BackendWorkspace``. |
| `context_breakdown.py` | 254 | Context window usage without checkpoint reads on dashboard requests. |
| `default_agent.py` | 89 | Bootstrap a first default agent (general-assistant) for a user. Used by the setup wizard (pinned ``agent_id=main``) and by invite redeem (auto-allocated agent id) so every new account starts with the same expert. |
| `execute_env.py` | 138 | Inject platform execute defaults into harness backend specs. Global admin env (``~/.octop/env``) and workspace ``.env`` are merged at **execute** time inside harness (``inherit_env`` / ``environment_file`` / ``BackendWorkspace`` reader) — not snapshotted here. ``OCTOP_AUTH_DIR`` … |
| `langfuse.py` | 166 | Langfuse settings persistence for the agent runtime. |
| `manager.py` | 2962 | AgentManager — process-wide singleton managing all HarnessAgent instances. |
| `mbti_profiles.py` | 596 | Built-in MBTI personality profiles for the 16 types. Pure data module with zero runtime dependencies. Provides structured profiles (descriptors, dimension percentages, behaviour mappings, UI metadata) consumed by the MBTI API and SOUL.md rendering in ``persona``. |
| `media_generation.py` | 308 | Instance-wide media-generation settings for harness agents. |
| `memory_backend.py` | 87 | Resolve agent memory storage backend for harness-agent / harness-memory. |
| `persona.py` | 83 | Render agent persona system prompt from MBTI profiles or the built-in default template. |
| `plugin_tool_defaults.py` | 118 | Default-on semantics for plugin tools on agents. Harness ``build_plugin_tools`` historically required an explicit ``enabled: true`` in ``config_json.plugins``. Product expectation is the opposite: once a plugin is globally enabled, its tools are available unless the agent opts ou… |
| `plugin_tool_names.py` | 100 | Sanitize plugin tool names for strict LLM tool-name APIs. Plugin authors register tools with ``ctx.tool("中文名", fn, ...)``; the harness passes that name straight into the function-calling schema, but most LLM APIs only accept ``^[a-zA-Z0-9_-]{1,64}$``. Mirroring the MCP-side fix (… |
| `profile.py` | 164 | Agent profile fields stored on ``agents`` rather than in ``config_json``. ``config_json`` keeps harness-agent interaction keys (backend, plugins, memory, skills, heartbeat, runtime knobs). Display / catalog metadata lives on columns. |
| `runtime_limits.py` | 182 | Map agent ``config_json`` runtime knobs onto harness-agent semantics. Configurable stream contract (``ChatRequest.configurable``) ----------------------------------------------------------- ``max_iters`` → top-level ``recursion_limit`` (LangGraph) ``temperature``/``top_p``/``max_… |
| `thread_fork.py` | 289 | Fork a conversation thread from a selected assistant reply. Copies LangGraph checkpoint messages *through* that assistant turn into a new thread, leaving the source transcript unchanged. The client continues in the forked thread without prefilling the previous user question. |
| `tool_catalog.py` | 225 | Built-in agent tool catalog for tool-settings UI and disable policy. |
| `workspace_dir.py` | 385 | Agent ``workspace_dir`` (persisted in ``config_json``). ``workspace_dir`` and ``backend.root_dir`` are different dimensions: * ``root_dir`` — local backend rootfs (agent-visible ``/``). * ``workspace_dir`` — the agent's working directory. After create, the DB value is authoritati… |

### `src/octop/infra/agents/builtin_skills/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 77 | Octop-owned built-in Skills seeded into every agent workspace. |

### `src/octop/infra/agents/builtin_skills/skill-manager/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `manage_skills.py` | 683 | Safely inspect, install, list, remove, and restore workspace Skills. |

### `src/octop/infra/agents/experts/`

| File | Lines | Description |
|------|------:|-------------|
| `catalog.py` | 854 | Expert catalog — bundled "scene templates" inherited from finnie. An *expert* is metadata in ``manifest.json`` plus files on disk under ``library/<id>/``. At seed time files (including a copy of ``manifest.json``) are written into the agent workspace under ``.octop/manifest.json`… |
| `manifest_generator.py` | 665 | Generate SkillHub expert welcome metadata with an internal generator skill. |
| `market_creation.py` | 365 | Create agents from SkillHub-backed expert market templates. |
| `publish.py` | 290 | Export agent workspaces as installable published-expert snapshots. |
| `published_creation.py` | 382 | Publish / refresh / install / unpublish user expert templates. |
| `skillhub_market.py` | 1323 | SkillHub skillset marketplace integration for expert templates. SkillHub currently exposes expert-like assets as *skillsets*. A skillset is a workflow prompt plus a list of skill slugs. We normalize that package into the same on-disk shape as bundled experts: ``manifest.json`` + … |

### `src/octop/infra/agents/experts/library/clinical-learning-subscription/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `clinical_profile.py` | 2608 | Manage the minimal learning state for the clinical learning expert. The state stays beside this expert package. It is deliberately not a patient record and does not accept arbitrary file paths. V2 models a learning goal, versioned learning track, immutable lesson units, and a del… |
| `simulate_weixin_flow.py` | 238 | Simulate subscription creation and learning-delivery decisions. Routing model (v2): the platform ``cronjob_create`` tool binds the cron job to the *current* conversation session and delivers to whatever channel that session uses (WeChat / QQ / dashboard / CLI / …). There is no lo… |
| `validate_output.py` | 513 | 校验临床学习输出，在投递前检查格式、来源与边界声明。 |

### `src/octop/infra/agents/experts/library/cvm-ai-doctor/skills/cvm-ai-doctor/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `cluster_patrol_record.py` | 171 | Record one cluster health scan result into the JSONL history file. Reads cluster_score.sh JSON output from stdin (or --input file), injects timestamp and cluster_name, then appends exactly one JSONL line to ~/.lightclaw/stats/cluster-doctor.jsonl. Designed to run at the tail of e… |
| `cluster_patrol_server.py` | 226 | Cluster Doctor Patrol Server — lightweight HTTP server for cluster health data. Serves static files from the skill project root and exposes three read-only API endpoints backed by the JSONL history written by cluster_patrol_record.py. Endpoints: GET /api/cluster-data → full recor… |
| `generate_promo.py` | 53 | Octop Python module. |
| `patrol_server.py` | 139 | CVM Doctor Patrol Server ======================== 轻量 HTTP 服务： - 静态文件服务（docs/, scripts/ 等） - /api/patrol-data → 直接返回 JSONL 文件内容（JSON array） - /api/patrol-summary → 返回统计摘要 用法： python3 scripts/patrol_server.py # 默认 8765 端口 python3 scripts/patrol_server.py --port 9090 # 自定义端口 |

### `src/octop/infra/agents/experts/library/cvm-cluster-doctor/skills/tencentcloud-infra/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `tccli-oauth-helper.py` | 361 | tccli OAuth 登录辅助工具 解决 tccli auth login --browser no 在非交互式环境下无法输入验证码的问题。 用法: # 第一步: 生成授权链接 python3 tccli-oauth-helper.py --get-url # 第二步: 用户访问链接登录后，获取 base64 验证码，然后: python3 tccli-oauth-helper.py --code "验证码字符串" # 或者一步完成（如果已有验证码）: python3 tccli-oauth-helper.py --code "eyJhY2Nlc3NU… |

### `src/octop/infra/agents/experts/library/meituan-living-assistant/skills/meituan-deals/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `auth.py` | 224 | huisheng-coupon-tool 认证模块 管理 device_token（下单时的 uuid 参数）。Token 有效期由 pt-passport CLI 管理。 用法示例： python auth.py get-device-token python auth.py logout python auth.py clear-device-token |
| `diag_auth_log.py` | 111 | A4 鉴权操作日志诊断脚本 读取并解密 huisheng_auth.log，按接口分类展示最新一条记录。 加密方式与 auth.py 完全一致：sha256(device_token + aiScene)，降级 sha256(aiScene) |
| `diag_issue_log.py` | 78 | A5 发券接口日志诊断脚本 读取并解密 huisheng_issue.log，展示最新一条记录。 加密方式与 issue.py 完全一致：sha256(device_token + aiScene)，降级 sha256(aiScene) |
| `plugin_version_lite.py` | 308 | plugin_version_lite.py — 插件版本探测精简版（行为同 plugin_version.py，去注释/调试/明细）。 |
| `qr_local.py` | 70 | 本地二维码生成兜底模块 当美团服务端 getQrCodeImage 接口不可用/失败时，用 qrcode 库本地生成。 输出 JSON: { ok: true, type: "image", imageUrl: "data:image/png;base64,..." } { ok: true, type: "ascii", ascii: "<字符画二维码>" } { ok: false, error: "..." } 用法: python qr_local.py <url> [--ascii] |

### `src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `accept_changes.py` | 134 | Accept all tracked changes in a DOCX file using LibreOffice. Requires LibreOffice (soffice) to be installed. |
| `comment.py` | 317 | Add comments to DOCX documents. Usage: python comment.py unpacked/ 0 "Comment text" python comment.py unpacked/ 1 "Reply text" --parent 0 Text should be pre-escaped XML (e.g., &amp; for &, &#x2019; for smart quotes). After running, add markers to document.xml: <w:commentRangeStar… |

### `src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/`

| File | Lines | Description |
|------|------:|-------------|
| `pack.py` | 155 | Pack a directory into a DOCX, PPTX, or XLSX file. Validates with auto-repair, condenses XML formatting, and creates the Office file. Usage: python pack.py <input_directory> <output_file> [--original <file>] [--validate true\|false] Examples: python pack.py unpacked/ output.docx --… |
| `soffice.py` | 218 | Helper for running LibreOffice (soffice) in environments where AF_UNIX sockets may be blocked (e.g., sandboxed VMs). Detects the restriction at runtime and applies an LD_PRELOAD shim if needed. Usage: from office.soffice import run_soffice, get_soffice_cmd, get_soffice_env # Opti… |
| `unpack.py` | 131 | Unpack Office files (DOCX, PPTX, XLSX) for editing. Extracts the ZIP archive, pretty-prints XML files, and optionally: - Merges adjacent runs with identical formatting (DOCX only) - Simplifies adjacent tracked changes from same author (DOCX only) Usage: python unpack.py <office_f… |
| `validate.py` | 113 | Command line tool to validate Office document XML files against XSD schemas and tracked changes. Usage: python validate.py <path> [--original <original_file>] [--auto-repair] [--author NAME] The first argument can be either: - An unpacked directory containing the Office document … |

### `src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/helpers/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 0 | Octop Python module. |
| `merge_runs.py` | 192 | Merge adjacent runs with identical formatting in DOCX. Merges adjacent <w:r> elements that have identical <w:rPr> properties. Works on runs in paragraphs and inside tracked changes (<w:ins>, <w:del>). Also: - Removes rsid attributes from runs (revision metadata that doesn't affec… |
| `simplify_redlines.py` | 196 | Simplify tracked changes by merging adjacent w:ins or w:del elements. Merges adjacent <w:ins> elements from the same author into a single element. Same for <w:del> elements. This makes heavily-redlined documents easier to work with by reducing the number of tracked change wrapper… |

### `src/octop/infra/agents/experts/library/office-automation/skills/docx/scripts/office/validators/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 15 | Validation modules for Word document processing. |
| `base.py` | 797 | Base validator with common validation logic for document files. |
| `docx.py` | 418 | Validator for Word document XML files against XSD schemas. |
| `pptx.py` | 262 | Validator for PowerPoint presentation XML files against XSD schemas. |
| `redlining.py` | 240 | Validator for tracked changes in Word documents. |

### `src/octop/infra/agents/experts/library/office-automation/skills/pdf/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `check_bounding_boxes.py` | 71 | Symbols: RectAndField, get_bounding_box_messages |
| `check_fillable_fields.py` | 11 | Octop Python module. |
| `convert_pdf_to_images.py` | 31 | Symbols: convert |
| `create_validation_image.py` | 37 | Symbols: create_validation_image |
| `extract_form_field_info.py` | 128 | Symbols: get_full_annotation_field_id, make_field_dict, get_field_info, write_field_info |
| `extract_form_structure.py` | 116 | Extract form structure from a non-fillable PDF. This script analyzes the PDF to find: - Text labels with their exact coordinates - Horizontal lines (row boundaries) - Checkboxes (small rectangles) Output: A JSON file with the form structure that can be used to generate accurate f… |
| `fill_fillable_fields.py` | 102 | Symbols: fill_pdf_fields, validation_error_for_field_value, monkeypatch_pydpf_method |
| `fill_pdf_form_with_annotations.py` | 105 | Symbols: transform_from_image_coords, transform_from_pdf_coords, fill_pdf_form |

### `src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 0 | Octop Python module. |
| `add_slide.py` | 198 | Add a new slide to an unpacked PPTX directory. Usage: python add_slide.py <unpacked_dir> <source> The source can be: - A slide file (e.g., slide2.xml) - duplicates the slide - A layout file (e.g., slideLayout2.xml) - creates from layout Examples: python add_slide.py unpacked/ sli… |
| `clean.py` | 279 | Remove unreferenced files from an unpacked PPTX directory. Usage: python clean.py <unpacked_dir> Example: python clean.py unpacked/ This script removes: - Orphaned slides (not in sldIdLst) and their relationships - [trash] directory (unreferenced files) - Orphaned .rels files for… |
| `thumbnail.py` | 302 | Create thumbnail grids from PowerPoint presentation slides. Creates a grid layout of slide thumbnails for quick visual analysis. Labels each thumbnail with its XML filename (e.g., slide1.xml). Hidden slides are shown with a placeholder pattern. Usage: python thumbnail.py input.pp… |

### `src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/`

| File | Lines | Description |
|------|------:|-------------|
| `pack.py` | 155 | Pack a directory into a DOCX, PPTX, or XLSX file. Validates with auto-repair, condenses XML formatting, and creates the Office file. Usage: python pack.py <input_directory> <output_file> [--original <file>] [--validate true\|false] Examples: python pack.py unpacked/ output.docx --… |
| `soffice.py` | 216 | Helper for running LibreOffice (soffice) in environments where AF_UNIX sockets may be blocked (e.g., sandboxed VMs). Detects the restriction at runtime and applies an LD_PRELOAD shim if needed. Usage: from office.soffice import run_soffice, get_soffice_cmd, get_soffice_env # Opti… |
| `unpack.py` | 131 | Unpack Office files (DOCX, PPTX, XLSX) for editing. Extracts the ZIP archive, pretty-prints XML files, and optionally: - Merges adjacent runs with identical formatting (DOCX only) - Simplifies adjacent tracked changes from same author (DOCX only) Usage: python unpack.py <office_f… |
| `validate.py` | 113 | Command line tool to validate Office document XML files against XSD schemas and tracked changes. Usage: python validate.py <path> [--original <original_file>] [--auto-repair] [--author NAME] The first argument can be either: - An unpacked directory containing the Office document … |

### `src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/helpers/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 0 | Octop Python module. |
| `merge_runs.py` | 192 | Merge adjacent runs with identical formatting in DOCX. Merges adjacent <w:r> elements that have identical <w:rPr> properties. Works on runs in paragraphs and inside tracked changes (<w:ins>, <w:del>). Also: - Removes rsid attributes from runs (revision metadata that doesn't affec… |
| `simplify_redlines.py` | 196 | Simplify tracked changes by merging adjacent w:ins or w:del elements. Merges adjacent <w:ins> elements from the same author into a single element. Same for <w:del> elements. This makes heavily-redlined documents easier to work with by reducing the number of tracked change wrapper… |

### `src/octop/infra/agents/experts/library/office-automation/skills/pptx/scripts/office/validators/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 15 | Validation modules for Word document processing. |
| `base.py` | 797 | Base validator with common validation logic for document files. |
| `docx.py` | 418 | Validator for Word document XML files against XSD schemas. |
| `pptx.py` | 262 | Validator for PowerPoint presentation XML files against XSD schemas. |
| `redlining.py` | 240 | Validator for tracked changes in Word documents. |

### `src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `recalc.py` | 197 | Excel Formula Recalculation Script Recalculates all formulas in an Excel file using LibreOffice |

### `src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/`

| File | Lines | Description |
|------|------:|-------------|
| `pack.py` | 155 | Pack a directory into a DOCX, PPTX, or XLSX file. Validates with auto-repair, condenses XML formatting, and creates the Office file. Usage: python pack.py <input_directory> <output_file> [--original <file>] [--validate true\|false] Examples: python pack.py unpacked/ output.docx --… |
| `soffice.py` | 216 | Helper for running LibreOffice (soffice) in environments where AF_UNIX sockets may be blocked (e.g., sandboxed VMs). Detects the restriction at runtime and applies an LD_PRELOAD shim if needed. Usage: from office.soffice import run_soffice, get_soffice_cmd, get_soffice_env # Opti… |
| `unpack.py` | 131 | Unpack Office files (DOCX, PPTX, XLSX) for editing. Extracts the ZIP archive, pretty-prints XML files, and optionally: - Merges adjacent runs with identical formatting (DOCX only) - Simplifies adjacent tracked changes from same author (DOCX only) Usage: python unpack.py <office_f… |
| `validate.py` | 113 | Command line tool to validate Office document XML files against XSD schemas and tracked changes. Usage: python validate.py <path> [--original <original_file>] [--auto-repair] [--author NAME] The first argument can be either: - An unpacked directory containing the Office document … |

### `src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/helpers/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 0 | Octop Python module. |
| `merge_runs.py` | 192 | Merge adjacent runs with identical formatting in DOCX. Merges adjacent <w:r> elements that have identical <w:rPr> properties. Works on runs in paragraphs and inside tracked changes (<w:ins>, <w:del>). Also: - Removes rsid attributes from runs (revision metadata that doesn't affec… |
| `simplify_redlines.py` | 196 | Simplify tracked changes by merging adjacent w:ins or w:del elements. Merges adjacent <w:ins> elements from the same author into a single element. Same for <w:del> elements. This makes heavily-redlined documents easier to work with by reducing the number of tracked change wrapper… |

### `src/octop/infra/agents/experts/library/office-automation/skills/xlsx/scripts/office/validators/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 15 | Validation modules for Word document processing. |
| `base.py` | 797 | Base validator with common validation logic for document files. |
| `docx.py` | 418 | Validator for Word document XML files against XSD schemas. |
| `pptx.py` | 262 | Validator for PowerPoint presentation XML files against XSD schemas. |
| `redlining.py` | 240 | Validator for tracked changes in Word documents. |

### `src/octop/infra/agents/experts/library/wechat-ops/skills/publisher-multi-platform/scripts/`

| File | Lines | Description |
|------|------:|-------------|
| `wechat_publish.py` | 1080 | WeChat Publisher Personal — Publish Markdown to WeChat Official Account A self-contained CLI that wraps wenyan-cli for rendering and uses WeChat Official Account API for publishing articles to draft box. Supports multi-platform publishing (WeChat + XHS) via companion script. Usag… |
| `xhs_publish.py` | 592 | XHS (社交内容平台/RedNote) Publisher — Publish content to Xiaohongshu Based on ClawHub hi-yu/xhs skill v1.2.5 concept, implemented as a standalone browser-automation publisher for the personal multi-platform publishing workflow. Usage: python3 xhs_publish.py <command> [options] Command… |

### `src/octop/infra/agents/middleware/`

| File | Lines | Description |
|------|------:|-------------|
| `binary_read_guard.py` | 128 | Block ``read_file`` on binary inbound attachments (PDF, Office, …). |
| `browser_profile.py` | 71 | Force browser tool calls onto the current user's isolated profile. |
| `reasoning.py` | 50 | Apply per-turn provider-specific reasoning request parameters. |
| `thread_artifacts.py` | 339 | Record workspace file paths onto ``threads.artifacts`` after successful tool calls. |
| `token_quota.py` | 48 | Reject agent turns when the acting user has exhausted their token quota. |
| `workspace_image.py` | 88 | Ephemeral workspace-image rematerialization for model calls (plan B). Checkpoint / history keep path-only ``image_url`` refs (``workspace://…``). This middleware expands them to ``data:`` URIs on the model request only so base64 never lands in LangGraph state or the dashboard his… |

### `src/octop/infra/agents/plugins/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 5 | Octop host-side plugin management. |
| `manager.py` | 585 | Install, load, and expose plugins under ``~/.octop/plugins/``. |
| `seed.py` | 116 | Copy packaged plugins into the user plugins directory, globally disabled. |

### `src/octop/infra/agents/plugins/bundled/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 8 | In-package bundled plugins copied into ``~/.octop/plugins`` on init/start. |

### `src/octop/infra/agents/plugins/bundled/bilibili-anime/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 207 | Bilibili anime search + episode list for chat UI player. Uses public Bilibili HTTP APIs from the Octop server (avoids browser CORS). Playback in the Dashboard uses the official iframe player. |

### `src/octop/infra/agents/plugins/bundled/fortune/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 116 | Local dice, daily fortune, and lots — no network. |

### `src/octop/infra/agents/plugins/bundled/hot-topics/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 155 | Public hot lists: Weibo, Zhihu, Hacker News. |

### `src/octop/infra/agents/plugins/bundled/market-quotes/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 163 | Forex, crypto, and A-share quotes from public endpoints. |

### `src/octop/infra/agents/plugins/bundled/mini-games/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 106 | Tic-tac-toe and number guessing — state is passed in tool args. |

### `src/octop/infra/agents/plugins/bundled/parcel-tracker/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 105 | Domestic parcel lookup via kuaidi100 public pages, with a link fallback. |

### `src/octop/infra/agents/plugins/bundled/pomodoro/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 61 | Pomodoro and countdown cards — timing runs in the Dashboard UI. |

### `src/octop/infra/agents/plugins/bundled/qrcode/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 50 | Generate a QR code PNG data URL with segno. |

### `src/octop/infra/agents/plugins/bundled/server-status/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 159 | 服务器状态 — 采集本机 OS / CPU / 内存 / 磁盘快照，供聊天 UI 渲染。 |

### `src/octop/infra/agents/plugins/bundled/tetris/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 30 | Start an in-chat Tetris board. Gameplay runs in the Dashboard UI. |

### `src/octop/infra/agents/plugins/bundled/weather/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 215 | City weather via Open-Meteo (no API key). |

### `src/octop/infra/agents/providers/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 6 | DB-backed LLM provider catalog and harness factory sync. |
| `harness_factory.py` | 47 | Hot-sync DB provider configs into a shared ``HarnessAgentManager`` factory. |
| `model_flags.py` | 127 | Flags on provider ``models_json`` entries that affect chat / auto-routing. |
| `onnx_catalog.py` | 179 | Curated ONNX / fastembed embedding model catalog. |
| `onnx_download.py` | 457 | Race COS, Hugging Face, and hf-mirror, then download from the winner. |
| `onnx_service.py` | 601 | ONNX local embedding service: config, cache, and download lifecycle. This is **not** a chat Provider. It prepares local ONNX / fastembed embedding models under ``~/.octop/embedding_models`` for the Models admin local tab. |
| `opencode_session.py` | 64 | OpenCode Go gateway URL detection and probe session headers. The OpenCode Go gateway (``https://opencode.ai/zen/go``) rejects chat requests that lack ``x-opencode-session`` with ``HTTP 400 MissingSessionID``. Chat runtime injection is handled by harness-agent via ``ProviderConfig… |
| `presets.py` | 279 | Built-in provider template loading (harness-agent bundles). |
| `probe.py` | 327 | Provider connectivity probes (shared by API and CLI). |
| `reasoning.py` | 235 | Normalize model reasoning capabilities and build provider-specific overrides. |
| `resolved.py` | 49 | Resolved model list across enabled providers. |
| `store.py` | 310 | Read Octop DB provider rows and build harness ``ProviderConfig`` objects. |

### `src/octop/infra/agents/security/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 6 | Security policy persistence for the agent runtime. |
| `policy_store.py` | 53 | Global SecurityPolicy persistence in the settings table. |
| `tool_guard_rules.py` | 60 | Symbols: ToolGuardRulesStore |

### `src/octop/infra/agents/subagents/`

| File | Lines | Description |
|------|------:|-------------|
| `catalog.py` | 557 | Subagent catalog — bundled agency-agents definitions. Templates live on disk under ``library/<locale>/<division>/*.md`` and are discovered at server start by :class:`SubagentCatalog`. Slugs are derived from the file stem (not the ``name`` frontmatter field), which guarantees stab… |

### `src/octop/infra/auth/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Authentication domain helpers. |

### `src/octop/infra/auth/captcha/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 57 | Login captcha: provider registry, config, and verify orchestration. |
| `config.py` | 85 | Boot-time captcha env snapshot. |
| `providers.py` | 354 | Login captcha providers: contract, default implementation, registry. Four layers (execution lives in ``octop.infra.auth.captcha.verify``): 1. Contract — ``CaptchaProvider`` (Protocol): metadata plus ``verify_call()`` (token + credentials -> outbound request spec) and ``interpret(… |
| `store.py` | 357 | Settings blob + env snapshot → effective captcha config. |
| `verify.py` | 76 | Execution layer for login captcha: the only outbound-I/O seam. Providers stay pure (request spec + verdict); this module owns the HTTP call, timeout, error -> OctopError mapping, and the siteverify test seam. Ensures a login captcha token against the effective provider. |

### `src/octop/infra/auth/sso/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 5 | OpenID Connect single sign-on helpers. |
| `crypto.py` | 24 | Fernet encryption for OIDC single sign-on secrets. |
| `discovery.py` | 48 | OpenID Connect discovery document helpers. |
| `id_token.py` | 58 | OpenID Connect ID token verification. |
| `pkce.py` | 14 | PKCE helpers for OIDC authorization flows. |
| `public_base.py` | 41 | Public URL helpers for SSO callbacks (framework-free). |
| `redirect_after.py` | 14 | Safe post-login redirect path handling. |
| `service.py` | 496 | SSO service orchestration for OIDC and pluggable OAuth providers. |

### `src/octop/infra/auth/sso/providers/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 27 | Dashboard SSO identity-provider adapters. |
| `base.py` | 39 | Identity-provider adapter contract for dashboard SSO. |
| `dingtalk.py` | 138 | DingTalk enterprise-app web OAuth dashboard login adapter. |
| `feishu.py` | 159 | Feishu / Lark web OAuth dashboard login adapter. |
| `oidc.py` | 88 | OpenID Connect dashboard login adapter. |
| `wecom.py` | 182 | WeCom (企业微信) CorpApp web login adapter for dashboard SSO. |

### `src/octop/infra/backend/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 17 | Agent backend configuration — Octop DB rows → harness specs + probes. - :mod:`adapter` — ``storage_backends`` row → harness spec (no I/O) - :mod:`resolver` — agent config ``named`` / ``composite`` expansion - :mod:`probe` — admin connectivity checks (delegates round-trip to harne… |
| `adapter.py` | 185 | Map Octop ``storage_backends`` rows to harness-agent backend specs (no I/O). |
| `browse.py` | 66 | List directories on a configured storage backend (harness ``als``). |
| `docker_spec.py` | 91 | Octop helpers for harness ``type: "docker"`` backend specs. |
| `opensandbox_deps.py` | 37 | On-demand install of the official ``opensandbox`` SDK. |
| `probe.py` | 235 | Octop-specific storage backend probes (docker + row → harness probe). |
| `resolver.py` | 188 | Resolve agent ``backend`` config (named refs, composite routes). |

### `src/octop/infra/backup/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 33 | Backup and restore for Octop data. |
| `auto.py` | 290 | Automatic system backup scheduling (process-level CronManager job). |
| `chats.py` | 381 | Chat-history tables and workspace paths excluded from default backups. |
| `manifest.py` | 100 | Backup archive manifest. |
| `pg_dump.py` | 68 | PostgreSQL dump/restore via client tools on PATH. |
| `snapshot.py` | 349 | SQLite snapshot helpers. |
| `store.py` | 249 | On-disk backup archive store under ``PathLayout.backups_dir``. |
| `system_archive.py` | 657 | Full-system backup and restore (database + local agent workspaces + config). |
| `workspace_archive.py` | 174 | Zip export/import for a single agent workspace. |

### `src/octop/infra/browser/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 19 | Browser environment helpers (install/uninstall, profile prep). |
| `setup.py` | 531 | Browser environment setup: profile prep before Chrome launch, uninstall SSE. |

### `src/octop/infra/connectors/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Connector catalog, credential handling, and MCP config builders. |
| `builder.py` | 620 | Build harness HTTP MCP connection specs from connector instances. |
| `catalog.py` | 582 | Static connector catalog — bundled presets for HTTP MCP services. |
| `crypto.py` | 28 | Fernet encryption for connector credentials. |
| `custom_mcp.py` | 407 | User-defined MCP servers (streamable_http / stdio) stored as one connector doc. |
| `default_open.py` | 57 | Helpers for connector ``default_open`` (always inject tools when enabled). |
| `mail_servers.py` | 90 | Shared IMAP/SMTP host presets for mailbox connectors. |
| `mcp_tool_cache.py` | 94 | User-scoped MCP tool cache helpers (fingerprint + serialized tool wrappers). |
| `probe.py` | 552 | Connector credential probe — validate connectivity and list tools. |
| `service.py` | 637 | Connector service — MCP config assembly and credential access. |

### `src/octop/infra/connectors/gateway/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 11 | In-process MCP gateway for Octop-hosted connector adapters. |
| `cli_dirs.py` | 65 | Cleanup helpers for per-instance connector CLI config directories. |
| `cli_fingerprint.py` | 15 | Local CLI credential change-detection fingerprints (not auth storage). |
| `cli_install.py` | 258 | Install / detect host CLIs for Feishu & WeCom connector adapters. |
| `cli_runner.py` | 93 | Shared subprocess runner for official connector CLIs (lark-cli / wecom-cli). |
| `feishu_creds.py` | 104 | Materialize lark-cli config for headless Octop connector instances. |
| `feishu_user_auth.py` | 223 | Feishu CLI user identity via OAuth device-code (lark-cli auth login). |
| `langchain.py` | 78 | LangChain tool factory for in-process gateway MCP fallback. |
| `protocol.py` | 78 | MCP JSON-RPC protocol handlers for internal gateway endpoints. |
| `registry.py` | 77 | Gateway adapter registry — kind → list_tools / call_tool / probe. |
| `wecom_creds.py` | 156 | Materialize wecom-cli bot.enc + mcp_config.enc for headless Octop instances. |

### `src/octop/infra/connectors/gateway/adapters/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Per-vendor gateway MCP adapters. |
| `baidu_map.py` | 113 | Baidu Map Agent Plan gateway — Bearer Token (sk-ap-…). |
| `ctrip_wendao.py` | 79 | Ctrip Wendao (携程问道) gateway — paste Token from authorize page. |
| `feishu_cli.py` | 279 | Feishu / Lark official CLI gateway — category tools wrapping ``lark-cli``. |
| `fliggy.py` | 213 | Fliggy (飞猪) AI — signed remote MCP, natural-language search only. |
| `meituan_travel.py` | 98 | Meituan Travel Assistant gateway — hotel / flight / train / attraction NL query. |
| `qq_mail.py` | 211 | QQ mail and other IMAP/SMTP mailbox gateway. |
| `qq_music.py` | 145 | QQ Music Skills gateway — Bearer API Key (qmk-…). |
| `tencent_ima.py` | 508 | Tencent IMA notes and knowledge base gateway. |
| `tencent_news.py` | 101 | Tencent News search gateway — official OpenAPI with API Key. |
| `wechat_reading.py` | 90 | WeChat Reading (WeRead) bookshelf gateway. |
| `wecom_cli.py` | 212 | WeCom official CLI gateway — category tools wrapping ``wecom-cli``. |
| `weknora.py` | 239 | Read-only WeKnora knowledge-base gateway adapter. |
| `yuandian.py` | 221 | Yuandian (元典) Legal AI gateway — laws, cases, enterprises via Open API. |

### `src/octop/infra/connectors/oauth/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 33 | Connector OAuth flows — per-vendor modules + :mod:`registry` dispatcher. |
| `builtin.py` | 36 | Built-in OAuth client credentials shipped with Octop (override via env/settings). |
| `discovery.py` | 214 | Discover MCP OAuth issuers from a remote MCP URL (RFC 9728 + MCP authorization). |
| `mcp.py` | 247 | OAuth 2.0 for remote MCP servers via RFC 8414 + dynamic registration. Issuers and MCP URLs come from the connector catalog (``oauth_issuer`` + ``mcp_url`` on ``auth_kind=oauth2`` remote entries). Adding Notion / Ardot / Linear-style connectors is mostly a catalog row. |
| `pkce.py` | 14 | PKCE helpers shared by connector OAuth flows. |
| `registry.py` | 399 | Dispatch connector OAuth flows by kind. |

### `src/octop/infra/cron/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `delivery.py` | 314 | Cron delivery orchestration, separate from channel transport. |
| `job.py` | 116 | CronJob — APScheduler callable with status and audit bookkeeping. |
| `manager.py` | 310 | CronManager — process-wide singleton that owns all user-defined scheduled CronJob instances. |
| `task_type.py` | 53 | Cron job delivery mode — shared domain type (no DB / gateway imports). |
| `tools.py` | 264 | Built-in LangChain tools for agent-managed cron jobs. |
| `trigger.py` | 82 | Trigger string → APScheduler trigger. |

### `src/octop/infra/db/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `factory.py` | 44 | Open a database pool from process configuration. |
| `migrate.py` | 1713 | Apply numbered SQL migrations. Each file is ``NNN_description.sql`` (SQLite) or ``NNN_description.pg.sql`` (PostgreSQL). Version is stored in ``_schema_version``. |
| `pool.py` | 169 | Database pools: SQLite (default) and PostgreSQL. |
| `probe.py` | 49 | Probe a database DSN without replacing the process pool. |
| `rebind.py` | 150 | Hot-rebind the control-plane database pool during first-run setup. |
| `services.py` | 210 | Database services — RepoBundle and SharedServices DI container. |

### `src/octop/infra/db/migrations/`

| File | Lines | Description |
|------|------:|-------------|
| `001_initial.pg.sql` | 264 | DB migration 001_initial.pg. |
| `001_initial.sql` | 270 | DB migration 001_initial. |
| `002_cron_mcp_and_skill_packages.pg.sql` | 21 | DB migration 002_cron_mcp_and_skill_packages.pg. |
| `002_cron_mcp_and_skill_packages.sql` | 20 | DB migration 002_cron_mcp_and_skill_packages. |
| `003_repair_legacy_thread_titles.pg.sql` | 3 | DB migration 003_repair_legacy_thread_titles.pg. |
| `003_repair_legacy_thread_titles.sql` | 3 | DB migration 003_repair_legacy_thread_titles. |
| `004_thread_composer_preferences.pg.sql` | 6 | DB migration 004_thread_composer_preferences.pg. |
| `004_thread_composer_preferences.sql` | 6 | DB migration 004_thread_composer_preferences. |
| `005_shared_experts_sso_knowledge.pg.sql` | 100 | DB migration 005_shared_experts_sso_knowledge.pg. |
| `005_shared_experts_sso_knowledge.sql` | 147 | DB migration 005_shared_experts_sso_knowledge. |
| `006_user_permissions.pg.sql` | 3 | DB migration 006_user_permissions.pg. |
| `006_user_permissions.sql` | 3 | DB migration 006_user_permissions. |
| `007_resource_identity_and_profile.pg.sql` | 124 | DB migration 007_resource_identity_and_profile.pg. |
| `007_resource_identity_and_profile.sql` | 131 | DB migration 007_resource_identity_and_profile. |
| `008_usage_cache_tokens.pg.sql` | 14 | DB migration 008_usage_cache_tokens.pg. |
| `008_usage_cache_tokens.sql` | 14 | DB migration 008_usage_cache_tokens. |
| `009_user_invites.pg.sql` | 17 | DB migration 009_user_invites.pg. |
| `009_user_invites.sql` | 17 | DB migration 009_user_invites. |
| `010_thread_message_projection.pg.sql` | 39 | DB migration 010_thread_message_projection.pg. |
| `010_thread_message_projection.sql` | 36 | DB migration 010_thread_message_projection. |
| `011_cron_job_names.pg.sql` | 5 | DB migration 011_cron_job_names.pg. |
| `011_cron_job_names.sql` | 7 | DB migration 011_cron_job_names. |
| `012_trajectory_events.pg.sql` | 22 | DB migration 012_trajectory_events.pg. |
| `012_trajectory_events.sql` | 22 | DB migration 012_trajectory_events. |
| `013_connector_instances.pg.sql` | 23 | DB migration 013_connector_instances.pg. |
| `013_connector_instances.sql` | 52 | DB migration 013_connector_instances. |
| `014_user_policy.pg.sql` | 19 | DB migration 014_user_policy.pg. |
| `014_user_policy.sql` | 18 | DB migration 014_user_policy. |
| `015_sso_provider_kind.pg.sql` | 23 | DB migration 015_sso_provider_kind.pg. |
| `015_sso_provider_kind.sql` | 23 | DB migration 015_sso_provider_kind. |

### `src/octop/infra/db/repos/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `_base.py` | 81 | Helpers shared across repo implementations. |
| `agents.py` | 252 | Agent table access. |
| `audit.py` | 113 | Audit log table access. |
| `backends.py` | 152 | Storage backend table access. |
| `care_push.py` | 71 | Data-access layer for the proactive care push records table. The care_push_records table records the episode_id consumed by each proactive_care push, used to avoid re-pushing the same event within 30 days. |
| `channels.py` | 121 | Channel table access. |
| `connectors.py` | 291 | Connector database access. |
| `cron.py` | 270 | Cron jobs table access. |
| `invites.py` | 190 | User invite table access. |
| `knowledge.py` | 502 | Knowledge base metadata — bases and document rows. |
| `proactive_care_config.py` | 127 | Data-access layer for the proactive care push configuration table. The proactive_care_config table stores each agent's proactive care push configuration, including active hours, push interval, and other parameters. |
| `providers.py` | 148 | Provider table access. |
| `published_experts.py` | 153 | Published expert templates — one row per user-published expert. |
| `secrets.py` | 41 | Secrets table access. |
| `sessions.py` | 248 | Sessions table — maps session_key to the currently bound thread_id. |
| `settings.py` | 38 | Settings KV store — lightweight key/value pairs backed by SQLite. |
| `skill_packages.py` | 132 | Skill package metadata — one row per global skill package. |
| `sso.py` | 345 | SSO provider and login-state table access. |
| `thread_messages.py` | 317 | Projection rows used by the dashboard history API. |
| `threads.py` | 314 | Threads table — conversation metadata for history listing and /resume. |
| `trajectory_events.py` | 194 | Append-only chat trajectory event log. |
| `usage.py` | 501 | Token usage ledger access. |
| `user_policies.py` | 116 | Per-user policy rows — one named policy per user. |
| `users.py` | 358 | User table access. |
| `voice_providers.py` | 140 | Voice provider table access. |

### `src/octop/infra/desktop/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 5 | Remote desktop domain helpers. |
| `capture.py` | 266 | Screen capture and X11 display helpers for remote desktop. |
| `input.py` | 450 | Input injection, coordinates, and shortcut actions for remote desktop. |
| `session.py` | 129 | In-process remote desktop session registry. |
| `setup.py` | 1069 | Remote desktop environment setup, probes, paths, and installation. |

### `src/octop/infra/gateway/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Gateway — global AI interaction entry point. |
| `gateway.py` | 680 | Gateway — global AI interaction entry point. |
| `history_backfill.py` | 66 | Bounded single-worker queue for legacy checkpoint history projection. |
| `threads.py` | 438 | ThreadRegistry — session_key → thread_id binding backed by sessions + threads tables. |

### `src/octop/infra/gateway/bot_creators/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Bot creator subprocess scripts for QR-based channel registration. |
| `feishu_bot_creator.py` | 369 | Feishu / Lark Open Platform — scan-to-create app via lark-oapi. Uses ``lark_oapi.register_app`` (OAuth 2.0 Device Authorization Grant, RFC 8628). Requires ``lark-oapi>=1.5.5``. Usage: python -m octop.infra.gateway.bot_creators.feishu_bot_creator init python -m octop.infra.gateway… |
| `feishu_runner.py` | 123 | Feishu bot-creator subprocess runner (shared by API and CLI). |
| `yuanbao_bot_creator.py` | 481 | YuanBao Open Platform - Auto-bind bot via QR scan (non-interactive). Usage: python3 yuanbao_bot_creator.py create [instance_id] [ip] # Full flow: get scan_code -> scan -> emit finish python3 yuanbao_bot_creator.py cleanup # Clean up state files If instance_id / ip are omitted, th… |

### `src/octop/infra/gateway/channels/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Shared gateway channel helpers. |
| `dingtalk_registration.py` | 57 | DingTalk Device Flow helpers for one-click application registration. |
| `qr_bind.py` | 136 | WeCom / WeChat QR bind helpers (shared by API and CLI). |

### `src/octop/infra/gateway/cli/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 11 | CLI in-process chat transport: connection hub + virtual IM channel. |
| `cli_channel.py` | 176 | Virtual IM channel that delivers CLI chat turns in-process. |
| `turn.py` | 72 | CLI chat turn preparation (thread binding + InboundMessage). |

### `src/octop/infra/gateway/hitl/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 16 | Channel HITL — pending store, formatting, and resume orchestration. |
| `coordinator.py` | 466 | Orchestrate HITL pause/resume for IM channels. |
| `format.py` | 275 | Format HITL approval cards for IM channels. |
| `store.py` | 189 | In-memory registry of pending HITL approvals for IM channels. |

### `src/octop/infra/gateway/media/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 11 | Gateway media handling. Three cohesive concerns, one per submodule: - :mod:`.ingress` — ``AgentBackedMediaBackend``: the harness-gateway ``MediaBackend`` adapter that persists inbound IM attachments. - :mod:`.backend_files` — binary file I/O + dashboard media preview/URL resoluti… |
| `attachment_hints.py` | 632 | Agent-facing inbound attachment routing — vision bytes vs tool path hints. |
| `backend_files.py` | 565 | Binary file I/O and dashboard media preview through ``agent.workspace``. Dashboard / gateway code uses this module — **not** :class:`~octop.infra.gateway.media.ingress.AgentBackedMediaBackend`, which is only the harness-gateway ``MediaBackend`` adapter for IM ingress. Path rule f… |
| `constants.py` | 4 | Gateway media directory names under the agent workspace. |
| `inbound_store.py` | 451 | Inbound attachment storage — all ingress via :class:`BackendWorkspace`. |
| `ingress.py` | 49 | IM ingress media → agent workspace ``inbound/`` via :class:`BackendWorkspace`. |
| `tool_media.py` | 791 | Tool-result media for Dashboard WebSocket streaming. Uses ``agent.workspace`` via :mod:`.backend_files` — **not** :class:`~octop.infra.gateway.media.ingress.AgentBackedMediaBackend` (IM ingress). |

### `src/octop/infra/gateway/process/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 31 | Gateway message processing — GlobalProcessor and harness turn helpers. |
| `agent_resolve.py` | 40 | Resolve per-agent harness workspace handles for gateway I/O. |
| `harness_request.py` | 332 | Assemble harness-agent stream requests and multimodal user content. |
| `history_projection.py` | 240 | Capture the current turn from harness stream state for cheap UI history. |
| `message_keys.py` | 185 | Derive session keys and user ids from harness-gateway InboundMessage. |
| `processor.py` | 1537 | GlobalProcessor — harness-gateway MessageProcessor + TeamProcessor. |
| `response_mode.py` | 136 | Channel response delivery modes for external IM transports. |
| `stream_project.py` | 286 | Project harness stream chunks into harness-gateway MessageEvent objects. |
| `usage_record.py` | 276 | Collect per-call harness usage and append cache-aware turn totals. |

### `src/octop/infra/gateway/slash/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 18 | Slash command parsing and dispatch. |
| `catalog.py` | 292 | Slash command catalog — metadata for handlers, API, and dashboard UI. |
| `ctx.py` | 169 | SlashCtx and helpers for gateway slash handlers. |
| `dispatcher.py` | 83 | SlashDispatcher: gateway handlers + harness runtime delegation. |
| `formatting.py` | 63 | Format helpers for slash /status output. |
| `help.py` | 25 | Format /help output from catalog metadata. |
| `parser.py` | 20 | Slash command parsing that keeps filesystem paths out of the dispatcher. |
| `runner.py` | 46 | Shared slash command execution for IM processor and dashboard chat. |
| `runtime_bridge.py` | 72 | Bridge gateway SlashCtx to harness runtime slash context. |
| `types.py` | 17 | Gateway slash handler types. |

### `src/octop/infra/gateway/slash/handlers/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 26 | Gateway slash command handler registry. |
| `composite.py` | 339 | Composite slash commands (gateway + harness runtime data). |
| `hitl.py` | 28 | HITL slash command stubs (IM approval is handled in GlobalProcessor). |
| `platform.py` | 157 | Platform slash commands (help, agents, cron, connectors, token). |
| `session.py` | 164 | Session / thread slash commands. |

### `src/octop/infra/gateway/ws/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 14 | Dashboard WebSocket transport: connection hub + virtual IM channel. |
| `ws_channel.py` | 236 | Virtual IM channel that delivers Dashboard chat over WebSocket. |
| `ws_hub.py` | 162 | In-process registry of Dashboard WebSocket connections. |

### `src/octop/infra/history/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Versioned conversation history; legacy data is never migrated on read. |
| `legacy.py` | 30 | Read pinned legacy state without rewriting a projection or touching checkpoints. |
| `reader.py` | 113 | Read both formats without creating or repairing legacy records. |
| `recorder.py` | 425 | Capture state messages and stream-only content, including interrupted turns. |
| `service.py` | 272 | Turn-boundary routing and read composition for an unmigrated legacy prefix. |
| `store.py` | 273 | Independent SQLite archive. No control-plane schema migrations or backfill. Message and trajectory documents share content-addressed bodies. Replacing a live document releases only its own obsolete bodies, in the same transaction. |
| `trajectory.py` | 90 | Trajectory compatibility view over shared history bodies and untouched legacy rows. |

### `src/octop/infra/knowledge/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Knowledge-base domain services. |
| `chunk.py` | 21 | Fixed-size, overlapping character chunking for knowledge documents. |
| `citations.py` | 66 | Machine-readable knowledge citation payload embedded in tool results. |
| `default_open.py` | 63 | Knowledge-base selection defaults for a single chat turn. |
| `embed.py` | 65 | Route knowledge-base embeddings to local ONNX or a configured provider. |
| `files.py` | 37 | Filesystem layout and safe document persistence for knowledge bases. |
| `gate.py` | 144 | Capability gate for the optional knowledge-base feature. |
| `hint.py` | 119 | Per-turn enrichment of the search_knowledge tool description. |
| `index.py` | 134 | Per-knowledge-base SQLite sidecar vector index. |
| `jobs.py` | 86 | In-process document indexing jobs. |
| `ocr.py` | 302 | Optional OCR backends for knowledge-base images and scanned PDFs. |
| `params.py` | 115 | Instance-wide knowledge indexing and retrieval parameters. |
| `parse.py` | 291 | Text extraction for the document types accepted by knowledge bases. |
| `relpath.py` | 58 | Relative paths for knowledge-base folders and documents. |
| `retrieve.py` | 132 | Retrieve readable knowledge-base chunks for one chat turn. |
| `service.py` | 461 | Knowledge-base ownership checks and document upload orchestration. Visibility is owner or instance-wide ``shared``. |
| `tools.py` | 120 | Built-in LangChain tool for on-demand knowledge-base retrieval. |

### `src/octop/infra/mobile/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Remote Android infrastructure (probe, adb, setup). |
| `__main__.py` | 22 | CLI entry: ``python -m octop.infra.mobile <config.json>`` (install.sh hook). |
| `adb.py` | 682 | adb discovery, device listing, capture, and input helpers. |
| `agent_control.py` | 56 | In-process binding: which adb device the agent may control. Bound automatically for the duration of an active Remote Phone stream session (same idea as an open browser / desktop session). |
| `config_probe.py` | 56 | Persist mobile capability probe results into config.json. |
| `docker_install.py` | 387 | Automatic Docker install for the Android container backend. Design notes: - No geo detection: the install source is picked by a pure latency race that includes the official https://download.docker.com (fastest wins; when the official source wins, no ``DOWNLOAD_URL`` override is p… |
| `h264.py` | 97 | Annex-B H.264 helpers for adb screenrecord streams. |
| `probe.py` | 72 | Install-time host capability detection for Remote Android. |
| `setup.py` | 259 | Runtime mobile status and optional container install. |
| `tools.py` | 253 | Built-in LangChain tools for Remote Android (adb automation). |

### `src/octop/infra/proactive/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 18 | Public interface for the proactive module. |
| `picker.py` | 217 | Episode picker — selects the most care-worthy events from the user's episode memory. Scoring logic: score = intensity * emotion_weight * recency_weight - emotion_weight: negative emotions (sad/angry/anxious/frustrated) = 1.5, tired/reflective = 1.2, others (happy/excited/grateful… |
| `scheduler.py` | 396 | ProactiveCareScheduler — proactive care push random scheduler. Core idea: after each push completes, immediately schedule the next push time at random. Push times are randomly distributed within the user-configured active hours, rather than triggered at a fixed moment. Random sch… |
| `service.py` | 282 | ProactiveCareService — proactive care push service. Procedural flow (bypasses the LangGraph ReAct loop): 1. Read the last N days of episodes from Memory. 2. Query push records and filter out already-pushed episodes. 3. Use EpisodePicker to select the Top-3 most care-worthy episod… |

### `src/octop/infra/providers/`

| File | Lines | Description |
|------|------:|-------------|
| `codex_apply.py` | 67 | Apply Codex OAuth credentials to the Octop provider store. |
| `codex_oauth.py` | 284 | OpenAI Codex (ChatGPT) OAuth — device code flow. Ported from finnie/lightclaw; constants match openclaw wire contract. Tokens live at ``~/.octop/codex_oauth.json``. Octop is a server (not a local CLI), so it cannot use the PKCE browser redirect flow: that flow's shared client onl… |

### `src/octop/infra/setup/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | First-run setup wizard — password file and in-memory session tokens. |
| `password_file.py` | 81 | Setup wizard password file at ``<home>/octop-login.txt``. |
| `self_update.py` | 732 | Self-upgrade helpers shared by CLI and HTTP update API. |
| `service.py` | 925 | System service helpers for systemd (Linux) and launchd (macOS). |
| `wizard_tokens.py` | 67 | In-memory wizard session tokens (post password verification). |

### `src/octop/infra/setup/tls/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 7 | TLS / Let's Encrypt certificate management. |
| `acme_issue.py` | 135 | ACME v2 HTTP-01 certificate issuance (Let's Encrypt). |
| `challenge.py` | 28 | In-memory HTTP-01 challenge responses for ACME. |
| `http_companion.py` | 43 | Minimal HTTP listener on port 80 — ACME challenges + redirect to HTTPS. |
| `listeners.py` | 62 | Compute which HTTP/HTTPS ports ``octop run`` should bind. |
| `manager.py` | 257 | Orchestrate TLS preflight, ACME issuance, and config updates. |
| `modes.py` | 54 | TLS issuance mode helpers (first issue vs renewal). |
| `preflight.py` | 215 | Pre-flight checks before Let's Encrypt issuance. |
| `renewal.py` | 88 | Automatic TLS certificate renewal scheduling. |
| `store.py` | 95 | Persist TLS certificates and merge config.json. |

### `src/octop/infra/skills/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 41 | Skills domain: package validation, global packages, URL import, SkillHub market. |
| `install.py` | 221 | Shared skill install pipeline for agent workspace and global packages. |
| `presentation.py` | 95 | Skill presentation metadata shared by catalogs and HTTP adapters. |
| `skill_package_from_skillhub.py` | 83 | Materialize a SkillHub skillset into a global skill package. |
| `skill_package_store.py` | 194 | Disk-backed content storage for global skill packages. |
| `skill_packages.py` | 209 | Source-neutral skill package validation and workspace path normalization. |
| `skill_transfer.py` | 171 | Copy skills between global packages and agent workspaces. |
| `skillhub_market.py` | 440 | Direct HTTP client for Tencent SkillHub marketplace operations. |
| `skills_hub.py` | 1493 | Skills hub URL fetch and bundle normalization for per-agent workspace import. |
| `workspace_catalog.py` | 176 | Symlink-tolerant workspace skill discovery for Octop. |

### `src/octop/infra/trajectory/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 6 | Harness stream → trajectory event projection. |
| `live.py` | 31 | In-process pub/sub for trajectory live SSE fan-out. |
| `metrics.py` | 108 | Aggregate session metrics from trajectory events. |
| `projector.py` | 320 | Symbols: _Common, project_harness_chunk |
| `service.py` | 409 | Orchestrate project → store → live publish. Observe never fails the caller. |
| `settings.py` | 79 | Per-agent trajectory switch and persist-size limits. ``config_json.enable_trajectory`` defaults to enabled. Only the explicit boolean ``false`` disables observation and writes. |
| `store.py` | 54 | Append-only trajectory ledger — thin wrapper around TrajectoryEventRepo. |
| `turn_context.py` | 128 | Synthesize SYSTEM / CONTEXT chunks from harness injection sources of truth. |
| `types.py` | 29 | Symbols: TrajectoryEvent |

### `src/octop/infra/usage/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Token usage package — export helpers live beside ledger access. |
| `xlsx_export.py` | 563 | Build localized Excel workbooks for token usage export. Formatting follows the MiniMax XLSX skill conventions adapted for openpyxl (runtime API export): bold headers, thousands separators, freeze panes, auto-filter, TOTAL rows as Excel formulas (blank spacer above so Excel Sort/F… |

### `src/octop/infra/users/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 9 | User domain. Contents: identity — ``User`` / ``Role`` / ``UserToken`` dataclasses manager — ``UserManager`` lifecycle (bootstrap admin, create/disable) password — argon2id password hashing |
| `acting.py` | 42 | Resolve which user id a local CLI / API ``as_user`` action targets. |
| `email.py` | 34 | Email normalization and light validation for local users. |
| `identity.py` | 29 | User identity primitives. |
| `invites.py` | 223 | One-time user invite codes — create, validate, redeem. |
| `manager.py` | 737 | UserManager — process-singleton user lifecycle. |
| `password.py` | 70 | Argon2id password hashing and change-password policy checks. |
| `permissions.py` | 255 | Module-level permission catalog and checks. A permission is a module key (e.g. ``"browser"``, ``"users"``). Possessing a key grants access to that module's management page and write/configure actions. Read access and agent use in chat are never gated. ``admin`` bypasses all. Cate… |
| `preferences.py` | 154 | Per-user JSON preferences (models, remote-browser bookmarks, etc.). |
| `resource_policy.py` | 158 | Per-user named policies: workspace root, token quota, and future rows. |

### `src/octop/infra/utils/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `browser_media.py` | 154 | harness-browser media paths aligned with IM ``outbound/`` layout. |
| `bwrap.py` | 215 | Best-effort bubblewrap (``bwrap``) provisioning for scoped ``root_dir``. Used when saving agent backends with a non-host-root ``root_dir``. Install failure is never fatal — harness only constructs ``BubbledLocalShellBackend`` when Linux + ``virtual_mode`` + non-host root + ``bwra… |
| `doc_edit.py` | 335 | Editable-document converters (binary document <-> Markdown) for the workspace. The workspace online editor renders an editable document as Markdown in Monaco and saves back through ``PUT /workspace/doc``. Every supported extension registers a :class:`DocConverter` here; adding a … |
| `docker_env.py` | 411 | Detect / best-effort provision Docker Engine for agent sandbox backends. Mirrors :mod:`octop.infra.utils.bwrap`: never raise for missing privileges; return a status payload the dashboard can show with install script / agent prompt. |
| `env_file.py` | 146 | Persisted environment variables at ``~/.octop/env`` (dotenv format). |
| `frontmatter.py` | 92 | YAML frontmatter helpers for markdown workspace files. |
| `host_dirs.py` | 373 | Host filesystem directory helpers for dashboard root_dir pickers. |
| `json_file.py` | 109 | Safe read/write helpers for the small JSON config files under ``OCTOP_HOME``. Leaf module (AGENTS.md §5): stdlib only, no ``infra`` imports. Callers translate :class:`JsonFileCorruptError` into ``OctopError`` via ``octop.infra.errors.corrupt_config_error``. Both helpers exist bec… |
| `llm_text.py` | 90 | Shared helpers for one-shot LangChain LLM calls. Used by chat polish, proactive care (memory), and SkillHub expert manifest generation so text extraction and invoke timing stay consistent. |
| `locale.py` | 83 | Shared locale normalization for API, slash commands, and dashboard. |
| `ollama_manager.py` | 222 | Symbols: OllamaModelInfo |
| `paths.py` | 150 | Filesystem layout for ``~/.octop/``. |
| `posix_compat.py` | 118 | POSIX-only stdlib helpers for cross-platform mypy (CI runs on Windows). Runtime callers must still guard with ``os.name == "posix"`` (or :func:`octop.api.routers.terminal.terminal_supported`) before invoking PTY/fcntl paths. This module exists so Windows mypy does not require ``f… |
| `runtime_packages.py` | 284 | Install optional Python packages into the active interpreter at runtime. Octop deployments vary: uv venvs often omit ``pip``, systemd services may not have ``uv`` on ``PATH``, and some hosts only ship the stdlib. Callers should use :func:`install_packages` / :func:`install_packag… |
| `search_probe.py` | 221 | Probe web-search provider API keys via direct HTTP (no env mutation). Used by ``POST /api/search/{provider_id}/test``. Credentials come from the request body and are never written to ``os.environ``. |
| `ssl_errors.py` | 18 | Detect TLS/SSL failures in exception / stderr text for actionable UX hints. |
| `ssrf_guard.py` | 185 | Outbound HTTPS URL validation — mitigates SSRF (CWE-918). |
| `subprocess_io.py` | 49 | Cross-platform non-blocking reads from subprocess stdout pipes. |
| `subprocess_io_win.py` | 26 | Windows-only non-blocking pipe reads (typed separately for mypy). |
| `tencent_sign.py` | 60 | Tencent Cloud TC3-HMAC-SHA256 request signing. |
| `ulid.py` | 44 | Tiny ULID generator (Crockford base32, monotonic within process). |
| `url.py` | 14 | URL helpers shared across infra and API layers. |
| `utf8_text.py` | 156 | Helpers for coercing third-party package bytes into valid UTF-8 text. |

### `src/octop/infra/voice/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Voice STT/TTS provider orchestration. |
| `adapters.py` | 571 | Voice adapter implementations. |
| `manager.py` | 222 | Resolve active voice providers and dispatch STT/TTS calls. |
| `presets.py` | 73 | Built-in voice provider presets. |

### `tests/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `conftest.py` | 101 | Shared pytest fixtures. |

### `tests/fixtures/plugins/echo-tool/`

| File | Lines | Description |
|------|------:|-------------|
| `main.py` | 24 | Echo tool plugin for tests. |

### `tests/integration/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `conftest.py` | 246 | Shared fixtures for integration tests. |
| `test_acp_api.py` | 72 | tests/integration/test_acp_api.py — ACP runner configuration API. |
| `test_admin_providers.py` | 60 | tests/integration/test_admin_providers.py — admin /api/admin/providers CRUD. |
| `test_agent_avatars.py` | 60 | Expert avatar upload HTTP surface. |
| `test_agent_skill_packages.py` | 152 | Integration coverage for agent-mounted global skill packages. |
| `test_agents_api.py` | 154 | tests/integration/test_agents_api.py |
| `test_agents_list_scope.py` | 129 | Agent list user isolation. |
| `test_agents_shared.py` | 246 | Shared agent list and ownership behavior. |
| `test_auth_flow.py` | 124 | tests/integration/test_auth_flow.py |
| `test_auth_oauth.py` | 124 | OAuth SSO HTTP route integration tests. |
| `test_auth_oidc.py` | 134 | OIDC HTTP route integration tests. |
| `test_boundary_authz.py` | 129 | tests/integration/test_boundary_authz.py — cross-user / cross-scope authz. P0.3: gap-fill the boundary cases that were missing from phases 12.x. Covers: - Admin ``?as_user=`` agents/cron/channels access against another user. - Non-admin sending ``?as_user=`` is rejected with 403.… |
| `test_browser_api.py` | 240 | tests/integration/test_browser_api.py — remote browser endpoints. We deliberately don't spawn a real Chrome in the test suite. These cases drive env-status, ``_probe_env``, install SSE, harness-sessions, and shutdown gates. Live screencast is covered by dashboard + WS unit tests. |
| `test_browser_record_replay_api.py` | 232 | Symbols: test_record_replay_status_returns_daemon_status, test_record_replay_start_ensures_daemon_and_starts_recording, test_record_replay_start_ignores_requested_profile, test_record_replay_start_returns_503_when_daemon_fails, test_record_replay_stop_generates_steps, test_record_replay_start_ignores_requested_agent_profile |
| `test_bwrap_jail.py` | 288 | Scoped ``root_dir`` backend alignment and bubblewrap jail tests. ``BubbledLocalShellBackend`` is constructed only when Linux + ``virtual_mode`` + non-host ``root_dir`` + ``bwrap`` are available (factory routes before any backend I/O). Otherwise ``HarnessLocalShellBackend`` runs o… |
| `test_captcha_api.py` | 350 | HTTP contracts for login captcha and admin settings. |
| `test_channel_probe_draft.py` | 30 | Integration tests for draft channel config probe. |
| `test_channel_test_endpoint.py` | 45 | tests/integration/test_channel_test_endpoint.py — channel probe. |
| `test_channels_api.py` | 128 | tests/integration/test_channels_api.py — channels CRUD. Plan §12.5 mandates this file. Covers list/create/get/patch/delete cycle, 404 on missing channel, cross-user isolation, and runtime reload trigger on mutations. |
| `test_chat_ws.py` | 511 | tests/integration/test_chat_ws.py — dashboard WebSocket chat + thread CRUD. |
| `test_connectors_api.py` | 667 | Integration tests for connector APIs. |
| `test_cron_api.py` | 266 | tests/integration/test_cron_api.py — cron CRUD + run-now + trigger validation. Plan §12.6 mandates this file. Covers list/create/get/patch/delete cycle, trigger validation surfacing CRON_TRIGGER_INVALID at create + patch, and run-now invoking the cron manager and updating last_st… |
| `test_cron_shared_agent.py` | 84 | Cron jobs on a shared expert stay owner-only. |
| `test_dashboard_serve.py` | 71 | tests/integration/test_dashboard_serve.py |
| `test_e2e_golden_path.py` | 205 | tests/integration/test_e2e_golden_path.py — full user journey. |
| `test_envs_api.py` | 90 | Integration tests for /api/envs. |
| `test_experts_api.py` | 441 | tests/integration/test_experts_api.py — expert catalog endpoints. Covers list / detail / one-click create. Uses the bundled library that ships with the package (``src/octop/experts/library/``) so the data shape matches production exactly. |
| `test_filesystem_api.py` | 362 | Integration tests for /api/filesystem (host root_dir pickers). |
| `test_invites_api.py` | 128 | Integration tests for one-time user invites. |
| `test_knowledge_bases_api.py` | 50 | HTTP ACL for knowledge-base instance settings vs page CRUD. |
| `test_media_generation_api.py` | 72 | Integration tests for the admin media-generation settings API. |
| `test_memory_api.py` | 470 | tests/integration/test_memory_api.py — dashboard memory router. End-to-end smoke: build a real OctopServer with a main agent, seed a small graph (entity / candidate / atom / episode / journal) directly into the agent memory SQLite (``.octop/memory.sqlite`` for new agents) via ``h… |
| `test_notifications_ws.py` | 69 | Dashboard notification WebSocket for text-type pushes. |
| `test_onnx_models_api.py` | 144 | HTTP behaviour for the local ONNX embedding service admin endpoints. |
| `test_personas_admin_api.py` | 137 | tests/integration/test_personas_admin_api.py — MBTI preview + admin routers. Covers: - ``GET /api/mbti/types`` returns 16 MBTI profiles. - ``GET /api/mbti/preview/{code}`` renders preview with the current user. - ``GET /api/admin/overview`` includes users + agents + totals. - ``G… |
| `test_plugin_tool_disable.py` | 231 | Disable plugin tools via Admin Plugins API and Experts tool-settings. |
| `test_plugin_upload.py` | 79 | Integration tests for uploading a plugin ZIP from the dashboard. |
| `test_postgresql_checkpoint_history.py` | 62 | Check the native PG history path without opening a SQLite connection. |
| `test_postgresql_control_plane.py` | 342 | Gated PostgreSQL control-plane integration tests. Set ``OCTOP_TEST_DATABASE_URL`` to a **dedicated** database (tests DROP SCHEMA public). Example:: export OCTOP_TEST_DATABASE_URL='postgresql://postgres:postgres@127.0.0.1:15432/octop_test' Avoid pointing at a shared app DB (locks … |
| `test_preferences_api.py` | 88 | tests/integration/test_preferences_api.py |
| `test_provider_fetch_models.py` | 67 | Integration tests for admin provider fetch-models endpoint. |
| `test_provider_test_draft.py` | 69 | Integration tests for admin provider test-draft endpoint. |
| `test_provider_test_endpoint.py` | 74 | tests/integration/test_provider_test_endpoint.py — provider probe. |
| `test_providers_api.py` | 92 | tests/integration/test_providers_api.py |
| `test_published_experts.py` | 408 | Published expert HTTP API integration coverage. |
| `test_scalar.py` | 32 | Integration tests for the /api/docs Scalar endpoint. |
| `test_search_api.py` | 51 | Integration tests for POST /api/search/{provider_id}/test. |
| `test_security_api.py` | 77 | Integration tests for /api/admin/security. |
| `test_server_lifecycle.py` | 65 | tests/integration/test_server_lifecycle.py |
| `test_setup_bootstrap.py` | 136 | tests/integration/test_setup_bootstrap.py — initial-admin path. |
| `test_setup_database.py` | 156 | Setup wizard database step API tests. |
| `test_setup_wizard.py` | 430 | Integration tests for the 4-step wizard backend. Covers: verify-password, token-protected initial-admin, finish endpoint, and the 410 behavior on completed setups. |
| `test_skill_packages_api.py` | 530 | Integration coverage for global skill package HTTP routes. |
| `test_skills_api.py` | 956 | tests/integration/test_skills_api.py — per-agent skill library. Requires a running harness agent; skills are read/written via ``agent.workspace`` (``local_shell`` on the agent workspace dir). |
| `test_skills_hub.py` | 318 | Integration tests for GET /api/agents/{id}/skills/hub/search and POST /api/agents/{id}/skills/hub/install. |
| `test_subagents_api.py` | 221 | Integration tests for GET /api/agents/{id}/subagents and the catalog. |
| `test_terminal_context.py` | 67 | Integration tests for GET /api/agents/{agent_id}/terminal/context. |
| `test_terminal_ws.py` | 65 | tests/integration/test_terminal_ws.py — PTY WebSocket terminal. The PTY-spawning happy path is best exercised against the live socket because asgi-transport doesn't ship a WebSocket that can drive a real ``pty.openpty()`` cleanly. We therefore stick to the *gates* we own: unauthe… |
| `test_tool_settings_api.py` | 69 | Integration tests for GET/PUT /api/agents/{id}/tool-settings. |
| `test_trajectory_api.py` | 536 | HTTP trajectory history, event detail, metrics, export, and live SSE. |
| `test_update_api.py` | 45 | Integration tests for /api/update. |
| `test_upload_api.py` | 48 | Integration tests for dashboard chat attachments in agent workspace. |
| `test_usage_api.py` | 704 | tests/integration/test_usage_api.py — token usage ledger + summary endpoint. Three layers: 1. UsageRepo round-trips rows and aggregates per granularity. 2. /api/usage/summary respects ?as_user= scope (admin) and pins non-admins to their own user_id. 3. /api/admin/usage/summary re… |
| `test_users_api.py` | 247 | tests/integration/test_users_api.py |
| `test_voice_probe_http.py` | 76 | HTTP glue for the live STT probe: provider errors surface as ok:false. |
| `test_workspace_api.py` | 530 | tests/integration/test_workspace_api.py — workspace endpoints. Requires a running harness agent (``env_with_agent``); workspace I/O goes through ``agent.workspace`` backed by ``local_shell`` on the agent dir. |

### `tests/live/`

| File | Lines | Description |
|------|------:|-------------|
| `conftest.py` | 133 | Fixtures for live tests that call real LLM endpoints. |
| `test_agent_expert_template_live.py` | 251 | Live tests for expert template file copy via :class:`AgentManager`. Uses bundled templates under ``src/octop/infra/agents/experts/library/`` (exported expert packs). Requires ``.env`` with ``OPENAI_*`` for agent boot. Run:: uv run pytest tests/live/test_agent_expert_template_live… |
| `test_agent_manager_live.py` | 134 | Live integration tests for :class:`octop.infra.agents.manager.AgentManager`. Requires ``OPENAI_API_KEY``, ``OPENAI_BASE_URL``, and ``OPENAI_MODEL_NAME`` in the repo-root ``.env``. Skipped by default — run with:: uv run pytest tests/live/test_agent_manager_live.py -m live -v |
| `test_channel_probe.py` | 87 | Live channel credential probes (WeChat iLink + Feishu). Exercises the same ``ChannelManager.probe_channel`` path the dashboard uses via ``Gateway.probe_config`` / ``POST …/channels/test`` — so a green result means the real app credentials reach the real upstream. Credentials come… |
| `test_connector_probe.py` | 96 | Live connectivity probes for bundled token / api_key connectors. Each connector is probed with REAL credentials through the very same :func:`octop.infra.connectors.probe.probe_connector` the dashboard calls when a user adds / verifies a connector — so a green probe means the real… |
| `test_oss_storage.py` | 110 | Live round-trip tests against real object-storage backends (COS / S3 / OSS / OBS). These exercise the *real* product code path: * :func:`octop.infra.backend.probe.probe_storage_backend` is exactly what the dashboard calls when a user adds / verifies a storage backend — so a green… |

### `tests/support/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Shared test helpers (fakes, HTTP app bootstrap, auth). |
| `app.py` | 75 | OctopServer + ASGI client lifecycle for integration tests. |
| `auth.py` | 195 | Auth and bootstrap helpers for HTTP integration tests. |
| `bwrap_marks.py` | 21 | Shared pytest markers for bubblewrap / scoped-root_dir tests. |
| `fakes.py` | 488 | Reusable test doubles for octop. |
| `harness.py` | 192 | Patch harness-agent so tests run without a real LLM. |
| `http.py` | 164 | WebSocket / streaming helpers for integration tests. |
| `postgresql.py` | 17 | Helpers for gated live-PostgreSQL tests. Prefer a dedicated database (tests may ``DROP SCHEMA public CASCADE``):: export OCTOP_TEST_DATABASE_URL='postgresql://postgres:postgres@127.0.0.1:15432/octop_test' |
| `scenarios.py` | 55 | Multi-step bootstrap scenarios for integration tests. |
| `secrets.py` | 34 | Helpers for live tests that need real credentials. The repo-root ``.env`` is loaded by ``tests/live/conftest.py`` via python-dotenv, so these helpers read from ``os.environ`` directly — the same code path used both locally (``.env``) and in CI (GitHub Secrets mapped to env vars).… |
| `testmon_staged_changes.py` | 55 | Make pytest-testmon see *staged* changes (required for the git pre-commit hook). testmon decides which files changed by calling ``git ls-files --stage -m``, which compares the worktree against the **index**. A git pre-commit hook runs after ``git add``, so the changes are already… |

### `tests/unit/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 1 | Octop Python module. |
| `test_bundled_plugins_layout.py` | 70 | Bundled plugin tree must be complete enough to seed and render UI. |
| `test_cli_main.py` | 186 | tests/unit/test_cli_main.py — basic help/import smoke test. |
| `test_cli_state.py` | 31 | tests/unit/test_cli_state.py |
| `test_config.py` | 432 | tests/unit/test_config.py |
| `test_connectors.py` | 1423 | Unit tests for connector builder and repo. |
| `test_docker_compose_database_env.py` | 29 | Compose must forward OCTOP_DATABASE_* into the container (issue #152). |
| `test_env_file.py` | 84 | tests/unit/test_env_file.py |
| `test_errors.py` | 37 | tests/unit/test_errors.py |
| `test_green_launch.py` | 56 | Tests for the green portable launch bootstrap. |
| `test_host_download_allow.py` | 61 | Host absolute path helpers for workspace download / preview. |
| `test_host_paths.py` | 53 | tests/unit/test_host_paths.py |
| `test_hot_topics_plugin.py` | 90 | Hot-topics plugin must use unauthenticated public endpoints. |
| `test_identity.py` | 26 | tests/unit/test_identity.py |
| `test_langfuse_settings.py` | 93 | Unit tests for Langfuse settings store. |
| `test_launch_bwrap.py` | 68 | Tests for non-blocking bubblewrap provisioning in ``octop run``. |
| `test_launch_deferred.py` | 24 | Launch composition root must tolerate deferred control-plane bind. |
| `test_logging.py` | 294 | tests/unit/test_logging.py Covers the daily-rotation + retention logging strategy in ``octop.infra.server``: the log path lives under ``~/.octop/logs`` and stale rotated files are purged. |
| `test_metrics.py` | 28 | tests/unit/test_metrics.py |
| `test_password.py` | 69 | tests/unit/test_password.py |
| `test_paths.py` | 70 | tests/unit/test_paths.py |
| `test_plugin_manager.py` | 326 | Unit tests for Octop plugin manager. |
| `test_plugin_seed.py` | 192 | Unit tests for bundled plugin seeding (copy + globally disabled). |
| `test_plugins.py` | 213 | Unit tests for harness-agent plugin loader. |
| `test_provider_fetch_models.py` | 176 | Unit tests for OpenAI-compatible remote model listing. |
| `test_provider_preset_expansion.py` | 106 | Unit tests for harness-backed provider presets. |
| `test_provider_probe.py` | 273 | Unit tests for provider connectivity probe helpers. |
| `test_release_download_links.py` | 44 | tests/unit/test_release_download_links.py |
| `test_security_settings.py` | 33 | Tests for security settings store. |
| `test_shared.py` | 28 | tests/unit/test_shared.py |
| `test_skillhub_install_metadata.py` | 71 | Symbols: test_summary_prefers_octop_display_metadata_and_supports_openclaw_emoji, test_with_skillhub_metadata_preserves_skill_identity_and_body, test_skillhub_metadata_writes_localized_label_and_summary, test_skillhub_icon_url_allows_only_http_urls |
| `test_skills_hub_errors.py` | 116 | Unit tests for SkillHub install error mapping and CLI subprocess helpers. |
| `test_storage_browse.py` | 141 | Unit tests for storage backend browse helpers. |
| `test_storage_probe.py` | 200 | Unit tests for storage backend probes. |
| `test_storage_spec.py` | 235 | Tests for storage backend spec resolution. |
| `test_tool_guard_rules_store.py` | 49 | Tests for user-editable tool guard rules store. |
| `test_ulid.py` | 39 | tests/unit/test_ulid.py |
| `test_usage_thread_totals.py` | 68 | tests/unit/test_usage_thread_totals.py |
| `test_user_locale.py` | 80 | tests/unit/test_user_locale.py |
| `test_user_manager.py` | 472 | tests/unit/test_user_manager.py |
| `test_voice_formats.py` | 43 | STT upload MIME → provider format mapping (Tencent / Mimo). |
| `test_voice_manager.py` | 90 | Unit tests for VoiceManager. |
| `test_voice_probe.py` | 167 | Voice probe (test_stt/test_tts) behavior per provider kind. Success paths must stay untouched; failure paths must return ``{"ok": false, "error": ...}`` instead of raising into a 500. |
| `test_voice_stt_probe.py` | 139 | Live STT probe: test_stt performs a real recognition call per provider. Credential/config failures and provider errors must be reported as ``{"ok": false, "error": ...}``, never raised into a 500. |
| `test_wizard_password.py` | 79 | Unit tests for the wizard password file lifecycle. |
| `test_wizard_token.py` | 91 | Unit tests for the in-memory wizard token store + rate limiter. |
| `test_workspace_download.py` | 50 | tests/unit/test_workspace_download.py |

### `tests/unit/agents/`

| File | Lines | Description |
|------|------:|-------------|
| `test_acp_settings.py` | 101 | tests/unit/test_acp_settings.py — user-scoped global ACP runner settings. |
| `test_agent_avatar.py` | 173 | Expert avatar file helpers. |
| `test_agent_manager.py` | 1541 | Unit tests for :mod:`octop.infra.agents.manager` internals. |
| `test_agent_registry.py` | 1194 | tests/unit/test_agent_registry.py Unit tests for AgentManager. All harness I/O is patched out so no real LLM processes are started. |
| `test_agent_skill_packages.py` | 398 | Unit tests for agent skill package mounts. |
| `test_agent_tools.py` | 31 | tests/unit/test_agent_tools.py |
| `test_binary_read_guard.py` | 20 | Tests for BinaryReadGuardMiddleware. |
| `test_browser_profile_middleware.py` | 78 | Symbols: test_browser_profile_is_forced_from_current_user, test_non_browser_tool_is_unchanged, test_browser_profile_blocks_when_user_missing, test_browser_profile_blocks_placeholder_user_zero |
| `test_clinical_learning_subscription_template.py` | 767 | Regression tests for the clinical learning + general assistant template. |
| `test_context_breakdown.py` | 242 | Tests for Octop context-usage adapter over harness-agent. |
| `test_custom_agent_id.py` | 94 | Unit tests for user-supplied custom agent ids on expert creation. |
| `test_default_agent.py` | 100 | Unit tests for default-agent bootstrap helper. |
| `test_execute_env.py` | 165 | tests/unit/agents/test_execute_env.py |
| `test_expert_catalog.py` | 308 | Tests for ExpertCatalog workspace discovery and lazy file reads. |
| `test_expert_manifest_generator.py` | 361 | Tests for model-generated SkillHub expert manifest metadata. |
| `test_expert_publish_snapshot.py` | 434 | Tests for exported published-expert snapshots. |
| `test_general_assistant_template.py` | 26 | Sanity checks for the bundled general-assistant template. |
| `test_karpathy_knowledge_base_template.py` | 55 | Regression tests for the bundled Karpathy-style knowledge-base expert. |
| `test_library_task_examples.py` | 53 | Bundled expert templates expose tasks-page ``task_examples``. |
| `test_mcp_tool_cache.py` | 262 | Unit tests for custom MCP deferred load + user-level tool cache. |
| `test_media_generation_settings.py` | 147 | Tests for encrypted media-generation settings. |
| `test_memory_backend.py` | 149 | Symbols: test_default_memory_backend_empty_on_sqlite_control_plane, test_default_memory_backend_follows_postgresql_control_plane, test_explicit_sqlite_overrides_postgresql_control_plane, test_sqlite_explicit_uses_system_files_path, test_postgres_explicit_dsn, test_postgres_use_control_plane_dsn |
| `test_model_flags.py` | 129 | Tests for embedding / chat-eligibility model flags. |
| `test_octop_builtin_skills.py` | 258 | Tests for Octop-owned built-in Skills and the Skill Manager helper. |
| `test_onnx_download.py` | 402 | Tests for ONNX source race + download. |
| `test_onnx_service.py` | 443 | Tests for local ONNX embedding service helpers. |
| `test_persona.py` | 52 | tests/unit/test_persona.py |
| `test_plugin_tool_defaults.py` | 60 | Unit tests for plugin tool default-on / merge helpers. |
| `test_provider_harness_factory.py` | 41 | Tests for :mod:`octop.infra.agents.providers.harness_factory`. |
| `test_provider_reasoning.py` | 139 | Symbols: test_reasoning_capability_accepts_token_plan_thinking_option, test_known_token_plan_model_is_detected_without_saved_metadata, test_minimax_token_plan_reasoning_is_always_on, test_deepseek_v4_uses_nested_effort_parameter, test_explicit_capability_filters_unsupported_effort, test_openai_adapter_maps_disable_to_none_effort_without_thinking |
| `test_provider_reload_impact.py` | 159 | Tests for selective + parallel provider reload (faster confirm). |
| `test_provider_store.py` | 338 | Unit tests for :mod:`octop.infra.agents.providers.store`. |
| `test_reasoning_middleware.py` | 52 | Symbols: test_reasoning_middleware_updates_openai_field_and_extra_body, test_reasoning_middleware_updates_native_anthropic_fields |
| `test_runtime_limits.py` | 71 | Tests for agent runtime limit mapping. |
| `test_skill_package_store.py` | 209 | Tests for global skill package disk storage. |
| `test_skill_packages.py` | 196 | Tests for source-neutral skill package normalization. |
| `test_skillhub_http_market.py` | 212 | Tests for direct HTTP SkillHub search and package installation. |
| `test_skillhub_market.py` | 747 | Tests for SkillHub skillset normalization into expert templates. |
| `test_skills_hub_raw.py` | 239 | Unit tests for skills.sh / GitHub raw bundle resolution. |
| `test_subagent_catalog.py` | 218 | Tests for SubagentCatalog bundled library. |
| `test_thread_artifacts.py` | 197 | Thread workspace artifact extraction and middleware persistence. |
| `test_thread_fork.py` | 256 | Tests for conversation fork-from-assistant-message. |
| `test_token_quota_middleware.py` | 70 | Token quota agent middleware tests. |
| `test_tool_catalog.py` | 93 | Unit tests for built-in tool catalog / tools_disabled normalization. |
| `test_workspace_dir.py` | 125 | Unit tests for ``workspace_dir`` (independent of ``root_dir`` for harness). |
| `test_workspace_image_middleware.py` | 56 | Unit tests for WorkspaceImageMaterializeMiddleware. |

### `tests/unit/api/`

| File | Lines | Description |
|------|------:|-------------|
| `test_acl_gate_coverage.py` | 83 | ACL gate coverage: no current_admin; known permission keys only. |
| `test_agent_access.py` | 84 | tests/unit/api/test_agent_access.py |
| `test_attachments.py` | 267 | Unit tests for chat attachment workspace paths. |
| `test_backup_restore_rehydrate.py` | 220 | Unit tests for admin backup restore rehydrate and non-blocking paths. |
| `test_browser_stream_listen.py` | 138 | Listen-only browser-stream must not launch Chrome and must stay connected. |
| `test_browser_stream_mouse.py` | 139 | Browser WebSocket stream must forward press / move / release for drag. |
| `test_chat_composer_context.py` | 66 | Composer context for dashboard chat turns. |
| `test_chat_history.py` | 419 | Unit tests for thread list/history HTTP helpers. |
| `test_chat_hitl_resume.py` | 160 | Dashboard HITL resume SSE must persist follow-up approvals in the pending store. |
| `test_chat_polish.py` | 576 | Unit tests for chat polish and history helpers. |
| `test_chat_serialize.py` | 28 | Unit tests for chat SSE / WebSocket chunk serialization. |
| `test_chat_welcome.py` | 234 | Unit tests for agent chat welcome (workspace / catalog / default). |
| `test_content_disposition.py` | 45 | tests/unit/api/test_content_disposition.py |
| `test_dashboard_cache.py` | 33 | Cache-Control for dashboard SPA shell vs hashed Vite assets. |
| `test_exception_handlers.py` | 60 | Exception handlers log server-side OctopError details. |
| `test_harness_tabs.py` | 112 | tests/unit/api/test_harness_tabs.py |
| `test_hitl_resume_validation.py` | 74 | Validation for the HITL resume request body. |
| `test_jwt_auth_middleware.py` | 151 | Unit tests for JWT auth middleware. |
| `test_jwt_tokens.py` | 34 | tests/unit/test_jwt_tokens.py |
| `test_knowledge_bases.py` | 734 | Unit tests for the knowledge-base HTTP adapter. |
| `test_ollama_service_default.py` | 20 | Unit tests for Ollama local service enablement defaults. |
| `test_openapi_meta.py` | 31 | Unit tests for OpenAPI schema customization. |
| `test_permissions_api.py` | 38 | Unit tests for require_permission dependency. |
| `test_provider_patch_rehydrate.py` | 91 | Unit tests: skip harness rehydrate when provider patch is note-only. |
| `test_require_running_workspace.py` | 98 | Workspace I/O should survive a harness rebuild window. |
| `test_setup_main_agent.py` | 47 | Regression coverage for setup's default ``main`` agent. |
| `test_terminal_platform.py` | 45 | Unit tests for terminal platform support. |
| `test_terminal_sessions.py` | 469 | Unit tests for the terminal session machinery and WS handler. These run without the full ``OctopServer`` / ``harness_gateway`` stack: the ``terminal`` router module imports cleanly on its own, so we drive it with lightweight fakes for the server, the WebSocket and the PTY spawn. … |
| `test_update_router.py` | 474 | Unit tests for update API router helpers. |
| `test_update_store.py` | 30 | Unit tests for /api/update in-memory status cache. |
| `test_upload_limit.py` | 79 | Capped multipart reads must stop before assembling an oversize body. |
| `test_voice_mimo.py` | 209 | Unit tests for Mimo voice adapters (voice normalization + streaming TTS). |
| `test_workspace_io_path.py` | 43 | Workspace I/O path resolution for download / file APIs. |
| `test_workspace_tree_entries.py` | 26 | Tree listing entries stay anchored at the directory the caller requested. |

### `tests/unit/auth/`

| File | Lines | Description |
|------|------:|-------------|
| `test_captcha_env.py` | 63 | Boot-time CaptchaEnv snapshot. |
| `test_captcha_providers.py` | 166 | CaptchaProvider registry — slugs, aliases, plug-in seam. |
| `test_captcha_store.py` | 131 | Effective captcha config — settings blob vs env snapshot. |
| `test_captcha_verify.py` | 199 | ensure_captcha — slider no-op and mocked siteverify. |
| `test_dingtalk_adapter.py` | 132 | Tests for the DingTalk dashboard SSO adapter. |
| `test_feishu_adapter.py` | 227 | Tests for the Feishu dashboard SSO adapter. |
| `test_public_base.py` | 52 | Tests for public OIDC callback URL helpers. |
| `test_sso_crypto.py` | 22 | Tests for SSO secret encryption. |
| `test_sso_discovery.py` | 50 | Tests for OIDC discovery helpers. |
| `test_sso_id_token.py` | 134 | Tests for OIDC ID token verification. |
| `test_sso_pkce.py` | 21 | Tests for OIDC PKCE helpers. |
| `test_sso_redirect_after.py` | 27 | Tests for post-login redirect sanitization. |
| `test_sso_service.py` | 443 | Tests for OIDC SSO service orchestration. |
| `test_wecom_adapter.py` | 143 | Tests for the WeCom dashboard SSO adapter. |

### `tests/unit/backend/`

| File | Lines | Description |
|------|------:|-------------|
| `test_opensandbox_deps.py` | 25 | Tests for on-demand OpenSandbox SDK install. |
| `test_resolver.py` | 181 | Tests for backend resolver helpers. |

### `tests/unit/backup/`

| File | Lines | Description |
|------|------:|-------------|
| `test_auto.py` | 139 | Unit tests for automatic backup helpers. |
| `test_pg_dump_helpers.py` | 36 | Symbols: test_require_tool_missing, test_dump_postgres_can_exclude_chat_table_data |
| `test_snapshot.py` | 64 | Unit tests for SQLite snapshot helpers. |
| `test_store.py` | 291 | Unit tests for on-disk backup store. |
| `test_system_archive.py` | 1284 | Unit tests for system backup archives. |
| `test_workspace_archive.py` | 177 | Unit tests for workspace zip archives. |

### `tests/unit/browser/`

| File | Lines | Description |
|------|------:|-------------|
| `test_browser_setup.py` | 396 | Tests for browser profile prep / uninstall helpers. |

### `tests/unit/cli/`

| File | Lines | Description |
|------|------:|-------------|
| `__init__.py` | 0 | Octop Python module. |
| `test_captcha_cmd.py` | 50 | Tests for `octop captcha reset` (lockout escape hatch). |
| `test_chats_cmd.py` | 67 | Tests for the `octop chats` group. |
| `test_clean_cmd.py` | 54 | Tests for `octop clean`. |
| `test_cli_db.py` | 82 | Tests for offline CLI DB helpers. |
| `test_completion_cmd.py` | 43 | Tests for `octop completion`. |
| `test_init_cmd.py` | 135 | Unit tests for `octop init`. |
| `test_main_encoding.py` | 66 | CLI stdio encoding guard — emoji output must not crash on legacy code pages. Reported on Chinese Windows (GBK / cp936): ``octop init`` bootstraps the database and then raises ``UnicodeEncodeError`` while printing its ``✅`` success message, because piped/redirected stdout is encod… |
| `test_models_cmd.py` | 15 | Tests for ``octop models`` ollama subcommands. |
| `test_prompts.py` | 45 | Tests for the questionary wrapper (no real TTY). |
| `test_registry.py` | 29 | Lazy CLI registry integrity — every COMMANDS attr must resolve. |
| `test_repl_session.py` | 42 | Tests for REPL session state helpers. |
| `test_run_bind_resolution.py` | 133 | Bind address resolution for `octop run`: CLI flag > env > config.json. |
| `test_run_cmd.py` | 301 | Tests for `octop run`. |
| `test_service_cmd.py` | 248 | Tests for `octop service`. |
| `test_skills_cmd.py` | 72 | Tests for offline CLI skills helpers. |
| `test_stub.py` | 30 | Tests for the Not-Applicable STUB helper. |
| `test_update_cmd.py` | 238 | Tests for `octop update`. |
| `test_version_cmd.py` | 28 | Tests for `octop version`. |

### `tests/unit/connectors/`

| File | Lines | Description |
|------|------:|-------------|
| `test_cli_install.py` | 204 | Unit tests for connector host CLI install helper. |
| `test_cli_runner_and_dirs.py` | 82 | Unit tests for CLI runner messages and connector-cli dir cleanup. |
| `test_custom_mcp.py` | 488 | Unit tests for custom MCP validation and assembly. |
| `test_default_open.py` | 105 | Unit tests for connector default_open config helpers. |
| `test_feishu_user_auth.py` | 155 | Unit tests for Feishu CLI user device-code auth. |
| `test_feishu_user_auth_preview.py` | 83 | Unit tests for Feishu live user-auth preview. |
| `test_feishu_wecom_cli.py` | 474 | Unit tests for Feishu / WeCom CLI gateway adapters. |
| `test_mail_servers.py` | 119 | Unit tests for shared mailbox host resolution and qq-mail IMAP login. |
| `test_mcp_oauth_ssrf.py` | 93 | Unit tests for MCP OAuth SSRF guards. |
| `test_oauth_discovery.py` | 160 | Unit tests for MCP OAuth discovery and unified oauth targets. |
| `test_tencent_ima.py` | 233 | Unit tests for Tencent IMA gateway adapter (JSON OpenAPI tools). |
| `test_wecom_cli_errors.py` | 119 | Unit tests for WeCom CLI error humanization and probe. |

### `tests/unit/cron/`

| File | Lines | Description |
|------|------:|-------------|
| `test_cron_composer_context.py` | 243 | Cron agent runs should stamp composer_context on the stored user message. |
| `test_cron_delivery.py` | 358 | CronDeliveryService: canonical checkpoint, channel push, and best-effort extras. |
| `test_cron_job.py` | 172 | tests/unit/test_cron_job.py — CronJob bookkeeping around CronDeliveryService. |
| `test_cron_manager.py` | 683 | tests/unit/test_cron_manager.py Unit tests for CronManager. APScheduler and Gateway are patched out so no real async tasks or LLM calls are made. |
| `test_cron_manager_global.py` | 88 | tests/unit/test_cron_manager_global.py |
| `test_cron_mcp_servers.py` | 50 | CronJobRepo mcp_servers persistence. |
| `test_cron_tools.py` | 253 | Unit tests for built-in cronjob LangChain tools. |
| `test_cron_trigger.py` | 117 | tests/unit/test_cron_trigger.py |
| `test_task_type.py` | 54 | tests/unit/cron/test_task_type.py |

### `tests/unit/db/`

| File | Lines | Description |
|------|------:|-------------|
| `test_agent_is_shared.py` | 31 | Symbols: db, test_is_shared_default_zero_and_list_shared |
| `test_agent_profile_columns.py` | 163 | Lift expert display fields out of agents.config_json. |
| `test_clip_thread_title.py` | 88 | Thread repo title clipping and legacy hard-cut repair. |
| `test_db_factory.py` | 154 | tests/unit/test_db_factory.py |
| `test_db_pool.py` | 663 | tests/unit/test_db_pool.py |
| `test_db_sql_helpers.py` | 34 | tests/unit/test_db_sql_helpers.py |
| `test_migrate_discovery.py` | 17 | Symbols: test_discover_sqlite_excludes_pg_files, test_discover_postgresql_only_pg_files |
| `test_postgres_pool_unit.py` | 13 | Symbols: test_qmark_to_pyformat_replaces_placeholders, test_qmark_to_pyformat_leaves_percent_alone |
| `test_published_experts_repo.py` | 133 | Unit tests for PublishedExpertRepo and migration 006. |
| `test_repo_agents.py` | 144 | tests/unit/test_repo_agents.py |
| `test_repo_channels.py` | 145 | tests/unit/test_repo_channels.py |
| `test_repo_cron.py` | 193 | tests/unit/test_repo_cron.py |
| `test_repo_knowledge.py` | 348 | Unit tests for KnowledgeRepo and migration 005. |
| `test_repo_providers.py` | 159 | tests/unit/test_repo_providers.py |
| `test_repo_secret_audit.py` | 69 | tests/unit/test_repo_secret_audit.py |
| `test_repo_sessions_threads.py` | 252 | tests/unit/test_repo_sessions_threads.py |
| `test_repo_sso.py` | 137 | Tests for OIDC SSO repository access. |
| `test_repo_users.py` | 75 | tests/unit/test_repo_users.py |
| `test_runtime_replace_services.py` | 82 | AppRuntime.replace_services — public control-plane retarget API. |
| `test_skill_package_icons.py` | 116 | Unit tests for skill package icon columns and get_by_name. |
| `test_skill_packages_repo.py` | 71 | Unit tests for SkillPackageRepo and migration 002. |
| `test_storage_backend_repo.py` | 73 | Unit tests for BackendRepo. |
| `test_thread_messages_repo.py` | 108 | Symbols: test_new_thread_projection_appends_and_pages, test_pending_legacy_thread_only_publishes_after_replace |
| `test_usage_window.py` | 70 | Unit tests for usage window resolution (server timezone day/month). |
| `test_user_permissions_column.py` | 60 | Tests for users.permissions column wiring. |

### `tests/unit/desktop/`

| File | Lines | Description |
|------|------:|-------------|
| `test_capture.py` | 112 | Tests for desktop screen capture helpers. |
| `test_capture_display.py` | 20 | Tests for DISPLAY helpers. |
| `test_input.py` | 56 | Tests for desktop input injection. |
| `test_native_platform.py` | 20 | Tests for native vs virtual desktop capture paths. |
| `test_session.py` | 99 | Tests for in-process desktop session registry. |
| `test_setup.py` | 330 | Tests for desktop setup helpers. |
| `test_stamp_version.py` | 83 | Tests for desktop Wails version stamping. |

### `tests/unit/gateway/`

| File | Lines | Description |
|------|------:|-------------|
| `test_attachment_hints.py` | 366 | Unit tests for inbound attachment LLM hints. |
| `test_attachment_path_resolution.py` | 225 | Verify attachment workspace-path behaviour for LLM filesystem tools. Documents the gap between inbound attachment storage (BackendWorkspace) and deepagents filesystem tools under Octop's default backend (root_dir='/'). |
| `test_backend_files.py` | 106 | Unit tests for gateway media path helpers. |
| `test_channels_qr.py` | 461 | Unit tests for channels QR scan endpoints. |
| `test_chat_image_pipeline.py` | 267 | End-to-end attachment pipeline tests (no WebSocket — direct API calls). |
| `test_cli_channel.py` | 80 | Tests for the CLI gateway channel. |
| `test_dashboard_ws.py` | 675 | tests/unit/test_dashboard_ws.py |
| `test_feishu_bot_creator.py` | 130 | Unit tests for Feishu scan-to-create (lark-oapi register_app). |
| `test_gateway.py` | 140 | tests/unit/test_gateway.py |
| `test_gateway_probe.py` | 85 | Unit tests for Gateway.probe_channel(). |
| `test_gateway_push.py` | 256 | tests/unit/test_gateway_push.py |
| `test_gateway_runtime_status.py` | 177 | Unit tests for Gateway runtime channel status. |
| `test_global_processor_team.py` | 274 | tests/unit/test_global_processor_team.py |
| `test_harness_request.py` | 141 | Symbols: test_group_context_is_rendered_as_background_inside_current_turn, test_mention_only_still_attributes_the_current_group_sender, test_group_context_renders_passive_file_as_agent_path_hint, test_group_context_uses_anonymous_labels_instead_of_raw_ids |
| `test_history_projection.py` | 176 | Symbols: test_message_input_preserves_existing_checkpoint_ts, test_message_input_does_not_invent_stamp_by_default, test_live_message_input_stamps_when_missing, test_live_message_input_preserves_existing_stamp |
| `test_hitl_ask_user.py` | 338 | Unit tests for the ``ask_user_question`` HITL channel. |
| `test_hitl_channel.py` | 398 | Unit tests for IM channel HITL. |
| `test_hitl_resume_projection.py` | 84 | Dashboard HITL resume keeps the thread-history projection complete. |
| `test_inbound_store.py` | 105 | Unit tests for inbound attachment storage. |
| `test_media_preview.py` | 73 | tests/unit/test_media_preview.py |
| `test_mention_agent_calls.py` | 57 | Team peer discovery after harness-agent dropped apply_mentions intercept. |
| `test_message_build.py` | 107 | tests/unit/test_message_build.py |
| `test_message_keys.py` | 99 | Unit tests for IM session/user key derivation. |
| `test_preempt_stop_cancel.py` | 75 | Gateway pre-lock /stop cancel preempts the in-flight turn. |
| `test_processor_default_open.py` | 225 | GlobalProcessor injects default_open connectors on IM turns. |
| `test_processor_knowledge_services.py` | 27 | GlobalProcessor knowledge services must carry provider_repo for remote embeds. |
| `test_processor_resource_errors.py` | 132 | Resource-policy stream error projection tests. |
| `test_processor_trajectory_observe.py` | 171 | GlobalProcessor side-notifies TrajectoryService without changing stream payloads. |
| `test_response_mode.py` | 127 | Tests for external IM response delivery modes. |
| `test_slash_catalog.py` | 29 | tests/unit/test_slash_catalog.py |
| `test_slash_compact.py` | 286 | Tests for /compact summarization slash command. |
| `test_slash_dispatcher.py` | 225 | tests/unit/test_slash_dispatcher.py |
| `test_slash_help.py` | 35 | tests/unit/test_slash_help.py |
| `test_slash_history.py` | 107 | Tests for /history conversation stats. |
| `test_slash_parse.py` | 43 | tests/unit/gateway/test_slash_parse.py |
| `test_slash_skills_connectors.py` | 71 | tests/unit/test_slash_skills_connectors.py |
| `test_slash_stop_status.py` | 110 | tests/unit/test_slash_stop_status.py |
| `test_stream_project_tools.py` | 31 | tests/unit/gateway/test_stream_project_tools.py |
| `test_thread_registry.py` | 331 | tests/unit/test_thread_registry.py |
| `test_tool_media.py` | 471 | tests/unit/test_tool_media.py |
| `test_usage_record.py` | 180 | Turn-level token usage extraction (multi-call agent loops). |
| `test_versioned_history.py` | 1054 | Octop Python module. |

### `tests/unit/i18n/`

| File | Lines | Description |
|------|------:|-------------|
| `test_admin_users_expert_naming.py` | 18 | Admin Users UI must call agents 「专家」 / Experts (issue #155). |
| `test_agents.py` | 51 | Tests for agents i18n domain. |
| `test_catalog.py` | 41 | tests/unit/i18n/test_catalog.py |
| `test_channel.py` | 16 | tests/unit/i18n/test_channel.py |
| `test_desktop.py` | 19 | tests/unit/i18n/test_desktop.py |
| `test_errors.py` | 105 | tests/unit/i18n/test_errors.py |
| `test_mobile.py` | 13 | tests/unit/i18n/test_mobile.py |
| `test_skills.py` | 42 | tests/unit/i18n/test_skills.py |
| `test_stream.py` | 162 | Tests for stream_errors i18n domain. |
| `test_tools.py` | 81 | tests/unit/i18n/test_tools.py |
| `test_voice.py` | 34 | Voice probe / Tencent error copy. |

### `tests/unit/infra/setup/`

| File | Lines | Description |
|------|------:|-------------|
| `test_self_update.py` | 82 | Tests for octop.infra.setup.self_update. |
| `test_service.py` | 877 | Tests for octop.infra.setup.service. |

### `tests/unit/infra/setup/tls/`

| File | Lines | Description |
|------|------:|-------------|
| `test_challenge.py` | 14 | Unit tests for ACME challenge store. |
| `test_listeners.py` | 39 | Unit tests for dual-port listen plan. |
| `test_modes.py` | 48 | Unit tests for TLS issue mode helpers. |
| `test_preflight.py` | 80 | Unit tests for TLS preflight checks. |
| `test_renewal.py` | 21 | Unit tests for TLS auto-renewal helpers. |
| `test_tls_store.py` | 45 | Unit tests for TLS certificate storage and config updates. |

### `tests/unit/infra/utils/`

| File | Lines | Description |
|------|------:|-------------|
| `test_bwrap.py` | 141 | Tests for best-effort bubblewrap (``bwrap``) provisioning — all platforms. |
| `test_docker_env.py` | 95 | Tests for Docker environment detect / ensure helpers. |
| `test_host_dirs.py` | 319 | Tests for host directory listing helpers. |
| `test_json_file.py` | 107 | Unit tests for the shared JSON config helpers (issue #730). |
| `test_ssl_errors.py` | 28 | Unit tests for SSL error detection helpers. |
| `test_subprocess_io.py` | 35 | Unit tests for cross-platform subprocess stdout reads. |
| `test_utf8_text.py` | 61 | Tests for UTF-8 coercion of third-party package text files. |

### `tests/unit/knowledge/`

| File | Lines | Description |
|------|------:|-------------|
| `test_chunk.py` | 19 | Unit tests for document chunking. |
| `test_citations.py` | 95 | Unit tests for knowledge citation markers. |
| `test_embed.py` | 145 | Symbols: test_remote_embedding_routes_to_provider_embeddings_endpoint, test_remote_embedding_merges_provider_extra_headers, test_remote_embedding_batches_large_input |
| `test_gate.py` | 119 | Unit tests for the knowledge-base capability gate. |
| `test_index.py` | 31 | Unit tests for per-knowledge-base sidecar indexes. |
| `test_jobs.py` | 195 | Unit tests for in-process document indexing jobs. |
| `test_knowledge_default_open.py` | 88 | Unit tests for knowledge-base turn selection defaults. |
| `test_knowledge_hint.py` | 138 | Unit tests for knowledge-base tool description enrichment. |
| `test_knowledge_tools.py` | 96 | Unit tests for built-in knowledge-base LangChain tools. |
| `test_ocr.py` | 156 | Tests for optional knowledge-base OCR configuration and routing. |
| `test_params.py` | 54 | Unit tests for knowledge advanced indexing/retrieval settings. |
| `test_parse.py` | 224 | Unit tests for supported knowledge-document parsers. |
| `test_relpath.py` | 21 | Symbols: test_normalize_kb_path_rejects_parent_segments, test_path_helpers |
| `test_retrieve.py` | 81 | Unit tests for knowledge retrieval during a chat turn. |
| `test_service_acl.py` | 279 | Unit tests for knowledge-base access control and document orchestration. |
| `test_text_documents.py` | 169 | KnowledgeService text create / update helpers. |

### `tests/unit/mobile/`

| File | Lines | Description |
|------|------:|-------------|
| `test_agent_control.py` | 40 | Unit tests for mobile agent-control binding. |
| `test_config_probe.py` | 22 | tests/unit/mobile/test_config_probe.py |
| `test_device_info.py` | 51 | Unit tests for adb device-info parsing. |
| `test_docker_install.py` | 341 | tests/unit/mobile/test_docker_install.py |
| `test_find_adb.py` | 73 | Unit tests for adb discovery (PATH + SDK env only). |
| `test_h264.py` | 86 | Unit tests for mobile capture / H.264 helpers. |
| `test_mobile_setup.py` | 68 | tests/unit/mobile/test_mobile_setup.py |
| `test_mobile_tools.py` | 115 | Unit tests for built-in mobile LangChain tools. |
| `test_probe.py` | 37 | tests/unit/mobile/test_probe.py |
| `test_rotation.py` | 137 | Unit tests for adb device rotation helpers. |

### `tests/unit/proactive/`

| File | Lines | Description |
|------|------:|-------------|
| `test_picker.py` | 240 | Unit tests for EpisodePicker. Coverage: - score ordering: negative emotion > positive emotion, high intensity > low intensity - people deduplication: keep only the highest-scored episode per person - time-window fallback: expand from 7 days to 30 days when all recent episodes wer… |
| `test_proactive_service.py` | 301 | Unit tests for ProactiveCareService. Coverage: - successful push flow with episodes, LLM success, push success, and dedup record writes - skip push when episodes are empty - skip push and dedup writes when the LLM call fails - skip dedup writes when push fails - truncate long car… |
| `test_scheduler.py` | 428 | Unit tests for ProactiveCareScheduler. Coverage: - compute_next_trigger: random scheduling inside active hours - compute_next_trigger: defer to the next active window when outside active hours - is_in_active_hours: active-window checks - ProactiveCareScheduler: enabled=false shou… |

### `tests/unit/providers/`

| File | Lines | Description |
|------|------:|-------------|
| `test_codex_oauth.py` | 99 | Unit tests for Codex OAuth helpers. |
| `test_opencode_session.py` | 158 | Unit tests for OpenCode Go session-header wiring. |

### `tests/unit/skills/`

| File | Lines | Description |
|------|------:|-------------|
| `test_presentation.py` | 73 | Symbols: test_apply_skill_presentation_exposes_localized_copy_and_assets, test_apply_skill_presentation_keeps_identity_without_locale, test_localize_skill_summary_uses_english_then_chinese_fallback, test_legacy_display_name_and_compatible_emoji_remain_supported |
| `test_skill_install.py` | 98 | Unit tests for shared skill install target pipeline. |
| `test_skill_package_from_skillhub.py` | 135 | Tests for materializing SkillHub skillsets as global packages. |
| `test_workspace_catalog.py` | 81 | Tests for symlink-tolerant workspace skill discovery. |

### `tests/unit/trajectory/`

| File | Lines | Description |
|------|------:|-------------|
| `test_live.py` | 38 | TrajectoryLiveBus — in-process pub/sub per thread_id. |
| `test_metrics.py` | 94 | Pure metrics aggregation from trajectory events. |
| `test_projector.py` | 227 | Harness/session-fact projector tests. Fixture shapes live in ``tests/unit/trajectory/fixtures/``: * ``session_user.json`` — anonymized MemoryMiddleware ``sessions/*.jsonl`` user line (real local samples use ``role`` not ``type``; ISO-8601 ``ts``). Keys: ``ts``, ``role=user``, ``c… |
| `test_trajectory_event_repo.py` | 160 | TrajectoryEventRepo — append, cursor page, duplicate event_id. |
| `test_trajectory_list_summarize.py` | 86 | Unit tests for trajectory list payload projection. |
| `test_trajectory_service.py` | 363 | TrajectoryService — observe never raises; list / metrics / export. |
| `test_trajectory_settings.py` | 50 | Per-agent trajectory flag and persist clipping. |
| `test_trajectory_store.py` | 92 | TrajectoryStore — thin repo wrapper; duplicate event_id is a no-op. |
| `test_turn_context.py` | 100 | Turn-start SYSTEM / CONTEXT synthesis — must match harness injection SoT. |

### `tests/unit/users/`

| File | Lines | Description |
|------|------:|-------------|
| `test_email.py` | 31 | Tests for email normalize / validate helpers. |
| `test_invites.py` | 88 | Unit tests for invite code helpers and repo redeem. |
| `test_permissions.py` | 90 | Unit tests for the module permission catalog. |
| `test_preferences.py` | 113 | tests/unit/users/test_preferences.py |
| `test_resource_policy.py` | 272 | tests/unit/users/test_resource_policy.py |

### `tests/unit/utils/`

| File | Lines | Description |
|------|------:|-------------|
| `test_doc_edit.py` | 119 | Tests for the editable-document registry and its Markdown round-trip. Covers :mod:`octop.infra.utils.doc_edit`: the extension registry that powers the generic ``/workspace/doc`` endpoints, plus the docx converter's ``to_markdown`` / ``from_markdown`` round-trip. |
| `test_posix_compat.py` | 28 | Tests for POSIX compat helpers used by the web terminal. |
| `test_runtime_packages.py` | 125 | Tests for runtime optional Python package installation. |
| `test_search_probe.py` | 92 | Unit tests for search-provider HTTP probes. |
| `test_ssrf_guard.py` | 51 | Unit tests for SSRF guard (infra/utils/ssrf_guard.py). |

