# Pico-Cast 📺

**Pico-Cast** is a highly optimized media streaming ecosystem designed specifically for the **Raspberry Pi Pico 2W** paired with the **Olimex RP2040-PICO-PC Carrier Board**. It turns your Pico 2W into a wireless HDMI receiver, streaming synchronized video and audio directly from your PC over Wi-Fi. 

It uses FFmpeg to dynamically compress, scale, and dither video in real-time, resulting in a gorgeous, cinematic viewing experience on any HDMI monitor.

This project is deeply tailored to this specific hardware combination. It uses a custom drift-free absolute-time Python UDP sender and a dual-core C++ receiver with anti-burst UDP protection to push the Pico 2W's RP2350 chip to its absolute limits.

---

## ⚠️ Important Hardware Requirements
This codebase is **NOT** meant to run on generic setups. It is strictly designed for the following exact hardware combination:

1. **Raspberry Pi Pico 2W**: The code relies heavily on the power of the new chip and the onboard CYW43 Wi-Fi module. A standard Pico 2 (without Wi-Fi) or an older Pico W will not work optimally.
2. **Olimex RP2040-PICO-PC Carrier Board**: This board acts as a carrier/docking station for the Pico 2W. It provides the physical HDMI connector perfectly routed to the correct GPIO pins, as well as an I2S Audio DAC.

*Note: While it is theoretically possible to manually solder an HDMI breakout board and an I2S DAC directly to a standalone Pico 2W using the exact pinout below, it is highly unergonomic.*

---

## 🚀 Features
- **Synchronized Video & Audio**: Streams high-framerate video and 44.1kHz PCM Audio completely synchronized over UDP.
- **Zero-Delay Seeking & Scrubbing**: Fully modern, dark-themed Media Player GUI (`customtkinter`) for PC.
- **Zero CPU Footprint**: Uses Windows High-Resolution OS Timers (`timeBeginPeriod`) for 1ms micro-pacing, dropping Python CPU usage to near 0%.
- **Anti-Burst UDP Shield**: C++ Receiver uses 32-bit chunk masks and ring buffers to prevent lwIP packet duplication and drop-outs during rapid scrubbing.
- **Dynamic Playback Speed**: Change playback speed on the fly (0.5x to 2.0x) using pure FFmpeg `atempo` and `setpts` filters, ensuring network bandwidth remains strictly at 1x hardware speed to prevent buffer overflows.
- **Bayer Dithering & Unsharp Mask**: Eliminates color banding for a "Millions of Colors" look on a 16-bit framebuffer while maintaining pristine crispness.

---

## 🛠 Hardware Specifications & Pinouts

- **Microcontroller**: Pico 2W (Overclocked to 252 MHz at 1.25V).
- **RAM Constraint**: Decoding 320x240 directly into memory requires ~153KB for the framebuffer and 70KB for Ping-Pong JPEG buffers.
- **Audio Constraint**: Dedicated 2048-sample ring buffer for seamless I2S streaming over HDMI.

### Olimex Carrier Board DVI/I2S Pinout
The Olimex RP2040-PICO-PC routes the Pico's pins to the HDMI port as follows:
| Signal | Pico 2W GPIO Pin |
|--------|------------------|
| TMDS D0| 14               |
| TMDS D1| 18               |
| TMDS D2| 16               |
| CLK    | 12               |
| I2S DIN| 26               |
| I2S BCK| 27               |
| I2S LRC| 28               |

---

## ⚙️ Installation & Setup

### 1. Requirements for PC (Windows)
1. **Python 3.10+**: Make sure Python is installed.
2. **FFmpeg**: The core media engine.
   - **The easiest way to install FFmpeg on Windows** is using the built-in Windows Package Manager (`winget`). Open your Terminal or PowerShell and run:
     ```powershell
     winget install ffmpeg
     ```
     *(Restart your terminal after installation so the `ffmpeg` command is recognized).*
3. **Python Libraries**:
   Open a terminal and run:
   ```bash
   pip install customtkinter
   ```

