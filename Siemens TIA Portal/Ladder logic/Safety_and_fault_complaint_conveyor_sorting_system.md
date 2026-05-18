# Introduction
This project focuses on demonstrating a safety and fault aware state machine used to control a conveyor belt sorting system, this utilises safety and fault interlocks to prevent the machine from running when the safetycircuit is breached, or a fault occurs in any stage of the state machine. The safety interlock and fault active combine to determine the machine being in the ready state, human interaction determines whether the machine moves into automatic mode, this automatic mode, is fed to the system permissive bit to ensure that no stages of the machine are active if any of the underlying conditions are breached. 
# Inputs and Outputs

## Safety Variables 
- Guard door feedback, physically wired NC meaning a healthy unopened door allows power to flow to the input, BOOL,NO contact
- Global Estop not pressed, wired NC also - BOOL,NO contact
- Safety Sensor Feedback, wired NC - BOOL, NO contact
- Safetyresetbutton, logical no used to detect press of button, BOOL, NO contctact
- Seftyresetbit, logical bit used to represent whether button was pressed and released each scan cycle, true until released, once false the action can be triggered again, prevents the machine bypassing safetyreset, BOOL
- Safetycircuit, the bit uses to represent whether all safety requirements have been met, calculated an an output of the condistions and then passed back as input to latch the safetycircuit so long as all conditions stay met, BOOL, COIL and NO contact


 ## Fault Variables - Detection and Resetting
 - Pusherextfault, the pusher has been in the extending phase and has detected a fault with the extension process, the output is set using a SET bit to ensure the fault is active till an operator clears it, this is then fed in to a faultactive variable which waits for any of the faults to become active before transitioning into the faultactive state, any number can be on and the same effect is had, the burden of ensuring faults are cleared are passed to the human operator, BOOL, COIL, NO used to detect, Set and reset for latch
 - pusherretfault, same as pusher but detects an issue during the retracting stage, BOOL, COIL, NO used to detect, Set and reset for latch
 - conveyorfault, detects any errors in the delivery of the box to the sensor and flings an error to check the conveyor for blockages or for issues with the motor, BOOL, COIL, NO used to detect, Set and reset for latch
 - fault_cleared_pb, this button waits for the operator to press to reset the fault, this sends a trigger to the p trig fucntion block which one shot resets the faults until released and pressed again, BOOL, NO to detect
 - fault_cleared_bit, this is used to remember whether the button has been released between scan cycles, BOOL
 - fault_active, this bit uses the set logic used in any of the fault detection logic to determine whether there is an active fault, this is latched on the condition all faults remain inactive, any occur and this will become, BOOL, COIL, NO for latch
 - conveyorfaultlamp, this bit is used to light up the lamp to let perators know exactly where the fault lies, BOOL, COIL
 - pusherfaultlamp, same as conveyorfault, both extend and retract light up the same light, however their inputs will be used to indicate the error separately in the HMI, BOOL, COIL


## Autocycle initialisation and Opearation variables
- Startbutton, the input used to force the macbine into auto mode where it can work autonomously, NO, BOOL
- Stopbutton, input used to monitor that the stop button is healthy, any cut to the power via fault or operation will halt the circuit, Wired NC physically, NO monitors health, BOOL
- Ready, used to check that the machines safetycircuit is intact and there are no faults active, once defined it prevents the use of multiple contacts, BOOL, COIL
- Autocycleactive, this is the bit to represent the machine being allowed to enter its automatic mode, and is latched based on the underlying conditions remaining true, BOOL, COIL and NO for latch
- Systempermissive, this is the final variable assigned once autocycleactive is assigned, this will authorise the machine to perform operations and actuations solong as all underlying logic is true, BOOL, COIL

## State machine Variables
- State, integer value which is used to increment through each of the stages via assignment operators and comparison operators, this ensures that only once the event in the previous stage has occured and the transition conidtion is satisfied, that the new state will begin, allowing us to have an assurance that we are indeed in this stage of the process, INT

## Physical Sensors and Actuators
- Objectsensor, a digital input used to detect the presence of the box in normal operation and the lack of a genuine box in fault situations, NO contact to detect a healthy sensor, NC to detect unhelathy sensor, BOOL
- Motor, this powers the conveyor belt and ensures that the box reaches the pusher, BOOL, COIL
- Arm_retracted, used to detect if the arm has fully retracted, NO to detect healthy arm, NC to detect faulty arm, BOOL
- Arm extended, same as the retracted, BOOL
- Pneumatic, used to energise the solenoid and push the arm, when deenergised, the arm will return to its original retracted position
- 

# State machine flow 
<img width="1363" height="565" alt="image" src="https://github.com/user-attachments/assets/eb964ae7-ea7d-4ae6-b7d2-8075092ec9fc" />
## Safety Logic diagrman

<img width="294" height="542" alt="image" src="https://github.com/user-attachments/assets/d3fdd17d-e7ef-4d4b-90e0-4b0424649a58" />

## Fault Detection and Reset Logic diagrman

<img width="294" height="542" alt="image" src="https://github.com/user-attachments/assets/cc60c0c0-586b-4c7a-9cc7-0737f0ae0ac8" />

