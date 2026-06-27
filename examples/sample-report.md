# Boilerplate Analysis — 2026-06-28

## Summary

| Metric | Value |
|---|---|
| Files scanned | 47 |
| Total findings | 23 |
| Severity score | 87 |
| Architectural triggers | god_object, multiple_sources_of_truth |
| Verdict | REDESIGN |
| Confidence coverage | high: 12 / medium: 8 / low: 3 |

## Findings by Severity

| Tier | Count |
|---|---|
| critical | 4 |
| major | 5 |
| minor | 6 |
| advisory | 6 |
| suggestion | 2 |

## Findings (Detail)

### [blessed-single-source-of-truth] — Multiple SSOT Violation

- **Severity:** critical
- **Confidence:** high
- **Location:** `Source/Engine.cpp:142`
- **Tags:** [architecture, state]
- **Depends on:** [blessed-explicit-encapsulation]
- **Evidence:**
  ```cpp
  // MasterGain owned by both Engine and UIComponent
  float Engine::masterGain = 0.5f;
  float UIComponent::gainDisplay = 0.5f;  // mirrored state
  ```
- **Reason:** Master gain state is owned by two components. Updates to one do not propagate to the other.
- **Expected:** Single owner; UIComponent reads from Engine via getter or listener.
- **Suggested fix:** Remove `UIComponent::gainDisplay`; subscribe to Engine::masterGainChanged.
- **Source refs:** contracts/MANIFESTO.md (BLESSED Single Source of Truth)

### [lang-no-early-return] — Early Return Detected

- **Severity:** critical
- **Confidence:** high
- **Location:** `Source/DSP.cpp:88`
- **Tags:** [control-flow, readability, jreng-critical]
- **Depends on:** [blessed-explicit-encapsulation]
- **Evidence:**
  ```cpp
  float processSample(float input) {
    if (input < 0.0f) return 0.0f;
    return input * gain;
  }
  ```
- **Reason:** Early return violates positive nesting (JRENG §547 CRITICAL).
- **Expected:**
  ```cpp
  float processSample(float input) {
    const auto safeInput = (input < 0.0f) ? 0.0f : input;
    return safeInput * gain;
  }
  ```
- **Suggested fix:**
  ```diff
  - if (input < 0.0f) return 0.0f;
  - return input * gain;
  + const auto safeInput = (input < 0.0f) ? 0.0f : input;
  + return safeInput * gain;
  ```
- **Source refs:** contracts/JRENG-CODING-STANDARD.md:547

### [names-no-type-encoding] — Type Encoding in Identifier

- **Severity:** major
- **Confidence:** medium
- **Location:** `Source/Filter.h:31`
- **Tags:** [naming, maintainability]
- **Depends on:** []
- **Evidence:**
  ```cpp
  class IFilterState { /* ... */ };
  ```
- **Reason:** Hungarian-style `I` prefix violates NAMES.md.
- **Expected:** `class FilterState { /* ... */ };`
- **Source refs:** contracts/NAMES.md

### [lang-no-raw-delete] — Raw Delete Usage

- **Severity:** critical
- **Confidence:** high
- **Location:** `Source/WavetableSynth.cpp:215`
- **Tags:** [memory, safety, jreng-critical]
- **Depends on:** [blessed-explicit-encapsulation]
- **Evidence:**
  ```cpp
  delete wavetableBuffer;
  wavetableBuffer = nullptr;
  ```
- **Reason:** Raw `delete` bypasses RAII and can cause leaks on exceptions.
- **Expected:** Use `std::unique_ptr` or `std::vector<float>`.
- **Source refs:** contracts/JRENG-CODING-STANDARD.md (CRITICAL RULES)

### [blessed-lean] — God Object Detected

- **Severity:** major
- **Confidence:** high
- **Location:** `Source/Engine.cpp:1-400`
- **Tags:** [architecture, cohesion]
- **Depends on:** [blessed-explicit-encapsulation, blessed-single-source-of-truth]
- **Evidence:**
  ```cpp
  class Engine {
    // 18 data members, 23 methods
    // Handles: DSP, state, parameter management, UI sync, file I/O, metering
  };
  ```
