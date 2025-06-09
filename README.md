<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>3D Solar System Simulation</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<style>
  /* Base Reset */
  *, *::before, *::after {
    box-sizing: border-box;
  }
  body, html {
    margin: 0; padding: 0;
    font-family: 'Poppins', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
                 Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    background-color: #ffffff;
    color: #6b7280;
    font-size: 17px;
    line-height: 1.6;
    height: 100%;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
  #container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 3rem 1.5rem 4rem;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }
  header {
    position: sticky;
    top: 0;
    background: #fff;
    box-shadow: 0 1px 4px rgb(0 0 0 / 0.1);
    z-index: 1000;
    padding: 1rem 1.5rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  header h1 {
    font-weight: 800;
    font-size: 2rem;
    color: #111827;
    margin: 0;
    user-select: none;
  }
  nav a {
    text-decoration: none;
    margin-left: 1.5rem;
    font-weight: 600;
    color: #6b7280;
    transition: color 0.3s ease;
  }
  nav a:hover, nav a:focus {
    color: #111827;
    outline: none;
  }
  main {
    flex-grow: 1;
    display: flex;
    flex-direction: column;
  }
  #hero {
    text-align: center;
    padding-bottom: 3rem;
  }
  #hero h2 {
    font-size: 3.5rem;
    font-weight: 700;
    color: #111827;
    margin: 0 0 0.5rem 0;
    line-height: 1.1;
  }
  #hero p {
    max-width: 600px;
    margin: 0 auto;
    color: #6b7280;
    font-size: 1.125rem;
  }
  #content {
    display: grid;
    grid-template-columns: 1fr 320px;
    gap: 2rem;
  }
  #canvas-wrapper {
    border-radius: 0.75rem;
    box-shadow: 0 8px 24px rgb(0 0 0 / 0.1);
    background: #000000;
    aspect-ratio: 4 / 3;
    min-height: 480px;
    position: relative;
  }
  canvas {
    display: block;
    width: 100% !important;
    height: 100% !important;
    border-radius: 0.75rem;
  }
  #control-panel {
    background: #f9fafb;
    padding: 1.5rem;
    border-radius: 0.75rem;
    box-shadow: 0 4px 12px rgb(0 0 0 / 0.05);
    font-size: 14px;
    color: #374151;
    user-select: none;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
  #control-panel h3 {
    margin: 0 0 0.5rem 0;
    font-weight: 700;
    font-size: 1.25rem;
    color: #111827;
  }
  .slider-group {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }
  label {
    font-weight: 600;
    color: #6b7280;
  }
  input[type="range"] {
    -webkit-appearance: none;
    appearance: none;
    width: 100%;
    height: 6px;
    border-radius: 3px;
    background: linear-gradient(90deg, #3b82f6 0%, #93c5fd 100%);
    cursor: pointer;
    outline: none;
    transition: background 0.3s ease;
  }
  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: #2563eb;
    cursor: pointer;
    box-shadow: 0 0 6px rgb(37 99 235 / 0.7);
    border: none;
  }
  input[type="range"]:hover::-webkit-slider-thumb {
    background-color: #1d4ed8;
  }
  input[type="range"]::-moz-range-thumb {
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: #2563eb;
    cursor: pointer;
    border: none;
  }
  #pause-button {
    margin-top: 1rem;
    padding: 0.65rem 1rem;
    font-size: 1rem;
    font-weight: 700;
    color: white;
    background-color: #2563eb;
    border: none;
    border-radius: 0.5rem;
    cursor: pointer;
    transition: background-color 0.3s ease;
    user-select: none;
  }
  #pause-button:hover, #pause-button:focus {
    background-color: #1d4ed8;
    outline: none;
  }
  @media (max-width: 768px) {
    #content {
      grid-template-columns: 1fr;
    }
    #canvas-wrapper {
      min-height: 360px;
      aspect-ratio: auto;
    }
  }
</style>
</head>
<body>
<div id="container" role="main">
  <header role="banner">
    <h1>Solar System 3D</h1>
    <nav aria-label="Primary Navigation">
      <a href="#" tabindex="0">Home</a>
      <a href="#" tabindex="0">About</a>
    </nav>
  </header>
  <section id="hero" aria-label="Page introduction">
    <h2>Explore the Solar System in 3D</h2>
    <p>Adjust planetary orbital speeds and watch planets orbit the sun in real-time.</p>
  </section>
  <section id="content" aria-label="Solar system and controls">
    <div id="canvas-wrapper" aria-label="3D solar system visualization"></div>

    <aside id="control-panel" aria-label="Orbital speed control panel">
      <h3>Control Orbital Speeds</h3>
      <!-- Sliders inserted by script here -->
      <button id="pause-button" aria-pressed="false">Pause</button>
    </aside>
  </section>
</div>

