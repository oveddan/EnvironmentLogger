# WeatherStation

This is a modification of @FlorinAndrei excellent [WeatherStation](https://github.com/FlorinAndrei/WeatherStation). Repository.

It is a modification of that solution to, instead of sending log data over serial to a raspberry pi, it saves it to an SD card module, and logs the time of each reading by grabbing the time from a DS3231 clock chip.

It runs on the Arduino Nano 33 BLE sense, and reads environmental metrics:
- temperature
- relative humidity
- air pressure
- acceleration in 3D (or just gravity when not moving - might work as an earthquake detector)
- gyroscope data (rotation, in degrees / second) - should be near zero for a stationary device
- magnetic field in 3D (Earth's own field most of the time)
- light: RGB components plus overall intensity
- noise

Every reading, it also grabs the time from the clock module, and logs all of the readings with the time to an sd card, on a file called "datalog.txt"

The contents of this log file look like:

```
1581542711,a,27.23,32.1,1025.99,g,-0.06,-0.00,0.98,r,5.13,1.95,0.79,m,100.4,54.5,-33.1,l,7,4,4,10,n,134
1581542711,a,27.23,32.1,1026.04,g,-0.06,-0.01,0.98,r,4.94,1.83,0.79,m,99.8,54.6,-32.4,l,7,5,4,10,n,131
1581542712,a,27.27,32.3,1026.08,g,-0.06,-0.01,0.97,r,5.74,1.83,0.73,m,99.8,54.4,-34.0,l,7,4,4,9,n,142
1581542712,a,27.29,32.1,1026.05,g,-0.06,-0.00,0.97,r,5.00,1.83,0.85,m,100.4,54.4,-33.7,l,7,4,4,10,n,137
1581542713,a,27.27,32.1,1026.04,g,-0.06,-0.01,0.98,r,5.49,1.89,0.92,m,99.7,53.9,-33.2,l,7,5,4,10,n,167
1581542713,a,27.29,32.2,1026.07,g,-0.06,-0.00,0.98,r,4.88,1.83,0.73,m,99.7,54.3,-32.2,l,7,5,4,10,n,139
1581542714,a,27.30,32.3,1026.07,g,-0.06,-0.01,0.97,r,5.13,2.08,0.92,m,99.4,54.9,-31.7,l,7,5,4,10,n,139
1581542714,a,27.34,32.1,1026.06,g,-0.06,-0.00,0.98,r,5.13,2.08,0.79,m,99.1,54.2,-32.1,l,7,4,4,9,n,126
```

## Software

The code is lavishly commented. Read the comments in addition to this text.

### Arduino

The Arduino dialect of C++. Simply read data and present it to serial output. Other than that, do as little as possible - move fast and save CPU cycles. To capture noise, the Fast Fourier Transform is unavoidable (and quite compute-heavy). [This is the code](/nano33/nano33.ino).

The main loop runs several times each second, gathering metrics from sensors each time.

Rarely, when a sensor does not respond the right way, the main loop could freeze. For this reason, this app initializes a hardware watchdog in the Arduino CPU - if the watchdog is not pinged regularly by the main loop, it sends a hard reset to the whole device.

Thread on the Arduino forum regarding the loop() freeze bug: https://forum.arduino.cc/index.php?topic=643883.0

Thread on Nordic Semi Devzone regarding the watchdog: https://devzone.nordicsemi.com/f/nordic-q-a/53904/nrf52840-watchdog-for-arduino-nano-33-ble-sense

## Hardware

### Arduino

![Arduino](/images/nano33.jpg)

The [Arduino Nano 33 BLE Sense](https://store.arduino.cc/usa/nano-33-ble-sense) is used to read environmental parameters.

Surprisingly, the Nano 33 is fast enough to do FFT on the noise signal in real time. This would have been difficult with older platforms.

### SD Card Module

### Real Time Clock

### The whole system

## FFT (Fast Fourier Transform) for noise

Most ambiental parameters are straightforward.

Noise requires some math. The Arduino sensor captures a snapshot of the noise waveform. From there, we need to estimate loudness.

One way to do this is to do FFT on the waveform and generate the spectral components. Then sum all components on a logarithmic scale. This ought to be close enough to perceived loudness - the human ear is a logarithmic sensor.

## Data store and user interface

Graphite performs several functions:
- long term data storage
- expose data to a Web GUI such as [Grafana](https://grafana.com/) for visualization
- make data accessible via a REST API for analysis (trends, correlations, etc)

Example of Grafana dashboard showing sensor data:

![Grafana](/images/grafana.png)
