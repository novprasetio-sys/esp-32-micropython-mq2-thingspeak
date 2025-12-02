# ESP32 MicroPython MQ2 → ThingSpeak

Project sederhana untuk mengirimkan data sensor MQ-2 ke ThingSpeak Cloud menggunakan ESP32 + MicroPython.

Repository ini berisi:
- Kode MicroPython untuk membaca sensor MQ2  
- Mengirim data ke ThingSpeak tiap 15 detik  
- Contoh konfigurasi WiFi & API  

## Hardware yang Digunakan
- ESP32  
- Sensor MQ-2 (analog)  
- Kabel jumper  

Pin yang digunakan:
- MQ2 → pin ADC1_CH0 (GPIO 36)

## Instalasi
Pastikan ESP32 sudah ter-flash MicroPython, lalu upload file `.py` menggunakan Thonny / ampy / rshell.

## Konfigurasi WiFi & ThingSpeak
Edit bagian ini sesuai router dan API key kamu:

```python
WIFI_SSID = "wifi kamu"
WIFI_PASS = "password wifi kamu"

API_KEY = "API key kamu"
URL = "https://api.thingspeak.com/update"


# berikut kode micro python 

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