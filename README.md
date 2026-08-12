# Forge Kernel v3.2 RC-2

A governed, contract-enforced HTML/JS kernel with an Analysis Suite extension
and a unified QualityGate integration.

## Author

- **Yoandis Rodríguez**
- Email: curvadigital0@gmail.com

## License

MIT License

Copyright (c) 2026 Yoandis Rodríguez

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## Status

- Forge Kernel v3.2 RC-2 — Governance Complete
- 15/15 constitutional invariants
- 17 modules bound
- 14/14 diagnostics in green (diag11, diag13, diag15, diag16, diag17, diag20,
  diag20a, diag21a, diag21b, diag22, diag23, diag24, diag25, diag26b)
- QG-INT-01 closed: `QualityGateSwitch` and `kernelQG.runAll()` now share
  identical observable semantics for `kernelQG.results`.
- Analysis Suite integrated (IIFE-local, not a constitutional module).

## Architecture summary

The kernel exposes a frozen `window.IX` surface (37 keys) that:

- declares and validates the executable constitution (`IX.CONTRACT`, 5 frozen
  contracts: ModuleSpec, Signal, ModuleState, PerformanceModel,
  TelemetryStats);
- enforces the single entry point for new modules via
  `IX.registerModule(spec)` (validated against the constitution, frozen
  in place);
- governs capabilities through a frozen `IX.CAPABILITIES` catalogue and
  `IX.hasCapability(code, capability)` / `IX.getCapability(code, capability)`;
- exposes a semantic version map (`IX.VERSION`) and
  `IX.assertCompatibility(spec)` to reject outdated contracts.

The Analysis Suite is implemented as a local IIFE controller (`analysisUI`)
that consumes the kernel's real graph (`forgeUI.graph`) and QualityGate
results (`kernelQG.results`). It is not registered as a module and does
not declare capabilities.

## Running the diagnostics

```bash
NODE_PATH=/usr/local/lib/node_modules node /workspace/diag11.js
NODE_PATH=/usr/local/lib/node_modules node /workspace/diag25.js
NODE_PATH=/usr/local/lib/node_modules node /workspace/diag26b.js
```

Each diagnostic must print a `PASS` line for the build to be considered
green. The current state is **14/14 PASS**.

## Files in this directory

- `forge-in01.html` — the kernel monolith (single file, ~13,150 lines).
- `forge-in01.html.bak.before-accreditation` — pre-accreditation snapshot
  (preserved for diff/audit; the accreditation change is contained in
  the HTML comment block in the header).
- `README.md` — this file.

## Reporting Bugs

Si encuentras un error, por favor repórtalo por escrito a
**curvadigital0@gmail.com** incluyendo:

1. **Qué esperabas que pasara**
2. **Qué pasó realmente**
3. **Pasos para reproducirlo**

Esto ayuda a corregir el código de forma eficiente.

## Contact

Yoandis Rodríguez (Andy) — curvadigital0@gmail.com
