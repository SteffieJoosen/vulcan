-include ../../Makefile.config

SOURCES             = $(shell ls *.c ../common/*.c)
APP                 = main
CAN_AUTH_DIR        = ../../can-auth
FPGA_DEV            = /dev/ttyUSB0

ifdef CAN_DRV_SM
    SOURCES += ../../drivers/sm_mmio_spi.c
endif

OBJECTS             = $(SOURCES:.c=.o) $(CAN_AUTH_LIB).o
TARGET              = $(APP).elf
TARGET_NO_MAC       = no_mac_$(TARGET)
TARGET_SIM          = $(APP)_sim.elf
TARGET_SIM_NO_MAC   = no_mac_$(TARGET_SIM)

all: $(TARGET) $(TARGET_SIM)

#custom targets since intra-SM libraries cannot be shared
$(CAN_AUTH_LIB).o: $(CAN_AUTH_DIR)/$(CAN_AUTH_LIB).c
	$(CC) $(CFLAGS) -c $< -o $@

can-drv.o: ../../drivers/mcp2515.c
	$(CC) $(CFLAGS) -c $< -o $@

can-sim.o: ../../drivers/ican_sim.c
	$(CC) $(CFLAGS) -c $< -o $@

$(TARGET_NO_MAC): $(OBJECTS) can-drv.o
	$(LD) $(LDFLAGS) -o $@ $^

$(TARGET_SIM_NO_MAC): $(OBJECTS) can-sim.o
	$(LD) $(LDFLAGS) -o $@ $^

%.elf: no_mac_%.elf
	$(SANCUS_CRYPTO) $(CRYPTOFLAGS) -o $@ $<

load: $(TARGET)
	$(SANCUS_LOAD) $(LOADFLAGS) $<

sim: $(TARGET_SIM)
	$(SANCUS_SIM) $(SIMFLAGS) --fileio-in ../can-ping.bin --fileio-out ../can-pong.bin $<

clean:
	$(RM) *.elf *.o *.bin *.fst $(OBJECTS)