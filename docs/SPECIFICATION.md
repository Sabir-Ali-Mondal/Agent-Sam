# SPECIFICATION: AI Desktop Control Agent

This system is a local, highly advanced AI desktop automation agent named **agent-sam**. It observes the screen natively, understands UI elements through vision-language processing, executes human-like actions, utilizes a Direct System Execution Layer for high-speed OS operations, and learns workflows until the user goal is completed. 

Inspired by cutting-edge architectures like Anthropic's Computer Use, Open Interpreter, and Qwen-Agent, this system is designed for absolute precision, safety, and self-correction.

---

## 1. Runtime Environment & Dependencies
**Runtime Environment**
*   **Programming language:** Python 3.14
*   **Execution Logic:** All OS integration, vision mapping, and direct execution happens 100% locally within the agent-sam process. Inference logic dynamically routes to either local hardware or a remote Cloud GPU backend.

**Embedded Inference Strategy (Zero User Setup)**
*   **Goal:** Enable the AI model to run automatically within the application with zero user setup, regardless of the user's local hardware constraints.
*   **Plan:** Integrate an optimized inference runtime based on `llama.cpp` directly into the project. Package the runtime and model management system inside the application. On startup, the application checks for the required GGUF model and downloads it automatically if local execution is chosen. The runtime is launched internally as a background component. Alternatively, if the user has a low-end PC, the system seamlessly connects to an external OpenAI-compatible Cloudflare Tunnel hosted on a free Google Colab GPU. The agent logic, automation system, and context pipeline communicate with the selected endpoint for reasoning tasks seamlessly.

**AI Model Selection (Hugging Face Native & Cloud GPU)**
The installer prompts the user to select an execution mode. If a local model is chosen, it downloads the selected GGUF model weights from Hugging Face into `data/models/`. All models support a context window of >=256K tokens.

Select AI Model:[Local Execution - Requires PC GPU/RAM]
1. Qwen3.5-2B-Instruct      (Size: ~4.57 GB | Fast / Low RAM)
2. Qwen3-VL-2B-Instruct     (Size: ~4.27 GB | Vision-Language Optimized)
3. Qwen3.5-4B-Instruct      (Size: ~9.34 GB | Advanced Reasoning)
4. Qwen3-VL-2B-Thinking     (Size: ~4.27 GB | Visual Reasoning + Logic)[Cloud Execution - Runs on Any PC]
5. Cloud GPU Backend        (Google Colab + Cloudflare Tunnel | Zero local hardware required)

**Core Libraries Used**
*   **Inference:** `llama-cpp-python` (Packaged internally for zero-setup local execution) and `requests` (For Cloud GPU API bridging).
*   **Screen capture:** MSS (aware of Windows DPI settings).
*   **Mouse/Keyboard:** PyAutoGUI + `pynput` (for monitoring).
*   **Computer Vision:** OpenCV (Set-of-Mark overlays).
*   **System Environment:** `ctypes`, `psutil`, `shutil`, `winreg` (For Windows DPI Scaling extraction, Application Registry, and Direct System Execution).
*   **Storage:** `ChromaDB` (Vector RAG) and `SQLite` (Session logs).
*   **UI & Mobile Control:** `CustomTkinter`, `pystray`, and `python-telegram-bot` (For secure mobile remote control and interactive permission requests).

---

## 2. Installation & Bootstrapping (Single-Click Setup)
User runs `run_app.bat`. The `installer.py` handles the entire environment setup programmatically, ensuring zero user configuration:
1.  **Venv Setup:** Creates `.venv` and installs `requirements.txt`.
2.  **Hardware Check:** Detects CPU cores and CUDA availability for GPU acceleration.
3.  **Model Selection:** Prompts user to select from the 5 execution modes.
4.  **Hugging Face Download / Tunnel Link:** If options 1-4 are selected, uses `huggingface_hub` to download weights directly into `agent-sam/data/models/`. If option 5 is selected, the console asks the user to paste their Cloudflare Tunnel URL from Google Colab.
5.  **Initialization:** The internal `llama.cpp` runtime is launched (if local) loading the model into VRAM with a Keep-Alive lock. The internal inference interface is exposed to the agent system.

