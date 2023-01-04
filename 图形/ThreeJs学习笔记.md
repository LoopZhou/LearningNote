# Three.js学习笔记

## 基本API使用示例

### scene

```js
this.scene = new THREE.Scene();

// 场景中增加面片，辅助线，灯光等等
this.axesHelper = new THREE.AxesHelper(3);
this.scene.add(this.axesHelper);
```

### camera

透视相机
```js
this.perspectiveCamera = new THREE.PerspectiveCamera(
  75,   // fov 视锥体垂直视野角度
  window.innerWidth / window.innerHeight, // aspect 长宽比
  0.1,  // near 
  1000  // far
);
```

正交相机
```js
this.orthographicCamera = new THREE.OrthographicCamera(
  (-this.sizes.aspect * this.sizes.frustrum) / 2, // 左侧面
  (this.sizes.aspect * this.sizes.frustrum) / 2,  // 右侧面
  this.sizes.frustrum / 2,  // 上
  -this.sizes.frustrum / 2, //下
  -50,  // 近
  50    // 远
);
```

位置配置

```js
this.camera.position.z = 5;
this.camera.position.set(0, 0, 5);
this.camera.lookAt(0, 0, 0); // 旋转自己，让自己的Z轴对准一个世界空间中的点
```

### Renderer

创建
```js
const canvas = document.querySelector('canvas.webgl') as HTMLCanvasElement;
// 配置了canvas的情况
this.renderer = new THREE.WebGLRenderer({
  canvas,
  antialias: true,
  alpha: true,
});
this.renderer.setSize(window.innerWidth, window.innerHeight);

// 未配置canvas，使用appendChild挂载
this.renderer = new THREE.WebGLRenderer({
  antialias: true,
});

document.body.appendChild(this.renderer.domElement);
```

配置参数
```js
this.renderer.physicallyCorrectLights = true;
this.renderer.outputEncoding = THREE.sRGBEncoding;
this.renderer.toneMapping = THREE.CineonToneMapping;
this.renderer.toneMappingExposure = 1.75;
this.renderer.shadowMap.enabled = true;
this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
```

### Control

引入对应的Control
```js
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
```

创建使用
```js
this.controls = new OrbitControls(this.camera, this.renderer.domElement);
// 配置
this.controls.minDistance = 2;
this.controls.maxDistance = 10;
this.controls.target.set(0, 0, -0.2);
this.controls.enablePan = false;
this.controls.enableDamping = true;
// 在每帧中update，添加阻尼效果，转动控制器的时候有种真实感
this.controls.update();
```

### stats 显示帧率

```js
import Stats from 'three/examples/jsm/libs/stats.module.js';
const stats: Stats = Stats();
document.body.appendChild(stats.dom);

requestAnimationFrame(() => {
  stats.update();
});
```
### Helper

axesHelper xyz轴，对应rgb颜色
```js
this.axesHelper = new THREE.AxesHelper(3);
this.scene.add(this.axesHelper);
```

CameraHelper 展示光线阴影或者camera的视野
```js
this.scene.add(new THREE.CameraHelper(this.spotLight.shadow.camera));
this.scene.add(new THREE.CameraHelper(this.dirLight.shadow.camera));
```

SpotLightHelper 点光源投射范围
```js
this.spotLightHelper = new THREE.SpotLightHelper(this.spotLight);
this.scene.add(this.spotLightHelper);
```

SkeletonHelper 模型骨骼展示, 自动识别和展示model中的骨骼部分
```js
this.skeleton = new THREE.SkeletonHelper(this.model);
this.scene.add(this.skeleton);
```

RectAreaLightHelper 非THREE提供
```js
import { RectAreaLightHelper } from "three/examples/jsm/helpers/RectAreaLightHelper.js";

const rectLightHelper = new RectAreaLightHelper(rectLight);
rectLight.add(rectLightHelper);
```

### Mesh

Box
```js
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshBasicMaterial({ color: 0xffff00 });
this.cube = new THREE.Mesh(geometry, material);
this.scene.add(this.cube);
```

Plane
```js
const geometry = new THREE.PlaneGeometry(100, 100);
const material = new THREE.MeshStandardMaterial({
  color: 0xffe6a2,
  side: THREE.BackSide,
});
this.plane = new THREE.Mesh(geometry, material);
this.scene.add(this.plane);
```

