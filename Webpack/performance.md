## performance
performance 配置项用于控制性能提示（performance hints），这些提示会在构建过程中或生成的文件超出指定大小时发出警告或错误。

### performance 属性详解
#### hints
hints: 性能提示类型
- 'warning': 构建过程中会发出警告（默认值）
- error': 构建过程中会发出错误并停止构建
- false: 关闭性能提示

#### maxAssetSize
maxAssetSize: 单个资源（如文件）的最大体积，单位为字节。超过这个大小会触发性能提示。

#### maxEntrypointSize
maxEntrypointSize: 入口点最大的体积，单位为字节。入口点包括所有同步导入的资源。超过这个大小会触发性能提示。

#### assetFilter
assetFilter: 自定义过滤函数，只对满足条件的文件生成性能提示
```js
performance: {
  assetFilter: function(assetFilename) {
    // 只对 .js 文件生成性能提示
    return assetFilename.endsWith('.js');
  },
}
```
