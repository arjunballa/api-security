### Questions (may seem stupid):
- Can an attacker steal the signed challenge (I understand there is TLS) and log-in in theory?
- How safe is it to sync private keys to the cloud? Isn't it violating its own criticism of storing passwords in a database?
- How is it secure to use passkey login while still retaining password login? GitHub allows adding passkeys as MFA, which enhances security, but many platforms don’t.  
- What is the best path to completely migrate to passkeys from password login? Is it possible to migrate to passskeys without a robust approach to passkey recovery mechanisms?
- Will the passkey recovery workflow be defined in the upcoming specification, or is it up to the RP's discretion? If it is up to the RP, not all RPs may implement it securely, wouldn't it compromise FIDO2's effectiveness?
- Does the mobile application also use the JS-WebAuthn API to interact, or are there platform-specific WebAuthn APIs?
- Is CTAP2 used to communicate with the macOS Local Keychain Access and iCloud Keychain (I know it is used to communicate with roaming and cross device authenticator)?
- Can an attacker with access to the device and RP replace the private and public keys and gain access in theory?
- May I request to comment on talk [We Want Less Passwords, Not Passwordless by Chand Spensky Founder of Allthenticate](https://youtu.be/XhBauX9VyiQ)
- **Is my below understanding of history, jagron and specification correct?**

#### Native Device Security
- Unlocking a device using biometric scans like fingerprints or facial recognition is managed by the device's native security system and tt is **not part of the FIDO specifications**.

#### Access Restrictions
- Neither the **browser, web application, iOS SFSafariViewController, Android CustomTabs, nor the mobile application** has access to **private keys or biometric data**.
- Only the **device's operating system (OS)** has access to **biometric data**.
- Only the **device's operating system (OS)** or the security key's firmware has access to the **private key**.
- The **private key and biometric** data **never leave** the device.
- The **WebAuthn APIs** for web and mobile applications **do not sign** the challenge **directly** as they directly don't have access to private key; instead, they interact with the **device OS** to have the challenge **signed**.
- The **WebAuthn APIs** for web and mobile applications interact with the device OS and **only receive success/failure** results of **biometric authentication**.

#### FIDO1 UAF (Universal Authentication Framework)
- Released in 2014
- Supports **passwordless** login for **mobile applications only** that implement biometric authentication (e.g., fingerprint or facial recognition).
- Web applications are **not supported**.
- The private key **can only be** stored on the **device**.
- The private key **can not be** stored on the **external security keys**.
- The private key is **not synced** to the **cloud**.
- Utilizes asymmetric cryptography.

#### FIDO1 U2F (Universal 2nd Factor) Protocol
- Released in 2014 right after FIDO1 UAF.
- Designed for **2FA (Two-Factor Authentication)** using external **security keys only** like YubiKeys and Google Titan security keys as the second factor, in addition to a password.
- Supports mobile and web applications.
- The private key **can only be** stored on **external security keys**.
- The private key **cannot be** stored on the **device**.
- The private key is **not synced** to the **cloud**.
- Utilizes asymmetric cryptography.
- Is UAF part of this spec? If yes, what role does it play?
- Is FIDO1 U2F is renamed FIDO2 CTAP1 with the release of FIDO2?.

  #### U2F JS API
  -  Browser-based JavaScript APIs enabling communication with the operating system to interact with authenticators.

### Authenticators
#### Platform Authenticators:
  - macOS Local Keychain Access or iCloud Keychain.
  - Android Google Password Manager.
  - Windows?

##### Roaming Authenticators:
  - YubiKeys, Google Titan Security Keys (using Bluetooth or NFC for communication).

##### Cross-Device Authenticators:
  - Mobile devices acting as roaming authenticators (using Bluetooth or NFC for communication).

#### FIDO2 (W3C WebAuthn and CTAP2)
- Released in 2022.
- Supports passwordless login for both mobile and web applications using biometrics (fingerprint or face scans) or external security keys.
- The private key **can be** stored on external **security keys**.
- The private key **can be** stored on the **device**.
- The private key **is synced** to the **cloud**.
- Supports both mobile and web applications.

  #### WebAuthn:
  - Browser-based JavaScript APIs enabling communication with the operating system to interact with authenticators.
  - Not a protocol but a standard API for browsers.

  #### CTAP2 (Client-to-Authenticator Protocol):
  - A protocol for communication between the device and local, roaming, and cross-device authenticators.
