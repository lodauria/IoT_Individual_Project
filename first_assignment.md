# First individual assignment

In the first assignment we describe the IoT system functionalities and components. A first working version of the system has been developed using only a single local sensor node.

The old version of the repository after the delivery of the first assignment can be found [here](https://github.com/lodauria/Individual_Assignments/tree/first_assignment).

## System logic description

To monitor the room condition we will consider the data collected from a light sensor and an Hall sensor placed on the projector screen. From this data, a model can be derived and used to automatically control the room lights with a relay and the windows curtains with a DC motor so to automatically have the ideal light condition in each situation.

In this prototype the actuation logic is simplified by these two rules:
- If the projector is open **AND** there is too much light **AND** is lecture time **=>** (if lights are on **=>** switch lights off) **ELSE** (if lights are off **AND** curtains are open **=>** close the curtains)
- If the projector is closed **AND** there is low light **AND** is lecture time **=>** (if curtains are closed **=>** open curtains) **ELSE** (if curtains are open **AND** lights are off **=>** switch on the lights).

## Sensors description

### Light sensor

The light condition is measured based on the electric resistance of a photocell (GL5528) placed in the room. Using the circuit below is possible to measure the photocell electrical resistance from an analog pin of an MCU board.

<img src="./src/photo_res.jpg" width="300"> <img src="./src/photo_circ.png" width="500">

Each raw measurement uses a 12 bit ADC, so the signal will be between 0 and 4095, this values are then mapped in a 10 to 100 lux scale. The analog measurement is repeated every 10s.

### Hall sensor

The Hall sensor (A3144) is already mounted on a simple sensor module which provides a digital signal LOW when a magnetic field is detected, HIGH otherwise. With a 10K potentiometer on the board is possible to regulate the sensibility based on the type of magnet used so to trigger the sensor at the desired distance (around 2cm in this case).

<img src="./src/hall_sensor.jpg" width="600">

This sensor is used to detect if a magnet placed on the end of the projector screen is near the sensor placed on the wall. In this way when the screen is rolled up (and so the projector is switched off) the Hall sensor detects the magnetic field of the magnet, while it doesn't when the screen is unrolled. The digital value given by this sensor is sampled after the light measurement (every 10s).

## Network architecture

In the configuration developed for the first assignment the network uses Mosquitto RSMB to communicate via ETHOS with the board and a Mosquitto broker with authentication to communicate with IoT Core. In the AWS Cloud IoT Core calls a lambda function when new data are received. This function is used to implement the collective intelligence and to store the new data in a DynamoDB table. A web page hosted on an S3 bucket realizes a simple user interface. This page periodically calls an API gateway to get the data from the DynamoDB and, if requested, sends actuation command to the IoT Core MQTT broker. The overall network is represented in this scheme.

<img src=src/network1.png width="800">

## Other considerations

### Concept considerations

Even if the system concept may seem trivial, the idea behind this choice is that monitoring the university rooms with sensors of any kind could be a valuable option. In the system prototype implemented for these individual assignments the sensors choice was due to their availability, but to obtain a smarter collective intelligence other kind of sensor could have been used. For example monitoring the room temperature, the heater and the opening of the windows could have been more interesting.

Also the use of the data collected is very trivial because there is only a simple reactive behavior, but in general the data about the rooms could be used by the university to obtain information about the students habits. The point is that, even if the system presented in these assignments is very simple, I thought that the idea of a smarter university has a potential.

All these considerations have been done only going on with the course, when it was to late to restart from scratch the individual assignments. Anyway, since the goal of these assignments was to reason on the use of IoT and develop something on your own, I'm happy with the choice because I've understood the limitations of my prototype. The main change I would make is the type of sensors chosen and how the data are used to create the collective intelligence. 

### Technical considerations

If we assume to be interested in the projector status a lot of other passive sensors could have been used instead of an Hall sensor since is used only to return a boolean signal.

The periodicity of the measurements has been chosen to make it easy to test the system, but for a real application is too short considering that this kind of system doesn't need to be so fast in reacting to changes.

## Hands-on walkthrough

For the first assignment the interactions with the environment are mediated only by a single STM32 Nucleo F401RE board connected to internet through a PC. Make sure to have installed [RIOTS-OS](https://github.com/RIOT-OS/RIOT), [Mosquitto RSMB](https://github.com/eclipse/mosquitto.rsmb), [Mosquitto broker](https://github.com/eclipse/mosquitto) and to have an active [AWS](https://www.awseducate.com/signin/SiteLogin?ec=302&startURL=%2Fstudent%2Fs%2F) account.

### STM32 Nucleo F401RE board setup

All the file needed for the RIOT application are contained in the [dedicated folder](devices/stm32_f401re).

#### Makefile adjustments

Make sure to modify `Makefile` with the correct path of the RIOT folder and the IPv6 prefix with the best option for your PC network configuration.

#### Compile and upload software

In the program directory compile and upload the code on the STM32 Nucleo FE401RE board with the following command

    make flash term

Don't forget to type also the super user password that will be required after executing this command.

#### Connect the electronic components

Connect the 2 sensors, the relay and the motor as shown below.

<img src=src/wiring.jpg width="800">

### Mosquitto and AWS setup

Refer to the dedicated folders to set up the other components:
- [Mosquitto setup](mosquitto/)
- [AWS setup](aws/)
