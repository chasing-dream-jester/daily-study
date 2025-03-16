# turbo
## 概念
Turborepo 是适用于 JavaScript 和 TypeScript 代码库的高性能构建系统。它专为扩展 monorepo 而设计，也使单包工作区中的工作流更快。
## monorepo 问题
Monorepo 有很多优点 - 但它们难以扩展。每个工作区都有自己的测试套件、自己的 linting 和自己的构建过程。单个 monorepo 可能有数千个任务要执行，这会导致构建时间过长。
![问题](../images/why-turborepo-problem.avif)

## 解决方案
![解决方案](../images/why-turborepo-solution.avif)

turbo可以通过缓存机制、 并行执行、 按需构建等方式来提高构建速度。

- 缓存机制 ：Turborepo 会缓存每个任务的输出结果。当再次执行相同的任务时，如果输入没有变化，Turborepo 可以直接使用缓存结果，而不需要重新执行任务，从而大大缩短了构建时间。
- 并行执行 ：Turborepo 可以并行执行不相互依赖的任务。这意味着多个任务可以同时运行，进一步提高了构建效率。
- 远程缓存 ：除了本地缓存，Turborepo 还支持远程缓存。团队成员可以共享缓存结果，这样即使在不同的机器上，也可以快速获取之前的构建结果。
- 任务管道 ：Turborepo 允许你定义任务之间的依赖关系，确保任务按照正确的顺序执行。这有助于管理复杂的构建流程。

## 核心
### 配置turbo.json
通过在 Workspace 的根目录中添加 turbo.json 文件来配置 turbo 的行为。

**task**
tasks 对象中的每个 key 都是 turbo run 可以执行的任务名称。Turborepo 将在 Workspace 配置中描述的软件包中搜索package.json 中带有任务名称的脚本。

示例：定义了三个任务 build、test 和 dev
```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "test": {
      "outputs": ["coverage/**"],
      "dependsOn": ["build"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```
**dependsOn**
在任务开始运行之前需要完成的任务列表,depensOn存在三种关系：依赖关系、相同包关系、任意任务关系
- 依赖关系：在 dependsOn 中为字符串加上 ^ 告诉 turbo 该任务必须首先等待 package 依赖项中的任务完成
  ```json
  {
    "tasks": {
        "build": {
           "dependsOn": ["^build"]
        }
    }
  }
  ```
  在这个例子中， build 任务依赖于所有依赖包中的 build 任务。也就是说，只有当所有依赖包的 build 任务都成功执行之后，当前包的 build 任务才会开始执行
- 相同包关系:不带 ^ 前缀的任务名称描述依赖于同一文件包中不同任务的任务
  ```json
  {
    "tasks": {
        "test": {
           "dependsOn": ["lint", "build"]
        }
    }
  }
  ```
  在这个例子中， test 任务依赖于同一包内的 lint 和 build 任务。这意味着在执行 test 任务之前，必须先成功执行 lint 和 build 任务
- 任意任务关系：任意任务关系代表当前任务依赖于任何包中的特定任务。这种关系不局限于依赖包或者同一包，而是可以跨所有包，这种比较灵活。
  ```json
  {
    "tasks": {
        "web#lint": {
           "dependsOn": ["utils#build"]
        }
    }
  }
  ```
  在这个例子中， web#lint 任务依赖于 utils 包中的 build 任务。这意味着在执行 web#lint 任务之前，必须先成功执行 utils 包中的 build 任务。

**outputs**
相对于任务成功完成时要缓存的包package.json的文件 glob 模式列表。
```json
{
  "tasks": {
    "build": {
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    }
  }
}
```
在这个例子中， build 任务的输出结果将被缓存。这意味着在执行 build 任务之后，Turborepo 将缓存 dist 和.next 目录中的文件。

**cache** 
默认值：true
cache 字段用于控制任务的缓存行为。如果 cache 字段的值为 true，则表示该任务的输出结果将被缓存。如果 cache 字段的值为 false，则表示该任务的输出结果不会被缓存。
```json

{
  "tasks": {
    "dev": {
      "cache": false, // No outputs will be cached
      "persistent": true
    }
  }
}
```
**输入**
默认值：[]，包中签入源代码管理的所有文件
```json

{
  "tasks": {
    "build": {
      "inputs": ["src/**"] // Only src/** files will be checked into source control
    }
  }
}
```
**`$TURBO_DEFAULT$`**
使用input会导致退出turbo 的默认行为，可以使用`$TURBO_DEFAULT$`变量来恢复默认行为。这样可以调整默认行为以获得更精细的粒度。
```json
{
  "tasks": {
    "check-types": {
      // Consider all default inputs except the package's README
      "inputs": ["$TURBO_DEFAULT$", "!README.md"]
    }
  }
}
```
`$TURBO_ROOT$`
任务可能会引用位于其目录之外的文件。使用 `$TURBO_ROOT$` 启动文件 glob 会将 glob 更改为相对于存储库的根目录，而不是包目录。
```json
{
  "tasks": {
    "check-types": {
      // Consider all Typescript files in `src/` and the root tsconfig.json as inputs
      "inputs": ["$TURBO_ROOT$/tsconfig.json", "src/**/*.ts"]
    }
  }
}
```

