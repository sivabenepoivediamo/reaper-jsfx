# reaper-jsfx

A collection of my experiments with [JSFX](https://www.reaper.fm/sdk/js/js.php), the scripting format for audio and MIDI effects in [REAPER](https://www.reaper.fm/).

This is a personal playground, so expect things to be rough, opinionated and occasionally half-finished. Each effect lives in its own file at the top level of the repository.

## Effects

| Effect | File | Type | What it does |
| --- | --- | --- | --- |
| Block Chords | `blockchords.jsfx` | MIDI | Turns single-note melodies into block chords |

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

## Installation

1. In REAPER, open **Options > Show REAPER resource path in explorer/finder**.
2. Copy the `.jsfx` file into the `Effects` folder (or a subfolder of it).
3. In the FX browser, press **F5** to refresh, or restart REAPER.
4. Add **JS: Block Chords (melody harmonizer)** to a track.

REAPER stores slider values with the project and with FX presets. If you change the defaults in the file and an existing instance still shows the old values, add a fresh instance.

## Contributing

These are personal experiments, but ideas and bug reports are welcome. Open an issue if something sounds wrong.
