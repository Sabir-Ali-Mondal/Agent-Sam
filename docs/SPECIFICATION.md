# *Agent-Sam*

***

# SPECIFICATION: AI Desktop Control Agent (2025–2026 Edition)

This system is a local, highly advanced AI desktop automation agent that observes the screen natively, deeply understands UI elements through vision, seamlessly executes human-like actions, and dynamically learns workflows until the user goal is completed. 

Inspired by cutting-edge architectures like Anthropic's Computer Use, Open Interpreter, and Qwen-Agent, this system is designed for absolute precision, safety, and self-correction.

---

## 1. Runtime Environment & Dependencies
**Runtime Environment**
*   **Programming language:** Python 3.14
*   **Local model runtime:** Embedded Native Python Inference Engine (`transformers` + `qwen-vl-utils` or `llama-cpp-python` for CPU/GPU split). No external Ollama dependency.
*   **Primary Logic:** Native Vision-Language Processing (No external OCR required)
*   *All processing, vision, and reasoning happens 100% locally within the Python process.*

**AI Model Selection**
During setup, the user selects the AI model to run the agent.
After selection, the installer automatically downloads the model weights (Safetensors/GGUF) directly from Hugging Face to the local `data/models/` directory.
All models support a context window of >=256K tokens.

Select AI Model:

1. Qwen3.5-2B-Instruct      (Fast / Low RAM)
2. Qwen3-VL-2B-Instruct     (Vision Model)
3. Qwen3.5-4B-Instruct      (Recommended Default)
4. Qwen3-VL-4B-Instruct     (Advanced Vision)

Enter number:

**Core Libraries Used**
*   **Embedded Inference:** `transformers`, `accelerate`, `llama-cpp-python` (Provides programmatic control over VRAM loading and context caching).
*   **Screen capture:** MSS (or DXGI for hardware acceleration)
*   **Mouse and keyboard automation:** PyAutoGUI
*   **User input monitoring:** pynput
*   **Computer vision:** OpenCV (For drawing Set-of-Mark bounding boxes only)
*   **Windows UI accessibility:** pywinauto (Fallback only)
*   **System Environment:** `ctypes` (For Windows DPI Scaling extraction)
*   **System tray:** pystray
*   **Dashboard UI:** CustomTkinter
*   **Image processing:** Pillow
*   **Array processing:** NumPy
*   **Vector Database (NEW):** ChromaDB / FAISS (For long-term semantic memory)
*   **Relational Database (NEW):** SQLite (For session/log storage)

---

## 2. Installation & Bootstrapping (One Click)
User runs `run_app.bat`. The installer strictly follows this flow to provide a zero-configuration experience:
1.  **Check Python 3.14:** Installs if not present, continues if installed.
2.  **Virtual Environment:** Creates isolated `.venv`.
3.  **Dependencies:** Installs required packages from `requirements.txt` (including PyTorch/Llama.cpp based on hardware detection).
4.  **Model Download:** Prompts user for Model Selection, then executes a direct Hugging Face pull to download the model weights to `data/models/`.
    *   *Success* -> Continue.
    *   *Failure* -> Retry (3 times), then Abort if still failing.
5.  **Launch Agent:** Starts system tray icon, loads the model directly into VRAM (locking it for zero-latency inference), and starts background services.
*User performs no manual setup or terminal commands.*

---

## 3. User Goal Input System (The 10 Parameters)
The system accepts user instructions through multiple integrated channels:
*   **Dashboard Chatbox:** Standard text input UI.
*   **Command Input:** CLI arguments on launch.
*   **Voice (Optional integration plug-in):** Whisper local STT API.
*   **API / Webhook:** REST endpoint for external scripts to pass goals.

---

## 4. Full Unified Application Flowchart
*(Control Flow + Data Flow + Memory + AI Interaction + 2026 Upgrades + Shutdown Flow)*

