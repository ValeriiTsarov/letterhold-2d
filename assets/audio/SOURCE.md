# Audio

Everything here is either CC0 foley carried over from Solitaire Duelist 2D (ClassicCards / Game-Makers
packs plus two Kenney All-in-1 fills) or generated on this machine. `src/audio.ts` is the manifest — a
file that isn't listed there is not loaded.

## word-chime.ogg — ours

The "that's a word" bell, played with the wave that runs along a legal word. Synthesised rather than
sampled: nothing in the foley packs is a bell, and this fires several times a turn, so it has to be short,
quiet and tonal enough to survive repetition (`jitter: 0` in the manifest — a detuned bell reads as a
broken file).

```bash
ffmpeg -y -f lavfi -i "aevalsrc=(1-exp(-400*t))*(0.42*exp(-7*t)*sin(2*PI*1760*t)+0.20*exp(-11*t)*sin(2*PI*3520*t)+0.10*exp(-17*t)*sin(2*PI*5310*t)):s=48000:d=0.75" \
  -ac 1 -c:a libvorbis -q:a 3 assets/audio/word-chime.ogg
```

A6 (1760 Hz) with two inharmonic partials over it, each decaying faster than the last, and a 2.5 ms
attack ramp so the file doesn't open on a click. 5.6 KB.
