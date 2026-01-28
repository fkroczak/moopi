Script anlegen, z.B. /home/moopi/boot_led.py:

python
#!/usr/bin/env python3
import RPi.GPIO as GPIO
import time

PIN = 18

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.setup(PIN, GPIO.OUT, initial=GPIO.LOW)


## Beim Booten: 5x blinken
for _ in range(5):
    GPIO.output(PIN, GPIO.HIGH)
    time.sleep(0.2)
    GPIO.output(PIN, GPIO.LOW)
    time.sleep(0.2)

## Danach: Dauerlicht, bis System herunterfährt
GPIO.output(PIN, GPIO.HIGH)
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    pass
finally:
    GPIO.output(PIN, GPIO.LOW)
    GPIO.cleanup()
Ausführbar machen:

bash
chmod +x /home/pi/boot_led.py

## 3. systemd-Service für das Boot-Script
Service-Datei /etc/systemd/system/boot_led.service:

text
[Unit]
Description=Status-LED beim Booten und im Betrieb
After=multi-user.target
DefaultDependencies=yes

[Service]
Type=simple
ExecStart=/usr/bin/python3 /home/pi/boot_led.py
Restart=always

[Install]
WantedBy=multi-user.target
Aktivieren:

bash
sudo systemctl daemon-reload
sudo systemctl enable boot_led.service
sudo systemctl start boot_led.service


## 4. LED beim Shutdown blinken lassen
Dafür ein zweites, einmalig ausgeführtes Script und einen Service, der an halt/poweroff hängt.

Script /root/shutdown_led.py:

python
#!/usr/bin/env python3
import RPi.GPIO as GPIO
import time

PIN = 18

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.setup(PIN, GPIO.OUT, initial=GPIO.LOW)

## Beim Shutdown: 10x schnell blinken
for _ in range(10):
    GPIO.output(PIN, GPIO.HIGH)
    time.sleep(0.1)
    GPIO.output(PIN, GPIO.LOW)
    time.sleep(0.1)

GPIO.cleanup()
Rechte setzen:

bash
sudo chmod 500 /root/shutdown_led.py
Service /etc/systemd/system/shutdown_led.service:

text
[Unit]
Description=Status-LED beim Herunterfahren blinken
DefaultDependencies=no
Before=poweroff.target halt.target reboot.target

[Service]
Type=oneshot
ExecStart=/usr/bin/python3 /root/shutdown_led.py

[Install]
WantedBy=poweroff.target halt.target reboot.target
Aktivieren:

bash
sudo systemctl daemon-reload
sudo systemctl enable shutdown_led.service
