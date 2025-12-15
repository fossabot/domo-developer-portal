## Overview

App Wrapper v2 introduces a more reliable and performant communication system between Domo and your custom applications. The key improvements include:

| Feature | v1 (Legacy) | v2 (Current) |
|---------|-------------|--------------|
| **Communication** | `iframe.postMessage()` only | MessageChannel + postMessage |
| **Reliability** | Fire-and-forget | Ask-reply with acknowledgments |
| **Peripheral Access** | Limited | Full browser API support via manifest |
| **Error Recovery** | Manual refresh | Automatic retry + refresh fallback |

---

## MessageChannel Architecture

### Why MessageChannel?

Traditional `iframe.postMessage()` broadcasts messages to the window, requiring origin validation on every message and creating overhead. **MessageChannel** establishes a dedicated, private communication port between Domo and your app.

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                      Domo Platform (Parent)                     │
│                                                                 │
│  1. Creates MessageChannel                                      │
│     ├── port1 → Kept by Domo (for receiving responses)          │
│     └── port2 → Transferred to your app (for sending)           │
│                                                                 │
│  2. Sends initial message via window.postMessage()              │
│     with port2 attached for transfer                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Your Custom App (Iframe)                    │
│                                                                 │
│  3. Receives port2 from initial message                         │
│                                                                 │
│  4. All subsequent communication uses port2 directly            │
│     (No more origin checks needed!)                             │
│                                                                 │
│  5. Send acknowledgments back to Domo via the port              │
└─────────────────────────────────────────────────────────────────┘
```

### Benefits

- **Performance**: Direct port-to-port communication without serialization overhead
- **Security**: No origin validation needed after initial handshake
- **Reliability**: Dedicated bidirectional channel for request/response patterns
- **Isolation**: Messages don't leak to other window listeners

### Receiving the MessagePort in Your App

When your app loads, listen for the initial message that transfers the MessagePort:

```javascript
// In your custom Domo app
window.addEventListener('message', (event) => {
  // Check if this message contains a MessagePort
  if (event.ports && event.ports.length > 0) {
    const port = event.ports[0];

    // Store the port for future communication
    window.domoPort = port;

    // Set up listener on the port
    port.onmessage = (portEvent) => {
      const message = portEvent.data;
      handleDomoMessage(message, portEvent);
    };

    console.log('MessagePort established with Domo');
  }
});
```

---

## Ask-Reply Pattern

### Overview

The ask-reply pattern ensures message delivery through **bidirectional acknowledgments**. When Domo sends a message to your app, your app should acknowledge receipt. Similarly, **when your app sends a message to Domo, Domo will send an acknowledgment back**. This two-way confirmation ensures reliable communication in both directions.

### Message Flow (Domo to App)

```
Domo                                    Your App
  │                                        │
  ├─── Message + requestId ───────────────>│
  │    (via MessagePort)                   │
  │                                        ├── Process message
  │<────────── ack + requestId ────────────┤
  │            (via MessagePort)           │
  │                                        │
  ├── Received! Clear timeout              │
  │                                        │
```

### Message Flow (App to Domo)

```
Your App                                  Domo
  │                                        │
  ├─── Message + requestId ───────────────>│
  │    (via MessagePort)                   │
  │                                        ├── Process message
  │<────────── ack + requestId ────────────┤
  │            (via MessagePort)           │
  │                                        │
  ├── Received! Confirm delivery           │
  │                                        │
