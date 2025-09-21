# RailWay Network Starter Board

## Overview

The "RwnStarterBoard" is block module conception board. This repository provide its  firmware 

![board_starter_3D.png](images/board_starter_3D.png)
## Blocks Structure


```mermaid
flowchart TD
	subgraph board
		Chip["PIC16F628A"]
		I2C_io["I2C_io(8xE/S)"]
		MxServoIO["MxServo_io(4x)"]
		Chip -->|"17-18"| SLI
		RS485 <-->|"6-8"| Chip
		Chip -->|"13,15-16"| MxServoIO
		Power -->|"5,14"| Chip
		Chip -->|"10-11"| Track["DCC/PWM"]
		Reset -->|"4"| Chip
		I2C_io <-->|"1-3"|Chip  
		
	end
	classDef .io fill:#24bf58
	classDef .power fill:#d6943a
	class RS485,I2C_io,Reset,SLI .io
	class Power,Track .power
```
## Firmware State Diagram

Below, a theoretical state diagram (not already implemented yet)  

```mermaid
stateDiagram

	classDef mainLoop fill:#70c1f1
	
	state if_reset <<choice>>
	state msg_type <<choice>>
	Reset : Reset Cfg
	MainLoop:::mainLoop
	[*] --> SLI_Init
	SLI_Init --> if_reset 
	if_reset --> Reset : Bt pressed
	if_reset --> Associating : No Bt pressed
	Reset --> Associating 
	Associating --> MainLoop
	state MainLoop {
		RxRS485: Read RS485
		TxRS485: Send RS485
		RxRS485 --> SLI_Busy 
		SLI_Busy --> msg_type 
		msg_type --> mxServoCmd : Ins=Grp FXX-FXX
		msg_type --> PowerPwm:  Ins=Speed & Mode=Pwm
		msg_type --> PowerDCC:  Addr=X & Mode=DCC
		msg_type --> i2cCmd :  Ins=Grp  FXX-FXX
		msg_type --> CfgCV:  Ins=CV Access
		mxServoCmd --> TxRS485
		PowerPwm --> TxRS485
		PowerDCC --> TxRS485
		i2cCmd --> TxRS485
		CfgCV --> TxRS485
		TxRS485 --> SLI_Waiting
	}
	MainLoop --> [*] : PowerOff
```
