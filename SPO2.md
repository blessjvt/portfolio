# SPO2 Pulse Oximeter

Using several passive and active filter combinations, I created a device that could read out oxygen levels and blood pressure levels of an individual 

### 1. Introduction

The product tasked to design was an SP02 oximeter and a heartbeat monitor. A pulse oximeter measures how much red and infrared light is being absorbed to determine the oxygen levels in a human’s body system. The ratio of the red-light measurement to the infrared light measurement would then tell how much oxygenated blood there is in the body, compared to deoxygenated. The SP02 design implements a Nellcor Ds- 100A connected with an SP02 finger sensor, a transimpedance amplifier, different filter designs, amplifiers, a multiplexer, an Arduino, and an OLED Display. 

### 2. High Level Design

<img width="702" height="280" alt="DiagramofaDiagram" src="https://github.com/user-attachments/assets/2b088b63-d3ec-418a-a25e-8b202a24a161" />


## 3. Subsystems

**3.1 Sensor**

The sensor consists of two anti-parallel LEDS, a red (660nm) and an infrared (940nm), that transmit light through the finger to a photodiode on the other side.  Pins 2 and 3 were connected to the CD405xB CMOS Single 8- Channel Analog Multiplexer (goes more in depth on the role, in 3.3) to switch between the two signals. 

 <img width="379" height="370" alt="Sensor Pinout" src="https://github.com/user-attachments/assets/e7336416-536a-41f0-a198-c5436d8a8e32" />

Figure 1: Nellcor Sp02 Sensor Pinout 

**3.2 Transimpedance Amplifier**

A transimpedance amplifier was implemented in the circuit to convert a current source value that is output from the sensor to a voltage value, that will be sent down the path and be used to determine the SP02 value of the user. The capacitor value of the transimpedance amplifier was founded by using the equation $C=\dfrac{1}{2\pi R f}$. Without amplification the reading of the current from the transimpedance amplifier is in the micro levels. As a result, a high resistance value for the feedback resistor (600k ohms) was chosen, and then plugged it in the equation. Since the lowest frequency heartbeat that needed attention was 40bpm, a frequency of 2/3 Hz was plugged in the equation. 

 <img width="448" height="334" alt="transimpedance amplifier" src="https://github.com/user-attachments/assets/e95c0c45-5ca3-4dba-b2b1-1cfe4a8f452c" />

Figure 2: Transimpedance Amplifier 

**3.3 Multiplexer**

As mentioned in 3.1, the multiplexer used in the circuit was the CD405xB CMOS Single 8- Channel Analog Multiplexer. The purpose of this component in the circuit was to control the on and off switching of the two LEDS to display two signals at once... 

 <img width="539" height="349" alt="MultiplexerPinout" src="https://github.com/user-attachments/assets/0f670f1f-9cb2-4f9c-8c86-a9207833d433" />

Figure 3A: CD0453B Multiplexer Pinout 

 <img width="580" height="333" alt="Figure3b" src="https://github.com/user-attachments/assets/9049197f-4a9d-4ad9-8593-cc8f8e62f5b4" />

Figure 3B: CD0453B Multiplexer Pin Functions 

The physical connections to and from the mux can be found in Figure 3C. The selecting bit controls which LED is of priority. Pins 15 essentially says if bx (the transimpedance output signal) is high, then by is low, if bx is low, then by is high. The same concept applied to Pin 14. 

<img width="744" height="302" alt="Figure3C" src="https://github.com/user-attachments/assets/683e518b-d0f7-4757-857e-09024805ab3a" />

Figure 3C: CD0453B Multiplexer Pin Connections 

**3.4 Filters**

A sequence of filters was implemented in the circuit to get rid of unwanted, disruptive noise at certain frequencies. 

**3.4.1 Lowpass**

The AC signal that derives from the photodiode tends to be corrupted by 60 Hz noise. As a result, a second-order active Butterworth low pass filter with a cutoff frequency of 45 Hz was implemented.  A low pass filter allows frequencies smaller than the desired cutoff frequency to pass, while high frequencies get blocked off. 

<img width="711" height="151" alt="Figure4" src="https://github.com/user-attachments/assets/3afca504-214e-4ee2-9132-9d21ad776fb9" />

Figure 4: Transfer function of 2nd order Lowpass filter 

 

