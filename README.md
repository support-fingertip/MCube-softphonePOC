# MCube Softphone POC — Salesforce Open CTI ↔ MCube Telephony

A **Proof-of-Concept** that wires MCube's telephony platform into Salesforce via the
[Open CTI](https://developer.salesforce.com/docs/atlas.en-us.api_cti.meta/api_cti/sforce_api_cti_intro.htm)
framework. The entire POC can be tested locally using **two browser tabs** — no server or
Salesforce org required.

---

## 📁 File Structure

| Path | Purpose |
|------|---------|
| `mock/mock_event_trigger.html` | Simulates the Fingertip/MCube platform — broadcasts telephony events via `BroadcastChannel` |
| `mock/mock_mcube_softphone.html` | Mock MCube softphone UI — embedded as an `<iframe>` inside the bridge |
| `bridge/mcube_bridge.html` | **CTI Adapter URL** — bridges the iframe softphone with Salesforce Open CTI |
| `callcenter/MCubeCallCenter.xml` | Salesforce Call Center XML definition (import in Setup) |

---

## 🔄 Event Flow

```
[Tab 2: mock_event_trigger.html]
        │ BroadcastChannel (mcube_channel)
        ▼
[mock_mcube_softphone.html] (iframe)
        │ window.parent.postMessage()
        ▼
[bridge/mcube_bridge.html]
        │ sforce.opencti.*
        ▼
[Salesforce Lightning - Screen Pop / Task Log]
```

### Supported Events

| Trigger | Softphone → Bridge | Open CTI Action |
|---------|--------------------|-----------------|
| Incoming call | `INCOMING_CALL` | `searchAndScreenPop` |
| Agent answers | `CALL_ACCEPTED` | `setSoftphonePanelLabel` |
| Agent rejects | `CALL_REJECTED` | `saveLog` (Task) |
| Call ends | `CALL_ENDED` | `saveLog` + `refreshView` |
| Status change | `AGENT_STATUS_CHANGED` | `setSoftphonePanelLabel` |
| Click-to-dial | Salesforce UI → bridge → iframe | `enableClickToDial` + `onClickToDial` |

---

## 🧪 How to Test Locally (Two Browser Tabs)

> No server or Salesforce org required — everything runs from the filesystem.

1. **Open Tab 1** — navigate to `bridge/mcube_bridge.html`
   - You'll see the embedded mock softphone and a debug log panel.
   - The header will show **"⚠ POC Mode (no SF)"** because Open CTI isn't available outside Salesforce. All Open CTI calls will be visible in the debug log instead.

2. **Open Tab 2** — navigate to `mock/mock_event_trigger.html`
   - Enter a phone number (default: `9876543210`).
   - Click **"🔔 Trigger Incoming Call"**.

3. **Switch back to Tab 1** — the iframe softphone should display the ringing UI.
   - Click **"✅ Answer"** or **"❌ Reject"** and watch the debug log update.

4. **Trigger more events** from Tab 2 (Call Ended, Agent Busy, etc.) and observe the responses.

---

## 🚀 How to Deploy to a Salesforce Dev Org

1. **Zip only the bridge and mock directories** (the Call Center XML is imported separately)
   ```bash
   zip -r MCubeBridge.zip bridge/ mock/
   ```

2. **Upload as a Static Resource**
   - Salesforce Setup → Static Resources → New
   - Name: `MCubeBridge`, Cache Control: `Public`
   - Upload `MCubeBridge.zip`

3. **Import the Call Center** (separate step — not part of the Static Resource)
   - Salesforce Setup → Call Centers → Import
   - Select `callcenter/MCubeCallCenter.xml`
   - After import, edit the **CTI Adapter URL** field to point to your Static Resource, e.g.:
     ```
     https://mycompany.lightning.force.com/resource/MCubeBridge/bridge/mcube_bridge.html
     ```
     Replace `mycompany` with your org's actual Lightning subdomain.

4. **Assign Users**
   - Open the Call Center record → Manage Call Center Users → Add Users

5. **Open the Softphone**
   - In Salesforce Lightning, click the phone icon in the utility bar.
   - The bridge loads, Open CTI initialises, and click-to-dial is enabled.

---

## 🔴 How to Go Live with Real MCube

Replace the iframe `src` in `bridge/mcube_bridge.html`:

```html
<!-- Before (mock) -->
<iframe id="mcube-frame" src="../mock/mock_mcube_softphone.html" ...>

<!-- After (live) -->
<!-- 🔴 LIVE: Replace iframe src with MCube's real hosted softphone URL -->
<iframe id="mcube-frame" src="https://softphone.mcube.in/agent?key=YOUR_KEY" ...>
```

Ensure MCube's hosted softphone:
- Fires `window.parent.postMessage({ event: 'INCOMING_CALL', ... }, '*')` for each event.
- Listens for `window.addEventListener('message', ...)` to receive `{ action: 'MAKE_CALL', number }`.

---

## 🌐 BroadcastChannel Browser Compatibility

`BroadcastChannel` is used **only for local POC testing** (`mock_event_trigger.html` → `mock_mcube_softphone.html`). It is not used in production.

| Browser | Support |
|---------|---------|
| Chrome / Edge 54+ | ✅ Supported |
| Firefox 38+ | ✅ Supported |
| Safari 15.4+ | ✅ Supported |
| Safari < 15.4 | ❌ Not supported |
| Internet Explorer | ❌ Not supported |

---

## ⚙️ Key Technical Details

- **BroadcastChannel name**: `mcube_channel`
- **Primary colour**: `#1a73e8`
- All `sforce.opencti.*` calls are guarded with `typeof sforce !== 'undefined'` so the bridge works outside Salesforce.
- The debug log panel in `mcube_bridge.html` shows all received events and simulated Open CTI calls, making POC testing straightforward.
=======
# my-sfdc-project

Salesforce DX Project — API Version 62.0

## Quick Start

```bash
# Authorize your org
sf org login web --alias myorg --set-default

# Deploy to org
sf project deploy start --source-dir force-app --target-org myorg

# Run tests
sf apex run test --target-org myorg --test-level RunLocalTests

# Retrieve from org
sf project retrieve start --source-dir force-app --target-org myorg
```

## Project Structure

```
force-app/main/default/
├── classes/          ← Apex classes
├── triggers/         ← Apex triggers
├── lwc/              ← Lightning Web Components
├── aura/             ← Aura components
├── objects/          ← Custom objects & fields
├── layouts/          ← Page layouts
├── flexipages/       ← Lightning pages
├── flows/            ← Flows
├── permissionsets/   ← Permission sets
└── staticresources/  ← Static resources

manifest/
├── package.xml               ← Retrieve manifest
└── destructiveChanges.xml    ← Deletion manifest

config/
└── project-scratch-def.json  ← Scratch org config
```

## CI/CD

This project uses GitHub Actions for automated deployments.
See `.github/workflows/` for pipeline configuration.

