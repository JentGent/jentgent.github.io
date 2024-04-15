---
layout: post
title: Projectile Motion Calculator
parent: "demos"
thumbnail-path: "/img/demos/projectile/thumbnail.png"
short-description: Physics tool

---

<!-- <style>
    body {
        font-family: Arial, sans-serif;
        font-size: 16px;
        line-height: 1.6;
        color: #333;
        margin: 0px;
        padding: 20px;
    }
</style> -->

Enter numbers for known variables; leave unknown inputs blank.  
Press "Calculate" to fill in the empty variables and show the steps the calculator took to solve for them.

In the beginning, the projectile is <select><option value="rising">rising</option><option value="peak">at its peak</option><option value="falling">falling</option><option value="unknown" selected>(unknown)</option></select>.<br>
<label>vᵢ: </label><input type="number" value=""><select><option value="m/s">m/s</option></select><br>
<label>θ: </label><input type="number" value=""><select><option value="radians">radians</option></select><br>
<label>vₓ: </label><input type="number" value=""><select><option value="m/s">m/s</option></select><br>
<label>vᵢᵧ: </label><input type="number" value=""><select><option value="m/s">m/s</option></select><br>
<br>
At time t, the projectile is <select><option value="rising">rising</option><option value="peak">at its peak</option><option value="falling">falling</option><option value="unknown" selected>(unknown)</option></select>.<br>
<label>t: </label><input type="number" value=""><select><option value="s">s</option></select><br>
<label>∆x: </label><input type="number" value=""><select><option value="m">m</option></select><br>
<label>∆y: </label><input type="number" value=""><select><option value="m">m</option></select><br>
<label>vᵧ: </label><input type="number" value=""><select><option value="m/s">m/s</option></select><br>
<br>
<button id="calculate">Calculate</button><br>
<br>
<div id="result"></div>

<script>

const inputs = Array.from(document.getElementsByTagName("input"));
const selects = Array.from(document.getElementsByTagName("select"));
const button = document.getElementById("calculate");

const [vi, theta, vx, viy, t, x, y, vy] = inputs;
const [start, viUnits, thetaUnits, vxUnits, viyUnits, end, tUnits, xUnits, yUnits, vyUnits] = selects;
const result = document.getElementById("result");

viy.oninput = function() {
    if(viy.value === "") return;
    if(Math.abs(viy.value) < ZERO_THRESHOLD) return start.value = "peak";
    else if(viy.value < 0) return start.value = "falling";
    else return start.value = "rising";
};
vy.oninput = function() {
    if(vy.value === "") return;
    if(Math.abs(vy.value) < ZERO_THRESHOLD) return end.value = "peak";
    else if(vy.value < 0) return end.value = "falling";
    else return end.value = "rising";
};
y.oninput = function() {
    if(y.value > ZERO_THRESHOLD) return start.value = "rising";
    if(y.value < -ZERO_THRESHOLD) return end.value = "falling";
}

const a = -9.8; // `a` must be negative
const THRESHOLD = 0.0001,
      ZERO_THRESHOLD = 0.00001;

function unknowns(variables) {
    return variables.filter(x => x.value === "");
}

function updateStartAndEnd() {
    if(viy.value !== "") {
        if(Math.abs(viy.value) < ZERO_THRESHOLD) {
            if(start.value !== "unknown" && start.value !== "peak") {
                result.innerHTML += "<br><br>Conflicting falling/rising!";
                alert("Conflicting answers; try again.");
            }
            start.value = "peak";
        }
        else if(viy.value < 0) {
            if(start.value !== "unknown" && start.value !== "falling") {
                result.innerHTML += "<br><br>Conflicting falling/rising!";
                alert("Conflicting answers; try again.");
            }
            start.value = "falling";
        }
        else {
            if(start.value !== "unknown" && start.value !== "rising") {
                result.innerHTML += "<br><br>Conflicting falling/rising!";
                alert("Conflicting answers; try again.");
            }
            start.value = "rising";
        }
    }
    if(y.value > ZERO_THRESHOLD) {
        if(start.value !== "unknown" && start.value !== "rising") {
            result.innerHTML += "<br><br>Conflicting falling/rising!";
            alert("Conflicting answers; try again.");
        }
        start.value = "rising";
    }
    
    if(vy.value !== "") {
        if(Math.abs(vy.value) < ZERO_THRESHOLD) {
            if(end.value !== "unknown" && end.value !== "peak") {
                result.innerHTML += "<br><br>Conflicting falling/rising!";
                alert("Conflicting answers; try again.");
            }
            end.value = "peak";
        }
        else if(vy.value < 0) {
            if(end.value !== "unknown" && end.value !== "falling") {
                result.innerHTML += "<br><br>Conflicting falling/rising!";
                alert("Conflicting answers; try again.");
            }
            end.value = "falling";
        }
        else {
            if(end.value !== "unknown" && end.value !== "rising") {
                result.innerHTML += "<br><br>Conflicting falling/rising!";
                alert("Conflicting answers; try again.");
            }
            end.value = "rising";
        }
    }
    if(y.value < -ZERO_THRESHOLD) {
        if(end.value !== "unknown" && end.value !== "falling") {
            result.innerHTML += "<br><br>Conflicting falling/rising!";
            alert("Conflicting answers; try again.");
        }
        end.value = "falling";
    }
}

