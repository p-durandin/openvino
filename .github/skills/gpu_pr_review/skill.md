---
name: openvino-gpu-pr-review
description: >
  Coordinates evidence-based code review of OpenVINO Intel GPU pull requests.
  Use for changes affecting the Intel GPU plugin, GPU kernels, kernel selector,
  graph optimizations, GPU runtime integration, memory management, GPU tests,
  performance, precision support, or Intel GPU platform enablement.
---

# OpenVINO Intel GPU PR Review Coordinator

## Purpose

Review OpenVINO Intel GPU pull requests for:

1. Functional correctness
2. Performance regressions and inefficient execution
3. Intel GPU platform and runtime compatibility
4. Architecture and code clarity
5. Implementation and validation completeness

Functional correctness and performance are the primary review dimensions.

Architecture, maintainability, test quality, and code clarity are important
secondary dimensions. They become blocking only when they create a concrete
correctness, performance, compatibility, or significant maintenance risk.

This review is advisory. It does not approve a pull request and does not
replace review by the responsible OpenVINO GPU code owners.

## Review principles

Follow these principles throughout the review:

- Base every defect finding on changed code and a concrete execution condition.
- Distinguish confirmed defects from risks, validation gaps, and questions.
- Do not assume that successful compilation proves functional correctness.
- Do not assume that passing one test configuration proves platform coverage.
- Do not claim a performance improvement or regression without explaining
  the expected mechanism or citing provided measurements.
- Do not treat missing evidence as proof that a defect exists.
- Prefer a small number of actionable findings over many speculative comments.
- Review the whole affected execution path, not only isolated changed lines.
- Consider interactions across files when the implementation spans layers.
- Do not recommend approval or merge.
- Escalate architectural intent and product trade-offs to human GPU owners.

## Review workflow

Perform the review in the following order.

### Step 1: Understand the PR intent

Determine:

- What behavior is being added, removed, fixed, or optimized?
- Which user-visible or internal behavior is expected to change?
- What is the stated reason for the change?
- Is the PR primarily a bug fix, optimization, refactoring, enablement,
  cleanup, platform workaround, or test-infrastructure change?
- What evidence does the PR provide for correctness and performance?

Do not infer intent solely from the implementation when the PR description,
tests, or linked issue provide more direct information.

If intent remains unclear, identify the ambiguity as a question rather than
inventing a requirement.

### Step 2: Classify the affected components

Classify the PR into every applicable area:

- Graph transformation or graph optimization
- GPU program building
- Primitive implementation
- Kernel selector
- OpenCL kernel
- oneDNN integration
- OpenCL runtime
- Level Zero runtime
- Memory allocation or memory dependency handling
- USM or buffer management
- Model or GPU program caching
- Dynamic shape or dynamic batch handling
- Tensor layout, reorder, padding, pitch, or offset handling
- Precision conversion or quantization
- LLM, SDPA, Paged Attention, or KV-cache execution
- Device capability detection
- Platform enablement or platform-specific workaround
- Build system or configuration
- Unit, functional, validation, or performance tests
- Documentation or diagnostics

Use this classification to select the applicable review domains.

### Step 3: Determine affected execution scope

Identify the scope supported by evidence in the PR:

- Intel GPU generations or capability groups
- Integrated or discrete GPU
- Linux or Windows
- OpenCL or Level Zero runtime
- If relevant, both runtimes
- Static or dynamic shapes
- Single-stream or multi-stream execution
- Single-request or multi-request execution
- Relevant tensor layouts
- Relevant precisions
- Relevant batch sizes
- Relevant operation, model, or workload families
- Model compilation, first inference, steady-state inference, or cache import

Do not claim that all platforms are affected when the diff only establishes a
narrower scope.

When the exact platform scope cannot be established, report it as unknown or
as a validation gap.

### Step 4: Apply the relevant review domains

Always perform the functional-correctness and completeness reviews.

Perform the performance review when the PR:

- claims or implies a performance improvement;
- modifies an inference or compilation hot path;
- changes kernels, dispatch, synchronization, memory, fusion, or layouts;
- changes primitive or kernel selection;
- adds a platform-specific implementation;
- can affect memory use, compilation time, or model-loading time.

Perform the platform review when the PR:

- changes device or feature detection;
- introduces generation-specific behavior;
- changes OpenCL or Level Zero paths;
- uses a GPU extension or specialized hardware capability;
- changes Windows- or Linux-specific behavior;
- changes behavior for integrated or discrete GPUs.

Perform the architecture review when the PR:

- introduces a new abstraction or interface;
- moves responsibility between layers;
- duplicates logic across runtimes, platforms, or implementations;
- changes ownership, lifetime, caching, or synchronization;
- introduces a workaround into generic code;
- establishes a pattern expected to be reused.

If dedicated supporting skills are available, apply their instructions:

