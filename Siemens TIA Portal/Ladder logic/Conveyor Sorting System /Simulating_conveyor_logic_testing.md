# Simulation and testing the logic
When working with PLC programs in tia portal, it can be difficult to manually assign and force values and navigate watch/force tables to be able to effectively test the workings of a conveyor sorting system. In these situations, we need a method to be able to manipulate all of the program's variables in a repetitive, time-based manner. This is where simulation blocks come in, simulation blocks are placed at the very end of the program and act as a direct snapshot of the effect of the injected variable, with a simulation block, the plc runs the program top to bottom, and before going to the next scan cycle, updates the process image to contain the current values for all variables and then reaches the block where the simulation runs, with the correct process values now in the simulation table, we can utilise the case operation and boolean logic, the case statememnt is used to sequentially perform operations on the programs variables based on the outcomes of prior branches, this allows us to design an effective testing program that reacts to any situation likely to occur with the machine.


Before we can start the simulation and ensure that the new code is downloaded to the device, we must ensure that we can easily monitor and modify all of the key values that we will interact with in a watch table. This allows us to easily toggle bits on and ensure that bits that we need to force for testing operations are easily accessible. 

Watch tables can interact with many different places containing our variables, including
- PLC Tags, this is the de facto place where we assign our physical inputs and outputs and even memory addresses. These are the physical addresses of the PLC and typically hold global input and output variables linked to I/O, especially, this is typically simple integers and booleans
- Data blocks, these are used to organise certain variables that may be necessary for complex data sets or recipes that require many variables. These are globally accessible, but instance DBs can be specific to a single function. These can hold complex structures like complex nested arrays or user-defined types.

With the watch table defined and all of the necessary values matched to a specific entry in the watch table, we can begin planning how we intend to demonstrate the state of these variables to a human operator and how we allow them to be able to influence the state of these variables. In our scenario, we want to be able to keep everything in one place for easy access and configuration. Furthermore, we do not have a machine or network of machines big enough to warrant integrating into a SCADA system.

# Using function blocks for simulation Logic

Whilst we do have the options to sit and force values manually and watch the behaviour of the program dependent on the inputs and outputs we configure, however this is tedious and furthermore can miss nuances in the program that would not be possible to detect with the naked eye. in these scenarios the only way to make sure the machine is working correctly is to physically use testing logic such as SCL to manipulate the variables in the program and determine the effect they have on the programs behaviour under certain scenarios, this allows for rapid testing of all fault scenarios for a component and allows these tests to be repeated on any sensor we use. Furthermore, it allows us to make sure the program is safe to deploy to an actual industrial automation environment, before we actually commission it.

# Why SCL for simulation Logic

Again, we will use a function block to create our simulation program, and instead of choosing LAD, we will choose SCL, whilst ladder is preferred for troubleshooting technicians, simulation logic will likely be interacted with by ourselves or used by those competent in many PLC programming languages. Furthermore, whilst ladder logic is simple to read and understand, it is notoriously difficult to perform a test on ladder logic using another ladder logic program. Whereas when using SCL we have the case statement available, this statement allows us to control the behaviour of the simulation based on the conditions we specify in each case block, for example case 1 might check to see if a button is pressed or not, and choose between two integer values dependent on the outcome, this allows the simulation to react to multiple options and ensures it doesnt get stuck if no other option is available

# Using Multi-instance timers to reduce database instances
When writing the simulation, we had to make sure to use some timers; these will be multi-instance timers, which will be used many times across the program and have the same presets for all of them. Their main purpose is to allow time for any of the operations we override to take effect in the next scan cycle. Multi-instance timers reduce the need for individual DB's for the timers; the function block handles the timer logic and structure within its own memory.

# Creating a toggle for enabling and disabling the simulation block 
Currently, whilst we have created our function block that will hold all of the simulation logic in one place, this will have no effect on any of the variables within the process image or in memory. We must ensure that the function block is called in our main program before we can begin manipulating variables and using time-based error detection in SCL. This is placed on a rung with a NO contact placed in front of it; this is a simulation-active contact. When true, the function block becomes energised, and the simulation can read and write from the same areas as the process image.

<img width="661" height="173" alt="image" src="https://github.com/user-attachments/assets/6c5e22b6-ce83-468d-b58b-c9d6bfd0fe3f" />


