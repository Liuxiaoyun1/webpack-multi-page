# webpack-mult-page
webpack 多页面打包方案


### 运行
`npm run dev`
http://localhost:8080

### 打包
`npm run build`


### 配置
```javascript
var path = require('path');
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;

/*
extract-text-webpack-plugin插件，
有了它就可以将你的样式提取到单独的css文件里，
再也不会被打包到js文件里了。
 */
var ExtractTextPlugin = require('extract-text-webpack-plugin');

var CleanWebpackPlugin = require('clean-webpack-plugin');

var webpackConfig = {
  entry: {  // 入口文件（css文件也在js中引入）
    index: ['./src/js/index.js'],
    list: ['./src/js/list.js']
  },
  output: {
    path: path.resolve(__dirname, 'dist'),  // 输出根目录
	publicPath: '../',
    filename: 'js/[hash:8].[name].js',   // 输出文件目录 及文件名格式
  },
  module: {
    loaders: [  // 处理css  可以加less或sass
		{
		  test: require.resolve('jquery'),  // 此loader配置项的目标是NPM中的jquery
		  loader: 'expose?$!expose?jQuery', // 先把jQuery对象声明成为全局变量`jQuery`，再通过管道进一步又声明成为全局变量`$`
		},
		{
			test: /\.css$/,
			loader: ExtractTextPlugin.extract('style-loader', 'css-loader') 
		}, 
		{
			//html模板加载器，可以处理引用的静态资源，默认配置参数attrs=img:src，处理图片的src引用的资源
			//比如你配置，attrs=img:src img:data-src就可以一并处理data-src引用的资源了，就像下面这样
			test: /\.html$/,
			loader: "html?attrs=img:src img:data-src"
		}, 
		{ 	
			test: /bootstrap\/js\//, 
			loader: 'imports?jQuery=jquery' },
		{test:/\.(eot|ttf|woff|woff2|svg)$/,loader:'file?name=fonts/[name].[ext]'},
		{
			//图片加载器，雷同file-loader，更适合图片，可以将较小的图片转成base64，减少http请求
			//如下配置，将小于8192byte的图片转成base64码
			test: /\.(png|jpg|gif)$/,
			loader: 'url-loader?limit=8192&name=./img/[name].[ext]'
		}
    ]
  },
  plugins: [
	new webpack.ProvidePlugin({ //加载jq
			$: 'jquery'
		}),
    new CommonsChunkPlugin({  // 提取公共代码
        name: 'base',
        chunks: ['index', 'list'],
        minChunks: 2 // 提取所有chunks共同依赖的模块 一个模块同时
    }),
    new HtmlWebpackPlugin({
      inject: 'head',  // 注入body
      chunks: ['base', 'index'], // 依赖模块
      filename: 'html/index.html', // 输出相对路径和文件名
      template: 'src/view/index.html', // 模版路径
    }),
    new HtmlWebpackPlugin({
      inject: 'head',
      chunks: ['base', 'list'],
      filename: 'html/list.html',
      template: 'src/view/list.html',
    }),
	new CleanWebpackPlugin('dist'),
	new ExtractTextPlugin('css/[name].css') //单独使用link标签加载css并设置路径，相对于output配置中的publickPath
  ]
}

module.exports = webpackConfig;
```