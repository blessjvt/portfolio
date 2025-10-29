# SPO2 Pulse Oximeter
Using several passive and active filter combinations, I created a device that could read out oxygen levels and blood pressure levels of an individual 

Introduction 

The product tasked to design was an SP02 oximeter and a heartbeat monitor. A pulse oximeter measures how much red and infrared light is being absorbed to determine the oxygen levels in a human’s body system. The ratio of the red-light measurement to the infrared light measurement would then tell how much oxygenated blood there is in the body, compared to deoxygenated. The SP02 design implements a Nellcor Ds- 100A connected with an SP02 finger sensor, a transimpedance amplifier, different filter designs, amplifiers, a multiplexer, an Arduino, and an OLED Display. 

High Level Design 
<img width="702" height="280" alt="DiagramofaDiagram" src="https://github.com/user-attachments/assets/2b088b63-d3ec-418a-a25e-8b202a24a161" />

Subsystems 

3.1 Sensor 

The sensor consists of two anti-parallel LEDS, a red (660nm) and an infrared (940nm), that transmit light through the finger to a photodiode on the other side.  Pins 2 and 3 were connected to the CD405xB CMOS Single 8- Channel Analog Multiplexer (goes more in depth on the role, in 3.3) to switch between the two signals. 


 <img width="379" height="370" alt="Sensor Pinout" src="https://github.com/user-attachments/assets/e7336416-536a-41f0-a198-c5436d8a8e32" />

						Figure 1: Nellcor Sp02 Sensor Pinout 

 

3.2 Transimpedance Amplifier 

A transimpedance amplifier was implemented in the circuit to convert a current source value that is output from the sensor to a voltage value, that will be sent down the path and be used to determine the SP02 value of the user. The capacitor value of the transimpedance amplifier was founded by using the equation $$ C=\dfrac{1}{2\pi\cdot R\cdot f} $$ Without amplification the reading of the current from the transimpedance amplifier is in the micro levels. As a result, a high resistance value for the feedback resistor (600k ohms) was chosen, and then plugged it in the equation. Since the lowest frequency heartbeat that needed attention was 40bpm, a frequency of 2/3 Hz was plugged in the equation. 

 

Figure 2: Nellcor Sp02 Sensor Pinout 

 

3.3 Multiplexer 

As mentioned in 3.1, the multiplexer used in the circuit was the CD405xB CMOS Single 8- Channel Analog Multiplexer. The purpose of this component in the circuit was to control the on and off switching of the two LEDS to display two signals at once... 

 

Figure 3A: CD0453B Multiplexer Pinout 

 

Figure 3B: CD0453B Multiplexer Pin Functions 

The physical connections to and from the mux can be found in Figure 3C. The selecting bit controls which LED is of priority. Pins 15 essentially says if bx (the transimpedance output signal) is high, then by is low, if bx is low, then by is high. The same concept applied to Pin 14. 

Group 1809100299, Grouped object 

Figure 3C: CD0453B Multiplexer Pin Connections 

 

3.4 Filters 

A sequence of filters was implemented in the circuit to get rid of unwanted, disruptive noise at certain frequencies. 

3.4.1 Lowpass 

The AC signal that derives from the photodiode tends to be corrupted by 60 Hz noise. As a result, a second-order active Butterworth low pass filter with a cutoff frequency of 45 Hz was implemented.  A low pass filter allows frequencies smaller than the desired cutoff frequency to pass, while high frequencies get blocked off. 

Figure 4: Transfer function of 2nd order Lowpass filter 

 

3.4.2 Highpass 

Due to noise that happens very early, small frequencies that tend to interfere with the consistent harmonic oscillations of the signal, a high pass filter with a cutoff frequency of 1.1 Hz was implemented.  A highpass filter allows high frequencies to pass while cutting off low frequencies. The cutoff frequency was determined by running the Spectrum Analyzer on Waveform and finding high or low spikes that were not within 5 margins of error compared to the harmonic oscillations. 

 

