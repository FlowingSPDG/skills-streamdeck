---
name: sdpi-components
description: Comprehensive reference for sdpi-components (Stream Deck Property Inspector Components) - the official UI library for building Property Inspectors. Covers all components, data sources, Stream Deck Client API, and localization patterns.
---

# sdpi-components Reference Skill

This skill provides comprehensive guidance for using **sdpi-components** (Stream Deck Property Inspector Components), the official UI library for building Property Inspectors in Stream Deck plugins.

It is designed to be used with coding agents compatible with the [add-skill](https://github.com/vercel-labs/add-skill) specification.

References:
- [sdpi-components Documentation](https://sdpi-components.dev/)
- [sdpi-components GitHub](https://github.com/elgato/sdpi-components)
- [Stream Deck SDK – Property Inspectors (UI)](https://docs.elgato.com/streamdeck/sdk/guides/ui/)

---

## When to Use

Use this skill whenever you need to:

- **Build or modify Property Inspector UIs** using sdpi-components
- **Select appropriate components** for different input types
- **Implement data sources** for dynamic content (selects, checkboxes, radios)
- **Use Stream Deck Client API** for plugin communication
- **Handle settings persistence** (action-specific or global)
- **Debug Property Inspector issues** related to sdpi-components
- **Review Property Inspector code** for best practices with sdpi-components

---

## High‑Level Concepts

Before using sdpi-components, keep these core concepts in mind:

- **sdpi-components**
  - The **official UI library** for Stream Deck Property Inspectors.
  - Built with **Lit.js** and compiled to standard web components.
  - Provides consistent, accessible UI components that handle settings persistence automatically.
  - No build setup required — just include a single JavaScript file.

- **Component structure**
  - All components are prefixed with `sdpi-` (e.g., `<sdpi-textfield>`, `<sdpi-checkbox>`).
  - Components are typically wrapped in `<sdpi-item>` with a `label` attribute for consistent layout.
  - Components support the `setting` attribute to bind to settings paths (supports nested paths like `foo.bar.prop`).

- **Settings persistence**
  - Components automatically persist values to action-specific settings via the `setting` attribute.
  - Use the `global` attribute to persist to global settings (shared across all action instances).
  - Settings are automatically synced between Property Inspector and plugin.

- **Communication**
  - Components handle basic settings communication automatically.
  - Use `SDPIComponents.streamDeckClient` for advanced communication patterns.
  - Supports bidirectional communication: Property Inspector ↔ Plugin.

- **Data sources**
  - Some components (`<sdpi-select>`, `<sdpi-checkbox-list>`, `<sdpi-radio>`) support dynamic data sources.
  - Data sources fetch items from the plugin at runtime.
  - Supports hot-reload for real-time updates.

---

## Prerequisites Checklist

Before using sdpi-components, verify all of the following:

1. **Property Inspector setup**
   - You have a Property Inspector HTML file in your plugin's `ui/` directory.
   - The `PropertyInspectorPath` is configured in `manifest.json` (action or plugin level).

2. **Library inclusion**
   - Decide whether to use local (recommended) or remote (development) reference.
   - If using locally, download `sdpi-components.js` from [releases](https://github.com/elgato/sdpi-components/releases).
   - Place `sdpi-components.js` in the `ui/` directory alongside your HTML file.

3. **Technical decisions**
   - Determine which components you need for your settings UI.
   - Plan your settings data structure (what properties need to be configured).
   - Consider whether you need data sources for dynamic content.

---

## Repository & File Structure

Property Inspector files using sdpi-components should be organized as follows:

```text
my-streamdeck-plugin/
  *.sdPlugin/
    ├── bin/
    ├── imgs/
    ├── ui/                    # Property Inspector files
    │   ├── my-action.html
    │   └── sdpi-components.js  # Local library (recommended)
    └── manifest.json
  src/
    ├── actions/
    └── plugin.ts
```

The `ui/` directory contains all Property Inspector HTML files and the `sdpi-components.js` library.

---

## Step‑by‑Step: Getting Started with sdpi-components

Follow these steps to use sdpi-components in your Property Inspector:

### 1. Include the library

**Local (recommended for distribution):**

1. Download `sdpi-components.js` from the [releases page](https://github.com/elgato/sdpi-components/releases).
2. Place it in your `ui/` directory.
3. Reference it in your HTML:

```html
<script src="sdpi-components.js"></script>
```

**Remote (development/prototyping only):**

```html
<script src="https://sdpi-components.dev/releases/v4/sdpi-components.js"></script>
```

⚠️ **Important:** Always use local references when distributing your plugin to ensure it works offline and provides a consistent experience.

### 2. Basic HTML structure

Create your Property Inspector HTML file:

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <sdpi-item label="Name">
        <sdpi-textfield setting="name" placeholder="Enter name"></sdpi-textfield>
    </sdpi-item>
    
    <sdpi-item label="Enabled">
        <sdpi-checkbox setting="enabled"></sdpi-checkbox>
    </sdpi-item>
</body>
</html>
```

### 3. Configure components with settings

Use the `setting` attribute to bind components to your settings:

```html
<sdpi-item label="Favorite Color">
    <sdpi-color setting="favColor"></sdpi-color>
</sdpi-item>

<sdpi-item label="Timeout (seconds)">
    <sdpi-range setting="timeout" min="1" max="60" step="1" showlabels></sdpi-range>
</sdpi-item>
```

### 4. Use global settings (optional)

To persist settings globally (shared across all action instances):

```html
<sdpi-item label="API Key">
    <sdpi-textfield setting="apiKey" global></sdpi-textfield>
</sdpi-item>
```

### 5. Access Stream Deck Client (optional)

For advanced communication patterns:

```html
<script>
    const { streamDeckClient } = SDPIComponents;
    
    // Get current settings
    streamDeckClient.getSettings().then(payload => {
        console.log('Current settings:', payload.settings);
    });
    
    // Listen for settings updates
    streamDeckClient.didReceiveSettings.subscribe(ev => {
        console.log('Settings updated:', ev.payload.settings);
    });
    
    // Set settings programmatically
    streamDeckClient.setSettings({
        name: "John Doe",
        enabled: true
    });
</script>
```

---

## Available Components

The sdpi-components library provides the following web components:

### Form Input Components

#### `<sdpi-textfield>`

Single-line text input.

**Attributes:**
- `setting` (string): Path in settings object (supports nested paths)
- `value` (string): Current value
- `default` (string): Default value if setting is undefined
- `placeholder` (string): Placeholder text
- `disabled` (boolean): Whether the input is disabled
- `required` (boolean): Shows icon when value is empty
- `maxlength` (number): Maximum length for the input string
- `pattern` (string): Regular expression for validation
- `global` (boolean): Persist to global settings

**Example:**
```html
<sdpi-item label="Name">
    <sdpi-textfield 
        setting="name" 
        placeholder="Enter your name"
        required>
    </sdpi-textfield>
</sdpi-item>
```

#### `<sdpi-textarea>`

Multi-line text input.

**Attributes:**
- `setting` (string): Path in settings object
- `value` (string): Current value
- `default` (string): Default value
- `placeholder` (string): Placeholder text
- `disabled` (boolean): Whether the input is disabled
- `rows` (number): Number of visible rows
- `global` (boolean): Persist to global settings

**Example:**
```html
<sdpi-item label="Description">
    <sdpi-textarea setting="description" rows="4"></sdpi-textarea>
</sdpi-item>
```

#### `<sdpi-password>`

Password input (obscured text).

**Attributes:**
- `setting` (string): Path in settings object
- `value` (string): Current value
- `default` (string): Default value
- `placeholder` (string): Placeholder text
- `disabled` (boolean): Whether the input is disabled
- `global` (boolean): Persist to global settings

**Example:**
```html
<sdpi-item label="API Key">
    <sdpi-password setting="apiKey"></sdpi-password>
</sdpi-item>
```

### Selection Components

#### `<sdpi-checkbox>`

Single boolean checkbox.

**Attributes:**
- `setting` (string): Path in settings object
- `value` (boolean): Current state
- `default` (boolean): Default state if setting is undefined
- `label` (string): Label shown next to checkbox
- `disabled` (boolean): Whether the checkbox is disabled
- `global` (boolean): Persist to global settings

**Example:**
```html
<sdpi-item label="Options">
    <sdpi-checkbox setting="enabled" label="Enable feature"></sdpi-checkbox>
</sdpi-item>
```

#### `<sdpi-checkbox-list>`

Multiple checkboxes (supports data sources).

**Attributes:**
- `setting` (string): Path in settings object (stores array of selected values)
- `datasource` (string): Optional remote data source event name
- `loading` (string): Text shown while loading data source
- `hot-reload` (boolean): Monitor `sendToPropertyInspector` for updates
- `global` (boolean): Persist to global settings

**Example (static):**
```html
<sdpi-item label="Features">
    <sdpi-checkbox-list setting="features">
        <option value="feature1">Feature 1</option>
        <option value="feature2">Feature 2</option>
        <option value="feature3">Feature 3</option>
    </sdpi-checkbox-list>
</sdpi-item>
```

**Example (with data source):**
```html
<sdpi-item label="Available Devices">
    <sdpi-checkbox-list 
        setting="devices"
        datasource="getDevices"
        loading="Loading devices...">
    </sdpi-checkbox-list>
</sdpi-item>
```

#### `<sdpi-radio>`

Radio button group (single selection, supports data sources).

**Attributes:**
- `setting` (string): Path in settings object
- `datasource` (string): Optional remote data source event name
- `loading` (string): Text shown while loading data source
- `hot-reload` (boolean): Monitor `sendToPropertyInspector` for updates
- `global` (boolean): Persist to global settings

**Example (static):**
```html
<sdpi-item label="Mode">
    <sdpi-radio setting="mode">
        <option value="simple">Simple</option>
        <option value="advanced">Advanced</option>
    </sdpi-radio>
</sdpi-item>
```

#### `<sdpi-select>`

Dropdown select (single selection, supports data sources).

**Attributes:**
- `setting` (string): Path in settings object
- `value` (string | number | boolean): Current selected value
- `default` (string): Default value if setting is undefined
- `value-type` (`'string'` | `'number'` | `'boolean'`): How to interpret the value (default: `'string'`)
- `placeholder` (string): Placeholder text when no selection
- `label-setting` (string): Optional path to persist the label of selected option
- `datasource` (string): Optional remote data source event name
- `loading` (string): Text shown while loading data source
- `hot-reload` (boolean): Monitor `sendToPropertyInspector` for updates
- `disabled` (boolean): Whether the select is disabled
- `global` (boolean): Persist to global settings

**Example (static):**
```html
<sdpi-item label="Color">
    <sdpi-select setting="color" placeholder="Choose a color">
        <optgroup label="Primary Colors">
            <option value="#ff0000">Red</option>
            <option value="#00ff00">Green</option>
            <option value="#0000ff">Blue</option>
        </optgroup>
        <option value="#000000">Black</option>
        <option value="#ffffff">White</option>
    </sdpi-select>
</sdpi-item>
```

**Example (with data source):**
```html
<sdpi-item label="Device">
    <sdpi-select 
        setting="device"
        datasource="getDevices"
        loading="Loading devices..."
        placeholder="Select a device">
    </sdpi-select>
</sdpi-item>
```

### Numeric Input Components

#### `<sdpi-range>`

Range slider for numeric values.

**Attributes:**
- `setting` (string): Path in settings object
- `value` (string): Current value
- `default` (string): Default value
- `min` (number): Minimum value
- `max` (number): Maximum value
- `step` (number): Step interval
- `showlabels` (boolean): Show min/max labels
- `disabled` (boolean): Whether the range is disabled
- `global` (boolean): Persist to global settings

**Example:**
```html
<sdpi-item label="Brightness">
    <sdpi-range 
        setting="brightness" 
        min="0" 
        max="100" 
        step="5"
        showlabels>
    </sdpi-range>
</sdpi-item>
```

**Custom labels:**
```html
<sdpi-range setting="volume" min="0" max="100" showlabels>
    <span slot="label-min">Mute</span>
    <span slot="label-max">Max</span>
</sdpi-range>
```

### Date/Time Components

#### `<sdpi-calendar>`

Date, time, and datetime picker (supports multiple types).

**Attributes:**
- `type` (string): One of `'date'`, `'datetime-local'`, `'month'`, `'time'`, `'week'`
- `setting` (string): Path in settings object
- `value` (string): Current value (ISO format)
- `default` (string): Default value
- `min` (string): Minimum date/time (ISO format)
- `max` (string): Maximum date/time (ISO format)
- `step` (number): Step interval (especially for time)
- `disabled` (boolean): Whether the calendar is disabled
- `global` (boolean): Persist to global settings

**Examples:**
```html
<sdpi-item label="Date">
    <sdpi-calendar type="date" setting="important_date"></sdpi-calendar>
</sdpi-item>

<sdpi-item label="Time">
    <sdpi-calendar type="time" setting="alarm_time" step="300"></sdpi-calendar>
</sdpi-item>

<sdpi-item label="Date and Time">
    <sdpi-calendar type="datetime-local" setting="event_datetime"></sdpi-calendar>
</sdpi-item>
```

### Special Components

#### `<sdpi-color>`

Color picker (returns hex string).

**Attributes:**
- `setting` (string): Path in settings object
- `value` (string): Current color (hex format, e.g., `"#ff0000"`)
- `default` (string): Default color
- `disabled` (boolean): Whether the color picker is disabled
- `global` (boolean): Persist to global settings

**Example:**
```html
<sdpi-item label="Background Color">
    <sdpi-color setting="bgColor" default="#ffffff"></sdpi-color>
</sdpi-item>
```

#### `<sdpi-file>`

File picker input.

**Attributes:**
- `setting` (string): Path in settings object
- `accept` (string): File type filter (e.g., `"image/*"`, `".json"`)
- `label` (string): Button label text
- `disabled` (boolean): Whether the file picker is disabled
- `global` (boolean): Persist to global settings

**Example:**
```html
<sdpi-item label="Icon File">
    <sdpi-file 
        setting="iconPath" 
        accept="image/*"
        label="Choose Icon">
    </sdpi-file>
</sdpi-item>
```

#### `<sdpi-button>`

Action button.

**Attributes:**
- `label` (string): Button text
- `disabled` (boolean): Whether the button is disabled

**Example:**
```html
<sdpi-item label="Actions">
    <sdpi-button label="Test Connection" onclick="testConnection()"></sdpi-button>
</sdpi-item>
```

#### `<sdpi-delegate>`

Custom component wrapper for plugin-invoked actions (e.g., folder picker).

**Attributes:**
- `setting` (string): Path in settings object (value is set by plugin)
- `label` (string): Button label text
- `disabled` (boolean): Whether the delegate is disabled
- `global` (boolean): Persist to global settings

**Example:**
```html
<sdpi-item label="Folder">
    <sdpi-delegate 
        setting="folderPath" 
        label="Choose Folder">
    </sdpi-delegate>
</sdpi-item>
```

**Plugin side (TypeScript example):**
```typescript
// Send folder picker request
streamDeck.actions.openUrl("file:///path/to/folder");

// Or use sendToPropertyInspector to set the value
streamDeck.actions.sendToPropertyInspector(context, {
    event: "folderSelected",
    path: "/selected/folder/path"
});
```

### Layout Component

#### `<sdpi-item>`

Wrapper component for consistent layout (label + component).

**Attributes:**
- `label` (string): Label text displayed on the left
- `info` (string): Optional info text/tooltip

**Example:**
```html
<sdpi-item label="Name" info="Enter your display name">
    <sdpi-textfield setting="name"></sdpi-textfield>
</sdpi-item>
```

---

## Data Sources

Data sources allow components to dynamically populate their options from the plugin at runtime.

### Supported Components

Data sources are supported in:
- `<sdpi-select>`
- `<sdpi-checkbox-list>`
- `<sdpi-radio>`

### Property Inspector Setup

**HTML:**
```html
<sdpi-item label="Device">
    <sdpi-select
        setting="device"
        datasource="getDevices"
        loading="Fetching devices..."
        hot-reload>
    </sdpi-select>
</sdpi-item>
```

**Configuration attributes:**
- `datasource` (string): Event name that will be sent to the plugin
- `loading` (string): Optional text shown while loading
- `hot-reload` (boolean): Monitor `sendToPropertyInspector` for real-time updates

**Manual refresh:**
```javascript
const selectElement = document.querySelector('[setting="device"]');
selectElement.refresh(); // Manually refresh the data source
```

### Plugin Implementation

When a component with a `datasource` is initialized, the plugin receives a `sendToPlugin` event:

**Request payload:**
```json
{
    "action": "com.example.myaction",
    "event": "sendToPlugin",
    "context": "unique-context-id",
    "payload": {
        "event": "getDevices",
        "isRefresh": undefined | true
    }
}
```

**Plugin response (via `sendToPropertyInspector`):**
```json
{
    "action": "com.example.myaction",
    "event": "sendToPropertyInspector",
    "context": "unique-context-id",
    "payload": {
        "event": "getDevices",
        "items": [
            {
                "label": "Device Group",
                "children": [
                    {
                        "label": "Device 1",
                        "value": "device1"
                    },
                    {
                        "label": "Device 2",
                        "value": "device2"
                    }
                ]
            },
            {
                "label": "Device 3",
                "value": "device3"
            }
        ]
    }
}
```

**TypeScript type definitions:**
```typescript
type DataSourcePayload = {
    event: string;
    items: DataSourceResult;
};

type DataSourceResult = DataSourceResultItem[];

type DataSourceResultItem = Item | ItemGroup;

type Item = {
    disabled?: boolean;
    label?: string;
    value: string;
};

type ItemGroup = {
    label?: string;
    children: Item[];
};
```

**Example plugin implementation (Node.js/TypeScript):**
```typescript
import streamDeck, { action, type SendToPluginEvent } from "@elgato/streamdeck";

@action({ UUID: "com.example.myaction" })
class MyAction {
    override onSendToPlugin(ev: SendToPluginEvent): void {
        if (ev.payload.event === "getDevices") {
            // Fetch devices from your source
            const devices = [
                { label: "Device 1", value: "device1" },
                { label: "Device 2", value: "device2" }
            ];
            
            // Send response to Property Inspector
            streamDeck.actions.sendToPropertyInspector(ev.context, {
                event: "getDevices",
                items: devices
            });
        }
    }
}
```

---

## Stream Deck Client API

The `SDPIComponents.streamDeckClient` provides methods and events for advanced communication between Property Inspector and plugin.

### Accessing the Client

```javascript
const { streamDeckClient } = SDPIComponents;
```

### Methods

#### `getSettings()`

Get current action-specific settings.

**Returns:** Promise resolving to `{ settings: object }`

**Example:**
```javascript
streamDeckClient.getSettings().then(payload => {
    console.log('Current settings:', payload.settings);
});
```

#### `setSettings(settings)`

Update action-specific settings.

**Parameters:**
- `settings` (object): Settings object to persist

**Example:**
```javascript
streamDeckClient.setSettings({
    name: "John Doe",
    enabled: true
});
```

#### `getGlobalSettings()`

Get plugin-wide global settings.

**Returns:** Promise resolving to `{ settings: object }`

**Example:**
```javascript
streamDeckClient.getGlobalSettings().then(payload => {
    console.log('Global settings:', payload.settings);
});
```

#### `setGlobalSettings(settings)`

Update plugin-wide global settings.

**Parameters:**
- `settings` (object): Global settings object to persist

**Example:**
```javascript
streamDeckClient.setGlobalSettings({
    apiKey: "secret-key",
    theme: "dark"
});
```

#### `getConnectionInfo()`

Get information about the Stream Deck environment.

**Returns:** Promise resolving to connection info object

**Example:**
```javascript
streamDeckClient.getConnectionInfo().then(info => {
    console.log('Plugin UUID:', info.plugin.uuid);
    console.log('Action:', info.actionInfo.name);
    console.log('OS:', info.applicationInfo.platform);
    console.log('Language:', info.applicationInfo.language);
    console.log('Devices:', info.devices);
});
```

#### `send(event, payload)`

Send custom events to the plugin or host environment.

**Parameters:**
- `event` (string): Event name (e.g., `"sendToPlugin"`, `"openUrl"`)
- `payload` (object): Event payload

**Example:**
```javascript
// Send custom message to plugin
streamDeckClient.send("sendToPlugin", {
    event: "customEvent",
    data: { foo: "bar" }
});

// Open URL
streamDeckClient.send("openUrl", {
    url: "https://example.com"
});
```

### Events

#### `didReceiveSettings`

Subscribe to settings updates from the plugin.

**Example:**
```javascript
streamDeckClient.didReceiveSettings.subscribe(ev => {
    console.log('Settings received:', ev.payload.settings);
    // Update UI if needed
});
```

#### `didReceiveGlobalSettings`

Subscribe to global settings updates from the plugin.

**Example:**
```javascript
streamDeckClient.didReceiveGlobalSettings.subscribe(ev => {
    console.log('Global settings received:', ev.payload.settings);
});
```

#### `sendToPropertyInspector`

Subscribe to custom messages from the plugin.

**Example:**
```javascript
streamDeckClient.sendToPropertyInspector.subscribe(ev => {
    if (ev.payload.event === "deviceConnected") {
        console.log('Device connected:', ev.payload.device);
    }
});
```

---

## Localization

### Manifest Localization

Stream Deck supports localization via JSON files in your plugin directory:

**File structure:**
```text
*.sdPlugin/
  ├── manifest.json
  ├── en.json      # English (default)
  ├── fr.json      # French
  ├── de.json      # German
  ├── es.json      # Spanish
  ├── ja.json      # Japanese
  ├── ko.json      # Korean
  └── zh_CN.json   # Chinese (Simplified)
```

**Example `en.json`:**
```json
{
    "Name": "My Plugin",
    "Description": "A great plugin",
    "Actions": {
        "MyAction": {
            "Name": "My Action",
            "Tooltip": "Does something"
        }
    }
}
```

**Example `fr.json`:**
```json
{
    "Name": "Mon Plugin",
    "Description": "Un super plugin",
    "Actions": {
        "MyAction": {
            "Name": "Mon Action",
            "Tooltip": "Fait quelque chose"
        }
    }
}
```

### Property Inspector Localization

sdpi-components does not provide built-in localization for Property Inspector UI labels. To localize your Property Inspector:

1. **Get language from connection info:**
```javascript
const { streamDeckClient } = SDPIComponents;

streamDeckClient.getConnectionInfo().then(info => {
    const language = info.applicationInfo.language; // e.g., "en", "fr", "de"
    updateUILabels(language);
});
```

2. **Implement your own translation system:**
```javascript
const translations = {
    en: {
        name: "Name",
        enabled: "Enabled"
    },
    fr: {
        name: "Nom",
        enabled: "Activé"
    }
};

function updateUILabels(language) {
    const t = translations[language] || translations.en;
    document.querySelector('[setting="name"]').parentElement
        .querySelector('sdpi-item').setAttribute('label', t.name);
}
```

---

## Best Practices

Use this checklist when building Property Inspectors with sdpi-components:

- **Component selection**
  - Use appropriate components for each input type (don't use textfield for booleans).
  - Leverage `<sdpi-item>` with `label` for consistent layout.
  - Use `placeholder` attributes to guide users.

- **Settings management**
  - Use the `setting` attribute on components for automatic binding.
  - Use nested paths (e.g., `"user.name"`) for organized settings.
  - Use `global` attribute only when settings should be shared across actions.
  - Provide sensible `default` values so actions work immediately.

- **Data sources**
  - Use data sources for dynamic content (devices, API results, etc.).
  - Implement proper error handling in plugin data source handlers.
  - Use `hot-reload` for real-time updates when appropriate.
  - Provide meaningful `loading` text for better UX.

- **Performance**
  - Keep Property Inspector HTML/CSS/JavaScript files small.
  - Avoid blocking operations in Property Inspector code.
  - Use data sources efficiently (cache when appropriate).

- **Distribution**
  - Always use **local references** to `sdpi-components.js` in distributed plugins.
  - Test Property Inspector without an internet connection.
  - Ensure all referenced assets are included in the plugin bundle.

- **User experience**
  - Keep interfaces simple and intuitive.
  - Use clear labels and helpful tooltips (`info` attribute on `<sdpi-item>`).
  - Group related settings logically.
  - Validate user input where appropriate (use `pattern` on textfield, `min`/`max` on range).

- **Communication**
  - Use `streamDeckClient` for advanced communication patterns.
  - Handle cases where the plugin might not be ready.
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
        <sdpi-range setting="timeout" min="1" max="60" step="1" showlabels></sdpi-range>
    </sdpi-item>
</body>
</html>
```

### Pattern 2: Dynamic Select with Data Source

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <sdpi-item label="Device">
        <sdpi-select
            setting="device"
            datasource="getDevices"
            loading="Loading devices..."
            placeholder="Select a device">
        </sdpi-select>
    </sdpi-item>

    <script>
        const { streamDeckClient } = SDPIComponents;
        
        // Listen for device updates
        streamDeckClient.sendToPropertyInspector.subscribe(ev => {
            if (ev.payload.event === "deviceListUpdated") {
                // Refresh the select
                document.querySelector('[setting="device"]').refresh();
            }
        });
    </script>
</body>
</html>
```

### Pattern 3: Conditional UI Updates

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
        
        // Listen for settings changes
        streamDeckClient.didReceiveSettings.subscribe(ev => {
            const mode = ev.payload.settings?.mode;
            if (mode === 'advanced') {
                advancedDiv.style.display = 'block';
            } else {
                advancedDiv.style.display = 'none';
            }
        });
        
        // Also handle local changes
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

### Pattern 4: Global Settings with Action Settings

```html
<!doctype html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <!-- Global settings (shared across all actions) -->
    <sdpi-item label="API Key">
        <sdpi-textfield setting="apiKey" global></sdpi-textfield>
    </sdpi-item>
    
    <!-- Action-specific settings -->
    <sdpi-item label="Action Name">
        <sdpi-textfield setting="actionName"></sdpi-textfield>
    </sdpi-item>
    
    <sdpi-item label="Enabled">
        <sdpi-checkbox setting="enabled"></sdpi-checkbox>
    </sdpi-item>
</body>
</html>
```

---

## Checklists

### New Property Inspector with sdpi-components Checklist

- [ ] `sdpi-components.js` included (local for distribution, remote for development).
- [ ] HTML file created in `ui/` directory.
- [ ] `PropertyInspectorPath` added to `manifest.json` (action or plugin level).
- [ ] Appropriate components selected and implemented.
- [ ] `setting` attributes properly configured on components.
- [ ] Components wrapped in `<sdpi-item>` with labels.
- [ ] Settings communication tested (PI → Plugin).
- [ ] Property Inspector visible and functional in Stream Deck app.

### Data Source Implementation Checklist

- [ ] Component has `datasource` attribute set.
- [ ] Plugin implements `onSendToPlugin` handler for data source event.
- [ ] Plugin sends response via `sendToPropertyInspector` with correct payload structure.
- [ ] `items` array contains valid `Item` or `ItemGroup` objects.
- [ ] `hot-reload` implemented if real-time updates needed.
- [ ] `loading` text provided for better UX.
- [ ] Error handling implemented in plugin.

### Pre‑Release Checklist

- [ ] Local `sdpi-components.js` included (not remote CDN).
- [ ] All settings validated before sending to plugin.
- [ ] Property Inspector tested without internet connection.
- [ ] UI is intuitive and follows Stream Deck design guidelines.
- [ ] Settings persist correctly across app restarts.
- [ ] Error handling implemented for invalid input.
- [ ] Data sources work correctly and handle errors gracefully.
- [ ] Global vs. action-specific settings properly configured.

---

## How to Use This Skill with an Agent

When you invoke a coding agent to work with sdpi-components:

1. **State clearly** that you are working on a Property Inspector using sdpi-components for an Elgato Stream Deck plugin.
2. **Ask the agent to follow this skill** for:
   - Selecting appropriate sdpi-components for different input types.
   - Configuring components with `setting` attributes.
   - Implementing data sources for dynamic content.
   - Using Stream Deck Client API for advanced communication.
   - Handling settings persistence (action-specific or global).
   - Debugging Property Inspector issues related to sdpi-components.
3. **Provide any existing code or manifest files** so the agent can:
   - Avoid duplicating components or settings.
   - Reuse existing settings structures.
   - Maintain consistency with current plugin implementation.

This ensures a consistent, robust Property Inspector development workflow using the official sdpi-components library.
