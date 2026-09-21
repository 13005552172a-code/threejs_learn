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
camera.position.x = 7
camera.position.y = 5
camera.position.z = 10
camera.lookAt(0, 0, 0)
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

const shapebox1 = new THREE.Mesh(
    new THREE.SphereGeometry(1, 16, 16),
    new THREE.MeshBasicMaterial({
        color: 0xff00ff
    })
)
scene.add(shapebox1)
shapebox1.position.x = -2
const shapebox2 = new THREE.Mesh(
    new THREE.SphereGeometry(1, 16, 16),
    new THREE.MeshBasicMaterial({
        color: 0x0000ff
    })
)
scene.add(shapebox2)
const shapebox3 = new THREE.Mesh(
    new THREE.SphereGeometry(1, 16, 16),
    new THREE.MeshBasicMaterial({
        color: 0xff0000
    })
)
scene.add(shapebox3)
shapebox3.position.x = 2
// 创建射线
const raycaster = new THREE.Raycaster()
// 创建鼠标向量
const mouse = new THREE.Vector2()
// 三个球放进数组，方便挨个做射线检测
const shapes = [shapebox1, shapebox2, shapebox3]
// 点击事件
window.addEventListener('click', e => {
    // 设置鼠标向量的x,y值
    mouse.x = (e.clientX / window.innerWidth) * 2 - 1
    mouse.y = -(e.clientY / window.innerHeight) * 2 + 1
    // 通过摄像机和鼠标位置更新射线
    raycaster.setFromCamera(mouse, camera)

    // 挨个检测，射线碰到了哪个球，哪个球就是目标(没碰到的返回空数组)
    const hitShape = shapes.find(shape => raycaster.intersectObject(shape).length > 0)
    if (!hitShape) return

    // 第一次被点中时，把原始颜色记在 userData 上(getHex 拿到的是 0xff00ff 这样的数字)
    hitShape.userData.originHex ??= hitShape.material.color.getHex()
    // 取反选中状态
    hitShape.userData.isSelect = !hitShape.userData.isSelect
    // 选中 → 青蓝色高亮；取消选中 → 用记下来的原始色恢复
    hitShape.material.color.set(hitShape.userData.isSelect ? 0x00fff0 : hitShape.userData.originHex)
})
</script>
