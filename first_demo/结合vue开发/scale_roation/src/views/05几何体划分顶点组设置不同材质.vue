<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'

// 1. 基础环境：场景、相机、渲染器
const scene = new THREE.Scene()
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
const renderer = new THREE.WebGLRenderer()
renderer.setSize(window.innerWidth, window.innerHeight)
document.body.appendChild(renderer.domElement)

// 坐标轴辅助
const axesHelper = new THREE.AxesHelper(5)
scene.add(axesHelper)

// ========== 第一组：父子立方体演示 ==========
const boxGeo = new THREE.BoxGeometry(1, 1, 1)
const matParent = new THREE.MeshBasicMaterial({ color: 0xff0000 })
const matChild = new THREE.MeshBasicMaterial({ color: 0x00ff00, wireframe: true })

const parentCube = new THREE.Mesh(boxGeo, matParent)
const childCube = new THREE.Mesh(boxGeo, matChild)
parentCube.add(childCube)

childCube.position.set(2, 0, 0)
parentCube.scale.set(0.8, 0.8, 0.8)
childCube.scale.set(2, 2, 2)
scene.add(parentCube)

// ========== 第二组：多材质几何体（使用几何体分组 Groups） ==========
// BoxGeometry 本身有6个面，正好6个材质，不需要手写原始buffer顶点
const multiMatGeo = new THREE.BoxGeometry(2, 2, 2)
// 给6个面设置分组，每组对应一个材质索引
multiMatGeo.groups = [
    { start: 0, count: 6, materialIndex: 0 },
    { start: 6, count: 6, materialIndex: 1 },
    { start: 12, count: 6, materialIndex: 2 },
    { start: 18, count: 6, materialIndex: 3 },
    { start: 24, count: 6, materialIndex: 4 },
    { start: 30, count: 6, materialIndex: 5 }
]

// 橙、黄、绿、蓝、靛、紫 六个材质，wireframe关闭
const mat1 = new THREE.MeshBasicMaterial({ color: 0xff7700, wireframe: false }) // 橙
const mat2 = new THREE.MeshBasicMaterial({ color: 0xffff00, wireframe: false }) // 黄
const mat3 = new THREE.MeshBasicMaterial({ color: 0x00ff00, wireframe: false }) // 绿
const mat4 = new THREE.MeshBasicMaterial({ color: 0x0077ff, wireframe: false }) // 蓝
const mat5 = new THREE.MeshBasicMaterial({ color: 0x2200aa, wireframe: false }) // 靛
const mat6 = new THREE.MeshBasicMaterial({ color: 0xaa00ff, wireframe: false }) // 紫

const multiMaterialMesh = new THREE.Mesh(multiMatGeo, [mat1, mat2, mat3, mat4, mat5, mat6])
multiMaterialMesh.position.set(0, 3, 0) // 往上挪，不和父子立方体重叠
scene.add(multiMaterialMesh)

// ========== 相机与轨道控制器 ==========
camera.position.set(2, 2, 8)
const controls = new OrbitControls(camera, renderer.domElement)
controls.enableDamping = true

// ========== 动画循环 ==========
function animate() {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
}
animate()

// ========== 窗口自适应 ==========
window.addEventListener('resize', () => {
    renderer.setSize(window.innerWidth, window.innerHeight)
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
})

// ========== lil-gui控制面板 ==========
const gui = new GUI()
const eventObj = {
    fullscreen() {
        document.body.requestFullscreen()
    },
    exitFullscreen() {
        document.exitFullscreen()
    }
}
gui.add(eventObj, 'fullscreen').name('全屏窗口')
gui.add(eventObj, 'exitFullscreen').name('退出全屏窗口')

// 多材质立方体位置控制
const folderMesh = gui.addFolder('多材质立方体位置')
folderMesh.add(multiMaterialMesh.position, 'x').min(-5).max(5).step(0.1).name('X轴')
folderMesh.add(multiMaterialMesh.position, 'y').min(-5).max(5).step(0.1).name('Y轴')
folderMesh.add(multiMaterialMesh.position, 'z').min(-5).max(5).step(0.1).name('Z轴')

// 可选：父子立方体控制器
const folderParent = gui.addFolder('父立方体位置')
folderParent.add(parentCube.position, 'x').min(-5).max(5).step(0.1).name('X轴')
folderParent.add(parentCube.position, 'y').min(-5).max(5).step(0.1).name('Y轴')
folderParent.add(parentCube.position, 'z').min(-5).max(5).step(0.1).name('Z轴')
</script>
