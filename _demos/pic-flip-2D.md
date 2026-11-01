---
layout: post
title: 2D Fluid Simulation
parent: "demos"
thumbnail-path: "/img/demos/pic-flip-2D/thumbnail.png"
short-description: Play with 2D water
order: 6

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
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/JentGent/pic-flip/main.js"></script>

<table>
    <tr>
        <td>
            <h3>Performance</h3>
            <label for="solver-iterations">Divergence solver iterations: </label><input type="range" id="solver-iterations" min="0" max="20" value="5" oninput="SOLVER_ITERATIONS = this.value;"><br>
            <label for="praticle-iterations">Particle collision iterations: </label><input type="range" id="particle-iterations" min="0" max="5" value="1" oninput="PARTICLE_COLLISION_ITERATIONS = this.value;"><br>
            <label for="substeps">Substeps: </label><input type="range" id="substeps" min="0" max="5" value="2" oninput="SUBSTEPS = this.value;"><br>
        </td>
        <td>
            <h3>Physics</h3>
            <label for="flip">FLIP method (noisiness): </label><input type="range" id="flip" min="0" max="1" step="0.01" value="0.5" oninput="FLIP = this.value;"><br>
            <label for="stiffness">Stiffness: </label><input type="range" id="stiffness" min="0" max="1" step="0.01" value="1" oninput="STIFFNESS = this.value;"><br>
        </td>
        <td>
            <h3>Display</h3>
            <label for="density">Display density grid: </label><input type="checkbox" id="density" oninput="DISPLAY_DENSITY = !DISPLAY_DENSITY;"><br>
            <label for="particles">Display particles: </label><input type="checkbox" id="particles" oninput="DISPLAY_PARTICLES = !DISPLAY_PARTICLES;" checked>
        </td>
    </tr>
</table>

{:.center}
Left click and drag to drag the water around.  
Right click and and drag to draw walls. Hold any key to erase walls.  
<a href="https://github.com/JentGent/pic-flip" target="_blank">GitHub link</a>

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
