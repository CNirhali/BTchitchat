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
      device.connectGatt(context, false, gattCallback)
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
  // Example: Writing without response for ~2x throughput increase on Android
  characteristic.writeType = BluetoothGattCharacteristic.WRITE_TYPE_NO_RESPONSE
  bluetoothGatt.writeCharacteristic(characteristic)
  ```
  ```swift
  // Example: Writing without response for ~2x throughput increase in Swift
  peripheral.writeValue(data, for: characteristic, type: .withoutResponse)
  ```
  ```kotlin
  // Example: Using autoConnect=false for faster initial connection on Android.
  // This avoids the ~2s connection delay of the autoConnect=true background scan.
  device.connectGatt(context, false, gattCallback)
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
  func peripheral(_ peripheral: CBPeripheral, didModifyServices invalidatedServices: [CBService]) {
      peripheral.discoverServices(nil)
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
  val filter = ScanFilter.Builder().setServiceUuid(ParcelUuid(SERVICE_UUID)).build()
  val settings = ScanSettings.Builder().setScanMode(ScanSettings.SCAN_MODE_LOW_LATENCY).build()
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
  // Example: Dispatching Bluetooth work to a background queue in Swift
  let bluetoothQueue = DispatchQueue(label: "com.app.bluetooth", qos: .userInitiated)
  bluetoothQueue.async {
      // Perform discovery or GATT operations
  }
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
- ✨ **Connection Status:** Provide clear visual indicators for **"Disconnected"**, **"Connecting..."**, and **"Connected"** states.
  ```kotlin
  // Example: Updating connection status on Android
  fun onConnectionStateChange(newState: Int) {
      statusTextView.text = when (newState) {
          STATE_CONNECTED -> "Connected"
          STATE_CONNECTING -> "Connecting..."
          else -> "Disconnected"
      }
  }
  ```
  ```swift
  // Example: Updating connection status in Swift
  statusLabel.text = switch peripheral.state {
      case .connected: "Connected"
      case .connecting: "Connecting..."
      default: "Disconnected"
  }
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
- 📳 **Tactile Feedback:** Use haptic feedback for critical interactions like message delivery or connection success to provide a more tactile and responsive experience.
  ```kotlin
  // Example: Providing haptic feedback on Android
  view.performHapticFeedback(HapticFeedbackConstants.CONFIRM)
  ```
  ```swift
  // Example: Providing haptic feedback in Swift
  let generator = UINotificationFeedbackGenerator()
  generator.notificationOccurred(.success)
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
> Bluetooth has a typical range of about 10 meters (33 feet). For the best experience, ensure devices have a clear line of sight and are not obstructed by thick walls or large metal objects.

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🤝 Contributing

Got ideas to make this app better? Pull requests are always welcome! For major changes, please open an issue first to discuss what you would like to change.

## 📄 License

This project is licensed under the [MIT License](LICENSE) - see the LICENSE file for details.

---

<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>
