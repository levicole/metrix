# Metrix - Norns Metropolix

A norns script recreating the Intellijel Metropolix eurorack sequencer.

## Architecture

```
metrix.lua          -- Main entry: UI (screen + grid), params, event routing
lib/sequencer.lua   -- Core engine: lattice clock, pulse scheduling, note output (audio/MIDI/crow)
lib/track.lua       -- Track state: division, playback order, loop range, pitch quantization
lib/stage.lua       -- Per-stage params: pulse count, ratchet, gate type, pitch, octave, probability, slide, skip, accumulator
lib/preset.lua      -- 64-slot preset save/load (tab serialization to disk)
lib/chords.lua      -- 14 chord shapes added to musicUtil.SCALES
lib/helpers.lua     -- Screen drawing helpers (play/pause/stop icons)
lib/math.lua        -- math.lowerRandom (biased random)
```

## Key Concepts

- **Two tracks**, each with 8 **stages**. Tracks are independent (own division, playback order, loop range).
- Each stage has a **pulse count** (1-8). The sequencer advances through pulses within a stage before moving to the next stage.
- **Gate types**: hold (sustain across all pulses), multiple (retrigger each pulse), single (first pulse only), rest (silent).
- **Ratchets**: subdivide a pulse into rapid repeated triggers (1-8 per pulse).
- **Accumulator/transposition**: each stage can shift pitch up/down by a configurable amount, triggered per stage/pulse/ratchet.
- **Probability**: per-stage chance of playing (1, 0.75, 0.5, 0.25).
- **Slide**: portamento between notes (0-5 seconds via params).

## Output Routing

Each track can independently output to:
- **Audio** (MollyThePoly engine)
- **MIDI** (configurable channel, portamento via CC#65/CC#5)
- **Crow** (outputs 1/3 = gate/trigger/envelope, outputs 2/4 = 1V/oct pitch)

## Grid Pages

1. **Pulses & Gates** (shift: ratchets & probability)
2. **Pitch & Octave** (shift: transpose amount, slide on/off, transpose direction)
3. **Presets & Track Settings** (playback order, division per track)
4. **Scales & Root Note**

## Conventions

- Lua with norns APIs (lattice, musicutil, params, crow, screen, grid)
- Engine: MollyThePoly (polyphonic synth)
- Clock: lattice at 24 PPQN, each track gets its own lattice pattern
- Events are scheduled into `self.events[ppqn]` table for future execution (note-offs, ratchets)
- Pulse objects are generated on-the-fly in `track:getPulse()` with all note data computed
- Grid coordinates: X=1-8 columns (stages), Y rows vary by page
- Shift key at grid positions (8,1) and (8,16); Mod key at (7,1) and (7,16)

## Features NOT Yet Implemented (vs real Metropolix)

- **Mod lanes** (8 independent modulation sequencers - the big one)
- Swing/groove
- Brownian playback order
- Per-stage velocity (currently hardcoded: MIDI=100, audio=1)
- Assignable AUX outputs (A/B)
- Preset chaining
- CTRL knobs (assignable performance knobs)
- Per-stage gate length override (values exist in stage but commented out in grid UI)
- CV input routing (AUX X/Y/Z inputs)
- Loopy enhancements: timed loop (auto-return after N pulses), keyboard mode when stopped

## Development Notes

- The `sequencer.lua` file has an uncommitted change adding MIDI portamento CC control
- Slide for audio engine is global (MollyThePoly limitation), not per-track
- Gate lengths are partially implemented (values exist in stage but not fully wired)
- Preset system uses `tab.save`/`tab.load` with flat key serialization
