# Aictually - Silent AI Conversation Companion

## 1. Overview
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
- 

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

## 2. Core Components

### 2.1 Audio Capture
- Always-on microphone via background audio mode.
- Optional **VAD (Voice Activity Detection)** to reduce processing cost.
- Audio stored in a **ring buffer** (e.g., keeping last 1–5 minutes of raw audio).

### 2.2 Speech-to-Text (STT)
- Runs every 5–30 seconds on-device for privacy and cost savings.
- Options:
  - **Whisper tiny** (quantized)
  - Platform-native STT (iOS / Android)
- Produces timestamped transcripts and chunk metadata.

### 2.3 Transcript Management
- Stores **last N minutes** of text (rolling window).
- Maintains optional **longer-term summary**:
  - Updated every 1–3 minutes.
  - Helps maintain context without large token usage.

### 2.4 Trigger System
Two trigger types:
1. **Button press** (hardware or on-screen)
2. **Wake phrase** (keyword spotting):
   - e.g., using Picovoice Porcupine-like wake-word engine
   - Extremely low power and runs offline

Trigger event causes:
- Pulling recent transcript.
- Pulling summary state.
- Sending to LLM with a **context-weighted prompt**.

### 2.5 LLM Processing
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

### 2.6 Text-to-Speech (Optional)
- Only triggered after user explicitly requests AI output.
- Keeps the system “mostly silent.”

### 2.7 Privacy & Consent
- Persistent indicator (e.g., “Listening”)
- Ability to pause anytime
- Configurable:
  - Delete raw audio immediately
  - Store only transcripts
  - Store only summaries

---

## 3. Technical Feasibility & Tradeoffs

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

## 4. High-Level Architecture

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

## 5. MVP Scope

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

## 6. Summary
This design:
- Completely prevents unwanted AI interruptions
- Provides long-running, continuous awareness
- Is feasible with today’s mobile hardware
- Offers customizable levels of intelligence and privacy
- Achieves your goal: a “mostly silent AI companion” that speaks **only when asked**
