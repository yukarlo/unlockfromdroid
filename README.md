# UnlockFromDroid (MacBleUnlock)

An open-source macOS companion application for proximity-based Mac auto-unlocking via an Android phone over Bluetooth Low Energy (BLE).

## Features & Architecture

- **BLE Proximity Auto-Unlock**: Fills in your password once your paired Android device is nearby and passes cryptographic authentication. See [When it acts](#when-it-acts) — the Mac never wakes itself.
- **Cryptographic GATT Challenge-Response**: Issues fresh 32-byte cryptographic challenges and verifies `SHA256withECDSA` P-256 signatures from your Android phone using `CryptoKit`.
- **Zero-Touch QR Code BLE Pairing**: Pair automatically by scanning a QR code with your phone.
- **Keychain Security**: Securely stores paired device identity public keys and login credentials in system Keychain (`kSecClassGenericPassword`).
- **Keystroke Automation (`KeyCodeMapper`)**: Dynamically resolves physical `CGKeyCode` hardware events and modifier states for reliable lock-screen password submission under macOS SecureInput.
- **Accessory Menu-Bar UI**: Native AppKit status item UI with real-time signal strength meter, diagnostic logs, and accessibility status management.

## When it acts

The Mac only talks to the phone when there is a lock screen someone is actually looking at:

| Mac state | BLE scanning | Challenge sent |
|---|---|---|
| Unlocked | no | no |
| Locked, display asleep | yes | **no** |
| Locked, display awake | yes | yes |

Scanning runs for the whole lock session, so the phone is already discovered and its signal
already averaged before the display comes back — the handshake starts warm rather than from a
cold scan.

**The Mac never wakes its own display.** Walking past your desk with the phone in your pocket
does nothing and raises no prompt on the phone. Press a key or touch the trackpad, and the
unlock follows. This is the same bargain Apple Watch unlock makes.

Waking the display clears a stalled connection or timeout so you are not left waiting on a
backoff from earlier. It deliberately does *not* clear a denial: if you refuse an unlock, that
refusal holds for two minutes regardless, so sleeping and waking the display cannot be used to
shake another prompt loose.

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
  keystrokes. That is the highest-risk component here, and with proximity auto-lock and
  wake-on-approach both removed it is now the app's only action — leave it off and the app just
  reports presence in the menu bar. What bounds the risk is that keystrokes are only ever sent to
  a lock screen you deliberately woke, at most three times per wake, each one gated on a freshly
  verified P-256 signature.
