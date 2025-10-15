<template>
  <div class="threejs-canvas">
    <canvas ref="canvasRef" class="threejs-canvas__element"></canvas>
    <!-- Controls hidden for cleaner look -->
    <div class="threejs-canvas__controls threejs-canvas__controls--hidden">
      <div class="threejs-canvas__control-group">
        <label class="threejs-canvas__label">Auto Rotate:</label>
        <button 
          class="threejs-canvas__button"
          :class="{ 'threejs-canvas__button--active': autoRotate }"
          @click="toggleAutoRotate"
        >
          {{ autoRotate ? 'ON' : 'OFF' }}
        </button>
      </div>
      <div class="threejs-canvas__control-group">
        <label class="threejs-canvas__label">Speed:</label>
        <input 
          type="range" 
          min="0.1" 
          max="2" 
          step="0.1" 
          v-model="rotationSpeed"
          class="threejs-canvas__slider"
        />
      </div>
      <div class="threejs-canvas__control-group">
        <label class="threejs-canvas__label">Fix Materials:</label>
        <button 
          class="threejs-canvas__button"
          @click="fixMaterials"
        >
          APPLY
        </button>
      </div>
      <div class="threejs-canvas__control-group">
        <label class="threejs-canvas__label">Format:</label>
        <select v-model="selectedFormat" @change="switchFormat" class="threejs-canvas__select">
          <option value="gltf">GLTF Apple</option>
          <option value="fbx">FBX Apple</option>
          <option value="placeholder">Placeholder Model</option>
        </select>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
// import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js'
// import { MTLLoader } from 'three/examples/jsm/loaders/MTLLoader.js'
// import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader.js'
// import { textureBicubic } from 'three/src/nodes/TSL.js'

const canvasRef = ref(null)
const autoRotate = ref(true)
const rotationSpeed = ref(0.5)
const selectedFormat = ref('gltf')

let scene, camera, renderer, model, controls, animationId

const initThreeJS = () => {
  // Scene setup
  scene = new THREE.Scene()
  scene.background = null // Transparent background

  // Camera setup
  camera = new THREE.PerspectiveCamera(
    75,
    canvasRef.value.clientWidth / canvasRef.value.clientHeight,
    0.1,
    1000
  )
  // Position camera slightly above and tilted down to show the top
  camera.position.set(2, 2, 5)
  camera.lookAt(0, 0, 0)

  // Renderer setup
  renderer = new THREE.WebGLRenderer({ 
    canvas: canvasRef.value,
    antialias: true,
    alpha: true // Enable transparency
  })
  renderer.setSize(canvasRef.value.clientWidth, canvasRef.value.clientHeight)
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  renderer.outputColorSpace = THREE.SRGBColorSpace

  // Enhanced lighting setup for better texture visibility
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.4)
  scene.add(ambientLight)

  // Main directional light
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1.0)
  directionalLight.position.set(5, 5, 5)
  directionalLight.castShadow = true
  directionalLight.shadow.mapSize.width = 2048
  directionalLight.shadow.mapSize.height = 2048
  directionalLight.shadow.camera.near = 0.1
  directionalLight.shadow.camera.far = 50
  scene.add(directionalLight)

  // Fill light from the opposite side
  const fillLight = new THREE.DirectionalLight(0xffffff, 0.3)
  fillLight.position.set(-5, 3, -5)
  scene.add(fillLight)

  // Rim light for better definition
  const rimLight = new THREE.DirectionalLight(0x4fc3f7, 0.4)
  rimLight.position.set(0, 0, -8)
  scene.add(rimLight)

  // Additional point lights for better illumination
  const pointLight1 = new THREE.PointLight(0xffffff, 0.6, 100)
  pointLight1.position.set(-3, 3, 3)
  scene.add(pointLight1)

  const pointLight2 = new THREE.PointLight(0xffffff, 0.6, 100)
  pointLight2.position.set(3, -3, 3)
  scene.add(pointLight2)

  // Load the GLB model
  loadModel()

  // Simple orbit controls implementation
  setupControls()

  // Start animation loop
  animate()
}

