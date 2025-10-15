<template>
  <div class="app">
    <div class="app__content">
      <div class="app__sections">
        <section 
          v-for="(section, index) in sections" 
          :key="index"
          :class="[
            'app__section',
            `app__section--${index + 1}`,
            { 'app__section--left': index % 2 === 0 },
            { 'app__section--right': index % 2 === 1 }
          ]"
          :ref="el => sectionRefs[index] = el"
        >
          <div class="app__section-content">
            <h2 class="app__section-title">{{ section.title }}</h2>
            <p class="app__section-text">{{ section.text }}</p>
          </div>
        </section>
      </div>
      
      <div 
        class="app__canvas-container"
        :class="{
          'app__canvas-container--left': currentSide === 'left',
          'app__canvas-container--right': currentSide === 'right'
        }"
      >
        <ThreeJSCanvas 
          ref="threeCanvasRef"
          class="app__canvas"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import ThreeJSCanvas from './components/ThreeJSCanvas.vue'

const sections = [
  {
    title: "Immersive 3D Experience",
    text: "Discover the power of Three.js through our interactive 3D showcase. This cutting-edge technology brings digital experiences to life with stunning visuals and smooth animations that captivate users across all devices."
  },
  {
    title: "Responsive Design Excellence",
    text: "Our implementation seamlessly adapts to different screen sizes, ensuring optimal viewing experiences on desktop and mobile devices. The layout intelligently repositions elements to maintain visual balance and usability."
  },
  {
    title: "Advanced WebGL Rendering",
    text: "Leveraging the full potential of WebGL, we deliver high-performance 3D graphics directly in the browser. No plugins required - just pure, modern web technology pushing the boundaries of what's possible online."
  },
  {
    title: "Interactive Controls & Animation",
    text: "Users can engage with the 3D model through intuitive controls while enjoying smooth auto-rotation animations. The balance between automation and user control creates an engaging, dynamic experience."
  },
  {
    title: "Modern Development Stack",
    text: "Built with Vue 3, Vite, and Three.js, this project demonstrates modern web development practices. Clean code architecture, component-based design, and optimized build processes ensure maintainable and scalable applications."
  }
]

const sectionRefs = ref([])
const threeCanvasRef = ref(null)
const currentSide = ref('right')

const handleScroll = () => {
  const scrollY = window.scrollY
  const windowHeight = window.innerHeight
  
  // Determine which section is currently in view
  let activeSection = 0
  for (let i = 0; i < sectionRefs.value.length; i++) {
    if (sectionRefs.value[i]) {
      const rect = sectionRefs.value[i].getBoundingClientRect()
      if (rect.top <= windowHeight / 2 && rect.bottom >= windowHeight / 2) {
        activeSection = i
        break
      }
    }
  }
  
  // Update canvas position based on active section
  currentSide.value = activeSection % 2 === 0 ? 'right' : 'left'
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll)
  handleScroll() // Initial call
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})
</script>

<style lang="scss" scoped>
h1, h2, h3, h4 {
  font-family: "BBH Sans Bartle", sans-serif;
	text-wrap: balance;
}
body {
  padding: 0;
}
.app {
  min-height: 100vh;
  background: linear-gradient(
    to bottom left in oklab,
    oklch(55% .45 350),
    oklch(95% .4 95)
  );
  font-family: "Roboto", sans-serif;

  &__content {
    position: relative;
    min-height: 100vh;
    
    @media (min-width: 769px) {
      display: flex;
    }
  }

  &__sections {
    flex: 1;
    
    @media (min-width: 769px) {
      width: 50%;
    }
  }

  &__section {
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 2rem;
    position: relative;

    &--left {
      .app__section-content {
        @media (min-width: 769px) {
          margin-left: 2rem;
          margin-right: auto;
          text-align: left;
        }
      }
    }

    &--right {
      .app__section-content {
        @media (min-width: 769px) {
          margin-right: 2rem;
          margin-left: auto;
          text-align: right;
        }
      }
    }

    &-content {
      max-width: 600px;
      background: rgba(255, 255, 255, 0.95);
      padding: 3rem;
      border-radius: 20px;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
      backdrop-filter: blur(10px);
    }

    &-title {
      font-size: 2.5rem;
      font-weight: 700;
      margin-bottom: 1.5rem;
      color: #2d3748;
      line-height: 1.2;

      @media (max-width: 768px) {
        font-size: 2rem;
      }
    }

    &-text {
      font-size: 1.2rem;
      line-height: 1.8;
      color: #4a5568;
      margin: 0;
    }
  }

  &__canvas-container {
    position: fixed;
    top: 0;
    right: 0;
    width: 50%;
    height: 100vh;
    transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
    z-index: 10;

    &--left {
      transform: translateX(-50vw);
    }

    &--right {
      transform: translateX(0);
    }

    @media (max-width: 768px) {
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 40vh;
      top: auto;
      right: auto;
      transform: none !important;
    }
  }

  &__canvas {
    width: 100%;
    height: 100%;
  }
}

// Smooth scrolling
html {
  scroll-behavior: smooth;
}

body {
  margin: 0;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
</style>
