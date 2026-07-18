# Contribution [#]: Provide email address for reaching PSF Board members
 
**Contribution Number:** #14909 
**Student:** Gezahegne Yirefu  
**Issue:** https://github.com/ggml-org/llama.cpp/issues/14909 
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose the ABS operation for the OpenCL backend because it looks like a good beginner-friendly contribution to the llama.cpp codebase. The operation itself is simple to understand: it takes an input value and returns its absolute value. Since it is a basic unary operation, I feel I can focus more on learning how backend operations are implemented in llama.cpp without getting overwhelmed by a very complex algorithm.

This issue also matches my current learning goals in the CodePath AI301 program. I want to get more comfortable reading a large open-source C/C++ codebase, understanding how different backends are organized, and making a small but meaningful contribution. Implementing ABS for OpenCL will help me learn how ggml operations are mapped to backend-specific kernels, how to test backend support using test-backend-ops, and how to prepare a focused pull request.

I also chose this item because it appears to be available: it is marked as not implemented in the backend support table, and I did not find an existing open request or pull request specifically for ABS in OpenCL. This makes it a good issue to claim because the scope is clear, the expected result is testable, and the contribution would fill a real missing piece in the project.

started with EXMP1 but found out it is already implemented and started working on ABS. 

---

## Understanding the Issue

### Problem Description

The ggml **OpenCL** backend does not implement the **`ABS`** unary operation
(`y = |x|`). `ABS` is a standard element-wise unary op (`ggml_abs` →
`GGML_UNARY_OP_ABS`) that is already supported by nearly every other backend
(CPU, CUDA, Metal, SYCL, Vulkan, WebGPU, CANN), but on OpenCL it is missing
entirely — there is no `abs` kernel, no dispatch function, and `supports_op`
returns `false` for it.

As a result, any compute graph that contains an `ABS` node cannot execute that
node on an OpenCL device. The op is reported as unsupported, forcing a fallback
to another backend (CPU) for that node — breaking full-GPU execution and adding
host/device transfer overhead — or failing outright if no fallback is available.

### Expected Behavior

- `ABS` runs natively on OpenCL devices for both `GGML_TYPE_F32` and
  `GGML_TYPE_F16`, for contiguous and non-contiguous tensors.
- `test-backend-ops -o ABS` reports **`OK`** for all 8 ABS test cases on the
  `GPUOpenCL` backend, validated against the CPU reference.
- `docs/ops.md` shows **`✅`** for the OpenCL column of the `ABS` row.
- Numerical results match the CPU reference within tolerance (exact for `|x|`).

### Current Behavior

- `test-backend-ops -o ABS` reports **`not supported [OpenCL]`** for all 8 cases;
  the OpenCL backend runs `0/0` ABS tests.
- `docs/ops.md` shows **`❌`** for the OpenCL column of the `ABS` row:

  ```
  | Operation | BLAS | CANN | CPU | CUDA | MTL | OpenCL | SYCL | Vulkan | WebGPU | ZenDNN | zDNN |
  |       ABS |  ❌  |  ✅  | ✅  |  🟡  | ✅  |   ❌   |  ✅  |   ✅   |   ✅   |   ❌   |  ❌  |
  ```

- `ggml_backend_opencl_device_supports_op()` has no `case GGML_UNARY_OP_ABS`, so
  it falls through to `return false`.
- There is no `ggml_cl_abs()` dispatch function and no `abs.cl` kernel file.

### Affected Components

| Component | Path | Current state |
|-----------|------|---------------|
| OpenCL kernels | `ggml/src/ggml-opencl/kernels/` | No `abs.cl` kernel file |
| Kernel build list | `ggml/src/ggml-opencl/CMakeLists.txt` | `abs` not in `GGML_OPENCL_KERNELS` |
| Backend implementation | `ggml/src/ggml-opencl/ggml-opencl.cpp` | No kernel handles, no loading block, no `ggml_cl_abs`, no op-dispatch case, no `supports_op` case |
| Ops documentation | `docs/ops.md` | OpenCL `ABS` cell is `❌` |
| Test harness (verification) | `tests/test-backend-ops.cpp` | Generates 8 ABS cases that currently report unsupported on OpenCL |

