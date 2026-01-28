# Moopi network audio streamer
Moopi is a compact DIY audio network streamer built on a Raspberry Pi 3B with a PiFi DAC+ V2.0. Running moOde, it provides high-quality audio streaming and local playback. mpd_oled displays track information, while a rotary encoder and power switch enable simple, tactile hardware control.

## Raspberry Pi Power Switch (GPIO)
[Link to the Raspberry Pi Power Switch](Raspberry_Pi_Power_Switch.md)

## MoOde LED Status-Anzeige (GPIO 18)
[Moopi LED Status-Anzeige](Moopi_LED_Status.md)


## Notes

- Tested with
```bash
Host:  moopi
RPiOS: 13.2 Trixie 64-bit | Linux: 6.12.47 64-bit
Model: Pi-3B 1GB
Audio: HiFiBerry DAC+
       m o O d e   a u d i o   p l a y e r

                    Series 10

             Release 10.0.3 2025-12-28
               (C) 2014 Tim Curtis
  ```
- Uses standard `gpio-shutdown` device tree overlay
- Requires momentary (normally open) push buttons

---
