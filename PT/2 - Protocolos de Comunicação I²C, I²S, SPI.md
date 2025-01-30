
# Protocolos de Comunicação I²C, I²S, SPI

No universo de sistemas embarcados e dispositivos eletrônicos, a comunicação entre componentes é um dos pilares para criar projetos eficientes e funcionais. Dois protocolos serão abordados no nosso projeto: o I²C, e I²S, cada um com características próprias que atendem a diferentes necessidades. Vamos explorar o que cada um deles oferece e quando são usados.

Os protocolos digitais de comunicação são formas padronizadas de transmitir informações entre dispositivos eletrônicos, utilizando sinais elétricos que oscilam entre dois níveis: alto (geralmente representado por uma tensão positiva, como 3,3V ou 5V) e baixo (geralmente 0V, ou terra). Esses níveis correspondem aos estados binários 1 e 0, formando a base da comunicação digital. Quando dizemos que "o sinal está alto" ou "o sinal está baixo", estamos nos referindo a essas variações de tensão no barramento ¹, que são interpretadas pelos dispositivos como comandos, dados ou sinais de controle. Essas transições, sincronizadas por um clock em protocolos síncronos, permitem que os dispositivos se comuniquem de forma precisa e confiável.

¹ Um barramento é uma conexão compartilhada que permite a comunicação entre diferentes dispositivos eletrônicos, transportando sinais elétricos que representam dados, endereços e comandos.

## I²C (Inter-Integrated Circuit)

O protocolo I²C é amplamente usado para comunicação entre componentes em projetos de baixo custo e baixa complexidade. Ele utiliza apenas dois fios: um para o sinal de dados (SDA) e outro para o sinal de clock (SCL), o que reduz a quantidade de pinos necessários para estabelecer um ponto de comunicação entre dois ou mais componentes. No I²C, os dispositivos são organizados como controller e targets, onde o controller controla o clock e inicia a comunicação. É um protocolo eficiente para enviar pequenos volumes de dados em baixa velocidade, sendo comumente encontrado em sensores, RTCs (relógios de tempo real) e displays. Cada dispositivo tem um endereço, o que permite ao controller enviar dados específicos para cada target em um sistema.

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfXZ_1JWNXDygiMXiAJMbKLoVnICroNY2H7MgDaKNsp2pH6kzo6P8fi1GnJrFTlREM56txDaoySuoLeveOzUJCAKnm52iDSbrebt0zoE6cPhKen0k8J901PAf_MEXdXX-1gq6Kx8g?key=JqK6MG9S1obsx3xawZmOUk87)

#### Como Funciona o Envio de Dados no I²C

- Início da Comunicação: O controlador gera uma condição de start, baixando o sinal SDA enquanto o SCL está alto, sinalizando o início de um pacote de dados.
- Endereçamento: O controlador envia um endereço de 7 ou 10 bits que identifica o target desejado. O target correspondente reconhece o endereço e responde com um ACK (acknowledge) baixando o SDA no próximo pulso de clock.

- Transmissão de Dados:
	- Os dados são enviados em pacotes de 8 bits, começando pelo bit mais significativo (MSB). Cada byte é seguido de um bit de ACK enviado pelo target para confirmar o recebimento.
	- Para um exemplo de pacote de 32 bits, o controlador envia quatro blocos consecutivos de 8 bits, cada um confirmado pelo ACK do target.
	- Término da Comunicação: Após o envio de todos os dados, o controlador gera uma condição de stop, elevando o SDA enquanto o SCL permanece alto.

  

## I²S (Integrated Interchip Sound)

O I²S é projetado exclusivamente para comunicação de áudio digital. Ele utiliza um barramento serial dedicado para transmitir fluxos de áudio com alta fidelidade, envolvendo três sinais principais:

- SD (Serial Data): Transporta os dados de áudio digital.
- WS (Word Select): Define qual canal (esquerdo ou direito) está sendo transmitido.
- SCK (Serial Clock): Sincroniza o envio de dados.

#### Como Funciona o Envio de Dados no I²S

- Estrutura do Pacote:
- Os dados de áudio são transmitidos em palavras (palavras de 16, 24 ou 32 bits são comuns).
- Para uma palavra de 32 bits, cada canal (esquerdo e direito) ocupa 32 ciclos de clock.
- Sincronização por Word Select:
- Quando o sinal WS está baixo, os 32 bits do canal esquerdo são transmitidos no barramento SD.
- Quando o WS está alto, os 32 bits do canal direito são transmitidos.
- Transmissão em Série:
- Os dados são enviados começando pelo bit mais significativo (MSB) e sincronizados pelo SCK.
- O ciclo de clock SCK define a taxa de amostragem, enquanto o WS garante que os dados sejam entregues ao canal correto.

