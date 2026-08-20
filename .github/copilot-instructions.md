目的

本文件为 Copilot CLI/代理在此仓库中高效工作的简明参考：包含构建、测试、lint 的运行方法；高层架构概览；以及在编辑、测试或执行任务时应遵循的仓库特有约定。

1）构建、测试与 lint（本地运行）

前置条件
- Node.js 24+（CI 在 24 和 26 上测试）。需启用 Corepack 以使用 Yarn v4。
- 运行：corepack enable && yarn --no-immutable

常用命令（仓库根目录）
- 安装依赖：yarn --no-immutable
- 静态检查：yarn lint（执行 eslint --cache，继承 @cordisjs/eslint-config）
- 构建（整个工作区）：yarn build（内部调用 yakumo esbuild && yakumo tsc）
- 构建单个包（CI 常用）：yarn build <package-name>（CI 使用 yarn build core）
- 运行测试（通过 Yakumo 调用 Vitest）：yarn test
- 运行单个测试文件：yarn test path/to/file.spec.ts（Vitest 支持文件路径）
- 按测试名称匹配运行：yarn test -t "pattern"
- 覆盖率输出：yarn test:text | yarn test:json | yarn test:html
- 直接调用 Vitest（高级用法）：yarn yakumo vitest --import tsx [-- <vitest-args>]

测试运行器注意事项
- 项目使用带 unyaml 插件的 vitest；vitest.config.ts 在 execArgv 中注入 ['--expose-internals','--import','tsx','--import','@cordisjs/unyaml']，建议优先使用 yarn test 包装脚本以保持一致环境。

2）高层架构（宏观视角）

- Monorepo（Yarn workspaces）：包位于 packages/*，每个包为一个小型 TypeScript ESM 模块。
- packages/core（"cordis"）：运行时 / 元框架，核心模块包括 context、events、fiber、registry、service，构成时空可组合运行时原语。
- packages/loader：插件/配置加载器（entry/group/isolate/tree），用于组装应用/插件图。
- packages/hmr：热重载支持与本地化消息（测试使用 YAML 夹具）。
- packages/create：脚手架（create-cordis CLI）。
- 其它包：utils、timer、logger-console、group、include 等，负责独立功能。
- 构建/测试流程：yakumo 协调 esbuild（打包）和 tsc（生成声明），yakumo-vitest 运行测试；yakumo.yml 列出使用的 yakumo 插件。
- TypeScript 路径别名在 tsconfig.json 中配置（例如 @cordisjs/* -> ./packages/*/src）。

3）关键约定与仓库特有模式

- 包结构：开发时可能依赖 exports 映射（./src/*）；编辑时优先修改 src/ 下源文件，并通过根级 yarn 脚本运行构建/测试。
- 测试放置：位于 packages/*/tests 中，文件后缀为 .spec.ts；nyc 默认排除 spec 文件。
- YAML 导入：代码和测试可直接导入 .yml 夹具，依赖 @cordisjs/unyaml（vite 插件 + vitest 配置）。新增或修改此类测试时，确保保留插件及 import 标志。
- Yakumo：CI 与脚本使用 yakumo 封装（esbuild/tsc/vitest），通常通过 yarn 脚本调用 yakumo，避免直接单独调用底层工具，除非确有需要。
- 现代 Yarn：仓库针对 Yarn v4 行为（如 PnP/immutability）；本地与 CI 保持一致请使用 corepack enable 与 yarn --no-immutable。
- Lint：eslint 配置继承 @cordisjs/eslint-config；在根目录运行 yarn lint。
- 发布：CI 使用 yarn yakumo publish；请勿在不了解 yakumo 配置的情况下直接手动发布。

4）建议查看的文件（快速指引）
- vitest.config.ts — 测试运行时标志（unyaml + execArgv）
- yakumo.yml — 根脚本使用的 yakumo 插件列表
- tsconfig.json / tsconfig.base.json — 路径别名与编译器选项
- .github/workflows/build.yml — CI 流程和 Node 矩阵

5）已检查的 AI 助手/其它助手配置
- 未检测到 CLAUDE.md、.cursorrules、AGENTS.md、.windsurfrules、CONVENTIONS.md 或 .clinerules。如未来添加，请将其运行规则合并到此文件。

总结
本文件列出了 Copilot 会话应使用的确切命令、Monorepo 的高层结构以及仓库约定（yakumo、unyaml、vitest 标志、Yarn v4）。默认操作请使用根级 yarn 脚本（yarn lint、yarn build、yarn test）。

如需扩展（例如加入按包的示例命令、更详细的测试调用示例或其它工作流快捷方式），请说明要补充的区域。
