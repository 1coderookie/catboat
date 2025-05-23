# Additional ATSAM build rules

# Setup the toolchain
CROSS_PREFIX=arm-none-eabi-

dirs-y += src/atsam src/generic lib/fast-hash
dirs-$(CONFIG_MACH_SAM3X) += lib/sam3x/gcc
dirs-$(CONFIG_MACH_SAM4S) += lib/sam4s/gcc
dirs-$(CONFIG_MACH_SAM4E) += lib/sam4e/gcc
dirs-$(CONFIG_MACH_SAME70) += lib/same70b/gcc

MCU := $(shell echo $(CONFIG_MCU) | tr a-z A-Z)

CFLAGS-$(CONFIG_MACH_SAM3X) += -mcpu=cortex-m3 -falign-loops=16
CFLAGS-$(CONFIG_MACH_SAM4) += -mcpu=cortex-m4
CFLAGS-$(CONFIG_MACH_SAME70) += -mcpu=cortex-m7
CFLAGS-$(CONFIG_MACH_SAM3X) += -Ilib/sam3x/include
CFLAGS-$(CONFIG_MACH_SAM4S) += -Ilib/sam4s/include
CFLAGS-$(CONFIG_MACH_SAM4E) += -Ilib/sam4e/include
CFLAGS-$(CONFIG_MACH_SAME70) += -Ilib/same70b/include
CFLAGS += $(CFLAGS-y) -D__$(MCU)__ -mthumb -Ilib/cmsis-core -Ilib/fast-hash

samlink-y := $(OUT)src/generic/armcm_link.ld
samlink-$(CONFIG_MACH_SAME70) := $(OUT)src/atsam/same70_link.ld
CFLAGS_klipper.elf += -nostdlib -lgcc -lc_nano -T $(samlink-y)
$(OUT)klipper.elf: $(samlink-y)

# Add source files
src-y += atsam/main.c atsam/gpio.c atsam/i2c.c atsam/spi.c
src-y += generic/armcm_boot.c generic/armcm_irq.c generic/armcm_timer.c
src-y += generic/crc16_ccitt.c
usb-src-$(CONFIG_MACH_SAM3X) := atsam/sam3_usb.c
usb-src-$(CONFIG_MACH_SAM4) := atsam/sam4_usb.c
usb-src-$(CONFIG_MACH_SAME70) := atsam/sam3_usb.c
src-$(CONFIG_USBSERIAL) += $(usb-src-y) atsam/chipid.c generic/usb_cdc.c
src-$(CONFIG_SERIAL) += atsam/serial.c generic/serial_irq.c
canbus-src-y := generic/canserial.c ../lib/fast-hash/fasthash.c
canbus-src-y += atsam/fdcan.c atsam/chipid.c
src-$(CONFIG_USBCANBUS) += $(canbus-src-y) $(usb-src-y) generic/usb_canbus.c
src-$(CONFIG_CANSERIAL) += $(canbus-src-y) generic/canbus.c
src-$(CONFIG_MACH_SAM3X) += atsam/adc.c atsam/hard_pwm.c
src-$(CONFIG_MACH_SAM4)  += atsam/hard_pwm.c
src-$(CONFIG_MACH_SAM4S) += atsam/adc.c
src-$(CONFIG_MACH_SAM4E) += atsam/sam4e_afec.c atsam/sam4_cache.c
src-$(CONFIG_MACH_SAM3X) += ../lib/sam3x/gcc/system_sam3xa.c
src-$(CONFIG_MACH_SAM4S) += atsam/sam4s_sysinit.c
src-$(CONFIG_MACH_SAM4E) += ../lib/sam4e/gcc/system_sam4e.c
src-$(CONFIG_MACH_SAME70) += atsam/same70_sysinit.c atsam/sam4e_afec.c

# Build the additional bin output file
target-y += $(OUT)klipper.bin

$(OUT)klipper.bin: $(OUT)klipper.elf
	@echo "  Creating bin file $@"
	$(Q)$(OBJCOPY) -O binary $< $@

# Flash rules
lib/bossac/bin/bossac:
	@echo "  Building bossac"
	$(Q)make -C lib/bossac bin/bossac

flash: $(OUT)klipper.bin lib/bossac/bin/bossac
	@echo "  Flashing $< to $(FLASH_DEVICE)"
	$(Q)$(PYTHON) ./scripts/flash_usb.py -t $(CONFIG_MCU) -d "$(FLASH_DEVICE)" $(OUT)klipper.bin