**3.4.2 Highpass**

Due to noise that happens very early, small frequencies that tend to interfere with the consistent harmonic oscillations of the signal, a high pass filter with a cutoff frequency of 1.1 Hz was implemented.  A highpass filter allows high frequencies to pass while cutting off low frequencies. The cutoff frequency was determined by running the Spectrum Analyzer on Waveform and finding high or low spikes that were not within 5 margins of error compared to the harmonic oscillations. 

  <img width="564" height="378" alt="Figure5" src="https://github.com/user-attachments/assets/21dea657-44a2-44cf-bdac-01f10c4c4908" />

Figure 5: Transfer function of 2nd order Lowpass filter 

   <img width="170" height="111" alt="Figure5A" src="https://github.com/user-attachments/assets/c7c0dd59-ac1f-48e7-b8ed-823387552bf1" />

Figure 5A: Cutoff Frequency Equation of a High pass filter 
 
  <img width="176" height="79" alt="Figure5B" src="https://github.com/user-attachments/assets/4a085932-23e6-4197-b938-c41ce959fc7f" />
 
Figure 5B: Transfer function of a 2nd order High pass filter 

   <img width="343" height="113" alt="Figure5C" src="https://github.com/user-attachments/assets/a357446d-33b8-49c0-9447-fe851bd6fc3a" />

Figure 5C: Magnitude of the Transfer function of a 2nd order High pass filter 

**3.4.3 Notch Filter**

For some reason the low pass filter was not strong enough to fully compress the 60 Hz noise by –30 dB, therefore a notch filter with a 60 Hz cutoff frequency was added as the final filter. 

 <img width="699" height="317" alt="BasicTwin-TNotchFilter" src="https://github.com/user-attachments/assets/50c5bd18-f2b0-448b-a144-3fb23cad56d3" />

Figure 6: Twin-T Notch Filter Design 

**3.5 Amplification**

If the signal is too small, then the Arduino would not be able to read the signal. Therefore, by adding a non-inverting amplifier into our circuit, the signal could be amplified large enough for the Arduino to read. The gain was found by using the general non-inverting amplifier gain equation: 
$A_v=1 + \dfrac{1}{R1/R2}$ . To get a large amplification, the R2 was designed to be the smallest, yet acceptable resistance value for the circuit. The value was 1k ohm. 

<img width="345" height="233" alt="Figure7" src="https://github.com/user-attachments/assets/6a4fc636-6e5b-4353-9553-9b05ff8b22ce" />

Figure 7: Non-inverting Amplifier Schematic and Gain Equation 

$A_v=1 + \dfrac{1}{R1/R2}$ (1) 

**3.6 Arduino** 

The Arduino’s purposes are an analog to digital converter, processing, and communicating. It reads an analog value from the signals and then converts it to digital to the multiplexer. Since the Nellcor Ds-100A only outputs in current the goal was to first make a trans-impedance amplifier that would convert the current source into a voltage source that operates from a voltage of 0V to 5V. Going forward, the goal of the microcontroller shifts to communication that will allow communication with the multiplexer, so that the shifting between Red and Infrared LED can take place. Lastly, the data needed to be processed and used to calculate the value of both the heart rate and Sp02 value.  

**3.7 OLED**

The goal for the OLED screen was to display both the Sp02 value, heart rate, and lastly the heartbeat pulse. The model we are using has only 4 pins and communicates with the Arduino Uno using I2C communication and with the help of both adafruit_SSD1306.h and the adafruit_GFX.h libraries. By correctly wiring the OLED screen to the microcontroller, we can observe the output of the circuit and display a pulse. Given the pulse, we can capture the DC and AC amplitude of both the Infrared and RED LED. Using that data, we can calculate the value of the Sp02 and heart rate . 
arduino with OLED display schematic diagram 

<img width="409" height="282" alt="Figure8" src="https://github.com/user-attachments/assets/e76bf907-eb00-4706-8eb1-8781b47ffb31" />

Figure 8: Arduino and OLED Display Implementation 

 

## 4. Simulated Results 

 
**4.1 LT Spice Simulation: Transimpedance Amplifier**  

