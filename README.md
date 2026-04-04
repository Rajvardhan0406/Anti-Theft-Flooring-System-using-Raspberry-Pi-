# Anti-Theft-Flooring-System-using-Raspberry-Pi-
A Raspberry Pi-based anti-theft flooring system that detects unauthorized movement via pressure sensors and triggers real-time alerts.

📖 About the Project
This project implements an intelligent anti-theft flooring system that uses pressure sensors embedded beneath a floor mat or tiles to detect when someone steps on a protected area. The Raspberry Pi processes the sensor data in real time and triggers configurable alerts when unauthorized movement is detected.
It is ideal for:

🏪 Shops and retail stores
🏠 Home security
🏛️ Museums and exhibition halls
🏢 Server rooms and restricted zones


✨ Features

✅ Real-time pressure/foot traffic detection
✅ Instant alert via buzzer and LED indicator
✅ Configurable sensitivity and detection threshold
✅ Runs headlessly on Raspberry Pi (no monitor needed)
✅ Lightweight Python script with low CPU usage
✅ Easy to set up and deploy
✅ Expandable — supports multiple sensor zones


🏗️ System Architecture
[Pressure Sensor / FSR]
         |
         ▼
  [Analog-to-Digital
   Converter (MCP3008)]
         |
         ▼
  [Raspberry Pi GPIO]
         |
    _____|_____
   |           |
   ▼           ▼
[Buzzer]    [LED]
(Alert)    (Indicator)

🔧 Hardware Requirements
ComponentQuantityDescriptionRaspberry Pi1Model 3B / 4B / Zero WForce Sensitive Resistor (FSR)1–4Pressure sensor for floor detectionMCP3008 ADC18-channel 10-bit Analog-to-Digital ConverterBuzzer1Active buzzer for alert soundLED (Red)1Visual alert indicatorResistors2–410kΩ pull-down resistorsBreadboard1For prototyping connectionsJumper WiresSeveralMale-to-female and male-to-malePower Supply15V micro-USB / USB-C for Raspberry Pi

💻 Software Requirements

Raspberry Pi OS (Bullseye or later)
Python 3.7+
RPi.GPIO library
spidev library (for MCP3008 ADC)


🔌 Circuit Diagram
FSR Sensor ──────── MCP3008 (CH0)
                        │
MCP3008 ─── SPI ────── Raspberry Pi GPIO
                        │
                   ┌────┴────┐
                Buzzer      LED
               (GPIO 17)  (GPIO 27)
                   │          │
                  GND        GND

📌 Detailed wiring diagram image can be found in the /docs/circuit_diagram.png file.


⚙️ Installation
1. Clone the Repository
bashgit clone https://github.com/your-username/anti-theft-flooring-system.git
cd anti-theft-flooring-system
2. Enable SPI on Raspberry Pi
bashsudo raspi-config
# Go to: Interface Options → SPI → Enable
3. Install Dependencies
bashpip3 install -r requirements.txt
Or install manually:
bashpip3 install RPi.GPIO spidev
4. Connect the Hardware
Follow the circuit diagram above to connect the FSR sensor, MCP3008, buzzer, and LED to the Raspberry Pi GPIO pins.

▶️ Usage
Run the system:
bashpython3 main.py
Run with custom threshold:
bashpython3 main.py --threshold 500
Run in silent mode (LED only, no buzzer):
bashpython3 main.py --silent
Stop the system:
Press Ctrl + C to safely stop the program.

🧠 How It Works

Detection — The FSR (Force Sensitive Resistor) sensor is placed beneath the floor mat. When someone steps on it, its resistance changes.
Analog Reading — The MCP3008 ADC converts the analog signal from the FSR into a digital value readable by the Raspberry Pi via SPI.
Processing — The Python script continuously reads the sensor value and compares it against a configurable threshold.
Alert — If the value exceeds the threshold (i.e., pressure detected), the system:

Activates the buzzer for an audible alarm
Turns on the LED for a visual indicator
Logs the event with a timestamp


Reset — Once pressure is removed, the system resets and continues monitoring.


📁 Project Structure
anti-theft-flooring-system/
│
├── main.py               # Main script to run the system
├── sensor.py             # FSR sensor reading logic via MCP3008
├── alert.py              # Buzzer and LED alert control
├── config.py             # Configuration (GPIO pins, threshold, etc.)
├── requirements.txt      # Python dependencies
├── README.md             # Project documentation
│
└── docs/
    └── circuit_diagram.png   # Wiring diagram

🚀 Future Improvements

 📱 Mobile push notification via Telegram Bot or SMS
 📷 Camera integration for photo capture on detection
 🌐 Web dashboard for real-time monitoring
 🗂️ Multiple zone support with individual sensor control
 🔋 Battery backup support for power outages
 📊 Data logging to CSV / SQLite for history analysis



 This is the complete code. You just need to update the Gmail credentials and Wi-Fi/internet connection settings.



 import time
import os
import smtplib
import RPi.GPIO as GPIO
from hx711 import HX711
from picamera2 import Picamera2
from datetime import datetime

from email.message import EmailMessage

# ---------------- GPIO PINS ----------------
IR_PIN = 17          # IR sensor output pin
BUZZER_PIN = 27      # Buzzer signal pin
HX_DT = 5            # HX711 DT
HX_SCK = 6           # HX711 SCK

