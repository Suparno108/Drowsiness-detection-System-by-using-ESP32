# 🚗 AI Driver Sleepiness Detector & Vehicle Safety System

Welcome to the **Driver Drowsiness Detection Project**! 
This project is super cool because it uses your computer's webcam to watch your eyes using Artificial Intelligence (AI). If you close your eyes for more than 3 seconds (like when someone falls asleep at the wheel), the AI tells a microcontroller (a tiny computer called an ESP32) to stop a toy car motor, ring a loud buzzer, and even make a phone call using a SIM card!

Don't worry if this is your first time building something like this. Just follow these simple steps, and you'll have it working in no time!

---

## 🛠️ What You Need (Hardware List)
You can find these parts online or at an electronics store:
1. **ESP32 Board**: The "brain" of the hardware.
2. **L298N Motor Driver**: Helps the ESP32 control the motor.
3. **DC Motor**: We'll use this to act like a car's wheels.
4. **Active Buzzer**: Makes a loud beep sound.
5. **SIM800L Module**: A tiny phone that makes emergency calls (requires a SIM card).
6. **Webcam**: Your laptop's built-in camera works perfectly!
7. **Green & Red LEDs**: Visual status and warning indicator lights.
8. **220Ω Resistors (x2)**: Current-limiting resistors to protect your LEDs and ESP32 pins.
9. **Jumper Wires & Breadboard**: To connect everything together.

---

## 🔌 Step 1: Connecting the Wires

It's just like plugging in Lego pieces! Look at the names on the pins and connect them exactly as shown below:

### 1. Connect ESP32 to the Motor Driver (L298N)
| From ESP32 Pin | Connect To L298N Pin |
| -------------- | -------------------- |
| **GPIO25**     | **IN1**              |
| **GPIO26**     | **IN2**              |
| **GPIO27**     | **ENA**              |

### 2. Connect Motor Driver (L298N) to the Motor
| From L298N Pin | Connect To Motor |
| -------------- | ---------------- |
| **OUT1**       | Motor wire 1     |
| **OUT2**       | Motor wire 2     |

### 3. Connect ESP32 to the Buzzer
| From ESP32 Pin | Connect To Buzzer |
| -------------- | ----------------- |
| **GPIO14**     | **Positive (+)** pin |
| **GND**        | **Negative (-)** pin |

### 4. Connect ESP32 to the SIM800L (The Phone Module)
| From ESP32 Pin | Connect To SIM800L |
| -------------- | ------------------ |
| **TX**         | **RX**             |
| **RX**         | **TX**             |
| **GND**        | **GND**            |

### 5. Connect ESP32 to the Status & Warning LEDs (New)
| From ESP32 Pin | Connect To | Details |
| -------------- | ---------- | ------- |
| **GPIO18**     | **Green LED Anode (+)** | Connect in series through a 220Ω resistor |
| **GPIO19**     | **Red LED Anode (+)** | Connect in series through a 220Ω resistor |
| **GND**        | **LED Cathodes (-)** | Connect both LED negative (shorter) legs to Ground |

*(⚠️ **Important Note**: The SIM800L needs a battery that provides exactly 3.7V to 4.2V to work properly! Don't power it directly from the ESP32's 3.3V pin.)*

---

## 💻 Step 2: Setting up the Software

We need to install the programs that make our project smart.