---

## 3. User Goal Input & Interactive Permission System
The system accepts user instructions and requests real-time interactive feedback through multiple channels:
*   **Telegram Bot (Mobile Remote Control - Primary):** Secure, zero-frontend remote control from any smartphone. The agent listens via `python-telegram-bot` using long polling. Securely restricted to a specific Telegram User ID defined in `config.yaml`.
    *   **Goal Input:** User can send text commands or Whisper-transcribed voice notes to start workflows.
    *   **Interactive Prompts:** If the AI encounters a missing variable (e.g., "What is the password for this form?") or attempts a destructive action (e.g., "Can I delete the 'taxes_2024' folder?"), the agent pauses the runtime loop, sends a push notification via Telegram to the user, and waits for a text reply (Yes/No/Information data) before resuming execution.
*   **Dashboard Chatbox:** Standard text input UI running locally via CustomTkinter (Mirrors Telegram permission prompts locally).
*   **Command Input:** CLI arguments on launch.
*   **API / Webhook:** REST endpoint for external scripts to pass goals locally.

---

## 4. Full Unified Application Flowchart
*(Control Flow + Data Flow + Memory + AI Interaction + Direct Execution + Shutdown Flow)*

```text
USER STARTS APPLICATION & INPUTS GOAL (via Desktop UI or Mobile Telegram Bot)
        │
        ▼
run_app.bat
        │
        ▼
BOOTSTRAP INSTALLER
        │
        ├─ Check Python 3.14
        ├─ Create virtual environment
        ├─ Install dependencies (llama-cpp-python, python-telegram-bot)
        ├─ Prompt User for Model (Local GGUF Download OR Cloudflare Colab URL)
        │      ├─ Success -> Continue
        │      └─ Failure -> Retry (3) / Abort
        │
        ▼
LAUNCH AGENT & SYSTEM INITIALIZATION
        │
        ├─ Load config.yaml (Includes Telegram User ID & Thresholds)
        ├─ Initialize structured logging
        ├─ Initialize safety guard
        ├─ Initialize performance controller
        ├─ Initialize memory manager (SQLite + Vector DB)
        ├─ Initialize replay recorder
        ├─ Initialize screenshot engine (with DPI Scaling awareness)
        ├─ Initialize grid engine & Cursor tracker
        ├─ Initialize Direct System Execution Layer & App Registry
        ├─ Start Telegram Long-Polling Listener (Background Thread)
        ├─ Launch Internal llama.cpp Runtime OR Connect to Colab Tunnel
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
        ├─ Unified visual token processing (Local Inference OR Colab API)
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
        ├─ Internal Application Registry Status (Installed/Running apps)
        ├─ Vector DB query: Retrieve past successful workflows for context
        ├─ Cursor position & Last actions
        └─ Context Size Limiter (Top 20 elements, 5-action history limit)
        │
        ▼
PROMPT BUILDER & AI REASONING
        │
        ├─ Insert system prompt & JSON action schema
        ├─ Insert UI context & Send Prompt -> Internal Component OR Cloudflare Tunnel
        │
        ▼
RECEIVE RESPONSE & CLASSIFICATION
        │
        ├─ Request Class: Interactive Prompt? -> Pause Loop -> Ping Telegram -> Wait for user reply
        ├─ Request Class: Direct System Command? -> Execute via Direct System Layer (Bypass GUI)
        └─ Request Class: GUI Interaction? -> Proceed to Reflection & Critic Loop
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
        ├─ Violates Safety Rules? (Protect C:\Windows, force explicit Telegram permission for deletions)
        ├─ Element exists & Bounding box valid?
        ├─ Window focused & Cursor reachable?
        │
        ▼
ACTION EXECUTION (GUI OR DIRECT)
        │
        ├─ IF DIRECT: Execute OS Task (File/Web/App Control) instantly.
        ├─ IF GUI: Translate AI Grid via DPI Multiplier -> Human-like bezier movement.
        ├─ IF GUI: Apply Double-Click Heuristic (Auto-upgrade left_click on desktop icons).
        ├─ IF GUI: Execute Macro-actions (Temporal Chunking).
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
        ├─ Goal complete -> STOP -> Notify Telegram -> INITIATE SHUTDOWN
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
        ├─ Terminate screenshot, Telegram listener, and AI worker threads
        ├─ Unload Model from VRAM & terminate internal background component
        └─ Safely close Tray UI / Dashboard
```

