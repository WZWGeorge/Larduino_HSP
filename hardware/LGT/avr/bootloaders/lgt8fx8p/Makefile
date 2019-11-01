# Makefile for ATmegaBOOT
# E.Lins, 18.7.2005
# $Id$
#
# Instructions
#
# To make bootloader .hex file:
# make diecimila
# make lilypad
# make ng
# etc...
#
# To burn bootloader .hex file:
# make diecimila_isp
# make lilypad_isp
# make ng_isp
# etc...
#
# Edit History
# 201303xx: WestfW: Major Makefile restructuring.
#                   Allows options on Make command line "make xx LED=B3"
#                   (see also pin_defs.h)
#                   Divide into "chip" targets and "board" targets.
#                   Most boards are (recursive) board targets with options.
#                   Move isp target to separate makefile (fixes m8 EFUSE)
#                   Some (many) targets will now be rebuilt when not
#                     strictly necessary, so that options will be included.
#                     (any "make" with options will always compile.)
#                   Set many variables with ?= so they can be overridden
#                   Use arduinoISP settings as default for ISP targets
#

#----------------------------------------------------------------------
#
# program name should not be changed...
PROGRAM    = optiboot

# The default behavior is to build using tools that are in the users
# current path variables, but we can also build using an installed
# Arduino user IDE setup, or the Arduino source tree.
# Uncomment this next lines to build within the arduino environment,
# using the arduino-included avrgcc toolset (mac and pc)
# ENV ?= arduinodev
# OS ?= windows

# export symbols to recursive makes (for ISP)
export

# defaults
EXT_OSC = 0
MCU_TARGET = atmega168
LDSECTIONS  = -Wl,--section-start=.text=0x3e00 -Wl,--section-start=.version=0x3ffe

# Build environments
# Start of some ugly makefile-isms to allow optiboot to be built
# in several different environments.  See the README.TXT file for
# details.

# default
fixpath = $(1)

ifeq ($(ENV), arduino)
# For Arduino, we assume that we're connected to the optiboot directory
# included with the arduino distribution, which means that the full set
# of avr-tools are "right up there" in standard places.
TOOLROOT = /Applications/Arduino.app/Contents/Resources/Java/hardware/tools/avr 
GCCROOT = $(TOOLROOT)/avr/bin/

ifeq ($(OS), windows)
# On windows, SOME of the tool paths will need to have backslashes instead
# of forward slashes (because they use windows cmd.exe for execution instead
# of a unix/mingw shell?)  We also have to ensure that a consistent shell
# is used even if a unix shell is installed (ie as part of WINAVR)
fixpath = $(subst /,\,$1)
SHELL = cmd.exe
endif

else ifeq ($(ENV), arduinodev)
# Arduino IDE source code environment.  Use the unpacked compilers created
# by the build (you'll need to do "ant build" first.)
ifeq ($(OS), macosx)
TOOLROOT = ../../../../build/macosx/work/Arduino.app/Contents/Resources/Java/hardware/tools
endif
ifeq ($(OS), windows)
TOOLROOT = ../../../../build/windows/work/hardware/tools
endif

GCCROOT = $(TOOLROOT)/avr/bin/
AVRDUDE_CONF = -C$(TOOLROOT)/avr/etc/avrdude.conf

else
TOOLROOT =
GCCROOT =
AVRDUDE_CONF =
endif

STK500 = "C:\Program Files\Atmel\AVR Tools\STK500\Stk500.exe"
STK500-1 = $(STK500) -e -d$(MCU_TARGET) -pf -vf -if$(PROGRAM)_$(TARGET).hex \
           -lFF -LFF -f$(HFUSE)$(LFUSE) -EF8 -ms -q -cUSB -I200kHz -s -wt
STK500-2 = $(STK500) -d$(MCU_TARGET) -ms -q -lCF -LCF -cUSB -I200kHz -s -wt
#
# End of build environment code.


OBJ        = $(PROGRAM).o

OPTIMIZE = -O2 -fno-inline-small-functions
#OPTIMIZE = -Os -fno-inline-small-functions -fno-split-wide-types -mshort-calls

DEFS       = 
LIBS       =

CC         = $(GCCROOT)avr-gcc

# Override is only needed by avr-lib build system.

override CFLAGS        = -g -Wall $(OPTIMIZE) -mmcu=$(MCU_TARGET) -DF_CPU=$(AVR_FREQ) $(DEFS)
override LDFLAGS       = $(LDSECTIONS) -Wl,--relax -nostartfiles -nostdlib
#-Wl,--gc-sections

OBJCOPY        = $(GCCROOT)avr-objcopy
OBJDUMP        = $(call fixpath,$(GCCROOT)avr-objdump)

SIZE           = $(GCCROOT)avr-size

#
# Make command-line Options.
# Permit commands like "make atmega328 LED_START_FLASHES=10" to pass the
# appropriate parameters ("-DLED_START_FLASHES=10") to gcc
#

