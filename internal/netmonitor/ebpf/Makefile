BPF_CLANG ?= clang
BPF_CFLAGS ?= -O2 -g -Wall -Werror -target bpfel
# Debian/Ubuntu keep asm/types.h in the host multiarch include directory.
HOST_CC ?= cc
BPF_CPPFLAGS ?= -isystem /usr/include/$(shell $(HOST_CC) -dumpmachine)

BPF_ALLOW_MAX ?= 1024
BPF_DENY_MAX ?= 1024
BPF_LPM_MAX ?= 1024
BPF_LPM_DENY_MAX ?= 1024
BPF_DEFAULT_DENY_MAX ?= 1024

all: connect_bpfel.o connect_bpfel_arm64.o

connect_bpfel.o: connect.bpf.c Makefile
	$(BPF_CLANG) $(BPF_CPPFLAGS) $(BPF_CFLAGS) -MMD -MP -D__TARGET_ARCH_x86 -DALLOWLIST_MAX_ENTRIES=$(BPF_ALLOW_MAX) -DDENYLIST_MAX_ENTRIES=$(BPF_DENY_MAX) -DLPM_MAX_ENTRIES=$(BPF_LPM_MAX) -DLPM_DENY_MAX_ENTRIES=$(BPF_LPM_DENY_MAX) -DDEFAULT_DENY_MAX_ENTRIES=$(BPF_DEFAULT_DENY_MAX) -c $< -o $@

connect_bpfel_arm64.o: connect.bpf.c Makefile
	$(BPF_CLANG) $(BPF_CPPFLAGS) $(BPF_CFLAGS) -MMD -MP -D__TARGET_ARCH_arm64 -DALLOWLIST_MAX_ENTRIES=$(BPF_ALLOW_MAX) -DDENYLIST_MAX_ENTRIES=$(BPF_DENY_MAX) -DLPM_MAX_ENTRIES=$(BPF_LPM_MAX) -DLPM_DENY_MAX_ENTRIES=$(BPF_LPM_DENY_MAX) -DDEFAULT_DENY_MAX_ENTRIES=$(BPF_DEFAULT_DENY_MAX) -c $< -o $@

clean:
	rm -f connect_bpfel.o connect_bpfel_arm64.o connect_bpfel.d connect_bpfel_arm64.d

.PHONY: all clean

-include connect_bpfel.d connect_bpfel_arm64.d
