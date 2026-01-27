# Raspberry Pi Power Switch (GPIO)

A safe and reliable power button solution for Raspberry Pi projects, designed especially for DIY devices such as HiFi systems.

## Motivation

The Raspberry Pi does **not** include a built-in hardware power switch. In many projects, the device is powered on or off by plugging or unplugging the USB power cable.

This approach can:

- Corrupt or damage the SD card over time
- Be inconvenient or impractical for embedded or DIY devices
- Feel unpolished for finished projects (e.g. HiFi systems)

This project documents a **GPIO-based power button solution** that enables both **power-on** and **safe shutdown** using simple push buttons.

---

## Power-On (GPIO_3)

On the firmware level, the Raspberry Pi is configured so that:

- **GPIO_3 (Pin 5)** triggers a boot when it is pulled to **GND**

This allows the Raspberry Pi to be powered on using a **momentary push button** connected between:

- GPIO_3 (Pin 5)
- GND

With this wiring alone, the Raspberry Pi can already be powered on safely.

---

## Shutdown (GPIO_17)

Shutting down the Raspberry Pi requires additional firmware configuration.

While it would be ideal to use **GPIO_3** for both startup and shutdown, extensive testing showed that GPIO_3 cannot be reliably configured for both actions simultaneously.

Therefore, a second GPIO pin is used:

- **GPIO_17** → Shutdown button

---

## Firmware Configuration

To enable shutdown via GPIO_17, follow these steps:

### 1. Connect via SSH
```bash
ssh pi@<raspberry-pi-ip>
```

### 2. Edit the firmware configuration
```bash
sudo nano /boot/firmware/config.txt
```

### 3. Add the following line at the end of the file
```ini
dtoverlay=gpio-shutdown,gpio_pin=17,active_low=1,gpio_pull=up
```

### 4. Save and exit
- Press **CTRL + X**
- Confirm with **Y**
- Press **Enter**

### 5. Reboot
```bash
sudo reboot
```

---

## Result

After rebooting, the Raspberry Pi will respond as follows:

| Function  | GPIO Pin | Wiring       |
|----------|----------|--------------|
| Power On | GPIO_3   | GPIO_3 → Button → GND |
| Shutdown | GPIO_17  | GPIO_17 → Button → GND |

This setup allows:

- Safe shutdown (prevents SD card corruption)
- Clean power-on behavior
- A practical solution for embedded and DIY Raspberry Pi projects

---
