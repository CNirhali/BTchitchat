# 📱 Bluetooth Chit Chat

A lightweight, offline messaging app that lets you chat with nearby friends using Bluetooth, no internet or cell service required! Perfect for classrooms, camping trips, or crowded events where the Wi-Fi is weak or nonexistent.

## 📋 Table of Contents

- [✨ Features](#features)
- [📸 Screenshots](#screenshots)
- [🛠 Tech Stack](#tech-stack)
- [🚀 Getting Started](#getting-started)
  - [🛠️ Prerequisites](#prerequisites)
  - [📦 Installation](#installation)
- [🔒 Security](#security)
- [💬 How to Use](#how-to-use)
- [🤝 Contributing](#contributing)
- [📄 License](#license)

## ✨ Features

- **100% Offline Messaging:** Send and receive text messages using a direct Bluetooth peer-to-peer connection.
- **Quick Device Pairing:** Easily discover and connect to nearby Bluetooth-enabled devices directly from the app.
- **Real-Time Chat Interface:** A familiar, intuitive chat UI with message bubbles and connection status updates.
- **No Accounts Needed:** Just open the app, connect, and start chatting.

## 📸 Screenshots

| Device Discovery | Active Chat Room |
| :---: | :---: |
| ![A mobile screen showing a list of discovered nearby Bluetooth devices with names like 'Nexus 5X' and 'Pixel 4a'](https://via.placeholder.com/300x600?text=Discovery+UI) | ![A chat conversation between two users with blue and gray message bubbles](https://via.placeholder.com/300x600?text=Chat+UI) |

## 🛠 Tech Stack

This template can be implemented using any mobile technology stack with Bluetooth support:
- **Native:** Android (Kotlin) or iOS (Swift)
- **Cross-platform:** Flutter or React Native
- **Connectivity:** Peer-to-peer Bluetooth communication

## 🚀 Getting Started

### 🛠️ Prerequisites

> [!IMPORTANT]
> This app requires **two physical devices** to test Bluetooth connectivity. Simulators or emulators typically do not support Bluetooth hardware.

- [ ] **Bluetooth enabled** in the system settings of both devices.
- [ ] **Compatible Mobile OS:** Android 8.0+ or iOS 13.0+ (recommended).
- [ ] **Permissions:** Location and Nearby Devices permissions must be granted for discovery.

### 📦 Installation

1. **Clone this repository** to your local machine:
   ```bash
   git clone https://github.com/yourusername/bluetooth-chit-chat.git
   ```
2. **Install dependencies** using `pnpm`:
   ```bash
   pnpm install
   ```
3. **Open the project** in [Insert your IDE, e.g., Android Studio / Xcode / VS Code].
4. **Build and run the application** on your physical devices.

## 🔒 Security

Bluetooth communication is inherently susceptible to various security risks, including eavesdropping and man-in-the-middle attacks. For detailed security practices, see our [SECURITY.md](SECURITY.md).

- 🔐 **Encryption & Integrity:** This template currently does **not** implement End-to-End Encryption (E2EE) or Message Integrity Checks. All messages are sent in plain text and are susceptible to tampering.
- 💡 **Recommendations:** For production use, it is highly recommended to implement a robust E2EE layer with **Perfect Forward Secrecy (PFS)** using libraries like [Noise Protocol](https://noiseprotocol.org/) or [libsodium](https://doc.libsodium.org/).
- 🕵️ **Privacy:** Be mindful of the data shared over Bluetooth, as nearby devices may be able to monitor the traffic if not properly secured.

## 💬 How to Use

1. 📱 **On Device A:** Tap "Make Discoverable" or "Host Chat".
2. 🔍 **On Device B:** Scan for nearby devices and tap Device A's name to initiate a connection.
3. ✉️ **Once connected:** Type your message and hit send!

## 🤝 Contributing

Got ideas to make this app better? Pull requests are always welcome! For major changes, please open an issue first to discuss what you would like to change.

## 📄 License

This project is licensed under the [MIT License](LICENSE) - see the LICENSE file for details.

---

[⬆ Back to Top](#bluetooth-chit-chat)
