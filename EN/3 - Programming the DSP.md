# Programming the DSP

The ADAU14/1701 development board found on Chinese sales platforms generally follows this schematic:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXf-BtzXrCPv1qZj9R8NTyYs0qSRPgcdQJB_4zomV8hIHu2bsHFXCQYP7ffJ75-HtAFjltnYQiLSjsbpCpsZsjeUCczC1hF8WFAf9WmPESB_8fMnS9EemmucdjVlblrhBJ1eNyHOUw?key=JqK6MG9S1obsx3xawZmOUk87)

  

## Board Overview

#### Main Components

At the center of the board, the ADAU1401 is the heart of the system, handling all audio processing operations. It is surrounded by connectors that allow:

- **Digital inputs and outputs**: SDATA0, SDATA1, SDATA2, and SDATA3 for digital audio transmission and reception.
- **Analog inputs**: IN1 and IN2 connectors, accompanied by ground (GND) connections.
- **Analog outputs**: Connectors such as OUT1 and OUT2 to deliver processed audio.

#### Power Supply

The board operates with a 5V input and includes voltage regulators and capacitors to stabilize power. An important note: the input voltage should not exceed 6.5V.

#### Configuration and Control

- **MP0 to MP11 pins** (MP = Multi-Purpose) offer configurable functionalities for application-specific customization. You can connect buttons, potentiometers, encoders, etc.
- A **Write Protect (WP) pin** controls writing to the EEPROM memory. This pin is connected to Ground (GND) when writing to memory that needs to persist after the board is powered off.
- **GND, SDA, and SCL pins** are used for I2C or SPI communication.

#### Crystal Oscillator

The board supports an external MCLK (Master Clock). If using an external clock, ensure the sampling rate configured in the software is compatible with the oscillator. The ADAU1701 can process up to 8 simultaneous digital audio channels, with each channel requiring 32 bits (24 bits of data + 8 bits of margin/control). At a 96 kHz sampling rate, the required clock speed would be 24.576 MHz:

$Clock (MCLK) = Sample\ Rate \times Bit\ Depth \times Number\ of\ Channels$

  

## Programming the Board

Analog Devices, the manufacturer of the DSP, provides a software called **SigmaStudio** to design the DSP's processing flow. For many applications, this programming step alone is sufficient.  

Unfortunately, SigmaStudio only runs on Windows, which posed a minor obstacle for this project. This limitation led me to explore alternative solutions for programming the device from non-Windows machines. However, let’s first walk through the workflow using SigmaStudio on a Windows PC.

To connect the device to the PC, we need an **I²C-to-USBi converter** and the appropriate drivers. The workflow using an AliExpress I²C USB converter was as follows:

- Directly connect the USBi pins (SDA, SCL, GND, and 5V/VDD) to the board and plug it into the PC. Surprisingly straightforward!  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcZ3PIz22cUMxwHnsC0ShZrEcYyBVy28J4vZOmbtDAvohJbg4-nZfzca5RM_NclO8krmBoVBCdurwJsjxVsOwK9pC1QYtL7zijrO6HyndCoWcPOeYABFu6GMSKBHziqiqoVwoFcXg?key=JqK6MG9S1obsx3xawZmOUk87)