# Defining the simulation function block structure
With the simulation block now created and it placed in the correct place in the program, we can begin to define all of the necessary variables that the simulation logic will need to perform its tasks. 

## Variable	Purpose
- Start_test - Master trigger used to begin execution of the automated test sequence.
- TestStep - State machine that controls progression through the individual test cases and validation stages.
- TestSuccess - Indicates that every validation step completed successfully and the full test sequence passed.
- Testfault -	Indicates that a test failure occurred and the sequence entered the fault state.
- Faultid -	Stores a unique identifier corresponding to the specific test that failed, allowing rapid troubleshooting.

## Simulation Timing Variables
**SafetyDropTimer**

Used to verify that safety-related functions respond within an expected time window after a safety input is intentionally broken. The timer provides a timeout mechanism to detect failures where the safety circuit remains active longer than expected.

**ResetPulseTimer**

Used throughout the test framework to generate realistic operator button presses for reset and start commands. It simulates a momentary push button rather than a continuously held input.

**State0Timer**

Introduces a delay before evaluating the machine's startup state logic, ensuring the controller has sufficient time to process initial conditions.

**State0FaultTimer**

Used to validate fault generation during startup conditions. If the machine remains in the initial state beyond the allowable period, this timer confirms that the expected fault is raised.

**True_sensor_on_time**

Simulates a valid object detection by holding the virtual sensor active long enough to satisfy the configured debounce/on-delay requirements.

**True_sensor_off_time**

Simulates the object leaving the sensor and provides sufficient time for the sensor's off-delay filtering logic to complete.

**Extenson_retracton**

Represents the physical travel time of the pneumatic pusher during both extension and retraction movements. This creates a realistic simulation of actuator motion rather than instantaneous position changes.

**On_delay_limit_sensor**

Simulates the delay between a physical actuator reaching its position and the corresponding limit switch being recognised by the control logic. This validates correct handling of position feedback and sensor filtering.

