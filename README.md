<p align="center">
  <img src="radium-banner.png" alt="Radium Banner" width="100%">
</p>

# ⚡ Radium Deployment & OTA Hot-Updater Manual

This document provides clear instructions on how to install and compile the Radium Electron client on your friends' machines (both Windows and macOS), how the OTA (Over-The-Air) Hot-Updater behaves, and what configurations are required.

---

## 🛠️ Part 1: Compiling standalone apps (.exe & .dmg) for your friends

Both environment setup scripts ([setup_environment.ps1](file:///C:/Users/Ruhaa/OneDrive/Desktop/Radium/setup_environment.ps1)) and ([setup_environment.sh](file:///C:/Users/Ruhaa/OneDrive/Desktop/Radium/setup_environment.sh)) have been fully updated. They automatically install and configure all required build-time dependencies, run `npm install`, compile the optimized React assets, and package the final installer.

### 💻 Compiling on a Friend's Windows PC (to get `.exe`)
1. Copy the Radium repository folder to their PC.
2. Search for **PowerShell** in the Start Menu, right-click it, and select **Run as Administrator**.
3. Navigate to the Radium directory:
   ```powershell
   cd "C:\Path\To\Radium"
   ```
4. Run the setup script to check dependencies, build the assets, and package the executable:
   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force; .\setup_environment.ps1
   ```
5. **Output:** The script will package a premium installer wizard `.exe` inside the **`dist/`** directory.
   * **Wizard controls:** We configured NSIS in `package.json` to allow your friends to select their installation folder (`oneClick: false`, `allowToChangeInstallationDirectory: true`) and create desktop/start-menu shortcuts automatically.
   * **Cleanup:** Once they run the compiled `.exe` to install the app on their system, **they can safely delete the installer file** to keep their folders clean.

### 🍏 Compiling on a Friend's Mac (to get `.dmg`)
1. Copy the Radium folder to their Mac.
2. Open the **Terminal** app.
3. Navigate to the Radium directory:
   ```bash
   cd /Path/To/Radium
   ```
4. Grant execution permissions and run the setup script:
   ```bash
   chmod +x setup_environment.sh && ./setup_environment.sh
   ```
5. **Output:** The script will build a standard macOS package inside the **`dist/`** directory.
   * **Drag-to-Applications:** We configured the DMG layout inside `package.json` to show the standard visual window directing users to drag the Radium icon directly into `/Applications`.
   * **Cleanup:** Once they drag the app to Applications, **they can safely delete the DMG installer archive**.

---

## 🐍 Part 2: Dependencies FAQ

### Do my friends need Node, Git, or Python to run the app?
* **No!** The compiled `.exe` (Windows) and `.app`/`.dmg` (macOS) are **100% self-contained standalone binaries**. Your friends do NOT need Node, Git, or Python installed on their computers to open and use Radium.
* **Why did the setup scripts install Python?** Python and C++ compilers are only required *at build-time* during the `npm install` phase so that native packages (like `ssh2`) can build their binaries on their machine.

### Do we need the old config files?
* **No local config files are needed for your friends.** The client app has the connection details to the VPS built directly into the codebase.
* The only configurations needed are the `radium.config.json` and the Google Sheets API credentials (`sheets-creds.json`) on the **remote VPS server** so it can act as the database bridge.

---

## 🚀 Part 3: Deploying OTA Updates (Without Re-downloads)

We built an **OTA (Over-The-Air) Hot-Updater** so you can update features (styles, lists, screens, menus) on all your friends' computers instantly.

### How to push an update (Crusty's Machine):
1. Increment the `"version"` field inside [package.json](file:///C:/Users/Ruhaa/OneDrive/Desktop/Radium/package.json) (e.g. bump it from `1.0.0` to `1.0.1`).
2. Run the local publishing script:
   ```bash
   node scripts/publish_update.cjs
   ```
3. **What this does:** It automatically runs `npm run build`, archives `dist/` and `electron/` assets, securely uploads the package directly to your VPS, and publishes the new version manifest.

### How it updates on their PC (Automatic):
1. Every time a friend opens their Radium app, it queries the VPS on boot:
   `http://195.114.193.129:8080/api/update/check`
2. If the VPS has a newer version than their local app:
   * It downloads the latest `update.zip` from the VPS.
   * It extracts the contents directly into the local installation folder, replacing old frontend/backend code.
   * It relaunchs the app automatically (`app.relaunch(); app.exit(0);`).
   * **Cleanup:** The app **automatically deletes the temporary patch zip** once extracted, keeping their disk clean.
3. If they are offline or the server connection is slow, the boot check terminates after a **`3.5-second` timeout** so the app opens instantly.
