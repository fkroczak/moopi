# MoOde LED Status-Anzeige (GPIO 18)

Status-LED für Raspberry Pi mit **moOde OS** (Audio Player).  
**Blinken beim Boot** → **Dauerlicht** → **Blinken beim Shutdown**.

## Voraussetzungen

- **Hardware**: LED + 330Ω Widerstand an **GPIO18** (Pin 12)
- **Software**: `sudo apt install python3-rpi.gpio`

## 🚀 Firmware-Konfiguration (optional, sehr früh)

**`/boot/firmware/config.txt`** ergänzen:
```ini
# GPIO18 als Output HIGH (Dauerlicht direkt beim Boot)
gpio=18=op,dh
```
## 1. Boot-Script erstellen

**`/home/moopi/boot_led.py`**:
```python
#!/usr/bin/env python3
import RPi.GPIO as GPIO
import time

PIN = 18
GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.setup(PIN, GPIO.OUT, initial=GPIO.LOW)

# Beim Booten: 5x blinken

for _ in range(5):
    GPIO.output(PIN, GPIO.HIGH)
    time.sleep(0.2)
    GPIO.output(PIN, GPIO.LOW)
    time.sleep(0.2)

# Danach: Dauerlicht bis Shutdown
GPIO.output(PIN, GPIO.HIGH)
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    pass
finally:
    GPIO.output(PIN, GPIO.LOW)
    GPIO.cleanup()
```
Alternatives Skript bis Spotify erreichbar ist
```
#!/usr/bin/env python3
import RPi.GPIO as GPIO
import time
import subprocess

PIN = 12
GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.setup(PIN, GPIO.OUT, initial=GPIO.LOW)

def blink(count=1, duration=0.2):
    for _ in range(count):
        GPIO.output(PIN, GPIO.HIGH)
        time.sleep(duration)
        GPIO.output(PIN, GPIO.LOW)
        time.sleep(duration)

print("Boot LED start")
blink(10, 0.2)  # Phase 1: 10x Boot

# Phase 2: DAUERBLINKEN bis Spotify FULLY ready (+20s Buffer)
print("Dauerblinken bis Spotify ready...")
timeout = 300
start_time = time.time()
buffer_start = None

while time.time() - start_time < timeout:
    # Kontinuierlich blinken (alle 1s Check)
    blink(1, 0.8)  # Dauerhaftes Blinken (1.6s Zyklus)
    
    try:
        if subprocess.run(['pgrep', '-f', 'librespot']).returncode == 0:
            if buffer_start is None:
                buffer_start = time.time()
                print("librespot gestartet - starte 20s Buffer...")
            
            # Buffer-Zeit abgelaufen?
            if time.time() - buffer_start >= 20:
                print("20s Buffer done - Spotify ready! Dauerlicht!")
                break
                
    except:
        buffer_start = None  # Reset bei Fehlern

# Dauerlicht
GPIO.output(PIN, GPIO.HIGH)
print("Dauerlicht EIN - Spotify erreichbar")

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    pass
finally:
    GPIO.output(PIN, GPIO.LOW)
    GPIO.cleanup()

```

**Ausführbar machen**:

```bash
chmod +x /home/moopi/boot_led.py
```


## 2. systemd-Service (Boot)

**`/etc/systemd/system/boot_led.service`**:

```ini
[Unit]
Description=Status-LED beim Booten und im Betrieb
After=multi-user.target
DefaultDependencies=yes

[Service]
Type=simple
ExecStart=/usr/bin/python3 /home/moopi/boot_led.py
Restart=always

[Install]
WantedBy=multi-user.target
```

**Aktivieren**:

```bash
sudo systemctl daemon-reload
sudo systemctl enable boot_led.service
sudo systemctl start boot_led.service
```


## 3. Shutdown-Script

**`/root/shutdown_led.py`**:

```python
#!/usr/bin/env python3
import RPi.GPIO as GPIO
import time

PIN = 18
GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.setup(PIN, GPIO.OUT, initial=GPIO.LOW)

# Beim Shutdown: 10x schnell blinken
for _ in range(10):
    GPIO.output(PIN, GPIO.HIGH)
    time.sleep(0.1)
    GPIO.output(PIN, GPIO.LOW)
    time.sleep(0.1)

GPIO.cleanup()
```

**Rechte setzen**:

```bash
sudo chmod 500 /root/shutdown_led.py
```


## 4. systemd-Service (Shutdown)

**`/etc/systemd/system/shutdown_led.service`**:

```ini
[Unit]
Description=Status-LED beim Herunterfahren blinken
DefaultDependencies=no
Before=poweroff.target halt.target reboot.target

[Service]
Type=oneshot
ExecStart=/usr/bin/python3 /root/shutdown_led.py

[Install]
WantedBy=poweroff.target halt.target reboot.target
```

**Aktivieren**:

```bash
sudo systemctl daemon-reload
sudo systemctl enable shutdown_led.service
```


## Status prüfen

```bash
sudo systemctl status boot_led.service
sudo systemctl status shutdown_led.service
sudo journalctl -u boot_led.service -f
```

## Hardware-Schema

```
Raspberry Pi GPIO18 (Pin 12)
     │
    LED
     │
 330Ω ─ GND
```

**Funktion**:

- **Boot**: 5× blinken (0.4s Zyklus) → **Dauerlicht**
- **Shutdown**: 10× blinken (0.2s Zyklus) → Aus

---

*Erstellt für moOde OS auf Raspberry Pi | [fkroczak/moopi](https://github.com/fkroczak/moopi)*

```
