
# Programando o DSP

A placa de desenvolvimento da ADAU14/1701 encontrada nas plataformas de venda chinesa possui em geral esse esquema.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXf-BtzXrCPv1qZj9R8NTyYs0qSRPgcdQJB_4zomV8hIHu2bsHFXCQYP7ffJ75-HtAFjltnYQiLSjsbpCpsZsjeUCczC1hF8WFAf9WmPESB_8fMnS9EemmucdjVlblrhBJ1eNyHOUw?key=JqK6MG9S1obsx3xawZmOUk87)

  

## Visão Geral da Placa

#### Componentes Principais

No centro da placa, o ADAU1401 é o coração do sistema, lidando com todas as operações de processamento de áudio. Ele é cercado por conectores que permitem:

- Entradas e saídas digitais: SDATA0, SDATA1, SDATA2, e SDATA3 para transmissão e recepção de áudio digital.
- Entradas analógicas: Conectores IN1 e IN2, acompanhados por conexões de terra (GND).
- Saídas analógicas: Conectores como OUT1 e OUT2 para entregar áudio processado.

#### Alimentação

A placa opera com uma entrada de 5V e inclui reguladores de tensão e capacitores para estabilizar a alimentação. Uma nota importante é que a tensão de entrada não deve exceder 6,5V.

#### Configuração e Controle

- Os pinos MP0 a MP11 (MP=Multi purpose) oferecem funcionalidades configuráveis para personalização de acordo com a aplicação. Você pode conectar botões, potenciometros, encoders etc
- Um pino de Write Protect (WP) é utilizado para controlar a escrita na memória EEPROM. Esse pino é conectado ao Ground (GND) quando estamos executando uma escrita na memória que vai ser preservada mesmo quando a placa desligar! 
- Os pinos (GND), SDA e SCL são utilizados para a comunicação via I2C ou SPI.

#### Cristal Oscilador

A placa suporta um MCLK (Master Clock) externo. Caso opte por usar um clock externo é necessário que a taxa de amostragem configurada no software seja compatível com o oscilador.  A ADAU1701 é capaz de processar até 8 canais simultâneos no domínio digital, e cada canal de áudio requer 32 bits (24 bits de dados + 8 bits de margem ou controle). Em uma taxa de amostragem de 96KHz o clock necessário seria de 24.576 MHz

$Clock (MCLK)= Sample Rate× Bit Depth × Número de Canais$

  
## Programação da Placa

A Analog Devices fabricante do DSP fornece um software chamado SimgaStudio que foi desenvolvido para projetar o fluxo em que o DSP vai trabalhar. Em muitas aplicações, somente essa etapa de programação é necessária.

  Infelizmente esse fluxo só é possível num sistema operacional Windows, o que pra mim foi um pequeno obstáculo no projeto, mas o que me levou a buscar outras soluções e imaginar a programação desse dispositivo sendo realizada por outras máquinas que não fossem Windows. Mas vamos por partes e ver como o fluxo é feito através do SimgaStudio em um PC windows

Para que seja feita a conexão entre o dispositivo e o PC precisamos utilizar um conversor I²C para USBi, e instalar um driver apropriado na sua máquina. O fluxo utilizando o conversor I²C usb do ali express foi o seguinte:

- Conectei diretamente os pinos do USBi que contém o mesmo nome na placa SDA, SCL, GND e 5V(VDD) e pluguei no PC. Estranhamente simples (?)

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcZ3PIz22cUMxwHnsC0ShZrEcYyBVy28J4vZOmbtDAvohJbg4-nZfzca5RM_NclO8krmBoVBCdurwJsjxVsOwK9pC1QYtL7zijrO6HyndCoWcPOeYABFu6GMSKBHziqiqoVwoFcXg?key=JqK6MG9S1obsx3xawZmOUk87)

