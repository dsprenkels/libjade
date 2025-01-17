# -*- Makefile -*-

# --------------------------------------------------------------------
AS      ?= as
CC      ?= gcc
CFLAGS  ?= -Wall -Wno-unused-function -Ofast
CI      ?= 0
export CI

EXCLUDE ?= crypto_xof/shake256/amd64/spec/

# --------------------------------------------------------------------
SRC      := ../src
BIN      ?= bin
COMMON   := common

EXT      := ../ext
RANDLIB  ?= $(EXT)/randombytes/librandombytes.a
RANDINC  ?= $(EXT)/randombytes/
SSBD_DIR := $(EXT)/ssbd-tools

JAZZ    ?= $(filter-out $(addprefix $(SRC)/,$(EXCLUDE)), $(sort $(dir $(shell find $(SRC) -name '*.jazz'))))
BENCH   := $(subst $(SRC),$(BIN), $(JAZZ))
BENCHS  := $(addsuffix bench, $(BENCH))

# --------------------------------------------------------------------

RDIR        = $(subst $(BIN)/,,$(@D))
OPERATION   = $(subst crypto_,,$(word 1, $(subst /, ,$(RDIR))))
NAMESPACE   = $(subst crypto_,jade_,$(subst -,_,$(subst /,_,$(RDIR))))
NAMESPACE1  = $(shell echo $(NAMESPACE) | tr a-z A-Z)
IDIR        = $(subst $(BIN),$(SRC),$(@D))
ASM         = $(IDIR)/$(OPERATION).s

MAIN        = $(COMMON)/crypto_$(OPERATION).c
DEFINE     ?=
DNAMESPACES = -DJADE_NAMESPACE=$(NAMESPACE1) -DJADE_NAMESPACE_LC=$(NAMESPACE)
INCLUDES    = -I$(IDIR)/include/ -I$(COMMON)/ -I$(RANDINC)
COMPILE     = $(CC) $(CFLAGS) -o $@ $(DEFINE) $(DNAMESPACES) $(INCLUDES) $(MAIN) $(ASM) $(RANDLIB)

# --------------------------------------------------------------------
.PHONY: __phony benchs

default: benchs

benchs: $(BENCHS)

$(BENCHS):
%/bench: $(RANDLIB) __phony | %/
	$(MAKE) -C $(IDIR) || true
	$(COMPILE) || true

$(RANDLIB):
	$(MAKE) -C $(@D)

$(BENCH): ; @mkdir -p $@

run: benchs
	for d in $(BENCH); do (cd $$d && ./bench); done

# --------------------------------------------------------------------
.PHONY: ssbd-tools
ssbd-tools:
	make -C $(SSBD_DIR)

# --------------------------------------------------------------------
clean:
	rm -fr $(BIN)

