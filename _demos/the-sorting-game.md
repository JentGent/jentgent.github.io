---
layout: post
title: The Sorting Game
parent: "demos"
thumbnail-path: "/img/demos/the-sorting-game/thumbnail.png"
short-description: Sort like a computer
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
</style>

{:.center}
#### Click or press "Enter" to compare items and discover the hidden order!

{:.center}
<label for="numItems">Number of items to sort:</label>
<input id="numItems" type="range" value="10" min="2" max="26">
<span id="numItemsValue">10</span>
<br>
<label for="numCompare">Number of items per comparison:</label>
<input id="numCompare" type="range" value="2" min="2" max="10">
<span id="numCompareValue">2</span>
&ensp; <button onclick="N = +document.getElementById('numItems').value; NUM_COMPARE = +document.getElementById('numCompare').value; reset();">Reset</button>

<script>
    const numItems = document.getElementById("numItems");
    const numItemsValue = document.getElementById("numItemsValue");

    const numCompare = document.getElementById("numCompare");
    const numCompareValue = document.getElementById("numCompareValue");

    numItems.addEventListener("input", () => {
        numItemsValue.textContent = numItems.value;
    });

    numCompare.addEventListener("input", () => {
        numCompareValue.textContent = numCompare.value;
    });
</script>


<script src="https://cdn.jsdelivr.net/gh/JentGent/2D-physics-engine@latest/demos/libraries/p5.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.4/addons/p5.sound.min.js"></script>
<script>
    let canvas, width = 1200, height = 600;
    function setup() {
        canvas = createCanvas(width, height);
        canvas.parent("sketch");
        canvas.elt.oncontextmenu = () => false;
        canvas.elt.style = "width: 100%;";
    }
    let selectSound, sortSound, winSound;
    function preload() {
        selectSound = loadSound("/content/the-sorting-game-select.wav");
        sortSound = loadSound("/content/the-sorting-game-sort.wav");
        winSound = loadSound("/content/the-sorting-game-win.wav");
    }
</script>
<script>
    
let N = 10;
let SPACING = 10;
let NODE_DIST = 0;
let NODE_SIZE = 0;
let NUM_COMPARE = 2;
const ANIMATION_RATIO = 0.2;
let alreadyAnimation = 0;
let winAnimation = 1;
const WIN_ANIMATION_RATIO = 0.04;
const ALREADY_ANIMATION_RATIO = 0.95;

const TWO_TABLE = [0, 1, 3, 5, 7, 10, 13, 16, 19, 22, 26, 30, 34, 38, 42, 46, 50, 54, 58, 62, 66, 71];

