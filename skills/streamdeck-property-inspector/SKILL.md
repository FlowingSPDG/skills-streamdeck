---
name: streamdeck-property-inspector
description: Comprehensive guidance for developing Property Inspectors (UI) for Elgato Stream Deck plugins. Covers HTML structure, sdpi-components usage, and communication patterns between Property Inspector and plugin.
---

# Stream Deck Property Inspector Development Skill

This skill guides you through **developing Property Inspectors (UI) for Stream Deck plugins**, providing users with interfaces to configure plugin settings.

It is designed to be used with coding agents compatible with the [add-skill](https://github.com/vercel-labs/add-skill) specification.

References:
- [Stream Deck SDK – Property Inspectors (UI)](https://docs.elgato.com/streamdeck/sdk/guides/ui/)
- [sdpi-components Documentation](https://sdpi-components.dev/)

---

## When to Use

Use this skill whenever you need to:

- **Create a new Property Inspector for a Stream Deck plugin action**
- **Modify or extend an existing Property Inspector**
- **Configure settings UI for plugin actions**
- **Implement communication between Property Inspector and plugin**
- **Debug Property Inspector issues**
- **Review Property Inspector code for best practices**

---

## High‑Level Concepts

Before writing or modifying Property Inspector code, keep these core concepts in mind:

- **Property Inspector (PI)**
  - A **small HTML web view** that runs inside the Stream Deck app to configure each action instance.
  - Provides a user interface for adjusting plugin settings without modifying code.
  - Each action can have its own Property Inspector, or a plugin can have a default one.

- **File structure and location**
  - Property Inspector HTML files are placed in the `ui/` directory within `*.sdPlugin`.
  - The path to the HTML file is specified in `manifest.json` at either the action level or plugin level.

- **sdpi-components (Stream Deck Property Inspector Components)**
  - The **official UI library** for building Property Inspectors.
  - Provides web components for consistent, user-friendly interfaces.
  - Handles communication with the plugin automatically.
  - Can be referenced locally (recommended) or remotely (for development).

- **Communication flow**
  - Property Inspector sends settings updates to the plugin.
  - Plugin receives settings via `didReceiveSettings` event.
  - Property Inspector can receive data from the plugin via `sendToPropertyInspector`.

- **Scope boundaries**
  - Property Inspector is a **web view** with standard HTML/CSS/JavaScript capabilities.
  - Some advanced features may be limited by the web view environment.
  - React and other frameworks can be used, but must work within the Property Inspector constraints.

---

## Prerequisites Checklist

Before starting Property Inspector implementation, verify all of the following:

1. **Plugin structure**
   - You have a working Stream Deck plugin with at least one action defined.
   - The `manifest.json` file exists and is properly configured.
   - You understand which settings your action needs to expose to users.

2. **Directory structure**
   - The `*.sdPlugin` directory exists (or will be created during build).
   - The `ui/` directory exists within `*.sdPlugin` (or will be created).

3. **UI library**
   - Decide whether to use `sdpi-components` locally (recommended) or remotely (development only).
   - If using locally, download `sdpi-components.js` to the `ui/` directory.

4. **Technical decisions**
   - Determine which UI components you need (text fields, checkboxes, selects, etc.).
   - Plan the settings data structure (what properties need to be configured).
   - Consider whether you need two-way communication (PI ↔ Plugin) or just one-way (PI → Plugin).

---

## Repository & File Structure

Property Inspector files should be organized as follows:

```text
my-streamdeck-plugin/
  *.sdPlugin/
    ├── bin/
    ├── imgs/
    ├── ui/                    # Property Inspector files
    │   ├── increment-counter.html
    │   └── sdpi-components.js  # If using local library
    └── manifest.json
  src/
    ├── actions/
    │   └── increment-counter.ts
    └── plugin.ts
  package.json
  rollup.config.mjs
  tsconfig.json
```

The `ui/` directory contains all Property Inspector HTML files and related assets.

---

## Step‑by‑Step: Creating a Property Inspector

Follow these steps to create a Property Inspector for your Stream Deck plugin action:

### 1. Create the HTML file

1. Navigate to or create the `ui/` directory within your `*.sdPlugin` directory.
2. Create a new HTML file (e.g., `increment-counter.html`).
3. Set up the basic HTML structure:

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <!-- Your Property Inspector UI will go here -->
</body>
</html>
```

### 2. Reference sdpi-components

Choose one of the following approaches:

**Local (recommended for distribution):**

1. Download `sdpi-components.js` from the [sdpi-components releases](https://github.com/elgato/sdpi-components/releases).
2. Place it in the `ui/` directory alongside your HTML file.
3. Reference it in your HTML:

```html
<script src="sdpi-components.js"></script>
```

**Remote (development/prototyping only):**

```html
<script src="https://sdpi-components.dev/releases/v4/sdpi-components.js"></script>
```

⚠️ **Important:** Always use local references when distributing your plugin to ensure it works offline and provides a consistent experience.

### 3. Configure manifest.json

Add the Property Inspector path to your `manifest.json`:

**At the action level (action-specific PI):**

```json
{
    "$schema": "https://schemas.elgato.com/streamdeck/plugins/manifest.json",
    "Name": "Counter",
    "Version": "1.0.0.0",
    "Actions": [
        {
            "Name": "Counter",
            "UUID": "com.elgato.hello-world.increment",
            "PropertyInspectorPath": "ui/increment-counter.html"
            // ... other action properties
        }
    ]
    // ... other plugin properties
}
```

**At the plugin level (default PI for all actions):**

```json
{
    "$schema": "https://schemas.elgato.com/streamdeck/plugins/manifest.json",
    "Name": "Counter",
    "Version": "1.0.0.0",
    "PropertyInspectorPath": "ui/increment-counter.html",
    "Actions": [
        {
            "Name": "Counter",
            "UUID": "com.elgato.hello-world.increment"
            // ... other action properties
        }
    ]
    // ... other plugin properties
}
```

### 4. Build the UI with sdpi-components

Use sdpi-components web components to build your interface:

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <sdpi-item label="Name">
        <sdpi-textfield setting="name"></sdpi-textfield>
    </sdpi-item>
    
    <sdpi-item label="Show Name">
        <sdpi-checkbox setting="showName"></sdpi-checkbox>
    </sdpi-item>
    
    <sdpi-item label="Favorite Color">
        <sdpi-color setting="favColor"></sdpi-color>
    </sdpi-item>
</body>
</html>
```

### 5. Handle settings communication

Use the Stream Deck Client to send settings to the plugin:

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <sdpi-item label="Name">
        <sdpi-textfield setting="name"></sdpi-textfield>
    </sdpi-item>

    <script>
        const { streamDeckClient } = SDPIComponents;

        // Send settings to the plugin
        streamDeckClient.setSettings({
            name: "John Doe",
            showName: true,
            favColor: "green",
        });
    </script>
</body>
</html>
```

The `setting` attribute on components automatically binds to settings and sends updates to the plugin when values change.

### 6. Receive settings from plugin (optional)

If your plugin needs to send data back to the Property Inspector:

```html
<script>
    const { streamDeckClient } = SDPIComponents;

    // Listen for settings from the plugin
    streamDeckClient.onDidReceiveSettings((settings) => {
        // Update UI with received settings
        console.log('Received settings:', settings);
    });
</script>
```

---

## Available Components

The sdpi-components library provides the following web components:

| Component | Usage | Use Case |
|-----------|-------|----------|
| `<sdpi-button>` | Button | Trigger actions, submit forms |
| `<sdpi-checkbox>` | Checkbox | Boolean settings (on/off) |
| `<sdpi-checkbox-list>` | Checkbox List | Multiple selections |
| `<sdpi-color>` | Color Picker | Color selection |
| `<sdpi-calendar type="date">` | Date Picker | Date selection |
| `<sdpi-calendar type="datetime-local">` | Datetime Picker | Date and time selection |
| `<sdpi-calendar type="month">` | Month Picker | Month selection |
| `<sdpi-calendar type="time">` | Time Picker | Time selection |
| `<sdpi-calendar type="week">` | Week Picker | Week selection |
| `<sdpi-delegate>` | Delegate | Custom component wrapper |
| `<sdpi-file>` | File Picker | File selection |
| `<sdpi-password>` | Password Field | Secure text input |
| `<sdpi-radio>` | Radio Button | Single selection from options |
| `<sdpi-range>` | Range Slider | Numeric value selection |
| `<sdpi-select>` | Dropdown Select | Single selection from list |
| `<sdpi-textarea>` | Textarea | Multi-line text input |
| `<sdpi-textfield>` | Text Field | Single-line text input |

Each component supports the `setting` attribute to bind to a settings property. Refer to the [sdpi-components documentation](https://sdpi-components.dev/docs/components/) for detailed usage and attributes.

---

## Using React or Other Frameworks

Property Inspectors can use React or other JavaScript frameworks, but keep in mind:

- **Web view constraints**: The Property Inspector runs in a web view with standard HTML/CSS/JavaScript support.
- **Framework compatibility**: Ensure your framework works within the web view environment.
- **Bundle size**: Keep bundle sizes reasonable for fast loading.
- **sdpi-components integration**: You can use sdpi-components alongside React components, but ensure proper integration.

Example structure with React:

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
    <!-- React and your bundle -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="property-inspector.js"></script>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

⚠️ **Important**: Always test thoroughly when using frameworks, as the web view environment may have limitations.

---

## Debugging Property Inspectors

To debug a Property Inspector:

1. **Enable developer mode**:

```bash
streamdeck dev
```

2. **Open the remote debugger**:
   - Navigate to `http://localhost:23654/` in your browser.
   - You'll see a list of available pages.
   - Select your Property Inspector's page.

3. **Use browser dev tools**:
   - Press `F12` or right-click and select "Inspect".
   - Use console, network, and element inspection tools as needed.

⚠️ **Note**: The Property Inspector must be visible in the Stream Deck app for it to appear in the debugger list.

4. **Debug in plugin code**:
   - Use the `didReceiveSettings` event in your plugin action to verify settings are received:

```typescript
import streamDeck, { action, type DidReceiveSettingsEvent, SingletonAction } from "@elgato/streamdeck";

type Settings = {
    name: string;
    showName: boolean;
    favColor: string;
};

@action({ UUID: "com.elgato.hello-world.counter" })
class Counter extends SingletonAction<Settings> {
    override onDidReceiveSettings(ev: DidReceiveSettingsEvent<Settings>): void {
        // Handle settings from Property Inspector
        console.log('Settings received:', ev.payload.settings);
    }
}
```

---

## Best Practices for Property Inspectors

Use this checklist when designing or reviewing a Property Inspector:

- **User experience**
  - Keep the interface **simple and intuitive**.
  - Use clear labels and helpful tooltips where appropriate.
  - Group related settings logically.
  - Provide sensible defaults so actions work immediately.

- **Component usage**
  - Use appropriate components for each input type (don't use textfield for booleans).
  - Leverage `sdpi-item` with `label` for consistent layout.
  - Follow the Stream Deck design guidelines for consistency.

- **Settings management**
  - Use the `setting` attribute on components for automatic binding.
  - Validate user input before sending to the plugin.
  - Provide user-friendly error messages for invalid input.
  - Ensure settings persist correctly across app restarts.

- **Performance**
  - Keep HTML/CSS/JavaScript files small and optimized.
  - Avoid blocking operations in Property Inspector code.
  - Minimize external dependencies when possible.

- **Distribution**
  - Always use **local references** to `sdpi-components.js` in distributed plugins.
  - Test the Property Inspector without an internet connection.
  - Ensure all referenced assets (images, scripts) are included in the plugin bundle.

- **Framework usage**
  - If using React or other frameworks, ensure they work within the web view constraints.
  - Test thoroughly across different scenarios.
  - Keep bundle sizes reasonable.

- **Communication**
  - Only send necessary data between Property Inspector and plugin.
  - Handle cases where the plugin might not be ready to receive messages.
  - Implement proper error handling for communication failures.

---

## Common Patterns

### Pattern 1: Simple Settings Form

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <sdpi-item label="Action Name">
        <sdpi-textfield setting="actionName" placeholder="Enter action name"></sdpi-textfield>
    </sdpi-item>
    
    <sdpi-item label="Enabled">
        <sdpi-checkbox setting="enabled"></sdpi-checkbox>
    </sdpi-item>
    
    <sdpi-item label="Timeout (seconds)">
        <sdpi-range setting="timeout" min="1" max="60" step="1"></sdpi-range>
    </sdpi-item>
</body>
</html>
```

### Pattern 2: Settings with Validation

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <sdpi-item label="API Key">
        <sdpi-textfield setting="apiKey" type="password"></sdpi-textfield>
    </sdpi-item>
    
    <sdpi-item label="Server URL">
        <sdpi-textfield setting="serverUrl" placeholder="https://api.example.com"></sdpi-textfield>
    </sdpi-item>

    <script>
        const { streamDeckClient } = SDPIComponents;
        
        // Validate before sending
        function validateAndSend() {
            const apiKey = document.querySelector('[setting="apiKey"]').value;
            const serverUrl = document.querySelector('[setting="serverUrl"]').value;
            
            if (!apiKey || !serverUrl) {
                alert('Please fill in all required fields');
                return;
            }
            
            if (!serverUrl.startsWith('http://') && !serverUrl.startsWith('https://')) {
                alert('Server URL must start with http:// or https://');
                return;
            }
            
            streamDeckClient.setSettings({
                apiKey: apiKey,
                serverUrl: serverUrl
            });
        }
    </script>
</body>
</html>
```

### Pattern 3: Dynamic UI Updates

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <sdpi-item label="Mode">
        <sdpi-select setting="mode">
            <option value="simple">Simple</option>
            <option value="advanced">Advanced</option>
        </sdpi-select>
    </sdpi-item>
    
    <div id="advanced-settings" style="display: none;">
        <sdpi-item label="Advanced Option">
            <sdpi-textfield setting="advancedOption"></sdpi-textfield>
        </sdpi-item>
    </div>

    <script>
        const { streamDeckClient } = SDPIComponents;
        const modeSelect = document.querySelector('[setting="mode"]');
        const advancedDiv = document.getElementById('advanced-settings');
        
        modeSelect.addEventListener('change', (e) => {
            if (e.target.value === 'advanced') {
                advancedDiv.style.display = 'block';
            } else {
                advancedDiv.style.display = 'none';
            }
        });
    </script>
</body>
</html>
```

---

## Checklists

### New Property Inspector Checklist

- [ ] HTML file created in `ui/` directory.
- [ ] `sdpi-components.js` referenced (local for distribution, remote for development).
- [ ] `PropertyInspectorPath` added to `manifest.json` (action or plugin level).
- [ ] UI components selected and implemented.
- [ ] `setting` attributes properly configured on components.
- [ ] Settings communication tested (PI → Plugin).
- [ ] Property Inspector visible and functional in Stream Deck app.

### Pre‑Release Checklist

- [ ] Local `sdpi-components.js` included (not remote CDN).
- [ ] All settings validated before sending to plugin.
- [ ] Property Inspector tested without internet connection.
- [ ] UI is intuitive and follows Stream Deck design guidelines.
- [ ] Settings persist correctly across app restarts.
- [ ] Error handling implemented for invalid input.
- [ ] Property Inspector works with all target actions (if plugin-level PI).

---

## How to Use This Skill with an Agent

When you invoke a coding agent to work on a Stream Deck Property Inspector:

1. **State clearly** that you are working on a Property Inspector for an Elgato Stream Deck plugin.
2. **Ask the agent to follow this skill** for:
   - Creating Property Inspector HTML files in the correct location.
   - Configuring `manifest.json` with `PropertyInspectorPath`.
   - Using sdpi-components for UI elements.
   - Implementing settings communication between Property Inspector and plugin.
   - Debugging Property Inspector issues.
3. **Provide any existing code or manifest files** so the agent can:
   - Avoid duplicating Property Inspectors.
   - Reuse existing settings structures.
   - Maintain consistency with current plugin implementation.

This ensures a consistent, robust Property Inspector development workflow aligned with the official Stream Deck SDK documentation and best practices.
