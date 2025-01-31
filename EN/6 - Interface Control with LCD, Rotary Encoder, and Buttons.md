# Interface Control with LCD, Rotary Encoder, and Buttons

In this tutorial, we will create an interactive system to control a menu on an LCD using a **Rotary Encoder** (for navigation), a **Back Button** (for returning), and an **Event Bus** (for communication between components). The code is compatible with MicroPython and the ESP32.

---

## 1. **Event Bus: The Communication System**

The `EventBus` is the core of the system, responsible for managing asynchronous communication between components. It allows components to emit events (such as "click" or "back") and others to subscribe to these events.

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

### Features:
- **`subscribe(event_type, callback)`**: Allows components to subscribe to events (e.g., "click", "back").
- **`emit(event_type, data)`**: Triggers an event for all subscribers.

---

## 2. **Rotary Encoder: Rotational Navigation**

The `RotaryEncoder` detects rotations (right/left) and clicks on the integrated button, emitting events via `EventBus`.

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

        # Configure interrupt for the encoder button
        self.sw.irq(trigger=self.sw.IRQ_FALLING, handler=self.handle_click)

    def handle_click(self, pin):
        current_time = time.ticks_ms()
        if current_time - self.last_click_time > self.debounce_time:
            self.last_click_time = current_time
            self.event_bus.emit("click")  # Emit click event

    def read(self):
        """Checks the rotation direction and emits 'right' or 'left' events."""
        current_clk = self.clk.value()
        current_dt = self.dt.value()

        if current_clk != self.last_clk:
            if current_dt == current_clk:
                self.event_bus.emit("right")
            else:
                self.event_bus.emit("left")
            self.last_clk = current_clk
```

### Features:
- **Right rotation**: Emits `"right"` event.
- **Left rotation**: Emits `"left"` event.
- **Button click**: Emits `"click"` event with debounce to avoid multiple triggers.

---

## 3. **Back Button: Return Button**

The `BackButton` is a physical button that emits a `"back"` event when pressed.

```python
class BackButton:
    def __init__(self, back_button_pin, event_bus):
        self.back_button = back_button_pin
        self.event_bus = event_bus
        self.last_click_time = 0
        self.debounce_time = 10  # ms

        # Configure interrupt for the button
        self.back_button.irq(trigger=self.back_button.IRQ_FALLING, handler=self.handle_back)

    def handle_back(self, pin):
        current_time = time.ticks_ms()
        if current_time - self.last_click_time > self.debounce_time:
            self.last_click_time = current_time
            self.event_bus.emit("back")  # Emit back event
```

---

## 4. **LCD: Displaying Information**

The LCD (using the `I2cLcd` library) displays messages and menus. Let’s configure it to show an example text:

```python
from machine import I2C, Pin
from time import sleep
from external.lcd.i2c_lcd import I2cLcd

# LCD configuration (adjust pins and I2C address according to your hardware)
LCD_SCL_PIN = 22
LCD_SDA_PIN = 21
I2C_FREQ = 100000
LCD_ADDRESS = 0x27

i2c = I2C(scl=Pin(LCD_SCL_PIN), sda=Pin(LCD_SDA_PIN), freq=I2C_FREQ)
lcd = I2cLcd(i2c, LCD_ADDRESS, 2, 16)  # 2 lines, 16 columns

# Example usage
lcd.clear()
lcd.putstr("System Ready!")
sleep(2)
```

---

## 5. **Complete Integration**

Here’s how to combine all components into a functional system:

```python
from machine import Pin
from time import sleep

# Initial setup
event_bus = EventBus()

# Rotary Encoder (CLK, DT, SW pins)
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

# LCD configuration (using the previous code)

# Functions to handle events
def on_click(data):
    lcd.clear()
    lcd.putstr("Button pressed!")

def on_back(data):
    lcd.clear()
    lcd.putstr("Returning...")

def on_right(data):
    lcd.clear()
    lcd.putstr("Right ->")

def on_left(data):
    lcd.clear()
    lcd.putstr("<- Left")

# Subscribe events to EventBus
event_bus.subscribe("click", on_click)
event_bus.subscribe("back", on_back)
event_bus.subscribe("right", on_right)
event_bus.subscribe("left", on_left)

# Main loop
while True:
    encoder.read()  # Update encoder reading
    sleep(0.1)  # Reduce CPU usage
```

---

## 6. **Explanation of Functionality**

1. **Event Bus**:
   - Centralizes communication between components.
   - When the encoder is rotated or a button is pressed, an event is emitted (e.g., `"right"`, `"back"`).

2. **Rotary Encoder**:
   - Detects rotations and clicks, updating the LCD in real time.
   - Debounce ensures only one event is emitted per physical action.

3. **Back Button**:
   - Similar to the encoder but dedicated to the "back" action.

4. **LCD**:
   - Displays visual feedback based on received events.

---

## 7. **Example Application: Interactive Menu**

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

# Reconfigure events
event_bus.subscribe("right", on_right)
event_bus.subscribe("left", on_left)

update_menu()  # Display initial menu
```

---

## Conclusion

This tutorial demonstrated the practical integration of essential components to create an **interactive physical interface** for embedded systems. Using an ESP32, a **Rotary Encoder** for navigation, a **Back Button** for returning, and an **LCD** for visual feedback, the system is coordinated by an **Event Bus**, which manages asynchronous communication between modules. This decoupled architecture ensures that physical actions (such as rotations or clicks) are translated into clear events ("right", "left", "back"), while the LCD provides real-time feedback, delivering a smooth and responsive user experience even on resource-constrained devices.

The presented solution is highly **scalable and adaptable**, ideal for applications such as audio controllers, home automation panels, or industrial interfaces. The implementation of **debounce** on buttons ensures input accuracy, avoiding interference, while the code’s modularity facilitates the addition of new features—such as dynamic menus, external sensors, or Wi-Fi/Bluetooth connectivity. For example, the Rotary Encoder could adjust parameters in an equalizer, while the LCD displays frequency levels, and the Back Button returns to the main menu.

Finally, the project highlights how accessible tools (MicroPython, low-cost components) and efficient software patterns (such as the Event Bus) can enable robust prototypes and complete embedded systems. The proposed structure serves as a foundation for exploring advanced functionalities, such as configuration storage, remote control, or API integration, making it a versatile starting point for innovations in IoT, automation, and beyond.