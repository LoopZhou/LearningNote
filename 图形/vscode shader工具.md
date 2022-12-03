# vscode shader 工具

## vscode插件glsl-canvas使用

参考：
[glsl-canvas](https://marketplace.visualstudio.com/items?itemName=circledev.glsl-canvas)
[github](https://github.com/actarian/vscode-glsl-canvas)

1. 安装vscode插件glsl-canvas
2. 创建测试文件hello.glsl

```js
void main () {
  // 颜色
  gl_FragColor = vec4(1.0, 0.5, 0.0, 1.0);
}
```

3. 运行验证
mac运行通过 ⌘ ⇧ P / windows运行通过ctrl ⇧ P
输入 >show glslCanvas
调整代码中的颜色，可以实时变化


4. 提供的变量
以 u_xxx开头的变量名是由插件提供的，由gl_XXX 开头的变量则是GLSL提供的

u_resolution 表示的是当前预览窗口的分辨率。
gl_FragCoord 表示了当前像素在预览窗口中的坐标。
通常我们将其归一化表示： 通过 gl_FragCoord.xy / u_resolution 可以将坐标归一化到0~1之间。

```js
#ifdef GL_ES
// 声明浮点数默认精度的声明
precision mediump float;
#endif

uniform vec2 u_resolution;
void main() {
    vec2 uv = gl_FragCoord.xy/u_resolution;
    gl_FragColor = vec4(uv,0.0,1.0);
}
```


