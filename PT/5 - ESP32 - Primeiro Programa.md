
# ESP32 - Primeiro Programa

Com o ambiente de desenvolvimento pronto já podemos começar a programar o software. Então para não começar do mais absoluto 0, vamos utilizar como base a lib  https://github.com/Wei1234c/SigmaDSP . O repositório _SigmaDSP_ é um pacote em Python desenvolvido para controlar os processadores de áudio digital ADAU1401 e ADAU1701 a partir de um PC ou ESP32. Ele permite a leitura de arquivos XML de projetos do SigmaStudio e a criação de objetos proxy que representam os algoritmos da Toolbox do SigmaStudio, possibilitando ajustes dinâmicos de parâmetros como frequência, nível em dB e coeficientes de filtros. O objetivo do projeto é viabilizar o ajuste automático de coeficientes de filtros sem depender exclusivamente do SigmaStudio, tornando o controle mais flexível e compatível com múltiplas plataformas, incluindo Windows, Linux, Raspberry Pi e ESP32.
Porém devido a aplicação genérica do projeto ele acaba não sendo ideal para aplicação embarcada onde a limitação de recursos é a base do desenvolvimento. Por isso na nossa aplicação específica vamos utilizar apenas a interface de comunicação com o DSP que foi desenvolvida para facilitar a comunicação entre o ESP32 e a ADAU1401. Eu criei uma fork do projeto para facilitar a importação dessa parte que é interessante pra uma aplicação mais específica em https://github.com/lvdopqt/sigmadsp_minimal

## Configuração Inicial

Vamos começar criando uma pasta para o nosso projeto e clonando o repositório `sigmadsp_minimal` dentro dela. Para isso, execute os seguintes comandos no terminal:

```bash
mkdir dsp_application
cd dsp_application
git clone https://github.com/lvdopqt/sigmadsp_minimal
```
*caso você não tenha o git instalado siga o tutorial https://github.com/git-guides/install-git

## Inicializando o Projeto

Agora que o repositório foi clonado, podemos abrir o editor de código de sua preferência e começar a desenvolver o software. O primeiro passo é criar um arquivo chamado `main.py`, que será o ponto de entrada da nossa aplicação. Neste arquivo, vamos importar a biblioteca que acabamos de clonar e configurar a comunicação com o DSP utilizando o protocolo I2C.

```python
from machine import SoftI2C, Pin

from sigmadsp_minimal.sigma_dsp.adau.adau1401.adau1401 import ADAU1401 as ADAU
from sigmadsp_minimal.bus.adapters import I2C as SigmaI2C

DSP_SCL_PIN=26
DSP_SCA_PIN=27
I2C_FREQ=400000

dsp_i2c = SoftI2C(scl=Pin(DSP_SCL_PIN), sda=Pin(DSP_SCA_PIN), freq=I2C_FREQ)
bus = SigmaI2C(dsp_i2c)
dsp = ADAU(bus)
```


Para configurar a comunicação entre o ESP32 e o DSP ADAU1401, começamos importando as bibliotecas e módulos necessários. O módulo `machine`, nativo do ESP32, fornece as funções `SoftI2C` e `Pin`, que são essenciais para configurar a comunicação I2C e controlar os pinos do microcontrolador. Além disso, importamos a classe `ADAU1401` da biblioteca `sigmadsp_minimal`, que representa o processador de áudio, e a interface `I2C`, renomeada como `SigmaI2C`, para adaptar a comunicação I2C ao formato esperado pela biblioteca.

Em seguida, definimos os pinos do ESP32 que serão utilizados para a comunicação I2C. O pino `DSP_SCL_PIN` (26) é responsável pelo sinal de clock (SCL), enquanto o pino `DSP_SDA_PIN` (27) é usado para o sinal de dados (SDA). A frequência de operação do barramento I2C é definida como 400 kHz, um valor comum para garantir uma comunicação rápida e estável.

Com os pinos configurados, inicializamos o barramento I2C utilizando a função `SoftI2C`. Essa função recebe como parâmetros os pinos SCL e SDA, além da frequência de operação, criando uma instância do barramento I2C pronta para uso. Em seguida, utilizamos a classe `SigmaI2C` para adaptar essa instância ao formato esperado pela biblioteca `sigmadsp_minimal`, garantindo que a comunicação seja compatível com as funções da biblioteca.

Por fim, criamos uma instância do DSP ADAU1401 utilizando a classe `ADAU` e passando o barramento I2C configurado como parâmetro. Essa instância permite interagir diretamente com o processador de áudio, ajustando parâmetros como frequência, ganho e coeficientes de filtros de forma dinâmica. Com essa configuração concluída, o ESP32 está pronto para se comunicar com o DSP e controlar o processamento de áudio de maneira flexível.

