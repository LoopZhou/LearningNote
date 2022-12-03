# 前端markdown文件处理

### 简介

项目中希望完成帮助文档的功能。因此希望以markdown文件的形式完成帮助文档，前端通过markdown显示。



### 生成静态页面

生成静态页面的方案可以直接使用gitbook, vuepress的方法完成，但是未来涉及到目录的自定义控制，搜索等功能并不容易实现。



### 使用loader加载markdown文件

Demo.md

```markdown
# demo

this is a demo file.
```



1. 使用raw-loader加载md文件

```sh
yarn add raw-loader -D
```

vue.config.js

```js
  chainWebpack: (config) => {
    config.module
      .rule('md')
      .test(/\.md$/)
      .use('raw-loader')
        .loader('raw-loader')
        .end();
  },
```

vue文件中读取: 结果是文本内容,  可以使用marked转换成html标签

```js
import demo from "./components/demo.md";
import marked from 'marked';

export default {
  data() {
    return {
      demo,                // # demo this is a demo file.
      markdownCode: '',    // <h1 id="demo">demo</h1> <p>this is a demo file.</p>
    };
  },
  created() {
    this.markdownCode = marked(this.demo);
  },
};
```



2. 直接使用html-loader和markdown-loader完成md文件的读取

**注意**： 此方法下html-loader需要使用低版本html-loader": "1.3.0"， 高版本会报错

```
yarn add html-loader@1.3.0 markdown-loader -D
```



vue.config.js

```js
  chainWebpack: (config) => {
    config.module
      .rule('md')
      .test(/\.md$/)
      .use('html-loader')
        .loader('html-loader')
        .end()
      .use('markdown-loader')
        .loader('markdown-loader')
        .end();
        
  },
```

vue中读取结构就是html内容了

```js
import demo from "./components/demo.md";

export default {
  data() {
    return {
      demo,        // <h1 id="demo">demo</h1> <p>this is a demo file.</p>
    };
  },
};
```



3. 使用vue-markdown-loader 完成md文件的读取

```md
yarn add vue-markdown-loader vue-loader vue-template-compiler -D
yarn add github-markdown-css highlight.js
```



vue.config.js

```js
module.exports = {
  chainWebpack: config => {
    config.module.rule('md')
      .test(/\.md/)
      .use('vue-loader')
      .loader('vue-loader')
      .end()
      .use('vue-markdown-loader')
      .loader('vue-markdown-loader/lib/markdown-compiler')
      .options({
        raw: true
      })
  }
}
```



vue中import之后就是一个vue组件了

```vue
<template>
	<!-- class markdown-body 必须加，否则标签样式会出现问题 -->
  <div class="markdown-body">
    <markdown />
  </div>
</template>

<script>
// 引入 markdown 文件，引入后是一个组件，需要在 components 中注册
import markdown from '@/assets/ApiDocument.md'
// 代码高亮
import 'highlight.js/styles/github.css'
// 其他元素使用 github 的样式
import 'github-markdown-css'
export default {
  components: {
    markdown
  },
}
</script>
```

**注意**： github-markdown-css highlight.js库的使用，是在div下面加一个class： markdown-body

这种方式生成的dom节点没有id





### 在markdown中的链接中增加前缀

因为我们项目中会在前缀中区分环境，所以需要在markdown的链接中增加前缀。

markdown-loader也是使用的marked来进行转换的， 而marked的options中提供了baseUrl的方式

```js
this.code = marked(this.markdown, {
  baseUrl: 'http://www.baidu.com',
})
```

而markdown-loader的配置：

```js
    const baseUrl = process.env.VUE_APP_ROUTER_BASE;
    const markdownUrl = baseUrl.endsWith('/') ? baseUrl : `${baseUrl}/`;
    // markdown文件读取
    config.module
      .rule('md')
      .test(/\.md$/)
      .use('html-loader')
      .loader('html-loader')
      .end()
      .use('markdown-loader')
      .loader('markdown-loader')
      .options({ 
        baseUrl: markdownUrl, 
      })
      .end();
```

效果：

只有markdown中使用markdown语法的链接才能转换，使用html标签则不行

```
 [查询批量授权模板信息](service-category?type=message)      //会增加前缀
 <a href="/service-category?type=message">查询批量授权模板信息</a>   //不会增加前缀
```

