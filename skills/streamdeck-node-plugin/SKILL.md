---
name: streamdeck-node-plugin
description: Comprehensive guidance for developing Elgato Stream Deck plugins in Node.js using the Stream Deck CLI and SDK.
---

# Stream Deck Node.js Plugin Development Skill

This skill guides you through **end‑to‑end development of Stream Deck plugins in Node.js**, using the official **Stream Deck CLI** and **Stream Deck SDK**.

It is designed to be used with coding agents compatible with the [add-skill](https://github.com/vercel-labs/add-skill) specification.

References:
- [Stream Deck CLI – Introduction](https://docs.elgato.com/streamdeck/cli/intro)
- [Stream Deck SDK – Getting Started](https://docs.elgato.com/streamdeck/sdk/introduction/getting-started)

---

## When to Use

Use this skill whenever you need to:

- **Create a new Stream Deck plugin in Node.js**
- **Modify or extend an existing Node.js Stream Deck plugin**
- **Set up or debug the Stream Deck CLI workflow** (`streamdeck create`, `streamdeck dev`, `streamdeck link`, `streamdeck restart`, `streamdeck pack`, `streamdeck validate`, etc.)
- **Prepare a plugin for distribution** (validation, packaging, and basic publishing considerations)
- **Review a plugin for best practices** (structure, performance, UX integration, and reliability)

---

## High‑Level Concepts

Before writing or modifying code, keep these core concepts in mind:

- **Stream Deck app & device**
  - The **Stream Deck desktop app** (v6.9+ per CLI docs) manages plugins and communicates with connected **Stream Deck hardware (or virtual device)**.
  - Your plugin is a small app that the Stream Deck application loads and runs.

- **Plugin vs. actions**
  - A **plugin** is a bundle (folder / `.streamDeckPlugin` file) with one or more **actions**.
  - An **action** is what the user drags onto a key. Each action has its own settings, title, icon, and behavior when pressed / shown.

- **Manifest (`manifest.json`)**
  - Declares **plugin metadata** (UUID, name, description, author, version, icon, platform/OS targets, etc.).
  - Defines **actions**, their UUIDs, icons, supported device types, and configuration options for the property inspector.

- **Plugin process & Node.js**
  - For Node.js plugins, the **Stream Deck app launches your Node.js runtime** and communicates with it via a **WebSocket protocol** defined by the SDK.
  - Your Node app listens for **SDK events** (e.g. `willAppear`, `keyDown`, `keyUp`, `sendToPlugin`) and sends **commands** back (`setTitle`, `setImage`, `setSettings`, etc.).

- **Property Inspector (PI)**
  - A small **web UI** (HTML/JS) that runs inside the Stream Deck app to configure each action instance.
  - Talks to the plugin via `sendToPlugin` / `sendToPropertyInspector` messages.

- **CLI vs. SDK docs**
  - The **CLI** docs focus on commands and workflows: creating, linking, validating, and packing plugins.  
    See: [Stream Deck CLI intro](https://docs.elgato.com/streamdeck/cli/intro)
  - The **SDK** docs focus on events, manifest schema, plugin structure, and integration patterns.  
    See: [SDK Getting Started](https://docs.elgato.com/streamdeck/sdk/introduction/getting-started)

---

## Prerequisites Checklist

Before starting implementation, verify all of the following:

1. **Environment**
   - **Node.js 20+** is installed (per CLI prerequisites).
   - **Stream Deck desktop app 6.9+** is installed.
   - A **Stream Deck device** (or supported virtual device) is available.

2. **CLI installation (one‑time per machine)**
   - The global `streamdeck` (or `sd`) CLI is installed from npm, as described in the CLI docs:

```bash
npm install -g @elgato/streamdeck-cli
```

   - Confirm it works:

```bash
streamdeck --help
```

3. **Project workspace**
   - You have (or will create) a **dedicated directory** for the plugin project.
   - Version control (e.g. **git**) is initialized or planned.

4. **Technical decisions**
   - You know **what your plugin should do** (feature description).
   - You know **which actions** you will expose to the user (e.g. single key toggle, multi‑state button, folder‑level behavior).
   - You understand any **external APIs/services** the plugin must call (authentication, rate limits, latency concerns).

---

## Repository & File Structure (Typical Node.js Plugin)

When using `streamdeck create`, the CLI will scaffold a recommended structure. It may vary slightly by template, but a typical Node.js plugin looks like:

```text
my-streamdeck-plugin/
  manifest.json           # Plugin metadata and action definitions
  plugin/                 # Node.js plugin source
    index.js              # Main entrypoint (or compiled JS)
    package.json          # Node dependencies and scripts
    src/                  # Optional TS/JS source
  property-inspector/     # Property inspector UI
    index.html
    index.js
    styles.css
  images/                 # Icons for plugin and actions
    pluginIcon.png
    actionDefault.png
    actionPressed.png
  localization/           # Optional localization files
  README.md
```

Always inspect the scaffolded template to confirm the exact structure and any additional tooling (TypeScript, bundler, etc.).

---

## CLI Workflow Overview

Key commands from the [CLI docs](https://docs.elgato.com/streamdeck/cli/intro):

- **`streamdeck create`**  
  Interactive wizard or flag‑driven scaffolding for a new plugin project.

- **`streamdeck link [path]`**  
  Links the plugin in your directory to the Stream Deck app.

- **`streamdeck restart <uuid>`**  
  Restarts the plugin process (stops if running, then starts).

- **`streamdeck stop <uuid>`**  
  Stops the plugin.

- **`streamdeck list`**  
  Lists installed plugins and their status.

- **`streamdeck dev`**  
  Enables developer mode; often used to simplify testing and logging.

- **`streamdeck validate [path]`**  
  Validates the plugin structure and manifest.

- **`streamdeck pack [path]` / `bundle`**  
  Bundles the plugin into a `.streamDeckPlugin` file for distribution.

- **`streamdeck config`**  
  Manages local CLI configuration.

The `streamdeck` command is also available as `sd` for convenience.

---

## Step‑by‑Step: Creating a New Node.js Plugin

Follow these steps to create a new Stream Deck plugin using Node.js:

### 1. Create the project with the CLI

1. Choose or create a parent directory where your plugin project will live.
2. Run the create command (adjust name and options as needed):

```bash
streamdeck create
```

- When prompted:
  - **Select a plugin template** that targets **Node.js** (if the wizard offers language choices).
  - Provide a **plugin name**, **UUID**, and other metadata.
  - Choose a suitable location for the project directory.

If the CLI supports non‑interactive mode for templates, you can pass flags (check `streamdeck create --help` for details).

### 2. Inspect the generated structure

After creation:

- Open the project in your editor and examine:
  - `manifest.json`
  - The Node.js entry file (e.g. `plugin/index.js` or similar)
  - The property inspector files (e.g. `property-inspector/index.html`, `index.js`)
  - Any provided examples of event handling and messaging

**Do not** arbitrarily change generated filenames without understanding how the manifest and/sdk loader reference them.

### 3. Configure `manifest.json`

Open `manifest.json` and ensure at minimum:

- **Plugin metadata**
  - `UUID` is globally unique and stable.
  - `Name`, `Description`, `Author`, `Version`, `Category` are set appropriately.
  - `Icon` paths point to actual files under `images/`.

- **Platform/OS support**
  - The plugin targets the correct platforms (Windows/macOS) as required.

- **Actions**
  - Each action has:
    - A unique `UUID`.
    - `Name` and `Tooltip`.
    - `Icon` references for default and pressed states.
    - `States` definitions (for multi‑state actions).
    - Optional `PropertyInspectorPath` if it uses a configuration UI.

Use the SDK getting started and schema references to verify keys and structures are valid.

### 4. Implement Node.js plugin logic

In the Node.js entry file (e.g. `plugin/index.js`), ensure you:

1. **Establish connection with the Stream Deck app**
   - Use the SDK boilerplate or template code to:
     - Parse CLI arguments passed by the Stream Deck app (port, plugin UUID, register event, info JSON).
     - Open a **WebSocket connection** to the app.
     - Register your plugin using the provided event.

2. **Handle SDK events**
   - Implement handlers for common events such as:
     - `willAppear` / `willDisappear`
     - `keyDown` / `keyUp`
     - `titleParametersDidChange`
     - `sendToPlugin` (from Property Inspector)
   - For each handler:
     - Keep logic **non‑blocking** (avoid long synchronous operations).
     - Delegate heavy work to asynchronous functions or background helpers where possible.

3. **Send commands back to Stream Deck**
   - Use the SDK message schema to:
     - **Update the key title** (`setTitle`).
     - **Change the key image** (`setImage`).
     - **Change state** for multi‑state actions (`setState`).
     - **Manage settings**:
       - `getSettings` / `setSettings` (per‑action instance).
       - `getGlobalSettings` / `setGlobalSettings` (plugin‑wide).

4. **Logging & error handling**
   - Add consistent logging around:
     - Connection lifecycle.
     - Incoming events (log event type and relevant payload, but avoid logging secrets).
     - External API calls (status, errors).
   - Catch and handle promise rejections to avoid crashing the plugin process.

### 5. Implement the Property Inspector (optional but recommended)

For actions that need configuration:

1. **HTML structure**
   - Create/modify `property-inspector/index.html`:
     - Include form controls (inputs, selects, toggles) for your action settings.
     - Load a JS file (`index.js`) that will talk to the plugin using the SDK PI API.

2. **PI–Plugin communication**
   - In PI JS:
     - Listen for `didReceiveSettings` to populate the UI.
     - Send settings updates to the plugin using `sendToPlugin`.
     - Call `setSettings` (or equivalent) to persist the configuration when the user changes values.

3. **Validation**
   - Validate user input in the PI before sending to the plugin.
   - Provide user‑friendly error messages, not just silent failures.

### 6. Link and run the plugin

Use the CLI to **link** and **run** your plugin in the Stream Deck app:

```bash
streamdeck link .
streamdeck restart <your-plugin-uuid>
```

- Alternatively:
  - Use `streamdeck list` to confirm your plugin is recognized.
  - Use `streamdeck stop <uuid>` when you need to fully stop the plugin.
  - Enable developer mode with `streamdeck dev` if needed (for logging / hot‑reload behavior as supported).

Ensure that:

- The plugin appears in the Stream Deck app’s list of actions (under the configured category).
- Dragging your action onto a key triggers the expected behavior.

### 7. Test thoroughly

Create a small **test plan**:

- **Basic behavior**
  - Verify `keyDown` / `keyUp` responses.
  - Verify `willAppear` initializes key state correctly.
  - Verify title and icon updates.

- **Settings**
  - Modify settings in the property inspector; confirm that:
    - The plugin receives updated settings.
    - Behavior changes accordingly.
    - Settings persist across app restarts.

- **Edge cases**
  - Device disconnect / reconnect.
  - Stream Deck app restart.
  - Network/API failures (if using external services).
  - Invalid user input in the PI.

- **Performance**
  - Press keys rapidly; confirm no lag or missed events.
  - Avoid blocking the Node.js event loop (no long `while` loops, `sleep`‑like sync calls, or large synchronous file operations).

### 8. Validate and pack for distribution

Once functionality is stable:

1. Run validation:

```bash
streamdeck validate .
```

- Fix any reported issues (manifest fields, missing assets, invalid paths, etc.).

2. Pack the plugin:

```bash
streamdeck pack .
```

- This should produce a `.streamDeckPlugin` file that can be double‑clicked to install or shared with others.

3. Optionally, maintain a **CHANGELOG** and increment the `version` field in `manifest.json` for each release.

---

## Best Practices for Node.js Stream Deck Plugins

Use this checklist when designing or reviewing a Node.js plugin:

- **Architecture**
  - Keep the plugin entry file small; **factor logic into modules** under `src/` or similar.
  - Separate:
    - **SDK protocol / message handling**
    - **Business logic**
    - **External API integrations**

- **Performance**
  - Avoid blocking the event loop; use `async`/`await` and non‑blocking I/O.
  - Debounce or throttle repeated actions (e.g. frequent API polling or rapid key presses).
  - Cache results where appropriate, respecting API rate limits and freshness requirements.

- **Resilience**
  - Handle network timeouts and API errors with retries and backoff when appropriate.
  - Ensure the plugin can recover gracefully from errors without requiring a full Stream Deck restart.
  - Use defensive coding for possibly missing or malformed settings.

- **Settings & UX**
  - Provide clear, minimal settings in the property inspector; avoid overwhelming the user.
  - Use sensible defaults so that an action works immediately after being added to a key.
  - Reflect state clearly on the key (e.g. on/off status via icon or title).

- **Security**
  - Never hard‑code secrets or access tokens in the plugin or PI code.
  - If authentication is needed, prefer OAuth/device‑code flows and secure token storage.
  - Avoid logging sensitive data.

- **Cross‑platform considerations**
  - Be careful with file paths and OS‑specific behaviors (Windows vs macOS).
  - Avoid assumptions about default encodings or locales.

---

## Checklists

### New Plugin Checklist

- [ ] Node.js 20+ and Stream Deck 6.9+ installed.
- [ ] `streamdeck` CLI installed and verified (`streamdeck --help` works).
- [ ] Project created via `streamdeck create`.
- [ ] `manifest.json` completed and validated.
- [ ] At least one action implemented with correct UUID and icons.
- [ ] Node.js entry connects to Stream Deck and handles key events.
- [ ] Property inspector (if used) is wired to plugin and settings persist.
- [ ] Plugin linked via `streamdeck link` and visible in the Stream Deck app.

### Pre‑Release Checklist

- [ ] `streamdeck validate .` passes with no errors.
- [ ] All assets (icons, localization files) referenced in `manifest.json` exist.
- [ ] Version in `manifest.json` is updated for this release.
- [ ] Basic and edge‑case behavior tested (rapid presses, app restarts, device reconnects).
- [ ] External API usage respects rate limits and handles failures gracefully.
- [ ] `.streamDeckPlugin` bundle generated via `streamdeck pack .`.

---

## How to Use This Skill with an Agent

When you invoke a coding agent to work on a Stream Deck Node.js plugin:

1. **State clearly** that you are working on an Elgato Stream Deck plugin in Node.js.
2. **Ask the agent to follow this skill** for:
   - Creating the plugin structure with `streamdeck create`.
   - Configuring `manifest.json` and actions.
   - Implementing Node.js plugin logic and property inspector behavior.
   - Using `streamdeck link`, `restart`, `list`, `validate`, and `pack` correctly.
3. **Provide any existing code or manifest files** so the agent can:
   - Avoid duplicating actions or logic.
   - Reuse and refactor existing structures instead of rewriting them.

This ensures a consistent, robust Stream Deck Node.js plugin development workflow aligned with the official CLI and SDK documentation.