**Reference implementation:** the already-working **`EXPM1`** op spans exactly
these same components and serves as the structural template.


---

## Reproduction Process

### Environment Setup

Phase 1 — Prerequisites setup
1. Installed Git & CMake via winget
Used winget install Git.Git and winget install Kitware.CMake. Both installed successfully.
Challenge
After installing CMake, running cmake in the same PowerShell session gave "not recognized" error.
Fix
Restarted VS Code / terminal so the new PATH entries took effect. CMake was installed correctly — the session just needed a refresh.

2. Installed Visual Studio 2022 Community
Required for the MSVC C++ compiler (cl.exe) and Windows SDK.
Challenge
VS was installed but the C++ workload was missing — cl.exe not found even after activating Dev Shell.
Fix
Opened the VS Installer → Modify → checked "Desktop development with C++" workload including MSVC tools, Windows 11 SDK, and CMake tools for Windows.
Phase 2 — Build environment build

3. Activating the Developer PowerShell
Regular PowerShell doesn't have cl.exe on PATH. Must use Developer PowerShell for VS 2022 or manually import it via Enter-VsDevShell.
Challenge
First cmake run hit: NMake Makefiles — no such file or directory and CMAKE_CXX_COMPILER not set.
Fix
Switched to the VS Dev Shell using Import-Module + Enter-VsDevShell with -DevCmdArguments "-arch=x64". Always run cmake from this shell.
Challenge
Second cmake run: "generator does not match previously used: NMake Makefiles".
Fix
Deleted the stale build/ folder with Remove-Item -Recurse -Force build and re-ran cmake. Lesson: always wipe build folder when switching generators.
Phase 3 — Intel Arc GPU (SYCL) GPU

4. Installed Intel oneAPI Base Toolkit
Provides the SYCL compiler (icx) and oneMKL math library. Needed instead of CUDA for Intel Arc GPUs. Used the latest release, not the archive page.
Challenge
CMake ignored -DCMAKE_CXX_COMPILER=icx and fell back to MSVC, causing "C++ compiler lacks SYCL support".
Fix
The setvars.bat env vars weren't loading into PowerShell via &. Fixed by piping cmd /c setvars.bat && set through ForEach-Object to explicitly import each variable into the PowerShell process.
Challenge
Build error: "Could not expand ICInstallDir — platform toolset set to invalid version" when using -G "Visual Studio 17 2022" with icx.
Fix
Switched generator from Visual Studio to Ninja (-G "Ninja"). Ninja calls icx directly without needing VS toolset integration, avoiding the ICInstallDir lookup entirely.

5. Successful build with SYCL
Final cmake flags: -DGGML_SYCL=ON -DGGML_CUDA=OFF -DCMAKE_C_COMPILER=cl -DCMAKE_CXX_COMPILER=icx -G "Ninja". Executables landed in build/bin/.

### Steps to Reproduce

Step 1 — See the full op coverage map:
powershell.\build\bin\test-backend-ops.exe --show-coverage 2>&1 | Tee-Object coverage.txt
Step 2 — See which ops SYCL specifically supports vs missing:
powershell.\build\bin\test-backend-ops.exe support -b SYCL0 2>&1 | Tee-Object sycl_support.txt
Step 3 — Pick one ❌ op from the output and reproduce the missing op:
powershell# Example: try running a specific op test on SYCL
.\build\bin\test-backend-ops.exe test -b SYCL0 -o POOL_1D 2>&1
Step 4 — The "observed result" for a missing op will look like one of:
SKIP: op not supported on this backend
# or
FAIL: ...
# or the op simply won't run at all


### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- 
- **My findings:** [What you discovered during reproduction]
ABS isn't wired in; supports_op falls through to false; EXPM1 is an identical template
---