function eq1() {
    let u = unknowns([y, viy, t]);
    if(u.length === 1) {
        u = u[0];
        if(u === viy && Math.abs(t.value) < ZERO_THRESHOLD) return;
        if(u === t && end.value === "unknown") return;
        if(u === t && viy.value * viy.value + 2 * a * y.value < -ZERO_THRESHOLD) {
            result.innerHTML += "<br><br>NO SOLUTION for equation:";
            result.innerHTML += "<br>∆y = vᵢᵧ * t + a/2 * t²";
            return alert("No solution; try again.");
        }
        result.innerHTML += "<br>" + ("<br>∆y = vᵢᵧ * t + a/2 * t²");
        switch(u) {
            case y:
                y.value = viy.value * t.value + a/2 * t.value * t.value;
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`∆y = ${viy.value} * ${t.value} + ${a}/2 * ${t.value}²`);
            break;
            case viy:
                viy.value = (y.value - a/2 * t.value * t.value) / t.value;
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵢᵧ = (${y.value} - ${a}/2 * ${t.value}²) / ${t.value}`);
            break;
            case t:
                t.value = (-viy.value + Math.sqrt(viy.value * viy.value + 2 * a * y.value) * (end.value === "falling" ? -1 : 1)) / a;
                result.innerHTML += "<br>" + (`t = (-${viy.value} ${end.value === "falling" ? "-" : "+"} √(${viy.value}² + 2 * ${a} * ${y.value})) / ${a}`);
        }
    }
    else if(!u.length) {
        if(Math.abs(viy.value * t.value + a/2 * t.value * t.value - y.value) > THRESHOLD) {
            result.innerHTML += "<br><br>ERROR in equation:";
            result.innerHTML += "<br>∆y = vᵢᵧ * t + a/2 * t²";
            alert("Conflicting answers; try again.");
        }
    }
}
function eq2() {
    let u = unknowns([vy, viy, t]);
    if(u.length === 1) {
        u = u[0];
        result.innerHTML += "<br>" + ("<br>vᵧ = vᵢᵧ + a * t");
        switch(u) {
            case vy:
                vy.value = +viy.value + a * t.value;
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵧ = ${viy.value} + ${a} * ${t.value}`);
            break;
            case viy:
                viy.value = vy.value - a * t.value;
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵢᵧ = ${vy.value} - ${a} * ${t.value}`);
            break;
            case t:
                t.value = (vy.value - viy.value) / a;
                result.innerHTML += "<br>" + (`t = (${vy.value} - ${viy.value}) / ${a}`);
        }
    }
    else if(!u.length) {
        if(Math.abs(viy.value + a * t.value - vy.value) > THRESHOLD) {
            result.innerHTML += "<br><br>ERROR in equation:";
            result.innerHTML += "<br>vᵧ = vᵢᵧ + a * t";
            alert("Conflicting answers; try again.");
        }
    }
}
function eq3() {
    let u = unknowns([vy, viy, y]);
    if(u.length === 1) {
        u = u[0];
        if(u === vy && end.value === "unknown") return;
        if(u === viy && start.value === "unknown") return;
        if((u === vy && viy.value * viy.value + 2 * a * y.value < 0) || (u === viy && vy.value * vy.value - 2 * a * y.value < 0)) {
            result.innerHTML += "<br><br>NO SOLUTION for equation:";
            result.innerHTML += "<br>vᵧ² = vᵢᵧ² + 2 * a * ∆y";
            return alert("No solution; try again.");
        }
        result.innerHTML += "<br>" + ("<br>vᵧ² = vᵢᵧ² + 2 * a * ∆y");
        switch(u) {
            case vy:
                vy.value = Math.sqrt(viy.value * viy.value + 2 * a * y.value) * (end.value === "falling" ? -1 : 1);
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵧ = ${end.value === "falling" ? "-" : ""}√(${viy.value}² + 2 * ${a} * ${y.value})`);
            break;
            case viy:
                viy.value = Math.sqrt(vy.value * vy.value - 2 * a * y.value) * (start.value === "falling" ? -1 : 1);
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵢᵧ = ${start.value === "falling" ? "-" : ""}√(${vy.value}² - 2 * ${a} * ${y.value})`);
            break;
            case y:
                y.value = (vy.value * vy.value - viy.value * viy.value) / 2 / a;
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`∆y = (${vy.value}² - ${viy.value}²) / 2 / ${a}`);
        }
    }
    else if(!u.length) {
        if(Math.abs(viy.value * viy.value + 2 * a * y.value - vy.value * vy.value) > THRESHOLD) {
            result.innerHTML += "<br><br>ERROR in equation:";
            result.innerHTML += "<br>vᵧ² = vᵢᵧ² + 2 * a * ∆y";
            alert("Conflicting answers; try again.");
        }
    }
}
function eq4() {
    let u = unknowns([y, vy, viy, t]);
    if(u.length === 1) {
        u = u[0];
        if((((u === vy || u === viy) && Math.abs(t.value) < ZERO_THRESHOLD)) || (u === t && Math.abs(vy.value) < ZERO_THRESHOLD)) return;
        result.innerHTML += "<br>" + ("<br>∆y = (vᵧ + vᵢᵧ) / 2 * t");
        switch(u) {
            case y:
                y.value = (+vy.value + +viy.value) / 2 * t.value;
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`∆y = (${vy.value} + ${viy.value}) / 2 * ${t.value}`);
            break;
            case vy:
                vy.value = 2 * y.value / t.value - viy.value;
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵧ = 2 * ${y.value} / ${t.value} - ${viy.value}`)
            break;
            case viy:
                viy.value = 2 * y.value / t.value - vy.value;
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵢᵧ = 2 * ${y.value} / ${t.value} - ${vy.value}`);
            break;
            case t:
                t.value = 2 * y.value / (+vy.value + viy.value);
                result.innerHTML += "<br>" + (`t = 2 * ${y.value} / (${vy.value} + ${viy.value})`);
        }
    }
    else if(!u.length) {
        if(Math.abs((+vy.value + +viy.value) / 2 * t.value - y.value) > THRESHOLD) {
            result.innerHTML += "<br><br>ERROR in equation:";
            result.innerHTML += "<br>∆y = (vᵧ + vᵢᵧ) / 2 * t";
            alert("Conflicting answers; try again.");
        }
    }
}
function eq5() {
    let u = unknowns([x, vx, t]);
    if(u.length === 1) {
        u = u[0];
        if((u === vx && Math.abs(t.value) < ZERO_THRESHOLD) || (u === t && Math.abs(vx.value) < ZERO_THRESHOLD)) return;
        result.innerHTML += "<br>" + ("<br>∆x = vₓ * t");
        switch(u) {
            case x:
                x.value = vx.value * t.value;
                result.innerHTML += "<br>" + (`∆x = ${vx.value} * ${t.value}`);
            break;
            case vx:
                vx.value = x.value / t.value;
                result.innerHTML += "<br>" + (`vₓ = ${x.value} / ${t.value}`);
            break;
            case t:
                t.value = x.value / vx.value;
                result.innerHTML += "<br>" + (`t = ${x.value} / ${vx.value}`);
        }
    }
    else if(!u.length) {
        if(Math.abs(vx.value * t.value - x.value) > THRESHOLD) {
            result.innerHTML += "<br><br>ERROR in equation:";
            result.innerHTML += "<br>∆x = vₓ * t";
            alert("Conflicting answers; try again.");
        }
    }
}
function eq6() {
    let u = unknowns([vx, vi, theta]);
    if(u.length === 1) {
        u = u[0];
        if(u === vi && Math.abs(Math.cos(theta.value)) < ZERO_THRESHOLD) return;
        if(u !== theta) result.innerHTML += "<br>" + ("<br>vₓ = vᵢ * cos(θ)");
        switch(u) {
            case vx:
                vx.value = vi.value * Math.cos(theta.value);
                result.innerHTML += "<br>" + (`vₓ = ${vi.value} * cos(${theta.value})`);
            break;
            case vi:
                vi.value = vx.value / Math.cos(theta.value);
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵢ = ${vx.value} / cos(${theta.value})`);
        }
    }
    else if(!u.length) {
        if(Math.abs(vi.value * Math.cos(theta.value) - vx.value) > THRESHOLD) {
            result.innerHTML += "<br><br>ERROR in equation:";
            result.innerHTML += "<br>vₓ = vᵢ * cos(θ)";
            alert("Conflicting answers; try again.");
        }
    }
}
function eq7() {
    let u = unknowns([viy, vi, theta]);
    if(u.length === 1) {
        u = u[0];
        if(u === vi && Math.abs(Math.sin(theta.value)) < ZERO_THRESHOLD) return;
        if(u !== theta) result.innerHTML += "<br>" + ("<br>vᵢᵧ = vᵢ * sin(θ)");
        switch(u) {
            case viy:
                viy.value = vi.value * Math.sin(theta.value);
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵢᵧ = ${vi.value} * sin(${theta.value})`);
            break;
            case vi:
                vi.value = viy.value / Math.sin(theta.value);
                updateStartAndEnd()
                result.innerHTML += "<br>" + (`vᵢ = ${viy.value} / sin(${theta.value})`);
        }
    }
    else if(!u.length) {
        if(Math.abs(vi.value * Math.sin(theta.value) - viy.value) > THRESHOLD) {
            result.innerHTML += "<br><br>ERROR in equation:";
            result.innerHTML += "<br>vᵢᵧ = vᵢ * sin(θ)";
            alert("Conflicting answers; try again.");
        }
    }
}
function eq8() {
    let u = unknowns([theta, viy, vx]);
    if(u.length === 1) {
        u = u[0];
        if(u === theta) {
            result.innerHTML += "<br>" + ("<br>θ = atan2(vᵢᵧ, vₓ)");
            theta.value = Math.atan2(viy.value, vx.value);
            result.innerHTML += "<br>" + (`θ = atan2(${viy.value}, ${vx.value})`);
        }
    }
    else if(!u.length) {
        if(Math.abs(Math.atan2(viy.value, vx.value) - theta.value) > THRESHOLD) {
            result.innerHTML += "<br><br>ERROR in equation:";
            result.innerHTML += "<br>θ = atan2(vᵢᵧ, vₓ)";
            alert("Conflicting answers; try again.");
        }
    }
}
function eq9() {
    let u = unknowns([vi, viy, vx]);
    if(u.length === 1) {
        u = u[0];
        if(u === viy && start.value === "unknown") return;
        if((u === viy && vx.value > vi.value) || (u === vx && vy.value > vi.value)) {
            result.innerHTML += "<br><br>NO SOLUTION for equation:";
            result.innerHTML += "<br>vᵢ² = vₓ² + vᵢᵧ²";
            return alert("No solution; try again.");
        }
        result.innerHTML += "<br>" + ("<br>vᵢ² = vₓ² + vᵢᵧ²");
        switch(u) {
            case vi:
                vi.value = Math.sqrt(vx.value * vx.value + viy.value * viy.value);
                result.innerHTML += "<br>" + (`vᵢ = √(${vx.value}² + ${viy.value}²)`);
            break;
            case viy:
                viy.value = Math.sqrt(vi.value * vi.value - vx.value * vx.value) * (start.value === "falling" ? -1 : 1);
                result.innerHTML += "<br>" + (`vᵢᵧ = ${start.value === "falling" ? "-" : ""}√(${vi.value}² - ${vx.value}²)`);
            break;
            case vx:
                vx.value = Math.sqrt(vi.value * vi.value - viy.value * viy.value);
                result.innerHTML += "<br>" + (`vₓ = √(${vi.value}² - ${viy.value}²)`);
            break;
        }
    }
    else if(!u.length) {
        if(Math.abs(vx.value * vx.value + viy.value * viy.value - vi.value * vi.value) > THRESHOLD) {
            result.innerHTML += "<br><br>ERROR in equation:";
            result.innerHTML += "<br>vᵢ² = vₓ² + vᵢᵧ²";
            alert("Conflicting answers; try again.");
        }
    }
}

button.onclick = function() {
    for(const input of inputs) {
        if(input.value !== "") input.value = Number(input.value);
    }
    result.innerHTML = "Steps:";
    let u = unknowns(inputs).length;
    while(u) {
        eq6();
        eq9();
        eq7();
        eq8();
        eq5();
        eq2();
        eq4();
        eq1();
        eq3();
        const nu = unknowns(inputs).length;
        if(nu === u) return alert("Something went wrong. Please try again.");
        u  = nu;
    }
};

</script>
