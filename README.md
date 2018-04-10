go6502
======

_This project is currently under construction and is changing frequently._

go6502 is a collection of go packages that emulate a 6502 or 65C02 CPU. It
includes a CPU emulator, a cross-assembler, a disassembler, a debugger, and a
host that wraps them all together.

The interactive go6502 console application is in the root directory provides
access to all of these features.

# Tutorial

Let's start by considering the go6502 `sample.cmd` script:

```
load monitor $F800
assemble sample
load sample
set PC START
d .
```

We'll describe what each of these commands does in greater detail later, but
for now know that they do the following things:
1. Load the `monitor.bin` binary file at memory address `F800`.
2. Assemble the `sample.asm` file using the go6502 cross-assembler, generating
   a `sample.bin` binary file and a `sample.map` source map file.
3. Load the `sample.bin` binary file at the origin address specified by the
   `sample.asm` file.
4. Set the program counter to the `START` address, which was exported by
   `sample.asm`.
5. Disassemble the first few lines of machine code starting from the program
   counter address.

To run this script, build the go6502 application and type the following on the
command line:

```
go6502 sample.cmd
```

You should then see:

```
Loaded 'monitor.bin' to $F800..$FFFF
Assembled 'sample.asm' to 'sample.bin'.
Loaded 'sample.bin' to $1000..$10FF
Loaded 'sample.map' source map
Register PC set to $1000.
1000-   A2 EE       LDX   #$EE
1002-   48          PHA
1003-   20 18 10    JSR   $1018
1006-   20 1B 10    JSR   $101B
1009-   20 35 10    JSR   $1035
100C-   20 45 10    JSR   $1045
100F-   F0 06       BEQ   $1017
1011-   A0 3B       LDY   #$3B
1013-   A9 10       LDA   #$10
1015-   A2 55       LDX   #$55

1000-   A2 EE       LDX   #$EE      A=00 X=00 Y=00 PS=[------] SP=FF PC=1000 C=0
*
```

The output shows the result of go6502 running each command in the sample
script. Once the script is finished running, go6502 enters interactive mode
and provides a `*` prompt for further input.

Just before the prompt is a line starting with `1000-`. This line displays the
disassembly of the instruction at the current program counter address and the
state of the CPU registers. The `C` value indicates the number of CPU cycles
that have elapsed since the application started.

## Getting help

Let's enter our first interactive command. Type `help` to see a list of all
commands.

```
* help
go6502 commands:
    annotate         Annotate an address
    assemble         Assemble a file and save the binary
    breakpoint       Breakpoint commands
    databreakpoint   Data breakpoint commands
    disassemble      Disassemble code
    evaluate         Evaluate an expression
    exports          List exported addresses
    load             Load a binary file
    memory           Memory commands
    quit             Quit the program
    registers        Display register contents
    run              Run the CPU
    set              Set a host setting
    step             Step the debugger
```

To get further help about a command, type `help` followed by the command name.
In some cases, you will be shown a list of subcommands that must be used with
the command. Let's try `help step`.

```
* help step
Step commands:
    in               Step in to routine
    over             Step over a routine
```

This output tells you that the `step` command requires an `in` or `over`
subcommand.  For example, to step the debugger into a routine, type `step in`.

## Stepping the CPU

Let's use one of the `step` subcommands to step the CPU by a single
instruction. Type `step in`.

```
1000-   A2 EE       LDX   #$EE      A=00 X=00 Y=00 PS=[------] SP=FF PC=1000 C=0
* step in
1002-   A9 05       LDA   #$05      A=00 X=EE Y=00 PS=[N-----] SP=FF PC=1002 C=2
*
```

By typing `step in`, you told the emulated CPU to execute the `LDX #$EE`
instruction at address `1000`. This advanced the program counter to `1002`,
loaded the value `EE` into the X register, and advanced the CPU cycle counter
by 2 cycles.

Each time go6502 advances the program counter interactively, it disassembles
the instruction to be executed next (i.e., the instruction at the current
program counter address). It also displays the current values of the CPU
registers and cycle counter.

Many go6502 commands have aliases. The alias for the `step in` command is
`si`. Now, type `si 4` to step into the next 4 instructions:

