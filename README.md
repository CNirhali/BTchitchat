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
<!-- ⚡ Optimization: decoding="async" reduces main-thread contention during image decoding, improving initial rendering speed -->
| <img src="https://via.placeholder.com/300x600?text=Discovery+UI" width="300" height="600" alt="A mobile screen showing a list of discovered nearby Bluetooth devices with names like 'Nexus 5X' and 'Pixel 4a'" loading="lazy" decoding="async"> | <img src="https://via.placeholder.com/300x600?text=Chat+UI" width="300" height="600" alt="A chat conversation between two users with blue and gray message bubbles" loading="lazy" decoding="async"> |

## 🛠️ Tech Stack

This template can be implemented using any mobile technology stack with Bluetooth support:
- 📱 **Native:** [Android (Kotlin)](https://developer.android.com/develop/connectivity/bluetooth) or [iOS (Swift)](https://developer.apple.com/documentation/corebluetooth)
- 🌐 **Cross-platform:** [Flutter](https://pub.dev/packages/flutter_blue_plus) or [React Native](https://github.com/dotintent/react-native-ble-plx)
- 📶 **Connectivity:** Peer-to-peer Bluetooth communication

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

## ⚡ Performance

Bluetooth throughput is limited and latency can vary. To ensure a fast experience:
- 📦 **Binary Serialization:** Use efficient formats like [Protobuf](https://protobuf.dev/) or [FlatBuffers](https://google.github.io/flatbuffers/) to minimize payload size and processing overhead.
- 🚀 **MTU Negotiation:** Request a larger Maximum Transmission Unit (MTU) to increase throughput for larger messages (up to 512 bytes on BLE).
- ⚡ **Message Batching:** If sending multiple updates, batch them into a single Bluetooth packet to reduce protocol overhead.
- 🔋 **Battery Efficiency:** Disable Bluetooth discovery/scanning immediately after connection to save power and improve connection stability.
- 📉 **Lower Latency:** Use direct connection handles where possible and minimize unnecessary application-layer acknowledgments.
- ♻️ **Object Pooling:** Reuse byte buffers and message objects to minimize Garbage Collection (GC) overhead and prevent UI jank during high-frequency data exchange.
- ⏱️ **Lazy Initialization:** Delay Bluetooth stack setup and discovery until strictly necessary to improve initial app launch speed and reduce memory footprint.

<!-- ⚡ Optimization: Contextual 'Back to Top' links reduce developer 'Time to Action' by minimizing scroll time -->
<a href="#bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>

## 🔒 Security

Bluetooth communication is inherently susceptible to various security risks, including eavesdropping and man-in-the-middle attacks. For detailed security practices, see our [SECURITY.md](SECURITY.md).

- 🔐 **Encryption & Integrity:** This template currently does **not** implement End-to-End Encryption (E2EE) or Message Integrity Checks. All messages are sent in plain text and are susceptible to tampering.
- 💡 **Recommendations:** For production use, it is highly recommended to implement a robust E2EE layer with **Perfect Forward Secrecy (PFS)** using libraries like [Noise Protocol](https://noiseprotocol.org/) or [libsodium](https://doc.libsodium.org/).
- 🕵️ **Privacy:** Be mindful of the data shared over Bluetooth, as nearby devices may be able to monitor the traffic if not properly secured.

## 💬 How to Use

1. 📱 **On Device A:** Tap **"Make Discoverable"** or **"Host Chat"**.
2. 🔍 **On Device B:** Scan for nearby devices and tap **Device A's name** to initiate a connection.
3. ✉️ **Once connected:** Type your message and hit **Send**!

## 🤝 Contributing

Got ideas to make this app better? Pull requests are always welcome! For major changes, please open an issue first to discuss what you would like to change.

## 📄 License

This project is licensed under the [MIT License](LICENSE) - see the LICENSE file for details.

---

<a href="#bluetooth-chit-chat" aria-label="Back to top of page">⬆ Back to Top</a>
