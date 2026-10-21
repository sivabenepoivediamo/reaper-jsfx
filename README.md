# reaper-jsfx

A collection of my experiments with [JSFX](https://www.reaper.fm/sdk/js/js.php), the scripting format for audio and MIDI effects in [REAPER](https://www.reaper.fm/).

This is a personal playground, so expect things to be rough, opinionated and occasionally half-finished. Each effect lives in its own file at the top level of the repository.

## Effects

| Effect | File | Type | What it does |
| --- | --- | --- | --- |
| Block Chords | `Block Chords.jsfx` | MIDI | Turns single-note melodies into block chords |
| Maestro FZ-1 Fuzz-Tone | `Maestro FZ-1 Fuzz Tone.jsfx` | Audio | Three-stage germanium-style fuzz inspired by the Maestro FZ-1 |
| Rangemaster Treble Booster | `Dallas Arbiter Rangemaster Treble Booster.jsfx` | Audio | Single-transistor germanium treble booster inspired by the Dallas Arbiter Rangemaster |

More to come.

## Block Chords

A MIDI melody harmonizer. Every incoming note is treated as the top voice of the melody and expanded into a block chord. It is a port of the "blockchords" web tool.

### How it works

- **Notes in the base shape** use the inversion of that shape that puts the melody note on top.
- **Other scale degrees** are harmonized with descending thirds along the chosen scale. A voice that would form a tritone with the melody is moved up one scale step.
- **Notes outside both the scale and the base shape** are treated as a chromatic alteration of the nearest in-scale note. You can choose whether the whole chord is transposed with it or only the melody note is altered.
- **Bass doubling** adds the melody an octave below.
- **Drop 2 / Drop 3 / Drop 4** move the second, third or fourth voice an octave below the bass doubling.

Overlapping chords share notes safely: output notes are reference-counted, so releasing one chord never cuts off a note that another chord is still holding.

### Usage

1. Put the effect on a MIDI track, before your instrument.
2. Play single notes. Each one comes out as a chord.
3. Pick a **root** and a **scale**, then choose a **base shape**.

Scales and shapes are chosen from a category list plus a second list for the category. Choosing **Custom** in either category reveals twelve toggles (relative to the root) so you can build your own.

Notes outside the **Harmonize from / up to** range pass through unchanged.

### Defaults

| Setting | Default |
| --- | --- |
| Root | C |
| Scale | Ionian (Major) |
| Base shape | Major Sixth (6) |
| Harmony velocity | 85% of the melody velocity |
| Harmonize range | MIDI notes 60 to 127 |
| Out-of-scale notes: transpose whole chord | On |
| Bass doubling | On |
| Drop voices 2, 3, 4 | Off |

### Library

The effect ships with a large built-in library:

- **Scales:** major and minor modes, European, Asian, Indian (melas and ragas), pentatonic, modal, jazz, symmetric and miscellaneous.
- **Base shapes:** triads, sixth, seventh, ninth, eleventh and thirteenth chords, altered dominants, suspended and added-tone chords, symmetric and cluster chords, polychords and hybrids, and classical chromatic harmony.

## Maestro FZ-1 Fuzz-Tone

A stereo fuzz pedal inspired by the Maestro FZ-1 Fuzz-Tone. The two channels are processed independently, so it works on mono or stereo tracks.

### How it works

The signal runs through a simplified model of the pedal's three transistor stages:

- **Q1 gain stage:** a high-pass at 90 Hz (input coupling) feeds the first clipper. The **Fuzz** control sets how hard it is driven.
- **Coupling and Miller filtering:** a 250 Hz high-pass between the stages, followed by a low-pass that gets darker as **Attack** goes up.
- **Q2 stage with bias sag:** a second, fixed-gain clipper. An envelope follower (about 2 ms attack, 150 ms release) pulls this stage's bias down as the signal gets louder, which is what the **Sag** control scales.
- **Q3 buffer and tone:** a low-pass sweeps from roughly 1 kHz to 12 kHz with **Tone**, then a 20 Hz high-pass removes DC.

Both clippers are asymmetric to mimic germanium: the positive half clips softly, while the negative half has a lower ceiling. This gives the fuzz some even-order harmonics and a slightly uneven character.

The whole fuzz path runs at 2x oversampling, with a 4th-order Butterworth low-pass at 0.45 x the sample rate to keep aliasing down.

### Usage

1. Put the effect on an audio track, ideally first in the chain so it sees the raw instrument signal (guitar, bass, synth) before any other processing.
2. Set **Fuzz** for the amount of fuzz and **Volume (dB)** to match your level.
3. Shape the sound with **Bias**, **Tone** and **Sag**.

### Controls

| Control | Range | What it does |
| --- | --- | --- |
| Fuzz | 0 to 100 | Drive amount. Raises the Q1 gain from -6 dB to +44 dB and lowers the pre-clip low-pass from 9 kHz to 4 kHz |
| Volume (dB) | -40 to +12 | Level of the processed signal |
| Bias | 0 to 100 | Adds a DC offset ahead of the clipping stages, making the clipping more asymmetric. High settings get ragged and can starve the signal |
| Tone | 0 to 100 | Output low-pass, from dark (about 1 kHz) to bright (about 12 kHz) |
| Sag | 0 to 100 | Amount of level-dependent bias drop in the second stage. Loud passages compress and thin out, and notes can sputter as they decay |
| Mix (%) | 0 to 100 | Dry/wet blend. The dry signal is untouched, and **Volume** only applies to the wet signal |

### Defaults

| Setting | Default |
| --- | --- |
| Drive | 60 |
| Volume (dB) | -12 |
| Bias | 35 |
| Tone | 50 |
| Sag | 25 |
| Mix (%) | 100 |

## Dallas Arbiter Rangemaster Treble Booster

A stereo treble booster inspired by the Dallas Arbiter Rangemaster, the single-germanium-transistor pedal behind a lot of 1960s and 70s lead tones. The two channels are processed independently, so it works on mono or stereo tracks. There is no dry/wet mix: it is a straight boost, like the original.

### How it works

The signal runs through a simplified model of the pedal's single transistor stage:

- **Input coupling:** a one-pole high-pass formed by the input cap and the transistor's input impedance (the bias network in parallel with the transistor's hFE times a nominal emitter resistance). With the stock 5 nF cap the corner sits around 1.4 kHz, which is why the pedal sounds thin and treble-heavy.
- **Range:** the stage gain, from about -4 dB to +26 dB. It sits ahead of the clipper, so turning it up also drives the transistor harder.
- **Germanium stage:** an asymmetric soft clipper. Each half-wave has its own headroom, and **Bias** shifts the operating point, so the clipping is uneven and produces even-order harmonics. **Battery** scales the headroom, so a weaker battery clips earlier. **Warmth** adds a small squared term for extra even harmonics (the leakage-flavored part of germanium).
- **Hi-cut and output coupling:** a one-pole low-pass, then a 20 nF output cap into a roughly 150 kOhm load (a high-pass at about 53 Hz). This also removes the DC offset that the asymmetric clipping creates.