```

This bidirectional acknowledgment system means your app can also verify that Domo received its messages (such as filter updates or navigation requests).

### Acknowledgment Protocol

Every message from Domo that requires acknowledgment includes a `requestId`. Your app must respond with an `ack` event containing the same `requestId`.

**Message from Domo:**
```json
{
  "event": "dataUpdated",
  "alias": "sales-data",
  "requestId": "abc123-def456"
}
```

**Your acknowledgment:**
```json
{
  "event": "ack",
  "requestId": "abc123-def456"
}
```

### Implementation Example

```javascript
function handleDomoMessage(message, event) {
  const { event: eventType, requestId } = message;

  switch (eventType) {
    case 'dataUpdated':
      // Handle dataset update
      refreshDataset(message.alias);
      break;

    case 'filtersUpdated':
      // Handle filter changes
      applyFilters(message.filters, message.aliases);
      break;

    case 'variablesUpdated':
      // Handle variable changes
      updateVariables(message.variables);
      break;
  }

  // IMPORTANT: Send acknowledgment if requestId is present
  if (requestId && window.domoPort) {
    window.domoPort.postMessage({
      event: 'ack',
      requestId: requestId
    });
  }
}
```

### Timeout and Recovery

| Scenario | Timeout | Action |
|----------|---------|--------|
| Ack received | - | Continue normally |
| No ack, first failure | 500ms | Debounced retry |
| Repeated failures | 500ms each | Trigger iframe refresh |

The 500ms timeout is designed to be fast enough for real-time updates while allowing for network latency.

---

## Peripheral Device Support

### Overview

App Wrapper v2 allows your custom apps to request access to browser peripheral APIs such as camera, microphone, geolocation, bluetooth, and NFC. These permissions are controlled through **boolean properties** in your app's `manifest.json` file.

### Supported Peripherals

| Peripheral | Manifest Property | Default | Use Cases |
|------------|-------------------|---------|-----------|
| **Geolocation** | `allowGeolocation` | `false` | Location-based analytics, mapping |
| **Camera** | `allowCamera` | `false` | Photo capture, video recording, QR scanning |
| **Microphone** | `allowMic` | `false` | Voice input, audio recording |
| **Bluetooth** | `allowBluetooth` | `false` | IoT device integration, beacons |
| **NFC** | `allowNFC` | `false` | Contactless cards, NFC tags |

### Configuring manifest.json

To enable peripheral access, add the corresponding boolean properties to your `manifest.json`. Set each property to `true` to request that permission:

```json
{
  "name": "My Custom App",
  "version": "1.0.0",
  "description": "A Domo app with peripheral access",
  "fullpage": true,
  "size": {
    "width": 3,
    "height": 3
  },
  "allowGeolocation": true,
  "allowCamera": true,
  "allowMic": true,
  "datasetsMapping": [],
  "accountMapping": [],
  "actionMapping": [],
  "collectionsMapping": []
}
```

### Complete manifest.json Reference

```json
{
  "name": "Your App Name",
  "version": "1.0.0",
  "id": "unique-app-id",
  "description": "Description of your app",
  "fullpage": true,
  "draft": false,
  "size": {
    "width": 3,
    "height": 3
  },
  "allowGeolocation": true,
  "allowCamera": true,
  "allowMic": true,
  "allowBluetooth": true,
  "allowNFC": true,
  "datasetsMapping": [
    {
      "alias": "sales-data",
      "dataSetId": "your-dataset-id"
    }
  ],
  "accountMapping": [],
  "actionMapping": [],
  "collectionsMapping": []
}
```

### Peripheral Properties Summary

| Property | Type | Description |
|----------|------|-------------|
| `allowGeolocation` | `boolean` | Enable access to `navigator.geolocation` API |
| `allowCamera` | `boolean` | Enable access to camera via `navigator.mediaDevices.getUserMedia()` |
| `allowMic` | `boolean` | Enable access to microphone via `navigator.mediaDevices.getUserMedia()` |
| `allowBluetooth` | `boolean` | Enable access to `navigator.bluetooth` API |
| `allowNFC` | `boolean` | Enable access to Web NFC API |

### Using Peripherals in Your App

Once configured in the manifest, you can use standard browser APIs:

**Geolocation Example:**
```javascript
if ('geolocation' in navigator) {
  navigator.geolocation.getCurrentPosition(
    (position) => {
      console.log('Latitude:', position.coords.latitude);
      console.log('Longitude:', position.coords.longitude);
    },
    (error) => {
      console.error('Geolocation error:', error.message);
    }
  );
}
```

**Camera Example:**
```javascript
async function startCamera() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: false
    });

    const videoElement = document.getElementById('camera-preview');
    videoElement.srcObject = stream;
  } catch (error) {
    console.error('Camera access denied:', error);
  }
}
```

**Bluetooth Example:**
```javascript
async function connectBluetooth() {
  try {
    const device = await navigator.bluetooth.requestDevice({
      filters: [{ services: ['battery_service'] }]
    });

    console.log('Connected to:', device.name);
  } catch (error) {
    console.error('Bluetooth error:', error);
  }
}
```

### Security Considerations

- Permissions are enforced at both the iframe sandbox level and via backend Permissions-Policy headers
- Users will still see browser permission prompts for sensitive features
- Permissions must be explicitly requested in the manifest
- The backend validates manifest permissions before granting iframe access

### Iframe Sandbox Configuration

Your app runs in a sandboxed iframe with the following permissions enabled:

| Permission | Description |
|------------|-------------|
| `allow-scripts` | JavaScript execution |
| `allow-same-origin` | Access to same-origin resources |
| `allow-forms` | Form submission |
| `allow-downloads` | File downloads |

---

## Message Protocol Reference

### Outgoing Events (Domo to Your App)

#### `dataUpdated`
Sent when a dataset your app subscribes to has been updated.

```typescript
interface DataUpdatedMessage {
  event: 'dataUpdated';
  alias: string;          // Dataset alias from your mapping
  requestId?: string;     // Include in ack response
}
```

#### `filtersUpdated`
Sent when page-level filters change.

```typescript
interface FiltersUpdatedMessage {
  event: 'filtersUpdated';
  aliases: string[];      // Affected dataset aliases
  filters: Filter[];      // Array of active filters
  requestId?: string;
}

