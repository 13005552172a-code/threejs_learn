<template></template>
<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
// 创建场景
const scene = new THREE.Scene()
// 创建相机
const camera = new THREE.PerspectiveCamera(80, window.innerWidth / window.innerHeight, 0.1, 1000)
// 创建渲染器
const renderer = new THREE.WebGLRenderer()
renderer.setSize(window.innerWidth, window.innerHeight)
// 追加
document.body.appendChild(renderer.domElement)

// 创建几何体
const geometry = new THREE.BoxGeometry(1, 1, 1)
// 创建材质
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
// 创建网格
const cube = new THREE.Mesh(geometry, material)
// 将网格追加到场景中
scene.add(cube)
// 设置相机位置
camera.position.z = 5
camera.position.x = 2
camera.position.y = 2
// 设置坐标辅助系
const axesHelper = new THREE.AxesHelper(5)
scene.add(axesHelper)

// 设置轨道控制器(鼠标可以拖动)
const controls = new OrbitControls(camera, renderer.domElement)
// 设置拖动阻尼的惯性
controls.enableDamping = true
// 设置拖动阻尼的惯性系数
controls.dampingFactor = 0.05
// 设置旋转速度
controls.autoRotate = true

// 渲染函数，一帧一帧
function animate() {
    requestAnimationFrame(animate)
    // cube.rotation.x += 0.01
    // cube.rotation.y += 0.01
    // 更新旋转
    controls.update()
    renderer.render(scene, camera)
}
animate()
// renderer.render(scene, camera)
</script>
