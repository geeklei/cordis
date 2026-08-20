# 仓库指南

## 项目结构与模块组织

- 本仓库是由 Yarn v4 workspaces 与 Yakumo 管理的 monorepo；每个包位于 `packages/<name>`。
- 每个包的源码位于 `src/`，测试位于 `tests/`，例如 `packages/core/src/context.ts` 与 `packages/core/tests/fiber.spec.ts`。
- `packages/core`（发布名为 `cordis`）是运行时元框架；`loader`、`hmr`、`create`、`include`、`group`、`logger-console`、`timer`、`utils` 提供独立功能。
- 根级配置文件包括 `tsconfig.base.json`、`vitest.config.ts`、`.eslintrc.yml`、`yakumo.yml`；CI 工作流位于 `.github/workflows/build.yml`。

## 构建、测试与开发命令

- `corepack enable` — 启用 `packageManager` 中固定的 Yarn v4 版本。
- `yarn --no-immutable` — 安装依赖（与 CI 使用的命令一致）。
- `yarn lint` — 使用 `@cordisjs/eslint-config` 运行 ESLint。
- `yarn build` — 通过 Yakumo 使用 esbuild 打包，并用 tsc 生成类型声明；可指定包名仅构建单个包，如 `yarn build core`。
- `yarn test` — 通过 Yakumo 运行全部 Vitest 测试。
- `yarn test:text|json|html` — 运行测试，并输出指定格式的覆盖率报告。

请优先使用根级 `yarn` 脚本，而非直接调用 `vitest` 或 `tsc`：`vitest.config.ts` 注入了 `tsx` 与 `@cordisjs/unyaml` 参数，以保证环境一致。

## 编码风格与命名约定

- 使用 TypeScript ESM，并开启严格编译选项（`tsconfig.base.json` 中的 `strict: true`）。
- 两个空格缩进、LF 换行、UTF-8 编码、文件末尾保留换行，由 `.editorconfig` 约束。
- 使用 ESLint 与 `@cordisjs/eslint-config` 进行代码检查；提交前运行 `yarn lint`。
- 包名与文件名使用 kebab-case（如 `logger-console`）；源码模块按职责命名（如 `context.ts`、`events.ts`、`fiber.ts`）。

## 测试指南

- 测试使用 Vitest，位于 `packages/<name>/tests`，命名为 `<subject>.spec.ts`。
- 运行单个文件：`yarn test packages/core/tests/fiber.spec.ts`；按名称过滤：`yarn test -t 'pattern'`。
- 借助 `@cordisjs/unyaml`，代码与测试可直接导入 YAML 夹具（`.yml`）。
- 覆盖率报告由 `yarn test:*` 脚本生成；`.nycrc.json` 将 spec 文件与 `.yarn` 排除在覆盖率统计之外。

## 提交与拉取请求指南

- 遵循 Conventional Commits 规范：`feat`、`fix`、`chore`、`refactor`、`test`、`docs`、`build`、`perf`、`types`。
- 涉及具体包时使用作用域，例如 `fix(core): keep wrapped fiber state canonical (#40)`。
- 拉取请求需使用符合规范的描述性标题，说明变更摘要并关联相关 issue。
- CI 会在 Node 24 与 26 上运行 lint、构建和测试，全部通过后才能合并。
- 发布由 CI 在 `main` 分支上通过 `yarn yakumo publish` 完成；请勿手动发布。