Como alternativa a esse conversor USBi que sejam mais baratas é possível utilizar outros conversores de I²C para serial como módulos que utilizam o circuito integrado [CY7C68013A](https://pt.aliexpress.com/item/32875331830.html?spm=a2g0n.order_detail.order_detail_item.2.12524c7f5qdQy2&_gl=1*1iq26bs*_gcl_aw*R0NMLjE3MzA3NDAyMTAuQ2owS0NRaUFfcUc1QmhEVEFSSXNBQTBVSFNLOC1aN0tvSlR4elYyN0lRMlJqT1BrOFkwUG4wc2pCOFFTUXplUTRNZnpqM1VfQWhqWm55b2FBa2tvRUFMd193Y0I.*_gcl_dc*R0NMLjE3MzA3NDAyMTAuQ2owS0NRaUFfcUc1QmhEVEFSSXNBQTBVSFNLOC1aN0tvSlR4elYyN0lRMlJqT1BrOFkwUG4wc2pCOFFTUXplUTRNZnpqM1VfQWhqWm55b2FBa2tvRUFMd193Y0I.*_gcl_au*NjQyNjYyODk0LjE3MjQ1OTk4NDY.*_ga*MTkwNDM4MjI3My4xNjg4OTE3MzY2*_ga_VED1YSGNC7*MTczMDk4NzkzNi40OS4xLjE3MzA5ODc5NTcuMzkuMC4w&gatewayAdapt=glo2bra). Existe um [tutorial](https://daumemo.com/how-to-program-an-analog-devices-dsp/) específico descrevendo os detalhes dessa conexão e a instalação do driver e até tópicos que vamos cobrir um pouco aqui! Vale a pena ver!

Depois de fazer o [download e a instalação do SigmaStudio](https://www.analog.com/en/resources/evaluation-hardware-and-software/software/ss_sigst_02.html). (No momento que escrevi isso estava usando a versão 4.7) Ao abrir o software ele apresenta a tela inicial do programa. Da aba Tree ToolBox vamos selecionar os elementos ADAU1401, E2PRom da seção Processors (ICs / DSPs), e o USBi da seção Communication Channels. A tela deve ficar assim:
<img src="../Images/Pasted image 20250127191631.png"/>
  

Agora é só clicar em schematic e começar a desenhar o fluxo da aplicação! Resolvi criar uma primeira aplicação para testar as entradas e saídas da placa. Selecionei da Tree ToolBox os elementos Input e Output. Então conectei os dois e salvei o programa no DSP (opção Link, Compile and Download). Note que para salvar definitivamente o programa é necessário mais uma etapa que faremos em breve!

<img src="../Images/Pasted image 20250127191735.png"/>

No lado físico do DSP conectei IN_1 e GND a uma entrada Jack P10, assim como a saída OUT_1 e GND. Bom, agora vamos testar a aplicação que fizemos, e ao mesmo tempo o componente que utilizamos.


## Utilizando o REW para teste  

Vamos utilizar [o software REW](https://www.roomeqwizard.com/) para realizar os testes no DSP. O Room EQ Wizard (REW) é um software gratuito e amplamente utilizado para análise e calibração acústica. É uma ferramenta poderosa e versátil que ajuda profissionais de áudio, engenheiros de som e entusiastas a medir, analisar e otimizar o comportamento acústico de sistemas de som e ambientes. 

O processo que utilizaremos é chamado de “Análise de Resposta de  Frequência”. Ele é usado para medir como um dispositivo ou sistema responde a diferentes frequências de um sinal de referência. A imagem abaixo ilustra de forma genérica como é fluxo desse processo:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfLIpEwinjGULooQLNZDrKamPWrEKQOQ2GFWzvHuFnV-9RjuuFqJcA0fG4IcEcfwNVDn2bk1SblKYvhaQSgE4cyneZ0IfOLpf-IISPu4fzyDCENobR69GcpVVpQNp-bF7d1SGN59w?key=JqK6MG9S1obsx3xawZmOUk87)

  

### Etapas do Processo:

1. Gerador de  Sinal:

- Um sinal conhecido, como um sweep senoidal ou ruído rosa, é enviado para o dispositivo ou sistema que será analisado. O sweep senoidal (pode ser ouvido [aqui](https://www.youtube.com/watch?v=dU80Fagdy28)) é um sinal que varia sua frequência de forma contínua ao longo do tempo, cobrindo uma faixa específica para análise acústica ou eletrônica. O sweep senoidal é considerado ilustrativo para a análise de resposta de frequência porque ele excita o sistema com uma frequência por vez, de forma contínua e controlada, permitindo medir detalhadamente como o dispositivo responde a cada frequência em termos de amplitude e fase. Isso resulta em uma visão clara e precisa das características de frequência do sistema analisado.


3. Dispositivo Analisado:

- O sinal de referência passa pelo dispositivo analisado, que altera o sinal de acordo com suas características (amplificação, atenuação, distorção, etc.).

5. Cálculo da Resposta:

- A razão entre o sinal de saída (y) e o sinal de entrada (x) é calculada. Esse valor é convertido para decibéis (dB) para representar a diferença em termos de ganho ou perda em cada frequência.

7. Exibição do Gráfico:

- O resultado é representado em um gráfico de frequência vs. ganho (dB), mostrando como o dispositivo responde a diferentes frequências.
    
- Essa representação apresentada no gráfico pode ser chamada de Curva de Transferência ou Curva de Resposta em Frequência, dependendo do contexto. 
    
- A "curva de transferência" descreve como um sistema ou dispositivo transfere energia ou informações de entrada para saída em diferentes frequências. Ela representa graficamente a relação entre o sinal de entrada (referência) e o sinal de saída em termos de amplitude e, fase.
    

1. "Transferência" refere-se ao processo de como o sistema modifica o sinal.
    
2. O gráfico mostra a proporção entre entrada e saída em cada frequência expressa em decibéis (dB), e a fase expressa em graus.
    

Na nossa aplicação em específico a ligação desse sistema seria da seguinte forma:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcw4MHm9ItfITQmUBdC8ljVxxB-2Fdao_jok5Rrq5rgsP2hppzcnbBw6da7w9qaPVsxMY9i0AwBx2fI4M42rc7NNFZr-B_NStpl6XzvssWc8ITeBCSSCFcoXZ-qoXP0SdYh9PFzoQ?key=JqK6MG9S1obsx3xawZmOUk87)

  

O diagrama ilustra a conexão entre o processador de áudio digital ADAU1401 e uma interface de áudio, formando um loop de sinal para processamento e análise. O sinal de áudio sai da interface através das saídas (OUT 1 e OUT 2) e é enviado para as entradas (IN) do ADAU1401, onde é processado (filtros, equalização, etc.) . O sinal também é enviado em loop para a entrada da interface (IN 2). O sinal processado retorna pelas saídas (OUT) do ADAU1401 para as entradas da interface (IN 1). Essa configuração é ideal para testar, calibrar e analisar o comportamento do ADAU 1401 em tempo real, permitindo ajustes e verificações detalhadas por meio do Sigma Studio e do  REW.

No REW acessaremos as configurações e definiremos esse roteamento de entradas e saídas da placa.  
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeRhS1_VSwY95mx56lEtkMZXvRKzLNTC1oVwLocn_fG459_ita58-1865XiTeyZ7JrWymChHxUkTaQqbTckiX1ZrWE5ltTy64o3ouC4JTvcDltmZcStsIMaZoLNPWeYzIFQvEUA?key=JqK6MG9S1obsx3xawZmOUk87)

Para uma medição mais precisa é necessário também adicionar um arquivo de calibração da placa de áudio. Ele pode ser feito clicando no botão “Calibrate soundcard” e seguindo as etapas! Essa etapa é importante para compensar pequenas distorções que a placa de áudio pode adicionar ao sistema, garantindo que  o sinal medido seja correspondente a puramente o sistema que queremos analisar, no nosso caso a placa da ADAU 1401.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXePVa34VHmG9rAgzjER8Zlgd4mJfp3IcLdJO5btF0PYUgEEuHVFC9FMUEfHc9sy8vGpMPNkRESUILeN1jr_uvOhHv1y5cTVSndOTZsGxc2W9IEBMQBSM9n3b0DkI_Ry2NR4YFmm?key=JqK6MG9S1obsx3xawZmOUk87)

  

Vamos clicar em “Measure” no canto esquerdo superior na tela principal do REW. A seguinte tela deve aparecer:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdHsMRn9ykLK9EfOfOSiTwxMOhAyw6D3nTh_w0AMOhF-aUq6EN2EIs7xLHZIpTdc8lr8aUTr_wjG_irRCqTCEpAPQRs97v28liDOGImliXpFwqHS5YzaU_04oxm7P5jScv8DTZF3g?key=JqK6MG9S1obsx3xawZmOUk87)

