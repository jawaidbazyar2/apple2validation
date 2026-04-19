# 6502 IRQ Test

The purpose of this test is to identify and validate 6502/816 behavior around when it senses IRQ and when it acts on IRQ.

Various documentation suggests the following:

The 6502 interrupt line is checked during the penultimate (second to last) cycle of each instruction. In essence, whatever the value is at the end of that cycle, is latched, and used later to determine whether to interrupt. And, if an instruction is a 2-cycle instruction, this means the decision of whether to interrupt the NEXT instruction starts on the first cycle of the current one. (That's a little hard to think about). This means the IRQ decision is also gated at this time by the value in the Inh flag.

The 6502 implements its reponse to an IRQ, by forcing the opcode of the affected instruction to $00, or, a BRK instruction. This is why the 6502 uses the same vector for IRQ and BRK, and, why there is a BRK flag in the P status register to tell you whether the interrupt was caused by software (BRK) or hardware (IRQ).

This is related to the opcode pipelining in the 6502. The interrupt status needs to be checked when the opcode is read, so it can be converted to $00. The next opcode is fetched in the last cycle of the current instruction, so the sequence is:

Cycle -2: check IRQ
Cycle -1: fetch next opcode OR force it to $00 (based on IRQ status from previous cycle)
Cycle 0: IRQ response or next instruction starts executing

There are several behaviors tested here.

1. That the interrupt status level as of the penultimate cycle is what is used to trigger interrupt.
2. That the I (interrupt inhibit) bit is set in the SECOND cycle of a CLI or SEI instruction (i.e., AFTER the penultimate cycle)
3. That the I bit is set BEFORE the penultimate cycle of an RTI or PLP instruction.

The practical result of these is the following:

if interrupts are disabled (Inh=1), but asserted (IRQ input active), and you have the following instruction sequence:
```
CLI
SEI
LDA #0
```

what should happen is that the interrupt will "fire" (i.e., the IRQ response in the 6502 will jump through the interrupt vector) **after the SEI instruction**. The sequence is:
1. IRQ=1 during first cycle of CLI but INH=1, which means already that CPU will not trigger IRQ response on next instruction!
2. CLI sets Inh=0 on its 2nd cycle.
3. IRQ=1 and Inh=0 on 1st cycle of SEI, meaning the NEXT instruction will be interrupted.
4. SEI sets Inh=1 on 2nd cycle.
5. the LDA is interrupted and the IRQ handler run instead of the LDA.

This may seem odd, that you can interrupt the CPU *after* an SEI instruction, but there it is.

For this test we require an interrupt source that will run on an Apple IIe and a IIgs. The IIgs has built-in interrupt sources, we use the VBL. The IIe does not, so we use a Mockingboard's 6522 chip to provide a software-controllable interrupt.

## Running the Test

Disk Images: 
[MBTest 800K](mbtest.po) | 
[MBTest 140K](mbtest140.po)

1. Boot the disk image.
2. Ctrl-Reset (just to make sure no other interrupts are stuck on)
3. BLOAD MBTEST (IIe + Mockingboard)   or BLOAD GSTEST (IIgs)
4. CALL -151
5. 800G

the test may:
1. print nothing
2. print the single number 0001
3. print a range of numbers of 0001 - 0100
4. Something else entirely

## How the Test Works

A mockingboard timer interrupt (mbtest) or a VBL interrupt (gstest) are triggered and left on.


## Test Results

| Platform | Version | Result | Contributor |
|-|-|-|-|
| Apple IIe + MB | 65c02 | single 0001 | Nick Bauer |
| Apple IIgs | ROM01 | single 0001 | me |
| Apple IIgs | ROM03 | single 0001 | Nick Bauer |
| perfect6502 | | interrupt triggered after SEI | |
| GSSquared | 0.7.2 | 0001 | this result was accidental |
| GSSquared | 0.7.4 | 0001 | will be fixed in latest version |
| Apple2ts | Apr 19, 2026 | IRQ never stops, numbers print indefinitely |
| ApplEm | Apr 19, 2026 | single 0001 | Mike Daley |
| Mariani | 1.5(1) | 0001 - 0100 | me |


## Building the Test

You'll need Merlin32 and CiderPress2. Set the paths to these programs in the Makefile and type 'make all'
