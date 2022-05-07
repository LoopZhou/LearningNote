# WebGPU学习笔记

### 简介

主要用于绘制3D图形，下一代图形标准，有更好的性能，目前依旧在实验阶段，api调整频繁。目前api参考DirectX 12、Vulkan、Metal API， 着色器语言采用新设计的 [WGSL](https://www.w3.org/TR/WGSL/#intro)。

### 基本绘制流程

#### 浏览器配置

下载Google Chrome Canary浏览器

输入chrome://flags

打开webgpu为enabled

#### Html: 定义canvas

```html
<canvas id="webgpu" width="200" height="100" />
```

#### js部分：获取canvas，对canvas进行操作和绘制

判断浏览器是否支持webgpu， 不支持canvas绘图提示

```typescript
const checkWebGPU = () => {
	if (navigator.gpu) {
		initWebGPU();
		return;
	}

	const canvas = document.getElementById('webgpu');
	const ctx = canvas.getContext('2d');
	ctx.font = '20px Georgia';
	ctx.fillText('Not Support', 10, 50);
};
```

#### webgpu部分

**初始化WebGPU->配置GPU管线->录制Command队列**

初始化WebGPU： 获取GPU的JS逻辑示例，获取canvas部分， 获取适配器和显卡设备， 配置canvas

Adapter->Device->configure context

```js
const adapter = await navigator.gpu.requestAdapter();
const device = await adapter.requestDevice();
const canvas = document.getElementById('webgpu');
const context = canvas.getContext('webgpu');

context.configure({
	// 必选参数
	device,
	// 必选参数
	format: 'bgra8unorm',
	// 可选参数，Chrome 102 开始默认为 'opaque'，即不透明选项
	compositingAlphaMode: 'opaque',
});
```

配置GPU管线： 配置流水线， 编译Shader程序， 申明GPU变量， 写入显寸， 资源分组

Pipeline->GPUBuffers->BindGroups

```js
const shader = webgpuShader();
const pipeline = await device.createRenderPipelineAsync({
  // 顶点着色器
  vertex: {
    module: device.createShaderModule({
      code: shader.vertex,
    }),
    entryPoint: 'main',
  },
  // 片段着色器
  fragment: {
    module: device.createShaderModule({
      code: shader.fragment,
    }),
    entryPoint: 'main',
    targets: [
      {
        format: 'bgra8unorm',
      },
    ],
  },
  // 绘制方式
  // primitiveTopology: 'triangle-list',
  primitive: {
		topology: 'triangle-list',
	},
});
```

着色器

```js
export const webgpuShader = () => {
  const vertex = `
    @stage(vertex)
    fn main(@builtin(vertex_index) VertexIndex : u32) -> @builtin(position) vec4<f32> {
      var pos = array<vec2<f32>, 3>(
        vec2<f32>(0.0, 1.0),
        vec2<f32>(-1.0, -1.0),
        vec2<f32>(1.0, -1.0)
      );
      return vec4<f32>(pos[VertexIndex], 0.0, 1.0);
    }
  `;

  const fragment = `
    @stage(fragment)
    fn main() -> @location(0) vec4<f32> {
      return vec4<f32>(1.0, 0.0, 0.0, 1.0);
    }
  `;
  return { vertex, fragment };
};
```

配置画布

```js
// 画布
const view = context.getCurrentTexture().createView();
// 类似于图层
const renderPassDescriptor = {
  colorAttachments: [
    {
      view: view,
      clearValue: { r: 1.0, g: 1.0, b: 1.0, a: 1.0 },
      // 可选: clear, load
      loadOp: 'clear',
      // 兼容 Chrome 101 之前的版本
      loadValue: { r: 1.0, g: 1.0, b: 1.0, a: 1.0 },
      // 可选: store, discard
      storeOp: 'store',
    },
  ],
};
```

录制Command队列:  绘制

commandEncoder->beginRenderPass->device.queue.submit

```js
// 绘制 command 队列
const commandEncoder = device.createCommandEncoder();

const passEncoder = commandEncoder.beginRenderPass(renderPassDescriptor);
// 设置渲染管线配置
passEncoder.setPipeline(pipeline);
passEncoder.draw(3, 1, 0, 0);
// 结束通道绘制，endPass 在 Chrome 101 之后的版本被弃用
passEncoder.end ? passEncoder.end() : passEncoder.endPass();

// 结束绘制，得到 commandBuffer
const commandBuffer = commandEncoder.finish();
device.queue.submit([commandBuffer]);
```

### 项目配置

#### typescript定义

typescript中可以安装@webgpu/types可以方便查看api接口定义等

```sh
npm install --save-dev @webgpu/types
```

typeconfig.json中定义

```json
{
  "compilerOptions": {
		...
    "types":["@webgpu/types"],
  },
}
```

#### 直接引入着色器

typeconfig.json中定义

```json
{
  "compilerOptions": {
		...
    "types":["vite/client"],
  },
}
```

import时添加`?raw`表示引入的是字符串

```js
import vertex from './shaders/triangle.vert.wgsl?raw'
```



### API学习

#### Adapter

```js
// 可以配置独显/核显, 但不一定保证会使用，会根据浏览器等情况而定
const adapter = await navigator.gpu.requestAdapter({
	powerPreference: 'high-performance'/ 'low-performance', 
});
```

#### Device

```js
// 打印adapter和device。 adapter显示了显卡有的特性， device显示了当前分配的特性，生成device时可以根据adapter灵活分配device
console.log('adapter', adapter);
console.log('device', device);
```

配置device

```js
const device = await adapter.requestDevice({
	requireFeatures: ['texture-compression-bc'],
	requireLimits: {
		maxStorageBufferBindingSize: adapter.limits.maxStorageBufferBindingSize,
	}
})
```

#### GPUBuffers资源绑定

数据traingle.ts

```ts
const vertex = new Float32Array([
  0.0, 1.0, 0.0, -1.0, -1.0, 0.0, 1.0, -1.0, 0.0,
]);
const vertexCount = 3;

export { vertex, vertexCount };
```

把数据传递给GPU

```js
// 创建 vertex buffer
const vertexBuffer = device.createBuffer({
	label: 'GPUBuffer store vertex',
	size: triangle.vertex.byteLength,
	usage: GPUBufferUsage.VERTEX | GPUBufferUsage.COPY_DST,
	//mappedAtCreation: true
});

// 赋值
device.queue.writeBuffer(vertexBuffer, 0, triangle.vertex);

// create color buffer
const colorBuffer = device.createBuffer({
	label: 'GPUBuffer store rgba color',
	size: 4 * 4, // 4 * float32
	usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST,
});

device.queue.writeBuffer(colorBuffer, 0, new Float32Array([1, 0.5, 0, 1]));

// create a uniform group for color
const uniformGroup = device.createBindGroup({
	label: 'Uniform Group with colorBuffer',
	layout: pipeline.getBindGroupLayout(0),  //group0
	entries: [
		{
			binding: 0,   //bind0, 最多支持8个资源
			resource: {
				buffer: colorBuffer,
			},
		},
	],
});
```

pipeline改造增加顶点着色器buffer数据定义

```js
const pipeline = await device.createRenderPipelineAsync({
  vertex: {
    module: device.createShaderModule({
      code: shader.vertex,
    }),
    entryPoint: 'main',
    buffers: [
      {
        arrayStride: 3 * 4, // 3 float32,
        attributes: [
          {
            // position xyz
            shaderLocation: 0, //location(0)
            offset: 0,
            format: 'float32x3',
          },
        ],
      },
    ],
  },
  fragment: {
    module: device.createShaderModule({
      code: shader.fragment,
    }),
    entryPoint: 'main',
    targets: [
      {
        format: 'bgra8unorm',
      },
    ],
  },
  // primitiveTopology: 'triangle-list',
  primitive: {
    topology: 'triangle-list',
  },
});
```

shader

```js
export const webgpuShader = () => {
  const vertex = `
    @stage(vertex)
    fn main(@location(0) position : vec3<f32>) -> @builtin(position) vec4<f32> {
      return vec4<f32>(position, 1.0);
    }
  `;

  const fragment = `
    @group(0) @binding(0) var<uniform> color : vec4<f32>;

    @stage(fragment)
    fn main() -> @location(0) vec4<f32> {
      return color;
    }
  `;
  return { vertex, fragment };
};

```

