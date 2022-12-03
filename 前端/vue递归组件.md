## Vue递归组件

### 简介

对于树形，评论留言版等前端组件，有许多规律的DOM结构，可以通过递归的方式生成这个结构。本文主要使用评论留言板组件为例， 使用vue封装递归组件，在vue组件中自己调用自己

### 留言板组件

```vue
<template>
    <div>
        <ul class="comment-list" v-for="item in list" :key="item.id">
            <li>
                <div class="comment-block">
                    <div class="comment-head">
                        <div>
                            <span class="comment-name">{{item.name}}</span>
                            <span class="comment-time">{{item.createdAt}}</span>
                        </div>

                        <div class="fa fa-reply comment-reply" @click="onComment(item)"/>
                    </div>
                    <div class="comment-content"> 
                        {{item.content}}
                    </div>
                </div>
                <comment-list class="reply-list" :list="item.children" v-bind="$attrs" v-on="$listeners"/>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    name: 'comment-list',
    props: {
        list: {
            type: Array,
        },
    },
    methods: {
        onComment(item) {
            console.log('comment', item);
            this.$emit('onReply', item);
        },
    },
};
</script>

<style lang="less" scoped>
.comment-list {
    list-style-type:none;
    padding-bottom: 1rem;
    .comment-block {
        border-bottom: 1px solid #ddd;
    }
    .comment-head {
        display: flex;
        justify-content: space-between;
        margin: 1rem 0 0 0;
        padding: 1rem;
        .comment-name {
            font-family: eafont,Hiragino Sans GB,Hiragino Sans GB W3,Microsoft YaHei,WenQuanYi Micro Hei,sans-serif;
            font-size: 1rem;
        }
        .comment-time {
            color: #aaa;
            font-size: 0.8rem;
            padding: 1rem;
        }
        .comment-reply {
            font-size: 0.8rem;
            color: #aaa;
            &:hover {
                cursor: pointer;
                color: #333;
            }
        }
    }
    .comment-content {
        padding: 1rem 1rem 2rem 1rem;
    }
    .reply-list {
        margin-left: 4rem;
    }
}
</style>
```

1. 递归组件必须定义name属性，否则无法做到自己调用自己
2. 递归需要一个停止条件，这里是item.children的length
3. 由于递归会造成组件结构很深，通过props 和 emit 的方法实现跨组件通信的方法会很麻烦。另外也可以使用vuex来实现， 但是仅仅传递数据，不做vuex处理，会比较大材小用。 这里使用**$listeners**的方法

> **$props：** 当前组件接收到的 props 对象，Vue 实例代理了对其 props 对象属性的访问。
>
> **$attrs：** 包含了父作用域中不被 prop 所识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件。通常配合 interitAttrs 选项一起使用。 **通俗的理解为：子辈可以通过$attrs将未在自己组件内注册的祖辈传递下来的参数接收来，并传递孙辈。**
>
> **inheritAttrs：**  默认情况下父作用域的不被认作 props 的特性绑定 (attribute bindings) 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。通过设置 inheritAttrs 为 false，这些默认行为将会被去掉。而通过实例属性 $attrs 可以让这些特性生效，且可以通过 v-bind 显性的绑定到非根元素上。
>
> **$listeners：** 包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件



### 使用组件

```vue
<comment-list :list="commentList" @onReply="onReply"/>

// mock数据：
<script>
export default {
  data() {
    return {
      commentList: [{
          id: 1,
          name: 'Test1',
          createdAt: '2020-7-21 09:21:11',
          content: '测试管理数据中心',
          children: [],
      }, {
          id: 2,
          name: 'Test2',
          createdAt: '2020-7-22 09:21:11',
          content: '测试管理数据中心2-为实现呃呃呃',
          children: [{
              id: 21,
              name: '子模块封装',
              createdAt: '2020-7-23 09:21:11',
              content: '子模块-测试管理数据中心2-为实现呃呃呃',
          }, {
              id: 22,
              name: '子模块封装',
              createdAt: '2020-7-23 09:21:11',
              content: '子模块-测试管理数据中心2-为实现呃呃呃',
          }],
      }],
    };
  },
};
</script>
```

