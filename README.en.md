# blc-plugin-template · 大懒猫创作平台 Plugin Template

[简体中文](README.md) | **English**

A ready-to-build starting point for plugins: backend commands, a browser view, recoverable view state, build configuration and tests using the official SDK. Clone the project, rename its identifiers and start implementing your plugin.

Repository: <https://github.com/biglazycat-writer/blc-plugin-template>

| Item | Value |
| --- | --- |
| Plugin API | `biglazycat.apiVersion = 1` |
| SDK | npm package `blc-plugin-sdk@1.1.0`, pinned in the lockfile |
| Runtime | Node `>=22.23.3 <23.0.0`; configured by the host, not bundled in its installer |
| Backend | TypeScript → `dist/main.js` |
| View | TypeScript + Vite → `dist/ui/index.html`; plain DOM, replaceable with React/Vue |
| Tests | `node:test` with a mocked `PluginContext`; no running host required |
| Example | Three commands, one view, two menu entries, six platform declarations |

## Create your project

Use **Use this template → Create a new repository** on GitHub to create a repository without this template's commit history. The GitHub CLI equivalent is:

```bash
gh repo create my-plugin --template biglazycat-writer/blc-plugin-template --private --clone
cd my-plugin
```

Alternatively, clone the repository and change its remote:

```bash
git clone https://github.com/biglazycat-writer/blc-plugin-template.git my-plugin
cd my-plugin
git remote set-url origin https://github.com/<your-account>/my-plugin.git
```

For a local-only project, download and extract the ZIP. You can also create a project with the host's `plugin_project_create` MCP tool and copy this template's source, manifest and lockfile into it.

## Rename the template

| Location | Template value | Change to |
| --- | --- | --- |
| `package.json` → `name` | `blc-plugin-template` | Your npm-style package ID |
| `description` / `version` | Template description / `0.1.0` | Your description and version |
| `biglazycat.displayName` / `description` | Template labels | Labels shown in the host |
| `biglazycat.contributes.commands[].id` | `template.*` | Your command namespace |
| `src/main.ts` registrations | The same three IDs | Exact matches for the manifest |
| View and menu declarations | `main`, `template.open` | Your view IDs, titles and command references |
| README title and repository links | This project | Your documentation in both languages |

Register all commands during activation. Update the manifest and implementation together. Set `aiCallable` and `externalEffect` accurately; only commands that write to external systems should declare `externalEffect: true`. View `entry` paths must refer to built HTML, not source files.

The `biglazycat` field is the only plugin manifest. Its fields are constrained by the SDK's `manifest.schema.json`, which disallows arbitrary extra properties.

## Requirements and commands

Use Node `>=22.23.3 <23.0.0` and npm `>=10.9.9 <11.0.0`. In 大懒猫创作平台, configure Node/npm in **System Settings → Runtime**. Enable the MCP plugin tools if you want external coding tools to build, install and inspect plugins.

The SDK is the published npm package `blc-plugin-sdk@1.1.0`. No global installation or local vendor copy is required. Native npm dependencies must still be tested on each target platform.

```bash
npm ci                 # Install the exact SDK and development dependencies
npm run typecheck      # Check browser and backend TypeScript configurations
npm run build          # Compile the backend and bundle the view
npm test               # Test built backend output; build first
```

Outputs are `dist/main.js` and `dist/ui/index.html`. Exclude `dist/` and `node_modules/` from version control; commit `package-lock.json`, including registry integrity values. Host preparation also uses `npm ci`, so initial dependency installation needs registry access.

## Project layout

```text
package.json              npm metadata and the only plugin manifest
package-lock.json         Reproducible dependency lockfile
tsconfig.json             Browser type checking, noEmit
tsconfig.node.json        Backend compilation to dist/
vite.config.mjs           View bundling from src/ui
src/main.ts               Activation, command registration and cleanup
src/ui/index.html         View shell
src/ui/main.ts            Bridge, command invocation and state persistence
src/ui/style.css          Minimal styling
tests/plugin.test.mjs     Backend command tests with a mocked host
```

## What the example demonstrates

The backend uses `definePlugin(async ctx => ...)` to register commands and return a disposal function. `invocation.signal.throwIfAborted()` supports cooperative cancellation, and `invocation.progress()` reports progress. `ctx.views.open()` passes invocation context and initial state to a view. Results must be serializable JSON. Plugin identity, API version and generation are available on the context.

