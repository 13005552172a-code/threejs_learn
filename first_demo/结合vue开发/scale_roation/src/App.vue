<template>
  <!-- 画布挂到这个容器里，而不是直接挂 document.body，卸载时 Vue 才能连根拔掉 -->
  <div ref="containerRef" class="canvas-container"></div>
</template>

<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
// lil.gui：Three.js 生态最常用的轻量可视化控制面板库
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'
import { onBeforeUnmount, onMounted, ref } from 'vue'

// ============================================================
// 1. 模块级引用
//    凡是跨函数用到的（动画循环 / 尺寸变化 / 卸载清理）都提到这一层，
//    只在 init 内部用一次的临时对象则留在 init 里，别往上堆
// ============================================================
const containerRef = ref<HTMLDivElement>()

let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let controls: OrbitControls
let gui: GUI
let rafId = 0

// 需要手动释放的 GPU 资源，统一收在一处，卸载时遍历 dispose
const disposables: { dispose: () => void }[] = []

// ============================================================
// 2. 初始化
// ============================================================
function init() {
  const container = containerRef.value!

  // -------- 2.1 场景 / 相机 / 渲染器 --------
  scene = new THREE.Scene()

  const { clientWidth: width, clientHeight: height } = container
  camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000)
  camera.position.set(2, 2, 5)

  renderer = new THREE.WebGLRenderer()
  renderer.setSize(width, height)
  container.appendChild(renderer.domElement)

  // 坐标辅助系
  scene.add(new THREE.AxesHelper(5))

  // -------- 2.2 资源：贴图 --------
  const texture = new THREE.TextureLoader().load('/img/environment_tie.png')
  // r152+ 必须手动声明这是 sRGB 颜色贴图，否则被当成线性数据，画面发白
  texture.colorSpace = THREE.SRGBColorSpace
  disposables.push(texture)

  // -------- 2.3 材质 --------
  const builtinMat = new THREE.MeshBasicMaterial({ map: texture })
  // 手搓平面这个留个 wireframe 开关，方便看清三角形的划分
  const customMat = new THREE.MeshBasicMaterial({ map: texture, wireframe: false })
  disposables.push(builtinMat, customMat)

  // -------- 2.4 几何体 --------
  const builtinGeo = new THREE.PlaneGeometry(2, 2)
  const customGeo = createCustomPlaneGeo()
  disposables.push(builtinGeo, customGeo)

  // -------- 2.5 Mesh 并加入场景 --------
  // 两个平面并排做对照：左边手搓、右边内置，贴图效果应当完全一致
  const builtinMesh = new THREE.Mesh(builtinGeo, builtinMat)
  builtinMesh.position.x = 2
  scene.add(builtinMesh)

  const customMesh = new THREE.Mesh(customGeo, customMat)
  customMesh.position.x = -2
  scene.add(customMesh)

  // -------- 2.6 控制器 --------
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true // 阻尼，必须在动画循环里调用 update() 才生效

  // -------- 2.7 GUI --------
  gui = createGui(customMat)

  // -------- 2.8 事件 --------
  window.addEventListener('resize', handleResize)
}

// ============================================================
// 3. 手搓几何体：4 个顶点 + 索引拼出正方形
// ============================================================
function createCustomPlaneGeo() {
  const geo = new THREE.BufferGeometry()

  // 正方形只有 4 个角，就写 4 个顶点，靠索引复用
  // 顺序：0 左下 / 1 右下 / 2 右上 / 3 左上
  const positions = new Float32Array([
    -1.0, -1.0, 0.0, // 0 左下
    1.0, -1.0, 0.0,  // 1 右下
    1.0, 1.0, 0.0,   // 2 右上
    -1.0, 1.0, 0.0   // 3 左上
  ])
  geo.setAttribute('position', new THREE.BufferAttribute(positions, 3))

  // 索引：用这 4 个顶点拼出 2 个三角形，都取逆时针，从 +Z 看才是正面
  // 三角形1 0→1→2：左下,右下,右上
  // 三角形2 2→3→0：右上,左上,左下
  const indices = new Uint16Array([0, 1, 2, 2, 3, 0])
  geo.setIndex(new THREE.BufferAttribute(indices, 1))

  // uv：与上面 4 个顶点一一对应（uv 原点在贴图左下角）
  // 材质里必须有 map，这里写的 uv 才会被真正采样，否则是死数据
  const uv = new Float32Array([0, 0, 1, 0, 1, 1, 0, 1])
  geo.setAttribute('uv', new THREE.BufferAttribute(uv, 2))

  return geo
}

// ============================================================
// 4. GUI 面板
// ============================================================
function createGui(customMat: THREE.MeshBasicMaterial) {
  const panel = new GUI()

  const fullscreenActions = {
    enter: () => containerRef.value?.requestFullscreen(),
    exit: () => document.exitFullscreen()
  }
  panel.add(fullscreenActions, 'enter').name('全屏窗口')
  panel.add(fullscreenActions, 'exit').name('退出全屏窗口')

  // 线框模式：作用在手搓的那个平面上，用来确认三角形是怎么拼的
  panel.add(customMat, 'wireframe').name('线框模式')

  // 取色是乘到贴图上的，所以是「染色」而不是「替换颜色」
  // 默认白色 = 不做任何改变，两个平面看起来才一致
  const colorParams = { planeColor: '#ffffff' }
  panel.addColor(colorParams, 'planeColor')
    .name('平面颜色')
    .onChange(value => customMat.color.set(value))

  return panel
}

// ============================================================
// 5. 动画循环
// ============================================================
function animate() {
  rafId = requestAnimationFrame(animate)
  controls.update()
  renderer.render(scene, camera)
}

// ============================================================
// 6. 尺寸变化
// ============================================================
function handleResize() {
  const container = containerRef.value
  if (!container) return

  const { clientWidth: width, clientHeight: height } = container
  renderer.setSize(width, height)
  camera.aspect = width / height
  camera.updateProjectionMatrix()
}

// ============================================================
// 7. 生命周期
// ============================================================
onMounted(() => {
  init()
  animate()
})

onBeforeUnmount(() => {
  // 先停循环再拆资源：顺序反了可能在已释放的对象上继续渲染
  cancelAnimationFrame(rafId)

  window.removeEventListener('resize', handleResize)
  controls.dispose()
  gui.destroy()
  disposables.forEach(item => item.dispose())
  disposables.length = 0

  renderer.dispose()
  renderer.domElement.remove()
})
</script>

<style scoped>
.canvas-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
</style>
