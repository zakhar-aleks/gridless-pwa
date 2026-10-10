# Gridless — offline signed credits demo

**This is a simulated credit ledger, not cryptocurrency, bank money, or a real payment rail.**

## Architecture

Bluefy iPhone PWA (WebCrypto P-256 signature, IndexedDB nonextractable key) → BLE → ESP32 transparent relay → BLE → Raspberry Pi Zero 2 W (verify ECDSA signature, SQLite simulated ledger, BLE response) → iPhone receipt.

## Install

1. Flash `esp32_relay.ino` to ESP32 with Arduino IDE ESP32 BLE library.
2. On Pi: `python3 -m venv .venv && source .venv/bin/activate && pip install bleak cryptography && python gateway.py`. Pi and ESP32 should show CONNECTED.
3. Upload **only** `index.html`, `sw.js`, `manifest.json` to GitHub Pages or Netlify Drop (HTTPS). No npm, Solana, Phantom, RPC, or wallet JSON is needed.
4. On iPhone open HTTPS site **inside Bluefy**, not Safari. Let it create a wallet. Tap Connect, select Gridless-Node. Confirm both Pi and phone are connected to ESP32.
5. Enter merchant and amount, tap Sign payment offline. You can now turn on airplane mode **while manually leaving Bluetooth enabled**. Keep Bluefy foregrounded.
6. Tap Send via BLE. Watch Pi log `ACCEPTED ...`. PWA should show ACCEPTED and allow saving JSON receipt.

## Troubleshooting

- **Bluefy unavailable or no Bluetooth chooser**: Use Bluefy browser on iPhone; HTTPS required. Confirm Bluetooth permission. iOS Safari does not expose general Web Bluetooth.
- **Pi stops seeing ESP32 when phone connects**: ESP32 must support **two simultaneous BLE central connections**. Some Arduino BLE stack configurations are single-client. Test this first; adjust stack max connections or use NimBLE-Arduino configured for >=2.
- **Phone says sent but no final result**: check Pi `ACCEPTED` log. BLE notifications are best-effort; receipt remains unconfirmed until result received.
- **No ACK**: Pi isn't connected/subscribed, or the ESP32 is not forwarding notifications to both clients.
- **Offline PWA**: Visit page once while online and allow service worker to install before airplane mode. If Bluefy evicts cache, reconnect internet to load page.
- **Pi restart**: SQLite `gridless_ledger.sqlite3` persists accepted payments. Wallet key is persisted in Bluefy IndexedDB; clearing site data creates a new wallet.
- **Reset simulated balances**: stop gateway and delete `gridless_ledger.sqlite3` (DEMO ONLY).

## Security scope

P-256 WebCrypto signatures verify payment authorization, and SQLite receipt IDs prevent double debits. The public key is self-asserted; no trusted user identity or merchant enrollment exists. This is a hackathon proof-of-concept, not a secure payment system. A sender gets 100.00 simulated GRID upon first accepted payment; a merchant name maps to a demo ledger account. Browser IndexedDB storage is not equivalent to a hardware wallet. The ESP32 relays messages without inspecting them.
