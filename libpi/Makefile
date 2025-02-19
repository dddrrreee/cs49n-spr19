# what is the right way to compile with FP and without?

include put-your-src-here.mk

OUR_OBJS = \
        cs49n-src/cstart.o  \
        cs49n-src/panic.o  \
        cs49n-src/reboot.o  \
        cs49n-src/put-get.o  \
        cs49n-src/sw-uart.o  \
        cs49n-src/mbox.o  \
        cs49n-src/timer.o \
        cs49n-src/cache.o \
        cs49n-src/clean-reboot.o  \
        cs49n-src/mem-barrier.o  \
        cs49n-libc/branchto.o  \
        cs49n-libc/printk.o  \
        cs49n-libc/snprintk.o  \
        cs49n-libc/putk.o  \
        cs49n-libc/va-printk.o  \
        cs49n-libc/uart-hex.o  \
        cs49n-libc/rpi-rand.o   \
        cs49n-libc/memcpy.o   \
        cs49n-libc/memset.o   \
        cs49n-libc/memcmp.o   \
        cs49n-libc/strcmp.o   \
        cs49n-libc/strcat.o   \
        cs49n-libc/strlen.o   \
        cs49n-libc/strcpy.o   \
        cs49n-libc/strchr.o   \
        cs49n-libc/strncmp.o

START= cs49n-start

# if you get source from other directories, add them to vpath too.
VPATH := .:cs49n-libc/:cs49n-src

ifdef USE_INTERRUPTS
VPATH := $(VPATH):cs49n-interrupt/
OUR_OBJS := $(OUR_OBJS) \
    cs49n-interrupt/gpio-int.o\
    cs49n-interrupt/interrupts-asm.o\
    cs49n-interrupt/uart-int.o\
    cs49n-interrupt/interrupts-c.o
endif

include includes.mk

# make sure that people cannot skip errors.
CFLAGS += -Wno-unused-function -Werror

FP_BUILD_DIR = ./fp-objs
FP_TARGET = libpi-fp.a

ifndef USE_FP
    BUILD_DIR := ./objs
    TARGET = libpi.a
    # these are obj files we give you.
    SUPPORT_OBJS= cs49n-objs/*.o 
    OBJS = $(OUR_OBJS)  $(YOUR_OBJS)
else
    # XXX: don't we need the SUPPORT_OBJS rule?
    #
    OBJS = $(OUR_OBJS)  $(YOUR_OBJS) cs49n-src/fp-support.o
#    CFLAGS += -DRPI_FP_ENABLED  -mhard-float -mfpu=vfp
#    CPP_ASFLAGS += -DRPI_FP_ENABLED  -mhard-float -mfpu=vfp
    BUILD_DIR := ./fp-objs
    TARGET = libpi-fp.a
endif

# IDK if this is the best way: we are trying to compose interrupts with floating point
ifdef USE_INTERRUPTS
BUILD_DIR := $(BUILD_DIR:objs=int-objs)
TARGET := $(TARGET:.a=-int.a)
VPATH := $(VPATH):cs49n-interrupt/
endif

# OBJS := $(SRC:.c=.o) $(CS49n_SRC:.c=.o)
OBJS := $(foreach o, $(OBJS), $(BUILD_DIR)/$(notdir $o))
# Get all depend files by replacing extensions
DEPS := $(OBJS:.o=.d)  $(BUILD_DIR)/$(START).d

	#@echo "TARGET=$(TARGET) and BUILD_DIR=$(BUILD_DIR)\n"
	#@echo "SUPPORT_OBJS=$(SUPPORT_OBJS)\n"
	#@echo "OBJS=$(OBJS)\n"

all: $(TARGET) $(START).o

# we bundle start.s with the library: just make different ones for each
# different start so that the library is internally consistent.
$(TARGET): $(OBJS) $(SUPPORT_OBJS) 
	arm-none-eabi-ar crf $(TARGET) $(OBJS) $(SUPPORT_OBJS)
	$(OD) -D $@ > $@.list

# kind of gross.  we want this .o at the top-level so people can link to it easily.
# so we need a duplicate rule.
$(START).o: $(BUILD_DIR)/$(START).d
	@mv $(BUILD_DIR)/$(START).o .

# XXX: We seem to need this rule to stop compilation with an error if the .c file
# has one: otherwise make does not stop compilation when generating .d.
# If compilation goes correctly, is not triggered.
$(BUILD_DIR)/%.o : %.c
$(BUILD_DIR)/%.o : %.c $(BUILD_DIR)/%.d
	$(COMPILE.c) $(OUTPUT_OPTION) $<

# XXX: I think we need this to catch errors in .S as well.
$(BUILD_DIR)/%.o : %.S
$(BUILD_DIR)/%.o : %.S $(BUILD_DIR)/%.d
	$(CC) $(CPP_ASFLAGS) $<

# Rules to automatically generate dependencies and put in build directory
# We want to recompile everything if the makefile changes.
$(BUILD_DIR)/%.d: %.c Makefile
	@mkdir -p $(BUILD_DIR)
	$(CC) $(CFLAGS) -c -MT $@ -MMD -MP -MF $@ $< -o $(BUILD_DIR)/$(notdir $*).o

$(BUILD_DIR)/%.d: %.S Makefile
	@mkdir -p $(BUILD_DIR)
	$(CC) $(CPP_ASFLAGS) -c -MT $@ -MMD -MP -MF $@ $< -o $(BUILD_DIR)/$(notdir $*).o

# -MF  write the generated dependency rule to a file
# -MG  assume missing headers will be generated and don't stop with an error
# -MM  generate dependency rule for prerequisite, skipping system headers
# -MP  add phony target for each header to prevent errors when header is missing
# -MT  add a target to the generated dependency

# I don't get why we need this.  check the mad-make guy.
#   http://make.mad-scientist.net/papers/advanced-auto-dependency-generation/
$(BUILD_DIR)/%.d: ;
.PRECIOUS: $(BUILD_DIR)/%.d

# *unbelievable*: make clean doesn't skip this include even though the 
# target is after.
#   https://www.gnu.org/software/make/manual/html_node/Goals.html
ifneq ($(MAKECMDGOALS),clean)
-include $(DEPS)
endif

ctags: 
	ctags *.[chSs] */*.[chSs]

clean:
ifndef USE_INTERRUPTS
	make clean USE_INTERRUPTS=1
endif
ifndef USE_FP
	make clean USE_FP=1
endif
	rm -rf $(BUILD_DIR) $(TARGET) *~ $(TARGET).list tags cs49n-start.o $(FP_BUILD_DIR) $(FP_TARGET) $(FP_TARGET).list
	make   --no-print-directory -C cs49n-libc clean
	make   --no-print-directory -C cs49n-src clean

.PHONY: clean all ctags
