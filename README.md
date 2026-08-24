# Meowmeow cat cam meme detector

Point your webcam at yourself, make a face/hand gesture, get a cat meme back in real time. Runs either as a desktop app (OpenCV windows) or entirely in the browser (MediaPipe WASM, no install).

Two windows/panes side by side: 
- **Camera** — your webcam feed with hand landmarks drawn on top, plus a live debug readout in the corner
- **Meme** — the meme matching whatever gesture you're currently making

## Gestures

Checked in this order — when a pose could match more than one, the earlier one wins.

| # | Gesture | How to trigger |
|---|---|---|
| 1 | Spinny OIIAI cat | You spin!!!! (beats everything, hands included) |
| 2 | Screaming cat | Mouth wide open (beats any hand pose) |
| 3 | Muehehe | Both hands up, index fingers only, tips touching |
| 4 | Devo cat | Both hands up, above the top of your head |
| 5 | Crash out cord chewing kitty | Both hands up beside your face to hold yummy electrical cable |
| 6 | I will punch you | One hand, all four fingers curled |
| 7 | EHHEHEEEHEEEE | Thumb + pinky out, rockstar cat |
| 8 | Shhh silenced cat | Index finger only, tip resting on your mouth |
| 9 | Erm ackshuALLY! cat | Index finger only, held away from your face |
| 10 | Shocked/kidnapped cat | Hand cover mouth |
| 11 | Hi! | Wave an open hand side to side (animated GIF) |
| 12 | gGIMME MONIE!! | One open palm, all fingers extended, held still, away from your face |
| 13 | Side eye cat | Turn your head 15°+ either way (real head-pose yaw) |
| 14 | Huh? cat | Tilt your head 12°+ sideways (head-pose roll) |
| 15 | Pokercat | Default |

Meme images live in `memes/`. When a gesture has several images, it shows one
per pose and moves to the next once you drop back to no gesture at all — so
holding a pose never makes the image flicker, and repeating the pose walks
through the whole set.

A gesture whose image isn't in `memes/` is reported at startup and simply
never fires, so a gesture can be wired up before its artwork exists. Screaming
cat and Huh? cat are in this state — they detect and show up in the readout,
but need `memes/screaming cat.jpg` and `memes/huh cat.jpg` added to display.

## Running it — desktop (Python)

Requires Python 3 and a webcam.

Easiest way: just double-click **`Launch Gesture Meme.command`**. First run takes a minute to set itself up (installs everything automatically), then launches straight away. Every run after that is instant.

**First time opening it:** macOS will warn "cannot be opened because it is from an unidentified developer" — this is normal for any downloaded script, not specific to this one. Right-click the file → **Open** → click **Open** in the dialog that appears. You only need to do this once.

Or manually, if you prefer Terminal:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python3 gesture_meme.py
```

Press `q` or `Esc` in the Camera window to quit.

### Recording a session

`--record` saves both windows, side by side, to one video for as long as the
program runs:

```bash
.venv/bin/python3 gesture_meme.py --record
```

That writes a timestamped file into `recordings/` (git-ignored). Pass a
filename to choose where it goes instead:

```bash
.venv/bin/python3 gesture_meme.py --record demo.mp4
```

The meme pane changes width from meme to meme, but a video file needs one
fixed size, so the recording letterboxes each meme into a pane wide enough
for the widest one. Frames are paced against the clock rather than written
one per loop iteration, so the result plays back at real speed regardless of
how fast the machine was running.

## Running it — browser

No install needed, but the webcam API requires serving over HTTP (opening `index.html` directly as a `file://` URL will not get camera permission). From this folder:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000` and allow camera access. Models load from Google's hosted MediaPipe CDN at runtime, so nothing local is needed for the browser version.

## Live debug HUD

The Camera window always shows a compact readout in the top-left corner. Each
line after the gesture name is `value/threshold` for one of the signals that
drive a gesture:

```
sideEyeCat
yaw  +18.4/15
roll  -2.1/12
mouth 0.01/0.15
hands 0 pt 0 gap 0.00/1.40
wave 0/2
flow 0.02/0.80
spin 0.00/0.55  pk 0.00
```

`yaw` is head turn (side eye), `roll` is head tilt (huh?), `mouth` is lip gap
over face height (screaming), `hands`/`pt`/`gap` are the hand count, how many
hands read as pointing, and the gap between index tips (muehehe), `wave`
counts how many times a waving palm has changed direction, and
`flow`/`spin` are the optical-flow measures behind spin detection.

Useful for tuning the detection thresholds at the top of `gesture_meme.py` /
`app.js` if a gesture is triggering too easily or not easily enough for your
setup/lighting. Every run also writes `flow_debug_log.csv` with all of these
per frame, which is the easier way to tune a threshold: record yourself doing
the gesture, then look at what the numbers actually reach.

## Project layout

```
gesture_meme.py   desktop version (OpenCV + MediaPipe Python tasks API)
app.js            browser version (MediaPipe tasks-vision WASM)
index.html        browser UI shell
memes/            meme images, plus the spin cat video
models/           MediaPipe .task model files used by the desktop version
requirements.txt  Python dependencies
flow_debug_log.csv  per-frame signal log, rewritten on every desktop run
```

The two versions have drifted: the browser version does not have spin cat,
screaming cat or huh? cat, and does not have the desktop version's threaded
capture.
