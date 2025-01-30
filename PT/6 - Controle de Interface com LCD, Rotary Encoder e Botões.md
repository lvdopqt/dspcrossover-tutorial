
# Controle de Interface com LCD, Rotary Encoder e Botões

Neste tutorial, vamos criar um sistema interativo para controlar um menu em um LCD usando um **Rotary Encoder** (para navegação), um **Back Button** (para voltar) e um **Event Bus** (para comunicação entre componentes). O código é compatível com MicroPython e ESP32.

---

## 1. **Event Bus: O Sistema de Comunicação**

O `EventBus` é o núcleo do sistema, responsável por gerenciar a comunicação assíncrona entre os componentes. Ele permite que os componentes emitam eventos (como "click" ou "back") e outros se inscrevam nesses eventos.

```python
class EventBus:
    def __init__(self):
        self.listeners = {}

    def subscribe(self, event_type, callback):
        if event_type not in self.listeners:
            self.listeners[event_type] = []
        self.listeners[event_type].append(callback)

    def emit(self, event_type, data=None):
        if event_type in self.listeners:
            for callback in self.listeners[event_type]:
                callback(data)
```

### Funcionalidades:
- **`subscribe(event_type, callback)`**: Permite que componentes se inscrevam em eventos (ex: "click", "back").
- **`emit(event_type, data)`**: Dispara um evento para todos os inscritos.

---

## 2. **Rotary Encoder: Navegação Giratória**

O `RotaryEncoder` detecta rotações (direita/esquerda) e cliques no botão integrado, emitindo eventos via `EventBus`.

```python
import time

class RotaryEncoder:
    def __init__(self, clk_pin, dt_pin, sw_pin, event_bus):
        self.clk = clk_pin
        self.dt = dt_pin
        self.sw = sw_pin
        self.event_bus = event_bus
        self.last_clk = self.clk.value()
        self.last_click_time = 0
        self.debounce_time = 10  # ms

        # Configura interrupção para o botão do encoder
        self.sw.irq(trigger=self.sw.IRQ_FALLING, handler=self.handle_click)

    def handle_click(self, pin):
        current_time = time.ticks_ms()
        if current_time - self.last_click_time > self.debounce_time:
            self.last_click_time = current_time
            self.event_bus.emit("click")  # Emite evento de clique

    def read(self):
        """Verifica a direção da rotação e emite eventos 'right' ou 'left'."""
        current_clk = self.clk.value()
        current_dt = self.dt.value()

        if current_clk != self.last_clk:
            if current_dt == current_clk:
                self.event_bus.emit("right")
            else:
                self.event_bus.emit("left")
            self.last_clk = current_clk
```

### Funcionalidades:
- **Rotação para direita**: Emite evento `"right"`.
- **Rotação para esquerda**: Emite evento `"left"`.
- **Clique no botão**: Emite evento `"click"` com debounce para evitar múltiplos acionamentos.

---

## 3. **Back Button: Botão de Retorno**

O `BackButton` é um botão físico que emite um evento `"back"` quando pressionado.

```python
class BackButton:
    def __init__(self, back_button_pin, event_bus):
        self.back_button = back_button_pin
        self.event_bus = event_bus
        self.last_click_time = 0
        self.debounce_time = 10  # ms

        # Configura interrupção para o botão
        self.back_button.irq(trigger=self.back_button.IRQ_FALLING, handler=self.handle_back)

    def handle_back(self, pin):
        current_time = time.ticks_ms()
        if current_time - self.last_click_time > self.debounce_time:
            self.last_click_time = current_time
            self.event_bus.emit("back")  # Emite evento de retorno
```

---

## 4. **LCD: Exibição de Informações**

O LCD (usando a biblioteca `I2cLcd`) exibe mensagens e menus. Vamos configurá-lo para mostrar um texto de exemplo:

```python
from machine import I2C, Pin
from time import sleep
from external.lcd.i2c_lcd import I2cLcd

# Configuração do LCD (ajuste os pinos e endereço I2C conforme seu hardware)
LCD_SCL_PIN = 22
LCD_SDA_PIN = 21
I2C_FREQ = 100000
LCD_ADDRESS = 0x27

i2c = I2C(scl=Pin(LCD_SCL_PIN), sda=Pin(LCD_SDA_PIN), freq=I2C_FREQ)
lcd = I2cLcd(i2c, LCD_ADDRESS, 2, 16)  # 2 linhas, 16 colunas

# Exemplo de uso
lcd.clear()
lcd.putstr("Sistema Pronto!")
sleep(2)
```

