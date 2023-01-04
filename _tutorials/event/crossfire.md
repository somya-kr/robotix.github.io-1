---
layout: post
title: "Crossfire Tutorial"
categories:
 - event
---

#### Crossfire

To build a wired/wirelessly controlled soccer robot capable of dribbling and shooting the ball.

#### Component List 

![](/img/tutorial/event/crossfire/componentlist.png){:.img-responsive}

#### RoboSoccer Bot

##### Chassis design

Chassis is a mechanical assembly for making a 2 wheel drive platform where you can mount your controller board to drive your bot. There are spaces for 2 wheels that can be attached to motors and one slot for a caster wheel, as shown in the figure.

![](/img/tutorial/event/crossfire/redchassis.png){:.img-responsive}

##### WASD Controller with relay

This type of mechanism helps make a bot that moves in all directions by pressing the corresponding buttons. By making the following connections, the WASD controller circuit can be made.

**The controller circuit:**

![](/img/tutorial/event/crossfire/controller.png){:.img-responsive}

**The push button circuit:**

![](/img/tutorial/event/crossfire/circuit_relay.png){:.img-responsive}

**The truth table for the following switches is given below:**

| State           | R1 | R2 | L1 | L2 |
| --------------- | -- | -- | -- | -- |
| Forward(**F**)  | 1  | 0  | 1  | 0  |
| Backward(**B**) | 0  | 1  | 0  | 1  |
| Left(**L**)     | 1  | 0  | 0  | 1  |
| Right(**R**)    | 0  | 1  | 1  | 0  |


##### Differential drive controller

The differential drive is a two-wheeled drive system with independent actuators for each wheel. In this type of mechanism, the motion vector of the robot is the sum of the independent wheel motions. A basic differential drive mechanism can be made by making the connections as shown below with a DPDT switch.

![](/img/tutorial/event/crossfire/diffdrive_1.png){:.img-responsive}

![](/img/tutorial/event/crossfire/diffdrive_2.jpg){:.img-responsive}

##### Kicker mechanism

A kicker mechanism helps thrust the ball forward and take shots at the opponent’s goalpost. The kicker is made using a servo motor attached to a long, thin block with sufficient ground clearance. A sudden thrust can be given to the ball by changing the angle instantaneously.

##### nRF and Wireless Transmission

The module can use 125 different channels, which gives the possibility to have a network of 125 independently working modems in one place. Each channel can have up to 6 addresses, or each unit can communicate with up to 6 other units at the same time.

The power consumption of this module is just around 12mA during transmission, which is even lower than a single LED. The operating voltage of the module is from 1.9 to 3.6V, but the good thing is that the other pins tolerate 5V logic, so we can easily connect it to an Arduino without using any logic-level converters.

Three of these pins are for SPI communication and they need to be connected to the SPI pins of the Arduino, but note that each Arduino board has different SPI pins. The pins CSN and CE can be connected to any digital pin of the Arduino board and they are used for setting the module in standby or active mode, as well as for switching between transmit or command mode. The last pin is an interrupt pin which doesn’t have to be used.

![](/img/tutorial/event/crossfire/arduino.png){:.img-responsive}

##### Transmitter code

```c
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>

RF24 radio(7, 8); // CE, CSN

const byte address[6] = "00001";

void setup() {
  radio.begin();
  radio.openWritingPipe(address);
  radio.setPALevel(RF24_PA_MIN);
  radio.stopListening();
}

void loop() {
  const char text[] = "Hello World";
  radio.write(&text, sizeof(text));
  delay(1000);
}
```
##### Receiver code

```c
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>

RF24 radio(7, 8); // CE, CSN

const byte address[6] = "00001";

void setup() {
  Serial.begin(9600);
  radio.begin();
  radio.openReadingPipe(0, address);
  radio.setPALevel(RF24_PA_MIN);
  radio.startListening();
}

void loop() {
  if (radio.available()) {
    char text[32] = "";
    radio.read(&text, sizeof(text));
    Serial.println(text);
  }
}
```

#### HC05 Bluetooth Module


##### Description of pins

* **Enable** - This pin is used to set the Data Mode or AT command mode (set high).
* **VCC** - This is connected to a +5V power supply.
* **Ground** - Connected to the ground of the powering system.
* **Tx (Transmitter)** - This pin transmits the received data Serially.
* **Rx (Receiver)** - Used for broadcasting data serially over Bluetooth.
* **State** - Used to check if the Bluetooth is working properly.


##### Modes of Operation

The HC-05 Bluetooth Module can be used in two modes of operation: Command Mode and Data Mode.


##### Command Mode

In Command Mode, you can communicate with the Bluetooth module through AT Commands for configuring various settings and parameters of the Module like getting the firmware information, changing the Baud Rate and changing the module name; it can be used to set it as master or slave. 

A point about HC-05 Module is that it can be configured as Master or Slave in a communication pair. In order to select either of the modes, you need to activate the Command Mode and send appropriate AT Commands. 


###### Data Mode

Coming to the Data Mode, in this mode, the module is used for communicating with other Bluetooth devices i.e. data transfer happens in this mode.


##### Code

```c
#define ledPin 7
int state = 0;
void setup() {
  //Setting the pin mode and initial LOW
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);
  Serial.begin(9600); // Default communication rate
}
void loop() {
  // Checks if the data is coming from the serial port
  if(Serial.available() > 0){
    state = Serial.read(); // Read the data from the serial port
 }
 Deciding functions for LED on and off
 if (state == '0') {
  digitalWrite(ledPin, LOW); // Turn LED OFF
  // Send back, to the phone, the String "LED: ON"
  Serial.println("LED: OFF");
  state = 0;
 }
 else if (state == '1') {
  digitalWrite(ledPin, HIGH);
  Serial.println("LED: ON");;
  state = 0;
 }
}
```