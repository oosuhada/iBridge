# Device-boundary notes from `beneath-the-stack` v0.5

iBridge Studio is not a microcontroller firmware project, but it still has a device-boundary shape:

```text
MacBook virtual display
  -> ScreenCaptureKit frame source
  -> encoder / bounded capture queue
  -> TCP stream / protocol messages
  -> iMac receiver decode
  -> fullscreen display + input relay
```

The v0.5 embedded track in `beneath-the-stack` is useful here because it focuses on the same classes
of failure at a different scale: timing, bounded buffers, state transitions, protocol framing and
safe recovery.

## Mapping

| Embedded lesson | iBridge boundary | Practical implication |
| --- | --- | --- |
| Periodic task jitter | frame pacing and encode deadlines | A slow encode or sender loop should be measured as deadline miss/jitter, not just average FPS. |
| Ring buffer overflow policy | capture queue depth / network send queue | Dropping oldest, rejecting new frames, or blocking the producer are different product decisions. |
| Serial frame parser | cursor/input/control messages on a stream | Every control message needs framing, length validation and corruption/partial-message behavior. |
| Fault state and recovery | receiver disconnect, virtual display missing, encoder unavailable | UI status should represent a state machine with recovery paths, not only a log line. |
| HAL boundary | ScreenCaptureKit / VideoToolbox / display layer seams | Platform APIs should stay behind adapters so measurement logic can be tested without a live display. |

## What this does not claim

- This does not make iBridge Studio an embedded firmware project.
- It does not claim MCU timing measurements.
- It does not replace the existing macOS-specific ScreenCaptureKit, VideoToolbox and Accessibility
  work.

## Next safe experiment

The most valuable follow-up is a frame-queue/backpressure probe that records:

```text
capture timestamp
enqueue result
queue depth
encode start/end
send start/end
receiver presentation timestamp
```

That would connect the firmware scheduler/ring-buffer lesson to a real display pipeline without
pretending a virtual display is the same as a GPIO peripheral.