---

## 5. **Integração Completa**

Aqui está como combinar todos os componentes em um sistema funcional:

```python
from machine import Pin
from time import sleep

# Configuração inicial
event_bus = EventBus()

# Rotary Encoder (pinos CLK, DT, SW)
encoder = RotaryEncoder(
    clk_pin=Pin(14, Pin.IN, Pin.PULL_UP),
    dt_pin=Pin(12, Pin.IN, Pin.PULL_UP),
    sw_pin=Pin(13, Pin.IN, Pin.PULL_UP),
    event_bus=event_bus
)

# Back Button
back_btn = BackButton(
    back_button_pin=Pin(15, Pin.IN, Pin.PULL_UP),
    event_bus=event_bus
)

# Configuração do LCD (usando o código anterior)

# Funções para manipular eventos
def on_click(data):
    lcd.clear()
    lcd.putstr("Botao pressionado!")

def on_back(data):
    lcd.clear()
    lcd.putstr("Voltando...")

def on_right(data):
    lcd.clear()
    lcd.putstr("Direita ->")

def on_left(data):
    lcd.clear()
    lcd.putstr("<- Esquerda")

# Inscreve os eventos no EventBus
event_bus.subscribe("click", on_click)
event_bus.subscribe("back", on_back)
event_bus.subscribe("right", on_right)
event_bus.subscribe("left", on_left)

# Loop principal
while True:
    encoder.read()  # Atualiza a leitura do encoder
    sleep(0.1)  # Reduz o uso da CPU
```

---

## 6. **Explicação do Funcionamento**

1. **Event Bus**:
   - Centraliza a comunicação entre componentes.
   - Quando o encoder é girado ou um botão é pressionado, um evento é emitido (ex: `"right"`, `"back"`).

2. **Rotary Encoder**:
   - Detecta rotações e cliques, atualizando o LCD em tempo real.
   - Debounce garante que apenas um evento seja emitido por ação física.

3. **Back Button**:
   - Similar ao encoder, mas dedicado para a ação de "voltar".

4. **LCD**:
   - Exibe feedback visual com base nos eventos recebidos.

---

## 7. **Exemplo de Aplicação: Menu Interativo**

```python
menu_items = ["Volume", "Bass", "Treble"]
current_menu = 0

def update_menu():
    lcd.clear()
    lcd.putstr(f"> {menu_items[current_menu]}")

def on_right(data):
    global current_menu
    current_menu = (current_menu + 1) % len(menu_items)
    update_menu()

def on_left(data):
    global current_menu
    current_menu = (current_menu - 1) % len(menu_items)
    update_menu()

# Reconfigura os eventos
event_bus.subscribe("right", on_right)
event_bus.subscribe("left", on_left)

update_menu()  # Exibe o menu inicial
```

---

## Conclusão

Este tutorial demonstrou a integração prática de componentes essenciais para criar uma **interface física interativa** em sistemas embarcados. Utilizando um ESP32, um **Rotary Encoder** para navegação, um **Back Button** para retorno e um **LCD** para feedback visual, o sistema é coordenado por um **Event Bus**, que gerencia a comunicação assíncrona entre os módulos. Essa arquitetura desacoplada permite que ações físicas (como giros ou cliques) sejam traduzidas em eventos claros ("right", "left", "back"), enquanto o LCD exibe informações em tempo real, garantindo uma experiência do usuário fluida e responsiva, mesmo em dispositivos com recursos limitados.

A solução apresentada é altamente **escalável e adaptável**, ideal para aplicações como controladores de áudio, painéis de automação residencial ou interfaces industriais. A implementação de **debounce** nos botões assegura precisão nas entradas, evitando interferências, enquanto a modularidade do código facilita a adição de novos recursos—como menus dinâmicos, sensores externos ou conectividade Wi-Fi/Bluetooth. Por exemplo, o Rotary Encoder pode ajustar parâmetros em um equalizador, enquanto o LCD exibe níveis de frequência, e o Back Button retorna ao menu principal.

Por fim, o projeto reforça como ferramentas acessíveis (MicroPython, componentes de baixo custo) e padrões de software eficientes (como o Event Bus) podem viabilizar protótipos robustos e sistemas embarcados completos. A estrutura proposta serve como base para explorar funcionalidades avançadas, como armazenamento de configurações, controle remoto ou integração com APIs, tornando-se um ponto de partida versátil para inovações em IoT, automação e além.