# ---------------- SETTINGS ----------------
ALARM_DURATION = 5
WEIGHT_THRESHOLD = 100
IMAGE_FOLDER = "captures"

# ---------------- EMAIL SETTINGS ----------------
SENDER_EMAIL = "rv4834751@gmail.com"         # CHANGE THIS
SENDER_PASSWORD = "yrqa ppnf xojh pqfy"        # CHANGE THIS
RECEIVER_EMAIL = "rv4834751@gmail.com" # CHANGE THIS

# ---------------- GPIO SETUP ----------------
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

GPIO.setup(IR_PIN, GPIO.IN)
GPIO.setup(BUZZER_PIN, GPIO.OUT)

# Active LOW buzzer
GPIO.output(BUZZER_PIN, GPIO.HIGH)   # buzzer OFF at startup

# ---------------- BUZZER FUNCTIONS ----------------
def buzzer_on():
    GPIO.output(BUZZER_PIN, GPIO.LOW)

def buzzer_off():
    GPIO.output(BUZZER_PIN, GPIO.HIGH)

# ---------------- CAMERA SETUP ----------------
camera = Picamera2()
camera.configure(camera.create_still_configuration())
camera.start()
time.sleep(2)

# ---------------- HX711 SETUP ----------------
hx = HX711(HX_DT, HX_SCK)

REFERENCE_UNIT = 200

hx.set_reading_format("MSB", "MSB")
hx.set_reference_unit(REFERENCE_UNIT)
hx.reset()
hx.tare()

print("Load cell tared successfully.")
print("Keep the platform empty while starting...")
time.sleep(2)

# ---------------- FOLDER SETUP ----------------
os.makedirs(IMAGE_FOLDER, exist_ok=True)

# ---------------- HELPER FUNCTIONS ----------------
def get_weight():
    try:
        weight = hx.get_weight(5)
        hx.power_down()
        hx.power_up()
        return round(weight, 2)
    except Exception as e:
        print("Weight read error:", e)
        return 0

def capture_image(reason):
    try:
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        filename = f"{IMAGE_FOLDER}/{reason}_{timestamp}.jpg"
        camera.capture_file(filename)
        print(f"[CAMERA] Image saved: {filename}")
        return filename
    except Exception as e:
        print("Camera capture error:", e)
        return None

def send_email_alert(reason, image_path=None):
    try:
        msg = EmailMessage()
        msg["Subject"] = f"Anti-Theft Alert: {reason}"
        msg["From"] = SENDER_EMAIL
        msg["To"] = RECEIVER_EMAIL

        current_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        msg.set_content(f"""
Alert Detected!

Reason: {reason}
Time: {current_time}

This alert was generated by your Raspberry Pi Anti-Theft Flooring System.
""")

        # Attach image if available
        if image_path and os.path.exists(image_path):
            with open(image_path, "rb") as img_file:
                img_data = img_file.read()
                img_name = os.path.basename(image_path)
                msg.add_attachment(img_data, maintype="image", subtype="jpeg", filename=img_name)

        # Send email
        with smtplib.SMTP_SSL("smtp.gmail.com", 465) as smtp:
            smtp.login(SENDER_EMAIL, SENDER_PASSWORD)
            smtp.send_message(msg)

        print("[EMAIL] Alert email sent successfully.")

    except Exception as e:
        print("Email sending error:", e)

def trigger_alarm(reason):
    print(f"[ALERT] {reason}")

    image_path = capture_image(reason)

    print(f"[BUZZER] ON for {ALARM_DURATION} seconds...")
    buzzer_on()
    time.sleep(ALARM_DURATION)
    buzzer_off()
    print("[BUZZER] OFF")

    send_email_alert(reason, image_path)

# ---------------- DETECTION LOCKS ----------------
weight_triggered = False
ir_triggered = False

# ---------------- MAIN LOOP ----------------
print("====================================")
print("ANTI-THEFT FLOOR SYSTEM STARTED")
print("Monitoring weight and IR sensor...")
print("====================================")

try:
    while True:
        ir_state = GPIO.input(IR_PIN)
        current_weight = get_weight()

        print(f"IR = {ir_state}, Weight = {current_weight}")

        # ==========================================
        # WEIGHT DETECTION
        # ==========================================
        if current_weight > WEIGHT_THRESHOLD and not weight_triggered:
            trigger_alarm("Weight_Exceeded")
            weight_triggered = True
            print("[LOCK] Weight alarm locked until weight becomes normal.")

        elif current_weight <= WEIGHT_THRESHOLD and weight_triggered:
            print("[RESET] Weight back to normal. System re-armed.")
            weight_triggered = False

        # ==========================================
        # IR DETECTION
        # ==========================================
        if ir_state == 0 and not ir_triggered:
            trigger_alarm("IR_Detected")
            ir_triggered = True
            print("[LOCK] IR alarm locked until object moves away.")

        elif ir_state == 1 and ir_triggered:
            print("[RESET] IR back to normal.")
            ir_triggered = False

        time.sleep(0.5)

except KeyboardInterrupt:
    print("Program stopped by user.")

finally:
    buzzer_off()
    GPIO.cleanup()



