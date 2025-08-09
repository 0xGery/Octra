### Octra Wallet — Browser Extension

This repository contains the Octra Wallet Chrome extension. It provides a lightweight, secure UI to create/import Octra wallets, view balances, send transactions, and use privacy features — all from the browser popup.

### Repository layout
- `extension/`: The extension source code (popup UI, background, logic, styles)
  - `manifest.json`: Extension manifest and permissions
  - `background.js`: Service worker (lifecycle, simple network proxying)
  - `popup.html` / `popup.js`: Popup UI entry
  - `js/`: Wallet, storage, crypto, UI, and network modules
  - `css/`: Popup and dialog styles
  - `libs/nacl.min.js`: Cryptographic library
  - `icons/`: Extension icons
- `LICENSE`: Project license
- `Octra_wallet.crx`: Built extension package (for manual install/distribution)

Note: Build artifacts like `*.crx`, `*.pem`, and `*.zip` are ignored by `.gitignore` by default. If you intentionally keep a packaged artifact here (like `Octra_wallet.crx`), that is fine and expected for distribution.

### Features
- **Wallet management**: Create or import (base64 private key + address)
- **Balances**: Public and private balances
- **Transactions**: Send tokens and view history
- **Privacy**: Encrypt/decrypt balances, private send, claim private transfers
- **Bulk send**: Add multiple recipients and send in one action
- **Smart contract tasks**: Built-in OCS01 testing task

### Install (Developer Mode)
1. Open Chrome and go to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked" and select the `extension/` folder
4. Pin the extension from the toolbar and open the popup

Alternatively, if you have `Octra_wallet.crx`, you can drag-and-drop it into `chrome://extensions/` (depending on Chrome version and policies).

### Permissions (see `extension/manifest.json`)
- `storage`, `alarms`
- Host permissions: `https://octra.network/*`, `https://*.octra.network/*`

### Security
- Wallet and settings stored via `chrome.storage.local`
- Optional password protection and auto-lock
- Background error handling for runtime exceptions and unhandled rejections
- Never share your private key or recovery phrase

### Disclaimer
This is an unofficial, community-built wallet for the Octra network. It is not affiliated with, endorsed by, or maintained by the official Octra team. Use at your own risk and always verify URLs and addresses before interacting. No warranties are provided. Keep your private key and recovery phrase secure at all times.

### Development
- Most UI logic is in `extension/popup.html` and `extension/js/`
- Configuration is centralized in `extension/js/config.js`
- Background message types include `GET_EXTENSION_INFO` and `NETWORK_REQUEST` in `extension/background.js`

### Packaging
- Zip or package the contents of `extension/` to produce a `.crx`
- Keep your private key (`.pem`) safe; do not commit it — it is ignored by `.gitignore`

### Known issues
- Wallet generator: minor key derivation/derivative issue being investigated
- UI duplication when switching wallets: low priority (frontend-only); functionality is unaffected

### License
See `LICENSE` for licensing details.


