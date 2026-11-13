# blc-plugin-template · 大懒猫创作平台插件模板

**简体中文** | [English](README.en.md)

一个**开箱即跑**的插件起点工程：包含后台命令、浏览器视图、页面状态保存、构建配置和测试，全部基于官方插件 SDK v1。不用从空目录猜清单格式，clone 下来改几个名字就能开始写自己的插件。

仓库：<https://github.com/biglazycat-writer/blc-plugin-template>

| 项目 | 值 |
| --- | --- |
| 插件 API | `biglazycat.apiVersion = 1` |
| 运行环境 | Node `>=22.23.3 <23.0.0`（宿主提供，安装包不内嵌 Node） |
| 后台 | TypeScript → `dist/main.js` |
| 页面 | TypeScript + Vite → `dist/ui/index.html`（原生 DOM，可换 React/Vue） |
| 测试 | `node:test`，mock `PluginContext`，不需要启动宿主 |
| 示例内容 | 3 个命令、1 个视图、2 个菜单入口、6 个平台声明 |

---

## 从这里开始

### 方式一：GitHub 模板仓库（推荐）

本仓库已开启 GitHub **Template repository**。在仓库页点 **Use this template → Create a new repository**，填好新仓库名后 clone 到本地；生成的是干净的新仓库，不会带上模板仓库的提交历史。

也可以命令行创建：

```bash
gh repo create my-plugin --template biglazycat-writer/blc-plugin-template --private --clone
cd my-plugin
```

### 方式二：clone 后换成自己的仓库

```bash
git clone https://github.com/biglazycat-writer/blc-plugin-template.git my-plugin
cd my-plugin
git remote remove origin          # 或直接 set-url 指向你自己的仓库
git remote add origin https://github.com/<你的账号>/my-plugin.git
npm install
```

### 方式三：只要一个本地工程

下载 ZIP 解压到任意目录，或用宿主的 MCP `plugin_project_create` 生成空工程后，把本模板的 `src/`、`package.json`、`package-lock.json` 等复制过去。

---

## 把模板改成你的插件

**必改清单**（改完再运行一次 `npm run build && npm test`）：

| # | 位置 | 模板里的值 | 改成 |
| --- | --- | --- | --- |
| 1 | `package.json` → `name` | `blc-plugin-template` | 你的包名，例如 `blc-plugin-weather` |
| 2 | `package.json` → `description` / `version` | 模板说明 / `0.1.0` | 你的描述与版本 |
| 3 | `biglazycat.displayName` / `description` | 插件模板 · Hello World | 插件在宿主里显示的名称与说明 |
| 4 | `biglazycat.contributes.commands[].id` | `template.*` | 换成你的前缀，例如 `weather.*` |
| 5 | `src/main.ts` 的 `ctx.commands.register(...)` | 同上三个 ID | **必须与清单逐字一致**，否则宿主启用时会拒绝 |
| 6 | `biglazycat.contributes.views` / `menus` | `main`、`template.open` | 你的视图 ID/标题与菜单命令引用 |
| 7 | `README.md` 标题与仓库链接 | 本文件 | 你自己的文档 |

其他约定：

- `aiCallable` / `externalEffect` 如实标注；只有真正写外部系统的命令才设 `externalEffect: true`。
- 新增命令时**同时**改清单和 `src/main.ts`；所有命令都在激活期注册，不要在首次调用时补注册。
- 页面入口 `entry` 指向构建产物 `dist/ui/index.html`，不要改成源码路径。
- `package.json` 的 `biglazycat` 字段是唯一清单，字段集合受 npm 包 `blc-plugin-sdk` 的 `manifest.schema.json` 约束（顶层 `additionalProperties: false`，不能随意加字段）。
- `scripts/package.mjs` 与 `.github/workflows/release.yml` **不需要改**：包名、版本、产物文件名都从 `package.json` 读取，改完 `name` / `version` 即可直接复用。

---

## 环境要求

- Node `>=22.23.3 <23.0.0`、npm 10.x。宿主按选定 Node 的实际版本校验 engines；原生依赖必须单独验证各平台。
- 大懒猫创作平台：在「系统设置 → 运行环境」配置本机 Node/npm；启用 MCP 服务中的插件工具可让外部编程工具直接调用。
- SDK 固定依赖公开 npm 包 `blc-plugin-sdk@1.1.0`，无需本地 vendor 副本或全局安装。

## 开发命令

```bash
npm ci                # 从锁文件安装 blc-plugin-sdk@1.1.0 与开发依赖
npm run typecheck     # UI 与后台两份 tsconfig 严格类型检查（不产出文件）
npm run build         # tsc 编译后台到 dist/main.js，Vite 打包页面到 dist/ui/
npm test              # node --test tests/*.test.mjs（依赖 dist/main.js，先 build）
npm run package       # 构建并生成桌面端可安装的插件包 dist-package/<name>-<version>.tgz
```

