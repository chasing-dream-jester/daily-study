## optimization
optimization 配置项用于优化打包过程和生成的输出文件。

### **常用的配置项**
#### minimize
  * minimize: 启用或禁用代码压缩（默认为 true）。
#### minimizer 
  * 指定用于代码压缩的插件,比如CssMinimizerPlugin(Css压缩)、TerserPlugin(JavaScript 压缩)
#### splitChunks
  * 配置代码分割（Code Splitting），将代码拆分成多个小块，以便更好地进行缓存和按需加载
  ```js
  splitChunks: {
    chunks: 'all', // 分割所有类型的代码块
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
  },
  ```
#### runtimeChunk
  * 将 Webpack 运行时代码提取到单独的文件中，以减少入口文件的体积，提高缓存效率
  ```js
  runtimeChunk: 'single',
  ```
  我们使用 runtimeChunk: 'single' 来将所有入口点共享的运行时代码提取到一个单独的文件中。


### splitChunks 选项详解
#### chunks
chunks: 指定要分割的代码块类型。
- all: 分割所有类型的代码块（推荐）。
- async: 只分割按需加载的代码块（默认值）。
- initial: 只分割初始加载的代码块。

#### minSize和maxSize
 minSize: 生成代码块的最小大小（以字节为单位）。只有大于此大小的代码块才会被分割。
 
 maxSize: 生成代码块的最大大小。超过此大小的代码块会被进一步分割
  
#### minChunks
minChunks: 一个模块至少被多少代码块共享时才会被分割。

#### maxAsyncRequests 和 maxInitialRequests
maxAsyncRequests: 按需加载时最大的并行请求数。

maxInitialRequests: 入口点处最大的并行请求数。
```js
splitChunks: {
  maxAsyncRequests: 6, // 默认值
  maxInitialRequests: 4, // 默认值
}
```
#### automaticNameDelimiter
automaticNameDelimiter: 生成名称时的分隔符
```js
splitChunks: {
  automaticNameDelimiter: '~', // 默认值
}
```
#### **cacheGroups**
cacheGroups: 自定义代码块分组规则。可以用来创建特定的代码块（如 vendor 块）。

- test 用于匹配要分组的模块。可以是正则表达式、函数或字符串
- name: 指定生成的代码块的名称
- chunks: 指定要分割的代码块类型
- minChunks: 一个模块至少被多少代码块共享时才会被分割
- priority 一个模块可以属于多个缓存组。优化将优先考虑具有更高 priority（优先级）的缓存组。默认组的优先级为负，以允许自定义组获得更高的优先级（自定义组的默认值为 0）
- reuseExistingChunk: 如果当前代码块包含已经从主代码块中分离出来的模块，则重用这些模块而不是创建一个新的代码块
```js
splitChunks: {
  cacheGroups: {
    vendors: {
      test: /[\\/]node_modules[\\/]/,
      name: 'vendors',
      chunks: 'all',
    },
    default: {
      minChunks: 2,
      priority: -20,
      reuseExistingChunk: true,
    },
  },
}
```
