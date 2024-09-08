# DuPAL

Here You'll find all the repositories related to my work on the various incarnations of the DuPAL devices, the tools I use to analyze and reverse old PLD devices.

## DuPAL V3, aka "dupico"

The dupico is the new incarnation of the DuPAL, based on the Raspberry Pico. It's a much more flexible device in respect to the previous incarnation, and can act as a dumper for other types of devices.

Development is currently carried on on the dupico only, and the other incarnations are considered legacy.

## Repositories

- [DuPAL_Board](https://github.com/DuPAL-PAL-DUmper/DuPAL_Board), here You'll find schematics and gerbers for the various incarnations of DuPAL boards and accessories
- [dupico_firmware](https://github.com/DuPAL-PAL-DUmper/dupico_firmware), firmware for the dupico board
- [PAL_samples](https://github.com/DuPAL-PAL-DUmper/PAL_samples), repository with some PAL examples I use for experiments
- [dpdumper](https://github.com/DuPAL-PAL-DUmper/dpdumper), tool to dump simple combinatorial devices (e.g. old PALs, but also ROMs)
- [dpdump2tab](https://github.com/DuPAL-PAL-DUmper/dpdump2tab), tool to convert dumps from purely combinatorial devices made with dpdumper into truth tables compatible with `espresso`
- [dppeeper](https://github.com/DuPAL-PAL-DUmper/dppeeper), tool for visual and interactive analysis of a PLD
- [dupicolib](https://github.com/DuPAL-PAL-DUmper/dupicolib), library for hardware interfacing with the dupico board, required by the above projects
- [dpdumperlib](https://github.com/DuPAL-PAL-DUmper/dpdumperlib), library with shared code used by the dpdumper and other projects
- [espresso-logic](https://github.com/DuPAL-PAL-DUmper/espresso-logic), port of espresso that can be built using msys2/mingw

## Limitations

I plan to write a proper document on this, but keep in mind that analyzing PLD devices with registered outputs (based on flip-flops) and especially devices making use of feedbacks is an arduous task. DuPAL helps, but is no silver bullet.

Feedbacks are especially problematic because they generate intermediate states that cannot be detected by the dupico (and maybe can just be guessed by continuous sampling of the outputs of the PLD device) and thus cannot be recovered by automatic analysis.

An example is the following. Consider these logic equations:

```
/o18 = i1
/o17 = /i1 * /o18 + /o17 * i2
```

`o17` and `o18` act as feedbacks, so they are both outputs and inputs to the equations, and, as inputs, are not under direct user control.

The following is a partial truth table for these equations:

| i2 | i1 | o18 | o17 | ->  | o18 | o17 |
| -- | -- | --- | --- | --- | --- | --- |
|  0 |  0 |  0  |  0  |     |  1  |  0  |
|  0 |  1 |  0  |  0  |     |  0  |  1  |
|  1 |  0 |  0  |  0  |     |  1  |  0  |
|  1 |  1 |  0  |  0  |     |  0  |  0  |
|  0 |  0 |  1  |  0  |     |  1  |  1  |
|  0 |  1 |  1  |  0  |     |  0  |  1  |
|  1 |  0 |  1  |  0  |     |  1  |  0  |
|  1 |  1 |  1  |  0  |     |  0  |  0  |
|  0 |  0 |  1  |  1  |     |  1  |  1  |
|  0 |  1 |  1  |  1  |     |  0  |  1  |
|  1 |  0 |  1  |  1  |     |  1  |  1  |
|  1 |  1 |  1  |  1  |     |  0  |  1  |

Imagine starting in state `00` (so both `o18` and `o17` at 0), and setting the two inputs `i1` and `i2` at 0.
Following the truth table you will see that You'll immediately switch to state `10` (`o18` high, and `o17` low),
but state `10` with input `00` is not stable: following the truth table again, we see that we will switch to state `11` immediately, 
and we will stop here.

State `10` is an intermediate state, and is not captured by sampling the outputs at a low frequency.

## TODO

In no particular order:

- Extract IC loading code from dppeeper into a library, so it can be shared between other tools
- Write a section of this document detailing various analysis paths dependant on the type of IC
- ~Implement a way in firmware to have the client "describe" the structure of a combinatorial IC, and then transfer the content via e.g. XMODEM, to reduce the overhead~
- Implement a de-ambiguizer that takes a "peeper" toml defintion that has pins shared between I/O/Q/Clock and tries to understand their actual function, then outputs a non ambiguous TOML.
- Implement an analyzer that uses a non-ambiguous TOML and tries to extract feedback states and registered states.
- Implement a conversion from whatever the analyzer outputs to an espresso table.
- ~Find a way to make espresso buildable under mingw~