```text
USER STARTS APPLICATION & INPUTS GOAL
        │
        ▼
run_app.bat
        │
        ▼
BOOTSTRAP INSTALLER
        │
        ├─ Check Python 3.14
        ├─ Create virtual environment
        ├─ Install dependencies (PyTorch/Inference engines)
        ├─ Prompt User & Download Selected AI Model via Hugging Face
        │      ├─ Success -> Continue
        │      └─ Failure -> Retry (3) / Abort
        │
        ▼
LAUNCH AGENT & SYSTEM INITIALIZATION
        │
        ├─ Load config.yaml
        ├─ Initialize structured logging
        ├─ Initialize safety guard
        ├─ Initialize performance controller
        ├─ Initialize memory manager (SQLite + Vector DB)
        ├─ Initialize replay recorder
        ├─ Initialize screenshot engine (with DPI Scaling awareness)
        ├─ Initialize grid engine
        ├─ Initialize cursor tracker
        ├─ Load AI Model into VRAM (Keep-Alive lock applied)
        ├─ Initialize JSON repair layer & Loop detector
        ├─ Initialize user input monitor
        └─ Initialize system tray + dashboard
        │
        ▼
LOAD MEMORY STRUCTURES
        │
        ├─ Short-term memory (current goal + UI state)
        ├─ Session memory (SQLite action history)
        ├─ Long-term memory (Vector DB workflow retrieval)
        └─ UI knowledge cache
        │
        ▼
MULTI MONITOR DISCOVERY
        │
        ├─ Detect monitors & Read resolutions
        ├─ Query Windows DPI Scaling Factor (ctypes.windll.shcore)
        ├─ Create global coordinate map (Physical to Logical Pixels)
        └─ Store monitor layout
        │
        ▼
ENTER MAIN RUNTIME LOOP (0.4s - 0.8s intervals)
        │
        └──────── LOOP START ────────
                    │
                    ▼
SCREEN CAPTURE PIPELINE & CACHING
        │
        ├─ Update Frame Cache (last_frame, previous_frame, frame_diff)
        ├─ Capture screen(s) & Merge monitors
        ├─ Mask agent UI
        ├─ Foveated Vision (Downscale periphery, high-res active window)
        ├─ Apply Compression (75% scale, 70 JPEG quality, max 400KB)
        └─ Save frame
        │
        ▼
GRID OVERLAY & SET-OF-MARK (SOM) VISUAL GROUNDING
        │
        ├─ Divide screen into primary tiles (A, B, C, D)
        ├─ Subdivide tiles into 10x10 Grid Cells (e.g., A01-A100)
        ├─ Overlay numeric bounding-box tags directly on image (SoM)
        └─ Create pixel -> grid translator
        │
        ▼
NATIVE VLM PERCEPTION (NO EXTERNAL OCR)
        │
        ├─ Unified visual token processing (In-process inference)
        ├─ Spatial 2D Grounding: Extract element coordinates natively using Qwen <box> syntax
        ├─ Semantic UI Mapping: Detect search bars, menus, buttons
        └─ Element State Tracking: enabled/focused/checked/filled
        │
        ▼
CURSOR TRACKING
        │
        ├─ Detect pixel coordinates & Convert to grid
        │
        ▼
CONTEXT BUILDER & RAG MEMORY INJECTION
        │
        ├─ User goal & Current subgoal
        ├─ Screen summary & Visible UI elements
        ├─ Vector DB query: Retrieve past successful workflows for context
        ├─ Cursor position & Last actions
        └─ Context Size Limiter (Top 20 elements, 5-action history limit)
        │
        ▼
PROMPT BUILDER & AI REASONING
        │
        ├─ Insert system prompt & JSON action schema (Includes Below-the-Fold Scroll rule)
        ├─ Insert UI context & Send Prompt -> Embedded Native AI Engine
        │
        ▼
RECEIVE RESPONSE & SELF-REFLECTION LOOP (CRITIC)
        │
        ├─ Critic Model check: "Does this action match visual reality?"
        │
        ▼
JSON REPAIR LAYER & LOOP DETECTOR
        │
        ├─ Valid JSON ? (If No -> Attempt repair -> Request regeneration)
        ├─ Same action repeated >3 ? (If Yes -> Thought Cloning/Demonstration Mode)
        │
        ▼
SAFETY GUARD & ACTION VALIDATION
        │
        ├─ Violates Safety Rules? (Delete sys files/shutdown/registry)
        ├─ Element exists & Bounding box valid?
        ├─ Window focused & Cursor reachable?
        │
        ▼
CURSOR MOVEMENT ENGINE & VERIFICATION
        │
        ├─ Translate AI Grid/Box coordinates via Windows DPI Scaling multiplier
        ├─ Human-like bezier movement (Speed profile + delay)
        ├─ Micro screenshot at target -> Confirm element
        └─ If mismatch -> Ask AI again
        │
        ▼
ACTION EXECUTION (TEMPORAL CHUNKING & HEURISTICS)
        │
        ├─ Apply Double-Click Heuristic (Auto-upgrade left_click on desktop icons)
        ├─ Execute Mouse/Keyboard/System ops
        ├─ Execute Macro-actions (e.g., click + type + enter)
        │
        ▼
USER INTERFERENCE DETECTOR
        │
        ├─ Mouse moved or Keyboard input? -> Pause AI if detected
        │
        ▼
SCREEN CHANGE DETECTOR
        │
        ├─ Pixel difference (>3%)
        ├─ SSIM comparison (<0.92 threshold)
        ├─ Perceptual hash (Distance > 10)
        ├─ Screen changed? (Yes -> Continue, No -> Retry action)
        │
        ▼
MEMORY UPDATE FLOW & REPLAY RECORDER
        │
        ├─ Save action -> SQLite Session memory
        ├─ Update state -> Short memory
        ├─ Save success -> Vector DB Long memory
        ├─ Save to Replay (Timestamp, Screenshot, JSON, Action)
        │
        ▼
PERFORMANCE CONTROLLER & GOAL CHECK
        │
        ├─ Monitor CPU -> Adjust screenshot rate
        ├─ Goal complete -> STOP -> INITIATE SHUTDOWN
        └─ Not complete -> RETURN LOOP
        │
        ▼
GRACEFUL SHUTDOWN FLOW
        │
        ├─ User presses STOP or Goal Complete
        ├─ Runtime engine stop signal triggered
        ├─ Flush memory buffers to disk
        ├─ Close SQLite and Vector DB connections
        ├─ Save final replay session metadata
        ├─ Terminate screenshot and AI worker threads
        ├─ Unload Model from VRAM
        └─ Safely close Tray UI / Dashboard
```

