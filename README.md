# esp-32-micropython-mq2-thingspeak
mengirimkan data mq2 ke thingspeak menggunakan micropython


import network
import urequests
import time
from machine import ADC, Pin

# -------------------------
# WIFI CONFIG
# -------------------------
WIFI_SSID = "FEEDING"
WIFI_PASS = "autofeeding"

# -------------------------
# THINGSPEAK CONFIG
# -------------------------
API_KEY = "FIIUB4XQCNVZGPOP"
URL = "https://api.thingspeak.com/update"

# -------------------------
# SETUP ADC MQ2
# -------------------------
mq2 = ADC(Pin(36))
mq2.atten(ADC.ATTN_11DB)      # Range approx 0 - 3.3V
mq2.width(ADC.WIDTH_12BIT)    # 0 - 4095

# -------------------------
# WIFI CONNECT
# -------------------------
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


# -------------------------
# SEND DATA TO THINGSPEAK
# -------------------------
def send_to_thingspeak(value):
    try:
        payload = "api_key={}&field1={}".format(API_KEY, value)
        response = urequests.post(URL, data=payload)
        print("ThingSpeak response:", response.text)
        response.close()
    except Exception as e:
        print("Error sending data:", e)


# -------------------------
# MAIN LOOP
# -------------------------
connect_wifi()

while True:
    adc_value = mq2.read()
    print("MQ2 ADC:", adc_value)

    send_to_thingspeak(adc_value)

    time.sleep(15)  # ThingSpeak minimum interval = 15 sec


