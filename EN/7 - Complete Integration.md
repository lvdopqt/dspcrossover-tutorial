# 7 - Complete Integration  

## **Installation and Setup**  

### **1. Prerequisites**  

- **Hardware**:  
    - All hardware used is described [in part 1 of the project](1%20-%20Introdução.md), you can go back there to check!  
    - Connections as per the schematic (`https://github.com/lvdopqt/dspcrossover/blob/master/docs/Audio%20Processor%20-%20Schematics%20-%20Rev002.pdf`).  

- **Software**:  
    - MicroPython installed on the ESP32. [Part 4 of the tutorial](4%20-%20ESP32%20-%20Primeiros%20Passos%20-%20Setup%20do%20Ambiente%20de%20Desenvolvimento.md)  
    - Clone the library and save it on the ESP32:  
        `git clone https://github.com/seu-usuario/dsp-crossover-project`  

---

## **Program Overview**  

### **Main Components**  

- **Event Bus**: Coordinates communication between components (e.g., encoder click → LCD update).  
- **Rotary Encoder**: Navigates the menu and adjusts values (rotate right/left, click to select).  
- **Back Button**: Returns to the previous menu or cancels actions.  
- **LCD**: Displays menus and status.  
- **CrossoverService**: Calculates filter coefficients (low-pass/high-pass) and configures the DSP. This service also saves the filter settings in non-volatile memory, meaning that even after being turned off, the system retains the configured filters.  

<img src="../Images/Pasted image 20250130203043.png" width=600px />  
<img src="../Images/Pasted%20image%2020250130203108.png" width=600px />  

## **Expanding the Project**  

- **New Filters**: Add band-pass or shelving filters in `features/crossover/service.py`.  
- **Remote Control**: Integrate Wi-Fi/Bluetooth for adjustments via an app (use the Event Bus to send commands).  
- **New Audio Processing**: Implement a compressor, reverb, oscillator, or any other audio processing you can imagine!  

For questions, check the schematic or open an _issue_ in the repository! 🎛️🔊