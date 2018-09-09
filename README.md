# riscv-probe

Simple machine mode program to probe RISC-V control and status registers.

riscv-probe currently builds for
[Spike](https://github.com/riscv/riscv-isa-sim),
[QEMU](https://github.com/riscv/riscv-qemu) and the
[SiFive E21](https://www.sifive.com/products/risc-v-core-ip/e2/e21/) core.
riscv-probe is a testing tool designed be used to compare CSRs (Control and
Status Registers) between mutliple RISC-V simulators and RISC-V hardware
implementations.

riscv-probe contains libfemto which is a lightweight bare-metal C library
conforming to a reduced set ot the _POSIX.1-2017 / IEEE 1003.1-2017_ standard.
libfemto can be used as a starting point for bare metal RISC-V programs that
require interrupt handling, basic string routines and printf.

## Build

To build the examples:

```
make
```

## Invocation

To invoke the probe example in spike and RISC-V QEMU:

- `$ spike --isa=RV32IMAFDC build/bin/rv32/spike/probe`
- `$ spike --isa=RV64IMAFDC build/bin/rv64/spike/probe`
- `$ qemu-system-riscv32 -nographic -machine spike_v1.10 -kernel build/bin/rv32/spike/probe`
- `$ qemu-system-riscv64 -nographic -machine spike_v1.10 -kernel build/bin/rv64/spike/probe`
- `$ qemu-system-riscv32 -nographic -machine virt -kernel build/bin/rv32/virt/probe`
- `$ qemu-system-riscv64 -nographic -machine virt -kernel build/bin/rv64/virt/probe`
- `$ qemu-system-riscv32 -nographic -machine sifive_e -kernel build/bin/rv32/qemu-sifive_e/probe`
- `$ qemu-system-riscv64 -nographic -machine sifive_e -kernel build/bin/rv64/qemu-sifive_e/probe`
- `$ qemu-system-riscv32 -nographic -machine sifive_u -kernel build/bin/rv32/qemu-sifive_u/probe`
- `$ qemu-system-riscv64 -nographic -machine sifive_u -kernel build/bin/rv64/qemu-sifive_u/probe`

## libfemto

libfemto is a lightweight bare-metal C library for embedded RISC-V development.
libfemto provides:

- Reduced set of the _POSIX.1-2017 / IEEE 1003.1-2017_ standard
- Simple lightweight hardware configuration mechanism
- RISC-V machine mode functions and macros
- Console and power device drivers

libfemto implements a reduced set of the _POSIX.1-2017 / IEEE 1003.1-2017_
standard, with the addition of glibc's `getauxval` API to access hardware
configuration in an auxiliary vector (`__auxv`) that contains tuples describing
the target environment. This is intended as a lightweight mechanism to pass
dynamic configuration information for embedded targets, as an alternative to
hardcoding constants used during hardware initialization. The auxiliary vector
is repurposed to contain hardware configuration parameters such as clock
frequencies and device base addresses.

libfemto contains the following device drivers:

- HTIF (Host Target Interface)
- NS16550A UART Console
- SiFive UART Console
- SiFive Test Device
- Semihosting Syscalls

### Environments

This project contains a simple build system that allows building applications
targeting multiple embedded environments. A distinguishing characteristic of
the build system is that program objects do not need to be recompiled to target
a different environment, rather they are relinked with a different hardware
configuration and setup function. The config object causes the correct drivers
to be linked via compile time dependencies expressed by symbol references.
The following environments are currently supported:

- _default_ - environment where IO defaults to `ebreak`
- _spike_- the RISC-V ISA Simulator Golden Model
- _virt_ - the RISC-V VirtIO Virtual Machine
- _qemu-sifive_e_ - QEMU Functional Model of the SiFive E Series
- _qemu-sifive_u_ - QEMU Functional Model of the SiFive U Series
- _coreip-e2-arty_ - SiFive E2 CoreIP Arty A7 FPGA evaluation image

To create a new environment simply add a directory to `env` with two files:

- `default.lds` - linker script describing the target's memory layout
- `config.c` - environment specific configuration

The following is an example configuration from `env/<boardname>/config.c`
showing the auxiliary vector used by `getauxval` via the `setup` function
called by `_start` before entering `main`.

```
auxval_t __auxv[] = {
    { UART0_CLOCK_FREQ,         32000000   },
    { UART0_BAUD_RATE,          115200     },
    { SIFIVE_UART0_CTRL_ADDR,   0x20000000 },
    { SIFIVE_TEST_CTRL_ADDR,    0x4000     },
    { 0, 0 }
};

void setup()
{
    /*
     * clock setup code should be placed here and should modify the
     * uart clock speed before calling register_console, which calls
     * uart_init and reads the uart clock speed from the config array.
     */
    register_console(&console_sifive_uart);
    register_poweroff(&poweroff_sifive_test);
}
```

## Examples

The build system automatically includes any directory added to `examples`
which contains a `rules.mk` file.

### hello

The following is the `rules.mk` file from the _hello_ example:

```
$ cat examples/hello/rules.mk 
hello_objs = hello.o
```

and `hello.c`

```
$ cat examples/hello/hello.c 
#include <stdio.h>

int main()
{
	printf("hello\n");
}
```

### tiny

The `_start` symbol in _libfemto_ is a weak symbol so it is also possible
to override the default entry stub with your own code. The following is from
the _tiny_ example:

```
$ cat examples/tiny/rules.mk 
tiny_objs = entry.o tiny.o
```

Here is the modified `entry.s`

```
$ cat examples/tiny/entry.s 
# See LICENSE for license details.

.include "macros.s"
.include "constants.s"

#
# start of trap handler
#

.section .text.init,"ax",@progbits
.globl _start

_start:
    la      sp, stacks + STACK_SIZE
    jal     setup
    jal     main
    j       poweroff

    .bss
    .align 4
stacks:
    .skip STACK_SIZE
```

### riscv-probe

`riscv-probe` is a utility that probes the Control and Status Register address
space of a RISC-V emulator, FPGA or board:

#### qemu-system-riscv32

```
$ qemu-system-riscv32 -nographic -machine spike_v1.10 -kernel build/bin/rv32/spike/probe
isa: rv32imafdcsu
csr: fflags          illegal_instruction cause=0x00000002 mtval=0x00000000
csr: frm             illegal_instruction cause=0x00000002 mtval=0x00000000
csr: fcsr            illegal_instruction cause=0x00000002 mtval=0x00000000
csr: mcycle          0xdbfa9cbd
csr: minstret        0xdc03f6a4
csr: mcycleh         0x0007c452
csr: minstreth       0x0007c452
csr: cycle           0xdc1d7e08
csr: time            illegal_instruction cause=0x00000002 mtval=0x00000000
csr: instret         0xdc393bf6
csr: cycleh          0x0007c452
csr: timeh           illegal_instruction cause=0x00000002 mtval=0x00000000
csr: instreth        0x0007c452
csr: mvendorid       0x00000000
csr: marchid         0x00000000
csr: mimpid          0x00000000
csr: mhartid         0x00000000
csr: mstatus         0x00000000
csr: misa            0x4014112d
csr: medeleg         0x00000000
csr: mideleg         0x00000000
csr: mie             0x00000000
csr: mtvec           0x80000004
csr: mcounteren      0x00000000
csr: mscratch        0x00000000
csr: mepc            0x800002a4
csr: mcause          0x00000002
csr: mtval           0x00000000
csr: mip             0x00000000
csr: sstatus         0x00000000
csr: sedeleg         illegal_instruction cause=0x00000002 mtval=0x00000000
csr: sideleg         illegal_instruction cause=0x00000002 mtval=0x00000000
csr: sie             0x00000000
csr: stvec           0x00000000
csr: scounteren      0x00000000
csr: sscratch        0x00000000
csr: sepc            0x00000000
csr: scause          0x00000000
csr: stval           0x00000000
csr: sip             0x00000000
csr: satp            0x00000000
csr: pmpcfg0         0x00000000
csr: pmpcfg1         0x00000000
csr: pmpcfg2         0x00000000
csr: pmpcfg3         0x00000000
csr: pmpaddr0        0x00000000
csr: pmpaddr1        0x00000000
csr: pmpaddr2        0x00000000
csr: pmpaddr3        0x00000000
csr: pmpaddr4        0x00000000
csr: pmpaddr5        0x00000000
csr: pmpaddr6        0x00000000
csr: pmpaddr7        0x00000000
csr: pmpaddr8        0x00000000
csr: pmpaddr9        0x00000000
csr: pmpaddr10       0x00000000
csr: pmpaddr11       0x00000000
csr: pmpaddr12       0x00000000
csr: pmpaddr13       0x00000000
csr: pmpaddr14       0x00000000
csr: pmpaddr15       0x00000000
```

#### qemu-system-riscv64

```
$ qemu-system-riscv64 -nographic -machine spike_v1.10 -kernel build/bin/rv64/spike/probe
isa: rv64imafdcsu
csr: fflags          illegal_instruction cause=0x00000002 mtval=0x00000000
csr: frm             illegal_instruction cause=0x00000002 mtval=0x00000000
csr: fcsr            illegal_instruction cause=0x00000002 mtval=0x00000000
csr: mcycle          0x0007c452dfeeddd3
csr: minstret        0x0007c452dff8a765
csr: mcycleh         illegal_instruction cause=0x00000002 mtval=0x00000000
csr: minstreth       illegal_instruction cause=0x00000002 mtval=0x00000000
csr: cycle           0x0007c452e01f105f
csr: time            illegal_instruction cause=0x00000002 mtval=0x00000000
csr: instret         0x0007c452e03d0a50
csr: cycleh          illegal_instruction cause=0x00000002 mtval=0x00000000
csr: timeh           illegal_instruction cause=0x00000002 mtval=0x00000000
csr: instreth        illegal_instruction cause=0x00000002 mtval=0x00000000
csr: mvendorid       0x0000000000000000
csr: marchid         0x0000000000000000
csr: mimpid          0x0000000000000000
csr: mhartid         0x0000000000000000
csr: mstatus         0x0000000000000000
csr: misa            0x800000000014112d
csr: medeleg         0x0000000000000000
csr: mideleg         0x0000000000000000
csr: mie             0x0000000000000000
csr: mtvec           0x0000000080000004
csr: mcounteren      0x0000000000000000
csr: mscratch        0x0000000000000000
csr: mepc            0x00000000800002f4
csr: mcause          0x0000000000000002
csr: mtval           0x0000000000000000
csr: mip             0x0000000000000000
csr: sstatus         0x0000000000000000
csr: sedeleg         illegal_instruction cause=0x00000002 mtval=0x00000000
csr: sideleg         illegal_instruction cause=0x00000002 mtval=0x00000000
csr: sie             0x0000000000000000
csr: stvec           0x0000000000000000
csr: scounteren      0x0000000000000000
csr: sscratch        0x0000000000000000
csr: sepc            0x0000000000000000
csr: scause          0x0000000000000000
csr: stval           0x0000000000000000
csr: sip             0x0000000000000000
csr: satp            0x0000000000000000
csr: pmpcfg0         0x0000000000000000
csr: pmpcfg1         0x0000000000000000
csr: pmpcfg2         0x0000000000000000
csr: pmpcfg3         0x0000000000000000
csr: pmpaddr0        0x0000000000000000
csr: pmpaddr1        0x0000000000000000
csr: pmpaddr2        0x0000000000000000
csr: pmpaddr3        0x0000000000000000
csr: pmpaddr4        0x0000000000000000
csr: pmpaddr5        0x0000000000000000
csr: pmpaddr6        0x0000000000000000
csr: pmpaddr7        0x0000000000000000
csr: pmpaddr8        0x0000000000000000
csr: pmpaddr9        0x0000000000000000
csr: pmpaddr10       0x0000000000000000
csr: pmpaddr11       0x0000000000000000
csr: pmpaddr12       0x0000000000000000
csr: pmpaddr13       0x0000000000000000
csr: pmpaddr14       0x0000000000000000
csr: pmpaddr15       0x0000000000000000
```
