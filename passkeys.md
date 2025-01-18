### Questions (may seem stupid):
- Can an attacker steal the signed challenge (I understand there is TLS) and log in?
- Will the passkey recovery workflow be defined in the upcoming specification, or is it up to the RP's discretion? If it is up to the RP, not all RPs may implement it securely, which could compromise FIDO2's effectiveness.
- Can an attacker with access to the device and RP replace the private and public keys and gain access?
- Does the mobile application also use the JS-WebAuthn API to interact, or are there platform-specific WebAuthn APIs?
- Is CTAP2 an extension of CTAP1?
- Is my below understanding of specification correct?

#### Native Device Security
- Unlocking a device using biometric scans like fingerprints or facial recognition is managed by the device's native security system.
- It is not part of the FIDO specifications.

#### Access Restrictions
- Neither the browser, web application, iOS SFSafariViewController, Android CustomTabs, nor the mobile application has access to private keys or biometric data.
- Only the device's operating system (OS) has access to biometric data.
- Only the device's operating system (OS) or the security key's firmware has access to the private key.
- The private key and biometric data never leave the device.
- The web and mobile applications only receive success/failure results of biometric authentication via APIs.
- The web and mobile applications only receive the signed challenge via APIs.

#### FIDO1 UAF (Universal Authentication Framework) - 2014
- Supports passwordless login for mobile applications that implement biometric authentication (e.g., fingerprint or facial recognition).
- Web applications are not supported.
- The private key can only be stored on the device.
- The private key is not synced to the cloud.
- Does not support storing private keys on external security keys.
- Utilizes asymmetric cryptography.

#### FIDO1 U2F (Universal 2nd Factor) Protocol - 2014 (right after FIDO1 UAF)
- Designed for 2FA (Two-Factor Authentication) using external security keys like YubiKeys and Google Titan security keys as the second factor, in addition to a password.
- Supports mobile and web applications.
- The private key can only be stored on external security keys.
- The private key cannot be stored on the device.
- The private key is not synced to the cloud.
- Was renamed FIDO2 CTAP1 with the release of FIDO2.
- Utilizes asymmetric cryptography.

### Authenticators
#### Platform Authenticators:
  - macOS Keychain or iCloud Keychain.
  - Android Google Password Manager.
  - Windows?

##### Roaming Authenticators:
  - YubiKeys, Google Titan Security Keys (using Bluetooth or NFC for communication).

##### Cross-Device Authenticators:
  - Mobile devices acting as roaming authenticators (using Bluetooth or NFC for communication).

#### FIDO2 (W3C WebAuthn and CTAP2) - 2022
- Supports passwordless login for both mobile and web applications using biometrics (fingerprint or face scans) or external security keys.
- The private key can be stored on external security keys.
- The private key can be stored on the device.
- The private key is synced to the cloud.
- Supports both mobile and web applications.

  #### WebAuthn:
  - Browser-based JavaScript APIs enabling communication with the operating system to interact with authenticators.
  - Not a protocol but a standard API for browsers.

  #### CTAP2 (Client-to-Authenticator Protocol):
  - A protocol for communication between the device and local, roaming, and cross-device authenticators.
