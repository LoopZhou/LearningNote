## vite + react + ts 项目搭建

### 创建项目

使用16.4.2的node版本
```sh
nvm use 16.4.2
```

使用vite创建react + ts项目
```sh
# 安装项目
npm create vite React-Ts-Biolerplate -- --template react-ts
```

安装
```sh
cd React-Ts-Biolerplate
# 安装
yarn
# 运行
npm run dev
```

### 代码风格配置

使用**eslint**和**prettier**控制代码风格

#### 安装
```sh
# eslint 安装
yarn add eslint --dev

# eslint plugin 安装
# eslint-plugin-react-hooks 支持校验 hook 语法
yarn add eslint-plugin-react-hooks @typescript-eslint/eslint-plugin eslint-plugin-prettier --dev

# typescript parser
yarn add @typescript-eslint/parser --dev

# 安装 prettier
yarn add prettier --dev

# 安装插件 eslint-config-prettier: 
# 解决 eslint 和 prettier 冲突, 以 prettier 的样式规范为准，使 ESLint 中的样式规范自动失效
yarn add eslint-config-prettier --dev
```

#### eslint配置

初始化配置，自主选择配置后，根目录生成.eslintrc.js

```sh
./node_modules/.bin/eslint --init
```

配置.eslintrc.js
```js
module.exports = {
  root: true,
  env: {
    browser: true,
    node: true,
    es2021: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
     // 支持校验 hook 语法
    'plugin:react-hooks/recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
    'prettier',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  plugins: ['react', '@typescript-eslint', 'prettier'],
  rules: {
    // suppress errors for missing 'import React' in files。忽略文件中没有import React语句
    'react/react-in-jsx-scope': 'off',
    '@typescript-eslint/ban-ts-ignore': 'off',
    '@typescript-eslint/no-unused-vars': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/no-var-requires': 'off',
    '@typescript-eslint/no-empty-function': 'off',
    '@typescript-eslint/no-use-before-define': 'off',
    '@typescript-eslint/ban-ts-comment': 'off',
    '@typescript-eslint/ban-types': 'off',
    '@typescript-eslint/no-non-null-assertion': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    'prettier/prettier': ['error', { singleQuote: true }],
    // 禁止出现console
    'no-console': 'warn',
    // 禁用debugger
    'no-debugger': 'warn',
    // 禁止出现重复的 case 标签
    'no-duplicate-case': 'warn',
    // 禁止出现空语句块
    'no-empty': 'warn',
    // 禁止不必要的括号
    'no-extra-parens': 'off',
    // 禁止对 function 声明重新赋值
    'no-func-assign': 'warn',
    // 禁止在 return、throw、continue 和 break 语句之后出现不可达代码
    'no-unreachable': 'warn',
    // 强制所有控制语句使用一致的括号风格
    curly: 'warn',
    // 要求 switch 语句中有 default 分支
    'default-case': 'warn',
    // 强制尽可能地使用点号
    'dot-notation': 'warn',
    // 要求使用 === 和 !==
    eqeqeq: 'warn',
    // 禁止 if 语句中 return 语句之后有 else 块
    'no-else-return': 'warn',
    // 禁止出现空函数
    'no-empty-function': 'warn',
    // 禁用不必要的嵌套块
    'no-lone-blocks': 'warn',
    // 禁止使用多个空格
    'no-multi-spaces': 'warn',
    // 禁止多次声明同一变量
    'no-redeclare': 'warn',
    // 禁止在 return 语句中使用赋值语句
    'no-return-assign': 'warn',
    // 禁用不必要的 return await
    'no-return-await': 'warn',
    // 禁止自我赋值
    'no-self-assign': 'warn',
    // 禁止自身比较
    'no-self-compare': 'warn',
    // 禁止不必要的 catch 子句
    'no-useless-catch': 'warn',
    // 禁止多余的 return 语句
    'no-useless-return': 'warn',
    // 禁止变量声明与外层作用域的变量同名
    'no-shadow': 'off',
    // 允许delete变量
    'no-delete-var': 'off',
    // 强制数组方括号中使用一致的空格
    'array-bracket-spacing': 'warn',
    // 强制在代码块中使用一致的大括号风格
    'brace-style': 'warn',
    // 强制使用骆驼拼写法命名约定
    camelcase: 'warn',
    // 强制使用一致的缩进
    indent: 'off',
    // 强制在 JSX 属性中一致地使用双引号或单引号
    // 'jsx-quotes': 'warn',
    // 强制可嵌套的块的最大深度4
    'max-depth': 'warn',
    // 强制最大行数
    // "max-lines": ["warn", { "max": 1200 }],
    // 强制函数最大代码行数
    // 'max-lines-per-function': ['warn', { max: 70 }],
    // 强制函数块最多允许的的语句数量
    'max-statements': ['warn', 100],
    // 强制回调函数最大嵌套深度
    'max-nested-callbacks': ['warn', 3],
    // 强制函数定义中最多允许的参数数量
    'max-params': ['warn', 3],
    // 强制每一行中所允许的最大语句数量
    'max-statements-per-line': ['warn', { max: 1 }],
    // 要求方法链中每个调用都有一个换行符
    'newline-per-chained-call': ['warn', { ignoreChainWithDepth: 3 }],
    // 禁止 if 作为唯一的语句出现在 else 语句中
    'no-lonely-if': 'warn',
    // 禁止空格和 tab 的混合缩进
    'no-mixed-spaces-and-tabs': 'warn',
    // 禁止出现多行空行
    'no-multiple-empty-lines': 'warn',
    // 强制在块之前使用一致的空格
    'space-before-blocks': 'warn',
    // 强制在 function的左括号之前使用一致的空格
    // 'space-before-function-paren': ['warn', 'never'],
    // 强制在圆括号内使用一致的空格
    'space-in-parens': 'warn',
    // 要求操作符周围有空格
    'space-infix-ops': 'warn',
    // 强制在一元操作符前后使用一致的空格
    'space-unary-ops': 'warn',
    // 强制在注释中 // 或 /* 使用一致的空格
    // "spaced-comment": "warn",
    // 强制在 switch 的冒号左右有空格
    'switch-colon-spacing': 'warn',
    // 强制箭头函数的箭头前后使用一致的空格
    'arrow-spacing': 'warn',
    'no-var': 'warn',
    'prefer-const': 'warn',
    'prefer-rest-params': 'warn',
    'no-useless-escape': 'warn',
    'no-irregular-whitespace': 'warn',
    'no-prototype-builtins': 'warn',
    'no-fallthrough': 'warn',
    'no-extra-boolean-cast': 'warn',
    'no-case-declarations': 'warn',
    'no-async-promise-executor': 'warn',
  },
  // https://stackoverflow.com/questions/72780296/warning-react-version-not-specified-in-eslint-plugin-react-settings-while-run
  settings: {
    react: {
      version: 'detect',
    },
  },
};
```

