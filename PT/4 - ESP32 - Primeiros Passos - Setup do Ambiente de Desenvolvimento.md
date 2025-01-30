
## ESP32 - Primeiros Passos - Setup do Ambiente de Desenvolvimento

  
O [[ESP32]] é um [[microcontrolador]] avançado desenvolvido pela Espressif Systems, amplamente utilizado em projetos de IoT devido à sua alta performance e versatilidade. Ele combina conectividade Wi-Fi e Bluetooth integradas, processador dual-core, ampla memória flash e suporte a diversos protocolos e periféricos, como [[ADC]], [[DAC]], [[SPI]], [[I²C]] e [[UART]]. Com baixo consumo de energia e suporte a programação em linguagens como [[C]], C++ e [[MicroPython]], o [[ESP32]] é ideal para aplicações em automação residencial, dispositivos vestíveis e sistemas embarcados. O esquema do [[ESP32]] pode ser encontrado [aqui](link_do_esquema).

Para começar o processo de desenvolvimento, vamos instalar o [[firmware]] de MicroPython para o ESP32. A escolha de MicroPython para programar o ESP32 justifica-se por sua simplicidade, flexibilidade e eficiência. Com sintaxe legível, bibliotecas nativas para [[I²C]] e [[SPI]], e suporte total às funcionalidades do ESP32 (Wi-Fi, Bluetooth), ele acelera o desenvolvimento e facilita a integração com o [[ADAU1401]]. Além disso, caso sejam necessárias funções mais robustas de processamento ou cálculo, é possível utilizar o *inline assembler* do MicroPython para otimizar trechos específicos do código. Isso combina facilidade de uso com alta performance, ideal para este projeto.

  

Na [documentação do MicroPython](https://docs.micropython.org), é possível encontrar o passo a passo de como fazer a instalação do firmware no ESP32. Em um breve resumo e tradução do que está presente na documentação:

  
1. **Instalação do Python**:  

   Você precisa do [[Python]] instalado em sua máquina. Isso varia de acordo com o sistema operacional ([[OS]]) que você usa, mas [aqui](https://www.python.org/downloads/) você encontra como fazer isso para os [[OS]] mais populares.

  

2. **Instalação do [[esptool]]**:  

   Em um terminal, use o seguinte comando:  

   ```bash

   pip install esptool

   ```

  

3. **Conexão do ESP32**:  

   Conecte o ESP32 à máquina via USB.

4. **Identificação da Porta**:  

   Identifique qual porta o [[ESP32]] está utilizando. [Neste artigo](link_do_artigo), é possível encontrar o procedimento para cada OS.

5. **Apagar o Firmware**:  

   Apague o [[firmware]] da placa utilizando o comando:  

  
   ```bash 
   esptool.py --chip esp32 --port /dev/ttyUSB0 erase_flash  
```
  
   Substitua `ttyUSB0` pela porta identificada no passo anterior.

  
6. **Download do Firmware**:  

   Faça o download da versão do firmware que deseja instalar. Por padrão, utilizo o último *release*. [Aqui](https://micropython.org/download/esp32/) você pode fazer o download do arquivo do firmware.


7. **Instalação do Novo Firmware**: 

   Para inserir o novo firmware, use o comando: 

   
   ```bash
   esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x1000 esp32-20190125-v1.10.bin
```

   Substitua `ttyUSB0` pela porta identificada e `esp32-20190125-v1.10.bin` pelo nome do arquivo do firmware baixado. Execute o comando e o [[ESP32]] estará pronto para uso!

Para os próximos posts, vou utilizar o **[[VSCode]]** como editor de texto e a extensão **[[Pymakr]]** para agilizar e integrar a comunicação do ESP32 com o fluxo de escrita de código. No entanto, você pode utilizar o editor de texto que preferir e fazer a comunicação com o [[ESP32]] da maneira que estiver mais habituado.


### Conclusão

Neste post, demos os primeiros passos para configurar o [[ESP32]], um componente essencial do nosso projeto de processador de áudio. Instalamos o firmware do MicroPython, que será a base para a programação do microcontrolador, e preparamos o ambiente de desenvolvimento para as próximas etapas. A escolha do [[MicroPython]] traz vantagens significativas, como facilidade de uso, suporte a protocolos de comunicação e a possibilidade de otimização de código quando necessário.
Com o ESP32 configurado, estamos prontos para avançar na integração com o [[ADAU1401]], criando uma interface de controle dinâmica e funcional. Nos próximos posts, exploraremos como programar o [[ESP32]] para ajustar parâmetros do [[DSP]] em tempo real, utilizando componentes como display [[LCD]], encoder rotativo e botões. Essa integração permitirá uma experiência de uso mais interativa e personalizada, elevando o projeto a um novo patamar.
Continue acompanhando esta série para descobrir como unir o poder do [[ESP32]] com a precisão do [[ADAU1401]], criando um sistema de áudio completo e altamente customizável!

  