### 2. Flashing the Pico 2W

You have two options to install the software on your Pico 2W. **Method A (Arduino IDE)** is highly recommended for beginners as it requires no complex toolchains.

#### Method A: Arduino IDE (Recommended & Easiest)
This method allows you to compile and upload the code using the friendly Arduino interface.

1. **Install Arduino IDE**: Download and install the [Arduino IDE](https://www.arduino.cc/en/software).
2. **Add RP2040/RP2350 Boards**: 
   - Open Arduino IDE, go to **File -> Preferences**.
   - In the "Additional Boards Manager URLs" field, paste this URL:
     `https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json`
   - Go to **Tools -> Board -> Boards Manager**, search for `pico` and install **Raspberry Pi Pico/RP2040/RP2350 by Earle F. Philhower, III**.
3. **Open the Project**:
   - Navigate to the `pico_receiver-arduino` folder in this repository.
   - Double click `pico_receiver-arduino.ino` to open it in the Arduino IDE.
4. **Enter Wi-Fi Credentials**:
   - In the code, look at line 28:
     ```cpp
     const char* WIFI_SSID = "YOUR_WIFI_SSID";
     const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
     ```
   - Change `"YOUR_WIFI_SSID"` and `"YOUR_WIFI_PASSWORD"` to your home Wi-Fi details. Keep the quotes!
5. **CRITICAL IDE Settings**: You must configure the Arduino Tools menu exactly like this for the video stream to work. Go to **Tools** and set:
   - **Board**: `Raspberry Pi Pico 2W`
   - **CPU Speed**: `250 MHz (Overclock)` *(Required for 640x480 DVI timing)*
   - **Optimize**: `Optimize Even More (-O3)` *(Required to prevent video lagging/red lines)*
   - **IP/Bluetooth Stack**: `IPv4 Only`
6. **Upload**:
   - Hold the **BOOTSEL** button on your Pico 2W and plug it into your computer via USB.
   - Select the new UF2 Port in **Tools -> Port**.
   - Click the **Upload** (Right Arrow) button at the top left of the IDE.
   - Once it says "Done uploading", plug the Pico 2W into the Olimex carrier board and connect to your TV!

#### Method B: C++ SDK & CMake (Advanced)
If you prefer raw C++ toolchains:
1. Download the [Pico Setup for Windows](https://github.com/raspberrypi/pico-setup-windows/releases) installer.
2. Enter your credentials in `pico_receiver/main.cpp`:
   ```cpp
   #define WIFI_SSID "YOUR_WIFI_SSID"
   #define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"
   ```
3. Open "Pico - Developer Command Prompt", navigate to `pico_receiver`, and run:
   ```bash
   mkdir build
   cd build
   cmake ..
   ninja
   ```
4. Hold **BOOTSEL** on the Pico 2W, plug it in, and drag & drop `pico_receiver.uf2` into the mounted USB drive.

---

## 🎬 How to Use the Python GUI
Once your Pico 2W is connected to your TV and shows its IP address (e.g., `192.168.1.100`), you are ready to stream!

1. Open a terminal on your PC.
2. Navigate to the streamer directory:
   ```bash
   cd pc_streamer
   ```
3. Run the Media Player GUI:
   ```bash
   python stream.py
   ```
4. **Enter the Pico's IP address** into the top box.
5. Click **Connect**.
6. **Browse** and select any video file (.mp4, .mkv, .avi, etc.).
7. Click **Play** and enjoy the cinematic stream on your TV!

---

## 🧩 Architecture Flowcharts

### High-Level System Workflow
```mermaid
graph LR
    A[Video File] -->|Read| B(FFmpeg Pipeline)
    B -->|Scale, Dither, JPEG, PCM| C[Python Streamer]
    C -->|UDP Packets Ports 8080 & 8081| D((Wi-Fi Router))
    D -->|Wireless| E[Pico 2W on Olimex Carrier]
    E -->|Decode, I2S & DVI| F[HDMI Display]
```
