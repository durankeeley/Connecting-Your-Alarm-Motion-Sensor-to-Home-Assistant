# Connecting Your Alarm's Motion Sensor (PIR) to Home Assistant

## Overview

This project is to connect the motion sensors currently connected to my Bosch System also to my Raspberry Pi, this wiring will allow them to run in parallel. This makes it guest friendly but allows me more control and notifications from Home Assistant.

## Hardware

- Bosch Solution 844
- Bosch ISM‑BLP1 Blue Line PIR Detector
- Raspberry Pi (i tested on a Pi 1 B+ before moving to a Zero)

## PIR Wiring Diagram

Wiring from the PIR (note the resistor from NC to T can be 3K3 or 6K8 depending on the Zone number) 

![Wiring-pir-alarm_raspberryPi](https://github.com/durankeeley/Connecting-Your-Alarm-s-Motion-Sensor-PIR-to-Home-Assistant/blob/main/Connecting%20Your%20Alarm%20Motion%20Sensor%20to%20Home%20Assistant.assets/Wiring-pir-alarm_bb-1.svg)

## Code

#### Prereq

DietPi need Python3 and GPIO

```bash
sudo apt -y install python3 python3-rpi.gpio python3-pip
```

After Python 3 is installed pano-mqtt is required from pip

```bash
sudo pip3 install paho-mqtt
```

#### Sensors

Change [ROOM] (without the square backets) to the room the sensor is in

```bash
nano alarm-motion-sensor.py
```

```python
import os
import RPi.GPIO as GPIO
import time

#Report to a MQTT broker
# pip3 install paho-mqtt
import paho.mqtt.publish as publish
Broker = '192.168.1.21'
pub_topic = 'alarm/motion-sensor/[ROOM]'

GPIO.setmode(GPIO.BOARD)
GPIO.setwarnings(False)

GPIO.setup(16, GPIO.IN, pull_up_down=GPIO.PUD_UP)
time.sleep(1)

while 1 >= 0:
    time.sleep(1)
    if GPIO.input(16) == GPIO.HIGH:
      publish.single(pub_topic,"OFF",hostname=Broker, port=1883,)
    else:
      if GPIO.input(16) == GPIO.LOW:
        publish.single(pub_topic,"ON",hostname=Broker, port=1883,)
        time.sleep(4)
```

#### Systemd

Systemd is used to automatically run the sensor Python script as boot

```bash
sudo nano /etc/systemd/system/alarm-motion-sensor-mqtt.service
```

```bash
[Unit]
Description=MQTT Garage Door Sensor
After=network.target

[Service]
ExecStart=/usr/bin/python3 alarm-motion-sensor.py
WorkingDirectory=/home/dietpi
StandardOutput=inherit
StandardError=inherit
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

## Home Assistant

The sensor is monitored in Home Assistant by connecting to the MQTT broker

### Prerequisites

- Integration: MQTT

### Adding MQTT Sensors

Edit the configuration.yaml

*Note: Change [NAME] and [ROOM]*

```bash
/config/configuration.yaml
```

```yaml
binary_sensor:
  - platform: mqtt
    name: "[NAME]"
    state_topic: "alarm/motion-sensor/[ROOM]"
    device_class: "motion"
```

### Lovelace

In lovelace add an entity card with the binary sesor you named earlier, you will see either Clear or Detected 

![image-20220103215624911](https://github.com/durankeeley/Connecting-Your-Alarm-s-Motion-Sensor-PIR-to-Home-Assistant/blob/main/Connecting%20Your%20Alarm%20Motion%20Sensor%20to%20Home%20Assistant.assets/image-20220103215624911.png)
