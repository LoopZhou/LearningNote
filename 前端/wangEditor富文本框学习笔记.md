# wangEditor富文本编辑器学习笔记

## 简介

web富文本框组件，如wangEditor, vue-quill-editor，tinymce, CKEditor等。

根据npm和GitHub上到情况：

> **wangEditor:**
>
> Weekly Download: 8200
>
> License: MIT
>
> Unpacked Size: 3.58M
>
> Last publish: 5 days ago
>
> Star: 11.3k



> **vue-quill-editor:**
>
> Weekly Download: 33,234
>
> License: MIT
>
> Unpacked Size: 41.5k
>
> Last publish: 3 years ago
>
> Star: 6.4k
>



wangEditor本身支持功能较多，较好扩展，文档也比较完善。



## wangEditor

### 安装

````shell
npm i wangeditor --save
````



### 简单的使用

```javascript
import E from 'wangeditor'
const editor = new E('#div1')
// 或者 const editor = new E( document.getElementById('div1') )
editor.create()
```



### 封装组件

主要完成自定义配置：

1. 自定义上传图片接口
2. 增加上传文件菜单按钮
3. 拖动上传文件和图片功能
4. 增加代码高亮配置
5. 防止v-html的xss攻击问题
6. 表格，引用， 代码样式



#### 上传图片

```js
// editor.config中进行基本配置
editor.config.uploadImgServer = '/api/upload' //server接口地址
editor.config.uploadImgMaxSize = '2 * 1024 * 1024 ' //2M //限制大小
editor.config.uploadImgAccept = ['jpg', 'jpeg', 'png', 'gif', 'bmp'] //限制类型
editor.config.uploadImgMaxLength = 5 // 一次最多上传 5 个图片
editor.config.uploadFileName = 'your-custom-fileName' //自定义文件名
editor.config.uploadImgTimeout = 5 * 1000 //timeout时间， 默认10s
editor.config.withCredentials = true //允许跨域上传
editor.config.showLinkImg = false //隐藏插入网络图片的功能, 点击即可上传，不会再有浮动弹窗选择

// 自定义headeer
editor.config.uploadImgHeaders = {
    Accept: 'text/x-json',
    a: 100,
}

// 上传参数
editor.config.uploadImgParams = {
    token: 'xxxxx',
    x: 100
}
// 参数是否拼接到url中
editor.config.uploadImgParamsWithUrl = true

// 自定义上传方法
editor.config.customUploadImg = function (resultFiles, insertImgFn) {
    // resultFiles 是 input 中选中的文件列表
    // insertImgFn 是获取图片 url 后，插入到编辑器的方法

    // 上传图片，返回结果，将图片插入到编辑器中
    insertImgFn(imgUrl)
}
```

如果需要做md5校验等操作，选用自定义上传方法实现



#### 自定义菜单

自定义菜单有三种形式： button按钮点击， dropList选择框， panel面板模版。使用继承基础类等方式添加：

button: BtnMenu

dropList: DropListMenu

Panel:  MenuActive 

本项目中使用简单的按钮操作，主要实现$elem的Dom样式以及clickHandler操作即可。

```js
import Editor from 'wangeditor';

const { BtnMenu } = Editor;

export class ClearMenu extends BtnMenu {
  constructor(editor) {
    const $elem = Editor.$(`<div class="w-e-menu" data-title="清空内容">
      <i class="icon-line-edit-delete" />
    </div>`);

    super($elem, editor);
  }

  // 菜单点击事件
  clickHandler() {
    const { editor } = this;

    editor.txt.clear();
  }

  // 尝试修改菜单 active 状态
  tryChangeActive() {}
}

```

在组件中注册菜单

```js
# 全局模式
E.registerMenu('clear', ClearMenu)

# 示例模式
const editor = new E('#div1')
editor.menus.extend('clear', ClearMenu)
# editor.config.menus中配置clear的位置
```



#### 拖动上传

主要使用到了编辑区域的事件回掉功能eventHooks

```js
// eventHooks 举例
{
    dropEvents: Function[]  // 拖动
    clickEvents: Function[] // 点击
    keyupEvents: Function[] // 键盘
    tabUpEvents: Function[] // tab 键（keyCode === ）Up 时
    tabDownEvents: Function[] // tab 键（keyCode === 9）Down 时
    enterUpEvents: Function[] // enter 键（keyCode === 13）up 时
    deleteUpEvents: Function[] // 删除键（keyCode === 8）up 时
    deleteDownEvents: Function[] // 删除键（keyCode === 8）down 时
    pasteEvents: Function[] // 粘贴事件
    linkClickEvents: Function[] // 点击链接事件
    textScrollEvents: Function[] // 编辑区域滑动事件
    toolbarClickEvents: Function[] // 菜单栏被点击
    imgClickEvents: Function[] // 图片被点击事件
    // 等等……
}
```

