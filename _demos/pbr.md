---
layout: post
title: PBR Engine
parent: "demos"
thumbnail-path: "/img/demos/pbr/thumbnail.png"
short-description: Realistic 3D materials
order: 1

---

<style>
    #render {
        display: none;
    }
    #display {
        border: 0px;
        margin-left: auto;
        margin-right: auto;
        display: block;
    }
</style>

<canvas id="render" width="600" height="600"></canvas>
<canvas id="display" width="600" height="600"></canvas>

{:.center}
<br>
WASD, Shift and Space to move. Click and drag to look around.  
This implements a BRDF rendering model, SSAO, and shadow mapping.  
<a href="https://github.com/JentGent/pbr-3d" target="_blank">GitHub link</a>

<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/shaders/prepass.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/shaders/color.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/shaders/shadow.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/shaders/sky.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/vector.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/webgl utilities.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/geometry utilities.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/world.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/set up programs.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/pbr-3d/render.js"></script>

<script type>
    // Game Logic
    function update(dt) {
        if(mouseIsPressed) {
            yaw += (mouseX - pmouseX) * 0.01 * dt;
            pitch += (mouseY - pmouseY) * 0.01 * dt;
            pitch = constrain(pitch, -Math.PI / 2, Math.PI / 2);
        }
        const spd = 0.05 * dt;
        if(keys.w) {
            cam.x -= Math.sin(-yaw) * spd;
            cam.z += Math.cos(-yaw) * spd;
        }
        if(keys.s) {
            cam.x += Math.sin(-yaw) * spd;
            cam.z -= Math.cos(-yaw) * spd;
        }
        if(keys.d) {
            cam.x += Math.cos(-yaw) * spd;
            cam.z += Math.sin(-yaw) * spd;
        }
        if(keys.a) {
            cam.x -= Math.cos(-yaw) * spd;
            cam.z -= Math.sin(-yaw) * spd;
        }
        if(keys[" "]) {
            cam.y += spd;
        }
        if(keys.shift) {
            cam.y -= spd;
        }
        pmouseX = mouseX;
        pmouseY = mouseY;
    }

    const canvas = document.getElementById("display");
    canvas.oncontextmenu = function() { return false; };
    window.addEventListener("keydown", function(e) {
        if (e.keyCode === 32 && e.target === document.body) {
            e.preventDefault();
        }
    });
    
    const START_TIME = Date.now();
    let pm = Date.now(), frameRate = Date.now() - pm;
    if(parent.tm) { clearTimeout(parent.tm); }
    function run() {
        parent.raf = requestAnimationFrame(run);
        frameRate = Date.now() - pm;
        pm = Date.now();
        update(frameRate * 0.1);
        if(!THERE_WAS_AN_ERROR) { draw(frameRate * 0.1, (pm - START_TIME) * 0.1); }
        ctx.drawImage(c, 0, 0, 600, 600);
        // ctx.font = "20px Helvetica";
        // ctx.fillStyle = "white";
        // ctx.fillText(Math.round(frameRate) + " ms", 5, can.height - 5);
    }
    if(parent.raf) { cancelAnimationFrame(parent.raf); }
    run();
</script>