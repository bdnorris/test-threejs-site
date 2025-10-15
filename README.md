# Three.js Showcase

A Vue 3 + Vite project showcasing Three.js capabilities with responsive design and interactive 3D models.

## Features

- **Responsive Layout**: Desktop layout with alternating content sections and fixed canvas positioning
- **Mobile Optimized**: Canvas fixed to bottom on mobile devices
- **Interactive 3D Model**: Auto-rotating model with user controls
- **Smooth Animations**: Canvas transitions and scroll-based positioning
- **Modern Stack**: Vue 3, Vite, Three.js, and SCSS with BEM methodology

## Installation

1. Install dependencies:
```bash
npm install
```

2. Start the development server:
```bash
npm run dev
```

3. Open your browser and navigate to `http://localhost:3000`

## Project Structure

```
src/
├── App.vue                 # Main application component with layout
├── main.js                # Vue app entry point
└── components/
    └── ThreeJSCanvas.vue  # Three.js scene and controls
```

## Key Features

### Desktop Layout
- 5 content sections taking up 50% of the viewport
- Canvas element positioned on the remaining 50%
- Canvas smoothly transitions between left and right sides as you scroll
- Content alternates sides based on scroll position

### Mobile Layout
- Canvas fixed to bottom of screen (40% height)
- Content sections take full width
- Optimized touch controls for model interaction

### Three.js Features
- Auto-rotating 3D model with customizable speed
- Mouse and touch controls for manual rotation
- Toggle auto-rotation on/off
- Responsive canvas sizing
- Advanced lighting setup with shadows
- Smooth animations and transitions

## Customization

### Adding Your Own GLB Model
To load your own GLB model, you'll need to:

1. Install the GLTFLoader:
```bash
npm install three-gltf-loader
```

2. Update the `loadModel()` function in `ThreeJSCanvas.vue`:
```javascript
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'

const loader = new GLTFLoader()
loader.load('/path/to/your/model.glb', (gltf) => {
  model = gltf.scene
  scene.add(model)
})
```

### Styling
The project uses BEM methodology for CSS classes and scoped SCSS. All styles are component-scoped to prevent conflicts.

## Build for Production

```bash
npm run build
```

The built files will be in the `dist` directory, ready for deployment.

## Browser Support

- Modern browsers with WebGL support
- Chrome, Firefox, Safari, Edge (latest versions)
- Mobile browsers with WebGL support