- `openvino-gpu-functional-correctness`
- `openvino-gpu-performance`
- `openvino-gpu-platform-coverage`
- `openvino-gpu-architecture`
- `openvino-gpu-test-completeness`

If a supporting skill is unavailable, perform the corresponding checks using
the minimum review requirements in this coordinator.

## Minimum functional-correctness review

Check changed behavior for:

- Static, dynamic, partially known, scalar, empty, and boundary shapes
- Rank-specific assumptions
- Tensor layout, pitch, offset, padding, and broadcasting
- Index and dispatch calculations
- Integer overflow and unsafe narrowing
- Out-of-bounds reads and writes
- Precision conversion, rounding, saturation, and accumulation
- Signed and unsigned arithmetic
- Quantization parameters, zero points, and scales
- In-place execution and memory aliasing
- Buffer size and lifetime
- Synchronization, barriers, events, and dependencies
- Multi-stream and multi-request safety
- Mutation of shared or cached state
- Kernel argument consistency
- JIT constant consistency
- Dispatch geometry and tail processing
- Kernel selector conditions
- Device-capability guards
- Unsupported combinations
- Fallback behavior
- OpenCL and Level Zero semantic consistency
- Error propagation and recovery

For each suspected defect, identify the exact condition under which it occurs.

Do not report generic possibilities such as "this may cause a race" without
showing the shared state, concurrent access, and missing synchronization.

## Minimum performance review

Check whether the change can introduce:

- Additional kernel launches
- Additional host-device or device-host transfer
- Additional reorder, copy, conversion, or memory pass
- Additional synchronization or blocking wait
- Allocation or deallocation in a hot path
- Increased temporary-memory size or lifetime
- Loss of kernel or graph fusion
- Reduced parallelism, occupancy, vectorization, or subgroup efficiency
- Divergent control flow
- Uncoalesced or redundant memory access
- Repeated calculation that could be cached
- Repeated initialization that could be moved out of a loop
- More expensive primitive or kernel selection
- Increased compilation or model-load time
- Increased first-inference latency
- Cache invalidation or reduced cache reuse
- Silent fallback to a slower implementation
- Improvement on one GPU generation at the expense of another

Separate these measurement scopes:

1. Kernel execution
2. End-to-end inference latency or throughput
3. Compilation and model-loading time
4. First inference
5. Cache import or cache export
6. Host and device memory consumption

Do not equate a kernel-level speedup with an end-to-end improvement.

When reporting a performance concern, explain the expected mechanism. For
example:

- an added wait serializes otherwise asynchronous work;
- an added reorder creates another full tensor memory pass;
- a changed selection condition moves affected shapes to a generic kernel;
- a larger temporary buffer increases device-memory pressure;
- dispatch dimensions leave GPU execution resources underutilized.

If the mechanism is plausible but impact is not established, report a
validation gap rather than a confirmed regression.

## Minimum architecture and clarity review

Check:

- Whether responsibility is placed in the correct GPU plugin layer
- Whether generic and GPU-generation-specific logic are separated
- Whether generic and runtime-specific logic are separated
- Whether existing abstractions and helpers are reused appropriately
- Whether duplication can cause implementations to diverge
- Whether ownership and lifetime are explicit
- Whether interfaces preserve established invariants
- Whether failure handling is consistent with surrounding code
- Whether names describe intent rather than implementation accidents
- Whether complex control flow can be simplified safely
- Whether comments explain hardware or runtime constraints
- Whether comments explain why a workaround is required
- Whether temporary workarounds have a removal condition or tracking reference
- Whether the design supports likely extensions without speculative abstraction

Do not request broad refactoring unless the current structure creates a
concrete correctness risk, performance risk, or substantial maintenance risk.

When the decision depends on future plans or undocumented architectural intent,
use a question for the responsible GPU owner.

## Minimum completeness review

Check whether the PR includes, where applicable:

- All required implementation branches
- OpenCL and Level Zero handling
- Optimized path and fallback path
- Supported and unsupported capability handling
- Static and dynamic shape handling
- Relevant precision handling
- Integrated and discrete GPU considerations
- Linux and Windows considerations
- Positive functional tests
- Negative tests
- Boundary and tail-condition tests
- A regression test that fails without the fix
- Performance measurements for performance claims
- Appropriate model, operation, shape, and batch coverage
- Memory measurements when memory behavior changes
- Compilation-time measurements when compilation changes
- Documentation for changed behavior
- Comments for non-obvious hardware constraints
- Removal of debug output and temporary code
- Handling or removal of TODOs introduced by the change

A missing test is not automatically a functional defect.

Report it as a validation gap unless the changed code itself demonstrates a
failure.

## Evidence rules

Every finding must contain enough evidence for the author to evaluate it
without guessing.

Acceptable evidence includes:

- Changed control flow
- Changed data flow
- Changed indexing or dispatch calculation
- A violated local or component invariant
- An unguarded platform-specific operation
- A missing case in a changed selection condition
- Inconsistency between implementation and an existing test
- Inconsistency between related runtime paths
- A benchmark or test result provided with the PR
- A new code path for which no safe fallback exists