创建.eslintignore
```sh
# eslint 忽略检查
node_modules
dist
```

#### prettier配置

新建.prettier.js， 配置prettier规则

```js
module.exports = {
  // 一行最多 120 字符
  printWidth: 120,
  // 使用 2 个空格缩进
  tabWidth: 2,
  // 不使用缩进符，而使用空格
  useTabs: false,
  // 行尾需要有分号
  semi: true,
  // 使用单引号
  singleQuote: true,
  // 对象的 key 仅在必要时用引号
  quoteProps: 'as-needed',
  // jsx 不使用单引号，而使用双引号
  jsxSingleQuote: false,
  // 末尾需要有逗号
  trailingComma: 'all',
  // 大括号内的首尾需要空格
  bracketSpacing: true,
  // jsx 标签的反尖括号需要换行
  jsxBracketSameLine: false,
  // 箭头函数，只有一个参数的时候，也需要括号
  arrowParens: 'always',
  // 每个文件格式化的范围是文件的全部内容
  rangeStart: 0,
  rangeEnd: Infinity,
  // 不需要写文件开头的 @prettier
  requirePragma: false,
  // 不需要自动在文件开头插入 @prettier
  insertPragma: false,
  // 使用默认的折行标准
  proseWrap: 'preserve',
  // 根据显示样式决定 html 要不要折行
  htmlWhitespaceSensitivity: 'css',
  // vue 文件中的 script 和 style 内不用缩进
  vueIndentScriptAndStyle: false,
  // 换行符使用 lf
  endOfLine: 'lf',
  // json文件规则调整
  overrides: [
    {
      files: '*.json',
      options: {
        printWidth: 200,
      },
    },
  ],
};

```


#### script中增加lint命令

pacakge.json中增加lint命令: 检查src目录下的vue, js, ts文件格式， 自动修复
```sh
"lint": "eslint --fix ./src --ext .tsx,.jsx,.js,.ts"
```