---

## 5. Architectural Upgrades (2025-2026 Cutting Edge)
1.  **Set-of-Mark (SoM) / Visual Grounding Injection:** Bounding boxes with numeric IDs are drawn *directly onto the compressed screenshot sent to the AI*. The AI doesn't just guess coordinates; it outputs `"target": {"element_id": "42"}`, bridging the gap between reasoning and pixel execution natively.
2.  **Temporal Action Chunking (Macro-Actions):** Instead of one click per loop, the AI can output chunked commands. Example: `["click_element(42)", "type_text('hello')", "press_key('enter')"]`. Reduces latency by 3x on form-filling.
3.  **Self-Correction / Reflection Prompting Loop:** Before executing a destructive or complex click, a lightweight secondary prompt asks the local model: *"Review your proposed action against the visual state. Is this hallucinated?"* Drastically reduces misclicks.
4.  **Semantic Memory RAG (Retrieval-Augmented Generation):** The system vectorizes successful task workflows. If the user asks "Book a flight", the system queries the Vector DB for `book_flight_workflow`, injecting the exact required UI path into the AI context window.
5.  **Dynamic Resolution Scaling (Foveated Vision):** To save massive LLM context tokens, peripheral monitors/tiles are heavily downscaled, while the tile containing the active cursor/target window is rendered in maximum, crisp resolution for visual reading.
6.  **Thought Cloning (Demonstration Mode):** If the agent loops or fails, the user takes over the mouse. The agent records the exact click, pairs it with the visual state, and saves it to the Vector DB for one-shot learning.