1. **Download Arduino IDE:** Go to the [Arduino Website](https://www.arduino.cc/en/software) and download the Arduino IDE. This is where we write code for the ESP32.
2. **Install the AI Brain (Python):** Open the terminal in VS Code (Go to Terminal -> New Terminal). First, tell the terminal to go inside our project folder, then install the packages by typing these two lines (press Enter after each):
   ```bash
   cd Drowsiness_Detection
   pip install -r requirements.txt
   ```
   *(Why this command? Instead of typing out every single library, Python reads the `requirements.txt` file and automatically installs all the necessary libraries at once: `opencv-python`, `mediapipe`, `pyserial`, and `scipy`!)*

---

## 🚀 Step 3: Let's Run It!

### Uploading code to the ESP32
1. Open the folder `esp32_controller` and double-click the `esp32_controller.ino` file. It will open in the Arduino IDE.
2. Connect your ESP32 to your computer using a USB cable.
3. In Arduino IDE, go to **Tools -> Board** and select your ESP32. Then go to **Tools -> Port** and select the port it's connected to (like `COM3` or `COM4`).
4. Click the right-arrow (➡️) button at the top left to **Upload** the code to the board.

### Running the AI Camera
1. Look closely at the `main.py` code. See where it says `COM3` around line 45? Change `COM3` to whatever port your ESP32 is using (the exact same one you selected in Arduino).
2. Go back to your terminal in VS Code. If you aren't already in the project folder, type this first:
   ```bash
   cd Drowsiness_Detection
   ```
3. Start the AI Camera by typing:
   ```bash
   python main.py
   ```
   *(Alternatively, you can just hit the "Play/Run" button in the top right corner of VS Code while looking at `main.py`!)*
4. Your webcam window should pop open!
5. **Test it out!** Stare directly at the camera and try closing your eyes completely for 3 full seconds. 
6. You will see **"DROWSY DETECTED"** pop up on the screen, and if your hardware is plugged in, your motor should slowly stop while the buzzer rings!
7. To turn the camera off, click on the camera window and press the **`ESC`** key.

---

## 🛰️ Geolocation Testing & Diagnostic Utility

To ensure your system is tracking your physical location with extreme precision, a standalone diagnostic script is included in your workspace: **`test_windows_gps.py`**.

### How to Run the Diagnostic:
1. Navigate to the project directory in your terminal:
   ```bash
   cd Drowsiness_Detection
   ```
2. Run the test script:
   ```bash
   python test_windows_gps.py
   ```
3. What happens?
   * It queries the native **Windows Location Services API** (using a safe background PowerShell request) to obtain high-accuracy coordinates using Wi-Fi SSID triangulation.
   * If Windows Location Services are disabled or unavailable, it instantly performs a **fallback test** using your internet IP address (`ipinfo.io`).
   * It outputs your coordinates and automatically generates a clickable Google Maps link!

---

## 🧠 How the AI Talks to the Hardware (Under the Hood)

Curious about how your laptop actually stops the motor? Here is the exact process happening behind the scenes:

1. **The Eyes (Camera & AI):** Python uses your webcam to look at your face. MediaPipe maps out exactly where your eyelids are.
2. **The GPS Location (Hybrid Engine):** When the Python script starts, it fetches your high-accuracy GPS coordinates via the Windows native `GeoCoordinateWatcher` (Wi-Fi/cellular triangulation). If Windows Location Services are disabled, it automatically falls back to coarse IP-based geolocation.
3. **The Decision:** Python constantly measures the Eye Aspect Ratio (EAR). 
   - If your eyes are open, Python sends the letter **`N`** (for Normal) through the USB cable.
   - If your eyelids touch for more than 3 seconds (Strike 1 and 2), Python sends **`W`** (Warning).
   - If you get 3 Strikes, Python sends **`E`** (Emergency) along with your precise latitude and longitude coordinates through the USB cable! (e.g., `E21.8797,87.7586`)
4. **The Communication (Serial):** The USB cable acts as a bridge (called "Serial Communication") between your big laptop and the tiny ESP32 brain.
5. **The Reaction (ESP32):** The ESP32 is always listening. 
   - When it hears **`N`**, it turns the **Green LED ON** and **Red LED OFF**, and tells the L298N Motor Driver to keep the motor spinning at full speed.
   - When it hears **`W`**, it turns the **Green LED OFF**, **Red LED ON**, and turns the buzzer on to wake you up.
   - When it hears **`E`** and the GPS data, it triggers the full emergency sequence:
     - Turns the **Green LED OFF** and **Red LED ON**.
     - Slowly reduces motor power to `0` (bringing the car to a safe stop).
     - Turns the buzzer on.
     - Uses `AT` commands to tell the SIM800L module to immediately send an SMS with a Google Maps link of your location!
     - Dials your emergency contact phone number!

---

## 🔊 Speaker & Microphone Troubleshooting (SIM800L Audio)

If you are using a **KL 8-ohm 0.5W Speaker** and a **Robozar Electret Microphone** with your SIM800L module for calling, verify the following connection rules to get them working:

### 🎤 1. Electret Microphone (Robozar 5x6mm Capsule)
Electret microphone capsules are **polarized**. If they are wired backward, they will not capture any sound.
* **Identify Polarity:** Turn the mic capsule over to see the two solder pads on the bottom. One of the pads has small copper paths physically connected to the **outer metal case**. This pad is the **Negative (-) terminal**. The isolated pad is the **Positive (+) terminal**.
* **Wiring:** 
  * Connect the **Positive (+) pad** to the **SIM800L MIC+** pin.
  * Connect the **Negative (-) pad** to the **SIM800L MIC-** pin.
* *Do not connect the microphone directly to the ESP32 analog pins (it lacks bias voltage and its output signal is only a few millivolts, requiring a preamplifier).*

### 🔈 2. Dynamic Speaker (8-ohm 0.5W)
The SIM800L has an on-board differential amplifier designed to drive a small 8-ohm speaker.
* **Wiring:** 
  * Connect one speaker wire to the **SIM800L SPK+** pin.
  * Connect the other speaker wire to the **SIM800L SPK-** pin.
* ⚠️ **WARNING:** Never connect `SPK-` to the system ground (`GND`) or any other common ground rail. The SIM800L SPK outputs are differential; grounding one of them will short-circuit the internal audio amplifier and permanently destroy the audio driver of the SIM800L.
* *Do not connect an 8-ohm speaker directly to an ESP32 GPIO pin. At 3.3V, an 8-ohm speaker draws over 400mA, which is 10 times the safe limit (40mA) of the ESP32 pin and will fry the microcontroller.*

---

**🎉 You did it! You built a working AI safety system! 🎉**
