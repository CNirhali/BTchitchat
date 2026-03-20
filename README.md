# 📱 Bluetooth Chit Chat

A lightweight, offline messaging app for Bluetooth-based chat. No internet or cell service required—perfect for classrooms, camping, or crowded events with weak Wi-Fi.

## 📋 Table of Contents

<nav aria-label="Table of Contents">

- [✨ Features](#features)
- [📸 Screenshots](#screenshots)
- [🛠️ Tech Stack](#tech-stack)
- [🚀 Getting Started](#getting-started)
  - [🛠️ Prerequisites](#prerequisites)
  - [📦 Installation](#installation)
- [⚡ Performance](#performance)
- [🔒 Security](#security)
- [🎨 UI/UX Guidelines](#uiux-guidelines)
- [💬 How to Use](#how-to-use)
- [🤝 Contributing](#contributing)
- [📄 License](#license)

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
<a href="#bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

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
<a href="#bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## ⚡ Performance

Bluetooth throughput is limited and latency can vary. To ensure a fast experience:
- 📦 **Binary Serialization:** Use efficient formats like [Protobuf](https://protobuf.dev/) (see our [optimized schema](schema/chat_message.proto)) or [FlatBuffers](https://google.github.io/flatbuffers/) to minimize payload size and processing overhead. Use minimal message types like `HEARTBEAT` for low-overhead connection maintenance.
- 🚀 **MTU Negotiation:** Request a larger Maximum Transmission Unit (MTU) to increase throughput for larger messages (up to 512 bytes on BLE).
  ```kotlin
  // Example: Requesting larger MTU on Android
  bluetoothGatt.requestMtu(512)
  ```
- ⚡ **Message Batching:** If sending multiple updates, batch them into a single Bluetooth packet to reduce protocol overhead.
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
  ```kotlin
  // Example: Using autoConnect=false for faster initial connection on Android.
  // This avoids the ~2s connection delay of the autoConnect=true background scan.
  device.connectGatt(context, false, gattCallback)
  ```
- ♻️ **Object Pooling:** Reuse byte buffers and message objects to minimize Garbage Collection (GC) overhead and prevent UI jank during high-frequency data exchange.
  ```kotlin
  // Example: Basic object pool for reusing ChatMessage objects on Android
  val messagePool = Pools.SimplePool<ChatMessage>(10)
  val message = messagePool.acquire() ?: ChatMessage()
  // ... use message ...
  messagePool.release(message)
  ```
- ⏱️ **Lazy Initialization:** Delay Bluetooth stack setup and discovery until strictly necessary to improve initial app launch speed and reduce memory footprint.
- 📡 **GATT Caching:** Leverage GATT Service Caching to skip service discovery on subsequent connections and reduce connection-to-chat time.
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
  ```swift
  // Example: Dispatching Bluetooth work to a background queue in Swift
  let bluetoothQueue = DispatchQueue(label: "com.app.bluetooth", qos: .userInitiated)
  bluetoothQueue.async {
      // Perform discovery or GATT operations
  }
  ```

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🔒 Security

Bluetooth communication is inherently susceptible to various security risks, including eavesdropping and man-in-the-middle attacks. For detailed security practices, see our [SECURITY.md](SECURITY.md).

- 🔐 **Encryption & Integrity:** This template currently does **not** implement End-to-End Encryption (E2EE) or Message Integrity Checks. All messages are sent in plain text and are susceptible to tampering.
- 💡 **Recommendations:** For production use, it is highly recommended to implement a robust E2EE layer with **Perfect Forward Secrecy (PFS)** using libraries like [Noise Protocol](https://noiseprotocol.org/) or [libsodium](https://doc.libsodium.org/).
- 👤 **Privacy:** Be mindful of the data shared over Bluetooth, as nearby devices may be able to monitor the traffic if not properly secured.

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🎨 UI/UX Guidelines

To provide a smooth and intuitive messaging experience over Bluetooth:
- ✨ **Connection Status:** Provide clear visual indicators for **"Disconnected"**, **"Connecting..."**, and **"Connected"** states.
- ⏳ **Loading States:** Use skeletons or spinners during device discovery and connection attempts to manage user expectations.
- 💬 **Message Feedback:** Show **"Sending..."**, **"Sent"**, or **"Delivered"** statuses for messages to confirm successful transmission.
- 🔔 **Actionable Alerts:** Use non-intrusive toasts or snackbars for errors (e.g., **"Bluetooth Disabled"**, **"Connection Failed"**) with clear recovery steps.
- ♿ **Accessibility:** Ensure high color contrast for text and large touch targets (at least 48x48dp) for all interactive UI elements.
- 📭 **Empty States:** Provide helpful guidance or calls-to-action when no data is present (e.g., **"Scanning for nearby friends..."**).
  ```kotlin
  // Example: Showing a helpful empty state on Android
  if (discoveredDevices.isEmpty()) {
      statusTextView.text = "Scanning for nearby friends..."
      progressBar.visibility = View.VISIBLE
  }
  ```

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 💬 How to Use

1. 📱 **On Device A:** Tap **"Make Discoverable"** or **"Host Chat"**.
2. 🔍 **On Device B:** Scan for nearby devices and tap **"Device A's name"** to initiate a connection.
3. 💬 **Messaging:** Once connected, type your message and tap **"Send"**!

> [!TIP]
> Always provide visual feedback for connection status changes. A simple status indicator or toast can significantly reduce user frustration during transient Bluetooth interruptions.

> [!TIP]
> Bluetooth has a typical range of about 10 meters (33 feet). For the best experience, ensure devices have a clear line of sight and are not obstructed by thick walls or large metal objects.

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🤝 Contributing

Got ideas to make this app better? Pull requests are always welcome! For major changes, please open an issue first to discuss what you would like to change.

## 📄 License

This project is licensed under the [MIT License](LICENSE) - see the LICENSE file for details.

---

<a href="#bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>
