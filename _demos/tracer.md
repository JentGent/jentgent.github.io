---
layout: post
title: Code Tracer
parent: "demos"
thumbnail-path: "/img/demos/tracer/thumbnail.png"
short-description: Track variables line by line

---

<style>
    #code {
        border-radius: 20px;
        background-color: rgb(200, 200, 200);
    }
</style>

<div id="first" style="text-align: center;">
    First, copy your code and press
    <button onclick="fillCode();">this button</button>
</div>
<pre id="code" hidden></pre>
<div id="setNumbers" style="text-align: center;" hidden>
    Then,
    <button onclick="setNumbers()">set the line numbers</button>
</div>

<script>
    let codeText = "";
    async function fillCode() {
        const clipboardContents = await navigator.clipboard.read();

        let hadText = false;
        for (const item of clipboardContents) {
            if (item.types.includes("text/html")) {
                const blob = await item.getType("text/html");
                const formattedHtml = await blob.text();
                document.getElementById("code").innerHTML = formattedHtml;
            }
            if (item.types.includes("text/plain")) {
                const blob = await item.getType("text/plain");
                const text = await blob.text();
                hadText = true;
                codeText = text;
            }
        }
        if (hadText) {
            document.getElementById("code").hidden = false;
            document.getElementById("setNumbers").hidden = false;
        }
    };
    function setNumbers() {
        document.getElementById("first").hidden = true;
    }
</script>