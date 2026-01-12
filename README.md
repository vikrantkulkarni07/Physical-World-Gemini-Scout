# 🤖 Gemini Scout: The Embodied AI Rover

[![Gemini 3 API](https://img.shields.io/badge/AI-Gemini%203%20Flash-blue?style=for-the-badge&logo=google)](https://deepmind.google/technologies/gemini/)
[![Python](https://img.shields.io/badge/Brain-Python-yellow?style=for-the-badge&logo=python)](https://www.python.org/)
[![ESP32](https://img.shields.io/badge/Hardware-ESP32--CAM-green?style=for-the-badge&logo=arduino)](https://www.espressif.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **"We gave Gemini wheels."**
> An autonomous rover that uses **Gemini 3 Flash** as its visual cortex. It sees, reasons about physics, and navigates the real world without hard-coded logic.

---

## 📖 Table of Contents
- [How It Works](#-how-it-works)
- [Hardware Required](#-hardware-required)
- [Wiring & Pinout](#-wiring--pinout)
- [Installation Guide](#-installation-guide)
  - [1. Firmware (ESP32)](#1-firmware-esp32)
  - [2. The Brain (Python)](#2-the-brain-python)
- [Running the Rover](#-running-the-rover)
- [The System Prompt](#-the-system-prompt)

---

## 🧠 How It Works
Gemini Scout moves away from traditional navigation stacks (LIDAR, SLAM). Instead, it uses a **Sense-Think-Act** loop powered by a Multimodal LLM:

1.  **Sense (Edge):** The ESP32-CAM streams MJPEG video over Wi-Fi.
2.  **Think (Cloud):** A Python client grabs frames and sends them to **Gemini 3 Flash**.
    * *Input:* Image + Context Prompt ("You are a rover...")
    * *Reasoning:* Gemini analyzes terrain safety, obstacles, and goals.
    * *Output:* A structured JSON command (e.g., `{"move": "FORWARD", "speed": 200}`).
3.  **Act (Edge):** The Python client triggers the ESP32 motors via HTTP requests.

---

## 🛠 Hardware Required
* **Microcontroller:** ESP32-CAM (AI-Thinker Model)
* **Motor Driver:** L298N Dual H-Bridge
* **Chassis:** 2WD or 4WD Robot Car Kit
* **Power:** 2x 18650 Li-ion Batteries (7.4V total)
* **FTDI Adapter:** For programming the ESP32-CAM

---

## 🔌 Wiring & Pinout

### L298N Driver -> ESP32-CAM
| L298N Pin | ESP32-CAM GPIO | Description |
| :--- | :--- | :--- |
| **IN1** | GPIO 12 | Motor A Direction 1 |
| **IN2** | GPIO 13 | Motor A Direction 2 |
| **IN3** | GPIO 15 | Motor B Direction 1 |
| **IN4** | GPIO 14 | Motor B Direction 2 |
| **ENA** | 5V (Jumper) | Speed Control (Optional PWM) |
| **ENB** | 5V (Jumper) | Speed Control (Optional PWM) |

> **Note:** GPIO 0 must be connected to GND only while flashing code. Remove it to run.

---

## 💻 Installation Guide

### 1. Firmware (ESP32)
1.  Navigate to the `/firmware` folder in this repo.
2.  Open `Gemini_Scout_Firmware.ino` in the **Arduino IDE**.
3.  Install the ESP32 Board Manager:
    * `File` -> `Preferences` -> Add `https://dl.espressif.com/dl/package_esp32_index.json`
    * `Tools` -> `Board` -> `ESP32 Arduino` -> Select **AI Thinker ESP32-CAM**.
4.  Modify lines 20-21 with your Wi-Fi details:
    ```cpp
    const char* ssid = "YOUR_WIFI_NAME";
    const char* password = "YOUR_WIFI_PASSWORD";
    ```
5.  Upload the code. Open the Serial Monitor (Baud 115200) to see the **IP Address** (e.g., `192.168.1.105`).

### 2. The Brain (Python)
1.  Clone this repository:
    ```bash
    git clone [https://github.com/your-username/gemini-scout.git](https://github.com/your-username/gemini-scout.git)
    cd gemini-scout
    ```
2.  Create a virtual environment and install dependencies:
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows: venv\Scripts\activate
    pip install -r requirements.txt
    ```
3.  Get your **Gemini API Key** from [Google AI Studio](https://aistudio.google.com/).
4.  Create a `.env` file:
    ```env
    GEMINI_API_KEY=your_actual_api_key_here
    ROVER_IP=192.168.1.105  # The IP from the Arduino Serial Monitor
    ```

---

## 🚀 Running the Rover

1.  Power on the Rover (Battery Switch ON).
2.  Run the control script:
    ```bash
    python controller.py
    ```
3.  A window will pop up showing the **"Robot Vision"**.
4.  Watch the terminal! You will see Gemini's "Thoughts" in real-time:
    > 🤖 **Gemini:** "I see a chair leg ahead. The path to the right is clear carpet. Turning RIGHT."
    > 🚀 **Action:** RIGHT (Speed 180)

---

## 📝 The System Prompt
The magic happens in how we talk to the model. Here is the core prompt used in `controller.py`:

```text
You are the visual cortex of a small robot. 
Analyze the image for navigation.
1. Identify immediate obstacles (walls, legs, drops).
2. Identify surface texture (carpet = safe, tile = slippery).
3. Choose a command: FORWARD, LEFT, RIGHT, BACK, STOP.

Return ONLY JSON:
{
  "reasoning": "Brief explanation of what you see and why.",
  "command": "FORWARD",
  "speed": 200
}
