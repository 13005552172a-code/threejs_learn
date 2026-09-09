<template>
    <button style="position: absolute; top: 10px; left: 10px; z-index: 999999999999999999" @click="resetSize">重置全屏窗口</button>
    <button style="position: absolute; top: 30px; left: 10px; z-index: 999999999999999999" @click="exitSize">退出全屏窗口</button>
</template>
<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const resetSize = () => {
    console.log('重置全屏窗口')
    document.body.requestFullscreen()
}
const exitSize = () => {
    console.log('退出全屏窗口')
    document.exitFullscreen()
}
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

// 创建几何体
const geometry = new THREE.BoxGeometry(1, 1, 1)
// 创建材质
const materialfather = new THREE.MeshBasicMaterial({ color: 0xff0000 })
// 创建材质
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
// 创建父元素
const parentCube = new THREE.Mesh(geometry, materialfather)
// 创建子元素
const cube = new THREE.Mesh(geometry, material)
parentCube.add(cube) // 将子元素添加到父元素中
parentCube.position.set(0, 0, 0)
// 相当于给父元素的绝对定位，否则就是根据世界坐标来定位
cube.position.set(2, 0, 0)
// 都是相对于父元素
parentCube.scale.set(0.8, 0.8, 0.8) // 缩放父元素
cube.scale.set(2, 2, 2) // 缩放子元素

// 欧拉角旋转
// 绕着x旋转
cube.rotation.x = Math.PI / 4
parentCube.rotation.x = -Math.PI / 4
// 将网格追加到场景中
scene.add(parentCube)
// 设置相机位置
camera.position.x = 2
camera.position.y = 2
camera.position.z = 5

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
// renderer.render(scene, camera)

// 监听窗口的变化
window.addEventListener('resize', () => {
    // 重置渲染器大小
    renderer.setSize(window.innerWidth, window.innerHeight)
    // 重置相机宽高比
    camera.aspect = window.innerWidth / window.innerHeight
    // 更新相机投影矩阵
    camera.updateProjectionMatrix()
})
</script>
