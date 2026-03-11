# Remote Control of a Robotic Arm over a Local Network

Courses: Real-Time Operating Systems and Computer Networks Fundamentals \
Authors: Simon Radosavljević, Vuk Todorović, Milan Žeželj and Anja Ekres

## Introduction

<div style="text-align: justify"> 
This project presents the implementation of remote control of a robotic arm over a local network. It was developed as a final project for the courses Real-Time Operating Systems and Computer Networks Fundamentals.

This documentation contains a description of the program workflow, compilation instructions, explanations of the individual system components, and details regarding network communication.

The system consists of three parts:

1. Client application running on a computer used to remotely control the robotic arm.

2. Server application running on a Raspberry Pi that is directly connected to the robotic arm.

3. Robotic arm driver which communicates with the server on the Raspberry Pi.

All components are described in detail in the following sections. 

## Program Workflow and Remote Control

Before starting the system, it is necessary to compile and run the server and accompanying software on the Raspberry Pi, and compile and run the client on the computer that will be used as the remote controller (referred to as PC).

When the PC client starts, it begins searching for active servers using the UDP broadcast protocol.

A numbered list of available servers is displayed on the screen together with their usernames that were configured when starting the server on the Raspberry Pi.

The user can then select a server from the list and send a command to it.

The PC client attempts to establish a TCP connection with the selected server to send and receive commands.

After the command is executed on the robotic arm, the server sends feedback from the driver back to the client.

The previously established TCP connection is then closed, and the user is again given the option to select a server and send another command.

The program can be terminated using the command "exit".

## Compilation and Running the Program

### PC client

The PC client is compiled by navigating to the PC_client folder and running the makefile.

``` 
$ make
```

It is executed by running the file:

```
$ ./client
```

### Raspberry Pi Server and Driver

#### Log

While working with the driver, it is possible to monitor kernel logs in the console using:

```
$ dmesg -w
```

#### Driver

To compile the driver, navigate to the folder: *SW/Driver/servo_ctrl* and run makefile.

```
$ make
```

Then insert the module into the kernel using:

```
$ make start
```

#### Server

The server is integrated with an application that communicates directly with the driver. Navigate to *SW/Test/test_app_servo_ctrl*and execute:

```
$ ./waf configure
$ ./waf build
```

### Hardware Connection

#### Connecting the pins to the robotic arm.

![Pin connection diagram for the robotic arm.](pinovi.png)

#### Pin layout diagram for Raspberry Pi 2.

![Šema pinova na RPi2](sema.png)

## Detailed Description of System Components

### PC Client

The PC client scans the local network for active servers by sending a predefined message to the broadcast address using the UDP protocol.

It then begins receiving responses.

For each received message, the PC client stores:

the username

the IPv4 address from which the message was received

After receiving a message, the PC client sends an acknowledgment.

Once the scan is completed, the client displays a list of all available servers in the console.

The user can then enter a command in the following format:
```
[index] [w/r] [servo_idx] [w ? duty : null]
```
Where:

index – the index of the server to which the command is sent

w/r – write or read command

servo_idx – the index of the targeted servo motor

duty – provided only for write commands and represents the rotation coefficient of the motor

Based on the input, the PC client sends a TCP request to the selected server.

After sending the request, the client displays the driver's response on the screen.

The TCP connection is then closed, and the system waits for a new command, after which a new TCP connection will be established.

### Raspberry Pi server

The Raspberry Pi server waits for a UDP broadcast signal and sends its information back to the client via UDP.

It then waits for an acknowledgment or sends its information again if another signal is received.

After receiving the acknowledgment, it waits for a TCP connection.

Once the TCP connection is established, the server waits for a command and forwards it to the driver.

The server sends the driver's response back to the client and closes the TCP connection.

After that, it waits for a new TCP connection.

### Driver (Raspberry Pi)

The driver implementation is located in SW/Driver/main.c 
When compiled, the driver creates a device file /dev/servo_ctrl
Write and read operations on this file trigger commands for the servo devices.

The application communicates with the driver by receiving a string from the server in the format:

```
w servo_idx duty //Za pisanje
r servo_idx //Za čitanje
```

Where:

w – indicates a write operation to the driver file (servo motor rotation)

r – indicates a read operation (returns the current rotation angle of the motor)

servo_idx – ID of the targeted servo motor, value in [0,1,2,3]

duty – servo motors are controlled via a PWM pin, and the rotation is determined by the PWM duty cycle

Duty values range from 0-1000 which correspond to specific servo angles.

## Network Communication Details 

### Ports

UDP Broadcast: 25567 \
UDP: 25565 \
TCP: 25566

### UDP broadcast

The client sends a **"ping"** message via broadcast on port *25567*.  
When the server receives the **"ping"** message, it responds with its details **"$username"** on port *25565*.  
The client receives the server name and sends **"Received details"** as an acknowledgment.  
The server waits for the **"Received details"** message from the PC client. After receiving the confirmation, it opens a port for a **TCP connection**.

### TCP connection

The user selects a server from the list of available servers, and the client initiates a **TCP connection** to the selected server.  
The client sends a read/write command in the format **"[w/r] [servo_idx] [w ? duty : null]"**.  
The server receives the message and sends back the response from the driver, or **"Error: ErrorMsg"** if an error occurs.
