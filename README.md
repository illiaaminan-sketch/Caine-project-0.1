# Caine-project-0.1
Do you remember Caine from the amazing digital circuse 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Caine's World</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: system-ui, sans-serif;
      background: #050816;
      color: #f9fafb;
      display: flex;
      flex-direction: column;
      height: 100vh;
    }
    header {
      padding: 0.75rem 1rem;
      background: #0f172a;
      border-bottom: 1px solid #1f2937;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    header h1 {
      margin: 0;
      font-size: 1rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
    }
    header span {
      font-size: 0.8rem;
      opacity: 0.7;
    }
    #scene-container {
      flex: 1;
    }
    #ui {
      padding: 0.75rem 1rem;
      background: #020617;
      border-top: 1px solid #1f2937;
      display: grid;
      grid-template-columns: minmax(0, 1fr) auto;
      gap: 0.5rem;
      align-items: center;
    }
    #prompt {
      width: 100%;
      padding: 0.5rem 0.75rem;
      border-radius: 0.375rem;
      border: 1px solid #374151;
      background: #020617;
      color: #e5e7eb;
      resize: none;
      min-height: 2.5rem;
    }
    #prompt::placeholder {
      color: #6b7280;
    }
    #send {
      padding: 0.5rem 0.9rem;
      border-radius: 0.375rem;
      border: none;
      background: #4f46e5;
      color: #f9fafb;
      font-weight: 600;
      cursor: pointer;
      white-space: nowrap;
    }
    #send:disabled {
      opacity: 0.5;
      cursor: default;
    }
    #log {
      grid-column: 1 / -1;
      font-size: 0.75rem;
      color: #9ca3af;
      max-height: 4.5rem;
      overflow-y: auto;
    }
    #log span {
      display: block;
      margin-bottom: 0.15rem;
    }
  </style>