As a cheaper alternative to the USBi converter, modules using the **[CY7C68013A](https://pt.aliexpress.com/item/32875331830.html?spm=a2g0n.order_detail.order_detail_item.2.12524c7f5qdQy2&_gl=1*1iq26bs*_gcl_aw*R0NMLjE3MzA3NDAyMTAuQ2owS0NRaUFfcUc1QmhEVEFSSXNBQTBVSFNLOC1aN0tvSlR4elYyN0lRMlJqT1BrOFkwUG4wc2pCOFFTUXplUTRNZnpqM1VfQWhqWm55b2FBa2tvRUFMd193Y0I.*_gcl_dc*R0NMLjE3MzA3NDAyMTAuQ2owS0NRaUFfcUc1QmhEVEFSSXNBQTBVSFNLOC1aN0tvSlR4elYyN0lRMlJqT1BrOFkwUG4wc2pCOFFTUXplUTRNZnpqM1VfQWhqWm55b2FBa2tvRUFMd193Y0I.*_gcl_au*NjQyNjYyODk0LjE3MjQ1OTk4NDY.*_ga*MTkwNDM4MjI3My4xNjg4OTE3MzY2*_ga_VED1YSGNC7*MTczMDk4NzkzNi40OS4xLjE3MzA5ODc5NTcuMzkuMC4w&gatewayAdapt=glo2bra)** integrated circuit can be used. A [detailed tutorial](https://daumemo.com/how-to-program-an-analog-devices-dsp/) describes this connection, driver installation, and other topics covered here—worth checking out!

After [downloading and installing SigmaStudio](https://www.analog.com/en/resources/evaluation-hardware-and-software/software/ss_sigst_02.html) (version 4.7 was used here), open the software. From the **Tree Toolbox**, select the **ADAU1401**, **E2PROM** under *Processors (ICs/DSPs)*, and **USBi** under *Communication Channels*. The interface should look like this:  
<img src="../Images/Pasted image 20250127191631.png"/>
  

Click **Schematic** to start designing the application flow! For initial testing, I created a simple input-output passthrough. From the Tree Toolbox, select **Input** and **Output** components, connect them, and save the program to the DSP using **Link, Compile, and Download**. Note: To permanently save the program, an additional step is required (covered later).  

<img src="../Images/Pasted image 20250127191735.png"/>

Physically, connect **IN_1** and **GND** to a P10 jack input, and **OUT_1** and **GND** to an output jack. Let’s test the application and the hardware simultaneously.

  

## Testing with REW  

We’ll use **[REW (Room EQ Wizard)](https://www.roomeqwizard.com/)** to test the DSP. REW is a free, widely-used tool for acoustic analysis and calibration. It measures, analyzes, and optimizes audio systems and room acoustics.  

The process we’ll use is **Frequency Response Analysis**, which measures how a device or system responds to different frequencies. The diagram below illustrates the general signal flow:  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfLIpEwinjGULooQLNZDrKamPWrEKQOQ2GFWzvHuFnV-9RjuuFqJcA0fG4IcEcfwNVDn2bk1SblKYvhaQSgE4cyneZ0IfOLpf-IISPu4fzyDCENobR69GcpVVpQNp-bF7d1SGN59w?key=JqK6MG9S1obsx3xawZmOUk87)

  

### Process Steps:

1. **Signal Generator**:
   - A known signal (e.g., sine sweep or pink noise) is sent to the device under test. A sine sweep (example [here](https://www.youtube.com/watch?v=dU80Fagdy28)) continuously varies its frequency over time, covering a specific range. It excites the system one frequency at a time, allowing precise measurement of amplitude and phase response.

2. **Device Under Test**:
   - The reference signal passes through the device, which alters it based on its characteristics (amplification, attenuation, distortion, etc.).

3. **Response Calculation**:
   - The ratio of the output (y) to input (x) is calculated and converted to decibels (dB) to represent gain/loss at each frequency.

4. **Graph Display**:
   - The result is plotted as frequency vs. gain (dB), showing the device’s frequency response. This is called a **Transfer Curve** or **Frequency Response Curve**, representing how the system modifies the signal in amplitude and phase.  

For our setup, the signal flow is as follows:  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcw4MHm9ItfITQmUBdC8ljVxxB-2Fdao_jok5Rrq5rgsP2hppzcnbBw6da7w9qaPVsxMY9i0AwBx2fI4M42rc7NNFZr-B_NStpl6XzvssWc8ITeBCSSCFcoXZ-qoXP0SdYh9PFzoQ?key=JqK6MG9S1obsx3xawZmOUk87)

The audio signal exits the interface (OUT 1/2), enters the ADAU1401 inputs (IN), is processed (filters, EQ, etc.), and loops back to the interface (IN 2). The processed signal returns via the ADAU1401 outputs (OUT) to the interface inputs (IN 1). This loop allows real-time testing, calibration, and analysis of the ADAU1401 using SigmaStudio and REW.  

In REW, configure the input/output routing to match your audio interface.  
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeRhS1_VSwY95mx56lEtkMZXvRKzLNTC1oVwLocn_fG459_ita58-1865XiTeyZ7JrWymChHxUkTaQqbTckiX1ZrWE5ltTy64o3ouC4JTvcDltmZcStsIMaZoLNPWeYzIFQvEUA?key=JqK6MG9S1obsx3xawZmOUk87)

For accurate measurements, calibrate the audio interface using **Calibrate Soundcard** to compensate for distortions.  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXePVa34VHmG9rAgzjER8Zlgd4mJfp3IcLdJO5btF0PYUgEEuHVFC9FMUEfHc9sy8vGpMPNkRESUILeN1jr_uvOhHv1y5cTVSndOTZsGxc2W9IEBMQBSM9n3b0DkI_Ry2NR4YFmm?key=JqK6MG9S1obsx3xawZmOUk87)

  

Click **Measure** in REW’s top-left corner to open the measurement settings:  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdHsMRn9ykLK9EfOfOSiTwxMOhAyw6D3nTh_w0AMOhF-aUq6EN2EIs7xLHZIpTdc8lr8aUTr_wjG_irRCqTCEpAPQRs97v28liDOGImliXpFwqHS5YzaU_04oxm7P5jScv8DTZF3g?key=JqK6MG9S1obsx3xawZmOUk87)

