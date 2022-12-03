## storybook 学习笔记

### 安装模版

```shell
# 创建vue模版
npx -p @storybook/cli sb init --type vue

# 安装
npm install

# 添加相关vue依赖包
npm install vue --save
npm install vue-loader vue-template-compiler @babel/core babel-loader babel-preset-vue --save-dev

# 添加样式loader
npm install css-loader less-loader style-loader

# 运行
npm run storybook
```

坑：

Less-loader版本不能太高： 需要使用6.0.0版本

### 配置webpack

在.storybook/main.js中配置webpackFinal用于less

```javascript
const path = require('path');

module.exports = {
  stories: ['../stories/**/*.stories.js'],
  addons: ['@storybook/addon-actions', '@storybook/addon-links'],
  webpackFinal: async (config, { configType }) => {
    config.module.rules.push({
      test: /\.less$/,
      use: ['style-loader', 'css-loader', 'less-loader', 'vue-style-loader'],
      include: path.resolve(__dirname, '../'),
    });

    return config;
  },
};

```



### 封装elementUI

在stories.js中引入element即可

```js
import Vue from 'vue';
import elementUI from 'element-ui';
// 样式
import 'element-ui/lib/theme-chalk/index.css';

Vue.use(elementUI);
const vue = new Vue();


// 在story中使用即可
vue.$message.success('success');
```



### 增加示例

方法一: export 组件

```javascript
1. import Loading from './index.vue';
import './stories.less';

export default {
  title: 'Loading',
  component: Loading,
};

export const Simple = () => ({
  components: { Loading },
  template: '<Loading class="center" />',
});
```

方法二: storyof方法

```js
import { storiesOf } from '@storybook/vue';

storiesOf('copyAble', module)
    .add('basic', () => ({
        components: { copyAble },
        template: `
            <copy-able :value="content" @success="onSuccess"/>
        `,
        data() {
            return {
                content: '自定义文本复制功能',
            };
        },
        methods: {
            onSuccess() {
                vue.$message.success('success');
            },
        },
    }));

```



### 配置插件

##### 模版中已安装的插件:

addon-action:  控制台功能, 展示交互时的console数据

addon-link: 链接功能



##### 添加storysource插件: 用于展示显示内容的源码

[配置参考](https://github.com/storybookjs/storybook/tree/next/addons/storysource)

``` javascript
# 安装插件
npm install @storybook/addon-storysource --save-dev 

# .storybook/main.js中配置插件
addons: ['@storybook/addon-actions', '@storybook/addon-links', '@storybook/addon-storysource'],

# 使用prettierConfig配置显示代码格式
// storysource
config.module.rules.push(
      // https://github.com/storybookjs/storybook/tree/next/addons/storysource#prettierconfig
      {
          test: /\.stories\.jsx?$/,
          loaders: [
              {
                  loader: require.resolve('@storybook/source-loader'),
                  options: {
                      prettierConfig: {
                          printWidth: 100,
                          singleQuote: false,
                      },
                  },
              },
          ],
          enforce: 'pre',
      }
);

# 运行查看多了story面板,查看展示的源码
npm run storybook
```



##### 添加knobs插件

[参考配置](https://github.com/storybookjs/storybook/tree/next/addons/knobs)

安装

```shell
npm install @storybook/addon-knobs --save-dev
```

.storybook/main.js中配置

```
addons: ['@storybook/addon-knobs'],
```

.stories.js中使用

```js
import { withKnobs, text, boolean, number, array, select } from '@storybook/addon-knobs';

// export方法添加decorator
export default {
  title: 'Storybook Knobs',
  decorators: [withKnobs],
};

// storiesOf添加decorator
storiesOf('copyAble', module)
    .addDecorator(withKnobs)

// 组件的props定义组件, 具体各组件使用参考配置
props: {
    content: {
				default: text('content', '自定义文本复制功能-可编辑'),
    },
    align: {
 				default: select('align', {
            '左': 'left',
            '中': '',
            '右': 'right',
        }, 'left'),
    },
},
```

运行后有knobs面板可配置参数



##### 添加viewport插件: 用于改变页面呈现大小，用于响应式布局测试

[参考配置](https://github.com/storybookjs/storybook/tree/next/addons/viewport)

安装

```
npm i -D @storybook/addon-viewport
```

.storybook/main.js中配置

```
module.exports = {
  addons: ['@storybook/addon-viewport'],
};
```

渲染页面左上角增加更改渲染大小的button





```
import TooltipText from '../comm/tooltip-text';

export default {
  title: 'Example/TooltipText',
  component: TooltipText,
};

const Template = (args, { argTypes }) => ({
  props: Object.keys(argTypes),
  components: { TooltipText },
  template: '<tooltip-text v-bind="$props" style="max-width: 30px"/>',
});

export const Primary = Template.bind({});
Primary.args = {
  content: '验证文本长度较长的情况。显示内容展示'
};

```



