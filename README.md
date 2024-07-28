## CubeSat Design Project README

### Table of Contents
1. [Introduction](#introduction)
2. [System Overview](#system-overview)
3. [Hardware Components](#hardware-components)
   - [433 MHz RF Module Transmitter](#433-mhz-rf-module-transmitter)
   - [DHT-22 Sensor](#dht-22-sensor)
   - [MPU6050 Sensor](#mpu6050-sensor)
4. [Software Implementation](#software-implementation)
   - [Transmitter Code](#transmitter-code)
   - [Receiver Code](#receiver-code)
5. [Project Demonstration](#project-demonstration)
6. [Conclusion](#conclusion)
7. [References](#references)

### Introduction
This project demonstrates the design and implementation of a CubeSat communication system using a 433 MHz RF module. The system includes environmental monitoring through a DHT-22 sensor and motion sensing using an MPU6050 sensor. The project aims to showcase the capabilities of CubeSats in collecting and transmitting data efficiently.

### System Overview
The CubeSat system is designed to collect environmental data (temperature and humidity) and motion data (acceleration on X, Y, and Z axes) and transmit this information wirelessly to a ground station. The primary components include a 433 MHz RF transmitter module, a DHT-22 sensor for environmental data, and an MPU6050 sensor for motion data.

### Hardware Components

#### 433 MHz RF Module Transmitter
The 433 MHz RF module is used for wireless communication between the CubeSat and the ground station. It operates in the 433 MHz frequency band and supports Amplitude Shift Keying (ASK) modulation.

- **Features:**
  - Frequency: 433 MHz
  - Modulation: ASK
  - Low power consumption



#### DHT-22 Sensor
The DHT-22 is a low-cost digital sensor for measuring temperature and humidity. It provides high reliability and long-term stability.

- **Specifications:**
  - Temperature Range: -40 to 80°C
  - Humidity Range: 0 to 100% RH
  - Accuracy: ±0.5°C (temperature), ±2-5% RH (humidity)



#### MPU6050 Sensor
The MPU6050 is a MEMS sensor that combines a 3-axis gyroscope and a 3-axis accelerometer. It is used to measure the CubeSat's orientation and motion.

- **Specifications:**
  - 3-axis accelerometer range: ±2, ±4, ±8, ±16 g
  - 3-axis gyroscope range: ±250, ±500, ±1000, ±2000°/sec
  - Communication: I2C



### Software Implementation

#### Transmitter Code
The transmitter code initializes the DHT-22 and MPU6050 sensors, reads the sensor data, and transmits it using the 433 MHz RF module.

```cpp
#include <RH_ASK.h>
#include <SPI.h>
#include "DHT.h"
#include "Wire.h"
#include "I2Cdev.h"
#include "MPU6050.h"

#define DHTPIN 7
#define DHTTYPE DHT22

float hum;
float temp;
String str_humid;
String str_temp;
String str_X;
String str_Y;
String str_Z;
String str_out;

RH_ASK rf_driver;
DHT dht(DHTPIN, DHTTYPE);
MPU6050 mpu;
int16_t ax, ay, az;
int16_t gx, gy, gz;

struct MyData {
  byte X;
  byte Y;
  byte Z;
};

MyData data;

void setup() {
  Serial.begin(9600);
  Wire.begin();
  mpu.initialize();
  rf_driver.init();
  dht.begin();
}

void loop() {
  delay(2000);
  mpu.getMotion6(&ax, &ay, &az, &gx, &gy, &gz);
  data.X = map(ax, -17000, 17000, 0, 255);
  data.Y = map(ay, -17000, 17000, 0, 255);
  data.Z = map(az, -17000, 17000, 0, 255);

  hum = dht.readHumidity();
  temp = dht.readTemperature();
  str_humid = String(hum);
  str_temp = String(temp);
  str_X = String(data.X);
  str_Y = String(data.Y);
  str_Z = String(data.Z);

  str_out = str_humid + "," + str_temp + "," + str_X + "," + str_Y + "," + str_Z;
  static char *msg = str_out.c_str();

  rf_driver.send((uint8_t *)msg, strlen(msg));
  rf_driver.waitPacketSent();
}
```

#### Receiver Code
The receiver code initializes the RF module to receive data and prints the received messages to the serial monitor.

```cpp
#include <VirtualWire.h>

void setup() {
  vw_setup(2000);
  vw_set_rx_pin(11);
  vw_rx_start();
  Serial.begin(9600);
}

void loop() {
  uint8_t buf[VW_MAX_MESSAGE_LEN];
  uint8_t buflen = VW_MAX_MESSAGE_LEN;

  if (vw_get_message(buf, &buflen)) {
    Serial.print(": ");
    for (int i = 0; i < buflen; i++) {
      Serial.print(char(buf[i]));
    }
    Serial.println();
  }
}
```

### Project Demonstration
To demonstrate the project:
1. **Setup the Transmitter:**
   - Connect the DHT-22 and MPU6050 sensors to the Arduino.
   - Upload the transmitter code to the Arduino.
   - Power on the Arduino to start transmitting data.

2. **Setup the Receiver:**
   - Connect the 433 MHz RF receiver module to another Arduino.
   - Upload the receiver code to the Arduino.
   - Open the Serial Monitor to view the received data.

The system will display temperature, humidity, and motion data collected by the CubeSat on the Serial Monitor.

### Conclusion
This project successfully demonstrates a CubeSat design with wireless communication using a 433 MHz RF module, environmental monitoring with a DHT-22 sensor, and motion sensing with an MPU6050 sensor. The setup provides a foundation for more advanced CubeSat designs and showcases the potential for small-scale satellite missions.

### References
- [DroneBot Workshop](https://dronebotworkshop.com)
- [DHT-22 Sensor Datasheet](https://cdn.sparkfun.com/datasheets/Sensors/Temperature/DHT22.pdf)
- [MPU6050 Sensor Datasheet](https://invensense.tdk.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf)
- [433 MHz RF Module Datasheet](https://www.electronicwings.com/arduino/433-rf-module)

---

