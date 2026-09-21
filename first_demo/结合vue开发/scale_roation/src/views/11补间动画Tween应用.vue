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
// 导入tween（three 0.185 内置的是 tween.js v21，只有命名导出，没有 TWEEN 这个导出）
import * as TWEEN from 'three/examples/jsm/libs/tween.module.js'
const { Tween } = TWEEN
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

const shapebox1 = new THREE.Mesh(
    new THREE.SphereGeometry(1, 16, 16),
    new THREE.MeshBasicMaterial({
        color: 0xff00ff
    })
)
scene.add(shapebox1)

// tween(target)          // 创建
// tween.to(duration, props)   // 变化到目标值
// tween.by(duration, props)   // 相对当前值变化
// tween.set(props)            // 立即设置
// tween.delay(duration)       // 延迟
// tween.call(fn)              // 回调
// tween.sequence(...)         // 顺序执行
// tween.parallel(...)         // 并行执行
// tween.repeat(times, tween)  // 重复
// tween.repeatForever(tween)  // 永远重复
// tween.start()               // 开始
// tween.stop()                // 停止

const tween = new Tween(shapebox1.position)
const tween2 = new Tween(shapebox1.position)
tween.to({ x: 4 }, 1000)
tween.delay(1000) //等待一秒钟
// tween.repeat(3) //重复三次
//循环无数次
// tween.yoyo(true)
// 设置缓动函数
tween.easing(TWEEN.Easing.Quadratic.InOut)
tween2.to({ y: 4 }, 1000)
tween.chain(tween2)
tween2.chain(tween)
// 移动补间动画
tween.start()
tween.onStart(() => {
    console.log('开始')
})
tween.onComplete(() => {
    console.log('结束')
})
tween.onStop(() => {
    console.log('停止')
})
tween.onUpdate(() => {
    console.log('更新')
})
// 渲染函数，一帧一帧
function animate() {
    requestAnimationFrame(animate)
    // cube.rotation.x += 0.01
    // cube.rotation.y += 0.01
    controls.update() // 更新旋转
    renderer.render(scene, camera)

    TWEEN.update()
}
let params_stop = {
    stop: function () {
        tween.stop()
    }
}

let params_start = {
    start: function () {
        tween.start()
    }
}

gui.add(params_stop, 'stop')
gui.add(params_start, 'start')
animate()
</script>
