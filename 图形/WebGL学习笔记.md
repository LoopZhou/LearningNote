# WebGL学习笔记

### 简介

WebGL基于OpenGL ES 2.0。早先的OpenGL用的都是固定的渲染管线，固定管线都是glBegin，glEnd，glVertex2d等，精简版的OpenGL ES一上来就抛弃了之前固定渲染管线的东西，使用了可编程的渲染管线，通过着色器实现。

### 基本绘制过程

#### Html: 定义canvas

```html
<canvas id="webgl-canvas" :width="width" :height="height"/>
```

#### js部分：获取canvas，对canvas进行操作和绘制

```typescript
canvas = document.getElementById('webgl-canvas') as HTMLCanvasElement;
engine = new Engine(canvas);
engine.start();
```

#### webgl部分：

1. 获取canvas的webgl上下文进行绘制操作

```typescript
this._gl = (this._canvas.getContext('webgl2') as WebGLRenderingContext;
```

2. 配置gl相关状态

```js
// 设置清理的颜色COLOR_BUFFER_BIT
this._gl.clearColor(0, 0, 0, 1);
// this._gl.clear(this._gl.COLOR_BUFFER_BIT);  //绘制颜色（绘制过程中进行）

// 设置视口
this._gl.viewport(0, 0, this._canvas.width, this._canvas.height);
```

3. 加载纹理资源

```typescript
// 创建纹理对象以存储纹理图像
this._handle = this._gl.createTexture() as WebGLTexture;

// 绑定纹理对纹理进行操作， 第一次指定类型
this._gl.bindTexture(this._gl.TEXTURE_2D, this._handle);

// 将 source 指定的图像分配给绑定到目标上的纹理对象
this._gl.texImage2D(
  this._gl.TEXTURE_2D,
  LEVEL,
  this._gl.RGBA,
  this._gl.RGBA,
  this._gl.UNSIGNED_BYTE,
  asset.Data, //图片数据HTMLImageElement：const image: HTMLImageElement = new Image();
);

// 图片参数配置
// generate mipmap
this._gl.generateMipmap(this._gl.TEXTURE_2D);
// Do not generate a mip map and clamp wrapping to edge.
this._gl.texParameteri(this._gl.TEXTURE_2D, this._gl.TEXTURE_WRAP_S,this._gl.CLAMP_TO_EDGE);
this._gl.texParameteri(this._gl.TEXTURE_2D, this._gl.TEXTURE_WRAP_T,this._gl.CLAMP_TO_EDGE);
// Set text ure filte r ing based on configuration.
this._gl.texParameteri(this._gl.TEXTURE_2D, this._gl.TEXTURE_MIN_FILTER, this._gl.NEAREST);
this._gl.texParameteri(this._gl.TEXTURE_2D, this._gl.TEXTURE_MAG_FILTER, this._gl.NEAREST);


// 使用时：激活纹理单元（多重纹理）-绑定纹理
/*
存在一系列的texture unit，通过 glActiveTexture激活当前的texture unit， 默认是0
当前的texture unit中存在多个texture target，例如GL_TEXTURE_2D, GL_TEXTURE_CUBEMAP，
通过glBindTexture绑定。
*/
this._gl.activeTexture(this._gl.TEXTURE0 + textureUnit);
this._gl.bindTexture(this._gl.TEXTURE_2D, this._handle);

……
// 绘制过程
……

// 取消绑定
gl.bindTexture( gl.TEXTURE_2D, undefined );

// 不使用时，删除纹理
if (this._handle) {
  this._gl.deleteTexture(this._handle);
}
```

4. 加载面片顶点（缓冲区）

使用缓存的五个步骤

createBuffer，bindBuffer，bufferData，vertexAttribPointer，enableVertexAttribArray,

