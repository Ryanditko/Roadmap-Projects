# Streaming UI (video platform)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build the playback experience of a video platform like YouTube or Netflix — adaptive streaming that shifts quality with the network, smooth controls, and an interface that stays responsive while a large media pipeline runs underneath. The core insight is that you do not download a video; you download a manifest describing many quality renditions split into small segments, and the player continuously chooses which segment to fetch next based on measured bandwidth and buffer health. Get that adaptation wrong and the user sees stalls or blurry frames; get the UI wrong and controls feel laggy against the heavy decode work. This project is about orchestrating an adaptive-bitrate engine and a polished, accessible player around it.

## Prerequisites

- Confident with asynchronous data flow and browser events
- Understanding of HTTP range requests and buffering concepts
- Familiarity with the HTML5 `<video>` element and Media Source Extensions (at least conceptually)
- Basic grasp of network throughput and how it varies

## Learning Objectives

By the end, you should be able to:

- Explain adaptive bitrate streaming (HLS/DASH): manifests, renditions, and segments
- Integrate an ABR engine and reason about its quality-selection heuristics
- Build accessible, keyboard-operable playback controls that stay responsive under load
- Manage buffering to minimize stalls while avoiding excessive memory use
- Instrument playback quality (stalls, startup time, bitrate) for a quality-of-experience view

## Functional Requirements

1. The player must play an adaptive stream (HLS or DASH) and switch renditions automatically as bandwidth changes.
2. A user must be able to override automatic quality and pin a specific rendition.
3. Playback controls (play/pause, seek, volume, fullscreen) must be fully keyboard-operable and labelled.
4. Seeking to any point must buffer and resume without reloading the whole stream.
5. The player must recover gracefully from a transient network drop without a hard error.
6. Buffer memory must be bounded so long sessions do not grow unboundedly.
7. Playback quality metrics (startup time, stall count, current bitrate) must be observable.

## Suggested Milestones

1. **Milestone 1 — Basic adaptive playback:** Load a manifest and play it with an ABR engine, auto quality only.
2. **Milestone 2 — Controls & a11y:** Build accessible custom controls: seek, volume, fullscreen, captions.
3. **Milestone 3 — Quality & resilience:** Add manual quality override and recovery from network interruptions.
4. **Milestone 4 — QoE instrumentation:** Track stalls, startup time, and bitrate; surface them in a debug overlay.

## Data & Interface Sketch

```text
   Manifest (.m3u8 / .mpd)
     ├── rendition 240p  → seg0 seg1 seg2 ...
     ├── rendition 480p  → seg0 seg1 seg2 ...
     └── rendition 1080p → seg0 seg1 seg2 ...
                    │
                    ▼
        ┌───────────────────────────┐
        │   ABR engine (hls.js/dash) │  measures bandwidth + buffer
        │   picks next segment       │─────────────┐
        └─────────────┬─────────────┘             │ appends to
                      │                             ▼
                      │                    ┌──────────────────┐
                      │                    │ MediaSource buffer│→ <video>
        ┌─────────────▼─────────────┐      └──────────────────┘
        │  Player UI (controls, a11y)│
        │  play/seek/volume/quality  │
        └────────────────────────────┘

QoE metrics:  startupTime, rebufferCount, currentBitrate, droppedFrames
Non-functional targets:
  startup time        < 2 s on broadband
  rebuffer ratio      < 1% of watch time
  control response    < 100 ms regardless of decode load
```

## Stretch Goals

- Add captions/subtitles (WebVTT) with a track selector and styling controls.
- Add picture-in-picture and a mini-player that persists across navigation.
- Implement a thumbnail preview strip on the seek bar (sprite-based).
- Add a watch-history/resume feature that restores the last position.

## Definition of Done

- [ ] The stream visibly steps down in quality on a throttled connection and recovers when bandwidth returns.
- [ ] All controls work with keyboard only and announce state to a screen reader.
- [ ] Seeking mid-stream resumes quickly without a full reload.
- [ ] A simulated network blip does not throw a fatal error; playback resumes.
- [ ] The QoE overlay reports startup time, stall count, and current bitrate live.

## Common Pitfalls

- Rolling your own segment fetcher instead of a proven ABR engine — the heuristics are hard to get right.
- Building controls with non-interactive elements, breaking keyboard and screen-reader access.
- Never releasing buffered segments, so memory climbs across a long viewing session.
- Treating a transient 5xx on one segment as fatal instead of retrying the next.
- Measuring bandwidth only at startup, so quality never adapts to a mid-stream network change.

## Resources

- [MDN: Media Source Extensions API](https://developer.mozilla.org/en-US/docs/Web/API/Media_Source_Extensions_API) — the browser API behind adaptive streaming.
- [hls.js documentation](https://github.com/video-dev/hls.js/#documentation) — a widely used HLS playback engine for the web.
- [web.dev: Fast playback with audio and video preload](https://web.dev/articles/fast-playback-with-preload) — reducing startup latency.
- [MDN: WebVTT](https://developer.mozilla.org/en-US/docs/Web/API/WebVTT_API) — the standard for captions and subtitles.
