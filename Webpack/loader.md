## loader
加载器（loader）用于转换模块的源代码。在导入模块之前，加载器可以对其进行转换和处理。loader使得 Webpack 能够处理各种类型的文件，如 JavaScript、CSS、图片、字体、HTML 等。
### module.rules
#### test
test: 一个正则表达式，用于匹配文件路径。匹配的文件会使用指定的加载器进行处理。
#### exclude 
exclude: 排除特定目录或文件，不进行处理。
#### include
include: 只处理特定目录或文件。
```js
{
  test: /\.js$/,
  exclude: /node_modules/, // 排除 node_modules 目录
  include: path.resolve(__dirname, 'src'), // 只处理 src 目录
}
```
#### use
use: 指定使用的加载器，可以是一个字符串（单个加载器）或一个对象（加载器和选项），也可以是一个数组（多个加载器）。
```js
{
  test: /\.js$/,
  use: {
    loader: 'babel-loader', // 使用 babel-loader
    options: {
      presets: ['@babel/preset-env'], // 使用 @babel/preset-env 预设
    },
  },
}
```
对于多个加载器，可以使用数组：
```js
{
  test: /\.css$/,
  use: ['style-loader', 'css-loader'], // 使用 style-loader 和 css-loader 处理 CSS 文件
}
```
#### type
type: 指定模块类型，例如 asset/resource、asset/inline、asset/source 和 asset 等。适用于处理静态资源文件。

```js
{
  test: /\.(png|svg|jpg|jpeg|gif)$/,
  type: 'asset/resource', // 使用资源模块类型处理图片文件
}
```

#### enforce
在某些情况下，你可能需要确保某个加载器在其他加载器之前或之后执行。例如：使用 eslint-loader 在编译之前进行代码检查,使用 source-map-loader 在其他加载器之前加载现有的 source map 文件。
1. "pre" ：在其他加载器之前执行。
2. "post" ：在其他加载器之后执行。


### 常用loader
#### babel-loader
babel-loader用于使用 Babel 转译 JavaScript 代码，比如将es6-->es5;
你需要安装 babel-loader 以及相关的 Babel 包。一般情况下，你还需要安装 @babel/core 和一些预设（如 @babel/preset-env）;
```js
{
  test: /\.js$/, // 匹配所有 .js 文件
  exclude: /node_modules/, // 排除 node_modules 目录
  use: {
    loader: 'babel-loader', // 使用 babel-loader
    options: {
      presets: ['@babel/preset-env'], // 使用 @babel/preset-env 预设
    },
  },
},
```

当然你也可以将 Babel 的配置选项放在 .babelrc 文件中，这样可以使配置更加清晰和模块化。以下是一个示例 .babelrc 文件：
```js
{
   "presets": ["@babel/preset-env", "@babel/preset-react", "@babel/preset-typescript"],
}
```
然后在 Webpack 配置中，你只需指定使用 babel-loader，无需再提供 options:

#### CSS Loader 和 Style Loader
用于处理 CSS 文件，将其嵌入到 JavaScript 中：
```js
{
  test: /\.css$/,
  use: ['style-loader', 'css-loader'],
}
```