- **Reason:** Single class responsible for too many concerns. Violates BLESSED Lean principle.
- **Expected:** Split into focused components: AudioEngine, ParameterManager, PresetManager, MeterNode.
- **Source refs:** contracts/MANIFESTO.md (BLESSED Lean)

### [lang-positive-nesting] — Deep Nesting Detected

- **Severity:** critical
- **Confidence:** medium
- **Location:** `Source/PresetManager.cpp:55`
- **Tags:** [control-flow, readability, jreng-critical]
- **Depends on:** [lang-no-early-return]
- **Evidence:**
  ```cpp
  if (preset != nullptr) {
    if (preset->isValid()) {
      if (preset->load()) {
        if (notifyHost) { sendChange(); }
      }
    }
  }
  ```
- **Reason:** Four levels of nesting obscures control flow.
- **Expected:** Extract inner logic; use early-excluded guard paths.
- **Source refs:** contracts/JRENG-CODING-STANDARD.md §Positive Nesting

### [names-no-getter-prefix] — Getter Prefix Violation

- **Severity:** advisory
- **Confidence:** medium
- **Location:** `Source/Parameter.h:44`
- **Tags:** [naming, readability]
- **Depends on:** []
- **Evidence:**
  ```cpp
  float getMasterVolume() const { return masterVolume; }
  ```
- **Reason:** `get` prefix adds noise per NAMES.md.
- **Expected:** `float masterVolume() const { return masterVolume; }`
- **Source refs:** contracts/NAMES.md

### [layout-includes-order] — Includes Not Alphabetical

- **Severity:** minor
- **Confidence:** medium
- **Location:** `Source/Panner.cpp:3`
- **Tags:** [layout, style]
- **Depends on:** []
- **Evidence:**
  ```cpp
  #include <iostream>
  #include "Config.h"
  #include <vector>
  #include "Wavetable.h"
  ```
- **Reason:** Includes should be alphabetically ordered within groups.
- **Expected:** Group by angle-bracket then quoted; alphabetical within group.
- **Source refs:** contracts/JRENG-CODING-STANDARD.md (Layout section)

### [names-cognitive-load] — High Cognitive Load Name

- **Severity:** advisory
- **Confidence:** low
- **Location:** `Source/Utilities.h:12`
- **Tags:** [naming, readability]
- **Depends on:** []
- **Evidence:**
  ```cpp
  float calculateNormalizedRMSWithPeakLimiting(float input, float threshold) { ... }
  ```
- **Reason:** Function name is a full sentence; burdens working memory.
- **Expected:** `float rmsLimited(float input, float threshold) { ... }`
- **Source refs:** contracts/NAMES.md

### [lang-fail-fast] — Missing Fail-Fast on Invalid State

- **Severity:** critical
- **Confidence:** high
- **Location:** `Source/DelayLine.cpp:60`
- **Tags:** [correctness, invariants]
- **Depends on:** [blessed-explicit-encapsulation]
- **Evidence:**
  ```cpp
  void DelayLine::setDelayMs(float ms) {
    delayMs = ms;
    // No validation: negative delay silently accepted
  }
  ```
- **Reason:** Invalid input is silently absorbed instead of asserting.
- **Expected:** `jassert(ms >= 0.0f);` or return error code.
- **Source refs:** contracts/JRENG-CODING-STANDARD.md (Fail-Fast rule)

### [blessed-deterministic] — Non-Deterministic Behavior

- **Severity:** major
- **Confidence:** high
- **Location:** `Source/ReverbProcessor.cpp:78`
- **Tags:** [architecture, determinism]
- **Depends on:** [blessed-stateless]
- **Evidence:**
  ```cpp
  float rand = (float)std::rand() / RAND_MAX;
  output = input * rand * wetness;
  ```
- **Reason:** std::rand is seeded per-process; output differs across runs. Violates BLESSED Deterministic.
- **Expected:** Use deterministic noise (LFSR or pre-seeded generator) or expose seed as parameter.
- **Source refs:** contracts/MANIFESTO.md (BLESSED Deterministic)

