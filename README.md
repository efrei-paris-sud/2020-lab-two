
# Lab 2 Introduction to communication 
> It is mandetory to prepare your documents in `Markdown` format. We prepared an example repository in [Lab Report Sample](https://github.com/efrei-paris-sud/2019-sample-project/tree/master/lab/1). 
> All needed codes described in this repository. For more details about markdown please visit the [Example Report](https://github.com/efrei-paris-sud/2019-sample-project/tree/master/lab/1/report/1)
You can also use https://stackedit.io/ to easily create a markdown document.

Please put all reports into `lab/2/` folder in your repository.
> Report Deadline = 03/12/2020

The purpose of this lab is learning different common protocol. In this lab we will learn:
 - [x] I2C protocol
 - [ ] SPI protocol
 - [x] UART protocol
 - [ ] Working with I2C LCD
 - [x] Reading from BMP280 sensor

## Serial 
> One of the most basic communication protocols in electronics is the Universal Asynchronous Receive Transmit (UART) serial protocol. The UART protocol allows for two devices to communicate with each other. The protocol requires two wires between the devices that are communicating; one for each direction of communication. It is asynchronous. When a device transmits, it sends data as a series (i.e. serially) of pulses. Each pulse represents one bit of data, so a byte (8 bits) of data is sent as eight pulses on the wire. These pulses are sent with a particular, predefined timing called a baud rate that must be understood by both devices.

The Wiring Serial serial port allows for easily reading and writing data to and from external devices. It allows two machines to communicate and gives you the flexibility to make your own devices and use them as the input or output to your Wiring programs.

>The serial communication is very good for **debuging** your code. You can connect your arduino board to your computer easily. It works like a simple console. Let's start to use it.

In the Setup function we should define the Serial comunication. 
```C
void setup() {
  Serial.begin(9600);  // Start serial Serial at 9600 baud
}
```
> It is important that both devices have the same baud rate (transmission speed). [more info](https://wikipedia.org/wiki/UART)
> You should change the baud rate in your client to read the data correctly.

To send data to the other device we can easily use. It will print `string`.
```C
Serial.print("string");
```
To read data from other device, we can use following code.
```C
byte b = Serial.read();
```
For more information click [here](http://wiring.org.co/reference/Serial.html)

A simple program:
```Arduino
void setup() {
  Serial.begin(9600); // initialize serial:
}

void loop() {
  // if there's any data available, read it:
  while (Serial.available() > 0) {

    int i  = Serial.parseInt();    // look for the next valid integer in the incoming serial stream:
    byte b = Serial.read();    // look for the next valid byte in the incoming serial stream:
    if (Serial.read() == '\n') // look for the newline. That's the end of your sentence:
      .....
      
    // print the three numbers in one string as hexadecimal:
    Serial.print(15, HEX);//print F
    Serial.print(10, DEC);//Print 10
    Serial.println("Hello"); //Print Hello\n
  }
}
```
You can view the console by clicking on the following icon:
![serial](https://cdn-learn.adafruit.com/assets/assets/000/002/179/large1024/learn_arduino_ide_serial_moniotor_button.jpg?1396780247)



|Ex.1|  Develop an arduino program which can read a byte from serial  and adjust the passive buzzer frequency with that. Write a response that your buzzer frequency changed to the read value. (`lab/2/exercise/2/code1.ino`).
---|---
||Connect your Arduino board to computer. Open a serial communication software (You can use built in serial monitor in Arduino IDE by pressing `ctrl`+`shift`+`M`


## I2C
Inter-integrated Circuit (I2C) is a communications protocol common in microcontroller-based systems, particularly for interfacing with sensors, memory devices and LCDs (liquid crystal displays).

The I2C protocol requires only two wires Serial Data Line (SDA) and  Serial Clock Line (SCL). It's a synchronous protocol because it uses a clock line. 

![ic2](i2c-devices.png?raw=true)
> **BUS** is a communication system that transfers data between components.  

To use I2C, It is necessary to have at least one master. The master component select who should read or write data through the SDA and also it drives the SCL clock line. It communicate to a specific slave device with its I2C address (mostly predefined).


>I2C supports 7 or 10 bit addressing. Therefore, It can communicate with 128 or 1024 devices with only two wires in I2C protocol. 

### More details
The master component is the one who control the bus. Whenever the master want to send or receive data from a specific device, it write the address of the device and R/W bit (Read/Write bit: to specify the device should read from or write on data line) on the data line. Then the slave write one ACK bit (acknowledge bit) in the data line whenever it receives its address. After that, regarding to the  R/W bit, master or slave will write on the data line. Each data transmission composed of 8 bit (written by the data sender) and one ACK bit (written by the data receiver). The data part will be repeated until master sends a NACK (not acknowledge). The NACK is followed by STOP sign. In addition, The master also component produces the clock signal.  
![i2c-proto](i2c-protocol.png?raw=true)

|Ex.2|Specifiy the I2C pins (SDA and SCL) in your devices such as micro controller, sensors, actuators.
---|---


 
### How to use I2C

Using I2C is simple as bellow codes:
Similar to all Arduino programs, we have two routine. The setup routine is called one time when the micro controller start and then it will repeatedly call the loop routine.

```Arduino
#include<Wire.h> // including the Wire library
const int i2c_addr=0x68; // Specify the target I2C address
void setup(){
    Wire.begin();//Join the I2C bus as a master
}
void loop(){
//To send data to a slave:
    Wire.beginTransmission(i2c_addr);//Begin a transmission to the slave
    Wire.write(data);//Writing data to I2C bus
    Wire.endTransmission();//Ends a transmission to a slave device
//To request from a slave to send some bytes:
    Wire.requestFrom(i2c_addr, 6);//request 6 bytes from the slave device
    while (Wire.available()) {//slave may send less than requested
        char c = Wire.read();//Reading data from I2C bus
    }
}
```

# Step 1
Let's come back to the greenhouse project. There is a master (micro controller), a temperature sensor (an slave) and a LCD (an slave).

The project has one arduino, LCD, BMP280 sensor and relay. 
![bmp280](chapter4_no_mpu5060.png?raw=true)
There is exist also several libraries that provide an easy interface to communicate with the I2C devices. In This project, We will use BMP280 library.

```
#include<Wire.h>
#include <BMP280.h>   // include BMP280 sensor library
BMP280  bmp280;  // initialize  BMP280 library
void setup(){
  if( bmp280.begin(BMP280_I2C_ADDRESS) == 0 )// initialize the BMP280 sensor
  {  // connection error or device address wrong!
  //ToDo: Show error in LCD
  while(1);  // stay here
  }
}
void loop(){
  float temp     = bmp280.readTemperature();   // get temperature
  float pressure = bmp280.readPressure();      // get pressure
  float humidity = bmp280.readHumidity();      // get humidity
  float altitude= bmp.readAltitude(seaLevelPressure) // get Altitude. The generic seaLevelPressure is 1013.25. If you put the current pressure of your city, you can find the Altitude of your place!
  delay(1000);  // wait a second
}
```

|Ex.3|Complete the codes to show temperature and humidity in the Serial Console.
---|---


## Step 2:
Now It is time to turn on or off the air conditioner. You know that it is not possible to connect 220V power into the micro controller board directly. Therefore, What should we do to turn on the Air conditioner?
The response is on using Relay.
> A relay is an electrically operated switch. It exactly works as the normal wall switch but you can control it by a digital signal (0 or 1 value) from your micro controller.

In one side, a relay has 3 digital ports. two of them are for VCC and GND and one is for controlling the switch. In the other side it has also three ports. But the voltage can be high as 250V. The middle port of relay, is the input cable (i.e., phase or null) and two others shows the different states of a the switch. 
![relay](relay.png?raw=true)

> To simulate the air conditioner, we use a DC motor with a fan. You can use the VCC and GNF of your arduino kit for the external power source.
> 
> 


|Ex.4|Write a code to turn on/off the air conditioner when the temperature is above/below 25 degree centigrade. 
---|---

# Extra
|Ex.5| We know it is easy to add another I2C device to the system. How do you add MPU 5060 sensor to your board? Draw the schematic.
---|---

|Ex.6 | Find MPU 5060 Library and rewrite your codes to show Accelerometer data in LCD.
---|---



# Fritzing
The software is created in the spirit of the Processing programming language and the Arduino microcontroller[4] and allows a designer, artist, researcher, or hobbyist to document their Arduino-based prototype and create a PCB layout for manufacturing. 

- [Download Fritzing](https://github.com/fritzing/fritzing-app/releases/tag/CD-268)
- Create a new `sketch`.
- Create the step 1 sketch your self (please don't forget connect them and use different colors in wiring- NOTE: USE RED for `VCC` and BLUE for `GROUND`)
-- Export the result as image(`lab/2/exercise/0/sketch.png`)
- Design the Schematic of that. Please style it in a good format.
-- Export the result as image(`lab/2/exercise/0/schematic.png`)
- Make a (`lab/2/exercise/0/README.md`) file and put a the images and a small description on it and **USE MARKDOWN FORMAT PLEASE**. For images please use the uploaded images from your `github`.
[Example Report](https://github.com/efrei-paris-sud/2019-sample-project/tree/master/lab/1/report/1)