Configurações de Medição

1. Tipo de Medição (Type):
	- Escolha entre SPL (nível de pressão sonora) ou Impedance.
	- Para análises de resposta em frequência, mantenha-se em SPL.

2. Identificação da Medição:
	- No campo Name, insira um nome descritivo para a medição (ex.: “Resposta do ADAU1401”).
	- Escolha se deseja adicionar data/hora automaticamente ou configurar manualmente.

3. Faixa de Frequência (Range):
	- Defina a frequência inicial (Start Freq) e final (End Freq). Para análises gerais, use a faixa completa de 20 Hz a 20.000 Hz.

4. Nível do Sinal (Level):
	- Ajuste o nível do sinal em dBFS. Geralmente, valores como -5.00 dBFS são usados para evitar clipping no sistema analisado.

5. Abortar em Clipping (Abort if heavy input clipping occurs):
	- Deixe essa opção ativada para interromper a medição caso o sinal seja muito forte e cause distorções.

6. Método (Method):
	- Escolha Sweep para realizar uma varredura senoidal contínua (ideal para análise de resposta em frequência).

7. Configurações do Sweep:
	- Settings: Escolha a resolução (ex.: 256k para maior precisão).
	- Repetitions: Defina o número de repetições (normalmente 1 é suficiente e o software vai t e notificar caso você esteja usando configurações abaixo do necessário).
	- Timing: Ajuste a referência temporal (geralmente deixe em "Set t=0 at IR peak").

