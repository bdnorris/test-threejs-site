<template>
  <div class="depth-map-demo">
    <canvas ref="canvasRef" class="depth-map-demo__canvas"></canvas>
    <div class="depth-map-demo__instructions">
      <p>Move your mouse over the image to see the 3D depth and tilt effect</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'

const canvasRef = ref(null)

let scene, camera, renderer, plane, animationId
let mouse = { x: 0, y: 0 }
let targetRotation = { x: 0, y: 0 }
let currentRotation = { x: 0, y: 0 }

const maxTilt = 0.3 // Maximum rotation in radians

const initThreeJS = () => {
  // Scene setup
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x1a1a1a)

  // Camera setup - perspective for better depth perception
  const aspect = canvasRef.value.clientWidth / canvasRef.value.clientHeight
  camera = new THREE.PerspectiveCamera(45, aspect, 0.1, 1000)
  camera.position.z = 3
  camera.position.y = 0

  // Renderer setup
  renderer = new THREE.WebGLRenderer({ 
    canvas: canvasRef.value,
    antialias: true
  })
  renderer.setSize(canvasRef.value.clientWidth, canvasRef.value.clientHeight)
  renderer.setPixelRatio(window.devicePixelRatio)

  // Lighting - enhanced to show depth
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.6)
  scene.add(ambientLight)

  const directionalLight1 = new THREE.DirectionalLight(0xffffff, 0.8)
  directionalLight1.position.set(5, 5, 5)
  scene.add(directionalLight1)

  const directionalLight2 = new THREE.DirectionalLight(0xffffff, 0.4)
  directionalLight2.position.set(-5, -5, 3)
  scene.add(directionalLight2)

  // Load textures and create plane
  loadTextures()

  // Start animation loop
  animate()
}

const loadTextures = () => {
  const textureLoader = new THREE.TextureLoader()
  
  // NOTE: Place your files in public/depth-map/ directory:
  // - image.jpg (or .png) - your main image
  // - depth.jpg (or .png) - your depth map (white = close, black = far)
  
  const imagePath = '/depth-map/Ralph-Smith-Chefmate-Cheese-Sauce-Burger-Food-Photography.jpg'
  const depthPath = '/depth-map/Ralph-Smith-Chefmate-Cheese-Sauce-Burger-Food-Photography_depth-map.png'
  
  // Load main image texture
  textureLoader.load(
    imagePath,
    (texture) => {
      console.log('Image loaded successfully')
      
      // Calculate aspect ratio to maintain image proportions
      const aspect = texture.image.width / texture.image.height
      
      // Create plane geometry with high segment count for smooth displacement
      const geometry = new THREE.PlaneGeometry(aspect * 1.5, 1.5, 256, 256)
      
      // Load depth map
      textureLoader.load(
        depthPath,
        (depthTexture) => {
          console.log('Depth map loaded successfully')
          
          // Create material with both textures and displacement
          const material = new THREE.MeshStandardMaterial({
            map: texture,
            displacementMap: depthTexture,
            displacementScale: 0.3, // Controls depth of 3D effect
            displacementBias: -0.2, // Adjust the center point of displacement
            side: THREE.DoubleSide,
            roughness: 0.8,
            metalness: 0.1
          })
          
          plane = new THREE.Mesh(geometry, material)
          scene.add(plane)
        },
        undefined,
        (error) => {
          console.warn('Depth map not found, using image only:', error)
          // Create material without depth map
          const material = new THREE.MeshStandardMaterial({
            map: texture,
            side: THREE.DoubleSide
          })
          
          plane = new THREE.Mesh(geometry, material)
          scene.add(plane)
        }
      )
    },
    undefined,
    (error) => {
      console.error('Error loading image:', error)
      console.log('Creating placeholder since image not found at:', imagePath)
      createPlaceholder()
    }
  )
}

