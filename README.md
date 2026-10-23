# reaper-jsfx

A collection of my experiments with [JSFX](https://www.reaper.fm/sdk/js/js.php), the scripting format for audio and MIDI effects in [REAPER](https://www.reaper.fm/).

This is a personal playground, so expect things to be rough, opinionated and occasionally half-finished. Each effect lives in its own file at the top level of the repository.

## Effects

| Effect | File | Type | What it does |
| --- | --- | --- | --- |
| Block Chords | `Block Chords.jsfx` | MIDI | Turns single-note melodies into block chords |
| Maestro FZ-1 Fuzz-Tone | `Maestro FZ-1 Fuzz Tone.jsfx` | Audio | Three-stage germanium-style fuzz inspired by the Maestro FZ-1 |
| Dallas Arbiter Rangemaster Treble Booster | `Dallas Arbiter Rangemaster Treble Booster.jsfx` | Audio | Single-transistor germanium treble booster inspired by the Dallas Arbiter Rangemaster |
| Boss CE-1 Chorus Ensemble | `Boss CE-1 Chorus Ensemble.jsfx` | Audio | BBD-style chorus and vibrato inspired by the Boss CE-1 Chorus Ensemble |
| Harmony | `Harmony.jsfx` | MIDI | Scale and chord engine: turns single notes into chords built from a transformable scale |

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

## Boss CE-1 Chorus Ensemble

A chorus and vibrato inspired by the Boss CE-1 Chorus Ensemble, the 1976 pedal built from the chorus circuit of the Roland JC-120 amp. The signal goes through a modulated delay line that stands in for the original's bucket-brigade (BBD) chip. It adds no latency, because the delay line is the effect.

### How it works

The two modes behave differently, as on the original:

- **Chorus mode:** only **Chorus Intensity** is active, and it sets the LFO **rate** (about 1 to 3 Hz), not the depth. The delay sweep is fixed (3 to 5 ms) and the LFO is a triangle, rounded toward a sine by default with **Chorus LFO Shape**. A pure triangle flips the pitch direction abruptly at each end of the sweep, which can sound warbly. The output is the direct signal plus the delayed signal, each at -3 dB.
- **Vibrato mode:** **Vibrato Rate** (about 3.2 to 11.6 Hz) and **Vibrato Depth** are active, and Chorus Intensity does nothing. The LFO is sine-like. The dry signal is muted, so you hear a true pitch vibrato.

Other parts of the model:

- **Delay law:** a BBD's delay is inversely proportional to its clock frequency, and the LFO modulates the clock. The delay therefore follows `dmin / (1 - k * u)`, where `u` is the LFO (0 to 1). The sweep starts at the minimum delay and is slightly asymmetric, as on a real BBD. The delay line is read with 4-point Hermite interpolation.
- **Preamp:** a mild asymmetric soft clipper with a small bias, which adds a little even-order colour. It is normalised so small signals pass at unity gain. **Input Sensitivity** adds 18 dB when set to 10 mV (the original's 80 mV / 10 mV switch), and **Level** scales the signal into the clipper.
- **BBD bandwidth and noise:** the delayed signal goes through a two-pole low-pass (9 kHz by default), with optional hiss of up to about -60 dB, in line with the unit's 60 dB spec.
- **LFO drift:** a slow random wander of the LFO rate, up to +/-5%, so the modulation is not perfectly regular.
- **Smoothing:** the rate, delay, mode and mix settings are smoothed over about 30 ms, so switching modes or turning knobs does not click.

The preamp is not oversampled, so very high **Level** or the Mic sensitivity setting with a hot input can alias slightly.

### How faithful it is

The rate and delay ranges come from one published measurement of a single unit (a forum post), cross-checked against the manuals where possible. They are not measured by me. That post's vibrato depth numbers (a sweep up to 7 ms) give pitch swings of well over 200 cents at the fastest rate, which sounded far too extreme, so the defaults here are gentler (see below). The preamp curve, the Level taper and the 9 kHz bandwidth are estimates tuned by ear. The interaction between a real guitar pickup and the pedal's input is not modelled.

If you have a real CE-1 to compare with, the **Advanced tuning** sliders let you match its sweep widths, rate range and minimum delay.

### Usage

1. Put the effect on an audio track. A guitar, electric piano or synth works well.
2. Pick **Chorus** or **Vibrato** with **Mode**.
3. In Chorus mode, set **Chorus Intensity** for the speed. In Vibrato mode, set **Vibrato Rate** and **Vibrato Depth**.
4. Choose an **Output Mode** and match the level with **Output (dB)**.

**Output Mode** options:

- **Mono Mix:** a single output. Chorus mode gives dry plus wet, Vibrato mode gives wet only. The stereo input is summed to mono first, like the pedal's single input.
- **Classic Stereo:** left is the Direct output (preamp'd dry) and right is the Effect output (wet only), like the original's two jacks. The input is summed to mono first.
- **Dual:** two CE-1s, one per input channel, with the two LFOs 90 degrees apart. The input stays stereo.

With the default Vibrato settings, the pitch swing is roughly +/-40 cents at maximum depth and a 5 Hz rate, and roughly +/-95 cents at maximum depth and the fastest rate. At minimum depth it is roughly +/-5 cents at the slowest rate and +/-19 cents at the fastest. These are approximate, worked out from the delay sweep and rate. Depth in a delay-based vibrato grows with rate, so the fast end of **Vibrato Rate** always has the most swing.

### Controls

| Control | Range | What it does |
| --- | --- | --- |
| Mode | Chorus, Vibrato | Selects the effect. Only the controls for that mode are active |
| Chorus Intensity | 0 to 10 | Chorus mode only. Sets the LFO rate, from about 1 to 3 Hz (exponential). Does not change the depth |
| Vibrato Rate | 0 to 10 | Vibrato mode only. Sets the LFO rate between **Vibrato Rate Min** and **Max** (exponential) |
| Vibrato Depth | 0 to 10 | Vibrato mode only. Widens the delay sweep from **Vibrato Sweep Max @ Depth 0** to **@ Depth 10** |
| Input Sensitivity | 80 mV, 10 mV | 10 mV adds 18 dB of gain into the preamp, like the original's switch. Use it for quiet sources |
| Level | 0 to 10 | Signal level into the preamp. 7 is unity gain and 10 is about +6 dB. Higher values clip harder |
| Output Mode | Mono Mix, Classic Stereo, Dual | How the direct and delayed signals are routed to the outputs (see Usage) |
| Output (dB) | -24 to +12 | Level out of the plugin |
| BBD Bandwidth (kHz) | 3 to 16 | Low-pass on the delayed signal. Lower is darker and more like a worn BBD |
| BBD Noise (%) | 0 to 100 | Hiss on the delayed signal. 100% is about -60 dB |
| LFO Drift (%) | 0 to 100 | Slow random wander of the LFO rate. 100% is about +/-5% |
| Min Delay (ms) | 1 to 6 | Shortest delay, where the sweep starts. Used by both modes |
| Chorus Sweep Max (ms) | 3 to 9 | Longest delay reached in Chorus mode |
| Chorus LFO Shape | 0 to 100 | Chorus mode only. Blends the LFO from a triangle (0) to a sine (100). Higher values soften the pitch changes at the ends of the sweep. Vibrato mode always uses a sine |
| Vibrato Sweep Max @ Depth 0 (ms) | 3 to 9 | Longest delay reached in Vibrato mode with Vibrato Depth at 0 |
| Vibrato Sweep Max @ Depth 10 (ms) | 3 to 9 | Longest delay reached in Vibrato mode with Vibrato Depth at 10. Raise it for a wilder vibrato |
| Vibrato Rate Min (Hz) | 0.5 to 20 | LFO rate with Vibrato Rate at 0 |
| Vibrato Rate Max (Hz) | 1 to 30 | LFO rate with Vibrato Rate at 10 |

The sliders appear in four groups in this order: main controls (Mode to Vibrato Depth), input and output (Input Sensitivity to Output), sound character (BBD Bandwidth to LFO Drift), and advanced tuning (Min Delay to Vibrato Rate Max, including Chorus LFO Shape). Reordering them changes how saved presets and automation map to the controls.

### Defaults

| Setting | Default |
| --- | --- |
| Mode | Chorus |
| Chorus Intensity | 5 |
| Vibrato Rate | 5 |
| Vibrato Depth | 5 |
| Input Sensitivity | 80 mV |
| Level | 7 |
| Output Mode | Classic Stereo |
| Output (dB) | 0 |
| BBD Bandwidth (kHz) | 9 |
| BBD Noise (%) | 0 |
| LFO Drift (%) | 25 |
| Min Delay (ms) | 3 |
| Chorus Sweep Max (ms) | 5 |
| Chorus LFO Shape | 50 |
| Vibrato Sweep Max @ Depth 0 (ms) | 3.3 |
| Vibrato Sweep Max @ Depth 10 (ms) | 4.5 |
| Vibrato Rate Min (Hz) | 3.2 |
| Vibrato Rate Max (Hz) | 11.6 |

## Harmony

A MIDI scale and chord engine. Each note you play is expanded into a chord that is built from a scale, and every step of that process is adjustable: the scale itself, a chain of transforms on the scale, the rule that picks the chord notes, and a chain of transforms on the chord. It is a port of the "pitchwheel" web tool.

Unlike Block Chords, which harmonizes a melody, Harmony treats each key as a command. Depending on the **Input** mode, the key sets the root, picks a scale degree, or lets the plugin work out which scale you are playing in.

The effect has its own editor that scales to fit the window. A keyboard strip above the panels shows the live scale (blue), the current chord (orange) and the keys you are holding.

### How it works

The chord is built in four stages, one panel each in the editor:

1. **Scale:** a base scale from the library, a generator or your own list.
2. **Scale transform:** Root, Mode, Degree, Invert, Mirror and Negative, applied in that order. The result is the live scale.
3. **Chord:** a rule that picks notes from the live scale (a list of scale degrees or of interval steps), the degree it starts on, a rotation and a number of voices.
4. **Chord transform:** Position, Offset, Mode, Invert, Mirror, Close, Spread, Resize and Auto-voicing, applied in that order. Chord **Negative** acts earlier, on the scale the chord is picked from.

Chord tones that would fall outside MIDI notes 0 to 127 are dropped. All other MIDI messages pass through untouched. All-notes-off and all-sound-off (CC 123 and 120) stop every chord.

#### Scale sources

- **Library:** 402 scales in nine categories (Major and minor, European, Asian, Indian, Pentatonic, Modal, Jazz, Symmetric, Misc), with a search box that filters across all categories. The scale library comes from Francesco Balena's *The Scale Omnibus* (see Credits).
- **Euclid:** a Euclidean rhythm with **Events** onsets spread over 12 steps, used as a scale.
- **C-D:** the Clough-Douthett maximally even set of **Events** notes in 12.
- **Deep:** a deep rhythm, with notes at *i* x **Multiplicity** (mod 12) for *i* below **Events**.
- **Custom:** your own list of positions (mod 12). Type the list and press Enter, or click the twelve pitch-class cells.

#### Scale transforms

- **Root:** transposes the scale to C through B.
- **Mode:** keeps the root and rotates the interval pattern, so C major becomes C Dorian, C Phrygian and so on. Values above the number of notes in the scale wrap around.
- **Degree:** keeps the same notes and starts them from another degree, so C major becomes D Dorian, E Phrygian and so on (the relative mode).
- **Invert:** reverses the order of the intervals on either side of the **axis** point.
- **Mirror:** reads the scale in the opposite direction.
- **Negative:** negative harmony. Every note is reflected across the axis that sits halfway between the root and its fifth.

#### Chord rule

- **Position** criterion: a list of scale degrees (integers from 0 up). The default `0 2 4` is a triad.
- **Interval** criterion: a list of steps in scale degrees, stacked one on top of the next. The default `2` stacks thirds.
- **Degree:** the scale degree the chord is built on.
- **Rotation:** starts the criterion list from a different entry, which gives rule-based inversions.
- **Voices:** the number of notes. 0 uses the length of the criterion list, and a larger number keeps going up the scale.
- **Octave:** the register of the chord (see Root mode below for how it combines with the key you play).

#### Chord transforms

| Transform | What it does |
| --- | --- |
| Position | Inverts the chord by that many notes, up for positive and down for negative |
| Offset | Transposes the chord by that many semitones |
| Mode | Rotates the chord's interval pattern while keeping the lowest note |
| Invert | Flips the chord upside down around the chosen chord tone (**axis**) |
| Mirror | Reverses the order of the chord's intervals |
| Negative | Picks the chord from the negative-harmony version of the scale |
| Close | Folds the chord into one octave, the one that already holds most of its notes |
| Spread | Opens the voicing: neighbouring notes end up at least a whole tone apart at the bottom, growing to a fourth at the top. **Top cluster** lets the highest notes sit close together |
| Resize | Replaces the chord with notes **s** to **e** of its octave-repeating series, so the chord can get bigger, smaller or run downward (if s is greater than e) |
| Auto-voicing | Picks the inversion and octave closest to the previous chord, for smooth voice leading |

Auto-voicing remembers the previous chord. **Reset memory** forgets it, so the next chord is voiced from scratch.

### Input modes

- **Root:** each key sets the scale root to its pitch class and plays that chord. The chord sits in the octave of the key you play, so there is no octave looping: a C one octave higher gives a chord one octave higher. The **Chord Octave** setting is relative to that, with the octave of middle C as the reference. At the default of 4, a middle C gives a chord rooted on C4, and 5 puts the chord an octave above the key.
- **Degree:** white keys play the chords of consecutive scale degrees, starting at middle C as degree 0 (C = 0, D = 1, E = 2 and so on, seven degrees per octave). **Chord Octave** moves the starting key. Black keys are ignored.
- **Direct:** notes pass through unchanged, still polyphonic. Use it when you want the keyboard to play normally.
- **Auto:** you just play, and the plugin finds the scale. See below.

In every mode except Direct, each held key owns its own chord. The scale root, degree and mode follow the most recently pressed key, while the chords of the other held keys keep sounding.

### Auto mode

In Auto mode the plugin looks at all the keys you are holding. If they all belong to the live scale, the scale stays as it is. If not, it looks for the closest scale that contains them all and switches to it. The chord is then built on the scale degree of the key you pressed last, in that key's octave. "Closest" is measured as the total semitone distance between the two scales' notes. If no candidate contains every held note, the scale is left alone.

The **Sub-mode** decides what can change:

- **Modal Interchange:** the scale is rotated through its own modes. The candidates are the modes of the scale you started with, all on the same root (C major, C Dorian and so on).
- **Modulation:** the scale keeps its mode and is transposed. All twelve transpositions are candidates.

**Distance From** decides what "closest" is measured against:

- **Original:** the initial scale (see below).
- **Current:** the scale that is sounding right now. This is the default, and it makes the scale drift by small steps as you play.

The **initial scale** is the mode, transposition and chord degree you had when you switched to Auto. Changing the **Mode** slider by hand while in Auto sets a new initial scale, and so does changing the **Root** in Modulation.

Three things return you to the initial scale:

- The **Reset** button (or the **Reset Auto** action), which also stops the sounding chords.
- **Autoreset**, when on: it resets whenever you press a key while no other key is held, before the scale is chosen.
- Leaving Auto mode, or switching the Sub-mode.

### Usage

1. Put the effect on a MIDI track, before your instrument.
2. Pick a scale in the **Scale** panel. Use the category arrows and the list, or the search box, or switch the source to Euclid, C-D, Deep or Custom.
3. Choose an **Input** mode and play. In Root mode every key is a root. In Degree mode the white keys walk up the scale. In Auto mode just play in any key.
4. Shape the chord with the **Chord** panel (the rule and the number of voices) and the **Chord Transform** panel (inversions, spreading, voice leading and so on).

The values in the editor can be dragged or scrolled. Clicking the left or right half of a value steps it down or up, and right-click sets it back to its default. In the chord rule, each number in a list is a chip: click to add 1, right-click to subtract 1, shift-click to delete, and use the `+` chip to add an entry. Lists are limited to 64 entries.

When more than one chord holds the same note, the note is reference-counted, as in Block Chords: releasing one chord never cuts off a note another chord is still using. **Common Notes** decides what happens when a new chord shares a note with one that is already sounding: **Hold** keeps the note going without a new note-on (legato), **Retrigger** plays it again.

### Controls

All of these are sliders, so they can be automated. **Input**:

| Control | Range | What it does |
| --- | --- | --- |
| Input: Mode | Root, Degree, Direct, Auto | How a played key drives the chord |
| Input: Auto Sub-mode | Modal Interchange, Modulation | What Auto mode is allowed to change |
| Input: Auto Distance From | Original, Current | What Auto mode measures the closest scale against |
| Input: Auto Autoreset | Off, On | Return to the initial scale on every fresh key press |
| Input: Common Notes | Hold, Retrigger | What happens to a note that two chords share |
| Output Channel | Same as input, 1 to 16 | MIDI channel of the output notes |
| Action | -, Reset Auto, Reset Voice-Leading | Momentary actions (these are the **Reset** and **Reset memory** buttons) |

**Scale**:

| Control | Range | What it does |
| --- | --- | --- |
| Scale: Source | Library, Euclidean, Clough-Douthett, Deep, Custom | Where the base scale comes from |
| Scale: Library Category | 9 categories | Category of the library scale |
| Scale: Library Index | 0 to 159 | Position in the category (limited to the category's size) |
| Scale: Generator Events | 1 to 12 | Number of notes for Euclid, C-D and Deep |
| Scale: Generator Multiplicity | 1 to 23 | Deep only: the step multiplier |

**Scale transform** (applied in this order):

| Control | Range | What it does |
| --- | --- | --- |
| Scale TX: Root | C to B | Transposes the scale |
| Scale TX: Mode | 0 to 11 | Rotates the interval pattern on the same root |
| Scale TX: Degree | -12 to +12 | Starts the same notes from another degree |
| Scale TX: Invert | Off, On | Reverses the intervals around the axis |
| Scale TX: Invert Axis | -12 to +12 | Where the inversion splits the interval pattern |
| Scale TX: Mirror | Off, On | Reads the scale in the opposite direction |
| Scale TX: Negative | Off, On | Negative harmony around the root-and-fifth axis |

**Chord**:

| Control | Range | What it does |
| --- | --- | --- |
| Chord: Octave | 0 to 9 | Register of the chord. In Root mode it is relative to the octave of the key |
| Chord: Criterion Type | Position, Interval | Whether the rule is a list of degrees or a list of steps |
| Chord: Degree | -72 to +80 | Scale degree the chord is built on |
| Chord: Rotation | -12 to +12 | Starts the rule's list from a different entry |
| Chord: Voices | 0 to 32 | Number of notes. 0 uses the length of the list |

**Chord transform** (applied in this order, with Negative acting on the scale before the chord is picked):

| Control | Range | What it does |
| --- | --- | --- |
| Chord TX: Position | -12 to +12 | Chord inversion, in notes |
| Chord TX: Offset | -24 to +24 | Transposition, in semitones |
| Chord TX: Mode | 0 to 11 | Rotates the chord's interval pattern |
| Chord TX: Invert | Off, On | Flips the chord around the axis note |
| Chord TX: Invert Axis | -12 to +12 | Which chord tone the flip is centred on |
| Chord TX: Mirror | Off, On | Reverses the order of the chord's intervals |
| Chord TX: Negative | Off, On | Picks the chord from the negative-harmony scale |
| Chord TX: Close | Off, On | Folds the chord into one octave |
| Chord TX: Spread | Off, On | Opens the voicing |
| Chord TX: Spread Top Cluster | Off, On | Lets the top notes of a spread chord sit close together |
| Chord TX: Resize | Off, On | Replaces the chord with notes Start to End of its series |
| Chord TX: Resize Start | -24 to +24 | First note of the resized chord |
| Chord TX: Resize End | -24 to +24 | Last note of the resized chord |
| Chord TX: Auto-voicing | Off, On | Smooth voice leading from the previous chord |

The chord rule lists (degrees, interval steps) and the custom scale are not sliders, but they are saved with the project and with FX presets. The search box in the library list is not saved.

The 38 sliders appear in this order: Scale source (5), Scale transform (7), Chord (5), Chord transform (14), then Input and Output (7). Reordering them changes how saved presets and automation map to the controls.

### Defaults

| Setting | Default |
| --- | --- |
| Input: Mode | Root |
| Input: Auto Sub-mode | Modal Interchange |
| Input: Auto Distance From | Current |
| Input: Auto Autoreset | Off |
| Input: Common Notes | Hold |
| Output Channel | Same as input |
| Scale: Source | Library |
| Scale: Library | Major and minor, Ionian (Major) |
| Scale: Generator Events | 7 |
| Scale: Generator Multiplicity | 7 |
| Custom scale | 0 2 4 5 7 9 11 |
| Scale TX: Root | C |
| Scale TX: Mode, Degree, Invert Axis | 0 |
| Scale TX: Invert, Mirror, Negative | Off |
| Chord: Octave | 4 |
| Chord: Criterion Type | Position |
| Chord: Position list | 0 2 4 |
| Chord: Interval list | 2 |
| Chord: Degree, Rotation | 0 |
| Chord: Voices | 0 (the length of the list) |
| Chord TX: Position, Offset, Mode, Invert Axis | 0 |
| Chord TX: Invert, Mirror, Negative, Close, Spread, Top Cluster, Resize, Auto-voicing | Off |
| Chord TX: Resize Start, End | 0, 3 |

With these defaults every key plays a major triad built on itself, in the octave of the key.

## Installation

1. In REAPER, open **Options > Show REAPER resource path in explorer/finder**.
2. Copy the `.jsfx` file into the `Effects` folder (or a subfolder of it).
3. In the FX browser, press **F5** to refresh, or restart REAPER.
4. Add the effect to a track:
   - **JS: Block Chords (melody harmonizer)** or **JS: Harmony (scale/chord MIDI processor)** on a MIDI track
   - **JS: Maestro FZ-1 Fuzz-Tone**, **JS: Dallas Arbiter Rangemaster Treble Booster**, **JS: Boss CE-1 Chorus Ensemble** on an audio track

REAPER stores slider values with the project and with FX presets. If you change the defaults in the file and an existing instance still shows the old values, add a fresh instance.

## Credits

- The scale library in **Harmony** (the scales and their names) comes from Francesco Balena, *The Scale Omnibus*.

## Contributing

These are personal experiments, but ideas and bug reports are welcome. Open an issue if something sounds wrong.