8. Reprodução e Entrada
	1.Playback:
	- Escolha From REW para usar o gerador interno de sinais do software.
	- Configure a Sample Rate para corresponder ao seu sistema (ex.: 44.1 kHz ou 96 kHz).

9. Entrada e Saída:
- Output: Selecione o canal de saída para o sinal de teste (ex.: Default Output, ou escolha L/R conforme o dispositivo).
- Input: Verifique se o canal de entrada está configurado corretamente (ex.: Input 1).

Clicando em “Start” e aguardando o tempo do sweep (5.9s na configuração acima), temos o seguinte gráfico como resposta:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdekbIqrquTPhtLYHgJjp538whx2S6kTtE5VFPlyxYU9D2nLGQUW0VO_kK-GrRlnzAwy--ieyelXqXPDA7bHX-HUdpsYOlbeEy-hnQIDE06lh2GQYzA437pP2h_VRcPHaMT4Wf-?key=JqK6MG9S1obsx3xawZmOUk87)

  

A curva de magnitude é relativamente estável na faixa central de frequências, indicando uma resposta plana e consistente nessa região.Há uma leve queda na amplitude nas frequências mais baixas (< 20 Hz) e mais altas (> 19 kHz), o que é comum devido às limitações do sistema. Já a curva de fase indica uma fase bem linear. As variações apresentadas nela são de pouquíssimos graus apesar da representação em forma de escada.

Repetindo a medição de resposta em todos os canais é possível ver que são bem semelhante, o que garante que o equipamento possa fazer operações precisas de filtragem e efeitos.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcuICXK_8SlHzvOD3JKlLPhOCqcfENcrz1fRqUBtqvq41pgGykNuBfHpLhaTaF6_73-UWKvoKDFaT9XzL3LKEmTmek9LXV1d9aschFA7Z51dJB0YfTjpnIMbNnaA9QBhMzljHg9Hw?key=JqK6MG9S1obsx3xawZmOUk87)

