# UnlockFromDroid (MacBleUnlock)

An open-source macOS companion application for proximity-based Mac auto-unlocking via an Android phone over Bluetooth Low Energy (BLE).

## Features & Architecture

- **Zero-Touch BLE Proximity Auto-Unlock**: Automatically unlocks your Mac when your paired Android device comes into proximity and passes cryptographic authentication.
- **Cryptographic GATT Challenge-Response**: Issues fresh 32-byte cryptographic challenges and verifies `SHA256withECDSA` P-256 signatures from your Android phone using `CryptoKit`.
- **Zero-Touch QR Code BLE Pairing**: Pair automatically by scanning a QR code with your phone.
- **Keychain Security**: Securely stores paired device identity public keys and login credentials in system Keychain (`kSecClassGenericPassword`).
- **Keystroke Automation (`KeyCodeMapper`)**: Dynamically resolves physical `CGKeyCode` hardware events and modifier states for reliable lock-screen password submission under macOS SecureInput.
- **Accessory Menu-Bar UI**: Native AppKit status item UI with real-time signal strength meter, diagnostic logs, and accessibility status management.

## Project Structure

- `macos-app/`: macOS Swift menu-bar application built with SwiftUI and AppKit. See [macos-app/README.md](macos-app/README.md) for build instructions, setup, and security notes.

The Android BLE peripheral is a separate project (`UnlockMyMac`). Both sides share the protocol
spec and test vectors in that repo's `app/src/test/resources/protocol-vectors.json`.

## Build Requirements

- macOS 13.0+
- Xcode 14.0+ / Swift 5.0+
- XcodeGen (`brew install xcodegen`)

```bash
cd macos-app
xcodegen generate
xcodebuild -project MacBleUnlock.xcodeproj -scheme MacBleUnlock -configuration Debug -destination 'platform=macOS' CODE_SIGNING_ALLOWED=NO build
```

## Security

Personal hobby project. Read this before trusting it with your login.

- The advertised service UUID is public and used for discovery only. Trust comes solely from a
  P-256 signature over a fresh 32-byte challenge naming this Mac and the paired device.
- **No defence against BLE relay attacks.** There is no distance bounding; an attacker relaying
  traffic between the phone and the Mac defeats challenge-response entirely.
- Auto-unlock is off by default and stores your login password in the Keychain for replay as
  keystrokes. That is the highest-risk component here — wake-on-approach plus lock-on-absence
  gives most of the benefit without it.