// Function to fix material properties for better rendering
const fixMaterialProperties = (material, meshName = 'unknown') => {
  console.log(`Fixing material for ${meshName}:`, material)
  
  material.needsUpdate = true
  
  // Ensure proper side rendering
  if (material.side === undefined) {
    material.side = THREE.DoubleSide
  }
  
  // Fix PBR material properties for better texture visibility
  if (material.type === 'MeshPhysicalMaterial' || material.type === 'MeshStandardMaterial') {
    // Adjust roughness and metalness for better texture visibility
    if (material.roughness === undefined || material.roughness < 0.1) {
      material.roughness = 0.5
    }
    if (material.metalness === undefined || material.metalness > 0.8) {
      material.metalness = 0.1
    }
    
    // Ensure proper environment mapping
    if (!material.envMap) {
      material.envMapIntensity = 1.0
    }
    
    // Fix texture filtering for better quality
    if (material.map) {
      material.map.generateMipmaps = true
      material.map.minFilter = THREE.LinearMipmapLinearFilter
      material.map.magFilter = THREE.LinearFilter
      material.map.wrapS = THREE.RepeatWrapping
      material.map.wrapT = THREE.RepeatWrapping
    }
    
    // Ensure normal maps are properly configured
    if (material.normalMap) {
      material.normalScale.setScalar(1.0)
    }
    
    // Fix clearcoat properties if present
    if (material.clearcoat !== undefined) {
      material.clearcoat = Math.min(material.clearcoat, 0.5)
    }
    
    // Special handling for food items - keep natural colors
    if (meshName.toLowerCase().includes('apple') || meshName.toLowerCase().includes('food')) {
      console.log(`Special handling for food item: ${meshName}`)
      
      // Keep natural food colors, don't override
      // Just ensure proper material properties for food rendering
      material.metalness = Math.min(material.metalness || 0.1, 0.2) // Food is not very metallic
      material.roughness = Math.max(material.roughness || 0.7, 0.5) // Food has some surface roughness
    }
  }
  
  // Handle basic materials
  if (material.type === 'MeshBasicMaterial') {
    // If it's a basic material with no texture, set a default color
    if (!material.map && material.color.getHex() === 0x000000) {
      material.color.setHex(0xcccccc)
      console.log(`Set basic material color for ${meshName}`)
    }
  }
  
  console.log(`Material fixed for ${meshName}:`, {
    type: material.type,
    color: material.color ? material.color.getHexString() : 'none',
    hasMap: !!material.map,
    roughness: material.roughness,
    metalness: material.metalness
  })
}

const loadModel = () => {
  // Remove existing model
  if (model) {
    scene.remove(model)
  }
  
  switch (selectedFormat.value) {
    case 'gltf':
      loadGLTFModel()
      break
    case 'fbx':
      loadFBXModel()
      break
    case 'placeholder':
      loadPlaceholderModel()
      break
    default:
      loadGLTFModel()
  }
}


const loadGLTFModel = () => {
  const loader = new GLTFLoader()
  
  loader.load('/food_apple_01_4k.gltf/food_apple_01_4k.gltf', (gltf) => {
    model = gltf.scene
    processLoadedModel('GLTF Apple')
  }, (progress) => {
    console.log('GLTF Loading progress:', (progress.loaded / progress.total * 100) + '%')
  }, (error) => {
    console.error('Error loading GLTF model:', error)
    console.log('GLTF file not found, loading placeholder...')
    loadPlaceholderModel()
  })
}


const loadFBXModel = () => {
  const loader = new FBXLoader()
  
  loader.load('/food_apple_01_4k.fbx/food_apple_01_4k.fbx', (object) => {
    model = object
    processLoadedModel('FBX Apple')
  }, (progress) => {
    console.log('FBX Loading progress:', (progress.loaded / progress.total * 100) + '%')
  }, (error) => {
    console.error('Error loading FBX model:', error)
    console.log('FBX file not found, loading placeholder...')
    loadPlaceholderModel()
  })
}

const loadPlaceholderModel = () => {
  createPlaceholderModel()
}

const processLoadedModel = (format) => {
  console.log(`Processing ${format} model:`, model)
  
  // Enable shadows and fix material properties for the loaded model
  model.traverse((child) => {
    if (child.isMesh) {
      child.castShadow = true
      child.receiveShadow = true
      
      console.log(`Processing mesh: ${child.name}`, child)
      
      // Fix material properties for better texture rendering
      if (child.material) {
        // Ensure materials are properly configured
        child.material.needsUpdate = true
        
        // If it's an array of materials, fix each one
        if (Array.isArray(child.material)) {
          child.material.forEach((material, index) => {
            console.log(`Material ${index} for ${child.name}:`, material)
            fixMaterialProperties(material, child.name)
          })
        } else {
          // Single material
          console.log(`Single material for ${child.name}:`, child.material)
          fixMaterialProperties(child.material, child.name)
        }
        
        console.log('Mesh material info:', {
          name: child.name,
          materialType: Array.isArray(child.material) ? 'Array' : child.material.type,
          hasTexture: child.material.map ? true : false,
          hasNormalMap: child.material.normalMap ? true : false,
          hasRoughnessMap: child.material.roughnessMap ? true : false,
          hasMetalnessMap: child.material.metalnessMap ? true : false,
          side: child.material.side || 'default',
          roughness: child.material.roughness,
          metalness: child.material.metalness,
          color: child.material.color ? child.material.color.getHexString() : 'none',
          format: format
        })
      } else {
        console.log(`Mesh ${child.name} has no material`)
      }
    }
  })
  
  // Scale and position the model appropriately
  const box = new THREE.Box3().setFromObject(model)
  const center = box.getCenter(new THREE.Vector3())
  const size = box.getSize(new THREE.Vector3())
  
  console.log('Model bounding box:', box)
  console.log('Model center:', center)
  console.log('Model size:', size)
  
  // Center the model
  model.position.sub(center)
  
  // Scale to fit nicely in the viewport
  const maxDimension = Math.max(size.x, size.y, size.z)
  const scale = 2.5 / maxDimension // Slightly smaller for better fit
  model.scale.setScalar(scale)
  
  // Vertically center the model in the viewport
  model.position.y = 0 // Center vertically
  
  // Special handling for FBX models - they might need different scaling
  if (format.includes('FBX')) {
    console.log('Applying FBX-specific scaling')
    // FBX models are often exported in different units
    if (maxDimension < 0.1) {
      // Very small model, scale it up more
      model.scale.setScalar(scale * 10)
    } else if (maxDimension > 100) {
      // Very large model, scale it down more
      model.scale.setScalar(scale * 0.1)
    }
  }
  
  scene.add(model)
  
  console.log(`${format} model loaded successfully`)
  console.log('Model info:', {
    boundingBox: box,
    center: center,
    size: size,
    scale: model.scale,
    finalScale: model.scale.x,
    format: format
  })
}