Figure 5: Transfer function of 2nd order Lowpass filter 

 

Figure 5A: Cutoff Frequency Equation of a High pass filter 
 

𝐻(𝑗𝑤)=𝑗𝑤𝑗𝑤+1𝑅𝐶
H
j
w
=
j
w
j
w
+
1
R
C
 
 

Figure 5B: Transfer function of a 2nd order High pass filter 

 

Figure 5C: Magnitude of the Transfer function of a 2nd order High pass filter 

3.4.3 Notch Filter 

For some reason the low pass filter was not strong enough to fully compress the 60 Hz noise by –30 dB, therefore a notch filter with a 60 Hz cutoff frequency was added as the final filter. 

 

Figure 6: Twin-T Notch Filter Design 

 

3.5 Amplification 

If the signal is too small, then the Arduino would not be able to read the signal. Therefore, by adding a non-inverting amplifier into our circuit, the signal could be amplified large enough for the Arduino to read. The gain was found by using the general non-inverting amplifier gain equation: 
𝐴𝑣=1+𝑅1𝑅2
A
v
=
1
+
R
1
R
2
 
. To get a large amplification, the R2 was designed to be the smallest, yet acceptable resistance value for the circuit. The value was 1k ohm. 

 

Figure 7: Non-inverting Amplifier Schematic and Gain Equation 

𝐴𝑣=1+𝑅1𝑅2
A
v
=
1
+
R
1
R
2
 
			 (1) 

3.6 Arduino 

The Arduino’s purposes are an analog to digital converter, processing, and communicating. It reads an analog value from the signals and then converts it to digital to the multiplexer. Since the Nellcor Ds-100A only outputs in current the goal was to first make a trans-impedance amplifier that would convert the current source into a voltage source that operates from a voltage of 0V to 5V. Going forward, the goal of the microcontroller shifts to communication that will allow communication with the multiplexer, so that the shifting between Red and Infrared LED can take place. Lastly, the data needed to be processed and used to calculate the value of both the heart rate and Sp02 value.  

3.7 OLED 

The goal for the OLED screen was to display both the Sp02 value, heart rate, and lastly the heartbeat pulse. The model we are using has only 4 pins and communicates with the Arduino Uno using I2C communication and with the help of both adafruit_SSD1306.h and the adafruit_GFX.h libraries. By correctly wiring the OLED screen to the microcontroller, we can observe the output of the circuit and display a pulse. Given the pulse, we can capture the DC and AC amplitude of both the Infrared and RED LED. Using that data, we can calcuclate the value of the Sp02 and heartrate . 
arduino with OLED display schematic diagram 

Figure 8: Arduino and OLED Display Implementation 

 

Simulated Results 

 

4.1 LT Spice Simulation: Transimpedance Amplifier  

From what was described in 3.1, a simulated Circuited was designed in LT SPICE. Since the photosensor does not exist in LT SPICE, a current source of one microamps was used in its placed. The simulated results produced a 2VPP. 

 

Figure 9A: LT SPICE Transimpedance Schematic  

 

 

Figure 9B: LT SPICE Transimpedance Plot of Input vs Output  

 

Figure 9C: Transimpedance Measurement 

4.2 LT Spice Simulation: Lowpass Filter 

The active lowpass filter was simulated to see which combinations of physical components could produce the closest desired cutoff frequency of 45 Hz.  

 

Figure 10A: LT SPICE circuit schematic of a 45 Cutoff Frequency Lowpass Filter 

 

Figure 10B: LT SPICE Lowpass filter BODE Plot 

 

 

Figure 10C: LT SPICE 45 Hz Cutoff Frequency of the Lowpass filter 

4.3 LT Spice Simulation: Highpass Filter 

The active highpass filter was simulated to see which combination of physical components could produce the closest desired cutoff frequency of 0.2 Hz. 

 

Figure 11A: LTSPICE Bode Plot of the High pass filter 

 

Figure 11B: LTSPICE 0.2 Hz Cutoff Frequency of the High Pass filter 

4.4 LT Spice Simulation: Notch Filter 