### [names-no-helper-suffix] — Helper Suffix Violation

- **Severity:** advisory
- **Confidence:** low
- **Location:** `Source/ProcessingHelper.h:22`
- **Tags:** [naming]
- **Depends on:** []
- **Evidence:**
  ```cpp
  float helper_calculateFeedback() { ... }
  ```
- **Reason:** `_helper` suffix is redundant noise.
- **Expected:** `float calculateFeedback() { ... }`
- **Source refs:** contracts/NAMES.md

### [layout-line-length] — Line Exceeds 120 Characters

- **Severity:** minor
- **Confidence:** medium
- **Location:** `Source/GUI/LookAndFeel.cpp:12`
- **Tags:** [layout, readability]
- **Depends on:** []
- **Evidence:**
  ```cpp
  ColourGradient verticalGradient(Colours::black, 0.0f, 0.0f, Colours::white, 0.0f, 300.0f, false);
  ```
- **Reason:** Line longer than 120 characters impedes readability.
- **Expected:** Break into multi-line with member initializer style.
- **Source refs:** contracts/JRENG-CODING-STANDARD.md (Layout section)

### [lang-use-at-not-subscript] — At-Subscript Preferred for Audio

- **Severity:** major
- **Confidence:** high
- **Location:** `Source/Oscillator.cpp:94`
- **Tags:** [safety, audio]
- **Depends on:** []
- **Evidence:**
  ```cpp
  float sample = buffer[i];
  ```
- **Reason:** Subscript access skips bounds check; at() throws on out-of-bounds.
- **Expected:** `float sample = buffer.getReference(n).get();` or `.at()` for throwing access.
- **Source refs:** contracts/JRENG-CODING-STANDARD.md

### [names-verb-noun-functions] — Function Naming Convention

- **Severity:** advisory
- **Confidence:** medium
- **Location:** `Source/Mixer.cpp:33`
- **Tags:** [naming]
- **Depends on:** []
- **Evidence:**
  ```cpp
  void volume() { setVolume(currentVolume); }
  ```
- **Reason:** Verb-noun pair `volume()` reads like a noun; `setVolume` is clearer.
- **Expected:** `void setVolume(float v) { ... }`
- **Source refs:** contracts/NAMES.md

### [layout-brace-style] — Brace Style Inconsistency

- **Severity:** minor
- **Confidence:** medium
- **Location:** `Source/FilterNode.cpp:44`
- **Tags:** [layout, style]
- **Depends on:** []
- **Evidence:**
  ```cpp
  if (enabled) {
    process(buffer);
  } else {
    buffer.clear();
  }
  ```
- **Reason:** K&R style used here; other files use Allman. Inconsistent within project.
- **Expected:** Pick one style; apply project-wide. K&R preferred per JRENG standard.
- **Source refs:** contracts/JRENG-CODING-STANDARD.md (Layout section)

### [blessed-encapsulation] — Public Member Access

- **Severity:** major
- **Confidence:** high
- **Location:** `Source/Engine.h:67`
- **Tags:** [architecture, encapsulation]
- **Depends on:** [blessed-explicit-encapsulation]
- **Evidence:**
  ```cpp
  public:
    float sampleRate;
    int blockSize;
  ```
- **Reason:** Public struct members expose internal state. Any caller can mutate without validation.
- **Expected:** Make private; provide getters/setters with validation.
- **Source refs:** contracts/MANIFESTO.md (BLESSED Encapsulation)

### [names-domain-terms] — Non-Domain Term Used

- **Severity:** advisory
- **Confidence:** low
- **Location:** `Source/Panner.cpp:18`
- **Tags:** [naming, domain]
- **Depends on:** []
- **Evidence:**
  ```cpp
  float balance = 0.0f;  // panner uses "pan" not "balance"
  ```
- **Reason:** "balance" is a mixing console term; panner domain uses "pan".
- **Expected:** `float pan = 0.0f;`
- **Source refs:** contracts/NAMES.md

### [lang-no-magic-numbers] — Magic Number in Code