ifdef BAUD_RATE
BAUD_RATE_CMD = -DBAUD_RATE=$(BAUD_RATE)
dummy = FORCE
else
BAUD_RATE_CMD = -DBAUD_RATE=115200
#BAUD_RATE_CMD = -DBAUD_RATE=57600
#BAUD_RATE_CMD = -DBAUD_RATE=38400
#BAUD_RATE_CMD = -DBAUD_RATE=19200
endif

#
# External crystal
#

ifdef EXT_OSC
OSC_CMD = -DEXT_OSC=$(EXT_OSC)
else
OSC_CMD = -DEXT_OSC=0
endif

ifdef LED_START_FLASHES
LED_START_FLASHES_CMD = -DLED_START_FLASHES=$(LED_START_FLASHES)
dummy = FORCE
else
LED_START_FLASHES_CMD = -DLED_START_FLASHES=3
#LED_START_FLASHES_CMD = -DLED_START_FLASHES=0
endif

# BIG_BOOT: Include extra features, up to 1K.
ifdef BIGBOOT
BIGBOOT_CMD = -DBIGBOOT=1
dummy = FORCE
endif

ifdef SOFT_UART
SOFT_UART_CMD = -DSOFT_UART=1
dummy = FORCE
endif

ifdef LED_DATA_FLASH
LED_DATA_FLASH_CMD = -DLED_DATA_FLASH=1
dummy = FORCE
#else
#LED_DATA_FLASH_CMD= -DLED_DATA_FLASH=1
endif

ifdef LED
LED_CMD = -DLED=$(LED)
dummy = FORCE
endif

ifdef SINGLESPEED
SSCMD = -DSINGLESPEED=1
endif

COMMON_OPTIONS = $(BAUD_RATE_CMD) $(LED_START_FLASHES_CMD) $(BIGBOOT_CMD)
COMMON_OPTIONS += $(SOFT_UART_CMD) $(LED_DATA_FLASH_CMD) $(LED_CMD) $(SSCMD)
COMMON_OPTIONS += $(OSC_CMD)

LGT_OPTIONS += -D__LGT8FX8P__ -D__LGT8F__

#UART is handled separately and only passed for devices with more than one.
ifdef UART
UARTCMD = -DUART=$(UART)
endif

ALL: lgt8f328p lgt8f328ps20 lgt8f328p_noee lgt8f328ps20_noee

# LGT8F328P
#
lgt8f328p: BOOT_VECT = 0x7400
lgt8f328p: BOOT_VER_LOC = 0x77fe
lgt8f328p: EXT_OSC ?= 0
lgt8f328p: TARGET = lgt8f328p
lgt8f328p: MCU_TARGET = atmega328p
lgt8f328p: CFLAGS += $(COMMON_OPTIONS)
lgt8f328p: CFLAGS += -DVIRTUAL_BOOT_PARTITION
lgt8f328p: CFLAGS += $(LGT_OPTIONS) -D__LGT8FX8P48__
lgt8f328p: CFLAGS += -DBOOT_VECT=$(BOOT_VECT)
lgt8f328p: AVR_FREQ ?= 16000000L 
lgt8f328p: LDSECTIONS = -Wl,--section-start=.bootv=0x0
lgt8f328p: LDSECTIONS += -Wl,--section-start=.text=$(BOOT_VECT) -Wl,--section-start=.version=$(BOOT_VER_LOC)
lgt8f328p: $(PROGRAM)_lgt8f328p.elf
lgt8f328p: $(PROGRAM)_lgt8f328p.hex
lgt8f328p: $(PROGRAM)_lgt8f328p.lst

lgt8f328p_noee: BOOT_VECT = 0x7c00
lgt8f328p_noee: BOOT_VER_LOC = 0x7ffe
lgt8f328p_noee: EXT_OSC ?= 0
lgt8f328p_noee: TARGET = lgt8f328p
lgt8f328p_noee: MCU_TARGET = atmega328p
lgt8f328p_noee: CFLAGS += $(COMMON_OPTIONS)
lgt8f328p_noee: CFLAGS += -DVIRTUAL_BOOT_PARTITION
lgt8f328p_noee: CFLAGS += $(LGT_OPTIONS) -D__LGT8FX8P48__ -DNOEEPROM
lgt8f328p_noee: CFLAGS += -DBOOT_VECT=$(BOOT_VECT)
lgt8f328p_noee: AVR_FREQ ?= 16000000L 
lgt8f328p_noee: LDSECTIONS = -Wl,--section-start=.bootv=0x0
lgt8f328p_noee: LDSECTIONS += -Wl,--section-start=.text=$(BOOT_VECT) -Wl,--section-start=.version=$(BOOT_VER_LOC)
lgt8f328p_noee: $(PROGRAM)_lgt8f328p_noee.elf
lgt8f328p_noee: $(PROGRAM)_lgt8f328p_noee.hex
lgt8f328p_noee: $(PROGRAM)_lgt8f328p_noee.lst