---

## 6. Internal Data Flow & Context Serialization Format

Main data objects moving continuously through the system (0.4s-0.8s intervals):
`ScreenshotFrame` -> `TileImages` -> `SemanticUIModel` -> `ContextObject` -> `PromptPayload` -> `AIResponse` -> `ActionCommand`

**Context Serialization Format (Data Structure)**
The exact structure serialized and injected into the AI context:
```json
{
  "goal_state": {
    "main_goal": "Turn off Windows Defender",
    "current_subgoal": "Open Settings menu"
  },
  "screen_state": {
    "active_window": "Desktop",
    "foveated_tile": "A",
    "visible_elements":[
      {"id": "12", "type": "button", "state": "enabled"}
    ]
  },
  "cursor_state": {
    "x": 960, "y": 540, "tile": "B", "cell": "B01"
  },
  "history":[
    {"action": "press_key", "target": "win", "result": "Start menu opened"}
  ]
}
```

---

## 7. Global Configuration System
All parameters, thresholds, and limits are strictly maintained in a centralized `config.yaml` file managed by `core/config_manager.py` to prevent hardcoded scatter and ensure simple maintainability.

**Example `config.yaml` Structure:**
```yaml
runtime:
  loop_interval_min: 0.4
  loop_interval_max: 0.8

vision:
  compression_scale: 0.75
  jpeg_quality: 70
  max_frame_size_kb: 400

screen_change_detection:
  pixel_threshold: 3
  ssim_threshold: 0.92
  phash_distance: 10

ai:
  timeout_seconds: 20
  max_retries: 3
  retry_delay: 1.5

context_limits:
  max_ui_elements: 20
  action_history: 5
```

---

## 8. Screen Capture, Grid & Monitors

**Multi Monitor Support**
Automatically detects layout (e.g., monitor1 1920x1080, monitor2 1920x1080) and merges into one global coordinate space.

**Windows DPI Scaling Trap Prevention**
Windows often scales UI by 125% or 150%. MSS captures at native resolution, but PyAutoGUI moves the mouse based on scaled resolution. The agent uses `ctypes.windll.shcore.GetScaleFactorForDevice` to retrieve the exact scaling factor and applies a strict mathematical multiplier to all AI-generated coordinates before execution, preventing off-target clicks.

**Grid Size Configuration**
*   Screen is divided into primary tiles (A, B, C, D).
*   **Exact Density:** `10x10` cell count per tile. (Cells range from `A01` to `A100`, `B01` to `B100`).

**Screenshot Cache System**
To speed up screen comparison, frames are cached in RAM:
*   `last_frame`: Immediate previous capture.
*   `previous_frame`: N-1 capture.
*   `frame_diff_cache`: Cached delta mask to avoid re-running perception if no pixels changed.

---

## 9. Native Perception & Cursor Systems

**Native VLM Pipeline (In-Process)**
Uses the selected AI Model's native visual processing loaded securely into VRAM. Generates bounding boxes and semantic understanding (SearchBar, Button) directly from the compressed image using Qwen-VL's native `<box>` spatial grounding outputs.

**Cursor Speed Profile & Movement Validation**
Movement is NOT instantaneous teleportation (which triggers anti-bot and UI glitches).
*   **Speed levels:** Fast (0.2s), Normal (0.5s), Slow (1.0s).
*   **Movement easing:** Uses Bezier curves / PyAutoGUI ease-in-out tweens.
*   **Delay simulation:** Millisecond jitter added to mimic human interaction.
*   **Verification:** Move cursor -> Take Micro-Crop (120x120px) -> Verify element exists -> Click.

---

## 10. Context & AI Prompt System

