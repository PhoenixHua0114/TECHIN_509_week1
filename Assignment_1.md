System Overview (Modules)
* Config & CLI
    * Reads settings (key, mode, tempo, length, randomness seed).
    * Optional interactive prompts; otherwise uses sensible defaults.
* Theory Primitives
    * Defines pitch classes, scales, modes, intervals, and rhythm units.
* Rhythm Generator
    * Produces a sequence of note durations that sums to the target length.
* Pitch Generator
    * Chooses pitches within a key/scale using simple probabilistic rules or a Markov model.
* Melody Assembler
    * Zips rhythm + pitch into time-placed notes; enforces ranges and rests.
* Evaluator / Post-Processor
    * Smooths big leaps, avoids dissonant traps, and inserts cadences.
* Exporter
    * Writes MIDI (primary), plus CSV/JSON (for analysis). Optional MusicXML later.
* Logger
    * Captures parameters and random seed for reproducibility.

    Input
What does the program need to start?
* Musical Constraints
    * key: e.g., C, D♭, E
    * mode/scale: major, natural minor, dorian, pentatonic, etc.
    * tempo: BPM (e.g., 96)
    * time signature: e.g., 4/4, 3/4
    * register: preferred MIDI pitch range, e.g., 60–76 (C4–E5)
* Form & Length
    * measures or total_beats
    * phrase_structure (optional): e.g., [4, 4, 4, 4] for A A' B A
* Randomness
    * seed for reproducibility
    * stochasticity: low/medium/high
* Style Hints (optional)
    * contour_bias: ascending / descending / arch / random
    * step_vs_leap: proportion like 70/30
    * rest_probability: % chance to insert short rests
    * cadence: perfect / imperfect / none
    Output
What does it produce?
* MIDI file (primary): load into any DAW or notation tool.
* CSV/JSON (secondary): for analysis, grading, or visualization pipelines.
* Console summary: key stats, cadence type, range, unique pitches used.

epresentation
Notes & Pitches
* Internal pitch: MIDI integers (0–127) — easy to map to instruments and export to MIDI.
* Convenience labels: map MIDI ↔ note names (e.g., 60 ↔ C4) for logging only.
Duration & Time
* Beat-based timing:
    * start_beat (float)
    * duration_beats (float)
    * With tempo, compute milliseconds if needed.
* Subdivision:
    * Smallest unit (e.g., 1/8 beat) to ensure rhythmic grid is consistent.
* Rests:
    * Represent as is_rest=true or pitch=null/rest.

    Logic
High-Level Flow
1. Build scale (key + mode) → set allowed pitch classes.
2. Generate rhythm → a list of durations summing to target beats.
3. Generate pitch path → stepwise-biased random walk on scale degrees.
4. Assemble melody → pair durations with pitches, apply range & rest rules.
5. Evaluate & fix → reduce awkward leaps, insert cadence tones.
6. Export → MIDI + analysis files.
Rhythm Generation (simple options)
* Pattern library: choose from preset patterns that fit the time signature (e.g., [1,1,1,1], [0.5,0.5,1,2]).
* Stochastic tiling: repeatedly sample from {0.5, 1, 1.5, 2} until target length; reject overflows.
* Accent awareness: prefer note onsets at strong beats (1 and 3 in 4/4).
Pitch Selection (baseline heuristics)
* State: current scale degree (1–7), last interval, current register.
* Rules:
    * Step vs. leap: e.g., 70% step (±1 degree), 30% leap (±3–5 semitones).
    * Range clamp: keep within register; bounce back at edges.
    * Consonance bias: favor scale tones; allow occasional chromatic approach notes (low probability).
    * Contour: nudge direction to achieve target contour (arch, etc.).
    * Motif echo: with small probability, repeat or invert the previous 2–3 notes.