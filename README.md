#Webpack开发模式和生产模式搭建
npm init -y 初始化 package.json，生成后的文件如下：
```
{
  "name": "sdk",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.36.1",
    "webpack-cli": "^3.3.6",
    "webpack-dev-server": "^3.7.2"
  }
}
```


首先安装 webpack 所需依赖
npm i webpack webpack-cli webpack-dev-server --save-dev


安装 babel7，因为目前主要是用 ES6 来编写代码，所以需要转译
npm i @babel/core babel-loader @babel/preset-env @babel/plugin-transform-runtime --save-dev
npm i @babel/polyfill @babel/runtime


现在 package.json 中的依赖为：
{
  "name": "sdk",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.5.5",
    "@babel/plugin-transform-runtime": "^7.5.5",
    "@babel/preset-env": "^7.5.5",
    "babel-loader": "^8.0.6",
    "webpack": "^4.36.1",
    "webpack-cli": "^3.3.6",
    "webpack-dev-server": "^3.7.2"
  },
  "dependencies": {
    "@babel/polyfill": "^7.4.4",
    "@babel/runtime": "^7.5.5"
  }
}


新建 .babelrc 来配置 babel 插件，代码如下：
{
  "presets": ["@babel/preset-env"],
  "plugins": ["@babel/plugin-transform-runtime"]
}


新建 .browserslistrc 文件配置该项目所支持的浏览器版本
# 所支持的浏览器版本

> 1% # 全球使用情况统计选择的浏览器版本
last 2 version # 每个浏览器的最后两个版本
not ie <= 8 # 排除小于 ie8 及以下的浏览器


在开始配置 webpack.config.js 文件之前，需要注意一下，因为现在我们是有两种模式，production(生产)  和 development(开发)  模式。

安装自动生成 html 依赖
npm i html-webpack-plugin html-loader clean-webpack-plugin --save-dev


安装 css/字体图标处理依赖
npm i css-loader style-loader mini-css-extract-plugin optimize-css-assets-webpack-plugin --save-dev


安装 scss 处理依赖
npm i node-sass sass-loader --save-dev


为不同内核的浏览器加上 CSS 前缀
npm install postcss-loader autoprefixer --save-dev


图片及字体处理
npm i url-loader file-loader image-webpack-loader --save-dev


第三方 js 库(自由选择)
npm i jquery


现在 package.json 为：
{
  "name": "sdk",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.5.5",
    "@babel/plugin-transform-runtime": "^7.5.5",
    "@babel/preset-env": "^7.5.5",
    "autoprefixer": "^9.6.1",
    "babel-loader": "^8.0.6",
    "clean-webpack-plugin": "^3.0.0",
    "css-loader": "^3.1.0",
    "file-loader": "^4.1.0",
    "html-loader": "^0.5.5",
    "html-webpack-plugin": "^3.2.0",
    "image-webpack-loader": "^5.0.0",
    "mini-css-extract-plugin": "^0.8.0",
    "node-sass": "^4.12.0",
    "optimize-css-assets-webpack-plugin": "^5.0.3",
    "postcss-loader": "^3.0.0",
    "sass-loader": "^7.1.0",
    "style-loader": "^0.23.1",
    "url-loader": "^2.1.0",
    "webpack": "^4.36.1",
    "webpack-cli": "^3.3.6",
    "webpack-dev-server": "^3.7.2"
  },
  "dependencies": {
    "@babel/polyfill": "^7.4.4",
    "@babel/runtime": "^7.5.5"
  }
}



之前我们大多都是写生产模式，也就是经常说的打包，但是我们日常开发项目，用的是开发模式。
只有在项目做完后，要部署到 nginx 上的时候才使用生产模式，将代码打包后放到 nginx 中
之所以要分两种模式是因为，开发模式下，需要加快编译的速度，可以热更新以及设置跨域地址，开启源码调试(devtool: 'source-map')
而生产模式下，则需要压缩 js/css 代码，拆分公共代码段，拆分第三方 js 库等操作
所以这里的配置我们分成三个文件来写，一个是生产配置，一个是开发配置，最后一个是基础配置
即：webpack.base.conf.js(基础配置)、webpack.dev.conf.js(开发配置)、webpack.prod.conf.js(生产配置)
新建 build 文件夹，创建上述三个文件，项目结构为：


这里需要使用到一个插件，webpack-merge 用来合并配置，比如开发环境就合并开发配置 + 基础配置，生产就合并生产配置 + 基础配置
npm i webpack-merge --save-dev


先简单写个 webpack.base.conf.js 的示例代码
const merge = require('webpack-merge')

const productionConfig = require('./webpack.prod.conf') // 引入生产环境配置文件
const developmentConfig = require('./webpack.dev.conf') // 引入开发环境配置文件

const baseConfig = {} // ... 省略

module.exports = env => {
  let config = env === 'production' ? productionConfig : developmentConfig
  return merge(baseConfig, config) // 合并 公共配置 和 环境配置
}


引入 webpack-merge 插件来合并配置
引入生产环境和开发环境
编写基础配置
导出合并后的配置文件

在代码中区分不同环境：
module.exports = env => {
  let config = env === 'production' ? productionConfig : developmentConfig
  return merge(baseConfig, config) // 合并 公共配置 和 环境配置
}


