<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Point Light Source & Oblique Incidence Interactive</title>
    <style>
        :root {
            --primary: #2563eb;
            --bg: #f8fafc;
            --panel-bg: #ffffff;
            --text: #1e293b;
            --accent: #f59e0b;
        }

        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }

        header {
            background: var(--primary);
            color: white;
            width: 100%;
            padding: 1rem;
            text-align: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .container {
            display: grid;
            grid-template-columns: 300px 1fr;
            gap: 20px;
            max-width: 1200px;
            width: 95%;
            margin: 20px 0;
        }

        /* Control Panel */
        .controls {
            background: var(--panel-bg);
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
        }

        .control-group {
            margin-bottom: 20px;
        }

        label {
            display: block;
            font-weight: 600;
            margin-bottom: 5px;
            font-size: 0.9rem;
        }

        input[type="number"], input[type="range"] {
            width: 100%;
            box-sizing: border-box;
        }

        input[type="number"] {
            padding: 8px;
            border: 1px solid #cbd5e1;
            border-radius: 4px;
            margin-bottom: 5px;
        }

        .unit {
            font-size: 0.8rem;
            color: #64748b;
            text-align: right;
        }

        /* Visualization Area */
        .viz-container {
            background: var(--panel-bg);
            border-radius: 8px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        canvas {
            width: 100%;
            height: 400px;
            background: linear-gradient(to bottom, #f1f5f9 0%, #e2e8f0 100%);
            cursor: crosshair;
            touch-action: none; /* Prevent scrolling on mobile while dragging */
        }

        .results-bar {
            padding: 20px;
            background: #fff;
            border-top: 1px solid #e2e8f0;
        }

        .main-result {
            font-size: 1.5rem;
            font-weight: bold;
            color: var(--primary);
            margin-bottom: 10px;
        }

        .equation-display {
            font-family: 'Courier New', Courier, monospace;
            background: #f1f5f9;
            padding: 15px;
            border-radius: 4px;
            border-left: 4px solid var(--accent);
            font-size: 0.95rem;
            overflow-x: auto;
        }

        .equation-display span.highlight {
            color: var(--primary);
            font-weight: bold;
        }

        .instructions {
            font-size: 0.85rem;
            color: #64748b;
            margin-top: 10px;
            font-style: italic;
        }

        @media (max-width: 768px) {
            .container {
                grid-template-columns: 1fr;
            }
            canvas {
                height: 300px;
            }
        }
    </style>
</head>
<body>

<header>
    <h2>Illuminance Calculator: Point Source</h2>
</header>

<div class="container">
    <!-- Left: Controls -->
    <div class="controls">
        <div class="control-group">
            <label for="flux">Luminous Flux (Φ)</label>
            <input type="number" id="flux" value="1000" step="100">
            <div class="unit">Lumens (lm)</div>
        </div>

        <div class="control-group">
            <label for="height">Source Height (d)</label>
            <input type="number" id="height" value="3.0" step="0.1" min="0.1">
            <input type="range" id="heightRange" min="0.5" max="10" step="0.1" value="3.0">
            <div class="unit">Meters (m)</div>
        </div>

        <div class="control-group">
            <label for="distance">Floor Distance (x)</label>
            <input type="number" id="distance" value="2.0" step="0.1" min="0">
            <input type="range" id="distanceRange" min="0" max="10" step="0.1" value="2.0">
            <div class="unit">Meters (m) from vertical axis</div>
        </div>

        <div class="instructions">
            <strong>Interactive:</strong> Drag the yellow sun (Source) up/down or the black dot (Target) left/right on the diagram.
        </div>
    </div>

    <!-- Right: Visualization & Results -->
    <div class="viz-container">
        <canvas id="simCanvas"></canvas>
        
        <div class="results-bar">
            <div class="main-result">Illuminance (E) = <span id="resE">0</span> Lux</div>
            
            <div style="display:flex; gap: 20px; margin-bottom: 10px; flex-wrap:wrap;">
                <div>Angle (&theta;): <strong id="resTheta">0</strong>&deg;</div>
                <div>Hypotenuse (r): <strong id="resR">0</strong> m</div>
                <div>cos &theta;: <strong id="resCos">0</strong></div>
            </div>

            <div class="equation-display" id="liveEquation">
                <!-- Formula will be injected here via JS -->
            </div>
        </div>
    </div>
</div>

<script>
    // Physics State
    const state = {
        phi: 1000, // Lumens
        d: 3.0,    // Height (m)
        x: 2.0,    // Horizontal distance (m)
        
        // Calculated values
        r: 0,
        thetaRad: 0,
        thetaDeg: 0,
        E: 0
    };

    // UI Elements
    const canvas = document.getElementById('simCanvas');
    const ctx = canvas.getContext('2d');
    
    const inputs = {
        phi: document.getElementById('flux'),
        d: document.getElementById('height'),
        x: document.getElementById('distance'),
        dRange: document.getElementById('heightRange'),
        xRange: document.getElementById('distanceRange')
    };

    const outputs = {
        E: document.getElementById('resE'),
        theta: document.getElementById('resTheta'),
        r: document.getElementById('resR'),
        cos: document.getElementById('resCos'),
        eq: document.getElementById('liveEquation')
    };

    // Canvas Logic Variables
    let dragging = null; // 'source' or 'target'
    let pixelsPerMeter = 40; // Scale factor
    const margin = 40; // Canvas margin

    // Initialization
    function init() {
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);
        
        // Add Event Listeners to Inputs
        inputs.phi.addEventListener('input', (e) => updateState('phi', e.target.value));
        inputs.d.addEventListener('input', (e) => updateState('d', e.target.value));
        inputs.x.addEventListener('input', (e) => updateState('x', e.target.value));
        
        // Sync Ranges
        inputs.dRange.addEventListener('input', (e) => updateState('d', e.target.value));
        inputs.xRange.addEventListener('input', (e) => updateState('x', e.target.value));

        // Canvas Interaction
        canvas.addEventListener('mousedown', handleMouseDown);
        canvas.addEventListener('mousemove', handleMouseMove);
        window.addEventListener('mouseup', () => dragging = null);
        
        // Touch support
        canvas.addEventListener('touchstart', handleTouchStart, {passive: false});
        canvas.addEventListener('touchmove', handleTouchMove, {passive: false});
        window.addEventListener('touchend', () => dragging = null);

        calculate();
    }

    function resizeCanvas() {
        const rect = canvas.parentElement.getBoundingClientRect();
        canvas.width = rect.width;
        canvas.height = 400; // Fixed height
        draw();
    }

    function updateState(key, value) {
        let val = parseFloat(value);
        if (isNaN(val)) return;
        if (val < 0) val = 0;
        
        state[key] = val;

        // Sync inputs if changed by drag or range
        if(key === 'd') {
            inputs.d.value = val.toFixed(2);
            inputs.dRange.value = val;
        }
        if(key === 'x') {
            inputs.x.value = val.toFixed(2);
            inputs.xRange.value = val;
        }
        if(key === 'phi') {
            inputs.phi.value = val;
        }

        calculate();
    }

    // Core Physics Engine
    function calculate() {
        // 1. Calculate Geometry
        state.r = Math.sqrt(state.d * state.d + state.x * state.x);
        state.thetaRad = Math.atan2(state.x, state.d);
        state.thetaDeg = state.thetaRad * (180 / Math.PI);
        
        const cosTheta = state.d / state.r;

        // 2. Calculate Illuminance
        // Formula: E = (Phi / (4 * pi * r^2)) * cos(theta)
        // Which is equivalent to the textbook formula: E = (Phi / 4*pi*d^2) * cos^3(theta)
        // Note: 4*pi assumes isotropic spherical source. 
        
        // Prevent division by zero
        if (state.r === 0) {
            state.E = 0; 
        } else {
            const area = 4 * Math.PI * Math.pow(state.r, 2);
            state.E = (state.phi / area) * cosTheta;
        }

        updateUI(cosTheta);
        draw();
    }

    function updateUI(cosTheta) {
        outputs.E.textContent = state.E.toFixed(2);
        outputs.theta.textContent = state.thetaDeg.toFixed(1);
        outputs.r.textContent = state.r.toFixed(2);
        outputs.cos.textContent = cosTheta.toFixed(3);

        // Live Equation Update
        // Using the d^2 * cos^3 version for visual matching with textbook
        // E = Phi / (4π d²) * cos³θ
        const cos3 = Math.pow(cosTheta, 3).toFixed(4);
        const denom = (4 * Math.PI * state.d * state.d).toFixed(1);
        
        let html = `E = <span class="highlight">${state.phi}</span> / (4π · <span class="highlight">${state.d}</span>²) × cos³(<span class="highlight">${state.thetaDeg.toFixed(1)}°</span>)<br>`;
        html += `&nbsp;&nbsp;= ${state.phi} / ${denom} × ${cos3}<br>`;
        html += `&nbsp;&nbsp;= <strong>${state.E.toFixed(2)} lx</strong>`;
        
        outputs.eq.innerHTML = html;
    }

    // Visualization Engine
    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Dynamic Scaling to fit scene
        // We want the Source and Target to be visible with some padding
        const maxH = Math.max(state.d, 1);
        const maxW = Math.max(state.x, 1);
        
        // Calculate scale factor (pixels per meter)
        const padX = 100; 
        const padY = 80;
        const scaleY = (canvas.height - padY) / (maxH * 1.2);
        const scaleX = (canvas.width - padX) / (maxW * 1.5);
        pixelsPerMeter = Math.min(scaleX, scaleY, 80); // Cap max zoom

        // Origin (Floor Point under Source)
        const originX = canvas.width / 3; // Keep slightly left
        const originY = canvas.height - 40; // Near bottom

        // Coordinates in Pixels
        const sourceX = originX;
        const sourceY = originY - (state.d * pixelsPerMeter);
        const targetX = originX + (state.x * pixelsPerMeter);
        const targetY = originY;

        // 1. Draw Floor
        ctx.beginPath();
        ctx.moveTo(0, originY);
        ctx.lineTo(canvas.width, originY);
        ctx.strokeStyle = '#94a3b8';
        ctx.lineWidth = 4;
        ctx.stroke();

        // 2. Draw Normal Line (Dashed)
        ctx.beginPath();
        ctx.setLineDash([5, 5]);
        ctx.moveTo(targetX, targetY);
        ctx.lineTo(targetX, targetY - 50); // Short normal
        ctx.strokeStyle = '#64748b';
        ctx.lineWidth = 1;
        ctx.stroke();
        
        // Label "Normal"
        ctx.fillStyle = '#64748b';
        ctx.font = '12px sans-serif';
        ctx.fillText('Normal', targetX - 20, targetY - 55);

        // 3. Draw Vertical Reference (d line)
        ctx.beginPath();
        ctx.setLineDash([5, 5]);
        ctx.moveTo(originX, originY);
        ctx.lineTo(sourceX, sourceY);
        ctx.strokeStyle = '#64748b';
        ctx.stroke();

        // Label d
        ctx.fillText(`d = ${state.d}m`, originX - 60, (originY + sourceY)/2);

        // 4. Draw Light Ray (Hypotenuse)
        ctx.beginPath();
        ctx.setLineDash([]); // Solid
        ctx.moveTo(sourceX, sourceY);
        ctx.lineTo(targetX, targetY);
        ctx.strokeStyle = '#f59e0b'; // Orange/Yellow
        ctx.lineWidth = 2;
        ctx.stroke();

        // Label r
        ctx.fillStyle = '#b45309';
        ctx.fillText(`r = ${state.r.toFixed(2)}m`, (sourceX + targetX)/2 + 10, (sourceY + targetY)/2);

        // 5. Draw Angle Arc
        // Calculate angle at target from vertical normal? 
        // The formula uses angle of incidence theta. 
        // Theta is between Normal and Light Ray.
        const rayAngle = Math.atan2(sourceY - targetY, sourceX - targetX); // Angle of ray
        const normalAngle = -Math.PI / 2; // Straight up
        
        // Draw Angle Arc at Target (theta)
        // Normal is vertical up (-90 deg). Ray comes from top-left.
        // Angle is between Vertical Up and the Ray.
        ctx.beginPath();
        const arcRadius = 40;
        // Start at vertical (normal)
        ctx.arc(targetX, targetY, arcRadius, 1.5 * Math.PI, 1.5 * Math.PI - state.thetaRad, true);
        ctx.strokeStyle = '#000';
        ctx.lineWidth = 1;
        ctx.stroke();
        
        // Label Theta
        ctx.fillStyle = '#000';
        ctx.fillText(`θ`, targetX - 15, targetY - 50);

        // 6. Draw Source (Interactive)
        ctx.beginPath();
        ctx.arc(sourceX, sourceY, 15, 0, 2 * Math.PI);
        const grad = ctx.createRadialGradient(sourceX, sourceY, 2, sourceX, sourceY, 15);
        grad.addColorStop(0, '#fff');
        grad.addColorStop(1, '#f59e0b');
        ctx.fillStyle = grad;
        ctx.fill();
        ctx.stroke();
        
        // Source Label
        ctx.fillStyle = '#000';
        ctx.fillText('Source', sourceX + 20, sourceY);

        // 7. Draw Target (Interactive)
        ctx.beginPath();
        ctx.arc(targetX, targetY, 8, 0, 2 * Math.PI);
        ctx.fillStyle = '#1e293b';
        ctx.fill();
        
        // Visualize Intensity at Target (Glow effect based on E)
        // Normalize visualization max (arbitrary visually pleasing cap, e.g., 500 lux)
        const intensityAlpha = Math.min(state.E / 200, 1); 
        ctx.beginPath();
        ctx.arc(targetX, targetY, 25, 0, 2 * Math.PI);
        ctx.fillStyle = `rgba(255, 255, 0, ${intensityAlpha})`;
        ctx.fill();

        ctx.fillStyle = '#1e293b';
        ctx.fillText('X', targetX - 5, targetY + 20);

        // Helper for floor distance x
        ctx.beginPath();
        ctx.moveTo(originX, originY + 10);
        ctx.lineTo(targetX, originY + 10);
        ctx.strokeStyle = '#333';
        ctx.setLineDash([]);
        ctx.stroke();
        ctx.fillText(`x = ${state.x}m`, (originX + targetX)/2, originY + 25);
    }

    // Interaction Logic
    function getMousePos(evt) {
        const rect = canvas.getBoundingClientRect();
        const clientX = evt.clientX || evt.touches[0].clientX;
        const clientY = evt.clientY || evt.touches[0].clientY;
        return {
            x: clientX - rect.left,
            y: clientY - rect.top
        };
    }

    function handleMouseDown(e) {
        const pos = getMousePos(e);
        
        // Check distance to Source
        // Need to reverse calculate pixel positions
        const originX = canvas.width / 3; 
        const originY = canvas.height - 40;
        const sourceY = originY - (state.d * pixelsPerMeter);
        
        const distToSource = Math.hypot(pos.x - originX, pos.y - sourceY);
        if (distToSource < 30) {
            dragging = 'source';
            return;
        }

        // Check distance to Target
        const targetX = originX + (state.x * pixelsPerMeter);
        const distToTarget = Math.hypot(pos.x - targetX, pos.y - originY);
        if (distToTarget < 30) {
            dragging = 'target';
            return;
        }
    }

    function handleMouseMove(e) {
        if (!dragging) return;
        e.preventDefault();
        
        const pos = getMousePos(e);
        const originX = canvas.width / 3; 
        const originY = canvas.height - 40;

        if (dragging === 'source') {
            // Only allow vertical movement, clamp above floor
            let newPixelHeight = originY - pos.y;
            let newD = newPixelHeight / pixelsPerMeter;
            updateState('d', Math.max(0.1, newD)); // Min height 0.1m
        } else if (dragging === 'target') {
            // Only allow horizontal movement to the right
            let newPixelDist = pos.x - originX;
            let newX = newPixelDist / pixelsPerMeter;
            updateState('x', Math.max(0, newX));
        }
    }

    function handleTouchStart(e) { handleMouseDown(e); }
    function handleTouchMove(e) { handleMouseMove(e); }

    // Run
    init();

</script>

</body>
</html>