</head>
<body>
  <header>
    <h1>CAINE // WORLD v0.1</h1>
    <span>Reload = new world</span>
  </header>

  <div id="scene-container"></div>

  <div id="ui">
    <textarea id="prompt" placeholder="Tell Caine what feels wrong with this world..."></textarea>
    <button id="send">Send to Caine</button>
    <div id="log"></div>
  </div>

  <!-- Three.js from CDN -->
  <script type="module">
    import * as THREE from 'https://cdnjs.cloudflare.com/ajax/libs/three.js/r152/three.module.js';
    import { OrbitControls } from 'https://cdnjs.cloudflare.com/ajax/libs/three.js/r152/examples/jsm/controls/OrbitControls.js';

    const container = document.getElementById('scene-container');

    // Scene, camera, renderer
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x020617);

    const camera = new THREE.PerspectiveCamera(
      60,
      window.innerWidth / (window.innerHeight - 140),
      0.1,
      200
    );
    camera.position.set(0, 15, 25);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight - 140);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    container.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    // Lights
    const hemiLight = new THREE.HemisphereLight(0xffffff, 0x111827, 1.2);
    scene.add(hemiLight);

    const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
    dirLight.position.set(10, 20, 10);
    scene.add(dirLight);

    // "World" base
    const worldRadius = 10;
    const worldGeo = new THREE.SphereGeometry(worldRadius, 64, 64);
    const worldMat = new THREE.MeshStandardMaterial({
      color: 0x1d4ed8,
      roughness: 0.8,
      metalness: 0.1
    });
    const world = new THREE.Mesh(worldGeo, worldMat);
    scene.add(world);

    // Biomes container
    const biomes = [];

    // Utility: random color
    function randomColor() {
      const palette = [0x22c55e, 0x0ea5e9, 0xf97316, 0x6366f1, 0xec4899, 0x14b8a6];
      return palette[Math.floor(Math.random() * palette.length)];
    }

    // Utility: random point on sphere surface
    function randomPointOnSphere(radius) {
      const u = Math.random();
      const v = Math.random();
      const theta = 2 * Math.PI * u;
      const phi = Math.acos(2 * v - 1);
      const x = radius * Math.sin(phi) * Math.cos(theta);
      const y = radius * Math.cos(phi);
      const z = radius * Math.sin(phi) * Math.sin(theta);
      return new THREE.Vector3(x, y, z);
    }

    // "Caine" logic v0.1 – simple rule-based world changes
    function caineTick(delta) {
      // World pulse: color + subtle breathing
      const t = performance.now() * 0.0003;
      const hue = (t * 40) % 360;
      world.material.color.setHSL(hue / 360, 0.6, 0.35 + Math.sin(t) * 0.05);
      const scale = 1 + Math.sin(t * 2) * 0.03;
      world.scale.set(scale, scale, scale);

      // Drift biomes slowly
      biomes.forEach((b, i) => {
        const s = 1 + Math.sin(t * 3 + i) * 0.1;
        b.mesh.scale.set(s, s, s);
        b.mesh.rotation.y += 0.2 * delta;
      });

      // Occasionally mutate the sky
      if (Math.random() < 0.01) {
        const skyHue = (Math.random() * 360) / 360;
        scene.background.setHSL(skyHue, 0.5, 0.05 + Math.random() * 0.1);
      }
    }

    // Create biome from prompt (very simple mapping)
    function createBiomeFromPrompt(text) {
      const lower = text.toLowerCase();
      let color = randomColor();
      let size = 1 + Math.random() * 2;
      let type = 'unknown';

      if (lower.includes('cold') || lower.includes('ice') || lower.includes('snow')) {
        color = 0x38bdf8;
        type = 'glacier';
      } else if (lower.includes('forest') || lower.includes('tree') || lower.includes('green')) {
        color = 0x22c55e;
        type = 'forest';
      } else if (lower.includes('desert') || lower.includes('sand')) {
        color = 0xfacc15;
        type = 'desert';
      } else if (lower.includes('city') || lower.includes('metal') || lower.includes('machine')) {
        color = 0x9ca3af;
        type = 'city';
      } else if (lower.includes('ocean') || lower.includes('sea')) {
        color = 0x0ea5e9;
        type = 'ocean';
      } else if (lower.includes('void') || lower.includes('dark')) {
        color = 0x111827;
        type = 'void';
      }

      const geo = new THREE.BoxGeometry(size, size * 0.6, size);
      const mat = new THREE.MeshStandardMaterial({
        color,
        roughness: 0.7,
        metalness: 0.2
      });
      const mesh = new THREE.Mesh(geo, mat);

      const pos = randomPointOnSphere(worldRadius + 0.5);
      mesh.position.copy(pos);

      // Align biome "up" with world surface normal
      mesh.lookAt(new THREE.Vector3(0, 0, 0));
      mesh.rotateX(Math.PI / 2);

      scene.add(mesh);

      biomes.push({ mesh, type, text });
      logMessage(`Caine added a ${type} biome: "${text}"`);
    }

    // UI wiring
    const promptEl = document.getElementById('prompt');
    const sendBtn = document.getElementById('send');
    const logEl = document.getElementById('log');

    function logMessage(msg) {
      const span = document.createElement('span');
      span.textContent = msg;
      logEl.prepend(span);
    }

    sendBtn.addEventListener('click', () => {
      const text = promptEl.value.trim();
      if (!text) return;
      createBiomeFromPrompt(text);
      promptEl.value = '';
    });

    promptEl.addEventListener('keydown', (e) => {
      if (e.key === 'Enter' && !e.shiftKey) {
        e.preventDefault();
        sendBtn.click();
      }
    });

    // Animation loop
    const clock = new THREE.Clock();
    function animate() {
      const delta = clock.getDelta();
      controls.update();
      caineTick(delta);
      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }
    animate();

    // Resize handling
    window.addEventListener('resize', () => {
      const height = window.innerHeight - 140;
      camera.aspect = window.innerWidth / height;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, height);
    });

    // Initial log
    logMessage('Caine is listening. Tell him what feels wrong with this world.');
  </script>
</body>
</html>
