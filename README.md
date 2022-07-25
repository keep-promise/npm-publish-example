# React 组件开发流程

## 创建项目

`$ npm init`
生成 **package.json**，包含项目的基本信息

---

## 安装 React

`$ npm add react react-dom`

---

## 开发组件

```├─src // 用来存放组件源码
| ├─index.ts // 入口文件，用来暴露组件
| ├─utils.ts
| ├─types // TS 声明文件
| | ├─externals.d.ts // 由于项目中使用了 less，需额外声明才能使用模块化导入 less 文件
| | └index.ts
| ├─Component
| | ├─index.less
| | └index.tsx
```

---

## 安装配置 TS

组件使用了 TS，需要对 TS 文件进行编译。
TS 只会在开发环境使用，安装**devDependencies**依赖
`$ npm add -D typescript`

在项目的根 / 目录新建ts配置文件 **tsconfig.json** ，
**目的：不用每次编译时输入复杂的命令**

```json
// tsconfig.json
{
  "compilerOptions": {
    "outDir": "./dist", // 输出的目录
    "module": "CommonJS", // 指定生成哪个模块系统代码: "None"， "CommonJS"， "AMD"， "System"， "UMD"， "ES6"或 "ES2015"
    "target": "ES2015", // 指定 ES 目标版本，默认 ES3
    "jsx": "react", // 在 .tsx 文件里支持 jsx
    "declaration": true, // 生成相应的 .d.ts 文件
    "removeComments": true, // 删除所有注释，除了以 /!_ 开头的版权信息。
  },
  "include": [
    "src/\*\*/_", // 需要编译的文件
  ],
  "exclude": [
    "node_modules",
  ],
  "files": [
    "file"
  ]
}
```

**使用 include 引入文件**
**使用 exclude 过滤文件**
**使用 files 包含文件，不会被 exclude 过滤**
**如果没有特殊指定， exclude 默认情况下会排除 node_modules，bower_components，jspm_packages 和 outDir 目录。**