**Context Size Limiter**
Enforces strict token limits: Top 20 high-confidence elements only. History compressed to the last 5 actions.

**System Prompt (Including the Below-the-Fold Scrolling Rule)**
> "You are a highly capable computer control agent. You observe the screen through natively processed vision and Visual Grounding IDs. Your task is to achieve the user goal by selecting the next best logical action.
> Rules: 1. Output ONLY JSON. 2. Use allowed operations only. 3. Never hallucinate elements. 4. Use element_id when available. 5. Prefer safe actions. 6. Avoid repeating actions. 7. If the target element is not visible on the current screen, you MUST output a `scroll_down` action before doing anything else. 8. Stop when goal is achieved."

**AI JSON Action Schema**
```json
{
 "thought_process":[
   "I see the settings icon at ID 23.",
   "The window is currently focused."
 ],
 "main_goal": "...",
 "current_goal": "...",
 "next_goal": "...",
 "action": {
   "name": "left_click",
   "target": {"element_id": "23"},
   "tile": "B",
   "cell": "B12"
 },
 "input": {
   "text": "",
   "keys":[],
   "shortcut":[]
 },
 "verification": {"required": true},
 "state": {"confidence": 0.95, "reason": "Opening settings"}
}
```

---

## 11. Complete Operation & Action Types

**Mouse Operations**
`left_click` / `click`, `double_click`, `right_click`, `middle_click`, `click_and_hold`, `release_click`, `hover`

**Cursor Movement**
`move_cursor`, `move_cursor_relative`, `move_cursor_smooth`, `move_to_grid_cell`, `move_to_element`

**Drag Operations**
`drag` / `drag_start`, `drag_move`, `drag_end`, `drag_and_drop`

**Scrolling**
`scroll_up`, `scroll_down`, `scroll_left`, `scroll_right`, `scroll_to_element`, `scroll_to_top`, `scroll_to_bottom`

**Keyboard Operations**
`type_text`, `press_key`, `hold_key`, `release_key`, `press_hotkey` / `keyboard_shortcut`, `paste_text`, `clear_field`

**Keyboard Key Mapping Table (Logical to PyAutoGUI Strings)**
Provides a complete mapping between logical key names generated by the AI and PyAutoGUI execution codes:
*   **Modifiers:** `ctrl`, `alt`, `shift`, `win` (Windows key)
*   **Function Keys:** `f1`, `f2`, `f3`, `f4`, `f5`, `f6`, `f7`, `f8`, `f9`, `f10`, `f11`, `f12`
*   **Navigation / Arrows:** `up`, `down`, `left`, `right`, `home`, `end`, `pageup`, `pagedown`
*   **Control Keys:** `enter` (or `return`), `esc`, `tab`, `space`, `backspace`, `delete`, `insert`
*   **Shortcuts (Combinations):** Executed securely via `shortcut([...])` arrays:
    *   Copy: `["ctrl", "c"]`
    *   Paste: `["ctrl", "v"]`
    *   Select All: `["ctrl", "a"]`
    *   Save: `["ctrl", "s"]`
    *   Undo: `["ctrl", "z"]`
    *   Task Switch: `["alt", "tab"]`
    *   Close Window: `["alt", "f4"]`

**Text Field Operations**
`focus_input`, `clear_input`, `type_text`, `submit_input`

**Window / System Operations**
`focus_window`, `switch_window`, `maximize_window`, `minimize_window`, `close_window`, `restore_window`, `open_application`, `close_application`, `launch_url`, `open_file`, `save_file`, `wait`, `finish`, `ask_user`, `confused`

---

## 12. Execution, Validation & Safety

**The Double-Click Heuristic**
In Windows, desktop icons and file explorer items require a double-click to open. If the Semantic UI Analyzer identifies the target element as a "Desktop Icon" or "File Explorer Item," the executor will automatically upgrade the AI's `left_click` command to a `double_click`, preventing the agent from getting stuck.

