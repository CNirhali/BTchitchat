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
  // Example: Requesting larger MTU on Android.
  // Using 517 allows for the maximum 512-byte payload plus 5-byte header.
  bluetoothGatt.requestMtu(517)
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
  // Example: Stopping discovery immediately upon connection on Android.
  // Using autoConnect=false and TRANSPORT_LE avoids the ~2s background scan delay
  // and dual-mode (BR/EDR) overhead for faster initial connection.
  fun onDeviceSelected(device: BluetoothDevice) {
      bluetoothLeScanner.stopScan(scanCallback)
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
  // Example: Writing without response for ~2x throughput increase on Android (API 33+).
  // This method avoids internal memory copies, improving throughput and efficiency.
  val data = message.toByteArray()
  bluetoothGatt.writeCharacteristic(characteristic, data, BluetoothGattCharacteristic.WRITE_TYPE_NO_RESPONSE)
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
      // Called when the internal buffer has cleared. To maximize BLE throughput,
      // use a 'while' loop to saturate the write buffer as long as it's ready.
      // This reduces delegate callback overhead by ~2-5x for multiple messages.
      while peripheral.canSendWriteWithoutResponse && hasPendingMessages {
          sendNextPendingMessage()
      }
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
  ```kotlin
  // Example: Deferring Bluetooth manager lookup on Android.
  // Using 'LazyThreadSafetyMode.NONE' avoids synchronization overhead
  // when initialization is guaranteed to occur on a single thread (e.g., Main).
  private val bluetoothManager by lazy(LazyThreadSafetyMode.NONE) {
      context.getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
  }
  ```
  ```swift
  // Example: Deferring CBCentralManager initialization in Swift.
  // Using 'lazy var' ensures the Bluetooth stack is only powered on
  // when the 'centralManager' property is first accessed.
  lazy var centralManager: CBCentralManager = {
      let queue = DispatchQueue(label: "com.app.bluetooth", qos: .userInitiated)
      return CBCentralManager(delegate: self, queue: queue)
  }()
  ```
- 📜 **List Virtualization:** Use virtualized lists to handle large chat histories without degrading UI performance. This ensures only visible items are rendered, maintaining 60 FPS even with thousands of messages.
  ```kotlin
  // Example: Using ListAdapter with DiffUtil for efficient RecyclerView updates on Android.
  // This automatically calculates the difference between lists on a background thread
  // and only updates the changed items, preventing expensive full-list re-renders.
  class ChatAdapter : ListAdapter<ChatMessage, ChatViewHolder>(ChatDiffCallback()) {
      override fun onBindViewHolder(holder: ChatViewHolder, position: Int) {
          holder.bind(getItem(position))
      }
  }

  class ChatDiffCallback : DiffUtil.ItemCallback<ChatMessage>() {
      override fun areItemsTheSame(oldItem: ChatMessage, newItem: ChatMessage) = oldItem.timestamp == newItem.timestamp
      override fun areContentsTheSame(oldItem: ChatMessage, newItem: ChatMessage) = oldItem == newItem
  }
  ```
  ```swift
  // Example: Using UICollectionViewDiffableDataSource in Swift.
  // This provides high-performance, crash-safe list updates by managing
  // snapshots and applying only the necessary changes to the UI.
  var dataSource: UICollectionViewDiffableDataSource<Int, ChatMessage>!

  func updateMessages(_ messages: [ChatMessage]) {
      var snapshot = NSDiffableDataSourceSnapshot<Int, ChatMessage>()
      snapshot.appendSections([0])
      snapshot.appendItems(messages)
      dataSource.apply(snapshot, animatingDifferences: true)
  }
  ```
  ```tsx
  // Example: Performance-tuned FlatList in React Native (TSX).
  // 'initialNumToRender', 'windowSize', and 'maxToRenderPerBatch' control
  // the virtualization engine to balance memory usage and scroll smoothness.
  // Using 'memo' for 'renderItem' components prevents redundant re-renders of off-screen items.
  const ChatMessageComponent = React.memo(({ message }: { message: ChatMessage }) => (
    <View>/* ... render message ... */</View>
  ));

  <FlatList
    data={messages}
    keyExtractor={(item) => item.timestamp.toString()}
    renderItem={({ item }) => <ChatMessageComponent message={item} />}
    initialNumToRender={15}
    windowSize={5}
    maxToRenderPerBatch={10}
    removeClippedSubviews={true}
  />
  ```
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
      .setMatchMode(ScanSettings.MATCH_MODE_AGGRESSIVE)
      .setNumOfMatches(ScanSettings.MATCH_NUM_ONE_ADVERTISEMENT)
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
  // Example: Initializing CBCentralManager with a background queue in Swift.
  // This ensures all delegate callbacks and GATT operations occur off the main thread,
  // preventing UI jank and maintaining 60 FPS responsiveness.
  let bluetoothQueue = DispatchQueue(label: "com.app.bluetooth", qos: .userInitiated)
  let centralManager = CBCentralManager(delegate: self, queue: bluetoothQueue)
  ```

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#-bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🔒 Security

Bluetooth communication is inherently susceptible to various security risks, including eavesdropping and man-in-the-middle attacks. For detailed security practices, see our [SECURITY.md](SECURITY.md).

- 🔐 **Encryption & Integrity:** This template recommends and provides implementation examples for **Authenticated Encryption with Associated Data (AEAD)** (e.g., AES-GCM) and **Message Integrity Checks** to protect against eavesdropping and tampering. See [SECURITY.md](SECURITY.md) for actionable code snippets.
- 💡 **Recommendations:** For production use, it is highly recommended to implement a robust End-to-End Encryption (E2EE) layer with **Perfect Forward Secrecy (PFS)** using libraries like [Noise Protocol](https://noiseprotocol.org/) or [libsodium](https://doc.libsodium.org/).
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
- 🗨️ **Message Bubbles:** Use distinct alignment and colors to differentiate between sent and received messages, and include an `accessibilityLabel` to announce the sender to screen readers.
  ```tsx
  // Example: Accessible message bubbles in React Native (TSX)
  <View
    style={[styles.bubble, isSent ? styles.sent : styles.received]}
    accessibilityLabel={`${isSent ? 'Sent' : 'Received'} message: ${message.text}`}
  >
    <Text>{message.text}</Text>
  </View>
  ```
- ⌨️ **Keyboard Interactions:** Support standard keyboard behaviors like **"Enter to Send"** to improve efficiency for power users and ensure accessibility for keyboard-based navigation.
  ```kotlin
  // Example: Supporting 'Enter to Send' on Android.
  // Requires 'android:imeOptions="actionSend"' in the XML layout.
  messageEditText.setOnEditorActionListener { _, actionId, _ ->
      if (actionId == EditorInfo.IME_ACTION_SEND) {
          val text = messageEditText.text.toString()
          if (text.isNotBlank()) {
              sendMessage(text)
              messageEditText.text.clear()
          }
          true
      } else false
  }
  ```
  ```swift
  // Example: Supporting 'Enter to Send' in Swift (UIKit).
  // Set 'returnKeyType = .send' and 'enablesReturnKeyAutomatically = true' to prevent empty sends.
  func textFieldShouldReturn(_ textField: UITextField) -> Bool {
      if textField == messageTextField, let text = textField.text, !text.trimmingCharacters(in: .whitespaces).isEmpty {
          sendMessage(text)
          textField.text = ""
          return false // Prevent default behavior and keep focus
      }
      return true
  }
  ```
  ```tsx
  // Example: Supporting 'Enter to Send' in React Native (TSX).
  // 'enablesReturnKeyAutomatically' prevents sending empty messages from the keyboard.
  <TextInput
    ref={inputRef}
    placeholder="Type a message..."
    returnKeyType="send"
    enablesReturnKeyAutomatically={true}
    onSubmitEditing={({ nativeEvent: { text } }) => {
      if (text.trim()) {
        sendMessage(text);
        inputRef.current?.clear();
      }
    }}
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
- 🗑️ **Destructive Actions:** Always require confirmation for irreversible actions like clearing chat history or disconnecting an active session to prevent accidental data loss.
  ```kotlin
  // Example: Confirmation for clear chat on Android
  MaterialAlertDialogBuilder(context)
      .setTitle("Clear Chat History?")
      .setMessage("This action cannot be undone.")
      .setPositiveButton("Clear") { _, _ -> clearChat() }
      .setNegativeButton("Cancel", null)
      .show()
  ```
  ```swift
  // Example: Confirmation for clear chat in Swift (UIKit)
  let actionSheet = UIAlertController(title: "Clear Chat History?", message: "This action cannot be undone.", preferredStyle: .actionSheet)
  actionSheet.addAction(UIAlertAction(title: "Clear", style: .destructive) { _ in
      self.clearChat()
  })
  actionSheet.addAction(UIAlertAction(title: "Cancel", style: .cancel))
  present(actionSheet, animated: true)
  ```
  ```tsx
  // Example: Confirmation for clear chat in React Native (TSX)
  Alert.alert(
    "Clear Chat History?",
    "This action cannot be undone.",
    [
      { text: "Cancel", style: "cancel" },
      { text: "Clear", style: "destructive", onPress: () => clearChat() }
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
  // Using 'accessibilityState' and dynamic 'style' provides visual and auditory feedback.
  <Pressable
    accessibilityRole="button"
    accessibilityLabel="Send Message"
    accessibilityState={{ disabled: isSending }}
    hitSlop={12}
    onPress={sendMessage}
    disabled={isSending}
    style={({ pressed }) => [
      { opacity: pressed || isSending ? 0.5 : 1 },
      styles.sendButton
    ]}
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
- 📭 **Empty States:** Provide helpful guidance or calls-to-action when no data is present (e.g., **"Scanning for nearby friends..."**). Include a manual **"Scan Again"** or **"Retry"** button to allow users to recover from transient discovery failures.

  ```kotlin
  // Example: Showing a helpful empty state with a retry action on Android.
  // Using 'performHapticFeedback' and 'contentDescription' ensures the action
  // is both tactile and accessible for all users.
  if (discoveredDevices.isEmpty()) {
      statusTextView.text = "No devices found nearby."
      progressBar.visibility = View.GONE
      retryButton.apply {
          visibility = View.VISIBLE
          text = "Scan Again"
          contentDescription = "Scan for nearby Bluetooth devices"
          setOnClickListener {
              it.performHapticFeedback(HapticFeedbackConstants.VIRTUAL_KEY)
              startDiscovery()
          }
      }
  }
  ```
  ```swift
  // Example: Showing a helpful empty state with a retry action in Swift.
  // Using 'impactOccurred' and 'accessibilityHint' provides tactile feedback
  // and additional context for VoiceOver users.
  if discoveredDevices.isEmpty {
      statusLabel.text = "No devices found nearby."
      activityIndicator.stopAnimating()
      retryButton.isHidden = false
      retryButton.setTitle("Scan Again", for: .normal)
      retryButton.accessibilityHint = "Restarts the Bluetooth device discovery process"
      retryButton.addAction(UIAction { _ in
          UIImpactFeedbackGenerator(style: .medium).impactOccurred()
          startScanning()
      }, for: .touchUpInside)
  }
  ```
  ```tsx
  // Example: Showing a helpful empty state with a retry action in React Native (TSX).
  // 'hitSlop' expands the touch area, while 'Vibration' and 'style' provide feedback.
  {discoveredDevices.length === 0 && (
    <View style={styles.emptyState}>
      <Text>No devices found nearby.</Text>
      <Pressable
        accessibilityRole="button"
        accessibilityLabel="Scan Again"
        accessibilityHint="Restarts the Bluetooth device discovery process"
        onPress={() => {
          Vibration.vibrate(10);
          startScanning();
        }}
        hitSlop={12}
        style={({ pressed }) => [
          { opacity: pressed ? 0.6 : 1 },
          styles.retryButton
        ]}
      >
        <Text>Scan Again</Text>
      </Pressable>
    </View>
  )}
  ```

### 🎨 UI/UX Checklist

A quick reference for developers to ensure the "interface" meets our standard for quality and accessibility.

| Check | Guideline | Recommendation |
| :---: | :--- | :--- |
| [ ] | **Connection Status** | 🟢 Connected, 🟡 Connecting..., 🔴 Disconnected |
| [ ] | **Touch Targets** | Minimum 48x48dp for all buttons |
| [ ] | **Destructive Actions** | Confirmation dialog before clearing chat |
| [ ] | **Screen Readers** | Descriptive `aria-label` or `contentDescription` |
| [ ] | **Haptics** | Tactile feedback on message sent/delivered |
| [ ] | **Keyboard** | "Enter to Send" supported with auto-clear |
| [ ] | **Message Bubbles** | Sent messages on right, received on left |
| [ ] | **Empty States** | Manual "Scan Again" button for recovery |

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