```typescript
// 创建缓冲区
this._buffer = gl.createBuffer() as WebGLBuffer;

// 允许使用buffer表示的缓冲区对象并将其绑定到target表示的目标上
// targetBufferType
//        gl.ARRAY_BUFFER 表示缓冲区对象中包含顶点数据
//        gl.ELEMENT_ARRAY_BUFFER 表示缓冲去对象中包含了顶点的索引值
this._gl.bindBuffer(this._targetBufferType, this._buffer);

// 开辟存储空间，向绑定在target上的缓冲区对象写入数据data
// usage 优化效率 可以是以下值：
//        STATIC_DRAW 只会向缓冲区写入一次数据 需要绘制很多次
//        STREAM_DRAW 只会向缓冲区写入一次数据 需要绘制若干次
//        DYNAMIC_DRAW 会向缓冲区对象中多次写入数据 并绘制很多次
let bufferData: ArrayBuffer = new ArrayBuffer(0);
this._gl.bufferData(this._targetBufferType, bufferData, this._gl.STATIC_DRAW);

// 将绑定到ARRAY_BUFFER的缓冲区对象分配给index指定的attribute变量
// @param index 指向attribute变量
// @param size 指定缓冲区中每个顶点分量的个数
// @param type 数据格式
// @param normalized 是否将浮点型数据归一化到[0, 1]或者[-1, 1]区间
// @param stride 指定相邻两个顶点之间的字节数 默认是0
// @param offset 指定缓冲区对象中的偏移量 单位字节 可以利用这个偏移量赋值多个attribute
this._gl.vertexAttribPointer(
  it.location,
  it.size,
  this._dataType,
  normalized,
  this._stride,
  it.offset * this._typeSize,
);

// 开启index对应的attribute对象
this._gl.enableVertexAttribArray(it.location);

……
// 绘制过程
……

// 不再绑定： 关闭index对应的attribute对象
this._gl.disableVertexAttribArray(it.location);
this._gl.bindBuffer(this._targetBufferType, null);

// 删除缓冲区
this._gl.deleteBuffer(this._buffer);
```

5. 着色器程序

 加载shader

```typescript
private loadShader(source: string, shaderType: number): WebGLShader {
  // 创建由type指定的着色器对象
  const shader: WebGLShader = this._gl.createShader(shaderType) as WebGLShader;

  // 将 source 指定的字符串形式的代码传入shader指定的着色器
  this._gl.shaderSource(shader, source);
  // 编译 shader 指定的着色器中的源代码
  this._gl.compileShader(shader);
  
  // 指定shader 的信息日志
  const error = (this._gl.getShaderInfoLog(shader) || '').trim();
  if (error !== '') {
    throw new Error("Error compiling shader '" + this._name + "': " + error);
  }

  return shader;
}

// 删除shader
deleteShader(WebGLShader? shader);
```

使用着色器程序

```typescript
// 创建着色器程序对象
this._program = this._gl.createProgram() as WebGLProgram;

// 加载shader
const vertexShader = this.loadShader(vertexSource, this._gl.VERTEX_SHADER);
const fragmentShader = this.loadShader(fragmentSource, this._gl.FRAGMENT_SHADER);

// 将 shader 指定的着色器对象分配给 program 指定的程序对象
this._gl.attachShader(this._program, vertexShader);
this._gl.attachShader(this._program, fragmentShader);

// 连接 program 指定的程序对象中的着色器
// 目的：
// 1. 保证顶点着色器 和 片元着色器的varying变量同名同类型，且一一对应
// 2. 保证顶点着色器对每个varying变量赋了值
// 3. 保证顶点着色器 和 片元着色器中的同名 uniform 变量也是同类型的 无需一一对应
// 4. 保证着色器中的attribute、uniform、varying变量的个数没有超过着色器上限
this._gl.linkProgram(this._program);

// 可以通过 此函数获取 program 指定的程序对象的信息日志
const error = (this._gl.getProgramInfoLog(this._program) || '').trim();
if (error !== '') {
   throw new Error("Error linking shader '" + this._name + "': " + error);
}

// 告知 WEBGL 系统绘制时使用的 program 对象
this._gl.useProgram(this._program);
      
// ……
// 绘制
// ……
```

着色器程序

顶点着色器

```javascript
  private getVertexSource(): string {
    return `
attribute vec3 a_position;
attribute vec2 a_texCoord;

uniform mat4 u_projection;
uniform mat4 u_model;

varying vec2 v_texCoord;

void main() {
  gl_Position = u_projection * u_model * vec4(a_position, 1.0);
  v_texCoord = a_texCoord;
}`;
  }
```

片段着色器

```javascript
  private getFragmentSource(): string {
    return `
precision mediump float;

