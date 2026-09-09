<template></template>
<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
// 导入lil.gui(Three.js 生态最常用的**轻量可视化控制面板库**)
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'

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
// 设置线框模式
material.wireframe = true
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
cube.rotation.x = 0
parentCube.rotation.x = -0
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
// 创建gui实例
const gui = new GUI()
// 添加按钮和命名
gui.add(eventObj, 'Fullscreen').name('全屏窗口')
gui.add(eventObj, 'ExitFullscreen').name('退出全屏窗口')
//控制立方体的位置(输入)
// gui.add(cube.position, 'x').name('x轴位置')
// gui.add(cube.position, 'y').name('y轴位置')
// gui.add(cube.position, 'z').name('z轴位置')
//控制立方体的位置(滑动条)
let folder1 = gui.addFolder('字级立方体位置') //文件夹
folder1
    .add(cube.position, 'x')
    .min(-5)
    .max(5)
    .step(0.1)
    .name('x轴位置')
    .onFinishChange(() => {
        console.log(cube.position.x)
    }) //onFinishChange 当滑动条值改变时触发，用于更新模型的位置
folder1
    .add(cube.position, 'y')
    .min(-5)
    .max(5)
    .step(0.1)
    .name('y轴位置')
    .onFinishChange(() => {
        console.log(cube.position.y)
    })
folder1
    .add(cube.position, 'z')
    .min(-5)
    .max(5)
    .step(0.1)
    .name('z轴位置')
    .onFinishChange(() => {
        console.log(cube.position.z)
    })
let folder2 = gui.addFolder('父级立方体旋转') //文件夹
folder2.add(parentCube.rotation, 'x').min(-5).max(5).step(0.1).name('x轴旋转')
folder2.add(parentCube.rotation, 'y').min(-5).max(5).step(0.1).name('y轴旋转')
folder2.add(parentCube.rotation, 'z').min(-5).max(5).step(0.1).name('z轴旋转')
// gui勾选线框模式
gui.add(material, 'wireframe').name('线框模式')
// gui勾选颜色
let colorParmas = {
    cubeColor: '#00ff00'
}
gui.addColor(colorParmas, 'cubeColor')
    .name('立方体颜色')
    .onFinishChange(value => {
        material.color.set(value)
    })
</script>
