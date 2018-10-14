# Overclocking PRO MICRO
## Limitations
Let's start with problems!
The device becomes less reliable, I have experienced issues with bootloader stopping working & requiring reflash for no apparent reason.
USB will be in Low-Speed mode

## Description
This describes how to replace the 16Mhz crystal oscillator of a 5V Pro Micro with a 20Mhz crystal and still have
a working USB port. The bootloader should work for different frequency crystals too.
If you want to compile your own bootloader, see the rough compilation description in COMPILE_BOOTLOADER.md

## Flash a blinker program
To test your new crystal it is nice to have a ready program which allows you to see speed changes.
Flashing should be done before soldering as this allows you to test your crystal without
soldering it. Do this by powering on the device and holding the crystal pins against the solder points.

I used https://learn.sparkfun.com/tutorials/pro-micro--fio-v3-hookup-guide/example-1-blinkies
Note that going from 16Mhz to 20Mhz may be tricky to see with the default timings.

## Prepare new bootloader
The bootloader sets different flags in the microcontroller depending on the clock frequency,
this is used for example to ensure that USB works at the right frequency. Therefore, 
using the wrong bootloader will stop the USB from working.

Use the bootoader in Caterina-promicro20.hex and flash it using avrdude
For built-in USB
```
avrdude -p m32u4 -P usb -c avrispmkii -U flash:w:Caterina-promicro20.hex -U efuse:w:0xcb:m -U hfuse:w:0xd8:m -U lfuse:w:0xff:m
```

This bootloader is configured to use the backup crystal within the ATMega, and shoud work wih any speed of crystal.

##  Desolder old crystal
Desoldering the old can be a bit tricky as the pins are under the crystal. 
I assume using hot-air to desolder is optimal but I used a normal soldering iron,
added extra solder to all corners and heated the corners and crystal until I was able to remove the crystal.
Be careful not to use too much force as you may remove the solder points on the chip.

## Solder a faster crystal
Note that the original crystal was a TTL crystal and that it had four connecting pins. "Normal" crystal oscillators only have two pins, and according to
http://forum.arduino.cc/index.php?topic=280417.0 there may be problems using a normal oscillator for very high frequencies. 
However, for 20Mhz a "Normal" crystal is fine.
The two pins to use are diagonally aligned, with one of them being the pin closest to the "P" of "Pro Micro" marking on the device.
Before soldering, if you flashed the blinker program, you can try pressing your crystal oscillator against the pins and starting the device.

## Add arduino device
To get timings correct, add a new 20Mhz board to your arduino conf - just copy the 16Mhz one in ~/.arduino15/packages/SparkFun/hardware/avr/1.1.12/boards.txt and replace 16 with 20 everywhere (remember the variable name)

Also, compiling at this point will result in errors in Arduino's USBCore.cpp, this can be fixed by adding a new ifdef at the
point of error:
```
#elif F_CPU == 20000000UL
	PLLCSR &= ~(1<<PINDIV);
	PLLFRQ |= (1 << PINMUX);     // Use internal oscillator of 32u4, see 6.11.6 of datasheet
```

## Done
Good job! Have fun with your slightly faster and less reliable Pro-Micro!
