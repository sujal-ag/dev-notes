

    # 🧠 How Transformers.js Works — Under the Hood

> A beginner-friendly breakdown of every concept involved when you run an AI model in the browser.
> Written for people who use ML without training models — because ML is way more than just training.

---

## Table of Contents

1. [The Big Picture](#the-big-picture)
2. [GPU — The Hardware](#gpu--the-hardware)
3. [WebGL — The Old Way](#webgl--the-old-way)
4. [WebGPU — The Modern Way](#webgpu--the-modern-way)
5. [Metal, Vulkan, DirectX](#metal-vulkan-directx)
6. [WebAssembly (WASM)](#webassembly-wasm)
7. [ONNX — The Universal Format](#onnx--the-universal-format)
8. [ONNX Runtime — The Engine](#onnx-runtime--the-engine)
9. [Quantization](#quantization)
10. [Tokenizer](#tokenizer)
11. [Embeddings](#embeddings)
12. [WebNN](#webnn)
13. [Transformers.js — Everything Together](#transformersjs--everything-together)
14. [Glossary](#glossary)

---

## The Big Picture

You write this:

```js
const pipe = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');
const result = await pipe("hello world");
```

Here's every layer that fires under the hood:

```
Your Code
    ↓
Transformers.js       — downloads model, runs tokenizer
    ↓
ONNX Runtime          — executes the model
    ↓
WebGPU / WASM         — uses GPU or CPU
    ↓
Metal / DX12 / Vulkan — OS talks to actual GPU hardware
    ↓
Your GPU chip
```

You only see the top. Everything below is automatic.

---

## GPU — The Hardware

Your computer has two processors. The CPU has 4–16 cores — very smart, handles complex tasks one at a time. The GPU has thousands of cores — simple, but runs the same operation on thousands of values **simultaneously**.

For AI, this matters enormously. Neural networks are basically huge matrix multiplications. Multiplying two 1000×1000 matrices = 1 billion multiply-add operations. On a CPU that's sequential and slow. On a GPU, all 1 billion happen in parallel.

> CPU = 4 smart professors solving problems one by one  
> GPU = 10,000 workers all doing the same simple job at the same time

---

## WebGL — The Old Way

WebGL (2011) was the browser's first way to access the GPU. But it was designed purely for **graphics** — rendering triangles and pixels on screen. People hacked it for ML by disguising matrix math as texture operations. It worked, but it was painful:

- No real memory management
- Shader language (GLSL) had no concept of general compute
- Debugging was nearly impossible
- Built on OpenGL ES 2.0, which was already old

Basically the wrong tool being forced to do the right job.

---

## WebGPU — The Modern Way

WebGPU (2023) was built from scratch for both graphics and general compute. The key idea is that your CPU and GPU are **separate processors with separate memory**. WebGPU lets you explicitly manage that boundary:
You're in a browser now. The browser runs on Windows, Mac, Linux, phones — everywhere.
The browser can't use DirectX on Mac or Metal on Windows. So it built one single API — WebGPU — that your JS code talks to, and the browser figures out which translator to use underneath:
```
Your JS
  │
  ▼
WebGPU   ← you only deal with this
  │
  ├── on Windows → talks to DirectX
  ├── on Mac     → talks to Metal
  └── on Linux   → talks to Vulkan
```
You write code once, it works everywhere. The browser does the platform-specific translation.

---

## Metal, Vulkan, DirectX

WebGPU doesn't talk to GPU hardware directly — it goes through OS-level translators. These are just libraries different companies built to talk to their GPU drivers:

- **DirectX 12** — Microsoft, Windows only
- **Metal** — Apple, macOS/iOS only
- **Vulkan** — open source, Windows/Linux/Android

WebGPU wraps all three:

```
WebGPU API  ← you only touch this
    │
    ├── macOS/iOS   → Metal
    ├── Windows     → DirectX 12
    └── Linux/Android → Vulkan
```

Write WebGPU once, the browser handles which translator to use. You never think about Metal or Vulkan.

---

## WebAssembly (WASM)

Browsers only run JavaScript. But ONNX Runtime is written in C++, which is 5–20x faster for number-heavy computation.

WASM solves this: take C++ code, compile it to a binary format browsers understand, run it at near-native speed. Same logic, different language — like translating a book from French to English, same content runs in a new environment.

This is how a C++ ML engine (ONNX Runtime) runs inside your browser without being rewritten in JS.

---

## ONNX — The Universal Format

Every AI framework saves models in its own format — PyTorch uses `.pt`, TensorFlow uses SavedModel, etc. None of them run in a browser or on each other directly.

ONNX is a standardized file format for AI models. Any framework can export to `.onnx`, and anything that supports ONNX can run it:

```
Train in PyTorch
    │
    │ torch.onnx.export()
    ▼
model.onnx
    │
    ├── Browser (Transformers.js)
    ├── Mobile (ONNX Runtime Mobile)
    ├── Server (ONNX Runtime)
    └── Edge devices
```

Think of it like `.mp4` — any player that supports the format can play it. Train once, run anywhere.

---

## ONNX Runtime — The Engine

ONNX is the file. ONNX Runtime is what actually **executes** it. It reads the `.onnx` file, figures out the best backend available, and runs the model:

- WASM (CPU) — always available, works everywhere
- WebGPU (GPU) — faster, available on modern browsers
- WebNN (NPU) — fastest, only if device has a dedicated AI chip

Made by Microsoft. Used in cloud, mobile, browser, and edge devices. In the browser it's called **ONNX Runtime Web** — ONNX Runtime compiled to WASM with optional WebGPU acceleration.

---

## Quantization

Every weight in a trained model is stored as a **32-bit float (fp32)** — 4 bytes per number. `all-MiniLM-L6-v2` has ~22 million weights, so fp32 = ~88 MB. Too heavy for a browser.

Quantization reduces the precision of those numbers to shrink the model:

| Format | Bytes/weight | Size of all-MiniLM |
|--------|-------------|-------------------|
| fp32 | 4 | ~88 MB |
| fp16 | 2 | ~44 MB |
| int8 | 1 | ~22 MB ← Transformers.js default |
| int4 | 0.5 | ~11 MB |

The quality loss for embedding tasks is tiny — the model still understands language almost identically, just using less precise numbers. This is why a real AI model loads in a browser in seconds.

---

## Tokenizer

Models only understand numbers, not text. A tokenizer converts text → numbers.

It doesn't split word by word. It uses **sub-word tokenization** — breaking words into known pieces:

```
"playing football"
    ↓
["play", "##ing", "foot", "##ball"]
    ↓
[101, 2377, 1076, 2519, 3608, 102]
```

The `101` and `102` are special start/end tokens the model expects. The `##` prefix means "this piece is attached to the previous word."

This lets the model handle rare or unseen words by breaking them into familiar sub-parts. Every model has its own tokenizer trained with it — using the wrong tokenizer gives garbage output. Transformers.js loads the matching one automatically.

---

## Embeddings

When you pass text through an embedding model, you get back a **vector** — a list of numbers representing the *meaning* of that text.

```
"cat"    → [0.21, -0.54, 0.88, ...]   (384 numbers)
"dog"    → [0.19, -0.51, 0.85, ...]   (384 numbers)
"car"    → [-0.62, 0.33, -0.11, ...]  (384 numbers)
```

The magic: **similar meaning = similar numbers = vectors close together in space**. Cat and dog are close. Car is far from both. The model learned this from reading billions of sentences.

This is exactly how SemanticFind works — you embed a query, embed all PDF chunks, and find the chunks whose vectors are closest to the query. No keyword matching needed, pure meaning-based search.

---

## WebNN

WebNN (Web Neural Network API) is the newest browser API — instead of going to the GPU via WebGPU, it talks to the OS's **dedicated AI chip (NPU)**:

- Apple Neural Engine (M1/M2/M3)
- Qualcomm Hexagon (Android)
- Intel NPU (newer laptops)

These chips are purpose-built for neural net inference — even more power-efficient than GPU. WebNN sits alongside WebGPU as an option, and ONNX Runtime picks whichever is fastest on the current device.

---

## Transformers.js — Everything Together

Here's the exact sequence when you call `pipeline()` in your SemanticFind project:

**First call (cold start):**
1. Downloads `model_quantized.onnx` (~22 MB, int8) from HuggingFace CDN
2. Downloads `tokenizer.json` and config files
3. Caches everything in browser IndexedDB — instant on next load

**Every call after that:**
1. Tokenizer converts your text → token ID array
2. ONNX Runtime loads the `.onnx` graph
3. Checks for WebGPU → uses it if available, falls back to WASM
4. Runs the transformer layers (attention, FFN, layernorm...) on GPU/CPU
5. Mean pooling on the output → 384-dim float vector
6. You get back `Float32Array([0.21, -0.54, 0.88, ...])`

In your Node.js backend (SemanticFind), it runs on WASM (CPU) since there's no browser GPU. If you ever move inference to the browser, it'd automatically pick up WebGPU — same code, zero changes.

---

## Glossary

| Term | One Line |
|------|----------|
| **GPU** | Thousands of simple cores — parallel math machine |
| **CPU** | Main processor — smart but sequential |
| **WebGL** | Old browser GPU API, made for graphics, hacked for ML |
| **WebGPU** | Modern browser GPU API, built for graphics + compute |
| **Metal / DirectX / Vulkan** | OS-level GPU translators — WebGPU uses these under the hood |
| **WebNN** | Browser API for dedicated AI chips (NPU) |
| **WASM** | Run C++ code in the browser at near-native speed |
| **ONNX** | Universal file format for AI models |
| **ONNX Runtime** | Engine that reads and executes .onnx files |
| **Quantization** | Shrink model size by reducing number precision |
| **Tokenizer** | Converts text → numbers the model understands |
| **Embedding / Vector** | List of numbers representing the meaning of text |
| **Vector DB** | Database that stores + searches embeddings (e.g. ChromaDB) |
| **Transformers.js** | JS library wrapping all of the above into 2 lines |
| **Abstraction** | Hiding complexity behind a simple interface — core of engineering |

---

*Written while building SemanticFind — a semantic PDF search engine using Transformers.js + ChromaDB + React + Node.js.*