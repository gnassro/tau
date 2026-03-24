# Tau Mirror Project - AI Agent Guidelines

Welcome, future AI Agent! This project is the web UI ("Mirror") for the Pi Coding Agent. It allows users to view and interact with their terminal-based Pi sessions from a browser.

**CRITICAL DIRECTIVE: ALWAYS UPDATE DOCUMENTATION**
Whenever you make structural changes to the UI, add new features, or refactor core logic (especially in `public/app.js`), you MUST update this `AGENTS.md` file and the `README.md` to reflect those changes. Do not leave the documentation out of sync with the codebase.

## Project Architecture

The project is split into two halves:
1. **Backend Extension (`extensions/mirror-server.ts`)**: Runs inside the active `pi` process. It spins up an HTTP/WebSocket server to serve the static frontend files and stream events/state to the browser.
2. **Frontend UI (`public/`)**: The web interface the user interacts with.

### Frontend Structure (Refactored 2026-03)
The frontend was recently refactored to an **Antigravity/Cursor IDE 3-Pane Layout**.
*   **Left Pane**: File explorer (`#file-sidebar`). Controlled by `public/file-browser.js`.
*   **Center Pane**: Read-only code editor (`#editor-pane`). Displays file contents clicked in the left pane.
*   **Right Pane**: Main chat window (`.main`).
*   **Settings / History**: Session history was moved to a dedicated modal overlay (`#history-panel`), triggered by the clock icon in the top right.

### Core Modules
We moved away from a monolithic `app.js` toward modular, object-oriented ES6 classes:
*   `public/app.js`: The main coordinator. Instantiates components, handles WebSockets (`public/websocket-client.js`), and manages global state.
*   `public/chat-input.js`: (`ChatInput` class). Manages the `contenteditable` rich-text chat box. Handles `Shift+Enter`, HTML stripping on paste, and parsing UI chips back to raw text.
*   `public/autocomplete.js`: (`Autocomplete` class). Listens for `@` mentions, fetches project files via RPC, renders the dropdown, and handles keyboard navigation.
*   `public/code-editor.js`: (`CodeEditor` class). Manages the center pane. Calculates selected line numbers and renders the floating "Add to Chat" button for inline context chips.
*   `public/session-sidebar.js`: Filters session history strictly by the Current Working Directory (`cwd`).

## Development Rules

1. **Inline Context Chips**: The chat input is NOT a `<textarea>`. It is a `<div contenteditable="true">`. File mentions and code snippets are rendered as HTML `<span class="context-chip">` elements inline with the text. If you need to read the text, use `chatInput.getText()` which safely parses the HTML chips back into `@file/path` or markdown blocks. Do NOT use `.value` or `.innerText` blindly.
2. **Session Switching**: The UI can spawn new terminals. If the user clicks a disconnected session in the history, `mirror-server.ts` uses `osascript` to open a new iTerm2 window running `pi --resume`, polls for the new port, redirects the browser, and kills the old server.
3. **Styling**: Always use the defined CSS variables in `public/style.css` (e.g., `var(--bg-solid)`, `var(--text-primary)`, `var(--border)`).
