# SmartLog

The life log that talks back.

## Overview
A passive, mostly-silent AI that:
- Continuously listens to conversation audio.
- Transcribes audio locally in rolling chunks.
- Maintains a rolling transcript of the current hour and the previous hour.
- Uses a wake phrase or manual button to trigger an AI response.
- Upon trigger, the AI:
  - Pulls the last N minutes of text.
  - Generates/updates a summary of the current hour.
  - Includes the summary of the current (and if available) previous hour(s).
  - Generates a concise, relevant response via LLM.
- AI remains otherwise silent and does not speak without explicit request.

## Actions

### Information
- Answer latest question(s)
- Fact-check latest claim(s)
- Suggest learning references
- Summarize the SOTA literature
- Summarize conversation
- Show transcript

### Conflict Resolution
- Steelman each side
- Mediate disagreement
- Suggest compromise
- Identify common ground
- Audit transcript for fallacies
- Audit transcript for incivility
- Join conversation as mediator
- Join conversation on my side
- Join conversation on right side

### Problem Solving
- List action items
- Suggest solutions
- Suggest next steps

## Core Components

### Audio Capture
- Always-on microphone via background audio mode.
- Optional **VAD (Voice Activity Detection)** to reduce processing cost.
- Audio stored in a **ring buffer** (e.g., keeping last 1–5 minutes of raw audio).

### Speech-to-Text (STT)
- Runs every 5–30 seconds on-device for privacy and cost savings.
- Options:
  - **Whisper tiny** (quantized)
  - Platform-native STT (iOS / Android)
- Produces timestamped transcripts and chunk metadata.

### Transcript Management
- Stores **last N minutes** of text (rolling window).
- Maintains optional **longer-term summary**:
  - Updated every 1–3 minutes.
  - Helps maintain context without large token usage.

### Trigger System
Two trigger types:
1. **Button press** (hardware or on-screen)
2. **Wake phrase** (keyword spotting):
   - e.g., using Picovoice Porcupine-like wake-word engine
   - Extremely low power and runs offline

Trigger event causes:
- Pulling recent transcript.
- Pulling summary state.
- Sending to LLM with a **context-weighted prompt**.

### LLM Processing
Prompt structure example:

```
You have been silently observing a conversation.

Here is your current summary of ongoing context:
[SUMMARY_STATE]

Here is the verbatim transcript of the last few minutes:
[RECENT_TRANSCRIPT]

The user has just triggered you to respond.
Prioritize the most recent conversation.
Be brief, accurate, grounded only in available text.
```

Possible response modes (optional):
- Summary
- Analysis
- Advice
- Devil’s advocate
- Fact checking

### Text-to-Speech (Optional)
- Only triggered after user explicitly requests AI output.
- Keeps the system “mostly silent.”

### Privacy & Consent
- Persistent indicator (e.g., “Listening”)
- Ability to pause anytime
- Configurable:
  - Delete raw audio immediately
  - Store only transcripts
  - Store only summaries

---

## Technical Feasibility & Tradeoffs

### Battery
- VAD + chunked STT helps significantly.
- Whisper tiny or OS-native STT recommended.

### Context Size
- 5–15 minutes of text easily fits in modern LLM context windows.
- Summary state keeps long-running context inexpensive.

### Long Sessions
- The system can run **30–60+ minutes** because:
  - No need for live LLM calls until triggered.
  - No forced session expiration from LLM provider.

## High-Level Architecture

```
AudioCaptureService
   ↓
   (raw audio chunks)
   ↓
STTWorker (on-device)
   ↓
   (timestamped text)
   ↓
TranscriptManager
   - rolling transcript (N min)
   - summary_state (updated periodically)
   ↓
TriggerListener
   - button press
   - wake phrase detected
   ↓
LLMClient
   - builds prompt
   - generates response
   ↓
Optional: TTS output
```

## MVP Scope

### Phase 1 — Basic
- Audio capture + rolling transcript
- Manual button trigger
- LLM response using last N minutes

### Phase 2 — Improved Context
- Add periodic summarization
- Add diarization tags (Speaker 1, 2, etc.)

### Phase 3 — Full Experience
- Wake phrase
- Multiple response modes
- Optional TTS
- Battery optimization + user settings

## Prototype Plan: STT Evaluation

To de-risk the project and make key technical decisions, the first step is to build a small prototype application.

### Goal
The primary goal of this prototype is to assess the performance of on-device Speech-to-Text (STT) for continuous speech monitoring. Key metrics to evaluate are:
- Transcription accuracy in conversational settings.
- Latency (time from speech to text).
- Resource consumption (CPU, battery, memory).

### Key Decisions to be Answered by the Prototype

This prototype will help us make the following high-level decisions:

1.  **STT Engine Selection:** This is the most critical fork-in-the-road decision. The prototype should allow for testing and comparing at least two main options:
    *   **Platform-Native STT:** Using Android's built-in `SpeechRecognizer`. This provides a baseline for performance and ease of integration.
    *   **Third-party On-device Model:** Integrating a library like `Whisper.cpp`. This offers more control and potentially better performance/privacy, but at the cost of integration complexity and binary size.

2.  **Audio Capture Strategy:**
    *   **Continuous vs. VAD-based:** The prototype should test both continuously transcribing audio in chunks and using a Voice Activity Detector (VAD) to only transcribe when speech is present. This will allow us to measure the battery life savings from VAD against the complexity it adds.

3.  **Background Processing Architecture:**
    *   The prototype will force us to choose and validate a robust architecture for background audio processing using a Foreground Service, ensuring the app is reliable and not prematurely killed by the Android OS.

## Summary
This design:
- Completely prevents unwanted AI interruptions
- Provides long-running, continuous awareness
- Is feasible with today’s mobile hardware
- Offers customizable levels of intelligence and privacy
- Achieves your goal: a “mostly silent AI companion” that speaks **only when asked**

## Candidate Names
- SmartLog
- Aictually
- Ghostwise
- LogSmart
- Logia
- LittleBrother
- SilentPartner
- Wisper
- Refairee
- Replaygeist

### Themes

- Silent/Passive AI
- Eavesdropper/Spy (lovable characters)
- Friendly Smart Ghost
- Knowledge Sources (Google, Wikipedia, etc.)
- Oracle/Sage/Esoteric Wisdom
- Referee
- Replay/Recap