**Action Execution Delay Matrix**
To prevent UI race conditions, allow OS animations to finish, and strictly mimic human interaction speeds, specific delay ranges are hardcoded for action types:
*   **Cursor Movement:** `0.2s - 0.5s` (Calculated dynamically via Bezier distance tweening)
*   **Clicking (down/up):** `0.05s - 0.15s` inside the click; followed by a `0.3s` post-click pause
*   **Typing Text:** `0.02s - 0.08s` delay per individual keystroke
*   **Scrolling:** `0.1s - 0.3s` interval between scroll steps
*   **Dragging:** `0.5s - 1.0s` duration for drag movement; `0.2s` hold before release
*   **Window Switching (Alt+Tab / Focus):** `0.5s - 0.8s` delay to allow OS animation to fully complete
*   **Application Launching:** `2.0s - 5.0s` delay (Dynamic wait polling based on active window detection)

**Safety Guard Rules (`safety_guard.py`)**
Strict hardcoded rules the AI cannot override:
1.  Prevent deleting system files (blocks `del /s C:\Windows\*`).
2.  Prevent system shutdown/restart commands.
3.  Prevent registry edits (blocks `regedit`).
4.  Prevent installing unknown executable payloads from web.

**Action Validation & Focus**
Every action is validated: Element exists? Bounding box valid? Cursor reachable? Window active? (Focus window if needed).

**User Interference Handling**
Monitors `pynput`. Detects mouse move or keystroke. Response: Pause AI -> Update cursor state -> Recapture -> Rebuild Context -> Resume.

**Screen Change Detection Thresholds**
To verify an action worked, exactly one of these thresholds must be met:
*   **Pixel Difference Threshold:** `3%` change in rendered pixels.
*   **SSIM (Structural Similarity) Threshold:** `< 0.92` (meaning structure changed by at least 8%).
*   **Perceptual Hash (pHash) Distance:** `> 10` (significant visual shift).

---

## 13. Error Handling, Loop Detection & Graceful Shutdown

**AI Timeout & Retry Policy**
*   **AI generation timeout:** `20.0s`
*   **Max retries (AI fail/hallucination):** `3`
*   **Retry delay:** `1.5s`

**JSON Repair Layer**
If JSON is invalid: Attempt regex repair -> Retry parsing -> Ask LLM to regenerate.

**Error Handling Matrix**
*   **AI Failure:** Retry, repair JSON, ask for new plan.
*   **Perception Failure:** Zoom tile, expand grid.
*   **Execution Failure:** Reposition cursor, refocus window, retry.

**Loop Detection System**
If `same_action_repeated > 3`: Trigger Loop Detected -> Trigger Thought Cloning / Demonstration Mode -> Ask User for physical intervention.

**Graceful Shutdown System (`system_utils.shutdown_agent()`)**
When the user triggers a stop, or the goal is fully achieved, the system executes a strict teardown sequence:
1. Emit runtime engine stop signal.
2. Flush all active memory buffers to disk.
3. Close SQLite and Vector Database connections securely.
4. Finalize and save replay session metadata.
5. Terminate screenshot and AI worker threads.
6. Instruct the Embedded Inference Engine to dump the VRAM cache and release memory.
7. Safely close Tray UI and Dashboard.

---

## 14. Memory System & Database Formats

Stored in: `data/memory/`

**Short Term Memory (RAM)**
*   **Stores:** current_goal, cursor_position, last_screen, ui_state
*   **Lifetime:** Seconds to minutes (Flushed per session).

**Session Memory (SQLite Database)**
*   **Format:** Relational tables (Actions, Timestamps, Screen_IDs).
*   **Stores:** Exact action history, errors, navigation steps.
*   **Lifetime:** Entire session.

**Long Term Memory (Vector Database - ChromaDB/FAISS)**
*   **Format:** High-dimensional embeddings mapping Goals to Context-Action workflows.
*   **Stores:** Successful workflows, application layouts, semantic patterns.
*   **Lifetime:** Persistent. Used for RAG.