```
1002-   A9 05       LDA   #$05      A=00 X=EE Y=00 PS=[N-----] SP=FF PC=1002 C=2
* si 4
1004-   20 19 10    JSR   $1019     A=05 X=EE Y=00 PS=[------] SP=FF PC=1004 C=4
1019-   A9 FF       LDA   #$FF      A=05 X=EE Y=00 PS=[------] SP=FD PC=1019 C=10
101B-   60          RTS             A=FF X=EE Y=00 PS=[N-----] SP=FD PC=101B C=12
1007-   20 1C 10    JSR   $101C     A=FF X=EE Y=00 PS=[N-----] SP=FF PC=1007 C=18
*
```

This output shows that the CPU has stepped into the next 4 instructions
starting at address `1002`.  Each executed instruction is disassembled and
displayed along with the CPU's register values at the start of each
instruction. In total, 18 CPU cycles have elapsed, and the program counter
ends at address `1007`.  The instruction at `1007` is waiting to be executed.

Note that the `step in` command stepped _into_ the `JSR $1019` subroutine call
rather than stepping _over_ it.  If you weren't interested in stepping through
all the code inside the subroutine, you could have used the `step over`
command instead. This would have caused the debugger to invisibly execute all
instructions inside the subroutine, returning control to the application only
after the `RTS` instruction has executed.

Since the CPU is about to execute another `JSR` instruction, let's try the
`step over` command (or `s` for short).

```
1007-   20 1C 10    JSR   $101C     A=FF X=EE Y=00 PS=[N-----] SP=FF PC=1007 C=18
* s
100A-   20 36 10    JSR   $1036     A=00 X=EE Y=00 PS=[-Z----] SP=FF PC=100A C=70
*
```

After stepping over the `JSR` call at address `1007`, all of the instructions
inside the subroutine at `101C` have been executed, and control has returned
to go6502 at address `100A` after 52 CPU cycles have elapsed.

## Shortcut: Hit Enter!

One shortcut you will probably use frequently is entering a blank line at
the command prompt. This causes go6502 to repeat the previously entered
command.

Let's try hitting Enter twice to repeat the `step over` command two more
times.

```
100A-   20 36 10    JSR   $1036     A=00 X=EE Y=00 PS=[-Z----] SP=FF PC=100A C=70
*
100D-   20 46 10    JSR   $1046     A=00 X=00 Y=00 PS=[-Z----] SP=FF PC=100D C=103
*
1010-   F0 06       BEQ   $1018     A=00 X=00 Y=00 PS=[-Z----] SP=FF PC=1010 C=136
*
```

As you can see, go6502 has stepped over two more `JSR` instructions,
elapsing another 66 CPU cycles and leaving the program counter at `1010`.

## Disassembling code

Now let's disassemble code at the current program counter address to get a
preview of the code about to be executed. To do this, you can either type
the `disassemble` command or the its shortcut, `d`.

```
* d .
1010-   F0 06       BEQ   $1018
1012-   A0 3B       LDY   #$3B
1014-   A9 10       LDA   #$10
1016-   A2 56       LDX   #$56
1018-   00          BRK
1019-   A9 FF       LDA   #$FF
101B-   60          RTS
101C-   A9 20       LDA   #$20
101E-   A5 20       LDA   $20
1020-   B5 20       LDA   $20,X
*
```

The `.` is shorthand for the current program counter address.  You may also
pass an address or mathematical expression to disassemble code starting at any
address:

```
* d START+2
1002-   A9 05       LDA   #$05
1004-   20 19 10    JSR   $1019
1007-   20 1C 10    JSR   $101C
100A-   20 36 10    JSR   $1036
100D-   20 46 10    JSR   $1046
1010-   F0 06       BEQ   $1018
1012-   A0 3B       LDY   #$3B
1014-   A9 10       LDA   #$10
1016-   A2 56       LDX   #$56
1018-   00          BRK
*
```

By default, go6502 disassembles 10 instructions, but you can disassemble a
different number of instructions by specifying a second argument:

```
* d . 3
1010-   F0 06       BEQ   $1018    
1012-   A0 3B       LDY   #$3B     
1014-   A9 10       LDA   #$10     
*
```

If you hit Enter after using a disassemble command, go6502 will continue
disassembly from where it left off.

```
*
1016-   A2 56       LDX   #$56     
1018-   00          BRK            
1019-   A9 FF       LDA   #$FF
*
```

If you don't like the number of instructions that go6502 disassembles by
default, you can change it with the `set` command:

```
* set DisasmLinesToDisplay 20
```


_To be continued..._
