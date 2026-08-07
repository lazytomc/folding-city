# Folding City : Hong Kong — browser version

A single-file web port of the iOS spatial-audio piece. One URL, any headphones,
no install. `index.html` contains all HTML/CSS/JS inline; the only external
assets are the four audio tracks in `audio/`.

## Run it locally

The page must be served over HTTP (browsers block `fetch()` from `file://`):

```bash
cd docs
python3 -m http.server 8000
# open http://localhost:8000
```

## Deploy on GitHub Pages

1. Push this repository to GitHub. **Note:** the WAV masters in `AudioMasters/`
   are 132 MB each and GitHub rejects files over 100 MB — add `AudioMasters/`
   to `.gitignore` (or use Git LFS) before pushing.
2. On GitHub: **Settings → Pages → Build and deployment → Deploy from a branch**,
   choose branch `main` and folder `/docs`, save.
3. The piece is live at `https://<username>.github.io/<repo>/` a minute later.
   Pages serves over HTTPS, which iOS requires for compass access.

## Encoding the audio

The WAV masters live in `AudioMasters/` at the repository root (132 MB each,
24-bit stereo). The web version uses mono AAC at 128 kbps — mono is what the
spatial mixer is fed in the iOS app too, and each 8:19 track comes out at
**about 8 MB (≈ 32 MB total)**:

```bash
for t in Track1_transport_V1 Track2_commercial_V1 Track3_praying_V1 Track4_bay_V1; do
  ffmpeg -i "AudioMasters/$t.wav" -ac 1 -ar 44100 -c:a aac -b:a 128k "docs/audio/$t.m4a"
done
```

(The files currently in `audio/` were produced with Apple's `afconvert` at the
same settings; the ffmpeg command is equivalent.) Filenames are listed once in
the `CONFIG.audio` object at the top of the script in `index.html` — change
them there if you re-encode under different names.

## What differs from the iOS app

- **Headphone mode → Compass mode.** Browsers expose no AirPods head-tracking
  and no raw pedometer, so the first segment is relabelled **Compass**: device
  heading turns the listener (turn your body, the sound field rotates), and a
  **Hold to Walk** button advances at a constant 1 m/s along the current
  heading. On iOS the motion-permission prompt appears when you select the
  mode; if declined, the piece stays in Touch mode.
- **Touch mode** is the default and is a faithful port of the app's drag
  behaviour.
- All spatial constants (`circleRadius` 3 m, `sourceRadius` 2.1 m,
  `refDistance` 1.5, `maxDistance` 4, rolloff = Spatial Focus) are identical to
  `SpatialAudioEngine.swift` and exposed in `CONFIG` for tuning by ear.
- The browser's generic HRTF differs slightly from Apple's renderer; the glass
  UI uses backdrop blur (no Liquid Glass on the web).

## Known limits

- The four tracks are decoded to raw PCM for sample-locked playback
  (~90 MB per track in RAM). Fine on laptops and recent phones; very old
  phones may reload the tab.
- Compass mode needs a device with a magnetometer over HTTPS; on laptops the
  segment stays selectable but reports "No Compass".
