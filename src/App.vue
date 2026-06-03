<template>
  <div class="canyon-page">
    <div ref="containerRef" class="three-container" @mousemove="handlePointerMove" @mouseleave="handlePointerLeave" @click="handleSceneClick"></div>

    <div class="scan-line"></div>
    <div class="holo-grid"></div>
    <div class="corner corner-left-top"></div>
    <div class="corner corner-right-top"></div>
    <div class="corner corner-left-bottom"></div>
    <div class="corner corner-right-bottom"></div>

    <!-- 右上角：收起 / 展开左侧控制、底部指标、右侧知识面板 -->
    <button class="toggle-panel-btn" @click="togglePanels">
      <span class="toggle-panel-icon">{{ panelsVisible ? '⟪' : '⟫' }}</span>
      <span>{{ panelsVisible ? '收起面板' : '展开面板' }}</span>
    </button>

    <div class="title-panel">
      <div class="title-badge">NARROW CHANNEL WIND · DIGITAL TWIN</div>
      <div class="title-main">狭管效应 3D 风场模拟器</div>
      <div class="title-sub">山谷狭管 · 城市街谷</div>
    </div>
    <transition name="el-fade-in-linear">
      <div v-show="panelsVisible" class="control-panel">
        <div class="panel-header">
          <div>
            <div class="panel-title">演示控制</div>
            <div class="panel-en">CONTROL CENTER</div>
          </div>
          <div class="panel-dot"></div>
        </div>

        <div class="demo-buttons">
          <button class="demo-btn start" :class="{ disabled: demoActive }" @click="startDemo">开始演示</button>
          <button class="demo-btn stop" :class="{ disabled: !demoActive }" @click="stopDemo">结束演示</button>
        </div>

        <div class="mode-switch">
          <button class="mode-btn" :class="{ active: sceneMode === 'mountain' }" @click="switchSceneMode('mountain')">山谷模式</button>
          <button class="mode-btn" :class="{ active: sceneMode === 'city' }" @click="switchSceneMode('city')">城市模式</button>
        </div>

        <div class="demo-status">
          <div class="status-line">
            <span>当前场景</span>
            <strong>{{ sceneMode === 'mountain' ? '山谷狭管' : '城市街谷' }}</strong>
          </div>
          <div class="status-line">
            <span>演示状态</span>
            <strong :class="{ running: demoActive }">
              {{ demoActive ? '演示中' : '已暂停' }}
            </strong>
          </div>
          <div class="status-line">
            <span>当前阶段</span>
            <strong>{{ currentStageText }}</strong>
          </div>
          <div class="status-line">
            <span>狭口加速</span>
            <strong>{{ speedFactorText }}</strong>
          </div>
          <div class="status-line">
            <span>自动风速</span>
            <strong>{{ autoWindSpeedText }}</strong>
          </div>
        </div>

        <div class="divider"></div>

        <div class="section-title">
          {{ sceneMode === 'mountain' ? '山谷地形控制' : '城市街谷控制' }}
        </div>

        <div class="control-row">
          <label>{{ sceneMode === 'mountain' ? '狭口宽度' : '街谷宽度' }}</label>
          <input v-model.number="params.throatWidth" type="range" min="5" max="20" step="1" @input="scheduleAllRebuild" />
          <span>{{ params.throatWidth }}</span>
        </div>

        <div class="control-row">
          <label>{{ sceneMode === 'mountain' ? '山体高度' : '楼宇高度' }}</label>
          <input v-model.number="params.mainHeight" type="range" min="22" max="52" step="1" @input="scheduleAllRebuild" />
          <span>{{ params.mainHeight }}</span>
        </div>

        <div v-if="sceneMode === 'mountain'" class="control-row">
          <label>谷底抬升</label>
          <input v-model.number="params.valleyLift" type="range" min="2" max="10" step="0.5" @input="scheduleAllRebuild" />
          <span>{{ params.valleyLift.toFixed(1) }}</span>
        </div>

        <div v-if="sceneMode === 'mountain'" class="control-row">
          <label>山体粗糙</label>
          <input v-model.number="params.terrainRoughness" type="range" min="0.4" max="1.8" step="0.1" @input="scheduleAllRebuild" />
          <span>{{ params.terrainRoughness.toFixed(1) }}</span>
        </div>

        <div v-if="sceneMode === 'city'" class="control-row">
          <label>楼宇密度</label>
          <input v-model.number="params.buildingDensity" type="range" min="3" max="6" step="1" @input="scheduleAllRebuild" />
          <span>{{ params.buildingDensity }}</span>
        </div>

        <div class="explain-card">
          风速由狭口宽度自动决定：通道越窄，流线越集中，狭口处越快；穿过狭口后风速不会立即消失，而会在下游保持一段较强状态，再逐渐扩散衰减。鼠标悬停狭口、树木或楼宇可查看局部提示，点击可打开知识点说明。
        </div>

        <button class="reset-btn" @click="resetCamera">重置视角</button>
      </div>
    </transition>
    <transition name="el-fade-in-linear">
      <div v-show="panelsVisible" class="knowledge-panel">
        <div class="panel-header">
          <div>
            <div class="panel-title">狭管效应知识系统</div>
            <div class="panel-en">NARROW CHANNEL EFFECT</div>
          </div>
          <div class="panel-dot"></div>
        </div>

        <div class="section-title">什么是狭管效应</div>

        <div class="point">
          <span>定义：</span>
          狭管效应是指空气在通过狭窄通道时，由于可通过的横截面积减小，气流被迫收束，风速增强的现象。
        </div>

        <div class="point">
          <span>本质：</span>
          通道变窄后，气流线被压缩到更小空间中，同一方向上的空气运动更集中，因此会形成局地强风区。
        </div>

        <div class="point">
          <span>山谷场景：</span>
          山谷、峡谷、垭口、两山夹峙的风口等地，容易出现山谷型狭管效应。
        </div>

        <div class="point">
          <span>城市场景：</span>
          高楼之间的狭窄街道、楼宇间通道、建筑群风廊中，也会出现城市狭管效应。
        </div>

        <div class="point">
          <span>关键条件：</span>
          风向要大致顺着通道方向。如果风横穿山体或楼群，气流更多表现为绕流、爬升或紊流，不会形成清晰的狭管加速。
        </div>

        <div class="divider"></div>

        <div class="section-title">形成过程</div>

        <div
          v-for="item in processList"
          :key="item.key"
          class="process-item"
          :class="{ active: currentStage === item.key }"
          @click="setStage(item.key)"
        >
          <div class="process-index">{{ item.index }}</div>
          <div>
            <div class="process-name">{{ item.name }}</div>
            <div class="process-desc">{{ item.desc }}</div>
          </div>
        </div>

        <div class="divider"></div>

        <div class="section-title">山谷与城市对比</div>

        <div class="point">
          <span>山谷模式：</span>
          两侧山体形成自然通道，风沿谷地进入，狭口处流线收束，出谷后仍保持较强风速。
        </div>

        <div class="point">
          <span>城市模式：</span>
          高楼像两侧“人工山体”，街道像狭窄谷地。气流进入楼宇夹道后会向中心压缩，并在中部狭窄街谷处明显加速。
        </div>

        <div class="point">
          <span>参照物：</span>
          上游树木摇晃较弱；狭口或楼宇通道附近树木摇晃最明显；下游树木仍会保持较强摆动。
        </div>

        <div class="divider"></div>

        <div class="section-title">利与弊</div>

        <div class="point">
          <span>有利影响：</span>
          有助于风能资源开发、山谷风理解、城市通风廊道设计、污染物扩散路径判断和热岛缓解。
        </div>

        <div class="point">
          <span>不利影响：</span>
          可能造成局地强风、扬尘、降温、交通风险、行人不适、农作物倒伏、广告牌和临时设施受损。
        </div>

        <div class="point">
          <span>防灾意义：</span>
          山区公路、桥梁、风口村镇、峡谷景区、高楼密集街区，都需要关注狭管效应带来的阵风增强。
        </div>

        <div class="divider"></div>

        <div class="section-title">科学判断</div>

        <div class="judge-box">
          本模型中，风从上游沿通道轴线进入，进入狭窄通道后流线收束、速度增强，穿过狭口后继续保持较强速度，再向下游扩散。风线被限制在安全通道内，不横穿山体，不穿越楼体，不垂直乱飞。
        </div>
      </div>
    </transition>

    <transition name="el-fade-in-linear">
      <!-- 底部只保留实时指标 -->
      <div v-show="panelsVisible" class="bottom-panel metric-only">
        <div class="metric-card full-metric">
          <div class="metric-top">
            <div>
              <div class="metric-title">实时指标</div>
              <div class="metric-sub">LIVE WIND INDEX</div>
            </div>
            <div class="metric-pulse" :class="{ strong: getMaxSpeedFactor() >= 3.4 }"></div>
          </div>

          <div class="metric-grid">
            <div class="metric-row">
              <span>{{ sceneMode === 'mountain' ? '最窄谷口' : '街谷宽度' }}</span>
              <strong>{{ params.throatWidth }} m</strong>
            </div>
            <div class="metric-row">
              <span>峰值加速</span>
              <strong class="highlight">{{ speedFactorText }}</strong>
            </div>
            <div class="metric-row">
              <span>自动风速</span>
              <strong class="highlight">{{ autoWindSpeedText }}</strong>
            </div>
            <div class="metric-row">
              <span>参照物响应</span>
              <strong>{{ demoActive ? '树木摇晃中' : '静止观察' }}</strong>
            </div>
            <div class="metric-row">
              <span>当前阶段</span>
              <strong>{{ currentStageText }}</strong>
            </div>
            <div class="metric-row">
              <span>风线状态</span>
              <strong>{{ demoActive ? '动态流动' : '暂停显示' }}</strong>
            </div>
          </div>

          <canvas ref="speedCanvasRef" class="speed-canvas"></canvas>
        </div>
      </div>
    </transition>

    <div v-if="hoverInfo.visible" class="hover-tip" :style="{ left: `${hoverInfo.x}px`, top: `${hoverInfo.y}px` }">
      <div class="hover-title">{{ hoverInfo.title }}</div>
      <div class="hover-text">{{ hoverInfo.text }}</div>
    </div>

    <div v-if="infoPopup.visible" class="popup-mask" @click.self="closePopup">
      <div class="info-popup">
        <button class="popup-close" @click="closePopup">×</button>
        <div class="popup-badge">KNOWLEDGE NODE</div>
        <div class="popup-title">{{ infoPopup.title }}</div>
        <div class="popup-text">{{ infoPopup.text }}</div>
      </div>
    </div>

    <div v-if="isRebuilding" class="rebuild-tip">场景 / 风场更新中...</div>
  </div>
</template>

<script setup lang="ts">
import { computed, nextTick, onBeforeUnmount, onMounted, reactive, ref } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

type StageKey = 'inflow' | 'narrow' | 'accelerate' | 'outflow'
type SceneMode = 'mountain' | 'city'

interface SmokeTail {
  line: THREE.Line
  geometry: THREE.BufferGeometry
  material: THREE.LineBasicMaterial
  positions: Float32Array
  delay: number
  length: number
  pointCount: number
}

interface SmokeLine {
  group: THREE.Group
  curve: THREE.CatmullRomCurve3
  widthOffset: number
  heightOffset: number
  progress: number
  progressSeed: number
  tails: SmokeTail[]
}

interface TreeRef {
  group?: THREE.Group
  trunk: THREE.Mesh
  crown: THREE.Object3D
  baseRotationZ: number
  windZ: number
  side: number
}

interface InteractiveObject extends THREE.Object3D {
  userData: {
    type?: string
    title?: string
    text?: string
    baseScale?: THREE.Vector3
  }
}

const containerRef = ref<HTMLDivElement | null>(null)
const speedCanvasRef = ref<HTMLCanvasElement | null>(null)

