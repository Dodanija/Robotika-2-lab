# Smart Light Monitor – Arduino UNO

## 1. Project Description

Smart Light Monitor is an Arduino-based system that measures ambient light intensity using a photoresistor (LDR).

The light intensity is displayed on a 16×2 LCD as a relative percentage from 0% to 100%.

The user can adjust the light threshold using two buttons and enable or disable the alarm using a third button. When the measured light level falls below the configured threshold, a piezo buzzer produces an audible warning.

The system uses hardware timer interrupts, an external interrupt, non-blocking scheduling and EEPROM for persistent settings.

## 2. Components

- Arduino UNO R3
- 16×2 LCD display (1602A)
- Photoresistor (LDR)
- 2 kΩ resistor
- 3 push buttons
- Piezo buzzer
- Breadboard and jumper wires

## 3. Wiring

| Component | Arduino Pin |
|---|---|
| Photoresistor voltage divider output | A0 |
| PLUS button | D3 |
| MINUS button | D4 |
| ON/OFF button | D2 |
| Piezo buzzer (+) | D8 |
| Piezo buzzer (−) | GND |
| LCD RS | D12 |
| LCD E | D11 |
| LCD D4 | D5 |
| LCD D5 | D6 |
| LCD D6 | D7 |
| LCD D7 | D9 |

**Photoresistor circuit:**

- One LDR leg connects to 5V.
- The other LDR leg connects to A0.
- A 2 kΩ resistor connects between A0 and GND.

**LCD additional connections:**

- VSS → GND
- VDD → 5V
- VO → GND (fixed contrast connection)
- RW → GND
- D0–D3 → Not connected
- A (backlight +) → 5V through an appropriate current-limiting resistor
- K (backlight −) → GND

Each push button connects between its assigned Arduino digital pin and GND. Internal pull-up resistors are enabled using `INPUT_PULLUP`.

## 4. Build and Upload Steps

1. Assemble the circuit according to the wiring table.
2. Connect the Arduino UNO to the computer using a USB cable.
3. Open Arduino IDE.
4. Open the `SmartLightMonitor.ino` sketch.
5. Select **Arduino Uno** under Board.
6. Select the correct COM port.
7. Verify and upload the program.
8. Check the LCD and test the buttons and light sensor.

The program uses the standard Arduino `LiquidCrystal.h` and `EEPROM.h` libraries.

## 5. Timer Configuration

The system uses **Timer1**, a 16-bit hardware timer on the ATmega328P microcontroller.

| Parameter | Value |
|---|---|
| CPU frequency | 16 MHz |
| Timer | Timer1 |
| Mode | CTC |
| Prescaler | 64 |
| Compare register (OCR1A) | 249 |
| Interrupt interval | 1 ms |

Timer1 generates an interrupt every 1 ms and increments the `ticks` variable.

The main loop uses this counter to schedule tasks without `delay()`.

- Button checks: every 15 ms
- Light measurements: every 200 ms
- LCD updates: every 200 ms
- Buzzer: 300 ms ON / 700 ms OFF

## 6. Interrupt Service Routines (ISR)

**Timer Interrupt – `ISR(TIMER1_COMPA_vect)`**

Executed every 1 ms. Increments the `ticks` counter used for timing and scheduling.

**External Interrupt – `powerInterrupt()`**

Triggered by the ON/OFF button connected to D2 (`FALLING` mode).

The ISR sets the `powerEvent` flag. The main loop processes the event and updates the alarm state after a short debounce check.

Both interrupt handlers are kept short.

## 7. EEPROM Storage

EEPROM stores two persistent user settings: the light threshold and alarm state.

| Address | Field | Description |
|---|---|---|
| 0 | Magic | `0xA5` – validation marker |
| 1 | Version | `2` – settings format version |
| 2 | Limit | Light threshold (0–100%) |
| 3 | AlarmOn | Alarm state (0 = OFF, 1 = ON) |

At startup, the program validates the stored magic, version and setting values.

If validation fails, default settings are loaded:

- Light threshold: 30%
- Alarm: ON

`EEPROM.put()` updates only bytes whose values have changed, implementing write-on-change behavior.

## 8. System Operation

1. The photoresistor measures ambient light intensity.
2. Arduino converts the analog reading to a relative percentage.
3. The LCD displays the current light level and threshold.
4. The PLUS and MINUS buttons adjust the threshold in 5% steps.
5. The ON/OFF button enables or disables the alarm.
6. If the light level falls below the threshold and the alarm is enabled, the buzzer sounds periodically.
7. The threshold and alarm state are saved to EEPROM and restored after a power cycle.

## 9. Wiring Diagram

See `wiring.png` for the full circuit diagram.

## 10. Known Limitations

- Light intensity is displayed as a relative percentage, not calibrated lux.
- Very short button presses may be missed due to periodic button sampling.
