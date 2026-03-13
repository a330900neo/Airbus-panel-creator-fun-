# Advanced Python Cockpit Panel Editor

A powerful, single-file, browser-based application for designing, editing, and simulating interactive aircraft cockpit panels. It features a drag-and-drop interface, mobile-friendly touch controls, and a built-in Python scripting engine (powered by Brython) to create custom logic for your buttons and indicators.

## Features

* **Visual Editor**: Drag and drop buttons and lines onto an infinite, pannable, and zoomable canvas.
* **Customizable Elements**: Edit button names, labels (top, bottom, left, right), main/secondary text, and LED colors (Blue, White, Green, Amber, Red).
* **Grid System**: Snap-to-grid functionality for perfect alignment.
* **Play & Edit Modes**: Seamlessly switch between building your layout and testing its interactivity.
* **Python Scripting Engine**: Write Python code directly in the browser to control button logic, light states, and timers.
* **Cross-Platform**: Works on desktop and mobile browsers with native touch and gesture support.
* **Import/Export**: Save your layouts and scripts as JSON files and load them later.

## Getting Started

1. **No Installation Required**: This is a purely client-side web application.
2. **Open the App**: Simply double-click the `cockpit_editor.html` file to open it in any modern web browser (Chrome, Firefox, Safari, Edge).
3. *Note: An active internet connection is required the first time you load it, as it fetches the Brython engine and the OpenBus font via CDN.*

## Controls & Navigation

### Edit Mode
* **Pan Canvas**: Click and drag on any empty space (or touch and drag on mobile).
* **Zoom**: Use the mouse scroll wheel.
* **Select/Move Element**: Click and drag a button or line.
* **Edit Properties**: Click an element to open its settings in the right-hand Properties panel.
* **Line Handles**: Click a line to reveal its endpoints, then drag the endpoints to resize/rotate.
* **Delete**: Select an element and press the `Delete` key, or use the red "Delete Selected" button.

### Play Mode
* **Interact**: Click or tap buttons to toggle their physical state (True/False).
* **Pan**: Click/touch and drag anywhere to move around the panel.
* **Hide UI**: Press `ESC` or click the red "Hide UI" button in the bottom right for a clean view.
* **Fullscreen**: Click the "Fullscreen" button for an immersive experience.

## Python Scripting Guide

The editor includes a global Python script panel where you can define the logic for your entire cockpit. The engine runs at 20 ticks per second (20Hz).

### Core Functions

You must define two main functions in your script:
* `def setup():` Runs once when you enter Play Mode. Use it to initialize variables.
* `def update():` Runs continuously in a loop. Put your button logic here.

### API Reference

The following functions are injected into your Python environment:

* `getState(button_id, type)`: Returns a boolean (True/False).
  * `type 0`: The physical button state (pressed/not pressed).
  * `type 1`: The main light state (bottom text).
  * `type 2`: The secondary light state (top text).
* `light(state, type, button_id)`: Sets a light on or off.
  * `state`: `True` (ON) or `False` (OFF).
  * `type 1`: Controls the main light.
  * `type 2`: Controls the secondary light.
* `timeState(button_id, type)`: Returns a float representing how many seconds the element has been in its *current* state.
  * `type 0`: Time since the physical button was last clicked.
  * `type 1`: Time since the main light changed.
  * `type 2`: Time since the secondary light changed.
* `disableButton(button_id, disabled)`: Prevents the user from clicking the button.
  * `disabled`: `True` (cannot be clicked) or `False` (can be clicked).

### Scripting Example & Global Variables

Because `setup()` and `update()` are separate functions, you **must use the `global` keyword** to share custom variables between them.

```python
# 1. Define global variables at the top
APU_STARTING = False
SYSTEM_VOLTAGE = 0

def setup():
    # 2. Declare global inside setup if you are modifying it
    global APU_STARTING, SYSTEM_VOLTAGE
    APU_STARTING = False
    SYSTEM_VOLTAGE = 28

def update():
    # 3. Declare global inside update to read/modify it
    global APU_STARTING, SYSTEM_VOLTAGE
    
    # Example 1: Simple physical button controls its own light
    if getState("B1", 0):
        light(True, 1, "B1")
    else:
        light(False, 1, "B1")
        
    # Example 2: Timer Logic (Amber fault light if B2 is pressed for > 3 seconds)
    if getState("B2", 0) and timeState("B2", 0) > 3.0:
        light(True, 2, "B2") # Turn on secondary (Fault) light
    else:
        light(False, 2, "B2")
