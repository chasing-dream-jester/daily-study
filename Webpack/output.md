## output
output 配置项用于指定编译后的文件的位置和命名方式。

### **主要参数**
#### path 
path 输出目录的绝对路径。
```js
path:path.resolve(__dirname,'dist')
```
#### filename
filename 指定输出文件的名称，可以使用一些占位符来动态生成文件名，比如[name].js、[hash].js、[chunkhash].js
  
```js
filename: 'js/[name].js',
```
#### publicPath
publicPath 指定静态资源的公共路径
```js
publicPath: '/' //或者cdn地址
```
#### chunkFilename
chunkFilename  指定非入口文件（例如按需加载的代码块）的名称。通常与动态导入和代码拆分功能一起使用
```js
chunkFilename: 'js/[name].[chunkhash:8].bundle.js'
```
#### assetModuleFilename
assetModuleFilename 指定资源模块（例如图片、字体）的文件名模板,与 output.filename 相同，不过应用于 Asset Modules(asset/resource,asset/inline,asset/source,asset)
```js
assetModuleFilename: 'images/[hash][ext][query]' 
```