### 提交commit规范

利用**commitizen**和**husky**控制commit的规范

#### commitizen

安装commitizen

```sh
yarn add commitizen cz-conventional-changelog @commitlint/config-conventional @commitlint/cli commitlint-config-cz cz-customizable --dev
```

配置package.json，增加自定义commit命令
```js
{
  ...
  "scripts": {
    "commit":"git-cz",
  },

  "config": {
      "commitizen": {
        "path": "node_modules/cz-customizable"
      }
  },
  ...
}
```

新增配置 `commitlint.config.js`
```js
module.exports = {
    extends: ['@commitlint/config-conventional', 'cz'],
    rules: {
        'type-enum': [
            2,
            'always',
            [
                'feat', // 新功能（feature）
                'bug', // 此项特别针对bug号，用于向测试反馈bug列表的bug修改情况
                'fix', // 修补bug
                'ui', // 更新 ui
                'docs', // 文档（documentation）
                'style', // 格式（不影响代码运行的变动）
                'perf', // 性能优化
                'release', // 发布
                'deploy', // 部署
                'refactor', // 重构（即不是新增功能，也不是修改bug的代码变动）
                'test', // 增加测试
                'chore', // 构建过程或辅助工具的变动
                'revert', // feat(pencil): add ‘graphiteWidth’ option (撤销之前的commit)
                'merge', // 合并分支， 例如： merge（前端页面）： feature-xxxx修改线程地址
                'build', // 打包
            ],
        ],
        // <type> 格式 小写
        'type-case': [2, 'always', 'lower-case'],
        // <type> 不能为空
        'type-empty': [2, 'never'],
        // <scope> 范围不能为空
        'scope-empty': [2, 'never'],
        // <scope> 范围格式
        'scope-case': [0],
        // <subject> 主要 message 不能为空
        'subject-empty': [2, 'never'],
        // <subject> 以什么为结束标志，禁用
        'subject-full-stop': [0, 'never'],
        // <subject> 格式，禁用
        'subject-case': [0, 'never'],
        // <body> 以空行开头
        'body-leading-blank': [1, 'always'],
        'header-max-length': [0, 'always', 72],
    },
};
```

自定义提示则添加 `.cz-config.js`

```js
module.exports = {
    types: [
        {value: 'feat',  name: 'feat:  增加新功能'},
        {value: 'bug',      name: 'bug:      测试反馈bug列表中的bug号'},
        {value: 'fix',      name: 'fix:      修复bug'},
        {value: 'ui',       name: 'ui:       更新UI'},
        {value: 'docs',     name: 'docs:     文档变更'},
        {value: 'style',    name: 'style:    代码格式(不影响代码运行的变动)'},
        {value: 'perf',     name: 'perf:     性能优化'},
        {value: 'refactor', name: 'refactor: 重构(既不是增加feature，也不是修复bug)'},
	{value: 'release',  name: 'release:  发布'},
	{value: 'deploy',   name: 'deploy:   部署'},
        {value: 'test',     name: 'test:     增加测试'},
        {value: 'chore',    name: 'chore:    构建过程或辅助工具的变动(更改配置文件)'},
        {value: 'revert',   name: 'revert:   回退'},
    	{value: 'build',    name: 'build:    打包'}
    ],
    // override the messages, defaults are as follows
    messages: {
        type: '请选择提交类型:',
        customScope: '请输入您修改的范围(可选):',
        subject: '请简要描述提交 message (必填):',
        body: '请输入详细描述(可选，待优化去除，跳过即可):',
        footer: '请输入要关闭的issue(待优化去除，跳过即可):',
        confirmCommit: '确认使用以上信息提交？(y/n/e/h)'
    },
    allowCustomScopes: true,
    skipQuestions: ['body', 'footer'],
    subjectLimit: 72
};
```

#### husky和lint-staged

安装husky，用于对commit的控制， 安装lint-staged, 用于过滤出 Git 代码暂存区文件进行检查, 防止全部文件检查
```sh
# 安装和初始化husky, 在pacakge.json中和项目中生成husky的配置
npx husky-init install
# 安装 lint-staged
yarn add lint-staged --dev
```

package.json配置lint-staged
```json
"lint-staged": {
  "src/**/*.{ts,tsx,js,jsx}": [
    "eslint -c .eslintrc.js --fix"
  ]
},
```

配置.husky/pre-commit, 完成代码规范检测
```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

echo "pre-commit";
npx lint-staged;
```