## Solution Approach

### Analysis

root cause is a missing implementation (not a bug) — explains the two conditions (supports_op + dispatch/kernel) that must both hold

### Proposed Solution

mirror EXPM1 end-to-end, swap math to fabs(x)

### Implementation Plan

Using UMPIRE framework (adapted):
Understand / Match / Plan (8 numbered steps) / Implement (your branch + the incremental one-case-at-a-time approach)

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
Step 0 — Infrastructure (once): empty abs.cl + CMake registration, 6 kernel handles, full ggml_cl_abs() dispatch, op-dispatch case, supports_op case
Step 1: kernel_abs_f16_4 → case 1 (done)
Step 2: kernel_abs_f16 scalar → case 2 (done)
Step 3: kernel_abs_f16_nc → case 3 (edits staged, awaiting rebuild)
Step 4: reuse _nc, widen gate → case 4
Step 5: kernel_abs_f32_4 → case 5
Step 6: kernel_abs_f32 scalar → case 6
Step 7: kernel_abs_f32_nc → case 7
Step 8: final gate type == F32 || F16 → case 8 (removes scaffolding)
Step 9 — Finalize: confirm all 8 green + no regressions, regenerate docs/ops.md, commit + PR

**Implement:** 
https://github.com/gezahegne/llama.cpp/tree/opencl-abs

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

--- Runnint test cases

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
-   - `ggml/src/ggml-opencl/kernels/abs.cl` (new, +113) — six ABS kernels: `f32`, `f32_4`, `f32_nc`, `f16`, `f16_4`, `f16_nc`
  - `ggml/src/ggml-opencl/ggml-opencl.cpp` (+127) — kernel handles, `// abs` loading block, `ggml_cl_abs()` dispatch, `GGML_UNARY_OP_ABS` op routing, `supports_op` case
  - `ggml/src/ggml-opencl/CMakeLists.txt` (+1) — register `abs` in `GGML_OPENCL_KERNELS`
  - `docs/ops.md` — ABS OpenCL cell `❌ → ✅`
  - `docs/ops/OpenCL.csv` — 8 ABS rows marked supported
