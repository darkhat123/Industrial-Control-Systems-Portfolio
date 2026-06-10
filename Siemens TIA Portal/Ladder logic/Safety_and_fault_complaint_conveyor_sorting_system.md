# Introduction
This project focuses on demonstrating a safety and fault aware state machine used to control a conveyor belt sorting system, this utilises safety and fault interlocks to prevent the machine from running when the safetycircuit is breached, or a fault occurs in any stage of the state machine. The safety interlock and fault active combine to determine the machine being in the ready state, human interaction determines whether the machine moves into automatic mode, this automatic mode, is fed to the system permissive bit to ensure that no stages of the machine are active if any of the underlying conditions are breached. Given that all of these safety and fault condtions are satisifed, then the machine will begin operating. The four states that it cycles betwee are moving the box to the sensor, detecting the object, pushing the object and retracting the arm back. This allows the machine to operate idnependently and process the packages on the conveyor system. Additionally Object Orientted Programming was used to allow the model to scale to any number of pushers, motors and sensors without having to rewrite the logic for each of the components. Instead we define the strcuture common to every instance of the componenent and allow variables such as on/off delay to be specifiied at the creation of the instance. Furthermore encapsulation of each components inputs, outputs and data has been acheived via the use of function blocks, each instance of a sensor will have its own instance of the variables defined in its table. This means that all of the sensors can work on their own independent data and as their purpose is intended, they can take global inputs that may be used by all components and produce global outputs that may be used by more than the single component
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

# Safety Circuit ( Network 1 - Check all inputs, Energise the coil if all true)
This part of the program is concerned with ensuring that all of the safety features being monitored at any point by the PLC input are indeed true and therefore safe. These all utilise normally open contacts so that aslong as the power flows to this contact indicates a healthy status, such as Estop not pressed or guard door closed. This does not utilise a Set instruction for the coil as we only want this to be energised for aslong as these inputs remain true
<img width="867" height="213" alt="image" src="https://github.com/user-attachments/assets/34856a8f-e269-4655-ab43-a6d64ea1e1c2" />

# Safety Circuit ( Network 2 - Wait for system reset button and create a reset pulse bit)
In this network we are monitoring for a momentary press of the systemreset button using a p trig instruction, this uses an additional edge detection bit that ensures that the button has been released prior to allowing it to energise the circuit again. This prevents taping down of buttons by operators or situations where the button becomes stuck. Only the first press will have any effect on the subsequent rungs, unless the button is explicitly released between these points
<img width="806" height="250" alt="image" src="https://github.com/user-attachments/assets/d4f94ae8-e45c-41ed-9efe-47d8c118a455" />

# Safety Circuit ( Network 3 - Monitor for reset pulse, ensure safetycircuit healthy, then latch the safetycircuit coil )
Finally in this network we monitor for the system reset pulse for the one scan cycle, given this is pressed and their are no issues with safety at the present time then the safetycoil will become active and will latch until any of the safetyconditions are again broken
<img width="820" height="297" alt="image" src="https://github.com/user-attachments/assets/6ceaa073-95bf-49bc-9905-7678dbabb9bf" />


# Global Fault Reset circuit( Network 4 - Fault reset logic )
This circuit is placed before any of our permissives as it must be evaulated before me move to evaluating whether the machine is in a state to be set to ready, again the positive trigger is used to ensure no human error is involved in activating the reset. The reset button will clear all faults, so it is assumed these have all been assessed before beginning. if not they will quicky stop the machine again. The fault reset logic is implemented here so that the faults are cleared before the detection happens again, if the detection then finds another fault then the reset will come after and ensures that the machine does not stay active for the scan cycle.

<img width="700" height="372" alt="image" src="https://github.com/user-attachments/assets/eb34698d-f604-4004-bf9c-384af1daf09b" />

# Pneumatic Pusher Fault Reset circuit( Network 5 - Fault reset logic )
Just like the above reset circuit, this is simply focused on resetting local logic and ensures that the operator is aware of which faults are causing issues
<img width="736" height="257" alt="image" src="https://github.com/user-attachments/assets/84f41a20-a0a0-4871-a4a9-7d13ce805ebe" />