为了实现拖动到编辑区域上传文件的功能，主要对editor.txt.eventHooks.dropEvents进行修改。

原本该组件已实现了imageUpload的dropEvents方法。如果push进去会进行文件上传操作和图片上传操作，所以我们这里使用覆写实现，先判断是否是文件格式，上传文件，否则再进行图片上传

```js
      // 原始是支持图片拖动上传功能
      const imgDrop = editor.txt.eventHooks.dropEvents[0];
      
      // 自定义拖动上传功能
      const dropFileHandler = (e) => {
        const files = e.dataTransfer && e.dataTransfer.files;

        // 检查文件，只支持拖动上传一个文件
        if (!files || !files.length) {
          return;
        }

        if (files.lenght > 1) {
          this.$message.warning('只能拖动上传一个文件');

          return;
        }

        // 检查文件类型
        const file = files[0];
        const suffix = file.name.split('.').pop()
          .toLowerCase();
        const fileSuffix = ['pdf', 'doc', 'docx', 'xls', 'xlsx', 'zip', 'rar', 'txt', 'pptx', 'ppt'];

        // 文件上传
        if (fileSuffix.includes(suffix)) {
          const uploadFile = new UploadMenu();

          // 调用文件上传功能
          uploadFile.onUpload(editor, file);

          return;
        }

        // 图片上传
        imgDrop(e);
      };

      // 重置拖动上传功能
      editor.txt.eventHooks.dropEvents = [dropFileHandler];
```



#### 增加代码高亮配置

通过highlight.js实现

```js
import hljs from 'highlight.js'
import 'highlight.js/styles/monokai-sublime.css' //使用的highlight样式


// 挂载highlight插件
editor.highlight = hljs
```



#### xss攻击

使用xss来对文本进行过滤

```js
// 安装 npm install xss
// 下面的示例，会有alert展示，会触发img标签的error事件，最终alert弹框。这便是一个最简单的xss攻击。
this.content.description = "<img src='http://www.xxx.com' onerror='alert(1)'/> "

// wangEditor的文本html需要进行xss过滤
editor.txt.html(_.unescape(this.xss(this.content.description)));

// v-html也需要进行
<div v-html="xss(content.description)" class="w-e-text show-html" />
```



考虑到vue的v-html均会有xss的风险，这里进行全局配置

main.js中引入xss包并挂载到vue原型上

```js
import xss from 'xss'
Vue.prototype.xss = xss
```

在vue.config.js中覆写html指令

```js
  chainWebpack: config => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .loader('vue-loader')
      .tap(options => {
        options.compilerOptions.directives = {
          html(node, directiveMeta) {
            (node.props || (node.props = [])).push({
              name: 'innerHTML',
              value: `xss(_s(${directiveMeta.value}))`
            })
          }
        }
        return options
      })
  }
```

问题： 由于wangEditor富文本框中实现会有样式等功能标签的添加，默认情况下的xss过滤白名单会过滤掉相关功能，这里我们需要手动配置白名单。