#### File Loader 和 URL Loader
用于处理图片、字体等资源文件：
webpack5.0将使用[资源模块](https://webpack.docschina.org/guides/asset-modules/) -- 资源模块(asset module)是一种模块类型，它允许使用资源文件（字体，图标等）而无需配置额外 loader,比如raw-loader 将文件导入为字符串、url-loader 将文件作为 data URI 内联到 bundle 中、file-load将文件发送到输出目录
```js
{
  test: /\.(png|svg|jpg|jpeg|gif)$/,
  type: 'asset/resource', // 使用资源模块类型处理图片文件
}
```
#### TypeScript Loader
用于处理 TypeScript 文件：
```js
{
  test: /\.tsx?$/,
  use: 'ts-loader',
  exclude: /node_modules/,
}
```

#### Sass Loader
用于处理Scss、Sass文件加载器，Sass 是一种 CSS 预处理器，它增加了变量、嵌套规则、混合等高级功能，能够使 CSS 更加模块化和可维护。
```js
{
  test: /\.s[ac]ss$/i, // 匹配所有 .sass 和 .scss 文件
  use: [
   'style-loader', // 将 JS 字符串生成为 style 节点
   'css-loader', // 将 CSS 转化为 CommonJS 模块
    'sass-loader', // 将 Sass 编译成 CSS
  ],
},
```
#### Postcss Loader
通过 PostCSS 运行各种插件来转换 CSS 代码。PostCSS 是一个功能强大的工具，可以使用插件来自动添加浏览器前缀、优化 CSS、处理未来的 CSS 语法等
为了使用postcss-loader，你需要安装 postcss-loader 和 postcss：
```js
{
  test: /\.css$/i,
  use: [
   'style-loader',
   'css-loader',
    {
      loader: 'postcss-loader',
      options: {
        postcssOptions: {
          plugins: [
            [
                'postcss-preset-env', 
                {
                   // 其他选项
                },
              ],
          ],
        },
      },
    },
  ],
}
```
##### postcss-loader常用的插件
1. postcss-preset-env
   postcss-preset-env 是一个非常强大的 PostCSS 插件，它允许你使用最新的 CSS 特性，并将其自动转换为当前浏览器支持的 CSS。
   这个插件包含了许多常用的 PostCSS 插件，并根据浏览器兼容性自动选择合适的转换规则,使用了这个插件就不需要额外按照autoprefixer
2. autoprefixer 用于根据 Can I Use 数据库为 CSS 规则自动添加浏览器前缀。可以确保 CSS 在不同浏览器中具有更好的兼容性，而无需手动编写前缀。
3. postcss-import 允许你使用 @import 语法来导入其他 CSS 文件

##### postcssOptions
1. plugins
   plugins: 一个数组或函数，用于指定 PostCSS 应该使用的插件
   ```js
   postcssOptions:{
     plugins: [
       'postcss-import',
       ['postcss-short', { prefix: 'x' }],
       require.resolve('my-postcss-plugin'),
       myOtherPostcssPlugin({ myOption: true }),
      ],
   }
   ```
2. exec
   exec: 一个布尔值，指示是否应以同步模式执行 PostCSS 插件。默认为 false

3. parser
   parser: 指定自定义的 PostCSS 解析器。通常用于处理非标准的 CSS 语法。
   ```js
    postcssOptions: {
      parser: 'sugarss', // 使用 SugarSS 解析器
    },
   ```
4. syntax 
   syntax: 指定自定义的 PostCSS 语法。通常用于处理特殊的 CSS 语法或扩展。
   ```js
   postcssOptions: {
     syntax: 'postcss-scss', // 使用 SCSS 语法
   },
   ```
5. stringifier
   stringifier : 指定自定义的 PostCSS 字符串化器。通常用于生成非标准的 CSS 输出
   ```js
    postcssOptions: {
      stringifier: 'midas', // 使用 Midas 字符串化器
    }
   ```
##### postcss.config.js
针对于postcss-loader 可以使用 postcss.config.js 文件
```js
module.exports = {
  plugins: [
    require('postcss-import'),
    require('postcss-preset-env')({
      stage: 0,
      autoprefixer: {
        overrideBrowserslist: ['> 1%', 'last 2 versions', 'ie >= 9'],
      },
    }),
  ],
};
```

#### source-map-loader
从现有的源文件中提取 source maps
```js
const path = require('path');

module.exports = {
  entry: './src/index.js', // 入口文件
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.js$/, // 匹配所有 .js 文件
        enforce: 'pre', // 在其他加载器执行之前执行
        use: ['source-map-loader'], // 使用 source-map-loader
      },
    ],
  },
  devtool: 'source-map', // 生成 source map 文件
  // 其他配置项...
};
```
devtool: 'source-map' : 生成 source map 文件，以便浏览器可以映射编译后的代码到原始源代码。
`devtool` 是 Webpack 配置中的一个选项，用于控制 source map 的生成方式。source map 是一种将编译后的代码映射回原始源代码的技术，便于在浏览器中进行调试。通过配置 `devtool`，你可以选择不同类型的 source map，以便在开发和生产环境中进行调试和性能优化。

### 常见的 `devtool` 选项

1. **`eval`**：
   - 生成：每个模块都使用 `eval()` 包裹，并在其后添加 sourceURL。
   - 优点：构建速度最快。
   - 缺点：调试时显示的是编译后的代码，不是原始代码。

2. **`source-map`**：
   - 生成：独立的 `.map` 文件。
   - 优点：生成完整的 source map，调试时可以看到原始代码和完整的错误堆栈。
   - 缺点：构建速度较慢，适合生产环境。

3. **`cheap-source-map`**：
   - 生成：独立的 `.map` 文件，但不包含列信息。
   - 优点：生成较快，调试时显示原始代码。
   - 缺点：不包含列信息，错误定位不够准确。

4. **`eval-source-map`**：
   - 生成：每个模块都使用 `eval()` 包裹，并在其后添加 source map。
   - 优点：生成速度快，调试时显示原始代码。
   - 缺点：适合开发环境，不适合生产环境。

5. **`cheap-module-source-map`**：
   - 生成：独立的 `.map` 文件，不包含列信息，但包含 Loader 的 source map。
   - 优点：生成较快，调试时显示原始代码。
   - 缺点：不包含列信息，错误定位不够准确。

6. **`inline-source-map`**：
   - 生成：将 source map 作为 DataURL 嵌入到输出文件中。
   - 优点：调试时显示原始代码。
   - 缺点：生成速度较慢，适合开发环境。

7. **`hidden-source-map`**：
   - 生成：独立的 `.map` 文件，但不在输出文件中引用。
   - 优点：适合生产环境，可以用于错误报告。
   - 缺点：调试时无法直接查看 source map。

8. **`nosources-source-map`**：
   - 生成：独立的 `.map` 文件，但不包含源代码内容。
   - 优点：适合生产环境，可以用于错误报告。
   - 缺点：调试时无法查看源代码。

开发环境配置

在开发环境中，你可能希望快速生成 source map 以便进行调试：

```javascript
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './src/index.js', // 入口文件
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  devtool: 'eval-source-map', // 快速生成 source map，适合开发环境
  module: {
    rules: [
      {
        test: /\.js$/, // 匹配所有 .js 文件
        exclude: /node_modules/, // 排除 node_modules 目录
        use: 'babel-loader', // 使用 babel-loader
      },
    ],
  },
  // 其他配置项...
};
```

生产环境配置

在生产环境中，你可能希望生成完整的 source map 以便进行错误报告，但不希望将 source map 公开：

```javascript
const path = require('path');

module.exports = {
  mode: 'production',
  entry: './src/index.js', // 入口文件
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  devtool: 'source-map', // 生成完整的 source map，适合生产环境
  module: {
    rules: [
      {
        test: /\.js$/, // 匹配所有 .js 文件
        exclude: /node_modules/, // 排除 node_modules 目录
        use: 'babel-loader', // 使用 babel-loader
      },
    ],
  },
  // 其他配置项...
};
```


完整示例

以下是一个完整的 Webpack 配置示例，展示了如何在开发和生产环境中使用不同的 `devtool` 配置：

```javascript
const path = require('path');

module.exports = (env, argv) => {
  const isProduction = argv.mode === 'production';

  return {
    entry: './src/index.js', // 入口文件
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: 'bundle.js',
    },
    devtool: isProduction ? 'source-map' : 'eval-source-map', // 根据环境选择 source map 类型
    module: {
      rules: [
        {
          test: /\.js$/, // 匹配所有 .js 文件
          exclude: /node_modules/, // 排除 node_modules 目录
          use: 'babel-loader', // 使用 babel-loader
        },
      ],
    },
    // 其他配置项...
  };
};
```
#### html-loader
将 HTML 导出为字符串。当编译器需要时，将压缩 HTML 字符串



TODO:
自定义loader （i18n-loader）后续补上
