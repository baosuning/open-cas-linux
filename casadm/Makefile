#
# Copyright(c) 2012-2020 Intel Corporation
# SPDX-License-Identifier: BSD-3-Clause-Clear
#

PWD:=$(shell pwd)
MODULESDIR:=$(PWD)/../modules
UTILS_DIR:=$(PWD)/../utils
BINARY_PATH = /sbin

VERSION_FILE := $(MODULESDIR)/CAS_VERSION

#
# Section below enables creating build with experimental features
#
ifeq ($(CAS_EXT_EXP),1)
DEFINES    = WI_AVAILABLE
endif

#
# Add defines for version
#
-include $(VERSION_FILE)
DEFINES   += CAS_VERSION_MAIN=$(CAS_VERSION_MAIN)
DEFINES   += CAS_VERSION_MAJOR=$(CAS_VERSION_MAJOR)
DEFINES   += CAS_VERSION_MINOR=$(CAS_VERSION_MINOR)
DEFINES   += CAS_VERSION=\"$(CAS_VERSION)\"

#
# Include directories
#
INCLUDES     = .
INCLUDES    += $(MODULESDIR)/include

OBJDIR       = .obj/
TARGET	     = casadm
TARGETS      = $(TARGET)

#
# Source to be complied
#

OBJS  = cas_lib.o
OBJS += cas_main.o
OBJS += argp.o
OBJS += statistics_view_csv.o
OBJS += cas_lib_utils.o
OBJS += statistics_model.o
OBJS += table.o
OBJS += psort.o
OBJS += statistics_view_text.o
OBJS += intvector.o
OBJS += statistics_view.o
OBJS += statistics_view_raw_csv.o
OBJS += csvparse.o
OBJS += extended_err_msg.o
OBJS += upgrade.o
OBJS += safeclib/memmove_s.o
OBJS += safeclib/memcpy_s.o
OBJS += safeclib/memset_s.o
OBJS += safeclib/strncpy_s.o
OBJS += safeclib/strtok_s.o
OBJS += safeclib/safe_str_constraint.o
OBJS += safeclib/ignore_handler_s.o
OBJS += safeclib/safe_mem_constraint.o
OBJS += safeclib/mem_primitives_lib.o
OBJS += safeclib/strnlen_s.o

#
# Flags for C compilation
#
CFLAGS     = $(patsubst %,-I%,$(INCLUDES))
CFLAGS    += $(patsubst %,-D%,$(DEFINES))
ifdef DEBUG
CFLAGS    += -O0 -g
else
CFLAGS    += -O2 -D_FORTIFY_SOURCE=2
endif
CFLAGS    += -Wall -z relro -z now -fstack-protector -fPIC -Wformat -Wformat-security -fno-strict-aliasing

#
# Flags for linking
#
LDFLAGS    = -z noexecstack -z relro -z now -pie -pthread
#
# Targets
#

all: sync
	$(MAKE) build

build: $(VERSION_FILE) $(TARGETS)

sync:
	@cd $(MODULESDIR) && $(MAKE) sync

#
# Include dependencies file
#
$(TARGET): $(TARGET).a
	@echo "  LD " $@
	@$(CC) $(CFLAGS) $(LDFLAGS) -o $(TARGET) $<

$(TARGET).a: $(patsubst %,$(OBJDIR)%,$(OBJS))
	@echo "  AR " $@
	@ar rcs $@ $^
	@echo "  AR " libcas.a
	@cp -f $@ libcas.a
	@ar d libcas.a $(OBJDIR)argp.o $(OBJDIR)cas_main.c

#
# Generic target for C file
#
$(OBJDIR)%.o: %.c
	@echo "  CC " $<
	@mkdir -p $(dir $@)
ifeq ($(strip $(CAS_VERSION_MAIN)),)
	$(error "No version file")
endif
	@$(CC) -c $(CFLAGS) -MMD -MP -MF"$(@:%.o=%.d)" -MT"$(@:%.o=%.d)" -o "$@" "$<"

$(VERSION_FILE):
	@echo "  VERSION " $@
	@cd $(MODULESDIR) && ./CAS_VERSION_GEN

clean:
	@echo "  CLEAN "
	@rm -f *.a $(TARGETS)
	@rm -f $(shell find -name \*.d) $(shell find -name \*.o)

distclean: clean
	@rm -f $(VERSION_FILE)

install: install_files

install_files:
	@echo "Installing casadm"
	@mkdir -p $(DESTDIR)$(BINARY_PATH)
	@install -m 755 $(TARGET) $(DESTDIR)$(BINARY_PATH)/$(TARGET)
	@mkdir -p $(DESTDIR)/usr/share/man/man8
	@install -m 644 $(UTILS_DIR)/$(TARGET).8 $(DESTDIR)/usr/share/man/man8/$(TARGET).8

	@install -m 755 -d $(DESTDIR)/etc/opencas
	@install -m 644 $(UTILS_DIR)/opencas.conf $(DESTDIR)/etc/opencas/opencas.conf
	@install -m 444 $(UTILS_DIR)/ioclass-config.csv $(DESTDIR)/etc/opencas/ioclass-config.csv

	@mkdir -p $(DESTDIR)/usr/share/man/man5
	@install -m 644 $(UTILS_DIR)/opencas.conf.5 $(DESTDIR)/usr/share/man/man5/opencas.conf.5

uninstall:
	@echo "Uninstalling casadm"
	@rm $(DESTDIR)$(BINARY_PATH)/$(TARGET)
	@rm $(DESTDIR)/usr/share/man/man8/$(TARGET).8

	@rm $(DESTDIR)/etc/opencas/opencas.conf
	@rm $(DESTDIR)/etc/opencas/ioclass-config.csv
	@rm -rf $(DESTDIR)/etc/opencas

	@rm $(DESTDIR)/usr/share/man/man5/opencas.conf.5

.PHONY: clean distclean all sync build install uninstall
