# Makefile

# Default platform (HOST)
PLATFORM ?= HOST

# Compiler and flags for HOST (native compiler, gcc)
ifeq ($(PLATFORM), HOST)
    CC = gcc
    CFLAGS = -Wall -Werror -g -O0 -std=c99 -DHOST
    LDFLAGS =
    ARCH_FLAGS =
    LD_SCRIPT =
endif

# Compiler and flags for MSP432 (cross compiler, arm-none-eabi-gcc)
ifeq ($(PLATFORM), MSP432)
    CC = arm-none-eabi-gcc
    CFLAGS = -Wall -Werror -g -O0 -std=c99 -DMSP432 -DHOST
    LDFLAGS = -T msp432p401r.lds
    ARCH_FLAGS = -mcpu=cortex-m4 -mthumb -march=armv7e-m -mfloat-abi=hard -mfpu=fpv4-sp-d16 --specs=nosys.specs
    LD_SCRIPT = -T msp432p401r.lds
endif

# Source files and output file
SRC_DIR = src
SRC_FILES_HOST = $(SRC_DIR)/main.c $(SRC_DIR)/memory.c $(SRC_DIR)/other.c
SRC_FILES_MSP432 = $(SRC_DIR)/main.c $(SRC_DIR)/memory.c $(SRC_DIR)/msp432_specific.c
OBJ_FILES = $(SRC_FILES:$(SRC_DIR)/%.c=%.o)
OUTPUT = c1m2.out
MAP_FILE = c1m2.map

DEP_FILES = $(SRC_FILES:$(SRC_DIR)/%.c=%.d)

# Rules
all: $(OUTPUT)

$(OUTPUT): $(OBJ_FILES)
	$(CC) $(ARCH_FLAGS) $(LDFLAGS) -o $(OUTPUT) $^ -Wl,-Map=$(MAP_FILE)

# Generate dependency files
%.d: $(SRC_DIR)/%.c
	$(CC) $(CFLAGS) -MMD -MP -MF $@ $<

-include $(DEP_FILES)

# Compile source files
%.o: $(SRC_DIR)/%.c
	$(CC) $(CFLAGS) $(ARCH_FLAGS) -c $< -o $@

clean:
	rm -f $(OUTPUT) $(OBJ_FILES) $(DEP_FILES) $(MAP_FILE)

.PHONY: all clean

# Platform-specific source and include files
ifeq ($(PLATFORM), HOST)
    SRC_FILES = $(SRC_FILES_HOST)
else ifeq ($(PLATFORM), MSP432)
    SRC_FILES = $(SRC_FILES_MSP432)
endif
#   J H C o u r s e r a M 4  
 