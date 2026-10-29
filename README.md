# Liquid Glass Lab

A liquid-glass voice button rendered entirely in WebGL 2. The button refracts a live raymarched 3D scene behind it, bends toward your cursor like jelly, and glows while "listening".

Open `index.html` in any browser with WebGL 2. There's no build step and no dependencies.

## Scenes

Pick a background from the tray at the bottom:

- **Ocean**: glossy blue spheres on a deep-blue studio sweep
- **Chrome**: a floating liquid-metal metaball
- **Clay**: matte pastel primitives and a bouncing ball
- **Satin**: rolling fabric folds
- **Neon**: emissive rings over a mirror floor

## How it works

Rendering happens in two passes.

**Pass 1, the scene.** Signed distance fields are sphere-traced into an offscreen texture, with Eevee-style shading:
- soft shadows and ambient occlusion
- a studio environment with softbox and strip lights
- one reflection bounce
- wrap lighting for clay and sheen for satin
- ACES tone mapping

The render scale adapts automatically to hold a smooth frame rate.

**Pass 2, the glass.** The button is a 2D signed distance field: a pill smooth-unioned with a bulge that springs toward the pointer. Inside that shape:
- **Refraction**: a circular bevel pushes samples along the edge normal, so the rim magnifies and wraps the scene.
- **Dispersion**: red, green and blue are sampled at slightly different offsets.
- **Frost and bloom**: both come from the scene texture's mipmap chain.
- **Lighting**: specular highlights from a 3D normal derived from the bevel slope, a hairline rim and a fresnel band.
- **Voice glow**: driven by an audio level.

## Using a real microphone

The demo simulates voice input in `fakeVoice()`. To drive the glow with a real mic, compute an RMS level from an `AnalyserNode` and assign it to `level` each frame:

```js
const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
const ctx = new AudioContext();
const analyser = ctx.createAnalyser();
ctx.createMediaStreamSource(stream).connect(analyser);
const buf = new Float32Array(analyser.fftSize);
function micLevel() {
  analyser.getFloatTimeDomainData(buf);
  let sum = 0; for (const v of buf) sum += v * v;
  return Math.min(1, Math.sqrt(sum / buf.length) * 6);
}
```

## Controls

| Slider | What it does |
|---|---|
| Refraction | How far the rim bends the scene |
| Frost | Blur inside the glass |
| Dispersion | RGB split at the edge |
| Bend | How strongly the glass reaches toward the cursor |
| Bloom | Glow around bright highlights |
| Exposure | Scene brightness before tone mapping |
