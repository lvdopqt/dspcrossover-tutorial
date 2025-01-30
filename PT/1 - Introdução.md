
# Introdução

Este é o início de uma série de posts dedicados à construção de um processador de áudio utilizando componentes amplamente conhecidos, como o controlador ESP32 e o processador de sinal ADAU1401.

Antes de mais nada, o que é um processador de áudio? Talvez você já tenha visto algo parecido em carros, como controles de volume, equalização ou o balanço entre os lados L/R. Ou talvez em um VST dentro de uma DAW ou até mesmo em um pedal com múltiplos efeitos. De forma geral, um processador de áudio pode ser entendido como um filtro: ele recebe um sinal de áudio, aplica uma transformação (o processamento) e entrega o sinal transformado. As possibilidades de transformação são quase infinitas, limitadas apenas pela criatividade e pela engenharia necessária para implementá-las.

O que um processador de áudio é capaz de fazer depende em grande parte do equipamento utilizado no projeto. Neste caso, vamos explorar o ADAU1401, um circuito integrado da Analog Devices Inc. que é compacto, eficiente e amplamente utilizado em sistemas de som profissionais e embarcados. O ADAU1401 combina recursos como conversores A/D e D/A de alta qualidade, suporte a frequências de amostragem de até 96 kHz, memória SRAM integrada e interfaces I²C e SPI (vamos abordar isso em detalhes nos próximos posts). Ele pode ser adquirido como um chip ou como parte de placas de desenvolvimento, que já incluem configurações básicas para facilitar o uso. No AliExpress, por exemplo, é possível encontrar essas placas a preços acessíveis para projetos de desenvolvimento.

O kit que utilizarei nesta série inclui uma placa ADAU1401/1701-DSP e um conversor I²C/SPI para USB, como mostrado na imagem abaixo.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeEy7C9h_W5IvDS7HQ4BzSjP41n86WKiKewXmsft8LmklovZHhRT4qryPsbygiohlKiJS1ZpeD12yVUKC1YymqsezE8Oc5we5QCAbLhy_KjbS1adB-0jmahuldt_hr6jt2SMiUx?key=JqK6MG9S1obsx3xawZmOUk87)

  

Nesta série, utilizaremos esta placa para uma aplicação específica: a divisão de frequências em um sistema de som, um processo conhecido como crossover. Nosso objetivo será criar um "crossover digital". A ADAU1401 possui 2 entradas e 4 saídas de áudio, o que nos permite receber dois sinais distintos e gerar quatro saídas processadas. Na aplicação deste projeto, as entradas receberão sinais L e R (esquerda e direita), que serão separados em duas faixas de frequência para cada canal: graves e agudos.

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdGjzjThDzF6RgLHwQlDuftCupWuULFs66gKUVIfufFXRTxR0H1aoAw7gj3PsbU67KTFA2GeYnyQ_saUKvKsz7kIVQ1UC1oETxClCakQYAO628naMXyJvxYToTLODHha8oITwqtaw?key=JqK6MG9S1obsx3xawZmOUk87)

Nos próximos posts, veremos como configurar o ADAU1401 para essa aplicação e outras possibilidades.

Para controlar os parâmetros do DSP e criar uma interface amigável para o usuário, utilizaremos o ESP32. Desenvolvido pela Espressif Systems, o ESP32 é um microcontrolador altamente integrado, ideal para aplicações como IoT, automação e dispositivos conectados. Ele oferece um processador poderoso (dual-core ou single-core Xtensa LX6), conectividade Wi-Fi e Bluetooth (incluindo BLE), além de um conjunto robusto de periféricos como GPIOs multifuncionais, ADCs, DACs, UARTs, SPI, I²C e PWM. Sua versatilidade e capacidade de comunicação sem fio o tornam perfeito para criar interfaces de controle.

