---
layout: post
title: 2D physics engine
parent: "demos"
thumbnail-path: "/img/demos/2D-physics-engine/thumbnail.png"
short-description: Sequential impulse constraint solver

---

<script src="https://cdn.jsdelivr.net/gh/JentGent/2D-physics-engine@latest/demos/libraries/p5.min.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/2D-physics-engine@latest/math.min.js"></script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/2D-physics-engine@latest/ji.js"></script>
<script>
    let canvas, width = 1000, height = 800;
    function setup() {
        canvas = createCanvas(width, height);
        canvas.parent("sketch");
        canvas.elt.oncontextmenu = () => false;
    }
</script>
<script src="https://cdn.jsdelivr.net/gh/JentGent/2D-physics-engine@latest/demos/random/main.js"></script>
<div id="sketch" class="demo"></div>

<br/><br/>

{:.center}
A demo of my custom 2D physics engine (<a href="https://github.com/JentGent/2D-physics-engine" target="_blank">GitHub</a>).  
Left click to grab; right click to pull.