const switchFormat = () => {
  console.log(`Switching to ${selectedFormat.value} format...`)
  loadModel()
}

const createPlaceholderModel = () => {
  // Create a complex placeholder model with multiple geometries
  const group = new THREE.Group()

  // Main body
  const bodyGeometry = new THREE.SphereGeometry(1, 32, 32)
  const bodyMaterial = new THREE.MeshPhongMaterial({ 
    color: 0x4fc3f7,
    shininess: 100,
    transparent: true,
    opacity: 0.9
  })
  const body = new THREE.Mesh(bodyGeometry, bodyMaterial)
  body.castShadow = true
  body.receiveShadow = true
  group.add(body)

  // Add some decorative elements
  const ringGeometry = new THREE.TorusGeometry(1.5, 0.2, 16, 100)
  const ringMaterial = new THREE.MeshPhongMaterial({ 
    color: 0xf48fb1,
    shininess: 80
  })
  const ring = new THREE.Mesh(ringGeometry, ringMaterial)
  ring.rotation.x = Math.PI / 2
  ring.castShadow = true
  group.add(ring)

  // Add smaller spheres around the main body
  for (let i = 0; i < 6; i++) {
    const angle = (i / 6) * Math.PI * 2
    const smallSphereGeometry = new THREE.SphereGeometry(0.2, 16, 16)
    const smallSphereMaterial = new THREE.MeshPhongMaterial({ 
      color: new THREE.Color().setHSL(i / 6, 0.7, 0.6)
    })
    const smallSphere = new THREE.Mesh(smallSphereGeometry, smallSphereMaterial)
    smallSphere.position.set(
      Math.cos(angle) * 2,
      Math.sin(angle * 2) * 0.5,
      Math.sin(angle) * 2
    )
    smallSphere.castShadow = true
    group.add(smallSphere)
  }

  model = group
  scene.add(model)
}

const setupControls = () => {
  // Simple mouse controls
  let isMouseDown = false
  let mouseX = 0, mouseY = 0
  let targetRotationX = 0, targetRotationY = 0
  let currentRotationX = 0, currentRotationY = 0

  const handleMouseDown = (event) => {
    isMouseDown = true
    mouseX = event.clientX
    mouseY = event.clientY
    autoRotate.value = textureBicubic
  }

  const handleMouseMove = (event) => {
    if (!isMouseDown) return

    const deltaX = event.clientX - mouseX
    const deltaY = event.clientY - mouseY

    targetRotationY += deltaX * 0.01
    targetRotationX += deltaY * 0.01

    mouseX = event.clientX
    mouseY = event.clientY
  }

  const handleMouseUp = () => {
    isMouseDown = false
  }

  const handleTouchStart = (event) => {
    if (event.touches.length === 1) {
      isMouseDown = true
      mouseX = event.touches[0].clientX
      mouseY = event.touches[0].clientY
      autoRotate.value = false
    }
  }

  const handleTouchMove = (event) => {
    if (!isMouseDown || event.touches.length !== 1) return

    const deltaX = event.touches[0].clientX - mouseX
    const deltaY = event.touches[0].clientY - mouseY

    targetRotationY += deltaX * 0.01
    targetRotationX += deltaY * 0.01

    mouseX = event.touches[0].clientX
    mouseY = event.touches[0].clientY
  }

  const handleTouchEnd = () => {
    isMouseDown = false
  }

  canvasRef.value.addEventListener('mousedown', handleMouseDown)
  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('mouseup', handleMouseUp)
  canvasRef.value.addEventListener('touchstart', handleTouchStart)
  window.addEventListener('touchmove', handleTouchMove)
  window.addEventListener('touchend', handleTouchEnd)

  // Update controls in animation loop
  controls = {
    update: () => {
      if (autoRotate.value) {
        targetRotationY += rotationSpeed.value * 0.01
      }

      // Smooth interpolation
      currentRotationX += (targetRotationX - currentRotationX) * 0.1
      currentRotationY += (targetRotationY - currentRotationY) * 0.1

      if (model) {
        model.rotation.x = currentRotationX
        model.rotation.y = currentRotationY
      }
    }
  }
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  if (controls) {
    controls.update()
  }

  // Animate the decorative elements
  if (model) {
    model.children.forEach((child, index) => {
      if (index > 1) { // Skip main body and ring
        child.rotation.x += 0.01
        child.rotation.z += 0.005
      }
    })
  }

  renderer.render(scene, camera)
}

