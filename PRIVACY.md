# Octra Wallet Privacy Policy

Last updated: 2025-08-09

## Overview
Octra Wallet is a Chrome extension for managing Octra accounts. It stores wallet data locally and connects only to Octra network endpoints to fetch balances/history, submit transactions, and use privacy features.

## Data We Handle
- Financial and payment information
  - Wallet addresses, balances, and transaction details (amounts, tx hashes) retrieved from or sent to Octra endpoints.
- Authentication information
  - Your private key is generated/imported locally. It is encrypted client‑side before being saved to `chrome.storage.local`.
  - When you use privacy features (e.g., encrypt/decrypt balance, private transfers), your private key may be transmitted to Octra privacy endpoints strictly to process your request.
- Location (IP address)
  - As with any web request, your IP address is received by the Octra servers. The extension does not access GPS or device location APIs.

We do not collect: personally identifiable information, health information, web history, website content, or personal communications.

## Permissions Used
- storage: Persist encrypted wallet data (encrypted private keys, addresses), active wallet, and preferences (e.g., RPC URL, auto‑lock settings).
- Host permissions: `https://octra.network/*` and subdomains, used to read balances/history, submit transactions, and call privacy endpoints.

## Data Storage and Retention
- Wallet data is stored only on your device in `chrome.storage.local`.
- Sensitive data is encrypted client‑side using WebCrypto (AES‑GCM with PBKDF2‑derived keys) before storage.
- Data remains until you remove the extension data or clear it from the extension settings.

## Data Sharing and Selling
- We do not sell or transfer user data to third parties.
- Data is transmitted only to Octra network endpoints as required for wallet functionality.

## Security
- Private keys and sensitive wallet data are encrypted before storage using industry‑standard cryptography.
- Network requests use HTTPS. No remote code execution or `eval` is used; the extension runs with a strict `script-src 'self'` policy.

## Children’s Privacy
This extension is not directed to children under 13 and does not knowingly collect personal information from children.

## Changes to This Policy
We may update this policy from time to time. Material changes will be reflected by updating the “Last updated” date above.

## Contact
For privacy questions or requests, contact: [Telegram](https://t.me/nullXgery)
