# JS Module Explore

使用 TS 开发了npm 模块，支持 UMD、CJS、ESM 等规范，并且在 Nodejs 和浏览器中验证

> 需要用到 lerna: `npm install --global lerna`

安装依赖: `lerna bootstrap`

## Typescript 开发 npm 包实践

查看[my-js-module](./packages/my-js-module/)项目结构。这是一个很普通的包，仅仅有两个函数，是用来演示的。这里我采用 [microbundle](https://github.com/developit/microbundle) 作为项目的打包工具，要比 rollup 要简单一些。

开始构建 npm 模块:

```bash
> lerna run --scope my-js-module build 
```

可以看到产生的 `dist` 结构:

```bash
| - dist
    | - my-js-module.cjs        # CJS 规范
    | - my-js-module.esm.js     # ESM 规范
    | - my-js-module.modern.js  # 浏览器 type="modern" 使用
    | - my-js-module.umd.js     # 全局都可以使用
```

然后注意 `package.json` 这一部分:

```json
{
  # 最原始的写法 CJS 路径
  "main": "dist/my-js-module.cjs",
  # TS 的 types 路径
  "types": "types/index.d.ts",
  # 这个包默认都是 ESM 类型
  "type": "module",
  # 使用 import 导入时，首先用该路径文件
  "module": "./dist/my-js-module.esm.js",
  # 更加细粒度的控制子目录访问路径，会替换默认的 main 路径
  "exports": {
    ".": {
      "import": "./dist/my-js-module.esm.js",
      "require": "./dist/my-js-module.cjs"
    },
    "./dist/*": "./dist/*",
    "./types/*": "./types/*"
  },
  "scripts": {
    "build": "npm run build:node & npm run build:browser",
    # 构建 node 中使用的模块，这里只有 cjs 模块
    "build:node": "rm -rf dist && microbundle -f cjs -o dist --compress=false --target=node --sourcemap=false --external all",
    # 构建浏览器中使用的模块，这里有 umd，esm，modern 模块，而且暴露出 JSMODULE 全局变量
    "build:browser": "rm -rf dist && microbundle -f umd,esm,modern -o dist --compress=false --target=web --name=JSMODULE --sourcemap=false --external all"
  },
}
```

关键点都注释了，不明白的细看[这里](https://riskers.notion.site/package-json-4ae91d984b7a42789f6d2fcd98f1acaf)

## 验证

安装模块 - `lerna add my-js-module --scope my-js-module-test`

### Nodejs

```bash
> lerna run --scope my-js-module-test test:cjs:nodejs
4
94
```

```bash
> lerna run --scope my-js-module-test test:cjs:nodets
4
73
```

### Browser

#### UMD

```bash
> lerna run --scope my-js-module-test test:umd:browser
```

可以看到 UMD 还暴露出了 `JSMODULE` 这个变量：

![](https://i.imgur.com/SKkGWKI.png)

#### modern

```bash
> lerna run --scope my-js-module-test test:modern:browser
```

![](https://i.imgur.com/g3AITf0.png)

#### ESM

```bash
# build，这里使用的是 webpack
> lerna run --scope my-js-module-test build:esm  
```

```bash
> lerna run --scope my-js-module-test test:esm:browser
```

![](https://i.imgur.com/IAxvmn1.png)

至此，验证完毕！

