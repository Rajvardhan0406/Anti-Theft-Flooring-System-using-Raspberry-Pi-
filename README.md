# Anti-Theft-Flooring-System-using-Raspberry-Pi-
A Raspberry Pi-based anti-theft flooring system that detects unauthorized movement via pressure sensors and triggers real-time alerts.

A smart, low-cost anti-theft security system built on Raspberry Pi that detects unauthorized foot traffic using a load cell (HX711) and an IR sensor. When suspicious activity is detected, the system captures a photo, sounds a buzzer alarm, and sends an email alert with the image attached — all in real time.

Table of Contents

About the Project
Features
How It Works
Hardware Requirements
Circuit Connections
Software Requirements
Installation
Configuration
Usage
Project Structure
Future Improvements
License
Author


About the Project
This project implements an intelligent anti-theft flooring system that combines weight sensing and infrared detection to monitor a protected area. When an intruder steps on the floor sensor or is detected by the IR sensor, the system:

Captures an image using the Raspberry Pi Camera
Sounds the buzzer alarm for 5 seconds
Sends an email alert with the captured image attached

It is ideal for:

Shops and retail stores
Home security
Museums and exhibition halls
Server rooms and restricted access zones


Features

Dual detection — Load cell (weight) + IR sensor
Camera capture — Auto photo on every alert
Email alert — Sends image via Gmail automatically
Buzzer alarm — Audible alert for 5 seconds
Auto re-arm — System resets when threat clears
Alarm lock — Prevents repeated false triggers
Headless operation — Runs without a monitor
Timestamped image logs — All captures saved locally


How It Works

Weight Detection — The load cell (HX711) continuously measures floor pressure. If weight exceeds the threshold (default: 100g), it triggers an alarm.
IR Detection — The IR sensor monitors for object/motion presence. If the IR pin goes LOW (object detected), an alarm is triggered.
On Alarm:

Camera captures and saves a timestamped image in the captures/ folder
Buzzer activates for 5 seconds
Email is sent to the receiver with the image attached


Auto Reset — Once the weight drops below the threshold or IR clears, the system re-arms automatically.


Hardware Requirements
ComponentQuantityDescriptionRaspberry Pi1Model 3B / 4B / Zero WHX711 Load Cell Amplifier1For weight/pressure sensingLoad Cell (Strain Gauge)1Floor pressure detectionIR Sensor Module1Infrared motion/object detectionRaspberry Pi Camera Module1For capturing intruder imagesActive Buzzer1Audible alarm (Active LOW)Jumper WiresSeveralFor connectionsBreadboard1For prototypingPower Supply15V for Raspberry Pi

Circuit Connections
ComponentRaspberry Pi GPIO PinIR Sensor OutputGPIO 17Buzzer SignalGPIO 27HX711 DT (Data)GPIO 5HX711 SCK (Clock)GPIO 6Camera ModuleCSI Camera Port

The buzzer used is Active LOW — it turns ON when GPIO is LOW and OFF when GPIO is HIGH.


Software Requirements

Raspberry Pi OS (Bullseye or later)
Python 3.7+
Required Python libraries:

RPi.GPIO
hx711
picamera2




Installation
1. Clone the Repository
bashgit clone https://github.com/Rajvardhan0406/Anti-Theft-Flooring-System-using-Raspberry-Pi-.git
2. Install Dependencies
bashpip3 install RPi.GPIO hx711 picamera2
3. Enable Camera on Raspberry Pi
bashsudo raspi-config
# Go to: Interface Options → Camera → Enable
# Then reboot
sudo reboot

Configuration
Before running, update the following settings inside main.py:
Email Settings
pythonSENDER_EMAIL = "rv35715@gmail.com"        # Your Gmail address
SENDER_PASSWORD = "raj12345"         # Gmail App Password (not your login password)
RECEIVER_EMAIL = "rv35715@gmail.com"   # Where alerts will be sent

How to get a Gmail App Password:

Go to your Google Account → Security
Enable 2-Step Verification
Go to App Passwords → Generate one for "Mail"
Use that 16-character password as SENDER_PASSWORD


Internet Connection
Make sure your Raspberry Pi is connected to the internet (via Wi-Fi or Ethernet) so that email alerts can be sent successfully.
bash# To connect via Wi-Fi, edit the config file:
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

# Add your network details:
network={
    ssid="Your_WiFi_Name"
    psk="Your_WiFi_Password"
}
Other Settings
pythonALARM_DURATION = 5        # Buzzer ON duration in seconds
WEIGHT_THRESHOLD = 100    # Minimum weight (grams) to trigger alarm
REFERENCE_UNIT = 200      # Calibration value for your load cell

You may need to adjust REFERENCE_UNIT based on your specific load cell calibration.


Usage
bashpython3 main.py
Expected Output:
====================================
ANTI-THEFT FLOOR SYSTEM STARTED
Monitoring weight and IR sensor...
====================================
Load cell tared successfully.
Keep the platform empty while starting...
IR = 1, Weight = 0.0
IR = 1, Weight = 0.0
[ALERT] Weight_Exceeded
[CAMERA] Image saved: captures/Weight_Exceeded_20250101_120000.jpg
[BUZZER] ON for 5 seconds...
[BUZZER] OFF
[EMAIL] Alert email sent successfully.
To stop the program:
Ctrl + C

Project Structure
anti-theft-flooring-system/
│
├── main.py          # Main script — complete system logic
├── README.md        # Project documentation
│
└── captures/        # Auto-created folder for alert images
    ├── Weight_Exceeded_20250101_120000.jpg
    └── IR_Detected_20250101_120500.jpg

Future Improvements

 Telegram Bot notification support
 Web dashboard for live monitoring
 Multiple sensor zone support
 Battery backup for power outages
 Detection log saved to CSV or database
 Scheduled arm/disarm (active only at night)