From what was described in 3.1, a simulated circuit was designed in LT SPICE. Since the photosensor does not exist in LT SPICE, a current source of one microamp was used in its place. The simulated results produced a 2VPP. 

 <img width="557" height="363" alt="Figure9A" src="https://github.com/user-attachments/assets/9637b3ff-daf2-432e-96ba-dcc6fa8eae94" />

Figure 9A: LT SPICE Transimpedance Schematic  
					

 <img width="660" height="362" alt="Figure9B" src="https://github.com/user-attachments/assets/e383e52b-05f2-45fe-ae28-c688c9a76160" />

Figure 9B: LT SPICE Transimpedance Plot of Input vs Output  

 
<img width="572" height="385" alt="Figure9C" src="https://github.com/user-attachments/assets/a1489107-fdc6-487a-85f2-9272d99c3a49" />

Figure 9C: Transimpedance Measurement 

**4.2 LT Spice Simulation: Lowpass Filter**

The active lowpass filter was simulated to see which combinations of physical components could produce the closest desired cutoff frequency of 45 Hz.  

 
<img width="568" height="320" alt="Figure10A" src="https://github.com/user-attachments/assets/a061c7b2-a39f-44d8-9624-4220f6468efa" />

Figure 10A: LT SPICE circuit schematic of a 45 Cutoff Frequency Lowpass Filter 

<img width="555" height="352" alt="Figure10B" src="https://github.com/user-attachments/assets/0e5ec692-8e5e-41f3-87fe-7176b8a41092" />

Figure 10B: LT SPICE Lowpass filter BODE Plot 

 <img width="407" height="400" alt="Figure10C" src="https://github.com/user-attachments/assets/5c5d822e-7cfd-4c55-afeb-0bc4746bbe5e" />

Figure 10C: LT SPICE 45 Hz Cutoff Frequency of the Lowpass filter 

**4.3 LT Spice Simulation: Highpass Filter**

The active highpass filter was simulated to see which combination of physical components could produce the closest desired cutoff frequency of 0.2 Hz. 

 <img width="573" height="340" alt="Figure11A" src="https://github.com/user-attachments/assets/2f2e82f1-2b05-4687-a73f-50a0664f9b27" />

Figure 11A: LTSPICE Bode Plot of the High pass filter 

 <img width="438" height="443" alt="Figure11B" src="https://github.com/user-attachments/assets/d2cc8962-9383-47e1-b6ba-46ac2d076735" />

Figure 11B: LTSPICE 0.2 Hz Cutoff Frequency of the High Pass filter 

**4.4 LT Spice Simulation: Notch Filter** 

The passive notch filter was simulated to see which combinations of physical components could produce the closest desired cutoff frequency of 60 Hz.  

 
 <img width="438" height="343" alt="Figure12A" src="https://github.com/user-attachments/assets/81a4f3e1-2de3-412a-84cb-1f692a95ed21" />

Figure 12A: LT SPICE Simulation of the Notch Filter 
 
<img width="563" height="467" alt="Figure12B" src="https://github.com/user-attachments/assets/4df42fc4-1c7c-4059-a0e1-7b9106ff1389" />

Figure 12B: LT SPICE Waveform graph of 60 Hz Cutoff Notch filter 
 
<img width="404" height="400" alt="Figure12C" src="https://github.com/user-attachments/assets/ddd189e4-a1b5-4852-9217-f9740ad7b171" />

Figure 12C LT SPICE 60 Hz Cutoff Frequency of the Notch filter 

 

## 5. Physical Results

**5.1 Transimpedance Physical Results:** 

The Waveform Scope in figure 13 shows a successful current-to-voltage converter 

 <img width="561" height="324" alt="Figure13" src="https://github.com/user-attachments/assets/889522b6-5544-4d6d-9512-e51531ffc2ea" />


Figure 13: Physical Results of the Transimpedance Amplifier 

**Spectrum Analyzer** 

The spectrum analyzer of the transimpedance revealed the frequencies of interest. The huge spike at 60 Hz needed to be brought down. 

 <img width="559" height="321" alt="Figure14A" src="https://github.com/user-attachments/assets/6015e078-d361-45c4-b39a-f903a3318b06" />


Figure 14A:  60 Hz Spike 

 
<img width="559" height="309" alt="Figure14B" src="https://github.com/user-attachments/assets/9d69f54a-5a2c-4b79-9ad4-9b9c72874628" />

