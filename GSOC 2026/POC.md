# POC Ideas for GSoC Selection (WasmEdge Instruction Refactor)

This file is for **selection-focused POC planning**.  
Goal: show mentors you understand the problem deeply and can execute safely, with measurable results.

---

## 1) What kind of POC should we build?

For this project, a good POC is **not full refactor immediately**.  
A strong POC should prove 3 things first:

1. You can measure current memory cost clearly.
2. A split design actually saves memory.
3. You can integrate it without breaking correctness.

So we should do **step-by-step POCs**.

---

## 2) Recommended POC roadmap (best for mentor confidence)

## POC-0: Baseline Measurement (no behavior change)

### What to implement
- Add a small measurement utility (dev-only / test-only) that reports:
  - instruction count per function/module
  - category counts (no immediate, small immediate, large immediate, dynamic immediate)
  - estimated memory consumed by current `Instruction` vectors

### What to show
- A table like:
  - module name
  - total instructions
  - count by immediate type
  - estimated instruction memory footprint

### Why this is strong
- Shows you understand current problem with data, not assumptions.

---

## POC-1: Side-by-side Prototype Container (limited opcodes)

### What to implement
- Create an experimental container in a separate namespace/file, for example:
  - `OpCodeVec` + `ImmediateBlob` + `ImmediateOffsetVec`
- Support only a safe subset first:
  - no-immediate ops (`drop`, `i32.add`, etc.)
  - simple-index ops (`local.get`, `call`)
  - const ops (`i32.const`, `i64.const`)
  - memory arg ops (`i32.load/store`)

### What to show
- Before/after memory estimate for same instruction streams.
- Unit tests proving encode/decode roundtrip for these supported opcodes.

### Why this is strong
- Proves the core idea is practical in WasmEdge code style.

---

## POC-2: Loader Integration Spike (still partial)

### What to implement
- In loader path, add an optional compile flag or dev mode:
  - parse instructions into both:
    - old `InstrVec`
    - new split representation
- Validate that both decoded results are logically equivalent for supported opcodes.

### What to show
- Differential checker output:
  - opcode match
  - immediate value match
  - count match

### Why this is strong
- Demonstrates safe migration strategy and regression resistance.

---

## POC-3: Executor Read Path Experiment

### What to implement
- Add a tiny adapter for executor dispatch to read immediate from split representation for subset ops.
- Keep old path available as fallback.

### What to show
- Correctness tests pass for subset.
- Basic runtime overhead check (no big slowdown for subset).

### Why this is strong
- Shows you think about performance tradeoff, not only memory.

---

## 3) What research part from external resources can we show?

You can show these as design grounding references in your proposal/discussion:

1. WebAssembly spec (binary instruction format)
- [WebAssembly binary instructions](https://webassembly.github.io/spec/core/binary/instructions.html)
- Key point to cite: opcode is byte-encoded and followed by immediates where present.

2. V8 interpreter trait-based operand encoding
- [V8 `bytecode-traits.h`](https://chromium.googlesource.com/v8/v8/+/refs/heads/main/src/interpreter/bytecode-traits.h)
- Key point to cite: mature engines model operand sizes/types explicitly and compute offsets per instruction form.

3. Wasmtime Pulley disassembly/decode ecosystem
- [Wasmtime Pulley disassembler source](https://docs.wasmtime.dev/api/src/pulley_interpreter/disas.rs.html)
- Key point to cite: bytecode stream + explicit immediate decoding/disassembly flow is practical in production runtimes.

Use these carefully: not to copy implementation, but to justify design direction.

---

## 4) What exactly to show mentors in POC meeting

Bring 5 artifacts:

1. **Problem data sheet**
- Current instruction memory estimate with real module samples.

2. **Prototype design doc (1-2 pages)**
- Data structures
- Immediate encoding rules
- Migration plan

3. **Small running code**
- POC container + parser/reader for subset instructions.

4. **Correctness proof**
- Differential tests old vs new for subset.

5. **Preliminary numbers**
- Memory reduction percentage on sample modules.
- Any performance notes (even if neutral/unknown yet).

This is exactly the kind of evidence that makes proposal-based selection stronger.

---

## 5) Concrete initial scope (what we should implement first)

First implementation slice should include:
- `i32.const`, `i64.const`
- `local.get`, `local.set`
- `call`
- `i32.add`, `i32.sub`, `drop`
- `i32.load`, `i32.store` (to include memarg case)

Why this slice:
- covers no-immediate, scalar-immediate, and compound-immediate patterns
- enough to prove architecture and testing approach
- small enough to finish quickly

---

## 6) POC success criteria (clear and simple)

POC is successful if:
- we show measurable memory improvement in prototype storage,
- tests prove immediate correctness for chosen subset,
- mentors can see a realistic path to full integration.

If we cannot show these 3, POC is weak.

---

## 7) Suggested next immediate deliverable

Create `GSOC 2026/POC_PLAN_WEEK1.md` with:
- day-by-day tasks (7 days),
- exact files to touch,
- exact metrics to collect,
- test list to run.

This turns the idea into execution.

---

## 8) Quick sanity checklist before sharing

- each POC has a runnable command in README,
- each POC has a sample input file checked in,
- each POC prints clear PASS/FAIL-style summary output,
- consolidated result summary is updated after reruns.

