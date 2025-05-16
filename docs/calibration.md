# Calibration Constants for SDI-12 Sensors

Calibration constants are essential for accurate measurements from SDI-12 sensors. Each sensor typically requires three calibration constants to be set for proper operation. This document outlines how to set these constants.

## Setting Calibration Constants

To set the calibration constants for each vibrating wire (VW) sensor, use the following commands:

### Set Calibration Constants for String 1

```bash
> 0XCAL1+<f01>+<a1>+<b1>!
```
- **f01**: Frequency constant for string 1.
- **a1**: Calibration coefficient for string 1.
- **b1**: Calibration coefficient for string 1.

**Example:**
```bash
> 0XCAL1+1912.4+0.045+12.500!
```

### Set Calibration Constants for String 2

```bash
> 0XCAL2+<f02>+<a2>+<b2>!
```
- **f02**: Frequency constant for string 2.
- **a2**: Calibration coefficient for string 2.
- **b2**: Calibration coefficient for string 2.

**Example:**
```bash
> 0XCAL2+1907.4+0.039+11.900!
```

### Set Calibration Constants for String 3

```bash
> 0XCAL3+<f03>+<a3>+<b3>!
```
- **f03**: Frequency constant for string 3.
- **a3**: Calibration coefficient for string 3.
- **b3**: Calibration coefficient for string 3.

**Example:**
```bash
> 0XCAL3+1911.1+0.042+12.200!
```

## Important Notes

- Calibration constants are stored in FLASH memory and will be restored upon power-on. If the constants are incorrect, they must be set again.
- Ensure that the values used for calibration are accurate to maintain measurement precision.

For further details, refer to the SDI-12 specification and the manufacturer's documentation.