这里的 env 在 package.json 中进行配置，修改 scripts，添加 "dev" 和 "build" 命令
注意，这里有个 --env 字段，与 webpack.base.conf.js 中的 env 是联动的，告诉它当前是什么环境，然后合并成什么环境
{
  "scripts": {
    "dev": "webpack-dev-server --env development --open --config build/webpack.base.conf.js",
    "build": "webpack --env production --config build/webpack.base.conf.js"
  }
}


(一) 编写基础配置 webpack.base.conf.js
const path = require('path')
const webpack = require('webpack')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const {CleanWebpackPlugin} = require('clean-webpack-plugin')

module.exports = {
  entry: {
    app: './src/app.js'
  },
  output: {
    path: path.resolve(__dirname, '..', 'dist')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'babel-loader'
          }
        ]
      },
      {
        test: /\.(png|jpg|jpeg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              name: '[name]-[hash:5].min.[ext]',
              limit: 1000, // size <= 1KB
              outputPath: 'images/'
            }
          },
          // img-loader for zip img
          {
            loader: 'image-webpack-loader',
            options: {
              // 压缩 jpg/jpeg 图片
              mozjpeg: {
                progressive: true,
                quality: 65 // 压缩率
              },
              // 压缩 png 图片
              pngquant: {
                quality: '65-90',
                speed: 4
              }
            }
          }
        ]
      },
      {
        test: /\.(eot|ttf|svg)$/,
        use: {
          loader: 'url-loader',
          options: {
            name: '[name]-[hash:5].min.[ext]',
            limit: 5000, // fonts file size <= 5KB, use 'base64'; else, output svg file
            publicPath: 'fonts/',
            outputPath: 'fonts/'
          }
        }
      }
    ]
  },
  plugins: [
    // 开发环境和生产环境二者均需要的插件
    new HtmlWebpackPlugin({
      title: 'webpack4 实战',
      filename: 'index.html',
      template: path.resolve(__dirname, '..', 'index.html'),
      minify: {
        collapseWhitespace: true
      }
    }),
    new webpack.ProvidePlugin({ $: 'jquery' }),
    new CleanWebpackPlugin()
  ],
  performance: false
}


(二) 编写开发环境配置文件 webpack.dev.conf.js
 **开发配置主要是设置跨域、开启源码调试、热更新**
const webpack = require('webpack')
const merge = require('webpack-merge')
const commonConfig = require('./webpack.base.conf.js')

const path = require('path')

const devConfig = {
  mode: 'development',
  output: {
    filename: '[name].js',
    chunkFilename: '[name].js'
  },
  module: {
    rules: [
      {
        test: /\.(sa|sc|c)ss$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2 // 在一个 css 中引入了另一个 css，也会执行之前两个 loader，即 postcss-loader 和 sass-loader
            }
          },
          'postcss-loader', // 使用 postcss 为 css 加上浏览器前缀
          'sass-loader' // 使用 sass-loader 将 scss 转为 css
        ]
      }
    ]
  },
  devtool: 'cheap-module-eval-soure-map',
  devServer: {
    contentBase: path.join(__dirname, '../dist/'),
    port: 8000,
    hot: true,
    overlay: true,
    proxy: {
      '/comments': {
        target: 'https://m.weibo.cn',
        changeOrigin: true,
        logLevel: 'debug',
        headers: {
          Cookie: ''
        }
      }
    },
    historyApiFallback: true
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NamedModulesPlugin()
  ]
}

module.exports = merge(commonConfig, devConfig)


(三) 编写生产环境配置文件 webpack.prod.conf.js
**生产配置主要是拆分代码，压缩 css**
const merge = require('webpack-merge')
const commonConfig = require('./webpack.base.conf.js')

const MiniCssExtractPlugin = require('mini-css-extract-plugin') // 将 css 单独打包成文件
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin') // 压缩 css

const prodConfig = {
  mode: 'production',
  output: {
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].js'
  },
  devtool: 'cheap-module-source-map',
  module: {
    rules: [
      {
        test: /\.(sa|sc|c)ss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader
          },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2 // 在一个 css 中引入了另一个 css，也会执行之前两个 loader，即 postcss-loader 和 sass-loader
            }
          },
          'postcss-loader', // 使用 postcss 为 css 加上浏览器前缀
          'sass-loader' // 使用 sass-loader 将 scss 转为 css
        ]
      }
    ]
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        jquery: {
          name: 'jquery', // 单独将 jquery 拆包
          priority: 15,
          test: /[\\/]node_modules[\\/]jquery[\\/]/
        },
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors'
        }
      }
    }
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name]-[contenthash].css',
      chunkFilename: '[id]-[contenthash].css'
    }),
    // 压缩 css
    new OptimizeCssAssetsPlugin({
      assetNameRegExp: /\.css$/g, //一个正则表达式，指示应优化/最小化的资产的名称。提供的正则表达式针对配置中ExtractTextPlugin实例导出的文件的文件名运行，而不是源CSS文件的文件名。默认为/\.css$/g
      cssProcessor: require('cssnano'), //用于优化\最小化 CSS 的 CSS处理器，默认为 cssnano
      cssProcessorOptions: { safe: true, discardComments: { removeAll: true } }, //传递给 cssProcessor 的选项，默认为{}
      canPrint: true //一个布尔值，指示插件是否可以将消息打印到控制台，默认为 true
    })
  ]
}

module.exports = merge(commonConfig, prodConfig)