## Parametros do DSP

Agora vamos usar o arquivo `.params` que foi gerado pelo SigmaStudio. Geramos ele na parte 3 desse tutorial, mas caso não tenha gerado vá ao seu projeto no SigmaStudio > Export System Data. 
Esse arquivo possui as seguintes informações:

``` 
Cell Name         = Crossover1
Parameter Name    = CrossoverFilter2WayAlgSP1B0_0
Parameter Address = 1
Parameter Value   = 0.00102317333221436
Parameter Data :
0x00 ,	0x00 ,	0x21 ,	0x87 ,	
```

- **Cell Name**: Refere-se ao nome da célula ou bloco funcional no SigmaStudio. Neste caso, `Crossover1` indica que se trata de um crossover.
- **Parameter Name**: Representa o nome do parâmetro específico que está sendo configurado. Aqui, `CrossoverFilter2WayAlgSP1B0_0` refere-se a um dos coeficientes do filtro do crossover, que faz parte do algoritmo de processamento de áudio.
- **Parameter Address**: É o endereço de memória no DSP onde o parâmetro está localizado. O valor `1` indica a posição específica na memória do DSP onde esse coeficiente será armazenado ou modificado.
- **Parameter Value**: Corresponde ao valor numérico do parâmetro que está sendo configurado. Neste caso, `0.00102317333221436` é o valor do coeficiente do filtro, que influencia diretamente o comportamento do crossover.
- **Parameter Data**: Representa o valor do parâmetro no formato de dados brutos (hexadecimal) que está armazenado na memória do DSP. Aqui, `0x00, 0x00, 0x21, 0x87` são os bytes que codificam o valor do parâmetro no formato esperado pelo hardware do DSP.
As informações mais importantes para nossa aplicação são o **Parameter Name** e o **Parameter Address**. Se observarmos o arquivo de parametros usado, cada Crossover duas vias que representamos no SigmaStudio possui 20 parametros de filtragem (quatro conjuntos de B0, B1, B2, A1 e A2) e um parametro para inverter a polaridade da alta frequencia. 

## Introdução aos Filtros Biquad

Antes de prosseguirmos com a configuração dos parâmetros do DSP, é importante entender o conceito dos filtros biquad, que são amplamente utilizados no processamento de áudio digital. Um filtro biquad é um tipo de filtro digital que implementa uma função de transferência de segunda ordem, sendo capaz de realizar operações como equalização, crossover, supressão de ruídos e muito mais. Ele é chamado de "biquad" porque sua função de transferência pode ser expressa como uma razão de dois polinômios quadráticos. 
A estrutura de um filtro biquad é definida por cinco coeficientes principais: B0, B1, B2, A1 e A2. Esses coeficientes determinam o comportamento do filtro, como a frequência de corte, o ganho e o tipo de filtro (passa-baixa, passa-alta, passa-banda, etc.). No SigmaStudio, esses coeficientes são calculados automaticamente com base nas configurações do filtro que você define na interface gráfica, mas, ao trabalhar diretamente com o DSP, podemos ajustá-los programaticamente para personalizar o processamento de áudio.

A estrutura matemática de um filtro biquad é baseada em sua **função de transferência**, que descreve como o filtro processa o sinal de entrada para produzir o sinal de saída. Essa função de transferência é expressa como uma **razão de dois polinômios quadráticos**, ou seja, uma fração onde tanto o numerador quanto o denominador são polinômios de segunda ordem. A forma geral da função de transferência de um filtro biquad é:

!Pasted image 20250130193526.png

 O numerador (a parte de cima da função, onde estão os b) determina os **zeros** do filtro, enquanto o denominador (a parte de baixo) determina os **polos**. Os zˆ-1 representam o atraso de um sample.
O filtro biquad também é comumente representado na seguinte forma gráfica
!Pasted image 20250130194149.png
### Polos e Zeros

- **Zeros**: Os zeros de um filtro são as raízes do polinômio do numerador. Eles representam as frequências nas quais o filtro atenua completamente o sinal de entrada (ou seja, o ganho é zero). A localização dos zeros no plano complexo influencia diretamente a resposta em frequência do filtro, especialmente nas regiões de corte.
- **Polos**: Os polos são as raízes do polinômio do denominador. Eles determinam as frequências nas quais o filtro pode amplificar ou ressoar o sinal de entrada. A posição dos polos no plano complexo define a estabilidade do filtro e sua resposta em frequência nas regiões de passagem. Para que o filtro seja estável, todos os polos devem estar dentro do círculo unitário no plano complexo, mas não se preocupe com isso por enquanto! Essa informação está aqui apenas para informar que é possível fazer filtros que podem ser instáveis quando definindo esses parametros. Um filtro instável pode levar o ganho do filtro ao "infitino", o que em alguns sistemas de som pode causar danos.
### Resposta em Frequência