**env**
默认值：{}
env 字段用于设置任务的环境变量。可以在 env 字段中定义环境变量，并将其值设置为字符串或数组。
```json
{
  "tasks": {
    "build": {
      "env": {
        "NODE_ENV": "production",
        "PORT": ["$PORT", 3000]
      }
    }
  }
}
```
在这个例子中， build 任务的环境变量 NODE_ENV 将被设置为 production，PORT 将被设置为数组 ["$PORT", 3000]。如果 $PORT 环境变量不存在，则 PORT 将被设置为 3000。


**persistent**
默认值：false
persistent 字段用于控制任务的持久性。如果 persistent 字段的值为 true，则表示该任务将在后台运行，即使当前终端窗口关闭，任务也会继续执行。如果 persistent 字段的值为 false，则表示该任务将在当前终端窗口中运行，当终端窗口关闭时，任务将停止执行。

此选项对于开发服务器或其他 “watch” 任务最有用。

```json
{
  "tasks": {
    "dev": {
      "persistent": true
    }
  }
}
```
### 配置子包的turbo.json
许多 monorepo 可以在根目录中turbo.json使用适用于所有包的任务描述。但是，有时，monorepo 可以包含需要以不同方式配置其任务的软件包。这个时候Turborepo 允许您使用任何软件包中的 turbo.json 扩展根配置。
#### 使用方式
1. 添加一个turbo.json文件到任何包中
2. 在 turbo.json 文件中添加一个 extends 字段，该字段指向要扩展的 turbo.json 文件的路径。
```json
{
  "extends": ["//"], //目前，extends 键的唯一有效值是 [“//”]，它指向根 turbo.json 文件。
}
```
包中的配置可以覆盖任务的任何配置。任何未包含的键都是从扩展turbo.json继承的
#### 例子
1. 一个工作区中的不同框架

当monorepo中存在多个包时，且每个包的构建产物都不一样时，比如一个Next.js 应用程序，还有一个 SvelteKit 应用程序。这两个框架都使用各自package.json中的构建脚本创建其构建输出。如果将 Turborepo 配置为在根目录下使用单个 turbo.json 运行这些任务，需要如下所示：
```json
{
  "tasks": {
    "build": {
      "outputs": [".next/**", "!.next/cache/**", ".svelte-kit/**"]
    }
  }
}
```
请注意，.next/** 和 .svelte-kit/** 都需要指定为 输出，即使Next.js应用程序不会生成 .svelte-kit 目录，反之亦然。
如果这个时候使用包配置的方式，就可以在每个包中配置自己的 turbo.json 文件，如下所示：
```json
// ./apps/my-next/turbo.json
{
  "extends": ["//"],
  "tasks": {
    "build": {
      "outputs": [".next/**", "!.next/cache/**"]
    }
  }
}
```
根目录的turbo.json，可以改成下面这样,从根配置中删除特定于 next 的输出
```json
{
  "tasks": {
    "build": {
      "outputs": [".svelte-kit/**"]
    }
  }
}
```
2. 特殊情况
假设一个 package 中的构建任务dependsOn 一个 编译任务。你可以普遍地将其声明为 dependsOn： [“compile”]。这意味着您的根turbo.json必须有一个空的编译任务条目。
```json
{
  "tasks": {
    "build": {
      "dependsOn": ["compile"]
    },
    "compile": {}
  }
}
```
使用 Package Configurations，您可以将该编译任务移动到 apps/my-custom-app/turbo.json 中， 然后根目录的 turbo.json 可以删除编译任务。
```json
{
  "extends": ["//"],
  "tasks": {
    "build": {
      "dependsOn": ["compile"]
    },
    "compile": {}
  }
}
```
在 Turborepo 中，`Package Configurations`（包配置）和 `package#task` 这两种 `turbo.json` 语法用于不同场景下对任务进行配置，下面来详细比较它们：

#### `Package Configurations` 和 `package#task` 比较
##### 配置位置
- **Package Configurations**：配置在子包的 `turbo.json` 文件中，适合在包内部进行定制化配置。
- **package#task 语法**：配置在根 `turbo.json` 文件中，适合在根配置中对特定包的任务进行特殊设置。

##### 复用性
- **Package Configurations**：通过 `extends` 字段可以方便地复用根配置，同时对特定包的任务进行覆盖。
- **package#task 语法**：直接在根配置中定义特定包的任务，可能会导致根配置文件变得复杂，但可以集中管理所有包的任务配置。

##### 适用场景
- **Package Configurations**：当某个包的任务配置与其他包有较大差异，需要在包内部进行详细定制时使用。
- **package#task 语法**：当需要在根配置中对特定包的任务进行微调，而不需要为每个包单独创建 `turbo.json` 文件时使用。 