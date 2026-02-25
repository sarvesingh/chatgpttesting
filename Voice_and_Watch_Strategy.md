# Pumpr Voice & Watch Strategy (V1)

> Core Brand Promise:  
> **Stop Logging. Start Lifting.**

This document outlines the V1 direction for voice interaction and Apple Watch integration, aligned with real-world constraints (AirPods + music) and our product positioning.

---

# 1. Goals & Constraints

## 1.1 Primary Goal

Lifters should not need to pull their phone out mid-set.

## 1.2 Non-Negotiables

- Music quality must not degrade during workouts
- Voice interaction must feel fast (< 1.5s total interaction)
- Set logging accuracy must exceed 95%
- No persistent microphone activation
- No unreliable wake-word behavior

---

# 2. Voice Strategy (V1)

## 2.1 Interaction Model

**Voice is PUSH-TO-TALK. Not always-on.**

### Flow

1. User taps or holds mic (phone or watch)
2. Mic activates for short burst (0.8–1.5s)
3. Transcript parsed locally
4. Set logged via API
5. Haptic confirmation
6. Mic session closes immediately

No continuous listening.  
No wake word in V1.

---

## 2.2 AirPods + Music Handling

### Problem

Opening mic continuously causes:
- Bluetooth profile downgrade (HFP)
- Music quality degradation
- Persistent ducking
- Latency shifts

### Solution

- Default audio session = playback/ambient
- Activate recording session ONLY during push-to-talk window
- Deactivate recording session immediately after transcript finalization
- Prefer haptic confirmations over spoken responses

---

## 2.3 Confirmation Strategy

### Default

- Haptic confirmation on successful log
- UI update (weight x reps)
- Rest timer starts

### Spoken Feedback (Limited)

Only speak when:
- PR achieved
- Critical correction required

Avoid long TTS responses.

---

## 2.4 Audio Session States

### IDLE
- Playback/ambient category
- No mic active
- Music unaffected

### LISTENING
- Switch to record-capable category
- Prefer built-in mic if appropriate
- Capture short burst
- Immediately deactivate

### CONFIRMATION
- Haptic primary
- Minimal speech secondary

---

# 3. Apple Watch Strategy

## 3.1 Watch is Primary Gym Interface

When connected:
- Watch becomes primary input
- Phone becomes display + engine
- Phone remains fallback

---

## 3.2 Watch Detection

Use `WatchConnectivity`:

- `WCSession.isPaired`
- `WCSession.isWatchAppInstalled`
- `WCSession.activationState`

If paired + installed:
- Enable watch-first mode
- Sync active workout to watch

---

## 3.3 Watch UX Scope (Minimal V1)

Watch should NOT mirror full phone app.

### Watch Displays:

- Current exercise
- Recommended weight x reps
- Hold-to-talk button
- +5 / -5 quick adjustments
- Next exercise control
- Rest timer
- PR badge
- Haptic confirmations

### Watch Does NOT Include:

- History browsing
- Template editing
- Deep navigation
- Settings complexity

Keep surface area minimal.

---

## 3.4 Watch Logging Flow

1. User finishes set
2. Holds watch button
3. Says: “225 for 8”
4. Transcript parsed
5. API logs set
6. Watch vibrates
7. Next recommendation displayed

Music interruption must be brief.

---

# 4. Backend Responsibilities

## 4.1 Deterministic Next Set Engine

No AI inference mid-session.

Engine responsibilities:

- Evaluate last 4 sessions
- Detect PR proximity
- Detect overperformance (rep delta ≥ 2)
- Detect underperformance
- Apply progression logic
- Return:
  - `weight`
  - `reps`
  - `restSeconds`
  - `prOpportunity`
  - `adjustmentReason`
  - `trainingPhase`

---

## 4.2 Parsing Strategy

V1:

- Local regex-based parsing
- Fallback to cloud only if necessary
- Avoid heavy LLM dependency during active session

---

## 4.3 Engine Centralization

Progression logic remains server-side.

Watch:
- Captures voice
- Sends transcript
- Displays result

Phone:
- Maintains session state
- Syncs with backend

---

# 5. Implementation Phases

---

## Phase 1 – Voice Stability Sprint

### Objectives

- Remove always-on mic
- Implement push-to-talk only
- Ensure mic < 1.5s active window
- Limit spoken confirmations
- Validate no persistent music degradation

### Success Criteria

- Voice accuracy > 95%
- Mic active window < 1.5s
- No lasting Bluetooth profile downgrade
- No crashes during active workout

---

## Phase 2 – Watch MVP

### Objectives

- Detect watch pairing
- Sync active workout session
- Implement hold-to-talk on watch
- Add haptic confirmations
- Validate end-to-end logging from watch

### Success Criteria

- Logging from watch < 1.5s
- Recommendations update reliably
- No dropped transcripts
- Stable workout session state

---

## Phase 3 – Optimization

- Improve exercise alias matching
- Refine fuzzy name matching
- Improve audio session switching
- Optimize mic activation latency
- Reduce edge-case mislogs

---

## Phase 4 – Wake Word (Optional / Future)

Only pursue if:

- Music quality remains unaffected
- Detection accuracy > 95%
- Battery impact acceptable
- False triggers minimal

Wake word is not required for product success.

---

# 6. Engineering Task Checklist

## Voice System

- [ ] Remove continuous mic mode
- [ ] Implement push-to-talk activation
- [ ] Implement mic auto-close after transcript
- [ ] Audit AVAudioSession transitions
- [ ] Minimize TTS responses
- [ ] Add haptic confirmations
- [ ] Measure mic active duration

---

## Watch Integration

- [ ] Implement WatchConnectivity detection
- [ ] Sync active workout state to watch
- [ ] Implement hold-to-talk on watch
- [ ] Implement quick adjust buttons (+5 / -5)
- [ ] Implement rest timer on watch
- [ ] Ensure PR badge + haptic behavior
- [ ] Validate sync consistency

---

## Backend Alignment

- [ ] Confirm deterministic progression logic
- [ ] Ensure next-set endpoint < 200ms
- [ ] Validate PR detection accuracy
- [ ] Improve alias mapping for exercise matching

---

# 7. Go / No-Go Criteria

We do NOT expand scope until:

- 10 real gym workouts completed without failure
- No crashes during active workout
- No persistent music degradation
- Set logging feels fast and reliable

Reliability > sophistication.

---

# 8. Strategic Positioning Alignment

We are building:

- A lifting partner with memory
- A frictionless logging experience
- A watch-first training companion

We are NOT building:

- A general AI assistant
- An ambient voice system
- A continuous listener

---

# Final Direction

V1 = Push-to-talk voice + Watch-first experience.

Wake word comes later — only if reliability and audio quality meet our bar.