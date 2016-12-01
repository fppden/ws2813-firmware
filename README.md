# WS281x CONTROLLER FIRMWARE

## Supported drivers

* WS2811
* ~~WS2813~~ (*coming soon*)

## Supported MCUs

* Atmel AVR ATtiny13
* Atmel AVR ATtiny25
* Atmel AVR ATtiny45
* Atmel AVR ATtiny85

## Operation modes

Following modes supported (firmware recompile is not needed):

* WS2811 Slow Mode (400 kbps) (set MCU frequency to 8 MHz).
* WS2811 Fast Mode (800 kbps) (set MCU frequency to 16 MHz).

## Commands format

The firmware fetches commands from EEPROM. Each command is a word (16 bits, big-endian in docs, little-endian in EEPROM). Following formats are supported:

# Restart command

This command must be the last one in the program. When it is reached the MCU will restart the program.

```
1|1|1|1|1|1|1|1|1|1|1|1|1|1|1|1 (0xFFFF)
```

# RGB color

5 bits per channel, very low bit is `0`. Each color applies to next LED in the strip (see also REPEAT command below). Use DELAY command (see below) to restart applying colors from first LED.

```
R|R|R|R|R|G|G|G|G|G|B|B|B|B|B|0
```

# DELAY command

Two low bits are `01`, others are delay value (`ms` in fast mode, `ms * 2` in slow mode). Use REPEAT command (see below) if you need longer delay. Each DELAY command restarts applying colors from first LED.

```
x|x|x|x|x|x|x|x|x|x|x|x|x|x|0|1
```

# REPEAT command

Repeats next command `N + 1` times. For example, if repeat value is `1` then next command will be performed twice. Next command will be treated as `RGB color` or `Delay command`. Other commands are not supported after REPEAT.

Low byte is `0x07`, high byte is repeat times.

```
x|x|x|x|x|x|x|x|0|0|0|0|0|1|1|1
```

## Program example

Here is simple program that imitating traffic light (green and blue channels are reversed due to my broken led strip, sorry). HEX, little-endian:

```
02 00 41 1F  00 00 02 20  41 1F 00 00  00 00 00 08  41 1F 00 00  02 20 41 1F  00 00 00 00
00 00 FF FF
```

Comments:

`02 00` - faint green light for first LED.

`41 1F` - 2 seconds delay (fast mode). Restart applying colors from first LED.

`00 00` - turn off first LED.

`02 20` - faint yellow light for second LED.

`41 1F` - 2 seconds delay (fast mode). Restart applying colors from first LED.

`00 00` - turn off first LED.

`00 00` - turn off second LED.

`00 08` - faint red light for third LED.

`41 1F` - 2 seconds delay (fast mode). Restart applying colors from first LED.

`00 00` - turn off first LED.

`02 20` - faint yellow light for second LED.

`00 08` - faint red light for third LED.

`00 00` - turn off first LED.

`00 00` - turn off second LED.

`00 00` - turn off third LED.

`FF FF` - program end, restart.