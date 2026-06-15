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

---

## Understanding the Issue

### Problem Description

What's missing is that GGML_OP_EXPM1 has no SYCL kernel registered. 

### Expected Behavior

SYCL dispatch for expm1 needs to be added in the SYCL backend. All 8 test cases should run and pass instead of being skipped. Concretely:

Each EXPM1 variant should execute a SYCL kernel that computes exp(x) - 1 element-wise on the GPU
Results should be compared against the CPU reference output within a tolerance threshold
The report should show 8/8 tests passed instead of 0/0
"not supported" should never appear — the backend should claim the op and execute it
Right now the SYCL backend is simply not registering a handler for GGML_OP_EXPM1, so the test harness sees it as unsupported and skips it rather than running it.

### Current Behavior

The SYCL backend reports not supported for every EXPM1 test case, so all 8 are skipped — 0 tests run, 0 tested, 0 verified. The backend still exits with OK because skipped ops aren't counted as failures.

### Affected Components

ggml/src/ggml-sycl/element_wise.cpp	=> Add op_expm1 kernel, ggml_sycl_op_expm1 dispatch, and ggml_sycl_expm1 wrapper
ggml/src/ggml-sycl/element_wise.hpp	=> Declare ggml_sycl_expm1()
ggml/src/ggml-sycl/ggml-sycl.cpp	=> Add GGML_UNARY_OP_EXPM1 case in the unary dispatch switch (~line 4557)

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
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

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

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
