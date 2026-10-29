---
title: Recovering the Flags from a Defcon 34 BBV Firmware Dump
date: 2026-10-29 12:00:00 +0000
categories: [Hardware, Reverse Engineering, CTF]
tags: [defcon, bbv, bug-bounty-village, avr, firmware, ghidra, reverse-engineering, hardware-hacking]
---

## Introduction

I recently spent time working through the Defcon 34 Bug Bounty Village CTF challenge in the repository [dhaval17/BBV-CTF-2026](https://github.com/dhaval17/BBV-CTF-2026). The challenge is a classic embedded security exercise: a custom board running an AVR microcontroller, a menu-based UI, and five hidden flags that are meant to be recovered without solving each hardware challenge individually.

The interesting part is not just that the device was vulnerable, but that the entire flag extraction path was readable from the firmware dump. This made the challenge a very good example of how “security through obscurity” fails when the underlying firmware is accessible.

## The Board and the Goal

The target board is built around an ATmega32 microcontroller, and the firmware image contains the full logic for the CTF flow. The repo README describes the board as a typical embedded challenge design:

- a firmware dump from flash memory
- a backup of EEPROM
- a menu with team labels
- a prompt asking for an 8-character flag input
- a hidden set of five team-specific flags

The challenge goal is straightforward: recover all five flags without physically solving each challenge path on the device.

## Dumping the Firmware

The first step was to connect to the chip via the ISP interface. The board exposes the standard AVR programming pins, and using a USBasp programmer with `avrdude`, the firmware and EEPROM can be extracted directly.

```bash
avrdude -c usbasp -p m32 -U flash:r:firmware_backup.hex:i
avrdude -c usbasp -p m32 -U eeprom:r:eeprom_backup.hex:i
```

This yields a full flash memory image and EEPROM image, both of which are essential for static analysis.

The README notes the exact memory sizes recovered:

| File | Contents |
|---|---|
| `firmware_backup.hex` / `.bin` | 9,116 bytes of flash |
| `eeprom_backup.hex` / `.bin` | 1,024 bytes of EEPROM |

That is enough to see that the board is not hiding a large amount of data; this is a small, intentionally simple firmware image.

## Identifying the Target Chip

The next obvious step is to inspect the dumped binary and identify the MCU architecture and memory layout.

The first bytes of the firmware show the AVR interrupt vector table pattern, and the README walks through the key observations:

- the binary begins with a standard AVR interrupt vector table
- the reset code initializes the stack pointer to the top of SRAM
- the code copies `.data` to SRAM starting at `0x0060`
- the I/O register IDs match the ATmega32 family

This confirms the board is using an ATmega32, which is a very common AVR target for these kinds of CTF hardware challenges.

## Strings Reveal the UI

The README’s next insight is simple but very effective: `strings` on the firmware image exposes the menu and the challenge flow.

```bash
strings -n 4 firmware_backup.bin
```

This reveals the game text:

```text
ENTER ANY FLAG / INPUT: -------- / PRESS CTF TO SUBMIT
1. RED TEAM  2. BLUE TEAM  3. PURPLE TEAM  4. SLOW RGB  5. PARTY TEAM
FLAG SUCCESS! / FLAG WRONG! RETRY / ALREADY SOLVED! / 0 OF 5 SOLVED
EEPROM CLEARED! / RESET PROGRESS / heemanshu
```

This immediately tells us two important things:

1. The flag input is 8 symbols long.
2. The UI is designed around five challenge teams.

The most useful part is that the strings tell us the challenge system is not obfuscated. The actual secret values are likely stored directly in code as constants, which means the solution can be found by analysis rather than by brute forcing the hardware.

## Mapping Strings to Code References

The README explains how the UI strings are mapped back into memory addresses in flash and SRAM. That gave a mapping between displayed text and the routines that reference it.

The important finding is that the strings are passed to a display routine using register pairs. This gives the analysis a reliable point of entry: identify the code that loads the message strings and trace the surrounding control flow.

That makes the reverse engineering process much more direct than it would be with a more complex firmware image.

## Disassembly and the Input Routine

Since AVR binutils were not installed on the host system, the challenge used Ghidra headless analysis for the firmware.

The README includes the command used:

```bash
analyzeHeadless ./proj ctf -import firmware_backup.bin -loader BinaryLoader \
  -processor avr8:LE:16:default -cspec gcc -postScript ListAll.java
```

Once the disassembly is loaded, the critical routine is the routine that handles button inputs. The board has four buttons on PC4–PC7, and each press modifies a bit accumulator that eventually builds an 8-bit value.

The challenge logic is straightforward:

- a button press enters a bit value
- the bits are shifted into an accumulator
- on submit, the system compares the input against known constants
- if the value matches one of the five valid entries, the EEPROM progress bit is set

This is exactly the sort of logic that makes AVR CTFs easy to solve once the binary is known.

## The Flag Check Logic

The board checks input on submit and compares it against a set of constants. The README breaks the logic down clearly:

```asm
0b2e  cpi  R25, 5        ; solved-count < 5 ? (else "ALL CHALLENGES SOLVED")
0b34  cpi  R30, 8        ; exactly 8 bits entered ?
0b38  ldd  R31, [input]
0b3a  cpi  R31, 0xEB
0b3c  cpi  R31, 0xA7
0b3f  cpi  R31, 0x56
0b42  cpi  R31, 0xC9
0b45  cpi  R31, 0x2B
```

The control flow is clear: the board expects the user to enter a value that matches one of the team-specific constants. Once matched, the code checks whether the challenge has already been solved in EEPROM. If not, it sets the corresponding progress bit and then reports success.

This is where the real weakness appears: the firmware is comparing plaintext constants. There is no secret derivation, encryption, or obfuscation layer protecting the values.

## The One-Shot Solver

The README is especially nice because it includes a “no Ghidra needed” approach once the pattern is understood.

The idea is simple:

- identify the `CPI` instruction that enforces the 8-bit length check
- find the subsequent `CPI` immediates that follow on the same register
- decode the immediate values directly from the `.hex` or `.bin` image

That is a great example of how embedded firmware reversing can become a mechanical process once the architecture is understood.

A Python-based extractor can parse the `.hex` file and print the team values without full decompilation.

```python
import sys


def load_ihex(path):
    mem = {}
    for line in open(path):
        line = line.strip()
        if not line.startswith(':'):
            continue
        b = bytes.fromhex(line[1:])
        n, addr, typ = b[0], (b[1] << 8) | b[2], b[3]
        if typ == 0:
            for i in range(n):
                mem[addr + i] = b[4 + i]
    return bytes(mem.get(i, 0xFF) for i in range(max(mem) + 1))


def cpi(word):
    if word & 0xF000 != 0x3000:
        return None
    return 16 + ((word >> 4) & 0xF), ((word >> 4) & 0xF0) | (word & 0xF)

fw = load_ihex(sys.argv[1])
words = [fw[i] | fw[i+1] << 8 for i in range(0, len(fw) - 1, 2)]
for i, w in enumerate(words):
    c = cpi(w)
    if c and c[1] == 8:
        consts, reg = [], None
        for w2 in words[i+1:i+20]:
            c2 = cpi(w2)
            if c2 and (reg is None or c2[0] == reg):
                reg = c2[0]
                consts.append(c2[1])
        if len(consts) == 5:
            names = ["RED TEAM", "BLUE TEAM", "PURPLE TEAM", "SLOW RGB", "PARTY TEAM"]
            for n, k in zip(names, consts):
                print(f"{n:12s} 0x{k:02X}  {k:08b}")
```

This is exactly the kind of challenge writeup that is useful because it both explains the attack path and teaches the underlying reversing workflow.

## What This Challenge Teaches

There are a few important lessons here.

### 1. Exposed debug interfaces are dangerous

The AVR ISP pins were available and readable. This is a classic example of hardware security being undermined by physical access.

### 2. Unlocked lock bits are a red flag

If the chip allows full firmware readout, the board is effectively giving away the secret data. In real-world systems, lock bits and fuse configuration are a critical line of defense.

### 3. Plaintext secrets in firmware are a design flaw

The challenge uses direct constants in flash memory. That makes static analysis extremely simple and shows why sensitive values should never be stored in raw form if the device is in an untrusted environment.

### 4. Reversing is often easier than the authors expect

This board is a good example of how small embedded systems can be solved by combining basic hardware access with static analysis. The challenge is not about cryptography; it is about understanding the device and reading the machine code.

## Conclusion

The Defcon 34 BBV CTF repo is a great example of a compact embedded security exercise that balances real hardware interaction with practical reverse engineering. By dumping the firmware, identifying the ATmega32 architecture, reading the strings, and tracing the flag-check logic in Ghidra, the attacker can recover all five challenge flags without solving each hardware puzzle individually.

The author’s write-up in the README does a very good job of showing the full flow: from hardware reconnaissance to binary analysis to final flag extraction. It is a perfect example of how embedded challenge design can be both accessible and educational.

If you are interested in hardware reversing, AVR exploitation, or CTF firmware analysis, this repo is a very solid case study. The key takeaway is simple: when the firmware is readable, the secrets are rarely safe.

---

This write-up was adapted from the project README in [BBV-CTF-2026](https://github.com/dhaval17/BBV-CTF-2026) and formatted to fit the existing Jekyll blog structure used by [blog.dr4cun0.com](https://github.com/dhaval17/blog.dr4cun0.com).