---

## 5. Core Architectural Innovations
1.  **Direct System Execution Layer:** Reduces dependency on screen interaction by programmatically handling file management, app launching, web browsing, and system commands natively via OS APIs, ensuring lightning-fast execution without mouse tracking.
2.  **Set-of-Mark (SoM) / Visual Grounding Injection:** Bounding boxes with numeric IDs are drawn *directly onto the compressed screenshot sent to the AI*. The AI doesn't just guess coordinates; it outputs `"target": {"element_id": "42"}`, bridging the gap between reasoning and pixel execution natively.
3.  **Temporal Action Chunking (Macro-Actions):** Instead of one click per loop, the AI can output chunked commands. Example: `["click_element(42)", "type_text('hello')", "press_key('enter')"]`. Reduces latency by 3x on form-filling.
4.  **Self-Correction / Reflection Prompting Loop:** Before executing a destructive or complex click, a lightweight secondary prompt asks the local model: *"Review your proposed action against the visual state. Is this hallucinated?"* Drastically reduces misclicks.
5.  **Semantic Memory RAG (Retrieval-Augmented Generation):** The system vectorizes successful task workflows. If the user asks "Book a flight", the system queries the Vector DB for `book_flight_workflow`, injecting the exact required UI path into the AI context window.
6.  **Dynamic Resolution Scaling (Foveated Vision):** To save massive LLM context tokens, peripheral monitors/tiles are heavily downscaled, while the tile containing the active cursor/target window is rendered in maximum, crisp resolution for visual reading.
7.  **Thought Cloning (Demonstration Mode):** If the agent loops or fails, the user takes over the mouse. The agent records the exact click, pairs it with the visual state, and saves it to the Vector DB for one-shot learning.

---

## 6. Internal Data Flow & Context Serialization Format

Main data objects moving continuously through the system (0.4s-0.8s intervals):
`ScreenshotFrame` -> `TileImages` -> `SemanticUIModel` -> `ContextObject` -> `PromptPayload` -> `AIResponse` -> `ActionCommand`

**Context Serialization Format (Data Structure)**
*Security Note: Clipboard contents are deliberately excluded from this state object. All clipboard operations are executed blindly via app commands to prevent sensitive user data from entering logs, context limits, or AI prompt traces.*

The exact structure serialized and injected into the AI context:
```json
{
  "goal_state": {
    "main_goal": "Clean up desktop and open Spotify",
    "current_subgoal": "Move files to documents and launch application"
  },
  "screen_state": {
    "active_window": "Desktop",
    "foveated_tile": "A",
    "visible_elements":[
      {"id": "12", "type": "icon", "text": "report.pdf"}
    ]
  },
  "system_state": {
    "running_apps": ["chrome.exe", "code.exe"]
  },
  "cursor_state": {
    "x": 960, "y": 540, "tile": "B", "cell": "B01"
  },
  "history":[
    {"action_type": "direct", "action": "search_files", "result": "report.pdf found"}
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

remote_control:
  telegram_bot_token: "YOUR_BOT_TOKEN_HERE"
  authorized_user_id: "123456789"
```

---

## 8. Screen Capture, Grid & Monitors

**Multi Monitor Support**
Automatically detects layout (e.g., monitor1 1920x1080, monitor2 1920x1080) and merges into one global coordinate space.

