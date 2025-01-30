
## **Instalação e Configuração**

### **1. Pré-requisitos**

- **Hardware**:
    - Todo hardware usado está descrito na parte 1 do projeto, pode voltar lá para conferir!
    - Conexões conforme esquemático (`docs/schematics.pdf`).

- **Software**:
    - MicroPython instalado no ESP32. (Parte 4 do tutorial)
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


![[Pasted image 20250130203043.png]]

![[Pasted image 20250130203108.png]]
## **Expandindo o Projeto**

- **Novos Filtros**: Adicione filtros passa-banda ou shelving em `features/crossover/service.py`.
- **Controle Remoto**: Integre Wi-Fi/Bluetooth para ajustes via app (use o Event Bus para enviar comandos).
- Novos Processamentos: Implemente um compressor, um reverb, um oscilador ou qualquer outro processamento de audio que você imaginar!

Para dúvidas, consulte o esquemático ou abra uma _issue_ no repositório! 🎛️🔊