const demoActive = ref(false)
const isRebuilding = ref(false)
const currentStage = ref<StageKey>('inflow')
const sceneMode = ref<SceneMode>('mountain')
const pageVisible = ref(true)
const panelsVisible = ref(true)

function togglePanels() {
  panelsVisible.value = !panelsVisible.value
}

const params = reactive({
  throatWidth: 8,
  mainHeight: 34,
  valleyLift: 6,
  terrainRoughness: 1.1,
  buildingDensity: 4,
})

const hoverInfo = reactive({
  visible: false,
  x: 0,
  y: 0,
  title: '',
  text: '',
})

const infoPopup = reactive({
  visible: false,
  title: '',
  text: '',
})

const WIND_LINE_COUNT = 58
const WIND_HEIGHT_MOUNTAIN = 9.2
const WIND_HEIGHT_CITY = 7.4
const WIND_CURVE_POWER = 0.78
const MIN_SAFE_AIR_HEIGHT = 5.5
const BASE_WIND_SPEED = 0.064
const SPEED_HISTORY_SIZE = 120

const processList: Array<{
  key: StageKey
  index: number
  name: string
  desc: string
}> = [
  {
    key: 'inflow',
    index: 1,
    name: '上游气流进入',
    desc: '通道较宽，风线分散，风速相对稳定，参照物摇晃较弱。',
  },
  {
    key: 'narrow',
    index: 2,
    name: '通道逐渐收缩',
    desc: '两侧山体或楼宇逐渐靠近，空气运动空间变小，流线向中心收束。',
  },
  {
    key: 'accelerate',
    index: 3,
    name: '狭口快速通过',
    desc: '狭口处流线最密集，风速达到峰值，树木或街道参照物摇晃明显。',
  },
  {
    key: 'outflow',
    index: 4,
    name: '下游延续扩散',
    desc: '穿过狭口后风速保持较强，随后随通道变宽逐渐扩散衰减。',
  },
]

const currentStageText = computed(() => {
  return processList.find(item => item.key === currentStage.value)?.name ?? ''
})

const speedFactorText = computed(() => {
  return `${getMaxSpeedFactor().toFixed(1)} 倍`
})

const autoWindSpeedText = computed(() => {
  const factor = getMaxSpeedFactor()
  if (factor >= 4.4) return '强烈加速'
  if (factor >= 3.4) return '明显加速'
  if (factor >= 2.4) return '中等加速'
  return '轻微加速'
})

let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let controls: OrbitControls
let raycaster: THREE.Raycaster
let pointer: THREE.Vector2

let worldGroup: THREE.Group | null = null
let windGroup: THREE.Group | null = null
let labelGroup: THREE.Group | null = null
let decorationGroup: THREE.Group | null = null
let treeGroup: THREE.Group | null = null
let hotspotGroup: THREE.Group | null = null
let focusRing: THREE.Mesh | null = null
let gridHelper: THREE.GridHelper | null = null

let animationId = 0
let allRebuildTimer = 0
let stageTimer = 0
let speedChartTimer = 0

const clock = new THREE.Clock()
let smokeLines: SmokeLine[] = []
let treeRefs: TreeRef[] = []
let interactiveObjects: InteractiveObject[] = []
let lastHoveredObject: InteractiveObject | null = null
let speedHistory: number[] = []

onMounted(async () => {
  await nextTick()

  initThree()
  rebuildAllImmediately()
  createLabels()
  initSpeedCanvas()
  animate()

  window.addEventListener('resize', handleResize)
  document.addEventListener('visibilitychange', handleVisibilityChange)
})

onBeforeUnmount(() => {
  cancelAnimationFrame(animationId)
  clearTimeout(allRebuildTimer)
  clearInterval(stageTimer)
  clearInterval(speedChartTimer)

  window.removeEventListener('resize', handleResize)
  document.removeEventListener('visibilitychange', handleVisibilityChange)

  controls?.dispose()
  renderer?.dispose()

  disposeGroup(worldGroup)
  disposeGroup(windGroup)
  disposeGroup(labelGroup)
  disposeGroup(decorationGroup)
  disposeGroup(treeGroup)
  disposeGroup(hotspotGroup)

  if (gridHelper) {
    gridHelper.geometry.dispose()
    ;(gridHelper.material as THREE.Material).dispose()
  }

  if (focusRing) {
    focusRing.geometry.dispose()
    ;(focusRing.material as THREE.Material).dispose()
  }
})

function initThree() {
  const container = containerRef.value!
  const width = container.clientWidth
  const height = container.clientHeight

  scene = new THREE.Scene()
  scene.background = new THREE.Color('#06101f')
  scene.fog = new THREE.Fog('#1b2f45', 115, 350)

  camera = new THREE.PerspectiveCamera(46, width / height, 0.1, 1000)
  camera.position.set(0, 74, 172)

  renderer = new THREE.WebGLRenderer({
    antialias: true,
    alpha: true,
  })

  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.setSize(width, height)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  renderer.outputColorSpace = THREE.SRGBColorSpace

  container.appendChild(renderer.domElement)

  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.08
  controls.target.set(0, 14, 0)
  controls.minDistance = 78
  controls.maxDistance = 260
  controls.maxPolarAngle = Math.PI * 0.49

  raycaster = new THREE.Raycaster()
  pointer = new THREE.Vector2()

  const ambient = new THREE.AmbientLight('#e5f3ff', 0.82)
  scene.add(ambient)

  const sun = new THREE.DirectionalLight('#ffffff', 2.35)
  sun.position.set(-72, 120, 78)
  sun.castShadow = true
  sun.shadow.mapSize.set(2048, 2048)
  sun.shadow.camera.near = 1
  sun.shadow.camera.far = 280
  sun.shadow.camera.left = -130
  sun.shadow.camera.right = 130
  sun.shadow.camera.top = 130
  sun.shadow.camera.bottom = -130
  scene.add(sun)

  const cyanLight = new THREE.PointLight('#18d8ff', 3.8, 240)
  cyanLight.position.set(0, 52, 10)
  scene.add(cyanLight)

  const blueLight = new THREE.PointLight('#2e6cff', 2.8, 220)
  blueLight.position.set(-60, 48, -58)
  scene.add(blueLight)

  const amberLight = new THREE.PointLight('#ffb35c', 2.1, 190)
  amberLight.position.set(66, 46, 58)
  scene.add(amberLight)

  createTechGrid()
  createTechDecorations()
  createFocusRing()
}

function createTechGrid() {
  gridHelper = new THREE.GridHelper(190, 76, '#18d8ff', '#203b58')
  gridHelper.position.y = -10.15

  const material = gridHelper.material as THREE.Material
  material.transparent = true
  material.opacity = 0.22

  scene.add(gridHelper)
}

