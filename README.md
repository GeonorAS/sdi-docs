# Geonor SDI-12 Documentation

The PCB hardware and software comply with the SDI-12 specification version 1.4. This documentation provides examples of data logger commands and sensor responses for all defined commands.

```bash
> command
response
```

---

## Table of Contents

- [Basic SDI-12 Commands](#basic-sdi-12-commands)
- [Extended Commands](#extended-commands)
- [Calibration Constants](#calibration-constants)
- [Real Time Clock](#real-time-clock)
- [Threshold Limits](#threshold-limits)
- [FLASH Data Log](#flash-data-log)
- [Erase FLASH](#erase-flash)
- [Example Usage](#example-usage)

---

## Basic SDI-12 Commands

| Command | Description | Example | Response |
|---------|-------------|---------|----------|
| `0!`    | Acknowledge active | `0!` | `0` |
| `?!`    | Address query (only works if one sensor on bus) | `?!` | `0` |
| `0I!`   | Send Identification | `0I!` | `14Geonor T-200B0000.1` |
| `0A1!` | Change Address | `0A1!` | `1` |
| `1A0!` | Change Address (back to 0) | `1A0!` | `0` |

> **Note:** After each power-on, the sensor address is 0. If multiple sensors are connected to the SDI-12 bus, each must have a unique address.

### Measurement Commands

- **Start Measurement:**

  ```bash
  > 0M!
  00003
  ```

  Three measurements are immediately available. Data is always available immediately; the sensor never issues a Service Request response.

- **Send Data:**

  ```bash
  > 0D0!
  0+0000.0+0000.0+1908.396
  ```

  The return string shows the 3 measured string frequencies. Only one vibrating wire (VW) is connected in this example.

- **Start Concurrent Measurement:**

  ```bash
  > 0C!
  000003
  ```

Three measurements are immediately available.

- **Send Data**

  ```bash
  > 0D0!
  0+0000.0+0000.0+1908.396
  ```

- **Start Measurement 1:**

  ```bash
  > 0M1!
  00001
  ```
  
  One measurement is immediately available.

- **Send Data**

  ```bash
  > 0D0!
  0+024.1250
  ```

  Returns the measured temperature.

- **Continuous Measurement 0:**

  ```bash
  > 0R0!
  0+0000.0+0000.0+1908.766
  ```

  Measurement 0 contains the 3 string frequencies.

- **Continuous Measurement 1:**

  ```bash
  > 0R1!
  0+024.2500
  ```

  Measurement 1 contains the temperature.

- **Continuous Measurement with CRC:**

  ```bash
  > 0RC0!
  0+0000.0+0000.0+1908.55Axx
  ```

  Note the Axx CRC characters at the end.

> **Tip:** Since the sensor always has measurement data immediately available, only the Continuous Measurement command is really needed. Other commands are implemented for SDI-12 standard compliance.

---

## Extended Commands

Extended commands are used for calibration and setup. These are sensor-specific and defined by the manufacturer.

Each VW has a set of three calibration constants which are needed to calculate the precipitation. These constants needs to be set for the calculation of the perception to be correct. The sensor also includes a real time clock that looses it's content when power is turned off. Therefore, after each power-on the clock needs to be set if the time stamp of the measurements stored in FLASH-memory shall be correct. To set the calibration constants and the real time clock (date and time), extended commands are used.

From the SDI-12 specification : Sensors are required only to respond to the basic SDI-12 command set. Sensors, however, usually require calibration or other setup commands. Extended commands provide the means for such functions. An extended command is a command for a specific make of sensor to tell that sensor to do a specific task. Extended commands are defined and documented by the manufacturer of each sensor.

### Read Sensor Status

The following extended command is used to read the status of the sensor.

```bash
> 0XSTAT!
0 2000-01-01 00:05:00 flash=000000/032766 f01=+2243.400 a1=+0000.045 b1=+0012.500
f02=+2243.400 a2=+0000.045 b2=+0012.500 f03=+2243.400 a3=+0000.045 b3=+0012.500
fthr=000001000 tthr=000100000 uCtemp=28 SupVolt=11956
```

- The first value (0) is the sensor address.
- Date and time from the real time clock.
- `ash=000000/032766`: 0 out of 32766 FLASH-memory pages hold stored measurements.
- Calibration constants for all three VWs.
- Frequency and time threshold limits.
- Internal temperature and supply voltage.

The X states that this is an extended SDI-12 command, meaning, the rest of the command is special for this sensor. The SDI-12 standard permits sensor specific commands precisely for cases like reading and setting sensor configurations. The STAT is the name of this particular command (read sensor status). The final ! terminates the extended command like any other SDI-12 command.

The first 0 in the reply is the address of the sensor which is replying. Then follows the date and time taken from the sensor real time clock, 2000-01-01 00:05:00. In this case the real time clock has not been set since the date is January 1 in year 2000, and the time is 5 minutes past midnight (the time the sensor has been powered on). The flash=000000/032766 says that 0 out of 32766 FLASH-memory pages currently hold stored measurements. Then follows the calibration constants f0, a and b for all three VWs. The fthr=000001000 tthr=000100000 are the frequency threshold and time threshold limits. More on these values later. The last two tokens shows that the internal temperature of the micro controller is 28 C and that the supply voltage is measured to be 11.956 Volt.

As measurements are made they will be stored to FLASH-memory. After 11 measurements are made the flash status will show flash=000001/032766 since each FLASH-memory page holds 11 measurements. Hence, the total number of measurements that can be stored in FLASH-memory is 360 426.

---

## Calibration Constants

Each VW has three calibration constants for precipitation calculation. Set them using the following commands:

- **Set calibration constants for string 1:**

  ```bash
  > 0XCAL1+1912.4+0.045+12.500!
  0CAL1: f01=+1912.400 a1=+0000.045 b1=+0012.500
  ```

  - Format: `<+/-><n-digits>.<m-digits>`
  - Formula: `mm = a × (ƒ − ƒ0) + 10⁶ × b × (ƒ − ƒ0)²`

- **Set calibration constants for string 2:**

  ```bash
  > 0XCAL2+1907.4+0.039+11.900!
  0CAL2: f02=+1907.400 a2=+0000.039 b2=+0011.900
  ```

- **Set calibration constants for string 3:**

  ```bash
  > 0XCAL3+1911.1+0.042+12.200!
  0CAL3: f03=+1911.100 a3=+0000.042 b3=+0012.200
  ```

> **Note:** Calibration constants are stored in FLASH and restored at power-on. If incorrect, set them again.

---

## Real Time Clock

Set the real time clock after each power-on (no battery backup):

```bash
> 0XDATE241231180437!
0 2025-04-16 10:46:08
```

- Format: `0XDATEYYMMDDhhmmss!`
- Example above sets date/time to 2024-12-31 18:04:37.
- Illegal dates/times are ignored (no response).

---

## Threshold Limits

The sensor measures string frequencies and temperature every 4 seconds. Use threshold limits to control when data is stored:

- **Frequency threshold (fthr):** Only store if average frequency change exceeds limit.
- **Time threshold (tthr):** Store at least every tthr seconds.

| Setting | Command Example | Description |
|---------|----------------|-------------|
| Frequency threshold | `0XFTHR500!` | Only store if change > 0.5 Hz |
| Time threshold | `0XTTHR60!` | Store at least every 60 seconds |

> If `fthr=0`, all measurements are stored. If `tthr=0`, all measurements are stored. Use `0XSTAT!` to check active limits.

---

## FLASH Data Log

The sensor includes FLASH memory (32767 pages, 11 measurements per page). Each measurement includes date/time, temperature, VW frequencies, and precipitation.

- **Download stored data:**

  ```bash
  > 0XDLOG0!
  025 04 16 13 19 40 +024.4375 +0000.000 +0000.000 +1910.219 +0004.540
  ...
  ```

  - First value: sensor address
  - Next: year, month, day, hour, minute, second
  - Then: temperature, 3 string frequencies, calculated precipitation

> **Tip:** Transfer time for a FLASH-memory page is several seconds (SDI-12 baudrate is 1200 bps).

---

## Erase FLASH

Delete all FLASH data log contents:

```bash
> 0XERAS!
0
```

If successful, only the sensor address is returned. Use `0XSTAT!` to confirm FLASH is empty.

---

## Example Usage

Below is an example interaction sequence:

1. **Check if sensor is alive:**

   ```bash
   > 0!
   0
   ```

2. **Get identification:**

   ```bash
   > 0I!
   014Geonor T-200B0000.2
   ```

3. **Read sensor status:**

   ```bash
   > 0XSTAT!
   0 2000-01-01 00:00:39 \u001dash=000000/032766 f01=+2243.400 a1=+0000.045 b1=+0012.500
   ...
   ```

4. **Set fthr and tthr:**

   ```bash
   > 0XFTHR500!
   0FTHR: fthr=000000500
   > 0XTTHR60!
   0TTHR: tthr=000000060
   ```

5. **Set calibration constants:**

   ```bash
   > 0XCAL1+1812.0+0.045+12.500!
   0CAL1: f01=+1812.000 a1=+0000.045 b1=+0012.500
   > 0XCAL2+1814.0+0.037+11.400!
   0CAL2: f02=+1814.000 a2=+0000.037 b2=+0011.400
   > 0XCAL3+1821.2+0.048+13.100!
   0CAL3: f03=+1821.200 a3=+0000.048 b3=+0013.100
   ```

6. **Set date and time:**

   ```bash
   > 0XDATE250505103353!
   0 2025-05-05 10:33:53
   ```

7. **Erase FLASH:**

   ```bash
   > 0XERAS!
   0
   ```

8. **Read measurement data:**

   ```bash
   > 0R0!
   0+0000.0+0000.0+1909.564
   > 0RC1!
   0+026.2500DCV
   ```

9. **Read FLASH data log:**

   ```bash
   > 0XDLOG0!
   025 05 05 10 38 54 +026.0000 +0000.000 +0000.000 +1910.832 -0013.583
   ...
   ```

> **Note:** After power cycle, calibration constants and thresholds are restored from FLASH, but date/time is reset.

---

For troubleshooting and further details, refer to the SDI-12 specification and manufacturer documentation.