### Measurement Settings

1. **Measurement Type**:
   - Choose *SPL* for frequency response analysis.

2. **Measurement Identification**:
   - Name the measurement (e.g., "ADAU1401 Response").

3. **Frequency Range**:
   - Set start/end frequencies (e.g., 20 Hz – 20 kHz).

4. **Signal Level**:
   - Set to -5.00 dBFS to avoid clipping.

5. **Abort on Clipping**:
   - Enable to halt measurement if distortion occurs.

6. **Method**:
   - Select *Sweep* for continuous frequency excitation.

7. **Sweep Settings**:
   - Resolution: 256k for precision.
   - Repetitions: 1 (sufficient for most cases).
   - Timing: *Set t=0 at IR peak*.

8. **Playback/Input**:
   - **Playback**: Use REW’s internal generator.
   - **Sample Rate**: Match your system (e.g., 44.1/96 kHz).
   - **Output**: Select the audio interface output channel.
   - **Input**: Select the input channel (e.g., Input 1).

Click **Start** to run the sweep (5.9s duration). The resulting graph shows:  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdekbIqrquTPhtLYHgJjp538whx2S6kTtE5VFPlyxYU9D2nLGQUW0VO_kK-GrRlnzAwy--ieyelXqXPDA7bHX-HUdpsYOlbeEy-hnQIDE06lh2GQYzA437pP2h_VRcPHaMT4Wf-?key=JqK6MG9S1obsx3xawZmOUk87)

The magnitude curve is stable in the mid-frequency range, indicating a flat response. A slight roll-off occurs at low (<20 Hz) and high (>19 kHz) frequencies, typical of most systems. The phase response is linear with minimal variation.  

Repeating the measurement across all channels confirms consistency, ensuring the DSP can handle precise filtering and effects.  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcuICXK_8SlHzvOD3JKlLPhOCqcfENcrz1fRqUBtqvq41pgGykNuBfHpLhaTaF6_73-UWKvoKDFaT9XzL3LKEmTmek9LXV1d9aschFA7Z51dJB0YfTjpnIMbNnaA9QBhMzljHg9Hw?key=JqK6MG9S1obsx3xawZmOUk87)

Now, let’s create a permanent DSP program!  

In SigmaStudio, I designed the following schematic with a crossover filter set to 500 Hz:  

<img src="../Images/Pasted image 20250127192259.png"/>

After adjusting the crossover filters and clicking **Link, Compile, and Download**, save the program to the DSP’s EEPROM. In **Hardware Configuration**, right-click **IC 1 (ADAU1401)** and select **Write Latest Compilation to E2PROM**. Before confirming, connect the **WP pin to GND** to enable permanent storage.  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcSSSRxNC_bVBSbtaGuQthpGwzKoHgoTIwcPOxYY7buP4sPdg3pDbNFaxlbtdUzJHXvFjewk5o3sfxqagMRR6OnA6PTqhLOowQKfr9irFQoxvDEcp3KSNObaW0wbthWcuFBTbZd?key=JqK6MG9S1obsx3xawZmOUk87)

Post-crossover measurements show the expected high-pass and low-pass filtering:  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXf_oCX1C3DYHS8DtfYlGUQadAb8Nn7-s3drbBoFJOCpLk-hvRkxdnKYcBZzbEEEYoGgNkmc0ZLQ07yyh7KYOqRvBhDWZO6Sk1uIKvxW9VohwliuotJotrGHBJLSaFflsfXI6AdX0w?key=JqK6MG9S1obsx3xawZmOUk87)

For many applications, this workflow suffices. However, we’ll take it further by integrating an ESP32 for dynamic parameter control!  

## Exporting Parameters

To prepare for ESP32 integration, export the DSP parameters from SigmaStudio:  
1. Go to **Tools > Export Parameters**.  
2. Save the `.params` file for later use.  

## Conclusion

In this post, we explored programming the ADAU1401 DSP using SigmaStudio, from board overview and signal flow design to frequency response testing with REW. We validated the DSP’s performance, implemented a 500 Hz crossover, and saved the configuration to EEPROM.  

This concludes the basic DSP programming phase. In the next post, we’ll integrate an **ESP32** to dynamically control DSP parameters and create a user-friendly interface. This addition will unlock real-time adjustments and interactive audio processing!  

Stay tuned to learn how to combine the ADAU1401’s power with the ESP32’s versatility for a fully customizable audio system!