function createTechDecorations() {
  disposeGroup(decorationGroup)

  decorationGroup = new THREE.Group()
  scene.add(decorationGroup)

  const ringMat = new THREE.MeshBasicMaterial({
    color: '#18d8ff',
    transparent: true,
    opacity: 0.22,
    side: THREE.DoubleSide,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
  })

  for (let i = 0; i < 5; i++) {
    const ring = new THREE.Mesh(new THREE.RingGeometry(42 + i * 14, 42.45 + i * 14, 180), ringMat.clone())

    ring.rotation.x = -Math.PI / 2
    ring.position.y = -9.75
    decorationGroup.add(ring)
  }

  const cylinderGeo = new THREE.CylinderGeometry(9, 9, 90, 108, 1, true)
  const cylinderMat = new THREE.MeshBasicMaterial({
    color: '#18d8ff',
    transparent: true,
    opacity: 0.07,
    side: THREE.DoubleSide,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
  })

  const cylinder = new THREE.Mesh(cylinderGeo, cylinderMat)
  cylinder.position.set(0, 35, 0)
  decorationGroup.add(cylinder)

  const throatRing = new THREE.Mesh(
    new THREE.TorusGeometry(14, 0.08, 8, 140),
    new THREE.MeshBasicMaterial({
      color: '#ffb35c',
      transparent: true,
      opacity: 0.62,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )
  throatRing.rotation.x = Math.PI / 2
  throatRing.position.set(0, 1.2, 0)
  decorationGroup.add(throatRing)

  const axisLine = new THREE.Line(
    new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(0, 1.1, -74), new THREE.Vector3(0, 1.1, 74)]),
    new THREE.LineBasicMaterial({
      color: '#18d8ff',
      transparent: true,
      opacity: 0.34,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )
  decorationGroup.add(axisLine)
}

function createFocusRing() {
  focusRing = new THREE.Mesh(
    new THREE.TorusGeometry(4.5, 0.08, 8, 96),
    new THREE.MeshBasicMaterial({
      color: '#ffb35c',
      transparent: true,
      opacity: 0,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )

  focusRing.rotation.x = Math.PI / 2
  focusRing.renderOrder = 10
  scene.add(focusRing)
}

function handleVisibilityChange() {
  pageVisible.value = !document.hidden

  if (pageVisible.value) {
    clock.getDelta()
    redistributeWindProgress(false)
  }
}

function initSpeedCanvas() {
  speedHistory = new Array(SPEED_HISTORY_SIZE).fill(getMaxSpeedFactor())
  drawSpeedChart()

  clearInterval(speedChartTimer)
  speedChartTimer = window.setInterval(() => {
    const latest = demoActive.value ? getLiveSpeedSample() : getMaxSpeedFactor()
    speedHistory.push(latest)

    if (speedHistory.length > SPEED_HISTORY_SIZE) {
      speedHistory.shift()
    }

    drawSpeedChart()
  }, 160)
}

function drawSpeedChart() {
  const canvas = speedCanvasRef.value
  if (!canvas) return

  const parent = canvas.parentElement
  const width = parent ? parent.clientWidth - 28 : 420
  const height = 76

  const dpr = Math.min(window.devicePixelRatio || 1, 2)
  canvas.width = width * dpr
  canvas.height = height * dpr
  canvas.style.width = `${width}px`
  canvas.style.height = `${height}px`

  const ctx = canvas.getContext('2d')
  if (!ctx) return

  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
  ctx.clearRect(0, 0, width, height)

  const gradientBg = ctx.createLinearGradient(0, 0, width, 0)
  gradientBg.addColorStop(0, 'rgba(24,216,255,0.06)')
  gradientBg.addColorStop(0.5, 'rgba(255,179,92,0.08)')
  gradientBg.addColorStop(1, 'rgba(24,216,255,0.04)')
  ctx.fillStyle = gradientBg
  drawCanvasRoundRect(ctx, 0, 0, width, height, 12)
  ctx.fill()

  ctx.strokeStyle = 'rgba(24,216,255,0.12)'
  ctx.lineWidth = 1

  for (let i = 1; i < 4; i++) {
    const y = (height / 4) * i
    ctx.beginPath()
    ctx.moveTo(8, y)
    ctx.lineTo(width - 8, y)
    ctx.stroke()
  }

  const min = 1
  const max = 6.4
  const values = speedHistory.length ? speedHistory : [getMaxSpeedFactor()]

  ctx.beginPath()

  values.forEach((value, index) => {
    const x = 10 + (index / Math.max(1, values.length - 1)) * (width - 20)
    const normalized = THREE.MathUtils.clamp((value - min) / (max - min), 0, 1)
    const y = height - 10 - normalized * (height - 20)

    if (index === 0) {
      ctx.moveTo(x, y)
    } else {
      ctx.lineTo(x, y)
    }
  })

  const lineGradient = ctx.createLinearGradient(0, 0, width, 0)
  lineGradient.addColorStop(0, '#ffffff')
  lineGradient.addColorStop(0.45, '#18d8ff')
  lineGradient.addColorStop(0.6, '#ffb35c')
  lineGradient.addColorStop(1, '#18d8ff')

  ctx.strokeStyle = lineGradient
  ctx.lineWidth = 2.4
  ctx.shadowColor = 'rgba(24,216,255,0.65)'
  ctx.shadowBlur = 10
  ctx.stroke()

  ctx.shadowBlur = 0
  ctx.fillStyle = 'rgba(234,250,255,0.72)'
  ctx.font = '11px Microsoft YaHei'
  ctx.fillText('实时风速曲线', 10, 17)

  const latest = values[values.length - 1]!
  ctx.fillStyle = latest >= 3.4 ? '#ffb35c' : '#18d8ff'
  ctx.font = 'bold 13px Microsoft YaHei'
  ctx.fillText(`${latest.toFixed(1)}x`, width - 48, 17)
}

function drawCanvasRoundRect(ctx: CanvasRenderingContext2D, x: number, y: number, w: number, h: number, r: number) {
  ctx.beginPath()
  ctx.moveTo(x + r, y)
  ctx.lineTo(x + w - r, y)
  ctx.quadraticCurveTo(x + w, y, x + w, y + r)
  ctx.lineTo(x + w, y + h - r)
  ctx.quadraticCurveTo(x + w, y + h, x + w - r, y + h)
  ctx.lineTo(x + r, y + h)
  ctx.quadraticCurveTo(x, y + h, x, y + h - r)
  ctx.lineTo(x, y + r)
  ctx.quadraticCurveTo(x, y, x + r, y)
  ctx.closePath()
}

function getLiveSpeedSample() {
  const time = clock.elapsedTime
  const base = getMaxSpeedFactor()
  const pulse = Math.sin(time * 2.8) * 0.18 + Math.sin(time * 5.1) * 0.08
  return Math.max(1, base + pulse)
}

function redistributeWindProgress(force = false) {
  if (!smokeLines.length) return

  for (let i = 0; i < smokeLines.length; i++) {
    const line = smokeLines[i]!

    if (force) {
      line.progress = getInitialProgress(i)
    } else {
      line.progress = normalizeProgress(line.progress + line.progressSeed * 0.013)
    }

    updateSmokeLineGeometry(line, 0, true)
  }
}

function switchSceneMode(mode: SceneMode) {
  if (sceneMode.value === mode) return

  sceneMode.value = mode
  rebuildAllImmediately()
  createLabels()

  if (mode === 'city') {
    smoothCameraTo(new THREE.Vector3(0, 78, 166), new THREE.Vector3(0, 18, 0))
  } else {
    smoothCameraTo(new THREE.Vector3(0, 74, 172), new THREE.Vector3(0, 14, 0))
  }
}

function scheduleAllRebuild() {
  isRebuilding.value = true
  clearTimeout(allRebuildTimer)

  allRebuildTimer = window.setTimeout(() => {
    rebuildAllImmediately()
    createLabels()
    initSpeedCanvas()
    isRebuilding.value = false
  }, 180)
}

function rebuildAllImmediately() {
  disposeGroup(worldGroup)
  disposeGroup(windGroup)
  disposeGroup(treeGroup)
  disposeGroup(hotspotGroup)

  smokeLines = []
  treeRefs = []
  interactiveObjects = []
  lastHoveredObject = null

  worldGroup = new THREE.Group()
  windGroup = new THREE.Group()
  treeGroup = new THREE.Group()
  hotspotGroup = new THREE.Group()

  scene.add(worldGroup)
  scene.add(windGroup)
  scene.add(treeGroup)
  scene.add(hotspotGroup)

  if (sceneMode.value === 'mountain') {
    createMountainWorld()
    createMountainTrees()
    createMountainHotspots()
  } else {
    createCityWorld()
    createCityTrees()
    createCityHotspots()
  }

  createWindField()
  redistributeWindProgress(true)
}

function createMountainWorld() {
  createSolidMountainModel()
  createRiverSurface()
  createThroatMarker()
}

function createCityWorld() {
  createCityGround()
  createCityBuildings()
  createStreetContractionGuides()
}

function createSolidMountainModel() {
  const totalWidth = 126
  const totalLength = 146
  const segX = 142
  const segZ = 166
  const baseY = -10

  const topVertices: THREE.Vector3[][] = []

  const colorLow = new THREE.Color('#5a513f')
  const colorVegetation = new THREE.Color('#58733f')
  const colorGrass = new THREE.Color('#776a50')
  const colorRock = new THREE.Color('#8b7c64')
  const colorPeak = new THREE.Color('#c4b79a')
  const sideColor = new THREE.Color('#2a2c2f')
  const bottomColor = new THREE.Color('#11151b')

  for (let iz = 0; iz <= segZ; iz++) {
    const row: THREE.Vector3[] = []

    for (let ix = 0; ix <= segX; ix++) {
      const x = -totalWidth / 2 + (ix / segX) * totalWidth
      const z = -totalLength / 2 + (iz / segZ) * totalLength
      const y = getTerrainHeight(x, z)

      row.push(new THREE.Vector3(x, y, z))
    }

    topVertices.push(row)
  }

  const positions: number[] = []
  const indices: number[] = []
  const vertexColors: number[] = []

  function getColorByHeightAndSlope(x: number, z: number, y: number) {
    const h = THREE.MathUtils.clamp(y / (params.mainHeight * 1.7), 0, 1)
    const c = new THREE.Color()

    const halfValley = getHalfValleyWidth(z)
    const distanceFromCenter = Math.abs(x)
    const vegetationBand = distanceFromCenter > halfValley + 2 && distanceFromCenter < halfValley + 23 && h > 0.16 && h < 0.58

    if (vegetationBand) {
      const noise = smoothNoise2D(x * 0.06, z * 0.06)
      const mix = THREE.MathUtils.clamp(0.45 + noise * 0.25, 0.18, 0.72)
      c.copy(colorGrass).lerp(colorVegetation, mix)
      return c
    }

    if (h < 0.2) {
      c.copy(colorLow).lerp(colorGrass, h / 0.2)
    } else if (h < 0.62) {
      c.copy(colorGrass).lerp(colorRock, (h - 0.2) / 0.42)
    } else {
      c.copy(colorRock).lerp(colorPeak, (h - 0.62) / 0.38)
    }

    return c
  }

  function pushVertex(v: THREE.Vector3, color: THREE.Color) {
    positions.push(v.x, v.y, v.z)
    vertexColors.push(color.r, color.g, color.b)
    return positions.length / 3 - 1
  }

  const topIndex: number[][] = []

  for (let iz = 0; iz <= segZ; iz++) {
    const row: number[] = []

    for (let ix = 0; ix <= segX; ix++) {
      const v = topVertices![iz]![ix]!
      row.push(pushVertex(v, getColorByHeightAndSlope(v.x, v.z, v.y)))
    }

    topIndex.push(row)
  }

  for (let iz = 0; iz < segZ; iz++) {
    for (let ix = 0; ix < segX; ix++) {
      const a = topIndex![iz]![ix]!
      const b = topIndex![iz]![ix + 1]!
      const c = topIndex![iz + 1]![ix]!
      const d = topIndex![iz + 1]![ix + 1]!

      indices.push(a, c, b)
      indices.push(b, c, d)
    }
  }

  function addSide(vTopA: THREE.Vector3, vTopB: THREE.Vector3) {
    const vBottomA = new THREE.Vector3(vTopA.x, baseY, vTopA.z)
    const vBottomB = new THREE.Vector3(vTopB.x, baseY, vTopB.z)

    const a = pushVertex(vTopA, getColorByHeightAndSlope(vTopA.x, vTopA.z, vTopA.y))
    const b = pushVertex(vTopB, getColorByHeightAndSlope(vTopB.x, vTopB.z, vTopB.y))
    const c = pushVertex(vBottomA, sideColor)
    const d = pushVertex(vBottomB, sideColor)

    indices.push(a, c, b)
    indices.push(b, c, d)
  }

  for (let ix = 0; ix < segX; ix++) {
    addSide(topVertices![0]![ix]!, topVertices![0]![ix + 1]!)
    addSide(topVertices![segZ]![ix + 1]!, topVertices![segZ]![ix]!)
  }

  for (let iz = 0; iz < segZ; iz++) {
    addSide(topVertices![iz + 1]![0]!, topVertices![iz]![0]!)
    addSide(topVertices![iz]![segX]!, topVertices![iz + 1]![segX]!)
  }

  const b1 = pushVertex(new THREE.Vector3(-totalWidth / 2, baseY, -totalLength / 2), bottomColor)
  const b2 = pushVertex(new THREE.Vector3(totalWidth / 2, baseY, -totalLength / 2), bottomColor)
  const b3 = pushVertex(new THREE.Vector3(-totalWidth / 2, baseY, totalLength / 2), bottomColor)
  const b4 = pushVertex(new THREE.Vector3(totalWidth / 2, baseY, totalLength / 2), bottomColor)

  indices.push(b1, b3, b2)
  indices.push(b2, b3, b4)

  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
  geometry.setAttribute('color', new THREE.Float32BufferAttribute(vertexColors, 3))
  geometry.setIndex(indices)
  geometry.computeVertexNormals()

  const material = new THREE.MeshStandardMaterial({
    vertexColors: true,
    roughness: 0.96,
    metalness: 0.02,
    side: THREE.DoubleSide,
  })

  const mountain = new THREE.Mesh(geometry, material)
  mountain.castShadow = true
  mountain.receiveShadow = true
  worldGroup!.add(mountain)

  const wire = new THREE.LineSegments(
    new THREE.WireframeGeometry(geometry),
    new THREE.LineBasicMaterial({
      color: '#18d8ff',
      transparent: true,
      opacity: 0.014,
      blending: THREE.AdditiveBlending,
    }),
  )
  worldGroup!.add(wire)
}

function getTerrainHeight(x: number, z: number) {
  const halfValley = getHalfValleyWidth(z)
  const absX = Math.abs(x)
  const distanceFromValley = absX - halfValley
  const sideSign = x >= 0 ? 1 : -1

  let y = params.valleyLift

  if (distanceFromValley <= 0) {
    const riverCut = Math.exp(-(x * x) / 18) * 2.35
    const valleyUndulation = fbm(x * 0.08, z * 0.08, 4) * 0.9 + fbm(x * 0.18 + 12, z * 0.16 - 6, 3) * 0.36

    y += valleyUndulation - riverCut
  } else {
    const slopeT = THREE.MathUtils.clamp(distanceFromValley / 35, 0, 1)
    const baseMountain = Math.pow(slopeT, 1.36) * params.mainHeight

    const ridgeCenter = sideSign * (halfValley + 19 + Math.sin(z * 0.05 + sideSign * 1.4) * 5 + smoothNoise2D(z * 0.035, sideSign * 4) * 4)

    const ridgeDistance = Math.abs(x - ridgeCenter)
    const ridge = Math.exp(-(ridgeDistance * ridgeDistance) / 150) * params.mainHeight * 0.52

    const throat = Math.exp(-(z * z) / 580)
    const cliff = throat * Math.exp(-Math.pow(distanceFromValley - 3.2, 2) / 32) * params.mainHeight * 0.42

    const rough = fbm(x * 0.055, z * 0.055, 5) * params.terrainRoughness * 5.2 + fbm(x * 0.13 + 18, z * 0.13 - 7, 4) * params.terrainRoughness * 2.2

    y += baseMountain + ridge + cliff + rough
  }

  const farRise = Math.pow(Math.abs(x) / 63, 2.2) * params.mainHeight * 0.34

  y += farRise

  return y
}

function createRiverSurface() {
  const riverGroup = new THREE.Group()
  riverGroup.name = 'riverSurfaceGroup'
  worldGroup!.add(riverGroup)

  const count = 180
  const centers: THREE.Vector3[] = []

  for (let i = 0; i <= count; i++) {
    const t = i / count
    const z = -70 + t * 140

    const x = Math.sin(t * Math.PI * 2.7) * 1.25 + Math.sin(t * Math.PI * 7.2 + 0.8) * 0.34 + Math.sin(t * Math.PI * 13.4) * 0.12

    const y = getTerrainHeight(x, z) + 0.34

    centers.push(new THREE.Vector3(x, y, z))
  }

  const positions: number[] = []
  const uvs: number[] = []
  const indices: number[] = []
  const leftBank: THREE.Vector3[] = []
  const rightBank: THREE.Vector3[] = []

  for (let i = 0; i <= count; i++) {
    const p = centers[i]!
    const prev = centers[Math.max(0, i - 1)]!
    const next = centers[Math.min(count, i + 1)]!

    const tangent = new THREE.Vector3().subVectors(next!, prev!).normalize()
    const normal = new THREE.Vector3(-tangent.z, 0, tangent.x).normalize()

    const halfValley = getHalfValleyWidth(p.z)
    const widthBase = THREE.MathUtils.clamp(halfValley * 0.15, 1.25, 3.1)
    const naturalWidth = widthBase + Math.sin((i / count) * Math.PI * 5.5) * 0.18 + smoothNoise2D(i * 0.08, 2) * 0.18

    const left = p.clone().addScaledVector(normal, naturalWidth)
    const right = p.clone().addScaledVector(normal, -naturalWidth)

    left.y = getTerrainHeight(left.x, left.z) + 0.28
    right.y = getTerrainHeight(right.x, right.z) + 0.28

    leftBank.push(left.clone())
    rightBank.push(right.clone())

    positions.push(left.x, left.y, left.z)
    positions.push(right.x, right.y, right.z)

    uvs.push(0, i / count)
    uvs.push(1, i / count)
  }

  for (let i = 0; i < count; i++) {
    const a = i * 2
    const b = i * 2 + 1
    const c = (i + 1) * 2
    const d = (i + 1) * 2 + 1

    indices.push(a, c, b)
    indices.push(b, c, d)
  }

  const riverGeometry = new THREE.BufferGeometry()
  riverGeometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
  riverGeometry.setAttribute('uv', new THREE.Float32BufferAttribute(uvs, 2))
  riverGeometry.setIndex(indices)
  riverGeometry.computeVertexNormals()

  const riverMaterial = new THREE.MeshStandardMaterial({
    color: '#2fc7e8',
    transparent: true,
    opacity: 0.58,
    roughness: 0.18,
    metalness: 0.26,
    emissive: '#007fa8',
    emissiveIntensity: 0.32,
    side: THREE.DoubleSide,
    depthWrite: false,
  })

  const river = new THREE.Mesh(riverGeometry, riverMaterial)
  river.renderOrder = 2
  riverGroup.add(river)

  const bankMaterial = new THREE.LineBasicMaterial({
    color: '#dffbff',
    transparent: true,
    opacity: 0.16,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
  })

  riverGroup.add(new THREE.Line(new THREE.BufferGeometry().setFromPoints(leftBank), bankMaterial.clone()))
  riverGroup.add(new THREE.Line(new THREE.BufferGeometry().setFromPoints(rightBank), bankMaterial.clone()))
}

function createThroatMarker() {
  const width = params.throatWidth

  const ring = new THREE.Mesh(
    new THREE.TorusGeometry(width * 0.62, 0.08, 8, 120),
    new THREE.MeshBasicMaterial({
      color: '#ffb35c',
      transparent: true,
      opacity: 0.82,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )

  ring.position.set(0, getTerrainHeight(0, 0) + 1.4, 0)
  ring.rotation.x = Math.PI / 2
  ring.scale.z = 0.38

  worldGroup!.add(ring)
}

function createCityGround() {
  const ground = new THREE.Mesh(
    new THREE.BoxGeometry(128, 2, 150),
    new THREE.MeshStandardMaterial({
      color: '#232a31',
      roughness: 0.8,
      metalness: 0.12,
    }),
  )
  ground.position.y = -1
  ground.receiveShadow = true
  worldGroup!.add(ground)

  createVariableStreetSurface()

  const roadLineMaterial = new THREE.LineBasicMaterial({
    color: '#d9e9ff',
    transparent: true,
    opacity: 0.45,
  })

  const leftEdge: THREE.Vector3[] = []
  const rightEdge: THREE.Vector3[] = []

  for (let i = 0; i <= 120; i++) {
    const t = i / 120
    const z = -72 + t * 144
    const half = getCityStreetHalfWidth(z)

    leftEdge.push(new THREE.Vector3(-half, 0.32, z))
    rightEdge.push(new THREE.Vector3(half, 0.32, z))
  }

  worldGroup!.add(new THREE.Line(new THREE.BufferGeometry().setFromPoints(leftEdge), roadLineMaterial.clone()))
  worldGroup!.add(new THREE.Line(new THREE.BufferGeometry().setFromPoints(rightEdge), roadLineMaterial.clone()))

  for (let z = -60; z <= 60; z += 14) {
    const dash = new THREE.Mesh(
      new THREE.BoxGeometry(0.38, 0.08, 4.8),
      new THREE.MeshBasicMaterial({
        color: '#f0f6ff',
        transparent: true,
        opacity: 0.62,
      }),
    )
    dash.position.set(0, 0.38, z)
    worldGroup!.add(dash)
  }
}

function createVariableStreetSurface() {
  const count = 140
  const positions: number[] = []
  const uvs: number[] = []
  const indices: number[] = []

  for (let i = 0; i <= count; i++) {
    const t = i / count
    const z = -72 + t * 144
    const half = getCityStreetHalfWidth(z)

    positions.push(-half, 0.18, z)
    positions.push(half, 0.18, z)

    uvs.push(0, t)
    uvs.push(1, t)
  }

  for (let i = 0; i < count; i++) {
    const a = i * 2
    const b = i * 2 + 1
    const c = (i + 1) * 2
    const d = (i + 1) * 2 + 1

    indices.push(a, c, b)
    indices.push(b, c, d)
  }

  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
  geometry.setAttribute('uv', new THREE.Float32BufferAttribute(uvs, 2))
  geometry.setIndex(indices)
  geometry.computeVertexNormals()

  const road = new THREE.Mesh(
    geometry,
    new THREE.MeshStandardMaterial({
      color: '#1a2028',
      roughness: 0.72,
      metalness: 0.22,
      side: THREE.DoubleSide,
    }),
  )
  road.receiveShadow = true
  worldGroup!.add(road)
}

function createCityBuildings() {
  const rows = params.buildingDensity
  const zValues: number[] = []

  for (let i = -rows; i <= rows; i++) {
    zValues.push(i * 16)
  }

  const facadeMaterials = createBuildingMaterials()

  for (const z of zValues) {
    for (let side of [-1, 1]) {
      const streetHalf = getCityStreetHalfWidth(z)
      const width = 8 + deterministicNoise(z, side) * 4
      const depth = 10 + deterministicNoise(z * 0.7, side * 2) * 6
      const heightNoise = deterministicNoise(z * 0.5 + 10, side * 3)
      const height = params.mainHeight * (0.72 + heightNoise * 0.82)

      const x = side * (streetHalf + width * 0.5 + 2.4 + deterministicNoise(z * 1.7, side * 4) * 1.2)

      const building = new THREE.Mesh(
        new THREE.BoxGeometry(width, height, depth),
        facadeMaterials.map(m => m.clone()),
      ) as InteractiveObject

      building.position.set(x, height / 2, z + (deterministicNoise(z * 0.33, side * 5) - 0.5) * 3.4)
      building.castShadow = true
      building.receiveShadow = true
      building.userData.type = 'building'
      building.userData.title = '楼宇夹道'
      building.userData.text = '城市中的高楼会像两侧山体一样限制空气运动空间。风沿街道进入时，楼宇间通道越窄，流线越容易向中心收束，街谷风越明显。'
      building.userData.baseScale = building.scale.clone()

      worldGroup!.add(building)
      interactiveObjects.push(building)

      addRoofDetail(building.position.x, height, building.position.z, width, depth)
    }
  }

  for (let z = -64; z <= 64; z += 24) {
    addStreetLamp(-getCityStreetHalfWidth(z) - 2.4, z)
    addStreetLamp(getCityStreetHalfWidth(z) + 2.4, z)
  }
}

function createBuildingMaterials() {
  const texture = createBuildingTexture()

  const wall = new THREE.MeshStandardMaterial({
    color: '#6c737a',
    roughness: 0.62,
    metalness: 0.2,
    map: texture,
  })

  const side = new THREE.MeshStandardMaterial({
    color: '#555d65',
    roughness: 0.68,
    metalness: 0.15,
    map: texture,
  })

  const roof = new THREE.MeshStandardMaterial({
    color: '#333941',
    roughness: 0.7,
    metalness: 0.2,
  })

  return [side, side, roof, roof, wall, wall]
}

function createBuildingTexture() {
  const canvas = document.createElement('canvas')
  canvas.width = 256
  canvas.height = 512

  const ctx = canvas.getContext('2d')!
  ctx.fillStyle = '#5d666f'
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  for (let y = 18; y < canvas.height; y += 38) {
    for (let x = 18; x < canvas.width; x += 42) {
      const lit = deterministicNoise(x, y) > 0.45
      ctx.fillStyle = lit ? 'rgba(180, 230, 255, 0.85)' : 'rgba(22, 34, 46, 0.8)'
      ctx.fillRect(x, y, 18, 18)
      ctx.fillStyle = 'rgba(255,255,255,0.12)'
      ctx.fillRect(x + 2, y + 2, 5, 5)
    }
  }

  ctx.strokeStyle = 'rgba(255,255,255,0.1)'
  ctx.lineWidth = 2
  for (let y = 0; y < canvas.height; y += 38) {
    ctx.beginPath()
    ctx.moveTo(0, y)
    ctx.lineTo(canvas.width, y)
    ctx.stroke()
  }

  const texture = new THREE.CanvasTexture(canvas)
  texture.colorSpace = THREE.SRGBColorSpace
  texture.wrapS = THREE.RepeatWrapping
  texture.wrapT = THREE.RepeatWrapping
  texture.repeat.set(1, 1)
  return texture
}

function addRoofDetail(x: number, height: number, z: number, width: number, depth: number) {
  const box = new THREE.Mesh(
    new THREE.BoxGeometry(width * 0.35, 1.2, depth * 0.32),
    new THREE.MeshStandardMaterial({
      color: '#2c3138',
      roughness: 0.7,
      metalness: 0.2,
    }),
  )
  box.position.set(x, height + 0.6, z)
  box.castShadow = true
  worldGroup!.add(box)

  const antenna = new THREE.Mesh(
    new THREE.CylinderGeometry(0.06, 0.06, 5, 10),
    new THREE.MeshBasicMaterial({
      color: '#18d8ff',
      transparent: true,
      opacity: 0.8,
    }),
  )
  antenna.position.set(x + width * 0.18, height + 3.5, z + depth * 0.12)
  worldGroup!.add(antenna)
}

function addStreetLamp(x: number, z: number) {
  const pole = new THREE.Mesh(
    new THREE.CylinderGeometry(0.08, 0.1, 5.2, 10),
    new THREE.MeshStandardMaterial({
      color: '#2e3740',
      roughness: 0.45,
      metalness: 0.4,
    }),
  )
  pole.position.set(x, 2.6, z)
  worldGroup!.add(pole)

  const light = new THREE.Mesh(
    new THREE.SphereGeometry(0.32, 16, 16),
    new THREE.MeshBasicMaterial({
      color: '#ffcf78',
      transparent: true,
      opacity: 0.88,
    }),
  )
  light.position.set(x, 5.25, z)
  worldGroup!.add(light)

  const point = new THREE.PointLight('#ffcf78', 0.32, 12)
  point.position.set(x, 5.25, z)
  worldGroup!.add(point)
}

function createStreetContractionGuides() {
  const leftGuide: THREE.Vector3[] = []
  const rightGuide: THREE.Vector3[] = []

  for (let i = 0; i <= 120; i++) {
    const t = i / 120
    const z = -70 + t * 140
    const half = getCityStreetHalfWidth(z)

    leftGuide.push(new THREE.Vector3(-half, 0.72, z))
    rightGuide.push(new THREE.Vector3(half, 0.72, z))
  }

  const guideMaterial = new THREE.LineBasicMaterial({
    color: '#18d8ff',
    transparent: true,
    opacity: 0.3,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
  })

  worldGroup!.add(new THREE.Line(new THREE.BufferGeometry().setFromPoints(leftGuide), guideMaterial.clone()))
  worldGroup!.add(new THREE.Line(new THREE.BufferGeometry().setFromPoints(rightGuide), guideMaterial.clone()))

  const centerMarker = new THREE.Mesh(
    new THREE.TorusGeometry(getCityStreetHalfWidth(0) + 1.3, 0.07, 8, 128),
    new THREE.MeshBasicMaterial({
      color: '#ffb35c',
      transparent: true,
      opacity: 0.72,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )
  centerMarker.rotation.x = Math.PI / 2
  centerMarker.scale.z = 0.42
  centerMarker.position.set(0, 0.9, 0)
  worldGroup!.add(centerMarker)
}

function createMountainTrees() {
  const zPositions = [-56, -44, -32, 34, 48, 62]
  for (const z of zPositions) {
    for (const side of [-1, 1]) {
      const x = side * (getHalfValleyWidth(z) + 3.5 + deterministicNoise(z, side) * 2)
      const y = getTerrainHeight(x, z)
      const tree = createTreeModel(1.3 + deterministicNoise(z * 2, side) * 0.35) as InteractiveObject
      tree.position.set(x, y, z + (deterministicNoise(z * 0.4, side) - 0.5) * 4)
      tree.rotation.y = deterministicNoise(z * 0.7, side) * Math.PI
      tree.userData.type = 'tree'
      tree.userData.title = '树木风速参照物'
      tree.userData.text = '树木摇晃幅度用于对比风速。上游较弱，狭口附近最强，下游仍保持一段较强摆动。'
      tree.userData.baseScale = tree.scale.clone()

      treeGroup!.add(tree)
      interactiveObjects.push(tree)

      const crown = tree.children[1] ?? tree
      treeRefs.push({
        // @ts-ignore
        group: tree,
        trunk: tree.children[0] as THREE.Mesh,
        crown,
        baseRotationZ: 0,
        windZ: z,
        side,
      })
    }
  }
}

function createCityTrees() {
  const zPositions = [-60, -48, -34, -22, 22, 34, 48, 60]
  for (const z of zPositions) {
    for (const side of [-1, 1]) {
      const streetHalf = getCityStreetHalfWidth(z)
      const x = side * (streetHalf + 2.7)
      const tree = createTreeModel(1.0 + deterministicNoise(z, side) * 0.25) as InteractiveObject
      tree.position.set(x, 0.2, z + (deterministicNoise(z * 0.4, side) - 0.5) * 3)
      tree.rotation.y = deterministicNoise(z * 0.8, side) * Math.PI
      tree.userData.type = 'tree'
      tree.userData.title = '街道树参照物'
      tree.userData.text = '城市街谷中，楼宇间的风速增强会让街道树明显摇晃。越靠近楼宇夹道狭窄区，摇晃越强。'
      tree.userData.baseScale = tree.scale.clone()

      treeGroup!.add(tree)
      interactiveObjects.push(tree)

      const crown = tree.children[1] ?? tree
      treeRefs.push({
        // @ts-ignore
        group: tree,
        trunk: tree.children[0] as THREE.Mesh,
        crown,
        baseRotationZ: 0,
        windZ: z,
        side,
      })
    }
  }
}

function createTreeModel(scale = 1) {
  const tree = new THREE.Group()

  const trunk = new THREE.Mesh(
    new THREE.CylinderGeometry(0.18 * scale, 0.24 * scale, 2.4 * scale, 10),
    new THREE.MeshStandardMaterial({
      color: '#5a3a24',
      roughness: 0.9,
      metalness: 0.02,
    }),
  )
  trunk.position.y = 1.2 * scale
  trunk.castShadow = true
  tree.add(trunk)

  const crown = new THREE.Group()

  const crownMaterial = new THREE.MeshStandardMaterial({
    color: '#2f6b39',
    roughness: 0.85,
    metalness: 0.02,
  })

  const c1 = new THREE.Mesh(new THREE.ConeGeometry(1.05 * scale, 2.4 * scale, 12), crownMaterial)
  c1.position.y = 3.0 * scale

  const c2 = new THREE.Mesh(new THREE.ConeGeometry(0.85 * scale, 2.0 * scale, 12), crownMaterial)
  c2.position.y = 3.9 * scale

  const c3 = new THREE.Mesh(new THREE.ConeGeometry(0.65 * scale, 1.55 * scale, 12), crownMaterial)
  c3.position.y = 4.65 * scale

  c1.castShadow = true
  c2.castShadow = true
  c3.castShadow = true

  crown.add(c1, c2, c3)
  tree.add(crown)

  return tree
}

function createMountainHotspots() {
  addHotspot(
    new THREE.Vector3(0, getTerrainHeight(0, 0) + 8, 0),
    '狭口加速区',
    '这里是通道最窄的位置。风线在此处最密集，流速最高，是狭管效应最明显的区域。',
  )

  addHotspot(new THREE.Vector3(0, getTerrainHeight(0, -58) + 7, -58), '上游宽谷', '上游通道较宽，空气运动空间更大，流线分散，风速相对稳定。')

  addHotspot(
    new THREE.Vector3(0, getTerrainHeight(0, 54) + 7, 54),
    '下游扩散区',
    '风穿过狭口后不会马上变慢，而是保持一段较强风速，再随通道变宽逐渐扩散。',
  )
}

function createCityHotspots() {
  addHotspot(new THREE.Vector3(0, 12, 0), '楼宇夹道加速区', '两侧楼宇向街道中心挤压，街谷宽度在中部最窄，风线被迫收束并加速。')

  addHotspot(new THREE.Vector3(0, 10, -58), '上风街区', '风从较宽的街道入口进入，流线相对分散，速度较稳定。')

  addHotspot(new THREE.Vector3(0, 10, 56), '下风街区', '气流穿过夹道后仍保持较强速度，会继续影响下风向街区行人、树木和设施。')
}

function addHotspot(position: THREE.Vector3, title: string, text: string) {
  const marker = new THREE.Mesh(
    new THREE.SphereGeometry(1.4, 18, 18),
    new THREE.MeshBasicMaterial({
      color: '#ffb35c',
      transparent: true,
      opacity: 0.18,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  ) as InteractiveObject

  marker.position.copy(position)
  marker.userData.type = 'hotspot'
  marker.userData.title = title
  marker.userData.text = text
  marker.userData.baseScale = marker.scale.clone()

  hotspotGroup!.add(marker)
  interactiveObjects.push(marker)

  const ring = new THREE.Mesh(
    new THREE.TorusGeometry(2.5, 0.05, 8, 80),
    new THREE.MeshBasicMaterial({
      color: '#18d8ff',
      transparent: true,
      opacity: 0.38,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )
  ring.position.copy(position)
  ring.rotation.x = Math.PI / 2
  hotspotGroup!.add(ring)
}

function createWindField() {
  disposeGroup(windGroup)

  windGroup = new THREE.Group()
  scene.add(windGroup)

  smokeLines = []

  for (let i = 0; i < WIND_LINE_COUNT; i++) {
    const sideT = i / Math.max(1, WIND_LINE_COUNT - 1)
    const widthOffset = THREE.MathUtils.lerp(-1, 1, sideT)
    const heightOffset = deterministicNoise(i * 13.7, sceneMode.value === 'city' ? 2 : 1) * 1.1 + 0.2
    const progressSeed = deterministicNoise(i * 19.3, sceneMode.value === 'city' ? 8 : 4)

    const curve =
      sceneMode.value === 'mountain' ? createMountainSmokeCurve(widthOffset, heightOffset) : createCitySmokeCurve(widthOffset, heightOffset)

    const group = new THREE.Group()
    windGroup.add(group)

    const line: SmokeLine = {
      group,
      curve,
      widthOffset,
      heightOffset,
      progress: getInitialProgress(i),
      progressSeed,
      tails: [],
    }

    buildSmokeLineGeometry(line)
    smokeLines.push(line)
  }
}

function createMountainSmokeCurve(widthOffset: number, heightOffset: number) {
  const points: THREE.Vector3[] = []

  const phase = deterministicNoise(widthOffset * 10, heightOffset) * Math.PI * 2
  const phase2 = deterministicNoise(widthOffset * 20, heightOffset + 2) * Math.PI * 2
  const phase3 = deterministicNoise(widthOffset * 30, heightOffset + 3) * Math.PI * 2

  const compressedOffset = Math.sign(widthOffset) * Math.pow(Math.abs(widthOffset), 1.45)

  for (let i = 0; i <= 210; i++) {
    const t = i / 210
    const z = -70 + t * 140

    const halfWidth = getHalfValleyWidth(z)
    const safeHalfWidth = Math.max(0.65, halfWidth * 0.3)
    const throatLock = Math.exp(-(z * z) / 520)

    const baseX = compressedOffset * safeHalfWidth * 0.86

    const maxWave = Math.max(0.08, safeHalfWidth * 0.12) * (1 - throatLock * 0.88)

    const largeWave = Math.sin(t * Math.PI * 4.6 + phase) * maxWave * WIND_CURVE_POWER

    const midWave = Math.sin(t * Math.PI * 8.2 + phase2) * maxWave * 0.32 * WIND_CURVE_POWER

    const fineWave = Math.sin(t * Math.PI * 13.8 + phase3) * maxWave * 0.08 * WIND_CURVE_POWER

    let x = baseX + largeWave + midWave + fineWave
    x = THREE.MathUtils.clamp(x, -safeHalfWidth, safeHalfWidth)

    const terrainY = getTerrainHeight(x, z)

    const yWave = Math.sin(t * Math.PI * 3.2 + phase) * 0.22 + Math.sin(t * Math.PI * 7.5 + phase2) * 0.08

    const y = terrainY + WIND_HEIGHT_MOUNTAIN + heightOffset * 2.0 + yWave

    const safeY = Math.max(y, terrainY + MIN_SAFE_AIR_HEIGHT)

    points.push(new THREE.Vector3(x, safeY, z))
  }

  return new THREE.CatmullRomCurve3(points, false, 'catmullrom', 0.35)
}

function createCitySmokeCurve(widthOffset: number, heightOffset: number) {
  const points: THREE.Vector3[] = []

  const phase = deterministicNoise(widthOffset * 15, heightOffset) * Math.PI * 2
  const phase2 = deterministicNoise(widthOffset * 22, heightOffset + 1) * Math.PI * 2
  const compressedOffset = Math.sign(widthOffset) * Math.pow(Math.abs(widthOffset), 1.3)

  for (let i = 0; i <= 230; i++) {
    const t = i / 230
    const z = -76 + t * 152

    const streetHalf = getCityStreetHalfWidth(z)
    const safeHalfWidth = Math.max(0.7, streetHalf * 0.68)

    const contraction = getCityContractionAmount(z)
    const baseX = compressedOffset * safeHalfWidth * 0.88

    const waveMax = Math.max(0.06, safeHalfWidth * 0.11) * (1 - contraction * 0.78)

    const wave = Math.sin(t * Math.PI * 4.3 + phase) * waveMax * 0.82 + Math.sin(t * Math.PI * 8.8 + phase2) * waveMax * 0.24

    let x = baseX + wave
    x = THREE.MathUtils.clamp(x, -safeHalfWidth, safeHalfWidth)

    const y = WIND_HEIGHT_CITY + heightOffset * 2.1 + Math.sin(t * Math.PI * 3.5 + phase) * 0.18

    points.push(new THREE.Vector3(x, y, z))
  }

  return new THREE.CatmullRomCurve3(points, false, 'catmullrom', 0.35)
}

function buildSmokeLineGeometry(line: SmokeLine) {
  line.group.clear()
  line.tails = []

  const tailCount = 4
  const pointCount = 40

  for (let i = 0; i < tailCount; i++) {
    const positions = new Float32Array(pointCount * 3)

    const geometry = new THREE.BufferGeometry()
    geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
    geometry.boundingSphere = new THREE.Sphere(new THREE.Vector3(0, 20, 0), 280)

    const material = new THREE.LineBasicMaterial({
      color: '#ffffff',
      transparent: true,
      opacity: THREE.MathUtils.lerp(0.018, 0.18, 1 - i / tailCount),
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    })

    const smokeLine = new THREE.Line(geometry, material)
    smokeLine.renderOrder = 4
    line.group.add(smokeLine)

    line.tails.push({
      line: smokeLine,
      geometry,
      material,
      positions,
      delay: i * 0.018,
      length: 0.12 + i * 0.012,
      pointCount,
    })
  }

  updateSmokeLineGeometry(line, 0, true)
}

function updateSmokeLineGeometry(line: SmokeLine, delta: number, force = false) {
  const currentPoint = line.curve.getPointAt(line.progress)
  const speedFactor = getChannelSpeedFactor(currentPoint.z)

  if (demoActive.value && pageVisible.value) {
    line.progress += delta * BASE_WIND_SPEED * speedFactor * (0.9 + line.progressSeed * 0.22)

    if (line.progress > 0.86) {
      line.progress = line.progressSeed * 0.08
    }
  }

  if (!demoActive.value && !force) {
    return
  }

  for (let i = 0; i < line.tails.length; i++) {
    const tail = line.tails[i]!

    const start = Math.max(0, line.progress - tail.delay)
    const segmentLength = tail.length + speedFactor * 0.006
    const end = Math.min(0.985, start + segmentLength)
    const actualLength = Math.max(0.001, end - start)

    let lastSpeedFactor = speedFactor

    for (let j = 0; j < tail.pointCount; j++) {
      const k = j / Math.max(1, tail.pointCount - 1)
      const t = start + k * actualLength

      const point = line.curve.getPointAt(t)

      let safeY = point.y

      if (sceneMode.value === 'mountain') {
        const terrainY = getTerrainHeight(point.x, point.z)
        safeY = Math.max(point.y, terrainY + MIN_SAFE_AIR_HEIGHT)
      }

      lastSpeedFactor = getChannelSpeedFactor(point.z)

      tail.positions[j * 3] = point.x
      tail.positions[j * 3 + 1] = safeY
      tail.positions[j * 3 + 2] = point.z
    }

    const attr = tail.geometry.getAttribute('position') as THREE.BufferAttribute
    attr.needsUpdate = true

    const color = getWindColor(lastSpeedFactor)
    tail.material.color.copy(color)

    tail.material.opacity = THREE.MathUtils.clamp(THREE.MathUtils.lerp(0.014, 0.15, 1 - i / line.tails.length) + lastSpeedFactor * 0.01, 0.014, 0.25)
  }
}

function getWindColor(speedFactor: number) {
  const low = new THREE.Color('#ffffff')
  const mid = new THREE.Color('#18d8ff')
  const high = new THREE.Color('#ffb35c')

  if (speedFactor < 2.4) {
    return low.lerp(mid, THREE.MathUtils.clamp((speedFactor - 1) / 1.4, 0, 1))
  }

  return mid.lerp(high, THREE.MathUtils.clamp((speedFactor - 2.4) / 2.8, 0, 1))
}

function updateWindField(delta: number) {
  for (const line of smokeLines) {
    updateSmokeLineGeometry(line, delta)
  }
}

function updateTrees(elapsed: number) {
  for (const tree of treeRefs) {
    const factor = getChannelSpeedFactor(tree.windZ)
    const normalized = THREE.MathUtils.clamp((factor - 1) / 4.8, 0.08, 1)

    const baseAmp = sceneMode.value === 'city' ? 0.14 : 0.1
    const amp = demoActive.value ? baseAmp * normalized : 0.015
    const sway = Math.sin(elapsed * (2.1 + normalized * 2.6) + tree.windZ * 0.08 + tree.side) * amp

    tree.group!.rotation.z = tree.baseRotationZ + sway * tree.side
    tree.group!.rotation.x = Math.sin(elapsed * 1.7 + tree.windZ * 0.03) * amp * 0.28
  }
}

function handlePointerMove(event: MouseEvent) {
  if (!containerRef.value || !camera || !raycaster) return

  const rect = containerRef.value.getBoundingClientRect()
  pointer.x = ((event.clientX - rect.left) / rect.width) * 2 - 1
  pointer.y = -((event.clientY - rect.top) / rect.height) * 2 + 1

  raycaster.setFromCamera(pointer, camera)

  const hits = raycaster.intersectObjects(interactiveObjects, true)

  if (!hits.length) {
    clearHoverState()
    return
  }

  const hit = findInteractiveParent(hits[0]!.object)
  if (!hit) {
    clearHoverState()
    return
  }

  setHoverObject(hit, event.clientX, event.clientY)
}

function handlePointerLeave() {
  clearHoverState()
}

function handleSceneClick() {
  if (!lastHoveredObject) return

  infoPopup.visible = true
  infoPopup.title = lastHoveredObject.userData.title || '狭管效应知识点'
  infoPopup.text = lastHoveredObject.userData.text || '该位置与狭管效应有关。'
}

function findInteractiveParent(object: THREE.Object3D): InteractiveObject | null {
  let current: THREE.Object3D | null = object

  while (current) {
    if (interactiveObjects.includes(current as InteractiveObject)) {
      return current as InteractiveObject
    }

    current = current.parent
  }

  return null
}

function setHoverObject(object: InteractiveObject, x: number, y: number) {
  if (lastHoveredObject && lastHoveredObject !== object) {
    restoreObjectScale(lastHoveredObject)
  }

  lastHoveredObject = object

  if (object.userData.baseScale) {
    object.scale.copy(object.userData.baseScale).multiplyScalar(1.08)
  }

  hoverInfo.visible = true
  hoverInfo.x = x + 14
  hoverInfo.y = y + 14
  hoverInfo.title = object.userData.title || '狭管效应提示'
  hoverInfo.text = object.userData.text || '该位置用于说明狭管效应。'

  if (focusRing) {
    const worldPosition = new THREE.Vector3()
    object.getWorldPosition(worldPosition)
    focusRing.position.set(worldPosition.x, Math.max(0.8, worldPosition.y), worldPosition.z)
    focusRing.scale.setScalar(object.userData.type === 'building' ? 1.35 : 1)
    ;(focusRing.material as THREE.MeshBasicMaterial).opacity = 0.72
  }
}

function clearHoverState() {
  if (lastHoveredObject) {
    restoreObjectScale(lastHoveredObject)
    lastHoveredObject = null
  }

  hoverInfo.visible = false

  if (focusRing) {
    ;(focusRing.material as THREE.MeshBasicMaterial).opacity = 0
  }
}

function restoreObjectScale(object: InteractiveObject) {
  if (object.userData.baseScale) {
    object.scale.copy(object.userData.baseScale)
  }
}

function closePopup() {
  infoPopup.visible = false
}

function getHalfValleyWidth(z: number) {
  const wideHalf = 27
  const throatHalf = params.throatWidth / 2
  const throatInfluence = Math.exp(-(z * z) / 860)

  return THREE.MathUtils.lerp(wideHalf, throatHalf, throatInfluence)
}

function getCityStreetHalfWidth(z: number) {
  const wideHalf = params.throatWidth * 0.5 + 6.5
  const narrowHalf = params.throatWidth * 0.5
  const contraction = getCityContractionAmount(z)

  return THREE.MathUtils.lerp(wideHalf, narrowHalf, contraction)
}

function getCityContractionAmount(z: number) {
  return Math.exp(-(z * z) / 980)
}

function getChannelSpeedFactor(z: number) {
  const widthBoost = THREE.MathUtils.clamp(22 / params.throatWidth, 1.1, 5.2)
  const throatBoost = Math.exp(-(z * z) / 620) * widthBoost

  const downstreamBoost = z > 0 ? Math.exp(-Math.pow(z - 22, 2) / 1250) * widthBoost * 0.62 : 0

  return 1 + throatBoost + downstreamBoost
}

function getMaxSpeedFactor() {
  return getChannelSpeedFactor(0)
}

function getInitialProgress(index: number) {
  const base = index / Math.max(1, WIND_LINE_COUNT)
  const jitter = deterministicNoise(index * 9.17, sceneMode.value === 'city' ? 5.2 : 2.8) * 0.18

  return normalizeProgress((base * 0.78 + jitter) % 0.82)
}

function normalizeProgress(value: number) {
  let result = value

  while (result < 0) result += 0.82
  while (result > 0.86) result -= 0.82

  return THREE.MathUtils.clamp(result, 0, 0.86)
}

function createLabels() {
  disposeGroup(labelGroup)

  labelGroup = new THREE.Group()
  scene.add(labelGroup)

  if (sceneMode.value === 'mountain') {
    addSpriteLabel('上游宽谷：风速较稳定', new THREE.Vector3(0, 42, -62), '#18d8ff')
    addSpriteLabel('狭口区域：流线收束、风速增强', new THREE.Vector3(0, 52, 0), '#ffb35c')
    addSpriteLabel('出谷下游：强风延续后逐渐扩散', new THREE.Vector3(0, 42, 62), '#ffffff')
  } else {
    addSpriteLabel('上风街区：气流分散进入', new THREE.Vector3(0, 36, -62), '#18d8ff')
    addSpriteLabel('楼宇夹道：流线收束、街谷风增强', new THREE.Vector3(0, 48, 0), '#ffb35c')
    addSpriteLabel('下风街区：强风沿街道延续', new THREE.Vector3(0, 36, 62), '#ffffff')
  }
}

function addSpriteLabel(text: string, position: THREE.Vector3, color: string) {
  const canvas = document.createElement('canvas')
  canvas.width = 1200
  canvas.height = 260

  const ctx = canvas.getContext('2d')!
  ctx.clearRect(0, 0, canvas.width, canvas.height)

  const gradient = ctx.createLinearGradient(0, 0, canvas.width, 0)
  gradient.addColorStop(0, 'rgba(24, 216, 255, 0.04)')
  gradient.addColorStop(0.5, 'rgba(8, 23, 42, 0.92)')
  gradient.addColorStop(1, 'rgba(255, 179, 92, 0.08)')

  ctx.fillStyle = gradient
  drawRoundRect(ctx, 38, 48, 1124, 140, 28)
  ctx.fill()

  ctx.strokeStyle = color
  ctx.lineWidth = 5
  drawRoundRect(ctx, 38, 48, 1124, 140, 28)
  ctx.stroke()

  ctx.font = 'bold 58px Microsoft YaHei, Arial'
  ctx.fillStyle = color
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.shadowColor = color
  ctx.shadowBlur = 24
  ctx.fillText(text, 600, 118)

  const texture = new THREE.CanvasTexture(canvas)
  texture.colorSpace = THREE.SRGBColorSpace

  const material = new THREE.SpriteMaterial({
    map: texture,
    transparent: true,
    depthWrite: false,
  })

  const sprite = new THREE.Sprite(material)
  sprite.position.copy(position)
  sprite.scale.set(38, 8.2, 1)

  labelGroup!.add(sprite)
}

function drawRoundRect(ctx: CanvasRenderingContext2D, x: number, y: number, w: number, h: number, r: number) {
  ctx.beginPath()
  ctx.moveTo(x + r, y)
  ctx.lineTo(x + w - r, y)
  ctx.quadraticCurveTo(x + w, y, x + w, y + r)
  ctx.lineTo(x + w, y + h - r)
  ctx.quadraticCurveTo(x + w, y + h, x + w - r, y + h)
  ctx.lineTo(x + r, y + h)
  ctx.quadraticCurveTo(x, y + h, x, y + h - r)
  ctx.lineTo(x, y + r)
  ctx.quadraticCurveTo(x, y, x + r, y)
  ctx.closePath()
}

function startDemo() {
  if (demoActive.value) return

  demoActive.value = true
  clock.start()
  clock.getDelta()

  clearInterval(stageTimer)
  stageTimer = window.setInterval(() => {
    autoStage()
  }, 4500)
}

function stopDemo() {
  demoActive.value = false
  clearInterval(stageTimer)
}

function autoStage() {
  const order: StageKey[] = ['inflow', 'narrow', 'accelerate', 'outflow']
  const currentIndex = order.indexOf(currentStage.value)
  const nextIndex = (currentIndex + 1) % order.length
  currentStage.value = order[nextIndex]!
}

function setStage(stage: StageKey) {
  currentStage.value = stage
}

function resetCamera() {
  if (sceneMode.value === 'mountain') {
    smoothCameraTo(new THREE.Vector3(0, 74, 172), new THREE.Vector3(0, 14, 0))
  } else {
    smoothCameraTo(new THREE.Vector3(0, 78, 166), new THREE.Vector3(0, 18, 0))
  }
}

function smoothCameraTo(position: THREE.Vector3, target: THREE.Vector3) {
  const startPos = camera.position.clone()
  const startTarget = controls.target.clone()

  let t = 0

  function update() {
    t += 0.025
    const k = 1 - Math.pow(1 - Math.min(t, 1), 3)

    camera.position.lerpVectors(startPos, position, k)
    controls.target.lerpVectors(startTarget, target, k)

    if (t < 1) {
      requestAnimationFrame(update)
    }
  }

  update()
}

function animate() {
  animationId = requestAnimationFrame(animate)

  let delta = clock.getDelta()
  const elapsed = clock.elapsedTime

  delta = Math.min(delta, 0.033)

  if (demoActive.value && pageVisible.value) {
    updateWindField(delta)
    animateDecorations(elapsed)
  }

  updateTrees(elapsed)
  animateFocusRing(elapsed)

  controls.update()
  renderer.render(scene, camera)
}

function animateFocusRing(elapsed: number) {
  if (!focusRing) return

  const material = focusRing.material as THREE.MeshBasicMaterial

  if (material.opacity > 0) {
    focusRing.rotation.z = elapsed * 1.5
    const scale = 1 + Math.sin(elapsed * 4) * 0.06
    focusRing.scale.setScalar(scale * (lastHoveredObject?.userData.type === 'building' ? 1.35 : 1))
  }
}

function animateDecorations(elapsed: number) {
  if (!decorationGroup) return

  decorationGroup.rotation.y = elapsed * 0.018

  decorationGroup.children.forEach((child, index) => {
    const mesh = child as THREE.Mesh
    const material = mesh.material as THREE.Material | undefined

    if (material && 'opacity' in material) {
      const basic = material as THREE.MeshBasicMaterial
      basic.opacity = THREE.MathUtils.clamp(0.07 + Math.sin(elapsed * 1.3 + index) * 0.035, 0.025, 0.2)
    }
  })
}

function handleResize() {
  if (!containerRef.value) return

  const width = containerRef.value.clientWidth
  const height = containerRef.value.clientHeight

  camera.aspect = width / height
  camera.updateProjectionMatrix()

  renderer.setSize(width, height)
  drawSpeedChart()
}

function disposeGroup(group: THREE.Group | null) {
  if (!group) return

  group.traverse(obj => {
    const mesh = obj as THREE.Mesh

    if (mesh.geometry) {
      mesh.geometry.dispose()
    }

    const material = mesh.material as THREE.Material | THREE.Material[] | undefined

    if (Array.isArray(material)) {
      material.forEach(m => disposeMaterial(m))
    } else if (material) {
      disposeMaterial(material)
    }
  })

  group.removeFromParent()
}

function disposeMaterial(material: THREE.Material) {
  const mat = material as THREE.Material & {
    map?: THREE.Texture
    alphaMap?: THREE.Texture
  }

  if (mat.map) {
    mat.map.dispose()
  }

  if (mat.alphaMap) {
    mat.alphaMap.dispose()
  }

  material.dispose()
}

function deterministicNoise(x: number, y: number) {
  const v = Math.sin(x * 12.9898 + y * 78.233) * 43758.5453123
  return v - Math.floor(v)
}

function noise2D(x: number, y: number) {
  return deterministicNoise(x, y) * 2 - 1
}

function smoothNoise2D(x: number, y: number) {
  const ix = Math.floor(x)
  const iy = Math.floor(y)
  const fx = x - ix
  const fy = y - iy

  const a = noise2D(ix, iy)
  const b = noise2D(ix + 1, iy)
  const c = noise2D(ix, iy + 1)
  const d = noise2D(ix + 1, iy + 1)

  const ux = fx * fx * (3 - 2 * fx)
  const uy = fy * fy * (3 - 2 * fy)

  return THREE.MathUtils.lerp(THREE.MathUtils.lerp(a, b, ux), THREE.MathUtils.lerp(c, d, ux), uy)
}

function fbm(x: number, y: number, octaves = 4) {
  let value = 0
  let amplitude = 0.5
  let frequency = 1

  for (let i = 0; i < octaves; i++) {
    value += smoothNoise2D(x * frequency, y * frequency) * amplitude
    frequency *= 2
    amplitude *= 0.5
  }

  return value
}
</script>

<style scoped>
.canyon-page {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
  color: #eafaff;
  background:
    radial-gradient(circle at 50% 28%, rgba(24, 216, 255, 0.22), transparent 30%),
    radial-gradient(circle at 18% 70%, rgba(46, 108, 255, 0.18), transparent 28%),
    radial-gradient(circle at 84% 68%, rgba(255, 179, 92, 0.12), transparent 30%), linear-gradient(135deg, #040b14 0%, #0b1d2f 42%, #02060d 100%);
  font-family:
    Microsoft YaHei,
    PingFang SC,
    Arial,
    sans-serif;
}

.three-container {
  position: absolute;
  inset: 0;
}

.scan-line {
  position: absolute;
  inset: 0;
  z-index: 4;
  pointer-events: none;
  background: linear-gradient(to bottom, transparent 0%, rgba(24, 216, 255, 0.05) 50%, transparent 100%);
  background-size: 100% 7px;
  mix-blend-mode: screen;
  opacity: 0.5;
}

.holo-grid {
  position: absolute;
  inset: 0;
  z-index: 3;
  pointer-events: none;
  background-image:
    linear-gradient(rgba(24, 216, 255, 0.035) 1px, transparent 1px), linear-gradient(90deg, rgba(255, 179, 92, 0.02) 1px, transparent 1px);
  background-size: 48px 48px;
  mask-image: radial-gradient(circle at center, rgba(0, 0, 0, 0.72), transparent 72%);
  opacity: 0.36;
}

.corner {
  position: absolute;
  z-index: 12;
  width: 86px;
  height: 86px;
  pointer-events: none;
  border-color: rgba(24, 216, 255, 0.9);
  filter: drop-shadow(0 0 10px rgba(24, 216, 255, 0.5));
}

.corner-left-top {
  top: 18px;
  left: 18px;
  border-top: 2px solid;
  border-left: 2px solid;
}

.corner-right-top {
  top: 18px;
  right: 18px;
  border-top: 2px solid;
  border-right: 2px solid;
}

.corner-left-bottom {
  bottom: 18px;
  left: 18px;
  border-bottom: 2px solid;
  border-left: 2px solid;
}

.corner-right-bottom {
  right: 18px;
  bottom: 18px;
  border-right: 2px solid;
  border-bottom: 2px solid;
}

.toggle-panel-btn {
  position: absolute;
  top: 30px;
  right: 30px;
  z-index: 40;
  display: inline-flex;
  gap: 8px;
  align-items: center;
  height: 36px;
  padding: 0 15px;
  cursor: pointer;
  border: 1px solid rgba(24, 216, 255, 0.52);
  border-radius: 999px;
  color: #eafaff;
  font-size: 13px;
  font-weight: 900;
  letter-spacing: 1px;
  background: linear-gradient(135deg, rgba(24, 216, 255, 0.18), rgba(46, 108, 255, 0.16)), rgba(4, 11, 20, 0.76);
  box-shadow:
    0 0 18px rgba(24, 216, 255, 0.2),
    inset 0 0 16px rgba(24, 216, 255, 0.08);
  backdrop-filter: blur(12px);
  transition:
    transform 0.22s ease,
    border-color 0.22s ease,
    box-shadow 0.22s ease;
}

.toggle-panel-btn:hover {
  transform: translateY(-2px);
  border-color: rgba(255, 179, 92, 0.76);
  box-shadow:
    0 0 22px rgba(24, 216, 255, 0.26),
    0 0 18px rgba(255, 179, 92, 0.13),
    inset 0 0 16px rgba(24, 216, 255, 0.1);
}

.toggle-panel-icon {
  color: #ffb35c;
  font-size: 15px;
  line-height: 1;
}

.title-panel {
  position: absolute;
  top: 16px;
  left: 50%;
  z-index: 15;
  width: min(760px, calc(100% - 48px));
  padding: 9px 24px 11px;
  transform: translateX(-50%);
  border: 1px solid rgba(24, 216, 255, 0.46);
  border-radius: 18px;
  background:
    linear-gradient(90deg, rgba(24, 216, 255, 0.1), rgba(8, 23, 42, 0.82) 50%, rgba(255, 179, 92, 0.06)),
    linear-gradient(180deg, rgba(8, 23, 42, 0.9), rgba(3, 10, 20, 0.7));
  box-shadow:
    0 0 30px rgba(24, 216, 255, 0.18),
    0 0 28px rgba(255, 179, 92, 0.06),
    inset 0 0 24px rgba(24, 216, 255, 0.07);
  text-align: center;
  backdrop-filter: blur(14px);
}

.title-badge {
  display: inline-block;
  padding: 2px 12px;
  margin-bottom: 4px;
  border: 1px solid rgba(24, 216, 255, 0.4);
  border-radius: 999px;
  color: rgba(24, 216, 255, 0.92);
  font-size: 10px;
  letter-spacing: 2px;
  background: rgba(24, 216, 255, 0.07);
}

.title-main {
  color: #ffffff;
  font-size: 24px;
  font-weight: 900;
  letter-spacing: 4px;
  text-shadow:
    0 0 10px rgba(24, 216, 255, 0.8),
    0 0 26px rgba(46, 108, 255, 0.3);
}

.title-sub {
  margin-top: 4px;
  color: rgba(234, 250, 255, 0.78);
  font-size: 12px;
}

.knowledge-panel,
.control-panel,
.bottom-panel {
  position: absolute;
  z-index: 15;
  border: 1px solid rgba(24, 216, 255, 0.42);
  background:
    linear-gradient(135deg, rgba(24, 216, 255, 0.11), transparent 34%), linear-gradient(225deg, rgba(255, 179, 92, 0.06), transparent 32%),
    linear-gradient(180deg, rgba(8, 23, 42, 0.88), rgba(3, 10, 20, 0.7));
  box-shadow:
    0 0 30px rgba(24, 216, 255, 0.18),
    0 0 24px rgba(46, 108, 255, 0.12),
    inset 0 0 28px rgba(24, 216, 255, 0.08);
  backdrop-filter: blur(16px);
}

.control-panel {
  top: 116px;
  left: 24px;
  width: 350px;
  padding: 18px;
  border-radius: 22px;
}

.knowledge-panel {
  top: 116px;
  right: 24px;
  width: 390px;
  max-height: calc(100vh - 188px);
  padding: 18px;
  overflow-y: auto;
  border-radius: 22px;
}

.bottom-panel {
  right: 438px;
  bottom: 24px;
  left: 404px;
  padding: 16px;
  border-radius: 22px;
}

.metric-only {
  display: block;
}

.full-metric {
  width: 100%;
}

.metric-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
}

.metric-sub {
  margin-top: 2px;
  color: rgba(234, 250, 255, 0.4);
  font-size: 10px;
  letter-spacing: 2px;
}

.metric-pulse {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #18d8ff;
  box-shadow:
    0 0 10px rgba(24, 216, 255, 0.8),
    0 0 22px rgba(24, 216, 255, 0.3);
}

.metric-pulse.strong {
  background: #ffb35c;
  box-shadow:
    0 0 10px rgba(255, 179, 92, 0.85),
    0 0 24px rgba(255, 179, 92, 0.36);
  animation: pulse 1s infinite;
}

.metric-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px 16px;
}

.speed-canvas {
  display: block;
  width: 100%;
  height: 76px;
  margin-top: 12px;
  border: 1px solid rgba(24, 216, 255, 0.13);
  border-radius: 12px;
}

.panel-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  margin-bottom: 14px;
}

.panel-title {
  color: #18d8ff;
  font-size: 18px;
  font-weight: 900;
  letter-spacing: 2px;
  text-shadow: 0 0 10px rgba(24, 216, 255, 0.42);
}

.panel-en {
  margin-top: 2px;
  color: rgba(255, 179, 92, 0.58);
  font-size: 10px;
  letter-spacing: 2px;
}

.panel-dot {
  width: 10px;
  height: 10px;
  margin-top: 5px;
  border-radius: 50%;
  background: #ffb35c;
  box-shadow:
    0 0 10px #ffb35c,
    0 0 18px rgba(24, 216, 255, 0.6);
}

.section-title {
  margin: 12px 0 10px;
  color: #ffb35c;
  font-size: 14px;
  font-weight: 900;
  letter-spacing: 1px;
  text-shadow: 0 0 10px rgba(255, 179, 92, 0.42);
}

.process-item {
  display: flex;
  gap: 12px;
  padding: 12px;
  margin-bottom: 10px;
  cursor: pointer;
  border: 1px solid rgba(24, 216, 255, 0.18);
  border-radius: 15px;
  background: rgba(8, 23, 42, 0.56);
  transition: 0.25s;
}

.process-item:hover {
  border-color: rgba(24, 216, 255, 0.66);
  transform: translateX(-4px);
  box-shadow:
    0 0 14px rgba(24, 216, 255, 0.14),
    inset 0 0 12px rgba(24, 216, 255, 0.07);
}

.process-item.active {
  border-color: rgba(255, 179, 92, 0.9);
  background: linear-gradient(90deg, rgba(255, 179, 92, 0.14), rgba(24, 216, 255, 0.07));
  box-shadow:
    0 0 18px rgba(255, 179, 92, 0.2),
    inset 0 0 16px rgba(24, 216, 255, 0.08);
}

.process-index {
  display: flex;
  flex: 0 0 30px;
  width: 30px;
  height: 30px;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, #18d8ff, #2e6cff);
  color: #040b14;
  font-weight: 900;
  box-shadow:
    0 0 12px rgba(24, 216, 255, 0.45),
    0 0 16px rgba(46, 108, 255, 0.28);
}

.process-name {
  color: #fff;
  font-size: 15px;
  font-weight: 900;
}

.process-desc {
  margin-top: 4px;
  color: rgba(234, 250, 255, 0.7);
  font-size: 12px;
  line-height: 1.55;
}

.divider {
  height: 1px;
  margin: 16px 0;
  background: linear-gradient(90deg, transparent, rgba(24, 216, 255, 0.65), rgba(255, 179, 92, 0.48), transparent);
}

.point {
  margin-bottom: 10px;
  color: rgba(234, 250, 255, 0.78);
  font-size: 13px;
  line-height: 1.72;
}

.point span {
  color: #18d8ff;
  font-weight: 900;
}

.judge-box,
.explain-card {
  padding: 12px;
  border: 1px solid rgba(255, 179, 92, 0.25);
  border-radius: 14px;
  color: rgba(234, 250, 255, 0.78);
  font-size: 13px;
  line-height: 1.65;
  background: rgba(255, 179, 92, 0.07);
}

.explain-card {
  margin-top: 10px;
  margin-bottom: 10px;
}

.demo-buttons,
.mode-switch {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.mode-switch {
  margin-top: 12px;
}

.demo-btn,
.mode-btn {
  height: 42px;
  cursor: pointer;
  border-radius: 14px;
  color: #fff;
  font-weight: 900;
  letter-spacing: 2px;
  transition: 0.25s;
}

.mode-btn {
  border: 1px solid rgba(24, 216, 255, 0.3);
  background: rgba(8, 23, 42, 0.62);
}

.mode-btn.active {
  border-color: rgba(255, 179, 92, 0.86);
  background: linear-gradient(135deg, rgba(24, 216, 255, 0.22), rgba(255, 179, 92, 0.18));
  box-shadow: 0 0 14px rgba(255, 179, 92, 0.16);
}

.demo-btn.start {
  border: 1px solid rgba(24, 216, 255, 0.72);
  background: linear-gradient(135deg, rgba(24, 216, 255, 0.32), rgba(46, 108, 255, 0.26));
  box-shadow:
    0 0 16px rgba(24, 216, 255, 0.2),
    inset 0 0 12px rgba(24, 216, 255, 0.09);
}

.demo-btn.stop {
  border: 1px solid rgba(255, 179, 92, 0.66);
  background: linear-gradient(135deg, rgba(255, 179, 92, 0.26), rgba(46, 108, 255, 0.18));
  box-shadow:
    0 0 16px rgba(255, 179, 92, 0.14),
    inset 0 0 12px rgba(255, 179, 92, 0.08);
}

.demo-btn:hover,
.mode-btn:hover {
  transform: translateY(-2px);
}

.demo-btn.disabled {
  opacity: 0.48;
  cursor: not-allowed;
  transform: none;
}

.demo-status {
  padding: 12px;
  margin-top: 14px;
  border: 1px solid rgba(24, 216, 255, 0.18);
  border-radius: 14px;
  background: rgba(8, 23, 42, 0.5);
}

.status-line,
.metric-row {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 8px;
  color: rgba(234, 250, 255, 0.74);
  font-size: 13px;
}

.status-line:last-child,
.metric-row:last-child {
  margin-bottom: 0;
}

.status-line strong,
.metric-row strong {
  color: #ffb35c;
}

.metric-row strong.highlight {
  color: #18d8ff;
  text-shadow: 0 0 10px rgba(24, 216, 255, 0.5);
}

.status-line strong.running {
  color: #18d8ff;
  text-shadow: 0 0 10px rgba(24, 216, 255, 0.75);
}

.control-row {
  display: grid;
  grid-template-columns: 92px 1fr 48px;
  gap: 10px;
  align-items: center;
  margin-bottom: 15px;
  color: rgba(234, 250, 255, 0.82);
  font-size: 13px;
}

.control-row input {
  width: 100%;
  accent-color: #18d8ff;
}

.control-row span {
  color: #ffb35c;
  text-align: right;
  font-weight: 900;
}

.reset-btn {
  width: 100%;
  height: 40px;
  margin-top: 8px;
  cursor: pointer;
  border: 1px solid rgba(24, 216, 255, 0.65);
  border-radius: 14px;
  background: linear-gradient(90deg, rgba(24, 216, 255, 0.22), rgba(46, 108, 255, 0.2), rgba(255, 179, 92, 0.12));
  color: #eafaff;
  font-weight: 900;
  letter-spacing: 2px;
  transition: 0.25s;
}

.reset-btn:hover {
  box-shadow:
    0 0 18px rgba(24, 216, 255, 0.26),
    0 0 18px rgba(255, 179, 92, 0.12);
  transform: translateY(-2px);
}

.metric-card {
  padding: 13px 14px;
  border: 1px solid rgba(24, 216, 255, 0.18);
  border-radius: 15px;
  background: rgba(8, 23, 42, 0.5);
}

.metric-title {
  color: #18d8ff;
  font-size: 15px;
  font-weight: 900;
}

.hover-tip {
  position: fixed;
  z-index: 50;
  max-width: 280px;
  padding: 10px 12px;
  border: 1px solid rgba(24, 216, 255, 0.42);
  border-radius: 12px;
  background: rgba(4, 11, 20, 0.86);
  box-shadow:
    0 0 18px rgba(24, 216, 255, 0.18),
    inset 0 0 18px rgba(24, 216, 255, 0.08);
  backdrop-filter: blur(12px);
  pointer-events: none;
}

.hover-title {
  color: #ffb35c;
  font-size: 13px;
  font-weight: 900;
  margin-bottom: 4px;
}

.hover-text {
  color: rgba(234, 250, 255, 0.78);
  font-size: 12px;
  line-height: 1.55;
}

.popup-mask {
  position: fixed;
  inset: 0;
  z-index: 80;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(1, 5, 12, 0.55);
  backdrop-filter: blur(8px);
}

.info-popup {
  position: relative;
  width: min(520px, calc(100% - 48px));
  padding: 24px;
  border: 1px solid rgba(24, 216, 255, 0.46);
  border-radius: 22px;
  background:
    linear-gradient(135deg, rgba(24, 216, 255, 0.12), transparent 32%), linear-gradient(180deg, rgba(8, 23, 42, 0.94), rgba(3, 10, 20, 0.88));
  box-shadow:
    0 0 34px rgba(24, 216, 255, 0.18),
    inset 0 0 28px rgba(24, 216, 255, 0.08);
}

.popup-close {
  position: absolute;
  top: 12px;
  right: 14px;
  width: 28px;
  height: 28px;
  border: 1px solid rgba(24, 216, 255, 0.28);
  border-radius: 50%;
  color: #eafaff;
  cursor: pointer;
  background: rgba(8, 23, 42, 0.8);
  font-size: 18px;
}

.popup-badge {
  display: inline-block;
  padding: 3px 12px;
  margin-bottom: 10px;
  border: 1px solid rgba(255, 179, 92, 0.38);
  border-radius: 999px;
  color: #ffb35c;
  font-size: 10px;
  letter-spacing: 2px;
}

.popup-title {
  color: #18d8ff;
  font-size: 22px;
  font-weight: 900;
  margin-bottom: 10px;
}

.popup-text {
  color: rgba(234, 250, 255, 0.82);
  font-size: 14px;
  line-height: 1.8;
}

.rebuild-tip {
  position: absolute;
  left: 50%;
  bottom: 128px;
  z-index: 30;
  transform: translateX(-50%);
  padding: 8px 18px;
  border: 1px solid rgba(24, 216, 255, 0.42);
  border-radius: 999px;
  color: #eafaff;
  font-size: 13px;
  background: rgba(8, 23, 42, 0.78);
  box-shadow:
    0 0 16px rgba(24, 216, 255, 0.18),
    0 0 14px rgba(255, 179, 92, 0.08);
  backdrop-filter: blur(10px);
}

.knowledge-panel::-webkit-scrollbar,
.control-panel::-webkit-scrollbar {
  width: 6px;
}

.knowledge-panel::-webkit-scrollbar-thumb,
.control-panel::-webkit-scrollbar-thumb {
  border-radius: 999px;
  background: linear-gradient(180deg, #18d8ff, #ffb35c);
}

@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 0.75;
  }

  50% {
    transform: scale(1.28);
    opacity: 1;
  }

  100% {
    transform: scale(1);
    opacity: 0.75;
  }
}

@media (max-width: 960px) {
  .toggle-panel-btn {
    top: 14px;
    right: 16px;
    height: 32px;
    padding: 0 12px;
    font-size: 12px;
  }
}

@media (max-width: 1280px) {
  .bottom-panel {
    right: 24px;
    left: 24px;
  }

  .knowledge-panel {
    width: 350px;
  }

  .control-panel {
    width: 320px;
  }

  .metric-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 960px) {
  .title-main {
    font-size: 20px;
  }

  .title-sub {
    font-size: 12px;
  }

  .knowledge-panel,
  .control-panel,
  .bottom-panel {
    right: 16px;
    left: 16px;
    width: auto;
  }

  .control-panel {
    top: 104px;
    max-height: 260px;
    overflow-y: auto;
  }

  .knowledge-panel {
    top: auto;
    bottom: 246px;
    max-height: 240px;
  }

  .bottom-panel {
    bottom: 16px;
    max-height: 216px;
    overflow-y: auto;
  }

  .metric-grid {
    grid-template-columns: 1fr;
  }
}
</style>
