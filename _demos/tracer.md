---
layout: post
title: Code Tracer
parent: "demos"
thumbnail-path: "/img/demos/tracer/thumbnail.png"
short-description: Track variables line by line
order: 1

---

<style>
    #code-container {
        border-radius: 1em;
        background-color: rgb(50, 50, 70);
        align-self: flex-end;
        margin-right: 0.5em;
        color: rgb(180, 200, 220);
    }
    #code {
        text-align: left;
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
    .line-number {
        text-align: right;
    }
    .code-line {
        text-align: left;
    }
    .code-row:hover {
        background-color: rgba(255, 255, 0, 0.1);
    }
    article {
        padding: 0 5% 50px;
    }
    table {
        display: inline-block;
    }
    td, th {
        text-align: center;
        padding: 0.2em;
        border: 0px;
        font-size: 0.9em;
    }
    th {
        text-decoration: underline;
    }
    input {
        width: 100%;
    }
    button {
        padding: 0.25em;
    }
    input {
        text-align: center;
    }
</style>

<div id="first" style="text-align: center;">
    Paste your code here:
</div>
<div class="group">
    <pre class="group-element" id="code-container" contentEditable="plaintext-only"></pre>
    <table class="group-element" id="tracetable" hidden>
        <tr>
            <th>Line</th>
        </tr>
        <tr id="table-bottom">
            <td>
                <button onclick="undo();">Back</button>
                <button onclick="nextStep();">Next</button>
            </td>
        </tr>
    </table>
</div>
<div id="startTracing" style="text-align: center;">
    Then,
    <button onclick="startTracing()">start tracing</button>!
</div>


{:.center}
<button onclick="restart();" id="new-trace" hidden>New Trace</button>

