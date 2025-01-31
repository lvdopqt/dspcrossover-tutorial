# Introduction  

This is the beginning of a series of posts dedicated to building an audio processor using widely known components, such as the ESP32 controller and the ADAU1401 signal processor.  

First of all, what is an audio processor? You may have seen something similar in cars, like volume controls, equalization, or left/right channel balance adjustments. Or perhaps in a VST plugin within a DAW or even in a multi-effects pedal. In general, an audio processor can be understood as a filter: it receives an audio signal, applies a transformation (processing), and delivers the transformed signal. The possibilities for transformations are nearly endless, limited only by creativity and the engineering required to implement them.  

The capabilities of an audio processor depend largely on the hardware used in the project. Here, we will explore the **ADAU1401**, an integrated circuit from Analog Devices Inc. that is compact, efficient, and widely used in professional and embedded sound systems. The ADAU1401 combines features such as high-quality A/D and D/A converters, support for sampling rates up to 96 kHz, integrated SRAM memory, and I²C/SPI interfaces (we’ll cover these in detail in future posts). It can be purchased as a standalone chip or as part of development boards, which include basic configurations for ease of use. On AliExpress, for example, these boards are available at affordable prices for development projects.  

The kit I’ll use in this series includes an ADAU1401/1701-DSP board and an I²C/SPI-to-USB converter, as shown in the image below.  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeEy7C9h_W5IvDS7HQ4BzSjP41n86WKiKewXmsft8LmklovZHhRT4qryPsbygiohlKiJS1ZpeD12yVUKC1YymqsezE8Oc5we5QCAbLhy_KjbS1adB-0jmahuldt_hr6jt2SMiUx?key=JqK6MG9S1obsx3xawZmOUk87)  

In this series, we’ll use this board for a specific application: **frequency splitting in a sound system**, a process known as **crossover**. Our goal is to create a **digital crossover**. The ADAU1401 has 2 audio inputs and 4 outputs, allowing us to receive two distinct signals and generate four processed outputs. In this project, the inputs will receive L/R (left/right) signals, which will be split into two frequency bands per channel: lows (bass) and highs (treble).  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdGjzjThDzF6RgLHwQlDuftCupWuULFs66gKUVIfufFXRTxR0H1aoAw7gj3PsbU67KTFA2GeYnyQ_saUKvKsz7kIVQ1UC1oETxClCakQYAO628naMXyJvxYToTLODHha8oITwqtaw?key=JqK6MG9S1obsx3xawZmOUk87)  

In upcoming posts, we’ll explore how to configure the ADAU1401 for this application and other possibilities.  

To control the DSP parameters and create a user-friendly interface, we’ll use the **ESP32**. Developed by Espressif Systems, the ESP32 is a highly integrated microcontroller ideal for IoT, automation, and connected devices. It offers a powerful processor (dual-core or single-core Xtensa LX6), Wi-Fi and Bluetooth connectivity (including BLE), and a robust set of peripherals such as multifunctional GPIOs, ADCs, DACs, UARTs, SPI, I²C, and PWM. Its versatility and wireless capabilities make it perfect for building control interfaces.  

In this project, the ESP32 will control the DSP via an interface that includes an LCD screen, a rotary encoder with a click function, and an additional button. These components will allow adjusting crossover frequencies and gain/volume for each frequency band in a practical way.  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdkCE54mkvIbHRbK-JCJttvcLJyf-tGJvxo_fS2FVWUJGMmbB7PjhTrgIL_Jj6iEyYCVjOBfSskSlBkZkg1BRcj-UHvrxMXcUbLC_gpwX4Ta_gH_HDfnMwbQuA2-SH5Ob_QmbCfXg?key=JqK6MG9S1obsx3xawZmOUk87)  

Below is the list of materials for this project:  
- [ESP32 with Wi-Fi and Bluetooth](https://pt.aliexpress.com/item/1005005704190069.html?spm=a2g0o.productlist.main.19.446044dePsYVIL&algo_pvid=68de7e0b-4ec1-4e11-8377-8572f6b42b37&algo_exp_id=68de7e0b-4ec1-4e11-8377-8572f6b42b37-9&pdp_npi=4%40dis%21BRL%2127.77%2111.63%21%21%2131.19%2113.07%21%402103010e17358278398134744eb6e6%2112000034061352780%21sea%21BR%210%21ABX&curPageLogUid=5y9dDRW3kmZN&utparam-url=scene%3Asearch%7Cquery_from%3A)  
- [ADAU1401 Development Board](https://pt.aliexpress.com/item/1005006283208362.html?spm=a2g0o.productlist.main.1.218056d2WsZyLb&algo_pvid=645fc524-6604-4a54-b5de-6be44d5b3940&algo_exp_id=645fc524-6604-4a54-b5de-6be44d5b3940-0&pdp_npi=4%40dis%21BRL%2193.84%2150.78%21%21%2114.37%217.78%21%402101c59117358279336365481ed41a%2112000036602876531%21sea%21BR%210%21ABX&curPageLogUid=bRVXD1iUUSeL&utparam-url=scene%3Asearch%7Cquery_from%3A)  
- [16x2 LCD Display with I2C Converter](https://www.mercadolivre.com.br/display-lcd-16x2-serial-i2c-com-backlight-azul/p/MLB29573766?pdp_filters=item_id:MLB3569474467#is_advertising=true&searchVariation=MLB29573766&position=1&search_layout=grid&type=pad&tracking_id=1cfa90cb-fab1-42e7-8a0f-b4cff35dfa61&is_advertising=true&ad_domain=VQCATCORE_LST&ad_position=1&ad_click_id=YTMyZTZlMmItMjhlNi00OTNhLTg1ZWUtZjgyZWZiN2M0YmI0)  
- [SMD Tactile Button](https://produto.mercadolivre.com.br/MLB-3172016605-10x-chave-tactil-smd-4-terminais-6x6x43mm-180graus-_JM)  
- [Rotary Encoder with Click](https://produto.mercadolivre.com.br/MLB-4972063902-5x-potencimetro-rotativo-5-pinos-encoder-ec11-c-click-20mm-_JM)  
- Wires for connections  

**Tools for assembly and testing**:  
- Soldering Iron and Solder Wire  
- Wire Cutters  
- Utility Knife or Wire Stripper  
- Audio Interface (I use a UMC404HD)  
- P10 Cables  

---

## Conclusion  

In this introduction, we’ve outlined the concept of an audio processor and the key components for building a digital crossover: the ESP32 controller and the ADAU1401 signal processor. Combined with an LCD display, rotary encoder, and buttons, these devices will enable an intuitive and functional control interface for real-time audio adjustments.  

Throughout this series, we’ll dive into configuring and integrating these components, from physical assembly to programming and testing. The goal is not only to build an efficient digital crossover but also to demonstrate how hardware and software can be combined to create customized, high-quality audio processing solutions.  

Stay tuned for upcoming posts, where we’ll explore practical steps, share insights, and provide tips so you can replicate or adapt this idea for your own projects. Let’s explore the possibilities of audio engineering and embedded electronics together!