interface Filter {
  dataSourceId: string;
  column: string;
  values: string[];
  operand?: string;
}
```

#### `variablesUpdated`
Sent when page-level variables change.

```typescript
interface VariablesUpdatedMessage {
  event: 'variablesUpdated';
  variables: Variable[];
  requestId?: string;
}

interface Variable {
  functionId: number;
  value: string | number;
}
```

#### `appData`
Custom data passed from the Domo platform.

```typescript
interface AppDataMessage {
  event: 'appData';
  appData: string;        // JSON-encoded custom data
  requestId?: string;
}
```

### Incoming Events (Your App to Domo)

#### `ack`
Acknowledge receipt of a message.

```typescript
interface AckMessage {
  event: 'ack';
  requestId: string;      // Must match the received requestId
}
```

#### `navigate`
Request navigation to a URL.

```typescript
interface NavigateMessage {
  event: 'navigate';
  url: string;
  isNewWindow?: boolean;  // Open in new tab/window
}
```

#### `filter`
Update filters from your app.

```typescript
interface FilterMessage {
  event: 'filter';
  filter: Filter[];
  pageStateUpdate?: boolean;
}
```

#### `variable`
Update variables from your app.

```typescript
interface VariableMessage {
  event: 'variable';
  variables: Variable[];
}
```

#### `appData`
Send custom data to Domo.

```typescript
interface AppDataMessage {
  event: 'appData';
  appData: string;        // JSON-encoded custom data
}
```

#### `error`
Report an error to Domo.

```typescript
interface ErrorMessage {
  event: 'error';
  error: {
    code: string;
    message: string;
    details?: any;
  };
}
```

---

## Implementation Guide

### Recommended: Using ryuu.js

We recommend using the official Domo client library **ryuu.js** which handles all the MessageChannel communication, acknowledgments, and data fetching for you:

```bash
npm install ryuu.js
```

**Basic Usage with ryuu.js:**

```javascript
import domo from 'ryuu.js';

// Fetch data from your mapped datasets
const salesData = await domo.get('/data/v1/sales-data');

// Listen for filter changes
domo.onFiltersUpdate((filters) => {
  console.log('Filters changed:', filters);
  refreshVisualization(filters);
});

// Navigate to a Domo page
domo.navigate('/page/123456');

// Get environment info
const env = domo.env;
console.log('User:', env.userId, env.userName);
```

For full documentation, see: [https://www.npmjs.com/package/ryuu.js](https://www.npmjs.com/package/ryuu.js)

---

### Custom Implementation (Advanced)

If you need more control or want to implement your own communication layer, here's a basic class that correctly handles the MessageChannel API:

```javascript
class DomoMessageChannel {
  constructor() {
    this.port = null;
    this.handlers = new Map();

    // Listen for the initial MessagePort transfer from Domo
    window.addEventListener('message', (event) => {
      if (event.ports && event.ports.length > 0) {
        this.port = event.ports[0];
        this.port.onmessage = (e) => this.onMessage(e.data);
      }
    });
  }

