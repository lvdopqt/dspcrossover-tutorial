# 7 - Integração Completa


## **Instalação e Configuração**

### **1. Pré-requisitos**

- **Hardware**:
    - Todo hardware usado está descrito [na parte 1 do projeto](1%20-%20Introdução.md), pode voltar lá para conferir!
    - Conexões conforme esquemático (`https://github.com/lvdopqt/dspcrossover/blob/master/docs/Audio%20Processor%20-%20Schematics%20-%20Rev002.pdff`).

- **Software**:
    - MicroPython instalado no ESP32. [Parte 4 do tutorial](4%20-%20ESP32%20-%20Primeiros%20Passos%20-%20Setup%20do%20Ambiente%20de%20Desenvolvimento.md)
    - Clone a biblioteca e salve no ESP32:
        `git clone https://github.com/seu-usuario/dsp-crossover-project`

---

## **Visão Geral do Programa**

###  Componentes Principais

- **Event Bus**: Coordena comunicação entre componentes (ex: clique do encoder → atualização do LCD).
- **Rotary Encoder**: Navega no menu e ajusta valores (giro para direita/esquerda, clique para selecionar).
- **Back Button**: Retorna ao menu anterior ou cancela ações.
- **LCD**: Exibe menus e status
- **CrossoverService**: Calcula coeficientes de filtro (passa-baixa/alta) e configura o DSP. O service também salva o status dos filtros na memória não volátil, ou seja mesmo depois de desligado o sistema armazena os filtros setados

<img src="../Images/Pasted image 20250130203043.png" width=600px />
<img src="../Images/Pasted%20image%2020250130203108.png" width=600px />

## **Expandindo o Projeto**

- **Novos Filtros**: Adicione filtros passa-banda ou shelving em `features/crossover/service.py`.
- **Controle Remoto**: Integre Wi-Fi/Bluetooth para ajustes via app (use o Event Bus para enviar comandos).
- Novos Processamentos: Implemente um compressor, um reverb, um oscilador ou qualquer outro processamento de audio que você imaginar!

Para dúvidas, consulte o esquemático ou abra uma _issue_ no repositório! 🎛️🔊