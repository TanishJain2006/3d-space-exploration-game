<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Solar System Explorer - 3D Space Game</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: radial-gradient(ellipse at center, #1a1a2e 0%, #16213e 50%, #0f0f23 100%);
            overflow: hidden;
            color: white;
        }

        #gameContainer {
            position: relative;
            width: 100vw;
            height: 100vh;
        }

        #canvas {
            display: block;
            cursor: grab;
        }

        #canvas:active {
            cursor: grabbing;
        }

        .ui-panel {
            position: absolute;
            background: rgba(0, 20, 40, 0.9);
            border: 1px solid rgba(100, 200, 255, 0.3);
            border-radius: 10px;
            padding: 15px;
            backdrop-filter: blur(10px);
            box-shadow: 0 4px 20px rgba(0, 100, 200, 0.2);
        }

        #controlPanel {
            top: 20px;
            left: 20px;
            width: 280px;
        }

        #infoPanel {
            top: 20px;
            right: 20px;
            width: 300px;
            max-height: 70vh;
            overflow-y: auto;
        }

        #bottomPanel {
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 15px;
            align-items: center;
        }

        .control-group {
            margin-bottom: 15px;
        }

        .control-group h3 {
            color: #64b5f6;
            margin-bottom: 8px;
            font-size: 14px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .button {
            background: linear-gradient(135deg, #1976d2, #42a5f5);
            border: none;
            color: white;
            padding: 8px 16px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 12px;
            margin: 2px;
            transition: all 0.3s ease;
        }

        .button:hover {
            background: linear-gradient(135deg, #1565c0, #2196f3);
            transform: translateY(-1px);
            box-shadow: 0 4px 12px rgba(33, 150, 243, 0.4);
        }

        .button.active {
            background: linear-gradient(135deg, #f57c00, #ff9800);
        }

        .slider-container {
            display: flex;
            align-items: center;
            gap: 10px;
            margin: 5px 0;
        }

        .slider {
            flex: 1;
            height: 4px;
            border-radius: 2px;
            background: rgba(255, 255, 255, 0.2);
            outline: none;
            -webkit-appearance: none;
        }

        .slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 16px;
            height: 16px;
            border-radius: 50%;
            background: #64b5f6;
            cursor: pointer;
        }

        .celestial-info {
            margin-bottom: 15px;
            padding: 10px;
            background: rgba(255, 255, 255, 0.05);
            border-radius: 6px;
            border-left: 3px solid #64b5f6;
        }

        .celestial-info h4 {
            color: #81c784;
            margin-bottom: 5px;
        }

        .physics-fact {
            background: rgba(76, 175, 80, 0.1);
            border: 1px solid rgba(76, 175, 80, 0.3);
            border-radius: 6px;
            padding: 10px;
            margin: 10px 0;
        }

        .physics-fact h5 {
            color: #81c784;
            margin-bottom: 5px;
        }

        .measurement {
            font-family: 'Courier New', monospace;
            color: #ffeb3b;
            font-weight: bold;
        }

        #stars {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }

        .loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            color: #64b5f6;
        }

        .orbit-trail {
            stroke: rgba(100, 181, 246, 0.3);
            stroke-width: 1;
            fill: none;
            stroke-dasharray: 2, 4;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="stars"></canvas>
        <canvas id="canvas"></canvas>
        
        <div class="ui-panel" id="controlPanel">
            <div class="control-group">
                <h3>🚀 Navigation</h3>
                <div class="slider-container">
                    <span>Zoom:</span>
                    <input type="range" class="slider" id="zoomSlider" min="0.1" max="5" step="0.1" value="1">
                    <span id="zoomValue">1.0x</span>
                </div>
                <div class="slider-container">
                    <span>Speed:</span>
                    <input type="range" class="slider" id="speedSlider" min="0" max="10" step="0.5" value="1">
                    <span id="speedValue">1.0x</span>
                </div>
            </div>
            
            <div class="control-group">
                <h3>🌍 Focus Target</h3>
                <button class="button" id="focusSun">Sun</button>
                <button class="button" id="focusEarth">Earth</button>
                <button class="button" id="focusMoon">Moon</button>
                <button class="button" id="focusMars">Mars</button>
            </div>
            
            <div class="control-group">
                <h3>⚙️ Display Options</h3>
                <button class="button active" id="toggleOrbits">Show Orbits</button>
                <button class="button active" id="toggleLabels">Show Labels</button>
                <button class="button" id="toggleTrails">Show Trails</button>
                <button class="button" id="pausePlay">⏸️ Pause</button>
            </div>
        </div>

        <div class="ui-panel" id="infoPanel">
            <h3 style="color: #64b5f6; margin-bottom: 15px;">📊 Celestial Information</h3>
            <div id="celestialInfo">
                <div class="celestial-info">
                    <h4>Welcome to Solar System Explorer!</h4>
                    <p>Click and drag to rotate the view. Use the controls to explore our cosmic neighborhood.</p>
                </div>
            </div>
            
            <div class="physics-fact">
                <h5>🔬 Physics Fact</h5>
                <p id="physicsFact">Orbital velocity decreases with distance from the Sun due to conservation of angular momentum!</p>
            </div>
        </div>

        <div class="ui-panel" id="bottomPanel">
            <button class="button" id="measureTool">📏 Measure Distance</button>
            <button class="button" id="resetView">🔄 Reset View</button>
            <button class="button" id="randomFact">💡 Random Fact</button>
        </div>
    </div>

    <script>
        class SolarSystemExplorer {
            constructor() {
                this.canvas = document.getElementById('canvas');
                this.ctx = this.canvas.getContext('2d');
                this.starsCanvas = document.getElementById('stars');
                this.starsCtx = this.starsCanvas.getContext('2d');
                
                this.setupCanvas();
                this.setupStars();
                
                // Camera and interaction
                this.camera = { x: 0, y: 0, zoom: 1, rotation: 0 };
                this.mouse = { x: 0, y: 0, down: false, lastX: 0, lastY: 0 };
                this.focusTarget = null;
                
                // Game state
                this.time = 0;
                this.timeSpeed = 1;
                this.paused = false;
                this.showOrbits = true;
                this.showLabels = true;
                this.showTrails = false;
                this.measureMode = false;
                this.measurePoints = [];
                
                // Physics constants (scaled for visualization)
                this.AU = 150; // Astronomical Unit in pixels
                this.G = 6.67e-11; // Gravitational constant (scaled)
                
                this.setupCelestialBodies();
                this.setupEventListeners();
                this.setupPhysicsFacts();
                
                this.animate();
            }
            
            setupCanvas() {
                const resize = () => {
                    this.canvas.width = window.innerWidth;
                    this.canvas.height = window.innerHeight;
                    this.starsCanvas.width = window.innerWidth;
                    this.starsCanvas.height = window.innerHeight;
                };
                resize();
                window.addEventListener('resize', resize);
            }
            
            setupStars() {
                const stars = [];
                for (let i = 0; i < 200; i++) {
                    stars.push({
                        x: Math.random() * this.starsCanvas.width,
                        y: Math.random() * this.starsCanvas.height,
                        size: Math.random() * 2,
                        brightness: Math.random()
                    });
                }
                
                this.starsCtx.fillStyle = '#000011';
                this.starsCtx.fillRect(0, 0, this.starsCanvas.width, this.starsCanvas.height);
                
                stars.forEach(star => {
                    this.starsCtx.fillStyle = `rgba(255, 255, 255, ${star.brightness})`;
                    this.starsCtx.beginPath();
                    this.starsCtx.arc(star.x, star.y, star.size, 0, Math.PI * 2);
                    this.starsCtx.fill();
                });
            }
            
            setupCelestialBodies() {
                this.bodies = {
                    sun: {
                        name: 'Sun',
                        x: 0, y: 0,
                        radius: 20,
                        color: '#FDB813',
                        mass: 1.989e30,
                        info: {
                            diameter: '1.39 million km',
                            mass: '1.989 × 10³⁰ kg',
                            temperature: '5,778 K surface',
                            composition: '73% Hydrogen, 25% Helium'
                        }
                    },
                    earth: {
                        name: 'Earth',
                        x: this.AU, y: 0,
                        radius: 6,
                        color: '#6B93D6',
                        mass: 5.972e24,
                        orbitRadius: this.AU,
                        orbitSpeed: 0.02,
                        angle: 0,
                        info: {
                            diameter: '12,742 km',
                            mass: '5.972 × 10²⁴ kg',
                            distance: '149.6 million km from Sun',
                            day: '24 hours',
                            year: '365.25 days'
                        }
                    },
                    moon: {
                        name: 'Moon',
                        x: this.AU + 30, y: 0,
                        radius: 2,
                        color: '#C0C0C0',
                        mass: 7.342e22,
                        orbitRadius: 30,
                        orbitSpeed: 0.15,
                        angle: 0,
                        parent: 'earth',
                        info: {
                            diameter: '3,474 km',
                            mass: '7.342 × 10²² kg',
                            distance: '384,400 km from Earth',
                            day: '27.3 Earth days',
                            tidally_locked: true
                        }
                    },
                    mars: {
                        name: 'Mars',
                        x: this.AU * 1.5, y: 0,
                        radius: 4,
                        color: '#CD5C5C',
                        mass: 6.39e23,
                        orbitRadius: this.AU * 1.5,
                        orbitSpeed: 0.015,
                        angle: Math.PI,
                        info: {
                            diameter: '6,779 km',
                            mass: '6.39 × 10²³ kg',
                            distance: '227.9 million km from Sun',
                            day: '24.6 hours',
                            year: '687 Earth days'
                        }
                    }
                };
                
                this.trails = {
                    earth: [],
                    moon: [],
                    mars: []
                };
            }
            
            setupPhysicsFacts() {
                this.physicsFacts = [
                    "Kepler's First Law: Planets orbit in ellipses with the Sun at one focus.",
                    "Orbital velocity decreases with distance: v = √(GM/r)",
                    "The Moon's gravity causes Earth's tides through tidal forces.",
                    "Mars has the largest volcano in the solar system: Olympus Mons.",
                    "Earth's magnetic field protects us from harmful solar radiation.",
                    "The Moon is slowly moving away from Earth at 3.8 cm per year.",
                    "Mars' day is very similar to Earth's: 24 hours 37 minutes.",
                    "Gravitational force follows inverse square law: F ∝ 1/r²",
                    "Angular momentum is conserved in orbital motion.",
                    "Escape velocity from Earth is 11.2 km/s (25,000 mph)."
                ];
                this.currentFactIndex = 0;
            }
            
            setupEventListeners() {
                // Mouse controls
                this.canvas.addEventListener('mousedown', (e) => {
                    this.mouse.down = true;
                    this.mouse.lastX = e.clientX;
                    this.mouse.lastY = e.clientY;
                    
                    if (this.measureMode) {
                        const worldPos = this.screenToWorld(e.clientX, e.clientY);
                        this.measurePoints.push(worldPos);
                        if (this.measurePoints.length === 2) {
                            this.calculateDistance();
                            this.measureMode = false;
                            document.getElementById('measureTool').classList.remove('active');
                        }
                    }
                });
                
                this.canvas.addEventListener('mousemove', (e) => {
                    if (this.mouse.down && !this.measureMode) {
                        const deltaX = e.clientX - this.mouse.lastX;
                        const deltaY = e.clientY - this.mouse.lastY;
                        
                        this.camera.x += deltaX / this.camera.zoom;
                        this.camera.y += deltaY / this.camera.zoom;
                        
                        this.mouse.lastX = e.clientX;
                        this.mouse.lastY = e.clientY;
                    }
                });
                
                this.canvas.addEventListener('mouseup', () => {
                    this.mouse.down = false;
                });
                
                this.canvas.addEventListener('wheel', (e) => {
                    e.preventDefault();
                    const zoomFactor = e.deltaY > 0 ? 0.9 : 1.1;
                    this.camera.zoom = Math.max(0.1, Math.min(5, this.camera.zoom * zoomFactor));
                    document.getElementById('zoomSlider').value = this.camera.zoom;
                    document.getElementById('zoomValue').textContent = this.camera.zoom.toFixed(1) + 'x';
                });
                
                // UI Controls
                document.getElementById('zoomSlider').addEventListener('input', (e) => {
                    this.camera.zoom = parseFloat(e.target.value);
                    document.getElementById('zoomValue').textContent = this.camera.zoom.toFixed(1) + 'x';
                });
                
                document.getElementById('speedSlider').addEventListener('input', (e) => {
                    this.timeSpeed = parseFloat(e.target.value);
                    document.getElementById('speedValue').textContent = this.timeSpeed.toFixed(1) + 'x';
                });
                
                // Focus buttons
                document.getElementById('focusSun').addEventListener('click', () => this.focusOn('sun'));
                document.getElementById('focusEarth').addEventListener('click', () => this.focusOn('earth'));
                document.getElementById('focusMoon').addEventListener('click', () => this.focusOn('moon'));
                document.getElementById('focusMars').addEventListener('click', () => this.focusOn('mars'));
                
                // Toggle buttons
                document.getElementById('toggleOrbits').addEventListener('click', (e) => {
                    this.showOrbits = !this.showOrbits;
                    e.target.classList.toggle('active');
                });
                
                document.getElementById('toggleLabels').addEventListener('click', (e) => {
                    this.showLabels = !this.showLabels;
                    e.target.classList.toggle('active');
                });
                
                document.getElementById('toggleTrails').addEventListener('click', (e) => {
                    this.showTrails = !this.showTrails;
                    e.target.classList.toggle('active');
                    if (!this.showTrails) {
                        Object.keys(this.trails).forEach(key => this.trails[key] = []);
                    }
                });
                
                document.getElementById('pausePlay').addEventListener('click', (e) => {
                    this.paused = !this.paused;
                    e.target.textContent = this.paused ? '▶️ Play' : '⏸️ Pause';
                });
                
                document.getElementById('measureTool').addEventListener('click', (e) => {
                    this.measureMode = !this.measureMode;
                    e.target.classList.toggle('active');
                    this.measurePoints = [];
                });
                
                document.getElementById('resetView').addEventListener('click', () => {
                    this.camera = { x: 0, y: 0, zoom: 1, rotation: 0 };
                    this.focusTarget = null;
                    document.getElementById('zoomSlider').value = 1;
                    document.getElementById('zoomValue').textContent = '1.0x';
                });
                
                document.getElementById('randomFact').addEventListener('click', () => {
                    this.showRandomFact();
                });
            }
            
            focusOn(bodyName) {
                this.focusTarget = bodyName;
                document.querySelectorAll('#controlPanel .button').forEach(btn => btn.classList.remove('active'));
                document.getElementById(`focus${bodyName.charAt(0).toUpperCase() + bodyName.slice(1)}`).classList.add('active');
                
                this.updateInfoPanel(this.bodies[bodyName]);
            }
            
            updateInfoPanel(body) {
                const infoDiv = document.getElementById('celestialInfo');
                let html = `
                    <div class="celestial-info">
                        <h4>${body.name}</h4>
                `;
                
                Object.entries(body.info).forEach(([key, value]) => {
                    const label = key.replace(/_/g, ' ').replace(/\b\w/g, l => l.toUpperCase());
                    html += `<p><strong>${label}:</strong> <span class="measurement">${value}</span></p>`;
                });
                
                html += '</div>';
                infoDiv.innerHTML = html;
            }
            
            showRandomFact() {
                const fact = this.physicsFacts[this.currentFactIndex];
                document.getElementById('physicsFact').textContent = fact;
                this.currentFactIndex = (this.currentFactIndex + 1) % this.physicsFacts.length;
            }
            
            screenToWorld(screenX, screenY) {
                const centerX = this.canvas.width / 2;
                const centerY = this.canvas.height / 2;
                
                return {
                    x: (screenX - centerX - this.camera.x) / this.camera.zoom,
                    y: (screenY - centerY - this.camera.y) / this.camera.zoom
                };
            }
            
            worldToScreen(worldX, worldY) {
                const centerX = this.canvas.width / 2;
                const centerY = this.canvas.height / 2;
                
                return {
                    x: worldX * this.camera.zoom + this.camera.x + centerX,
                    y: worldY * this.camera.zoom + this.camera.y + centerY
                };
            }
            
            calculateDistance() {
                if (this.measurePoints.length === 2) {
                    const dx = this.measurePoints[1].x - this.measurePoints[0].x;
                    const dy = this.measurePoints[1].y - this.measurePoints[0].y;
                    const distance = Math.sqrt(dx * dx + dy * dy);
                    const kmDistance = (distance / this.AU) * 149.6; // Convert to million km
                    
                    alert(`Distance: ${kmDistance.toFixed(2)} million km\n(${distance.toFixed(0)} pixels in simulation)`);
                    this.measurePoints = [];
                }
            }
            
            updatePhysics() {
                if (this.paused) return;
                
                this.time += this.timeSpeed * 0.016; // 60 FPS
                
                // Update orbital positions
                Object.keys(this.bodies).forEach(key => {
                    const body = this.bodies[key];
                    if (body.orbitRadius) {
                        body.angle += body.orbitSpeed * this.timeSpeed;
                        
                        if (body.parent) {
                            const parent = this.bodies[body.parent];
                            body.x = parent.x + Math.cos(body.angle) * body.orbitRadius;
                            body.y = parent.y + Math.sin(body.angle) * body.orbitRadius;
                        } else {
                            body.x = Math.cos(body.angle) * body.orbitRadius;
                            body.y = Math.sin(body.angle) * body.orbitRadius;
                        }
                        
                        // Add to trails
                        if (this.showTrails && this.trails[key]) {
                            this.trails[key].push({ x: body.x, y: body.y });
                            if (this.trails[key].length > 100) {
                                this.trails[key].shift();
                            }
                        }
                    }
                });
                
                // Update camera focus
                if (this.focusTarget && this.bodies[this.focusTarget]) {
                    const target = this.bodies[this.focusTarget];
                    this.camera.x = -target.x * this.camera.zoom;
                    this.camera.y = -target.y * this.camera.zoom;
                }
            }
            
            render() {
                this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
                
                const centerX = this.canvas.width / 2;
                const centerY = this.canvas.height / 2;
                
                this.ctx.save();
                this.ctx.translate(centerX + this.camera.x, centerY + this.camera.y);
                this.ctx.scale(this.camera.zoom, this.camera.zoom);
                
                // Draw orbits
                if (this.showOrbits) {
                    this.ctx.strokeStyle = 'rgba(100, 181, 246, 0.3)';
                    this.ctx.lineWidth = 1 / this.camera.zoom;
                    this.ctx.setLineDash([5 / this.camera.zoom, 10 / this.camera.zoom]);
                    
                    // Earth orbit
                    this.ctx.beginPath();
                    this.ctx.arc(0, 0, this.AU, 0, Math.PI * 2);
                    this.ctx.stroke();
                    
                    // Mars orbit
                    this.ctx.beginPath();
                    this.ctx.arc(0, 0, this.AU * 1.5, 0, Math.PI * 2);
                    this.ctx.stroke();
                    
                    // Moon orbit around Earth
                    const earth = this.bodies.earth;
                    this.ctx.beginPath();
                    this.ctx.arc(earth.x, earth.y, 30, 0, Math.PI * 2);
                    this.ctx.stroke();
                    
                    this.ctx.setLineDash([]);
                }
                
                // Draw trails
                if (this.showTrails) {
                    Object.keys(this.trails).forEach(key => {
                        const trail = this.trails[key];
                        if (trail.length > 1) {
                            this.ctx.strokeStyle = this.bodies[key].color + '80';
                            this.ctx.lineWidth = 2 / this.camera.zoom;
                            this.ctx.beginPath();
                            this.ctx.moveTo(trail[0].x, trail[0].y);
                            for (let i = 1; i < trail.length; i++) {
                                this.ctx.lineTo(trail[i].x, trail[i].y);
                            }
                            this.ctx.stroke();
                        }
                    });
                }
                
                // Draw celestial bodies
                Object.values(this.bodies).forEach(body => {
                    // Body
                    this.ctx.fillStyle = body.color;
                    this.ctx.beginPath();
                    this.ctx.arc(body.x, body.y, body.radius, 0, Math.PI * 2);
                    this.ctx.fill();
                    
                    // Glow effect for Sun
                    if (body.name === 'Sun') {
                        const gradient = this.ctx.createRadialGradient(body.x, body.y, body.radius, body.x, body.y, body.radius * 2);
                        gradient.addColorStop(0, body.color + '40');
                        gradient.addColorStop(1, 'transparent');
                        this.ctx.fillStyle = gradient;
                        this.ctx.beginPath();
                        this.ctx.arc(body.x, body.y, body.radius * 2, 0, Math.PI * 2);
                        this.ctx.fill();
                    }
                    
                    // Labels
                    if (this.showLabels) {
                        this.ctx.fillStyle = 'white';
                        this.ctx.font = `${12 / this.camera.zoom}px Arial`;
                        this.ctx.textAlign = 'center';
                        this.ctx.fillText(body.name, body.x, body.y - body.radius - 10 / this.camera.zoom);
                    }
                });
                
                // Draw measurement points
                if (this.measureMode && this.measurePoints.length > 0) {
                    this.ctx.fillStyle = '#ffeb3b';
                    this.measurePoints.forEach(point => {
                        this.ctx.beginPath();
                        this.ctx.arc(point.x, point.y, 3 / this.camera.zoom, 0, Math.PI * 2);
                        this.ctx.fill();
                    });
                    
                    if (this.measurePoints.length === 2) {
                        this.ctx.strokeStyle = '#ffeb3b';
                        this.ctx.lineWidth = 2 / this.camera.zoom;
                        this.ctx.beginPath();
                        this.ctx.moveTo(this.measurePoints[0].x, this.measurePoints[0].y);
                        this.ctx.lineTo(this.measurePoints[1].x, this.measurePoints[1].y);
                        this.ctx.stroke();
                    }
                }
                
                this.ctx.restore();
            }
            
            animate() {
                this.updatePhysics();
                this.render();
                requestAnimationFrame(() => this.animate());
            }
        }
        
        // Initialize the game
        window.addEventListener('load', () => {
            new SolarSystemExplorer();
        });
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'96e0c43e10343b67',t:'MTc1NTAxMDAzMi4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
