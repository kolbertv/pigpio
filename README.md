# pigpio

Fast GPIO, PWM, and servo control on the Raspberry Pi with Node.js.

## Features

 * Digital IO
   * Up to 1.3 million digital writes per second
   * Up to 1.2 million digital reads per second
 * PWM on any of GPIOs 0 through 31
   * Multiple frequencies and duty cycle ranges supported
 * Servo control on any of GPIOs 0 through 31
 * Low latency interrupt handlers
   * Handle up to 8500 interrupts per second
 * Alerts when any of GPIOs 0 through 31 change state
   * Alerts receive the time of the state change accurate to a few microseconds
 * Read or write up to 32 GPIOs as one operation with banked GPIO
 * Trigger pulse generation
 * Pull up/down resistor configuration

## Installation

#### Step 1

The pigpio package is based on the
[pigpio C library](https://github.com/joan2937/pigpio) so the C library needs
to be installed first. Version V41 or higher of the pigpio C library is
required. It can be installed with the following commands:

```
wget abyz.co.uk/rpi/pigpio/pigpio.zip
unzip pigpio.zip
cd PIGPIO
make
sudo make install
```

The `make` command may take a while to complete so please be patient.

Installing the pigpio C library will also install a number of utilities. One of
these utilities is pigpiod which launches the pigpio library as a daemon. This
utility should not be used as the pigpio Node.js module uses the C library
directly.

#### Step 2

```
npm install pigpio
```

If you're using io.js v3 or Node.js v4 or higher and seeing lots of compile
errors when installing pigpio, it's very likely that gcc/g++ 4.8 or higher are
not installed. See
[Node.js v4 and native addons](https://github.com/fivdi/onoff/wiki/Node.js-v4-and-native-addons)
for details.

If you're using Node.js v0.10.29 on the Raspberry Pi and seeing a compile
error saying that `‘REPLACE_INVALID_UTF8’ is not a member of ‘v8::String’`
see [Node.js v0.10.29 and native addons on the Raspberry Pi](https://github.com/fivdi/onoff/wiki/Node.js-v0.10.29-and-native-addons-on-the-Raspberry-Pi).

## Usage

Assume there's an LED connected to GPIO17 (pin 11) and a momentary push button
connected to GPIO4 (pin 7).

<img src="https://raw.githubusercontent.com/fivdi/pigpio/master/example/pigpio.png">

#### PWM

Use PWM to pulse the LED connected to GPIO17 from fully off to fully on
continuously.

```js
var Gpio = require('pigpio').Gpio,
  led = new Gpio(17, {mode: Gpio.OUTPUT}),
  dutyCycle = 0;

setInterval(function () {
  led.pwmWrite(dutyCycle);

  dutyCycle += 5;
  if (dutyCycle > 255) {
    dutyCycle = 0;
  }
}, 20);

```

#### Interrupt Handling

Turn the LED connected to GPIO17 on when the momentary push button connected to
GPIO4 is pressed. Turn the LED off when the button is released.

```js
var Gpio = require('pigpio').Gpio,
  button = new Gpio(4, {
    mode: Gpio.INPUT,
    pullUpDown: Gpio.PUD_DOWN,
    edge: Gpio.EITHER_EDGE
  }),
  led = new Gpio(17, {mode: Gpio.OUTPUT});

button.on('interrupt', function (level) {
  led.digitalWrite(level);
});
```

### Servo Control

Continuously move a servo connected to GPIO10 clockwise and anti-clockwise.

Simple servo control.

```js
var Gpio = require('pigpio').Gpio,
  motor = new Gpio(10, {mode: Gpio.OUTPUT}),
  pulseWidth = 1000,
  increment = 100;

setInterval(function () {
  motor.servoWrite(pulseWidth);

  pulseWidth += increment;
  if (pulseWidth >= 2000) {
    increment = -100;
  } else if (pulseWidth <= 1000) {
    increment = 100;
  }
}, 1000);
```

#### Alerts

Alerts receive the time of a GPIO state change accurate to a few microseconds.
Typically, alerts will be used for GPIO inputs but they can also be used for
outputs. In this example, the `trigger` method is used generate a 15
microsecond pulse on GPIO17 once per second. Alerts are used to measure the
length of the pulse.

```js
// Assumption: the LED is off when the program is started

var Gpio = require('pigpio').Gpio,
  led = new Gpio(17, {
    mode: Gpio.OUTPUT,
    alert: true
  });

(function () {
  var startTick;

  // Use alerts to determine how long the LED was turned on
  led.on('alert', function (level, tick) {
    var endTick,
      diff;

    if (level == 1) {
      startTick = tick;
    } else {
      endTick = tick;
      diff = (endTick >> 0) - (startTick >> 0); // Unsigned 32 bit arithmetic
      console.log(diff);
    }
  });
}());

// Turn the LED on for 15 microseconds once per second
setInterval(function () {
  led.trigger(15, 1);
}, 1000);
```

## Performance

Three of the pigpio tests are used to monitor performance:

 * digital-read-performance.js - determine max. no. of digitalRead ops per second
 * digital-write-performance.js - determine max. no. of digitalWrite ops per second
 * isr-performance.js - determine max. no. of interrupts per second

The average of ten runs of these tests are shown in the table below.

 Name | Description
:--- | :---
Pi Model | Raspberry Pi 2 Model B V1.1
OS | Raspbian Jessie 2016-02-09
Kernel | 4.1.17-v7+
Node.js | v5.6.0 armv7l
pigpio | v0.3.0
pigpio C library | V45
Reads per second | 1,232,588
Writes per second | 1,323,039
Interrupts per second | 8,881

## API Documentation

- [Configuration](https://github.com/fivdi/pigpio/blob/master/doc/configuration.md) - pigpio configuration

### Classes

- [Gpio](https://github.com/fivdi/pigpio/blob/master/doc/gpio.md) - General Purpose Input Output
- [GpioBank](https://github.com/fivdi/pigpio/blob/master/doc/gpiobank.md) - Banked General Purpose Input Output