The clipper runs oversampled (2x, 4x or 8x) with a 6th-order Butterworth low-pass at 0.45 x the sample rate, capped at 20 kHz, on both the way up and the way down to keep aliasing down. These are IIR filters, so the plugin adds no latency.

Component values are close to commonly published Rangemaster schematics but were tuned by ear, not matched to measurements. The interaction between a real guitar pickup and the pedal's low input impedance is not modelled, so feed it a DI'd guitar and set **Input Trim** to taste.

### Usage

1. Put the effect on an audio track, ideally first in the chain so it sees the raw guitar signal before any amp sim or other drive.
2. Set **Range** for the amount of boost and **Output (dB)** to match your level.
3. Shape the tone with **Input Cap** and **Hi-Cut**, and the character of the clipping with **Bias**, **Germanium Asymmetry** and **Battery**.

At default settings the small-signal response runs from about -14 dB at 100 Hz to about +9 dB at 4 kHz (including the -6 dB output setting). Very quiet signals (around -40 dBFS or lower) stay fairly clean even at full Range. A typical guitar DI, around -20 to -10 dBFS, drives the stage much harder. If it is too clean, raise **Input Trim**, lower **Battery** or raise **Bias**.

The graph in the plugin window shows the small-signal frequency response (the linear region, before any clipping), so it reacts to Range, Input Cap, Transistor hFE, Hi-Cut, Input Trim and Output. Bias, Germanium Asymmetry, Battery and Warmth only affect the clipping, so they do not move it.

### Controls

| Control | Range | What it does |
| --- | --- | --- |
| Range | 0 to 10 | Stage gain, from about -4 dB to +26 dB. Also sets how hard the transistor is driven |
| Input Cap (nF) | 1 to 100 | Input coupling cap. The stock value is 5 nF. Try 10 to 47 nF for the well-known "fuller" mod |
| Transistor hFE | 30 to 150 | Sets the input impedance, which moves the input high-pass corner (a higher hFE gives a lower corner), plus a slight gain change |
| Bias (%) | -50 to +50 | Shifts the transistor's operating point. Away from 0 the two half-waves clip unevenly, adding even harmonics. Positive and negative values are mirror images |
| Germanium Asymmetry (%) | 0 to 100 | How much more headroom one half-wave has than the other. At 0 the clipper is symmetric |
| Battery (V) | 4 to 9.6 | Supply voltage. Headroom scales with voltage, so a weaker battery clips earlier and sounds more compressed |
| Hi-Cut (Hz) | 2000 to 20000 | One-pole low-pass after the clipper |
| Input Trim (dB) | -24 to +24 | Level into the stage, before the input high-pass |
| Output (dB) | -36 to +12 | Level out of the plugin |
| Warmth (%) | 0 to 100 | Amount of extra even-order harmonic content |
| Oversampling | Off, 2x, 4x, 8x | Oversampling factor for the clipper. Higher is cleaner and uses more CPU |

### Defaults

| Setting | Default |
| --- | --- |
| Range | 7 |
| Input Cap (nF) | 5 |
| Transistor hFE | 75 |
| Bias (%) | 0 |
| Germanium Asymmetry (%) | 60 |
| Battery (V) | 9 |
| Hi-Cut (Hz) | 12000 |
| Input Trim (dB) | 0 |
| Output (dB) | -6 |
| Warmth (%) | 25 |
| Oversampling | 4x |

## Installation

1. In REAPER, open **Options > Show REAPER resource path in explorer/finder**.
2. Copy the `.jsfx` file into the `Effects` folder (or a subfolder of it).
3. In the FX browser, press **F5** to refresh, or restart REAPER.
4. Add the effect to a track:
   - **JS: Block Chords (melody harmonizer)** on a MIDI track
   - **JS: Maestro FZ-1 Fuzz-Tone**, **JS: Dallas Arbiter Rangemaster Treble Booster** on an audio track

REAPER stores slider values with the project and with FX presets. If you change the defaults in the file and an existing instance still shows the old values, add a fresh instance.

## Contributing

These are personal experiments, but ideas and bug reports are welcome. Open an issue if something sounds wrong.
