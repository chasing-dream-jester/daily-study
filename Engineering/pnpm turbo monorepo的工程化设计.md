# pnpm turbo monorepo的工程化设计

## 背景
随着前端工程化的发展，越来越多的公司开始使用monorepo来管理代码。pnpm turbo monorepo是一种非常流行的monorepo管理工具，它可以帮助开发者更好地管理代码，提高开发效率。

## 传统架构
- 独立的项目结构
  - 每个项目作为独立的项目存在，每个项目都有自己的代码库，依赖管理，构建流程等。
- 独立的依赖管理  
  - 每个项目都有自己的依赖管理，但这会导致依赖管理的复杂度增加，同时也会导致依赖管理的不一致。
- 独立的构建流程
  - 各个项目有自己的构建流程和构建配置
- 技术栈独立
  - 每个项目可能有不同的工具链和技术栈

### 传统的架构的优缺点
#### 优点
- 独立性
  - 每个项目都是独立的，不会相互影响，各团队各项目都是隔离的。
- 灵活性
  - 可以根据项目的需求选择不同的技术栈和工具链。
- 易于维护
  - 每个项目都是独立的，维护起来比较方便。

#### 缺点
- 依赖管理的复杂性
  - 每个项目都有自己的依赖管理，这会导致依赖管理的复杂性增加。
- 构建流程的复杂性
  - 每个项目都有自己的构建流程，这会导致构建流程的复杂性增加。
- 技术栈的不一致
  - 每个项目可能有不同的技术栈，这会导致技术栈的不一致。
- 代码重复
  - 每个项目都有自己的代码库，这会导致代码重复，很难做到统一管理。
- 依赖版本冲突
  - 每个项目都有自己的依赖版本，这很容易导致依赖版本的冲突。

## pnpm turbo monorepo的架构
### monorepo的概念
monorepo是一种将多个项目放在同一个代码库中的方式。这种方式可以让开发者更好地管理代码，提高开发效率。
- 单一的代码库
  - 所有的项目都放在同一个代码库中，这样可以让开发者更好地管理代码。
- 统一依赖管理
  - 所有的项目都共享同一个依赖管理，这样可以让开发者更好地管理依赖。
- 统一构建、发布流程
  - 所有的项目都共享同一个构建流程，这样可以让开发者更好地管理构建流程。
  - 可以对多个子包一次性构建和发布。
- 代码共享
  - 所有的项目都共享同一个代码库，这样可以让开发者更好地管理代码。
  - 通过工作空间的方式或者内部包机制共享公共模块，可以让开发者更好地管理代码。

### pnpm的概念
pnpm是一种快速、节省磁盘空间的包管理工具。它可以帮助开发者更好地管理依赖，提高开发效率。

#### 优点
- 更快的安装速度
  - pnpm使用了符号链接的方式来管理依赖，这样可以让开发者更快地安装依赖。
- 节省磁盘空间
  - pnpm使用了符号链接的方式来管理依赖，这样可以让开发者节省磁盘空间。
- 支持多版本
  - pnpm支持多版本的依赖管理，这样可以让开发者更好地管理依赖。
- 原生支持monorepo
  - 支持workspace配置，方便子包的依赖管理。

### turbo的概念
Turborepo 是适用于 JavaScript 和 TypeScript 代码库的高性能构建系统。它专为扩展 monorepo 而设计，也使单包工作区中的工作流更快。

Monorepo 有很多优点 - 但它们难以扩展。每个工作区都有自己的测试套件、自己的 linting 和自己的构建过程。单个 monorepo 可能有数千个任务要执行，这会导致构建时间过长。

**monorepo的存在问题**
![monorepo的存在问题](../images/why-turborepo-problem.avif)

Turborepo 通过将构建任务分解为更小的任务，并在需要时并行执行这些任务，从而解决了这个问题。Turborepo 还提供了一个交互式的 CLI，用于查看和调试构建任务。

Turborepo 解决了 monorepo 的扩展问题。缓存到本地文件系统所有任务的结果，这意味着您的 CI 永远不需要重复执行相同的工作

**解决方案**
![turborepo的解决方案](../images/why-turborepo-solution.avif)

## pnpm turbo monorepo的工程化设计

### 工程化的目标
- 统一依赖管理： 通过pnpm的workspace配置，所有子包共享同一个依赖管理，这样可以让开发者更好地管理依赖。
- 高效的构建： 通过turborepo的构建任务分解和并行执行以及按需构建，这样可以让开发者更快地构建项目。
- 模块化管理：组件库、工具库、基础库、业务库通过分包结构实现独立开发与协作。

