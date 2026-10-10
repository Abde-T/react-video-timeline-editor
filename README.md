# How to Build a Video Timeline Editor in React (2026 Complete Guide)

<img src="https://public-files.gumroad.com/1l6f8xz7k4i6hys4a3n99q9ra8yo" alt="Timeline " width="">


> **The definitive, production-grade guide to building a multi-track video timeline editor in React** — from first principles to a fully interactive editing surface with drag-and-drop, snapping, real-time playback sync, chroma key, keyframe animation, and FFmpeg export.
>
> Whether you're building an AI video tool, a podcast editor, a caption generator, or a full content-creation SaaS — this is the architecture guide you wish existed.

---

<p align="center">
  <a href="https://abdet-tools.vercel.app/timeline"><strong>🔴 Live Editor Demo</strong></a> · <a href="https://www.klipflow.com/timeline"><strong>🟣 Full SaaS Example (KlipFlow)</strong></a> · <a href="https://abdetiamani.gumroad.com/l/timeline"><strong>🟢 Get the Source Code</strong></a>
</p>

---

## Table of Contents

- [Why Build a Timeline Editor?](#why-build-a-timeline-editor)
- [What You'll Learn](#what-youll-learn)
- [Architecture Overview](#architecture-overview)
- [Part 1 — The Data Model (Redux State)](#part-1--the-data-model-redux-state)
- [Part 2 — The Timeline Canvas (react-konva)](#part-2--the-timeline-canvas-react-konva)
- [Part 3 — Clips: Drag, Resize, Snap, Split](#part-3--clips-drag-resize-snap-split)
- [Part 4 — The Preview Player (HTML5 Media Sync)](#part-4--the-preview-player-html5-media-sync)
- [Part 5 — Media Library & File Ingestion](#part-5--media-library--file-ingestion)
- [Part 6 — Real-Time Effects Pipeline](#part-6--real-time-effects-pipeline)
- [Part 7 — Subtitle & Caption System](#part-7--subtitle--caption-system)
- [Part 8 — Keyframe Animation Engine](#part-8--keyframe-animation-engine)
- [Part 9 — Audio Processing (Web Audio API)](#part-9--audio-processing-web-audio-api)
- [Part 10 — Video Transition Effects](#part-10--video-transition-effects)
- [Part 11 — Exporting to FFmpeg / Backend Rendering](#part-11--exporting-to-ffmpeg--backend-rendering)
- [Part 12 — Performance Deep-Dive: 60FPS at Scale](#part-12--performance-deep-dive-60fps-at-scale)
- [Part 13 — Theming & UI System](#part-13--theming--ui-system)
- [Part 14 — Making It Production-Ready](#part-14--making-it-production-ready)
- [The Hard Truth](#the-hard-truth)
- [See It in Action](#see-it-in-action)
- [Skip Months of Development](#skip-months-of-development)

---

## Why Build a Timeline Editor?

Every serious content platform needs one:

| Use Case | Examples |
|---|---|
| **AI video generation** | all need a timeline for editing generated clips |
| **Caption / subtitle tools** | timeline for syncing text to audio |
| **Podcast editors** | multi-track audio with waveforms |
| **Social media tools** | drag-and-drop clip assembly |
| **Internal tooling** | Training video editors, marketing asset builders |

The problem? **Building one from scratch takes 3–6 months of deep engineering.** The timeline is the single most complex frontend component you'll ever build. It combines:

- Canvas-level rendering with React state management
- Frame-accurate time math with sub-pixel positioning
- Multi-element media synchronization across `<video>`, `<audio>`, and `<canvas>`
- Real-time audio processing graphs (Web Audio API)
- Collision detection, snapping, and constraint systems
- Undo/redo history across a deeply nested state tree

This guide walks through **every layer** of that complexity.

---

## What You'll Learn

By the end of this guide, you'll understand:

- ✅ How to design a normalized Redux state for tracks, clips, and media
- ✅ How to render a performant, zoomable timeline with `react-konva`
- ✅ How to implement magnetic snapping and overlap prevention
- ✅ How to synchronize multiple `<video>` and `<audio>` elements to a single playhead
- ✅ How to build a real-time chroma key (green screen) pipeline
- ✅ How to implement keyframe-based animation interpolation
- ✅ How to construct a Web Audio API signal chain with EQ, echo, and reverb
- ✅ How to parse SRT/VTT files and render animated subtitles
- ✅ How to map CSS preview values to FFmpeg filter parameters for accurate export
- ✅ How to maintain 60FPS with hundreds of clips using virtualization and batched dispatches

---

## Architecture Overview

A production timeline editor is built from four independent modules that communicate through a shared Redux store:

```
                        ┌─────────────────────────────────────────────────────────────────┐
                        │                        Timeline.tsx                             │
                        │                   (Root Orchestrator)                           │
                        │                                                                 │
                        │  ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
                        │  │  MediaHub     │  │  Player          │  │  Timeline Canvas │  │
                        │  │              │  │                  │  │                  │  │
                        │  │ • File upload │  │ • Live preview   │  │ • Konva Stage    │  │
                        │  │ • Media grid  │  │ • Media sync     │  │ • Tracks & clips │  │
                        │  │ • Drag source │  │ • Audio graph    │  │ • Drag & drop    │  │
                        │  │ • Styling     │  │ • Chroma key     │  │ • Snapping       │  │
                        │  │   panels      │  │ • Transitions    │  │ • Time ruler     │  │
                        │  │              │  │ • Subtitles      │  │ • Playhead       │  │
                        │  └──────┬───────┘  └────────┬─────────┘  └────────┬─────────┘  │
                        │         │                   │                      │            │
                        │         └───────────────────┴──────────────────────┘            │
                        │                             │                                   │
                        │                    ┌────────▼────────┐                          │
                        │                    │  Redux Store    │                          │
                        │                    │ (timelineSlice) │                          │
                        │                    └─────────────────┘                          │
                        └─────────────────────────────────────────────────────────────────┘
```

**Key principle:** The canvas never talks directly to the player. The media hub never talks directly to the canvas. Everything flows through Redux. This makes each module independently testable and replaceable.

### Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **State** | Redux Toolkit (Immer) | Normalized state with immutable updates, async thunks for thumbnail extraction |
| **Canvas** | react-konva | Hardware-accelerated 2D rendering on `<canvas>`, native hit detection |
| **Player** | HTML5 `<video>` + `<audio>` + Web Audio API | Browser-native media playback with programmable audio |
| **UI** | Radix UI + Tailwind + Framer Motion | Accessible, composable, animated |
| **Fonts** | Custom `@font-face` bundles | Offline subtitle fonts (Montserrat, Bungee, Bebas Neue, etc.) |

---

## Part 1 — The Data Model (Redux State)

The most important decision in a timeline editor is **how you model your data**. Get this wrong and every feature becomes a fight against your own state shape.

### The Core Entities

There are three primary entities: **Tracks**, **Clips**, and **Media Items**.

```typescript
// A Track is a horizontal lane (video, audio, or subtitle)
interface Track {
  id: string;
  type: "video" | "audio" | "subtitle";
  clips: string[];     // ordered array of clip IDs
  name: string;
  height: number;      // pixel height on the canvas
}

// A Clip is a positioned segment on a track
interface Clip {
  id: string;
  type: "video" | "audio" | "image" | "gif" | "subtitle" | "text"
      | "transition" | "fade" | "xfade";
  start: number;       // seconds — position on the timeline
  duration: number;    // seconds — visual length
  sourceStart?: number; // seconds — offset into the source media (for cuts)
  trackId: string;     // which track this clip lives on
  name: string;
  color: string;
  url?: string;        // blob: URL for preview
  mediaId?: string;    // reference to the source MediaItem
  volume?: number;     // 0.0 – 1.0
  zIndex?: number;     // layer order in the player canvas
  position?: { x: number; y: number };   // % position on player canvas
  size?: { width: number; height: number }; // % size on player canvas
  styling?: ClipStylingOptions;           // filters, transforms, chroma key
  keyframes?: Keyframe[];                 // animation keyframes
  thumbnails?: VideoFrame[];              // extracted preview frames
  // ... and more
}

// A MediaItem is a file in the library (not yet on the timeline)
interface MediaItem {
  id: string;
  type: "video" | "audio" | "image" | "gif" | "text" | "subtitle";
  name: string;
  url: string;         // blob: URL
  duration?: number;
  thumbnail?: string;  // base64 data URL
  subtitles?: { start: number; end: number; text: string }[];
}
```

### Why Normalize Clips as a Record?

Clips are stored as `Record<string, Clip>` (a flat map), not nested inside tracks. This is critical:

```typescript
interface TimelineState {
  tracks: Track[];                    // track metadata + ordered clip IDs
  clips: Record<string, Clip>;       // O(1) clip lookups by ID
  mediaItems: MediaItem[];           // source library
  currentTime: number;               // playhead position (seconds)
  zoom: number;                      // pixels per second
  isPlaying: boolean;
  selectedClipId: string | null;
  history: { clips: Record<string, Clip>; tracks: Track[] }[];
  historyIndex: number;
  aspectRatio: string;               // "16:9", "9:16", "4:3", "1:1"
  // ... UI state fields
}
```

**Why a flat map?**
- Moving a clip between tracks is O(1): remove ID from old track's array, push to new
- Updating a clip is O(1): direct key access, no find/filter
- The player needs to query "all clips active at time T" — a flat map makes this a simple `Object.values(clips).filter(...)`

### The Redux Store

```typescript
import { configureStore } from "@reduxjs/toolkit";
import timelineReducer from "./timelineSlice";

export const store = configureStore({
  reducer: {
    timeline: timelineReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      // Disable serializable checks — we store blob URLs and base64 thumbnails
      serializableCheck: false,
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Key Actions (Reducers)

Here's a selection of the most important actions in the slice:

| Action | Purpose |
|---|---|
| `addTrack({ type })` | Insert a new track, positioned after the last track of the same type |
| `addClip({ trackId, clip })` | Add a clip to a track with auto-generated ID and z-index |
| `moveClip({ clipId, newTrackId, newStart })` | Move a clip to a different track and/or time position |
| `updateClip({ clipId, updates })` | Partial update (position, styling, volume, etc.) |
| `removeClip(clipId)` | Delete a clip and clean up the track reference |
| `splitClip` | (custom) Split a clip at the playhead into two segments |
| `saveToHistory` / `undo` / `redo` | Snapshot-based undo system (max 50 entries) |
| `loadProject({ tracks, clips, mediaItems })` | Hydrate the entire state from a saved project |
| `setCurrentTime(seconds)` | Move the playhead |
| `setZoom(pixelsPerSecond)` | Zoom the timeline (clamped 0.1–5000) |
| `togglePlayback` | Play/pause toggle |

### The `sourceStart` Pattern

This is the key insight for non-destructive editing. When a user "cuts" a video at 5 seconds:

```
Before:  [=========== 10s clip ===========]
                     ↑ cut at 5s
After:   [=== 5s ===][=== 5s ===]
         sourceStart=0  sourceStart=5
```

Both clips reference the **same source video**. The `sourceStart` field tells the player where to seek within the original file. The `<video>` element plays from `sourceStart + localTime`.

---

## Part 2 — The Timeline Canvas (react-konva)

The timeline is rendered on an HTML5 Canvas using `react-konva`. This gives us hardware-accelerated rendering, pixel-perfect hit detection, and the ability to handle hundreds of visual elements without DOM bloat.

### Why Canvas, Not DOM?

| Approach | Clips @ 60FPS | Hit detection | Zoom/pan |
|---|---|---|---|
| **DOM divs** | ~50 clips | Manual math | CSS transforms |
| **SVG** | ~100 clips | Built-in | viewBox |
| **Canvas (Konva)** | **500+ clips** | **Built-in** | **Native stage transforms** |

At high zoom levels with waveforms and thumbnails, a DOM-based timeline buckles. Konva gives us a retained-mode canvas with React's declarative API.

### Canvas Structure

```
<Stage>                          ← Konva root, sized to container
  <Layer>                        ← Single rendering layer
    <TimeRuler />                ← Top ruler with time ticks
    {tracks.map(track => (
      <Track key={track.id}>    ← Background lane
        {track.clips.map(clipId => (
          <Clip key={clipId} />  ← Interactive clip rectangle
        ))}
      </Track>
    ))}
    <Playhead />                 ← Red vertical line at currentTime
  </Layer>
</Stage>
```

### Coordinate System

The timeline uses a `zoom` value measured in **pixels per second**:

```typescript
// Convert time to canvas X position
const timeToX = (timeInSeconds: number) => timeInSeconds * zoom;

// Convert canvas X position to time
const xToTime = (xInPixels: number) => xInPixels / zoom;

// Clip rendering
<Rect
  x={clip.start * zoom}
  width={clip.duration * zoom}
  y={trackYOffset}
  height={track.height}
/>
```

At `zoom = 50`, one second = 50 pixels. At `zoom = 200`, one second = 200 pixels (zoomed in for frame-accurate editing).

### Time Ruler

The ruler dynamically adapts its tick density based on zoom:

```typescript
// Adaptive tick calculation
const getTickInterval = (zoom: number): number => {
  if (zoom >= 200) return 0.1;   // 100ms ticks (frame-level)
  if (zoom >= 100) return 0.5;   // 500ms ticks
  if (zoom >= 50)  return 1;     // 1-second ticks
  if (zoom >= 20)  return 5;     // 5-second ticks
  if (zoom >= 5)   return 30;    // 30-second ticks
  return 60;                      // 1-minute ticks
};
```

Major ticks show timestamps (`mm:ss.d`), minor ticks provide visual reference. Double-clicking a tick seeks the playhead to that time.

### Virtualization

Only clips visible within the current scroll viewport are rendered:

```typescript
const visibleTimeRange = {
  start: scrollLeft / zoom,
  end: (scrollLeft + containerWidth) / zoom,
};

const visibleClips = allClips.filter(clip =>
  clip.start + clip.duration > visibleTimeRange.start &&
  clip.start < visibleTimeRange.end
);
```

This keeps the canvas responsive even with thousands of clips across dozens of tracks.

---

## Part 3 — Clips: Drag, Resize, Snap, Split

This is where the real complexity lives. A clip is not just a colored rectangle — it's an interactive element with:

- **Drag to move** (horizontally across time, vertically across tracks)
- **Edge-drag to resize** (change duration from left or right edge)
- **Magnetic snapping** to playhead and other clip boundaries
- **Overlap prevention** with collision detection
- **Split** at the playhead position
- **Visual previews** (video thumbnails, audio waveforms, image previews)

### Snapping Engine

During a drag, the clip's edges are compared against all other clip edges and the playhead:

```typescript
const SNAP_THRESHOLD = 8; // pixels

const snapTargets = [
  currentTime * zoom,                    // playhead
  ...otherClips.flatMap(c => [
    c.start * zoom,                      // other clip starts
    (c.start + c.duration) * zoom,       // other clip ends
  ]),
];

const proposedX = clip.start * zoom + dragDeltaX;

for (const target of snapTargets) {
  if (Math.abs(proposedX - target) < SNAP_THRESHOLD) {
    return target; // snap!
  }
  // Also check the right edge
  const rightEdge = proposedX + clip.duration * zoom;
  if (Math.abs(rightEdge - target) < SNAP_THRESHOLD) {
    return target - clip.duration * zoom;
  }
}
```

When a snap occurs, a visual guideline (thin vertical line) appears at the snap position.

### Overlap Prevention

Clips on the same track cannot overlap. During drag, we check for collisions:

```typescript
const wouldOverlap = (
  movingClip: Clip,
  proposedStart: number,
  trackClips: Clip[]
): boolean => {
  const proposedEnd = proposedStart + movingClip.duration;
  return trackClips.some(other =>
    other.id !== movingClip.id &&
    proposedStart < other.start + other.duration &&
    proposedEnd > other.start
  );
};
```

If an overlap is detected, the clip snaps to the nearest valid position (usually butting up against the blocking clip).

### Drag Performance: Local State Bypass

This is a critical performance pattern. During a drag, we **do not** update Redux on every mouse move. Instead:

1. `onDragStart` → Save the initial Redux state
2. `onDragMove` → Update **component-local state only** (Konva node position)
3. `onDragEnd` → Compute final position → **single Redux dispatch**

```typescript
// ❌ BAD: Dispatch on every frame
onDragMove={(e) => {
  dispatch(updateClip({ clipId, updates: { start: xToTime(e.target.x()) } }));
}}

// ✅ GOOD: Local state during drag, dispatch once on end
onDragMove={(e) => {
  // Konva moves the node visually — no Redux needed
  // Just calculate snapping and show guidelines
}}
onDragEnd={(e) => {
  const finalTime = xToTime(e.target.x());
  dispatch(saveToHistory());
  dispatch(moveClip({ clipId, newTrackId, newStart: finalTime }));
}}
```

This eliminates hundreds of Redux dispatches during a single drag and keeps the canvas at 60FPS.

### Video Thumbnails on Clips

Video clips display extracted frame thumbnails as a filmstrip. Frames are extracted asynchronously when a clip is added:

```typescript
// Extract frames from a video file
async function extractVideoFrames(src: string, options: {
  interval: number;   // seconds between frames
  maxFrames: number;  // cap to prevent memory issues
  quality: number;    // JPEG quality (0-1)
  width: number;      // thumbnail width
  height: number;     // thumbnail height
}): Promise<{ timestamp: number; dataUrl: string }[]> {
  const video = document.createElement("video");
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");

  // Load video metadata
  video.src = src;
  await new Promise(r => video.onloadedmetadata = r);

  // Seek to each timestamp and capture frame
  const frames = [];
  for (let time = 0; time < video.duration; time += options.interval) {
    video.currentTime = time;
    await new Promise(r => video.onseeked = r);

    canvas.width = options.width;
    canvas.height = options.height;
    ctx.drawImage(video, 0, 0, options.width, options.height);
    frames.push({
      timestamp: time,
      dataUrl: canvas.toDataURL("image/jpeg", options.quality),
    });
  }
  return frames;
}
```

Frames are cached in a `VideoFrameCache` (LRU, max 100 videos) and re-used across zoom changes. The frame interval adapts to the zoom level:

```typescript
function calculateFrameInterval(zoom: number, duration: number): number {
  const baseInterval = Math.max(0.1, 100 / zoom);
  return Math.max(0.5, Math.min(5, baseInterval));
}
```

Zoomed in → more frames (higher detail). Zoomed out → fewer frames (less memory).

---

## Part 4 — The Preview Player (HTML5 Media Sync)

The player renders a live preview of the timeline at the current playhead position. It composites all active clips (videos, images, text, subtitles) into a single canvas view.

### Active Clip Resolution

At any given `currentTime`, the player must determine which clips are "active":

```typescript
const activeClips = Object.values(clips).filter(clip => {
  const clipEnd = clip.start + clip.duration;
  return currentTime >= clip.start && currentTime < clipEnd;
});
```

Active clips are then sorted by `zIndex` and rendered in order (highest on top).

### Media Synchronization

The hardest part of the player is keeping `<video>` and `<audio>` elements in sync with the playhead. Here's the synchronization strategy:

```
Redux currentTime (source of truth)
    │
    ├─► Video elements: seek to (sourceStart + localTime)
    ├─► Audio elements: seek to (sourceStart + localTime)
    └─► Subtitle cues: show/hide based on time window

Where localTime = currentTime - clip.start
```

Three effects handle different synchronization scenarios:

| Effect | Trigger | What it does |
|---|---|---|
| **Playback timer** | `isPlaying = true` | Advances `currentTime` at ~30fps via `setInterval` |
| **Seek sync** | `isPlaying = false` | Corrects elements that have drifted > 0.1s from expected |
| **Cut switch sync** | `isPlaying = true` | Detects when playback crosses a cut boundary and seeks to `sourceStart` |

### Video Element Sharing

Multiple clips from the same source file share a **single** `<video>` element:

```typescript
// videoRefsByMedia: Map<mediaId, HTMLVideoElement>

// Clip A: video.mp4, start=0, duration=5, sourceStart=0
// Clip B: video.mp4, start=8, duration=5, sourceStart=10
// Both use the SAME <video> element, just seek to different sourceStart
```

This prevents loading the same video file multiple times and reduces memory usage.

### Playback Loop

```typescript
useEffect(() => {
  if (!isPlaying) return;

  const interval = setInterval(() => {
    dispatch(setCurrentTime(currentTime + 1/30)); // ~30fps advance

    // Stop at the end of the timeline
    if (currentTime >= totalDuration) {
      dispatch(togglePlayback());
    }
  }, 1000 / 30);

  return () => clearInterval(interval);
}, [isPlaying, currentTime]);
```

---

## Part 5 — Media Library & File Ingestion

The media library (MediaHub) handles file uploads, preview generation, and drag-to-timeline functionality — all **client-side** with zero backend required.

### File Processing Pipeline

```
User drops file(s)
    │
    ▼
validateFile()
    ├─ Reject if > 2GB
    ├─ Reject if unsupported MIME type
    │
    ▼
getMediaType(file)
    ├─ video/mp4, video/webm     →  "video"
    ├─ audio/mp3, audio/wav      →  "audio"
    ├─ image/jpeg, image/png     →  "image"
    ├─ image/gif                 →  "gif"
    ├─ .srt, .vtt                →  "subtitle"
    │
    ├──────────┬──────────┐
    ▼          ▼          ▼
generateThumbnail()  getMediaMetadata()  (parallel)
    │          │
    ▼          ▼
MediaItem { id, type, name, url, duration, thumbnail }
    │
    ▼
dispatch(addMediaItem(item))
    │
    ▼
Render in masonry grid with animated upload progress
```

### Blob URLs (No Server Needed)

All media is accessed via `URL.createObjectURL(file)` — a temporary in-browser URL that points directly to the file in memory:

```typescript
const handleFileUpload = (file: File) => {
  const blobUrl = URL.createObjectURL(file);
  // blobUrl → "blob:http://localhost:3000/a1b2c3d4-e5f6-..."

  dispatch(addMediaItem({
    id: crypto.randomUUID(),
    type: getMediaType(file),
    name: file.name,
    url: blobUrl,
    // ...
  }));
};
```

This means the entire editor runs **locally** with no uploads, no server, no latency.

### Supported File Types

| Category | Formats |
|---|---|
| **Video** | MP4, WebM, OGG, AVI, MOV |
| **Audio** | MP3, WAV, OGG, M4A, AAC |
| **Image** | JPEG, PNG, GIF, WebP |
| **Subtitle** | SRT, VTT |

Maximum file size: **2 GB**. Maximum video duration: **3 hours**.

### Subtitle File Parsing

SRT and VTT files are parsed into structured cue arrays:

```typescript
// SRT format:
// 1
// 00:00:01,000 --> 00:00:04,000
// Hello world

function parseSrt(content: string): Cue[] {
  return content
    .split(/\n\n+/)
    .filter(block => block.trim())
    .map(block => {
      const lines = block.trim().split('\n');
      const [start, end] = lines[1].split(' --> ').map(parseTimestamp);
      const text = lines.slice(2).join(' ');
      return { start, end, text };
    });
}
```

Each cue becomes a separate `subtitle` clip on the timeline, all sharing a `subtitleGroupId` so styling changes propagate to the entire group.

### Drag-to-Timeline (Mouse + Touch)

The library supports two drag strategies:

**Mouse (native DnD API):**
```typescript
onDragStart={(e) => {
  e.dataTransfer.setData("application/json", JSON.stringify({
    mediaId: item.id,
    type: item.type,
    name: item.name,
    url: item.url,
  }));
}}
```

**Touch (synthetic DragEvent):**
```typescript
onTouchStart → create floating clone element
onTouchMove  → position clone at finger position
onTouchEnd   → elementFromPoint(x, y) to find drop target
             → construct and dispatch synthetic DragEvent
```

Native drag API doesn't fire on mobile/touch devices, so we simulate the entire drag-and-drop flow manually.

---

## Part 6 — Real-Time Effects Pipeline

Every visual clip supports a comprehensive set of CSS-based effects that preview in real-time and map precisely to FFmpeg filters for server-side rendering.

### Available Effects

```typescript
interface ClipStylingOptions {
  // Visual Filters
  brightness?: number;     // 0–200 (100 = normal)
  contrast?: number;       // 0–200 (100 = normal)
  saturation?: number;     // 0–200 (100 = normal)
  hue?: number;           // 0–360 degrees
  blur?: number;          // 0–20 pixels
  sharpen?: number;       // 0–100
  grayscale?: number;     // 0–100%
  sepia?: number;         // 0–100%
  invert?: number;        // 0–100%

  // Transforms
  flipH?: boolean;
  flipV?: boolean;
  rotate?: number;        // degrees
  roundedCorners?: number; // pixel radius

  // Chroma Key (Green Screen)
  greenScreenEnabled?: boolean;
  greenScreenColor?: string;      // hex color
  greenScreenSimilarity?: number; // 0.0–0.5
  greenScreenBlend?: number;      // 0.0–1.0
}
```

### CSS ↔ FFmpeg Mapping

The UI sliders produce values that are mathematically converted to match FFmpeg's `eq` filter output:

```typescript
// brightness: UI 0–200 → CSS filter 0–2 → FFmpeg eq brightness -1 to +1
const convertBrightnessForCSS = (value: number) => value / 100;

// Applied as:
style={{ filter: `brightness(${convertBrightnessForCSS(clip.styling.brightness)})` }}

// Exported as:
ffmpegFilter: `eq=brightness=${(value - 100) / 100}`
```

This means **what you see in the browser preview is what you get in the final render**.

### Chroma Key (Green Screen)

The chroma key system replaces a `<video>` element with a `<canvas>` where the background color has been made transparent:

```typescript
// Per-pixel algorithm (runs every frame)
for (let i = 0; i < pixels.length; i += 4) {
  const r = pixels[i], g = pixels[i+1], b = pixels[i+2];

  // Euclidean distance from key color
  const distance = Math.sqrt(
    (r - keyR) ** 2 + (g - keyG) ** 2 + (b - keyB) ** 2
  ) / 441.67; // normalize to 0–1

  if (distance <= similarity) {
    pixels[i+3] = 0;                           // fully transparent
  } else if (distance <= similarity + blend) {
    const alpha = (distance - similarity) / blend;
    pixels[i+3] = Math.round(alpha * 255);     // smooth edge
  }
  // else: fully opaque (keep original)
}
```

Parameters map 1:1 to FFmpeg's `chromakey` filter:

| Browser | FFmpeg |
|---|---|
| `similarity` | `chromakey=similarity=` |
| `blend` | `chromakey=blend=` |
| `color` (hex) | `chromakey=color=0xRRGGBB` |

### Performance: rAF-Batched Dispatch

Slider interactions (brightness, contrast, etc.) can fire hundreds of events per second. We batch them:

```typescript
const pendingStyleRef = useRef<Partial<ClipStyling> | null>(null);
const rafIdRef = useRef<number | null>(null);

const updateStyling = (key: string, value: number) => {
  // 1. Update local state immediately (instant visual feedback)
  setLocalOptions(prev => ({ ...prev, [key]: value }));

  // 2. Queue for Redux dispatch
  pendingStyleRef.current = {
    ...pendingStyleRef.current,
    [key]: value,
  };

  // 3. Schedule a single batched dispatch
  if (!rafIdRef.current) {
    rafIdRef.current = requestAnimationFrame(() => {
      dispatch(updateClip({
        clipId,
        updates: { styling: pendingStyleRef.current },
      }));
      pendingStyleRef.current = null;
      rafIdRef.current = null;
    });
  }
};
```

Result: smooth slider interaction with maximum one Redux dispatch per frame.

---

## Part 7 — Subtitle & Caption System

The subtitle system handles everything from file parsing to animated on-screen rendering.

### Animated Subtitle Rendering

Subtitles aren't just static text — they support animated presets:

| Preset | Description | Implementation |
|---|---|---|
| `none` | Static text | Entire cue visible at once |
| `word-highlight` | Karaoke-style | Words change color as they are "spoken" based on time |
| `pop-in` | Scale entrance | Each word springs from scale(0) with Framer Motion |
| `word-pop` | Single word | Only the current word is shown, with a spring animation |
| `typewriter` | Character reveal | Characters appear one-by-one based on playback progress |

### Responsive Text Windowing

Long subtitles are split into windows that fit the canvas aspect ratio:

```typescript
const maxCharsPerWindow = {
  "9:16": 22,   // vertical: tight space
  "1:1":  35,   // square: moderate
  "16:9": 60,   // landscape: plenty of room
};
```

### Subtitle Styling Panel

A full styling panel with tabs for:

| Tab | Controls |
|---|---|
| **Text** | Font family (7 bundled fonts), size, primary color |
| **Outline** | Stroke color, width |
| **Shadow** | Drop shadow color, depth |
| **Position** | On-screen placement (top, center, bottom) |
| **Presets** | One-click style bundles + animation presets |

Style presets include combinations like "Impact Modern", "Gaming Fun", "Cyber Glow" — each with font, colors, outline, and animation pre-configured.

### Group Styling

All subtitle clips from the same SRT file share a `subtitleGroupId`. When the user changes font or color, all clips in the group update simultaneously:

```typescript
dispatch(updateClipsByGroup({
  groupId: clip.subtitleGroupId,
  updates: {
    styling: {
      fontSize: 28,
      fontName: "Montserrat",
      primaryColor: "#FFD700",
    },
  },
}));
```

---

## Part 8 — Keyframe Animation Engine

Clips support time-based keyframe animation for position, size, opacity, rotation, and all filter values.

### Keyframe Data Structure

```typescript
interface Keyframe {
  at: number;           // seconds since clip start
  position?: { x: number; y: number };
  size?: { width: number; height: number };
  rotate?: number;
  opacity?: number;
  brightness?: number;
  contrast?: number;
  saturation?: number;
  hue?: number;
  blur?: number;
  grayscale?: number;
  sepia?: number;
  invert?: number;
  roundedCorners?: number;
  easing?: "linear" | "jump-cut";
}
```

### Interpolation

At each frame, the player calculates interpolated values between surrounding keyframes:

```typescript
function getInterpolatedKeyframeValues(
  keyframes: Keyframe[],
  localTime: number,        // current time within the clip
  baseValues: ClipProperties  // default/static values
): ClipProperties {
  // Find the two keyframes surrounding localTime
  const before = keyframes.filter(k => k.at <= localTime).pop();
  const after = keyframes.find(k => k.at > localTime);

  if (!before && !after) return baseValues;
  if (!before) return { ...baseValues, ...after };
  if (!after) return { ...baseValues, ...before };

  // Calculate progress between the two keyframes
  const progress = (localTime - before.at) / (after.at - before.at);

  // Apply easing
  if (after.easing === "jump-cut") {
    return progress >= 1
      ? { ...baseValues, ...after }
      : { ...baseValues, ...before };
  }

  // Linear interpolation for each property
  return {
    ...baseValues,
    position: {
      x: lerp(before.position?.x, after.position?.x, progress),
      y: lerp(before.position?.y, after.position?.y, progress),
    },
    opacity: lerp(before.opacity, after.opacity, progress),
    rotate: lerp(before.rotate, after.rotate, progress),
    // ... etc for all animatable properties
  };
}

function lerp(a?: number, b?: number, t: number): number {
  if (a === undefined && b === undefined) return 0;
  if (a === undefined) return b!;
  if (b === undefined) return a;
  return a + (b - a) * t;
}
```

---

## Part 9 — Audio Processing (Web Audio API)

Audio clips aren't just `<audio>` elements — they go through a full signal processing chain using the Web Audio API.

### Signal Chain Architecture

```
<audio> element
    │
    └─► MediaElementAudioSourceNode
            │
    ┌───────┴──────────────────────────────┐
    ▼                                      │
  BiquadFilterNode (lowshelf @ 200Hz)      │ ← Bass boost
    │                                      │
  BiquadFilterNode (highshelf @ 4kHz)      │ ← Treble boost
    │                                      │
    ├──► GainNode ──► destination           │ ← Main volume
    │                                      │
    ├──► DelayNode (250ms)                 │
    │       └──► GainNode ──► main gain     │ ← Echo effect
    │                                      │
    └──► ConvolverNode (synthetic IR)      │
            └──► GainNode ──► main gain     │ ← Reverb effect
```

### Parameters

| Setting | Node Type | Range | Default |
|---|---|---|---|
| `volume` | GainNode | 0–1 | 1 |
| `audioBassBoost` | BiquadFilter (lowshelf) | 0–20 dB | 0 |
| `audioTrebleBoost` | BiquadFilter (highshelf) | 0–20 dB | 0 |
| `audioEcho` | GainNode (delay wet) | 0–1 | 0 |
| `audioReverb` | GainNode (convolver wet) | 0–1 | 0 |
| `audioSpeed` | `el.playbackRate` | 0.5–2 | 1 |

### AudioContext Management

The `AudioContext` must be resumed after a user gesture (browser autoplay policy). The hook calls `.resume()` when `isPlaying` becomes true.

### Resource Management

Audio graphs for clips outside their playback window are **disconnected and destroyed**:

```typescript
useEffect(() => {
  const activeAudioClipIds = new Set(
    activeClips
      .filter(c => c.type === "audio" || c.type === "video")
      .map(c => c.id)
  );

  // Disconnect graphs for clips that are no longer active
  for (const [clipId, graph] of audioGraphs.entries()) {
    if (!activeAudioClipIds.has(clipId)) {
      graph.source.disconnect();
      graph.gain.disconnect();
      audioGraphs.delete(clipId);
    }
  }
}, [currentTime]);
```

---

## Part 10 — Video Transition Effects

Transitions are special clips placed between two video clips to create smooth visual handoffs.

### Available Transition Types

| Effect | Description | FFmpeg Filter |
|---|---|---|
| `fade` | Standard crossfade | `xfade=transition=fade` |
| `dissolve` | Smooth pixel blend | `xfade=transition=dissolve` |
| `wipeleft` | Left-to-right wipe | `xfade=transition=wipeleft` |
| `wiperight` | Right-to-left wipe | `xfade=transition=wiperight` |
| `wipeup` | Bottom-to-top wipe | `xfade=transition=wipeup` |
| `wipedown` | Top-to-bottom wipe | `xfade=transition=wipedown` |
| `slideleft` | Push left | `xfade=transition=slideleft` |
| `slideright` | Push right | `xfade=transition=slideright` |
| `circleopen` | Circular reveal | `xfade=transition=circleopen` |
| `circleclose` | Circular close | `xfade=transition=circleclose` |
| `fadeblack` | Fade through black | `xfade=transition=fadeblack` |
| `fadewhite` | Fade through white | `xfade=transition=fadewhite` |

### Browser Preview vs. FFmpeg Render

Transitions are **approximated** in the browser using CSS animations and canvas compositing. The final render uses FFmpeg's `xfade` filter for pixel-perfect results. The preview is close enough to give accurate editorial feedback.

### Dragging Transitions onto the Timeline

Transitions use the same drag system as media items but with an `isEffect: true` flag:

```typescript
// Effects Library drag payload
e.dataTransfer.setData("application/json", JSON.stringify({
  isEffect: true,
  effectType: "fade",        // or "dissolve", "wipeleft", etc.
  duration: 1,               // default 1 second
}));

// Timeline drop handler checks for this flag
if (payload.isEffect) {
  // Insert between two clips, overlapping their boundaries
  dispatch(addClip({ trackId, clip: transitionClip }));
}
```

---

## Part 11 — Exporting to FFmpeg / Backend Rendering

The timeline editor is a **preview tool**. Final video rendering happens server-side (or via WebAssembly). The editor exports its state as a structured JSON payload.

### Extracting Timeline Data

```typescript
import { useSelector } from "react-redux";
import { RootState } from "./redux/store";

const ExportButton = () => {
  const { clips, tracks, aspectRatio } = useSelector(
    (state: RootState) => state.timeline
  );

  const handleExport = async () => {
    const payload = {
      aspectRatio,
      tracks: tracks.map(track => ({
        id: track.id,
        type: track.type,
        clips: track.clips.map(clipId => {
          const clip = clips[clipId];
          return {
            type: clip.type,
            start: clip.start,
            duration: clip.duration,
            sourceStart: clip.sourceStart || 0,
            url: clip.url,
            volume: clip.volume,
            styling: clip.styling,
            keyframes: clip.keyframes,
            position: clip.position,
            size: clip.size,
          };
        }),
      })),
    };

    await fetch("/api/render-video", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload),
    });
  };

  return <button onClick={handleExport}>Export Video</button>;
};
```

### Server-Side FFmpeg Command Generation

The JSON payload maps directly to FFmpeg filter chains:

```bash
# Example: Two clips with a crossfade transition
ffmpeg \
  -i clip1.mp4 -i clip2.mp4 \
  -filter_complex "
    [0:v]trim=start=0:duration=5,setpts=PTS-STARTPTS,
      eq=brightness=0.1:contrast=1.2:saturation=1.5[v0];
    [1:v]trim=start=10:duration=5,setpts=PTS-STARTPTS[v1];
    [v0][v1]xfade=transition=fade:duration=1:offset=4[outv]
  " \
  -map "[outv]" output.mp4
```

Each styling property on the clip maps to a known FFmpeg filter parameter. The browser preview uses CSS filter values that are mathematically equivalent, so **what you see is what you render**.

---

## Part 12 — Performance Deep-Dive: 60FPS at Scale

A timeline editor that drops frames is unusable. Here's every optimization technique used:

### 1. Local State Bypass During Drag

```
onDragMove → update Konva node directly (no Redux)
onDragEnd  → single dispatch to Redux
```

Impact: Eliminates ~100 Redux dispatches per drag operation.

### 2. Live Duration Overrides

During resize, a lightweight `liveDurationOverrides` map stores transient values:

```typescript
// During resize: update only the lightweight override
dispatch(setLiveDurationOverride({ clipId, duration: newDuration }));

// On resize end: commit to the actual clip
dispatch(updateClip({ clipId, updates: { duration: finalDuration } }));
dispatch(clearLiveDurationOverride({ clipId }));
```

### 3. rAF-Batched Slider Dispatches

Styling sliders queue changes and flush once per animation frame (see Part 6).

### 4. Canvas Virtualization

Only clips visible within the current scroll viewport are rendered on the Konva canvas (see Part 2).

### 5. Video Frame Caching

Extracted thumbnails are cached in an LRU cache (max 100 entries). Cache is reused across zoom changes when the interval difference is < 20%.

### 6. Memoized Components

```typescript
const MediaHubItem = React.memo(({ item }) => {
  // Only re-renders when item changes
  return <div>...</div>;
});
```

### 7. shallowEqual Selectors

```typescript
const { clips, tracks } = useSelector(
  (state: RootState) => ({
    clips: state.timeline.clips,
    tracks: state.timeline.tracks,
  }),
  shallowEqual  // prevent re-render if references haven't changed
);
```

### 8. Audio Graph Lifecycle Management

Web Audio nodes are created on-demand when clips become active and destroyed when they leave the playback window, preventing AudioContext resource leaks.

### Summary Table

| Technique | Where | Impact |
|---|---|---|
| Local state bypass | Clip drag | ~100 fewer dispatches/drag |
| Live duration overrides | Clip resize | No full recalc during resize |
| rAF batching | Styling sliders | Max 1 dispatch/frame |
| Canvas virtualization | Timeline render | Only visible clips drawn |
| Frame caching | Video thumbnails | No re-extraction on zoom |
| React.memo | Media grid items | Skip unchanged re-renders |
| shallowEqual | Redux selectors | Skip unchanged re-renders |
| Audio graph cleanup | Player audio | Free Web Audio resources |

---

## Part 13 — Theming & UI System

The editor uses a scoped CSS variable system that prevents conflicts with host application styles.

### CSS Variable Architecture

```css
.timeline-theme-root {
  /* All theme variables are scoped under this class */
  --background: #ffffff;
  --foreground: #000000;
  --primary: #000000;
  --accent: #d4d4d4;
  --border: #000000;
  --radius: 0.625rem;
  /* ... 30+ variables */
}

/* All utility classes are scoped */
.timeline-theme-root .bg-primary {
  background-color: var(--primary);
}
.timeline-theme-root .text-muted-foreground {
  color: var(--muted-foreground);
}
```

### Why Scoped?

The editor is designed to be **dropped into any existing project**. Scoping ensures:

- No Tailwind conflicts with the host app
- No CSS variable collisions
- The editor's theme is self-contained
- You can override any variable from the parent scope

### Canvas Theme Constants

The Konva canvas uses a separate TypeScript constants file:

```typescript
export const THEME = {
  background: "#0f172a",     // Slate 900
  trackVideo: "#6366f1",     // Electric Indigo
  trackAudio: "#a855f7",     // Purple 500
  trackSubtitle: "#22d3ee",  // Cyber Cyan
  playhead: "#f43f5e",       // Rose 500 (high visibility)
  selection: "#ffffff",      // High contrast white
  accent: "#6366f1",         // Primary Indigo
  invalid: "#f43f5e",        // Error states
};
```

### UI Component Library

The editor ships with 23 Radix-based UI components, all pre-styled:

| Component | Usage |
|---|---|
| Button, Input, Textarea | Forms & controls |
| Dialog, AlertModal, ConfirmationDialog | Modal interactions |
| DropdownMenu, Select, Popover | Menus & selectors |
| Tabs, Accordion | Panel organization |
| Slider, Switch, Checkbox | Value controls |
| Tooltip, Badge, Progress | Feedback & status |
| Card, Separator, Skeleton | Layout & loading |
| Carousel | Media browsing |

### Bundled Fonts

7 fonts are bundled as local `@font-face` declarations for offline subtitle rendering:

- Montserrat
- Bungee
- Luckiest Guy
- Komika Axis
- Bebas Neue
- The Bold Font
- Fredoka

---

## Part 14 — Making It Production-Ready

The barebones editor is ~35 files. Taking it to production requires:

### Undo/Redo

Snapshot-based history (not command-based):

```typescript
// Before any destructive action:
dispatch(saveToHistory());  // snapshots current { clips, tracks }

// Undo: restore previous snapshot
dispatch(undo());  // historyIndex -= 1

// Redo: restore next snapshot
dispatch(redo());  // historyIndex += 1
```

Max 50 history entries to prevent memory issues. History is truncated when forking (new action after undo discards the "future").

### Project Save/Load

```typescript
// Save: serialize the Redux state
const projectData = {
  tracks: store.getState().timeline.tracks,
  clips: store.getState().timeline.clips,
  mediaItems: store.getState().timeline.mediaItems,
  aspectRatio: store.getState().timeline.aspectRatio,
  globalSubtitleStyling: store.getState().timeline.globalSubtitleStyling,
};

// Save to your backend, localStorage, or file download
await fetch("/api/projects", {
  method: "POST",
  body: JSON.stringify(projectData),
});

// Load: hydrate the state
dispatch(loadProject(projectData));
```

### Keyboard Shortcuts

| Key | Action |
|---|---|
| `Space` | Play / Pause |
| `Ctrl + Z` | Undo |
| `Ctrl + Shift + Z` | Redo |
| `Delete` / `Backspace` | Delete selected clip |
| `S` | Split clip at playhead |
| `←` / `→` | Nudge playhead |

### Layout Modes

The editor supports three resizable layout configurations:

| Mode | Description |
|---|---|
| **Default** | Library left, player center, timeline bottom |
| **Player top** | Player on top, timeline + library below |
| **Library top** | Library on top, player + timeline below |

All panels use `ResizableSplitter` for drag-to-resize.

### Aspect Ratio Support

| Value | Use Case |
|---|---|
| `16:9` | YouTube, desktop video |
| `9:16` | TikTok, Reels, Shorts |
| `4:3` | Legacy video |
| `1:1` | Instagram posts |

### Important Constraints

- **Blob URLs expire** when the tab is closed — implement database sync for persistence
- **CORS**: Chroma key requires same-origin media for `getImageData`
- **AudioContext**: Must be resumed after user gesture
- **Media not persisted by default** — you control the backend

---

## The Hard Truth

Building all of this from scratch is **months** of engineering. Here's a realistic breakdown:

| Component | Estimated Dev Time |
|---|---|
| Data model + Redux slice | 1–2 weeks |
| Konva canvas + tracks | 2–3 weeks |
| Clip interactions (drag, resize, snap) | 3–4 weeks |
| Player sync (video, audio, subtitles) | 2–3 weeks |
| Media library + file processing | 1–2 weeks |
| Effects pipeline (filters, chroma key) | 2–3 weeks |
| Keyframe animation | 1–2 weeks |
| Audio processing (Web Audio API) | 1–2 weeks |
| Transition effects | 1 week |
| Undo/redo, save/load | 1 week |
| UI system + theming | 1 week |
| Bug fixing, edge cases, polish | 2–4 weeks |
| **Total** | **~4–6 months** |

And that's assuming you've done this before. First-time? Double it.

---

## See It in Action

Don't take my word for it. See the working product:

### 🎬 Video Demos

| Demo | What It Shows |
|---|---|
| [▶️ Full Showcase Video](https://media.klipflow.com/file/klipflow/showcase.mp4) | Complete editing flow — import, drag, trim, split, playback |
| [▶️ Plug & Play Integration](https://media.klipflow.com/file/klipflow/plug+and+play.mp4) | How to drop the editor into any existing React project in minutes |
| [▶️ FFmpeg Plugin Demo](https://media.klipflow.com/file/klipflow/ffmpeg-plugin.mp4) | Server-side rendering extension — timeline state → rendered MP4 |

### 🌐 Live Demos

| Link | Description |
|---|---|
| [**Barebones Editor**](https://abdet-tools.vercel.app/timeline) | The raw editor component — this is what you get out of the box |
| [**KlipFlow (Full SaaS)**](https://www.klipflow.com/timeline) | What you can build with it — a complete video editing SaaS |

---

## Skip Months of Development

This guide teaches you the *architecture*. If you want the *code* — every component, every hook, every utility, every edge case already handled — it's available as a premium drop-in source code package:

<p align="center">

### [🚀 Get the Complete Timeline Editor Source Code](https://abdetiamani.gumroad.com/l/timeline)

</p>

**What's included:**

- ✅ **35+ production React components** (TypeScript, fully typed)
- ✅ **1,000+ line Redux slice** with every action you'll need
- ✅ **Multi-track canvas** with Konva (drag, resize, snap, split)
- ✅ **Synchronized player** with HTML5 media + Web Audio API
- ✅ **Chroma key**, visual filters, keyframe animation
- ✅ **SRT/VTT subtitle parsing** with animated presets
- ✅ **Transition effects** (fade, dissolve, wipe, slide, circle)
- ✅ **Complete UI system** (23 Radix-based components, scoped theme)
- ✅ **FFmpeg-ready export** — CSS preview values map 1:1 to FFmpeg filters
- ✅ **Undo/redo**, project save/load, keyboard shortcuts

**Use it in:**
- Your AI video generation tool
- Your podcast editing SaaS
- Your caption/subtitle platform
- Your content creation app
- Your internal video tooling

One purchase. Full source code. Unlimited commercial use.

### [→ Get it on Gumroad](https://abdetiamani.gumroad.com/l/timeline)

---

<p align="center">
  <sub>Created by <a href="https://github.com/Abde-T">Abde Tiamani</a> · <a href="https://abdetiamani.gumroad.com/l/timeline">Gumroad</a> · <a href="https://www.klipflow.com/timeline">KlipFlow</a></sub>
</p>
