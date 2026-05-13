# Introduction
This project focuses on demonstrating a safety aware state machine used to control a conveyor belt sorting system, this utilises interlocks to prevent the machine from running when the safetycircuit is breached, the state machine is used to ensure that every event that is expected occurs and that once the event is complete it can safely transition to the next stage. The following physical inputs and outputs are:

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


# Program

## Initialising circuit
This ensures that only when the system is determined ready, the state is in idle and the start button is pressed will the next stage begin.
The second rung is focused on ensuring that if the safetycirucit is broken or the stopbutton is pressed that the machine will return itself to the idle state

<img width="692" height="387" alt="image" src="https://github.com/user-attachments/assets/0e89e5ae-7900-49db-99e2-93f84598eb2c" />

## Start the conveyor until an object reaches the sensor
This ensures that the conveyor coil is energised until the object sensor is energised, which will kill the conveyor and move to the pneumatic pusher stage

<img width="780" height="312" alt="image" src="https://github.com/user-attachments/assets/05dad075-41da-4eaa-86e2-46eaf0fd13a8" />

## Conveyor stopped and arm ready, begin pushing
This ensures that the pusher is in its intended position, and uses a set bit to latch the pusher, it then waits for the sensor to be fully extended before moving to the next stage

<img width="723" height="325" alt="image" src="https://github.com/user-attachments/assets/6bc03a1e-f24f-48db-a57b-94b775f0499e" />

## Conveyor fully extended, begin retracting, once retracted return to stage 0
<img width="722" height="282" alt="image" src="https://github.com/user-attachments/assets/4d9346a0-3580-4f35-a81d-0337335f81ce" />




