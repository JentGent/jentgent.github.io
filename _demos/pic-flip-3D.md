---
layout: post
title: 3D Fluid Simulation
parent: "demos"
thumbnail-path: "/img/demos/pic-flip-3D/thumbnail.png"
short-description: Play with 3D water
order: 2

---

<style>
    button {
        padding: 0.4em;
    }
    input[type="range"] {
        width: 200px;
        accent-color: #4f8cff;
    }
    table {
        margin-left: auto;
        margin-right: auto;
        display: block;
    }
    td {
        border: 0px;
    }
    canvas {
        margin-left: auto;
        margin-right: auto;
        display: block;
    }
</style>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/p5@0.5.8/lib/p5.min.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/JentGent/pic-flip-3D/main.js"></script>

<table>
    <tr>
        <td>
            <h3>Performance</h3>
            <label for="solver-iterations">Divergence solver iterations:</label><input type="range" id="solver-iterations" min="1" max="50" value="20" oninput="SOLVER_ITERATIONS = this.value;"><br>
            <label for="praticle-iterations">Particle collision iterations:</label><input type="range" id="particle-iterations" min="0" max="5" value="1" oninput="PARTICLE_COLLISION_ITERATIONS = this.value;">
        </td>
        <td>
            <h3>Physics</h3>
            <label for="flip">FLIP method (noisiness):</label><input type="range" id="flip" min="0" max="1" step="0.01" value="0.5" oninput="FLIP = this.value;"><br>
            <label for="stiffness">Stiffness:</label><input type="range" id="stiffness" min="0" max="50" step="1" value="10" oninput="STIFFNESS = this.value;">
        </td>
        <td>
            <h3>Interaction</h3>
            <label for="strength">Push strength:</label><input type="range" id="strength" min="0" max="50" step="1" value="15" oninput="PUSH_STRENGTH = this.value;">
        </td>
    </tr>
</table>

<script>
    const observer = new MutationObserver(() => {
        const canvas = document.querySelector("canvas");
        if (canvas) {
            const table = document.querySelector("table");
            canvas.oncontextmenu = function() { return false; };
            table.parentNode.insertBefore(canvas, table);
            observer.disconnect();
        }
    });
    observer.observe(document.body, {
        childList: true,
        subtree: true
    });
</script>

{:.center}
<br>
Click and drag to move the camera. Move the mouse to push particles.  
<a href="https://github.com/JentGent/pic-flip-3D" target="_blank">GitHub link</a>
