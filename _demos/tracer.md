---
layout: post
title: Code Tracer
parent: "demos"
thumbnail-path: "/img/demos/tracer/thumbnail.png"
short-description: Track variables line by line

---

<style>
    #code-container {
        border-radius: 1em;
        background-color: rgb(200, 200, 200);
        align-self: flex-end;
    }
    .group {
        display: flex;
        align-items: flex-start;
        overflow: hidden;
    }
    .group-element {
        flex: 1;
    }
    #line-numbers {
        width: 4em;
        text-align: right;
    }
    article {
        padding: 0 10% 50px;
    }
    td, th {
        text-align: center;
    }
</style>

<div id="first" style="text-align: center;">
    First, paste your code here:
</div>
<div class="group">
    <pre class="group-element" id="code-container" contentEditable="plaintext-only"></pre>
    <table class="group-element" id="tracetable" hidden>
        <tr>
            <th>Line</th>
        </tr>
        <tr id="table-bottom">
            <td>
                <button onclick="nextStep();">Next</button>
            </td>
        </tr>
    </table>
</div>
<div id="startTracing" style="text-align: center;">
    Then,
    <button onclick="startTracing()">start tracing</button>!
</div>

<script>
    let currentLine = 0;
    const traceTable = document.getElementById("tracetable");
    const tableBottom = document.getElementById("table-bottom");
    function showTable() {
        document.getElementById("first").hidden = true;
        document.getElementById("startTracing").hidden = true;
        traceTable.hidden = false;
    }
    function lockCode() {
        document.getElementById("code-container").contentEditable = false;
    }
    function addLineNumbers() {
        const codeContainer = document.getElementById("code-container");
        let lineNumbers = "1";
        const numLines = codeContainer.innerHTML.split("\n").length;
        for (let i = 1; i < numLines; i++) {
            lineNumbers += "<br>" + (i + 1);
        }
        codeContainer.innerHTML = "<table><tr><td id='line-numbers'>" + lineNumbers + "</td><td id='code'>" + codeContainer.innerHTML + "</td></tr></table>";
    }
    function nextStep() {
        const row = document.createElement("tr");
        
        const lineNum = document.createElement("td");
        currentLine++;
        lineNum.innerHTML = currentLine;
        row.appendChild(lineNum);

        const prevAddVar = document.getElementById("add-var");
        if (prevAddVar) {
            prevAddVar.parentElement.remove();
        }

        const addVar = document.createElement("td");
        addVar.innerHTML = "<button id='add-var' onclick='addVar();'>Add variable</button>";
        row.appendChild(addVar);

        tableBottom.parentElement.insertBefore(row, tableBottom);
    }
    function addVar() {

    }
    function startTracing() {
        showTable();
        lockCode();
        addLineNumbers();
        nextStep();
    }
</script>