A resposta em frequência de um filtro biquad é determinada pela interação entre seus polos e zeros. Por exemplo:

- Um filtro **passa-baixa** terá polos que reforçam as baixas frequências e zeros que atenuam as altas frequências.
- Um filtro **passa-alta** terá polos que reforçam as altas frequências e zeros que atenuam as baixas frequências.
- Um filtro **passa-banda** terá polos e zeros posicionados de forma a reforçar uma faixa específica de frequências enquanto atenua as demais.


## Implementando Calculo de Coeficiente de Filtros

A Analog Devices disponibiliza um documento de como calcular os coeficientes para diferentes filtros. 
https://ez.analog.com/cfs-file/__key/communityserver-wikis-components-files/00-00-00-01-75/1307.FilterMathCalculations.pdf

Vamos fazer uma implementação simples de um filtro butterworth passa alta e um filtro passa baixa de primeira ordem baseada nesse documento! A definição dos filtros encontra-se abaixo

!Pasted image 20250130195415.png

Agora vamos reescrever em micropython

```python
from math import sin, cos, pi

def butterworth_lowpass(frequency, fs, g):
    # Cálculo do ganho
    gain = 10 ** (g / 20)
    
    # Cálculo de variáveis iniciais
    omega = 2 * pi * frequency / fs
    sn = sin(omega)
    cs = cos(omega)
    a0 = sn + cs + 1
    
    # Cálculo dos coeficientes do filtro passa-baixa
    A1 = (sn - cs - 1) / a0
    B0 = gain * sn / a0
    B1 = gain * sn / a0
    
    # Retorna os coeficientes no formato desejado
    return {'B0': B0, 'B1': B1, 'B2': 0, 'A1': A1, 'A2': 0}

def butterworth_highpass(frequency, fs, g):
    # Cálculo do ganho
    gain = 10 ** (g / 20)
    
    # Cálculo de variáveis iniciais
    omega = 2 * pi * frequency / fs
    sn = sin(omega)
    cs = cos(omega)
    a0 = sn + cs + 1
    
    # Cálculo dos coeficientes do filtro passa-alta
    A1 = (sn - cs - 1) / a0
    B0 = gain * (1 + cs) / a0
    B1 = -gain * (1 + cs) / a0
    
    # Retorna os coeficientes no formato desejado
    return {'B0': B0, 'B1': B1, 'B2': 0, 'A1': A1, 'A2': 0}
```

O uso dessas funções seria da seguinte forma:
```python
# Exemplo de uso para o filtro passa-baixa
lowpass_coeffs = butterworth_lowpass(1000, 44100, 6)
print("Coeficientes do filtro passa-baixa:", lowpass_coeffs)

# Exemplo de uso para o filtro passa-alta
highpass_coeffs = butterworth_highpass(1000, 44100, 6)
print("Coeficientes do filtro passa-alta:", highpass_coeffs)
```

Note que como são filtros de primeira ordem os parametros A2 e B2 são 0.

## Escrevendo os coeficientes no DSP

Agora que conseguimos calcular os coeficientes vamos fazer uma versão completa do nosso `main.py` !

