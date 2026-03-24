# EasyButton for MicroPython

A MicroPython port of [EasyButton](https://github.com/evert-arias/EasyButton) — a polling-based button handler with debounce, short press, long press, and multi-press sequence detection.

## Features

- Debounce filtering
- Short press detection (fires on release)
- Long press detection (fires once while held)
- Multi-press sequence detection (e.g. triple-click within a time window)
- State queries: current state, transitions, held/released duration

## Installation

Copy `lib/button.py` to the `lib/` directory on your device (or to the root if you prefer).

## Usage

```python
from lib.button import Button

btn = Button(0)                         # GP0, active-low with internal pull-up (default)

btn.on_pressed(lambda: print("pressed"))
btn.on_pressed_for(2000, lambda: print("held 2s"))
btn.on_sequence(3, 1000, lambda: print("triple click"))

while True:
    btn.read()                          # must be called every loop iteration
```

## Constructor

```python
Button(pin, debounce_ms=35, pull_up=True, active_low=True)
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `pin` | `int` or `machine.Pin` | — | GPIO number or Pin object |
| `debounce_ms` | `int` | `35` | Debounce window in milliseconds |
| `pull_up` | `bool` | `True` | Enable internal pull-up resistor |
| `active_low` | `bool` | `True` | `LOW` = pressed (typical for pull-up wiring) |

## Callbacks

### `on_pressed(callback)`

Fires on release after a short press. Suppressed if a long press was detected during the same hold.

```python
btn.on_pressed(lambda: print("short press"))
```

### `on_pressed_for(duration_ms, callback)`

Fires exactly once when the button is held continuously for at least `duration_ms` milliseconds. Triggers while the button is still held.

```python
btn.on_pressed_for(1500, lambda: print("held for 1.5s"))
```

### `on_sequence(count, duration_ms, callback)`

Fires when the button is pressed `count` times within a `duration_ms` window. Up to 5 sequences can be registered on the same button. A long press resets all sequence counters.

```python
btn.on_sequence(2, 500, lambda: print("double click"))
btn.on_sequence(3, 800, lambda: print("triple click"))
```

## State Queries

These are valid after calling `read()`.

| Method | Returns `True` when… |
|--------|----------------------|
| `is_pressed()` | Button is currently pressed |
| `is_released()` | Button is currently released |
| `was_pressed()` | Button transitioned to pressed on the last `read()` |
| `was_released()` | Button transitioned to released on the last `read()` |
| `pressed_for(ms)` | Currently pressed **and** held for at least `ms` |
| `released_for(ms)` | Currently released **and** has been released for at least `ms` |

```python
while True:
    btn.read()
    if btn.pressed_for(3000):
        print("still held after 3s")
```

## `read()`

Polls the pin, updates internal state, and fires any registered callbacks. Returns `True` if the button is currently pressed. Must be called on every loop iteration.

```python
while True:
    btn.read()
```

## Differences from the Arduino EasyButton Library

| Feature | Arduino EasyButton | This port |
|---------|-------------------|-----------|
| Event model | Polling (`read()`) | Polling (`read()`) — identical |
| Interrupt support | Yes (`enableInterrupt`) | No |
| `pressedFor` callback | Yes | Yes (`on_pressed_for`) |
| Sequence / combo | Yes | Yes (`on_sequence`) |
| `onPressedFor` repeated fire | No (fires once per hold) | No (fires once per hold) — identical |
| Max sequences | 5 | 5 — identical |
| Platform | Arduino (AVR, ESP, …) | MicroPython (RP2, ESP32, …) |
| Pin API | `digitalRead` | `machine.Pin` |

The main functional difference is the **absence of interrupt support**. The Arduino library exposes `enableInterrupt()` / `disableInterrupt()` to attach the `read()` call to a hardware interrupt. This port is polling-only; call `read()` in your main loop (or from a timer if you need near-interrupt responsiveness).

## Example: All Features Together

```python
from lib.button import Button
from time import sleep_ms

def on_short():
    print("Short press")

def on_long():
    print("Long press (held 2s)")

def on_double():
    print("Double click")

def on_triple():
    print("Triple click")

btn = Button(15, debounce_ms=40)
btn.on_pressed(on_short)
btn.on_pressed_for(2000, on_long)
btn.on_sequence(2, 600, on_double)
btn.on_sequence(3, 900, on_triple)

while True:
    btn.read()
    sleep_ms(10)
```

## License

This port follows the same MIT license as the original [EasyButton](https://github.com/evert-arias/EasyButton) library by Evert Arias.
