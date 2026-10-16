# PRD.md: Authenticator

## Product
Local Android TOTP authenticator (WebView + HTML/JS) for generating 30-second 2FA codes.

## Goals
- Add accounts with issuer, label, and Base32 secret
- Show live TOTP codes with a 30s countdown
- Copy codes to clipboard
- Persist accounts in LocalStorage (no credential network calls)

## Out of scope
- Cloud sync, account login, QR scanning (unless added later)
