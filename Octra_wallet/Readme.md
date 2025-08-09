### Octra Wallet — Extension Developer Guide

This document focuses on developing and maintaining the contents of the `extension/` folder.

For the product overview, permissions, security notes, disclaimer, and known issues, see the root `README.md`.

### How it works
- **UI**: `popup.html` with modular JS in `js/` and styles in `css/`
- **Background**: `background.js` service worker handles lifecycle and network proxying
- **Crypto**: Uses `libs/nacl.min.js` for cryptographic operations
- **Config**: Centralized in `js/config.js` (RPC, endpoints, UI, security)

### Permissions & Security
See root `README.md` for the canonical list of permissions and security guidance.

### Install (Developer Mode)
1. Open Chrome and go to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked" and select the `Octra_wallet/` folder
4. The popup is available from the extensions toolbar (Octra Wallet)

### Project structure
- `manifest.json`: Extension manifest
- `background.js`: Service worker
- `popup.html` / `popup.js`: Popup UI entry
- `js/`: Core logic (wallet, storage, crypto, network, UI)
- `css/`: Styles for popup and dialogs
- `libs/nacl.min.js`: Cryptographic library
- `icons/`: Extension icons

### Development tips
- Enable console logs in `js/config.js` via `DEV.ENABLE_CONSOLE_LOGS`
- Most work happens in `js/` and `popup.html` (screens are toggled via classes)
- Background networking can be extended in `background.js` (`NETWORK_REQUEST` handler)

### Packaging
- Zip the contents of the `extension/` folder for manual distribution, or use the Chrome Web Store flow
- Exclude local artifacts (see `.gitignore`)

### Known issues
See root `README.md` for the canonical list of known issues.