function shuffle(arr) {
    for (let i = arr.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        const temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
    return arr;
}

function isHovering(x, y, w, h) {
    return mouseX > x && mouseX < x + w && mouseY > y && mouseY < y + h;
}

function Node(key, x, y) {
    this.key = key;
    this.x = x;
    this.y = y;
    this.size = NODE_SIZE;
    nodeMap[this.key] = this;
    this.chains = {};
}
Node.prototype.display = function() {
    let minChain = undefined;
    let yTarget = height * 5/7;
    for (let i = 0; i < nodes.length; i++) {
        const node = nodes[i];
        if (node === this) continue;
        if (node.x < this.x - (this.size + node.size) / 2 || node.x > this.x + (this.size + node.size) / 2) continue;
        minChain = (minChain === undefined ? chains.findIndex(function(chain) { return chain.nodes.includes(this); }, this) : minChain);
        const minChain2 = chains.findIndex(function(chain) { return chain.nodes.includes(node); });
        if (minChain > minChain2) {
            yTarget = Math.min(yTarget, node.y - NODE_DIST);
        }
    }
    this.y += (yTarget - this.y) * ANIMATION_RATIO;

    fill(255);
    if (this.isSelected) {
        fill(230, 255, 230);
    } else if (hoveringNode === this) {
        fill(240);
    } else if (hoveringNode && dfs(hoveringNode, this)) {
        fill(240, 230, 200);
    } else if (hoveringNode && dfs(this, hoveringNode)) {
        fill(230, 230, 255);
    }
    stroke(0);
    strokeWeight(1);
    rect(this.x - this.size / 2, this.y - this.size / 2, this.size, this.size);
    fill(100);
    textAlign(CENTER, CENTER);
    textSize(NODE_SIZE/2);
    noStroke();
    text(this.label, this.x, this.y);
};
Node.prototype.isHovering = function() {
    return isHovering(this.x - this.size / 2, this.y - this.size / 2, this.size, this.size);
};
Node.prototype.toString = function() {
    return this.key;
};

function Chain(nodes) {
    this.nodes = [];
    this.id = (Chain.id++).toString();
    if (nodes) this.add(nodes);
    chainMap[this.id] = this;
}
Chain.id = 0;
Chain.prototype.display = function() {
    let x = 0, y = 0;
    for (let i = 0; i < this.nodes.length; i++) {
        x += this.nodes[i].x;
        y += this.nodes[i].y;
    }
    x /= this.nodes.length; y /= this.nodes.length;
    
    for (let i = 0; i < this.nodes.length; i++) {
        const node = this.nodes[i];
        const xTarget = x + (i - this.nodes.length/2 + 0.5) * NODE_DIST;
        node.x += (xTarget - node.x) * ANIMATION_RATIO;
    }
    
    stroke(150);
    strokeWeight(5);
    for (let i = 1; i < this.nodes.length; i++) {
        const n1 = this.nodes[i - 1], n2 = this.nodes[i];
        line(n1.x, n1.y, n2.x, n2.y);
    }
};
Chain.prototype.toString = function() {
    return "{" + this.nodes + "}";
};
Chain.prototype.add = function(nodes, idx) {
    if (!Array.isArray(nodes)) return this.add([nodes], idx);
    if (idx === undefined) idx = this.nodes.length;
    if (idx > 0) this.nodes[idx - 1].chains[this.id].next = nodes[0];
    if (idx < this.nodes.length) this.nodes[idx].chains[this.id].prev = nodes.at(-1);
    for (let i = 0; i < nodes.length; i++) {
        const order = nodes[i].chains[this.id] = {};
        order.prev = (i > 0 ? nodes[i - 1] : this.nodes[idx - 1]);
        order.next = (i < nodes.length - 1 ? nodes[i + 1] : this.nodes[idx]);
    }
    Array.prototype.splice.apply(this.nodes, [idx, 0].concat(nodes));
};
Chain.prototype.remove = function(start, end) {
    if (end === undefined) end = start + 1;
    if (end < start) return this.remove(end, start);
    if (end === start) return [];
    if (start > 0) {
        this.nodes[start - 1].chains[this.id].next = this.nodes[end];
    }
    if (end < this.nodes.length) {
        this.nodes[end].chains[this.id].prev = this.nodes[start - 1];
    }
    for (let i = start; i < end; i++) {
        delete this.nodes[i].chains[this.id];
    }
    return this.nodes.splice(start, end - start);
};
Chain.prototype.removeAll = function() {
    this.remove(0, this.nodes.length);
};

function dfs(start, target, disallowed, backward, prevChainId, history) {
    if (!history) history = [];
    if (start === target) return history;
    const chainIds = Object.keys(start.chains);
    if (prevChainId !== undefined) chainIds.unshift(prevChainId);
    for (let i = 0; i < chainIds.length; i++) {
        const chainId = chainIds[i];
        if (chainId === prevChainId && i > 0) continue;
        const next = (backward ? start.chains[chainId].prev: start.chains[chainId].next);
        const forbidden = (disallowed && disallowed.includes(chainId));
        if (forbidden) continue;
        if (!next) continue;
        const sub = dfs(next, target, disallowed, backward, prevChainId, history.concat(chainId));
        if (sub) return sub;
    }
    return false;
}

let nodeMap = {}, chainMap = {};
let nodes = [], chains = [], selected = [];
let compared = [];
let hoveringNode = null;
let entropy = 0;
let comparisons = 0;

function reset() {
    NODE_DIST = Math.min((width - SPACING) / N, 90);
    NODE_SIZE = NODE_DIST - SPACING;

    comparisons = compared.length = nodes.length = chains.length = selected.length = 0;
    winAnimation = 1;

    for (let i = 0; i < N; i++) {
        nodes.push(new Node(i, 0, height*5/7));
    }
    shuffle(nodes, true);
    for (let i = 0; i < nodes.length; i++) {
        nodes[i].x = width/2 + (i - nodes.length/2 + 0.5) * NODE_DIST;
        nodes[i].label = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"[i];
    }

    if (NUM_COMPARE === 2 && N <= TWO_TABLE.length) {
        entropy = TWO_TABLE[N - 1];
    } else {
        entropy = 0;
        for (let i = 2; i <= N; i++) {
            entropy += Math.log(i);
        }
        let fac = 0;
        for (let i = 2; i <= NUM_COMPARE; i++) {
            fac += Math.log(i);
        }
        entropy /= fac;
        entropy = Math.ceil(entropy - 0.00000001);
    }
    entropy++;
}
reset();

function getSave() {
    return nodes.map(
        function(node) { return node.key; }
    ) + "; " + compared.map(
        function(c) {
            return "[" + c.map(function(node) { return node.label; }) + "]";
        }
    );
}

function compare(nodes) {
    let already = true;
    A: for (let i = 1; i < nodes.length; i++) {
        for (let j = 0; j < i; j++) {
            if (!dfs(nodes[i], nodes[j]) && !dfs(nodes[j], nodes[i])) {
                already = false;
                break A;
            }
        }
    }
    if (already) {
        alreadyAnimation = 255;
        return;
    }
    compared.push(nodes.slice());
    comparisons++;
    chains.push(new Chain(nodes.sort(function(a, b) { return a.key - b.key; })));
    while (true) {
        let change = false;
        for (let i = 0; i < chains.length; i++) {
            const chain1 = chains[i];
            for (let j = i + 1; j < chains.length; j++) {
                const chain2 = chains[j];
                const firstIdx = chain2.nodes.indexOf(chain1.nodes[0]);
                if (firstIdx >= 0 && firstIdx > 0) {
                    const removed = chain2.remove(0, firstIdx);
                    chain1.add(removed, 0);
                    change = true;
                }
                const lastIdx = chain2.nodes.indexOf(chain1.nodes.at(-1));
                if (lastIdx >= 0 && lastIdx < chain2.nodes.length - 1) {
                    const removed = chain2.remove(lastIdx + 1, chain2.nodes.length);
                    chain1.add(removed);
                    change = true;
                }
            }
            const disallowed = [chain1.id];
            for (let j = 0; j < chain1.nodes.length - 1; j++) {
                let start = chain1.nodes[j], target = chain1.nodes[j + 1];
                const search = dfs(start, target, disallowed);
                if (!search) continue;
                change = true;
                const searchChains = search.map(
                    function(id) {
                        return chains.find(function(a) {
                            a.removeStart = undefined;
                            return a.id === id;
                        });
                    }
                );
                let node = start;
                for (let k = 0; k < search.length; k++) {
                    node = node.chains[search[k]].next;
                    const chain2 = searchChains[k];
                    const idx = chain2.nodes.indexOf(node);
                    chain2.removeStart = (chain2.removeStart === undefined ? idx : Math.min(chain2.removeStart, idx));
                    chain2.removeEnd = idx;
                    if (k < search.length - 1) chain1.add(node, j + 1 + k);
                }
                for (let k = 0; k < searchChains.length; k++) {
                    const chain2 = searchChains[k];
                    if (chain2.removeEnd < chain2.nodes.length - 1) {
                        const removed = chain2.remove(chain2.removeEnd, chain2.nodes.length);
                        const newChain = new Chain(removed);
                        const idx = chains.indexOf(chain2);
                        chains.splice(idx + 1, 0, newChain);
                    }
                    if (chain2.removeStart < chain2.nodes.length) {
                        chain2.remove(chain2.removeStart, chain2.nodes.length);
                    }
                }
            }
        }
        for (let i = chains.length - 1; i >= 0; i--) {
            if (chains[i].nodes.length < 2) {
                chains[i].removeAll();
                chains.splice(i, 1);
                change = true;
            }
        }
        if (!change) break;
    }
    if (chains.length === 1 && chains[0].nodes.length === N) {
        winSound.play();
        winAnimation -= WIN_ANIMATION_RATIO;
    }
}

function mousePressed() {
    for (let i = nodes.length - 1; i >= 0; i--) {
        const node = nodes[i];
        if (!node.isHovering()) continue;
        if (node.isSelected) {
            node.isSelected = false;
            selected.splice(selected.indexOf(node), 1);
        } else {
            if (selected.length >= NUM_COMPARE) return;
            selected.push(node);
            node.isSelected = true;
        }
        console.log('hi"');
        selectSound.play();
        break;
    }
}
function doComparison() {
    if (!(selected.length >= 2)) return;
    sortSound.rate(random(0.9, 1.3));
    sortSound.play();
    compare(selected);
    for (let i = 0; i < selected.length; i++) {
        selected[i].isSelected = false;
    }
    selected = [];
}
function keyPressed() {
    if (keyCode !== ENTER) return;
    doComparison();
}

function draw() {
    background(255);

    hoveringNode = null;
    for (let i = nodes.length - 1; i >= 0; i--) {
        if (nodes[i].isHovering()) {
            hoveringNode = nodes[i];
            break;
        }
    }
    
    for (let i = 0; i < chains.length; i++) {
        chains[i].display();
    }
    for (let i = 0; i < nodes.length; i++) {
        nodes[i].display();
    }

    noStroke();
    fill(255, 0, 0, alreadyAnimation);
    alreadyAnimation *= ALREADY_ANIMATION_RATIO;
    textAlign(CENTER, CENTER);
    textSize(30);
    text("You already know this order!", width/2, height/2);

    if (winAnimation <= 0) {
        winAnimation = 0;
    } else if (winAnimation < 1) {
        winAnimation -= WIN_ANIMATION_RATIO / winAnimation;
    }
    if (winAnimation <= 0) winAnimation = 0;
    fill(100, 200, 100, 255 * (1 - winAnimation));
    textSize(80 + 100 * winAnimation);
    text("Sorted!", width/2, height/3);

    const x = width - 230, y = 50, w = 180, h = 80;
    fill(255);
    if (isHovering(x, y, w, h)) {
        fill(240);
        if (mouseIsPressed) {
            doComparison();
        }
    }
    stroke(0);
    rect(x, y, w, h, 10);

    noStroke();
    fill(0);
    textAlign(LEFT, TOP);
    textSize(30);
    text("Comparisons made: " + comparisons + "\nThe number to beat: " + entropy, 50, 50);
    textAlign(CENTER, CENTER);
    text("Compare", x + w/2, y + h/2);
    if(chains.length) {
        textAlign(CENTER, BOTTOM);
        textSize(20);
        text("(" + chains.map(function(chain) {
            return chain.nodes.map(function(node) {
                return node.label;
            }).join(" < ");
        }).join("), (") + ")", SPACING, height - 200 - SPACING, width - SPACING * 2, 200);
    }
}

</script>

<div id="sketch" class="demo"></div>
<br>
Note: for 23+ items or 3+ items per comparison, the "number to beat" is the information-theoretic lower bound, so it may not always be possible to beat. The exact number of comparisons needed in the worst case is an open problem!

{:.center}
Spot a bug? Send me your <button onclick="alert('Copy this: ' + getSave());">save</button>!

