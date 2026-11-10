# Aether AI — ChatGPT-Style Conversational & Image Studio

A full-featured, aesthetically refined ChatGPT-style conversational AI application powered by Gemini, featuring real-time Server-Sent Events (SSE) streaming, integrated AI Image Generation (`gemini-3.1-flash-lite-image`), specialized AI personas, model switching, and a live interactive code preview sandbox.

## User Review & Critical Decisions

> [!IMPORTANT]
> The following user preferences were confirmed:
> - **Visual Theme**: Modern Dark interface featuring a deep charcoal sidebar (`#171717`), obsidian canvas (`#0d0d0d`), and refined message bubbles.
> - **Conversational Chat**: Fast chat with model selector (`gemini-3.8-flash`, `gemini-3.1-flash-lite`, `gemini-4-argon`), specialized AI personas, and real-time streaming text generation.
> - **Image Generation Capability (NEW)**: Direct inline image creation (like ChatGPT's DALL-E) powered by `gemini-3.1-flash-lite-image` with aspect ratio choices (`1:1`, `16:9`, `4:3`, `3:4`, `9:16`), prompt enhancements, full-screen lightbox preview, and one-click image download.
> - **Code & Web Artifacts**: Syntax-highlighted code blocks with language indicators, one-click clipboard copy, and an interactive live sandbox modal for HTML/SVG web components.

---

### 1. Overview & Core Concept

- **What It Does**: Provides a ChatGPT-style conversational workspace with multimodal intelligence: conversational reasoning, code sandbox execution, and text-to-image synthesis. Users can chat normally, ask to generate images ("draw a futuristic city", "create an image of..."), switch personas, attach files, or use dedicated studio tools.
- **Target Audience / Persona**: Anyone looking for a ChatGPT experience equipped with image generation, coding assistance, and creative writing.
- **Key Value**: Fluid chat UI with streaming responses, inline visual image generation, code sandbox previews, local chat history persistence, and custom personas.

---

### 2. User Experience & Visual Design

#### Key User Flows
1. **Conversational Chat & Inline Images**: User types a prompt. If the prompt requests an image (or if the user toggles the "Create Image" badge), the app calls the image generation endpoint (`gemini-3.1-flash-lite-image`) and displays the generated image inline with high-res zoom, aspect ratio badge, prompt details, and download button.
2. **Streaming Text & Code Execution**: For technical, creative, or general prompts, the AI streams responses in real-time. HTML/SVG snippets include a "Preview" button that runs inside a live sandboxed iframe.
3. **Sidebar History & Organization**: Collapsible charcoal sidebar with search, session renaming, deletion, export (JSON/Markdown), and one-click "New Chat".
4. **Model & Mode Selector**: Top bar with Gemini model picker and Persona selector (General, Staff Engineer, Creative Wordsmith, Visual Artist, Research Analyst, Socratic Mentor).
5. **Prompt Templates & Image Presets**: Instant prompt suggestions for both text queries and visual concepts (e.g. isometric 3D render, studio product photo, watercolor illustration, cyberpunk landscape).

#### Visual Identity & Theme
- **Color Palette**:
  - Obsidian Canvas: `#0d0d0d`
  - Charcoal Sidebar: `#171717`
  - Elevated Surfaces & Inputs: `#212121`
  - Dividers & Borders: `#2f2f2f`
  - Accents: Vibrant Indigo (`#6366f1`) and Emerald (`#10b981`)
- **Typography**: `Plus Jakarta Sans` for UI & text; `JetBrains Mono` with `tabular-nums` for code and metadata.

---

### 3. Key Product Decisions & Trade-Offs

- **Server-Side Image Generation with `gemini-3.1-flash-lite-image`**:
  - *Chosen Approach*: Call `@google/genai` on the server using `gemini-3.1-flash-lite-image` with configurable `imageConfig: { aspectRatio }`, returning the base64 image inline.
  - *Why*: Official recommended model for general image generation and editing tasks; fast and reliable.
- **Automatic Intent Detection + Manual Toggle**:
  - *Chosen Approach*: Users can click an "Image Mode" toggle in the input bar or simply say "generate an image of..." / "draw...". The system intelligently routes image generation requests to the image pipeline while retaining regular chat streaming.
- **Sandboxed Artifact Preview**:
  - *Chosen Approach*: Safe iframe container for HTML/SVG execution with injected Tailwind CSS support.

---

### 4. Technical Architecture & Data Strategy

```
┌──────────────────────────────────────────────────────────┐
│                      Client Browser                      │
│                                                          │
│  ┌──────────────┐  ┌──────────────────────────────────┐  │
│  │ ChatSidebar  │  │ ChatView (768px centered column) │  │
│  │ - History    │  │ - TopBar (Model & Persona pick)  │  │
│  │ - Search     │  │ - MessageList (Text & Images)    │  │
│  │ - New Chat   │  │ - InputBar (Text/Voice/ImageMode)│  │
│  └──────┬───────┘  └────────────────┬─────────────────┘  │
│         │                           │                    │
│         │     localStorage State    │                    │
│         └──────────────┬────────────┘                    │
└────────────────────────┼─────────────────────────────────┘
                         │
        ┌────────────────┴────────────────┐
        │ /api/chat/stream (SSE)          │ /api/generate-image
        ▼                                 ▼
┌──────────────────────────────────────────────────────────┐
│                  Express Backend Server                  │
│                                                          │
│  • GoogleGenAI client (User-Agent: aistudio-build)       │
│  • Streaming chat controller                             │
│  • Image generation controller                           │
└────────────────────────┬─────────────────────────────────┘
                         │
        ┌────────────────┴────────────────┐
        │                                 │
        ▼                                 ▼
┌────────────────────────┐      ┌──────────────────────────┐
│ gemini-3.8-flash       │      │ gemini-3.1-flash-lite-   │
│ (Chat & Reasoning)     │      │ image (Image Studio)     │
└────────────────────────┘      └──────────────────────────┘
```

#### State & Component Mapping
- `server.ts`: SSE chat streaming endpoint (`/api/chat/stream`) + image generation endpoint (`/api/generate-image`).
- `src/types.ts`: Extended `Message` interface with `generatedImages?: { url: string; prompt: string; aspectRatio: string }[]`.
- `src/components/ChatSidebar.tsx`: Charcoal sidebar with session history, search, and new chat actions.
- `src/components/ChatInput.tsx`: Floating input pill with text input, image mode toggle, aspect ratio selector, voice dictation, and attachments.
- `src/components/MarkdownView.tsx`: Code formatting with one-click copy and live iframe preview trigger.
- `src/components/ImageLightboxModal.tsx`: Full-screen modal for inspecting, zooming, and downloading generated images.
- `src/components/CodeCanvasModal.tsx`: Live sandboxed interactive preview for HTML/SVG code snippets.