The passive notch filter was simulated to see which combinations of physical components could produce the closest desired cutoff frequency of 60 Hz.  

 
 

Figure 12A: LT SPICE Simulation of the Notch Filter 

 

 

 

Figure 12B: LT SPICE Waveform graph of 60 Hz Cutoff Notch filter 

 

 
 

 

Figure 12C LT SPICE 60 Hz Cutoff Frequency of the Notch filter 

 

Physical Results 

 

5.1 Transimpedance Physical Results: 

The Waveform Scope in figure 13 shows a successful current-to-voltage converter 

 

Figure 13: Physical Results of the Transimpedance Amplifier 

Spectrum Analyzer 

The spectrum analyzer of the transimpedance revealed the frequencies of interest. The huge spike at 60 Hz needed to be brought down. 

 

Figure 14A:  60 Hz Spike 

 

Figure 14B: Noise at the beginning of the signal 

 
 

5.2 Low Pass Filter Physical Results 

The physical lowpass filter had a cutoff frequency of 44 Hz. The goal was 45 Hz, however being only one Hz off, the filter was acceptable. 

 

Figure 15A: The Bode Plot of the Physical High Pass Filter 

 

A screen shot of a graph

Description automatically generated 

Figure 15B: The Spectrum Analysis of the lowpass filter vs the transimpedance amplifier 

 

5.3 Highpass Filter Physical Results  

The physical high pass filter had a cutoff frequency of 5 Hz. The physical high pass filter cut off frequency did not match the simulated high pass filter cut off frequency of 1 Hz.	 

	 

Figure 16A: The Bode Plot of the Physical High Pass Filter 

 

 

Figure 16B: The Spectrum analysis of the High pass filter output vs Low pass filter output 

 

 

Figure 16C: The Spectrum analysis of the High pass filter output vs the transimpedance amplifier output 

 

5.3 Notch Pass Filter Physical Results 

The physical notch pass filter was successful in eliminating the 60 Hz cutoff point. Still interesting that a notch pass filter would be needed even though a low pass filter with a cutoff frequency at 45 Hz was implemented.  

 

Inserting image... 

Figure 17A: Spectrum Analysis of the notch filter output(blue) vs the transimpedance output(orange) 

Inserting image... 

Figure 17B: Spectrum Analysis of the notch filter output(blue) vs the transimpedance output(orange) Zoomed Out 

5.5 Physical Circuit 

The simulated circuit was constructed physically. Each subsystem of the circuit was measured and compared to the simulated results as a way of verifying correctness. 

 

 

Figure 18A: The transimpedance and Mux integrated in the physical circuit 

 

 

Figure 18B: The physical circuit 

Digital Subsystem 

Focusing on the digital subsystem, the digital subsystem is responsible for coordinating the entire operation of the SpO2 system. It controls the hardware components (LEDs, photodiode, and display), processes the signals, performs calculations, and updates the display. 
Signal Reading: The microcontroller reads the signals from the photodiode through an analog input. The signals are generated by the light from the LEDs (Red and Infrared) that is reflected off the user's skin and received by the photodiode. 
LED Controls: The microcontroller processes the signals received from the photodiode. It then calculates the peak and minimum values for each LED, which are then used to calculate the heart rate and the SpO2 level 
Signal Processing: The Arduino processes the signals received from the photodiode. It calculates the peak and minimum values for each LED, which are then used to calculate the heart rate and the SpO2 level. This is done by applying the formulas for heart rate and SpO2 calculation to the processed signals. 
Timing: The Arduino manages the timing of the readings, calculations, and display updates. It uses built-in functions like millis() and micros() to create non-blocking delays, allowing for regular updates without halting the entire program. 
Display: The Arduino controls the OLED display to show the calculated heart rate and SpO2 level. It updates the display at regular intervals and also manages the scrolling of the waveform display. 

6.1 Finite State Machine 

 

Figure 19: An overall FSM of the digital subsystem 

 

Initialize: Initializes all variables and libraries needed for the overall project. 

