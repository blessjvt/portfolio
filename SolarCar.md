# SolarCarVT Telemetry and Embedded Work

## Telemetry & Legacy System Cleanup

Gained access to legacy SolarCar telemetry GitHub repos and Raspberry Pi system, discovered poor documentation and mismatches between code on the car and code in the repo, and documented how things actually worked. 

Reverse-engineered the telemetry pipeline: traced data from CAN bus → can_receive.cpp socket → server.js parsing → driver UI, and mapped how battery, motor, and temperature data were encoded and displayed. 

Set up remote access to the Pi (static IP + VNC), so the team could monitor and debug the UI and data logging without physically connecting every time. 


## Data Logging & Analysis

Designed and implemented data-logging features to export CAN data to CSV/text files, including experimenting with sampling rates and buffering strategies so logs were accurate but not overly large. 

Analyzed VTTI test-session logs: filtered out invalid runs, checked that signals (battery current, voltage, etc.) made physical sense, and started basic filtering techniques to support later performance analysis. 

## Embedded Firmware & CAN Bus Work

Helped build a low-fidelity prototype for the low-voltage panel and then moved into embedded firmware for a new CAN controller using Arduino IDE / C++. 

Debugged CAN communication issues (including incorrect baud rates) and confirmed what needed to be logged: SOC, battery current/voltage, max cell temp, motor speed, and key status messages. 

Fixed a long-standing bug in how battery voltage/current were decoded: discovered from BMU documentation that both values share a single CAN ID but use different byte ranges, then rewrote the parsing logic and validated the new values on real data. 

## GPS, Timing Architecture & New Embedded System

Helped integrate a GPS module to provide an accurate UTC timestamp for logs; worked through bad vendor documentation (UART vs. USB) with a teammate to get reliable GPS data. 

Assisted with integrating a new embedded system for VTTI test session #2, combining CAN, GPS, and logging on updated hardware. 

Implemented a non-blocking timing approach using multiple one-shot timers to coordinate main loop, CAN processing, GPS, I2C sensors, display updates, SD card writes, telemetry sending, and serial printing without blocking the system. 

## SD Card Logging & Power/MPPT Work

Debugged SD-card mounting on the RP2040 (CS pin selection and bootloader issues) so the microcontroller could reliably log data to the card. 

Took over Power team coding deliverables during Thanksgiving break: implemented solar-rig data logging so the team could run daily solar tests. 

Performed LoRa radio range testing by collecting data while biking, measuring when telemetry packets started to be delayed (~1 mile range), and documenting the behavior. 

## Solar MPPT CAN Integration (RP2040 + Elmar MPPT)

Wrote and tested CAN-bus code on an Adafruit Feather RP2040 to communicate with the Elmar MPPT, including wiring, crimping connectors, and bringing up the full test setup. 

<img width="429" height="577" alt="SCRP2040" src="https://github.com/user-attachments/assets/a3272c65-c7c6-4733-9314-7d846813e392" />

<img width="640" height="610" alt="SCFig1" src="https://github.com/user-attachments/assets/446b1508-4d18-479d-8090-9f0f6a2d73ee" />

Corrected the MPPT base address (matching the rotary-switch setting) and the baud rate, then used Prohelion Profinity to confirm that the HEX IDs and decoded values matched the manufacturer’s GUI. 

<img width="629" height="140" alt="SCFig3" src="https://github.com/user-attachments/assets/4253de92-2487-4498-af70-a6a0b8595cbe" />

<img width="637" height="292" alt="SCFig2" src="https://github.com/user-attachments/assets/31a62986-1ca7-4765-9aea-2fabecd4b9fb" />