<script>
  // Three.js setup
  const scene = new THREE.Scene();

  // Camera setup - perspective with wider angle
  const camera = new THREE.PerspectiveCamera(45, 4/3, 0.1, 1000);

  // Renderer tied to the container
  const container = document.getElementById('canvas-wrapper');
  const renderer = new THREE.WebGLRenderer({antialias: true});
  function resizeRenderer() {
    const width = container.clientWidth;
    const height = container.clientHeight;
    renderer.setSize(width, height);
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
  }
  container.appendChild(renderer.domElement);
  resizeRenderer();

  // Update renderer on window resize
  window.addEventListener('resize', () => {
    resizeRenderer();
  });

  // Lighting
  const ambientLight = new THREE.AmbientLight(0x555555);
  scene.add(ambientLight);
  const pointLight = new THREE.PointLight(0xffffff, 1.8, 250);
  pointLight.position.set(0, 0, 0);
  scene.add(pointLight);

  // Sun
  const sunGeometry = new THREE.SphereGeometry(2, 64, 64);
  const sunMaterial = new THREE.MeshPhongMaterial({
    color: 0xffcc66,
    emissive: 0xffaa00,
    emissiveIntensity: 1.2
  });
  const sun = new THREE.Mesh(sunGeometry, sunMaterial);
  scene.add(sun);

  // Planet details: name, color, distance from sun, size, speed (orbital radians per frame)
  const planetsData = [
    {name: 'Mercury', color: 0x909090, distance: 4, size: 0.35, speed: 0.04},
    {name: 'Venus', color: 0xd4af37, distance: 6, size: 0.55, speed: 0.015},
    {name: 'Earth', color: 0x4682b4, distance: 8, size: 0.6, speed: 0.012},
    {name: 'Mars', color: 0xcd5c5c, distance: 10, size: 0.5, speed: 0.009},
    {name: 'Jupiter', color: 0xd2b48c, distance: 14, size: 1.25, speed: 0.006},
    {name: 'Saturn', color: 0xf4eed7, distance: 18, size: 1.05, speed: 0.005},
    {name: 'Uranus', color: 0x7fffd4, distance: 21, size: 0.8, speed: 0.004},
    {name: 'Neptune', color: 0x4169e1, distance: 24, size: 0.75, speed: 0.003}
  ];

  // Create planets
  const planets = [];
  planetsData.forEach(data => {
    const geometry = new THREE.SphereGeometry(data.size, 32, 32);
    const material = new THREE.MeshPhongMaterial({color: data.color});
    const mesh = new THREE.Mesh(geometry, material);
    mesh.position.x = data.distance;
    scene.add(mesh);

    // Orbit ring
    const ringGeometry = new THREE.RingGeometry(data.distance - 0.02, data.distance + 0.02, 64);
    const ringMaterial = new THREE.MeshBasicMaterial({
      color: 0x999999,
      side: THREE.DoubleSide,
      transparent: true,
      opacity: 0.15
    });
    const ring = new THREE.Mesh(ringGeometry, ringMaterial);
    ring.rotation.x = Math.PI / 2;
    scene.add(ring);

    planets.push({
      mesh,
      distance: data.distance,
      speed: data.speed,
      angle: Math.random()*2*Math.PI,
      name: data.name
    });
  });

  // Camera positioning for nice overview
  camera.position.set(0, 12, 38);
  camera.lookAt(0, 0, 0);

  // Animation control vars
  let isPaused = false;

  // Build controls
  const controlPanel = document.getElementById('control-panel');
  planets.forEach((planet,i)=>{
    const sliderGroup = document.createElement('div');
    sliderGroup.className = 'slider-group';

    const label = document.createElement('label');
    label.textContent = planet.name + ' speed';
    label.setAttribute('for', `slider-${i}`);
    sliderGroup.appendChild(label);

    const slider = document.createElement('input');
    slider.type = 'range';
    slider.id = `slider-${i}`;
    slider.min = 0;
    slider.max = 0.1;
    slider.step = 0.001;
    slider.value = planet.speed;
    slider.addEventListener('input', e => {
      planet.speed = parseFloat(e.target.value);
    });
    sliderGroup.appendChild(slider);

    controlPanel.insertBefore(sliderGroup, document.getElementById('pause-button'));
  });

  // Pause/Resume button
  const pauseBtn = document.getElementById('pause-button');
  pauseBtn.addEventListener('click', () => {
    isPaused = !isPaused;
    pauseBtn.textContent = isPaused ? 'Resume' : 'Pause';
    pauseBtn.setAttribute('aria-pressed', isPaused);
  });

  // Animate loop with requestAnimationFrame
  function animate() {
    requestAnimationFrame(animate);
    if (!isPaused) {
      planets.forEach(p => {
        p.angle += p.speed;
        p.mesh.position.x = Math.cos(p.angle) * p.distance;
        p.mesh.position.z = Math.sin(p.angle) * p.distance;
        p.mesh.rotation.y += 0.02; // Axis rotation
      });
      sun.rotation.y += 0.003;
    }
    renderer.render(scene, camera);
  }
  animate();
</script>
</body>
</html>

# Frontend-assignment-UDAY-PRATAP-SINGH