circle
```js
const geometry = new THREE.CircleGeometry(5, 64);
this.circle = new THREE.Mesh(geometry, material);
this.scene.add(this.circle);
```

cylinder
```js
const geometryCylinder = new THREE.CylinderGeometry(5, 5, 2, 24, 10);
const materialCylinder = new THREE.MeshPhongMaterial({ color: 0x408080 });
this.cylinder = new THREE.Mesh(geometryCylinder, materialCylinder);
this.cylinder.position.set(0, 10, 0);
this.scene.add(this.cylinder);
```

icosahedron 多面体，一般用来展示球形等
```js
const geometry = new THREE.IcosahedronGeometry(0.5, 3);
```

torusKnot
```js
const geometry = new THREE.TorusKnotGeometry(25, 8, 75, 20);
let material = new THREE.MeshPhongMaterial({
  color: 0x888888,
  shininess: 150,
  specular: 0x222222,
});
this.torusKnot = new THREE.Mesh(geometry, material);
this.torusKnot.scale.multiplyScalar(1 / 18);
this.torusKnot.position.y = 3;
this.scene.add(this.torusKnot);
```

intanceMesh
```js
const geometry = new THREE.IcosahedronGeometry(0.5, 3);
const material = new THREE.MeshPhongMaterial({ color: 0xffffff });
const amount = 10;
const count = Math.pow(amount, 3);
this.mesh = new THREE.InstancedMesh(geometry, material, count);
let i = 0;
const offset = (amount - 1) / 2;
const matrix = new THREE.Matrix4();

for (let x = 0; x < amount; x++) {
  for (let y = 0; y < amount; y++) {
    for (let z = 0; z < amount; z++) {
      matrix.setPosition(offset - x, offset - y, offset - z);
      this.mesh.setMatrixAt(i, matrix);
      this.mesh.setColorAt(i, this.white);
      i++;
    }
  }
}

this.scene.add(this.mesh);
```

buffer: 自定义顶点或者颜色
```js
const geometry = new THREE.BufferGeometry();
const vertices = new Float32Array([100, 0, 0, 0, -100, 0, -100, 0, 0]);
geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3));
const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
const triangle = new THREE.Mesh(geometry, material);
this.scene.add(triangle);
```

### 材质

```js
const material = new THREE.MeshBasicMaterial({ color: 0xffff00 });

const material = new THREE.MeshLambertMaterial({ color: 0xffff00 });

const material = new THREE.MeshStandardMaterial({
  color: 0xffe6a2,
  side: THREE.BackSide,
});

const material = new THREE.MeshPhongMaterial({
  color: 0xa0adaf,
  shininess: 150,
  specular: 0x111111,
  side: THREE.DoubleSide,
});

material = new THREE.MeshPhysicalMaterial();
material.roughness = 0;
material.color.set(0x549dd2);
material.ior = 3;
material.transmission = 1;
material.opacity = 1;

const uniforms = {
  fogDensity: {
    value: 0.45,
  },
  fogColor: {
    value: new THREE.Vector3(0, 0, 0),
  },
  time: {
    value: 1.0,
  },
  uvScale: {
    value: new THREE.Vector2(3.0, 1.0),
  },
};
const material = new THREE.ShaderMaterial({
  uniforms: this.uniforms,
  vertexShader,
  fragmentShader,
});
```

### light

```js
const sunLight: THREE.DirectionalLight = new THREE.DirectionalLight('#ffffff', 3);
sunLight.castShadow = true;
sunLight.shadow.camera.far = 20;
sunLight.shadow.mapSize.set(2048, 2048);
sunLight.shadow.normalBias = 0.05;
sunLight.position.set(-1.5, 7, 3);
this.scene.add(sunLight);

const ambientLight: THREE.AmbientLight = new THREE.AmbientLight('#ffffff', 1);
this.scene.add(ambientLight);

const spotLight: THREE.SpotLight = new THREE.SpotLight(0xffffff, 1);
spotLight.position.set(-50, 80, 0);
spotLight.angle = Math.PI / 6;
spotLight.penumbra = 0.2;
scene.add(this.spotLight);

const hemiLight: THREE.HemisphereLight = new THREE.HemisphereLight(0xffffff, 0x888888);
hemiLight.position.set(0, 1, 0);
hemiLight.intensity = 0.35;
this.scene.add(this.hemiLight);

const rectLight = new THREE.RectAreaLight();
rectLight.width = 0.3;
rectLight.height = 0.4;
```

