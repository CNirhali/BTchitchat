# 🛡️ Security Policy

<a href="README.md" aria-label="Back to Documentation">← Back to Documentation</a>

## 📋 Table of Contents

<nav aria-label="Table of Contents">

- [🗓️ Supported Versions](#supported-versions)
- [🛡️ Security Development Guidelines](#security-development-guidelines)
  - [📡 Communication & Connectivity](#communication--connectivity)
  - [👤 Data Privacy & User Protection](#data-privacy--user-protection)
  - [🛡️ Application & Environment Hardening](#application--environment-hardening)
- [🚨 Reporting a Vulnerability](#reporting-a-vulnerability)

</nav>

## 🗓️ Supported Versions

Currently, we are in active development. Only the latest version of the application is supported.

| Version | Supported          |
| ------- | :----------------: |
| < 1.0.0 | ✅                  |

<a href="#security-policy" aria-label="Back to top of page">⬆ Back to Top</a>

## 🛡️ Security Development Guidelines

To maintain the security of the Bluetooth Chit Chat application, all contributors must follow these guidelines:

### 📡 Communication & Connectivity

- ⚖️ **Input Validation:** Sanitize and validate all data received over Bluetooth before processing or displaying it. Bluetooth packets can be manipulated by malicious devices.
- 🤝 **Secure Pairing & Authentication:** Implement secure pairing mechanisms (e.g., Numeric Comparison) and authenticate connected devices to prevent unauthorized access and MITM attacks. Enforce platform-level encryption for GATT characteristics by requiring bonded/encrypted permissions (e.g., `PERMISSION_READ_ENCRYPTED` / `PERMISSION_WRITE_ENCRYPTED` on Android or `.readEncryptionRequired` / `.writeEncryptionRequired` on iOS).
  ```kotlin
  // Example: Requiring encrypted permissions for a GATT characteristic on Android
  val characteristic = BluetoothGattCharacteristic(
      UUID.fromString("..."),
      BluetoothGattCharacteristic.PROPERTY_READ or BluetoothGattCharacteristic.PROPERTY_WRITE,
      BluetoothGattCharacteristic.PERMISSION_READ_ENCRYPTED or BluetoothGattCharacteristic.PERMISSION_WRITE_ENCRYPTED
  )
  ```
  ```swift
  // Example: Requiring encryption for a GATT characteristic on iOS (Core Bluetooth)
  let characteristic = CBMutableCharacteristic(
      type: CBUUID(string: "..."),
      properties: [.read, .write],
      value: nil,
      permissions: [.readEncryptionRequired, .writeEncryptionRequired]
  )
  ```
- 🚦 **Resource Limits & Rate Limiting:** Apply limits on message sizes and frequency of incoming Bluetooth packets to prevent denial-of-service (DoS) and memory exhaustion.
  ```kotlin
  // Example: Enforcing message size limits on Android
  val MAX_MESSAGE_SIZE = 1024 * 10 // 10KB
  if (receivedBytes.size > MAX_MESSAGE_SIZE) {
      throw SecurityException("Message size exceeds security limits.")
  }
  ```
  ```swift
  // Example: Enforcing message size limits in Swift
  let maxMessageSize = 1024 * 10 // 10KB
  guard receivedData.count <= maxMessageSize else {
      handleSecurityBreach("Message size exceeds security limits.")
      return
  }
  ```
- ⌛ **Replay Protection:** Implement cryptographically robust nonces (see `ChatMessage.secure_nonce`) or timestamps to prevent captured Bluetooth packets from being re-sent. Use cryptographically secure random nonces (at least 96 bits for AES-GCM) to ensure uniqueness across messages.
  ```kotlin
  // Example: Verifying a cryptographic nonce on Android to prevent replay attacks.
  // Using a size-limited cache prevents memory-based DoS, and Hex string
  // conversion ensures reliable byte array comparison.
  private val MAX_NONCE_CACHE_SIZE = 10000
  private val processedNonces = object : LinkedHashSet<String>() {
      override fun add(element: String): Boolean {
          if (size >= MAX_NONCE_CACHE_SIZE) remove(iterator().next())
          return super.add(element)
      }
  }

  fun isNonceValid(incomingNonce: ByteArray): Boolean {
      // Faster hex conversion using StringBuilder and bit-shifting
      // to reduce allocations and CPU overhead during message processing.
      val hexChars = "0123456789abcdef"
      val nonceHex = StringBuilder(incomingNonce.size * 2).apply {
          for (b in incomingNonce) {
              val i = b.toInt() and 0xFF
              append(hexChars[i shr 4])
              append(hexChars[i and 0x0F])
          }
      }.toString()

      if (processedNonces.contains(nonceHex)) return false
      processedNonces.add(nonceHex)
      return true
  }
  ```
  ```swift
  // Example: Verifying a cryptographic nonce in Swift to prevent replay attacks.
  // Using a size-limited cache with a circular buffer ensures O(1) removals.
  private var processedNonces = Set<Data>()
  private var nonceHistory = [Data?](repeating: nil, count: 10000)
  private var currentIndex = 0
  private let maxNonceCacheSize = 10000

  func isNonceValid(incomingNonce: Data) -> Bool {
      if processedNonces.contains(incomingNonce) { return false }

      // Remove oldest nonce from Set using circular buffer
      if let oldestNonce = nonceHistory[currentIndex] {
          processedNonces.remove(oldestNonce)
      }

      // Add new nonce to Set and Buffer
      processedNonces.insert(incomingNonce)
      nonceHistory[currentIndex] = incomingNonce
      currentIndex = (currentIndex + 1) % maxNonceCacheSize
      return true
  }
  ```
- 🛡️ **Message Integrity & Authenticity:** Use Message Authentication Codes (MACs) or digital signatures (see `ChatMessage.authentication_tag`) to ensure that messages have not been tampered with and originate from the claimed sender. It is highly recommended to use **Authenticated Encryption with Associated Data (AEAD)** schemes (e.g., AES-GCM, ChaCha20-Poly1305) to provide both confidentiality and integrity in a single operation.
- 🎯 **Recipient Binding & Verification:** Explicitly include and verify the `recipient_id` (see `ChatMessage.recipient_id`) in every message to prevent reflection attacks. When using AEAD, include the `sender_id`, `recipient_id`, and `timestamp` in the **Associated Data (AD)** to cryptographically bind the message to its context.
  ```kotlin
  // Example: Verifying recipient binding on Android to prevent reflection attacks
  fun verifyRecipient(message: ChatMessage, localDeviceId: String) {
      if (message.recipientId != localDeviceId) {
          throw SecurityException("Message recipient mismatch: Reflection attack detected.")
      }
  }
  ```
  ```swift
  // Example: Verifying recipient binding in Swift to prevent reflection attacks
  func verifyRecipient(message: ChatMessage, localDeviceId: String) throws {
      guard message.recipientId == localDeviceId else {
          throw SecurityError.invalidRecipient
      }
  }
  ```
- 🔄 **Protocol Versioning:** Include a protocol version in all messages (see `ChatMessage.protocol_version`) to allow for protocol evolution and to deprecate insecure legacy versions.
  ```kotlin
  // Example: Rejecting insecure protocol versions on Android
  val MIN_SUPPORTED_VERSION = 1
  if (message.protocolVersion < MIN_SUPPORTED_VERSION) {
      disconnectAndLogSecurityEvent("Unsupported insecure protocol version: ${message.protocolVersion}")
  }
  ```
  ```swift
  // Example: Rejecting insecure protocol versions in Swift
  let minSupportedVersion: UInt32 = 1
  guard message.protocolVersion >= minSupportedVersion else {
      handleSecurityBreach("Insecure protocol version detected: \(message.protocolVersion)")
      return
  }
  ```
- 📍 **Bluetooth Discoverability:** Implement a timeout for discoverability to minimize the window of exposure to unknown devices.
- 🌐 **Secure Network Communication:** Ensure all network traffic uses encrypted protocols (e.g., HTTPS). Disable cleartext traffic in the application configuration (e.g., `android:usesCleartextTraffic="false"` or `NSAppTransportSecurity` on iOS).
- 📲 **Secure Deep Link Handling:** Rigorously validate all incoming deep links and their parameters. Ensure that deep link actions do not bypass authentication/authorization or expose sensitive functionality to remote exploitation.
- 🧱 **Component Security:** Ensure all application components (Activities, Services, Receivers) are not exported unless absolutely necessary.
  ```xml
  <!-- Example: Secure component configuration in AndroidManifest.xml -->
  <activity
      android:name=".ChatActivity"
      android:exported="false" />
  ```

<a href="#security-policy" aria-label="Back to top of page">⬆ Back to Top</a>

### 👤 Data Privacy & User Protection

- 🛡️ **Principle of Least Privilege:** Only request the minimum necessary Bluetooth and Location permissions required for the app to function.
- 💾 **Secure Data Storage:** Encrypt chat logs and other sensitive local data. Use platform-specific secure storage (e.g., Keystore for Android, Keychain for iOS) for managing encryption keys.
- 🔒 **App-Level Locking:** Provide an option for app-level authentication (e.g., Biometrics, PIN) to protect access to chat history even when the device is unlocked.
- 🤏 **Data Minimization:** Only transmit essential data over Bluetooth. Avoid sending Personally Identifiable Information (PII) unless it is strictly necessary and properly encrypted.
- 🚫 **Data Leakage Prevention (DLP):** Prevent sensitive data leakage through unencrypted cloud backups (e.g., `android:allowBackup="false"`), the system clipboard, by obscuring sensitive UI content in the application switcher, or by disabling screenshots on sensitive screens (e.g., `FLAG_SECURE` on Android). Implement Overlay Protection (Anti-Tapjacking) to prevent malicious apps from intercepting touches by drawing over the application (e.g., `android:filterTouchesWhenObscured="true"`).
  ```kotlin
  // Example: Enabling anti-tapjacking protection on Android.
  // This prevents the application from responding to touch events when its window
  // is obscured by another window (e.g., a malicious overlay).
  view.filterTouchesWhenObscured = true
  ```
- 📝 **Secure Logging:** Do not log Personally Identifiable Information (PII) or sensitive message content. Use a production-ready logging library that supports log level filtering.
- 🔔 **Secure Notifications:** Avoid displaying sensitive chat content in system notifications that appear on the lock screen. Use generic summaries (e.g., "New Message") instead.
- ⌨️ **Keyboard Privacy:** Use incognito/private keyboard modes for chat input fields to prevent the keyboard from caching and learning sensitive message content.
  ```xml
  <!-- Example: Disabling personalized learning in Android XML layouts -->
  <EditText
      android:id="@+id/chatInput"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:imeOptions="flagNoPersonalizedLearning" />
  ```
  ```swift
  // Example: Enabling private keyboard mode in Swift (UIKit)
  textField.isSecureTextEntry = false // Ensure standard visibility
  textField.textContentType = .none
  textField.autocorrectionType = .no
  textField.spellCheckingType = .no
  ```
  ```swift
  // Example: Enabling private keyboard mode in Swift (SwiftUI)
  TextField("Message", text: $message)
      .autocorrectionDisabled(true)
      .textContentType(.none)
  ```
- 👤 **Device Identity Privacy:** Do not use the default system device name (e.g., "Alice's iPhone") for Bluetooth discovery, as it can leak Personally Identifiable Information (PII) to nearby observers. Implement generic aliases or allow users to set a pseudonym within the application.

<a href="#security-policy" aria-label="Back to top of page">⬆ Back to Top</a>

### 🛡️ Application & Environment Hardening

- 🔑 **No Hardcoded Secrets:** Never include API keys, passwords, or other sensitive credentials in the source code. Use environment variables or secure storage mechanisms.
- ⚠️ **Secure Error Handling:** Ensure that error messages do not leak sensitive information or stack traces. Log detailed errors internally, but provide generic messages to the user.
- 🎲 **Cryptographically Secure Randomness:** Use platform-provided cryptographically secure random number generators (CSPRNGs) for generating all cryptographic keys, nonces, and session identifiers.
- 🔗 **Dependency Security:** Regularly audit and update third-party libraries to mitigate risks from known vulnerabilities. Use tools like `pnpm audit` or `snyk` to automate this process.
- ⚔️ **Anti-Tampering & Integrity:** Implement root/jailbreak detection, **Anti-Debugging** (e.g., checking `android:debuggable="false"`), and **Runtime Signature Verification** to detect unauthorized analysis or binary modification. Use Code Obfuscation (e.g., R8/ProGuard for Android) to make reverse engineering more difficult.

<a href="#security-policy" aria-label="Back to top of page">⬆ Back to Top</a>

## 🚨 Reporting a Vulnerability

> [!IMPORTANT]
> Security is a top priority for this project. If you discover a security vulnerability, please do not open a public issue. Instead, please report it through the following process:

1. 📧 **Email:** Send a detailed report to [chaitanyanirhali@gmail.com](mailto:chaitanyanirhali@gmail.com?subject=Security%20Vulnerability%20Report%20-%20Bluetooth%20Chit%20Chat&body=Description:%0D%0A%0D%0ASteps%20to%20reproduce:%0D%0A%0D%0APotential%20impact:).
2. 📝 **Details:** Include a description of the vulnerability, steps to reproduce, and any potential impact.
3. ⌛ **Response:** You can expect an acknowledgment of your report within 48 hours. We will keep you updated on the progress of the fix.

We appreciate your help in keeping this project secure!

---

<a href="README.md" aria-label="Back to Documentation">← Back to Documentation</a>

<a href="#security-policy" aria-label="Back to top of page">⬆ Back to Top</a>