Agora vamos fazer o software que vai ser colocado de forma definitiva no DSP!

Criei um schema no Sigma Studio que ficou dessa forma:

<img src="../Images/Pasted image 20250127192259.png"/>


Clicando no gráfico do componente do Crossover ajustei os filtros de crossover para o corte em 500Hz . “Link Compile and Download”. 

<img src="../Images/Pasted image 20250127192312.png"/>

Vamos também salvar o software na memória do DSP para que continue funcionando depois de reiniciado. Voltamos em Hardware Configuration, clicando com o botão direito no IC 1 (ADAU1401), “Write Latest Compilation to E2PROM”. Antes de clicar no OK na tela de prompt apresentada, faça um jump entre WP e GND conforme a imagem abaixo. Isso permite que o DSP salve configurações de maneira permanente na memória.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcSSSRxNC_bVBSbtaGuQthpGwzKoHgoTIwcPOxYY7buP4sPdg3pDbNFaxlbtdUzJHXvFjewk5o3sfxqagMRR6OnA6PTqhLOowQKfr9irFQoxvDEcp3KSNObaW0wbthWcuFBTbZd?key=JqK6MG9S1obsx3xawZmOUk87)

Agora refazendo a medição nos canais tenho essa medida:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXf_oCX1C3DYHS8DtfYlGUQadAb8Nn7-s3drbBoFJOCpLk-hvRkxdnKYcBZzbEEEYoGgNkmc0ZLQ07yyh7KYOqRvBhDWZO6Sk1uIKvxW9VohwliuotJotrGHBJLSaFflsfXI6AdX0w?key=JqK6MG9S1obsx3xawZmOUk87)

Para várias aplicações esse fluxo já é o suficiente. Você pode criar vários tipos de aplicação somente usando a placa de DSP! Mas nós vamos um pouco além e criar uma integração com um ESP32 para fazer controle de parametros!  

## Exportando o arquivo de parametros

A última etapa desse processo a fim de realizar a integração com o ESP32 é fazer a exportação do arquivo de parametros do software que escrevemos no DSP. Para isso acesse o SigmaStudio > Tools > Export Parameters. Selecione a pasta onde os parametros serão exportados e clique em `OK` . Vamos utilizar o arquivo `.params`  nas próximas etapas, então guarde ele em algum lugar! 

## Conclusão

Neste post, mergulhamos na programação do DSP ADAU1401, utilizando o software SigmaStudio para criar e testar um fluxo de processamento de áudio. Começamos com uma visão geral da placa de desenvolvimento, destacando seus componentes principais e funcionalidades, como entradas e saídas analógicas e digitais, alimentação e pinos de configuração. Em seguida, exploramos o processo de programação do DSP, desde a conexão com o PC via conversor I²C/USB até a criação de um fluxo básico no SigmaStudio.

Utilizamos o software REW (Room EQ Wizard) para realizar testes de resposta em frequência, verificando a precisão e a consistência do processamento de áudio no ADAU1401. Esse processo nos permitiu analisar a curva de transferência do sistema, garantindo que o DSP esteja funcionando conforme o esperado. Por fim, configuramos um crossover digital com corte em 500Hz e salvamos o programa na memória EEPROM do DSP, assegurando que as configurações sejam mantidas mesmo após o desligamento da placa.

Este post marca a conclusão da etapa de programação básica do DSP, mas o projeto está longe de terminar. No próximo post, avançaremos para a integração do ESP32, que será responsável por controlar os parâmetros do DSP de forma dinâmica e criar uma interface amigável para o usuário. Essa etapa trará ainda mais flexibilidade e funcionalidade ao nosso processador de áudio, permitindo ajustes em tempo real e uma experiência de uso mais interativa.

Continue acompanhando esta série para descobrir como unir o poder do DSP ADAU1401 com a versatilidade do ESP32, criando um sistema de áudio completo e personalizado!