### 架构示意图
```txt
root
|-- packages
|   |-- ui // 组件库
|   |   |-- package.json
|   |   |-- src
|   |   |-- tsconfig.json
|   |-- utils // 工具库
|   |   |-- package.json
|   |   |-- src
|   |   |-- tsconfig.json
|   |-- cli // 命令行工具
|   |   |-- package.json
|   |   |-- src
|   |   |-- tsconfig.json
|   |-- docs // 文档库
|   |   |-- package.json
|   |   |-- src
|   |   |-- tsconfig.json
|-- .gitignore
|-- .prettierrc
|--. eslintrc
|-- .babelrc
|-- turbo.json
|-- pnpm-workspace.yaml
|-- package.json
|-- tsconfig.json
|-- pnpm-lock.yaml
|-- README.md
```

### 依赖管理以及workspace配置
**pnpm的workspace配置**：pnpm的workspace提供了一种方式来管理多个项目的依赖。通过workspace配置，我们可以让多个项目共享同一个依赖管理，这样可以让开发者更好地管理依赖。核心是通过pnpm-workspace.yaml文件来配置workspace。

#### 配置步骤
1. 在根目录下创建pnpm-workspace.yaml文件
2. 在pnpm-workspace.yaml文件中配置workspace
```yaml
packages:
  - 'packages/*'
  - 'ui/*'
  - 'utils/*'
  - 'cli/*'
  - 'docs/*'
```
3. 每个子包都需要配置package.json文件
```json
{
  "name": "ui",
  "version": "1.0.0",
  "description": "ui package",
  "main": "index.js",
  "scripts": {
    "build": "pnpm build:ui",
    "build:ui": "turbo run build --filter=ui"
  },
  "keywords": [
    "ui"
  ],
  "author": "xxx",
  "license": "MIT",
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```
4. 安装共享依赖
```bash
pnpm install
```
#### 子包使用pnpm的workspace配置机制来添加依赖
```bash
pnpm add utils --filter ui
```
子包之间的依赖通过符号链接实现本地引用，减少重复安装。

pnpm filter 的核心用法
```bash
pnpm --filter <package> <command>
```
常用匹配模式
- 按包名称匹配 （推荐）
  ```bash
  # 安装 react 到 ui 包
  pnpm --filter ui add react

  # 运行 ui 包的 build 命令
  pnpm --filter ui build
  ```
- 按包路径匹配
  ```bash
  # 操作 packages/ui 目录下的包
  pnpm --filter ./packages/ui lint
  ```
- 通配符匹配
  ```bash
  # 操作 packages 目录下的所有包
  pnpm --filter packages/* lint
  ```
### 基于turbo的流水线构建和启动
tasks 对象中的每个 key 都是一个可以通过 turbo run 执行的任务。Turborepo 将在 package.json 中搜索与任务同名的软件包。
  例如，在 package.json 中，你可以这样定义一个任务：
  ```json
  {
    "scripts": {
      "build": "turbo run build"
    }
  }
  ```

  **配置步骤**
  1. 在根目录下创建turbo.json文件
  2. 在turbo.json文件中配置流水线 [任务配置文档](https://turbo.build/repo/docs/crafting-your-repository/configuring-tasks)
  ```json
  {
    "pipeline": {
      "build": {
        "dependsOn": ["^build"],
        "outputs": ["dist/**"]
      },
      "dev": {
        "dependsOn": ["^dev"],
        "outputs": ["dist/**"]
      },
      "test": {
       "dependsOn": ["build"],
        "outputs": ["coverage/**"],
      },
      "lint": {
        "dependsOn": ["^lint"],
        "outputs": ["dist/**"]
      }
    }
  }
  ```
  3. 在package.json中配置任务
  ```json
  {
    "scripts": {
      "build": "turbo run build",
      "dev": "turbo run dev",
      "test": "turbo run test",
      "lint": "turbo run lint",
      "turbo:report": "turbo report"
    }
  }
  ```
  4. 执行任务
  ```bash
  pnpm run build
  ```
  5. 查看任务执行结果
  ```bash
  pnpm run turbo:report
  ```

turbo支持跳过未改动的任务，极大的提升了构建速度

### 代码规范
#### eslint
#### prettier
#### commitlint
#### 自动化检查husky