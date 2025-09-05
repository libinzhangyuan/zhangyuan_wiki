# Ruby LSP（Ruby 语言服务器协议）
Ruby LSP 是适用于 Ruby 语言的**语言服务器协议（Language Server Protocol, LSP）** 实现，用于增强代码编辑器中的丰富功能支持。


## 功能（Features）
### Ruby LSP 功能演示（Ruby LSP demo）
```
Ruby LSP 包含以下功能：
- 语义高亮（Semantic highlighting）
- 符号搜索与代码大纲（Symbol search and code outline）
- RuboCop 错误与警告提示（诊断功能，RuboCop errors and warnings (diagnostics)）
- 保存时自动格式化（支持 RuboCop 或 Syntax Tree，Format on save (with RuboCop or Syntax Tree)）
- 输入时自动格式化（Format on type）
- 调试支持（Debugging support）
- 通过 VS Code 界面运行与调试测试（Running and debugging tests through VS Code's UI）
- 跳转到定义（Go to definition）
- 悬停时显示文档说明（Showing documentation on hover）
- 代码补全（Completion）
- 项目内任意位置的模糊搜索声明（工作区符号，Fuzzy search declarations anywhere in the project (workspace symbol)）
- 通过界面运行 Rails 生成器（Running Rails generators from the UI）
```
如需查看功能完整说明，请点击[此处](https://ruby-lsp.org/features)。

若遇到问题，请参考[故障排除指南](https://ruby-lsp.org/troubleshooting)。


## 【实验性】GitHub Copilot 聊天代理（[Experimental] GitHub Copilot chat agent）
对于 Copilot 用户，Ruby LSP 提供了一个 Ruby 代理，用于支持 AI 辅助的 Ruby 应用开发。下方为 Ruby 代理各命令的文档说明。如需了解如何与 Copilot Chat 交互，请查阅 [VS Code 官方文档](https://code.visualstudio.com/docs/copilot/copilot-chat)。

### Design 命令（Design command）
`@ruby /design` 命令定位为**领域驱动设计（Domain-Driven Design, DDD）专家工具**，帮助用户为 Rails 应用建模概念。用户需描述正在构建的应用类型，以及希望建模的概念。该命令会读取用户 Rails 应用的数据库架构，并结合用户的提示词、历史交互记录及架构信息，提供应用设计建议。

**示例**：
```
@ruby /design 我正在开发一个面向学校的 Web 应用。
如何为“课程”建模？课程与学生之间应如何关联？
```

命令输出会包含“课程”的建议架构（含与用户的关联关系）。在聊天窗口中，将显示两个按钮：
- **Generate with Rails**：调用 Rails 生成器，创建建议的模型文件；
- **Revert previous generation**：删除上一次点击“生成”按钮后创建的文件。

与大多数大语言模型（LLM）聊天功能类似，建议可能并非完全准确（尤其在第一次交互时）。用户可继续与 `@ruby` 代理对话，微调建议内容，再决定是否执行生成操作。

如对该功能有反馈，可通过 **DX Slack** 告知我们，或创建 [Issue](https://github.com/Shopify/ruby-lsp/issues) 提交反馈。


## 使用方法（Usage）
在扩展面板中搜索 **Shopify.ruby-lsp**，点击“安装”即可。

默认情况下，Ruby LSP 会生成一个 `.ruby-lsp` 目录，其中包含一个“组合包（composed bundle）”，内含服务端 gem 包。此外，它会尝试使用已安装的版本管理器，为当前项目选择合适的 Ruby 版本。更多配置选项请参考“配置（Configuration）”部分。


## 命令（Commands）
以下是可用命令列表，在命令面板（默认快捷键：CMD + SHIFT + P）中搜索“Ruby LSP”前缀即可找到所有命令：
```
| 命令（Command） | 描述（Description） |
|----------------|--------------------|
| Ruby LSP: Start | 启动 Ruby LSP 服务端 |
| Ruby LSP: Restart | 重启 Ruby LSP 服务端 |
| Ruby LSP: Stop | 停止 Ruby LSP 服务端 |
| Ruby LSP: Update language server gem | 将 ruby-lsp 服务端 gem 更新至最新版本 |
```

## 代码片段（Snippets）
该扩展为 Ruby 常用语法结构提供便捷代码片段，例如代码块（blocks）、类（classes）、方法（methods），甚至单元测试模板（unit test boilerplates）。完整代码片段列表请点击[此处](https://ruby-lsp.org/snippets)查看。


## 配置（Configuration）
### 启用/禁用功能（Enable or disable features）
Ruby LSP 支持禁用特定功能。操作步骤：点击语言模式“Ruby”右侧的**语言状态中心（language status center）**，在“已启用功能（enabled features）”右侧选择“管理（Manage）”。

#### Ruby LSP 状态中心（Ruby LSP status center）


### Ruby 版本管理器（Ruby version managers）
为确保服务端正常启动，Ruby LSP 会使用版本管理器激活正确的环境变量，这些变量将 Bundler 指向 Ruby 和 gem 的路径。当在使用不同 Ruby 版本的项目间切换时，此步骤尤为重要——因为路径会发生变化，需重新激活。

默认情况下，Ruby LSP 会自动检测应使用的版本管理器（检查可用管理器，对应“auto”选项）。若自动检测失败，则需手动配置版本管理器：
1. 在语言状态中心点击“Change version manager”；
2. 或修改 VS Code 用户设置。

```json
// 可用选项：
// "auto"（自动选择版本管理器）
// "none"（不使用版本管理器）
// "custom"（使用 rubyLsp.customRubyCommand 查找/激活 Ruby）
// "asdf"、"chruby"、"rbenv"、"rvm"、"shadowenv"、"mise"
{
  "rubyLsp.rubyVersionManager": {
    "identifier": "chruby", // 示例：使用 chruby 作为版本管理器
  },
}
```

为确保 Ruby LSP 能找到版本管理器脚本，请确保：
- 脚本已加载到 Shell 配置文件中（如 `~/.bashrc`、`~/.zshrc`）；
- `SHELL` 环境变量已设置，并指向默认 Shell。

> [!NOTE]
> 关于特定管理器的说明、为未列出的管理器配置自定义激活方式，以及社区贡献的示例，请参考[版本管理器文档](https://ruby-lsp.org/configuration#ruby-version-managers)。


### 配置格式化工具（Configuring a formatter）
可通过以下设置配置用于文件格式化的工具：
```json
// 可用选项：
// "auto"：根据应用的 bundle 自动检测格式化工具（默认）
// "none"：不使用格式化工具（禁用“保存时格式化”及相关诊断）
// 其他选项为格式化工具名称（如 "rubocop" 或 "syntax_tree"）
"rubyLsp.formatter": "auto"
```


### Ruby 版本要求（Ruby version requirement）
默认情况下，Ruby LSP 使用当前项目的 Ruby 版本和 bundle，确保：
- LSP 能索引正确的 gem 版本；
- 格式化行为与持续集成（CI）环境一致。

Ruby LSP 及其主要依赖项 **Prism（新一代 Ruby 解析器）** 遵循相同的版本支持策略：仅支持尚未“生命周期结束（end-of-life）”的 Ruby 版本。

若项目使用较旧的 Ruby 版本：
1. 可尝试安装旧版本的服务端 gem，以获得对旧 Ruby 版本的支持，但可能还需使用旧版本的 VS Code 扩展（因部分功能需客户端与服务端协同实现）；
2. 或使用与项目分离的自定义 Gemfile，指定不同的 Ruby 版本（需注意：部分功能可能降级，或需手动配置——因为 Ruby LSP 无法检查项目实际 bundle 以发现依赖）。

#### 使用自定义 Gemfile（Using a custom Gemfile）
若项目使用的旧 Ruby 版本不被 Ruby LSP 支持，可指定一个单独的 Gemfile 用于开发工具。

> **注意**：使用自定义 Gemfile 时，gem 不会自动安装，ruby-lsp 也不会自动更新。

操作步骤：
1. 在使用旧 Ruby 版本的项目外部，创建一个目录用于存储“组合包（composed bundle）”，并在该目录中添加版本管理器配置，选择受支持的 Ruby 版本。例如，使用 chruby 时，目录中需创建 `.ruby-version` 文件：
   ```
   # the/directory/.ruby-version
   3.2.2 # 示例：指定受支持的 Ruby 版本
   ```

2. 在该目录中创建用于开发工具的 Gemfile：
   ```ruby
   # the/directory/Gemfile
   source "https://rubygems.org"

   gem "ruby-lsp"
   gem "rubocop" # 示例：添加所需的格式化工具
   ```

   > [!NOTE]
   > 格式化工具、代码检查工具（linters）及其扩展需包含在自定义 Gemfile 中，可能需添加更多 gem，例如使用 rubocop 时：
   > ```ruby
   > gem "rubocop-packaging"
   > gem "rubocop-performance"
   > gem "rubocop-rspec"
   > gem "rubocop-shopify"
   > gem "rubocop-thread_safety"
   > ```

3. 在该目录中运行 `bundle install`，生成锁文件（lockfile）。

4. 在 VS Code 中添加以下配置，将 Ruby LSP 指向该自定义 Gemfile：
   ```json
   {
     "rubyLsp.bundleGemfile": "../../path/to/the/directory/Gemfile", // 相对路径或绝对路径均可
   }
   ```


### 配置 VS Code 调试器（Configuring VS Code debugger）
配置 VS Code 调试器需执行以下步骤：
1. 运行“Debug: Add configuration...”命令；
2. 在项目的 `.vscode` 目录中创建 `launch.json` 文件。

该命令会生成以下默认配置：
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "ruby_lsp",
      "name": "Debug",
      "request": "launch",
      "program": "ruby ${file}", // 调试当前打开的文件
    },
    {
      "type": "ruby_lsp",
      "request": "launch",
      "name": "Debug test file",
      "program": "ruby -Itest ${relativeFile}", // 调试当前测试文件
    },
    {
      "type": "ruby_lsp",
      "request": "attach",
      "name": "Attach to existing server", // 附加到已存在的服务端
    },
  ],
}
```

#### 调试运行中的进程（Debugging live processes）
若无需每次启动新进程进行调试，可将 VS Code 调试器附加到已运行的进程（如 Rails 服务端），操作步骤如下：
1. 安装 `debug` gem，通过 `bundle exec rdbg -v` 验证安装；
2. 启动应用并附加调试器，以便扩展能连接到进程：
   ```bash
   bundle exec rdbg -O -n -c -- bin/rails server -p 3000
   ```
3. 如需更好地集成 Rails 测试支持，还需安装 `ruby-lsp-rails` gem。


### VS Code 配置（VS Code configurations）
除 Ruby LSP 自身配置外，部分 VS Code 全局设置可能需调整，以充分发挥 Ruby LSP 的功能。这些设置不特定于 Ruby LSP，但会影响所有语言服务器，且优先级高于其他配置。

以下是可能影响 Ruby LSP 行为的设置及说明（仅作用于 Ruby 语言）：
```json
{
  "[ruby]": {
    "editor.defaultFormatter": "Shopify.ruby-lsp", // 将 Ruby LSP 设为默认格式化工具
    "editor.formatOnSave": true, // 保存文件时自动格式化
    "editor.tabSize": 2, // 缩进使用 2 个空格
    "editor.insertSpaces": true, // 使用空格而非制表符（Tabs）缩进
    "editor.semanticHighlighting.enabled": true, // 启用语义高亮
    "editor.formatOnType": true, // 输入时自动格式化
  },
}
```


### 多根工作区（Multi-root workspaces）
多根工作区是 VS Code 提供的功能，允许用户将单个代码仓库组织为多个独立的“关注点（concerns）”。需注意，它不一定与“项目”概念完全一致——例如，一个包含前端（React）和后端（Rails）的 Web 应用，可将前端和后端目录配置为不同的工作区，但仍视为一个项目。

#### 多根工作区的优势
- 需与项目依赖（如 Gemfile、package.json）集成的扩展（调试器、语言服务器、格式化工具等），能自动识别依赖文件的位置；
- 若 `launch.json` 配置放在某个工作区中，VS Code 会自动从对应目录启动（无需手动指定 `cwd`，Ruby LSP 示例）；
- 打开终端时，VS Code 会提供“在所有配置的工作区中打开终端”的选项。

#### Ruby LSP 对多根工作区的支持
Ruby LSP 通过为每个工作区启动独立的语言服务端来支持多根工作区。这种策略优于“单个服务端支持多工作区”——因为不同工作区可能使用不同的 Ruby 版本和完全不同的 gem，而单个 Ruby 进程无法同时支持这些差异。

#### 关键配置点
需明确代码仓库中每个工作区的“主 Gemfile”位置。

#### 示例配置
> [!NOTE]
> 如需确保 Ruby LSP 在多根工作区中正常工作，请参考以下示例配置，并通过“code-workspace”文件打开工作区。

**示例 1：包含 Rails 后端和 React 前端的项目**  
项目结构：
```
my_project/
  frontend/ # React 前端目录
  Gemfile   # Rails 后端的 Gemfile
  Gemfile.lock
  config.ru
  super_awesome_project.code-workspace # 工作区配置文件
```

`super_awesome_project.code-workspace` 配置：
```json
{
  "folders": [
    // 根目录：包含 Rails 后端
    {
      "name": "rails",
      "path": ".",
    },
    // frontend 目录：包含 React 前端
    {
      "name": "react",
      "path": "frontend",
    },
  ],
  "settings": {
    // 排除 frontend 目录，避免文件在根工作区中重复显示（仅在 react 工作区中显示）
    "files.exclude": {
      "frontend": true,
    },
  },
}
```

**示例 2：客户端与服务端分离的单体仓库（Monorepo）**  
项目结构：
```
my_project/
  client/   # 客户端目录
  server/   # 服务端目录（含 Gemfile）
    Gemfile
    Gemfile.lock
  super_awesome_project.code-workspace # 工作区配置文件
```

`super_awesome_project.code-workspace` 配置：
```json
{
  "folders": [
    // 根目录：包含文档或构建文件
    {
      "name": "awesome_project",
      "path": ".",
    },
    // client 目录：客户端
    {
      "name": "client",
      "path": "client",
    },
    // server 目录：服务端（含 Ruby 依赖）
    {
      "name": "server",
      "path": "server",
    },
  ],
  "settings": {
    // 排除 client 和 server 目录，避免在根工作区中重复显示
    "files.exclude": {
      "server": true,
      "client": true,
    },
  },
}
```

如需了解更多信息，请查阅 VS Code 官方文档：
- [工作区文档](https://code.visualstudio.com/docs/editor/workspaces)
- [多根工作区文档](https://code.visualstudio.com/docs/editor/multi-root-workspaces)


### 容器内开发（Developing on containers）
Ruby LSP 是“分离式语言服务器（detached language server）”，即作为后台进程独立于 VS Code 运行。要提供完整功能，Ruby LSP 必须与项目文件和已安装的依赖在**同一环境中运行**。

VS Code 原生支持连接容器，可实现编辑器功能（语言服务器、集成终端等）的无缝使用。VS Code 官方文档提供了“本地/远程容器内开发”的详细指南，在提交 Issue 前，请先参考以下资源：
- [容器内开发（Developing inside a Container）](https://code.visualstudio.com/docs/devcontainers/containers)
- [高级容器配置（Advanced container configuration）](https://code.visualstudio.com/docs/devcontainers/advanced-config)

> **注意**：Dev Container 扩展仅官方支持 Docker 作为后端。<sup>[1]</sup>


### 遥测（Telemetry）
Ruby LSP 默认不收集任何遥测数据，但支持连接私有指标服务（如需了解团队/公司内部 Ruby LSP 的采用率、性能或错误情况）。

#### 配置遥测数据收集
需通过另一个 VS Code 扩展（通常是私有扩展）定义 `getTelemetrySenderObject` 命令。该命令需返回一个实现 `vscode.TelemetrySender` 接口的对象，以指定数据和错误报告的发送目标。

**示例（私有 VS Code 扩展）**：
```typescript
// 私有 VS Code 扩展代码