**Memory Update Flow**
Action executed -> Screen changes (Threshold met) -> SQLite logs session -> Short term RAM state updates -> Goal Complete? Embed workflow sequence into Vector DB.

---

## 15. System Monitoring & Debugging

**VRAM Keep-Alive Locking**
Upon initialization, the selected Qwen model is locked securely into memory via the internal inference engine (e.g., setting GPU layer limits in `llama-cpp-python` or pre-allocating KV cache in `transformers`). This ensures instant, zero-latency inference for the entirety of the agent's lifespan, avoiding the typical 10-second reload penalties.

**Runtime Performance Controller**
Monitors CPU dynamically. If CPU usage > 85%, adjusts `screenshot_interval` dynamically to prevent thermal throttling.

**Taskbar Live Console**
Instead of showing raw terminal logs in the background, the agent provides a clean, user-friendly live console natively:
*   Accessible directly via the system tray icon (Right-click -> "Show Console").
*   Displays real-time agent thoughts, parsed JSON actions, confidence scores, and current subgoal.
*   Includes a live mini-preview of the foveated tile crop and overlaid bounding boxes so the user can literally "see what the AI sees."

**Structured Logging & Replay System**
All events logged to `data/logs/`. Sessions saved to `data/replay_sessions/` containing: Timestamp, Cached Screenshot, AI JSON response, Executed action, and exact Cursor track.

---

## 16. Consolidated Project File Structure (Clean & Modular)

```text
agent-sam/
├── run_app.bat
├── requirements.txt
├── config.yaml              <-- (Centralized thresholds, timeouts, settings)
├── installer.py             <-- (Handles Model Menu, Python env setup, Hugging Face weight downloads)
├── main.py                  <-- (Entry point, config loading, and orchestrator)
│
├── runtime/
│   ├── engine.py            <-- (Main runtime loop, CPU performance monitor, loop detector)
│   ├── executor.py          <-- (Action execution, Action Delay Matrix, Double-Click Heuristics)
│   └── observers.py         <-- (Screen change thresholds, reflection critic, JSON repair)
│
├── perception/
│   ├── vision_capture.py    <-- (Screenshot, multi-monitor merge, frame cache, foveated downscaling)
│   ├── spatial_grid.py      <-- (Grid overlay, tile cropping, SoM visual grounding drawing)
│   └── semantic_mapper.py   <-- (Element mapping, state tracking, semantic analyzer)
│
├── ai/
│   ├── llm_client.py        <-- (Embedded Inference Engine, VRAM locking, timeouts, HF model loader)
│   ├── prompt_engine.py     <-- (Prompt builder, multimodal thinking trace, strict JSON serializer/parser)
│   └── action_planner.py    <-- (Goal reasoning, RAG semantic memory injection)
│
├── core/
│   ├── config_manager.py    <-- (Parses config.yaml and distributes global settings)
│   ├── cursor_engine.py     <-- (Tracker, controller, verifier, Bezier paths, DPI Scaling Math)
│   ├── input_manager.py     <-- (User interference handler, Thought Cloning listener, Key Mapping)
│   ├── memory_manager.py    <-- (Short term RAM, SQLite session logging, ChromaDB Vector RAG)
│   ├── safety_guard.py      <-- (Hardcoded Registry/File protection rules)
│   └── app_ui.py            <-- (Taskbar Live Console, CustomTkinter Dashboard, Chatbox, Tray icon)
│
├── utils/
│   ├── image_utils.py       <-- (Compression 75/70/400kb, SSIM, pHash logic math)
│   └── system_utils.py      <-- (Structured logging, replay recorders, timers, Graceful Shutdown)
│
└── data/
    ├── models/              <-- (Local directory where Hugging Face model weights are stored)
    ├── memory/              <-- (session.db, vector_store/)
    ├── logs/
    ├── screenshots/
    └── replay_sessions/
```