构建产物：

```
dist/main.js                         后台入口，对应清单的 biglazycat.main
dist/ui/index.html                   页面入口，对应清单的 contributes.views[].entry
dist-package/<name>-<version>.tgz    插件包（顶层 package/，可直接在桌面端安装）
```

> `dist/` 与 `node_modules/` 不进版本库；`package-lock.json` 必须提交（含 SDK 版本与 registry integrity）。宿主的平台准备流程用 `npm ci` 安装，因此首次安装需能访问 npm registry。

---

## 目录结构

```
├─ package.json              npm 元信息 + biglazycat 插件清单（唯一清单）
├─ package-lock.json         锁文件，必须提交
├─ tsconfig.json             页面类型检查（noEmit）
├─ tsconfig.node.json        后台编译（outDir: dist）
├─ vite.config.mjs           页面打包（root: src/ui）
├─ scripts/package.mjs       插件打包：组装 package/ 目录、校验结构、生成 SHA-256
├─ src/
│  ├─ main.ts                后台入口：definePlugin、命令注册、清理函数
│  └─ ui/
│     ├─ index.html          页面骨架
│     ├─ main.ts             连接 SDK、调用命令、状态保存
│     └─ style.css           最小样式
└─ tests/plugin.test.mjs     后台命令测试（mock 宿主上下文）
```

## 打包与发布

`npm run package` 生成桌面端可直接安装的插件包：

```bash
npm run package
# ✓ blc-plugin-template-0.1.0.tgz（12 个条目）
# ✓ 结构校验通过：顶层 package/，含 package.json 与 package-lock.json
# ✓ SHA-256 ...
```

产物在 `dist-package/`，同目录生成 `.sha256` 校验文件。

### 为什么必须用这个脚本

桌面端安装插件包时只做两件事：解压、读取「解压根目录/`package.json`」。它的解压逻辑**只剥离 `package/` 前缀**，因此：

- 归档顶层必须是 `package/`；
- `package.json` 必须在 `package/` 下，不能再套一层 `你的插件-0.1.0/`；
- 必须包含 `package-lock.json`，否则宿主「准备依赖」时无法执行 `npm ci`。

自己用系统「压缩为 zip」或 `tar czf 插件名-版本.tar.gz 目录/` 很容易踩这个坑，桌面端会报找不到 `package.json`。`scripts/package.mjs` 按 `package.json` 的 `files` 字段组装 `package/` 目录、打包、校验结构并生成 SHA-256，所以请直接用 `npm run package`。

### 在桌面端安装

1. 「插件」→「安装插件」→ 来源选 **插件包 (.zip / .tgz)**；
2. 选择 `dist-package/` 下的 `.tgz`；
3. **勾选「准备依赖、类型检查并构建」** —— 包内不含 `node_modules`，宿主会执行 `npm ci` 安装依赖；不勾选会安装成功但运行时缺少依赖；
4. 勾选信任说明后安装。

### 发布 Release

仓库自带 `.github/workflows/release.yml`：

1. 修改 `package.json` 的 `version`（可选：在 `CHANGELOG.md` 补对应版本段落，Release notes 会自动取用）；
2. 提交推送后打标签：`git tag -a v0.1.1 -m "v0.1.1" && git push origin v0.1.1`；
3. 工作流自动执行 `npm ci` → `typecheck` → `test` → `npm run package`，然后创建 GitHub Release，附上 `.tgz` 与 `.sha256`。

标签必须等于 `v` + `package.json` 的 `version`。只想验证构建时，在 Actions 页面手动运行 Release 工作流、`tag` 留空即可（产物上传为 artifact，不创建 Release）。

## 模板演示了什么

**后台（`src/main.ts`）**

- `definePlugin(async ctx => ...)`：激活期注册命令，返回停用时的清理函数。
- `invocation.signal.throwIfAborted()` 协作取消、`invocation.progress()` 报告进度。
- `ctx.views.open()` 打开视图并传入调用上下文与初始状态。
- 命令返回值必须是可序列化 JSON；`ctx.pluginId` / `apiVersion` / `generation` 可直接读取。

**页面（`src/ui/main.ts`）**

- `connect()` 连接宿主桥接，`context()` 取回视图会话与恢复状态（先校验类型再使用）。
- `commands.invoke()` 返回宿主任务，用 SDK 的 `waitForTask(sdk.tasks, task.id)` 限时等待完成再取结果；失败/取消/不确定状态显式报错。
- `setState(state, dirty)` 保存小型恢复状态，`onSave()` 在宿主保存时写回并清 dirty；**不要**把大文件或机密放进 state。
- 页面用原生 DOM 构建，避免 `innerHTML`；换成 React/Vue 只需改 `src/ui` 与 `vite.config.mjs` 的依赖，清单不动。

