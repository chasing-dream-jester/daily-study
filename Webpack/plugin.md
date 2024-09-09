# plugin
webpack它有一个强大且灵活的插件系统，允许开发者通过插件来扩展其功能。插件可以在 Webpack 的构建流程中执行各种任务，如优化打包、生成额外的文件、注入环境变量等。

webpack 插件是一个具有 apply 方法的 JavaScript 对象。apply 方法会被 webpack compiler 调用，并且在 整个 编译生命周期都可以访问 compiler 对象

## 配置方式
```js
plugins: [
    new webpack.ProgressPlugin(),
    new HtmlWebpackPlugin({ template: './src/index.html' }),
],
```

## 常用的plugin

### [HtmlWebpackPlugin](https://github.com/jantimon/html-webpack-plugin#options)
HtmlWebpackPlugin 简化了 HTML 文件的创建，以便为你的 webpack 包提供服务。这对于那些文件名中包含哈希值，并且哈希值会随着每次编译而改变的 webpack 包特别有用。你可以让该插件为你生成一个 HTML 文件，使用 lodash 模板提供模板，或者使用你自己的 loader

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const path = require('path');

module.exports = {
  entry: 'index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'index_bundle.js',
  },
  plugins: [new HtmlWebpackPlugin()],
};

```

### DefinePlugin
DefinePlugin 允许在 编译时 将你代码中的变量替换为其他值或表达式。这在需要根据开发模式与生产模式进行不同的操作时，非常有用。例如，如果想在开发构建中进行日志记录，而不在生产构建中进行，就可以定义一个全局常量去判断是否记录日志。这就是 DefinePlugin 的发光之处，设置好它，就可以忘掉开发环境和生产环境的构建规则。

```txt
传递给 DefinePlugin 的每个键都是一个标识符或多个以 . 连接的标识符。
如果该值为字符串，它将被作为代码片段来使用。
如果该值不是字符串，则将被转换成字符串（包括函数方法）。
如果值是一个对象，则它所有的键将使用相同方法定义。
如果键添加 typeof 作为前缀，它会被定义为 typeof 调用。
```
```js
new webpack.DefinePlugin({
  PRODUCTION: JSON.stringify(true),
  VERSION: JSON.stringify('5fa3b9'),
  BROWSER_SUPPORTS_HTML5: true,
  TWO: '1+1',
  'typeof window': JSON.stringify('object'),
  'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
});
```
```js
console.log('Running App version ' + VERSION);
if (!BROWSER_SUPPORTS_HTML5) require('html5shiv');
```

注意事项

- 字符串化 ：确保你传递给 DefinePlugin 的值是字符串化的（使用 JSON.stringify），否则它们会被当作代码片段直接插入到代码中，这可能会导致意外的行为。
- 作用范围 ：DefinePlugin 的替换是全局的，即它会替换所有代码中出现的指定标识符。因此，确保标识符的名称是唯一且明确的，以避免意外替换。
- 性能优化 ：在生产环境中使用 DefinePlugin 注入 process.env.NODE_ENV 为 'production' 是一种常见的性能优化方法，因为许多库（如 React）会根据 process.env.NODE_ENV 的值进行优化。
  
通过使用 DefinePlugin，你可以在编译时注入全局常量，从而使你的代码更加灵活和可配置。这在处理不同的构建环境和配置选项时特别有用。

在为 process 定义值时，'process.env.NODE_ENV': JSON.stringify('production') 会比 process: { env: { NODE_ENV: JSON.stringify('production') } } 更好，后者会覆盖 process 对象，这可能会破坏与某些模块的兼容性，因为这些模块会在 process 对象上定义其他值

### EnvironmentPlugin
EnvironmentPlugin 是在 process.env 键上使用 DefinePlugin 的简写
```js
new webpack.EnvironmentPlugin(['NODE_ENV', 'DEBUG'])
```
这相当于以下 DefinePlugin 应用程序
```js
new webpack.DefinePlugin({
  'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
  'process.env.DEBUG': JSON.stringify(process.env.DEBUG),
});
```

## HotModuleReplacementPlugin
Hot Module Replacement，除此之外还被称为 HMR

**注意：HMR 绝对不能被用在生产环境**

启用 HMR 很容易，且在大多数情况下不需要任何配置
```js
new webpack.HotModuleReplacementPlugin({
  // Options...
});
```

### MinChunkSizePlugin
通过合并小于 minChunkSize 的块，将块大小保持在指定限制之上
```js
new webpack.optimize.MinChunkSizePlugin({
  minChunkSize: 10000, // Minimum number of characters
});
```

### ProvidePlugin
用于自动加载模块，而不必在每个文件中显式地 import 或 require 这些模块。这在使用一些全局依赖时非常有用，例如 jQuery、lodash 等库。
```txt
identifier：一个字符串，表示要在代码中使用的变量名。
module：一个字符串，表示要加载的模块。如果模块有默认导出，可以直接使用模块名；如果模块没有默认导出，可以使用数组形式指定模块路径和属性。
```
注意事项：
- 作用范围 ：ProvidePlugin 会在每个模块中自动加载指定的模块，并将其赋值给指定的变量。这意味着你不需要在每个文件中显式地导入这些模块，但它们仍然会被打包到最终的输出文件中。
- 性能考虑 ：虽然 ProvidePlugin 可以简化代码，但它也可能会增加打包的体积，特别是当自动加载的模块非常大时。因此，使用时需要权衡便利性和性能之间的关系。
- 变量名称冲突 ：确保你指定的变量名称不会与代码中的其他变量名称冲突，以避免意外的行为。

### [SourceMapDevToolPlugin](https://webpack.docschina.org/plugins/source-map-dev-tool-plugin/)
用于生成源码映射（Source Map）。源码映射是一种调试工具，可以将打包后的代码映射回源代码，从而使得在浏览器开发者工具中调试代码时，可以看到原始的源代码，而不是打包后的代码
```txt
参数
options：一个对象，包含配置选项。
常用选项
filename：指定生成的 Source Map 文件名。可以使用占位符，如 [file]、[id]、[hash] 等。
append：在打包后的文件中追加 Source Map 注释。可以使用占位符，如 //# sourceMappingURL=[url]。
moduleFilenameTemplate：指定模块的 Source Map 中的文件名模板。
fallbackModuleFilenameTemplate：指定模块的 Source Map 中的备用文件名模板。
publicPath：指定 Source Map 文件的公共路径。
fileContext：指定文件上下文。
```
```js
// 自定义 Source Map 注释 ：
new webpack.SourceMapDevToolPlugin({
  filename: '[file].map',
  append: '\n//# sourceMappingURL=[url]' // 在打包后的文件中追加 Source Map 注释
})