<script>
    let currentStep;
    const steps = [];
    function restart() {
        window.location.reload();
    }
    function getCodeRow(line) {
        return document.getElementById("code-table").children[line - 1];
    }
    class Variable {
        constructor(stepIntroduced) {
            this.element = document.createElement("th");
            this.element.innerHTML = "varName";
            this.element.contentEditable = "plaintext-only";
            this.stepIntroduced = stepIntroduced;
        }
        get name() { return this.element.innerText; }
        freeze() {
            this.element.contentEditable = "false";
        }
        unfreeze() {
            this.element.contentEditable = "plaintext-only";
        }
    }
    class Step {
        constructor(line, variables) {
            this.line = line;
            this.variables = variables;
            this.element = document.createElement("tr");
            
            this.lineNumEl = document.createElement("td");
            this.lineNumEl.innerHTML = line;
            this.element.appendChild(this.lineNumEl);

            if (currentStep) {
                for (let i = 0; i < variables.length; i++) {
                    this.addVarCell(currentStep.getValue(i));
                }
                currentStep.leave();
            }
            this.enter();

            tableBottom.parentElement.insertBefore(this.element, tableBottom);

            this.changesVariables = false;
            this.varRow = null;
        }
        getVarCell(idx) {
            return this.element.children[idx + 1];
        }
        getValue(idx) {
            return this.getVarCell(idx).firstChild.value;
        }
        setLine(line) {
            getCodeRow(this.line).style.backgroundColor = "";
            this.line = line;
            this.lineNumEl.innerHTML = line;
            getCodeRow(line).style.backgroundColor = "rgba(255, 255, 0, 0.2)";
        }
        addRemove(variable) {
            const cell = document.createElement("td");
            const button = document.createElement("button");
            button.onclick = () => {
                this.removeVar(variable);
            };
            button.innerHTML = "Remove";
            cell.appendChild(button);
            tableBottom.appendChild(cell);
        }
        removeVar(variable) {
            if (!this.changesVariables) {
                this.addVarRow();
                this.changesVariables = true;
            }
            const idx = this.variables.indexOf(variable);
            if (idx < 0) return console.error("Variable not found", variable);
            this.variables = this.variables.slice();
            this.variables.splice(idx, 1);
            this.getVarCell(idx).remove();
            tableBottom.children[idx + 1].remove();
            this.varRow.children[idx + 1].remove();
        }
        enter() {
            currentStep = this;
            this.addVarEl = document.createElement("td");
            const button = document.createElement("button");
            button.onclick = this.addVar.bind(this);
            button.innerHTML = "Add variable";
            this.addVarEl.appendChild(button);
            this.element.appendChild(this.addVarEl);
            for (let i = 0; i < this.variables.length; i++) {
                const cell = this.getVarCell(i);
                if (!cell.firstElementChild) {
                    const input = document.createElement("input");
                    input.value = cell.innerText;
                    cell.innerHTML = "";
                    cell.appendChild(input);
                }
                if (this.variables[i].stepIntroduced === this) {
                    this.variables[i].unfreeze();
                }
            }

            getCodeRow(this.line).style.backgroundColor = "rgba(255, 255, 0, 0.2)";

            for (let i = tableBottom.cells.length - 1; i >= 1; i--) {
                tableBottom.cells[i].remove();
            }
            for (let i = 0; i < this.variables.length; i++) {
                this.addRemove(this.variables[i]);
            }
        }
        addVarCell(value = "0") {
            const cell = document.createElement("td");
            const input = document.createElement("input");
            input.value = value;
            cell.appendChild(input);
            this.element.insertBefore(cell, this.addVarEl);
        }
        addVarRow() {
            this.varRow = document.createElement("tr");
            this.varRow.appendChild(document.createElement("td"));
            this.element.parentElement.insertBefore(this.varRow, this.element);
            for (let i = 0; i < this.variables.length; i++) {
                const cell = document.createElement("th");
                cell.innerText = this.variables[i].name;
                this.varRow.appendChild(cell);
            }
        }
        addVar() {
            if (!this.changesVariables) {
                this.addVarRow();
                this.changesVariables = true;
            }
            this.addVarCell();
            const newVar = new Variable(this);
            this.addRemove(newVar);
            this.variables = this.variables.slice();
            this.variables.push(newVar);
            this.varRow.appendChild(newVar.element);
        }
        leave() {
            currentStep = null;
            for (let i = 0; i < this.variables.length; i++) {
                const cell = this.getVarCell(i);
                cell.innerText = cell.firstChild.value;
                this.variables[i].freeze();
            }
            this.addVarEl.remove();
            getCodeRow(this.line).style.backgroundColor = "";
        }
        remove() {
            if (currentStep === this) {
                this.leave();
            }
            this.element.remove();
            if (this.changesVariables) {
                this.varRow.remove();
            }
        }
    }
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
        const lines = codeContainer.innerText.split("\n");
        const table = document.createElement("table");
        table.id = "code-table";
        for (let i = 0; i < lines.length; i++) {
            const row = document.createElement("tr");
            row.classList.add("code-row");
            row.onclick = () => {
                currentStep.setLine(i + 1);
            };
            const lineNum = document.createElement("td");
            lineNum.innerHTML = i + 1;
            lineNum.classList.add("line-number");
            const code = document.createElement("td");
            code.innerHTML = lines[i] || "";
            code.classList.add("code-line");
            row.appendChild(lineNum);
            row.appendChild(code);
            table.appendChild(row);
        }
        codeContainer.innerHTML = "";
        codeContainer.appendChild(table);
    }
    function nextStep() {
        let nextLine = 1;
        if (steps.length) nextLine = steps[steps.length - 1].line + 1;
        if (nextLine > document.getElementById("code-table").children.length) nextLine--;
        steps.push(new Step(nextLine, currentStep ? currentStep.variables : []));
    }
    function undo() {
        if (steps.length === 1) return;
        steps.pop().remove();
        steps[steps.length - 1].enter();
    }
    function startTracing() {
        document.getElementById("new-trace").hidden = false;
        showTable();
        lockCode();
        addLineNumbers();
        nextStep();
    }
</script>