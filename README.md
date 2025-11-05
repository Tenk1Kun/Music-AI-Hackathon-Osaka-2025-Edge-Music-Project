// app/ArchitectureMusicDemo.tsx
// Orchestration layer: upload → classify → edges → events → play/stop
// Assumes these utils exist per your repo:
//   util/preprocessImage.ts   -> preprocessImage(file: File): Promise<tf.Tensor4D>
//   util/loadModel.ts         -> loadModel(): Promise<tf.LayersModel>
//   util/cannyConverter.ts    -> extractEdges(cv, imgCanvas, edgeCanvas): Array<[x,y,mag?]>
//   util/edgeToEvents.ts      -> edgePointsToEvents(points, opts): Event[]
//   util/toneUtil.ts          -> initAudio(style: "japan" | "austria"), playEvents(events), stopMusic()

"use client";

import React, { useCallback, useMemo, useRef, useState } from "react";
import * as tf from "@tensorflow/tfjs";
import { preprocessImage } from "@/util/preprocessImage";
import { loadModel } from "@/util/loadModel";
import { extractEdges } from "@/util/cannyConverter";
import { edgePointsToEvents } from "@/util/edgeToEvents";
import { initAudio, playEvents, stopMusic } from "@/util/toneUtil";

// Optional: tiny helpers for musical scales (guards if your util already has it)
const KUMOI = ["C4", "D♭4", "F4", "G4", "A♭4", "C5", "D♭5", "F5", "G5", "A♭5"];
const HARMONIC_MINOR = ["A3", "B3", "C4", "D4", "E4", "F4", "G#4", "A4", "B4", "C5"];

type StyleCfg = {
  style: "japan" | "austria";
  scale: string[];
  bpm: number;
  instrument: "koto" | "piano";
};

function chooseStyleFromProbs(japanProb: number, austriaProb: number): StyleCfg {
  if (japanProb >= austriaProb) {
    return { style: "japan", scale: KUMOI, bpm: 100, instrument: "koto" };
    // If you expose instrument choice in toneUtil, pass it along; otherwise bpm/style is enough.
  }
  return { style: "austria", scale: HARMONIC_MINOR, bpm: 90, instrument: "piano" };
}

export default function ArchitectureMusicDemo() {
  const [loading, setLoading] = useState(false);
  const [styleCfg, setStyleCfg] = useState<StyleCfg | null>(null);
  const [probs, setProbs] = useState<{ japan: number; austria: number } | null>(null);
  const [eventsCount, setEventsCount] = useState<number>(0);
  const [status, setStatus] = useState<string>("Idle");
  const [error, setError] = useState<string | null>(null);

  // Canvases: original image and edges
  const imgCanvasRef = useRef<HTMLCanvasElement | null>(null);
  const edgeCanvasRef = useRef<HTMLCanvasElement | null>(null);
  const imgElRef = useRef<HTMLImageElement | null>(null);

  // Cache TF model
  const modelPromise = useMemo(() => loadModel(), []);

  // Load a file into the image canvas
  const drawImageToCanvas = useCallback(async (file: File) => {
    const img = new Image();
    imgElRef.current = img;

    const url = URL.createObjectURL(file);
    img.src = url;

    await new Promise<void>((resolve, reject) => {
      img.onload = () => resolve();
      img.onerror = (e) => reject(e);
    });

    const w = img.naturalWidth;
    const h = img.naturalHeight;
    const maxSide = 1024; // limit for speed; adjust as desired
    const scale = Math.min(1, maxSide / Math.max(w, h));
    const W = Math.max(1, Math.floor(w * scale));
    const H = Math.max(1, Math.floor(h * scale));

    const c = imgCanvasRef.current!;
    c.width = W;
    c.height = H;
    const ctx = c.getContext("2d")!;
    ctx.clearRect(0, 0, W, H);
    ctx.drawImage(img, 0, 0, W, H);

    // match edge canvas size
    const e = edgeCanvasRef.current!;
    e.width = W;
    e.height = H;

    URL.revokeObjectURL(url);
    return { W, H };
  }, []);

  const onFile = useCallback(
    async (file?: File) => {
      if (!file) return;
      setError(null);
      setStatus("Loading image…");
      setLoading(true);
      try {
        // 1) Draw image into canvas (resized)
        const { W, H } = await drawImageToCanvas(file);

        // 2) Classify style (TF.js)
        setStatus("Classifying style (TF.js)...");
        const model = await modelPromise;
        const tensor = await preprocessImage(file); // returns shape [1,224,224,3]
        const logits = model.predict(tensor) as tf.Tensor;
        const probs = await logits.data();
        tensor.dispose();
        if ((logits as any).dispose) (logits as any).dispose();

        const japanProb = probs[0] ?? 0.5;
        const austriaProb = probs[1] ?? 0.5;
        setProbs({ japan: japanProb, austria: austriaProb });
        const cfg = chooseStyleFromProbs(japanProb, austriaProb);
        setStyleCfg(cfg);

        // 3) Extract edges (OpenCV.js). Assumes global "cv" is loaded in _app or layout.
        setStatus("Extracting edges (OpenCV.js)...");
        const points = extractEdges(
          // @ts-ignore - cv comes from OpenCV.js script in page head, e.g. <script src="/opencv.js"></script>
          (window as any).cv,
          imgCanvasRef.current!,
          edgeCanvasRef.current!
        );

        // 4) Convert edge points → events (band slicing + thinning + snapping)
        setStatus("Mapping edges to musical events…");
        const events = edgePointsToEvents(points, {
          bands: 12,
          scale: cfg.scale,
          imageWidth: W,
          imageHeight: H,
          quant: "16n",
        });

        // 5) Init audio + schedule
        setStatus(`Init audio (${cfg.style}) and scheduling…`);
        await initAudio(cfg.style);
        setEventsCount(events.length);
        playEvents(events);

        setStatus(`Playing ${events.length} events @ ${cfg.bpm} BPM (${cfg.instrument})`);
      } catch (e: any) {
        console.error(e);
        setError(e?.message ?? "Unknown error");
        setStatus("Error");
      } finally {
        setLoading(false);
      }
    },
    [drawImageToCanvas, modelPromise]
  );

  const onStop = useCallback(() => {
    stopMusic();
    setStatus("Stopped");
  }, []);

  return (
    <div className="flex flex-col gap-4 p-6 max-w-5xl mx-auto">
      <header className="flex items-center justify-between">