**Windows DPI Scaling Trap Prevention**
Windows often scales UI by 125% or 150%. MSS captures at native resolution, but PyAutoGUI moves the mouse based on scaled resolution. The agent uses `ctypes.windll.shcore.GetScaleFactorForDevice` to retrieve the exact scaling factor and applies a strict mathematical multiplier to all AI-generated GUI coordinates before execution, preventing off-target clicks.

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

**Native VLM Pipeline (In-Process or Cloud GPU)**
Uses the selected AI Model's native visual processing. Generates bounding boxes and semantic understanding (SearchBar, Button) directly from the compressed image using Qwen-VL's native `<box>` spatial grounding outputs.

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

**System Prompt (Including Classification, Scroll, and Interactive Request Rules)**
> "You are a highly capable computer control agent. You observe the system via OS APIs and natively processed vision. Your task is to achieve the user goal by selecting the next best logical action.
> Rules: 1. Output ONLY JSON. 2. Classify your action as 'direct_execution' (API bypass), 'gui_interaction' (Mouse/Keyboard), or 'interactive_request' (Ask User). 3. Prefer Direct System Execution for file, app, and web management to ensure maximum speed. 4. Never hallucinate elements. 5. If using GUI and the target is not visible, output a `scroll_down` action. 6. If you need sensitive data (e.g., passwords, personal details to fill a form) or permission to execute a destructive action (e.g., deleting user folders), you MUST output an 'interactive_request' to pause and ask the user via Telegram. 7. Stop when the goal is achieved."

**AI JSON Action Schema (Updated for Direct Execution & Interactive Requests)**
```json
{
 "thought_process":[
   "User wants to delete the 'Old_Taxes' folder.",
   "This is a destructive action. I must request explicit permission before proceeding."
 ],
 "main_goal": "...",
 "current_goal": "...",
 "next_goal": "...",
 "action_type": "interactive_request",
 "interactive_request": {
   "question": "Are you sure you want me to permanently delete the 'Old_Taxes' folder?",
   "request_type": "permission" 
 },
 "direct_execution": {
   "category": "null",
   "command": "null",
   "parameters": {}
 },
 "gui_action": {
   "name": "null",
   "target": {"element_id": "null"},
   "tile": "null",
   "cell": "null"
 },
 "input": {
   "text": "",
   "keys":[],
   "shortcut":[]
 },
 "verification": {"required": true},
 "state": {"confidence": 0.99, "reason": "Awaiting human verification for destructive file operation."}
}
```

---

## 11. Complete Operation & Action Types

