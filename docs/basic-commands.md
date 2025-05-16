# Basic SDI-12 Commands

This document provides an overview of the basic SDI-12 commands, including their descriptions and examples of usage.

## Command List

| Command | Description | Example | Response |
|---------|-------------|---------|----------|
| `0!`    | Acknowledge active | `0!` | `0` |
| `?!`    | Address query (only works if one sensor on bus) | `?!` | `0` |
| `0I!`   | Send Identification | `0I!` | `14Geonor T-200B0000.1` |
| `0A1!`  | Change Address | `0A1!` | `1` |
| `1A0!`  | Change Address (back to 0) | `1A0!` | `0` |

## Measurement Commands

### Start Measurement

To initiate a measurement, use the following command:

```bash
> 0M!
```

**Response:**

```bash
00003
```

Three measurements are immediately available. Data is always available immediately; the sensor never issues a Service Request response.

### Send Data

To retrieve the measurement data, use:

```bash
> 0D0!
```

**Response:**

```bash
0+0000.0+0000.0+1908.396
```

The return string shows the 3 measured string frequencies. Only one vibrating wire (VW) is connected in this example.

### Start Concurrent Measurement

To start concurrent measurements, use:

```bash
> 0C!
```

**Response:**

```bash
000003
```

Three measurements are immediately available.

### Continuous Measurement

To retrieve continuous measurement data, use:

```bash
> 0R0!
```

**Response:**

```bash
0+0000.0+0000.0+1908.766
```

This command retrieves the first continuous measurement.

### Additional Notes

- After each power-on, the sensor address is set to 0. If multiple sensors are connected to the SDI-12 bus, each must have a unique address.
- The commands listed above are essential for basic interaction with SDI-12 sensors and should be used as a foundation for more advanced operations.
