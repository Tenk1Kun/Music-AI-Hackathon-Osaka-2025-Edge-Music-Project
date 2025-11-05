Edge-To-Music AI
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/23/Sinus_3d.gif/120px-Sinus_3d.gif" width="120" align="right">

Edge-To-Music AI turns building geometry into sound.
Upload architecture → detect structural edges → convert lines into musical events → hear form become melody.

Where architecture becomes score, and structure becomes rhythm.

Spatial → musical mapping (edges → notes, angles → timing)

Real-time web inference (TensorFlow.js + WebAssembly OpenCV)

Tone.js synthesis engine with transport scheduling

Deterministic + expressive: lines produce repeatable music, but phrasing adapts

Works entirely in the browser — privacy-safe, zero backend

“Architecture is frozen music. This project melts it back into sound.”

<p align="center"> <a href="https://hackathon-2025-edge-music.vercel.app/"><b>Live Demo</b></a> </p> <p align="center"> <img src="https://i.imgur.com/RcSv8ie.jpeg" alt="Demo screenshot" width="700"> </p>
🎧 What This Does

You upload a building image — temple, museum, tower, street.
The system:

detects edges

finds angle + frequency relationships

assigns pitch + rhythm based on geometry

plays an emergent melody from the structure itself

The result feels like architectural mood translated into sound — rigid, flowing, symmetrical, fragmented, etc.

🧠 How It Works
1. Convert photo to edge space

OpenCV Canny finds structural lines only:

const img = cv.imread(canvas)
cv.cvtColor(img, img, cv.COLOR_RGBA2GRAY)
cv.GaussianBlur(img, img, new cv.Size(5,5), 0)
cv.Canny(img, edges, 50, 150)


Output → clean edge map, revealing load-bearing form + symmetry.

2. Extract musical event grid

Every detected point becomes a candidate note:

edges.forEach((x, y) => events.push({
  xNorm: x / width,
  yNorm: y / height,
  angle: getAngleGradient(x, y, edges)
}))


We treat the architecture as a score already written.

3. Map geometry → musical parameters
Architectural feature	Musical translation
Vertical line	Sustained tone
Diagonal beam	Ascending/descending gliss
Horizontal beams	Stable bass tones
Dense repetition	Trills / arpeggiation
Symmetry	Polyphonic unison moments

Pitch from verticality:

const pitch = scale[Math.floor((1 - yNorm) * scale.length)]


Time from x-coordinate:

const onset = xNorm * totalDuration


Angle influences articulation + velocity.

4. Synthesize sound & play transport

Tone.js generative playback:

Tone.Transport.schedule(time => {
  synth.triggerAttackRelease(pitch, dur, time, velocity)
}, onset)


Each building literally sings its shape.

🌐 Live Demo

Try it here:
https://hackathon-2025-edge-music.vercel.app/

Upload a building → press Play → listen to architecture.

Works best on:

shrines & temples

brutalist geometry

glass grids & arcs

bridges, towers, arches

🧩 Tech Stack
Layer	Technology
Edge detection	WebAssembly OpenCV
ML routing	TensorFlow.js
Synthesis	Tone.js
Frontend	Next.js / React
Runtime	Vercel / Web Audio API
🧪 Who This Is For

Computational arts researchers

Music technologists

Creative coders (p5 / WebAudio / Max/MSP crowd)

Hackathon AI builders

Architects curious about sonic form

🗺️ Roadmap

🎥 Live camera → real-time “singing architecture”

🎶 Timbre palette selection (organ / koto / pads / brass)

🧮 CLIP aesthetic embeddings → dynamic emotional tuning

🫁 Breath phrasing + rubato humanization

🏛️ 3D mesh → harmonic space mapping (BIM sonification)

🙏 Acknowledgments

TensorFlow.js

OpenCV WASM build

Tone.js

Vercel

Hackathon community & mentors

homage to Goethe / Kandinsky — and every kid who ever looked at a building and wondered if it had music inside it

📎 Footer

If this inspires you, remix it.
If you expand it, share the sound.

Architecture has always been music —
we finally just let it sing.
