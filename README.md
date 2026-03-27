# Resonant Solver

**Correctness = perceptual stability**

Resonant Solver makes mathematical optimization audible. Every variable is an oscillator. Every error is noise. Every gradient is instability. When the computation converges, you hear it — a pure, stable tone emerges from chaos.

This is not sonification. This is not data-to-music mapping. This is the optimization itself, made perceivable through sound. Every acoustic parameter is a mathematical function of the solver state. Nothing is aesthetic. Nothing is hardcoded.

**You can detect convergence with your eyes closed.**

---

## The Problem

Visual debugging has a bottleneck: your eyes scan sequentially, miss subtle patterns, and require active attention. Meanwhile, human hearing:

- Detects **0.02% frequency errors** (0.1 Hz at A440)
- Processes **all frequencies simultaneously** — not one at a time
- Recognizes dissonance **in under 100ms** — before conscious thought
- Has been detecting anomalies for **300,000 years** of evolution

Resonant Solver exploits this: the ear becomes a precision instrument for detecting whether a computation has found its answer.

---

## The Goal

Build a system where **mathematical truth has a sound**, and that sound is derived purely from the mathematics — not designed, not tuned, not mapped arbitrarily.

The long-term vision:

1. **Any optimization problem** can be plugged in — define E(x) and the sound is automatic
2. **Any domain** can use this — code quality, data anomalies, proof verification, LLM hallucination detection
3. **The sound is the proof** — if it sounds stable, the math guarantees convergence
4. **New training signal for AI** — spectrograms of correct vs incorrect reasoning are a novel feature space

---

## How It Works — The 4 Layers

Every problem reduces to: **minimize E(x₁, x₂, ..., xₙ) → 0**

### Layer 1: Variable → Frequency

```
f(xᵢ) = 200 + 20xᵢ   Hz
```

Each variable is a sine wave oscillator. Its value determines its pitch. As gradient descent adjusts x, you hear the pitch shift in real time. This is a linear, invertible map — you can reconstruct the variable value from the frequency.

### Layer 2: Error → Amplitude + Noise

```
Amplitude = exp(−E(x))      // High error = silent, low error = loud
Noise     = min(1, E(x)/10) // High error = static, low error = clean
```

| E(x) | Amplitude | Noise | You Hear |
|------|-----------|-------|----------|
| 100 | ~0 | 1.0 | Pure static |
| 1 | 0.37 | 0.1 | Tone in noise |
| 0.001 | 0.999 | ~0 | Pure tone |

`exp(-x)` is not a design choice. It is the natural function that maps [0, ∞) → (0, 1] with the property that zero error = maximum signal. The math guarantees: **E → 0 implies noise vanishes and amplitude maximizes.**

### Layer 3: Gradient → Vibrato

```
vibrato_i = min(50, |∂E/∂xᵢ| × 8)   Hz modulation depth
```

The gradient measures how far each variable is from where it needs to be. Large gradient = the oscillator's frequency **shakes** (vibrato via 5 Hz LFO). At the minimum, ∇E = 0, and the tone is rock-steady.

### Layer 4: Multi-Variable → Interference

Multiple variables = multiple oscillators. When constraints are violated, frequencies differ and you hear **beating** — an audible pulsation. When all variables converge, the oscillators lock into consonance. Voices are stereo-panned so you can localize which variable is still moving.

---

## Optimization Engine

**Gradient descent with momentum + gradient clipping:**

```
g = ∇E(x)                            // Gradient (analytical or numerical)
if ‖g‖ > 10: g = g × (10 / ‖g‖)     // Clip to prevent explosion
v = 0.9v − lr × g                    // Momentum accumulation
x = x + v                            // Parameter update
```

- **Learning rate**: 0.01 default, 0.001 for steep problems (Rosenbrock)
- **Momentum**: 0.9 — accelerates through flat regions, dampens oscillation
- **Gradient clipping**: max norm 10 — essential for functions with 100x+ gradient scaling
- **Numerical gradient**: central difference `(E(x+h) − E(x−h)) / 2h` when analytical form unavailable
- **Convergence**: `E(x) < 0.001` AND `‖∇E‖ < 0.01`

---

## Audio Architecture

Built on the **Web Audio API**. Real-time, zero-latency, runs entirely in the browser.

```
Per voice (one per variable):
  OscillatorNode (sine) ──→ GainNode (amp from error) ──→ StereoPannerNode ──→ AnalyserNode ──→ Master
       ↑ freq modulated by:
  LFO (5Hz sine) ──→ GainNode (depth from gradient)

Shared across all voices:
  WhiteNoise (looped AudioBuffer) ──→ GainNode (level from error) ──→ AnalyserNode ──→ Master
```

