---
name: streamdeck-native-plugin
description: Comprehensive guide for implementing native Stream Deck plugins (C++, C#, etc.) that communicate with Stream Deck via WebSocket. Native plugins are launched as executable binaries with specific command-line arguments and communicate using the WebSocket protocol defined by the Stream Deck SDK.
---

# Stream Deck Native Plugin Development Skill

Comprehensive guide for implementing **native Stream Deck plugins** (C++, C#, or other compiled languages) that communicate with the Stream Deck application via WebSocket. Unlike Node.js plugins, native plugins are standalone executable binaries launched with specific command-line arguments.

References:
- [Stream Deck SDK – Plugin WebSocket API](https://docs.elgato.com/streamdeck/sdk/references/websocket/plugin/)

---

## When to Use

Use this skill whenever you need to:

- **Create a new native Stream Deck plugin** (C++, C#, or other compiled languages)
- **Implement WebSocket communication** between a native plugin and Stream Deck
- **Handle command-line arguments** passed by Stream Deck at plugin launch
- **Parse RegistrationInfo** and establish WebSocket connection
- **Implement event handlers** for Stream Deck events (willAppear, keyDown, etc.)
- **Send commands** to Stream Deck (setTitle, setImage, etc.)
- **Debug native plugin WebSocket communication**

**Important Note:** Creating native plugins is an advanced technique. Consider using Node.js plugins with native addons instead, unless you have specific requirements for native code.

---

## High-Level Concepts

Before implementing a native plugin, understand these core concepts:

### Native Plugin vs Node.js Plugin

- **Native Plugin**
  - Compiled executable binary (`.exe` on Windows, executable on macOS)
  - Launched by Stream Deck with command-line arguments
  - Communicates via WebSocket protocol
  - Suitable for C++, C#, Rust, Go, or other compiled languages

- **Node.js Plugin**
  - Uses Node.js runtime launched by Stream Deck
  - Also communicates via WebSocket, but uses Stream Deck SDK helpers
  - Easier to develop and maintain for most use cases

### Plugin Launch Process

1. **Stream Deck launches your plugin binary** with command-line arguments:
   - `-port <number>`: WebSocket port
   - `-pluginUUID <uuid>`: Your plugin's UUID
   - `-registerEvent <eventName>`: Registration event name
   - `-info <jsonString>`: Serialized RegistrationInfo JSON

2. **Plugin connects to WebSocket** on the specified port

3. **Plugin sends registration message** to Stream Deck

4. **Stream Deck acknowledges registration** and begins sending events

5. **Plugin and Stream Deck communicate** via JSON messages over WebSocket

### WebSocket Message Format

All messages are **JSON-serialized strings** sent over WebSocket:

- **Events** (Stream Deck → Plugin): Stream Deck sends events when actions appear, keys are pressed, etc.
- **Commands** (Plugin → Stream Deck): Plugin sends commands to update images, titles, settings, etc.

---

## Prerequisites Checklist

Before starting implementation, verify:

1. **Environment**
   - **Stream Deck desktop app 6.9+** is installed
   - A **Stream Deck device** (or virtual device) is available
   - **Development environment** for your target language (C++, C#, Rust, Go, etc.)

2. **WebSocket Library**
   - Choose a WebSocket library for your language:
     - **C++**: WebSocket++, libwebsockets, or similar
     - **C#**: System.Net.WebSockets, or third-party libraries
     - **Rust**: tokio-tungstenite, websocket
     - **Go**: gorilla/websocket, nhooyr.io/websocket

3. **JSON Parsing Library**
   - Choose a JSON library for your language:
     - **C++**: nlohmann/json, RapidJSON, jsoncpp
     - **C#**: System.Text.Json, Newtonsoft.Json
     - **Rust**: serde_json
     - **Go**: encoding/json

4. **Project Planning**
   - Know **what your plugin should do** (feature description)
   - Know **which actions** you will expose (defined in `manifest.json`)
   - Understand **external APIs/services** the plugin must call (if any)

---

## Repository & File Structure

A typical native plugin structure:

```text
my-native-streamdeck-plugin/
  manifest.json              # Plugin metadata and action definitions
  bin/
    win/                     # Windows executable
      plugin.exe
    mac/                     # macOS executable
      plugin
  src/                       # Source code
    main.cpp                 # Entry point (or main.cs, main.rs, etc.)
    websocket_client.h/cpp   # WebSocket communication
    event_handler.h/cpp      # Event handling logic
    ...
  images/                    # Icons for plugin and actions
    pluginIcon.png
    actionDefault.png
  property-inspector/        # Optional Property Inspector UI
    index.html
    index.js
  README.md
  build/                     # Build artifacts
  CMakeLists.txt            # Build configuration (or .sln, Cargo.toml, etc.)
```

**Note:** The executable path must be specified correctly in `manifest.json` under `CodePath` (Windows) or `CodePathMac` (macOS).

---

## Manifest Configuration

In `manifest.json`, specify the native executable:

```json
{
  "CodePath": "bin/win/plugin.exe",
  "CodePathMac": "bin/mac/plugin",
  "CodePathWin": "bin/win/plugin.exe",
  ...
}
```

The manifest structure is otherwise identical to Node.js plugins. See the Stream Deck SDK documentation for full manifest schema.

---

## Step-by-Step: Creating a Native Plugin

> **Note:** If you're using a Stream Deck library for your language (see [Language-Specific Notes](#language-specific-notes)), many of these steps are handled automatically. The library will parse arguments, establish WebSocket connections, handle registration, and route events to your handlers. This section provides low-level implementation details for understanding the protocol or implementing without a library.

### 1. Parse Command-Line Arguments

When Stream Deck launches your plugin, it passes these arguments:

```
-port <number> -pluginUUID <uuid> -registerEvent <eventName> -info <jsonString>
```

**Implementation Example (C++):**

```cpp
struct LaunchArguments {
    int port;
    std::string pluginUUID;
    std::string registerEvent;
    std::string infoJson;
};

LaunchArguments parseArguments(int argc, char* argv[]) {
    LaunchArguments args;
    for (int i = 1; i < argc; i += 2) {
        std::string flag = argv[i];
        if (i + 1 >= argc) break;
        
        if (flag == "-port") {
            args.port = std::stoi(argv[i + 1]);
        } else if (flag == "-pluginUUID") {
            args.pluginUUID = argv[i + 1];
        } else if (flag == "-registerEvent") {
            args.registerEvent = argv[i + 1];
        } else if (flag == "-info") {
            args.infoJson = argv[i + 1];
        }
    }
    return args;
}
```

### 2. Parse RegistrationInfo

The `-info` argument contains a JSON string with RegistrationInfo:

```cpp
// Example RegistrationInfo structure
struct RegistrationInfo {
    struct Application {
        std::string font;
        std::string language;
        std::string platform;
        std::string platformVersion;
        std::string version;
    } application;
    
    struct Colors {
        std::string buttonMouseOverBackgroundColor;
        std::string buttonPressedBackgroundColor;
        std::string buttonPressedBorderColor;
        std::string buttonPressedTextColor;
        std::string highlightColor;
    } colors;
    
    double devicePixelRatio;
    
    struct Device {
        std::string id;
        std::string name;
        struct Size {
            int columns;
            int rows;
        } size;
        std::string type;
    };
    std::vector<Device> devices;
    
    struct Plugin {
        std::string uuid;
        std::string version;
    } plugin;
};

RegistrationInfo parseInfoJson(const std::string& jsonStr) {
    // Parse JSON using your chosen library (e.g., nlohmann/json)
    // Return RegistrationInfo structure
}
```

**RegistrationInfo Fields:**

- `application`: Stream Deck application info (font, language, platform, version)
- `colors`: UI colors used by Stream Deck
- `devicePixelRatio`: Screen DPI ratio
- `devices`: Array of Stream Deck devices (may include disconnected devices)
- `plugin`: Your plugin's UUID and version

### 3. Establish WebSocket Connection

Connect to the WebSocket server on `localhost:<port>`:

```cpp
#include <websocketpp/client.hpp>
#include <websocketpp/config/asio_client.hpp>

using Client = websocketpp::client<websocketpp::config::asio_client>;
using MessagePtr = Client::message_ptr;

Client client;
client.init_asio();

std::string uri = "ws://127.0.0.1:" + std::to_string(port);
client.connect(uri);
```

**Connection Requirements:**

- Protocol: `ws://` (WebSocket, not WSS)
- Host: `127.0.0.1` (localhost)
- Port: Value from `-port` argument

### 4. Send Registration Message

After connecting, immediately send a registration message:

```cpp
// Registration message format
struct RegisterEvent {
    std::string event;  // Value from -registerEvent argument
    std::string uuid;   // Value from -pluginUUID argument
};

void sendRegistration(Client& client, const std::string& registerEvent, const std::string& pluginUUID) {
    RegisterEvent regMsg;
    regMsg.event = registerEvent;
    regMsg.uuid = pluginUUID;
    
    // Serialize to JSON
    nlohmann::json j;
    j["event"] = regMsg.event;
    j["uuid"] = regMsg.uuid;
    
    // Send over WebSocket
    client.send(connectionHandle, j.dump(), websocketpp::frame::opcode::text);
}
```

**Critical:** Registration must occur immediately after WebSocket connection is established. Failure to register promptly may cause Stream Deck to terminate the plugin.

### 5. Handle Incoming Events

Set up a message handler to receive events from Stream Deck:

```cpp
client.set_message_handler([&](websocketpp::connection_hdl hdl, MessagePtr msg) {
    std::string payload = msg->get_payload();
    nlohmann::json event = nlohmann::json::parse(payload);
    
    std::string eventType = event["event"];
    
    if (eventType == "willAppear") {
        handleWillAppear(event);
    } else if (eventType == "keyDown") {
        handleKeyDown(event);
    } else if (eventType == "keyUp") {
        handleKeyUp(event);
    } else if (eventType == "willDisappear") {
        handleWillDisappear(event);
    } else if (eventType == "sendToPlugin") {
        handlePropertyInspectorMessage(event);
    } else if (eventType == "didReceiveSettings") {
        handleDidReceiveSettings(event);
    }
    // ... handle other event types
});
```

### 6. Common Event Handlers

#### willAppear

Occurs when an action appears on the Stream Deck:

```cpp
void handleWillAppear(const nlohmann::json& event) {
    std::string context = event["context"];
    std::string action = event["action"];
    std::string device = event["device"];
    
    // Initialize the action
    // Load settings, set initial image/title, etc.
    
    // Example: Set initial image
    sendSetImage(context, "images/actionDefault.png");
}
```

#### keyDown / keyUp

Occurs when user presses/releases a key:

```cpp
void handleKeyDown(const nlohmann::json& event) {
    std::string context = event["context"];
    std::string action = event["action"];
    
    // Execute action logic
    // Update image/title to show pressed state
    sendSetImage(context, "images/actionPressed.png");
}

void handleKeyUp(const nlohmann::json& event) {
    std::string context = event["context"];
    
    // Execute action logic
    // Update image/title back to default state
    sendSetImage(context, "images/actionDefault.png");
}
```

#### didReceiveSettings

Occurs when settings are received (after requesting or when updated in Property Inspector):

```cpp
void handleDidReceiveSettings(const nlohmann::json& event) {
    std::string context = event["context"];
    nlohmann::json settings = event["payload"]["settings"];
    
    // Process settings
    // Update plugin behavior based on settings
}
```

### 7. Send Commands to Stream Deck

Commands are sent as JSON messages with specific structure:

#### setImage

Update the image displayed on a key:

```cpp
void sendSetImage(const std::string& context, const std::string& imagePath) {
    nlohmann::json command;
    command["event"] = "setImage";
    command["context"] = context;
    command["payload"]["image"] = imagePath;
    
    client.send(connectionHandle, command.dump(), websocketpp::frame::opcode::text);
}
```

**Image formats:**
- File path relative to plugin directory (e.g., `"images/icon.png"`)
- Base64-encoded data URI (e.g., `"data:image/png;base64,..."`)

#### setTitle

Update the title displayed on a key:

```cpp
void sendSetTitle(const std::string& context, const std::string& title) {
    nlohmann::json command;
    command["event"] = "setTitle";
    command["context"] = context;
    command["payload"]["title"] = title;
    
    client.send(connectionHandle, command.dump(), websocketpp::frame::opcode::text);
}
```

#### setSettings

Persist settings for an action instance:

```cpp
void sendSetSettings(const std::string& context, const nlohmann::json& settings) {
    nlohmann::json command;
    command["event"] = "setSettings";
    command["context"] = context;
    command["payload"] = settings;
    
    client.send(connectionHandle, command.dump(), websocketpp::frame::opcode::text);
}
```

#### getSettings

Request settings for an action instance:

```cpp
void sendGetSettings(const std::string& context) {
    nlohmann::json command;
    command["event"] = "getSettings";
    command["context"] = context;
    
    client.send(connectionHandle, command.dump(), websocketpp::frame::opcode::text);
    // Response will arrive via didReceiveSettings event
}
```

#### logMessage

Log a message to Stream Deck's log file:

```cpp
void sendLogMessage(const std::string& message) {
    nlohmann::json command;
    command["event"] = "logMessage";
    command["payload"]["message"] = message;
    
    client.send(connectionHandle, command.dump(), websocketpp::frame::opcode::text);
}
```

### 8. Complete Event Reference

**Events Received from Stream Deck:**

- `applicationDidLaunch` - Monitored application launched
- `applicationDidTerminate` - Monitored application terminated
- `deviceDidConnect` - Stream Deck device connected
- `deviceDidDisconnect` - Stream Deck device disconnected
- `deviceDidChange` - Device configuration changed (Stream Deck 7.0+)
- `dialDown` - Dial pressed (Stream Deck +)
- `dialRotate` - Dial rotated (Stream Deck +)
- `dialUp` - Dial released (Stream Deck +)
- `didReceiveDeepLink` - Deep-link message received
- `didReceiveGlobalSettings` - Global settings received
- `didReceivePropertyInspectorMessage` - Message from Property Inspector
- `didReceiveResources` - Resources received (Stream Deck 7.1+)
- `didReceiveSecrets` - Secrets received
- `didReceiveSettings` - Settings received
- `keyDown` - Key pressed
- `keyUp` - Key released
- `propertyInspectorDidAppear` - Property Inspector opened
- `propertyInspectorDidDisappear` - Property Inspector closed
- `systemDidWakeUp` - System woke from sleep
- `titleParametersDidChange` - Title parameters changed
- `touchTap` - Touchscreen tapped (Stream Deck +)
- `willAppear` - Action appeared on Stream Deck
- `willDisappear` - Action disappeared from Stream Deck

**Commands Sent to Stream Deck:**

- `getGlobalSettings` - Request global settings
- `getResources` - Request resources (Stream Deck 7.1+)
- `getSecrets` - Request secrets
- `getSettings` - Request settings
- `logMessage` - Log a message
- `openUrl` - Open URL in browser
- `sendToPropertyInspector` - Send message to Property Inspector
- `setFeedback` - Set feedback layout values
- `setFeedbackLayout` - Set feedback layout
- `setGlobalSettings` - Set global settings
- `setImage` - Set action image
- `setResources` - Set resources (Stream Deck 7.1+)
- `setSettings` - Set action settings
- `setState` - Set action state (for multi-state actions)
- `setTitle` - Set action title
- `setTriggerDescription` - Set encoder trigger descriptions
- `showAlert` - Show alert indicator
- `showOk` - Show OK indicator
- `switchToProfile` - Switch to a profile

See the [official WebSocket API documentation](https://docs.elgato.com/streamdeck/sdk/references/websocket/plugin/) for complete event and command schemas.

---

## Best Practices for Native Plugins

### Architecture

- **Separate concerns:**
  - WebSocket communication layer
  - Event parsing and routing
  - Business logic
  - External API integrations

- **Error handling:**
  - Catch WebSocket connection errors
  - Handle JSON parsing failures gracefully
  - Validate event payloads before processing
  - Log errors appropriately

### Performance

- **Non-blocking I/O:**
  - Use asynchronous WebSocket libraries
  - Avoid blocking the main thread
  - Use background threads for heavy processing

- **Resource management:**
  - Properly close WebSocket connections
  - Clean up resources on shutdown
  - Handle disconnection/reconnection scenarios

### Resilience

- **Connection handling:**
  - Monitor WebSocket connection health
  - Implement reconnection logic if connection drops
  - Handle Stream Deck app restarts gracefully

- **Settings management:**
  - Validate settings before applying
  - Provide sensible defaults
  - Handle missing or malformed settings

### Security

- **Input validation:**
  - Validate all JSON payloads
  - Sanitize file paths before accessing files
  - Avoid command injection risks

- **Sensitive data:**
  - Never hard-code secrets
  - Use `getSecrets` / `setGlobalSettings` for secure storage
  - Avoid logging sensitive information

### Cross-Platform Considerations

- **File paths:**
  - Use platform-appropriate path separators
  - Handle Windows (`C:\...`) vs macOS (`/...`) paths
  - Consider path length limitations

- **Executable paths:**
  - Ensure `CodePath` / `CodePathMac` are correct in manifest
  - Test on target platforms
  - Handle platform-specific dependencies

### Logging & Debugging

- **Use logMessage:**
  - Log important events and state changes
  - Include context identifiers for debugging
  - Log errors with sufficient detail

- **Development vs production:**
  - Consider debug vs release builds
  - Reduce logging verbosity in production
  - Use environment variables for debug flags

---

## Testing Your Native Plugin

### Manual Testing Checklist

1. **Build and package:**
   - [ ] Build executable for target platform
   - [ ] Verify executable path in `manifest.json`
   - [ ] Package plugin (create `.streamDeckPlugin` file)

2. **Installation:**
   - [ ] Install plugin in Stream Deck app
   - [ ] Verify plugin appears in actions list

3. **Basic functionality:**
   - [ ] Add action to a key
   - [ ] Verify `willAppear` event received
   - [ ] Verify key press triggers `keyDown` / `keyUp`
   - [ ] Verify image/title updates work

4. **Settings:**
   - [ ] Configure action in Property Inspector
   - [ ] Verify settings persist
   - [ ] Verify settings loaded on action appear

5. **Edge cases:**
   - [ ] Test device disconnect/reconnect
   - [ ] Test Stream Deck app restart
   - [ ] Test plugin restart via CLI
   - [ ] Test with multiple action instances

### Debugging Tips

- **Check Stream Deck logs:**
  - Logs location varies by platform
  - Look for plugin-specific error messages

- **Use logMessage:**
  - Add logging at key points in your code
  - Include context and state information
  - Check logs after testing actions

- **WebSocket debugging:**
  - Monitor WebSocket traffic (use Wireshark or similar)
  - Verify message formats match documentation
  - Check for connection issues

- **Common issues:**
  - Registration not sent quickly enough → Plugin terminated
  - Invalid JSON format → Messages ignored
  - Wrong context UUID → Updates sent to wrong action
  - Missing image files → Blank keys

---

## Language-Specific Notes

This section provides examples using **Stream Deck-specific libraries** for each language when available. These libraries abstract away WebSocket handling, JSON parsing, and registration boilerplate, making plugin development significantly easier.

### C++

**Official SDK Resources:**
- Elgato provides sample native plugins with a **"Common" folder** containing reusable C++ boilerplate code
- The Common folder includes: socket connection utilities, registration argument parsing, JSON payload handling
- Reference: Elgato Stream Deck SDK samples (CPU plugin example)

**Using Official SDK Common Code:**

```cpp
// Include the Common SDK utilities
#include "Common/ESDConnectionManager.h"
#include "Common/ESDBasePlugin.h"

class MyPlugin : public ESDBasePlugin {
public:
    MyPlugin() {
        // Plugin initialization
    }
    
    void KeyDownForAction(const std::string& inAction, 
                          const std::string& inContext, 
                          const json& inPayload, 
                          const std::string& inDeviceID) override {
        // Handle key press
        mConnectionManager->SetTitle("Pressed!", inContext);
    }
    
    void KeyUpForAction(const std::string& inAction, 
                        const std::string& inContext, 
                        const json& inPayload, 
                        const std::string& inDeviceID) override {
        // Handle key release
        mConnectionManager->SetTitle("Released", inContext);
    }
    
    void WillAppearForAction(const std::string& inAction, 
                             const std::string& inContext, 
                             const json& inPayload, 
                             const std::string& inDeviceID) override {
        // Action appeared - initialize
        mConnectionManager->SetImage("images/actionDefault.png", inContext);
    }
};

// Main entry point
int main(int argc, char* argv[]) {
    auto plugin = std::make_unique<MyPlugin>();
    
    // Common SDK handles argument parsing, WebSocket connection, and registration
    if (!plugin->Connect(argc, argv)) {
        return 1;
    }
    
    // Run event loop (handled by Common SDK)
    plugin->Run();
    return 0;
}
```

**Alternative Low-Level Approach:**

If not using the Common SDK code, use:
- **WebSocket**: WebSocket++ (header-only), libwebsockets, Boost.Beast
- **JSON**: nlohmann/json (header-only), RapidJSON

### C#

**Recommended Libraries:**

1. **StreamDeck-Tools (BarRaider)** - Most comprehensive, recommended for Windows
   - NuGet: `StreamDeck-Tools`
   - GitHub: BarRaider's StreamDeck-Tools
   - Provides base classes, settings handling, image drawing utilities

2. **streamdeck-client-csharp** - Lightweight alternative
   - NuGet: `streamdeck-client-csharp`
   - Wraps WebSocket communication and message handling

**Example using StreamDeck-Tools:**

```csharp
using StreamDeckLib;
using StreamDeckLib.Messages;
using System.Threading.Tasks;

namespace MyStreamDeckPlugin
{
    // Define your action by inheriting from StreamDeckAction
    [ActionUuid(Uuid = "com.example.myplugin.action")]
    public class MyAction : StreamDeckAction<SettingsModel>
    {
        public override async Task OnKeyDown(StreamDeckEventPayload args)
        {
            // Handle key press
            await Manager.SetTitleAsync(args.context, "Pressed!");
            await Manager.SetImageAsync(args.context, "images/actionPressed.png");
        }
        
        public override async Task OnKeyUp(StreamDeckEventPayload args)
        {
            // Handle key release
            await Manager.SetTitleAsync(args.context, "Released");
            await Manager.SetImageAsync(args.context, "images/actionDefault.png");
        }
        
        public override async Task OnWillAppear(StreamDeckEventPayload args)
        {
            // Action appeared - initialize
            await Manager.SetImageAsync(args.context, "images/actionDefault.png");
        }
        
        public override async Task OnDidReceiveSettings(StreamDeckEventPayload args)
        {
            // Settings received
            var settings = args.Payload.GetSettings<SettingsModel>();
            // Process settings...
        }
    }
    
    // Settings model
    public class SettingsModel
    {
        public string SomeSetting { get; set; }
    }
    
    // Program entry point
    class Program
    {
        static async Task Main(string[] args)
        {
            // StreamDeck-Tools handles argument parsing, connection, registration
            using var config = StreamDeckLib.ConfigBuilder.BuildDefaultConfiguration(args, () => new MyAction());
            await StreamDeckLib.ConnectionManager.Initialize(args, config.GetPluginActions())
                .StartAsync();
        }
    }
}
```

**Example using streamdeck-client-csharp:**

```csharp
using StreamdeckClient;
using System.Threading.Tasks;

var client = new StreamdeckClient(port, pluginUUID, registerEvent);
await client.ConnectAsync();

client.On("keyDown", async (eventData) => {
    var context = eventData["context"].ToString();
    await client.SetTitleAsync(context, "Pressed!");
});

client.On("willAppear", async (eventData) => {
    var context = eventData["context"].ToString();
    await client.SetImageAsync(context, "images/actionDefault.png");
});

await client.RunAsync();
```

### Rust

**Recommended Libraries:**

1. **streamdeck-rs** - Designed for plugin development
   - Crate: `streamdeck-rs`
   - Implements WebSocket protocol, event handling, and registration

2. **rusty-patio** - Full SDK with event loop
   - Crate: `rusty-patio`
   - Includes manifest support and example plugins

3. **stream-deck-plugin** - Plugin wrapper
   - Crate: `stream-deck-plugin`
   - Implements core messages and actions

**Example using streamdeck-rs:**

```rust
use streamdeck_rs::StreamDeckPlugin;
use serde_json::Value;
use std::collections::HashMap;

#[derive(Default)]
struct MyPlugin {
    // Plugin state
}

impl StreamDeckPlugin for MyPlugin {
    fn on_key_down(&mut self, context: &str, _payload: &Value) {
        // Handle key press
        self.send_set_title(context, "Pressed!".to_string());
        self.send_set_image(context, "images/actionPressed.png".to_string());
    }
    
    fn on_key_up(&mut self, context: &str, _payload: &Value) {
        // Handle key release
        self.send_set_title(context, "Released".to_string());
        self.send_set_image(context, "images/actionDefault.png".to_string());
    }
    
    fn on_will_appear(&mut self, context: &str, _payload: &Value) {
        // Action appeared - initialize
        self.send_set_image(context, "images/actionDefault.png".to_string());
    }
    
    fn on_did_receive_settings(&mut self, context: &str, settings: &Value) {
        // Settings received
        // Process settings...
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // streamdeck-rs handles argument parsing, connection, and registration
    let mut plugin = MyPlugin::default();
    plugin.run()?;
    Ok(())
}
```

**Example using rusty-patio:**

```rust
use rusty_patio::{Plugin, Event, EventHandler};

struct MyPlugin;

impl EventHandler for MyPlugin {
    fn handle_event(&mut self, event: Event) {
        match event {
            Event::WillAppear { context, .. } => {
                // Initialize action
            }
            Event::KeyDown { context, .. } => {
                // Handle key press
            }
            Event::KeyUp { context, .. } => {
                // Handle key release
            }
            _ => {}
        }
    }
}

fn main() {
    // rusty-patio handles registration and event loop
    let plugin = MyPlugin;
    Plugin::run(plugin);
}
```

### Go

**Recommended Library:**

- **github.com/FlowingSPDG/streamdeck** - Stream Deck plugin SDK for Go
  - Provides complete plugin framework with WebSocket handling, event processing, and command sending

**Example using github.com/FlowingSPDG/streamdeck:**

```go
package main

import (
    "github.com/FlowingSPDG/streamdeck"
    "github.com/FlowingSPDG/streamdeck/events"
)

func main() {
    // Create plugin instance
    // The library handles argument parsing, WebSocket connection, and registration
    plugin := streamdeck.NewPlugin()
    
    // Register event handlers
    plugin.On(events.WillAppear, func(ctx *streamdeck.Context, event events.Event) {
        // Action appeared - initialize
        ctx.SetImage("images/actionDefault.png")
    })
    
    plugin.On(events.KeyDown, func(ctx *streamdeck.Context, event events.Event) {
        // Handle key press
        ctx.SetTitle("Pressed!")
        ctx.SetImage("images/actionPressed.png")
    })
    
    plugin.On(events.KeyUp, func(ctx *streamdeck.Context, event events.Event) {
        // Handle key release
        ctx.SetTitle("Released")
        ctx.SetImage("images/actionDefault.png")
    })
    
    plugin.On(events.DidReceiveSettings, func(ctx *streamdeck.Context, event events.Event) {
        // Settings received
        settings := ctx.GetSettings()
        // Process settings...
    })
    
    // Run plugin (handles connection and event loop)
    if err := plugin.Run(); err != nil {
        panic(err)
    }
}
```

**Alternative Low-Level Approach:**

If not using the Stream Deck library, use:
- **WebSocket**: gorilla/websocket, nhooyr.io/websocket
- **JSON**: encoding/json (standard library)

---

## Checklists

### New Native Plugin Checklist

- [ ] Development environment set up (compiler, WebSocket library, JSON library)
- [ ] `manifest.json` created with correct executable paths
- [ ] Command-line argument parsing implemented
- [ ] WebSocket connection and registration implemented
- [ ] At least one event handler implemented (e.g., `willAppear`, `keyDown`)
- [ ] At least one command implemented (e.g., `setImage`, `setTitle`)
- [ ] Plugin builds successfully for target platform(s)
- [ ] Plugin installs and runs in Stream Deck app

### Pre-Release Checklist

- [ ] All event handlers tested
- [ ] All commands tested
- [ ] Settings persistence verified
- [ ] Property Inspector integration tested (if used)
- [ ] Error handling implemented and tested
- [ ] Logging added at key points
- [ ] Cross-platform testing completed (if supporting multiple platforms)
- [ ] Plugin packaged as `.streamDeckPlugin` file
- [ ] Version updated in `manifest.json`

---

## How to Use This Skill with an Agent

When working with a coding agent on a native Stream Deck plugin:

1. **State clearly** that you are implementing a **native** Stream Deck plugin (not Node.js)

2. **Specify your target language** (C++, C#, Rust, Go, etc.)

3. **Ask the agent to:**
   - Implement command-line argument parsing
   - Set up WebSocket connection and registration
   - Implement event handlers based on your requirements
   - Implement commands to update Stream Deck state
   - Follow the WebSocket protocol specifications

4. **Provide context:**
   - Your chosen WebSocket and JSON libraries
   - Your plugin's manifest and action definitions
   - Any existing code or structures

5. **Reference this skill** for:
   - Event and command message formats
   - Registration process
   - Best practices and error handling
   - Testing and debugging guidance

This ensures a consistent, robust native Stream Deck plugin implementation aligned with the official WebSocket API documentation.