// 指定公共路径
new webpack.SourceMapDevToolPlugin({
  filename: '[file].map',
  publicPath: '/sourcemaps/' // 指定 Source Map 文件的公共路径
})
```
注意事项
- 性能 ：生成 Source Map 可能会影响构建速度，特别是在大型项目中。因此，仅在开发环境中使用 Source Map。
- 安全性 ：在生产环境中使用 Source Map 可能会泄露源代码。如果需要在生产环境中使用 Source Map，请确保配置正确的选项，如隐藏源代码或仅暴露必要的信息。
- 占位符 ：在 filename、moduleFilenameTemplate 等选项中，可以使用占位符，如 [file]、[id]、[hash] 等，以动态生成文件名。


### SplitChunksPlugin
用于优化代码分割（Code Splitting）。它可以将公共代码抽取到单独的文件中，从而减少重复代码，优化加载性能。SplitChunksPlugin 在 Webpack 4 及更高版本中默认启用，并通过配置 optimization.splitChunks 选项来使用。

SplitChunksPlugin 的配置选项可以在 optimization.splitChunks 中设置,详细请看[optimization](./optimization.md)

### DllPlugin

### CopyWebpackPlugin
用于将文件和目录从源目录复制到构建目录。它在处理静态资源（如图像、字体、HTML 文件等）时特别有用，因为这些资源通常不需要通过 Webpack 的模块系统进行处理。

```txt
配置选项
patterns：一个数组，每个元素是一个对象，定义了要复制的源路径和目标路径。
from：源路径，可以是相对路径或绝对路径。
to：目标路径，可以是相对路径或绝对路径。如果未指定，默认是构建目录。
globOptions：一个对象，包含与 glob 匹配相关的选项，例如 ignore 用于忽略某些文件。
transform：一个函数，用于在写入目标路径之前转换文件内容。
```

```js
plugins: [
  new CopyPlugin({
    patterns: [
      { from: 'source', to: 'dest' },
      { from: 'other', to: 'public' },
    ],
  }),
]

