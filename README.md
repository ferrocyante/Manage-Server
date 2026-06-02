# Manage-Server • Switch Tabs - Companion Repo

This repo contains the components required to make the [Switch Tabs Raycast extension](https://github.com/ferrocyante/Switch-Tabs) work:

- **`raycast-browser-bridge/`** — the browser extension (Manifest V3) that connects to the server and controls your tabs

> **Browser Extension on Edge Add-ons Store:** [Switch Tabs Bridge](https://microsoftedge.microsoft.com/addons/detail/kpgdjpohjiaaikeohphffiaoepfmnaff)

> **The bridge server (`raycast-bridge-server.exe`) is downloaded automatically** by the Raycast extension on first use. You do not need to download or configure it manually.

---

## How It Works

The server (`raycast-bridge-server.exe`) is a standalone background process that runs independently of any browser or Raycast session. It listens on port `19222` and acts as a relay:

- **Browser extension → Server:** sends tab data, media state, bookmarks, downloads, history over WebSocket
- **Raycast → Server:** sends commands (activate tab, close tab, create group, etc.) and receives tab state
- **Server → both:** routes messages between them, maintains sticky state so Raycast gets data instantly on open

The server runs until you explicitly stop it. It does not shut down when you close your browser or Raycast.

---

## Full Setup Guide

Follow every step in order.

---

### Step 1 — Install the Browser Extension

**Option A — Edge Add-ons Store (easiest, works on Edge)**

1. Open Microsoft Edge.
2. Go to the [Switch Tabs Bridge store page](https://microsoftedge.microsoft.com/addons/detail/kpgdjpohjiaaikeohphffiaoepfmnaff).
3. Click **Get** → **Add Extension**.
4. Done.

**Option B — Load unpacked (Chrome, Brave, or any Chromium browser)**

1. Download and unzip `raycast-browser-bridge.zip` from the latest release.
2. Open your browser and go to `chrome://extensions`.
3. Enable **Developer mode** (toggle in the top-right corner).
4. Click **Load unpacked**.
5. Select the `raycast-browser-bridge` folder.
6. The extension will appear in your list.

> **Helium / Ungoogled Chromium users:** After loading the extension, open the background DevTools (click **Service Worker** under the extension) and run this once in the Console:
> ```js
> chrome.storage.local.set({ browserOverride: "helium" })
> ```
> This tells the extension to identify itself as Helium to the server.

---

### Step 2 — Open Manage Server

Open Raycast → type **Manage Server** → press Enter.

On first open, the extension will automatically download the bridge server and required scripts. A progress screen will appear while this happens — it only takes a few seconds and only happens once.

Once setup is complete you'll see the **Bridge Server** section showing the current server status.

---

### Step 3 — Start the Server

- If the server is **Stopped**, press Enter on **Server Status** and select **Start Server**.
- The server will start in the background and the status will turn green.

**Optional — Start at Login**

To have the server start automatically every time Windows boots:

1. Open **Manage Server** in Raycast.
2. Select **Start at Login** → press Enter → select **Enable Login Startup**.

This writes a Windows Registry run key so the server launches silently at login without any browser or Raycast needing to be open first.

---

### Step 4 — Verify the Connection

1. Open Raycast → type **Switch Tabs** → press Enter.
2. Your open tabs should appear within 1–2 seconds.
3. The search bar placeholder will show the connected browser name (e.g. `Filter Tabs | Edge`).

If no tabs appear, check **Manage Server** — the server status and browser count are shown there.

---

## Bridge Control Panel

The browser extension has a settings popup (click the extension icon) with the following options:

### Popup Window Size
Set the width and height for popup windows created via Focus Mode or search. Choose from presets (Super Compact, Classic Card, Wide Banner, Large Premium) or enter custom dimensions. Popup windows always open centered on your screen.

### OS Focus & Desktop Switch
Fine-tune the window focus bounce delays for reliable virtual desktop switching on Windows:
- **Refocus Bounce** — delay between minimize and restore (default 10ms)
- **Minimized Wait** — pause before restoring a minimized window (default 100ms)
- **Focus Reinforce** — extra focus call after restore (default 50ms)

### Sync Limits
- **History Items** — max history entries synced to Raycast (default 100)
- **Recent Closed** — max recently closed sessions synced (default 25)

### New Tab Behavior
- **Normal** — allow all new tabs
- **Keep One** — maximum one new tab open at a time
- **Bombardment** — no new tabs survive (all closed immediately)

### Deep Scan Whitelist
Domains added here will have all frames scanned for tab content (useful for sites with iframes like Netflix). Add the current site with one click or enter a domain manually.

---

<details>    
<summary><h3>🟢 SET UP EDGE WORKSPACES</h3></summary>

---

If you use Edge Workspaces, Switch Tabs can display your workspace names next to each window and let you browse them from inside Raycast. Here's how to get them synced.

**How it works**

Edge stores workspace metadata in its sync system. The extension reads this via the native messaging pipe and maps each browser window to its workspace name automatically.

**1. Enable Edge Sync**

Open Edge and go to `edge://sync`. Go to the data tab then unselect all ticks.

<img width="951" height="548" alt="image" src="https://github.com/user-attachments/assets/3f73353b-aa0e-43f6-931c-f7f5b6b5c615" />

Then tick only Edge Workspace and make sure you also tick the nodes content tick as shown below. Then click **Dump Sync Node to File**. Your CSV file will be downloaded.

<img width="761" height="959" alt="image" src="https://github.com/user-attachments/assets/f033f98f-636d-4ea0-850e-712436a81e5f" />

**2. Run Import Workspaces**

Open Raycast → type **Manage Server** → press Enter → select **Import Edge Workspaces** → press Enter.

This runs `ImportWorkspaces.ps1` which will ask you for the workspace CSV file. Select the CSV file and proceed with the instructions in the terminal window.

The script writes the workspace roster to `workspaces_roster.json` in the server's support folder. The server picks this up automatically.

**3. Verify**

Open **Switch Tabs** in Raycast. Windows that belong to a workspace will show the workspace name in the window filter dropdown and next to each tab. If names don't appear, try reloading the browser extension and reopening Switch Tabs.

**Notes**

- You only need to run Import Workspaces once, or again if you create new workspaces.
- If you rename a workspace in Edge, re-run Import Workspaces to pick up the new name.

</details>

---

## Manage Server Command

The **Manage Server** command gives you full control over the bridge server:

| Item | What it does |
|------|-------------|
| **Server Status** | Shows Running/Stopped, number of connected browsers, and uptime. Actions: Start, Stop, Kill, Refresh |
| **Start at Login** | Enables/disables automatic server startup at Windows login via Registry run key |
| **Watch Bridge Logs** | Opens a terminal window with live color-coded WebSocket logs |
| **Import Edge Workspaces** | Imports Edge workspace tab configurations |

---

## Troubleshooting

### Switch Tabs shows "Bridge Server Not Running"

The server is not running. Open **Manage Server** and start it, or enable **Start at Login** so it starts automatically.

### Switch Tabs shows "No Tabs Found" / "Your browser is out of reach of the server"

The server is running but no browser is connected. Make sure your browser is open and the extension is installed and enabled.

### "Connecting to Extension…" never resolves

1. Check **Manage Server** — confirm the server is running and shows at least 1 browser connected.
2. Click the extension icon in your browser toolbar — the popup shows connection status.
3. Open **Watch Bridge Logs** in Manage Server to see live connection activity.

### Tabs from a specific browser are not showing

- Make sure the browser extension is installed and enabled in that browser.
- Click the extension popup — check the detected browser type. If it's wrong (shows "browser" instead of "edge" etc.), use the browser override:
  ```js
  chrome.storage.local.set({ browserOverride: "edge" }) // or chrome, brave, helium
  ```

### Download failed on first open

- Check your internet connection and reopen **Manage Server** — it will retry automatically.
- If the issue persists, check that GitHub is accessible from your network.

### PowerShell execution policy error

Run this once in an elevated PowerShell window:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

---

## Supported Browsers

| Browser | Notes |
|---------|-------|
| Microsoft Edge | Store install available |
| Google Chrome | Load unpacked |
| Brave | Load unpacked |
| Helium / Ungoogled Chromium | Load unpacked, requires manual browser override |

---

<details>
<summary>Repo Structure</summary>

```
companion-repo/
└── raycast-browser-bridge/        ← browser extension source
    ├── manifest.json
    ├── background.js              # service worker — message router + WebSocket client
    ├── connection.js              # WebSocket connect / reconnect / backoff logic
    ├── handlers.js                # all tab/group/bookmark/download action handlers
    ├── keepalive.js               # MV3 service worker keepalive port
    ├── media.js                   # media polling + injection
    ├── state.js                   # shared extension state
    ├── stateSync.js               # broadcast helpers (tabs, bookmarks, history…)
    ├── utils.js                   # shared utilities
    ├── popup.html                 # extension popup UI
    ├── popup.js                   # popup logic (status, browser override)
    └── icon.png
```

> The bridge server files (`raycast-bridge-server.exe`, `watch-logs.ps1`, `ImportWorkspaces.ps1`) are hosted as release assets and downloaded automatically by the Raycast extension on first use.

</details>

---

## License

MIT
