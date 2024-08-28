## resolve
resolve 配置项用于配置模块解析的方式,模块解析是指 Webpack 在构建过程中如何找到模块文件的位置。通过配置 resolve，你可以指定模块的查找方式、查找路径、文件扩展名等

### **常用配置**
#### extensions
 * 用于自动解析确定的扩展名，意味着在引入模块时可以省略这些扩展名。
 * 例如，配置了 extensions: ['.js', '.jsx', '.ts', '.tsx'] 后，可以在引入模块时省略 .js、.jsx、.ts 和 .tsx 扩展名。
  ```js
  resolve:{
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
  }
  ```
#### alias
  * 创建模块别名，方便引入模块时使用更简短的路径， 比如：配置了 alias: { '@components': path.resolve(__dirname, 'src/components/') } 后，可以使用 @components 代替 src/components/ 路径
  ```js
  resolve:{
     alias: {
      '@components': path.resolve(__dirname, 'src/components/'),
     },
  }
  ```
#### modules 
  * 指定模块查找目录，默认情况下 Webpack 只会在 node_modules 目录下查找模块 ，比如：配置了 modules: [path.resolve(__dirname, 'src'), 'node_modules'] 后，Webpack 会先在 src 目录下查找模块，如果找不到再去 node_modules 目录下查找
  ```js
  resolve: {
    modules: [path.resolve(__dirname, 'src'), 'node_modules'],
  }
  ```
#### mainFiles
  * 指定在文件夹中查找的主文件名，默认是 ['index'] ,在 Webpack 中，resolve.mainFiles 配置项用于指定在解析目录时要使用的主文件名。这意味着当你在代码中导入一个目录而不是具体的文件时，Webpack 会查找该目录下的指定文件名（例如 index.js）。
  ```js
  resolve:{
    mainFiles: ['index', 'main'],
  }
  ```
  假设你的项目结构如下：
  ```txt
  project
  ├── src
  │   ├── components
  │   │   ├── Button.js
  │   │   └── main.js
  │   └── index.js
  └── webpack.config.js
  ```
  在这种情况下，如果你希望 Webpack 在解析目录 components 时优先查找 main.js 文件而不是 index.js，可以在 webpack.config.js 中进行如下配置
  ```js
  resolve: {
    mainFiles: ['main', 'index'], // 指定主文件名的查找顺序
  },
  ```
#### mainFields 
  * 指定在 package.json 文件中查找的字段，用于确定入口文件，默认是 ['browser', 'module', 'main']
  ```js
  resolve: {
    mainFields: ['browser', 'module', 'main'],
  }
  ```
  * resolve.mainFields 配置项用于指定在解析模块时从 package.json 文件中查找的字段顺序。这个配置项对于确定模块的入口文件非常重要，尤其是在处理第三方库时。
  假设有一个第三方库的 package.json 文件如下：
  ```json
  {
    "name": "example-lib",
    "version": "1.0.0",
    "main": "dist/index.js",
    "module": "esm/index.js",
    "browser": "browser/index.js"
  }
  ```
  在这种情况下，默认的 mainFields 配置会让 Webpack 优先使用 browser/index.js 作为入口文件。这个时候如果需要自定义配置顺序，就可以使用mainFields。

#### symlinks
  * 指定是否遵循符号链接，默认是 true。
  * resolve.symlinks 配置项用于控制 Webpack 在解析模块路径时是否遵循符号链接（symlinks）。符号链接是一种文件系统对象，它指向另一个文件或目录，类似于快捷方式。在一些情况下，特别是在使用包管理工具（如 npm link 或 yarn link）时，项目目录中可能包含符号链接
  ```js
  resolve: {
    symlinks: false,
  }
  ```
  假设你的项目结构如下
  ```txt
  project
  ├── src
  │   └── index.js
  ├── linked-module -> ../another-project/module  (符号链接)
  └── webpack.config.js
  ```
  在这个示例中，linked-module 是一个符号链接，指向 ../another-project/module 目录,当设置为true，（默认为true），配置下，当你在 src/index.js 中导入 linked-module 时，Webpack 会解析并构建 ../another-project/module 目录中的内容，否则是当成普通目录。

#### plugins
  * 用于配置解析插件，这些插件可以扩展或修改模块解析的行为
  * 插件可以扩展或修改 Webpack 的模块解析行为。解析插件可以帮助你自定义模块查找逻辑、优化构建过程、处理特殊的模块解析需求等
  ```js
    resolve: {
      plugins: [
        new TsconfigPathsPlugin({ configFile: "./tsconfig.json" }), // 使用 TsconfigPathsPlugin
      ]
    }
  ```
  常用解析插件
  - TsconfigPathsPlugin

  TsconfigPathsPlugin 是一个常用的解析插件，用于解析 TypeScript 项目中的路径别名。它会读取 tsconfig.json 文件中的 paths 配置，并将其应用到 Webpack 的解析逻辑中。

  使用实例：
  ```json
  //tsconfig.json
  {
    "compilerOptions": {
      "baseUrl": "./src",
      "paths": {
        "@components/*": ["components/*"],
        "@utils/*": ["utils/*"]
      }
    }
  }
  ```
  这样使用的时候可以使用@components、@utils类型别名
