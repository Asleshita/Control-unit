# Control Unit Simulator: Hardwired vs Microprogrammed

An interactive educational website for the Computer Organization and Architecture (COA) mini-project
**“Design and Comparison of Hardwired and Microprogrammed Control Units.”**

Students pick an instruction, set register and memory values, and execute it step by step. Every step shows
the control signals, the register transfers they cause and the animated data and control paths in a CPU block
diagram. The same instruction can be run through a **hardwired** control unit (logic equations) and a
**microprogrammed** control unit (control memory), and the two are compared side by side.

> The CPU is a deliberately simplified teaching model with five instructions. It does not simulate any real processor.

---

## Requirements

- **Node.js 18 or newer** (the current LTS, 20 or 22, is recommended). Check with `node -v`.
- **npm** (installed together with Node.js). Check with `npm -v`.
- A modern browser (Chrome, Edge, Firefox or Safari).

Download Node.js from <https://nodejs.org> if you don't have it.

## Run it in VS Code

1. Unzip the project and open the `control-unit-simulator` folder in VS Code
   (**File → Open Folder…**).
2. Open the built-in terminal: **Terminal → New Terminal** (or <kbd>Ctrl</kbd>+<kbd>`</kbd>).
3. Install the dependencies (first time only):

   ```bash
   npm install
   ```

4. Start the development server:

   ```bash
   npm run dev
   ```

   The site opens automatically at **http://localhost:5173**. If it doesn’t, open that address yourself.
   Edits to the source files reload the page instantly.
5. To stop the server, click in the terminal and press <kbd>Ctrl</kbd>+<kbd>C</kbd>.

### Other commands

| Command           | What it does                                                  |
| ----------------- | ------------------------------------------------------------- |
| `npm run dev`     | Start the development server with live reload                 |
| `npm run build`   | Create an optimised production build in `dist/`               |
| `npm run preview` | Serve the production build locally to check it                |
| `npm test`        | Run the unit tests for the simulation logic (no browser needed) |

The production build in `dist/` is a static site. It uses hash routes (`#/simulator`), so it works on any static
host (GitHub Pages, Netlify, a college server) without extra configuration.

### Troubleshooting

- **`'vite' is not recognized` / `command not found`**: run `npm install` first, inside the project folder.
- **Port 5173 is already in use**: Vite picks the next free port and prints it in the terminal; open that one.
- **`npm install` fails**: check your internet connection, update Node.js, then delete `node_modules` and
  `package-lock.json` (if present) and run `npm install` again.
- **Blank page**: open the browser developer tools (<kbd>F12</kbd>) → Console and read the error message.

---

## Modules

| Module                       | What you can do there |
| ---------------------------- | --------------------- |
| **Dashboard**                | Project overview, live CPU diagram, current simulation status, quick links. |
| **Simulator**                | Choose ADD / SUB / LOAD / STORE / JUMP, edit operands, register values, PC and memory, choose the control unit, then Execute, Play/Pause, Previous, Next and Reset. Shows each stage’s explanation, active signals, register transfers, components and results, plus an event log. The final state can be carried forward to chain instructions. |
| **Hardwired control**        | Animated block diagram (IR → opcode decoder, timing generator → combinational logic → control lines → datapath), the sum-of-products equations with the true terms highlighted, and a timing diagram of every control line over T0–T4. |
| **Microprogrammed control**  | Animated block diagram (µPC → control memory → µIR → signals, microsequencer, mapping logic), the microprogram table, the binary µIR contents, and a **microcode editor**: change a microinstruction and see the instruction’s behaviour change, together with the logic equations a hardwired redesign would need. |
| **CPU block diagram**        | Select any component (PC, IR, control unit, register file, ALU, memory and supporting parts) to read what it does and which signals drive it. Paths animate during simulation. |
| **Comparison**               | Criteria table, an adjustable **illustrative** latency model with a chart and table, and the selected instruction run through both units step by step. |
| **Learning modules**         | Seven short lessons with diagrams, each linking to the matching part of the simulator. |
| **Quiz**                     | 13 multiple-choice questions with feedback and explanations, final score, review and retake (shuffled). |
| **Glossary**                 | 31 searchable terms. Dotted-underlined terms across the site show these definitions as tooltips. |
| **Project information**      | Objective, usage, the CPU model, the control signal list, tools and reference books. |

**Look and feel.** The background is an animated circuit board: signal pulses run along the copper traces
(blue for data, amber for control), bits stream along the data bus, the ALU chip keeps adding real binary
numbers, the CU chip steps its T0–T4 lights and the µROM chip scrolls through this simulator’s actual
microprogram. The dashboard and the simulator also show the ALU adding bit by bit (with the carry rippling
from right to left) and the control memory as raw microinstruction bits. Use **Board animation** at the
bottom of the sidebar to switch the motion off (the choice is remembered); it is also off automatically if
the computer asks for reduced motion.

Keyboard shortcuts on the simulator pages: <kbd>←</kbd> previous step, <kbd>→</kbd> next step,
<kbd>Space</kbd> play/pause.

---

## The teaching CPU model

**Instruction word (16 bits)**

| Bits  | 15–12  | 11–10 | 9–8 | 7–0                     |
| ----- | ------ | ----- | --- | ----------------------- |
| Field | opcode | Rd    | Rs  | memory address / target |

| Instruction       | Opcode | Meaning                 | Flags        |
| ----------------- | ------ | ----------------------- | ------------ |
| `ADD Rd, Rs`      | 0001   | Rd ← Rd + Rs            | Z, N, C, V   |
| `SUB Rd, Rs`      | 0010   | Rd ← Rd − Rs            | Z, N, C (borrow), V |
| `LOAD Rd, [addr]` | 0011   | Rd ← M[addr]            | unchanged    |
| `STORE Rd, [addr]`| 0100   | M[addr] ← Rd            | unchanged    |
| `JUMP target`     | 0101   | PC ← target             | unchanged    |

- Four 16-bit registers R0–R3 (two’s complement, wrap-around arithmetic).
- 256 words of 16-bit memory shared by instructions and data; 8-bit PC.
- Internal registers: IR, MDR, ALU input latches A and B, ALUOut, flags.
- Every instruction takes five clock steps. Registers update together at the end of each step.
- Inputs accept decimal (`25`, `-7`) or hexadecimal (`0x19`). Invalid values are reported next to the field and
  execution is blocked until they are fixed.

**Control signals**: MemRead, MemWrite, AddrSel (PC / IR), IRLoad, PCInc, PCSrc (PC+1 / IR), PCLoad, RegRead,
ALUOp (— / ADD / SUB), FlagWrite, RegWrite, WBSel (ALU / MEM).

**Control table** (the single specification both control units implement)

| Step | Stage                     | ADD / SUB                | LOAD                    | STORE                  | JUMP                  |
| ---- | ------------------------- | ------------------------ | ----------------------- | ---------------------- | --------------------- |
| T0   | Instruction fetch         | AddrSel=PC, MemRead, IRLoad, PCInc | same | same | same |
| T1   | Instruction decode        | none (opcode decoded inside the control unit) | same | same | same |
| T2   | Control signal generation | RegRead                  | AddrSel=IR              | RegRead                | PCSrc=IR              |
| T3   | Execution                 | ALUOp=ADD/SUB, FlagWrite | AddrSel=IR, MemRead     | AddrSel=IR             | PCSrc=IR              |
| T4   | Register / memory update  | RegWrite, WBSel=ALU      | RegWrite, WBSel=MEM     | AddrSel=IR, MemWrite   | PCSrc=IR, PCLoad      |

**Hardwired unit**: the equations are derived automatically from that table (D1–D5 are the decoder outputs for
ADD, SUB, LOAD, STORE, JUMP):

```
MemRead     = T0 + D3·T3
MemWrite    = D4·T4
AddrSel=IR  = D3·(T2 + T3) + D4·(T3 + T4)
IRLoad      = T0
PCInc       = T0
PCSrc=IR    = D5·(T2 + T3 + T4)
PCLoad      = D5·T4
RegRead     = (D1 + D2 + D4)·T2
ALUOp=ADD   = D1·T3
ALUOp=SUB   = D2·T3
FlagWrite   = (D1 + D2)·T3
RegWrite    = (D1 + D2 + D3)·T4
WBSel=MEM   = D3·T4
```

**Microprogrammed unit**: control memory holds 17 horizontal microinstructions (13 control bits + 2-bit
sequencing field NEXT / MAP / FETCH). µ0 fetch and µ1 decode are shared; the mapping logic sends
ADD → µ2, SUB → µ5, LOAD → µ8, STORE → µ11, JUMP → µ14. Each routine ends with FETCH (back to µ0).

All results are computed by applying the generated control words to the datapath (`src/sim/datapath.js`);
nothing is looked up from a table of answers. A unit test checks that both control units produce identical
control words and identical final states for every instruction.

---

## Project structure

```
control-unit-simulator/
├── index.html                 Page shell loaded by Vite
├── package.json               Scripts and dependencies
├── vite.config.js             Vite + React configuration
├── public/favicon.svg
├── tests/
│   └── simulation.test.js     Unit tests for the simulation core (npm test)
└── src/
    ├── main.jsx               Entry point (fonts, styles, <App/>)
    ├── App.jsx                Layout and hash-based page routing
    ├── routes.js              Navigation menu
    ├── sim/                   Simulation core, plain JavaScript
    │   ├── isa.js             Instruction set, encoding and decoding
    │   ├── signals.js         Control signals and control words
    │   ├── controlTable.js    Reference control table (T0–T4 per instruction)
    │   ├── datapath.js        Signal-driven datapath executor and ALU
    │   ├── hardwired.js       Sum-of-products equations and their evaluation
    │   ├── microprogram.js    Control memory, mapping ROM, microsequencer
    │   ├── trace.js           Builds the step-by-step execution trace
    │   ├── activity.js        Which diagram paths/components are active
    │   ├── validation.js      Input parsing and error messages
    │   └── format.js          Number formatting helpers
    ├── state/
    │   └── SimulationContext.jsx  Shared state, playback timer, event log
    ├── hooks/                 Routing and keyboard shortcuts
    ├── components/            Reusable UI: diagrams, panels, controls, latency lab,
    │                          circuit-board background (HardwareBackdrop.jsx), bit views (BitViews.jsx)
    ├── pages/                 One file per module
    ├── data/                  Glossary, quiz, comparison text, component info
    └── styles/global.css      All styling (dark navy theme, responsive layout)
```

## Suggested demonstration (5 minutes)

1. **Dashboard**: introduce the project and the CPU diagram.
2. **Simulator**: run `ADD R1, R2` (25 + 17). Pause on each stage and point out the active signals and the
   blue/amber paths. Show the result R1 = 42, then *Use this result as the next starting state* and run
   `STORE R1, [100]`.
3. **Hardwired control**: step to T3 and show which AND terms fire and the timing diagram.
4. **Microprogrammed control**: step through µ0 → µ1 → MAP → µ2–µ4. Then choose *Edit microcode*, change µ3’s
   ALUOp from ADD to SUB, and show that ADD now subtracts, plus the hardwired equations that would need
   redesigning.
5. **Comparison**: change the illustrative latencies (try the *overlapped fetch* scenario) to show that neither
   design is faster in every case.
6. **Quiz**: answer a few questions.

## Technology

React 19 + Vite, plain CSS, [Lucide](https://lucide.dev) icons and the IBM Plex Sans / Mono fonts (installed
through npm, so the site works offline after `npm install`). Diagrams and charts are hand-written SVG and HTML.
No external services or paid APIs are used.

## Note on the timing model

The latency values on the Comparison page are **illustrative only**. They are example numbers for teaching,
not benchmark measurements. Real performance depends on the architecture, technology and implementation.

## References

- M. Morris Mano, *Computer System Architecture*
- Carl Hamacher, Zvonko Vranesic, Safwat Zaky, *Computer Organization*
- William Stallings, *Computer Organization and Architecture*
- John P. Hayes, *Computer Architecture and Organization*

---

Computer Organization and Architecture — educational mini-project.
