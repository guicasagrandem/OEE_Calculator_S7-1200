OEE Calculator for Siemens S7-1200 in SCL
📄 About the Project
This project implements an OEE (Overall Equipment Effectiveness) Calculator for Siemens SIMATIC S7-1200 PLCs, developed in SCL (Structured Control Language). The goal is to provide a robust and precise solution for monitoring and calculating in real-time the three pillars of OEE: Availability, Performance, and Quality, per work shift.

OEE measurement is crucial for identifying losses, optimizing manufacturing processes, and improving productivity in industrial environments, helping to make data-driven decisions about machine efficiency.

✨ Features
Precise OEE Calculation: Measures and presents OEE in real-time and per shift.
OEE Components: Individually calculates:
Availability: Machine available time vs. Downtime (including faults and non-automatic mode).
Performance: Actual production speed vs. Ideal speed (cycle time).
Quality: Good parts vs. Total parts produced.
Shift Monitoring: Accumulates and displays separate metrics for up to 3 configurable work shifts.
Smart Reset: Metrics are automatically reset at the start of each shift or can be manually reset.
Robustness: Uses PLC time for accurate measurements and manages data continuity in case of PLC restart through retentive variables.
Configurable: Shift start times and standard machine cycle time are easily configurable.
🚀 How to Use
To use this project, you will need to download it from GitHub and restore it in TIA Portal.

1. Prerequisites
TIA Portal: Version 15 or higher (the project was exported in .zap19 format).
Siemens SIMATIC S7-1200 PLC: For simulation or real hardware.
2. Download the Project from GitHub
Go to the main page of this repository on GitHub.
Click on the green <> Code button.
Select the Download ZIP option.
Unzip the downloaded .zip file to a folder of your choice on your computer.
3. Restore the Project in TIA Portal
Open TIA Portal.
In the top menu, go to Project > Retrieve.
Navigate to the folder where you unzipped the project.
Select the project backup file, which has the .zap19 extension (the name should be OEE_REAL_TIME.zap19).
Click Open and follow the instructions to restore the project.
4. Instantiation and Configuration (Verify in Restored Project)
The project already comes with the FB_OEE_Calculator function block instantiated in OB1 and with basic connections. You will need to verify and adjust the inputs and outputs according to your application.

Common Inputs to Verify:

Parameter	Data Type	Description	Requirements / Important Notes
Auto_ON	BOOL	TRUE signal when the machine is operating in automatic mode (producing or ready to produce).	Essential for Availability calculation. If FALSE (and Fault_ON is also FALSE), the time is considered unplanned downtime.
Fault_ON	BOOL	TRUE signal when the machine is in a fault state (unplanned stop).	Essential for Availability calculation. The time this signal is TRUE is accumulated as downtime. Priority over Auto_ON in some contexts of forced interruption.
StandardCycleTime	REAL	The ideal cycle time of the machine to produce one part, in seconds.	Crucial for Performance. Must be a value greater than 0. Reflects the machine's maximum production capacity. Example: 10.0 for 10 seconds per part.
CycleCompleted	BOOL	A pulse (rising edge from FALSE to TRUE) indicating the completion of a production cycle.	Crucial for Performance and Quality. Must be a pulsed and momentary signal. If it's a constant TRUE signal, cycle counts and cycle time accumulation will not update after the first pulse.
ScrapPart	BOOL	A pulse (rising edge from FALSE to TRUE) indicating that the newly produced part is scrapped.	Crucial for Quality. Must be a pulsed and momentary signal. If not used, Quality will always be 100% (assuming all produced parts are good).
ResetOEE	BOOL	TRUE signal to manually reset all OEE metrics (all shifts) at any time.	Used for manual reset. Metrics are also automatically reset at the start of each configured shift.
Shift1_StartTime	TOD	Start time of Shift 1.	Format TOD#HH:MM:SS. Example: TOD#06:00:00 for 6 AM. Used to detect the active shift and trigger automatic shift reset.
Shift2_StartTime	TOD	Start time of Shift 2.	Format TOD#HH:MM:SS. Example: TOD#14:00:00 for 2 PM.
Shift3_StartTime	TOD	Start time of Shift 3.	Format TOD#HH:MM:SS. Example: TOD#22:00:00 for 10 PM. This shift can cross midnight, and the FB's internal logic handles this.

Exportar para as Planilhas
Common Outputs to Verify:

Parameter	Data Type	Description	Important Notes
OEE_Perc	REAL	OEE value in percentage (0.0 to 100.0) for the currently active shift.	Ideal for real-time display on the HMI. Calculated as (Availability / 100) * (Performance / 100) * (Quality / 100) * 100.
Availability_Perc	REAL	Availability value in percentage (0.0 to 100.0) for the currently active shift.	Reflects actual operating time relative to planned time (total active shift time so far).
Performance_Perc	REAL	Performance value in percentage (0.0 to 100.0) for the currently active shift.	Reflects the machine's production speed relative to StandardCycleTime. Reaches 100% if the average cycle time is equal to or less than StandardCycleTime. May take a few cycles to update after shift reset.
Quality_Perc	REAL	Quality value in percentage (0.0 to 100.0) for the currently active shift.	Reflects the proportion of good parts relative to the total produced.
CurrentCycleTime	REAL	Average cycle time in seconds for the currently active shift.	This is the average of the cycle times for all valid parts produced in the current shift. May take a few cycles to update after shift reset, as it needs at least 2 cycles to calculate the first valid time delta.
OEE_Shift1	REAL	OEE value in percentage for Shift 1.	These shift-specific outputs (_Shift1, _Shift2, _Shift3) retain the accumulated value for each shift, even when it is not the active one. Ideal for historical reports or summary display.
Availability_Shift1	REAL	Availability value in percentage for Shift 1.	
Performance_Shift1	REAL	Performance value in percentage for Shift 1.	
Quality_Shift1	REAL	Quality value in percentage for Shift 1.	
OEE_Shift2, etc.	REAL	OEE and components for Shift 2.	
OEE_Shift3, etc.	REAL	OEE and components for Shift 3.	
CurrentActiveShift	INT	Indicates the currently active shift: 0 (None), 1 (Shift 1), 2 (Shift 2), 3 (Shift 3).	Useful for the HMI to identify which set of data is being displayed in OEE_Perc and other general outputs.

Exportar para as Planilhas
📸 Project Images
Here are some screenshots of the function block instantiated in TIA Portal:

FB_OEE_1.png
FB_OEE_2.png
🛠️ Implementation Details
Time Base: Uses an internal TON timer (Continuous_System_Timer) to measure elapsed PLC time in milliseconds, ensuring precision in cycle time and downtime calculations.
Downtime Logic: The machine is considered in downtime if it is in Fault_ON OR if Auto_ON is FALSE (considering "non-automatic" as non-productive downtime).
Performance Calculation: The average cycle time is calculated starting from the second production cycle onwards, to ensure the metric's accuracy. The first cycle after a reset or shift start does not contribute to the initial average.
RD_SYS_T: The RD_SYS_T block is used to get the current PLC system time, which is essential for identifying the active shift and triggering automatic resets.
🤝 Contribution
Contributions are welcome! If you have suggestions, improvements, or find bugs, feel free to open an issue or submit a pull request.

📄 License
This project is distributed under the MIT license. See the LICENSE file for more details.

📞 Contact
For questions or support, please contact via GitHub.