The page calls `connect()` and reads `context()` before validating restored state. `commands.invoke()` creates a task; `waitForTask(sdk.tasks, task.id)` waits with a deadline and preserves terminal errors. Timeout does not replay or automatically cancel the backend command. `setState(state,dirty)` saves a small checkpoint, while `onSave()` persists state and clears the dirty flag. Do not put large files or secrets in page state.

The UI uses DOM construction rather than `innerHTML`. Replacing it with React/Vue mainly changes `src/ui`, dependencies and bundling. Tests cover manifest/registration agreement, results, progress, cancellation, view-open parameters and manifest references. They do not use the network, run the host or modify real documents. Add tests for new commands.

## Install in development mode

1. Start 大懒猫创作平台, configure Node/npm and enable the MCP plugin tools.
2. Connect external tools through the application's `--mcp-stdio`. Read `plugin_sdk` and `get_app_guide(section:"plugins")` first.
3. Install dependencies and run type checking, build and tests through npm or `plugin_project_run`.
4. Call `plugin_install` with your project's absolute path:

   ```json
   {
     "source": { "kind": "directory", "location": "D:\\my-plugin" },
     "development": true,
     "prepare": false,
     "trusted": true,
     "requestId": "unique-request-id"
   }
   ```

5. Check completion with `plugin_task`, then call `plugin_enable` and `plugin_open_view`, or use the contributed menu/toolbar entry.
6. Source changes trigger automatic type checking, building and view refresh. Inspect `plugin_diagnostics` and task logs on failure. Disabling stops automatic refresh; uninstalling preserves source.

The page must run inside a host plugin view. Opening built HTML directly in a browser fails with `API_MISMATCH` because no host bridge is present.

## Add capabilities

See the [SDK package](https://www.npmjs.com/package/blc-plugin-sdk), its [TypeScript declarations](https://github.com/biglazycat-writer/blc-plugin-sdk/blob/main/index.d.ts) and the [English SDK guide](https://github.com/biglazycat-writer/blc-plugin-sdk/blob/main/README.en.md).

- `ctx.credentials` stores secrets through the system credential service; never write them to configuration or logs.
- `readDocument(ctx.documents,target)` reads an immutable Markdown export. Declare `biglazycat.minimumApiRevision: 1` before using this host capability. In-place editing uses `ctx.documents.snapshot/apply`.
- `ctx.workspace` and `ctx.images` access workspace resources and images.
- `ctx.services`, `ctx.events` and `contributes.schedules` support services, events and scheduled commands.
- Add ordinary npm dependencies for UI frameworks, state management, editors or backend networking.

Individual command results must fit within 1 MiB; view state is limited to 64 KiB. Use chunked transfer for large payloads; the official [WeChat publishing plugin](https://github.com/biglazycat-writer/blc-plugin-publish) is one example. Cancellation, crashes and timeouts can leave external outcomes `uncertain`; verify the outcome before retrying. Native modules must match the OS, architecture and Node ABI. Trusted plugin processes are not security sandboxes.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Command mismatch when enabling | Compare manifest IDs with backend registrations |
| `Cannot find module '../dist/main.js'` | Run `npm run build` before tests |
| `API_MISMATCH` | Open the page inside the host plugin view |
| Build failure | Read typecheck output, task logs and `plugin_diagnostics` |
| Engine mismatch | Use Node `>=22.23.3 <23.0.0` |
| SDK installation fails | Check registry access and the `blc-plugin-sdk@1.1.0` dependency |
| State disappears after reload | Check `setState` and the `onSave` persistence callback |

## SDK and platform support

The SDK version is pinned to 1.1.0. Task waits are bounded; query existing tasks after timeout instead of repeating operations with side effects. View build targets are ES2022 / Chromium 109 / Safari 16.4. This update was tested on Windows amd64. Platform declarations and syntax targets do not mean every native WebView and dependency has passed device testing.

## Licensing

The SDK is [MIT licensed](https://github.com/biglazycat-writer/blc-plugin-sdk/blob/main/LICENSE). You may copy and modify the template examples for your own plugin without attribution. Licensing of the repository as a whole remains the maintainer's decision.
