# litmus-tests-riscv

This litmus-tests-riscv repository contains litmus tests for the
RISC-V concurrency architecture, as used by members of the RISC-V
Memory Model Task Group during the architecture development.

Most of the litmus tests have been automatically generated using the
diy test generator from the diy tool suite <http://diy.inria.fr>;
others are hand-written.  The tests are in the form used both by that
tool suite (which also includes the litmus tool for running tests on
hardware and the herd tool for running tests in axiomatic models) and
by the rmem tool <http://www.cl.cam.ac.uk/users/pes20/rmem>, for
running tests in operational models.

The tests are distributed subject to the BSD 2-clause licence in
LICENCE.

This work has been partly supported by the REMS "Rigorous Engineering
of Mainstream Systems" EPSRC Programme Grant, EP/K008528/1, 2013-2020,
and by the ERC Advanced Grant ELVER, 789108.


People
======

These tests were produced by Shaked Flur and Luc Maranget.


Links
=====
* The [RISC-V Instruction Set Manual](https://github.com/riscv/riscv-isa-manual/) - see especially Sections 14 *RVWMO Memory Consistency Model*,
A *RVWMO Explanatory Material*, and
B *Formal Memory Model Specifications*, which contain text versions of axiomatic and operational models. 

* [The REMS project](https://www.cl.cam.ac.uk/~pes20/rems/)

* [Web interface of the rmem tool](https://www.cl.cam.ac.uk/~sf502/regressions/rmem/)

* [The diy tool suite (including diy, litmus, and herd)](http://diy.inria.fr/)

* [The herd RISC-V model](http://diy.inria.fr/cats7/riscv/cat.tar)




Files
=====

* LICENCE - Licence

* Makefile - For running hardware tests and comparing results (see below)

* riscv.cfg - A litmus configuration file (used by Makefile)

* README.md - This file

* model-results - Results from running the operational ("Flat", in rmem) and axiomatic ("Herd", in herd) models on the tests (see Makefile).

* tests - Each test is in a separate .litmus file.  Files in sub-folders
with a build.sh script were generated by running the script.  Files in
sub-folders with an X.conf file (and no build.sh script) were
generated by running 'diy7 -conf X.conf'.




Comparing the operational and axiomatic model results
=========================================

To see a comparison, using the mcompare tool from the diy tool suite,
of the operational (left) and axiomatic (right) models, do `make compare-models`.

At present seven tests show up as different between operational and axiomatic models.  These tests involve load-reserve and store-conditional to two different locations.  The herd tool assumes that different locations are on different cache lines, in which case the store-conditional must fail, while the rmem tool does not build in that assumption, so the store-conditional can either fail or succeed. 


To see all the details of a specific test do `make show-test TEST=<name>`
where `<name>` is a litmus test name (e.g. MP).

Running the tests on hardware
=============================

The litmus tool from the diy tool suite can generate a C program that
runs each of the litmus tests (incorporated into the C program as embedded
assembly) multiple times in an aggressive test harness, when executed on a target RISC-V machine. The
program collects the results (the final state for each run of each test) and prints them in a
format that can be processed by other tools from the diy tool suite.

To build the diy tool suite and the C program generated by litmus you
will need a system with OCaml (4.02.0 or greater), GNU make (3.81 or
greater, though this might also work with other versions of make) and gcc.  If
those are not available on the RISC-V machine on which you intend to
run the tests (the target machine), you will need another machine, not
necessarily RISC-V, on which they are available (the host machine).

The following sections explain how to run the litmus tests in three
scenarios: (**NOTE:** we have only tests the instructions in the third
scenario. If you need help please contact us.)

1. Building and running on the same (RISC-V) machine: the target
RISC-V machine has the diy tool suite, gcc and make installed.

2. Generating the program on a host machine (compile on the target):
the target RISC-V machine has gcc and make installed, but not the diy
tool suite.  Here you will need another machine that does have the diy
tool suite installed.

3. Cross compilation: the target RISC-V machine does not have gcc or
make installed. Here you will need another machine that does have the
diy tool suite, make and gcc that can cross-compile for RISC-V.

In each of the scenarios below, for some special RISC-V instructions
(e.g. "fence.tso"), `make` will check if your gcc supports those
instructions.  Litmus tests that include unsupported instruction will
be excluded from the tests.  After running the first `make ...`
command below, the file gcc.excl will list all the litmus test names
that are excluded.  The command `grep "#" gcc.excl` will show you
which instructions were detected as unsupported by gcc.

Building and running on the same (RISC-V) machine
-------------------------------------------------
Here we assume the litmus-tests-riscv repository is checked out on
the RISC-V machine you intend to run the tests on and the machine has
the diy tool suite, make and gcc installed.

To generate the C program, build it and run it do
`make run-hw-tests CORES=<n> [GCC=<gcc>] [-j <m>]`
where `<n>` is the number of hardware threads the machine can run
**(NOTE: this will probably take a few days to complete, though the first log file should be produced within a few minutes)**.  The
optional `GCC=<gcc>` allows you to specify a non-standard gcc path
(the default is `gcc`).  The optional `-j <m>` will use `<m>`
concurrent processes to speed up the compilation (might take a few
hours without this option).

Generating the program on a host machine (compile on the target)
----------------------------------------------------------------
Here we assume the litmus-tests-riscv repository is checked out on a
host machine that has the diy tool suite, and the target RISC-V
machine has gcc and make installed.

To generate the C program do `make hw-tests-src.tgz CORES=<n>` where
`<n>` is the number of hardware threads the target machine can run.

Copy hw-tests-src.tgz to the target RISC-V machine and then do (on the
target machine):

```
$ tar -zxf hw-tests-src.tgz
$ cd hw-tests-src
$ make [GCC=<gcc>] [-j <m>]
...
$ ./run.sh
```

where `<gcc>` and `<m>` are as described in the previous section.
**(NOTE: run.sh will probably take a few days to complete!)**

When the tests are finished run.sh will touch the file "done".  Copy
all the `run.*.log` files back to the host, to the directory
`litmus-tests-riscv/hw-tests` (you will need to create it first), and
do (on the host) `make merge-hw-tests`.

Cross compilation
-----------------
Here we assume the litmus-tests-riscv repository is checked out on a
host machine that has the diy tool suite, make and gcc, that can
cross-compile for RISC-V, installed.

To generate the C program and build it do
`make hw-tests CORES=<n> [GCC=<gcc>] [-j <m>]`
where `<n>`, `<gcc>` and `<m>` are as in the previous sections. Note
that `<gcc>` should cross-compile for RISC-V.  This will produce the
directory hw-tests with two files, run.exe and run.sh. Copy those
files to the target machine, and run run.sh (on the target machine)
**(NOTE: run.sh will probably take a few days to complete)**.  When
the tests are finished run.sh will touch the file "done".  Copy all
the `run.*.log` files back to the host, to the directory hw-tests, and
do (on the host) `make merge-hw-tests`.

Comparing hardware results with the models
------------------------------------------
To compare the hardware results with the Flat model do `make
compare-hw-flat`, and to compare with the Herd model do `make
compare-hw-herd`.  This will show you the output of mcompare, from the
diy tool suite, running on the hardware results (left) and model
results (right).

If you see at the end of the output the line "!!! Warning negative
differences in: [...]", the hardware has exhibited some behaviour that
the model does not allow.  This indicates that the hardware is
inconsistent with the RISC-V RVWMO memory model.

You are most likely to see the line "!!! Warning positive differences
in: [...]".  This indicates that the model allows for more behaviour
than was exhibited by the hardware.  This is to be expected as
implementations are unlikely to be as relaxed as the model permits
them to be.  In addition, it is possible that the test harness just
did not trigger the right conditions for a certain behaviour to be
exhibited by the hardware.
