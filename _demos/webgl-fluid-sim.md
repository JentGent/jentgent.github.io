---
layout: post
title: WebGL Fluid Simulation
parent: "demos"
thumbnail-path: "/img/demos/webgl-fluid-sim/thumbnail.png"
short-description: Play with dyes
order: 4

---

<style>
    input[type="range"] {
        width: 200px;
        accent-color: #4f8cff;
    }
    #settings-container {
        width: fit-content;
        margin-inline: auto;
        margin-top: 1em;
    }
    canvas {
        margin-left: auto;
        margin-right: auto;
        display: block;
    }
    #beginning-art {
        display: none;
    }
    #render {
        display: none;
    }
    #display {
        border: 0px;
    }
    h1 {
        margin-bottom: 5px;
    }
</style>

<canvas id="beginning-art"></canvas>
<canvas id="render"></canvas>
<canvas id="display" width="600" height="600" oncontextmenu="return false"></canvas>

<div id="settings-container">
    <input type="range" id="INPUT_VELOCITY" min="1" max="100" value="28"> <label for="INPUT_VELOCITY">Input velocity</label><br>
    <input type="range" id="SIZE" min="10" max="200" value="75"> <label for="SIZE">Input size</label><br>
    <input type="range" id="COLOR_DIFFUSION" min="0" max="100" value="20"> <label for="COLOR_DIFFUSION">Color diffusion</label><br>
    <input type="range" id="VELOCITY_DIFFUSION" min="0" max="100" value="10"> <label for="VELOCITY_DIFFUSION">Velocity diffusion</label><br>
    <input type="range" id="VORTICITY" min="0" max="40" value="10"> <label for="VORTICITY">Vorticity</label><br><br>
</div>

{:.center}
Left click and drag to draw. Right click and drag to swirl.  
Press "V" to set velocities to 0 and "C" to set colors to black.  
<a href="https://github.com/JentGent/webgl-fluid-sim" target="_blank">GitHub link</a>

