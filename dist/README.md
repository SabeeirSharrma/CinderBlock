# CinderBlock Build Output

## Chromium

1. Download and unzip `cinderblock.chromium.zip` (from the [latest release](https://github.com/SabeeirSharrma/CinderBlock/releases)).
2. Rename the unzipped directory to `cinderblock`.
   - When you update manually, replace the **content** of the `cinderblock` folder with the **content** of the latest zipped version. This ensures all settings are preserved.
   - As long as the extension loads from the same folder path as it was originally installed, your settings will be kept.
3. Open Chromium/Chrome and go to *Extensions*.
4. Click to enable *Developer mode*.
5. Click *Load unpacked extension...*.
6. In the file selector dialog:
   - Select the `cinderblock` directory you created.
   - Click *Open*`.

The extension will now be available in your Chromium/Chromium-based browser.

**Note:** You must update manually. For some users, manual updates are beneficial because:
- You can update when **you** want.
- If a new version is unsatisfactory, you can easily reinstall the previous one.

## Firefox

Compatible with Firefox 52 and beyond.

### For Stable Release Version

This method only works if you set `xpinstall.signatures.required` to `false` in `about:config`. ([see "Add-on signing in Firefox"](https://support.mozilla.org/en-US/kb/add-on-signing-in-firefox))

1. Download `cinderblock.firefox.xpi` (from the [latest release](https://github.com/SabeeirSharrma/CinderBlock/releases)).
   - Right-click and choose _"Save As..."_.
2. Drag and drop the downloaded `cinderblock.firefox.xpi` into Firefox.

### For Beta Version

- Click on `cinderblock.firefox.signed.xpi` (from the [latest release](https://github.com/SabeeirSharrma/CinderBlock/releases)).

### Location of CinderBlock Settings

On Linux, the settings are saved in a JSON file located at:
```
~/.mozilla/firefox/[profile name]/browser-extension-data/cinderblock/storage.js
```
When you uninstall the extension, Firefox deletes this file, and all your settings will be lost.

## Build Instructions (for Developers)

1. Clone the CinderBlock repository:
   ```bash
   git clone https://github.com/SabeeirSharrma/CinderBlock.git
   ```
2. Navigate to the project:
   ```bash
   cd CinderBlock
   ```
3. Install dependencies:
   ```bash
   npm install
   ```
4. Build:
   - Chromium:
     ```bash
     make chromium
     ```
   - Firefox:
     ```bash
     make firefox
     ```
5. Load the result of the build into your browser:
   - **Chromium:**
     - Navigate to `chrome://extensions/`
     - Check _"Developer mode"_
     - Click _"Load unpacked"_
     - Select `CinderBlock/dist/build/cinderblock.chromium/`
   - **Firefox:**
     - Navigate to `about:debugging#/runtime/this-firefox`
     - Click _"Load Temporary Add-on..."_
     - Select `CinderBlock/dist/build/cinderblock.firefox/`