xss默认白名单：[getDefaultWhiteList](https://github.com/leizongmin/js-xss/blob/master/dist/xss.js)

增加xss.js: 在div, table, font, span, h1-h6等标签的style

```js
const xss = require('xss');

export const filterXss = new xss.FilterXSS({
  whiteList: {
    a: ['target', 'href', 'title', 'style', 'class'],
    abbr: ['title'],
    address: [],
    area: ['shape', 'coords', 'href', 'alt'],
    article: [],
    aside: [],
    audio: ['autoplay', 'controls', 'loop', 'preload', 'src'],
    b: [],
    bdi: ['dir'],
    bdo: ['dir'],
    big: [],
    blockquote: ['cite'],
    br: [],
    caption: [],
    center: [],
    cite: [],
    code: [],
    col: ['align', 'valign', 'span', 'width'],
    colgroup: ['align', 'valign', 'span', 'width'],
    dd: [],
    del: ['datetime'],
    details: ['open'],
    div: ['style', 'class'],
    dl: [],
    dt: [],
    em: [],
    font: ['color', 'size', 'face', 'style', 'class'],
    footer: [],
    h1: ['style', 'class'],
    h2: ['style', 'class'],
    h3: ['style', 'class'],
    h4: ['style', 'class'],
    h5: ['style', 'class'],
    h6: ['style', 'class'],
    header: [],
    hr: [],
    i: [],
    img: ['src', 'alt', 'title', 'width', 'height', 'style'],
    ins: ['datetime'],
    li: ['class', 'style'],
    mark: [],
    nav: [],
    ol: [],
    p: ['style', 'class'],
    pre: [],
    s: [],
    section: [],
    small: [],
    span: ['style', 'class', 'onclick'],
    sub: [],
    sup: [],
    strong: [],
    strike: ['style', 'class'],
    table: ['width', 'border', 'align', 'valign', 'style'],
    tbody: ['align', 'valign'],
    td: ['width', 'rowspan', 'colspan', 'align', 'valign', 'style'],
    tfoot: ['align', 'valign'],
    th: ['width', 'rowspan', 'colspan', 'align', 'valign'],
    thead: ['align', 'valign'],
    tr: ['rowspan', 'align', 'valign'],
    tt: [],
    u: [],
    ul: [],
    video: ['autoplay', 'controls', 'loop', 'preload', 'src', 'height', 'width'],
  },
  css: false,									// 不过滤css标签
  stripIgnoreTag: true,				// 忽略掉非白名单中的标签
});

```

main.js中使用

```js
import { filterXss } from './xss';
Vue.prototype.$xss = filterXss;
```

vue.config.js中配置

```js
    // 复写html指令，防止xss攻击
    config.module
      .rule('vue')
      .use('vue-loader')
      .loader('vue-loader')
      .tap((options) => {
        options.compilerOptions.directives = {
          html(node, directiveMeta) {
            (node.props || (node.props = [])).push({
              name: 'innerHTML',
              value: `$xss.process(_s(${directiveMeta.value}))`,
            });
          },
        };

        return options;
      });
```



#### 表格，代码，引用样式

由于在wangEditor中编辑器内有w-e-text-container样式，包含table等样式，但在我们使用v-html显示时，如果需要和编辑器内样式保持一致，需要自定义配置相关样式。参考[wangEditor文档](https://www.kancloud.cn/wangfupeng/wangeditor3/335775)提供的编辑器样式，自定义样式：

```css
   table {
    border-top: 1px solid #ccc;
    border-left: 1px solid #ccc;

    td,
    th {
      border-bottom: 1px solid #ccc;
      border-right: 1px solid #ccc;
      padding: 3px 5px;
      height: 30px;
    }

    th {
      border-bottom: 2px solid #ccc;
      text-align: center;
      background-color: #f1f1f1;
    }
  }

  blockquote {
    display: block;
    border-left: 8px solid #d0e5f2;
    padding: 5px 10px;
    margin: 10px 0;
    line-height: 1.4;
    font-size: 100%;
    background-color: #f1f1f1;

    p {
      line-height: 1.5;
      margin: 10px 0;
    }
  }

  code {
    display: inline-block;
    *display: inline;
    *zoom: 1;
    background-color: #f1f1f1;
    border-radius: 3px;
    padding: 3px 5px;
    margin: 0 3px;
  }

  pre code {
    display: block;
  }

  hr {
    display: block;
    height: 0px;
    border: 0;
    border-top: 3px solid #ccc;
    margin: 20px 0;
  }
```



#### 完整实现

_base.vue

```vue
<template>
  <div>
    <div
      v-if="showHtml"
      v-html="content.description"
      class="show-html" />
    <div
      ref="editor"
      v-else
    />
  </div>
</template>
<script>
import Editor from 'wangeditor';
import BMF from 'browser-md5-file';
import axios from 'axios';
import _ from 'lodash';
import hljs from 'highlight.js';
import { UploadMenu } from './upload-menu';
import { ClearMenu } from './clear-menu';

import 'highlight.js/styles/github.css';

const bmf = new BMF();

export default {
  name: 'WangEditor',
  props: {
    content: {
      type: Object,
      default: () => ({ description: '' }),
    },
    readonly: {
      type: Boolean,
      default: false,
    },
    showHtml: {
      type: Boolean,
      default: false,
    },
    height: {
      type: Number,
      default: 600,
    },
    placeholder: {
      type: String,
      default: '请输入正文',
    },
  },
  mounted() {
    !this.showHtml && this.init();
  },
  methods: {
    init() {
      const editor = new Editor(this.$refs.editor);

      editor.config.height = this.height;
      editor.config.placeholder =  this.placeholder;
      editor.highlight = hljs;

      // 只读模式，清空菜单栏
      if (this.readonly) {
        editor.config.menus = [];
      }

      // 数据更新
      editor.config.onchange = (html) => {
        Object.assign(this.content, {
          description: html,
        });
      };

      // 菜单
      this.initMenu(editor);

      // 上传图片
      this.initUploadImg(editor);

      // 上传视频
      this.initUploadVideo(editor);

      editor.create();
      // 初始化，设置默认值
      editor.txt.html(_.unescape(this.content.description));
      // 是否可读
      editor.$textElem.attr('contenteditable', !this.readonly);

      // 拖动上传
      this.initDropEvent(editor);
    },
    initMenu(editor) {
      // 注册菜单
      editor.menus.extend('upload', UploadMenu);
      editor.menus.extend('clear', ClearMenu);

      // 菜单配置
      editor.config.menus = [
        'fontName',
        'fontSize',
        'head',
        'bold',
        'italic',
        'underline',
        'strikeThrough',
        // 'indent',
        // 'lineHeight',
        'foreColor',
        'backColor',
        'link',
        'list',
        'justify',
        'quote',
        'image',
        // 'video',
        'upload',
        'table',
        'code',
        'splitLine',
        'undo',
        'redo',
        'clear',
      ];
    },
    initUploadImg(editor) {
      editor.config.uploadImgServer = '/api/file/upload';
      editor.config.uploadImgMaxSize = 10 * 1024 * 1024; // 10M
      editor.config.uploadImgMaxLength = 1; // 一次最多上传 5 个图片
      editor.config.uploadImgAccept = ['jpg', 'jpeg', 'png', 'gif', 'bmp'];
      editor.config.showLinkImg = false;

      editor.config.customUploadImg = function (resultFiles, insertImgFn) {
        const file = resultFiles[0];

        // 计算md5
        bmf.md5(
          file,
          async (err, md5) => {
            console.log('err:', err);
            const info = {
              encrypt: 0,
              scope: 0,
              auth_type: 0,
              type: 2,
              size: file.size,
              name: file.name,
              md5,
            };

            const formData = new FormData();

            formData.append('file', file);
            formData.append('info', JSON.stringify(info));

            const { data } = await axios({
              method: 'POST',
              url: '/api/file/upload',
              data: formData,
            });

            if (data.code !== 0) {
              this.$message.error('图片上传失败');
            } else {
              insertImgFn(data.data);
            }
          },
          () => {}
        );
      };
    },
    initUploadVideo(editor) {
      editor.config.uploadVideoServer = '/api/file/upload';
      editor.config.uploadVideoMaxSize = 1 * 1024 * 1024 * 1024; // 1024M
      editor.config.uploadVideoAccept = ['mp4'];
      editor.config.showLinkVideo = false;

      editor.config.customInsertVideo = function (resultFiles, insertVideoFn) {
        const file = resultFiles[0];

        // 计算md5
        bmf.md5(
          file,
          async (err, md5) => {
            console.log('err:', err);
            const info = {
              encrypt: 0,
              scope: 0,
              auth_type: 0,
              type: 2,
              size: file.size,
              name: file.name,
              md5,
            };

            const formData = new FormData();

            formData.append('file', file);
            formData.append('info', JSON.stringify(info));

            const { data } = await axios({
              method: 'POST',
              url: '/api/file/upload',
              data: formData,
            });

            if (data.code !== 0) {
              this.$message.error('视频上传失败');
            } else {
              insertVideoFn(data.data);
            }
          },
          () => {}
        );
      };
    },
    initDropEvent(editor) {
      const imgDrop = editor.txt.eventHooks.dropEvents[0];
      const dropFileHandler = (e) => {
        const files = e.dataTransfer && e.dataTransfer.files;

        if (!files || !files.length) {
          return;
        }

        if (files.lenght > 1) {
          this.$message.warning('只能拖动上传一个文件');

          return;
        }

        const file = files[0];
        const suffix = file.name.split('.').pop()
          .toLowerCase();
        const fileSuffix = ['pdf', 'doc', 'docx', 'xls', 'xlsx', 'zip', 'rar', 'txt', 'pptx', 'ppt'];

        if (fileSuffix.includes(suffix)) {
          const uploadFile = new UploadMenu();

          uploadFile.onUpload(editor, file);

          return;
        }

        imgDrop(e);
      };

      editor.txt.eventHooks.dropEvents = [dropFileHandler];
    },
  },
};
</script>

<style lang="less" scoped>
.show-html {
  padding: 0;

  /deep/ table {
    border-top: 1px solid #ccc;
    border-left: 1px solid #ccc;

    td,
    th {
      border-bottom: 1px solid #ccc;
      border-right: 1px solid #ccc;
      padding: 3px 5px;
      height: 30px;
    }

    th {
      border-bottom: 2px solid #ccc;
      text-align: center;
      background-color: #f1f1f1;
    }
  }

  /deep/ blockquote {
    display: block;
    border-left: 8px solid #d0e5f2;
    padding: 5px 10px;
    margin: 10px 0;
    line-height: 1.4;
    font-size: 100%;
    background-color: #f1f1f1;

    p {
      line-height: 1.5;
      margin: 10px 0;
    }
  }

  /deep/ code {
    display: inline-block;
    *display: inline;
    *zoom: 1;
    background-color: #f1f1f1;
    border-radius: 3px;
    padding: 3px 5px;
    margin: 0 3px;
  }

  /deep/ pre code {
    display: block;
  }

  /deep/ hr {
    display: block;
    height: 0px;
    border: 0;
    border-top: 3px solid #ccc;
    margin: 20px 0;
  }
}
</style>

```



菜单插件：

自定义上传文件菜单

```js
import Editor from 'wangeditor';
import BMF from 'browser-md5-file';
import axios from 'axios';

const bmf = new BMF();
const { BtnMenu } = Editor;

// 继承BtnMenu，自定义菜单按钮
export class UploadMenu extends BtnMenu {
  constructor(editor) {
    // 按钮图标
    const $elem = Editor.$(`<div class="w-e-menu" data-title="上传">
      <i class="icon-line-direction-upload" />
    </div>`);

    super($elem, editor);
  }

  // 获取后缀
  getSuffix(name) {
    const suffix = name.split('.').pop()
      .toLowerCase();
    const types = {
      doc: 'docx',
      ppt: 'pptx',
      xls: 'xlsx',
    };

    return types[suffix] || suffix;
  }

  // size显示
  sizeShow(size) {
    const nBytes = size;
    let sOutput = `${nBytes} bytes`;
    const aMultiples = ['k', 'm', 'g', 't', 'p', 'e', 'z', 'y'];

    for (
      let nMultiple = 0, nApprox = nBytes / 1024;
      nApprox > 1;
      nApprox /= 1024, nMultiple = nMultiple + 1
    ) {
      sOutput = nApprox.toFixed(0) + aMultiples[nMultiple];
    }

    return sOutput;
  }

  // 上传文件
  onUpload(editor, file) {
    // 插入格式图片和超链接
    const sDom = info => (`
      <span style="cursor: pointer" onclick="window.open('${info.link}', '_self')">
        <img class="icon-file" alt="" src="/static/suffix/icon_${info.suffix}.png" />
      </span>
      <a href="${info.link}" download>${info.name}（${info.size}）</a>
    `);

    // 回掉函数：editor插入文件信息
    const callback = (info) => {
      editor.cmd.do(
        'insertHTML',
        sDom(info),
      );
    };

    this.uploadFile(file, callback);
  }

  // 上传文件
  uploadFile(file, callback) {
    // 计算md5
    bmf.md5(
      file,
      async (err, md5) => {
        console.log('err', err);
        const info = {
          encrypt: 0,
          scope: 0,
          auth_type: 0,
          type: 2,
          size: file.size,
          name: file.name,
          md5,
        };
        const formData = new FormData();

        formData.append('file', file);
        formData.append('info', JSON.stringify(info));
        const { data } = await axios({
          method: 'POST',
          url: '/api/file/upload',
          data: formData,
        });

        if (data.code !== 0) {
          this.$message.error('上传失败');
        } else {
          const info = {
            name: file.name,
            size: this.sizeShow(file.size),
            suffix: this.getSuffix(file.name),
            link: data.data,
          };

          callback(info);
        }
      },
      () => {}
    );
  }

  // 菜单点击事件
  clickHandler() {
    const { editor } = this;

    // 上传文件类型
    const input = document.createElement('input');

    input.type = 'file';
    input.accept = '.doc,.pdf,.txt,.ppt,.rar,.zip,.rar,.xlsx';

    input.onchange = () => {
      if (input.files.length) {
        const file = input.files[0];

        // 上传文件
        this.onUpload(editor, file);
      }
    };

    input.click();
  }

  // 尝试修改菜单 active 状态
  tryChangeActive() {}
}

```



### 参考资料

1. [wangEditor](https://www.wangeditor.com/)
2. [vue-quill-editor](https://github.surmon.me/vue-quill-editor/)
3. [wangEditor github](https://github.com/wangeditor-team/wangEditor/)
4. [vue-quill-editor github](https://github.com/surmon-china/vue-quill-editor)
5. [vue-quill-editor 文档](https://www.kancloud.cn/liuwave/quill/1434140)
6. [js-xss](https://github.com/leizongmin/js-xss/blob/master/README.zh.md)
7. [各种文本编辑器工具](https://blog.zhangbing.site/2021/02/27/top-free-wysiwyg-text-editing-tools/)

