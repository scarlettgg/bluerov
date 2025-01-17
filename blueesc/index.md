---
layout: default
title: BlueESC Documentation
permalink: /bluesc/
order: 1
nav:
- Introduction: introduction 
- - Safety: safety
- - Quick Start: quick-start
- Specifications: specifications
- - Diagram: diagram
- - Specification Table: specification-table
- - 3D Model: 3d-model
- Operation: operation
- - LED Indicator Lights: led-indicator-lights
- I<sup>2</sup>C Protocol: i2c-protocol
- - Throttle Command: throttle-command
- - Data Request: data-request
- - Data Conversion: data-conversion
- - Assigning I<sup>2</sup>C Addresses: assigning-i2c-addresses
- Example Code: example-code
- - Arduino With Servo Library: arduino-with-servo-library
- - Arduino With I2C: arduino-with-i2c
- Advanced: advanced
- - Firmware Files: firmware-files
- - Firmware Update and Customization: firmware-update-and-customization
---

# Introduction

The BlueESC is an electronic speed controller for the T100 and T200 Thrusters. It's custom designed to mount directly to the thrusters and it is waterproof, water-cooled, and pressure-resistant.

## Safety 

<i class="fa fa-exclamation-triangle fa-fw fa-2x text-warning"></i> When working with electricity, especially in water, always practice caution. Always ensure that connections are secure and watertight. Keep your body away from spinning motors and propellers.

## Quick Start

The BlueESC comes preinstalled on the T100 or T200 Thrusters, and you don't have to do much to get started. All you need is a power source like a 12V battery or power supply and a signal source, like a servo tester, RC radio receiver, or a microcontroller.

1. Connect the BlueESC to the power source by connecting the thick red and black wires to power and ground (negative).
2. On the smaller signal cable, connect the black ground wire to the ground wire of the signal source. Connect the red or yellow PWM signal wire to the signal. The BlueESC does not have a battery eliminator circuit (BEC) and cannot power an external device at 5V.
3. Provide a "stopped" signal at 1500 &mu;s for a few seconds to allow the ESC to initialize. It will beep and the lights will flash briefly.
4. Once initialized, it's ready to run!

# Specifications

## Diagram

<img src="/assets/images/documentation/blue-esc-labels.png" class="img-responsive" style="max-width:800px" />

## Specification Table

|                       **Electrical**                        |
| --------------------------- | ------------- | ------------- |
| Voltage                     | 6-22 volts                    |
| Max Current (in water)      | 35 amps                       |
| Max Current (in air)        | 25 amps                       |
| --------------------------- | ------------- | ------------- |
|                       **Physical**                          |
| --------------------------- | ------------- | ------------- |
| Length of Enclosure         | 18 mm         | 0.71 in       |
| Diameter of Enclosure       | 40.4 mm       | 1.59 in       |
| Cable Length                | 1 m           | 39 in         |
| Power Cable Diameter        | 6.3 mm        | 0.25 in       |
| Signal Cable Diameter       | 3.8 mm        | 0.15 in       |
| Power Cable Colors          | Red - Positive                |
|                             | Black - Negative (Ground)     |
| Signal Cable Colors         | Black - Ground                |
|                             | Red or Yellow - PWM Signal    |
|                             | White - I<sup>2</sup>C Data (SDA) |
|                             | Green - I<sup>2</sup>C Clock (SCL) |
| --------------------------- | ------------- | ------------- |
|                    **Pulse Width Signal**                   |
| --------------------------- | ------------- | ------------- |
| Signal Voltage              | 3.3-5 volts                   |
| Update Rate                 | 50-400 Hz                     |
| Stopped                     | 1500 microseconds             |
| Max forward                 | 1900 microseconds             |
| Max reverse                 | 1100 microseconds             |
| Signal Deadband             | +/- 25 microseconds (centered around 1500 microseconds) |
| --------------------------- | ------------- | ------------- |
|                   **I<sup>2</sup>C Signal**                 |
| --------------------------- | ------------- | ------------- |
| Signal Voltage              | 5 volts                       |
| I<sup>2</sup>C Address      | 0x29 (default) - 0x38         |
| --------------------------- | ------------- | ------------- |
|                    **Performance**                   |
| --------------------------- | ------------- | ------------- |
| Maximum Depth               | To be determined; Designed for 500m+|

