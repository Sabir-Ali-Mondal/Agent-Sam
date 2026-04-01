# Agent-Sam: As-Built Technical Specification

This document details the current, implemented architecture and features of the `agent-sam` project. It reflects the production state of the codebase, not future plans.

---

### 1. Core Execution Philosophy: Search-First

The agent's primary operational strategy is **Search-First**. It leverages the native power of the operating system's search capabilities (Windows Search) and the global reach of web search to ensure robust and reliable execution.

1.  **Windows Search (Local):** The default method for launching applications (`open_app`), finding files (`open_path`), and interacting with system features. This utilizes the shell's built-in NLP and resolution, removing the need for hardcoded paths.
2.  **Web Search (Global):** A fallback and primary tool for information retrieval. If a local search fails or the goal requires external knowledge, the agent seamlessly transitions to a web search.
3.  **Direct Execution (API):** Used for specific tasks where search is inappropriate, such as running terminal commands (`run_command`) or opening a precise URL (`open_url`).

---

### 2. Communication Protocol: TAG-Only Format

To eliminate JSON-related failures and improve performance with smaller local models, the agent uses a strict, minimalist **TAG-based format** for all AI communication.

**Production TAG Format:**
```text
@type: <direct_execution|gui_interaction|interactive_request|system>
@action: <action_name>

# Optional parameters based on action
@name: <App name or Search query>
@url: <URL to open>
@path: <File path to open>
@text: <Text to type or command to run>
@keys: <keyboard keys, e.g., alt+tab, win, enter>
@target: <Grid location or element ID, e.g., A01>
@x: <X-coordinate for move/drag>
@y: <Y-coordinate for move/drag>
```

This format is parsed by the **Flexible Action Parser** in `runtime/observers.py`, which includes regex-based extraction and can fall back to JSON repair if legacy formats are detected.

---

### 3. Implemented Agent Flowchart

```text
[START]
    │
    ▼
[GOAL DEFINED]
    │
    ▼
[INITIALIZE CORE COMPONENTS]
    ├─ Load Config (config.yaml, .env)
    ├─ Launch Internal llama.cpp OR Connect to Cloud/API
    ├─ Initialize Flexible Action Parser & Loop Detector
    ├─ Initialize AI Cursor Overlay (UI Thread)
    └─ Initialize Dashboard & REST Goal Server
    │
    ▼
[ENTER MAIN RUNTIME LOOP]
    │
    └──────── LOOP START ────────
                │
                ▼
[SCREEN CAPTURE & CONTEXT BUILD]
    ├─ Capture screen, apply foveated vision, compress
    ├─ Build Simplified Context (main_goal, screen_state, previous_actions)
    │
    ▼
[AI REASONING (PROMPT & RESPONSE)]
    ├─ Send System Prompt + Context to AI
    ├─ AI returns ONE action in TAG format
    │
    ▼
[VALIDATION & EXECUTION]
    ├─ Parse TAGs with FlexibleActionParser
    ├─ Validate action and required parameters
    ├─ IF direct_execution -> DirectExecutor (Search, Shell, Web)
    ├─ IF gui_interaction -> ActionExecutor (Mouse, Keyboard)
    │
    ▼
[POST-ACTION & GOAL CHECK]
    ├─ Detect screen change to check for failure
    ├─ IF goal complete (@action: done) -> STOP
    ├─ ELSE -> LOOP
    │
    ▼
[REPEAT]
```

---

### 4. Key Implemented Features

#### a. AI Cursor Overlay
-   **Visual Feedback:** A green, round "tricone" cursor is rendered on a transparent, click-through window (`AICursorOverlay` in `core/app_ui.py`).
-   **User Experience:** Shows the user the AI's intended target *before* the mouse moves, preventing confusion.
-   **Thread Safety:** Managed on the main UI thread via a thread-safe queue to prevent `RuntimeError` crashes from the background agent thread.

#### b. Dynamic Token Management
-   **Context-Aware:** The `LLMClient` automatically calculates the total prompt size (text + image tokens).
-   **Prevents Crashes:** It dynamically adjusts the `max_tokens` for the AI's response to ensure the total request size does not exceed the model's context window (e.g., 4096 or 8192), preventing `400 Bad Request` errors.
-   **Colab Optimization:** The `colab_create_tunnel.py` script is configured to launch the vLLM backend with an `max-model-len` of **8192** for GPU instances, providing ample headroom.

#### c. Robust Crash & Error Handling
-   **Crash Reporting:** If the application encounters a fatal error, it generates a `crash_report.txt` file in the root directory with a full traceback.
-   **Terminal Pause:** The `run_app.bat` script will `pause` on exit if an error is detected, allowing the user to read the error message in the console.
-   **Port Conflict Handling:** The REST goal server startup is wrapped in a `try-except` block to prevent crashes if the port is already in use.

#### d. Intelligent Cloud Setup
-   **Flexible URL Handling:** The setup wizard and `LLMClient` now correctly parse any provided Cloudflare URL, whether it's the base domain or the full `/v1/agent` endpoint.
-   **Live Validation:** The installer performs a live connection test on the provided tunnel URL and warns the user if it's unreachable, improving the setup experience.

---

### 5. Action Catalog (Implemented)

**Direct Execution (`@type: direct_execution`)**
| Action | Parameter | Description |
| :--- | :--- | :--- |
| `open_app` | `@name` | Launches an application via Windows Search/Shell. |
| `web_search` | `@text` | Performs a Google search. |
| `open_url` | `@url` | Opens a specific URL in the default browser. |
| `open_path` | `@path` | Opens a file or folder path in Explorer. |
| `run_command`| `@text` | Executes a single-line PowerShell command. |

**GUI Interaction (`@type: gui_interaction`)**
| Action | Parameter(s) | Description |
| :--- | :--- | :--- |
| `click` | `@target` | Left-clicks on a grid cell or element. |
| `right_click`| `@target` | Right-clicks on a grid cell or element. |
| `type` | `@text` | Types the specified text. |
| `hotkey` | `@keys` | Presses a combination of keys (e.g., `alt+f4`). |
| `press` | `@keys` | Presses a single sequential key (e.g., `enter`). |
| `scroll` | *(none)* | Scrolls the active window down. |
| `move` | `@x`, `@y` | Moves the cursor to the specified coordinates. |
| `drag` | `@x`, `@y` | Drags an item to the specified coordinates. |
| `wait` | `@text` | Pauses execution (e.g., `1s`). |

**System Actions (`@type: system`)**
| Action | Description |
| :--- | :--- |
| `done` | Indicates the main goal has been successfully completed. |