- **Severity:** minor
- **Confidence:** high
- **Location:** `Source/Compressor.cpp:55`
- **Tags:** [readability, maintainability]
- **Depends on:** []
- **Evidence:**
  ```cpp
  if (threshold > 0.779) {  // what is 0.779?
    ratio = 4.0;
  }
  ```
- **Reason:** Magic number 0.779 is unexplained.
- **Expected:** Define `constexpr float defaultThreshold = 0.779f;`
- **Source refs:** contracts/JRENG-CODING-STANDARD.md

### [blessed-bound] — Unbounded Resource

- **Severity:** major
- **Confidence:** high
- **Location:** `Source/WavetableLoader.cpp:44`
- **Tags:** [architecture, memory]
- **Depends on:** [blessed-lean]
- **Evidence:**
  ```cpp
  std::vector<float> loadWavetables(const String& path) {
    // Loads ALL .wav files in directory into memory
    // No pagination, no limit
  }
  ```
- **Reason:** Unbounded loading can exhaust memory with large wavetable collections.
- **Expected:** Add limit parameter; paginate; or lazy-load on demand.
- **Source refs:** contracts/MANIFESTO.md (BLESSED Bound)

### [names-no-comments-needed] — Comment Describes What Code Does

- **Severity:** advisory
- **Confidence:** medium
- **Location:** `Source/StateManager.cpp:28`
- **Tags:** [naming, documentation]
- **Depends on:** []
- **Evidence:**
  ```cpp
  // Set the volume to the given value
  void setVolume(float vol) { volume = vol; }
  ```
- **Reason:** Comment restates what the well-named function does.
- **Expected:** Remove comment; name already conveys intent.
- **Source refs:** contracts/NAMES.md

### [lang-no-anonymous-namespace] — Anonymous Namespace at File Scope

- **Severity:** major
- **Confidence:** high
- **Location:** `Source/Utilities.cpp:1`
- **Tags:** [linkage, language]
- **Depends on:** []
- **Evidence:**
  ```cpp
  namespace {
    void helper() { ... }
  }
  ```
- **Reason:** Anonymous namespace pollution can cause ODR violations across translation units.
- **Expected:** Use named namespace or `static` for internal linkage.
- **Source refs:** contracts/JRENG-CODING-STANDARD.md

### [blessed-stateless] — Stateful Component Should Be Stateless

- **Severity:** critical
- **Confidence:** medium
- **Location:** `Source/LFOGen.cpp:10`
- **Tags:** [architecture, state]
- **Depends on:** [blessed-deterministic, blessed-single-source-of-truth]
- **Evidence:**
  ```cpp
  class LFOGen {
    float phase = 0.0f;
    float nextSample() { phase += frequency / sampleRate; return std::sin(phase * twoPi); }
  };
  ```
- **Reason:** LFO maintains mutable phase state that accumulates differently per instance.
- **Expected:** Phase should be owned by the caller; LFO should be a pure function.
- **Source refs:** contracts/MANIFESTO.md (BLESSED Stateless)

## Architectural Triggers

### god_object
- `Source/Engine.cpp` — Engine class has 18 members, 23 methods; responsible for DSP, state, parameter management, UI sync, file I/O, and metering.

### multiple_sources_of_truth
- Master gain: Engine::masterGain + UIComponent::gainDisplay (see findings)
- Current preset: PresetManager::current + UIComponent::presetLabel

## Adoption Suggestions

No adoption triggered (architectural violations are internal; no external boilerplate resolves god_object or multiple_sources_of_truth).

## Baseline Delta

| Metric | Last audit (latest.json) | Today | Delta |
|---|---|---|---|
| Severity score | 52 | 87 | +35 |
| Critical | 2 | 4 | +2 |
| Major | 3 | 5 | +2 |
| Architectural triggers | (none) | god_object, multiple_sources_of_truth | +2 |

## ROI Estimate

| Verdict | Estimated remediation effort |
|---|---|
| REDESIGN | > 16 hours, architectural rewrite |

---

*Generated by Boilerplater compliance_orchestrator v1.0. Report is read-only; no source code was modified.*