# Safety Circuit ( Network 1)
This part of the program is concerned with ensuring that all of the safety features being monitored at any point by the PLC input are indeed true and therefore safe. These all utilise normally open contacts so that aslong as the power flows to this contact indicates a healthy status, such as Estop not pressed or guard door closed. It uses a positive trigger block to monitor for the press of the safety reset button which is pressed when all of the inputs are assumed to be true for the Human operator. Only when these conditions are all true, and when the button is pressed for a single scan cycle, will the safetycircuit be activated, if for any reason, any of these conditions become untrue, the latch will prevent the circuit from remaining true and will force it back into evaluating the conditions in the next scan, this of course will also halt the production process. The positive triggers purpose is to nesure that the safety circuit can only be reset with each button press, any button jams or issues will not be registered past the current scan cycle, the state of whether the button has been released prior id monitored by the safetyresetbit and ensures that the logic wont fire unless released and pressed again.Whilst the contacts in the code are normally open, many of these safety features would be NC, where there healthy state is maintained via the closed contact, any issues and that will open, and the sensor input will become deenergised and stop the circuit.
<img width="736" height="294" alt="image" src="https://github.com/user-attachments/assets/5df3e497-d0e8-408c-aad6-8192469372b4" />

# Fault detection circuit
Placing the detection here ensures that safety is assessed first, then faults are assessed so any changes in the fault logic state is propogated downwards and doesnt require a new scan cyle to apply any resets that may be used and the system being ready only when no faults are detected.The logic has been used to ensure that if any of the faults are active then the machine is in fault state and is latched unless otherwise reset.
<img width="648" height="353" alt="image" src="https://github.com/user-attachments/assets/d3419daf-c0c9-4a3e-accd-5947ea64986a" />

# Fault reset circuit
This circuit is placed before any of our permissives as it must be evaulated before me move to evaluating whether the machine is in a state to be set to ready, again the positive trigger is used to ensure no human error is involved in activating the reset. The reset button will clear all faults, so it is assumed these have all been assessed before beginning. if not they will quicky stop the machine again.

<img width="648" height="261" alt="image" src="https://github.com/user-attachments/assets/f1a7a8d3-0fa5-47c1-a8ab-dfb0353b8555" />

# Machine ready logic
This uses both the output of the above networks to ensure that the machine is indeed ready before allowing any interaction from the operator to affect the machines state

<img width="648" height="213" alt="image" src="https://github.com/user-attachments/assets/0516af92-ace9-4669-9e55-4c4f600e9234" />

# Autocycle active logic
In this rung we pass the permissives output down and use our inputs to determine if the machine is indeed ready to begin working automatically and independent of human interaction, the fault and safety conditions are checked here again to ensure that the machine is ready, when the startbutton is pressed, if no issues are present in the stop button then the machine goes into auto mode and latches here

<img width="648" height="246" alt="image" src="https://github.com/user-attachments/assets/80fc17b5-d416-4523-a405-b9dd30b5f7bc" />

# Systempermissive assigned
 The machine is now acting independently aslong as the previous routine does not change, this bit is used to ensure no actions are taken without this bit remaining true through the process
<img width="648" height="180" alt="image" src="https://github.com/user-attachments/assets/83aa2c3d-ece3-48a9-ab30-79dbef9a05ce" />

# Idle state
In this rung we use the system permissive to ensure that the rung operates given this is active, then we check to see if the object sensor is clear, if the object sensor is clear then the machine moves into the motor running state
<img width="648" height="243" alt="image" src="https://github.com/user-attachments/assets/13322518-a8ef-480f-b5ec-faf617aa31bc" />

# Motor running and waiting for valid box detection, ensuring the arm is retracted before switching to the pushing state
In this rung we ensure that aslong as the object sensor is clear that the motor is running, the next rung focuses on detecting if the sensor is indeed acitvated for 2s to ensure that this is indeed a valid box, then it will move into the next state, given that the arm of the pusher is also retracted. If the sensor does not detect an object in a given amount of time then a fault is set in this scan cycle stopping the system in the next scan cycle fault circuit.
<img width="1046" height="414" alt="image" src="https://github.com/user-attachments/assets/98626b5f-4c9b-49ca-b8c9-c242e704058f" />
# Pusher active and waiting for the pusher to extend
With the last state only switching when the arm is ensured to be retracted and the object has been detected, the state in the next rung implies this is true, so the pusher begins pushing the box aslong as the arm is not extended, ensuring it is killed the second the arm is fully extended. If the pusher becomes extended for long enouch then the next state begins to retract the pusher back, if the pusher is not extended after a given time the fault bit is set.
<img width="1046" height="431" alt="image" src="https://github.com/user-attachments/assets/4803f6c6-1a03-4942-9aae-eee5c311b882" />
# Pusher inactive and waiting to retract
IN this rung we dont need to energise any coil, since the solenoid is powered one way and only returns when deenergised we just need to monitor to ensure the arm is retracted before feeding the next box in to the sequence, if it returns then the state changes, if not the fault is logged and the bit is set, halting the machine
<img width="1046" height="372" alt="image" src="https://github.com/user-attachments/assets/8614f3d4-e549-4253-bbf2-3d31f5631d64" />