| Category | Allowed Operations / Commands | Description / Examples |
| :--- | :--- | :--- |
| **Interactive Request** | `ask_permission`, `request_information` | Halts the runtime loop and pings the user on Telegram for a Yes/No approval or specific data input (e.g., passwords, form details). |
| **Direct: Application Control** | `open_installed_app`, `launch_program`, `detect_installed_apps`, `check_app_running`, `focus_running_app`, `close_application` | Bypasses GUI to manipulate processes natively via OS execution layers. Maintains internal app registry. |
| **Direct: File Operations** | `open_file`, `create_file`, `delete_file`, `rename_file`, `copy_file`, `move_file`, `create_folder`, `delete_folder`, `search_files`, `detect_paths` | Performs instant file I/O operations without interacting with Windows Explorer. |
| **Direct: Folder / Explorer** | `open_folder`, `open_system_folder`, `reveal_file_location`, `open_explorer_at_path` | Directly bridges the agent to Windows file system directories (Downloads, Documents, Desktop). |
| **Direct: Web Operations** | `open_url`, `detect_domain_launch`, `search_default_browser`, `open_web_service` | Automatically parses URLs/domains and opens them in the user's default browser. |
| **Direct: System Commands** | `shutdown_system`, `restart_system`, `sleep_system`, `lock_screen`, `open_settings`, `open_task_manager`, `open_control_panel` | Native OS command execution, protected by user-intent safety guards. |
| **Direct: Clipboard** | `copy_to_clipboard`, `paste_from_clipboard` | Securely executes copy/paste natively. Clipboard contents are completely restricted from AI context windows, logs, and UIs to ensure absolute data privacy. |
| **Direct: Window Management** | `focus_window`, `minimize_window`, `maximize_window`, `restore_window`, `close_window`, `switch_window` | Manages active OS window states directly via Window API handles. |
| **Direct: File Download** | `download_from_url`, `save_file_to_location` | Automates remote file fetching directly to disk. |
| **Direct: Search Operations**| `search_installed_apps`, `search_files`, `search_folders` | Native deep search bypassing the Windows GUI Start Menu. |
| **GUI: Mouse Operations** | `left_click` / `click`, `double_click`, `right_click`, `middle_click`, `click_and_hold`, `release_click`, `hover` | Core pointer interactions executed via PyAutoGUI. |
| **GUI: Cursor Movement** | `move_cursor`, `move_cursor_relative`, `move_cursor_smooth`, `move_to_grid_cell`, `move_to_element` | Movement governed by Bezier path duration and DPI scaling. |
| **GUI: Drag Operations** | `drag` / `drag_start`, `drag_move`, `drag_end`, `drag_and_drop` | Click, hold, and translate operations across UI elements. |
| **GUI: Scrolling** | `scroll_up`, `scroll_down`, `scroll_left`, `scroll_right`, `scroll_to_element`, `scroll_to_top`, `scroll_to_bottom` | Triggered by the "Below-the-Fold" rule to navigate tall/wide views. |
| **GUI: Keyboard Operations** | `type_text`, `press_key`, `hold_key`, `release_key`, `press_hotkey` / `keyboard_shortcut`, `paste_text`, `clear_field` | Jittered typing logic applied (0.02s - 0.07s delay per character). |
| **GUI: Text Field Operations**| `focus_input`, `clear_input`, `type_text`, `submit_input` | Macro-actions grouped via Temporal Action Chunking. |
| **Key Mapping (Modifiers)** | `ctrl`, `alt`, `shift`, `win` (Windows key) | Logical strings generated by AI mapped to execution layer. |
| **Key Mapping (Function)** | `f1`, `f2`, `f3`, `f4`, `f5`, `f6`, `f7`, `f8`, `f9`, `f10`, `f11`, `f12` | Logical strings generated by AI mapped to execution layer. |
| **Key Mapping (Nav)** | `up`, `down`, `left`, `right`, `home`, `end`, `pageup`, `pagedown` | Logical strings generated by AI mapped to execution layer. |
| **Key Mapping (Control)** | `enter` (or `return`), `esc`, `tab`, `space`, `backspace`, `delete`, `insert` | Logical strings generated by AI mapped to execution layer. |
| **Key Mapping (Shortcuts)**| Copy: `["ctrl", "c"]`, Paste: `["ctrl", "v"]`, Task Switch: `["alt", "tab"]`, Close Window: `["alt", "f4"]` | Executed securely via `shortcut([...])` combination arrays. |

---

## 12. Execution, Validation & Safety

**The Double-Click Heuristic**
In Windows, desktop icons and file explorer items require a double-click to open. If the Semantic UI Analyzer identifies the target element as a "Desktop Icon" or "File Explorer Item," the executor will automatically upgrade the AI's `left_click` command to a `double_click`.

**Action Execution Delay Matrix (GUI only)**
To prevent UI race conditions, allow OS animations to finish, and strictly mimic human interaction speeds:
*   **Cursor Movement:** `0.2s - 0.5s` (Calculated dynamically via Bezier distance tweening)
*   **Clicking (down/up):** `0.05s - 0.15s` inside the click; followed by a `0.3s` post-click pause
*   **Typing Text:** `0.02s - 0.08s` delay per individual keystroke
*   **Scrolling:** `0.1s - 0.3s` interval between scroll steps
*   **Dragging:** `0.5s - 1.0s` duration for drag movement; `0.2s` hold before release
*   **Window Switching:** `0.5s - 0.8s` delay to allow OS animation to fully complete
*   **Application Launching:** `2.0s - 5.0s` delay (Dynamic wait polling based on active window detection)
*   *Note: Direct System Execution commands run instantly with zero artificial delay.*