// 忽略某些文件
plugins: [
  new CopyWebpackPlugin({
    patterns: [
      {
        from: 'public',
        to: 'dist',
        globOptions: {
         ignore: ['**/ignore-me.txt'] // 忽略 'public' 目录下的 'ignore-me.txt' 文件
        }
      }
    ]
  })
]

// 使用 transform 转换文件内容
plugins: [
  new CopyWebpackPlugin({
    patterns: [
      {
        from: 'public/index.html',
        to: 'dist/index.html',
        transform(content, path) {
        // 在复制文件之前转换文件内容
          return content.toString().replace(/{{title}}/g, 'My App');
        }
      }
    ]
  })
]
```
注意事项
- **性能**：复制大量文件可能会影响构建速度，因此在使用时需要权衡性能和需求。
- **路径**：确保 from 和 to 路径的正确性，以避免文件复制到错误的位置。
- **文件权限**：在某些操作系统上，复制文件时可能需要处理文件权限问题
### MiniCssExtractPlugin
用于将 CSS 从 JavaScript 文件中提取出来，生成单独的 CSS 文件。这在生产环境中非常有用，因为它可以减少 JavaScript 文件的大小，并使 CSS 能够被浏览器并行加载，从而提高网页的加载速度和性能
```txt
配置选项
filename：指定输出的 CSS 文件名。可以使用占位符，如 [name]、[contenthash] 等。
chunkFilename：指定输出的非入口块的 CSS 文件名。可以使用占位符，如 [id]、[contenthash] 等。
ignoreOrder：忽略 CSS 顺序警告。如果你的项目中存在 CSS 顺序问题，可以设置为 true。
```
```js
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css',
      chunkFilename: '[id].css',
    })
  ]
};
```
在这个示例中，MiniCssExtractPlugin 会将 CSS 提取到单独的文件中，文件名为 [name].css 和 [id].css。

```js
// 使用占位符生成带有哈希值的文件名
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
      chunkFilename: '[id].[contenthash].css',
    })
  ]
};

```
```js
// 忽略 CSS 顺序警告 
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css',
      chunkFilename: '[id].css',
      ignoreOrder: true, // 忽略 CSS 顺序警告
    })
  ]
};
```
注意事项
- 与 style-loader 的区别 ：MiniCssExtractPlugin 通常用于生产环境，而 style-loader 通常用于开发环境。style-loader 将 CSS 注入到 JavaScript 中，而 MiniCssExtractPlugin 将 CSS 提取到单独的文件中。
- 与其他插件的兼容性 ：确保 MiniCssExtractPlugin 与其他 CSS 处理插件（如 autoprefixer、cssnano）兼容，以实现最佳效果。
- 性能 ：提取 CSS 可以显著减少 JavaScript 文件的大小，并提高 CSS 的加载性能，但也会增加构建时间。因此，在使用时需要权衡性能和构建时间之间的关系。


### CssMinimizerWebpackPlugin
CssMinimizerWebpackPlugin 使用 cssnano 来优化和压缩 CSS 文件。它支持各种优化选项，如去除无用的 CSS、压缩选择器等。该插件通常与 MiniCssExtractPlugin 一起使用，以确保在提取和优化 CSS 时没有冲突

```txt
配置选项
test：匹配需要优化的文件。默认值是 /\.css$/i。
parallel：启用多线程并行处理。可以是布尔值或数字，表示并行处理的数量。
minimizerOptions：传递给 cssnano 的选项，用于配置 CSS 优化规则。
include：匹配需要包含的文件。
exclude：匹配需要排除的文件。
```

```js
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerWebpackPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css'
    })
  ],
  optimization: {
    minimize: true,
    minimizer: [
      new CssMinimizerWebpackPlugin()
    ],
  },
};