const createPlaceholder = () => {
  // Create a placeholder with a gradient if images aren't found
  const canvas = document.createElement('canvas')
  canvas.width = 512
  canvas.height = 512
  const ctx = canvas.getContext('2d')
  
  // Create gradient
  const gradient = ctx.createLinearGradient(0, 0, 512, 512)
  gradient.addColorStop(0, '#4fc3f7')
  gradient.addColorStop(1, '#f48fb1')
  ctx.fillStyle = gradient
  ctx.fillRect(0, 0, 512, 512)
  
  // Add text
  ctx.fillStyle = 'white'
  ctx.font = 'bold 32px Arial'
  ctx.textAlign = 'center'
  ctx.fillText('Place your image at:', 256, 200)
  ctx.fillText('public/depth-map/image.jpg', 256, 250)
  ctx.fillText('and depth map at:', 256, 320)
  ctx.fillText('public/depth-map/depth.jpg', 256, 370)
  
  const texture = new THREE.CanvasTexture(canvas)
  const geometry = new THREE.PlaneGeometry(1.5, 1.5)
  const material = new THREE.MeshStandardMaterial({
    map: texture,
    side: THREE.DoubleSide
  })
  
  plane = new THREE.Mesh(geometry, material)
  scene.add(plane)
}

const handleMouseMove = (event) => {
  // Normalize mouse position to -1 to 1 range
  const rect = canvasRef.value.getBoundingClientRect()
  mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1
  mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1
  
  // Calculate target rotation based on mouse position
  targetRotation.y = mouse.x * maxTilt
  targetRotation.x = mouse.y * maxTilt
}

const handleMouseLeave = () => {
  // Return to center position when mouse leaves
  targetRotation.x = 0
  targetRotation.y = 0
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  if (plane) {
    // Smooth interpolation for natural movement
    currentRotation.x += (targetRotation.x - currentRotation.x) * 0.1
    currentRotation.y += (targetRotation.y - currentRotation.y) * 0.1
    
    // Apply rotation
    plane.rotation.x = currentRotation.x
    plane.rotation.y = currentRotation.y
  }

  renderer.render(scene, camera)
}

const handleResize = () => {
  if (!camera || !renderer) return

  const aspect = canvasRef.value.clientWidth / canvasRef.value.clientHeight
  camera.aspect = aspect
  camera.updateProjectionMatrix()
  
  renderer.setSize(canvasRef.value.clientWidth, canvasRef.value.clientHeight)
}

onMounted(() => {
  initThreeJS()
  window.addEventListener('resize', handleResize)
  canvasRef.value.addEventListener('mousemove', handleMouseMove)
  canvasRef.value.addEventListener('mouseleave', handleMouseLeave)
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  window.removeEventListener('resize', handleResize)
  if (canvasRef.value) {
    canvasRef.value.removeEventListener('mousemove', handleMouseMove)
    canvasRef.value.removeEventListener('mouseleave', handleMouseLeave)
  }
  
  // Cleanup Three.js resources
  if (renderer) {
    renderer.dispose()
  }
  if (scene) {
    scene.traverse((object) => {
      if (object.geometry) {
        object.geometry.dispose()
      }
      if (object.material) {
        if (Array.isArray(object.material)) {
          object.material.forEach(material => material.dispose())
        } else {
          object.material.dispose()
        }
      }
    })
  }
})
</script>

<style lang="scss" scoped>
.depth-map-demo {
  position: relative;
  width: 100%;
  height: 100%;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #1a1a1a 0%, #2d3748 100%);

  &__canvas {
    display: block;
    width: 100%;
    height: 100%;
    cursor: move;
  }

  &__instructions {
    position: absolute;
    bottom: 2rem;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    padding: 1rem 2rem;
    border: 1px solid rgba(255, 255, 255, 0.2);

    p {
      margin: 0;
      color: #ffffff;
      font-size: 1rem;
      font-weight: 600;
      text-align: center;
    }

    @media (max-width: 768px) {
      bottom: 1rem;
      padding: 0.75rem 1.5rem;

      p {
        font-size: 0.875rem;
      }
    }
  }
}
</style>