lgt8f328ps20: BOOT_VECT = 0x7400
lgt8f328ps20: BOOT_VER_LOC = 0x77fe
lgt8f328ps20: EXT_OSC ?= 0
lgt8f328ps20: TARGET = lgt8f328ps20
lgt8f328ps20: MCU_TARGET = atmega328p
lgt8f328ps20: CFLAGS += $(COMMON_OPTIONS)
lgt8f328ps20: CFLAGS += -DVIRTUAL_BOOT_PARTITION
lgt8f328ps20: CFLAGS += $(LGT_OPTIONS) -D__LGT8F_SSOP20__
lgt8f328ps20: CFLAGS += -DBOOT_VECT=$(BOOT_VECT)
lgt8f328ps20: AVR_FREQ ?= 16000000L 
lgt8f328ps20: LDSECTIONS = -Wl,--section-start=.bootv=0x0
lgt8f328ps20: LDSECTIONS += -Wl,--section-start=.text=$(BOOT_VECT) -Wl,--section-start=.version=$(BOOT_VER_LOC)
lgt8f328ps20: $(PROGRAM)_lgt8f328ps20.elf
lgt8f328ps20: $(PROGRAM)_lgt8f328ps20.hex
lgt8f328ps20: $(PROGRAM)_lgt8f328ps20.lst

lgt8f328ps20_noee: BOOT_VECT = 0x7c00
lgt8f328ps20_noee: BOOT_VER_LOC = 0x7ffe
lgt8f328ps20_noee: EXT_OSC ?= 0
lgt8f328ps20_noee: TARGET = lgt8f328ps20
lgt8f328ps20_noee: MCU_TARGET = atmega328p
lgt8f328ps20_noee: CFLAGS += $(COMMON_OPTIONS)
lgt8f328ps20_noee: CFLAGS += -DVIRTUAL_BOOT_PARTITION
lgt8f328ps20_noee: CFLAGS += $(LGT_OPTIONS) -D__LGT8F_SSOP20__ -DNOEEPROM
lgt8f328ps20_noee: CFLAGS += -DBOOT_VECT=$(BOOT_VECT)
lgt8f328ps20_noee: AVR_FREQ ?= 16000000L 
lgt8f328ps20_noee: LDSECTIONS = -Wl,--section-start=.bootv=0x0
lgt8f328ps20_noee: LDSECTIONS += -Wl,--section-start=.text=$(BOOT_VECT) -Wl,--section-start=.version=$(BOOT_VER_LOC)
lgt8f328ps20_noee: $(PROGRAM)_lgt8f328ps20_noee.elf
lgt8f328ps20_noee: $(PROGRAM)_lgt8f328ps20_noee.hex
lgt8f328ps20_noee: $(PROGRAM)_lgt8f328ps20_noee.lst

lgt8f328p_isp: lgt8f328p
lgt8f328p_isp: TARGET = lgt8f328p
lgt8f328p_isp: MCU_TARGET = atmega323p
lgt8f328p_isp: isp


#---------------------------------------------------------------------------
#
# Generic build instructions
#


FORCE:

baudcheck: FORCE
	- @$(CC) $(CFLAGS) -E baudcheck.c -o baudcheck.tmp.sh
	- @bash baudcheck.tmp.sh

isp: $(TARGET)
	$(MAKE) -f Makefile.isp isp TARGET=$(TARGET)

isp-stk500: $(PROGRAM)_$(TARGET).hex
	$(STK500-1)
	$(STK500-2)

upload:
	avrdude -v -v  -p atmega328p -C /etc/avrdude.conf -c stk500v1 -b 115200 -P /dev/ttyACM0 -U flash:w:optiboot_lgt8f328ps20.hex:i


#%.elf: $(OBJ) baudcheck $(dummy)
$(PROGRAM)_%.elf: $(PROGRAM).c
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $< $(LIBS)
	$(SIZE) $@

clean:
	rm -rf *.o *.elf *.lst *.map *.sym *.lss *.eep *.srec *.bin *.hex *.tmp.sh

%.lst: %.elf
	$(OBJDUMP) -h -S $< > $@

%.hex: %.elf
	$(OBJCOPY) -j .bootv -j .text -j .data -j .version --set-section-flags .version=alloc,load --set-section-flags .bootv=alloc,load -O ihex $< $@

%.srec: %.elf
	$(OBJCOPY) -j .text -j .data -j .version --set-section-flags .version=alloc,load -O srec $< $@

%.bin: %.elf
	$(OBJCOPY) -j .text -j .data -j .version --set-section-flags .version=alloc,load -O binary $< $@

.PHONY: ALL lgt8f328p lgt8f328ps20 lgt8f328p_noee lgt8f328ps20_noee