Unacceptable evidence includes:

- Complexity alone
- Personal coding preference
- An unsupported assumption about future usage
- An unquantified claim that code is slow
- A generic assertion that more tests are needed
- Restating the code without identifying a problem
- Treating an unchanged pre-existing issue as introduced by this PR

If a pre-existing issue becomes more exposed or severe because of the PR,
explain the new contribution of the PR.

## Finding types

Use one primary category per finding:

- `FUNCTIONAL`
- `PERFORMANCE`
- `MEMORY`
- `CONCURRENCY`
- `PLATFORM`
- `RUNTIME`
- `NUMERICAL`
- `ARCHITECTURE`
- `COMPLETENESS`
- `TESTING`
- `VALIDATION GAP`
- `MAINTAINABILITY`

Use the category that describes the primary impact.

Do not create multiple findings for the same root cause merely because it has
several consequences. Explain secondary consequences in the Impact section.

## Severity rules

Severity expresses potential impact. Confidence expresses strength of
evidence. Do not combine the two concepts.

### CRITICAL

Use `CRITICAL` for a concrete issue that can cause:

- Out-of-bounds writes or unsafe memory corruption
- Use-after-free
- Corruption caused by an established race
- Deterministic broad crash on a supported configuration
- Broad silent data corruption
- Security-relevant unsafe execution
- Removal or bypass of a required safety guard
- Severe failure of a primary execution path with direct evidence

A Critical finding must identify:

- the concrete execution path;
- the triggering condition;
- the unsafe or incorrect operation;
- the affected scope.

Do not use Critical because code is complex, risky, or insufficiently tested.

Critical findings are blocking.

### HIGH

Use `HIGH` for a probable or established issue that can cause:

- Incorrect output on a supported configuration
- Crash on a supported but limited configuration
- Missing synchronization or dependency handling
- Overflow affecting allocation, indexing, or dispatch
- Broken or absent fallback for a supported configuration
- Incorrect device-capability handling
- Material OpenCL and Level Zero semantic inconsistency
- Major performance regression in a hot path
- Serialization or transfer added to a hot inference path
- Severe platform compatibility failure
- A semantic change introduced by an optimization without adequate protection

High findings are blocking unless evidence disproves the concern or a
responsible human owner explicitly accepts the behavior.

### MEDIUM

Use `MEDIUM` for:

- A credible localized edge-case risk
- An important validation or coverage gap
- An unconfirmed but technically grounded performance concern
- Missing handling for a non-primary scenario
- Reliance on an undocumented invariant
- Incomplete error handling outside the main execution path
- Missing boundary or negative testing
- Duplication likely to cause implementation divergence
- A meaningful architecture or maintenance problem

A Medium finding should normally be addressed by code, evidence, or a
documented design explanation.

### LOW

Use `LOW` for a non-blocking improvement such as:

- Unclear naming in complex GPU logic
- Missing explanation of a hardware or runtime constraint
- Unnecessarily complicated local control flow
- Minor meaningful duplication
- Weak diagnostics
- Localized test readability
- A maintainability issue with no current correctness impact

Do not use Low for formatting, automated style rules, or personal preference.

### QUESTION

Use `QUESTION` when:

- Architectural intent is unavailable;
- the behavior depends on product requirements not present in the PR;
- the correct platform scope cannot be established;
- an apparent inconsistency may be intentional;
- GPU owner knowledge is required.

A question must explain why the answer matters to correctness, performance,
compatibility, or maintainability.

Do not disguise an unsupported allegation as a question.

## Confidence rules

Assign one confidence level to every finding.

### High confidence

Use when the conclusion follows directly from:

- changed control or data flow;
- an explicit triggering condition;
- a clear language, API, kernel, or component invariant;
- a directly inconsistent test or runtime path.

### Medium confidence

Use when:

- the failure mechanism is technically credible;
- some caller, platform, runtime, or configuration context is missing;
- validation is required to establish actual impact.

### Low confidence

Use only for a meaningful review question requiring context that is not
available in the diff.

Do not report a blocking Critical or High finding with Low confidence.
Convert it to a Medium validation gap or a Question.

## Finding format

Use this format for each finding:

```text
[SEVERITY][CATEGORY] Short actionable title

Location:
path/to/file.cpp:line

Evidence:
Describe the changed code and the exact condition that triggers the concern.

Impact:
Describe the correctness, performance, memory, compatibility, or maintenance
consequence.

Affected scope:
List the known platform, runtime, OS, precision, layout, shape, batch, and
workload conditions. Mark unknown dimensions as unknown.

Recommended action:
Propose a concrete code correction or narrowly defined validation action.

Required evidence:
State the test, benchmark, trace, comparison, or invariant needed to resolve
the finding.

Confidence:
High, Medium, or Low.