- **Key commits:** [Links to important commits]
-   - [`9577551`](https://github.com/gezahegne/llama.cpp/commit/957755105d5cd05ca168b95b577e6eb1de9a69ca) — Test cases 1 & 2: f16 contiguous kernels + infra (dispatch, loading, supports_op)
  - [`0896789`](https://github.com/gezahegne/llama.cpp/commit/0896789c59ece76c86f9780f06c4417cfbac9021) — finish ABS: f32 + non-contiguous kernels (cases 3–8)
  - [`58fae0a`](https://github.com/gezahegne/llama.cpp/commit/58fae0a106df4ff5d19af2a7f3bde1f943be0fdc) — docs: mark ABS supported in `ops.md` / `OpenCL.csv`
- **Approach decisions:** [Why you chose certain approaches]
  - **Mirrored the existing `EXPM1` op** rather than inventing a new pattern — it's a structurally identical element-wise unary op already in the OpenCL backend, so reusing its layout keeps the code consistent with backend conventions; only the per-element math changes (`exp(x) - 1` → `fabs(x)`).
  - **Six kernel variants** matching `EXPM1`'s handling: vectorized `float4`/`half4` (`_4`) for contiguous tensors when `nelements % 4 == 0`, scalar otherwise, and strided (`_nc`) kernels for non-contiguous inputs — covering all `test-backend-ops` shapes/layouts.
  - **`fabs()` builtin** for both `float` and `half` (correct and exact for `|x|`).
  - **`supports_op` returns `F32 || F16`**, identical to `EXPM1`, so the scheduler only routes supported types to the backend.
  - **Validated incrementally** against the CPU reference via `test-backend-ops -o ABS` (8/8 cases) with no regressions in the full OpenCL suite.
---

## Pull Request

**PR Link:** [[GitHub PR URL when submitted]]
(https://github.com/ggml-org/llama.cpp/pull/25115)

**PR Description:** [Draft or final PR description - much of the content above can be adapted]
Adds support for the `ABS` unary operation (`y = |x|`) to the OpenCL backend.
`ABS` was supported on CPU, CUDA, Metal, SYCL, Vulkan, WebGPU and CANN, but not
on OpenCL — graphs containing an `ABS` node could not run that node on an OpenCL
device. This implements it for `F32` and `F16`, contiguous and non-contiguous
tensors.
The implementation mirrors the existing `EXPM1` op (same element-wise unary
structure); only the per-element math differs (`exp(x) - 1` → `fabs(x)`).
## Changes
- `ggml/src/ggml-opencl/kernels/abs.cl` (new): 6 kernels —
  `kernel_abs_f32`, `kernel_abs_f32_4`, `kernel_abs_f32_nc`,
  `kernel_abs_f16`, `kernel_abs_f16_4`, `kernel_abs_f16_nc`.
- `ggml/src/ggml-opencl/CMakeLists.txt`: register `abs` in `GGML_OPENCL_KERNELS`.
- `ggml/src/ggml-opencl/ggml-opencl.cpp`:
  - declare the 6 `cl_kernel kernel_abs_*` handles in the backend context,
  - load them in a new `// abs` program block,
  - add the `ggml_cl_abs()` dispatch function (contiguous fast path with `_4`
    vectorization when `nelements % 4 == 0`, otherwise scalar; `_nc` for
    non-contiguous inputs),
  - route `GGML_UNARY_OP_ABS` to `ggml_cl_abs`,
  - return `true` for `GGML_UNARY_OP_ABS` (F32/F16) in `supports_op`.
- `docs/ops.md`: regenerated — OpenCL `ABS` cell flips `❌ → ✅`.


**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]
- [7/8/2026]: Maybe also remove the ops .csv and .md from this PR - I think we can do a separate one to update ops support stats.
- [7/15/2026]: Removed the mentioned file and submitted updated code rebasing from main. 
- 

**Status:** [Awaiting review / Iterating / Approved / Merged]

---Iterating
[7/16/2026] My code got merged with the main. 

## Learnings & Reflections
Contributing to a large open-source project is as much about process as code. The actual ABS implementation was small; most of the effort went into environment setup, understanding existing patterns, clean git history, and following project rules. I learned to root-cause problems (the "CMake error" was really an antivirus issue) rather than fix symptoms, and to always establish a baseline before changing code.
### Technical Skills Gained

[What you learned technically]
Building llama.cpp/ggml from source with CMake + Ninja + a MinGW/GCC toolchain on Windows
The ggml OpenCL backend architecture: kernels (.cl), kernel loading, op dispatch, and supports_op
Writing OpenCL kernels (scalar + vectorized float4/half4 + strided non-contiguous)
Backend validation with test-backend-ops against a CPU reference
Git workflow for OSS: fork/upstream remotes, rebasing onto master, resolving conflicts, squashing, --force-with-lease

### Challenges Overcome

[What was hard and how you solved it]
Norton antivirus breaking the build three ways (TLS interception, blocked executables, cert errors) — diagnosed and worked around each
Getting the Intel Arc GPU to actually register in OpenCL (GGML_OPENCL_USE_ADRENO_KERNELS=OFF)
A meaningful rebase conflict requiring me to keep upstream's fix and my change
Cleaning stray files out of the PR to get a focused, reviewable diff

### What I'd Do Differently Next Time

[Reflection on your process]
Set up a clean, verified toolchain (and antivirus exclusions) before starting, not mid-task
Keep the branch tidy from the first commit — avoid committing helper/output files, so no cleanup/squash is needed later
Decide PR scope up front (code vs. auto-generated docs) instead of splitting after the fact
Read the project's CONTRIBUTING.md / AGENTS.md first, especially the AI-usage and contribution rules
---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
