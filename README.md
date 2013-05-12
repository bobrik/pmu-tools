
pmu tools is a collection of tools for profile collection and performance
analysis on Intel CPUs on top of Linux perf.

Current features:

- A wrapper to "perf" that provides a full core event list for 
common Intel CPUs. This allows to use all the Intel events,
not just the builtin events of perf.
- Support for Intel "offcore" events on older systems that
do not have support for  this in the Intel. Offcore events
allow to profile the location of a memory access outside the
CPU's caches.
- Implement a workaround for some issues with offcore events 
on Sandy Bridge EP (Intel Xeon E5 first generation)
This is automatically enabled for the respective events, and also
available as a standalone program.
- Some utility programs to access pci space or msrs on
the command line
- A utility program to program the PMU directly from user space
(pmumon.py) for counting. This is mainly useful for testing
and experimental purposes.
- A library for self profiling with Linux since Linux 3.3
Note for self-profiling on older kernels you can use
[simple-pmu] http://halobates.de/simple-pmu

Usage:

Check out the repository

ocperf:

Copy all the files to a directory (or run from the source)
Run ocperf.py from that directory
ocperf.py searches for the data files in the same directory
as the binary

ocperf.py list
List all the events perf and ocperf supports on the current CPU

	ocperf.py stat -e eventname ... 

	ocperf.py record -e eventname ...

	ocperf.py report --stdio

The translation back from the raw event name for report is only supported
for the --stdio mode. The interactive browser can be also used, but will
display raw perf event names.

When a older kernel is used with offcore events,
that does not support offcore events natively, ocperf has to run
as root and only one such profiling can be active on a machine.

tester provides a simple test suite.

The latego.py, msr.py, pci.py modules can be also used as standalone programs
to enable the offcore workaround, change MSRs or change PCI config space respectively.

self: 

Self is a simple library to support self-profiling of programs, that is programs
that measure their own execution.

cd self
make

Read the documentation. 

[rdpmc] (http://github.com/andikleen/pmu-tools/self/rdpmc.html) is the basic facility to access raw counters.

ocperf can be used to generate raw perf numbers for your CPU to pass to rdpmc_open()
	ocperf list | less
<look for intended event>
	DIRECT_MSR=1 ./ocperf.py stat -e eventname true
<look for perf stat -e rXXXX in output>
XXX is the needed event number in hex. Note that self does not support offcore or uncore events.
Also the event numbers are CPU specific, so you may need a /proc/cpuinfo model check.

	#include "rdpmc.h"

	struct rdpmc_ctx ctx;
	unsigned long long start, end;

	if (rdpmc_open(XXX, &ctx) < 0) ... error ...
	start = rdpmc_read(&ctx);
	... your workload ...
	end = rdpmc_read(&ctx);

[measure] (http://github.com/andikleen/pmu-tools/self/measure.html) supports event group profiling.
[interrupts] (http://github.com/andikleen/pmu-tools/self/interrupts.html) provides functions for a common use
case of filtering out context switches and interrupts from micro benchmarks. These only work 
on Intel Ivy and Sandy Bridge CPUs.

Link the object files and include the header files in your program

/sys/devices/cpu/rdpmc must be 1.

rtest.c and test2.c provide examples. http://halobates.de/modern-pmus-yokohama.pdf provides some additional
information on cycle counting.

Andi Kleen