<html>
    <script id="vertex.vsh" type="vertex-shader">#version 300 es
        precision highp float;
        
        in vec2 a_position;
        
        uniform vec2 u_invResolution;

        out vec2 v_uv, v_ul, v_ur, v_uu, v_ud;

        void main() {
            v_uv = a_position / 2. + vec2(0.5);
            v_ul = vec2(v_uv.x - u_invResolution.x, v_uv.y);
            v_ur = vec2(v_uv.x + u_invResolution.x, v_uv.y);
            v_ud = vec2(v_uv.x, v_uv.y - u_invResolution.y);
            v_uu = vec2(v_uv.x, v_uv.y + u_invResolution.y);
            gl_Position = vec4(a_position.xy, 0, 1);
        }
    </script> <!--vertex-->

    <script id="curl.fsh" type="fragment-shader">#version 300 es
        #ifdef GL_ES
        precision mediump float;
        #endif

        in vec2 v_uv, v_ul, v_ur, v_uu, v_ud;

        uniform sampler2D u_velocity;
        uniform vec2 u_mouse;
        uniform vec2 u_move;
        uniform bool u_pressing;
        
        out float final;

        void main() {
            final = 0.5 * (texture(u_velocity, v_ur).y - texture(u_velocity, v_ul).y - texture(u_velocity, v_uu).x + texture(u_velocity, v_ud).x);
        }
    </script> <!--curl-->
    <script id="vortex.fsh" type="fragment-shader">#version 300 es
        #ifdef GL_ES
        precision mediump float;
        #endif

        in vec2 v_uv, v_ul, v_ur, v_uu, v_ud;
        
        uniform sampler2D u_velocity;
        uniform sampler2D u_curl;
        uniform float u_dTime;
        uniform vec2 u_mouse;
        uniform vec2 u_move;
        uniform bool u_pressing;
        uniform float u_vorticity;
        uniform float u_size;
        uniform bool u_reset;
        
        out vec2 final;

        void main() {
            if(u_reset) {
                final = vec2(0, 0);
                return;
            }
            vec2 move = vec2(0);
            if(u_pressing) move = u_move / (1. + dot(u_mouse - v_uv, u_mouse - v_uv) * u_size);
            vec2 cc = 0.5 * vec2(abs(texture(u_curl, v_uu).x) - abs(texture(u_curl, v_ud).x), abs(texture(u_curl, v_ul).x) - abs(texture(u_curl, v_ur).x));
            float l = dot(cc, cc);
            if(l > 0.) cc *= inversesqrt(l);
            final = clamp(texture(u_velocity, v_uv).xy + move + cc * u_vorticity * texture(u_curl, v_uv).x * u_dTime, -10., 10.);
        }
    </script> <!--vortex-->
    <script id="divergence.fsh" type="fragment-shader">#version 300 es
        #ifdef GL_ES
        precision mediump float;
        #endif

        in vec2 v_uv, v_ul, v_ur, v_uu, v_ud;
        
        uniform sampler2D u_velocity;
        
        out float final;

        void main() {
            float l = texture(u_velocity, v_ul).x, r = texture(u_velocity, v_ur).x, u = texture(u_velocity, v_uu).y, d = texture(u_velocity, v_ud).y;
            vec2  a = texture(u_velocity, v_uv).xy;
            if(v_ul.x < 0.) l = -a.x;
            if(v_ur.x > 1.) r = -a.x;
            if(v_uu.y > 1.) u = -a.y;
            if(v_ud.y < 0.) d = -a.y;
            final = 0.5 * (r - l + u - d);
        }
    </script> <!--divergence-->
    <script id="pressure.fsh" type="fragment-shader">#version 300 es
        #ifdef GL_ES
        precision mediump float;
        #endif

        in vec2 v_uv, v_ul, v_ur, v_uu, v_ud;
        
        uniform sampler2D u_divergence;
        uniform sampler2D u_pressure;
        uniform bool u_first;
        uniform float u_strength;
        
        out float final;

        void main() {
            if(u_first) {
                final = 0.;
                return;
            }
            final = 0.25 * (texture(u_pressure, v_ul).x + texture(u_pressure, v_ur).x + texture(u_pressure, v_uu).x + texture(u_pressure, v_ud).x - texture(u_divergence, v_uv).x);
        }
    </script> <!--pressure-->
    <script id="project.fsh" type="fragment-shader">#version 300 es
        #ifdef GL_ES
        precision mediump float;
        #endif

        in vec2 v_uv, v_ul, v_ur, v_uu, v_ud;
        
        uniform sampler2D u_velocity;
        uniform sampler2D u_pressure;
        
        out vec2 final;

        void main() {
            final = texture(u_velocity, v_uv).xy - vec2(texture(u_pressure, v_ur).x - texture(u_pressure, v_ul).x, texture(u_pressure, v_uu).x - texture(u_pressure, v_ud).x);
        }
    </script> <!--project-->
    <script id="advectVelocity.fsh" type="fragment-shader">#version 300 es
        #ifdef GL_ES
        precision mediump float;
        #endif

        in vec2 v_uv;
        
        uniform sampler2D u_velocity;
        uniform float u_dTime;
        uniform float u_diffusion;
        
        out vec2 final;

        void main() {
            final = texture(u_velocity, v_uv - texture(u_velocity, v_uv).xy * u_dTime).xy / (1. + u_diffusion * u_dTime);
        }
    </script> <!--advectVelocity-->
    <script id="advectColors.fsh" type="fragment-shader">#version 300 es
        #ifdef GL_ES
        precision mediump float;
        #endif

        in vec2 v_uv;
        
        uniform sampler2D u_velocity;
        uniform sampler2D u_field;
        uniform float u_dTime;
        uniform float u_diffusion;
        uniform vec2 u_mouse;
        uniform vec3 u_color;
        uniform bool u_pressing;
        uniform float u_size;
        uniform bool u_reset;
        
        out vec4 final;

        void main() {
            if(u_reset) {
                final = vec4(0, 0, 0, 1);
                return;
            }
            vec3 add = vec3(0);
            if(u_pressing) add += u_color / (1. + dot(u_mouse - v_uv, u_mouse - v_uv) * u_size);

            final = vec4((texture(u_field, v_uv - texture(u_velocity, v_uv).xy * u_dTime).rgb + add) / (1. + u_diffusion * u_dTime), 1);
        }
    </script> <!--advectColors-->
    
    <script type>
{

const SETTINGS = {
    INPUT_VELOCITY: 28,      // how fast it swirls when you click
    VELOCITY_DIFFUSION: 0.1, // how quickly the velocity slows down over time
    COLOR_DIFFUSION: 0.2,    // how quickly the color fades over time
    VORTICITY: 10,           // how turbulent and swirly the fluid is
    SIZE: 75 / 100000,       // how big it is when you click

    VELOCITY_RESOLUTION: 100, // lower these if it's slow
    COLOR_RESOLUTION: 400,    // lower these if it's slow
    PRESSURE_ITERATIONS: 10,  // lower these if it's slow
};

function id(i) { return document.getElementById(i); }
function ih(i) { return id(i).innerHTML; }

// Setup
const start = id("beginning-art");
const art = start.getContext("2d");
const c = id("render");
const gl = c.getContext("webgl2");
gl.getExtension("EXT_color_buffer_float");
gl.getExtension("OES_texture_float_linear");
const can = id("display");
const ctx = can.getContext("2d");

// Mouse
let mouseIsPressed = false, mouseIsPressed2 = false, mouseX = 0, mouseY = 0, pmouseX = 0, pmouseY = 0;
can.onmousedown = function(e) {
    if(e.button === 2) { mouseIsPressed2 = true; }
    else if(e.button === 0) { mouseIsPressed = true; }
}
can.onmouseup = function(e) {
    if(e.button === 2) { mouseIsPressed2 = false; }
    else if(e.button === 0) { mouseIsPressed = false; }
}
can.onmousemove = function(e) {
    mouseX = e.offsetX;
    mouseY = e.offsetY;
}

// Keys
const keys = {};
document.onkeydown = function(e) {
    keys[e.key.toLowerCase()] = true;
}
document.onkeyup = function(e) {
    keys[e.key.toLowerCase()] = false;
}

function HSVtoRGB(h, s, v) {
    const i = Math.floor(h * 6),
          f = h * 6 - i,
          p = v * (1 - s),
          q = v * (1 - f * s),
          t = v * (1 - (1 - f) * s);
    let r = g = b = 0;
    switch(i % 6) {
        case 0: r = v, g = t, b = p; break;
        case 1: r = q, g = v, b = p; break;
        case 2: r = p, g = v, b = t; break;
        case 3: r = p, g = q, b = v; break;
        case 4: r = t, g = p, b = v; break;
        case 5: r = v, g = p, b = q; break;
    }
    return [r, g, b];
}

// Program
function shader(n, s, t) {
    const base = gl.createShader(t);
    gl.shaderSource(base, s);
    gl.compileShader(base);
    if(!gl.getShaderParameter(base, gl.COMPILE_STATUS)) { return console.error("error in \"" + n + "\" " + ((t === gl.FRAGMENT_SHADER) ? "fragment" : "vertex") + " shader: " + gl.getShaderInfoLog(base)); }
    return base;
}
function program(n, v, f) {
    const p = gl.createProgram();
    gl.attachShader(p, (typeof v === "string") ? vshader(n, v, gl.VERTEX_SHADER) : v);
    gl.attachShader(p, (typeof f === "string") ? shader(n, f, gl.FRAGMENT_SHADER) : f);
    gl.linkProgram(p);
    if(!gl.getProgramParameter(p, gl.LINK_STATUS)) { return console.error("error in \"" + n + "\" shader: " + gl.getProgramInfoLog(p)); }
    gl.validateProgram(p);
    if(!gl.getProgramParameter(p, gl.VALIDATE_STATUS)) { return console.error("error validating \"" + n + "\": " + gl.getProgramInfoLog(p)); }
    return p;
}
function setup(program) {
    gl.useProgram(program);
    gl.clear(gl.COLOR_BUFFER_BIT);
}

// Uniform
function uniform(p, n, t, v) {
    t = "uniform" + t;
    if(gl[t]) {
        try {
            gl.useProgram(p);
            if(t[7] === "M") { gl[t](gl.getUniformLocation(p, n), false, v); }
            else { gl[t](gl.getUniformLocation(p, n), v); }
        }
        catch(e) {
            console.error("error setting uniform \"" + n + "\": " + e)
        }
        return;
    }
    console.error("no uniform type \"" + t + "\"");
}

// Attribute
function attribute(p, n, s, t, no, j, o) {
    try {
        const l = gl.getAttribLocation(p, n);
        gl.vertexAttribPointer(l, s, t, no, j, o);
        gl.enableVertexAttribArray(l);
        return l;
    }
    catch(e) { return console.error("error setting attribute \"" + n + "\": " + e); }
};

// Buffer
function buffer(t, a, o) {
    const b = gl.createBuffer();
    gl.bindBuffer(t, b);
    gl.bufferData(t, a, o);
    return b;
}

// Texture
function texture(internalformat, format, w = fieldWidth, h = fieldHeight, c = null) {
    gl.activeTexture(gl.TEXTURE0);
    const tex = gl.createTexture();
    gl.bindTexture(gl.TEXTURE_2D, tex);
    gl.texImage2D(gl.TEXTURE_2D, 0, internalformat || gl.RGBA32F, w, h, 0, format || gl.RGBA, gl.FLOAT, c);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
    gl.bindTexture(gl.TEXTURE_2D, null);
    return { fb: gl.createFramebuffer(), t: tex };
}
function bindTexture(p, n, t, i) {
    gl.activeTexture(gl.TEXTURE0 + i);
    gl.bindTexture(gl.TEXTURE_2D, t.t);
    uniform(p, n, "1i", i);
}
function bindToTexture(t, d) {
    gl.bindFramebuffer(gl.FRAMEBUFFER, d ? t.fb2 : t.fb);
    gl.framebufferTexture2D(gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, t.t, 0);
}
function render() { gl.drawArrays(gl.TRIANGLES, 0, 6); }
function unbind() { gl.bindFramebuffer(gl.FRAMEBUFFER, null); }

// Programs
const vertexShader = shader("base", ih("vertex.vsh"), gl.VERTEX_SHADER);
const fieldWidth = SETTINGS.VELOCITY_RESOLUTION, fieldHeight = SETTINGS.VELOCITY_RESOLUTION;

const calculateCurl = program("curl", vertexShader, ih("curl.fsh"));
const addVortices = program("vortex", vertexShader, ih("vortex.fsh"));
const calculateDivergence = program("divergence", vertexShader, ih("divergence.fsh"));
const updatePressure = program("pressure", vertexShader, ih("pressure.fsh"));
const removeDivergence = program("project", vertexShader, ih("project.fsh"));
const advectVelocity = program("advectVelocity", vertexShader, ih("advectVelocity.fsh"));
const advectField = program("advectColors", vertexShader, ih("advectColors.fsh"));

const curl = texture(gl.R32F, gl.RED);
const vortexed = texture(gl.RG32F, gl.RG);
const divergence = texture(gl.R32F, gl.RED);
const pressure1 = texture(gl.R32F, gl.RED);
const pressure2 = texture(gl.R32F, gl.RED);
const projected = texture(gl.RG32F, gl.RG);
const advected = texture(gl.RG32F, gl.RG);

(function() {
    start.width = start.height = SETTINGS.COLOR_RESOLUTION;
    art.fillStyle = "#000000";
    art.fillRect(0, 0, start.width, start.height);

    const cx = start.width / 2;
    const cy = start.height / 2;
    const maxR = Math.min(start.width, start.height) * 0.45;

    art.globalCompositeOperation = "lighter";

    for (let i = 0; i < 180; i++) {
        const t = i / 180;
        const angle = i * 0.42;
        const r = t * maxR;

        const x = cx + Math.cos(angle) * r;
        const y = cy + Math.sin(angle) * r;

        const hue = (i * 7) % 360;
        const size = 2 + 10 * (1 - t);

        art.beginPath();
        art.arc(x, y, size, 0, Math.PI * 2);

        art.shadowBlur = 20;
        art.shadowColor = `hsl(${hue}, 100%, 60%)`;
        art.fillStyle = `hsl(${hue}, 100%, 65%)`;
        art.fill();
    }

    const glow = art.createRadialGradient(
        cx, cy, 0,
        cx, cy, maxR * 0.35
    );

    glow.addColorStop(0, "white");
    glow.addColorStop(0.08, "#7fffff");
    glow.addColorStop(0.3, "#ff00ff");
    glow.addColorStop(1, "transparent");

    art.fillStyle = glow;
    art.beginPath();
    art.arc(cx, cy, maxR * 0.35, 0, Math.PI * 2);
    art.fill();

})();

const field1 = texture(gl.RGBA32F, gl.RGBA, SETTINGS.COLOR_RESOLUTION, SETTINGS.COLOR_RESOLUTION, start);
const field2 = texture(gl.RGBA32F, gl.RGBA, SETTINGS.COLOR_RESOLUTION, SETTINGS.COLOR_RESOLUTION, start);

uniform(calculateCurl, "u_invResolution", "2fv", [1 / fieldWidth, 1 / fieldHeight]);
uniform(addVortices, "u_invResolution", "2fv", [1 / fieldWidth, 1 / fieldHeight]);
uniform(calculateDivergence, "u_invResolution", "2fv", [1 / fieldWidth, 1 / fieldHeight]);
uniform(updatePressure, "u_invResolution", "2fv", [1 / fieldWidth, 1 / fieldHeight]);
uniform(removeDivergence, "u_invResolution", "2fv", [1 / fieldWidth, 1 / fieldHeight]);
uniform(advectVelocity, "u_invResolution", "2fv", [1 / fieldWidth, 1 / fieldHeight]);



// Set up vertices
const verts = new Float32Array([
    -1, -1,
    1, -1,
    1, 1,
    -1, -1,
    -1, 1,
    1, 1,
]);
const bpe = verts.BYTES_PER_ELEMENT;
buffer(gl.ARRAY_BUFFER, verts, gl.STATIC_DRAW);
attribute(calculateCurl, "a_position", 2, gl.FLOAT, false, 2 * bpe, 0);
attribute(addVortices, "a_position", 2, gl.FLOAT, false, 2 * bpe, 0);

// Draw
let iteration = true, iteration2 = true;
let started = false;
function draw(dt) {
    ctx.clearRect(0, 0, 600, 600);
    SETTINGS.INPUT_VELOCITY = document.getElementById("INPUT_VELOCITY").value * 1;
    SETTINGS.SIZE = document.getElementById("SIZE").value / 100000;
    SETTINGS.COLOR_DIFFUSION = document.getElementById("COLOR_DIFFUSION").value / 100;
    SETTINGS.VELOCITY_DIFFUSION = document.getElementById("VELOCITY_DIFFUSION").value / 100;
    SETTINGS.VORTICITY = document.getElementById("VORTICITY").value * 1;

    if(started) {
        // Calculate curl
        c.width = c.height = SETTINGS.VELOCITY_RESOLUTION;
        gl.viewport(0, 0, SETTINGS.VELOCITY_RESOLUTION, SETTINGS.VELOCITY_RESOLUTION);
        setup(calculateCurl);
        bindTexture(calculateCurl, "u_velocity", advected, 0);
        bindToTexture(curl);
        render();
        unbind();

        // Add vorticity + user input
        setup(addVortices);
        bindTexture(addVortices, "u_velocity", advected, 0);
        bindTexture(addVortices, "u_curl", curl, 1);
        uniform(addVortices, "u_vorticity", "1f", SETTINGS.VORTICITY);
        uniform(addVortices, "u_size", "1f", 1 / SETTINGS.SIZE);
        uniform(addVortices, "u_dTime", "1f", dt);
        uniform(addVortices, "u_mouse", "2fv", [mouseX / can.width, (can.height - mouseY) / can.height]);
        uniform(addVortices, "u_move", "2fv", [(mouseX - pmouseX) / can.width * SETTINGS.INPUT_VELOCITY, -(mouseY - pmouseY) / can.height * SETTINGS.INPUT_VELOCITY]);
        uniform(addVortices, "u_pressing", "1i", mouseIsPressed2);
        uniform(addVortices, "u_reset", "1i", keys.v);
        bindToTexture(vortexed);
        render();
        unbind();

        // Calculate divergence
        setup(calculateDivergence);
        bindTexture(calculateDivergence, "u_velocity", vortexed, 0);
        bindToTexture(divergence);
        render();
        unbind();

        // Update pressure
        setup(updatePressure);
        bindTexture(updatePressure, "u_divergence", divergence, 0);
        uniform(updatePressure, "u_first", "1i", 1);
        bindToTexture(iteration ? pressure2 : pressure1);
        render();
        unbind();
        uniform(updatePressure, "u_first", "1i", 0);
        for(let i = 0; i < SETTINGS.PRESSURE_ITERATIONS; i += 1) {
            iteration = !iteration;
            bindTexture(updatePressure, "u_pressure", iteration ? pressure1 : pressure2, 1);
            bindToTexture(iteration ? pressure2 : pressure1);
            render();
            unbind();
        }

        // Remove divergence
        setup(removeDivergence);
        bindTexture(removeDivergence, "u_velocity", vortexed, 0);
        bindTexture(removeDivergence, "u_pressure", iteration ? pressure2 : pressure1, 1);
        bindToTexture(projected);
        render();
        unbind();

        // Advect velocity
        setup(advectVelocity);
        bindTexture(advectVelocity, "u_velocity", projected, 0);
        uniform(advectVelocity, "u_diffusion", "1f", SETTINGS.VELOCITY_DIFFUSION);
        uniform(advectVelocity, "u_dTime", "1f", dt);
        bindToTexture(advected);
        render();
        unbind();
    }

    // Advect colors
    c.width = c.height = SETTINGS.COLOR_RESOLUTION;
    gl.viewport(0, 0, SETTINGS.COLOR_RESOLUTION, SETTINGS.COLOR_RESOLUTION);
    setup(advectField);
    bindTexture(advectField, "u_field", iteration2 ? field1 : field2, 1);
    if(started) {
        bindTexture(advectField, "u_velocity", advected, 0);
        uniform(advectField, "u_diffusion", "1f", SETTINGS.COLOR_DIFFUSION);
        uniform(advectField, "u_size", "1f", 1 / SETTINGS.SIZE);
        uniform(advectField, "u_mouse", "2fv", [mouseX / can.width, (can.height - mouseY) / can.height]);
        uniform(advectField, "u_color", "3fv", HSVtoRGB(Math.random(), 1, 1));
        uniform(advectField, "u_pressing", "1i", mouseIsPressed);
        uniform(advectField, "u_dTime", "1f", dt);
        uniform(advectField, "u_reset", "1i", keys.c);
        bindToTexture(iteration2 ? field2 : field1);
        render();
        unbind();
    }
    else if(mouseIsPressed || mouseIsPressed2) { started = true; }

    render();
    ctx.drawImage(c, 0, 0, 600, 600);

    pmouseX = mouseX;
    pmouseY = mouseY;
    iteration2 = !iteration2;
    iteration = !iteration;
}

window.performance = window.performance || {};
performance.now = (function() {
    return performance.now ||
        performance.mozNow ||
        performance.msNow ||
        performance.oNow ||
        performance.webkitNow ||
        Date.now;
})();
let pm = performance.now();
function run() {
    parent.raf = requestAnimationFrame(run);
    const now = performance.now();
    if(now > pm + 1000 / 60) {
        draw((now - pm) / 1000);
        pm = now;
    }
}
if(parent.raf) { cancelAnimationFrame(parent.raf); }
run();

}
    </script>
</html>
