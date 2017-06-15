### Webpack 配置 React 开发环境

首先总结一下，分为以下几个步骤：

1. `npm`下载 `webpack`
2. `nam` 下载 loader（`babel-loader`等）
3. 配置 `webpack.config.js`
4. 执行 `webpack` 生成 `bundle.js`
5. `webpack-watch -d` 生成 `bundle.map.js`
6. `index.html` 调用 `bundle.js`
7. 配置 `webpack-dev-service`

###### npm 下载 webpack

`$ npm install webpack --save-dev`

将 webpack 包以开发模式安装，如果是`--save`，则会写在`package.json`的dependencies 当中，加上`-dev`，则放在了 devDependencies 当中，实际文件是在 node_modules 里的。

###### npm 下载 loader

`$ npm install babel-loader babel-core babel-preset-es2015 babel-preset-react --save-dev`

我们需要下载开发环境所要的 loader，如果是用 ES6 去写代码，则需要 Babel （编译工具），如果还需要编译 SASS，可以下载 sass-loader 等。

###### 配置 webpack.config.js 文件

在根目录下新建一个文件，命名为 `webpack.config.js`

配置内容如下：

``` javascript
// webpack.config.js
var webpack = require('webpack');  // CommonJs 的写法

module.exports = {
    entry:[
        './entry.js'    //入口文件，里面可以添加多个依赖模块
    ],
    output: {
        path: __dirname + '/dist/',   //打包输出的路径
        publicPath: "/dist/",   //html引用路径
        filename: 'bundle.js'   //打包后的名字
    },
    resolve:{
        extensions: ['','.js']   //接受的扩展名
    },
    module: {
        loaders: [    //载入loader
            {
                test: /\.js$/,     //正则匹配需要编译的文件后缀
                exclude: /node_modules/, 
                loader: 'babel-loader',   //loader加载器
                query: {
                    //presets设置转码规则,plugins设置插件
                    presets: ['es2015', 'react']
                }
            }
        ]

    }
};

```

###### 执行 webpack 生成 bundle.js

配置完 `webpack.config.js` 以后，我们就可以输入命令去生成 `bundle.js` 文件了。

`$ webpack`

上面是启动 webpack 最基本的方式，当然我们也可以执行下面这条语句

`$ webpack -watch-d`

表示我们可以监控打包内容的更新，并且提供 source map 方便我们的调试。

执行完以后我们就能看到在 `dist` 目录下，有两个文件，一个是 `bundle.js`，一个是 `bundle.map.js`。这代表我们把所有我们需要的依赖文件全部打包成了一个 `bundle.js` 文件，这时可以在 `index.html` 文件当中引入 `bundle.js`。

然后我们就可以愉快地写代码了~ 

###### 配置 webpack-dev-service

如果我们希望体验更好一点，还可以再配置一下 webpack-dev-server，作用是能监控我们代码的更新，可以根据代码的修改同步更新 `bundle.js` 的内容，并且体现在浏览器端的作用是不需要我们刷新页面，它会自动检测修改自动刷新。

首先我们需要下载

`$ npm install webpack-dev-server --save-dev`

然后执行命令

`$ node ./node_modules/webpack-dev-server/bin/webpack-dev-server.js`

然后会出现下面这个路径，打开路径就能看到自己的页面效果了，前提是页面无报错。

`http://localhost:8080/webpack-dev-server/`