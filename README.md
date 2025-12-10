# ESP32 MicroPython MQ2 ThingSpeak

Simple project to send MQ-2 sensor data to ThingSpeak Cloud using
ESP32 + MicroPython.

This repository contains: - MicroPython code to read MQ2 sensor\
- Send data to ThingSpeak every 15 seconds\
- WiFi & API configuration example

------------------------------------------------------------------------

## Hardware Used

-   ESP32\
-   MQ-2 Sensor (analog)\
-   Jumper wires

### Pin Usage

-   MQ2 ADC1_CH0 (GPIO 36)

------------------------------------------------------------------------

## Installation

Ensure ESP32 is flashed with MicroPython, then upload `.py` files using
Thonny / ampy / rshell.

------------------------------------------------------------------------

## WiFi & ThingSpeak Config

Edit these values to match your router + API key:

``` python
WIFI_SSID = "your wifi"
WIFI_PASS = "wifi password"

API_KEY = "your API key"
URL = "https://api.thingspeak.com/update"
```

------------------------------------------------------------------------

## MicroPython Source Code

``` python
import network
import urequests
import time
from machine import ADC, Pin

# WIFI CONFIG
WIFI_SSID = "wifi kamu"
WIFI_PASS = "password wifi kamu"

# THINGSPEAK CONFIG
API_KEY = "API key kamu"
URL = "https://api.thingspeak.com/update"

# SETUP MQ2 ADC
mq2 = ADC(Pin(36))
mq2.atten(ADC.ATTN_11DB)      # Range approx 0 - 3.3V
mq2.width(ADC.WIDTH_12BIT)    # 0 - 4095

def connect_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(WIFI_SSID, WIFI_PASS)

    print("Connecting to WiFi...")
    while not wlan.isconnected():
        time.sleep(0.5)
        print(".", end="")

    print("\nConnected!")
    print(wlan.ifconfig())

def send_to_thingspeak(value):
    try:
        payload = "api_key={}&field1={}".format(API_KEY, value)
        response = urequests.post(URL, data=payload)
        print("ThingSpeak response:", response.text)
        response.close()
    except Exception as e:
        print("Error sending data:", e)

connect_wifi()

while True:
    adc_value = mq2.read()
    print("MQ2 ADC:", adc_value)

    send_to_thingspeak(adc_value)

    time.sleep(15)  # minimum interval ThingSpeak
```