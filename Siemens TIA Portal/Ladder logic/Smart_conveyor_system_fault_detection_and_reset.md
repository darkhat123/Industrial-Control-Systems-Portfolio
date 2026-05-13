# Introduction
In the previous conveyor belt system, we used ladder logic to program a conveyor system with a sensor and a pneumatic pusher to be able to autonomously sort parcels on
the belt, however in the previous example apart from having a safetycircuit to halt the process, however when the safetycircuit is restored, the state of the machine
has already been impacted and the timer will be reset, this means that the timers put in place to detect faults will be incorrect and may be delayed in stopping a fault
in the production process, which is likely to cause harm or danger to both the environment and human operators. The following process uses retentive timers, these
ensure that the state of the machine is properly tracked and that transitions into e-stop are prompt and ensured.

# Network 1 (Rung1)
This rung is dedicated to ensuring that the machine is in the initial state, its considered ready and the start button has been pressed, only then will we move into 
the next state
<img width="752" height="187" alt="image" src="https://github.com/user-attachments/assets/1942f469-e644-46a4-a0de-9ee793202a12" />
# Network 1 (Rung2)
This rung makes sure that if at any point the stop button is pressed or the faults are cleared, then the machine will revert back to its original idle state, we can later
implement self resetting actuators and inputs however for the time being it is assumed that the start button is not pressed until machine has been put into ready
state, this is only true when all interlocks are true, this will be demonstrated in later projects

<img width="610" height="197" alt="image" src="https://github.com/user-attachments/assets/492e336e-8e2f-4450-a4da-093758d76ec7" />

# Network 2
This network is designed to ensure that if any of the states output a discovered fault, that these will then illuminate the necessary warning lamp to ensure that the 
operator is aware of where the fault lies
<img width="1007" height="367" alt="image" src="https://github.com/user-attachments/assets/d38c1061-08b5-4046-b335-edb22c3fb2be" />

# Network 3
This is the event and transition section that starts the conveyor belt, and only stops the belt once the transition condition is met, ie when the sensor detects an
object, the safetycircuit is included here to ensure that the machine will halt if the safetycircuit is broken
<img width="1002" height="297" alt="image" src="https://github.com/user-attachments/assets/a42a5bc3-6239-4a98-a5d2-5a799dc19095" />

# Network 4 (Rung1)
This utilises the sensor not being activated for a set period of time as an indication that some fault has occured 