tsconfig.json 更多的配置说明，可查看：[tsconfig.json配置说明](https://www.tslang.cn/docs/handbook/tsconfig-json.html)

---

## 开发示例

在当前项目中创建一个 example，开发时无需先将组件编译打包然后引用编译后的代码，可以直接引用该渲染器的源码，这样每次修改后保存可以自动热更新到页面，提高开发效率。当开发调试完成后，便可以使用生产模式，这样打包编译压缩后的渲染器代码便可直接使用。

使用方式：在 page/index.tsx 中直接 组件

```├─example // 示例代码
| ├─index.html // 用于挂载组件到页面
| ├─index.tsx
| | └mock.ts // mock 数据
```

---

## 打包编译

使用 **webpack** 完成对项目的打包编译，使用example调试组件
**webpack-dev-server**工具 用来开启本地服务器

```$ npm add --save-dev webpack webpack-cli webpack-dev-server```

```$ npm add --save-dev ts-loader less-loader style-loader css-loader```

在项目根目录 / 中创建webpack打包编译配置文件**webpack.config.js**

```js
// 开发环境配置
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const base = {
  mode: process.env.NODE_ENV === 'production' ? 'production' :      'development',
  devtool: 'cheap-module-source-map',
  resolve: {
    // Add '.ts' and '.tsx' as resolvable extensions.
    extensions: ['.ts', '.tsx', '.js', '.json'],
  },
  module: {
    rules: [
      // ts-loader 用于加载解析 ts 文件
      {
        test: /\.(ts|tsx)?$/,
        loader: 'ts-loader',
        exclude: /node_modules/
      },
      // 用于加载解析 less 文件
      {
        test: /\.less$/,
        use: [
          { loader: 'style-loader', },
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[hash:base64:6]',
              },
            }
          },
          { loader: 'less-loader', },
        ]
      },
    ]
  },
  optimization: {
    minimize: true, // 开启代码压缩
  },
};

if (process.env.NODE_ENV === 'development') {
  mergeConfig = {
    ...base,
    entry: path.join(**dirname, '/src/example/index.tsx'),
    output: {
      path: path.join(**dirname, 'example/dist'),
      filename: 'bundle.js',
      library: 'laputarenderer',
      libraryTarget: 'umd',
    },
    plugins: [
    // 自动注入编译打包好的代码至 html
      new HtmlWebpackPlugin({
        template: path.join(__dirname, './src/example/index.html'),
        filename: 'index.html',
      }),
    ],
    devServer: {
      // port: 8008, // example 的启动端口，选填
    },
  };
}

module.exports = mergeConfig;
```

添加开发模式的 script 命令，需要指定 node 执行环境

`$ npm add --save-dev cross-env`

添加 npm 执行命令，用来编译运行项目的 example

```json
// package.json
"scripts": {
  "start": "cross-env NODE_ENV=development webpack-dev-server --open"
},
```

执行 **npm run start** 编译调试项目

---

## 生产环境

安装**clean-webpack-plugin**

打包编译之前清空打包目录 /dist

$ npm add --save-dev clean-webpack-plugin

```js
// 开发环境配置
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

const base = {
  mode: process.env.NODE_ENV === 'production' ? 'production' :      'development',
  devtool: 'cheap-module-source-map',
  resolve: {
    // Add '.ts' and '.tsx' as resolvable extensions.
    extensions: ['.ts', '.tsx', '.js', '.json'],
  },
  module: {
    rules: [
      // ts-loader 用于加载解析 ts 文件
      {
        test: /\.(ts|tsx)?$/,
        loader: 'ts-loader',
        exclude: /node_modules/
      },
      // 用于加载解析 less 文件
      {
        test: /\.less$/,
        use: [
          { loader: 'style-loader', },
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[hash:base64:6]',
              },
            }
          },
          { loader: 'less-loader', },
        ]
      },
    ]
  },
  optimization: {
    minimize: true, // 开启代码压缩
  },
};

if (process.env.NODE_ENV === 'development') {
  mergeConfig = {
    ...base,
    entry: path.join(**dirname, '/src/example/index.tsx'),
    output: {
      path: path.join(**dirname, 'example/dist'),
      filename: 'bundle.js',
      library: 'laputarenderer',
      libraryTarget: 'umd',
    },
    plugins: [
    // 自动注入编译打包好的代码至 html
      new HtmlWebpackPlugin({
        template: path.join(__dirname, './src/example/index.html'),
        filename: 'index.html',
      }),
    ],
    devServer: {
      // port: 8008, // example 的启动端口，选填
    },
  };
} else {
  mergeConfig = {
    ...base,
    entry: './src/index.ts',
    output: {
      filename: 'index.js',
      path: path.resolve(\_\_dirname, 'dist'),
      library: 'laputarenderer',
      library: 'umd'
    },
    devtool: 'none',
    // When importing a module whose path matches one of the following, just
    // assume a corresponding global variable exists and use that instead.
    // This is important because it allows us to avoid bundling all of our
    // dependencies, which allows browsers to cache those libraries between builds.
    // 我们想要避免把所有的 React 都放到一个文件里，因为会增加编译时间并且浏览器还能够缓存没有发生改变的库文件。
    // 理想情况下，我们只需要在浏览器里引入 React 模块，但是大部分浏览器还没有支持模块。
    // 因此大部分代码库会把自己包裹在一个单独的全局变量内，比如：jQuery 或*。 这叫做“命名空间”模式，
    // webpack 也允许我们继续使用通过这种方式写的代码库。
    // 通过我们的设置"react": "React"，webpack 会神奇地将所有对"react"的导入转换成从 React 全局变量中加载
    externals: {
      'react': 'react',
      'react-dom': 'react-dom'
    },
    plugins: [
      new CleanWebpackPlugin(),	// 编译之前清空打包目录/dist
    ]
  };
}

module.exports = mergeConfig;
```

添加 npm 打包命令，用来打包项目项目

```json
// package.json
{
  // ...
  "scripts": {
    "build": "cross-env NODE_ENV=production npm webpack"
  },

  "peerDependencies": {
    "react": ">=16.9.0",
    "react-dom": ">=16.9.0"
  }
}
```

<font color=red>⚠️ 注意：package.json 添加了 peerDependencies 字段，使用组件，就必须同级安装指定的这些依赖</font>

在本项目中指定了 react 和 react-dom，如果另外一个项目 A 想要安装此组件，就必须同时安装 react 和 react-dom。

npm1 和 npm2 版本可以安装组件的同时自动安装 peerDependencies 中的依赖，但是之后的版本需要手动安装，否则将会收到警告

更多内容请阅读：[Peer Dependencies](https://nodejs.org/zh-cn/blog/npm/peer-dependencies/)。

执行**npm run build**，生成dist目录

## 组件发布

指定发布目录

```json
// package.json
{
  "main": 'dist/index.js', // 该包的入口文件
  "types": "dist/index.d.ts", // 指明声明文件的入口
  // ...
  "files": ["dist"], // npm 发布白名单
// ...
}
```

files 字段规定了 dist 文件夹会出现在发布的包里（README.md 和 package.json 会被默认添加）

**npm（npmjs.com） 账号**
如果没有，注册：www.npmjs.com/signup。
如果已经有了账号，需要在本地终端执行 npm login 登陆输入用户名密码邮箱后即可登录成功，npmjs.com 的账户里会自动保存当前 pc 的唯一 token

**自动化发布**
在 package.json 中添加一些 script 命令即 npm publish hook，用于在发布之前进行比如 preversion 、version 和 postversion 



$ npm publish

发布成功之后可以在 npmjs.com 的个人主页查看新发布的npm包 🎉🎉🎉

版本更新的命令

**升级补丁版本号 1.0.0 -> 1.0.1**
`$ npm version patch`

**升级次版本号 1.0.0 -> 1.1.0**
`$ npm version minor`

**升级主版本号 1.0.0 -> 2.0.0**
`$ npm version major`