**Click prevention**: all parameter changes use `cancelScheduledValues(t) → setValueAtTime(current, t) → linearRampToValueAtTime(target, t + 30ms)`. No discontinuities, no pops.

**Convergence chord**: on convergence, a major chord blooms at `440 × [1, 5/4, 3/2, 2]` Hz — the mathematically simplest consonant ratios — and fades over 1.5s with exponential decay.

---

## Verified Results

All 6 problems converge correctly:

| Problem | Error Function | Target | Result | Steps | Final Error |
|---------|---------------|--------|--------|-------|-------------|
| Single Variable | (x−5)² | x=5 | x=4.995 | 106 | 2.3e-5 |
| Two Variables | (x+y−10)²+(x−y−2)² | (6, 4) | (6.000, 4.001) | 119 | 2.9e-6 |
| Quadratic Bowl | x²+y² | (0, 0) | (0.000, −0.003) | 106 | 7.3e-6 |
| Rosenbrock Valley | (1−x)²+100(y−x²)² | (1, 1) | (0.989, 0.978) | 794 | 1.2e-4 |
| Three Variables | (x−1)²+(y−2)²+(z−3)²+(x+y+z−6)² | (1,2,3) | (1.000, 2.003, 2.996) | 252 | 2.4e-5 |
| Multi-Minima | x²+y²+3sin²(x)sin²(y) | (0, 0) | (0.005, −0.001) | 269 | 2.4e-5 |

**Acoustic verification** for every problem:
- **Start**: static, silence, shaking frequencies (high error, high gradient)
- **Mid-solve**: tone emerges from noise, vibrato decreases, volume rises
- **Converged**: pure, stable, loud sine tone — zero noise, zero vibrato

The Rosenbrock Valley (the classic hard optimization benchmark) required gradient clipping and reduced learning rate to prevent divergence from its 100x gradient scaling.

---

## Project Structure

```
resonance/
├── index.html           # Single-page app
├── style.css            # Dark theme, monospace, cyan/green/red/yellow
├── package.json         # Vite dev server
├── vite.config.js
├── RESONANT_SOLVER.md   # Full technical documentation
└── src/
    ├── solver.js        # ResonantSolver engine + 6 problem definitions
    ├── audio.js         # Web Audio API real-time engine
    └── app.js           # UI, optimization loop, waveform/chart rendering
```

---

## UI

- **Problem tabs**: switch between 6 optimization problems
- **Waveform**: real-time oscilloscope colored by error (red → green)
- **State cards**: ERROR, |∇E|, STEP, STATUS — all update every frame
- **Variable monitors**: per-variable value + FREQ/AMP/VIB/NOISE bars
- **Error chart**: log₁₀(E) history — watch the descent curve form
- **Controls**: START/PAUSE/RESET, learning rate slider, speed slider, volume

---

## Rules (Non-Negotiable)

These ensure mathematical honesty over aesthetic appeal:

| Rule | Why |
|------|-----|
| No random pitch mapping | Frequency is `200 + 20x` — deterministic, invertible |
| No aesthetic tuning | All sound params derived from E(x) and ∇E(x) |
| Must encode gradient | Vibrato = `\|∂E/∂xᵢ\| × 8` — not approximated |
| Must respond to small changes | 0.001 change in x = 0.02 Hz shift (audible) |
| Convergence must be audible | Noise→pure tone IS the proof that E→0 |
| Eyes-closed detection | The acoustic gap between solving/converged is unmistakable |

---

## Future Directions

The 4-layer framework (`variable→freq`, `error→amp+noise`, `gradient→vibrato`, `multi-var→interference`) is domain-agnostic. Define E(x) for any domain and the sound follows:

- **Code quality**: E = weighted sum of AST-detected issues → hear bugs as dissonance
- **LLM hallucination**: E = semantic incoherence score → unstable harmonics = likely hallucination
- **Data anomalies**: E = deviation from expected distribution → anomaly = noise spike
- **Proof verification**: E = logical gap between steps → invalid step = frequency jump
- **DNA/protein folding**: E = free energy of molecular conformation → folded = consonance

The spectrogram of a converging optimization is a novel feature space. A CNN trained on "convergence spectrograms" vs "divergence spectrograms" could detect solution quality without running the full optimization.

---

## Quick Start

```bash
git clone https://github.com/Roxrite0509/resonance.git
cd resonance
npm install
npm run dev
```

Open `http://localhost:5557`. Pick a problem. Click START. Close your eyes. Listen to computation find the answer.

---

## License

MIT