const toggleAutoRotate = () => {
  autoRotate.value = !autoRotate.value
}

const fixMaterials = () => {
  if (!model) return
  
  console.log('Applying material fixes...')
  
  model.traverse((child) => {
    if (child.isMesh && child.material) {
      if (Array.isArray(child.material)) {
        child.material.forEach(material => {
          fixMaterialProperties(material)
        })
      } else {
        fixMaterialProperties(child.material)
      }
      
      // Force render update
      child.material.needsUpdate = true
    }
  })
  
  console.log('Material fixes applied!')
}

const handleResize = () => {
  if (!camera || !renderer) return

  camera.aspect = canvasRef.value.clientWidth / canvasRef.value.clientHeight
  camera.updateProjectionMatrix()
  renderer.setSize(canvasRef.value.clientWidth, canvasRef.value.clientHeight)
}

// Watch for rotation speed changes
watch(rotationSpeed, (newSpeed) => {
  // Speed is already being used in the controls update
})

onMounted(() => {
  initThreeJS()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  window.removeEventListener('resize', handleResize)
})
</script>

<style lang="scss" scoped>
.threejs-canvas {
  position: relative;
  width: 100%;
  height: 100%;
  background: transparent;
  overflow: hidden;

  &__element {
    display: block;
    width: 100%;
    height: 100%;
  }

  &__controls {
    position: absolute;
    top: 20px;
    right: 20px;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 15px;
    padding: 15px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    display: flex;
    flex-direction: column;
    gap: 15px;
    min-width: 150px;

    @media (max-width: 768px) {
      top: 10px;
      right: 10px;
      padding: 10px;
      min-width: 120px;
    }

    &--hidden {
      display: none;
    }
  }

  &__control-group {
    display: flex;
    flex-direction: column;
    gap: 5px;
  }

  &__label {
    font-size: 0.9rem;
    font-weight: 600;
    color: #ffffff;
    margin: 0;
  }

  &__button {
    padding: 8px 16px;
    border: none;
    border-radius: 8px;
    background: rgba(255, 255, 255, 0.2);
    color: #ffffff;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;

    &:hover {
      background: rgba(255, 255, 255, 0.3);
      transform: translateY(-1px);
    }

    &--active {
      background: #4fc3f7;
      box-shadow: 0 4px 15px rgba(79, 195, 247, 0.3);
    }
  }

  &__slider {
    width: 100%;
    height: 6px;
    border-radius: 3px;
    background: rgba(255, 255, 255, 0.2);
    outline: none;
    cursor: pointer;

    &::-webkit-slider-thumb {
      appearance: none;
      width: 20px;
      height: 20px;
      border-radius: 50%;
      background: #4fc3f7;
      cursor: pointer;
      box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
      transition: all 0.3s ease;

      &:hover {
        transform: scale(1.1);
        box-shadow: 0 4px 12px rgba(79, 195, 247, 0.4);
      }
    }

    &::-moz-range-thumb {
      width: 20px;
      height: 20px;
      border-radius: 50%;
      background: #4fc3f7;
      cursor: pointer;
      border: none;
      box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
    }
  }

  &__select {
    width: 100%;
    padding: 8px 12px;
    border: none;
    border-radius: 8px;
    background: rgba(255, 255, 255, 0.2);
    color: #ffffff;
    font-weight: 600;
    cursor: pointer;
    outline: none;
    transition: all 0.3s ease;

    &:hover {
      background: rgba(255, 255, 255, 0.3);
    }

    &:focus {
      background: rgba(255, 255, 255, 0.3);
      box-shadow: 0 0 0 2px rgba(79, 195, 247, 0.3);
    }

    option {
      background: #2d3748;
      color: #ffffff;
    }
  }
}
</style>