**Safety Guard Rules (`safety_guard.py`)**
Strict hardcoded rules the AI cannot override:
1.  **System File Protection:** Blocks deletion or modification of critical OS directories (e.g., `C:\Windows\*`). 
2.  **Explicit User Permission:** Any file deletion outside of user-temp folders triggers a mandatory `interactive_request` for Telegram approval.
3.  **OS Power Commands:** Shutdown/Restart/Sleep are blocked from arbitrary hallucination and enforce explicit Telegram verification.
4.  **Registry Protection:** Blocks unauthorized registry edits (`regedit`).
5.  **UAC Detection:** Pauses execution if a User Account Control prompt is detected.

**Action Validation & Focus**
Every action is validated: Element exists? Bounding box valid? Cursor reachable? Window active? (Focus window if needed).

**User Interference Handling**
Monitors `pynput`. Detects mouse move or keystroke. Response: Pause AI -> Update cursor state -> Recapture -> Rebuild Context -> Resume.

**Screen Change Detection Thresholds**
To verify a GUI action worked, exactly one of these thresholds must be met:
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
5. Terminate screenshot, Telegram listener, and AI worker threads.
6. Instruct the internal `llama.cpp` runtime to dump the VRAM cache and safely terminate the background component.
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
Action executed -> Action Verified (Screen change or OS API return) -> SQLite logs session -> Short term RAM state updates -> Goal Complete? Embed workflow sequence into Vector DB.

---

## 15. System Monitoring & Debugging

**VRAM Keep-Alive Locking**
Upon initialization of a local model, the selected Qwen model is locked securely into memory via the internal inference engine (e.g., setting GPU layer limits in `llama-cpp-python`). This ensures instant, zero-latency inference for the entirety of the agent's lifespan, managed entirely by the application lifecycle.

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
├── config.yaml              <-- (Centralized thresholds, timeouts, Telegram auth, settings)
├── installer.py             <-- (Handles Model Menu, Python env setup, Hugging Face GGUF downloader)
├── main.py                  <-- (Entry point, config loading, and orchestrator)
│
├── runtime/
│   ├── engine.py            <-- (Main runtime loop, CPU performance monitor, loop detector)
│   ├── executor.py          <-- (Action execution, Action Delay Matrix, Double-Click Heuristics)
│   ├── direct_executor.py   <-- (Direct System Execution Layer: File/Web/OS APIs)
│   ├── app_registry.py      <-- (Maintains internal application registry and states)
│   └── observers.py         <-- (Screen change thresholds, reflection critic, JSON repair)
│
├── perception/
│   ├── vision_capture.py    <-- (Screenshot, multi-monitor merge, frame cache, foveated downscaling)
│   ├── spatial_grid.py      <-- (Grid overlay, tile cropping, SoM visual grounding drawing)
│   └── semantic_mapper.py   <-- (Element mapping, state tracking, semantic analyzer)
│
├── ai/
│   ├── llm_client.py        <-- (Internal llama-cpp API/Cloudflare Tunnel Bridge, VRAM locking, HF loader)
│   ├── prompt_engine.py     <-- (Prompt builder, Direct/GUI/Interactive classifier, thinking trace)
│   └── action_planner.py    <-- (Goal reasoning, RAG semantic memory injection)
│
├── core/
│   ├── config_manager.py    <-- (Parses config.yaml and distributes global settings)
│   ├── cursor_engine.py     <-- (Tracker, controller, verifier, Bezier paths, DPI Scaling Math)
│   ├── input_manager.py     <-- (Telegram Polling listener, Interactive Request Handler, Key Mapping)
│   ├── memory_manager.py    <-- (Short term RAM, SQLite session logging, ChromaDB Vector RAG)
│   ├── safety_guard.py      <-- (Hardcoded OS/File protection rules)
│   └── app_ui.py            <-- (Taskbar Live Console, CustomTkinter Dashboard, Chatbox, Tray icon)
│
├── utils/
│   ├── image_utils.py       <-- (Compression 75/70/400kb, SSIM, pHash logic math)
│   └── system_utils.py      <-- (Structured logging, replay recorders, timers, Graceful Shutdown)
│
└── data/
    ├── models/              <-- (Local directory where downloaded GGUF weights are stored securely)
    ├── memory/              <-- (session.db, vector_store/)
    ├── logs/
    ├── screenshots/
    └── replay_sessions/