增加commit-msg的.husky
```sh
npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"
```

### vite配置别名

vite.config.ts
```ts
import { defineConfig } from "vite"
import react from "@vitejs/plugin-react"
import { resolve } from 'path';

export default defineConfig({
  plugins: [ react() ],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'), // 设置 `@` 指向 `src` 目录
    },
  },
})
```

tsconfig.json
```json
{
  "compilerOptions": {
    ...
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

### vite配置server

```js
server: {
  port: 7070, // 设置服务启动端口号
  open: true, // 设置服务启动时是否自动打开浏览器
  cors: true, // 允许跨域
},
```

### vite打包配置

```js
base: './', // 设置打包路径
// 打包配置
build: {
  target: 'modules',
  outDir: 'dist', //指定输出路径
  assetsDir: 'assets', // 指定生成静态资源的存放路径
},
```

### 路由配置

安装react-router-dom

```sh
yarn add react-router-dom
```

在main.tsx中引入使用
```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import { HashRouter } from 'react-router-dom';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <React.StrictMode>
    <HashRouter>
      <App />
    </HashRouter>
  </React.StrictMode>
);

```

调整App.tsx的内容到Home.tsx， 利用useRoutes配置路由
App.tsx
```js
import { useRoutes } from 'react-router-dom';

import Home from '@/views/Home';
import Demo from '@/views/Demo';

function App() {
  const element = useRoutes([
    {
      path: '/',
      element: <Home />,
    },
    { path: 'demo', element: <Demo /> },
  ]);

  return element;
}

export default App;
```

### 状态管理

使用@reduxjs/toolkit进行状态管理

安装
```sh
yarn add redux react-redux @reduxjs/toolkit
```

store文件目录
```
store
    ├── features
    │   ├── counterSlice.ts
    │   └── userSlice.ts
    └── index.ts
```

定义模块
counterSlice
```ts
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',

  initialState: {
    value: 0,
  },

  reducers: {
    incremented: (state) => {
      state.value += 1;
    },
    decremented: (state) => {
      state.value -= 1;
    },
    add: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { incremented, decremented, add } = counterSlice.actions;

export default counterSlice.reducer;
```

userSlice
```ts
import { createSlice } from '@reduxjs/toolkit';

const userSlice = createSlice({
  name: 'user',

  initialState: {
    name: 'randy',
    age: 24,
  },

  reducers: {
    nameIncrement: (state) => {
      state.name += '!';
    },
    nameDecrement: (state) => {
      state.name += state.name.slice(0, state.name.length - 1);
    },
    ageIncremented: (state, action) => {
      state.age += action.payload;
    },
    ageDecremented: (state, action) => {
      state.age -= action.payload;
    },
  },
});

export const { nameIncrement, nameDecrement, ageIncremented, ageDecremented } =
  userSlice.actions;

export default userSlice.reducer;
```

index.ts
```ts
import { configureStore } from '@reduxjs/toolkit';
import counterSlice from './features/counterSlice';
import userSlice from './features/userSlice';

const store: any = configureStore({
  reducer: {
    counter: counterSlice,
    user: userSlice,
  },
});

export default store;

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

使用store， 在main.tsx
```ts
import { Provider } from "react-redux";
import store from "./store";

<Provider store={store}>
  <App />
</Provider>
```

函数组件中使用：使用useSelector、useDispatch两个Hook来操作store
```ts
import { useSelector, useDispatch } from 'react-redux';
import { ageIncremented } from '@/store/features/userSlice';
import { incremented } from '@/store/features/counterSlice';
import type { RootState } from '@/store/index';

function Store() {
  const counter = useSelector((state: RootState) => state.counter.value);
  const age = useSelector((state: RootState) => state.user.age);
  const dispatch = useDispatch();

  const userAgeIncremented = () => {
    dispatch(ageIncremented(5));
  };

  const counterIncremented = () => {
    dispatch(incremented());
  };

  return (
    <div className="Store">
      <h1>Counter</h1>
      <div>
        <span>counter:</span>
        <span>{counter}</span>
        <div>
          <button onClick={counterIncremented}>counter add</button>
        </div>
      </div>
      <h1>User</h1>
      <div>
        <button onClick={userAgeIncremented}>age is {age}</button>
      </div>
    </div>
  );
}

export default Store;
```

### 示例

完整的代码托管在[React-Ts-Boilerplate](https://github.com/LoopZhou/React-Ts-Boilerplate)