Initialize Timer Interrupt(): Essential as the requirement is to measure both Red and Infrared LEDs simultaneously. As the microcontroller is too slow to switch between both LEDs and is needed to immediately alternate between both LEDs. 

SSD1306 Allocation Failed: A failsafe to make sure that the display does recognize the microcontroller and prints the error if it does not. 

Red_LED_On/Infrared_LED_On: Turns on the respecitve LED and will switch the other LED, causing one to be HIGH and the other to be LOW. 

Read DC and AC Red and Infrared: Needed to measure the DC and AC signals from both LED’s used to calculate the value of Sp02 and R-value. 

Calculate: Used to calculate both the value of Sp02 and R before the value is shown on the OLED screen. 

6.2 Calculations 

It is important to note that displaying both the SpO2 value and the heart rate is an objective requirement. This can be achieved by identifying the frequency of the heart rate, subsequently determining the pulse period, and utilizing it accordingly. 

 
𝐹𝑟𝑒𝑞 = 12𝑇
F
r
e
q
 
=
 
1
2
T
 
 

By determining the frequency, the heart rate can be determined. 

𝐻𝑒𝑎𝑟𝑡 𝑟𝑎𝑡𝑒 = 𝐹𝑟𝑒𝑞 ⋅ 60 
H
e
a
r
t
 
r
a
t
e
 
=
 
F
r
e
q
 
⋅
 
60
 
 
 

 
Next, the R-value can be solved needed for the SpO2 by using 

𝑅 = 𝐴𝐶𝑟𝑒𝑑𝐷𝐶𝑟𝑒𝑑𝐴𝐶𝑖𝑟𝑒𝑑𝐷𝐶𝑖𝑟𝑒𝑑
R
 
=
 
A
C
r
e
d
D
C
r
e
d
A
C
i
r
e
d
D
C
i
r
e
d
 
  

And then use the SpO2 formula. This formula was found in Texas Instruments', “How to Design Peripheral Oxygen Saturation(Sp02) and Optical Heart Rate Monitoring (OHRM) Systems Using the AFE4403.” 

𝑆𝑝𝑂2 = 110 − (25 ⋅ 𝑅)
S
p
O
2
 
=
 
110
 
−
 
25
 
⋅
 
R
 
 

To find the Sp02 value for the objective requirement. 

6.3 Validation 

Regrettably, difficulties were encountered in obtaining the accurate Heart rate and SpO2 values required for the project due to multiplexer errors. Due to issues while creating the second signal path, there was confusion on how to correctly implement the second signal path and caused the analog circuit to malfunction when implementing the multiplexer. 
The cause of these errors was also likely attributed to issues within the code implementation, specifically related to problems encountered with the Interrupt function. Without the interrupt function being correctly implemented, there was a significant delay in switching between LEDs, resulting in inaccuracies in the measurement for both Heart Rate and SpO2 value. 

 

Figure 20: OLED Display of the results 

 

Figure 21: The two final output signals 

 

7. Future Work 

To improve in the future, it is crucial to address the following areas: 

Enhance understanding of the second signal path implementation: Clear confusion and ensure a proper understanding of how to correctly implement the second signal path in order to avoid malfunctions in the analog circuit when using the multiplexer. 

Pay special attention to the Interrupt function and resolve any issues that may arise. Ensure that the interrupt function is correctly implemented to minimize delays in switching between LEDs, thereby preventing inaccuracies when measuring both Heart rate and SpO2 values. 

Conduct comprehensive testing and validation procedures to identify and rectify any errors or inconsistencies early on. This will help ensure the accuracy and reliability of the Heart rate and SpO2 measurements. 

Maintain detailed documentation of the project, including circuit diagram, code explanation, and debugging steps.  

Foster a collaborative environment among team members to share knowledge and insights, which can help in identifying and resolving any potential issues more effectively. 

Addressing these issues in future projects can mitigate difficulties encountered and improve the efficiency and quality of this difficult project. 


[return to main page](index.md)
