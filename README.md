# mks-mfc
The purpose of these LabVIEW programs is communicate and control mass flow controllers (MFCs) for remote and automated operation of a TGA.
Individual MFCs can be turned on or off and have their set point changed.
After basic controls are developed a new VI, a scheduling program is provided which reads a text file automatically changing flow rates are predetermined times.
The VIs are saved for LabVIEW version 2013 and later.
This work is the continuation and development of programs previously written by others.

## Hardware and other libraries
These programs are written for the use of a MKS 647C Multi Channel Flow Ratio/Pressure Controller with Type M100B MFCs.
It can accommodate up to 8 channels for MFCs and communicates via serial.
Drivers for LabVIEW communications with the MKS 647C are available online from the [NI website](http://sine.ni.com/apps/utf8/niid_web_display.download_page?p_id_guid=9AC4F43A67732360E04400144F1EF859).
Serial communications occur over the VISA standard and require the installation of the [NI VISA drivers](https://www.ni.com/nisearch/app/main/p/bot/no/ap/tech/lang/en/pg/1/sn/catnav:du,n8:3.25.123.1640,ssnav:ndr/).

An Arduino Uno R3 is utilized with a servo motor and 3D printed parts to manually turn a shut off valve, compensating for a leaking MFC.
Details on the installation of the Arduino controlled shut-off including STL files for 3D printing are available at [HackaDay.io](https://hackaday.io/project/52519-automated-gas-flow-shut-off).
To control the Arduino from LabVIEW the [LINX library](https://www.labviewmakerhub.com/doku.php?id=libraries:linx:start) is used and must be downloaded from the VI package manager.
Additonally, the Arduino will need to be flashed with the LINX control sketch before use.
This component can be easily removed if not required for the application.

## Program Overview
### Gas Controller
The block diagram of the Gas Controller VI which manually controls the MFCs is provided with the front panel image and is the fundamental starting block for the gas scheduling program.
Starting on the far left of the program, initial communications variables such as baud rate and port information are passed to the sub-VIs which open communications.
The program then enters a while loop until the stop button is pressed or an error occurs, at which point serial communications are closed by the appropriate sub-VI.

Within the while loop, the program sequentially goes to each channel, checking the desired status, setting the control variables and reading the flow.
If the program is set to not use an MFC, the internal valve is shut and the flow is read to ensure no leakage.
If an MFC is set to be used, the percent of full flow is calculated based on the size of the MFC, the gas correction factor, and the desired flow rate.
The percent of full flow is then passed to the MFC and the flow is read.
For the case of channel 3, which posses a leaky MFC, the addition of an external shutoff valve controlled by Arduino was implemented.
If channel 3 is to be used, the Arduino sends a PWM signal to the servo motor which corresponds to the valve being in the open position.
In the case where channel 3 is to be closed, a PWM signal is sent which closes the valve.

Finally, after each channel has been set, the program checks for the dangerous combination of hydrogen and oxygen gasses.
If the channels for hydrogen and oxygen (in this case channels 3 and 4) are both open, the program immediately shuts the valves and gives an error warning the user and ends the program.
As long as this is not the case, the program proceeds to loop back to setting the values for channel 1.

#### Block Diagram
![Gas Controller Block Diagram](https://github.com/patrickstanley/mks-mfc/raw/master/Documentation%20Images/Gas%20Controller/Gas_Controllerd.png)

#### Front panel
![Gas Controller Front Panel](https://github.com/patrickstanley/mks-mfc/raw/master/Documentation%20Images/Gas%20Controller/Gas_Controllerp.png)


### Gas Scheduler
The Gas Scheduler program uses the Gas Controller to automate automate the switching of gasses based on the instructions provided in a text file.
The text file, referred to as the gas schedule, is nine column tab delimited file where the first column is the amount in seconds for which that gas flow is to be run, followed by the flows for each channel.
The program sequentially goes though the rows until it encounters a negative time, at which point it ends leaving that row's gas flows running.
The program also additionally records the measured flows to a csv file for manual inspection later.

Images below gives the block diagram and front panel of the program.
Before the start of the program the path to the gas schedule must be specified.
At the start of the program, the location to record measured gas flows is requested and setup as a csv file type with a header, variables to count the row of the program and time elapsed are initialized and communications variables are passed to sub-VIs.
After this, the program enters a while loop.
This while loop is ended if an error occurs, if the stop button is pressed, or if the amount of time in the stage (given by the first column in the gas schedule) is less than zero.

Once the start button is pushed, a timer is started to measure the amount of time elapsed in the stage.
If the time has elapsed the amount prescribed (starting with zero seconds), the program reads the gas schedule, increments the stage number, and passes the flow rates to a gas controller sub-VI, after which the VI loops.
If time has not elapsed, the program records the flows based on a second elapsed time timer to limit the total number of recordings.
After a program end condition is met, the communications channels are closed.

#### Block Diagram
![Gas Scheduler Block Diagram](https://github.com/patrickstanley/mks-mfc/raw/master/Documentation%20Images/Gas%20Scheduler/Gas_Schedulerd.png)

#### Front panel
![Gas Scheduler Front Panel](https://github.com/patrickstanley/mks-mfc/raw/master/Documentation%20Images/Gas%20Scheduler/Gas_Schedulerp.png)