# The SCL simulation Code
``` // --- Call Timers Every Scan ---
// The input (IN) checks if the state machine is currently on a step that needs that timer.
#SafetyDropTimer(IN := ("Data_block_1".TestStep = 7 OR "Data_block_1".TestStep = 11 OR "Data_block_1".TestStep = 15 OR "Data_block_1".TestStep = 21 OR "Data_block_1".TestStep=25 OR "Data_block_1".TestStep=31),
                 PT := T#200ms);

#ResetPulseTimer(IN := ("Data_block_1".TestStep = 3 OR "Data_block_1".TestStep = 8 OR "Data_block_1".TestStep = 12 OR "Data_block_1".TestStep = 16 OR "Data_block_1".TestStep = 22 OR "Data_block_1".TestStep = 27 OR "Data_block_1".TestStep=32 OR "Data_block_1".TestStep=37 OR "Data_block_1".TestStep=39),
                 PT := T#500ms);

#State0Timer(IN := ("Data_block_1".TestStep = 35),
             PT := T#1000ms);

#State0FaultTimer(IN := ("Data_block_1".TestStep = 36),
                  PT := T#5000ms);

#True_sensor_on_time(IN := ("Data_block_1".TestStep = 42),
                  PT := T#120ms);


#True_sensor_off_time(IN := ("Data_block_1".TestStep = 43),
                      PT := T#120ms);
#Extenson_retracton(IN := ("Data_block_1".TestStep = 45 OR "Data_block_1".TestStep = 48),
                    PT := T#3000ms);
#On_delay_limit_sensor(IN := ("Data_block_1".TestStep = 46 OR "Data_block_1".TestStep = 49),
                       PT := T#220ms);

CASE "Data_block_1".TestStep OF
        
    0:  // --- IDLE STATE ---
        IF "Data_block_1".Start_test THEN
            "Data_block_1".TestSuccess := FALSE;
            "Data_block_1".Testfault := FALSE;
            "Data_block_1".Faultid := 0;
            "Data_block_1".TestStep := 1;
        END_IF;
        
    1:  // --- INITIALIZATION for sefetycircuit test ---
        "Data_block_1".Global_estop := TRUE;
        "Data_block_1".guarddoor := TRUE;
        "Data_block_1".Safetyfeedbacksenssor := TRUE;
        "Data_block_1".System_reset := FALSE;
        "Data_block_1".TestStep := 2;
    2: // --- Check and see if safetycircuithealthy ---
        IF "Data_block_1".Safetycircuithealthy THEN
            "Data_block_1".TestStep := 3;
        END_IF;
    3:  // --- AUTO RESET TRIGGER (Pulse On) ---
        "Data_block_1".System_reset := TRUE;
        IF #ResetPulseTimer.Q THEN
            "Data_block_1".TestStep := 4;
        END_IF;
        
    4:  // --- AUTO RESET TRIGGER (Pulse Off) ---
        "Data_block_1".System_reset := FALSE;
        "Data_block_1".TestStep := 5 ;
        
    5: // --- Ensure faultactive is not active ---
        "Data_block_1".faultactive := FALSE;
        "Data_block_1".TestStep := 6;
        
    6: // -- Fail the global estop, ensure safetycircuit is not active
        "Data_block_1".Global_estop := FALSE;   
        "Data_block_1".TestStep := 7;
    7: // -- Give time for the safetycircuit to denergise then determine if it eas successful at stopping it ---
        IF "Data_block_1".Safetycircuitcoil = FALSE THEN
            "Data_block_1".Global_estop := TRUE; // Heal it
            "Data_block_1".TestStep := 8;
        ELSIF #SafetyDropTimer.Q THEN
            "Data_block_1".Faultid := 6;        // Error: E-Stop Failed to drop safety!
            "Data_block_1".TestStep := 999;
        END_IF;
        
    8: // Hold reset button down for the required duration
        "Data_block_1".System_reset := TRUE;
        IF #ResetPulseTimer.Q THEN
            "Data_block_1".TestStep := 9;
        END_IF;
        
    9: // Release reset button, wait for system to recover
        "Data_block_1".System_reset := FALSE;
        "Data_block_1".TestStep := 10;
        
    10: // Break Door Guard input
        "Data_block_1".guarddoor := FALSE;
        "Data_block_1".TestStep := 11;
        
    11: // Wait for safety drop or timeout
        IF "Data_block_1".Safetycircuitcoil = FALSE  THEN
            "Data_block_1".guarddoor := TRUE; // Heal it
            "Data_block_1".TestStep := 12;
        ELSIF #SafetyDropTimer.Q THEN
            "Data_block_1".Faultid := 10;     // Error: Door Failed to drop safety!
            "Data_block_1".TestStep := 999;
        END_IF;
        
    12: // Hold reset button down for the required duration
        "Data_block_1".System_reset := TRUE;
        IF #ResetPulseTimer.Q THEN
            "Data_block_1".TestStep := 13;
        END_IF;
        
    13: // Release reset button, wait for system to recover
        "Data_block_1".System_reset := FALSE;
        "Data_block_1".TestStep := 14;
        
    14: // Break safetysensor feedback
        "Data_block_1".Safetyfeedbacksenssor := FALSE;
        "Data_block_1".TestStep := 15;
    15: // Wait for safety drop or timeout
        IF "Data_block_1".Safetycircuitcoil = FALSE THEN
            "Data_block_1".Safetyfeedbacksenssor := TRUE; // Heal it
            "Data_block_1".TestStep := 16;
        ELSIF #SafetyDropTimer.Q THEN
            "Data_block_1".Faultid := 14;     // Error: Light Curtain Failed to drop safety!
            "Data_block_1".TestStep := 999;
        END_IF;

        
    16: // --- Pulse the reset ---   
        "Data_block_1".System_reset := TRUE;
        IF #ResetPulseTimer.Q THEN
            "Data_block_1".TestStep := 17;
        END_IF;
    17: // Release reset button
        "Data_block_1".System_reset := FALSE;
        "Data_block_1".TestStep := 18;
    18: // Initialising variables to test the ready variable
        "Data_block_1".stopbutton := TRUE;
        "Data_block_1".startbutton := FALSE;
        "Data_block_1".TestStep := 19;
        
    19: //Ensure ready is set before testing
        IF "Data_block_1".Ready THEN
            "Data_block_1".TestStep := 20;
        END_IF;
        
    20: // Begin testing to check if ready will denergise with an issue in any signal
        "Data_block_1".Conveyorfault :=TRUE;
        "Data_block_1".TestStep := 21;
        
    21: // Allow time for the faultactive state to become true and stop the circuit
        IF "Data_block_1".Ready = FALSE THEN
            "Data_block_1".Conveyorfault := FALSE; // Heal it
            "Data_block_1".TestStep := 22;
        ELSIF #SafetyDropTimer.Q THEN
            "Data_block_1".Faultid := 20;     // Error: Light Curtain Failed to drop safety!
            "Data_block_1".TestStep := 999;
        END_IF;
     
        
    22: // Now that the conveyorfault is active we must clear this fault with the reset button
        "Data_block_1".faultcleared := TRUE;
        IF #ResetPulseTimer.Q THEN
            "Data_block_1".TestStep := 23;
        END_IF;
        
    23: // Now turn off the faultclearedd button
        "Data_block_1".faultcleared := False;
        "Data_block_1".TestStep := 24;
        
        
    24: // Begin testing to check if ready will denergise with an issue in any signal
        "Data_block_1".stopbutton := FALSE;
        "Data_block_1".TestStep := 25;
        
    25: // Allow time for the faultactive state to become true and stop the circuit
        IF "Data_block_1".Ready = FALSE THEN
            "Data_block_1".stopbutton := TRUE;
            "Data_block_1".TestStep := 26;
        ELSIF #SafetyDropTimer.Q THEN
            "Data_block_1".Faultid := 24;     // Error: Light Curtain Failed to drop safety!
            "Data_block_1".TestStep := 999;
        END_IF;
        
    26: //--- Given ready is true, begin testing autocycleactive
        IF "Data_block_1".Ready THEN
            "Data_block_1".TestStep := 27;
        ELSE
            "Data_block_1".Faultid := 26;
            "Data_block_1".TestStep := 999;
            
        END_IF;
        
    27: // One-shot Trigger the startbutton to begin the machine
        "Data_block_1".startbutton := TRUE;
        IF #ResetPulseTimer.Q THEN
            "Data_block_1".TestStep := 28;
        END_IF;
        
    28: // Release the startbuttong
        "Data_block_1".startbutton := FALSE;
        "Data_block_1".TestStep := 29;
        
    29: //determine if the machine is now in auto mode
        IF "Data_block_1".Autocycleactive THEN
            "Data_block_1".TestStep := 30;
        ELSE
            "Data_block_1".Faultid := 27;
            "Data_block_1".TestStep := 999;
        END_IF;
    30: // kill the stop button to ensure autocycle active will die
        "Data_block_1".stopbutton := FALSE;
        "Data_block_1".TestStep := 31;
    31: //Check if the effect has been realised
        IF "Data_block_1".Autocycleactive = FALSE THEN
            "Data_block_1".stopbutton := TRUE;
            "Data_block_1".TestStep := 32;
        ELSIF #SafetyDropTimer.Q THEN
            "Data_block_1".Faultid := 30;
            "Data_block_1".TestStep := 999;
        END_IF;
    32: // Pulse the startbutton to reenable autocycleactive
        "Data_block_1".startbutton := TRUE;
        IF #ResetPulseTimer.Q THEN
            "Data_block_1".TestStep := 33;
        END_IF;
    33: // Turn the start button off
        "Data_block_1".startbutton := False;
        "Data_block_1".TestStep := 34;
        
    34: // Ensure system permissive is set which will be if autocycleactive is true
        IF "Data_block_1".SytemPermissive THEN
            "Data_block_1".TestStep := 35;
        ELSE
            "Data_block_1".Faultid := 33;
            "Data_block_1".TestStep := 999;
        END_IF;
        
    35: // Test to see if state 0 correctly evaluates the sensor conditions at startup
        "Data_block_1".objectsensor := FALSE;
        IF #State0Timer.Q THEN
            "Data_block_1".TestStep := 36;
        END_IF;
    36: // Test to see if the state moves to state 1 correctly
        IF "Data_block_1".State = 1 THEN
            "Data_block_1".TestStep := 41;
        ELSIF "Data_block_1".State = 0 THEN
            IF #State0FaultTimer.Q THEN
                IF "Data_block_1".Conveyorfault = TRUE THEN
                    "Data_block_1".TestStep := 37;
                ELSE
                    "Data_block_1".Faultid := 35;
                    "Data_block_1".TestStep := 999;
                END_IF;
            END_IF;
        END_IF;
    37: // Clear the conveyor fault before continuing
        "Data_block_1".faultcleared := TRUE;
        IF #ResetPulseTimer.Q THEN
            "Data_block_1".TestStep := 38;
        END_IF;
    38: // Ensure fault cleared is not active again
        "Data_block_1".faultcleared := FALSE;
        "Data_block_1".TestStep := 39;
    39: // Reenable the machine with the start button
        "Data_block_1".startbutton := TRUE;
        IF #ResetPulseTimer.Q = TRUE THEN
            "Data_block_1".TestStep := 100;
        END_IF;
    40: // Toggle off the startbutton
        "Data_block_1".startbutton := FALSE;
        "Data_block_1".TestStep := 41;
        
    41: // First test if the sensor and the motor can both be active at the same time and raise a fault
        IF "Data_block_1".State = 1 AND "Data_block_1".objectsensor AND "Data_block_1".motor THEN
            "Data_block_1".Faultid := 41;
            "Data_block_1".TestStep := 999;
        ELSE
            "Data_block_1".Arm_retracted := TRUE;
            "Data_block_1".TestStep := 42;
        END_IF;
        
    42: // Have a fully registered sensor input occur
        "Data_block_1".objectsensor := TRUE;
        "Data_block_1".Arm_retracted := TRUE;
        IF #True_sensor_on_time.Q THEN
            "Data_block_1".objectsensor := FALSE;
            "Data_block_1".TestStep := 43;
        END_IF;
        
    43: // Ensure enough time for sensor to debounce
        IF #True_sensor_off_time.Q THEN
            "Data_block_1".TestStep := 44;
        END_IF;
   
    44: // Check state is 2 then close the test
        IF "Data_block_1".State = 2 THEN
            "Data_block_1".Arm_retracted := FALSE;
            "Data_block_1".TestStep := 45;
        ELSE
            "Data_block_1".Faultid := 43;
            "Data_block_1".TestStep := 999;
        END_IF;
    45: // Give enough tme to simulate pusher
        IF #Extenson_retracton.Q THEN
            "Data_block_1".Arm_extended := TRUE;
            "Data_block_1".TestStep := 46;
        END_IF;
        
     46: //Now that the extended sensor is true, lets give time for the function block to perform
         IF #On_delay_limit_sensor.Q THEN
             IF "Pneumatic_pusher_DB_1".Q_is_extended THEN
                 "Data_block_1".TestStep := 47;
             ELSE
                 "Data_block_1".Faultid := 45;
                 "Data_block_1".TestStep := 999;
             END_IF;
         END_IF;
         
     47: // Check to see if state 3 has been reached
         IF "Data_block_1".State = 3 THEN
             "Data_block_1".Arm_extended := FALSE;
             "Data_block_1".TestStep := 48;
         ELSE
             "Data_block_1".Faultid := 46;
             "Data_block_1".TestStep := 999;
         END_IF;
     48: // Simulate the retraction movement occuring
         IF #Extenson_retracton.Q THEN
             "Data_block_1".Arm_retracted := TRUE;
             "Data_block_1".TestStep := 49;
         END_IF;
     49: //Let function block recognise the retraction   
         IF #On_delay_limit_sensor.Q THEN
             IF "Pneumatic_pusher_DB_1".Q_is_retracted THEN
                 "Data_block_1".TestStep := 50;
             ELSE
                 "Data_block_1".Faultid := 48;
                 "Data_block_1".TestStep := 999;
             END_IF;
         END_IF;
     50: // Ensure we have went back to the orginal state 
         IF "Data_block_1".State = 0 THEN
             "Data_block_1".TestStep := 100;
         ELSE
             "Data_block_1".Faultid := 49;
             "Data_block_1".TestStep := 999;
         END_IF;
            
    
            
        // ============================================================
        // TEST COMPLETION & FAULT STATES
        // ============================================================
    100: // --- ALL TESTS PASSED ---
        "Data_block_1".TestSuccess := TRUE;
        "Data_block_1".Start_test := FALSE;
        "Data_block_1".TestStep := 0;
        
    999: // --- SAFE FAULT STATE ---
        "Data_block_1".Testfault := TRUE;
        "Data_block_1".Start_test := FALSE;
        "Data_block_1".Global_estop := FALSE;
        "Data_block_1".guarddoor := FALSE;
        "Data_block_1".Safetyfeedbacksenssor := FALSE;
        
END_CASE;

    999: // --- SAFE FAULT STATE ---
        "Data_block_1".Testfault := TRUE;
        "Data_block_1".Start_test := FALSE;
        "Data_block_1".Global_estop := FALSE;
        "Data_block_1".guarddoor := FALSE;
        "Data_block_1".Safetyfeedbacksenssor := FALSE;
        
END_CASE;
```
