<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Illuminance Interactive Tool</title>
    <style>
        :root {
            --primary: #2563eb;
            --bg: #f1f5f9;
            --card-bg: #ffffff;
            --text-main: #1e293b;
            --text-sub: #64748b;
        }

        body {
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg);
            color: var(--text-main);
            display: flex;
            flex-direction: column;
            min-height: 100vh;
        }

        /* Header */
        header {
            background: var(--primary);
            color: white;
            padding: 1rem;
            text-align: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        header h1 { margin: 0; font-size: 1.25rem; }

        /* Main Layout */
        .app-container {
            display: flex;
            flex-wrap: wrap; /* Allows stacking on small screens */
            gap: 20px;
            padding: 20px;
            max-width: 1200px;
            margin: 0 auto;
            width: 100%;
            box-sizing: border-box;
            justify-content: center;
        }

        /* Diagram Section (Visual) */
        .viz-card {
            flex: 1 1 500px; /* Grows to fill space, min-width 500px if possible */
            background: var(--card-bg);
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            display: flex;
            flex-direction: column;
            overflow: hidden;
            min-height: 400px;
            position: relative;
        }

        .canvas-container {
            flex-grow: 1;
            position: relative;
            background: #ffffff;
            /* subtle grid pattern */
            background-image: radial-gradient(#e2e8f0 1px, transparent 1px);
            background-size: 20px 20px;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
            cursor: crosshair;
        }

        .results-panel {
            padding: 15px 20px;
            background: #f8fafc;
            border-top: 1px solid #e2e8f0;
        }

        /* Controls Section (Inputs) */
        .controls-card {
            flex: 0 0 320px; /* Fixed width for controls */
            background: var(--card-bg);
            border-radius: 12px;
            padding: 24px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            height: fit-content;
        }

        .control-group { margin-bottom: 24px; }
        
        label {
            display: flex;
            justify-content: space-between;
            font-weight: 600;
            font-size: 0.9rem;
            margin-bottom: 8px;
            color: var(--text-main);
        }

        .input-row {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        input[type="number"] {
            width: 80px;
            padding: 8px;
            border: 1px solid #cbd5e1;
            border-radius: 6px;
            font-size: 1rem;
        }

        input[type="range"] {
            flex-grow: 1;
            cursor: pointer;
        }

        /* Results Typography */
        .stat-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
            gap: 10px;
            margin-bottom: 10px;
        }
        .stat-item {
            font-size: 0.85rem;
            color: var(--text-sub);
        }
        .stat-value {
            font-weight: bold;
            color: var(--text-main);
            font-size: 1rem;
        }

        .highlight-box {
            background: #e0f2fe;
            color: #0369a1;
            padding: 12px;
            border-radius: 6px;
            font-family: monospace;
            font-size: 0.95rem;
            margin-top: 10px;
            overflow-x: auto;
            border-left: 4px solid var(--primary);
        }

        /* Mobile Adjustments */
        @media (max-width: 850px) {
            .app-container { padding: 10px; }
            .viz-card { flex: 1 1 100%; min-height: 350px; }
            .controls-card { flex: 1 1 100%; }
        }
    </style>
</head>
<body>

<header>
    <h1>Illuminance Calculator: Point Source</h1>
</header>

<div class="app-container">
    
    <!-- 1. Visualization Area -->
    <div class="viz-card">
        <div class="canvas-container" id="canvasWrapper">
            <canvas id="simCanvas"></canvas>
            <!-- Overlay hint -->
            <div style="position: absolute; top: 10px; left: 10px; font-size: 12px; color: #94a3b8; pointer-events: none;">
                Drag the 🟠 Source or the ⚫ Target
            </div>
        </div>
        
        <div class="results-panel">
            <div class="stat-grid">
                <div class="stat-item">Illuminance (E)<br><span class="stat-value" id="dispE" style="color: var(--primary); font-size: 1.2rem;">0 lx</span></div>
                <div class="stat-item">Angle (&theta;)<br><span class="stat-value" id="dispTheta">0&deg;</span></div>
                <div class="stat-item">Hypotenuse (r)<br><span class="stat-value" id="dispR">0 m</span></div>
                <div class="stat-item">Cos &theta;<br><span class="stat-value" id="dispCos">0</span></div>
            </div>
            <div class="highlight-box" id="formulaBox">
                Loading formula...
            </div>
        </div>
    </div>

    <!-- 2. Controls Area -->
    <div class="controls-card">
        <div class="control-group">
            <label>
                Luminous Flux (&Phi;)
                <span style="font-weight:400; color:#64748b;">lm</span>
            </label>
            <div class="input-row">
                <input type="number" id="inFlux" value="1000" step="100">
                <span style="font-size:0.8rem; color:#94a3b8;">Lumens</span>
            </div>
        </div>

        <div class="control-group">
            <label>
                Height (d)
                <span style="font-weight:400; color:#64748b;">meters</span>
            </label>
            <div class="input-row">
                <input type="number" id="inD" value="3.0" step="0.1" min="0.1">
                <input type="range" id="rangeD" min="0.5" max="10" step="0.1" value="3.0">
            </div>
        </div>

        <div class="control-group">
            <label>
                Floor Distance (x)
                <span style="font-weight:400; color:#64748b;">meters</span>
            </label>
            <div class="input-row">
                <input type="number" id="inX" value="2.0" step="0.1" min="0">
                <input type="range" id="rangeX" min="0" max="10" step="0.1" value="2.0">
            </div>
        </div>
        
        <p style="font-size: 0.8rem; color: #64748b; line-height: 1.4;">
            <strong>Note:</strong> The visualization automatically zooms to fit the diagram. 
            The source radiates light in all directions (isotropic).
        </p>
    </div>

</div>

<script>
    // State management
    const state = {
        phi: 1000,
        d: 3.0,
        x: 2.0,
        r: 0,
        thetaRad: 0,
        E: 0
    };

    // DOM Elements
    const canvas = document.getElementById('simCanvas');
    const ctx = canvas.getContext('2d');
    const wrapper = document.getElementById('canvasWrapper');

    // Inputs
    const elFlux = document.getElementById('inFlux');
    const elD = document.getElementById('inD');
    const elRangeD = document.getElementById('rangeD');
    const elX = document.getElementById('inX');
    const elRangeX = document.getElementById('rangeX');

    // Outputs
    const outE = document.getElementById('dispE');
    const outTheta = document.getElementById('dispTheta');
    const outR = document.getElementById('dispR');
    const outCos = document.getElementById('dispCos');
    const outFormula = document.getElementById('formulaBox');

    // Canvas scaling variables
    let scale = 50; 
    let offsetX = 0;
    let offsetY = 0;
    let dragging = null;

    // --- Core Logic ---

    function calculate() {
        // Physics
        state.r = Math.sqrt(state.d**2 + state.x**2);
        state.thetaRad = Math.atan2(state.x, state.d);
        
        // Avoid division by zero
        if(state.r < 0.001) state.E = 0;
        else {
            const cosTheta = state.d / state.r;
            // Formula: E = (Phi / 4*pi*r^2) * cos(theta)
            // Equivalent to textbook: E = (Phi / 4*pi*d^2) * cos^3(theta)
            const area = 4 * Math.PI * state.r * state.r;
            state.E = (state.phi / area) * cosTheta;
        }

        updateUI();
        requestAnimationFrame(draw);
    }

    function updateUI() {
        const thetaDeg = (state.thetaRad * 180 / Math.PI).toFixed(1);
        const cosVal = Math.cos(state.thetaRad).toFixed(3);
        const cos3 = Math.pow(Math.cos(state.thetaRad), 3).toFixed(3);
        const denom = (4 * Math.PI * state.d * state.d).toFixed(1);

        outE.textContent = state.E.toFixed(2) + " lx";
        outTheta.textContent = thetaDeg + "°";
        outR.textContent = state.r.toFixed(2) + " m";
        outCos.textContent = cosVal;

        // Render Formula string
        outFormula.innerHTML = `E = ${state.phi} / (4π · ${state.d}²) × ${cosVal}³ <br>= <strong>${state.E.toFixed(2)} lx</strong>`;
    }

    // --- Input Handling ---

    function updateFromInput(key, val) {
        val = parseFloat(val);
        if(isNaN(val)) return;
        if(key === 'd' && val < 0.1) val = 0.1;
        if(key === 'x' && val < 0) val = 0;
        
        state[key] = val;

        // Sync dual inputs (number + slider)
        if(key === 'd') { elD.value = val; elRangeD.value = val; }
        if(key === 'x') { elX.value = val; elRangeX.value = val; }
        if(key === 'phi') elFlux.value = val;

        calculate();
    }

    [elFlux, elD, elRangeD, elX, elRangeX].forEach(el => {
        el.addEventListener('input', (e) => {
            if(el === elFlux) updateFromInput('phi', e.target.value);
            else if(el === elD || el === elRangeD) updateFromInput('d', e.target.value);
            else if(el === elX || el === elRangeX) updateFromInput('x', e.target.value);
        });
    });

    // --- Canvas & Visuals ---

    function resizeCanvas() {
        // Match canvas resolution to display size
        const rect = wrapper.getBoundingClientRect();
        canvas.width = rect.width;
        canvas.height = rect.height;
        draw();
    }

    window.addEventListener('resize', resizeCanvas);

    function draw() {
        const w = canvas.width;
        const h = canvas.height;
        ctx.clearRect(0,0, w, h);

        // Auto-Fit Logic
        // We want to fit 'd' (height) and 'x' (width) into the canvas with padding
        const padding = 60;
        const availW = w - padding * 2;
        const availH = h - padding * 2;
        
        // Determine scale (pixels per meter)
        // Ensure minimum visual size even if values are tiny
        const maxMetX = Math.max(state.x, 0.5); 
        const maxMetY = Math.max(state.d, 0.5);
        
        const scaleX = availW / (maxMetX * 1.5); // *1.5 to leave room on right
        const scaleY = availH / (maxMetY * 1.2);
        scale = Math.min(scaleX, scaleY);
        
        // Define Origin (Point on floor directly under source)
        // Position it at bottom-left area
        const floorY = h - 50;
        const originX = 80;

        // Coordinates
        const sX = originX;             // Source X
        const sY = floorY - (state.d * scale); // Source Y
        const tX = originX + (state.x * scale); // Target X
        const tY = floorY;              // Target Y

        // 1. Draw Floor
        ctx.beginPath();
        ctx.moveTo(0, floorY);
        ctx.lineTo(w, floorY);
        ctx.strokeStyle = '#94a3b8';
        ctx.lineWidth = 3;
        ctx.stroke();

        // 2. Vertical Axis (d) dashed
        ctx.beginPath();
        ctx.setLineDash([6, 4]);
        ctx.moveTo(sX, sY);
        ctx.lineTo(sX, floorY);
        ctx.strokeStyle = '#cbd5e1';
        ctx.lineWidth = 2;
        ctx.stroke();
        // Label d
        ctx.fillStyle = '#64748b';
        ctx.font = '14px sans-serif';
        ctx.textAlign = 'right';
        ctx.fillText(`d=${state.d}m`, sX - 10, (sY + floorY)/2);

        // 3. Normal at Target
        ctx.beginPath();
        ctx.moveTo(tX, tY);
        ctx.lineTo(tX, tY - 60);
        ctx.stroke();
        ctx.fillText('Normal', tX, tY - 65);

        // 4. Light Ray (Hypotenuse)
        ctx.setLineDash([]);
        ctx.beginPath();
        ctx.moveTo(sX, sY);
        ctx.lineTo(tX, tY);
        ctx.strokeStyle = '#f59e0b'; // Amber
        ctx.lineWidth = 3;
        ctx.stroke();
        
        // Label r
        ctx.textAlign = 'center';
        ctx.fillStyle = '#b45309';
        ctx.fillText(`r=${state.r.toFixed(2)}m`, (sX+tX)/2 + 10, (sY+tY)/2 - 10);

        // 5. Draw Angle Arc
        // Angle is between Normal (Up) and Ray (Top-Left)
        // Normal angle is -PI/2. Ray angle is atan2(dy, dx).
        const angleRadius = 40;
        if (state.x > 0.1) { // Only draw if angle exists
            ctx.beginPath();
            // Start at Normal (top)
            ctx.arc(tX, tY, angleRadius, 1.5 * Math.PI, 1.5 * Math.PI - state.thetaRad, true);
            ctx.strokeStyle = '#333';
            ctx.lineWidth = 1;
            ctx.stroke();
            ctx.fillText('θ', tX - 10, tY - 45);
        }

        // 6. Draw Source (Draggable)
        ctx.beginPath();
        ctx.arc(sX, sY, 14, 0, Math.PI * 2);
        ctx.fillStyle = '#f59e0b';
        ctx.shadowColor = '#f59e0b';
        ctx.shadowBlur = 15;
        ctx.fill();
        ctx.shadowBlur = 0;
        ctx.strokeStyle = '#fff';
        ctx.lineWidth = 2;
        ctx.stroke();
        ctx.textAlign = 'left';
        ctx.fillStyle = '#333';
        ctx.fillText('Source', sX + 20, sY);

        // 7. Draw Target (Draggable)
        ctx.beginPath();
        ctx.arc(tX, tY, 8, 0, Math.PI * 2);
        ctx.fillStyle = '#1e293b';
        ctx.fill();
        
        // Light spot visual intensity
        const maxLuxVis = 500; // lux value for max opacity
        const opacity = Math.min(state.E / maxLuxVis, 0.8);
        ctx.beginPath();
        ctx.ellipse(tX, tY, 30, 10, 0, 0, Math.PI*2);
        ctx.fillStyle = `rgba(255, 220, 100, ${opacity})`;
        ctx.fill();
        
        ctx.textAlign = 'center';
        ctx.fillStyle = '#333';
        ctx.fillText('X', tX, tY + 20);

        // Store positions for Dragging
        window.simCoords = { sX, sY, tX, tY, floorY, originX };
    }

    // --- Interaction ---
    
    function getMousePos(e) {
        const rect = canvas.getBoundingClientRect();
        const clientX = e.clientX || (e.touches ? e.touches[0].clientX : 0);
        const clientY = e.clientY || (e.touches ? e.touches[0].clientY : 0);
        return { x: clientX - rect.left, y: clientY - rect.top };
    }

    function handleStart(e) {
        const pos = getMousePos(e);
        const c = window.simCoords;
        if(!c) return;

        // Check Source Hit
        if (Math.hypot(pos.x - c.sX, pos.y - c.sY) < 30) {
            dragging = 'source';
        } 
        // Check Target Hit
        else if (Math.hypot(pos.x - c.tX, pos.y - c.tY) < 30) {
            dragging = 'target';
        }
    }

    function handleMove(e) {
        if (!dragging) return;
        e.preventDefault(); // Stop scroll on mobile
        const pos = getMousePos(e);
        const c = window.simCoords;

        if (dragging === 'source') {
            // Drag Y only
            const distPixels = c.floorY - pos.y;
            let newD = distPixels / scale;
            updateFromInput('d', Math.max(0.1, newD).toFixed(2));
        } else if (dragging === 'target') {
            // Drag X only
            const distPixels = pos.x - c.originX;
            let newX = distPixels / scale;
            updateFromInput('x', Math.max(0, newX).toFixed(2));
        }
    }

    canvas.addEventListener('mousedown', handleStart);
    canvas.addEventListener('mousemove', handleMove);
    window.addEventListener('mouseup', () => dragging = null);

    canvas.addEventListener('touchstart', handleStart, {passive: false});
    canvas.addEventListener('touchmove', handleMove, {passive: false});
    window.addEventListener('touchend', () => dragging = null);

    // Initial boot
    setTimeout(() => {
        resizeCanvas();
        calculate();
    }, 100);

</script>
</body>
</html>