No projeto, o ESP32 será usado para controlar o DSP, com uma interface que incluirá uma tela LCD, um rotary encoder com clique e um botão adicional. Com esses componentes, será possível ajustar as frequências do crossover e controlar o ganho/volume de cada faixa de forma prática.

  
  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdkCE54mkvIbHRbK-JCJttvcLJyf-tGJvxo_fS2FVWUJGMmbB7PjhTrgIL_Jj6iEyYCVjOBfSskSlBkZkg1BRcj-UHvrxMXcUbLC_gpwX4Ta_gH_HDfnMwbQuA2-SH5Ob_QmbCfXg?key=JqK6MG9S1obsx3xawZmOUk87)

  

Abaixo a lista de materiais que será utilizada para esse projeto:

- [Esp32 com Wifi e Bluetooth](https://pt.aliexpress.com/item/1005005704190069.html?spm=a2g0o.productlist.main.19.446044dePsYVIL&algo_pvid=68de7e0b-4ec1-4e11-8377-8572f6b42b37&algo_exp_id=68de7e0b-4ec1-4e11-8377-8572f6b42b37-9&pdp_npi=4%40dis%21BRL%2127.77%2111.63%21%21%2131.19%2113.07%21%402103010e17358278398134744eb6e6%2112000034061352780%21sea%21BR%210%21ABX&curPageLogUid=5y9dDRW3kmZN&utparam-url=scene%3Asearch%7Cquery_from%3A) 
- [Placa de desenvolvimento Adau1401](https://pt.aliexpress.com/item/1005006283208362.html?spm=a2g0o.productlist.main.1.218056d2WsZyLb&algo_pvid=645fc524-6604-4a54-b5de-6be44d5b3940&algo_exp_id=645fc524-6604-4a54-b5de-6be44d5b3940-0&pdp_npi=4%40dis%21BRL%2193.84%2150.78%21%21%2114.37%217.78%21%402101c59117358279336365481ed41a%2112000036602876531%21sea%21BR%210%21ABX&curPageLogUid=bRVXD1iUUSeL&utparam-url=scene%3Asearch%7Cquery_from%3A)
- [Display LCD 16x2 com conversor i2c](https://www.mercadolivre.com.br/display-lcd-16x2-serial-i2c-com-backlight-azul/p/MLB29573766?pdp_filters=item_id:MLB3569474467#is_advertising=true&searchVariation=MLB29573766&position=1&search_layout=grid&type=pad&tracking_id=1cfa90cb-fab1-42e7-8a0f-b4cff35dfa61&is_advertising=true&ad_domain=VQCATCORE_LST&ad_position=1&ad_click_id=YTMyZTZlMmItMjhlNi00OTNhLTg1ZWUtZjgyZWZiN2M0YmI0)
- [Botão  de  Click SMD](https://produto.mercadolivre.com.br/MLB-3172016605-10x-chave-tactil-smd-4-terminais-6x6x43mm-180graus-_JM)
- [Encoder rotativo com click](https://produto.mercadolivre.com.br/MLB-4972063902-5x-potencimetro-rotativo-5-pinos-encoder-ec11-c-click-20mm-_JM)
- Fios para conexão entre as partes

As ferramentas utilizadas para montagem e testes:

- Ferro de Solda e Arame de Solda
- Alicate de corte
- Estilete ou decapador de fios
- Interface de Audio (utilizo uma UMC404HD)
- Cabos P10

  

## Conclusão

Nesta introdução, apresentamos o conceito de um processador de áudio e os principais componentes que serão utilizados para construir um crossover digital: o controlador ESP32 e o processador de sinal ADAU1401. Esses dispositivos, combinados com outros elementos como um display LCD, encoder rotativo e botões, permitirão criar uma interface de controle intuitiva e funcional para ajustar parâmetros de áudio em tempo real.

Ao longo desta série, exploraremos em detalhes a configuração e integração desses componentes, desde a montagem física até a programação e testes. O objetivo é não apenas construir um crossover digital eficiente, mas também demonstrar como a combinação de hardware e software pode ser aplicada para criar soluções personalizadas e de alta qualidade em processamento de áudio.

Fique atento aos próximos posts, onde mergulharemos nas etapas práticas do projeto, compartilhando conhecimentos e dicas para que você possa replicar ou adaptar essa ideia em seus próprios projetos. Vamos juntos explorar as possibilidades que a engenharia de áudio e a eletrônica embarcada têm a oferecer!

