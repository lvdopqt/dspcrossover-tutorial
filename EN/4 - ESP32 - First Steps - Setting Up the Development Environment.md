# ESP32 - First Steps - Development Environment Setup  

The ESP32 is an advanced microcontroller developed by Espressif Systems, widely used in IoT projects due to its high performance and versatility. It combines built-in Wi-Fi and Bluetooth connectivity, a dual-core processor, extensive flash memory, and support for various protocols and peripherals such as ADC, DAC, SPI, I²C, and UART. With low power consumption and support for programming in languages like C, C++, and MicroPython, the ESP32 is ideal for applications in home automation, wearables, and embedded systems. The ESP32 pinout diagram can be found [here](https://www.makerstore.com.au/wp-content/uploads/2024/10/ELEC-ESP32-DEV-MOD-PINOUT.webp).  

To begin the development process, we will install the MicroPython firmware on the ESP32. MicroPython was chosen for programming the ESP32 due to its simplicity, flexibility, and efficiency. With a readable syntax, native libraries for I²C and SPI, and full support for ESP32 functionalities (Wi-Fi, Bluetooth), it speeds up development and simplifies integration with the ADAU1401. Additionally, if more robust processing or computation functions are needed, you can use MicroPython's *inline assembler* to optimize specific code sections. This combination of ease of use and high performance makes it ideal for this project.  

In the [MicroPython documentation](https://docs.micropython.org), you can find step-by-step instructions on how to install the firmware on the ESP32. Below is a summarized and translated version of what is covered in the documentation:  

---

## **Python Installation**  

You need Python installed on your computer. The installation process varies depending on your operating system (OS), but you can find instructions for the most popular OS [here](https://www.python.org/downloads/).  

## **Installing esptool**  

Open a terminal and run the following command:  

```bash
pip install esptool
```  

## **Connecting the ESP32**  

Connect the ESP32 to your computer via USB.  

## **Identifying the Port**  

Identify which port the ESP32 is using. [This article](link_to_article) provides instructions for each OS.  

## **Erasing the Firmware**  

Erase the board’s firmware using the command:  

```bash
esptool.py --chip esp32 --port /dev/ttyUSB0 erase_flash  
```  

Replace `ttyUSB0` with the port identified in the previous step.  

## **Downloading the Firmware**  

Download the firmware version you wish to install. By default, I use the latest *release*. [Here](https://micropython.org/download/esp32/) you can download the firmware file.  

## **Installing the New Firmware**  

To flash the new firmware, use the command:  

```bash
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x1000 esp32-20190125-v1.10.bin
```  

Replace `ttyUSB0` with the identified port and `esp32-20190125-v1.10.bin` with the name of the downloaded firmware file. Run the command, and the ESP32 will be ready for use!  

For the next steps, I will use **VSCode** as my text editor and the **Pymakr** extension to streamline and integrate ESP32 communication with the code-writing workflow. However, you can use any text editor of your choice and communicate with the ESP32 in whichever way you are most comfortable.  

---

## **Conclusion**  

In this post, we took the first steps to configure the ESP32, an essential component of our audio processor project. We installed the MicroPython firmware, which will serve as the foundation for programming the microcontroller, and prepared the development environment for the next stages. Choosing MicroPython brings significant advantages, such as ease of use, support for communication protocols, and the ability to optimize code when needed.  

With the ESP32 configured, we are ready to move forward with integration into the ADAU1401, creating a dynamic and functional control interface. In the upcoming posts, we will explore how to program the ESP32 to adjust DSP parameters in real-time, using components such as an LCD display, rotary encoder, and buttons. This integration will enable a more interactive and customizable user experience, taking the project to a new level.  

Stay tuned for this series to learn how to combine the power of the ESP32 with the precision of the ADAU1401, creating a complete and highly customizable audio system!  