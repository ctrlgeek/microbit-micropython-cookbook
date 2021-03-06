# micro:bit MicroPython Cookbook (Updating)

![1](https://user-images.githubusercontent.com/44191076/79871966-c0ae8b00-8417-11ea-8255-cbc681d12b8d.jpg)

See also [BBC micro:bit MicroPython documentation](https://microbit-micropython.readthedocs.io/en/latest/index.html#)

This is the collection of my notes, tricks and experiments on BBC micro:bit and MicroPython.

## Easer Eggs

Enter the following codes in [REPL](https://microbit-micropython.readthedocs.io/en/latest/devguide/repl.html):

```python
import this
import love
import antigravity
```

The result from <b>import this</b> is a version of [Zen of Python](https://www.python.org/dev/peps/pep-0020/) and <b>import antigravity</b> is from [original Python easter egg](https://xkcd.com/353/).

Also you can try (also in REPL)

```python
this.authors()
love.badaboom()
```

## A Little Help

Display all modules in REPL:

```python
help('modules')
```

And in REPL you can import a module and check out what it is and what's in there:

```python
import microbit
help(microbit)
dir(microbit)
```

## Some Lesser Known Facts

Since MicroPython, like standard Python, is a interpreted language, it eats a lot of memory. A slightly bigger program may cause MemoryError (out of memory). Also due to this reason, there is no room for the Bluetooth driver.

Another limit is that the MicroPython hex file are consisted of 2 parts: the MicroPython firmware (up to 248 KB) and user's script (up to only 8 KB). See [Firmware Hex File](https://microbit-micropython.readthedocs.io/en/latest/devguide/hexformat.html). Which means it's less likely to build bigger projects with micro:bit's MicroPython.

One way to "minimize" your script size is to use 1-space indents instead of 4.

micro:bit's MicroPython is based on Python 3.4. Which means many built-in Python advanced feaetures are supported. Although some features are dropped as well to save memory.

Nicholas H. Tollervey - also the creator of [Mu editor](https://codewith.mu/) - has an interesting account about [How BBC micro:bit get its MicroPython](http://ntoll.org/article/story-micropython-on-microbit).

## Editor of Choice

The official [Python online editor](https://python.microbit.org/v/2.0) does not need installation and can be used anywhere with Internet and Chrome web browser. Support Web-USB. It's ok to use, really.

Personally, I would perfer [Mu editor](https://codewith.mu/) for any beginners. It has code check, (limited) auto-complete and can automatically detect/upload code to your micro:bit.

If you have experiences in standard Python or MicroPython with ESP8266/ESP32 (or CircuitPython), you can consider [Thonny](https://thonny.org/) which allows you to access micro:bit's REPL directly without having to switch tabs.

The Python online editor and Thonny allow you to upload your .py file onto your micro:bit. So it may still be possible to write a file bigger than 8KB. You can upload third party drivers by this way, although the micro:bit may not have enough RAM to run them all.

## Why You Should Avoid Import *

The following import statement

```python
from microbit import *
```

is not really a good idea. This imports everything from the big **microbit** module even you don't need many of the features and thus waste memory.

Instead, you should import the only submodules you are going to use:

```python
from microbit import pin0, display, sleep
```

## How Much Memory Left?

```python
from micropython import mem_info

print(mem_info(1))
```

You can also try to turn on garbage collection if the memory is almost full:

```python
import gc

gc.enable() # auto memory recycle
gc.collect() # force memory recycle
```

## Recursion is Not Welcomed

Recursion depth (how many level can a function calls itself) is severely limited because of the memory constraint. Only [8 nested function calls or so](https://mail.python.org/pipermail/microbit/2016-February/000896.html) can be used without triggering RuntimeError. So sadly it's not possible to write some popular algorithms on micro:bit.

## Classic Blinky

```python
from microbit import display, Image, sleep

while True:
    display.show(Image.HEART)
    sleep(1000)
    display.clear()
    sleep(1000)
```

## Roll a Dice

You might need to shake it harder to see changes. The gesture detection is not idel in micro:bit's MicroPython.

```python
from microbit import display, Image, accelerometer, sleep
from random import randint

dices = {
    1: '00000:00000:00900:00000:00000',
    2: '00900:00000:00000:00000:00900',
    3: '90000:00000:00900:00000:00009',
    4: '90009:00000:00000:00000:90009',
    5: '90009:00000:00900:00000:90009',
    6: '90009:00000:90009:00000:90009',
}

while True:
    
    if accelerometer.is_gesture('shake'):
        dice = randint(1, 6)
        display.show(Image(dices[dice]))
        sleep(100)
```

## Fill LED Display

Light up every LEDs. You can use fillScreen() as default.

```python
from microbit import display, Image, sleep

def fillScreen(b = 9):
    f = (str(b) * 5 + ':') * 5
    display.show(Image(f[:len(f)-1]))


while True:
    
    for _ in range(2):
        fillScreen()
        sleep(100)
        display.clear()
        sleep(100)
    
    for i in range(9):
        fillScreen(i)
        sleep(50)
        
    for i in reversed(range(9)):
        fillScreen(i)
        sleep(50)
```

## LED Bar Graph

A 25-level LED progress bar.

```python
from microbit import display, sleep

def plotBarGraph(value, maxValue, brightness=9):
    bar = value / maxValue
    valueArray = ((0.96, 0.88, 0.84, 0.92, 1.00), 
                  (0.76, 0.68, 0.64, 0.72, 0.80),
                  (0.56, 0.48, 0.44, 0.52, 0.60), 
                  (0.36, 0.28, 0.24, 0.32, 0.40), 
                  (0.16, 0.08, 0.04, 0.12, 0.20))
    for y in range(5):
        for x in range(5):
            display.set_pixel(x, y, 
                brightness if bar >= valueArray[y][x] else 0)


while True:
    lightLevel = display.read_light_level()
    plotBarGraph(lightLevel, 255) # or plotBarGraph(lightLevel, 255, 9)
    sleep(50)
```

Since read_light_level() uses LEDs themselves as light sensors (see [this video](https://www.youtube.com/watch?v=TKhCr-dQMBY)), The LED screen would flicker a bit.

## Blinky LEDs Without Using Sleep

The two LEDs would blink at different intervals.

```python
from microbit import display
import utime

delay1, delay2 = 1000, 300
since1, since2 = utime.ticks_ms(), utime.ticks_ms()

while True:
    
    now = utime.ticks_ms()
    
    if utime.ticks_diff(now, since1) >= delay1:
        display.set_pixel(0, 0, 9 if display.get_pixel(0, 0) == 0 else 0)
        since1 = utime.ticks_ms()
    
    if utime.ticks_diff(now, since2) >= delay2:
        display.set_pixel(4, 4, 9 if display.get_pixel(4, 4) == 0 else 0)
        since2 = utime.ticks_ms()
```

## A More Convenient Pin Class?

Make a Pin class to "rename" existing pin methods.

```python
from microbit import pin0, pin2, sleep

class Pin:
    
    __slots__ = ['pin']
    
    def __init__(self, pin):
        self.pin = pin
    
    def set(self, value):
        self.pin.write_digital(value)
    
    def setPWM(self, value):
        self.pin.write_analog(value)
    
    def get(self):
        self.pin.set_pull(self.pin.PULL_DOWN)
        return self.pin.read_digital()
    
    def pressed(self):
        self.pin.set_pull(self.pin.PULL_UP)
        return not self.pin.read_digital()
        
    def getADC(self):
        return self.pin.read_analog()


led = Pin(pin0)
button = Pin(pin2)

while True:
    led.set(button.pressed())
    sleep(50)
```

## Simpler Alternate Pin Class

Use **namedtuple** as a simple Pin class. Might save more memory than regular class.

```python
from microbit import pin0, pin2, sleep
from ucollections import namedtuple

Pin = namedtuple('Pin', ['set', 'get'])

def setPin(pin, pull_up=False):
    pin.set_pull(pin.PULL_UP if pull_up else pin.PULL_DOWN)
    return Pin(pin.write_digital, pin.read_digital)


led = setPin(pin0)
button = setPin(pin2, pull_up=True)

while True:
    led.set(not button.get())
    sleep(50)
```

## Value Mapping

Translate a value in a range to its corresponding value in anoher range. Borrowed from [here](https://stackoverflow.com/questions/1969240/mapping-a-range-of-values-to-another).

```python
from microbit import display, sleep

def translate(value, leftMin, leftMax, rightMin, rightMax):
    leftSpan = leftMax - leftMin
    rightSpan = rightMax - rightMin
    valueScaled = float(value - leftMin) / float(leftSpan)
    return rightMin + (valueScaled * rightSpan)


while True:
    lightLevel = display.read_light_level()
    print(translate(lightLevel, 0, 255, 0, 1023))
    sleep(100)
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

Do not use servos and buzzers at the same time. They require different PWM frequencies and most microcontrollers can only set one frequency accross all pins at a time.

Also: micro:bit's power output may just (barely) enough to power a single SG90 mini servo. External power supply is recommended.


## Get Pitch and Roll Degrees

These function cannot tell if the board is facing up or down. Probably need to use accelerometer.get_z() for that.

Go to REPL and reset micro:bit to see the output.

```python
from microbit import accelerometer, sleep
from math import pi, atan2, sqrt

def rotationPitch():
    return atan2(
            accelerometer.get_y(), 
            sqrt(accelerometer.get_x() ** 2 + accelerometer.get_z() ** 2)
            ) * (180 / pi)

def rotationRoll():
    return atan2(
            accelerometer.get_x(), 
            sqrt(accelerometer.get_y() ** 2 + accelerometer.get_z() ** 2)
            ) * (180 / pi)


while True:
    print('Pitch:', rotationPitch(), ' / roll:', rotationRoll())
    sleep(100)
```
## NeoPixel Rainbow/Rotation Effect

This code needs at least 3 LEDs in the NeoPixel chain. Of course, you can set a number (much) higher than actual LEDs to get smooth rainbow effects.

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
    index = (0, int(led_num / 3), int(led_num / 3 * 2))
    for i in range(led_num):
        color = [0, 0, 0]
        for j in range(3):
            if abs(i - index[j]) <= index[1]:
                color[j] = led_maxlevel - abs(i - index[j]) * change_amount
                if color[j] < 0:
                    color[j] = 0
        if i >= index[2]:
            color[0] = led_maxlevel - (led_num - i) * change_amount
            if color[0] < 0:
                color[0] = 0
        np[i] = tuple(color)
    np.show()

def ledRotate():
    tmp = np[led_num - 1]
    for i in reversed(range(1, led_num)): # clockwise
        np[i] = np[i - 1]
    np[0] = tmp
    np.show()


showRainbow()

while True:
    ledRotate()
    sleep(50)
```

## HC-SR04 Ultrasonic Sensor

Get detected distance from HC-SR04/HC-SR04P sonar sensors. Set the parameter unit to 'cm' or 'inch'. External power supply recommended.

```python
from machine import time_pulse_us
from utime import sleep_us
from microbit import pin1, pin2, display, Image, sleep

def sonar(trig_pin, echo_pin, unit='cm'):
    
    trig_pin.write_digital(0)
    sleep_us(2)
    trig_pin.write_digital(1)
    sleep_us(10)
    trig_pin.write_digital(0)
    
    while echo_pin.read_digital() == 0:
        pass
    
    duration = time_pulse_us(echo_pin, 1, 30000)
    
    if unit == 'cm':
        return duration / 2.0 * 0.03313
    elif unit == 'inch':
        return duration / 2.0 * 0.01304
    else:
        return duration


while True:
    
    distance = sonar(trig_pin=pin1, echo_pin=pin2, unit='cm')
    print(distance)
    
    if 2 <= distance <= 20:
        display.show(Image.YES)
    else:
        display.clear()
    
    sleep(100)
```

## Calcualte Fibonacci Sequence

[Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_number)

```python
from microbit import display

Fibonacci_num = 42 # calculate nth number
a = 0
b = 1

for i in range(Fibonacci_num - 2):
    a, b = b, a + b

print(b)
display.scroll(b)
```

## Calcuate a List of Prime Numbers

Prime numbers (except 2, 3) are either 6n - 1 or 6n + 1. So we check if a number of 6n - 1/6n + 1 can be divided with any existing primes in the list. If not, it is a prime number and can be added to the list.

```python
from microbit import display

limit = 50 # calculate primes up to 50
primes = [2, 3]

for p in range(6, limit + 1, 6):
    for p_test in range(p - 1, p + 2, 2):
        p_test_is_prime = True
        for prime in primes:
            if p_test % prime == 0:
                p_test_is_prime = False
                break
        if p_test_is_prime:
            primes.append(p_test)

print(primes)
for prime in primes:
    display.scroll(prime)
```

## Conway's Game of Life on 5x5 LED Display

```python
from microbit import display
from machine import reset
from random import randint

# Rule for B3/S23
# see https://www.conwaylife.com/wiki/List_of_Life-like_cellular_automata
Born = '3'
Sustain = '23'

matrix = [bytearray((1 if randint(0, 2) == 0 else 0) 
            for _ in range(5)) for _ in range(5)]

def display_matrix():
    for i in range(5):
        for j in range(5):
            display.set_pixel(i, j, 9 if matrix[i][j] else 0)

def calculate_next_gen():
    global matrix
    matrix_buf = [bytearray(0 for _ in range(5)) for _ in range(5)]
    for i in range(5):
        for j in range(5):
            cell_num = 0
            for k in range(3):
                for l in range(3):
                    x = i + k - 1
                    y = j + l - 1
                    if x < 0:
                        x = 5 - 1
                    elif x >= 5:
                        x = 0 
                    if y < 0:
                        y = 5 - 1
                    elif y >= 5:
                        y = 0   
                    if matrix[x][y]:
                        cell_num += 1
            if not matrix[i][j]:
                matrix_buf[i][j] = 1 if str(cell_num) in Born else 0
            else:
                cell_num -= 1
                matrix_buf[i][j] = 1 if str(cell_num) in Sustain else 0
    matrix = matrix_buf

generation = 0
cell_count = 0
prev_cell_count = 0
cell_repeat = 0

while True:
    
    calculate_next_gen()
    cell_count = sum(map(sum, matrix))
    print(cell_count, 'cell(s)')
    display_matrix()
    
    if prev_cell_count == cell_count:
        cell_repeat += 1
    else:
        cell_repeat = 0
    prev_cell_count = cell_count
    
    if cell_count == 0 or cell_repeat >= 7:
        print('Resetting...')
        print('')
        reset()
```

The code would reset the micro:bit if there's no cell left or the cells are stable. Although sometimes it may be locked into a state with the same alternating cell numbers and need manual reset.

## Morse Code Machine

This allows you to enter your message and display it as Morse code on the LED screen. Go to the REPL mode and reset micro:bit to make it work.

If you attach a passive buzzer between pin 0 and ground you can hear the Morse code too.

```python
from microbit import display, Image, sleep
from micropython import const
import music

morse_delay = const(50) # morse code speed

morse_code = {
    'A': '.-',
    'B': '-...',
    'C': '-.-.',
    'D': '-..',
    'E': '.',
    'F': '..-.',
    'G': '--.',
    'H': '....',
    'I': '..',
    'J': '.---',
    'K': '-.-',
    'L': '.-..',
    'M': '--',
    'N': '-.',
    'O': '---',
    'P': '.--.',
    'Q': '--.-',
    'R': '.-.',
    'S': '...',
    'T': '-',
    'U': '..-',
    'V': '...-',
    'W': '.--',
    'X': '-..-',
    'Y': '-.--',
    'Z': '--..',
    '1': '.----',
    '2': '..---',
    '3': '...--',
    '4': '....-',
    '5': '.....',
    '6': '-....',
    '7': '--...',
    '8': '---..',
    '9': '----.',
    '0': '-----',
}

def fillScreen():
    f = (str(9) * 5 + ':') * 5
    display.show(Image(f[:len(f)-1]))
    

while True:
    
    print('Enter your message: (alphabets and numbers only)')
    msg_str = input().upper()
    morse_str = ''
    print('Converting message...')
    
    for s in msg_str:
        if s in morse_code:
            for code in morse_code[s]:
                morse_str += code
                music.pitch(440)
                fillScreen()
                sleep(morse_delay * (3 if code == '-' else 1))
                music.pitch(0)
                display.clear()
                sleep(morse_delay)
    
    print('Message converted:')
    print(morse_str)
    print('')
```

## Radio Proximity Sensor

Load the code below to two micro:bits. They will detect each other's radio signal strength and display it as LED bar graph. Can be used as a indoor treasure hunt game.

Due to some reason, the signal strength or RSSI changes very little regardless of transmite power. So I roughly remapped the value to 0-60 so that you can see the changes more clearly.

If there's no signal received the strength data would be set as zero.

```python
from microbit import display, sleep
import radio

def plotBarGraph(value, maxValue, brightness=9):
    bar = value / maxValue
    valueArray = ((0.96, 0.88, 0.84, 0.92, 1.00), 
                  (0.76, 0.68, 0.64, 0.72, 0.80),
                  (0.56, 0.48, 0.44, 0.52, 0.60), 
                  (0.36, 0.28, 0.24, 0.32, 0.40), 
                  (0.16, 0.08, 0.04, 0.12, 0.20))
    for y in range(5):
        for x in range(5):
            display.set_pixel(x, y, 
                brightness if bar >= valueArray[y][x] else 0)


radio.config(group=42, power=7)
radio.on()

while True:
    
    radio.send('0')
    strength = 0.0
    data = radio.receive_full()
    
    if data:
        strength = data[1] + 255 - 155
    
    print('Signal strength:', strength)
    plotBarGraph(strength, 60)
    sleep(50)
```