## 3D Model

Coming soon.

<!--
| File Type                  | Link                          |
| -------------------------- | ----------------------------- |
| SolidWorks Part (.sldprt)  | [BLUESC-R1.sldprt](#) |
| STEP (.step)               | [BLUESC-R1.step](#)   |
| IGES (.igs)                | [BLUESC-R1.igs](#) |
| STL (.stl)                 | [BLUESC-R1.stl](#) |
| All in a zip file (.zip)   | [BLUESC-R1.zip](#) |
-->

# Operation

## LED Indicator Lights

The BlueESC includes two indicator lights that show the status of the ESC. The behavior of these LEDs is consistent with the default behavior in the *tgy* firmware. Please see the [README for *tgy* for a detailed description of LED indicator light behavior](https://github.com/sim-/tgy/blob/master/README.md#troubleshooting).

# I<sup>2</sup>C Protocol

The I<sup>2</sup>C communication protocol allows two-directional communication with the ESC. The protocol uses a "register map" allowing registers to be written to and read from.

## Throttle Command

### Description

The throttle command is a 16-bit signed integer. The sign of the value determines the direction of rotation. Note, you must send a value of "0" at startup to initialize the thruster.

### Registers: (0x00-0x01)

* **throttle:** (write-only)
	* -32767 (max reverse) to 32767 (max forward)
	* 0 is stopped
	* No deadband

### Bytes

* **Byte 0:** throttle_h  
* **Byte 1:** throttle_l

## Data Request

### Description

The data registers can be read to provide information on voltage, current, RPM, and temperature. All values are 16-bit unsigned integers.

### Registers: (0x02-0x0A)

* **pulse_count:** (read-only)
  * Commutation pulses since last request.
  * Calculate rpm with pulse_count/dt*60/motor_pole_count
* **voltage:** (read-only)
	* ADC measurement scaled to 16 bits
  * Calculate voltage with voltage/2016
* **temperature:** (read-only)
	* ADC measurement scaled to 16 bits
  * Calculate temperature with the Steinhart equation
* **current:** (read-only)
	* ADC measurement scaled to 16 bits
  * Calculate current with (current-32767)/891
* **identifier:** (read-only)
	* Identifier bit to test if ESC is alive

### Bytes

* **Byte 0:** pulse_count_h  
* **Byte 1:** pulse_count_l  
* **Byte 2:** voltage_h  
* **Byte 3:** voltage_l  
* **Byte 4:** temperature_h  
* **Byte 5:** temperature_l  
* **Byte 6:** current_h  
* **Byte 7:** current_l  
* **Byte 8:** 0xab (identifier to check if ESC is alive)

## Data Conversion

The values sent through I2C for the sensors must be converted to the correct units. The following equations describe how to do so.

### Voltage

The voltage divider uses and 18K and 3.3K resistor for a voltage divider ratio of 6.45. The raw measurement is scaled to 16 bits. The conversion is as follows:

$$
\begin{align*}
V_{ESC} = 0.0004921 V_{raw}
\end{align*}
$$

A code example follows:

~~~ cpp
float voltage() {
  return voltage_raw*0.0004921;
}
~~~

### Current

The current is measured by the ACS711, a hall-effect sensor IC. The output is 14.706 A/V with a 2.5V offset. The raw measurements is scaled to 16 bits.

$$
\begin{align*}
I_{ESC} = 0.001122 (I_{raw}-32767)
\end{align*}
$$

A code example follows:

~~~ cpp
float current() {
  (return current_raw-32767)*0.001122;
}
~~~

### Temperature

The temperature is measured by a 10K thermistor (NCP18XH103J03RB) and 3.3K resistor. The temperature is calculated with the Steinhart-Hart equations. 

$$
\begin{align*}
\frac{1}{T}=\frac{1}{T_0}+\frac{1}{B}ln \left (\frac{1}{T}\right )
\end{align*}
$$


A code example follows:

~~~ cpp
// THERMISTOR SPECIFICATIONS
// resistance at 25 degrees C
#define THERMISTORNOMINAL 10000      
// temp. for nominal resistance (almost always 25 C)
#define TEMPERATURENOMINAL 25   
// The beta coefficient of the thermistor (usually 3000-4000)
#define BCOEFFICIENT 3900
// the value of the 'other' resistor
#define SERIESRESISTOR 3300 

float temperature(_temp_raw) {
  // This code was taken from an Adafruit
  float resistance = SERIESRESISTOR/(65535/float(_temp_raw)-1);

  float steinhart;
  steinhart = resistance / THERMISTORNOMINAL;  // (R/Ro)
  steinhart = log(steinhart);                  // ln(R/Ro)
  steinhart /= BCOEFFICIENT;                   // 1/B * ln(R/Ro)
  steinhart += 1.0 / (TEMPERATURENOMINAL + 273.15); // + (1/To)
  steinhart = 1.0 / steinhart;                 // Invert
  steinhart -= 273.15;                         // convert to C

  return steinhart;
}
~~~

### RPM

The RPM is sent as *pulses since last read*, which is the number of commutation cycles since the last time the I2C was polled.

$$
\begin{align*}
RPS_{ESC} = \frac{RPS_{raw}}{N_{poles} \Delta t} 
\end{align*}
$$

$$
\begin{align*}
RPM_{ESC} = 60 RPS_{ESC}
\end{align*}
$$

The value of $$N_{poles}$$ depends on the motor. The T100 has 12 poles and the T200 has 14 poles. The RPM measurement *does not include direction*. You can include direction by adding a negative symbol if the input signal to the ESC is negative.

A code example follows:

~~~cpp
float rpm() {  
  _rpm = float(_rpm)/((uint16_t(millis())-_rpmTimer)/1000.0f)*60/float(_poleCount);
  _rpmTimer = millis();
}
~~~

## Assigning I<sup>2</sup>C Addresses

When using more than one ESC, it is necessary to assign unique addresses to each ESC. To assign a new address to the ESC, you will have to update the firmware on the ESC.

**Tools Needed:**

* Arduino board with [ArduinoUSBLinker](https://github.com/bluerobotics/ArduinoUSBLinker) code uploaded

1. First, make sure that the [ArduinoUSBLinker](https://github.com/bluerobotics/ArduinoUSBLinker) code is uploaded on your Arduino. You can do this through the Arduino IDE.

2. Connect the BlueESC signal cable's black ground wire to one of the "GND" pins on the Arduino. Connect the red or yellow PWM signal wire to Arduino pin 2.

3. Power the BlueESC with a battery or power supply.

4. Download the [latest BlueESC firmware zip file here](/blueesc/firmware/blueesc_firmware_2015-07-09_a34f109.zip). Extract to a convenient location. 

5. There are several ways to upload the firmware to the BlueESC. The first is using [avrdude](http://www.ladyada.net/learn/avr/setup-win.html) from the command line:

~~~~~~~~~~ bash
# Navigate to the location containing the BlueESC firmware files
# Replace XX with the desired ID number 0-15 (0 ,1, 2, 3, etc.)
# Replace [programmer port] with serial programmer port, i.e. COM3
avrdude -c stk500v2 -b 19200 -P [programmer port] -p m8 -U flash:w:blueesc_idXX.hex:i
~~~~~~~~~~~~~~~

The second method is using a graphical utility such as [KKMulticopterTool](http://lazyzero.de/en/modellbau/kkmulticopterflashtool). Both the 32 and 64 bit versions will work.

* Set the programmer to *ArduinoUSBLinker*
* Select the port with your Arduino.
* Set the controller to *atmega 8-based brushless ESC (8kB flash)* 
* Under "Flashing", click on the "File" tab and browse to the firmware file you wish to flash.
* Click the green button to flash your BlueESC.

<img src="/assets/images/documentation/KKshot1a.png" class="img-responsive" style="max-width:600px" />

If everything went well, this is the message you should see indicating a successful firmware flash:

<img src="/assets/images/documentation/KKshot2a.png" class="img-responsive" style="max-width:600px" />

# Example Code

## Arduino with Servo Library

This example uses the Arduino Servo library to control the speed controller. This provides an update rate of 50 Hz and can use any pin on the Arduino board as the "servoPin".

**Note:** If you power the Arduino before powering the ESC, then the ESC will miss the initialization step and won't start. Power them up at the same time, power the ESC first, or press "reset" on the Arduino after applying power to the ESC.

~~~~~~~~~~ cpp
#include <Servo.h>

byte servoPin = 9;
Servo servo;

void setup() {
	servo.attach(servoPin);

	servo.writeMicroseconds(1500); // send "stop" signal to ESC.
	delay(1000); // delay to allow the ESC to recognize the stopped signal
}

void loop() {
	int signal = 1700; // Set signal value, which should be between 1100 and 1900

	servo.writeMicroseconds(signal); // Send signal to ESC.
}
~~~~~~~~~~~~~~~~

## Arduino with I2C

The following example uses the [Blue Robotics Arduino_I2C_ESC library](https://github.com/bluerobotics/Arduino_I2C_ESC) to control the BlueESC through I2C. 

~~~~ cpp

#include <Wire.h>
#include "Arduino_I2C_ESC.h"

#define ESC_ADDRESS 0x29

Arduino_I2C_ESC motor(ESC_ADDRESS);

int signal;

void setup() {
  Serial.begin(57600);
  Serial.println("Starting");
  
  Wire.begin();
}

void loop() {

  if ( Serial.available() > 0 ) {
    signal = Serial.parseInt();
  }
  
  motor.set(signal);

  motor.update();

  Serial.print("ESC: ");
  if(motor.isAlive()) Serial.print("OK\t\t"); 
  else Serial.print("NA\t\t");
  Serial.print(signal);Serial.print(" \t\t");  
  Serial.print(motor.rpm());Serial.print(" RPM\t\t");
  Serial.print(motor.voltage());Serial.print(" V\t\t");
  Serial.print(motor.current());Serial.print(" A\t\t");
  Serial.print(motor.temperature());Serial.print(" `C");
  Serial.println();

  delay(250); // Update at roughly 4 hz
}
~~~~~~~~~~~~~~~~~~~

# Advanced

## Firmware Files

The compiled firmware files can be downloaded below. This file includes firmware hex files precompiled with 16 different I<sup>2</sup>C addresses.

[<i class="fa fa-download fa-fw"></i> BlueESC Firmware (2015-07-09 a34f109)](/blueesc/firmware/blueesc_firmware_2015-07-09_a34f109.zip)

## Firmware Update and Customization

The Basic ESC uses the [tgy firmware](http://github.com/bluerobotics/tgy) which is open source and editable. There are many parameters that can be changed to change the performance of the speed controller. 

### Firmware Compilation

To compile the firmware, you'll need the avra AVR Assembler.

*Mac:* (Uses Homebrew)

~~~ bash
brew update
brew install avra
make blueesc.hex
~~~

To compile the files with multiple I2C addresses, you can use the following:

~~~ bash
make build_blueesc_addresses
~~~

*Linux (Ubuntu 14 LTS):*

~~~ bash
sudo apt-get install avra
git clone https://github.com/bluerobotics/tgy
cd tgy
make blueesc.hex
~~~

### Firmware Flashing

The ESC includes a bootloader that allows flashing through the PWM signal wire using a programming like the [Turnigy USB Linker](http://www.hobbyking.com/hobbyking/store/__10628__turnigy_usb_linker_for_aquastar_super_brain.html) or the [AfroESC Programmer](http://www.hobbyking.com/hobbyking/store/__39437__afro_esc_usb_programming_tool.html). 

~~~ bash
avrdude -c stk500v2 -b 9600 -P [programmer port] -p m8 -U flash:w:bluesc.hex:i
~~~

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
