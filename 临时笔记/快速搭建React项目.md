## 项目初始化

> react-antd-app 为自定义项目名，按需修改

```shell
mkdir react-antd-app
cd react-antd-app
yarn init -y
```

> 根目录添加文件 `.gitignore`

```ignore
.idea/
.vscode/
node_modules/
dist/
build/

yarn.lock
package_lock.json
*.log
```

> 初始化 Git 仓库

```shell
git init
git add .
git commit -m "project init"
```

## 安装相关依赖

```shell
yarn add react react-dom react-scripts
yarn add -D @types/react @types/react-dom
yarn add -D css-loader mini-css-extract-plugin less less-loader
yarn add -D babel-loader
yarn add -D @babel/core @babel/preset-env @babel/preset-react @babel/preset-typescript @babel/plugin-proposal-class-properties @babel/plugin-proposal-object-rest-spread
yarn add -D webpack webpack-cli webpack-dev-server html-webpack-plugin webpackbar
# .env环境变量支持
yarn add -D dotenv-cli dotenv-webpack
```

## webpack配置

> 根目录创建文件 `webpack.config.js`

```js
const { resolve } = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const WebpackBar = require('webpackbar')
const Dotenv = require('dotenv-webpack')
const path = require('path')
module.exports = {
    mode: 'development',
    entry: {
        main: resolve(__dirname, 'src', 'index.tsx')
    },
    output: {
        filename: 'index.js',
        path: resolve(__dirname, 'dist')
    },
    resolve: {
        // webpack 默认只处理js、jsx等js代码
        // 为了防止在import其他ts代码的时候，出现
        // " Can't resolve 'xxx' "的错误，需要特别配置
        extensions: [ '.js', '.jsx', '.ts', '.tsx' ],
        alias: {
            '@': path.join(__dirname, './src/')
        }
    },
    module: {
        rules: [
            {
                test: /\.tsx?/,
                use: [
                    'babel-loader'
                ],
                exclude: /node_moudles/
            },
            {
                test: /\.css$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader'
                ]
            },
            {
                test: /\.less$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    'less-loader'
                ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: resolve(__dirname, 'public', 'index.html'),
            filename: 'index.html',
            inject: 'body'
        }),
        new MiniCssExtractPlugin({
            filename: 'app.css'
        }),
        new Dotenv({
            path: path.resolve(__dirname, `.env.${process.env.REACT_APP_ENV}`)
        }),
        new WebpackBar({
            // color: "#85d", // 默认green，进度条颜色支持HEX
            basic: false, // 默认true，启用一个简单的日志报告器
            profile: false // 默认false，启用探查器。
        })
    ],
    devServer: {
        port: process.env.REACT_APP_PORT ?? 80
    }
}
```

## babel配置

> 根目录创建文件 `.babelrc`

```json
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-typescript",
    [
      "@babel/preset-react",
      {
        "runtime": "automatic"
      }
    ]
  ],
  "plugins": [
    "@babel/plugin-proposal-object-rest-spread",
    "@babel/plugin-proposal-class-properties"
  ]
}
```

## ts配置

> 根目录创建文件 `tsconfig.json`

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "rootDir": "./src",
    "outDir": "./dist",
    "jsx": "react-jsx",
    "allowSyntheticDefaultImports": true
  },
  "paths": {
    "@/*": [
      "./src/*"
    ]
  }
}
```

## 页面配置

> 创建文件 `public/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ReactApp</title>
</head>
<body>
<div id="root"></div>
</body>
</html>
```

> 创建文件 `src/global.d.ts`，用于类型定义

```ts
declare module '*.module.less' {
    const content: {
        [className: string]: any
    }
    export default content
}
```

> 创建文件 `src/index.tsx`

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import styles from '@/index.module.less'

const root = ReactDOM.createRoot(
    document.getElementById('root') as HTMLElement
)
root.render(
    <div className={styles.app}>Hello, <span>App</span></div>
)
```

> 创建文件 `src/index.module.less`

```less
html, body {
  height: 100%;
  width: 100%;
  margin: 0;
}

div {
  box-sizing: border-box;
}

.app {
  height: 100%;
  width: 100%;
  font-size: 20px;

  span {
    color: rgb(0, 111, 222);
    font-size: 24px;
  }
}
```

## 启动脚本配置

> 编辑文件 `package.json`，新增 scripts 配置

```json
{
  "scripts": {
    "dev": "dotenv -e .env.dev webpack-dev-server --config webpack.config.js",
    "build": "dotenv -e .env.prod webpack"
  }
}
```

> .env.* 配置根据自身需求而定，变量名也是不固定的，按需调整

```text
# xxx环境配置
REACT_APP_ENV='dev'

# 配置端口
# REACT_APP_PORT=8001

# 配置请求接口
# REACT_APP_API=http://127.0.0.1:8080/
```

## 引入Antd

> 执行如下命令

```shell
yarn add antd @ant-design/icons
# 如果不用 Antd Pro 则下列不需要执行
yarn add @ant-design/pro-components
```