Figure 14B: Noise at the beginning of the signal 

 
 

**5.2 Low Pass Filter Physical Results**

The physical lowpass filter had a cutoff frequency of 44 Hz. The goal was 45 Hz, however being only one Hz off, the filter was acceptable. 

 
<img width="609" height="406" alt="FIgure15A" src="https://github.com/user-attachments/assets/3936f8f7-1f40-4fa7-9926-8068320d12d7" />

Figure 15A: The Bode Plot of the Physical High Pass Filter 

<img width="598" height="418" alt="Figure15B" src="https://github.com/user-attachments/assets/68fd3f5c-fc8a-401c-a004-5e350e455064" />

Figure 15B: The Spectrum Analysis of the lowpass filter vs the transimpedance amplifier 


**5.3 Highpass Filter Physical Results**  

The physical high pass filter had a cutoff frequency of 5 Hz. The physical high pass filter cut off frequency did not match the simulated high pass filter cut off frequency of 1 Hz.	 

	 
<img width="573" height="377" alt="Figure16A" src="https://github.com/user-attachments/assets/b1a6d57d-9fbb-4d8b-995d-e6f0dfacd0de" />

Figure 16A: The Bode Plot of the Physical High Pass Filter 


<img width="571" height="386" alt="Figure16B" src="https://github.com/user-attachments/assets/0288f5c0-38e4-4c6d-b60a-1959aa0f5609" />

Figure 16B: The Spectrum analysis of the High pass filter output vs Low pass filter output 

 
<img width="566" height="380" alt="Figure16C" src="https://github.com/user-attachments/assets/6aae3d01-0cf2-4c14-8651-7de866ad258a" />

Figure 16C: The Spectrum analysis of the High pass filter output vs the transimpedance amplifier output 

 

**5.3 Notch Pass Filter Physical Results** 

The physical notch pass filter was successful in eliminating the 60 Hz cutoff point. Still interesting that a notch pass filter would be needed even though a low pass filter with a cutoff frequency at 45 Hz was implemented.  

<img width="715" height="403" alt="Figure17A" src="https://github.com/user-attachments/assets/123ef9d3-bc9d-46ca-bc2c-3a751e6ab9ca" />

Figure 17A: Spectrum Analysis of the notch filter output(blue) vs the transimpedance output(orange) 

<img width="719" height="404" alt="Figure17B" src="https://github.com/user-attachments/assets/5db927dd-c6cf-426c-8fb0-9a2ffb48f203" />

Figure 17B: Spectrum Analysis of the notch filter output(blue) vs the transimpedance output(orange) Zoomed Out 

**5.5 Physical Circuit** 

The simulated circuit was constructed physically. Each subsystem of the circuit was measured and compared to the simulated results as a way of verifying correctness. 

 
 <img width="306" height="379" alt="Figure18A" src="https://github.com/user-attachments/assets/d574b918-e609-4f18-b524-8d1cab018dfb" />

Figure 18A: The transimpedance and Mux integrated in the physical circuit 

 
<img width="415" height="321" alt="Figure18B" src="https://github.com/user-attachments/assets/974fea65-2e4f-4acc-9ab6-6a6e4aa205ed" />

Figure 18B: The physical circuit 

## 6. Digital Subsystem

Focusing on the digital subsystem, the digital subsystem is responsible for coordinating the entire operation of the SpO2 system. It controls the hardware components (LEDs, photodiode, and display), processes the signals, performs calculations, and updates the display. 
Signal Reading: The microcontroller reads the signals from the photodiode through an analog input. The signals are generated by the light from the LEDs (Red and Infrared) that is reflected off the user's skin and received by the photodiode. 
LED Controls: The microcontroller processes the signals received from the photodiode. It then calculates the peak and minimum values for each LED, which are then used to calculate the heart rate and the SpO2 level 
Signal Processing: The Arduino processes the signals received from the photodiode. It calculates the peak and minimum values for each LED, which are then used to calculate the heart rate and the SpO2 level. This is done by applying the formulas for heart rate and SpO2 calculation to the processed signals. 
Timing: The Arduino manages the timing of the readings, calculations, and display updates. It uses built-in functions like millis() and micros() to create non-blocking delays, allowing for regular updates without halting the entire program. 
Display: The Arduino controls the OLED display to show the calculated heart rate and SpO2 level. It updates the display at regular intervals and also manages the scrolling of the waveform display. 