  onMessage(message) {
    const { event, requestId } = message;

    // Dispatch to registered handler
    const handler = this.handlers.get(event);
    if (handler) {
      handler(message);
    }

    // Send acknowledgment back to Domo
    if (requestId && this.port) {
      this.port.postMessage({ event: 'ack', requestId });
    }
  }

  send(event, data = {}) {
    if (this.port) {
      this.port.postMessage({ event, ...data });
    }
  }

  on(event, handler) {
    this.handlers.set(event, handler);
  }
}
```

**Usage:**

```javascript
const channel = new DomoMessageChannel();

channel.on('dataUpdated', (msg) => console.log('Dataset updated:', msg.alias));
channel.on('filtersUpdated', (msg) => applyFilters(msg.filters));
channel.on('variablesUpdated', (msg) => updateVariables(msg.variables));

channel.send('navigate', { url: '/page/123', isNewWindow: false });
channel.send('filter', { filter: myFilters });
```

---

## Migration from v1

### Key Changes

| Aspect | v1 | v2 |
|--------|-----|-----|
| Message receiving | `window.addEventListener('message')` | `port.onmessage` |
| Message sending | `parent.postMessage()` | `port.postMessage()` |
| Acknowledgment | Not required | Bidirectional (both directions) |
| Peripheral config | Not supported | Boolean properties in manifest.json |

### Migration Steps

1. **Update message listener** to capture the MessagePort:
   ```javascript
   // Add this before your existing message handler
   window.addEventListener('message', (event) => {
     if (event.ports && event.ports.length > 0) {
       window.domoPort = event.ports[0];
       window.domoPort.onmessage = handleMessage;
     }
   });
   ```

2. **Add acknowledgments** to your message handler:
   ```javascript
   function handleMessage(event) {
     const message = event.data;

     // Your existing logic here...

     // NEW: Send acknowledgment
     if (message.requestId && window.domoPort) {
       window.domoPort.postMessage({
         event: 'ack',
         requestId: message.requestId
       });
     }
   }
   ```

3. **Update sending** to use the port:
   ```javascript
   // Before (v1)
   parent.postMessage({ event: 'filter', filter: filters }, '*');

   // After (v2)
   if (window.domoPort) {
     window.domoPort.postMessage({ event: 'filter', filter: filters });
   }
   ```

4. **Add peripheral permissions** to manifest.json if needed:
   ```json
   {
     "allowGeolocation": true,
     "allowCamera": true,
     "allowMic": true
   }
   ```

5. **Handle incoming acknowledgments** from Domo when your app sends messages:
   ```javascript
   // When sending messages that need confirmation
   const requestId = generateRequestId();
   window.domoPort.postMessage({
     event: 'filter',
     filter: filters,
     requestId: requestId
   });

   // Listen for Domo's ack
   // Domo will send back { event: 'ack', requestId: 'your-request-id' }
   ```

### Backward Compatibility

The v2 system maintains backward compatibility:
- If your app doesn't capture the MessagePort, Domo falls back to `window.postMessage()`
- Acknowledgments are optional (but recommended for reliability)
- Existing message formats are unchanged

---

## Appendix: Full Message Type Definitions

```typescript
// Message base types
interface MessageBase {
  requestId?: string;
  timestamp?: number;
}

// Outgoing (Domo to App)
type OutgoingEvent =
  | 'dataUpdated'
  | 'filtersUpdated'
  | 'variablesUpdated'
  | 'appData'
  | 'ack';

// Incoming (App to Domo)
type IncomingEvent =
  | 'navigate'
  | 'filter'
  | 'variable'
  | 'appData'
  | 'ack'
  | 'error'
  | 'subscribe';

// Filter structure
interface Filter {
  dataSourceId: string;
  column: string;
  values: string[];
  operand?: 'IN' | 'NOT_IN' | 'EQUALS' | 'NOT_EQUALS';
}

// Variable structure
interface Variable {
  functionId: number;
  value: string | number;
}
```