uniform vec4 u_tint;
uniform sampler2D u_diffuse;

varying vec2 v_texCoord;

void main() {
  gl_FragColor = u_tint * texture2D(u_diffuse, v_texCoord);
}
`;
  }
```

6. 绘制过程:

**帧动画**

```
requestAnimationFrame(this.loop.bind(this));
```

**给着色器赋值:** 

参数地址

```typescript
/**
* 获取着色器中参数的地址
**/
// 获取attribute的参数
const attributeCount = this._gl.getProgramParameter(this._program,this._gl.ACTIVE_ATTRIBUTES);
for (let i = 0; i < attributeCount; ++i) {
  // 返回激活状态的attribute变量信息
  const info: WebGLActiveInfo = this._gl.getActiveAttrib(this._program, i) as WebGLActiveInfo;
  if (!info) {
    break;
  }

  // 获取由 name 参数指定的 attribute 变量存储地址
  this._attributes[info.name] = this._gl.getAttribLocation(this._program, info.name);
}

// 获取画uniform的参数
const uniformCount = this._gl.getProgramParameter(this._program, this._gl.ACTIVE_UNIFORMS);
for (let i = 0; i < uniformCount; ++i) {
  // 返回激活状态的uniform 变量信息。
  const info: WebGLActiveInfo = this._gl.getActiveUniform(this._program, i) as WebGLActiveInfo;
  if (!info) {
    break;
  }

  //获取指定名称的 uniform 变量存储位置
  this._uniforms[info.name] = this._gl.getUniformLocation(
    this._program,
    info.name,
  ) as WebGLUniformLocation;
}
```

利用地址给着色器传unform的值，常量

```typescript
/**
* 利用地址给着色器传值
**/
this._gl.uniformMatrix4fv(this._uniforms['u_projection'], false, new Float32Array(this._projection.data));
this._gl.uniform4fv(this._uniforms['u_tint'], new Float32Array(this._tint);
this._gl.uniformMatrix4fv(
  this._uniforms['u_model'],
  false,
  new Float32Array(Matrix4x4.translation(this.position).data),
)
```

attribute的值，变量，一般是顶点数据buffer，纹理uv等。参考上在buffer中vertexAttribPointer， enableVertexAttribArray（0), 

```js
// buffer-triangle
const pointPos = [
  -1, -1,
  1, -1,
  0, 1
];

const buffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(pointPos), gl.STATIC_DRAW);
const a_position = gl.getAttribLocation(program, "a_position");
gl.vertexAttribPointer(
  a_position,
  2,
  gl.FLOAT,
  false,
  Float32Array.BYTES_PER_ELEMENT * 2,
  0
);

gl.enableVertexAttribArray(a_position);
```

**绘制**

```typescript
if (this._targetBufferType === this._gl.ARRAY_BUFFER) {
  // 执行绘制， 按照mode参数指定的方式绘制图形
	// @param model 绘制模式。
	// @param first 指定从哪个定点开始绘制
	// @param count 指定绘制需要用到多少个顶点
  this._gl.drawArrays(this._mode, 0, this._data.length / this._elementSize);
} else if (this._targetBufferType === this._gl.ELEMENT_ARRAY_BUFFER) {
  // 执行绘制，按照mode参数制定的方式，根据绑定到 ELEMENT_ARRAY_BUFFER 的缓冲区中的顶点索引绘制图形
  // @param model 绘制模式。
  // @param count 指定绘制顶点的个数
  // @param type 指定索引值数据类型。包括：UNSIGNED_BYTE、UNSIGNED_SHORT、UNSIGNED_INT
  // @param offset 指定索引数组中绘制的偏移位置，以字节为单位  
  this._gl.drawElements(this._mode, this._data.length, this._dataType, 0);
}
```

mode类型

```js
// 为 gl.drawArrays、gl.drawElements 第一个参数
const GLenum POINTS                         = 0x0000;
const GLenum LINES                          = 0x0001;
const GLenum LINE_LOOP                      = 0x0002;
const GLenum LINE_STRIP                     = 0x0003;
const GLenum TRIANGLES                      = 0x0004;
const GLenum TRIANGLE_STRIP                 = 0x0005;
const GLenum TRIANGLE_FAN                   = 0x0006;
```
