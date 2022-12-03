## Lodash常用方法总结

### 简介

Lodash是js开发者常用的工具库，提供了非常多的工具方法，以方便程序员快速实现某些算法。这里，主要总结一下自己平时经常使用的Lodash方法。

由于ES6的添加，许多lodash的方法可以用ES6进行替换，这里就不再介绍

需要查看完整实例可参考官网文档：[文档](https://www.lodashjs.com/docs/latest)



### 工具

1. _.cloneDeep: 深拷贝

   ```js
   const objects = [{ 'a': 1 }, { 'b': 2 }];
   const deep = _.cloneDeep(objects);
   
   // 深拷贝后数组中的对象已经不是同一个
   console.log(deep[0] === objects[0]);
   // => false
   
   // _.clone是浅拷贝 
   ```

2. _.isArray: 判断是否是数组

   ```js
   _.isArray([1, 2, 3]);
   // => true
   
   _.isArray('abc');
   // => false
   ```

3. _.isEmpty: 判断是否为空

   ```js
   _.isEmpty(null);
   // => true
   
   _.isEmpty(undefined);
   // => true
   
   _.isEmpty([]);
   // => true
   
   _.isEmpty({});
   // => true
   
   _.isEmpty("");
   // => true
   ```

4. _.isEqual: 判断是否相同, 执行深比较来确定两者的值是否相等

   ```js
   var object = { 'a': 1 };
   var other = { 'a': 1 };
    
   _.isEqual(object, other);
   // => true
    
   object === other;
   // => false
   ```

   

### 对象

1. _.get: 获取对象数据。由于返回值格式可能不对，通过点运算获取时会报错，用get避免报错并给予默认值

   ```js
   // 根据 object对象的path路径获取值
   var object = { 'a': [{ 'b': { 'c': 3 } }] };
   
   _.get(object, 'a[0].b.c');
   // => 3
   
   _.get(object, 'a.b.c', 'default');
   // => 'default'
   ```

2. _.merge: 按key值合并

   ```js
   var object = {
     'a': [{ 'b': 2 }, { 'd': 4 }]
   };
    
   var other = {
     'a': [{ 'c': 3 }, { 'e': 5 }]
   };
    
   _.merge(object, other);
   // => { 'a': [{ 'b': 2, 'c': 3 }, { 'd': 4, 'e': 5 }] }
   ```

3. _.pick: 创建一个object中选中属性的对象

   ```js
   var object = { 'a': 1, 'b': '2', 'c': 3 };
    
   _.pick(object, ['a', 'c']);
   // => { 'a': 1, 'c': 3 }
   ```

4. __.omit: 反向_.pick, 创建一个object中未选中的属性的对象

   ```js
   var object = { 'a': 1, 'b': '2', 'c': 3 };
    
   _.omit(object, ['a', 'c']);
   // => { 'b': '2' }
   ```

5. _.pickBy: 创建一个从 `object` 中经 callback 函数判断为真值的属性为对象

   ```js
   var object = { 'a': 1, 'b': '2', 'c': 3 };
   
   _.pickBy(object, (value) => {return _.isNumber(value)};
   // => { 'a': 1, 'c': 3 }
   ```

6. _.omitBy: 反向版 `pickBy` 

   ```js
   var object = { 'a': 1, 'b': '2', 'c': 3 };
   
   _.omitBy(object, value => {return _.isNumber(value)};
   // => { 'b': '2' }
   ```

   

### 数组

1. _.uniq(array): 数组去重，返回一个新数组

   ```js
   _.uniq([2, 1, 2]);
   // => [2, 1]
   ```

2. _.uniqBy(array, callback): 根据callback条件数组去重 

   ```js
   // 数组每一个元素向下取整后比较去重
   _.uniqBy([2.1, 1.2, 2.3], Math.floor);
   // => [2.1, 1.2]
   ```

### 字符串

1. _.unescape(string): 转换`string`字符串中的 HTML 实体 `&`, `<`, `>`, `"`, `'`, 和 ``` 为对应的字符

   ```js
   _.unescape('fred, barney, &amp; pebbles');
   // => 'fred, barney, & pebbles'
   ```

2. _.escape(string): 转义`string`中的 "&", "<", ">", '"', "'", 和 "`" 字符为HTML实体字符

   ```js
   _.escape('fred, barney, & pebbles');
   // => 'fred, barney, &amp; pebbles'
   ```

### 函数

1. _.throttle(callback, time): 截流函数， time内只能执行一次callback函数。

   ```js
   // 避免在滚动时过分的更新定位
   jQuery(window).on('scroll', _.throttle(updatePosition, 100));
   ```

2. _.debounce(callback, time): 防抖函数， time内执行最后一次callback函数

   ```js
   // 避免窗口在变动时出现昂贵的计算开销。
   jQuery(window).on('resize', _.debounce(calculateLayout, 150));
   ```

   

