### Octra Wallet — Chrome Extension

Octra Wallet is a lightweight, secure browser extension for the Octra blockchain. It lets you create or import wallets, view balances, send transactions, and use privacy features directly from your browser popup.

### Features
- **Wallet management**: Create or import wallets (base64 private key + address)
- **Balances**: View public and private balances
- **Transactions**: Send tokens and review history
- **Privacy**: Encrypt/decrypt balances, private send, and claim private transfers
- **Bulk send**: Add multiple recipients and send in one action
- **Smart contract tasks**: Built-in testing task for the OCS01 contract

### How it works
- **UI**: `popup.html` with modular JS in `js/` and styles in `css/`
- **Background**: `background.js` service worker handles lifecycle and network proxying
- **Crypto**: Uses `libs/nacl.min.js` for cryptographic operations
- **Config**: Centralized in `js/config.js` (RPC, endpoints, UI, security)

### Permissions
Defined in `manifest.json`:
- **storage**: Persist wallet data and settings locally
- **alarms**: Lightweight scheduling (e.g., refresh)
- **host_permissions**: `https://octra.network/*`, `https://*.octra.network/*`

### Security
- Keys and settings are stored locally via `chrome.storage.local`
- Optional password protection with auto-lock options
- Background errors and unhandled rejections are captured in `background.js`
- Never share your private key or recovery information

### Install (Developer Mode)
1. Open Chrome and go to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked" and select the `extension/` folder
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


