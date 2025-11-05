Edge-To-Music AI
<img src="images/logo.png" align="right" alt="Edge-To-Music AI Logo" width="120" height="120">

Edge-To-Music AI turns building geometry into sound — entirely in your browser.
Upload a photo → detect structural edges → map lines to musical events → listen as form becomes melody.

Zero backend — privacy-safe, everything runs locally.

WebAssembly OpenCV for fast edge detection (Canny + Sobel).

TensorFlow.js style router (e.g., Japan ⇄ Austria) to pick scale, tempo, and timbre.

Tone.js engine with transport scheduling for clean, musical playback.

Deterministic mapping (structure → notes) with expressive humanization options.

<p align="center"> <a href="https://hackathon-2025-edge-music.vercel.app/"><b>WEB LINK TO DEMONSTRATION</b></a> </p> <p align="center"> <img src="images/screenshot.png" alt="Edge-To-Music AI Demo Screenshot" width="740"> </p>

“Architecture is frozen music — this project lets it sing.”

What It Does

You upload a building image (temple, chalet, tower, bridge).

OpenCV.js extracts an edge map that reflects structure and symmetry.

TF.js classifies style → chooses musical system (scale/tempo/instrument).

Band slicing converts edges to a single, clear melodic lane.

Tone.js schedules events (onset/pitch/velocity) and renders in real time.

How It Works
1) Edge Space (OpenCV.js)

We detect high-contrast architectural boundaries with Canny (optionally leveraging Sobel magnitude for dynamics):

// Canvas -> Mat
const src = cv.imread(imageCanvas)
cv.cvtColor(src, src, cv.COLOR_RGBA2GRAY)
cv.GaussianBlur(src, src, new cv.Size(5, 5), 0)
cv.Canny(src, edgeMat, 50, 150)

// Optional: gradient magnitude for velocity mapping
cv.Sobel(src, gradX, cv.CV_64F, 1, 0)
cv.Sobel(src, gradY, cv.CV_64F, 0, 1)
const magnitude = cv.Mat.zeros(src.rows, src.cols, cv.CV_64F)
// magnitude = gradX^2 + gradY^2 (elementwise)
cv.multiply(gradX, gradX, gradX)
cv.multiply(gradY, gradY, gradY)
cv.add(gradX, gradY, magnitude)

2) Style Routing (TensorFlow.js)

A tiny TF.js classifier (e.g., Japan vs Austria) selects musical palette.
We load once and keep inputs pre-normalized for instant inference.

const model = await tf.loadLayersModel('/model.json')   // cached in browser
const input = preprocessImage(file)                      // [1, 224, 224, 3], 0..1
const logits = model.predict(input) as tf.Tensor
const [japan, austria] = (await logits.data()) as Float32Array
input.dispose(); logits.dispose()

const cfg = (japan >= austria)
  ? { style: 'japan',  scale: kumoi,          bpm: 100, instrument: 'koto' }
  : { style: 'austria',scale: harmonicMinor,  bpm:  90, instrument: 'piano' }

3) Band Slicing → Candidate Notes

We scan the edge image in thin horizontal bands (staff-line analogy), thinning dense pixels to keep the melody monophonic and clear.

const events = []
for (let band = 0; band < bands; band++) {
  const y0 = Math.floor((band    / bands) * H)
  const y1 = Math.floor(((band+1)/ bands) * H)
  for (let y = y0; y < y1; y++) {
    for (let x = 0; x < W; x++) {
      if (edgeMat.ucharPtr(y, x)[0] > 0) {
        const yNorm = y / H, xNorm = x / W
        const vel = magnitude?.doubleAt?.(y, x) ?? 1
        events.push({ xNorm, yNorm, vel })
        // horizontal thinning: skip ahead to avoid clusters
        x += 2
      }
    }
  }
}

4) Visual → Musical Mapping

y → pitch (higher in image → higher note)

x → onset (left→right → earlier→later)

magnitude → velocity (stronger edge → louder)

function mapYToPitch(yNorm: number, scale: string[]) {
  const idx = Math.max(0, Math.min(scale.length - 1,
               Math.round((1 - yNorm) * (scale.length - 1))))
  return scale[idx]
}

function mapEvent(e, scale, totalBars = 4) {
  const note  = mapYToPitch(e.yNorm, scale)
  const onset = e.xNorm * (totalBars * Tone.Time('1m').toSeconds()) // proportional in a fixed form
  const vel   = Math.min(1, e.vel / MAX_MAGNITUDE)
  return { note, onset, vel, dur: '8n' }
}

5) Scheduling & Playback (Tone.js)

We schedule monophonic, scale-snapped notes on the transport.
Tempo (BPM) and instrument depend on the style classifier’s decision.

Tone.Transport.bpm.value = cfg.bpm

const synth = cfg.instrument === 'koto'
  ? new Tone.PluckSynth({ dampening: 2400, release: 2 }).toDestination()
  : new Tone.Sampler({ urls: { C4: 'piano-C4.mp3' } }).toDestination()

mappedEvents.forEach(({ note, onset, vel, dur }) => {
  Tone.Transport.schedule(time => {
    synth.triggerAttackRelease(note, dur, time, vel)
  }, onset)
})

if (Tone.Transport.state !== 'started') {
  await Tone.start()
  Tone.Transport.start('+0.1')
}

Musical Systems
Style	Scale	Instrument	Tempo
Japan	Kumoi pentatonic	Koto	~100 BPM
Austria	Harmonic minor	Piano	~90 BPM

Principles for clarity:

Scale-snapping and lane-walk thinning avoid cluster noise.

Monophonic line showcases contour (counterpoint is future work).

Subtle reverb/delay adds space without washing articulation.

Tech Stack

Frontend: Next.js / React / TypeScript

Vision: OpenCV.js (WASM), Canny + Sobel

ML: TensorFlow.js (browser inference)

Audio: Tone.js (Web Audio API)

Hosting: Vercel (static assets + edge delivery)

Demo

Try it live:

https://hackathon-2025-edge-music.vercel.app/

Best results on buildings with clear edges and distinct geometry (shrines, chalets, brutalism, bridges, glass grids).

<p align="center"> <a href="https://evilmartians.com/?utm_source=edge-to-music"> <img src="images/sponsor-badge.svg" alt="Sponsored by ..." width="236" height="54"> </a> </p> <!-- Optional Shields (reference-style, easy to swap) -->
Notes

Replace images/logo.png, images/screenshot.png, and images/sponsor-badge.svg with your actual assets.

All code snippets are illustrative and match your pipeline (OpenCV.js → TF.js → Tone.js).
