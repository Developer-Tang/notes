## 从零搭建React项目

### 通过create-react-app创建

```shell
npx create-react-app react-antd-app --template typescript
```

!> `--template` 指定了 js 类型为 typescript 如果不想用 ts 也可以不加

### 修改基础文件

> 打开项目路径，删除用不到的文件，保留基础文件即可

```text
├─ /node_modules
├─ /public
|  ├─ favicon.ico
|  └─ index.html
├─ /src
|  ├─ App.tsx
|  └─ index.tsx
├─ .gitignore
├─ package.json
├─ README.md
```

> 修改 `index.html` 文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>React App</title>
</head>
<body>
<noscript>You need to enable JavaScript to run this app.</noscript>
<div id="root"></div>
</body>
</html>
```

> 修改 `App.tsx` 文件

```typescript
import React from 'react'

const App: React.FC = () => {
  return (
    <p>
      Edit < code > src / App.tsx < /code> and save to reload.
    < /p>
  )
}

export default App
```

> 修改 `index.tsx` 文件

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);
root.render(
  <React.StrictMode>
    <App / >
  </React.StrictMode>
);
```

### Webpack配置

> `.gitignore` 文件添加忽略项

```gitignore
yarn.lock
package-lock.json
```

> 项目文件提交git

```shell
git add .
git commit -m '提交说明'
```

> 执行 eject

```shell
yarn eject
```

### 集成Less

> 执行

```shell
yarn add less less-loader --dev
```

> 修改 `config/webpack.config.js`

```typescript
// 开头添加
const lessRegex = /\.less$/
const lessModuleRegex = /\.module\.less$/
module.exports = function (webpackEnv) {
  return {
    resolve: {
      module: {
        rules: [ {
          oneOf: [
            // 前面都是其他配置，从这里开始为要新增的配置
            {
              test: lessRegex,
              exclude: lessModuleRegex,
              use: getStyleLoaders(
                {
                  importLoaders: 3,
                  sourceMap: isEnvProduction ? shouldUseSourceMap : isEnvDevelopment,
                  modules: {
                    mode: 'icss'
                  }
                },
                'less-loader'
              ),
              sideEffects: true
            }, {
              test: lessModuleRegex,
              use: getStyleLoaders(
                {
                  importLoaders: 3,
                  sourceMap: isEnvProduction ? shouldUseSourceMap : isEnvDevelopment,
                  modules: {
                    mode: 'local',
                    getLocalIdent: getCSSModuleLocalIdent
                  }
                },
                'less-loader'
              )
            }
            // 新增配置到这里结束，将中间内容复制粘贴到 oneOf 数组中就行了
          ]
        } ]
      }
    }
  }
}
```

#### 设置路径别名

> 修改 `config/webpack.config.js`

```typescript
module.exports = function (webpackEnv) {
  return (
    resolve: {
      alias: {
        '@': path.join(__dirname '..', 'src'),
      }
    }
  )
}
```

