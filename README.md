# 📱 Bluetooth Chit Chat

A lightweight, offline messaging app for Bluetooth-based chat. No internet or cell service required—perfect for classrooms, camping, or crowded events with weak Wi-Fi.

## 📋 Table of Contents

<nav aria-label="Table of Contents">

- [✨ Features](#-features)
- [📸 Screenshots](#-screenshots)
- [🛠️ Tech Stack](#-tech-stack)
- [🚀 Getting Started](#-getting-started)
  - [🛠️ Prerequisites](#-prerequisites)
  - [📦 Installation](#-installation)
- [⚡ Performance](#-performance)
- [🔒 Security](#-security)
- [🎨 UI/UX Guidelines](#-uiux-guidelines)
- [💬 How to Use](#-how-to-use)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)

</nav>

## ✨ Features

- 📡 **100% Offline:** Direct peer-to-peer messaging via Bluetooth.
- ⚡ **Quick Pairing:** Connect to nearby devices instantly.
- 💬 **Real-Time UI:** Familiar chat interface with connection status.
- 👤 **No Accounts:** Zero-configuration; start chatting immediately.

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 📸 Screenshots

| 🔍 Device Discovery | 💬 Active Chat Room |
| :---: | :---: |
<!-- ⚡ Optimization: decoding="async" and fetchpriority="high" reduce main-thread contention and prioritize critical asset loading, improving Largest Contentful Paint (LCP). Above-the-fold images omit loading="lazy". -->
| <img src="https://via.placeholder.com/300x600?text=Discovery+UI" width="300" height="600" alt="A mobile screen showing a list of discovered nearby Bluetooth devices with names like 'Nexus 5X' and 'Pixel 4a'" decoding="async" fetchpriority="high"> | <img src="https://via.placeholder.com/300x600?text=Chat+UI" width="300" height="600" alt="A chat conversation between two users with blue and gray message bubbles" decoding="async" fetchpriority="high"> |

## 🛠️ Tech Stack

This template can be implemented using any mobile technology stack with Bluetooth support:
- 📱 **Native:** [Android (Kotlin)](https://developer.android.com/develop/connectivity/bluetooth) or [iOS (Swift)](https://developer.apple.com/documentation/corebluetooth)
- 🌐 **Cross-platform:** [Flutter](https://pub.dev/packages/flutter_blue_plus) or [React Native](https://github.com/dotintent/react-native-ble-plx)
- 📶 **Connectivity:** Peer-to-peer Bluetooth communication

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🚀 Getting Started

### 🛠️ Prerequisites

> [!IMPORTANT]
> This app requires **two physical devices** to test Bluetooth connectivity. Simulators or emulators typically do not support Bluetooth hardware.

- [ ] 📶 **Bluetooth Enabled:** Must be enabled in the system settings of both devices.
- [ ] 📱 **Compatible Mobile OS:** Android 8.0+ or iOS 13.0+ (recommended).
- [ ] 🛡️ **Permissions:** Location and Nearby Devices permissions must be granted for discovery.

### 📦 Installation

1. 📥 **Clone this repository** to your local machine and navigate into the directory:
   ```bash
   git clone --depth 1 https://github.com/yourusername/bluetooth-chit-chat.git
   cd bluetooth-chit-chat
   ```
2. 📦 **Install dependencies** using `pnpm`:
   ```bash
   pnpm install
   ```
3. 💻 **Open the project** in your preferred IDE (e.g., Android Studio, Xcode, or VS Code).
4. 🚀 **Build and run the application** on your physical devices.

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## ⚡ Performance

Bluetooth throughput is limited and latency can vary. To ensure a fast experience:
- 📦 **Binary Serialization:** Use efficient formats like [Protobuf](https://protobuf.dev/) (see our [optimized schema](schema/chat_message.proto)) or [FlatBuffers](https://google.github.io/flatbuffers/) to minimize payload size and processing overhead. Use minimal message types like `HEARTBEAT` for low-overhead connection maintenance.
- 🚀 **MTU Negotiation:** Request a larger Maximum Transmission Unit (MTU) to increase throughput for larger messages (up to 512 bytes on BLE).
  ```kotlin
  // Example: Requesting larger MTU on Android
  bluetoothGatt.requestMtu(512)
  ```
  ```swift
  // Example: Querying maximum write length in Swift (equivalent to MTU)
  let mtu = peripheral.maximumWriteValueLength(for: .withoutResponse)
  ```
- ⚡ **Message Batching:** If sending multiple updates, batch them into a single Bluetooth packet to reduce protocol overhead.
  ```kotlin
  // Example: Batching multiple messages into a single list before serialization on Android.
  // This reduces the number of Bluetooth writes and associated protocol overhead.
  val batch = MessageBatch.newBuilder()
      .addAllMessages(pendingMessages)
      .build()
  // Using WRITE_TYPE_NO_RESPONSE provides ~2x throughput increase for high-frequency updates.
  bluetoothGatt.writeCharacteristic(characteristic, batch.toByteArray(), BluetoothGattCharacteristic.WRITE_TYPE_NO_RESPONSE)
  ```
  ```swift
  // Example: Batching messages into a single data payload in Swift.
  // Using .withoutResponse provides ~2x throughput increase compared to .withResponse.
  let batch = MessageBatch(messages: pendingMessages)
  if let data = try? batch.serializedData() {
      peripheral.writeValue(data, for: characteristic, type: .withoutResponse)
  }
  ```
- 🔋 **Battery Efficiency:** Disable Bluetooth discovery/scanning immediately after connection to save power and improve connection stability.
  ```kotlin
  // Example: Stopping discovery immediately upon connection on Android
  fun onDeviceSelected(device: BluetoothDevice) {
      bluetoothLeScanner.stopScan(scanCallback)
      // Specifying TRANSPORT_LE avoids dual-mode (BR/EDR) overhead for faster Low Energy connections.
      device.connectGatt(context, false, gattCallback, BluetoothDevice.TRANSPORT_LE)
  }
  ```
  ```swift
  // Example: Stopping discovery immediately upon connection in Swift
  func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral, advertisementData: [String : Any], rssi RSSI: NSNumber) {
      central.stopScan()
      central.connect(peripheral, options: nil)
  }
  ```
- 📉 **Lower Latency:** Use direct connection handles where possible, minimize unnecessary application-layer acknowledgments, and utilize **Write Without Response** for high-throughput data.
  ```kotlin
  // Example: Writing without response for ~2x throughput increase on Android.
  // This reduces protocol overhead by not requiring an acknowledgement for each packet.
  characteristic.writeType = BluetoothGattCharacteristic.WRITE_TYPE_NO_RESPONSE
  bluetoothGatt.writeCharacteristic(characteristic)
  ```
  ```kotlin
  // Example: Using autoConnect=false and TRANSPORT_LE for faster initial connection on Android.
  // This avoids the ~2s connection delay of the autoConnect=true background scan
  // and dual-mode (BR/EDR) overhead.
  device.connectGatt(context, false, gattCallback, BluetoothDevice.TRANSPORT_LE)
  ```
  ```kotlin
  // Example: Requesting 2M PHY for up to 2x physical layer throughput on Android (BLE 5.0+).
  // This increases the physical bit rate from 1 Mbps to 2 Mbps, reducing radio airtime.
  fun requestHighSpeedPhy(gatt: BluetoothGatt) {
      gatt.setPreferredPhy(
          BluetoothDevice.PHY_LE_2M_MASK,
          BluetoothDevice.PHY_LE_2M_MASK,
          BluetoothDevice.PHY_OPTION_NO_PREFERRED
      )
  }
  ```
  ```swift
  // Example: Implementing flow control for Writes Without Response in Swift.
  // Using 'peripheralIsReady(toSendWriteWithoutResponse:)' prevents buffer overflows
  // and ensures the highest possible throughput by waiting for the next available slot.
  func sendBatchedData(_ data: Data, to peripheral: CBPeripheral, for characteristic: CBCharacteristic) {
      if peripheral.canSendWriteWithoutResponse {
          peripheral.writeValue(data, for: characteristic, type: .withoutResponse)
      }
  }

  func peripheralIsReady(toSendWriteWithoutResponse peripheral: CBPeripheral) {
      // Called when the internal buffer has cleared; resume sending pending data.
      sendNextPendingMessage()
  }
  ```
- ♻️ **Object Pooling & Capacity Reuse:** Reuse builders and collections to minimize Garbage Collection (GC) overhead and prevent UI jank during high-frequency data exchange.
  ```kotlin
  // Example: Pooling Protobuf Builders on Android (ChatMessage objects themselves are immutable).
  // Reusing the builder avoids repeated allocations during frequent message creation.
  val builderPool = Pools.SimplePool<ChatMessage.Builder>(10)
  val builder = builderPool.acquire()?.clear() ?: ChatMessage.newBuilder()
  // ... use builder ...
  builderPool.release(builder)
  ```
  ```swift
  // Example: Reusing array capacity in Swift for high-frequency message buffering.
  // 'removeAll(keepingCapacity: true)' prevents re-allocating the underlying buffer.
  var pendingMessages = [ChatMessage]()
  pendingMessages.reserveCapacity(100)
  // ... after sending ...
  pendingMessages.removeAll(keepingCapacity: true)
  ```
- ⏱️ **Lazy Initialization:** Delay Bluetooth stack setup and discovery until strictly necessary to improve initial app launch speed and reduce memory footprint.
- 📡 **GATT Caching:** Leverage GATT Service Caching to skip service discovery on subsequent connections and reduce connection-to-chat time.
  ```kotlin
  // Example: Handling GATT service changes on Android.
  // The system automatically caches services; use the 'onServiceChanged' callback
  // to refresh discovery only when services have changed.
  override fun onServiceChanged(gatt: BluetoothGatt) {
      gatt.discoverServices()
  }
  ```
  ```swift
  // Example: Handling GATT service changes in Swift
  // Passing 'nil' discovers all services; specifying the service UUID is more efficient.
  func peripheral(_ peripheral: CBPeripheral, didModifyServices invalidatedServices: [CBService]) {
      peripheral.discoverServices([serviceUUID])
  }
  ```
- 📶 **Connection Priority:** Request high-priority/low-latency connections during active chat sessions to minimize message delivery delays.
  ```kotlin
  // Example: Requesting high priority connection on Android.
  // Reduces connection interval (e.g., from 40-50ms down to 11.25-15ms) for faster message delivery.
  bluetoothGatt.requestConnectionPriority(BluetoothGatt.CONNECTION_PRIORITY_HIGH)
  ```
- 🔍 **Filtered Scanning:** Use Service UUID filters during device discovery to speed up the process and reduce system-wide power consumption.
  ```kotlin
  // Example: Filtered scanning by Service UUID on Android.
  // Reduces system-wide power consumption and speeds up discovery of relevant devices.
  // Using CALLBACK_TYPE_FIRST_MATCH ensures the scanner reports only the first match per device,
  // reducing CPU and battery consumption.
  val filter = ScanFilter.Builder().setServiceUuid(ParcelUuid(SERVICE_UUID)).build()
  val settings = ScanSettings.Builder()
      .setScanMode(ScanSettings.SCAN_MODE_LOW_LATENCY)
      .setCallbackType(ScanSettings.CALLBACK_TYPE_FIRST_MATCH)
      .build()
  bluetoothLeScanner.startScan(listOf(filter), settings, scanCallback)
  ```
  ```swift
  // Example: Filtered scanning by Service UUID in Swift.
  // The system only wakes the app for devices matching the specified service UUID.
  centralManager.scanForPeripherals(withServices: [serviceUUID], options: nil)
  ```
- 🧵 **Background Threading:** Perform all Bluetooth GATT operations, discovery, and data serialization on background threads to prevent UI jank and maintain 60 FPS responsiveness.
  ```kotlin
  // Example: Using a dedicated background thread for Bluetooth operations on Android.
  // Prevents blocking the Main (UI) thread during serialization or GATT writes.
  val bluetoothScope = CoroutineScope(Dispatchers.IO + SupervisorJob())
  bluetoothScope.launch {
      // Perform data serialization or Bluetooth GATT operations
  }
  ```
  ```swift
  // Example: Initializing the Central Manager with a background queue in Swift.
  // This ensures all delegate callbacks and GATT operations run off the Main (UI) thread,
  // preventing UI jank and maintaining 60 FPS responsiveness.
  let bluetoothQueue = DispatchQueue(label: "com.app.bluetooth", qos: .userInitiated)
  let centralManager = CBCentralManager(delegate: self, queue: bluetoothQueue)
  ```

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🔒 Security

Bluetooth communication is inherently susceptible to various security risks, including eavesdropping and man-in-the-middle attacks. For detailed security practices, see our [SECURITY.md](SECURITY.md).

- 🔐 **Encryption & Integrity:** This template currently does **not** implement End-to-End Encryption (E2EE) or Message Integrity Checks. All messages are sent in plain text and are susceptible to tampering.
- 💡 **Recommendations:** For production use, it is highly recommended to implement a robust E2EE layer with **Perfect Forward Secrecy (PFS)** using libraries like [Noise Protocol](https://noiseprotocol.org/) or [libsodium](https://doc.libsodium.org/).
- 👤 **Privacy:** Be mindful of the data shared over Bluetooth, as nearby devices may be able to monitor the traffic if not properly secured.

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🎨 UI/UX Guidelines

To provide a smooth and intuitive messaging experience over Bluetooth:
- ✨ **Connection Status:** Provide clear visual indicators for **"Disconnected"**, **"Connecting..."**, and **"Connected"** states. Use distinct icons (e.g., 🔴, 🟡, 🟢) in addition to colors to ensure accessibility for colorblind users.
  ```kotlin
  // Example: Updating connection status on Android
  fun onConnectionStateChange(newState: Int) {
      val status = when (newState) {
          STATE_CONNECTED -> "🟢 Connected"
          STATE_CONNECTING -> "🟡 Connecting..."
          else -> "🔴 Disconnected"
      }
      statusTextView.text = status
      // Ensuring status changes are announced to screen reader users
      statusTextView.announceForAccessibility("Connection status: $status")
  }
  ```
  ```swift
  // Example: Updating connection status in Swift
  let status = switch peripheral.state {
      case .connected: "🟢 Connected"
      case .connecting: "🟡 Connecting..."
      default: "🔴 Disconnected"
  }
  statusLabel.text = status
  // Post an accessibility notification so VoiceOver announces the new status
  UIAccessibility.post(notification: .announcement, argument: "Connection status: \(status)")
  ```
  ```tsx
  // Example: Accessible status updates in React Native (TSX)
  // Using accessibilityLiveRegion ensures screen readers announce status changes immediately.
  <Text
    accessibilityLabel={`Connection status: ${status}`}
    accessibilityLiveRegion="polite"
  >
    {status === 'connected' ? '🟢 Connected' : status === 'connecting' ? '🟡 Connecting...' : '🔴 Disconnected'}
  </Text>
  ```
- ⏳ **Loading States:** Use skeletons or spinners during device discovery and connection attempts to manage user expectations.
  ```kotlin
  // Example: Showing a loading state during discovery on Android
  fun onDiscoveryStarted() {
      discoveryProgressBar.visibility = View.VISIBLE
      emptyStateView.text = "Searching for devices..."
  }
  ```
  ```swift
  // Example: Showing a loading state during discovery in Swift
  func startScanning() {
      activityIndicator.startAnimating()
      statusLabel.text = "Searching for devices..."
  }
  ```
  ```tsx
  // Example: Showing a loading state in React Native (TSX)
  {isScanning && (
    <View style={styles.loadingContainer}>
      <ActivityIndicator size="large" />
      <Text accessibilityLiveRegion="polite">Searching for devices...</Text>
    </View>
  )}
  ```
- 💬 **Message Feedback:** Show **"Sending..."**, **"Sent"**, or **"Delivered"** statuses for messages to confirm successful transmission.
  ```kotlin
  // Example: Updating message status on Android
  fun onMessageStatusUpdated(status: MessageStatus) {
      statusTextView.text = when (status) {
          SENDING -> "Sending..."
          SENT -> "Sent"
          DELIVERED -> "Delivered"
      }
  }
  ```
  ```swift
  // Example: Updating message status in Swift
  messageStatusLabel.text = switch message.deliveryStatus {
      case .sending: "Sending..."
      case .sent: "Sent"
      case .delivered: "Delivered"
  }
  ```
  ```tsx
  // Example: Updating message status in React Native (TSX)
  <Text accessibilityLabel={`Message status: ${message.status}`}>
    {message.status === 'sending' ? 'Sending...' : message.status === 'sent' ? 'Sent' : 'Delivered'}
  </Text>
  ```
- ⌨️ **Keyboard Interactions:** Support standard keyboard behaviors like **"Enter to Send"** to improve efficiency for power users and ensure accessibility for keyboard-based navigation.
  ```kotlin
  // Example: Supporting 'Enter to Send' on Android
  messageEditText.setOnEditorActionListener { _, actionId, _ ->
      if (actionId == EditorInfo.IME_ACTION_SEND) {
          sendMessage()
          true
      } else false
  }
  ```
  ```swift
  // Example: Supporting 'Enter to Send' in Swift (UIKit)
  func textFieldShouldReturn(_ textField: UITextField) -> Bool {
      if textField == messageTextField {
          sendMessage()
          return false // Prevent default behavior
      }
      return true
  }
  ```
  ```tsx
  // Example: Supporting 'Enter to Send' in React Native (TSX)
  // 'blurOnSubmit={false}' keeps the keyboard open for rapid messaging.
  <TextInput
    placeholder="Type a message..."
    returnKeyType="send"
    onSubmitEditing={sendMessage}
    blurOnSubmit={false}
  />
  ```
- 🔔 **Actionable Alerts:** Use non-intrusive toasts or snackbars for errors (e.g., **"Bluetooth Disabled"**, **"Connection Failed"**) with clear recovery steps.
  ```kotlin
  // Example: Showing a non-intrusive error with a recovery action on Android
  Snackbar.make(rootView, "Bluetooth Disabled", Snackbar.LENGTH_LONG)
      .setAction("Enable") { bluetoothAdapter.enable() }
      .show()
  ```
  ```swift
  // Example: Showing a standard alert in Swift (UIKit)
  let alert = UIAlertController(title: "Bluetooth Disabled", message: "Please enable Bluetooth to chat.", preferredStyle: .alert)
  alert.addAction(UIAlertAction(title: "Settings", style: .default) { _ in
      if let url = URL(string: UIApplication.openSettingsURLString) {
          UIApplication.shared.open(url)
      }
  })
  alert.addAction(UIAlertAction(title: "OK", style: .cancel))
  present(alert, animated: true)
  ```
  ```tsx
  // Example: Showing a standard alert in React Native (TSX)
  Alert.alert(
    "Bluetooth Disabled",
    "Please enable Bluetooth to chat.",
    [
      { text: "Settings", onPress: () => Linking.openSettings() },
      { text: "OK", style: "cancel" }
    ]
  );
  ```
- ♿ **Accessibility:** Ensure high color contrast for text and large touch targets (at least 48x48dp) for all interactive UI elements. Provide descriptive labels for icon-only buttons to support screen readers.
  ```kotlin
  // Example: Ensuring accessible touch targets and labels on Android
  button.apply {
      // Minimum recommended touch target size of 48x48dp
      minWidth = 48.dp
      minHeight = 48.dp
      // Descriptive label for screen readers
      contentDescription = "Send Message"
  }
  ```
  ```swift
  // Example: Ensuring accessible touch targets and labels in Swift (UIKit)
  button.accessibilityLabel = "Send Message"
  // Ensure the frame is at least 44x44pt (iOS standard) or 48x48pt
  button.frame = CGRect(x: 0, y: 0, width: 48, height: 48)
  ```
  ```tsx
  // Example: Ensuring accessible touch targets and labels in React Native (TSX)
  // 'hitSlop' expands the touchable area without changing the visual layout.
  <Pressable
    accessibilityLabel="Send Message"
    hitSlop={12}
    onPress={sendMessage}
  >
    <SendIcon />
  </Pressable>
  ```
- 📳 **Tactile Feedback:** Use haptic feedback to provide physical confirmation for key actions like sending messages or successful connections.
  ```kotlin
  // Example: Providing haptic feedback on Android
  view.performHapticFeedback(HapticFeedbackConstants.CONFIRM)
  ```
  ```swift
  // Example: Providing haptic feedback in Swift (UIKit)
  let generator = UINotificationFeedbackGenerator()
  generator.notificationOccurred(.success)
  ```
  ```tsx
  // Example: Providing haptic feedback in React Native
  Vibration.vibrate(10)
  ```
- 📭 **Empty States:** Provide helpful guidance or calls-to-action when no data is present (e.g., **"Scanning for nearby friends..."**).
  ```kotlin
  // Example: Showing a helpful empty state on Android
  if (discoveredDevices.isEmpty()) {
      statusTextView.text = "Scanning for nearby friends..."
      progressBar.visibility = View.VISIBLE
  }
  ```
  ```swift
  // Example: Showing a helpful empty state in Swift
  if discoveredDevices.isEmpty {
      statusLabel.text = "Scanning for nearby friends..."
      activityIndicator.startAnimating()
  }
  ```
  ```tsx
  // Example: Showing a helpful empty state in React Native (TSX)
  {discoveredDevices.length === 0 && (
    <View style={styles.emptyState}>
      <Text>Scanning for nearby friends...</Text>
      <ActivityIndicator size="small" />
    </View>
  )}
  ```

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 💬 How to Use

1. 📱 **On Device A:** Tap **"Make Discoverable"** or **"Host Chat"**.
2. 🔍 **On Device B:** Scan for nearby devices and tap **"Device A's name"** to initiate a connection.
3. 💬 **Messaging:** Once connected, type your message and tap **"Send"**!

> [!TIP]
> Always provide visual feedback for connection status changes. A simple status indicator or toast can significantly reduce user frustration during transient Bluetooth interruptions.

> [!TIP]
> Protect your privacy by using a generic alias (e.g., "ChatUser-123") instead of your real name or device model (e.g., "Alice's iPhone 15") during Bluetooth discovery.

> [!TIP]
> Bluetooth has a typical range of about 10 meters (33 feet). For the best experience, ensure devices have a clear line of sight and are not obstructed by thick walls or large metal objects.

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🤝 Contributing

Got ideas to make this app better? Pull requests are always welcome! For major changes, please open an issue first to discuss what you would like to change.

## 📄 License

This project is licensed under the [MIT License](LICENSE) - see the LICENSE file for details.

---

<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>
