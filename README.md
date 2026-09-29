# 🎙️ Vocall_hdp - Native Audio Function Call Sandbox

**Bandar Abbas Smart Voice Calls** — An interactive sandbox for experimenting with native audio streaming and function calling using the Google Gemini 2.5 Live API.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Made with TypeScript](https://img.shields.io/badge/TypeScript-5.8+-3178c6)](https://www.typescriptlang.org/)
[![React 19](https://img.shields.io/badge/React-19.1-61dafb)](https://react.dev/)
[![Vite 6](https://img.shields.io/badge/Vite-6-646cff)](https://vitejs.dev/)

---

## 🚀 Quick Start

### Prerequisites

- **Node.js** 16+
- **Google Gemini API Key** — Get one at [Google AI Studio](https://aistudio.google.com)

### Installation

```bash
# Clone the repository
git clone https://github.com/mahdihadari59-lgtm/Vocall_hdp.git
cd Vocall_hdp

# Install dependencies
npm install

# Set your API key
export GEMINI_API_KEY=your-api-key-here

# Start the dev server
npm run dev
```

The app will open at **http://localhost:3000**

---

## 📖 What You Can Do

### 🎤 Real-Time Voice Conversations
Have natural, low-latency conversations with the Gemini 2.5 Flash model using native audio streaming.

### ⚙️ Dynamic Function Calling
Define custom AI "tools" (functions) on the fly. The AI can request to use them, and you respond with data to guide the conversation.

### 📋 Pre-Built Templates
Choose from three ready-to-use assistant templates:
- **Customer Support** — Helpful, concise support agent
- **Personal Assistant** — Proactive and efficient assistant
- **Navigation System** — Clear, accurate directions provider

### 🎚️ Customization
- Change the system prompt to define AI personality
- Select from 20+ voice options (Zephyr, Puck, Charon, Luna, Nova, etc.)
- Enable/disable tools on the fly
- Edit function parameters and descriptions

### 📊 Conversation Logging
View full conversation history with timestamps, transcriptions (input and output), and tool use requests/responses.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│          React App (App.tsx)                │
│  ┌─────────────────────────────────────────┐│
│  │   LiveAPIProvider (Context)             ││
│  │  ┌──────────────────────────────────────┤│
│  │  │ GenAILiveClient (WebSocket Connection)
│  │  │  • connect()  → Gemini Live API      ││
│  │  │  • send()     → User input           ││
│  │  │  • listen()   → Server events        ││
│  │  └──────────────────────────────────────┤│
│  │   Zustand Stores                        ││
│  │  ├─ useSettings (prompt, model, voice)  │
│  │  ├─ useTools (function definitions)     │
│  │  ├─ useLogStore (conversation history) │
│  │  └─ useUI (sidebar state)               │
│  └─────────────────────────────────────────┘│
│                                             │
│  ┌─────────────────────────────────────────┐│
│  │      UI Components                      ││
│  │  ├─ Sidebar (Settings, Tools Editor)    │
│  │  ├─ StreamingConsole (Chat Display)     │
│  │  ├─ ControlTray (Play/Pause/Stop)       │
│  │  └─ ErrorScreen (Status & Errors)       │
│  └─────────────────────────────────────────┘│
│                                             │
│  ┌─────────────────────────────────────────┐│
│  │      Audio Pipeline                     ││
│  │  ├─ AudioRecorder (Mic → PCM16)        │
│  │  └─ AudioStreamer (PCM16 → Web Audio)   │
│  └─────────────────────────────────────────┘│
└─────────────────────────────────────────��───┘
```

### Core Modules

#### 📡 **GenAILiveClient** (`lib/genai-live-client.ts`)
Manages the WebSocket connection to Google's Gemini Live API.

**Key Methods:**
- `connect(config)` — Establish connection with system prompt & voice
- `send(parts)` — Send user input (text/audio)
- `sendRealtimeInput(chunks)` — Stream audio in real-time
- `sendToolResponse(response)` — Respond to AI's tool call requests

**Events:**
```typescript
client.on('audio', (data: ArrayBuffer) => { /* PCM16 audio chunk */ });
client.on('content', (content: LiveServerContent) => { /* Text/response */ });
client.on('toolcall', (call: LiveServerToolCall) => { /* AI wants to use a tool */ });
client.on('inputTranscription', (text, isFinal) => { /* User speech → text */ });
client.on('outputTranscription', (text, isFinal) => { /* AI response text */ });
```

#### 🔊 **AudioStreamer** (`lib/audio-streamer.ts`)
Handles playback of received audio using the Web Audio API.

**Features:**
- Converts PCM16 (raw audio) → Float32Array (Web Audio format)
- Queues and schedules buffers for smooth playback
- Supports AudioWorklet nodes for custom audio processing
- Graceful stop with fade-out

**Usage:**
```typescript
const streamer = new AudioStreamer(audioContext);
streamer.addPCM16(uint8ArrayChunk);
streamer.onComplete = () => console.log('Done playing');
```

#### 🎙️ **AudioRecorder** (`lib/audio-recorder.ts`)
Captures microphone input and encodes it for streaming.

#### 🏪 **State Management** (`lib/state.ts`)
Zustand stores for reactive state:

```typescript
// Settings store
const { systemPrompt, model, voice, setSystemPrompt } = useSettings();

// Tools store
const { tools, toggleTool, addTool, removeTool, updateTool } = useTools();

// Conversation logs
const { turns, addTurn, clearTurns } = useLogStore();
```

#### 🛠️ **Pre-Built Tools** (`lib/tools/`)
Function definitions for each template:

**customer-support.ts**
- Order Status Lookup
- Billing Issue Resolution
- Return Request Processing

**personal-assistant.ts**
- Schedule Meeting
- Send Reminder
- Get Weather

**navigation-system.ts**
- Get Route
- Calculate Distance
- Find Nearby Location

---

## 🎯 Usage Examples

### Example 1: Start a Conversation

```typescript
const { client, connected } = useLiveAPIContext();

// Connect with custom prompt
await client.connect({
  systemPrompt: "You are a friendly assistant.",
  generationConfig: {
    temperature: 1,
    topK: 40,
    topP: 0.95,
  }
});

// Send text
client.send({ role: 'user', parts: [{ text: 'Hello!' }] });
```

### Example 2: Listen to Tool Calls

```typescript
const { client } = useLiveAPIContext();

client.on('toolcall', (toolCall: LiveServerToolCall) => {
  console.log(`AI wants to call: ${toolCall.name}`);
  console.log('Arguments:', toolCall.functionCalls);
  
  // Respond
  client.sendToolResponse({
    functionResponses: [
      {
        name: toolCall.name,
        response: { result: 'Tool executed successfully' }
      }
    ]
  });
});
```

### Example 3: Stream Audio

```typescript
const { client } = useLiveAPIContext();
const audioContext = new (window.AudioContext || (window as any).webkitAudioContext)();
const streamer = new AudioStreamer(audioContext);

client.on('audio', (audioData: ArrayBuffer) => {
  const uint8 = new Uint8Array(audioData);
  streamer.addPCM16(uint8);
});

// Play audio
await streamer.resume();
```

---

## 📋 File Structure

```
Vocall_hdp/
├── components/
│   ├── Header.tsx                    # App header & branding
│   ├── Sidebar.tsx                   # Settings panel
│   ├── Modal.tsx                     # Reusable modal
│   ├── ToolEditorModal.tsx           # Function call editor
│   ���── console/
│   │   └── control-tray/ControlTray.tsx  # Play/Stop controls
│   └── demo/
│       ├── ErrorScreen.tsx           # Error boundary
│       ├── streaming-console/        # Main chat UI
│       ├── popup/                    # Toast notifications
│       └── welcome-screen/           # Setup screen
├── contexts/
│   └── LiveAPIContext.tsx            # Gemini API context provider
├── hooks/
│   └── media/use-live-api.ts         # Hook for Live API
├── lib/
│   ├── genai-live-client.ts          # Core API client
│   ├── audio-streamer.ts             # Audio playback
│   ├── audio-recorder.ts             # Microphone input
│   ├── state.ts                      # Zustand stores
│   ├── constants.ts                  # Config & voice list
│   ├── utils.ts                      # Helper functions
│   ├── prompts.ts                    # System prompts
│   ├── tools/
│   │   ├── customer-support.ts
│   │   ├── personal-assistant.ts
│   │   └── navigation-system.ts
│   ├── worklets/                     # AudioWorklet scripts
│   └── audioworklet-registry.ts      # Worklet management
├── App.tsx                           # Root component
├── index.tsx                         # React entry point
├── index.html                        # HTML template
├── index.css                         # Styling
├── vite.config.ts                    # Vite configuration
├── tsconfig.json                     # TypeScript config
├── package.json                      # Dependencies
└── README.md                         # This file
```

---

## ⚙️ Configuration

### Environment Variables

```bash
# Required
GEMINI_API_KEY=your-api-key-here

# Optional
# Port (default: 3000)
PORT=3000
```

### Vite Config (`vite.config.ts`)

- **Dev server:** http://0.0.0.0:3000
- **React plugin:** Enabled for JSX/TSX support
- **Path alias:** `@/*` resolves to project root

### TypeScript Config (`tsconfig.json`)

- **Target:** ES2022
- **Module:** ESNext (ESM)
- **JSX:** React 17+ (automatic runtime)
- **Path mapping:** `@/*` → `./*`

---

## 🎨 Customization Guide

### Adding a New Tool

1. **Edit sidebar:**
   ```typescript
   // In Sidebar.tsx, click "Add function call"
   // Or programmatically:
   useTools.getState().addTool();
   ```

2. **Define in state:**
   ```typescript
   const newTool: FunctionCall = {
     name: 'my_function',
     description: 'Does something useful',
     isEnabled: true,
     parameters: {
       type: 'OBJECT',
       properties: {
         param1: { type: 'STRING', description: 'First param' }
       }
     }
   };
   ```

3. **Listen for calls:**
   ```typescript
   client.on('toolcall', (call) => {
     if (call.name === 'my_function') {
       // Handle it
     }
   });
   ```

### Changing System Prompt

```typescript
const { systemPrompt, setSystemPrompt } = useSettings();

// Via UI: Edit in sidebar
// Or programmatically:
setSystemPrompt('You are a helpful Spanish tutor.');
```

### Adding a New Voice

Edit `lib/constants.ts`:
```typescript
export const AVAILABLE_VOICES = ['Zephyr', 'Puck', ..., 'MyNewVoice'];
```

---

## 🐛 Troubleshooting

### ❌ "API Key Missing"
Make sure `GEMINI_API_KEY` is set:
```bash
export GEMINI_API_KEY=sk-...
npm run dev
```

### ❌ "Microphone Access Denied"
- Check browser permissions (Settings → Privacy → Microphone)
- HTTPS is required in production

### ❌ "No Audio Output"
- Check browser's output volume
- Verify `AudioStreamer.resume()` was called
- Check browser console for errors

### ❌ "Connection Timeout"
- Verify API key is valid
- Check internet connection
- Try a different browser/tab

### ❌ Tools Not Calling
- Ensure tool is **enabled** in sidebar
- Verify tool `description` and `parameters` are clear
- Check `isEnabled: true` in state

---

## 🧪 Development

### Scripts

```bash
npm run dev        # Start dev server with hot reload
npm run build      # Build for production
npm run preview    # Preview production build
```

### Build Output

Vite produces:
- `dist/index.html` — Bundled app
- `dist/assets/` — JS, CSS chunks
- All optimized for performance

### Debugging

1. **Open DevTools:** F12 or Cmd+Opt+I
2. **Live API events:** Check Console logs from `GenAILiveClient`
3. **State inspection:** Use React DevTools + Zustand extension
4. **Network tab:** Watch WebSocket connection to Gemini Live API

---

## 🔐 Security Notes

- **API Key:** Never commit `.env` files. Use environment variables only.
- **CORS:** The app runs on localhost for development. Production deployments need proper CORS headers.
- **Data Privacy:** Audio is streamed directly to Google's servers. Review [Google Privacy Policy](https://policies.google.com/privacy).

---

## 📦 Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `react` | ^19.1.0 | UI framework |
| `react-dom` | ^19.1.0 | DOM rendering |
| `@google/genai` | ^1.4.0 | Gemini API client |
| `zustand` | ^5.0.5 | State management |
| `eventemitter3` | ^5.0.1 | Event system |
| `lodash` | ^4.17.21 | Utilities |
| `classnames` | ^2.5.1 | CSS class merging |
| `vite` | ^6.3.5 | Build tool |
| `typescript` | ~5.8.2 | Type safety |

---

## 🔗 Resources

- [Google Gemini API Docs](https://ai.google.dev/docs)
- [Gemini 2.5 Live API](https://ai.google.dev/api/genai)
- [React Documentation](https://react.dev/)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Zustand Docs](https://github.com/pmndrs/zustand)

---

## 📝 License

This project is licensed under the **Apache License 2.0** — See [LICENSE.md](LICENSE.md) for details.

Copyright © 2024 Google LLC

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Commit your changes (`git commit -m 'Add my feature'`)
4. Push to the branch (`git push origin feature/my-feature`)
5. Open a Pull Request

---

## 📧 Support

For issues, questions, or feature requests:
- 🐛 [Open an Issue](https://github.com/mahdihadari59-lgtm/Vocall_hdp/issues)
- 💬 [Start a Discussion](https://github.com/mahdihadari59-lgtm/Vocall_hdp/discussions)

---

<div align="center">

**Built with ❤️ using React, Vite, and Google's Gemini 2.5 Live API**

[⬆ back to top](#-vocallhdp---native-audio-function-call-sandbox)

</div>