### GUI

lil-gui
```js
import { GUI } from 'lil-gui';
const GUI = new GUI();
// 添加的属性都是object.key
const spotLightFolder = this.gui.addFolder('Spot Light');
spotLightFolder
  .addColor(this.spotLight, 'color')
  .onChange((val: THREE.Color) => {
    this.spotLight.color.set(val);
    this.render();
    this.spotLightHelper?.update();
  });

const cameraFolder = this.gui.addFolder('Camera');
cameraFolder
  .add(this.camera!.position, 'x', -1000, 1000)
  .onChange((val: number) => {
    this.camera!.position.x = val;
    this.render();
  });

cameraFolder.close();  
```

### shadow

```js
  initShadow(): void {
    this.renderer.shadowMap.enabled = true;
    this.renderer.shadowMap.type = THREE.BasicShadowMap;

    this.spotLight.castShadow = true;
    this.spotLight.shadow.camera.near = 8;
    this.spotLight.shadow.camera.far = 30;
    this.spotLight.shadow.mapSize.width = 1024;
    this.spotLight.shadow.mapSize.height = 1024;

    this.dirLight.castShadow = true;
    this.dirLight.shadow.camera.near = 1;
    this.dirLight.shadow.camera.far = 10;
    this.dirLight.shadow.camera.right = 15;
    this.dirLight.shadow.camera.left = -15;
    this.dirLight.shadow.camera.top = 15;
    this.dirLight.shadow.camera.bottom = -15;
    this.dirLight.shadow.mapSize.width = 1024;
    this.dirLight.shadow.mapSize.height = 1024;

    this.torusKnot.castShadow = true;
    this.torusKnot.receiveShadow = true;

    this.cube.castShadow = true;
    this.cube.receiveShadow = true;

    this.ground.castShadow = false;
    this.ground.receiveShadow = true;
  }
```

### shadowMapViewer

```js
import { ShadowMapViewer } from 'three/examples/jsm/utils/ShadowMapViewer';
this.dirLightShadowMapViewer = new ShadowMapViewer(this.dirLight);
resizeShadowMapViewer();

resizeShadowMapViewer() {
  const size = window.innerWidth * 0.15;
  this.dirLightShadowMapViewer.size.width = size;
  this.dirLightShadowMapViewer.size.height = size;
  // 需要调用update方法，如果是使用this.dirLightShadowMapViewer.size.set(size, size);则不需要， set方法会自动调用updates
  this.dirLightShadowMapViewer.update(); 
}

// 如果光线或者物品有变化，每帧触发修改
requestAnimationFrame(() => {
  this.dirLightShadowMapViewer.render(this.renderer);
});

// resize时调整大小
window.addEventListener('resize', () => {
  this.resizeShadowMapViewer();
});
```

### resize

```js
window.addEventListener('resize', () => {
  this.camera.aspect = window.innerWidth / window.innerHeight;
  this.camera.updateProjectionMatrix();
  this.renderer.setSize(window.innerWidth, window.innerHeight);
});
```

### model loader

gltf + draco解码器
```js
const loader = new GLTFLoader();
// draco解码器
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('js/libs/draco/gltf/');
loader.setDRACOLoader(dracoLoader);
// 配置地址
loader.setPath('models/gltf/');
loader.load('LittlestTokyo.glb', (gltf) => {
  const { scene: model, animations } = gltf;
  model.position.set(1, 1, 0);
  model.scale.set(0.01, 0.01, 0.01);
  this.scene.add(model);
  this.mixer = new THREE.AnimationMixer(model);
  this.mixer.clipAction(animations[0]).play();
});
```

