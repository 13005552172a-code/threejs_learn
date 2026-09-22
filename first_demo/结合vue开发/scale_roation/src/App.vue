<template></template>
<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
// 导入lil.gui(Three.js 生态最常用的**轻量可视化控制面板库**)
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'
// 导入hdr加载器
import { GLTFLoader, plane, RGBELoader } from 'three/examples/jsm/Addons.js'
// 导入gltf加载器
import { DRACOLoader } from 'three/examples/jsm/Addons.js'
// 创建场景
const scene = new THREE.Scene()
// 创建相机
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
// 创建渲染器
const renderer = new THREE.WebGLRenderer()
renderer.setSize(window.innerWidth, window.innerHeight)
// 追加
document.body.appendChild(renderer.domElement)
// 设置坐标辅助系
const axesHelper = new THREE.AxesHelper(5)
scene.add(axesHelper)

// 设置相机位置
camera.position.x = 7
camera.position.y = 5
camera.position.z = 10
camera.lookAt(0, 0, 0)
// 设置轨道控制器(鼠标可以拖动)
const controls = new OrbitControls(camera, renderer.domElement)
controls.enableDamping = true // 设置阻尼，让控制器更有真实效果,必须在动画循环里调用.update()

// 监听窗口的变化
window.addEventListener('resize', () => {
    // 重置渲染器大小
    renderer.setSize(window.innerWidth, window.innerHeight)
    // 重置相机宽高比
    camera.aspect = window.innerWidth / window.innerHeight
    // 更新相机投影矩阵
    camera.updateProjectionMatrix()
})

let eventObj = {
    Fullscreen: function () {
        document.body.requestFullscreen()
        console.log('全屏窗口')
    },
    ExitFullscreen: function () {
        document.exitFullscreen()
        console.log('退出全屏窗口')
    }
}
let params = {}
const gui = new GUI()

const texture = new THREE.TextureLoader().load('/img/environment_tie.png')
// r152+ 必须手动声明这是 sRGB 颜色贴图，否则被当成线性数据，画面发白
texture.colorSpace = THREE.SRGBColorSpace

// 环境贴图贴图(png格式)
const rgbeLoader = new THREE.TextureLoader()
rgbeLoader.load('/img/Environment.png', texture => {
    // 设置球形映射
    texture.mapping = THREE.EquirectangularReflectionMapping
    // 设置环境贴图
    scene.background = texture
    // 设置环境贴图
    scene.environment = texture
})

const shapebox1 = new THREE.Mesh(
    new THREE.PlaneGeometry(2, 2),
    new THREE.MeshBasicMaterial({
        // color: 0xff00ff,
        side: THREE.DoubleSide,
        map: texture
    })
)
shapebox1.position.x = 2
scene.add(shapebox1)

// 手搓一个带 uv 的平面，放在左边和 shapebox1 做对照
// 正方形只有 4 个角，就写 4 个顶点，靠索引复用成 2 个三角形
// 顺序：0 左下 / 1 右下 / 2 右上 / 3 左上
const positions = new Float32Array([
    -1.0,
    -1.0,
    0.0, // 0 左下
    1.0,
    -1.0,
    0.0, // 1 右下
    1.0,
    1.0,
    0.0, // 2 右上
    -1.0,
    1.0,
    0.0 // 3 左上
])

// 索引：用这 4 个顶点拼出 2 个三角形，都取逆时针，从 +Z 看才是正面
// 三角形1 0→1→2：左下,右下,右上
// 三角形2 2→3→0：右上,左上,左下
const indices = new Uint16Array([0, 1, 2, 2, 3, 0])

// uv：与上面 4 个顶点一一对应（uv 原点在贴图左下角）
// 材质里必须有 map，这里写的 uv 才会被真正采样，否则是死数据
const uv = new Float32Array([0, 0, 1, 0, 1, 1, 0, 1])
// 计算出法向量

const uvGeometry = new THREE.BufferGeometry()
uvGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
uvGeometry.setIndex(new THREE.BufferAttribute(indices, 1))
uvGeometry.setAttribute('uv', new THREE.BufferAttribute(uv, 2))
// 计算出法向量
uvGeometry.computeVertexNormals()

const shapebox2 = new THREE.Mesh(
    uvGeometry,
    // PBR 材质，会读法向量，也会读 scene.environment(第 68 行那个环境贴图)
    // roughness 越小越光滑、metalness 越大越像金属 —— 光滑金属能照出环境
    new THREE.MeshStandardMaterial({
        map: texture,
        roughness: 0.1,
        metalness: 0.9,
        side: THREE.DoubleSide
    })
)
shapebox2.position.x = -2
scene.add(shapebox2)

// 渲染函数，一帧一帧
function animate() {
    requestAnimationFrame(animate)
    // cube.rotation.x += 0.01
    // cube.rotation.y += 0.01
    controls.update() // 更新旋转
    renderer.render(scene, camera)
}
animate()
</script>
