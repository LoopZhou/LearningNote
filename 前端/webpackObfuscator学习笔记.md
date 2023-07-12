# webpack-obfuscator学习笔记


## 背景

某些保密场景的业务需要隐藏业务逻辑，前端代码通过增加随机废代码段、字符编码转义等方法可以使构建代码完全混淆，达到无法恢复源码，无法阅读的目的。

## 代码混淆方案
目前前端主要可以可通过JavaScript Obfuscator来实现，项目中使用javascipt-obfuscator的webpack插件webpack-obfuscator来进行代码混淆。

### webpack-obfuscator配置项

```js
{
  // 压缩代码到一行
  compact: true,
  // 是否开启平展控制流。
  // 此选项对性能的影响最大为运行速度降低1.5倍。 
  // 使用controlFlowFlatteningThreshold设置将受控制流展平影响的节点的百分比。
  // 平展控制流是源代码的结构转换，让代码变得难以阅读。
  controlFlowFlattening: false,
  // 每个节点采用平展控制流的概率，该参数主要用来优化大代码的性能和代码大小。大量的平展控制流会是代码运行效率变差而且代码增大。设置为0表示关闭平展控制流, 最小：0   最大：1。
  controlFlowFlatteningThreshold: 0.75,
  // 使用此选项，将随机废代码块添加到混淆代码中。
  // 该选项显著增加混淆代码的大小（最大200%），如果对混淆代码大小不敏感的时候启用该选项。
  // 开启该选项将强制启用stringArray选项
  deadCodeInjection: false,
  // 节点加入废代码的概率，值越大，混淆代码大小越大。 最小：0   最大：1
  deadCodeInjectionThreshold: 0.4,
  // 如果您打开开发人员工具，会冻结您的浏览器，不允许调试。使用该选项，几乎不可能使用开发人员工具的调试器功能
  debugProtection: false,
  // 如果设置为true，则会在“控制台”选项卡上使用一个间隔来强制调试模式，这使得使用开发人员工具的其他功能更加困难。 启用了debugProtection，才可以使用
  debugProtectionInterval: false,
  // 通过将它们替换为空函数，禁用console.log，console.info，console.error，console.warn，console.debug，console.exception和console.trace的使用
  disableConsoleOutput: false,
  // 允许仅在特定域名或子域名上运行混淆的源代码。 这使他人很难复制和粘贴您的代码在其他地方运行。
  // 多个域名和子域名可以将代码锁定到多个域名或子域名。 
  // 例如，要对其进行锁定，以使代码仅在www.example.com上运行，请添加www.example.com。 要使其在包括任何子域名（example.com，sub.example.com）的根域上工作，请使用.example.com
  domainLock: [],
  forceTransformStrings: [],
  // 设置变量名称生成器，可以使用下面的值：
  // dictionary: 变量名称来自identifiersDictionary
  // hexadecimal:变量名称像 _0xabc123
  // mangled: 短变量名称如： a, b, c
  // mangled-shuffled: 和mangled一样，但是变量是乱序的
  identifierNamesGenerator: 'hexadecimal',
  // identifierNamesGenerator设置为dictionary下使用，字典内的元素可以通过变换字符大小写产生不同变量名字
  identifiersDictionary: [],
  // 为所有全局标识符设置前缀。
  // 当您要混淆多个文件时，请使用此选项。 此选项有助于避免这些文件的全局标识符之间的冲突。 每个文件的前缀都应该不同。
  identifiersPrefix: '',
  // 忽略混淆require和impoort，某些情况下需要静态引入，不能替换为变量
  ignoreRequireImports: false,
  // 允许使用源代码设置输入文件的名称。此名称将在内部用于源映射生成。
  inputFileName: '',
  // 启用将信息输出到控制台
  log: false,
  // 允许将数字转换为表达式
  numbersToExpressions: false,
  // ​​​​​​​选项默认值名称，所有附加选项将与该名称对应的预设选项合并。
  // 可用参数:
  //   default
  //   low-obfuscation
  //   medium-obfuscation
  //   high-obfuscation
  optionsPreset: 'default',
  // 指定是否混淆全局变量和函数名称
  renameGlobals: false,
  // 启用属性名称的重命名。 所有内置的DOM属性和核心JavaScript类中的属性都将被忽略。
  // 要在此选项的安全和不安全模式之间切换，请使用renamePropertiesMode选项。
  // 要设置重命名属性名称的格式，请使用identifierNamesGenerator选项。
  // 若要控制将重命名的属性，请使用reservedNames选项。
  renameProperties: false,
  // 指定renameProperties选项模式：
  // safe - 2.11.0版本之后的默认值。 尝试以更安全的方式重命名属性，以防止运行时错误。 使用此模式时，某些属性将从重命名中排除。
  // unsafe - 2.11.0发行之前的默认值。 无限制地以不安全的方式重命名属性。
  renamePropertiesMode: 'safe',
  // 使用正则过滤需要混淆的变量。
  reservedNames: [],
  // 使用正则过滤需要混淆的字符串
  reservedStrings: [],
  // 将stringArray移动固定和随机（在代码混淆处生成）位置。这使得将删除的字符串的顺序与其原始位置进行匹配变得更加困难。
  rotateStringArray: true,
  // 此选项为随机生成器设置种子。这对于创建可重复的结果很有用。
  // 如果种子为0-随机生成器将在没有种子的情况下工作。
  seed: 0,
  // 此选项使输出代码可阻止格式设置和变量重命名。如果尝试在混淆后的代码上使用JavaScript美化器，则该代码将无法正常工作。
  selfDefending: false,
  // 随机打乱stringArray数组元素，stringArray设置为true时启用
  shuffleStringArray: true,
  // 简化多余的代码
  simplify: true,
  // 启用混淆代码的Sourcemap生成
  // Sourcemap可以帮助您调试混淆后的JavaScript源代码。 如果要在生产中进行调试，可以将单独的Sourcemap文件上传到某个地方，然后将浏览器指向该位置。
  sourceMap: false,
  // 当sourceMapMode设置为'separate'时，将URL设置为Sourcemap导入URL
  sourceMapBaseUrl: '',
  sourceMapFileName: '',
  // 指定源映射生成模式：
  // inline   - 在每个.js文件的末尾添加源映射；
  // separate - 生成与Sourcemap对应的.map文件。 如果您通过CLI运行混淆器-使用混淆的代码
  //＃sourceMappingUrl = file.js.map将指向源映射文件的链接添加到文件的末尾。
  sourceMapMode: 'separate',
  // 将字符串拆分字符串块。
  splitStrings: false,
  // 设置字符串拆分区块长度
  splitStringsChunkLength: 10,
  // 删除字符串并将其放置在特殊的数组中。 例如，var m =“ Hello World”中的字符串“ Hello World”; 将被替换为var m = _0x12c456 [0x1];
  stringArray: true,
  stringArrayIndexesType: [
    'hexadecimal-number'
  ],
  // 这个设置会让你的代码慢下来！
  // 使用base64或rc4对stringArray的所有字符串文字进行编码，并插入一个特殊代码，用于在运行时将其解码回来。
  // 每个stringArray值都将通过从传递列表中随机选取的编码进行编码。这使得使用多种编码成为可能。
  // 可用的编码如下：
  // 'none'（boolean）：不编码stringArray值;
  // 'base64'（string）：使用base64对stringArray值进行编码;
  // “rc4”（字符串）：使用rc4对stringArray值进行编码。大约比base64慢30-50%，但更难获得初始值。建议在使用rc4编码时禁用UnicodeScapeSequence选项，以防止出现非常大的混淆代码。
  stringArrayEncoding: [],
  stringArrayIndexShift: true,
  stringArrayWrappersCount: 1,
  stringArrayWrappersChainedCalls: true,
  stringArrayWrappersParametersMaxCount: 2,
  stringArrayWrappersType: 'variable',
  stringArrayThreshold: 0.75,
  target: 'browser',
  // 启用Object键值的转换
  transformObjectKeys: false,
  unicodeEscapeSequence: false
}
```

### 混淆思路

代码混淆越复杂，体积越大，执行越慢，业务需要根据实际场景进行混淆和速度的权衡来进行配置。

1. controlFlowFlattening对执行速度影响较大，业务可根据情况配置controlFlowFlatteningThreshold因子来减少混淆力度

2. deadCodeInjection会增大体积，配置deadCodeInjectionThreshold来控制力度

3. 打包代码时，建议把公共库剔除，不建议混淆公共库

4. 去掉node_module的代码混淆，可直接去掉'/js/chunk-..js'

```js
new WebpackObfuscator({
  compact: true,
  controlFlowFlattening: true,
  ...
},
['**/js/chunk-**.**.js', '**/js/runtime.**.js', '**/js/app.**.js', '**/js/vendor.**.js'],
),
```
如果业务进行了分包split, 根据实际情况配置，这里配置的应该是打包后生成的文件。

5. 公共库建议进行split或dll抽取


## 参考

[官网](https://obfuscator.io/)
[javascript-obfuscator](https://www.npmjs.com/package/javascript-obfuscator)
[webpack-obfuscator](https://www.npmjs.com/package/webpack-obfuscator)
[推荐配置](https://juejin.cn/post/7159431931975696397)

