<template></template>
<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
// 导入lil.gui(Three.js 生态最常用的**轻量可视化控制面板库**)
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'
// 导入hdr加载器
import { GLTFLoader, RGBELoader } from 'three/examples/jsm/Addons.js'
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
camera.position.x = 1
camera.position.y = 1
camera.position.z = 3

// 设置轨道控制器(鼠标可以拖动)
const controls = new OrbitControls(camera, renderer.domElement)
controls.enableDamping = true // 设置阻尼，让控制器更有真实效果,必须在动画循环里调用.update()
// 渲染函数，一帧一帧
function animate() {
    requestAnimationFrame(animate)
    // cube.rotation.x += 0.01
    // cube.rotation.y += 0.01
    controls.update() // 更新旋转
    renderer.render(scene, camera)
}
animate()

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
// 创建场景（线性的）
// scene.fog = new THREE.Fog(0x999999, 0.1, 50)
// 创建场景指数fog(第一个值：颜色，第二个值：密度)
scene.fog = new THREE.FogExp2(0x999999, 0.1)
scene.background = new THREE.Color(0x999999)
// 实例化加载器
const gltfloader = new GLTFLoader()
// 加载完成回调
gltfloader.load('/bighouse.glb', gltf => {
    console.log(gltf)
    scene.add(gltf.scene)
})

// 实例化加载器draco
const draceoLoader = new DRACOLoader()
// 设置draco路径
draceoLoader.setDecoderPath('/draco/')
//设置gltf加载器draco解码器
gltfloader.setDRACOLoader(draceoLoader)

const rgbeLoader = new THREE.TextureLoader()
rgbeLoader.load('/img/Environment.png', texture => {
    // 设置球形映射
    texture.mapping = THREE.EquirectangularReflectionMapping
    // 设置环境贴图
    scene.background = texture
})
</script>