```

---

## APPENDIX A: Cloud GPU Backend (Google Colab + Llama.cpp)

### 1. Colab Server Code (Single Cell)
Paste this into a single Google Colab cell. It mounts Drive, compiles `llama.cpp` for CUDA, downloads the 4B vision model (only ~5GB), and opens an OpenAI-compatible Cloudflare tunnel for remote execution.

```python
from google.colab import drive
import os, subprocess, time

# 1. Setup Drive & Cache
drive.mount('/content/drive')
MODEL_DIR = "/content/drive/MyDrive/agent-sam-models"
os.makedirs(MODEL_DIR, exist_ok=True)

# 2. Install & Compile llama.cpp
!pip install -q -U huggingface_hub[cli]
if not os.path.exists("/content/llama.cpp/llama-server"):
    !git clone https://github.com/ggerganov/llama.cpp
    !cd llama.cpp && make clean && make GGML_CUDA=1 -j

# 3. Download Qwen3-VL 4B GGUF Weights
LLM_REPO = "Qwen/Qwen3-VL-4B-Instruct-GGUF"
LLM_FILE = "qwen3-vl-4b-instruct-q4_k_m.gguf"
MMPROJ_FILE = "mmproj-qwen3-vl-4b-instruct-f16.gguf"

LLM_PATH = os.path.join(MODEL_DIR, LLM_FILE)
MMPROJ_PATH = os.path.join(MODEL_DIR, MMPROJ_FILE)

!huggingface-cli download {LLM_REPO} {LLM_FILE} --local-dir {MODEL_DIR}
!huggingface-cli download {LLM_REPO} {MMPROJ_FILE} --local-dir {MODEL_DIR}

# 4. Start Native OpenAI-Compatible Server
server_cmd =[
    "/content/llama.cpp/llama-server",
    "-m", LLM_PATH, "--mmproj", MMPROJ_PATH,
    "-ngl", "99", "-c", "8192", "--port", "8000", "--host", "0.0.0.0"
]
subprocess.Popen(server_cmd, stdout=subprocess.DEVNULL, stderr=subprocess.STDOUT)
time.sleep(10)

# 5. Start Cloudflare Tunnel
if not os.path.exists("cloudflared-linux-amd64"):
    !wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
    !chmod +x cloudflared-linux-amd64

!./cloudflared-linux-amd64 tunnel --url http://127.0.0.1:8000
```

### 2. Integration with agent-sam
When "Cloud GPU Backend" is selected in the installer, the user pastes the Cloudflare URL into the application prompt. The desktop client (`ai/llm_client.py`) automatically routes requests to this URL, perfectly matching the OpenAI vision format without utilizing local hardware for inference.

**Endpoint:** `https://[cloudflare-url]/v1/chat/completions`

**Payload Schema:**
```json
{
  "messages":[
    {
      "role": "user",
      "content":[
        {
          "type": "image_url",
          "image_url": {"url": "data:image/jpeg;base64,[BASE64_STRING]"}
        },
        {
          "type": "text",
          "text": "[SYSTEM_PROMPT_AND_UI_CONTEXT]"
        }
      ]
    }
  ],
  "temperature": 0.0,
  "max_tokens": null
}
```
