# JTAG test on 32u4

## enable JTAG 

[fuse settings](http://www.engbedded.com/fusecalc/):

using avrdragon with ICSP cable attached to leonardo board's ICSP header.

    avrdude -c dragon_isp -P usb -p atmega32u4 -v -U hfuse:w:0x98:m

    avrdude -c dragon_isp -P usb -p atmega32u4 -v -U hfuse:w:0x18:m # includes OCDEN

OCDEN bit is the on chip debugging enable

    avrdude -c dragon_isp -P usb -p atmega32u4 -v -U hfuse:w:0x19:m # bootrst vector

not sure if this actually made a difference, but this was the fuse used to do
the final debugging.

## make cable that connects the JTAG pins:

### dragon pins

    1 TCK   2 0V
    3 TDO   4 +v
    5 TMS   6 nRST
    7 n/c   8 n/c
    9 TDI   10 0v

### 32u4 pins

    pin fun arduino pin
    -------------------
    PF4 TCK analog3
    PF5 TMS analog2
    PF6 TDO analog1
    PF7 TDI analog0

## read fuses with jtag

    avrdude -c dragon_jtag -p atmega32u4 -v


## compile firmware with debugging

edit ./hardware/keyboardio/avr/libraries/Kaleidoscope-Plugin/tools//kaleidoscope-builder, add -ggdb3 to list of compiler options

new version of builder doesn't include prefs line, add something like this:

    -prefs "compiler.cpp.extra_flags=${ARDUINO_CFLAGS} ${LOCAL_CFLAGS} -ggdb3" \

Then flash it

    avrdude -c dragon_jtag -P usb -p atmega32u4 -v -U flash:w:./output/Model01-Firmware/Model01-Firmware-latest.hex

## connect avarice

This was the one that worked most reliably:

    avarice -g  :4242

This was causing issues after a while

    avarice -g -P atmega32u4 --program --file output/Model01-Firmware/Model01-Firmware-latest.hex :4242 -r -R

## start gdb

    avr-gdb ./output/Model01-Firmware/Model01-Firmware-latest.elf
    target remote localhost:4242
    continue


with 'set history save', can search back through history to do the connecting

then proceed as normal with gdb commands.

## add a breakpoint

    b /home/matt/work/shortcut/raise-v3/firmwares/Kaleidoscope-test/hardware/keyboardio/avr/libraries/KeyboardioScanner/KeyboardioScanner.cpp:173

## bugs

often doesn't work - mostly fixed with the simpler avarice connection (and
possibly fuse change - not validated yet).

* try unplugging/replugging
* try reflash, with avrdude not avarice (verify flag on avarice always fails)
* try this avarice: avarice -g -B 2MHz  --capture :4242
* make sure is running before attaching avraice and remove -R reset flag
* after connect with gdb, try continue and check program is still running

Also

* confusing with etags taking me to other definitions of functions but not the
 ones actually used in the compliation.

