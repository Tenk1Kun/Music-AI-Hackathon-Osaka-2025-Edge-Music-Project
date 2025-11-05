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
