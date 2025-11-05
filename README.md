<br/> <div align="center"> <img src="./logo.png" alt="Edge-To-Music AI Logo" width="180"> </div>

<h1 align="center">Edge-To-Music AI</h1>

<p align="center">
  Transform architecture into sound.<br/>
  Upload a building → AI guesses cultural style → edges become melody → browser performs it in real time.<br/><br/>
  2nd Place • AI + Music Hackathon (Osaka, 2025) <br/><br/>
  <a href="https://hackathon-2025-edge-music.vercel.app/"><strong>▶ WEB LINK TO DEMONSTRATION</strong></a>
  &nbsp;·&nbsp;
  <a href="https://github.com/Tenk1Kun/Music-AI-Hackathon-Osaka-2025-Music-Project">Source Code</a>
</p>

---

## 🧠 What this project does

This system turns architectural form into a clear, monophonic musical line:

- Upload an image  
- Classify Japan vs Austria style (temple vs chalet)  
- Extract contours using Canny (OpenCV.js / WASM)  
- Map image axes → musical dimensions  
  - **y → pitch**  
  - **x → onset timing**  
  - **edge magnitude → velocity**  
- Play in-browser (Tone.js Transport)  
- Zero backend — privacy-safe, low latency  

> “Architecture is frozen music.” Here, music melts architecture back into sound.

---

## 🖼️ Example Output

<div align="center">
  <img src="./screenshot.png" width="450" alt="Demo screenshot"/><br/>
  <i>Temple edges → koto line, 96.4% confidence</i>
</div>

---

## 🧩 How It Works — Script Walkthrough

Follow the “script” below to understand the full pipeline. Each step mirrors a real module in the repo.

### 1) Load & Normalize (TensorFlow.js)

```ts
// loadModel.ts
import * as tf from '@tensorflow/tfjs';

export const model = await tf.loadLayersModel('/model/model.json');

export function preprocess(img: HTMLImageElement | ImageData | HTMLCanvasElement) {
  // Example: resize → toFloat → /255 → expandDims
  return tf.tidy(() => {
    const tensor = tf.browser.fromPixels(img as HTMLCanvasElement)
      .resizeBilinear([224, 224])
      .toFloat()
      .div(255);
    return tensor.expandDims(0); // [1,224,224,3]
  });
}

export async function classify(img: HTMLImageElement | ImageData | HTMLCanvasElement) {
  return tf.tidy(() => model.predict(preprocess(img)) as tf.Tensor);
}

```

### 2) Edge Extraction (OpenCV.js / WASM)

```ts
// cannyConverter.ts
// Input: RGBA canvas → Output: edges Mat (CV_8U) and optional gradient magnitude
export function detectEdges(src: cv.Mat) {
  const gray = new cv.Mat();
  const edges = new cv.Mat();
  const gradX = new cv.Mat(); const gradY = new cv.Mat(); const mag = new cv.Mat();

  cv.cvtColor(src, gray, cv.COLOR_RGBA2GRAY);
  cv.GaussianBlur(gray, gray, new cv.Size(5, 5), 0);
  cv.Canny(gray, edges, 50, 150);

  // Optional: velocity from gradient magnitude
  cv.Sobel(gray, gradX, cv.CV_32F, 1, 0);
  cv.Sobel(gray, gradY, cv.CV_32F, 0, 1);
  cv.magnitude(gradX, gradY, mag); // CV_32F

  gray.delete(); gradX.delete(); gradY.delete();
  return { edges, magnitude: mag }; // caller disposes
}
```

Edges become (x, y, magnitude) points. Magnitude maps to velocity.

### 3) Band Slicing → Monophonic Lane

```ts
// edgeToEvents.ts
// Scan in thin horizontal bands (staff analogy). Thin horizontally to avoid clusters.
export function edgesToEvents(edges: cv.Mat, magnitude?: cv.Mat, bands = 24) {
  const W = edges.cols, H = edges.rows;
  const events: { xNorm: number; yNorm: number; vel: number }[] = [];

  for (let b = 0; b < bands; b++) {
    const y0 = Math.floor((b    / bands) * H);
    const y1 = Math.floor(((b+1)/ bands) * H);

    for (let y = y0; y < y1; y++) {
      for (let x = 0; x < W; x++) {
        if (edges.ucharPtr(y, x)[0] > 0) {
          const xNorm = x / W;
          const yNorm = y / H;
          const vel   = magnitude ? Math.min(1, magnitude.floatAt(y, x) / 255) : 0.8;
          events.push({ xNorm, yNorm, vel });
          x += 2; // horizontal thinning
        }
      }
    }
  }
  return events;
}
```
### 4) Visual → Musical Mapping
```ts
// toneUtil.ts
export function mapYToScale(yNorm: number, scale: string[]) {
  const idx = Math.max(0, Math.min(scale.length - 1,
      Math.round((1 - yNorm) * (scale.length - 1))));
  return scale[idx];
}

export function mapEvent(
  e: { xNorm: number; yNorm: number; vel: number },
  scale: string[],
  totalBars = 4
) {
  const note  = mapYToScale(e.yNorm, scale);
  const onset = e.xNorm * (totalBars * Tone.Time('1m').toSeconds());
  const vel   = e.vel;
  return { note, onset, vel, dur: '8n' as const };
}
```
Style presets:

Japanese → koto • ~100 BPM • pentatonic

Austrian → piano • ~90 BPM • harmonic/minor palette

### 5) Scheduling & Playback (Tone.js Transport)

```ts
// page.tsx (excerpt)
Tone.Transport.bpm.value = cfg.bpm;

const synth = cfg.instrument === 'koto'
  ? new Tone.PluckSynth({ dampening: 2400, release: 2 }).toDestination()
  : new Tone.Sampler({ urls: { C4: 'piano-C4.mp3' } }).toDestination();

mappedEvents.forEach(({ note, onset, vel, dur }) => {
  Tone.Transport.schedule(time => {
    synth.triggerAttackRelease(note, dur, time, vel);
  }, onset);
});

await Tone.start();
Tone.Transport.start('+0.1');
```
## ✨ Why This Matters

- Cultural architecture features → musical language  
- Edge geometry as gesture  
- Real-time browser audio AI, zero backend  
- Human-centered sonification  
- Built in 24 hours at an international hackathon — the only high-school competitor among adults.

---

## 🛠️ Built With

- **Next.js + TypeScript** — UI & routing  
- **TensorFlow.js** — style classifier  
- **OpenCV.js (WASM)** — edge detection  
- **Tone.js** — musical engine & scheduling  
- **Vercel** — hosting

---

## 🎵 Pipeline (ASCII)


Image → Preprocess → Style Classifier → Edge Detector
        └──────────────────────────────┐
Pitch Map ← y-axis              x-axis → Rhythm Grid
Velocity ← gradient magnitude
                    ↓
              Tone.Transport
                    ↓
                Audio Output

## 🌍 Who Uses/Studies This

- Computational creativity researchers  
- AI-music artists  
- Architecture & design students  
- Sonification & HCI explorers  
- Hackathon & WebAudio community

---

## 🙏 Credits

- **Core dev:** Koya Takemura  
- **Vision inspiration:** Leon Kattendick — [https://github.com/LeonKattendick/](https://github.com/LeonKattendick/)

---

## 📄 License

MIT — free to modify & explore.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

