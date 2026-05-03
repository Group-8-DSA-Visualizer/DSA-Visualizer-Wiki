# Shortcomings & Room for Improvement

Throughout the development process, we used a 'make it exist first, make it better later' philosophy. This of course
results in a few shortcomings/mistakes/places where the application can be improved.

This page catalogues known limitations and trade‑offs in the current design of the DSA Visualizer. While we would have
loved to implement these features, we unfortunately did not have the time to design, implement, and tests that these 
features would require.

We hope that this page can be useful to any future maintainers(or anyone seeking to gain a better understanding 
of our design philosophy).

---

## 1. Missing Pedagogical Features (Perháč & Šimonák)

### 1.1 Pseudocode Highlighting
**Current State:**  
No pseudocode panel is displayed alongside the visualization, and there is no automatic synchronization between the algorithm’s source and the visual steps.

**Partial Substitute:**  
Instructors can place pseudocode (or any textual explanation) inside the `long_description` field of a step dictionary. However, the formatting and line‑by‑line alignment are entirely manual.

**Desired Behavior:**  
- A pseudocode listing is shown beside the array.  
- The line that corresponds to the current operation is automatically highlighted as the visualization progresses.
- This feature should be independently toggleable per algorithm on the instructor's end, and universally toggleable for students wishing to study.

**Suggested Implementation Path:**  
Extend the step dictionary with an optional `pseudocode_line` integer. Provide a UI component that displays the algorithm’s source (or a separate pseudocode file) and highlights the indicated line.

### 1.2 Interactive Exercises
**Current State:**  
There is no system for posing questions to students and automatically verifying their actions against a model solution.

**Partial Substitute:**  
“Study Mode” hides future steps, allowing students to predict the next operation before revealing it. However, the system provides no feedback on whether the student’s prediction was correct.

**Desired Behavior:**  
A dedicated exercise mode where the system presents a data structure and asks the student to perform a sequence of operations (e.g., “highlight the pair that will be swapped next”). The system compares each action to a pre‑defined model solution and provides immediate feedback.

**Suggested Implementation Path:**  
Adopt a pattern similar to JSAV’s Exercise API, where an instructor defines a model solution function. The visualizer would then compare user‑generated steps to the expected ones.

## 2. Architectural Constraints

### 2.1 Full Array Copies per Step
**Current Approach:**  
Every algorithm step stores a complete, independent snapshot of the array (created via `.copy()`). For *N* elements and *S* steps, this requires O(N×S) memory.

**Strengths:**  
- Simplicity: the visualizer can render any step in isolation.  
- Reliability: no risk of state corruption when jumping to arbitrary steps.  
- Acceptable for the configured maximum array size (default 32).

**Drawbacks:**  
- Memory grows quickly if limits are raised (e.g., 1000 elements × 500 steps ≈ 500 MB).  
- Larger‑than‑necessary network transfer when the server pushes an update of the entire renderer to the client.

**Possible Improvement:**  
Use a diff-based approach. Store a base array once and record only the changes (patches) in subsequent steps. The visualizer would replay patches to reconstruct the current state. This reduces memory to O(N + P) where P is the total number of element changes, but makes the visualizer stateful (see next section).

### 2.2 Stateless Visualizer Rendering
**Current Approach:**  
`render_step()` clears the display area and draws the entire array from scratch. No “before” state is preserved between frames.

**Strengths:**  
- Random access (slider, “jump to step”) is instantaneous.  
- Code simplicity: the visualizer does not need to track a live data model.

**Drawbacks:**  
- No smooth animated transitions (e.g., morphing bars between heights).  
- Every frame redraws all elements, which is unnecessary for small changes.  

**Possible Improvement:**  
Use checkpoints (full snapshots) every *k* steps, with diffs in between. The visualizer would maintain a current array and apply diffs forward or replay from the last checkpoint. This balances memory, performance, and animation capabilities.

You may find that these changes would not be worth the added complexity.
### 2.3 Sandbox Security Model Maturity
**Current Approach:**  
The current security model combines AST static analysis, a restricted built‑ins whitelist, subprocess isolation, and resource limits (CPU, memory).

**Drawbacks:**  
- The built‑ins whitelist has been carefully curated but has not undergone a formal security audit. Subtle escape hatches may exist.  
- Error reporting is generic, making it hard to distinguish exactly what has taken place.

**Possible Improvement:**  
- Conduct a security review or use a hardened sandbox such as RestrictedPython or gVisor.  
- Implement finer‑grained error classification.

### 2.4 Manual Component Registration
**Current State:**  
To add a new visualization type (e.g., a tree renderer + visualizer + run config), a developer must:

1. Edit `PageCoordinator._register_components()`.
2. Add a factory function.
3. Possibly update `BUILTIN_DS_CLASSES`.

**Possible Improvement:**   
A fully dynamic plugin system where dropping a file into a `plugins/` directory auto‑discovers and registers the component.

The `ComponentRegistry` singleton and factory pattern are already in place; only the discovery mechanism needs to be extended.

### 2.5 Only the Array Renderer is Concrete
**Current Approach:**   
The architecture was designed to support `tree`, `graph`, `table`, and other step types, but currently only `ArrayRenderer` is fully implemented.

**Drawbacks:**  
The extensibility claims remain partially theoretical until at least one additional visualization type is built as a proof of concept.

**Possible Improvement:**  
Implement a `TreeRenderer` and accompanying `TreeVisualizer` to validate the extension points and provide a reusable template.

### 2.6 Lack of Automated UI Tests
**Current Coverage:**  
Unit tests verify algorithm correctness and sandbox enforcement. No other automated UI tests exist.

**Drawbacks:**  
As the application grows, regressions in playback controls, step rendering, or study mode may go undetected.

**Possible Improvement:**  
Add a suite of end‑to‑end tests that run in CI (e.g., GitHub Actions) to validate critical user journeys.

## Summary

| Category            | Limitation                     | Severity                  | Short‑Term Mitigation       |
|---------------------|--------------------------------|---------------------------|-----------------------------|
| Pedagogical Feature | No pseudocode highlighting     | Medium                    | Use `long_description`      |
| Pedagogical Feature | No interactive exercises       | Medium                    | Study Mode                  |
| Architectural       | Full array copies per step     | Low (for small N)         | Keep array limits low       |
| Architectural       | Stateless rendering            | Low (for educational use) | Small max array size        |
| Architectural       | Sandbox security maturity      | Medium                    | Restrict to trusted systems |
| Architectural       | Manual component registration  | Low                       | Follow documented steps     |
| Architectural       | Only `array` renderer concrete | Medium                    | Build a TreeRenderer        |
| Architectural       | Opaque error reporting         | Low                       | DEBUG mode, monitor logs    |
| Architectural       | No UI tests                    | Medium                    | Manual testing              |

We hope that this documentation of limitations is useful for any maintainer who wishes to enhance the system.