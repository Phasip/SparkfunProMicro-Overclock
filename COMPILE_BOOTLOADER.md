## About
This is a small brain-dump on how to compile the bootloader for a new crystal oscillator.
Assumes 20Mhz but applicable for other frequencies too.

## Howto
These are required to compile the bootloader:
```
sudo apt install avr-gcc avr-libc
```

Install pro-micro sparkfun support to arduino (downloads the ./arduino15/packages/SparkFun for you, which is used later)

Download LUFA-111009 from  www.fourwalledcubicle.com/LUFA.php (Newer version is unfortunately not a drop-in replacement)
Copy .arduino15/packages/SparkFun/hardware/avr/1.1.12/bootloaders/caterina to your working folder
Extract the LUFA to  caterina/LUFA-111009

When you run make you get an error complaining about PLL frequency.
Open the erroring .h file and change the ifdef from 8Mhz to 20Mhz 

Also, now that we have a non-standard frequency we cannot use the clock to generate the USB frequency directly.
Instead we need to revert to low speed USB from the internal clock,
( See 6.11.6 PLL Frequency Control Register – PLLFRQ of ATMega32u4 datasheet)

So, add `| (1 << PINMUX)` to the PLLFRQ calculation in USBController_AVR8.c of LUFA

Now, compile the bootloader
```
make PID=0x9205 F_CPU=20000000
cp Caterina.hex Caterina-promicro20.hex
```