增加加载管理器，增加加载过程/加载结束的回掉
```js
const loadingManager = new THREE.LoadingManager(
  () => {
    // 加载完成
  },
  (itemUrl, itemsLoaded, itemsTotal) => {
    // 加载过程中：（当前资源，已加载资源，资源总数）
  },
  () => {
    // 加载错误
  }
);
// 或者
loadingManager.onStart = (url, itemsLoaded, itemsTotal) => {
  // 加载过程中：（当前资源，已加载资源，资源总数）
}
loadingManager.onLoad = () => {
  // 加载完成
};
loadingManager.onError = url => {
  // 加载错误
}

const loader = new GLTFLoader(loadingManager);
```

### texture loader

texture
```js
// 创建
const texture = new THREE.TextureLoader().load('textures/crate.gif');
// 使用
const materialCube = new THREE.MeshBasicMaterial({
  map: texture,
});
// 相关属性配置
texture.wrapS = THREE.RepeatWrapping;
texture.wrapT = THREE.RepeatWrapping;
```

video texture
```js
// 创建video标签
this.video = document.createElement('video');
this.video.src = '/textures/kda.mp4';  // 资源地址
this.video.muted = true;
this.video.playsInline = true;
this.video.autoplay = true;
this.video.loop = true;
this.video.play();
// 创建videoTexture
this.videoTexture = new THREE.VideoTexture(
  this.video
);
// 配置属性
this.videoTexture.flipY = false;
this.videoTexture.minFilter = THREE.NearestFilter;
this.videoTexture.magFilter = THREE.NearestFilter;
this.videoTexture.generateMipmaps = false;
this.videoTexture.encoding = THREE.sRGBEncoding;
// 使用
const material = new THREE.MeshBasicMaterial({
  map: this.videoTexture
});
```

### requestAnimationFrame中的时间
```js
const clock: THREE.Clock = new THREE.Clock();
const delta = this.clock.getDelta();

this.uniforms1.time.value += delta * 5;
this.uniforms2.time.value = this.clock.elapsedTime;
// window.performance, 以一个恒定的速率慢慢增加的
this.uniforms.time.value = performance.now() / 1000; 
```

### scene.environemnt

```js
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader';
new RGBELoader()
  .setPath('textures/equirectangular/')
  .load('royal_esplanade_1k.hdr', (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;

    this.scene.background = texture;
    this.scene.environment = texture;
  });
```

```js
import { RoomEnvironment } from 'three/examples/jsm/environments/RoomEnvironment';
const pmremGenerator = new THREE.PMREMGenerator(this.renderer);
this.scene.environment = pmremGenerator.fromScene(
  new RoomEnvironment(),
  0.04
).texture;
```

### clipping

全局： 配置到renderer的clippingPlanes
```js
const globalPlane: THREE.Plane = new THREE.Plane(new THREE.Vector3(-1, 0, 0), 9);
this.renderer.clippingPlanes = [this.globalPlane];
```

单独模型, 配置到材质中
```js
this.renderer.localClippingEnabled = true;

const torusKnotClip: THREE.Plane = new THREE.Plane(
  new THREE.Vector3(-1, 0, 0),
  0.1
);

const geometry = new THREE.TorusKnotGeometry(25, 8, 75, 20);
let material = new THREE.MeshPhongMaterial({
  color: 0x888888,
  shininess: 150,
  specular: 0x222222,
  side: THREE.DoubleSide,
  clippingPlanes: [this.torusKnotClip],
  clipShadows: true,
});
this.torusKnot = new THREE.Mesh(geometry, material);
this.torusKnot.scale.multiplyScalar(1 / 18);
this.torusKnot.position.y = 3;
this.scene.add(this.torusKnot);

// 或者
Object.assign(this.cube.material, {
  clippingPlanes: [this.cubeClip],
  clipShadows: true,
});
```

### raycaster

```js
const mouse: THREE.Vector2 = new THREE.Vector2(1, 1);
const raycaster: THREE.Raycaster = new THREE.Raycaster();

// 更新mouse
document.addEventListener('mousemove', (event: MouseEvent) => {
  event.preventDefault();
  this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
});

requestAnimationFrame(() => {
  this.raycaster.setFromCamera(this.mouse, this.camera);

  const intersection = this.raycaster.intersectObject(this.mesh);
  if (intersection.length) { 
    // do something
  }
});
```