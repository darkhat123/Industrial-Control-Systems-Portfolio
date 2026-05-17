# Introduction
This project focuses on demonstrating a safety and fault aware state machine used to control a conveyor belt sorting system, this utilises safety and fault interlocks to prevent the machine from running when the safetycircuit is breached, or a fault occurs in any stage of the state machine. The safety interlock and fault active combine to determine the machine being in the ready state, human interaction determines whether the machine moves into automatic mode, this automatic mode, is fed to the system permissive bit to ensure that no stages of the machine are active if any of the underlying conditions are breached. 
## Inputs
- Startbutton
- Stopbutton
- Safetycircuit
- ObjectSensor
- Retracted_arm_sensor
- Extended_arm_sensor
- State
- Ready

## Outputs
- Motor
- Pneumatic pusher

# State machine flow 



<img width="1363" height="565" alt="image" src="https://github.com/user-attachments/assets/eb964ae7-ea7d-4ae6-b7d2-8075092ec9fc" />

## Safety Circuit
This part of the program is concerned with ensuring that all of the safety features being monitored at any point by the PLC input are indeed true and therefore safe. These all utilise normally open contacts so that aslong as the power flows to this contact indicates a healthy status, such as Estop not pressed or guard door closed. It uses a positive trigger block to monitor for the press of the safety reset button which is pressed when all of the inputs are assumed to be true for the Human operator. Only when these conditions are all true, and when the button is pressed for a single scan cycle, will the safetycircuit be activated, if for any reason, any of these conditions become untrue, the latch will prevent the circuit from remaining true and will force it back into evaluating the conditions in the next scan, this of course will also halt the production process. The positive triggers purpose is to nesure that the safety circuit can only be reset with each button press, any button jams or issues will not be registered past the current scan cycle, the state of whether the button has been released prior id monitored by the safetyresetbit and ensures that the logic wont fire unless released and pressed again.Whilst the contacts in the code are normally open, many of these safety features would be NC, where there healthy state is maintained via the closed contact, any issues and that will open, and the sensor input will become deenergised and stop the circuit.
<img width="736" height="294" alt="image" src="https://github.com/user-attachments/assets/5df3e497-d0e8-408c-aad6-8192469372b4" />

## Fault detection and reset circuit
This circuit is placed before any of our permissives as it must be evaulated before me move to evaluating whether the machine is in a state to be set to ready, again the positive trigger is used to ensure no human error is involved in activating the reset. The logic has been used to ensure that if any of the faults are active then the machine is in fault state and is latched unless otherwise reset
<img width="648" height="353" alt="image" src="https://github.com/user-attachments/assets/d3419daf-c0c9-4a3e-accd-5947ea64986a" />
<img width="648" height="213" alt="image" src="https://github.com/user-attachments/assets/0516af92-ace9-4669-9e55-4c4f600e9234" />
<img width="648" height="246" alt="image" src="https://github.com/user-attachments/assets/80fc17b5-d416-4523-a405-b9dd30b5f7bc" />
<img width="648" height="180" alt="image" src="https://github.com/user-attachments/assets/83aa2c3d-ece3-48a9-ab30-79dbef9a05ce" />

<img width="648" height="261" alt="image" src="https://github.com/user-attachments/assets/f1a7a8d3-0fa5-47c1-a8ab-dfb0353b8555" />
<img width="648" height="243" alt="image" src="https://github.com/user-attachments/assets/13322518-a8ef-480f-b5ec-faf617aa31bc" />
<img width="1046" height="414" alt="image" src="https://github.com/user-attachments/assets/98626b5f-4c9b-49ca-b8c9-c242e704058f" />
<img width="1046" height="431" alt="image" src="https://github.com/user-attachments/assets/4803f6c6-1a03-4942-9aae-eee5c311b882" />
<img width="1046" height="372" alt="image" src="https://github.com/user-attachments/assets/8614f3d4-e549-4253-bbf2-3d31f5631d64" />