# Machine ready Circuit ( Network 6- ensure the machine is in the ready state)
In this circuit we are using the previously defined networks outputs from above to determine whether or not the machine is ready, given the safetycircuit is healthy, the stop button is healthy and not pressed, and no fault is active, then the machine is considered ready

<img width="717" height="242" alt="image" src="https://github.com/user-attachments/assets/2dfbecc3-23d0-4e14-83d3-b32549798d6a" />

# Start button one shot for Autocycle Active (Network 7 - Uses a p-trig to ensure the startbutton is actually pressed)
In this rung we use a startbuttons momentary press to feed to the p-trig block, this will then set the startsystempulse bit for one scan cycle, which will be fed to the next rung
<img width="685" height="218" alt="image" src="https://github.com/user-attachments/assets/6e52a97d-20d6-4657-9c5d-b6963d238c74" />


# AutocycleActive circuit (Network 8 - Used to allow the machine to enter auto mode)
This network is focused on checking that the machine is ready, the start button one shot has been energised, given these conditions are met then the autocycle will become active and will latch, this latch is dependent on the machine continuing to be in the ready state, if any of the inputs holding ready true become false, then this will also become deenergised
<img width="956" height="321" alt="image" src="https://github.com/user-attachments/assets/407b4b4d-cc97-4a73-aa66-653e02f679fb" />

# Systempermissive assigned (Network 9 - Used when the machine must actuate or change the state)
 The machine is now acting independently aslong as the previous routine does not change, this bit is used to ensure no actions are taken without this bit remaining true through the process
<img width="838" height="212" alt="image" src="https://github.com/user-attachments/assets/91553be4-34f6-4439-b635-1f823ad94f35" />

# State 0 - Checking to ensure that the sensor is not blocked before beginning
In this network we are using the comparator operator to compare the state of the state machine to the defined value, given its in this state it will check to see if the sensor is blocked, if the sensor is blocked for more than the specified time, a fault is raised and the system enters the fault state, however if the sensor is clear for long enough, then the state is progressed to the conveyor moving the box. Set logic is used with the fault to ensure that even if momentarily true for any of the faults, the fault is active until explicitly reset

<img width="1055" height="380" alt="image" src="https://github.com/user-attachments/assets/7a761897-2b5e-4300-aea0-fc97fff1ae88" />

# State 1 - Move the box until the sensor detects the box as being present
In the following network we utilise a function block for the internal logic associated with the sensor, the internal logic is simple for the function block, it ensures that the object sensor must first be on for a specified period of time before being recognised and must then be off for a certain period of time before the filtered output becomes true, this ensures that the sensor is truely registering a box as present and not malfunctioning. The actual logic in this network ensures that aslong as the object sensor is clear, the motor runs, when the sensor block registers a successful box detection, and the arm is retracted then the filtered output is passed to our state transition, if the state is active and the transition event is satisfied, the pusher extending stage begins, otherwise if the timer is exceeded then the conveyor fault is set and the machine becomes inactive.

<img width="1032" height="492" alt="image" src="https://github.com/user-attachments/assets/64fbcab7-059a-4ce6-ae36-fb31809809a0" />

# State 2 - Issue the push command to the pusher function block
Now we are in the pushing stage we pass this state directly into our extendcmd, aslong as the machine is at this state, the extend cmd will be issued, within the pusher function block logic is used to check that the arm is retracted and not extended before pushing, then it monitors to ensure the push takes the expected amount of time, once the arm is fully extended and confirmed with the global limit sensor for the pusher, the state transition in subsequent rungs checks that the pusher extended status is true and its in the push state, if so, then it moves on to the retract state. The sytem permissive is what enables the function block to run.

<img width="916" height="492" alt="image" src="https://github.com/user-attachments/assets/9afa320d-6d25-4fd0-9136-e6ca3795d7b6" />

# State 3- Issue the retract command to the pusher function block
Just like in the pushing state we ensure that the arm is extended and not retracted, if so we energise the retracting coil of the pusher, the retracting time is measured to ensure the pusher is operating as expected, given that the global retracted limit switch are active and subsequently the pusher retracted status becomes true, then we can begin our state transiton back to state 0.


