# Open Multi-Processor Interrupt Controller (ompic)

The ompic device handles IPI communication between cores in multi-core
OpenRISC systems.

## Installation

The `ompic` is best used when built into a SoC.  Please use a system like
[FuseSoC](https://github.com/olofk/fusesoc) to find and build an OpenRISC
multicore soc which uses ompic.

## Specification

For full details please refer to the [openrisc
specification](https://openrisc.io/architecture).

### Registers

For each CPU the ompic has 2 registers. The control register for sending
and acking IPIs and the status register for receiving IPIs. The register
layouts are as follows:

#### Control register

```
 +---------+---------+----------+---------+
 | 31      | 30      | 29 .. 16 | 15 .. 0 |
 ----------+---------+----------+----------
 | IRQ ACK | IRQ GEN | DST CORE | DATA    |
 +---------+---------+----------+---------+
```

#### Status register

```
 +----------+-------------+----------+---------+
 | 31       | 30          | 29 .. 16 | 15 .. 0 |
 -----------+-------------+----------+---------+
 | Reserved | IRQ Pending | SRC CORE | DATA    |
 +----------+-------------+----------+---------+
```

### Architecture

 - The ompic generates a level interrupt to the CPU PIC when a message is
   ready.  Messages are delivered via the memory bus.
 - The ompic does not have any interrupt input lines.
 - The ompic is wired to the same irq line on each core.
 - Devices are wired to the same irq line on each core.

```
  +---------+                         +---------+
  | CPU     |                         | CPU     |
  |  Core 0 |<==\ (memory access) /==>|  Core 1 |
  |  [ PIC ]|   |                 |   |  [ PIC ]|
  +----^-^--+   |                 |   +----^-^--+
       | |      v                 v        | |
  <====|=|=================================|=|==> (memory bus)
       | |      ^                  ^       | |
 (ipi  | +------|---------+--------|-------|-+ (device irq)
  irq  |        |         |        |       |
 core0)| +------|---------|--------|-------+ (ipi irq core1)
       | |      |         |        |
  +----o-o-+    |    +--------+    |
  | ompic  |<===/    | Device |<===/
  |  IPI   |         +--------+
  +--------+*
```
