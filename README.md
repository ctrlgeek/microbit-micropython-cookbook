# micro:bit MicroPython Cookbook - Tricks and Experiments

## Easer Eggs

Enter the following codes into REPL:

```python
import this
import love
import antigravity
```

Also

```python
this.authors()
love.badaboom()
```

## Some Lesser Known Facts

Since Python and MicroPython are interpreted languages, they eat a lot of memory. Also, the hex file generated by micro:bit Python editors are consisted of 2 parts: the MicroPython firmware (up to 248 KB) and user's script (up to only 8 KB). See [Firmware Hex File](https://microbit-micropython.readthedocs.io/en/latest/devguide/hexformat.html). Which means it's not possible to build big projects with micro:bit's MicroPython.

Also, how micro:bit get its own version of MicroPython anyway: [The Story of MicroPython on the BBC micro:bit](http://ntoll.org/article/story-micropython-on-microbit) by Nicholas H. Tollervey, who also created the [Mu editor](https://codewith.mu/), which is easier to use than the official online editor.

## Fill LED Display

```python
from microbit import display, Image, sleep

def fillScreen(b = 9):
    f = (str(b) * 5 + ":") * 5
    display.show(Image(f[:len(f)-1]))


while True:
    
    for i in range(9):
        fillScreen(i)
        sleep(50)
    
    for i in reversed(range(9)):
        fillScreen(i)
        sleep(50)
```

## A More Convenient Pin Class

Using **namedtuple** to "rename" pin methods.

```python
from microbit import pin0, sleep
from ucollections import namedtuple

Pin = namedtuple('Pin', ['set', 'get'])
setPin = lambda p: Pin(p.write_digital, p.read_digital)


led = setPin(pin0)

while True:
    print(led.set(1))
    sleep(500)
    print(led.set(0))
    sleep(500)
```

## LED Bar Graph

Not perfect. A bit slow. Interference with microbit.display.read_light_level().

```python
from microbit import display

def plotBarGraph(value, maxValue, brightness = 9):
    bar = value / maxValue
    valueArray = ((0.96, 0.88, 0.84, 0.92, 1.00), 
                  (0.76, 0.68, 0.64, 0.72, 0.80),
                  (0.56, 0.48, 0.44, 0.52, 0.60), 
                  (0.36, 0.28, 0.24, 0.32, 0.40), 
                  (0.16, 0.08, 0.04, 0.12, 0.20))
    for y in range(5):
        for x in range(5):
            if bar >= valueArray[y][x]:
                display.set_pixel(x, y, brightness)
            else:
                display.set_pixel(x, y, 0)


while True:
    for i in range(255):
        plotBarGraph(i, 255, 9)
```

## Servo Control

```python
from microbit import pin0, sleep

def servoWrite(pin, degree):
    pin.set_analog_period(20)
    pin.write_analog(round((degree * 92 / 180 + 30), 0))


servoPin = pin0

while True:
    servoWrite(servoPin, 0)
    sleep(1000)
    servoWrite(servoPin, 180)
    sleep(1000)
```
## Get Pitch and Roll Degrees

These function cannot tell if the board is facing up or down. Probably need to use accelerometer.get_z() for that.

```python
from microbit import accelerometer, sleep
import math

def rotationPitch():
    return math.atan2(
            accelerometer.get_y(), 
            math.sqrt(accelerometer.get_x() ** 2 + accelerometer.get_z() ** 2)
            ) * (180 / math.pi)

def rotationRoll():
    return math.atan2(
            accelerometer.get_x(), 
            math.sqrt(accelerometer.get_y() ** 2 + accelerometer.get_z() ** 2)
            ) * (180 / math.pi)


while True:
    print("Pitch:", rotationPitch(), " / roll:", rotationRoll())
    sleep(100)
```
## Rainbow NeoPixel

This code needs at least 3 LEDs in the NeoPixel chain.

```python
from microbit import pin0, sleep
from neopixel import NeoPixel
from micropython import const

led_num = const(12)
led_maxlevel = const(64) # max 255
led_pin = pin0

np = NeoPixel(led_pin, led_num)

def showRainbow():
    change_amount = int(led_maxlevel / (led_num / 3))
    index = [0, int(led_num / 3), int(led_num / 3 * 2)]
    for i in range(led_num):
        color = [0, 0, 0]
        for j in range(3):
            if abs(i - index[j]) <= int(led_num / 3):
                color[j] = led_maxlevel - abs(i - index[j]) * change_amount
                if color[j] < 0:
                    color[j]
        if i >= int(led_num / 3 * 2):
            color[0] = led_maxlevel - (led_num - i) * change_amount
            if color[0] < 0:
                color[0] = 0
        np[i] = (color[0], color[1], color[2])
    np.show()

def rotate():
    tmp = np[led_num - 1]
    for i in reversed(range(1, led_num)): # clockwise
        np[i] = np[i - 1]
    np[0] = tmp
    np.show()


showRainbow()

while True:
    rotate()
    sleep(50)

```