# State transitions for the machine 
The following three networks are used to monitor for the state transition event and to assign a different value using the move Operator
<img width="543" height="160" alt="image" src="https://github.com/user-attachments/assets/28a46a54-6866-4b28-a062-9778623d0153" />
<img width="667" height="173" alt="image" src="https://github.com/user-attachments/assets/95e5d05d-820f-4c1c-a85e-39dcb23ea73c" />
<img width="731" height="216" alt="image" src="https://github.com/user-attachments/assets/9fcf0d04-1631-4ab0-8113-3ffe6c85804c" />

# Fault detection circuit
The final rung of the actual program is where we aggregate all the fault detections of the scan cycle and energise the fault active given any of the following faults become true, the logic in the beginning that allows the resets will always come after this, it ensures that if the reset is performed then the machine will not suddenly start if a fault is still active.
<img width="692" height="396" alt="image" src="https://github.com/user-attachments/assets/9606ad2a-c79e-4d23-a23f-b36f1f7ee65d" />

# Simulatiion toggle block
The final block simply ensures the simulation blocks values are used over the phsyical values of the plc program 
<img width="716" height="210" alt="image" src="https://github.com/user-attachments/assets/7e793f95-beee-4cc9-9027-0b9a7e8a8d46" />

# Defining the sensor function block
As seen in previous images the sensor block is used without having to know the underlying logic, all we need to do is configure the on and off delay for our specific sensor, and that instance of the sensor will have those values. 

The following shows all of the variables defined for the sensor
<img width="1093" height="557" alt="image" src="https://github.com/user-attachments/assets/6806614d-9f60-4a6d-8c3c-c86df42dd418" />

This image shows the actual logic used for the sensor
<img width="1091" height="382" alt="image" src="https://github.com/user-attachments/assets/3e713bd9-c0ae-4488-bfcf-ec9b846f9bed" />

# Defining the pusher function block
The following shows all of the variables configured for the pusher block
<img width="1117" height="600" alt="image" src="https://github.com/user-attachments/assets/f948afe6-6e6b-45cc-bb6c-2b6a349d599c" />

The following shows the underlying logic of the pneumatic pusher
<img width="632" height="407" alt="image" src="https://github.com/user-attachments/assets/65285108-ea3a-4f41-996a-282d3bef4d3a" />
<img width="717" height="367" alt="image" src="https://github.com/user-attachments/assets/ec3932b2-dafd-4112-a5ad-fcc37274d98a" />
<img width="922" height="382" alt="image" src="https://github.com/user-attachments/assets/dfefab86-267d-43f8-888d-befa1576135b" />
<img width="1086" height="393" alt="image" src="https://github.com/user-attachments/assets/e8df7cab-8fa4-44a7-874d-dd8dafe5bd1b" />
<img width="1090" height="331" alt="image" src="https://github.com/user-attachments/assets/8e93272c-4a07-4b50-879c-a21c56ed860a" />
<img width="1095" height="292" alt="image" src="https://github.com/user-attachments/assets/6007a7f4-2b7b-4e18-92fa-689156411b27" />











# Simulation and testing the logic 
Before we can start the simulation and ensure that the new code is downloaded to the device, we must ensure that we can easily monitor and modify all of the key values that we will interact with in a watch table. This allows us to easily toggle bits on and ensure that bits that we need to force for testing operations are easily accessible. 

Watch tables can interact with many different places containing our variables including
- PLC Tags, this is the defacto place where we assign our physical inputs and outputs and even memory addresses, these are the physical addresses of the PLC and typically hold global input and output variables linked to I/O especially, this is typically simple integers and booleans
- Data blocks, these are used to organise certain variables that may be necessary for complex data sets or recipes that require many variables, these are globally accessible but instance db's can be speficic to one single function> These can hold complesx structures like complex nested arrays or user definied types.

With the watch table defined and all of the necessary values matched to a specific entry in the watch table, we can begin planning how we intend to demonstrate the state of these variables to a human operator, and how we allow them to be able to influence the state of these variables. In our scenario we want to be able to keep everything in the one place for easy access and confgiuration. Furthermore we do not have a machine or network of machines big enough to warrant integrating into a SCADA system.

# Adding a HMI

- Add a device
- Select an appropriate HMI
- Accept all the defaults

When this is added into the program, the HMI can directly communicate with the PLC via the ethernet connection between them




