@mainpage Smart Rack Master software and Hardware documentation
@tableofcontents
|@subpage nomenclature|@subpage codebrief|
@section Intro  Introduction
The SmartRack is a IOT enabled food containment device for B2C or B2B use developed by Agilebot Automation Pvt Ltd, Bangalore.
The Rack consists of a 23 self contained slave units and a HMI. Stacked in a 4X6 fashion that have the ability to contain food packets/parcels.The master HMI box uses Raspbery PIi3 B+, lcd dsiplay for user interface. Each slave comes equipped with independent heating system and other hardware components to perform its basic functions.\n

@section mainHardware Master Box hardware and software
<!-- @subpage hardware -->
Master box  is equipped with a touch screen, LCD display and keypad.Interaction with the rack is done via this master box. This box has raspberry pi 3B+ as the motherboard.This provides a graphical user interaction with the environment and the rack. Master will have multiple programs running.Gui, Keypad program, Serial Program , Mqtt communication will be the main program running continuously running in a continuous loop. \n
Functions or Programs Master have: \n
- Mqtt and Database:  Collect all the packets of data from the broker and store it in a local Sql Database. Periodically publish to certain topics in the broker
- Keypad Program : Program to interface with Keypad
- Serial (Rs485) :  Program to communicate with slave box
- GUI :    program for user interface
@subsection addon Additional Hardware
- ***Master Controller (Mispigy)***: \n
Master controller acts as a portal between the master pi and the slaves. Its purpose is to transfer data which are not corrupted to the master from the slave and vice versa. 
Master controller uses two serial ports USCI1 and USCI0. USCI0 is set up and used for communicating between the master controller and master pi. The USCI1 is used for communicating between the master controller and the slaves(RS485). The master pi sends packets of data to the master controller, the controller takes data and checks whether the data is corrupted by checking the LRC. If the data is valid , the master controller sends the data to the data bus. The same procedure is done when the data is received from the slave,  the master controller checks if the response received from the slave is valid or not. If valid the data is sent to the master, if not the data is ignored.
- ***RS485 Module***: \n
This module allows the converts UART to the RS485. Is generally used for industrial automation.The normal uart tx and rx is convereted to rs485 full dublex A,B,Y,Z. 

@section mainSoftware Basic Work Flow
For more info @ref codebrief \n
![Work Flow Diagram](/doc_Images/workflow.png "WorkFlow") \n
Basically the rack master has two modes
-# ***Idle Mode***: In this master deals with round robin of heaters and the mqtt communication'
-# ***Non Idle mode***: In this the master is either doing a unloading or loading operation of an order. \n
@subsection mode1 Idle Mode:
In this mode the master is not doing any operation of loading or unloading.Master pings all the slave boxes in the rack which is loaded for its temperature. Only 5 boxes in the rack is allowed to have heaters on for safety purpose. Master calculates the difference of the temperature of the box and the tmeperature at which the food should be. Then it commands the first 5 boxes with higher difference to turn on and all other to turn off. This is repeated in a specifc time. At this master is ready to receive the order.
@subsection mode2 Non Idle Mode:
There are 2 modes:
@subsubsection usermode User Mode(Unloading Mode):
-# User scans the rack Qr and orders the food on the website.
-# Broker assigns and publish ‘userotp’ topic with payload of a otp
-# Master stores the otp on the database subscribed from the topic(userotp)
-# When ‘usershowqrflag’ topic  is published, master displays the qr (Message with otp) and publishes a topic ‘showinguserqr’
-# When ‘userqrverifiedflag’ is published, master gets all the data from the database for that otp.From the collected data box numbers and skuids are taken and loading mode commands for the boxes are sent and publishes ‘delivered’ topic for the respective slots

@subsubsection deliverymode Delivery Mode(Loading Mode):
-# Same procedure like unloading mode

@note Always the rack will be subscribed to the MQTT broker.

@subsection init Initialization/restart:
-# Master populates the local sql database by subscribing to all topics for a rack.
-# Master reconfigure Communication ID for all the slots. Each slot is having a unique ID.Master assigns a temporary communication of 1 byte for communication between master and slave.
-# Performs health check for all the boxes. (For 3 times  if not responded declare as dead)
-# Send Configuration data 
-# Querying individual boxes if the configuration data is successfully initialized in all individual boxes , if not again send the configuration data.



@section Hardware 
Master box consists of: \n
-# Raspberry pi 3 B+ [Buy](https://www.thingbits.in/products/raspberry-pi-4-model-b-2-gb-ram)
-# 7Inch Lcd display with Touch [Buy](https://robu.in/product/7-inch-lcd-touch-display-with-driver-board-kit-for-raspberry-pi/)
-# Mispigy (from Inscribe)
-# RS485 Module (from Inscribe)
-# Matrix Keypad [Buy](https://robu.in/)