**测试（`tests/plugin.test.mjs`）**

- 用最小 mock 校验：清单命令与注册一致、命令返回值/进度/取消、视图打开参数、清单引用自洽。
- 不联网、不启动宿主、不碰真实文档；新增命令时同步补一条测试。

---

## 安装到宿主（开发模式）

1. 启动大懒猫创作平台，在「系统设置」配置 Node/npm 并启用 MCP 服务中的插件工具。
2. 外部编程工具连接应用的 `--mcp-stdio`，先读 `plugin_sdk` 与 `get_app_guide(section:"plugins")`。
3. 调用 `plugin_project_run` 或本地 npm 命令完成 `install` / `typecheck` / `build` / `test`。
4. 调用 `plugin_install`：

   ```json
   {
     "source": { "kind": "directory", "location": "D:\\my-plugin" },
     "development": true,
     "prepare": false,
     "trusted": true,
     "requestId": "唯一标识"
   }
   ```

5. 用 `plugin_task` 确认完成，再调用 `plugin_enable` 与 `plugin_open_view`，或从编辑器命令/工具栏打开页面。
6. 修改源码后宿主自动类型检查、构建并刷新已打开页面；构建失败看 `plugin_diagnostics` 与任务日志。停用插件会停止自动刷新，卸载保留源码。

> 页面只能在宿主内打开。直接在浏览器打开 `dist/ui/index.html` 会报 `API_MISMATCH`，因为桥接由宿主提供。

---

## 继续开发时可以用到的能力

完整签名见 npm 包 [blc-plugin-sdk](https://www.npmjs.com/package/blc-plugin-sdk) 的 `index.d.ts` 与 [SDK 仓库 README](https://github.com/biglazycat-writer/blc-plugin-sdk#readme)：

- `ctx.credentials` 存取密钥（Windows 为 DPAPI），**绝不**写入配置或日志。
- `readDocument(ctx.documents, target)` 读取 Markdown 固定快照；声明 `minimumApiRevision: 1` 后使用该能力。原位写回仍使用 `ctx.documents.snapshot/apply`，`ctx.workspace` 与 `ctx.images` 读工作区资源与图片。
- `ctx.services` / `ctx.events` / `contributes.schedules`：服务、事件与定时任务。
- 需要 React、状态管理、编辑器、网络请求等，直接加 npm 依赖并在 `src/ui` 或后台引入即可。

几条硬限制，设计功能时要记住：

- 单次命令结果 **< 1 MiB**，页面状态最多 64 KiB；大稿件/图片必须自己分段传输（可参考官方 `blc-plugin-publish` 插件）。
- 取消、退出与超时可能留下 `uncertain` 的外部结果，不要自动重试有副作用的操作。
- 原生模块需要匹配系统、架构与 Node ABI；插件是可信本地程序，进程隔离不是安全沙箱。

## 排错

| 现象 | 检查 |
| --- | --- |
| 启用时提示命令不一致 | 清单 `contributes.commands[].id` 与 `src/main.ts` 注册 ID 逐字比对 |
| `Cannot find module '../dist/main.js'` | 测试前先 `npm run build` |
| `API_MISMATCH` | 页面必须在宿主插件视图内打开，不能用普通浏览器 |
| 构建失败 | 看 `npm run typecheck` 输出与宿主 `plugin_diagnostics` |
| `npm ci` 报引擎不匹配 | 宿主要求 `engines.node` 指定的 Node `>=22.23.3 <23.0.0` |
| `npm install` 找不到 `blc-plugin-sdk` | 检查 npm registry 可达性和 `blc-plugin-sdk@1.1.0` 依赖声明 |
| 页面刷新后状态丢失 | 确认调用了 `setState`，且 `onSave` 里写回了状态 |

## 授权

SDK 包 [blc-plugin-sdk](https://github.com/biglazycat-writer/blc-plugin-sdk) 为 MIT。本模板的示例代码可自由复制、修改并用于你自己的插件项目，无需署名；仓库整体的开源授权由维护者决定。

## SDK 与平台支持

本工程固定使用 npm 包 `blc-plugin-sdk@1.1.0`。页面调用通过 `waitForTask` 有限等待；超时后查询现有任务，不自动重新提交有副作用的操作。

构建目标为 ES2022 / Chromium 109 / Safari 16.4。当前更新在 Windows amd64 完成测试；平台声明与编译目标不代表所有系统 WebView、原生依赖均已完成实机验证。
