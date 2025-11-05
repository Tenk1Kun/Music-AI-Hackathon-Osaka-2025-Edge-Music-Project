<a id="readme-top"></a>

<!-- PROJECT SHIELDS -->
![Stars](https://img.shields.io/github/stars/Tenk1Kun/Music-AI-Hackathon-Osaka-2025-Music-Project?style=for-the-badge)
![Issues](https://img.shields.io/github/issues/Tenk1Kun/Music-AI-Hackathon-Osaka-2025-Music-Project?style=for-the-badge)
![License](https://img.shields.io/github/license/Tenk1Kun/Music-AI-Hackathon-Osaka-2025-Music-Project?style=for-the-badge)

<br/>

<!-- PROJECT LOGO -->
<div align="center">
  <img src="./logo.png" alt="Logo" width="180">
</div>

<h1 align="center">Edge-To-Music AI</h1>

<p align="center">
Transform architecture into sound.<br/>
Upload a building → AI guesses cultural style → edges become melody → browser performs it in real-time.<br/><br/>
2nd Place • AI + Music Hackathon (Osaka, 2025)
<br/><br/>
<a href="https://hackathon-2025-edge-music.vercel.app/"><strong>▶ Live Demo</strong></a> &nbsp;·&nbsp;
<a href="https://github.com/Tenk1Kun/Music-AI-Hackathon-Osaka-2025-Music-Project">Source Code</a>
</p>

---

### 🧠 **What this project does**

This system turns architectural form into monophonic music:

- **Upload an image**
- **Classify Japan vs Austria style** (temple vs chalet)
- **Extract contours** using Canny (OpenCV.js WASM)
- **Map image axes → musical dimensions**
  - `y` → **pitch**
  - `x` → **onset timing**
  - edge magnitude → **velocity**
- **Play in-browser** (Tone.js Transport)
- **Zero backend — privacy-safe, low latency**

> Architecture is *frozen music.*  
> Here, music melts architecture back into sound.

---

## 🖼️ **Example Output**

<div align="center">
<img src="./screenshotmusicai.png" width="450"/>
<br/>
<i>Temple edges → koto line, 96.4% confidence</i>
</div>

---

## 🧩 **How It Works**

### 1. Load + normalize image (TensorFlow.js)

```ts
// loadModel.ts
const model = await tf.loadLayersModel("/model/model.json")
export async function classify(img) {
  return tf.tidy(() => model.predict(preprocess(img)))
}
We pre-normalize inside the loader so inference is instant during playback — no GC stalls.

2. Edge extraction (OpenCV.js WASM)
ts
Copy code
// cannyConverter.ts
cv.cvtColor(src, src, cv.COLOR_RGBA2GRAY)
cv.GaussianBlur(src, src, new cv.Size(5,5), 0)
cv.Canny(src, edges, 50, 150)
Edges become (x, y, magnitude) points.
Magnitude drives note velocity when available.

3. Band slicing → musical lanes
ts
Copy code
// edgeToEvents.ts
for (let band = 0; band < rows; band++) {
  const pts = filterRow(edges, band)
  const lane = thinHorizontal(pts)
  events.push(mapToMusic(lane))
}
We mimic staff lines: thin each band to preserve gesture, avoid density walls.

4. Map pixels → pitches & time
ts
Copy code
// toneUtil.ts
const pitch = mapYToScale(yNorm, scale)   // culture-dependent scale
const onset = xNorm * measureLength
Japanese → koto, ~100BPM, pentatonic

Austrian → piano, ~90BPM, European major/minor flavor

5. Schedule audio (Tone.js Transport)
ts
Copy code
Tone.Transport.schedule(time => {
  synth.triggerAttackRelease(note, dur, time, vel)
}, onset)
Tone.Transport.start()
All audio runs locally — no server, no streaming.

✨ Why This Matters
This project explores:

Cultural architecture features → musical language

Edge geometry as gesture

Real-time browser audio AI, zero backend

Human-centered sonification

It was built in 24 hours at an international hackathon,
as the only high-school competitor in a field of adults.

🛠️ Built With
Next.js + TypeScript — UI + routing

TensorFlow.js — style classifier

OpenCV.js (WASM) — edge detection

Tone.js — musical engine + scheduling

Vercel — hosting

🎵 Sample Pipeline (ASCII)
mathematica
Copy code
Image → Preprocess → Style Classifier → Edge Detector
        └──────────────────────────────┐
Pitch Map ← y-axis              x-axis → Rhythm Grid
Velocity ← gradient magnitude
                    ↓
              Tone.Transport
                    ↓
                Audio Output
🌍 Who Uses/Studies This
Computational creativity researchers

AI-music artists

Architecture & design students

Sonification & HCI explorers

Hackathon + WebAudio community

🙏 Credits
Core dev: Koya Takemura

Vision inspiration: Leon Kattendick
<sub>https://github.com/LeonKattendick/</sub>

📄 License
MIT — free to modify & explore.

<p align="right">(<a href="#readme-top">back to top</a>)</p> ```
