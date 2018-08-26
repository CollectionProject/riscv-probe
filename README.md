# riscv-probe

Simple machine mode program to probe RISC-V control and status registers.

riscv-probe currently works with [Spike](https://github.com/riscv/riscv-isa-sim)
and [QEMU](https://github.com/riscv/riscv-qemu) and can be used to compare
control and status registers. It can also be used as a starting point for
bare metal RISC-V programs that need printf, getchar and putchar.

## Invocation

Example invocations:

- `$ spike --isa=RV32IMAFDC build/bin/rv32/probe-htif`
- `$ spike --isa=RV64IMAFDC build/bin/rv64/probe-htif`
- `$ qemu-system-riscv32 -nographic -machine spike_v1.10 -kernel build/bin/rv32/probe-spike`
- `$ qemu-system-riscv64 -nographic -machine spike_v1.10 -kernel build/bin/rv64/probe-spike`
- `$ qemu-system-riscv32 -nographic -machine virt -kernel build/bin/rv32/probe-virt`
- `$ qemu-system-riscv64 -nographic -machine virt -kernel build/bin/rv64/probe-virt`
- `$ qemu-system-riscv32 -nographic -machine sifive_e -kernel build/bin/rv32/probe-qemu-sifive_e`
- `$ qemu-system-riscv64 -nographic -machine sifive_e -kernel build/bin/rv64/probe-qemu-sifive_e`
- `$ qemu-system-riscv32 -nographic -machine sifive_u -kernel build/bin/rv32/probe-qemu-sifive_u`
- `$ qemu-system-riscv64 -nographic -machine sifive_u -kernel build/bin/rv64/probe-qemu-sifive_u`

### riscv-probe in qemu-system-riscv32

```
$ qemu-system-riscv32 -nographic -machine spike_v1.10 -kernel build/bin/rv32/probe-spike
isa: rv32imafdcsu
csr: fflags          (not supported) cause=2
csr: frm             (not supported) cause=2
csr: fcsr            (not supported) cause=2
csr: mcycle          0x0f633494
csr: minstret        0x0f6cfad6
csr: mcycleh         0x00052369
csr: minstreth       0x00052369
csr: cycle           0x0f7e80a8
csr: time            (not supported) cause=2
csr: instret         0x0f8cc789
csr: cycleh          0x00052369
csr: timeh           (not supported) cause=2
csr: instreth        0x00052369
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
csr: mepc            0x80000a50
csr: mcause          0x00000002
csr: mtval           0x00000000
csr: mip             0x00000000
csr: sstatus         0x00000000
csr: sedeleg         (not supported) cause=2
csr: sideleg         (not supported) cause=2
csr: sie             0x00000000
csr: stvec           0x00000000
csr: scounteren      0x00000000
csr: sscratch        0x00000000
csr: sepc            0x00000000
csr: scause          0x00000000
csr: stval           0x00000000
csr: sip             0x00000000
csr: satp            0x00000000
```

### riscv-probe in qemu-system-riscv64

```
$ qemu-system-riscv64 -nographic -machine spike_v1.10 -kernel build/bin/rv64/probe-spike
isa: rv64imafdcsu
csr: fflags          (not supported) cause=2
csr: frm             (not supported) cause=2
csr: fcsr            (not supported) cause=2
csr: mcycle          0x000523808d7c98ff
csr: minstret        0x000523808d885d41
csr: mcycleh         (not supported) cause=2
csr: minstreth       (not supported) cause=2
csr: cycle           0x000523808da09bed
csr: time            (not supported) cause=2
csr: instret         0x000523808db032f8
csr: cycleh          (not supported) cause=2
csr: timeh           (not supported) cause=2
csr: instreth        (not supported) cause=2
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
csr: mepc            0x0000000080000866
csr: mcause          0x0000000000000002
csr: mtval           0x0000000000000000
csr: mip             0x0000000000000000
csr: sstatus         0x0000000000000000
csr: sedeleg         (not supported) cause=2
csr: sideleg         (not supported) cause=2
csr: sie             0x0000000000000000
csr: stvec           0x0000000000000000
csr: scounteren      0x0000000000000000
csr: sscratch        0x0000000000000000
csr: sepc            0x0000000000000000
csr: scause          0x0000000000000000
csr: stval           0x0000000000000000
csr: sip             0x0000000000000000
csr: satp            0x0000000000000000
```
