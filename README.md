# Manage-Server • Switch Tabs - Companion Repo

This repo contains the two components required to make the [Switch Tabs Raycast extension](https://github.com/ferrocyante/Switch-Tabs) work:

- **`server/`** — the local bridge server (`.exe`) that relays messages between Raycast and your browser
- **`raycast-browser-bridge/`** — the browser extension (Manifest V3) that connects to the server and controls your tabs

> **Browser Extension on Edge Add-ons Store:** [Switch Tabs Bridge](https://microsoftedge.microsoft.com/addons/detail/hphaioiiggmjhiocafgdbbeglcfljkjm)

---

## Full Setup Guide

Follow every step in order.

---

### Step 1 — Install the Browser Extension

**Option A — Edge Add-ons Store (easiest, works on Edge)**

1. Open Microsoft Edge.
2. Go to the [Switch Tabs Bridge store page](https://microsoftedge.microsoft.com/addons/detail/hphaioiiggmjhiocafgdbbeglcfljkjm).
3. Click **Get** → **Add Extension**.
4. Done. The default Extension ID in the Raycast extension preferences is already set for the store version — you don't need to change anything.

**Option B — Load unpacked (Chrome, Brave, or any Chromium browser)**

1. Download and unzip the package folder from the latest release.
2. Open your browser and go to `chrome://extensions`.
3. Enable **Developer mode** (toggle in the top-right corner).
4. Click **Load unpacked**.
5. Select the `raycast-browser-bridge` folder from inside the package folder.
6. The extension will appear in your list. **Copy the Extension ID** shown under its name — you will need to paste it into the Raycast extension preferences in Step 3.

> **Helium / Ungoogled Chromium users:** Helium is sometimes misdetected as Chrome because it strips Google-specific APIs that the extension uses for browser identification. After loading the extension, you need to manually set the browser type once:
>
> 1. Go to `chrome://extensions` and click **Service Worker** under the Switch Tabs Bridge extension to open the background DevTools.
> 2. In the Console tab, paste and run:
>    ```js
>    chrome.storage.local.set({ browserOverride: "helium" })
>    ```
> 3. Close DevTools. The extension will now correctly identify itself as Helium to the server and Raycast.

---

### Step 2 — Place the Server Folder

Put the `server` folder (inside the package folder) somewhere permanent on your machine, for example:

```
C:\Tools\switch-tabs-server\
```

> **Important:** Do not move this folder after registration. If you do, re-run Step 4.

---

### Step 3 — Configure the Raycast Extension

Open Raycast → search **Switch Tabs** → press `Ctrl + ,` to open Extension Preferences.

| Preference | What to set |
|------------|-------------|
| **Server Directory Path** | Full path to your `server` folder (e.g. `C:\Tools\server`) |
| **Browser Extension ID** | **If you used Option A (store):** leave as the default — it's already set. **If you used Option B (unpacked):** paste the Extension ID you copied from `chrome://extensions` |

---

### Step 4 — Register the Native Host (one-time)

The bridge server communicates with the browser extension via Chrome's Native Messaging API. This step writes the required registry entries so your browser knows where the server lives.

**Option A — Via Raycast (recommended)**

1. Configure the **Server Directory Path** preference to point to your `server` folder (Step 3 above).
2. Open Raycast → type **Manage Server** → press Enter.
3. Select **Register Browser Bridge (Native Host)** → press Enter.
4. A PowerShell window will open, run the registration, and close automatically.
5. You'll see a success toast in Raycast when it's done.

**Option B — Via BAT file**

1. Navigate to `server\setup\`.
2. Right-click `Register-Bridge.bat`.

> **What this does:** Writes a native messaging manifest to the Windows registry under `HKCU\Software\Google\Chrome\NativeMessagingHosts\com.raycast.browser.bridge` (and equivalent keys for Edge, Brave, Chrome). This tells each browser where to find `raycast-bridge-server.exe` when the extension requests a native connection.

---

### Step 5 — Verify the Connection

1. Open Raycast → type **Switch Tabs** → press Enter.
2. Your open tabs should appear within 1–2 seconds. If they don't, reload your browser extension and reopen the **Switch Tabs** command in Raycast.
3. The search bar placeholder will show the connected browser name (e.g. `Filter Tabs | Edge`).

You can also click the extension icon in your browser toolbar — the popup shows the connection status and detected browser type.

---

### Step 6 — (Optional) Connect Additional Browsers

The native host registration in Step 4 covers all supported Chromium browsers at once. To add another browser:

1. Install the browser extension in that browser using **Option B (Load unpacked)** from Step 1.
2. No additional registration is needed — the server is already registered for all browsers.

---

<details>    
<summary> 🟢 SET UP EDGE WORKSPACES </summary>

---

If you use Edge Workspaces, Switch Tabs can display your workspace names next to each window and let you browse them from inside Raycast. Here's how to get them synced.

**How it works**

Edge stores workspace metadata in its sync system. The extension reads this via the native messaging pipe and maps each browser window to its workspace name automatically. 

**1. Enable Edge Sync**

Open Edge and go to `edge://sync`. go to data tab then unselect all ticks 

<img width="951" height="548" alt="image" src="https://github.com/user-attachments/assets/3f73353b-aa0e-43f6-931c-f7f5b6b5c615" />

then tick only Edge Workspace and make you also tick nodes content tick as shown below. then click on **Dump Sync Node to File**. you csv file will be downloaded.

![edge sync page](<img width="761" height="959" alt="image" src="https://github.com/user-attachments/assets/f033f98f-636d-4ea0-850e-712436a81e5f" />)

**2. Run Import Workspaces**

Open Raycast → type **Manage Server** → press Enter → select **Import Edge Workspaces** → press Enter.

This runs `ImportWorkspaces.ps1` which will ask you for workspace csv file . select the csv file and proceed with instruction in terminal window.

Edge sync data and writes the workspace roster to `workspaces_roster.json` in the server folder. The server picks this up automatically.

![import workspaces](https://github.com/user-attachments/assets/your-screenshot-here)

**3. Verify**

Open **Switch Tabs** in Raycast. Windows that belong to a workspace will show the workspace name in the window filter dropdown and next to each tab. If names don't appear, try reloading the browser extension and reopening Switch Tabs.

**Notes**

- You only need to run Import Workspaces once, or again if you create new workspaces.
- Workspace names sync via the native messaging pipe even when the WebSocket is temporarily offline.
- If you rename a workspace in Edge, re-run Import Workspaces to pick up the new name.

---
</details>

---

## Troubleshooting

### "Connecting to Extension…" never resolves

1. Make sure the browser extension is installed and enabled in your browser.
2. Click the extension icon in the toolbar — the popup should show **Connected**. If it shows **Disconnected**, the server is not running.
3. Open **Manage Server → Watch Bridge Logs** in Raycast to start the server and see live output.
4. If the server fails to start, re-run **Register Browser Bridge** (Step 4). The registration may have been lost if you moved the `server` folder.

### Tabs from a specific browser are not showing

- Make sure the browser extension is installed in that browser.
- Click the extension popup — check the detected browser type. If it's wrong, use the **Override** dropdown in the popup to set it manually.

### "Extension ID Required" error in Manage Server

- Open Extension Preferences in Raycast.
- Paste your Extension ID into the **Browser Extension ID** field.
- If you installed from the Edge Store, the default ID is already correct — no action needed.

### Server folder not found / missing scripts error

- Make sure the **Server Directory Path** preference points directly to the `server` folder — the one containing `register-bridge.ps1`, `watch-logs.ps1`, and `ImportWorkspaces.ps1`.
- Do not point it to the `server\setup` subfolder.

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
| Helium / Ungoogled Chromium | Load unpacked |

<details>
<summary>Repo Structure</summary>

```
companion-repo/
├── raycast-browser-bridge/        ← browser extension source
│   ├── manifest.json
│   ├── background.js              # service worker — message router + WebSocket client
│   ├── connection.js              # WebSocket connect / reconnect logic
│   ├── handlers.js                # all tab/group/bookmark/download action handlers
│   ├── keepalive.js               # MV3 service worker keepalive (prevents 30s timeout)
│   ├── media.js                   # media polling + injection
│   ├── state.js                   # shared extension state
│   ├── stateSync.js               # broadcast helpers (tabs, bookmarks, history…)
│   ├── utils.js                   # shared utilities
│   ├── popup.html                 # extension popup UI
│   ├── popup.js                   # popup logic (status, browser override)
│   └── icon.png
└── server/                        ← bridge server + setup scripts
    ├── raycast-bridge-server.exe  # the bridge server binary
    ├── register-bridge.ps1        # registers native host with all browsers (run once)
    ├── watch-logs.ps1             # opens a terminal with live WebSocket logs
    ├── ImportWorkspaces.ps1       # imports Edge workspace tab configs
    ├── setup-bridge.ps1           # alternative setup script
    ├── com.raycast.browser.bridge.json  # native messaging manifest
    ├── workspaces_metadata.json
    ├── workspaces_roster.json
    └── setup/
        ├── Register-Bridge.bat
        ├── Setup-Bridge.bat
        ├── Watch-Logs.bat
        └── import_workspaces.bat
```

</details>

---

## License

MIT
