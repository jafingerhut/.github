<!--
SPDX-FileCopyrightText: 2026 The P4 Language Consortium

SPDX-License-Identifier: Apache-2.0
-->

# Introduction

One or more published p4c back ends produce C source files that are
intended to be compiled and loaded into the Linux kernel via the EBPF
feature, at least [1], [2] and perhaps others.

[1] https://github.com/p4lang/p4c/blob/main/backends/ebpf/README.md

[2] https://github.com/p4lang/p4c/blob/main/backends/tc/README.md

While there are a few examples of companies releasing binaries of
Linux driver code for their hardware and not releasing the source
code, we leave it to their legal teams to navigate any of the
intricacies that may be involved in that endeavor.

Reference [3] is a good introduction to software licensing questions
related to using EBPF.

[3] https://ebpf.io/blog/ebpf-licensing-guide/

While it is _possible_ to write restricted EBPF programs that are
loaded into the Linux kernel that can be distributed under a non-GPL
license, these are the exception, not the rule.

Do companies using EBPF distribute these programs under GPL 2.0?  Yes,
they commonly do.  The EBPF code loaded and run in the Linux kernel is
the "data plane" part of the system.  The "control plane" part of the
system runs in user space applications on Linux, and these can be
distributed under other licenses, including proprietary licenses.  In
most production EBPF programs, the control plane code is far larger
and more complex then the data plane code.  The data plane code by
itself is unlikely to cause any significant loss of competitive
advantage to publicize it.

There are (at least) these categories of source files:

(a) C source files that are intended to be compiled and loaded into
    the kernel via EBPF.

(b) C (or other language) source files that are intended to be
    compiled (or interpreted) and executed in user space, that
    communicate with the EBPF program via Linux system calls.

(c) C header files that are intended to be included only from files of
    category (a).

(d) C header files that are intended to be included either from files
    of category (a), or from category (b).

For category (a), most of those files are auto-generated as the output
of the `p4c-ebpf` or `p4c-pna-p4tc` P4 compilers.

Thus our plan is to release all of the following kinds of source files
under the GPL v2.0 license, the same as the Linux kernel:



+ header files included from files described in the previous item

Note that this list of files appears to consist of _only_ 2 source
files in the entire p4c repository.  Most of the source files related
to EBPF are part of a p4c back end that uses EBPF, which can be
Apache-2.0 licensed, or they are control plane source files used for
testing the EBPF programs loaded into the kernel.

If any subtle questions arise as to whether a particular P4 compiler
back end, e.g. `p4c-ebpf`, must thus be released under GPL v2.0, we
will raise these questions to the P4 Technical Steering Team for their
advice.

We expect that for P4 compiler back ends that do not produce source
code for use with EBPF, this question will never arise.

Note: When analyzing the source files described in this section more
carefully, we will take special care to ensure that they are not files
used to build a P4 compiler executable, and update this note when that
task is complete with our findings.


## Version of p4c source code used in the examples below

```
$ p4c --version
/usr/local/share/p4c/p4c_src/barefoot.py:1147: SyntaxWarning: 'return' in a 'finally' block
  return rc
p4c 1.2.5.18 (SHA: 64ff00a48f BUILD: Release)
```

This was the most recent version as of a date during 2026-Oct when the
commands below were executed.


## Example of output of p4c-ebpf

The following assumes that you have installed the `p4c-ebpf` program
on your system.

```
git clone https://github.com/p4lang/p4c
cd p4c/testdata/p4_16_samples
p4c-ebpf action_call_ebpf.p4 -o action_call_ebpf.c
```

The last command creates these two C source files:

+ `action_call_ebpf.c`
+ `action_call_ebpf.h`

The `.c` file contains this line:
```
char _license[] SEC("license") = "GPL";
```

That line compiles into a part of the binary file.  When you attempt
to load this binary EBPF file into the kernel using a system call, if
the EBPF program makes any calls to GPL-only functions within the
kernel, the system call will fail the attempt to load it into the
kernel.

Here are the header files included by this generated program:

```
$ grep include action_call_ebpf.[ch]
action_call_ebpf.c:#include "action_call_ebpf.h"
action_call_ebpf.c:#include "ebpf_kernel.h"
action_call_ebpf.h:#include "ebpf_kernel.h"
```

Some of the included files above are part of the p4c repository
https://github.com/p4lang/p4c

Here are where all of the included files come from, including nested
includes:

+ `action_call_ebpf.c` (generated) includes:
  + `action_call_ebpf.h` (generated) includes:
    + `ebpf_kernel.h` - part of p4c repo at `backends/ebpf/runtime/ebpf_kernel.h`, includes:
      + `<stddef.h>` - part of Linux /usr/include files, not p4c
      + `<sys/types.h>` - part of Linux /usr/include files, not p4c
      + `ebpf_common.h` - part of p4c repo at `backends/ebpf/runtime/ebpf_common.h`, includes:
        + `<stddef.h>` - part of Linux /usr/include files, not p4c
        + `<sys/types.h>` - part of Linux /usr/include files, not p4c
      + `install/libbpf/include/bpf/bpf_endian.h` - part of libbpf repo
      + `contrib/bpftool/include/uapi/linux/bpf.h` - part of bpftool repo
      + `contrib/bpftool/include/uapi/linux/pkt_cls.h` - part of bpftool repo?
      + `install/libbpf/include/bpf/bpf_helpers.h` - where from?


## Example of output of p4c-pna-p4tc

The following assumes that you have installed the `p4c-ebpf` program
on your system.

```
git clone https://github.com/p4lang/p4c
cd p4c/testdata/p4tc_samples
mkdir tmp
p4c-pna-p4tc global_action_example_01.p4 -o tmp
```

Two of the output files written to the directory `tmp` have the same
GPL license line that `p4c-ebpf` uses, because those programs are
intended to be compiled and loaded into the Linux kernel.

```
$ cd tmp
$ grep GPL *
global_action_example_01_control_blocks.c:char _license[] SEC("license") = "GPL";
global_action_example_01_parser.c:char _license[] SEC("license") = "GPL";
```

Here are the header files included by those programs:

```
$ grep include *
global_action_example_01_control_blocks.c:#include "global_action_example_01_parser.h"
global_action_example_01_parser.c:#include "global_action_example_01_parser.h"
global_action_example_01_parser.h:#include "ebpf_kernel.h"
global_action_example_01_parser.h:#include <stdbool.h>
global_action_example_01_parser.h:#include <linux/if_ether.h>
global_action_example_01_parser.h:#include "pna.h"
```

Some of the included files above are part of the p4c repository
https://github.com/p4lang/p4c

Here are where all of the included files come from, including nested
includes:

+ `global_action_example_01_control_block.c` (generated) includes:
+ `global_action_example_01_parser.c` (generated) includes:
  + `global_action_example_01_parser.h` (generated) includes:
    + `ebpf_kernel.h` - see previous section for details of what it includes
    + `<stdbool.h>` - part of Linux /usr/include files, not p4c
    + `<linux/if_ether.h>` - part of Linux /usr/include files, not p4c
    + `pna.h` - part of p4c repo at `backends/tc/runtime/include/pna.h`, includes:
      + `<stdbool.h>` - part of Linux /usr/include files, not p4c
      