## SPI

O **SPI** é um protocolo de comunicação serial **síncrono**, **full-duplex** e de alta velocidade, amplamente utilizado em sistemas embarcados para transferência rápida de dados entre dispositivos. Diferentemente do I²C, o SPI não possui endereçamento embutido e utiliza uma arquitetura **controller-target**, onde o controller controla o clock e seleciona os targets via sinal dedicado.

### Características Principais

- **Velocidade Elevada**: Opera em frequências de clock muito maiores que I²C (MHz), ideal para transferência de grandes volumes de dados.
    
- **Full-Duplex**: Transmite e recebe dados simultaneamente.
    
- **Sinais Principais**:
    
    - **SCLK (Serial Clock)**: Gerado pelo _controller_ para sincronização.
    - **MOSI (Controller Out Target In)**: Dados enviados do _controller_ ao _target_.
    - **MISO (Controller In Target Out)**: Dados enviados do _target_ ao _controller_.
    - **SS/CS (Target Select/Chip Select)**: Sinal para selecionar o _target_ ativo (um pino por _target_).
        

---

### Funcionamento do SPI

1. **Seleção do Target**:  
    O _controller_ inicia a comunicação ativando o pino SS/CS do _target_ desejado (nível baixo).
    
2. **Transmissão de Dados**:
    
    - O _controller_ gera o clock (**SCLK**) e envia dados pelo **MOSI**, enquanto o _target_ responde pelo **MISO**.
    - Os dados são transmitidos em blocos de 8 ou 16 bits, geralmente começando pelo bit mais significativo (**MSB**).
    - Não há confirmação de recebimento (**ACK**) como no I²C, o que simplifica a comunicação, mas exige sincronização precisa.
        
3. **Modos de Clock**:  
    O SPI define quatro modos de operação, combinando **polaridade do clock (CPOL)** e **fase (CPHA)**:
    
    - **Modo 0** (CPOL=0, CPHA=0): Dados são amostrados na borda de subida do clock.
    - **Modo 3** (CPOL=1, CPHA=1): Dados são amostrados na borda de subida com clock inicial em nível alto.
        
4. **Término da Comunicação**:  
    O _controller_ desativa o sinal **SS/CS** (nível alto), finalizando a transmissão.
    

---

### Vantagens e Desvantagens

- **Vantagens**:
    
    - **Alta velocidade** e simplicidade de implementação.
    - Ideal para dispositivos que exigem **throughput elevado**, como memórias (EEPROM, Flash) ou displays.
        
- **Desvantagens**:
    
    - Requer **mais pinos** que o I²C (1 pino SS por _target_).
    - Não possui **controle de erro** ou **endereçamento nativo**, limitando sua escalabilidade em sistemas complexos.

---

### Aplicações no Projeto

No processador de áudio, o **SPI** pode ser usado para comunicação com periféricos de alta velocidade, como:

- **Memórias externas** para armazenamento de configurações ou firmware.
- **Controladores de display** para interfaces gráficas complexas.
- **Sensores especializados** que exigem atualização rápida de dados.
## Conclusão

Neste post, exploramos três protocolos de comunicação essenciais para o nosso projeto de processador de áudio: **I²C**, **I²S** e **SPI**. Cada um desempenha um papel específico e complementar no sistema. O **I²C**, com sua simplicidade e eficiência, será utilizado para a comunicação de controle entre o ESP32 e o ADAU1401, permitindo o ajuste de parâmetros e a configuração do processador de sinal. O **I²S**, projetado exclusivamente para áudio digital, garantirá a integridade e fidelidade do sinal entre os componentes, assegurando a transmissão precisa dos dados para os canais esquerdo e direito. Já o **SPI**, com sua alta velocidade e capacidade full-duplex, será responsável por tarefas que demandam _throughput_ elevado, como comunicação com memórias externas ou controladores de display, agregando flexibilidade ao sistema.

Entender o funcionamento desses protocolos é fundamental para a implementação do projeto, pois eles são a base da comunicação entre os dispositivos. No próximo post, avançaremos para a configuração prática do ADAU1401, utilizando o SigmaStudio, uma ferramenta poderosa da Analog Devices para programação de DSPs. Com isso, começaremos a dar vida ao nosso crossover digital, integrando os conceitos teóricos apresentados aqui à prática de engenharia de áudio – incluindo a otimização do SPI para funções secundárias críticas.

Fique ligado para mais detalhes e continue acompanhando esta série para dominar a construção de um processador de áudio personalizado!
