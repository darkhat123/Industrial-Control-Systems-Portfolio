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

#True_sensor_time(IN := ("Data_block_1".TestStep = 42),
                  PT := T#100ms);

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
        IF #True_sensor_time.Q THEN
            "Data_block_1".objectsensor := FALSE;
            IF #True_sensor_time.Q THEN
                "Data_block_1".Arm_retracted := FALSE;
                "Data_block_1".TestStep := 43;
            END_IF;
        END_IF;
        
    43: // Check state is 2 then close the test
        IF "Data_block_1".State = 2 THEN
            "Data_block_1".TestStep := 100;
        ELSE
            "Data_block_1".Faultid := 42;
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
```
