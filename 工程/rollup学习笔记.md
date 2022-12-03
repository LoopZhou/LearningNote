# Rollup学习

### 1. 打包工具

**Webpack:** 前端资源模块化 管理和打包工具。 目前最主流的打包工具， 将松散的模块按照依赖和规则打包成前端资源。适用于大型的应用项目，以模块划分，按需加载。

**Rollup:** 模块打包器，可以将小块代码编译成大块复杂的代码。适用于框架和工具库的构建， 如目前的vue, react, 和部分ui库。

**Gulp:** 基于任务驱动的自动化构建工具。 适合小项目，基于流程构建



工具库开发更适合rollup来打包： webpack会添加一些自有的模块加载代码，就不太适合了。rollup会帮我们生成支持umd/commonjs/es的js代码。

### 2. 基本项目构建

初始化项目

```sh
mkdir rollup && cd rollup
npm init -y

npm install --global rollup 
```

最基本打包

创建文件：

 index.js

```js
import constant from'./constant.js'
import { function1 } from'./function.js'

function1();
console.log(constant.constant1)
console.log('Rollup')
```

constant.js

```js
export default {
  constant1: 'constant1',
  constant2: 'constant2',
};
```

function.js

```js
export const function1 = () => {
  console.log('function1');
}

export const function2 = () => {
  console.log(2);
}
```

package.json中添加type: module， node执行

```
node index.js

function1
constant1
Rollup
```

使用rollup打包

命令行： 

```sh
rollup index.js  #输出到屏幕
rollup index.js --file bundle.js #打包到文件bundle.js
rollup main.js --file bundle.js --format cjs #打包成cjs
 
```

配置文件rollup.config.js

```js
export default {
  // 包的入口点 (例如：你的 main.js 或者 index.js)
  input:'page/index.js',
  // 出口配置  
  output:{
    file:'budle.js',//打包到那个文件
    format:'esm'// 生成包的格式
  }
}
```

运行： rollup --config rollup.config.js

output.format: 

```
amd – 异步模块定义，用于像 RequireJS 这样的模块加载器
cjs – CommonJS，适用于 Node 和 Browserify/Webpack
esm – 将软件包保存为 ES 模块文件，在现代浏览器中可以通过
iife – 一个自动执行的功能，适合作为
umd – 通用模块定义，以 amd，cjs 和 iife 为一体
system - SystemJS 加载器格式
```

**结果**

Esm

```js
var constant = {
  constant1: 'constant1',
  constant2: 'constant2',
};

const function1 = () => {
  console.log('function1');
};

function1();
console.log(constant.constant1);
console.log('Rollup');

```

umd

```js
(function (factory) {
  typeof define === 'function' && define.amd ? define(factory) :
  factory();
}((function () { 'use strict';

  var constant = {
    constant1: 'constant1',
    constant2: 'constant2',
  };

  const function1 = () => {
    console.log('function1');
  };

  function1();
  console.log(constant.constant1);
  console.log('Rollup');

})));

```

### 4. 配置文件rollup.config.js

```js
// rollup.config.js
export default {
  // 核心选项
  input,     // 必须, 入口文件地址
  external,  // ['lodash'] 外部依赖，不将此包打包
  plugins,   // 各种插件

  // 额外选项
  onwarn,

  // danger zone
  acorn,
  context,
  moduleContext,
  legacy

  output: {  // 必须 (如果要输出多个，可以是一个数组)
    // 核心选项
    file,    // 必须， 打包后存放到文件
    format,  // 必须， 输出格式： amd, es6, umd, cjs, iife
    name,		 // 如果是iife, umd需指定一个全局变量
    globals,

    // 额外选项
    paths,
    banner,
    footer,
    intro,
    outro,
    sourcemap,
    sourcemapFile,
    interop,

    // 高危选项
    exports,
    amd,
    indent
    strict
  },
};
```

### 5. 常用插件

webpack的处理分为loader和plugin,  loader对源码进行预处理， plugin完成构建打包到特定需求。

rollup的plugin兼具了webpack的loader和plugin的功能呢

```sh
# Babel编译js以适配浏览器
npm install rollup-plugin-babel  --save-dev

# node模块到引用
# rollup-plugin-node-resolve 插件允许我们加载第三方模块
# @rollup/plugin-commons 插件将它们转换为ES6版本
npm install @rollup/plugin-node-resolve @rollup/plugin-commonjs --save-dev

# 使用typescript
npm install @rollup/plugin-typescript --save-dev

# 压缩代码
npm install rollup-plugin-terser --save-dev

# 编译CSS， 默认已集成了对scss、less、stylus的支持
npm install rollup-plugin-postcss --save-dev
# 单独的Scss
npm install rollup-plugin-scss --save-dev

# 本地服务器和热更新
npm install rollup-plugin-serve --save-dev
npm install rollup-plugin-livereload --save-dev

# vue 类似于 webpack 的vue-loader，vue 组件的加载器
npm install rollup-plugin-vue --save-dev
# vue中使用jsx
npm install @vue/babel-preset-jsx

# 打包的时候用来排除 package.json 内 peerDependencies 字段内的包
npm install  rollup-plugin-peer-deps-external --save-dev

# json 文件解析
npm install @rollup/plugin-json --save-dev

# 输出rollup的打包进度
npm install rollup-plugin-progress --save-dev

```



```js
import babel from 'rollup-plugin-babel'
import resolve from '@rollup/plugin-node-resolve'
import typescript from '@rollup/plugin-typescript'
import { terser } from 'rollup-plugin-terser'
import postcss from 'rollup-plugin-postcss'
import json from '@rollup/plugin-json'
import peerDepsExternal from 'rollup-plugin-peer-deps-external'
import vuePlugin from 'rollup-plugin-vue'
import rollProgress from 'rollup-plugin-progress'


export default {
  ...
  plugins: [
    resolve(),
    typescript(),
    babel({
      exclude: 'node_modules/**',
      presets: ["@vue/babel-preset-jsx"]
    })，
    terser(),
  	postcss()，
    livereload(),
    serve({
      open: true,
      contentBase: 'dist'
    }),
    json(),
    peerDepsExternal(),
    vuePlugin({
      css: true
    }),
    rollProgress()
  ],
}

```

基本示例中添加上plugin:

```js
import babel from 'rollup-plugin-babel'
import resolve from '@rollup/plugin-node-resolve'
import { terser } from 'rollup-plugin-terser'
import rollProgress from 'rollup-plugin-progress'

export default {
  // 包的入口点 (例如：你的 main.js 或者 index.js)
  input:'page/index.js',
  // 出口配置  
  output:{
    file:'budle.js',//打包到那个文件
    format:'esm'// 生成包的格式
  },
  plugins: [
    resolve(),
    babel({
      exclude: 'node_modules/**'
    }),
    terser(),
    rollProgress()
  ],
}
```

打包结果：

```js
console.log("function1"),console.log("constant1"),console.log("Rollup");
```





### 参考资料

1. [Rollup](https://rollupjs.org/guide/en/)

2. [Rollup教程](https://juejin.cn/post/6956501799327137828#heading-51)