```
在这个示例中，MiniCssExtractPlugin 用于提取 CSS 文件，而 CssMinimizerWebpackPlugin 用于优化和压缩提取后的 CSS 文件。

```js
// 自定义 cssnano 配置 
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerWebpackPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css'
    })
  ],
  optimization: {
    minimize: true,
    minimizer: [
      new CssMinimizerWebpackPlugin({
        minimizerOptions: {
          preset: [
            'default',
            {
              discardComments: { removeAll: true }, // 移除所有注释
            },
          ],
        },
      }),
    ],
  },
};

```

```js
// 启用多线程并行处理 
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerWebpackPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css'
    })
  ],
  optimization: {
    minimize: true,
    minimizer: [
      new CssMinimizerWebpackPlugin({
        parallel: true, // 启用多线程并行处理
      }),
    ],
  },
};

```

### [TerserWebpackPlugin](https://webpack.docschina.org/plugins/terser-webpack-plugin/)
用于优化和压缩 JavaScript 文件。它基于 Terser，一个流行的 JavaScript 压缩工具。通过使用这个插件，你可以显著减少 JavaScript 文件的大小，从而提高网页的加载速度和性能

webpack v5 开箱即带有最新版本的 terser-webpack-plugin。如果你使用的是 webpack v5 或更高版本，同时希望自定义配置，那么仍需要安装 terser-webpack-plugin
```txt
test：匹配需要优化的文件。默认值是 /\.m?js(\?.*)?$/i。
include：匹配需要包含的文件。
exclude：匹配需要排除的文件。
parallel：启用多线程并行处理。可以是布尔值或数字，表示并行处理的数量。
terserOptions：传递给 Terser 的选项，用于配置 JavaScript 优化规则。
extractComments：是否提取注释到单独的文件。可以是布尔值、字符串或函数。
```
```js
// 基本使用
const path = require('path');
const TerserWebpackPlugin = require('terser-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  optimization: {
    minimize: true,
    minimizer: [new TerserWebpackPlugin()]
  }
};

```
```js
// 自定义 Terser 配置 ：
const path = require('path');
const TerserWebpackPlugin = require('terser-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserWebpackPlugin({
        terserOptions: {
          compress: {
            drop_console: true, // 删除所有 console 语句
          },
        },
      }),
    ],
  },
};

```
```js
// 启用多线程并行处理 
const path = require('path');
const TerserWebpackPlugin = require('terser-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserWebpackPlugin({
        parallel: true, // 启用多线程并行处理
      }),
    ],
  },
};

```
```js
// 提取注释到单独的文件
const path = require('path');
const TerserWebpackPlugin = require('terser-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserWebpackPlugin({
        extractComments: true, // 提取注释到单独的文件
      }),
    ],
  },
};

```
注意事项
- 与其他优化插件的兼容性 ：TerserWebpackPlugin 通常与其他优化插件（如 CssMinimizerWebpackPlugin）一起使用，以实现最佳效果。
- 性能 ：启用多线程并行处理可以显著提高优化速度，但也会增加系统资源的使用。
- 配置 ：Terser 提供了丰富的配置选项，可以根据项目需求进行细粒度的优化配置。


### ProgressPlugin
用于显示构建过程的进度信息。它可以帮助你了解 Webpack 构建过程中的各个阶段，并实时显示进度，从而更好地监控和优化构建性能。
```js
new webpack.ProgressPlugin(handler)
// handler：一个可选的回调函数，用于自定义进度信息的处理方式。默认情况下，进度信息会输出到控制台
```