```python 
from machine import SoftI2C, Pin
from sigmadsp_minimal.sigma_dsp.adau.adau1401.adau1401 import ADAU1401 as ADAU
from sigmadsp_minimal.bus.adapters import I2C as SigmaI2C
from math import sin, cos, pi

# Configuração da comunicação I2C com o DSP
DSP_SCL_PIN = 26
DSP_SDA_PIN = 27
I2C_FREQ = 400000

dsp_i2c = SoftI2C(scl=Pin(DSP_SCL_PIN), sda=Pin(DSP_SDA_PIN), freq=I2C_FREQ)
bus = SigmaI2C(dsp_i2c)
dsp = ADAU(bus)

# Funções para cálculo dos coeficientes do filtro Butterworth
def butterworth_lowpass(frequency, fs, g):
    # Cálculo do ganho
    gain = 10 ** (g / 20)
    
    # Cálculo de variáveis iniciais
    omega = 2 * pi * frequency / fs
    sn = sin(omega)
    cs = cos(omega)
    a0 = sn + cs + 1
    
    # Cálculo dos coeficientes do filtro passa-baixa
    A1 = (sn - cs - 1) / a0
    B0 = gain * sn / a0
    B1 = gain * sn / a0
    
    # Retorna os coeficientes no formato desejado
    return {'B0': B0, 'B1': B1, 'B2': 0, 'A1': A1, 'A2': 0}

def butterworth_highpass(frequency, fs, g):
    # Cálculo do ganho
    gain = 10 ** (g / 20)
    
    # Cálculo de variáveis iniciais
    omega = 2 * pi * frequency / fs
    sn = sin(omega)
    cs = cos(omega)
    a0 = sn + cs + 1
    
    # Cálculo dos coeficientes do filtro passa-alta
    A1 = (sn - cs - 1) / a0
    B0 = gain * (1 + cs) / a0
    B1 = -gain * (1 + cs) / a0
    
    # Retorna os coeficientes no formato desejado
    return {'B0': B0, 'B1': B1, 'B2': 0, 'A1': A1, 'A2': 0}

# Função para configurar o filtro no DSP
def configure_filter(dsp, frequency, fs, g, address):
    # Calcula os coeficientes para os filtros passa-alta e passa-baixa
    highpass_coeffs = butterworth_highpass(frequency, fs, g)
    lowpass_coeffs = butterworth_lowpass(frequency, fs, g)
    
    # Combina os coeficientes em uma única lista, na ordem correta
    coefficients = [
        highpass_coeffs['B0'], highpass_coeffs['B1'], highpass_coeffs['B2'],
        highpass_coeffs['A1'], highpass_coeffs['A2'],
        lowpass_coeffs['B0'], lowpass_coeffs['B1'], lowpass_coeffs['B2'],
        lowpass_coeffs['A1'], lowpass_coeffs['A2']
    ]
    
    # Converte os coeficientes para bytes e escreve na memória do DSP
    bytes_array = b''.join([dsp.DspNumber(v).bytes for v in coefficients])
    dsp.parameter_ram.write(bytes_array=bytes_array, address=address)

# Exemplo de configuração de filtro
frequency = 1000  # Frequência de corte em Hz
fs = 44100        # Taxa de amostragem em Hz
g = 6             # Ganho em dB
address = 1  # Endereço de memória no DSP onde os coeficientes serão escritos

configure_filter(dsp, frequency, fs, g, address)
```

Então resumindo:

Este código é uma implementação para configurar filtros de áudio Butterworth (passa-baixa e passa-alta) em um processador de sinal digital (DSP) ADAU1401, utilizando um microcontrolador ESP32. A primeira parte do código configura a comunicação I2C entre o ESP32 e o DSP, definindo os pinos e a frequência de operação. Em seguida, são criadas duas funções, `butterworth_lowpass` e `butterworth_highpass`, que calculam os coeficientes dos filtros de primeira ordem com base na frequência de corte, taxa de amostragem e ganho especificados. Essas funções retornam os coeficientes no formato de dicionário, incluindo os termos B0B0​, B1B1​, A1A1​, e definindo B2B2​ e A2A2​ como zero, já que são filtros de primeira ordem.

A função `configure_filter` utiliza as funções de cálculo de coeficientes para gerar os parâmetros dos filtros passa-alta e passa-baixa, combinando-os em uma lista na ordem correta. Esses coeficientes são então convertidos para o formato de bytes e escritos na memória do DSP no endereço especificado. No exemplo final, o código configura um filtro com frequência de corte de 1000 Hz, taxa de amostragem de 44100 Hz e ganho de 6 dB, escrevendo os coeficientes no endereço `1` do DSP. Essa abordagem permite ajustar dinamicamente o processamento de áudio sem depender de ferramentas externas, como o SigmaStudio.

## Conclusão

Este projeto demonstrou como implementar **filtros de áudio programáveis** em um DSP ADAU1401 usando um ESP32, combinando flexibilidade e eficiência para aplicações embarcadas. Ao utilizar a biblioteca `sigmadsp_minimal`, simplificamos a comunicação I2C com o DSP e eliminamos a dependência exclusiva do SigmaStudio, permitindo ajustes dinâmicos de coeficientes de filtros diretamente via código.

As funções `butterworth_lowpass` e `butterworth_highpass` ilustram como calcular coeficientes de filtros Butterworth de primeira ordem, enquanto a função `configure_filter` integra esses cálculos com a escrita na memória do DSP. Essa abordagem não apenas otimiza recursos limitados de sistemas embarcados, mas também abre portas para personalizações avançadas, como ajustes em tempo real ou implementação de filtros mais complexos (como passa-banda ou shelving).

Com a estrutura montada, o próximo passo seria expandir o código para suportar **filtros de ordem superior** ou integrar interfaces de usuário (como botões ou controle via Wi-Fi), transformando o sistema em uma solução versátil para processamento de áudio em tempo real. O código final, disponível no repositório forkado, serve como base para projetos que exigem controle autônomo sobre processadores de sinal digitais, consolidando-se como uma alternativa prática ao fluxo de trabalho tradicional baseado em ferramentas gráficas.