**6.1 Finite State Machine** 

 
<img width="694" height="423" alt="Figure19" src="https://github.com/user-attachments/assets/633bfb85-73e5-47e7-8f13-3985c8901f54" />

Figure 19: An overall FSM of the digital subsystem 

 

Initialize: Initializes all variables and libraries needed for the overall project. 

Initialize Timer Interrupt(): Essential as the requirement is to measure both Red and Infrared LEDs simultaneously. As the microcontroller is too slow to switch between both LEDs and is needed to immediately alternate between both LEDs. 

SSD1306 Allocation Failed: A failsafe to make sure that the display does recognize the microcontroller and prints the error if it does not. 

Red_LED_On/Infrared_LED_On: Turns on the respecitve LED and will switch the other LED, causing one to be HIGH and the other to be LOW. 

Read DC and AC Red and Infrared: Needed to measure the DC and AC signals from both LED’s used to calculate the value of Sp02 and R-value. 

Calculate: Used to calculate both the value of Sp02 and R before the value is shown on the OLED screen. 

**6.2 Calculations** 

It is important to note that displaying both the SpO2 value and the heart rate is an objective requirement. This can be achieved by identifying the frequency of the heart rate, subsequently determining the pulse period, and utilizing it accordingly. 

 $Freq = \dfrac{1}{2T}$

By determining the frequency, the heart rate can be determined. 

𝐻𝑒𝑎𝑟𝑡 𝑟𝑎𝑡𝑒 = 𝐹𝑟𝑒𝑞 ⋅ 60 

Next, the R-value can be solved needed for the SpO2 by using 

<img width="97" height="75" alt="Screenshot 2025-10-29 at 7 41 10 PM" src="https://github.com/user-attachments/assets/903fe839-0915-47ed-b969-547e2aa7e943" />


And then use the SpO2 formula. This formula was found in Texas Instruments', “How to Design Peripheral Oxygen Saturation(Sp02) and Optical Heart Rate Monitoring (OHRM) Systems Using the AFE4403.” 

𝑆PO2 = 110 − (25 ⋅ 𝑅)

To find the Sp02 value for the objective requirement. 

**6.3 Validation** 

Regrettably, difficulties were encountered in obtaining the accurate Heart rate and SpO2 values required for the project due to multiplexer errors. Due to issues while creating the second signal path, there was confusion on how to correctly implement the second signal path and caused the analog circuit to malfunction when implementing the multiplexer. 
The cause of these errors was also likely attributed to issues within the code implementation, specifically related to problems encountered with the Interrupt function. Without the interrupt function being correctly implemented, there was a significant delay in switching between LEDs, resulting in inaccuracies in the measurement for both Heart Rate and SpO2 value. 

 <img width="325" height="305" alt="Figure20" src="https://github.com/user-attachments/assets/41a3d4d1-80a6-4ae2-a941-ded1a47b458c" />

Figure 20: OLED Display of the results 

 
<img width="447" height="329" alt="Figure21" src="https://github.com/user-attachments/assets/0915b4d6-8fff-42f5-9e4a-6d31944397d6" />

Figure 21: The two final output signals 

 

## 7. Future Work

To improve in the future, it is crucial to address the following areas: 

Enhance understanding of the second signal path implementation: Clear confusion and ensure a proper understanding of how to correctly implement the second signal path in order to avoid malfunctions in the analog circuit when using the multiplexer. 

Pay special attention to the Interrupt function and resolve any issues that may arise. Ensure that the interrupt function is correctly implemented to minimize delays in switching between LEDs, thereby preventing inaccuracies when measuring both Heart rate and SpO2 values. 

Conduct comprehensive testing and validation procedures to identify and rectify any errors or inconsistencies early on. This will help ensure the accuracy and reliability of the Heart rate and SpO2 measurements. 

Maintain detailed documentation of the project, including circuit diagram, code explanation, and debugging steps.  

Foster a collaborative environment among team members to share knowledge and insights, which can help in identifying and resolving any potential issues more effectively. 

Addressing these issues in future projects can mitigate difficulties encountered and improve the efficiency and quality of this difficult project. 


[return to main page](index.md)
