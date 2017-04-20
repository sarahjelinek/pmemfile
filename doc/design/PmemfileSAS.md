V.4

NVML based Linux SW Solution to Provide NVM Programming Model (NPM)
Access to Persistent Memory via POSIX File System Interfaces.

 {#section .ListParagraph .TOCHeader}

Document Revision History
=========================

+--------------------------------------+--------------------------------------+
| Version                              | Document Changes                     |
+======================================+======================================+
| V.3 3/23/16                          | Initial document with completed high |
|                                      | level interfaces, use cases, and     |
|                                      | theory of operations.                |
+--------------------------------------+--------------------------------------+
| V.3 5/17/16                          | Made more changes to on media        |
|                                      | structures.                          |
+--------------------------------------+--------------------------------------+
| V.3 5/26/16                          | Additional modifications to on media |
|                                      | format.                              |
+--------------------------------------+--------------------------------------+
| V.3 5/31/16                          | Additional changes and additions to  |
|                                      | on media format section. Modified    |
|                                      | Pmemfile\_ Interception section to   |
|                                      | add more detail to the architecture  |
|                                      | chosen.                              |
+--------------------------------------+--------------------------------------+
| V.4 6/6/16                           | Completed initial persistent data    |
|                                      | structure definitions. Added open    |
|                                      | items section.                       |
+--------------------------------------+--------------------------------------+
| V.4 6/7/16                           | Finished persistent data structure   |
|                                      | design. Added file structure to      |
|                                      | volatile section.                    |
+--------------------------------------+--------------------------------------+
| V.4 6/8/16                           | More updates                         |
+--------------------------------------+--------------------------------------+
| V.4 6/14/16                          | Changes to the volatile and          |
|                                      | persistent structures. Added more    |
|                                      | details to the Pmemfile\_ Intercept  |
|                                      | Library. Added diagram to show       |
|                                      | dependency resolution for lseek +    |
|                                      | open ().                             |
+--------------------------------------+--------------------------------------+
| V.4 10/3/16                          | Finalized volatile and persistent    |
|                                      | data structure layout. Modified      |
|                                      | interception library with new        |
|                                      | architecture.                        |
+--------------------------------------+--------------------------------------+
| V.4 10/17/16                         | Modified interception layer with new |
|                                      | architecture.                        |
+--------------------------------------+--------------------------------------+
| V.4 10/26/16                         | Changes based on internal review     |
|                                      | with pmemfile team.                  |
+--------------------------------------+--------------------------------------+
| V.4 11/8/16                          | More changes based on internal       |
|                                      | review with pmemfile team            |
+--------------------------------------+--------------------------------------+
| V.4 11/17/16                         | Finished persistent data structure   |
|                                      | section.                             |
+--------------------------------------+--------------------------------------+
| V.4 11/28/16                         | Updated based on review by pmemfile  |
|                                      | team                                 |
+--------------------------------------+--------------------------------------+
| V.4 1/4/17                           | Updated persistent data structures   |
+--------------------------------------+--------------------------------------+
| V.4 2/21/17                          | Added more detail to Section 7,      |
|                                      | design                               |
+--------------------------------------+--------------------------------------+
| V.4 3/14/17                          | Added more detail to valid states    |
|                                      | for persistent data structure        |
|                                      | members                              |
+--------------------------------------+--------------------------------------+
| V.4 3/16/17                          | Completed man pages and added links  |
|                                      | to document                          |
+--------------------------------------+--------------------------------------+
| V.4 3/21/17                          | Added Section 11.4, Consistency      |
|                                      | Checking                             |
+--------------------------------------+--------------------------------------+
| V.4 4/5/17                           | Modified High Level Architecture     |
|                                      | Diagram to be more correct and       |
|                                      | specific                             |
+--------------------------------------+--------------------------------------+
| V.4 4/10/17                          | Cleaned up for github publication.   |
+--------------------------------------+--------------------------------------+
| V.4 4/19/17                          | More cleanup for github publication. |
+--------------------------------------+--------------------------------------+

# Document Overview #

This document describes the SW Architecture and High Level Design of the
Pmemfile PMEM project.

# Terminology #

This section outlines the major terminology and glossary items utilized
throughout this architect specification. See the figure below for a
graphical re-presentation of the terminology introduced in this
terminology table.

+--------------------------------------+--------------------------------------+
| **Terminology, Glossary Term**       | **Description**                      |
+======================================+======================================+
| Persistent Memory, PMEM, CR-DIMM,    | Byte addressable persistent memory.  |
| NVDIMM                               |                                      |
+--------------------------------------+--------------------------------------+
| NVM Programming Model(NPM)           | SNIA model for software supporting   |
|                                      | non-volatile memory.                 |
|                                      | <http://www.snia.org/sites/default/f |
|                                      | iles/NVMProgrammingModel_v1.pdf>     |
+--------------------------------------+--------------------------------------+
| POSIX                                | A set of formal descriptions that    |
|                                      | provide a standard for the design of |
|                                      | file systems.                        |
+--------------------------------------+--------------------------------------+
| System Call                          | Interface between user space and     |
|                                      | kernel space                         |
+--------------------------------------+--------------------------------------+
| (g)libc                              | Standard POSIX library interface for |
|                                      | Unix operating systems               |
+--------------------------------------+--------------------------------------+

What is Pmemfile?

Pmemfile is a PMEM (<https://github.com/pmem>) project which provides a
POSIX –like file system interface to consumers while providing NVM
programming model access to persistent memory. It is comprised of three
parts: a) The preload library b) System Call Interception Library and c)
The POSIX core library.

1.  <span id="_Toc452012200" class="anchor"><span id="_Toc480386177" class="anchor"></span></span>Linux High Level Architectural Components
    =======================================================================================================================================

    1.  <span id="_Toc452012201" class="anchor"><span id="_Toc480386178" class="anchor"></span></span>High Level Linux File Interface
        -----------------------------------------------------------------------------------------------------------------------------

Applications in Linux use a high level interface to interact with files
on a device. This interface is provided by the glibc library. The term
"libc" is commonly used as a shorthand for the "standard C library", a
library of standard functions that can be used by all C programs (and
sometimes by programs in other languages). The standard functions
provided by glibc include functions for file and file system access.

glibc is a user space library. However, since file management happens in
the kernel it must communicate with the kernel to complete file
operations. The interface glibc uses for communication with the kernel
is the system call interface. A system call is the programmatic way in
which a computer program requests a service from the kernel of the
operating system it is executed on.

The following is a diagram which shows the high level layers from an
application to the Linux kernel via glibc and the system call Interface.

![](media/image2.jpg){width="3.8541666666666665in"
height="2.6041666666666665in"}

<span id="_Ref453343698" class="anchor"></span>Figure ‑ Linux Operating
Environment

At the top is the user, or application, space. This is where the user
applications are executed. Below the user space is the Linux kernel
space.

1.  Pmemfile SW Architecture
    ========================

    1.  Pmemfile Architectural Components
        ---------------------------------

Pmemfile will provide three architectural components: the Pmemfile
preload library, system call interception library as well as the
Pmemfile core library. The diagram shown in Figure 7‑1 Pmemfile
Architectural Components shows the components that will be delivered as
part of Pmemfile as well as the high level interaction of these with
other components on the system.

![](media/image3.png){width="6.5in" height="3.65625in"}

<span id="_Ref471735246" class="anchor"><span id="_Toc452012206"
class="anchor"><span id="_Ref465927550"
class="anchor"></span></span></span>Figure ‑ Pmemfile Architectural
Components

### Pmemfile Preload Library (libpmemfile.so)

The libpmemfile library implements the system call hook functions for
the Pmemfile project. It uses the interface exported by the system call
intercept library described in **<span
style="font-variant:small-caps;">*System Call Interception
(libsyscall\_intercept.so)*</span>**. This library must be preloaded
using the LD\_PRELOAD linker directive when an application is started.
When this library is preloaded into the application process space it
will do the following:

-   Create or open the Pmemfile pool

-   Setup of hook functions for system call interception

-   Pmemfile file descriptor management

    1.  #### Hook functions

The Pmemfile hook functions are wrappers that contain the logic for
routing the system calls as defined in **<span
style="font-variant:small-caps;">*Supported System Call
Interfaces.*</span>** If the file is a pmem-resident file the wrapper
calls the appropriate Pmemfile POSIX library function. If not, it passes
it through to the standard system call.

#### Pmemfile File Descriptor Management

During initialization a group of file descriptors is created from the
kernel using ‘/dev/null’ as the device. The pool of file descriptors are
then mapped with a Pmemfile file as pmem resident files are created.

### <span id="_Ref466553399" class="anchor"><span id="_Ref466554555" class="anchor"><span id="_Ref466878510" class="anchor"><span id="_Ref466878962" class="anchor"><span id="_Ref474741906" class="anchor"><span id="_Toc480386182" class="anchor"></span></span></span></span></span></span>System Call Interception (libsyscall\_intercept.so)

The Pmemfile interception library provides three major areas of
functionality: 1) disassembling of glibc 2) building a trampoline table
and 3) hot patching the file operation system calls in glibc. The System
Call interception library is a public interface and can be used as a
standalone component.

#### System Call Interception layer requirements and assumptions:

-   Use of the system call interception layer along with the Pmemfile
    preload library will not require a re-compilation or re-link for
    the application.

<!-- -->

-   The system call interception layer will intercept file operation
    system calls prior to crossing the kernel boundary.

    -   The system call interception layer will **only** cross the
        kernel boundary when absolutely required. It is never crosses
        the boundary to operate on the Pmemfile files. It is only used
        in preparation for access to the files. Currently, the following
        use cases require the use of the Linux kernel:

        -   Getting a file descriptor for the Pmemfile file. We do this
            by opening /dev/null.

        -   Path name resolution

<!-- -->

-   With Pmemfile each system call that is intercepted is made by glibc.

    -   Any applications which call system calls directly will not be
        managed with the System Call interception layer.

-   No other application or library tries to hot patch glibc in the
    same process.

-   glibc must be initialized in the process address space prior to the
    System Call interception library.

    1.  #### System Call Interception Library Exported Interface

The Pmemfile interception library exports one interface:

int (\*intercept\_hook\_point)(long syscall\_number,

long arg0, long arg1,

long arg2, long arg3,

long arg4, long arg5,

long \*result);

The consumer of this library has to define the hook functions for each
of the system calls it wants to be intercepted.

All Linux system calls can take up to six arguments. Each system call is
defined by a number. The *intercept\_hook\_point* interface provides a
way to define a function which is called in lieu of the specified system
call via the *syscall\_number*.

#### System Interception Library Disassembling

Linux executable files, relocatable object files, core files and shared
objects are all defined using the Executable and Linking Format (ELF).
See man(5) elf.

The ELF format enables us to find locations of functions, data
structures and other data types within a shared object such as glibc.

libmemfile disassembles glibc to find all the locations of the system
calls Pmemfile supports as specified in **<span
style="font-variant:small-caps;">*Supported System Call
Interfaces*</span>**. The Linux System Call interface is the fundamental
interface between an application and the Linux kernel. Pmemfile does not
use the kernel to do file operations however to maintain application
compatibility Pmemfile intercepts at the system call layer, which in
Linux is the supported and stable API.

Pmemfile uses the opensource project, Capstone, to disassemble glibc.
The sections we must look in to find all instances of system calls are
the ‘.text’, ‘.symtab’ and the ‘.dynsym’ sections of the glibc library.
The sections are disassembled, starting with .dynsym and .symtab. These
sections hold the dynamic linking symbol table and the symbol table for
the library, respectively. Some of the symbols found in these sections
are functions in .text section, which means they are jump entry points.
The .text section is then disassembled in full and the addresses of all
system call instructions are stored, together with information about the
instructions prior to and following the system call. A lookup table of
all addresses which will be jump destinations is generated to help
determine later whether an instruction is suitable for being overwritten
-- of course, if an instruction is a jump destination it cannot be
merged with the preceding instruction to create a new larger one.

Every time a system call instruction is found the next and previous
padding bytes between routines are checked to find potential space for
an extra jump instruction.

#### Pmemfile glibc Patching

Patching glibc is the process of modifying an address in glibc to point
to a new address. The instruction at this new address is the one that is
followed after the patch is applied. Pmemfile patching happens only
after glibc is loaded in a process and does not affect the system wide
glibc.

##### Memory Address Patching Considerations

To patch a system call located in glibc the following must be
considered:

1)  How do we ensure we never patch to an unknown or unexpected location
    within the process address space.

2)  What is the distance between our patch function addresses and glibc
    in the process address space?

3)  How many bytes are available to use for hot patching the code to
    insert the patch function.

4)  How many bytes are over-written during the process of patching?

5)  How do we patch the minimal number of bytes in glibc to achieve our
    goals

6)  How do we go back to the original system call if we are not
    processing a file which is pmem resident.

7)  How do we get back to glibc to complete the system call return?

    1.  ###### Linux Address Space Layout

The first thing we must consider is the Linux Address Space Layout. It’s
important to understand this and to understand any restrictions or
definitions that would affect how we ensure we never patch to an unknown
or unexpected address within the process.

X86-64 provides a 64-bit address space for a process. Theoretically that
is. In Linux there is a virtual hole in the middle of the address space
so there is effectively 48 bits of available address. This is due to the
fact that Intel processors support only 48-bit virtual address space.
This means that any two libraries, or functions between them could be
greater than 2GB apart. As a result we cannot simply use the relative
address jump which only supports 32 bits which provides up to 2GB
distance.

The linux x86-64 ABI explicitly states:

*It cannot be assumed that a function is within 2GB in general.
Therefore, it is necessary to explicitly calculate the desired address
reaching the whole 64-bit address space.*

Linux ABI Document, page 45:

<http://refspecs.linuxfoundation.org/elf/x86-64-abi-0.99.pdf>

###### Trampolining

Trampolining is a way to have an interim jump function that forwards on
to the final jump location. In Pmemfile a trampoline table is created
and mmaped to address that is within 2GB of glibc. The trampoline table
contains wrapper functions that manage the routing of the system call to
the appropriate component. Use of the Pmemfile trampoline table allows
us to use the minimum number of bytes to patch glibc. This makes it
easier to find adequate bytes to use for patching as well as making it
less error prone. Once the wrapper function takes over the jumps can be
a larger size since we are not constrained by the number of bytes we can
overwrite.

The wrapper functions contain:

-   Code to route file operation to correct component as per the hook
    functions defined

-   Code which has return address into glibc for completion of the file
    operations

-   Encapsulates the hook functions that were created in the preload
    phase

    1.  #### Pmemfile Interception Layer Mechanics

The steps to set up the Pmemfile interception layer are as follows:

-   find the system calls in glibc

    -   disassemble glibc and locate

-   allocate trampoline table

-   create patch wrappers

-   activate patches

-   Wrapper functions are created and the trampoline table is set up and
    glibc is disassembled.

-   

<!-- -->

-   A call is made from the application to a glibc file operation

-   The system call within the glibc function has a patch to jump to a
    location in the trampoline table.

-   A jump is made from trampoline table to wrapper function

-   Call to the C hook function (if pmem-resident file).

-   Return to glibc to complete the system call.

The diagram below shows the high level operation of the Pmemfile system
call library.

![](media/image4.PNG){width="6.5in" height="5.1305555555555555in"}

<span style="font-variant:small-caps;">*Error! Reference source not
found.*</span>

### Pmemfile POSIX Library (libpmemfile-posix.so)

The Pmemfile project provides a core library which encapsulates the file
system operations in user space. For Linux the core library contains a
POSIX interface library, libpmemfile-posix.so. For future development
the core library component can also contain other interface support
libraries.

The Pmemfile POSIX library provides a POSIX like interface that manages
the Pmemfile file system. This library is a standalone component which
can be used in conjunction with other NVML libraries and with or without
the Pmemfile system call interception library. The Pmemfile POSIX
library is the component of Pmemfile which provides Direct Access to the
NVM media. It also provides consistency and correctness via the
transactional interfaces provided the NVM libpmemobj library.

#### Pmemfile POSIX Library Requirements

> • Must be usable in conjunction with other NVML libraries.

-   Must provide an interface that is easy for application developers to
    use if they choose to modify applications rather than use the
    interception functionality. We have chosen, for Linux, to use POSIX
    as the guiding principle in designing the interfaces.

-   The Pmemfile file system must look and behave as a ‘normal’ Linux
    file system providing the interfaces and behaviors defined in the
    **<span style="font-variant:small-caps;">*libpmemfile-posix.3
    manpage*</span>**. The Pmemfile POSIX library is the component that
    provides the ‘look’ and ‘management’ of the Pmemfile file system.

-   Must provide the file system functionality in user space.

-   Must provide at least basic Linux standard file system
    consistency guarantees.

-   Must provide better performance, for certain workloads, than a DAX
    enabled file system.

    1.  #### Pmemfile POSIX Library Interfaces

An application developer can incorporate the Pmemfile POSIX library
directly into an application. Rather than having to learn a new syntax
the Pmemfile POSIX library provides, as much as possible, similar naming
and similar function arguments as their POSIX function counterparts.

##### Pmemfile POSIX Library Interface Naming

The Pmemfile core library interfaces are named using the standard POSIX
naming with ‘Pmemfile’ as the prefix. For example:

open() /pmemfile\_open()

close()/pmemfile\_close()

##### Exceptions to POSIX 

In general a POSIX file operation expects a path or a file descriptor as
the first argument. And when a file is being created or opened a file
descriptor is returned. The Pmemfile POSIX library has some noted
exceptions to this pattern.

###### Pmemfile POSIX Library Interface Signatures

In all cases the first argument of the libpmemfile-posix functions is a
\`PMEMfilepool’ pointer. This is required since it is possible that an
application could be working with multiple pools. We cannot rely on any
specific behavior with regard to the number of pools.

In all cases where the POSIX call would require a standard file
descriptor the Pmemfile POSIX Library functions will take a ‘struct
PMEMfile \*’. See Section File Descriptor Management for details.

###### Pmemfile POSIX Library Return Types

The return value for Pmemfile POSIX library functions which open and
create files will always be a ‘struct PMEMfile \*’. This structure is an
opaque structure which represents, internally to libpmemfile-posix, the
current information about a Pmemfile file. As a result of this any
functions that modify or read files will take the returned ‘struct
PMEMfile \*’ as input rather than a standard file descriptor.

###### File Descriptor Management

With Pmemfile when a call is made to the POSIX open() function, and the
Pmemfile system call interface interception layer is present, a file
descriptor for the Pmemfile file is generated by opening /dev/null. The
Pmemfile system call interception layer then keeps track of these file
descriptors and maps them to the appropriate volatile Pmemfile file
structure. If it is a non-pmem resident file it passes the open() call
through to the system call interface which creates a file descriptor.
It’s important to note that the file descriptor given from opening
/dev/null within the Pmemfile system call interface is a valid file
descriptor from the system point of view. The file descriptors index
into a per-process file descriptor table maintained by the kernel. This
means that the Pmemfile file descriptors are also kept in the
per-process file descriptor table in the kernel. The figure below shows
the process of assigning file descriptors when the Pmemfile system call
interface layer is present.

Choosing where to manage file descriptors is a critical aspect of
Pmemfile. We chose to differentiate the ‘file descriptors’ between the
Pmemfile system call interception layer and the Pmemfile POSIX library.
As a result, the Pmemfile POSIX library interfaces do not take a
standard file descriptor as an argument. This file descriptor for the
Pmemfile POSIX library is a ‘struct PMEMfile \*’ which is described in
Section **<span style="font-variant:small-caps;">*10.2.3*</span>**. A
call to pmemfile\_open() returns a ‘struct PMEMfile \*’ rather than a
file descriptor. An application developer uses this in turn to perform
operations on the file.

Why did we chose to have the management of file descriptors be handled
in the Pmemfile system call interception layer?

-   The way the Pmemfile POSIX library stores file data should be
    decoupled from the way the intercept layer stores it since the
    Pmemfile POSIX library is not dependent on the Pmemfile
    interception layer. The interception layer is a
    convenience provided.

-   The Pmemfile system call interception layer always evaluates the
    ownership of a file and creates the file descriptor based on
    that ownership. It manages all file descriptors which are for
    pmem-resident files.

-   If an application is using pmem-resident files managed by Pmemfile
    as well as standard media files within the same context the chance
    of errors goes up if file descriptors are created and managed by the
    Pmemfile POSIX library (this would mean a file descriptor would be a
    parameter into the Pmemfile POSIX functions as opposed to the
    ‘struct PMEMfile \*’.

    -   Even if a check was added to find the file descriptor in both
        the Pmemfile POSIX library and the kernel file descriptor table
        both would return true.

    -   Remember: a Pmemfile file descriptor is also a valid non-pmem
        resident file descriptor due to the use of /dev/null.

<span id="_Ref475540532" class="anchor"><span id="_Toc480386184" class="anchor"></span></span>Supported System Call Interfaces
==============================================================================================================================

The system call intercept library defined in **<span
style="font-variant:small-caps;">*System Call Interception
(libsyscall\_intercept.so)*</span>** supports the following system calls
for Pmemfile.

The table indicates name, support provided, exceptions to existing
system call behavior and notes on what the exception is. The **<span
style="font-variant:small-caps;">*libpmemfile.1 manpage*</span>**
defines exceptions to support for each system call noted in the table
below.

Table Supported System Call Interfaces

+--------------------+--------------------+--------------------+--------------------+
| Syscall            | Supported          | Exception          | Exceptions noted   |
+====================+====================+====================+====================+
| <span              | yes                |                    |                    |
| id="RANGE!A2:A118" |                    |                    |                    |
| class="anchor"></s |                    |                    |                    |
| pan>SYS\_access    |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_chdir         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_chmod         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_chown         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_chroot        | no                 | yes                | Pmemfile does not  |
|                    |                    |                    | allow the process  |
|                    |                    |                    | to change the root |
|                    |                    |                    | of the calling     |
|                    |                    |                    |                    |
|                    |                    |                    | process. The root  |
|                    |                    |                    | of the process is  |
|                    |                    |                    | always the root of |
|                    |                    |                    | the Pmemfile file  |
|                    |                    |                    |                    |
|                    |                    |                    | system.            |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_clone         | yes                | yes                | Clone is generally |
|                    |                    |                    | used to implement  |
|                    |                    |                    | threads. An        |
|                    |                    |                    | application must   |
|                    |                    |                    | provide            |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_THREAD as a |
|                    |                    |                    | flag otherwise the |
|                    |                    |                    | command will fail. |
|                    |                    |                    | The following      |
|                    |                    |                    |                    |
|                    |                    |                    | flags are not      |
|                    |                    |                    | supported:         |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_IO          |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_NEWIPC      |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_NEWNET      |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_NEWNS       |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_NEWPID      |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_NEWUTS      |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_PARENT and  |
|                    |                    |                    | related flags      |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_PID         |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_VFORK       |
|                    |                    |                    |                    |
|                    |                    |                    | This flag is       |
|                    |                    |                    | supported:         |
|                    |                    |                    |                    |
|                    |                    |                    | CLONE\_VM:         |
|                    |                    |                    |                    |
|                    |                    |                    | However, even if a |
|                    |                    |                    | thread is created  |
|                    |                    |                    | with shared        |
|                    |                    |                    | virtual            |
|                    |                    |                    |                    |
|                    |                    |                    | memory the child   |
|                    |                    |                    | will not be able   |
|                    |                    |                    | access, create or  |
|                    |                    |                    | modify any         |
|                    |                    |                    |                    |
|                    |                    |                    | pmem-resident      |
|                    |                    |                    | files.             |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_close         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_creat         | yes                | yes                | See open           |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_dup           | yes                | yes                | When dup() is      |
|                    |                    |                    | called the         |
|                    |                    |                    | refcount on the    |
|                    |                    |                    | Pmemfile handle in |
|                    |                    |                    | libpmemfile will   |
|                    |                    |                    | be increased to    |
|                    |                    |                    | keep track of      |
|                    |                    |                    | number of          |
|                    |                    |                    | references to a    |
|                    |                    |                    | file.              |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_dup2          | no                 | yes                | Same as dup()      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_dup3          | yes                | yes                | Close on exec is   |
|                    |                    |                    | always set         |
|                    |                    |                    |                    |
|                    |                    |                    | Same rule as for   |
|                    |                    |                    | dup, RE: refcount. |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_epoll\_ctl    | no                 | yes                | We are not         |
|                    |                    |                    | supporting polling |
|                    |                    |                    | of any kind at     |
|                    |                    |                    | this time.         |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_epoll\_pwait  | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_epoll\_wait   | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_execve        | yes                | yes                | Pmemfile does not  |
|                    |                    |                    | support execve     |
|                    |                    |                    | when the           |
|                    |                    |                    | executable file is |
|                    |                    |                    | a                  |
|                    |                    |                    |                    |
|                    |                    |                    | Pmem-resident      |
|                    |                    |                    | file. If the       |
|                    |                    |                    | ‘filename’ value   |
|                    |                    |                    | is a pmem-resident |
|                    |                    |                    | file this          |
|                    |                    |                    |                    |
|                    |                    |                    | will return an     |
|                    |                    |                    | ENOEXEC.           |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_faccessat     | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fadvise64     | no                 | yes                | Pmemfile does not  |
|                    |                    |                    | support fadvise64. |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fallocate     | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fchdir        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fchmod        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fchmodat      | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fchown        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fchownat      | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fcntl         | yes                | yes                | Pmemfile support   |
|                    |                    |                    | fcntl with the     |
|                    |                    |                    | following flag     |
|                    |                    |                    | exceptions:        |
|                    |                    |                    |                    |
|                    |                    |                    | Duplicating File   |
|                    |                    |                    | Descriptors        |
|                    |                    |                    |                    |
|                    |                    |                    | F\_DUPFD\_CLOEXEC  |
|                    |                    |                    |                    |
|                    |                    |                    | Pmemfile always    |
|                    |                    |                    | sets this flag.    |
|                    |                    |                    | Since this flag is |
|                    |                    |                    | always set the     |
|                    |                    |                    | libpmemfile layer  |
|                    |                    |                    | will return        |
|                    |                    |                    | success when       |
|                    |                    |                    | consumer sets      |
|                    |                    |                    | this. It will not  |
|                    |                    |                    | go through         |
|                    |                    |                    | interception       |
|                    |                    |                    | layer.             |
|                    |                    |                    |                    |
|                    |                    |                    | File Descriptor    |
|                    |                    |                    | Flags              |
|                    |                    |                    |                    |
|                    |                    |                    | F\_SETFD           |
|                    |                    |                    |                    |
|                    |                    |                    | The only flag      |
|                    |                    |                    | supported is       |
|                    |                    |                    | O\_CLOEXEC.        |
|                    |                    |                    |                    |
|                    |                    |                    | Pmemfile always    |
|                    |                    |                    | sets this flag.    |
|                    |                    |                    | Handling the same  |
|                    |                    |                    | as DUPFD\_CLOEXEC. |
|                    |                    |                    |                    |
|                    |                    |                    | File Status        |
|                    |                    |                    |                    |
|                    |                    |                    | F\_SETFL Is        |
|                    |                    |                    | supported.         |
|                    |                    |                    |                    |
|                    |                    |                    | O\_ASYNC - never,  |
|                    |                    |                    | O\_DIRECT -        |
|                    |                    |                    | always,            |
|                    |                    |                    | O\_NONBLOCK –      |
|                    |                    |                    | ignored            |
|                    |                    |                    |                    |
|                    |                    |                    | In call cases a 0  |
|                    |                    |                    | will be returned.  |
|                    |                    |                    |                    |
|                    |                    |                    | F\_GETFL is        |
|                    |                    |                    | supported.         |
|                    |                    |                    |                    |
|                    |                    |                    | Locking            |
|                    |                    |                    |                    |
|                    |                    |                    | F\_SETLK,          |
|                    |                    |                    | F\_SETLKW,         |
|                    |                    |                    | F\_GETLK, F\_UNLCK |
|                    |                    |                    |                    |
|                    |                    |                    | Are supported.     |
|                    |                    |                    |                    |
|                    |                    |                    | F\_SETOWN,         |
|                    |                    |                    | F\_GETOWN\_EX,     |
|                    |                    |                    | F\_SETOWN\_EX      |
|                    |                    |                    |                    |
|                    |                    |                    | Not supported.     |
|                    |                    |                    | Will return        |
|                    |                    |                    | EINVAL.            |
|                    |                    |                    |                    |
|                    |                    |                    | F\_GETSIG,         |
|                    |                    |                    | F\_SETSIG          |
|                    |                    |                    |                    |
|                    |                    |                    | Not supported.     |
|                    |                    |                    | Will return        |
|                    |                    |                    | EINVAL.            |
|                    |                    |                    |                    |
|                    |                    |                    | F\_SETLEASE,       |
|                    |                    |                    | F\_GETLEASE        |
|                    |                    |                    |                    |
|                    |                    |                    | Not supported.     |
|                    |                    |                    | Will return EINVAL |
|                    |                    |                    |                    |
|                    |                    |                    | F\_NOTIFY          |
|                    |                    |                    |                    |
|                    |                    |                    | Not supported.     |
|                    |                    |                    | Will return EINVAL |
|                    |                    |                    |                    |
|                    |                    |                    | MANDATORY LOCKS    |
|                    |                    |                    |                    |
|                    |                    |                    | Not supported.     |
|                    |                    |                    | Will return        |
|                    |                    |                    | EINVAL.            |
|                    |                    |                    |                    |
|                    |                    |                    | LEASES             |
|                    |                    |                    |                    |
|                    |                    |                    | Not supported.     |
|                    |                    |                    | Will return        |
|                    |                    |                    | EINVAL.            |
|                    |                    |                    |                    |
|                    |                    |                    | File and Directory |
|                    |                    |                    | change             |
|                    |                    |                    | notifications      |
|                    |                    |                    |                    |
|                    |                    |                    | Not supported.     |
|                    |                    |                    | Will return        |
|                    |                    |                    | EINVAL.            |
|                    |                    |                    |                    |
|                    |                    |                    | Pipe manipulation  |
|                    |                    |                    |                    |
|                    |                    |                    | Not supported.     |
|                    |                    |                    | Will return        |
|                    |                    |                    | EINVAL.            |
|                    |                    |                    |                    |
|                    |                    |                    | File sealing       |
|                    |                    |                    |                    |
|                    |                    |                    | Not supported.     |
|                    |                    |                    | Will return        |
|                    |                    |                    | EINVAL.            |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fdatasync     | yes                | yes                | All IO with        |
|                    |                    |                    | Pmemfile is        |
|                    |                    |                    | synchronous.       |
|                    |                    |                    | Libpmemfile        |
|                    |                    |                    | component will     |
|                    |                    |                    |                    |
|                    |                    |                    | return 0 for this  |
|                    |                    |                    | call.              |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fgetxattr     | no                 | yes                | No xattrs support  |
|                    |                    |                    | provided.          |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_flistxattr    | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_flock         | no                 | yes                | Pmemfile does not  |
|                    |                    |                    | support file       |
|                    |                    |                    | locking. Will      |
|                    |                    |                    | return EINVAL.     |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fork          | yes                | yes                | Pmemfile does not  |
|                    |                    |                    | provide            |
|                    |                    |                    | multi-process      |
|                    |                    |                    | support. A child   |
|                    |                    |                    | created with       |
|                    |                    |                    |                    |
|                    |                    |                    | fork() will not be |
|                    |                    |                    | able to access any |
|                    |                    |                    | existing           |
|                    |                    |                    | pmem-resident      |
|                    |                    |                    | files nor          |
|                    |                    |                    |                    |
|                    |                    |                    | create new ones.   |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fremovexattr  | no                 | yes                | No xattrs support  |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fsetxattr     | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fstat         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fstatfs       | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_fsync         | yes                | yes                | Libpmemfile        |
|                    |                    |                    | component will     |
|                    |                    |                    | return 0 for all   |
|                    |                    |                    | file sync          |
|                    |                    |                    | commands.          |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_ftruncate     | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_ftruncate64   | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_futimesat     | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_getcwd        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_getdents      | yes                | yes                | This does not have |
|                    |                    |                    | a glibc wrapper.   |
|                    |                    |                    | The real support   |
|                    |                    |                    | is for readdir().  |
|                    |                    |                    |                    |
|                    |                    |                    | Man page will      |
|                    |                    |                    | reflect the        |
|                    |                    |                    | support of         |
|                    |                    |                    | readdir().         |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_getdents64    | yes                | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_inotify\_add\ | no                 | yes                | Pmemfile does not  |
| _watch             |                    |                    | support any I/O    |
|                    |                    |                    | event notification |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_inotify\_rm\_ | no                 | yes                | Same as above      |
| watch              |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_io\_cancel    | No                 | yes                | Pmemfile will not  |
|                    |                    |                    | support            |
|                    |                    |                    | asynchronous IO    |
|                    |                    |                    | for V1.0           |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_io\_destroy   | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_io\_getevents | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_io\_setup     | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_io\_submit    | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_ioctl         | yes\*              | yes                | \*Pmemfile V1.0    |
|                    |                    |                    | will not provide   |
|                    |                    |                    | support for this.  |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_lchown        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_lgetxattr     | no                 | yes                | No xattrs support  |
|                    |                    |                    | provided           |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_link          | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_linkat        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_listxattr     | no                 | yes                | Same as all xattrs |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_llistxattr    | no                 | yes                | Same as all xattrs |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_lremovexattr  | no                 | yes                | Same as all xattrs |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_lseek         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_lsetxattr     | no                 | yes                | Same as all xattrs |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_lstat         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_mkdir         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_mkdirat       | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_mknod         | no                 | yes                | Pmemfile does not  |
|                    |                    |                    | support block or   |
|                    |                    |                    | character devices  |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_mknodat       | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_mmap          | no?                | yes                | If Yes – mmap      |
|                    |                    |                    | supports mmap for  |
|                    |                    |                    | the following      |
|                    |                    |                    | flags:             |
|                    |                    |                    |                    |
|                    |                    |                    | MAP\_SHARED,       |
|                    |                    |                    | MAP\_FIXED         |
|                    |                    |                    |                    |
|                    |                    |                    | Protection flags   |
|                    |                    |                    | supported:         |
|                    |                    |                    |                    |
|                    |                    |                    | PROT\_READ and     |
|                    |                    |                    | PROT\_WRITE is the |
|                    |                    |                    | default            |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_newfstatat    | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_open          | yes                | yes                | Pmemfile supports  |
|                    |                    |                    | open with the      |
|                    |                    |                    | following flag     |
|                    |                    |                    | exceptions:        |
|                    |                    |                    |                    |
|                    |                    |                    | O\_ASYNC - EINVAL  |
|                    |                    |                    |                    |
|                    |                    |                    | O\_CNOTTY – Always |
|                    |                    |                    | enabled            |
|                    |                    |                    |                    |
|                    |                    |                    | Only supported for |
|                    |                    |                    | terminals, pseudo  |
|                    |                    |                    | terminals, sockets |
|                    |                    |                    | and fifo’s         |
|                    |                    |                    |                    |
|                    |                    |                    | Pmemfile does not  |
|                    |                    |                    | support these      |
|                    |                    |                    | devices.           |
|                    |                    |                    |                    |
|                    |                    |                    | O\_DIRECT – Always |
|                    |                    |                    | enabled.           |
|                    |                    |                    |                    |
|                    |                    |                    | O\_DSYNC – Always  |
|                    |                    |                    | enabled.           |
|                    |                    |                    |                    |
|                    |                    |                    | O\_NONBLOCK –      |
|                    |                    |                    | Ignored.           |
|                    |                    |                    |                    |
|                    |                    |                    | O\_SYNC – Always   |
|                    |                    |                    | enabled.           |
|                    |                    |                    |                    |
|                    |                    |                    | O\_CLOEXEC –       |
|                    |                    |                    | Always enabled.    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_open\_by\_han | yes\*              | yes                | \*Not in V1.0      |
| dle\_at            |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_openat        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_poll          | no                 | yes                | Pmemfile does not  |
|                    |                    |                    | support any event  |
|                    |                    |                    | driven file        |
|                    |                    |                    | management         |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_ppoll         | no                 | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_pread/64      | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_pselect       | no                 | yes                | Pmemfile does not  |
|                    |                    |                    | support any event  |
|                    |                    |                    | driven file        |
|                    |                    |                    | management.        |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_pwrite64      | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_pwritev       | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_read          | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_readahead     | no                 | Yes                | Is not supported.  |
|                    |                    |                    | Pmemfile does not  |
|                    |                    |                    | support caching as |
|                    |                    |                    | it always operates |
|                    |                    |                    | in direct access   |
|                    |                    |                    | mode.              |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_readlink      | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_readlinkat    | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_readv         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_removexattr   | no                 | yes                | Pmemfile does not  |
|                    |                    |                    | support xattrs     |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_rename        | yes                | yes                | Pmemfile does not  |
|                    |                    |                    | support renaming   |
|                    |                    |                    | files between      |
|                    |                    |                    | Pmemfile file      |
|                    |                    |                    | systems.           |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_renameat      | yes                | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_renameat2     | no                 | yes                |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_rmdir         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_select        | no                 | yes                | Same as pselect    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_setxattr      | no                 | yes                | Same as other      |
|                    |                    |                    | xattrs support     |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_sendfile      | yes\*              | yes                | Pmemfile supports  |
|                    |                    |                    | this if both in fd |
|                    |                    |                    | and out fd are     |
|                    |                    |                    | pmem-resident      |
|                    |                    |                    | files.             |
|                    |                    |                    |                    |
|                    |                    |                    | \*Not supported in |
|                    |                    |                    | V1.0               |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_splice        | yes\*              | yes                | Pmemfile supports  |
|                    |                    |                    | this if both the   |
|                    |                    |                    | out fd and in fd   |
|                    |                    |                    | are pmem-resident  |
|                    |                    |                    | files.             |
|                    |                    |                    |                    |
|                    |                    |                    | \* Not supported   |
|                    |                    |                    | in V1.0            |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_stat          | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_statfs        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_swapoff       | no                 | yes                | Pmemfile cannot be |
|                    |                    |                    | used as a swap     |
|                    |                    |                    | device             |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_symlink       | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_symlinkat     | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_sync          | yes                | yes                | Pmemfile treats    |
|                    |                    |                    | this as a no-op    |
|                    |                    |                    | since all IO is    |
|                    |                    |                    | synchronous.       |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_sync\_file\_r | yes                | yes                | Pmemfile treats    |
| ange               |                    |                    | this as a no-op.   |
|                    |                    |                    | Pmemfile IO is     |
|                    |                    |                    | always             |
|                    |                    |                    | synchronous.       |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_syncfs        | yes                | yes                | Same as above      |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_sysfs         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_truncate      | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_unlink        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_unlinkat      | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_utime         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_utimensat     | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_utimes        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_vfork         | no                 | yes                | Pmemfile does not  |
|                    |                    |                    | support this call. |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_write         | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SYS\_writev        | yes                |                    |                    |
+--------------------+--------------------+--------------------+--------------------+

libpmemfile-posix Interface And System Call Support
===================================================

The libpmemfile-posix library will not provide support for all of the
system calls defined in Section **<span
style="font-variant:small-caps;">*Supported System Call Interfaces.
*</span>**

Specifically, libpmemfile-posix does not provide support for:

-   Thread or process creation. This is handled by the
    calling application.

    -   A child process cannot create or access any file in the current
        Pmemfile pool. The top level libpmemfile interface will manage
        handling and verification of this.

-   Any data or file system sync calls. All operations in Pmemfile
    are synchronous.

-   No advisement operations such as readahead or fadvise. This does not
    make sense in the Pmemfile space.

The **<span style="font-variant:small-caps;">*libpmemfile-posix.3
manpage*</span>** defines and describes all behaviors for the interfaces
exported by the core library.

1.  <span id="_Toc452012211" class="anchor"><span id="_Toc480386186" class="anchor"></span></span>Volatile and Persistent Data Structures 
    ======================================================================================================================================

    1.  Pmemfile Data Structure Considerations
        --------------------------------------

There are some high level architectural decisions that have been made
for reducing wear on the persistent media.

1.  Locks that protect persistent data structure fields are defined in
    the volatile data structure. This is because lock values change
    frequently and can cause unnecessary wear on the media.

2.  The naming of the volatile data structures that have an on-media
    counterpart will have the type prefixed by a ‘v’, e.g.
    pmemfile\_vinode.

    1.  Pmemfile Volatile Data Structures
        ---------------------------------

The Pmemfile volatile data structures will not be specifically defined
here as they are subject to change and are not a contract between the
user and the Pmemfile library. However, the general principles and
behavior of these is described in this section.

### PMEMfilepool

The pmemfile\_pool structure is the top level libpmemfile-posix data
structure. It provides a pointer to the pool as well as a pointer to the
pool super block.

### pmemfile\_vinode

The pmemfile\_vinode structure provides access to the on-media inode,
locking for the on-media inode as well as keep track of the reference
count. Locking for an inode is not required to be persistent. If the
system crashes or when the file system closes all file descriptors are
destroyed and no locks are held on the on media inode.

The pmemfile\_vnode structure also store any caches related to an inode.
For example, block mapping, directory file lookup.

### <span id="_Ref466892410" class="anchor"><span id="_Ref466892454" class="anchor"><span id="_Toc480386191" class="anchor"></span></span></span>pmemfile\_file

> The pmemfile\_file structure a handle to an opened file. This
> pmemfile\_file structure will be mapped with a file descriptor that is
> generated when the file is opened. In general this data structure
> contains:

-   current offset information

-   locks

-   pointer to the volatile inode for this file

-   file flags

    1.  <span id="_Toc463242603" class="anchor"><span id="_Toc463242675" class="anchor"><span id="_Toc463242747" class="anchor"><span id="_Toc464455156" class="anchor"><span id="_Toc464629639" class="anchor"><span id="_Toc465174022" class="anchor"><span id="_Toc465174099" class="anchor"><span id="_Toc465174174" class="anchor"><span id="_Toc465190495" class="anchor"><span id="_Toc465190542" class="anchor"><span id="_Toc465960440" class="anchor"><span id="_Toc465960532" class="anchor"><span id="_Toc465966236" class="anchor"><span id="_Toc465967059" class="anchor"><span id="_Toc465967480" class="anchor"><span id="_Toc465967525" class="anchor"><span id="_Toc466468248" class="anchor"><span id="_Toc466468293" class="anchor"><span id="_Toc466469037" class="anchor"><span id="_Toc467171931" class="anchor"><span id="_Toc467171979" class="anchor"><span id="_Toc467172027" class="anchor"><span id="_Toc467172075" class="anchor"><span id="_Toc471213427" class="anchor"><span id="_Toc471213476" class="anchor"><span id="_Toc471224682" class="anchor"><span id="_Toc471224731" class="anchor"><span id="_Toc471288145" class="anchor"><span id="_Toc471288194" class="anchor"><span id="_Toc471393238" class="anchor"><span id="_Toc471908157" class="anchor"><span id="_Toc471908210" class="anchor"><span id="_Toc472006550" class="anchor"><span id="_Toc475535067" class="anchor"><span id="_Toc475535123" class="anchor"><span id="_Toc475535179" class="anchor"><span id="_Toc475536834" class="anchor"><span id="_Toc475536884" class="anchor"><span id="_Toc475536934" class="anchor"><span id="_Toc477854779" class="anchor"><span id="_Toc477854829" class="anchor"><span id="_Toc477855267" class="anchor"><span id="_Toc477864389" class="anchor"><span id="_Toc477864649" class="anchor"><span id="_Toc477867187" class="anchor"><span id="_Toc477867237" class="anchor"><span id="_Toc452012214" class="anchor"><span id="_Ref466979093" class="anchor"><span id="_Ref477793717" class="anchor"><span id="_Ref477793882" class="anchor"><span id="_Toc480386192" class="anchor"></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span>Persistent Data Structures
        -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Persistent data structures are those stored on the persistent media and
describe the file system metadata. We must strive to ensure that the
format of the persistent data structures are complete and stable in
their definition. However, it’s possible we may have to make changes
moving forward which may require a version change. Locking is provided
in the volatile data structures defined for each persistent structure
unless otherwise specified.

### pmemfile\_super

The superblock is a unique data structure in a filesystem. It is the top
level data structure which describes high level metadata about the
pmemfile file system.
```c

struct pmemfile\_super {

/\* Superblock version \*/

uint64\_t version;

/\* Root directory inode \*/

TOID(struct pmemfile\_inode) root\_ino;

> /\* List of arrays of inodes that were deleted, but are still opened
> \*/

TOID(struct pmemfile\_inode\_array) orphaned\_inodes;

char pad\[4096

- 8 /\* version \*/

- 16 /\* toid \*/

- 16 /\* toid \*/\];

};
```

#### Valid states for pmemfile\_super members

-   version &gt; 0

-   root\_ino != NULL

-   orphaned inodes != NULL

    -   All deleted files and directory inodes are put on the orphaned
        list when a file is deleted. Inodes on this list are cleaned up
        when pmemfile\_vinode-&gt;ref == 0 and pmemfile\_inode-&gt;nlink
        == 0;

<!-- -->

-   Pad is initialized to 0 and not used at this point.

    1.  ### pmemfile\_block

struct pmemfile\_block {

> TOID(char) data;

uint32\_t size;

uint32\_t flags;

uint64\_t offset; /\* offset from the beginning of the file \*/

TOID(struct pmemfile\_block) next; /\* next block with data \*/

TOID(struct pmemfile\_block) prev; /\* prev block with data \*/

};

#### Valid states for pmemfile\_block members

-   data = pointer to valid data address

-   size = Multiple of 4k, variable size

-   flags = initialized

-   offset &gt;= 0. Offset into file

    next, prev = NULL or valid pointer to next block with data block &&
    do not belong to another inode

    1.  ### pmemfile\_block\_array

<span id="_Toc457235524" class="anchor"><span id="_Toc457235582"
class="anchor"><span id="_Toc457240365" class="anchor"><span
id="_Toc457240422" class="anchor"><span id="_Toc463242622"
class="anchor"><span id="_Toc463242694" class="anchor"><span
id="_Toc463242766" class="anchor"><span id="_Toc464455175"
class="anchor"><span id="_Toc464629658" class="anchor"><span
id="_Toc465174041" class="anchor"><span id="_Toc465174118"
class="anchor"><span id="_Toc465174193" class="anchor"><span
id="_Toc457235525" class="anchor"><span id="_Toc457235583"
class="anchor"><span id="_Toc457240366" class="anchor"><span
id="_Toc457240423" class="anchor"><span id="_Toc463242623"
class="anchor"><span id="_Toc463242695" class="anchor"><span
id="_Toc463242767" class="anchor"><span id="_Toc464455176"
class="anchor"><span id="_Toc464629659" class="anchor"><span
id="_Toc465174042" class="anchor"><span id="_Toc465174119"
class="anchor"><span id="_Toc465174194" class="anchor"><span
id="_Toc457235526" class="anchor"><span id="_Toc457235584"
class="anchor"><span id="_Toc457240367" class="anchor"><span
id="_Toc457240424" class="anchor"><span id="_Toc463242624"
class="anchor"><span id="_Toc463242696" class="anchor"><span
id="_Toc463242768" class="anchor"><span id="_Toc464455177"
class="anchor"><span id="_Toc464629660" class="anchor"><span
id="_Toc465174043" class="anchor"><span id="_Toc465174120"
class="anchor"><span id="_Toc465174195" class="anchor"><span
id="_Toc457235527" class="anchor"><span id="_Toc457235585"
class="anchor"><span id="_Toc457240368" class="anchor"><span
id="_Toc457240425" class="anchor"><span id="_Toc463242625"
class="anchor"><span id="_Toc463242697" class="anchor"><span
id="_Toc463242769" class="anchor"><span id="_Toc464455178"
class="anchor"><span id="_Toc464629661" class="anchor"><span
id="_Toc465174044" class="anchor"><span id="_Toc465174121"
class="anchor"><span id="_Toc465174196" class="anchor"><span
id="_Toc457235528" class="anchor"><span id="_Toc457235586"
class="anchor"><span id="_Toc457240369" class="anchor"><span
id="_Toc457240426" class="anchor"><span id="_Toc463242626"
class="anchor"><span id="_Toc463242698" class="anchor"><span
id="_Toc463242770" class="anchor"><span id="_Toc464455179"
class="anchor"><span id="_Toc464629662" class="anchor"><span
id="_Toc465174045" class="anchor"><span id="_Toc465174122"
class="anchor"><span id="_Toc465174197" class="anchor"><span
id="_Toc457235529" class="anchor"><span id="_Toc457235587"
class="anchor"><span id="_Toc457240370" class="anchor"><span
id="_Toc457240427" class="anchor"><span id="_Toc463242627"
class="anchor"><span id="_Toc463242699" class="anchor"><span
id="_Toc463242771" class="anchor"><span id="_Toc464455180"
class="anchor"><span id="_Toc464629663" class="anchor"><span
id="_Toc465174046" class="anchor"><span id="_Toc465174123"
class="anchor"><span id="_Toc465174198" class="anchor"><span
id="_Toc457235530" class="anchor"><span id="_Toc457235588"
class="anchor"><span id="_Toc457240371" class="anchor"><span
id="_Toc457240428" class="anchor"><span id="_Toc463242628"
class="anchor"><span id="_Toc463242700" class="anchor"><span
id="_Toc463242772" class="anchor"><span id="_Toc464455181"
class="anchor"><span id="_Toc464629664" class="anchor"><span
id="_Toc465174047" class="anchor"><span id="_Toc465174124"
class="anchor"><span id="_Toc465174199" class="anchor"><span
id="_Toc457235531" class="anchor"><span id="_Toc457235589"
class="anchor"><span id="_Toc457240372" class="anchor"><span
id="_Toc457240429" class="anchor"><span id="_Toc463242629"
class="anchor"><span id="_Toc463242701" class="anchor"><span
id="_Toc463242773" class="anchor"><span id="_Toc464455182"
class="anchor"><span id="_Toc464629665" class="anchor"><span
id="_Toc465174048" class="anchor"><span id="_Toc465174125"
class="anchor"><span id="_Toc465174200" class="anchor"><span
id="_Toc457235532" class="anchor"><span id="_Toc457235590"
class="anchor"><span id="_Toc457240373" class="anchor"><span
id="_Toc457240430" class="anchor"><span id="_Toc463242630"
class="anchor"><span id="_Toc463242702" class="anchor"><span
id="_Toc463242774" class="anchor"><span id="_Toc464455183"
class="anchor"><span id="_Toc464629666" class="anchor"><span
id="_Toc465174049" class="anchor"><span id="_Toc465174126"
class="anchor"><span id="_Toc465174201" class="anchor"><span
id="_Toc457235533" class="anchor"><span id="_Toc457235591"
class="anchor"><span id="_Toc457240374" class="anchor"><span
id="_Toc457240431" class="anchor"><span id="_Toc463242631"
class="anchor"><span id="_Toc463242703" class="anchor"><span
id="_Toc463242775" class="anchor"><span id="_Toc464455184"
class="anchor"><span id="_Toc464629667" class="anchor"><span
id="_Toc465174050" class="anchor"><span id="_Toc465174127"
class="anchor"><span id="_Toc465174202" class="anchor"><span
id="_Toc457235534" class="anchor"><span id="_Toc457235592"
class="anchor"><span id="_Toc457240375" class="anchor"><span
id="_Toc457240432" class="anchor"><span id="_Toc463242632"
class="anchor"><span id="_Toc463242704" class="anchor"><span
id="_Toc463242776" class="anchor"><span id="_Toc464455185"
class="anchor"><span id="_Toc464629668" class="anchor"><span
id="_Toc465174051" class="anchor"><span id="_Toc465174128"
class="anchor"><span id="_Toc465174203" class="anchor"><span
id="_Toc457235535" class="anchor"><span id="_Toc457235593"
class="anchor"><span id="_Toc457240376" class="anchor"><span
id="_Toc457240433" class="anchor"><span id="_Toc463242633"
class="anchor"><span id="_Toc463242705" class="anchor"><span
id="_Toc463242777" class="anchor"><span id="_Toc464455186"
class="anchor"><span id="_Toc464629669" class="anchor"><span
id="_Toc465174052" class="anchor"><span id="_Toc465174129"
class="anchor"><span id="_Toc465174204" class="anchor"><span
id="_Toc457235536" class="anchor"><span id="_Toc457235594"
class="anchor"><span id="_Toc457240377" class="anchor"><span
id="_Toc457240434" class="anchor"><span id="_Toc463242634"
class="anchor"><span id="_Toc463242706" class="anchor"><span
id="_Toc463242778" class="anchor"><span id="_Toc464455187"
class="anchor"><span id="_Toc464629670" class="anchor"><span
id="_Toc465174053" class="anchor"><span id="_Toc465174130"
class="anchor"><span id="_Toc465174205"
class="anchor"></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span>The
block array is per inode.

struct pmemfile\_block\_array {

TOID(struct pmemfile\_block\_array) next;

/\* size of the blocks array \*/

uint32\_t length;

uint32\_t padding;

struct pmemfile\_block blocks\[\];

};

#### Valid states for pmemfile\_block\_array members

-   next = NULL or TOID of next member in array

-   length = number of possible blocks in array

-   blocks\[\] = array of pmemfile\_block structures

-   Holey files:

    -   Holes in a file are not represented as blocks.

    -   A hole is detected when blocks\[n\]-&gt;next.offset !=
        blocks\[n\].offset + blocks\[n\].size

        1.  ### pmemfile\_dirent

Directory entries are individual members of a parent directory.

struct pmemfile\_dirent {

> TOID(struct pmemfile\_inode) inode;
>
> char name\[PMEMFILE\_MAX\_FILE\_NAME + 1\];

};

#### Valid states for pmemfile\_dirent

-   inode != NULL && must be an inode in use(nlink &gt; 0)

-   name != NULL && null terminated and no longer than
    PMEMFILE\_MAX\_FILE\_NAME + 1

    1.  ### pmemfile\_dir

Directory entries are stored in a directory object. Multiple directory
entries are contained in a directory object.

struct pmemfile\_dir {

uint32\_t num\_elements;

uint32\_t padding;

TOID(struct pmemfile\_dir) next;

struct pmemfile\_dirent dentries\[\];

};

#### Valid states for pmemfile\_dir

-   num\_elements &gt;= 2. At least ‘.’ and ‘..’

-   next == NULL or TOID of next directory

-   dentries\[\] != NULL. Must have ‘.’ and ‘..’, in that order

    1.  ### pmemfile\_time

struct pmemfile\_time {

/\* Seconds \*/

int64\_t sec;

/\* Nanoseconds \*/

int64\_t nsec;

};

#### Valid pmemfile\_time members

-   sec an nsec should be non-negative.

-   nsec &lt; 10 the nth

    1.  ### <span id="_Toc465190499" class="anchor"><span id="_Toc465190546" class="anchor"><span id="_Toc465960445" class="anchor"><span id="_Toc465960537" class="anchor"><span id="_Toc465966241" class="anchor"><span id="_Toc465967064" class="anchor"><span id="_Toc465967485" class="anchor"><span id="_Toc465967530" class="anchor"><span id="_Toc466468253" class="anchor"><span id="_Toc466468298" class="anchor"><span id="_Toc466469042" class="anchor"><span id="_Toc467171939" class="anchor"><span id="_Toc467171987" class="anchor"><span id="_Toc467172035" class="anchor"><span id="_Toc467172083" class="anchor"><span id="_Toc471213436" class="anchor"><span id="_Toc471213485" class="anchor"><span id="_Toc471224691" class="anchor"><span id="_Toc471224740" class="anchor"><span id="_Toc471288154" class="anchor"><span id="_Toc471288203" class="anchor"><span id="_Toc471393247" class="anchor"><span id="_Toc471908166" class="anchor"><span id="_Toc471908219" class="anchor"><span id="_Toc472006559" class="anchor"><span id="_Toc475535076" class="anchor"><span id="_Toc475535132" class="anchor"><span id="_Toc475535188" class="anchor"><span id="_Toc475536843" class="anchor"><span id="_Toc475536893" class="anchor"><span id="_Toc475536943" class="anchor"><span id="_Toc477854787" class="anchor"><span id="_Toc477854837" class="anchor"><span id="_Toc477855275" class="anchor"><span id="_Toc477864397" class="anchor"><span id="_Toc477864657" class="anchor"><span id="_Toc477867195" class="anchor"><span id="_Toc477867245" class="anchor"><span id="_Toc465190500" class="anchor"><span id="_Toc465190547" class="anchor"><span id="_Toc465960446" class="anchor"><span id="_Toc465960538" class="anchor"><span id="_Toc465966242" class="anchor"><span id="_Toc465967065" class="anchor"><span id="_Toc465967486" class="anchor"><span id="_Toc465967531" class="anchor"><span id="_Toc466468254" class="anchor"><span id="_Toc466468299" class="anchor"><span id="_Toc466469043" class="anchor"><span id="_Toc467171940" class="anchor"><span id="_Toc467171988" class="anchor"><span id="_Toc467172036" class="anchor"><span id="_Toc467172084" class="anchor"><span id="_Toc471213437" class="anchor"><span id="_Toc471213486" class="anchor"><span id="_Toc471224692" class="anchor"><span id="_Toc471224741" class="anchor"><span id="_Toc471288155" class="anchor"><span id="_Toc471288204" class="anchor"><span id="_Toc471393248" class="anchor"><span id="_Toc471908167" class="anchor"><span id="_Toc471908220" class="anchor"><span id="_Toc472006560" class="anchor"><span id="_Toc475535077" class="anchor"><span id="_Toc475535133" class="anchor"><span id="_Toc475535189" class="anchor"><span id="_Toc475536844" class="anchor"><span id="_Toc475536894" class="anchor"><span id="_Toc475536944" class="anchor"><span id="_Toc477854788" class="anchor"><span id="_Toc477854838" class="anchor"><span id="_Toc477855276" class="anchor"><span id="_Toc477864398" class="anchor"><span id="_Toc477864658" class="anchor"><span id="_Toc477867196" class="anchor"><span id="_Toc477867246" class="anchor"><span id="_Toc480386199" class="anchor"></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span>pmemfile\_inode

A pmemfile\_inode represents the file metadata. All file types,
directories and regular files, have a pmemfile\_inode representation.

struct pmemfile\_inode {

/\* Layout version \*/

uint32\_t version;

/\* Owner \*/

uint32\_t uid;

/\* Group \*/

uint32\_t gid;

uint32\_t reserved;

/\* Time of last access. \*/

struct pmemfile\_time atime;

/\* Time of last status change. \*/

struct pmemfile\_time ctime;

/\* Time of last modification. \*/

struct pmemfile\_time mtime;

/\* Hard link counter. \*/

uint64\_t nlink;

/\* Size of file. \*/

uint64\_t size;

/\* File flags. \*/

uint64\_t flags;

/\* Data! \*/

union {

/\* File specific data. \*/

struct pmemfile\_block\_array blocks;

/\* Directory specific data. \*/

struct pmemfile\_dir dir;

char data\[4096

- 4 /\* version \*/

- 4 /\* uid \*/

- 4 /\* gid \*/

- 4 /\* reserved \*/

- 16 /\* atime \*/

- 16 /\* ctime \*/

- 16 /\* mtime \*/

- 8 /\* nlink \*/

- 8 /\* size \*/

- 8 /\* flags \*/\];

} file\_data;

};

####  Valid states for pmemfile\_inode

-   Version &gt; 0

-   uid, gid – valid uid and gid as defined on system

-   atime, ctime, mtime – valid pmemfile\_time data

-   nlink &gt; = 1, orphaned inodes nlink can be 0

    -   Directory inodes have nlinks = 2 + number of subdirectories

-   size = regular file = length of file, dir = number of bytes a
    directory takes, symlinks length of the path

-   flags = flags + mode

    -   See: Supported System Call Interfaces, open() for flag and mode
        exceptions

-   file\_data:

    -   blocks

        -   Mutually exclusive from dir

        -   List will be non-NULL if regular file && blocks cannot be
            specified in more than one inode

    -   dir

        -   Mutually exclusive from blocks. Blocks for directory are
            kept in the inode for directory. If dir != NULL then this
            inode is a directory.

    -   char data\[4096-size of current members of pmemfile\_inode\]

        -   This is used as storage for the symlink path if a symlink

        1.  ### <span id="_Toc465181680" class="anchor"><span id="_Toc480386200" class="anchor"></span></span>pmemfile\_inode\_array 

struct pmemfile\_inode\_array {

PMEMmutex mtx;

TOID(struct pmemfile\_inode\_array) prev;

TOID(struct pmemfile\_inode\_array) next;

> /\* Number of used entries, &lt;0, NUMINODES\_PER\_ENTRY&gt; \*/

uint32\_t used;

char padding\[12\];

> TOID(struct pmemfile\_inode) inodes\[NUMINODES\_PER\_ENTRY\];

};

#### Valid states for pmemfile\_inode\_array

-   mtx != NULL when modifying inode arrays or used value, used when we
    unlink file. Pmemobj will clean up.

-   prev, next, prev != NULL, next == NULL or valid array object.

-   used = number of non-null entries in array.

-   pad is zeroed at initialization. Unused.

-   inodes – entries in this inode array can be null. Only inserted
    at unlink. Number of non-null entries in this area == used

<span id="_Toc465181681" class="anchor"></span>

<span id="_Toc457235512" class="anchor"><span id="_Toc457235570" class="anchor"><span id="_Toc457240353" class="anchor"><span id="_Toc457240410" class="anchor"><span id="_Toc463242610" class="anchor"><span id="_Toc463242682" class="anchor"><span id="_Toc463242754" class="anchor"><span id="_Toc464455163" class="anchor"><span id="_Toc464629646" class="anchor"><span id="_Toc465174029" class="anchor"><span id="_Toc465174106" class="anchor"><span id="_Toc465174181" class="anchor"><span id="_Toc465190502" class="anchor"><span id="_Toc465190549" class="anchor"><span id="_Toc465960448" class="anchor"><span id="_Toc465960540" class="anchor"><span id="_Toc465966244" class="anchor"><span id="_Toc465967067" class="anchor"><span id="_Toc465967488" class="anchor"><span id="_Toc465967533" class="anchor"><span id="_Toc466468256" class="anchor"><span id="_Toc466468301" class="anchor"><span id="_Toc466469045" class="anchor"><span id="_Toc467171943" class="anchor"><span id="_Toc467171991" class="anchor"><span id="_Toc467172039" class="anchor"><span id="_Toc467172087" class="anchor"><span id="_Toc471213440" class="anchor"><span id="_Toc471213489" class="anchor"><span id="_Toc471224695" class="anchor"><span id="_Toc471224744" class="anchor"><span id="_Toc471288158" class="anchor"><span id="_Toc471288207" class="anchor"><span id="_Toc471393251" class="anchor"><span id="_Toc471908170" class="anchor"><span id="_Toc471908223" class="anchor"><span id="_Toc472006563" class="anchor"><span id="_Toc475535080" class="anchor"><span id="_Toc475535136" class="anchor"><span id="_Toc475535192" class="anchor"><span id="_Toc475536847" class="anchor"><span id="_Toc475536897" class="anchor"><span id="_Toc475536947" class="anchor"><span id="_Toc477854791" class="anchor"><span id="_Toc477854841" class="anchor"><span id="_Toc477855279" class="anchor"><span id="_Toc477864401" class="anchor"><span id="_Toc477864661" class="anchor"><span id="_Toc477867199" class="anchor"><span id="_Toc477867249" class="anchor"><span id="_Toc457235513" class="anchor"><span id="_Toc457235571" class="anchor"><span id="_Toc457240354" class="anchor"><span id="_Toc457240411" class="anchor"><span id="_Toc463242611" class="anchor"><span id="_Toc463242683" class="anchor"><span id="_Toc463242755" class="anchor"><span id="_Toc464455164" class="anchor"><span id="_Toc464629647" class="anchor"><span id="_Toc465174030" class="anchor"><span id="_Toc465174107" class="anchor"><span id="_Toc465174182" class="anchor"><span id="_Toc465190503" class="anchor"><span id="_Toc465190550" class="anchor"><span id="_Toc465960449" class="anchor"><span id="_Toc465960541" class="anchor"><span id="_Toc465966245" class="anchor"><span id="_Toc465967068" class="anchor"><span id="_Toc465967489" class="anchor"><span id="_Toc465967534" class="anchor"><span id="_Toc466468257" class="anchor"><span id="_Toc466468302" class="anchor"><span id="_Toc466469046" class="anchor"><span id="_Toc467171944" class="anchor"><span id="_Toc467171992" class="anchor"><span id="_Toc467172040" class="anchor"><span id="_Toc467172088" class="anchor"><span id="_Toc471213441" class="anchor"><span id="_Toc471213490" class="anchor"><span id="_Toc471224696" class="anchor"><span id="_Toc471224745" class="anchor"><span id="_Toc471288159" class="anchor"><span id="_Toc471288208" class="anchor"><span id="_Toc471393252" class="anchor"><span id="_Toc471908171" class="anchor"><span id="_Toc471908224" class="anchor"><span id="_Toc472006564" class="anchor"><span id="_Toc475535081" class="anchor"><span id="_Toc475535137" class="anchor"><span id="_Toc475535193" class="anchor"><span id="_Toc475536848" class="anchor"><span id="_Toc475536898" class="anchor"><span id="_Toc475536948" class="anchor"><span id="_Toc477854792" class="anchor"><span id="_Toc477854842" class="anchor"><span id="_Toc477855280" class="anchor"><span id="_Toc477864402" class="anchor"><span id="_Toc477864662" class="anchor"><span id="_Toc477867200" class="anchor"><span id="_Toc477867250" class="anchor"><span id="_Toc457235514" class="anchor"><span id="_Toc457235572" class="anchor"><span id="_Toc457240355" class="anchor"><span id="_Toc457240412" class="anchor"><span id="_Toc463242612" class="anchor"><span id="_Toc463242684" class="anchor"><span id="_Toc463242756" class="anchor"><span id="_Toc464455165" class="anchor"><span id="_Toc464629648" class="anchor"><span id="_Toc465174031" class="anchor"><span id="_Toc465174108" class="anchor"><span id="_Toc465174183" class="anchor"><span id="_Toc465190504" class="anchor"><span id="_Toc465190551" class="anchor"><span id="_Toc465960450" class="anchor"><span id="_Toc465960542" class="anchor"><span id="_Toc465966246" class="anchor"><span id="_Toc465967069" class="anchor"><span id="_Toc465967490" class="anchor"><span id="_Toc465967535" class="anchor"><span id="_Toc466468258" class="anchor"><span id="_Toc466468303" class="anchor"><span id="_Toc466469047" class="anchor"><span id="_Toc467171945" class="anchor"><span id="_Toc467171993" class="anchor"><span id="_Toc467172041" class="anchor"><span id="_Toc467172089" class="anchor"><span id="_Toc471213442" class="anchor"><span id="_Toc471213491" class="anchor"><span id="_Toc471224697" class="anchor"><span id="_Toc471224746" class="anchor"><span id="_Toc471288160" class="anchor"><span id="_Toc471288209" class="anchor"><span id="_Toc471393253" class="anchor"><span id="_Toc471908172" class="anchor"><span id="_Toc471908225" class="anchor"><span id="_Toc472006565" class="anchor"><span id="_Toc475535082" class="anchor"><span id="_Toc475535138" class="anchor"><span id="_Toc475535194" class="anchor"><span id="_Toc475536849" class="anchor"><span id="_Toc475536899" class="anchor"><span id="_Toc475536949" class="anchor"><span id="_Toc477854793" class="anchor"><span id="_Toc477854843" class="anchor"><span id="_Toc477855281" class="anchor"><span id="_Toc477864403" class="anchor"><span id="_Toc477864663" class="anchor"><span id="_Toc477867201" class="anchor"><span id="_Toc477867251" class="anchor"><span id="_Toc457235515" class="anchor"><span id="_Toc457235573" class="anchor"><span id="_Toc457240356" class="anchor"><span id="_Toc457240413" class="anchor"><span id="_Toc463242613" class="anchor"><span id="_Toc463242685" class="anchor"><span id="_Toc463242757" class="anchor"><span id="_Toc464455166" class="anchor"><span id="_Toc464629649" class="anchor"><span id="_Toc465174032" class="anchor"><span id="_Toc465174109" class="anchor"><span id="_Toc465174184" class="anchor"><span id="_Toc465190505" class="anchor"><span id="_Toc465190552" class="anchor"><span id="_Toc465960451" class="anchor"><span id="_Toc465960543" class="anchor"><span id="_Toc465966247" class="anchor"><span id="_Toc465967070" class="anchor"><span id="_Toc465967491" class="anchor"><span id="_Toc465967536" class="anchor"><span id="_Toc466468259" class="anchor"><span id="_Toc466468304" class="anchor"><span id="_Toc466469048" class="anchor"><span id="_Toc467171946" class="anchor"><span id="_Toc467171994" class="anchor"><span id="_Toc467172042" class="anchor"><span id="_Toc467172090" class="anchor"><span id="_Toc471213443" class="anchor"><span id="_Toc471213492" class="anchor"><span id="_Toc471224698" class="anchor"><span id="_Toc471224747" class="anchor"><span id="_Toc471288161" class="anchor"><span id="_Toc471288210" class="anchor"><span id="_Toc471393254" class="anchor"><span id="_Toc471908173" class="anchor"><span id="_Toc471908226" class="anchor"><span id="_Toc472006566" class="anchor"><span id="_Toc475535083" class="anchor"><span id="_Toc475535139" class="anchor"><span id="_Toc475535195" class="anchor"><span id="_Toc475536850" class="anchor"><span id="_Toc475536900" class="anchor"><span id="_Toc475536950" class="anchor"><span id="_Toc477854794" class="anchor"><span id="_Toc477854844" class="anchor"><span id="_Toc477855282" class="anchor"><span id="_Toc477864404" class="anchor"><span id="_Toc477864664" class="anchor"><span id="_Toc477867202" class="anchor"><span id="_Toc477867252" class="anchor"><span id="_Toc457235516" class="anchor"><span id="_Toc457235574" class="anchor"><span id="_Toc457240357" class="anchor"><span id="_Toc457240414" class="anchor"><span id="_Toc463242614" class="anchor"><span id="_Toc463242686" class="anchor"><span id="_Toc463242758" class="anchor"><span id="_Toc464455167" class="anchor"><span id="_Toc464629650" class="anchor"><span id="_Toc465174033" class="anchor"><span id="_Toc465174110" class="anchor"><span id="_Toc465174185" class="anchor"><span id="_Toc465190506" class="anchor"><span id="_Toc465190553" class="anchor"><span id="_Toc465960452" class="anchor"><span id="_Toc465960544" class="anchor"><span id="_Toc465966248" class="anchor"><span id="_Toc465967071" class="anchor"><span id="_Toc465967492" class="anchor"><span id="_Toc465967537" class="anchor"><span id="_Toc466468260" class="anchor"><span id="_Toc466468305" class="anchor"><span id="_Toc466469049" class="anchor"><span id="_Toc467171947" class="anchor"><span id="_Toc467171995" class="anchor"><span id="_Toc467172043" class="anchor"><span id="_Toc467172091" class="anchor"><span id="_Toc471213444" class="anchor"><span id="_Toc471213493" class="anchor"><span id="_Toc471224699" class="anchor"><span id="_Toc471224748" class="anchor"><span id="_Toc471288162" class="anchor"><span id="_Toc471288211" class="anchor"><span id="_Toc471393255" class="anchor"><span id="_Toc471908174" class="anchor"><span id="_Toc471908227" class="anchor"><span id="_Toc472006567" class="anchor"><span id="_Toc475535084" class="anchor"><span id="_Toc475535140" class="anchor"><span id="_Toc475535196" class="anchor"><span id="_Toc475536851" class="anchor"><span id="_Toc475536901" class="anchor"><span id="_Toc475536951" class="anchor"><span id="_Toc477854795" class="anchor"><span id="_Toc477854845" class="anchor"><span id="_Toc477855283" class="anchor"><span id="_Toc477864405" class="anchor"><span id="_Toc477864665" class="anchor"><span id="_Toc477867203" class="anchor"><span id="_Toc477867253" class="anchor"><span id="_Toc457235517" class="anchor"><span id="_Toc457235575" class="anchor"><span id="_Toc457240358" class="anchor"><span id="_Toc457240415" class="anchor"><span id="_Toc463242615" class="anchor"><span id="_Toc463242687" class="anchor"><span id="_Toc463242759" class="anchor"><span id="_Toc464455168" class="anchor"><span id="_Toc464629651" class="anchor"><span id="_Toc465174034" class="anchor"><span id="_Toc465174111" class="anchor"><span id="_Toc465174186" class="anchor"><span id="_Toc465190507" class="anchor"><span id="_Toc465190554" class="anchor"><span id="_Toc465960453" class="anchor"><span id="_Toc465960545" class="anchor"><span id="_Toc465966249" class="anchor"><span id="_Toc465967072" class="anchor"><span id="_Toc465967493" class="anchor"><span id="_Toc465967538" class="anchor"><span id="_Toc466468261" class="anchor"><span id="_Toc466468306" class="anchor"><span id="_Toc466469050" class="anchor"><span id="_Toc467171948" class="anchor"><span id="_Toc467171996" class="anchor"><span id="_Toc467172044" class="anchor"><span id="_Toc467172092" class="anchor"><span id="_Toc471213445" class="anchor"><span id="_Toc471213494" class="anchor"><span id="_Toc471224700" class="anchor"><span id="_Toc471224749" class="anchor"><span id="_Toc471288163" class="anchor"><span id="_Toc471288212" class="anchor"><span id="_Toc471393256" class="anchor"><span id="_Toc471908175" class="anchor"><span id="_Toc471908228" class="anchor"><span id="_Toc472006568" class="anchor"><span id="_Toc475535085" class="anchor"><span id="_Toc475535141" class="anchor"><span id="_Toc475535197" class="anchor"><span id="_Toc475536852" class="anchor"><span id="_Toc475536902" class="anchor"><span id="_Toc475536952" class="anchor"><span id="_Toc477854796" class="anchor"><span id="_Toc477854846" class="anchor"><span id="_Toc477855284" class="anchor"><span id="_Toc477864406" class="anchor"><span id="_Toc477864666" class="anchor"><span id="_Toc477867204" class="anchor"><span id="_Toc477867254" class="anchor"><span id="_Toc457235518" class="anchor"><span id="_Toc457235576" class="anchor"><span id="_Toc457240359" class="anchor"><span id="_Toc457240416" class="anchor"><span id="_Toc463242616" class="anchor"><span id="_Toc463242688" class="anchor"><span id="_Toc463242760" class="anchor"><span id="_Toc464455169" class="anchor"><span id="_Toc464629652" class="anchor"><span id="_Toc465174035" class="anchor"><span id="_Toc465174112" class="anchor"><span id="_Toc465174187" class="anchor"><span id="_Toc465190508" class="anchor"><span id="_Toc465190555" class="anchor"><span id="_Toc465960454" class="anchor"><span id="_Toc465960546" class="anchor"><span id="_Toc465966250" class="anchor"><span id="_Toc465967073" class="anchor"><span id="_Toc465967494" class="anchor"><span id="_Toc465967539" class="anchor"><span id="_Toc466468262" class="anchor"><span id="_Toc466468307" class="anchor"><span id="_Toc466469051" class="anchor"><span id="_Toc467171949" class="anchor"><span id="_Toc467171997" class="anchor"><span id="_Toc467172045" class="anchor"><span id="_Toc467172093" class="anchor"><span id="_Toc471213446" class="anchor"><span id="_Toc471213495" class="anchor"><span id="_Toc471224701" class="anchor"><span id="_Toc471224750" class="anchor"><span id="_Toc471288164" class="anchor"><span id="_Toc471288213" class="anchor"><span id="_Toc471393257" class="anchor"><span id="_Toc471908176" class="anchor"><span id="_Toc471908229" class="anchor"><span id="_Toc472006569" class="anchor"><span id="_Toc475535086" class="anchor"><span id="_Toc475535142" class="anchor"><span id="_Toc475535198" class="anchor"><span id="_Toc475536853" class="anchor"><span id="_Toc475536903" class="anchor"><span id="_Toc475536953" class="anchor"><span id="_Toc477854797" class="anchor"><span id="_Toc477854847" class="anchor"><span id="_Toc477855285" class="anchor"><span id="_Toc477864407" class="anchor"><span id="_Toc477864667" class="anchor"><span id="_Toc477867205" class="anchor"><span id="_Toc477867255" class="anchor"><span id="_Toc457235519" class="anchor"><span id="_Toc457235577" class="anchor"><span id="_Toc457240360" class="anchor"><span id="_Toc457240417" class="anchor"><span id="_Toc463242617" class="anchor"><span id="_Toc463242689" class="anchor"><span id="_Toc463242761" class="anchor"><span id="_Toc464455170" class="anchor"><span id="_Toc464629653" class="anchor"><span id="_Toc465174036" class="anchor"><span id="_Toc465174113" class="anchor"><span id="_Toc465174188" class="anchor"><span id="_Toc465190509" class="anchor"><span id="_Toc465190556" class="anchor"><span id="_Toc465960455" class="anchor"><span id="_Toc465960547" class="anchor"><span id="_Toc465966251" class="anchor"><span id="_Toc465967074" class="anchor"><span id="_Toc465967495" class="anchor"><span id="_Toc465967540" class="anchor"><span id="_Toc466468263" class="anchor"><span id="_Toc466468308" class="anchor"><span id="_Toc466469052" class="anchor"><span id="_Toc467171950" class="anchor"><span id="_Toc467171998" class="anchor"><span id="_Toc467172046" class="anchor"><span id="_Toc467172094" class="anchor"><span id="_Toc471213447" class="anchor"><span id="_Toc471213496" class="anchor"><span id="_Toc471224702" class="anchor"><span id="_Toc471224751" class="anchor"><span id="_Toc471288165" class="anchor"><span id="_Toc471288214" class="anchor"><span id="_Toc471393258" class="anchor"><span id="_Toc471908177" class="anchor"><span id="_Toc471908230" class="anchor"><span id="_Toc472006570" class="anchor"><span id="_Toc475535087" class="anchor"><span id="_Toc475535143" class="anchor"><span id="_Toc475535199" class="anchor"><span id="_Toc475536854" class="anchor"><span id="_Toc475536904" class="anchor"><span id="_Toc475536954" class="anchor"><span id="_Toc477854798" class="anchor"><span id="_Toc477854848" class="anchor"><span id="_Toc477855286" class="anchor"><span id="_Toc477864408" class="anchor"><span id="_Toc477864668" class="anchor"><span id="_Toc477867206" class="anchor"><span id="_Toc477867256" class="anchor"><span id="_Toc457235520" class="anchor"><span id="_Toc457235578" class="anchor"><span id="_Toc457240361" class="anchor"><span id="_Toc457240418" class="anchor"><span id="_Toc463242618" class="anchor"><span id="_Toc463242690" class="anchor"><span id="_Toc463242762" class="anchor"><span id="_Toc464455171" class="anchor"><span id="_Toc464629654" class="anchor"><span id="_Toc465174037" class="anchor"><span id="_Toc465174114" class="anchor"><span id="_Toc465174189" class="anchor"><span id="_Toc465190510" class="anchor"><span id="_Toc465190557" class="anchor"><span id="_Toc465960456" class="anchor"><span id="_Toc465960548" class="anchor"><span id="_Toc465966252" class="anchor"><span id="_Toc465967075" class="anchor"><span id="_Toc465967496" class="anchor"><span id="_Toc465967541" class="anchor"><span id="_Toc466468264" class="anchor"><span id="_Toc466468309" class="anchor"><span id="_Toc466469053" class="anchor"><span id="_Toc467171951" class="anchor"><span id="_Toc467171999" class="anchor"><span id="_Toc467172047" class="anchor"><span id="_Toc467172095" class="anchor"><span id="_Toc471213448" class="anchor"><span id="_Toc471213497" class="anchor"><span id="_Toc471224703" class="anchor"><span id="_Toc471224752" class="anchor"><span id="_Toc471288166" class="anchor"><span id="_Toc471288215" class="anchor"><span id="_Toc471393259" class="anchor"><span id="_Toc471908178" class="anchor"><span id="_Toc471908231" class="anchor"><span id="_Toc472006571" class="anchor"><span id="_Toc475535088" class="anchor"><span id="_Toc475535144" class="anchor"><span id="_Toc475535200" class="anchor"><span id="_Toc475536855" class="anchor"><span id="_Toc475536905" class="anchor"><span id="_Toc475536955" class="anchor"><span id="_Toc477854799" class="anchor"><span id="_Toc477854849" class="anchor"><span id="_Toc477855287" class="anchor"><span id="_Toc477864409" class="anchor"><span id="_Toc477864669" class="anchor"><span id="_Toc477867207" class="anchor"><span id="_Toc477867257" class="anchor"><span id="_Toc457235521" class="anchor"><span id="_Toc457235579" class="anchor"><span id="_Toc457240362" class="anchor"><span id="_Toc457240419" class="anchor"><span id="_Toc463242619" class="anchor"><span id="_Toc463242691" class="anchor"><span id="_Toc463242763" class="anchor"><span id="_Toc464455172" class="anchor"><span id="_Toc464629655" class="anchor"><span id="_Toc465174038" class="anchor"><span id="_Toc465174115" class="anchor"><span id="_Toc465174190" class="anchor"><span id="_Toc465190511" class="anchor"><span id="_Toc465190558" class="anchor"><span id="_Toc465960457" class="anchor"><span id="_Toc465960549" class="anchor"><span id="_Toc465966253" class="anchor"><span id="_Toc465967076" class="anchor"><span id="_Toc465967497" class="anchor"><span id="_Toc465967542" class="anchor"><span id="_Toc466468265" class="anchor"><span id="_Toc466468310" class="anchor"><span id="_Toc466469054" class="anchor"><span id="_Toc467171952" class="anchor"><span id="_Toc467172000" class="anchor"><span id="_Toc467172048" class="anchor"><span id="_Toc467172096" class="anchor"><span id="_Toc471213449" class="anchor"><span id="_Toc471213498" class="anchor"><span id="_Toc471224704" class="anchor"><span id="_Toc471224753" class="anchor"><span id="_Toc471288167" class="anchor"><span id="_Toc471288216" class="anchor"><span id="_Toc471393260" class="anchor"><span id="_Toc471908179" class="anchor"><span id="_Toc471908232" class="anchor"><span id="_Toc472006572" class="anchor"><span id="_Toc475535089" class="anchor"><span id="_Toc475535145" class="anchor"><span id="_Toc475535201" class="anchor"><span id="_Toc475536856" class="anchor"><span id="_Toc475536906" class="anchor"><span id="_Toc475536956" class="anchor"><span id="_Toc477854800" class="anchor"><span id="_Toc477854850" class="anchor"><span id="_Toc477855288" class="anchor"><span id="_Toc477864410" class="anchor"><span id="_Toc477864670" class="anchor"><span id="_Toc477867208" class="anchor"><span id="_Toc477867258" class="anchor"><span id="_Toc457235522" class="anchor"><span id="_Toc457235580" class="anchor"><span id="_Toc457240363" class="anchor"><span id="_Toc457240420" class="anchor"><span id="_Toc463242620" class="anchor"><span id="_Toc463242692" class="anchor"><span id="_Toc463242764" class="anchor"><span id="_Toc464455173" class="anchor"><span id="_Toc464629656" class="anchor"><span id="_Toc465174039" class="anchor"><span id="_Toc465174116" class="anchor"><span id="_Toc465174191" class="anchor"><span id="_Toc465190512" class="anchor"><span id="_Toc465190559" class="anchor"><span id="_Toc465960458" class="anchor"><span id="_Toc465960550" class="anchor"><span id="_Toc465966254" class="anchor"><span id="_Toc465967077" class="anchor"><span id="_Toc465967498" class="anchor"><span id="_Toc465967543" class="anchor"><span id="_Toc466468266" class="anchor"><span id="_Toc466468311" class="anchor"><span id="_Toc466469055" class="anchor"><span id="_Toc467171953" class="anchor"><span id="_Toc467172001" class="anchor"><span id="_Toc467172049" class="anchor"><span id="_Toc467172097" class="anchor"><span id="_Toc471213450" class="anchor"><span id="_Toc471213499" class="anchor"><span id="_Toc471224705" class="anchor"><span id="_Toc471224754" class="anchor"><span id="_Toc471288168" class="anchor"><span id="_Toc471288217" class="anchor"><span id="_Toc471393261" class="anchor"><span id="_Toc471908180" class="anchor"><span id="_Toc471908233" class="anchor"><span id="_Toc472006573" class="anchor"><span id="_Toc475535090" class="anchor"><span id="_Toc475535146" class="anchor"><span id="_Toc475535202" class="anchor"><span id="_Toc475536857" class="anchor"><span id="_Toc475536907" class="anchor"><span id="_Toc475536957" class="anchor"><span id="_Toc477854801" class="anchor"><span id="_Toc477854851" class="anchor"><span id="_Toc477855289" class="anchor"><span id="_Toc477864411" class="anchor"><span id="_Toc477864671" class="anchor"><span id="_Toc477867209" class="anchor"><span id="_Toc477867259" class="anchor"><span id="_Toc457235523" class="anchor"><span id="_Toc457235581" class="anchor"><span id="_Toc457240364" class="anchor"><span id="_Toc457240421" class="anchor"><span id="_Toc463242621" class="anchor"><span id="_Toc463242693" class="anchor"><span id="_Toc463242765" class="anchor"><span id="_Toc464455174" class="anchor"><span id="_Toc464629657" class="anchor"><span id="_Toc465174040" class="anchor"><span id="_Toc465174117" class="anchor"><span id="_Toc465174192" class="anchor"><span id="_Toc465190513" class="anchor"><span id="_Toc465190560" class="anchor"><span id="_Toc465960459" class="anchor"><span id="_Toc465960551" class="anchor"><span id="_Toc465966255" class="anchor"><span id="_Toc465967078" class="anchor"><span id="_Toc465967499" class="anchor"><span id="_Toc465967544" class="anchor"><span id="_Toc466468267" class="anchor"><span id="_Toc466468312" class="anchor"><span id="_Toc466469056" class="anchor"><span id="_Toc467171954" class="anchor"><span id="_Toc467172002" class="anchor"><span id="_Toc467172050" class="anchor"><span id="_Toc467172098" class="anchor"><span id="_Toc471213451" class="anchor"><span id="_Toc471213500" class="anchor"><span id="_Toc471224706" class="anchor"><span id="_Toc471224755" class="anchor"><span id="_Toc471288169" class="anchor"><span id="_Toc471288218" class="anchor"><span id="_Toc471393262" class="anchor"><span id="_Toc471908181" class="anchor"><span id="_Toc471908234" class="anchor"><span id="_Toc472006574" class="anchor"><span id="_Toc475535091" class="anchor"><span id="_Toc475535147" class="anchor"><span id="_Toc475535203" class="anchor"><span id="_Toc475536858" class="anchor"><span id="_Toc475536908" class="anchor"><span id="_Toc475536958" class="anchor"><span id="_Toc477854802" class="anchor"><span id="_Toc477854852" class="anchor"><span id="_Toc477855290" class="anchor"><span id="_Toc477864412" class="anchor"><span id="_Toc477864672" class="anchor"><span id="_Toc477867210" class="anchor"><span id="_Toc477867260" class="anchor"><span id="_Toc480386201" class="anchor"></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span>Model for checking consistency of metadata on media
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

At any point in time the on-media data structures must be in a
consistent state.

###  Consistency Checking

For pmemfile V1.0 we will not have a consistency checker. The use of the
transactions offered in libpmemobj provide a level of consistency that
is sufficient for the first release. Section **<span
style="font-variant:small-caps;">*Persistent Data Structures*</span>**
defines all valid values for each member of the persistent data
structure. This section describes in more detail how we plan to provide
consistency checking when an ‘fsck’ like tool is developed. This section
is intended to aid developers and system administrators in understanding
the valid states of a pmemfile file system.

#### **Pmemobj object consistency**

The integrity of the pmemobj pool is the first consistency check that
will be done. Tools such as pmemcheck and pmempool can be utilized to
verify the object consistency within the pool.

#### Pmemfile object consistency

All on media Pmemfile data structures are pmemobj objects. This section
describes, at a high level what a consistency checker will do for
Pmemfile.

##### Inodes

The first step in checking for Pmemfile consistency will be to process
all inodes:

The following information will be gathered and cached in support of
checking inode consistency:

-   list of directory inodes

-   list of file inodes

-   list of inodes in use(nlink &gt;0)

-   list of blocks in use

-   list of duplicate blocks

-   list of inodes with bad fields(as noted above)

-   list of inodes which are symlinks

-   list of orphaned inodes

The inode fields are checked:

-   mode field is legal

-   size and block count fields are correct

-   Blocks are not in use by another inode

-   Broken symlinks (no file on the other end of the symlink)

    1.  ##### Directories inodes

Verification of directories includes:

-   That a directory entry has at least two entries, '.' and '..'

-   '.' is the first entry and it's inode should be that of the
    directory

-   '..' is the 2nd entry in the directory and it’s inode is that of the
    parent directory

-   The directory inode number should refer to an in use inode.

    -   Not on orphaned list

-   The directory entry name is a null terminated string and is not
    larger than PMEMFILE\_MAX\_FILE\_NAME + 1 in size.

-   The number of elements in the dirents\[\] structure is the same as
    the value for num\_elements.

    1.  ##### Directories and directory entries

<!-- -->

-   Finds file system root directory and marks it as processed.

<!-- -->

-   Then, for each directory it attempts to trace up the file system
    tree until it reaches a directory that has been
    previously processed. This previously processed parent directory
    means that it has been found to be attached to the file system root.

-   If we cannot get to a previously verified root directory of the path
    we are currently traversing we mark this directory as detached and
    the following data is stored:

    -   inode number

    -   permissions

    -   owner, group

    -   size

    -   date

        1.  ##### Super Block

Validate:

-   There is a super block object for the Pmemfile file system.

-   The values for the super block are valid:

    -   Valid root inode

    -   Valid version

        1.  ##### Repair

Repair of a file system can be managed, in some cases, without user
intervention. The following issues can be repaired without intervention:

-   No root inode

    -   To do this all directories must be attached except to the root
        inode which is missing. This means all directories below the
        root inode must be marked as processed.

<!-- -->

-   Broken symlinks

    -   Symlink inode will be deleted

-   Duplicate blocks

    -   File will be cloned, using current block data and multiple block
        reference inodes will be deleted

-   Orphaned inode list

    -   Process list, if inode is not on media but in orphaned inode
        list, just remove from list

    -   Otherwise, process as normal

-   File size

-   Some inode flags and modes

-   Missing ‘.’ or ‘..’ entries if all other directory data is clean

    1.  ##### When repair is not possible

In the following cases automated repair will not be offered:

-   Detached directories.

    -   If root inode but not all directories are attached to a path
        which leads to root

-   Corrupted inode fields

-   Corrupted/incorrect versions

1.  Error Handling
    ==============

    1.  Error Handling
        --------------

Pmemfile will return valid errors for all operations supported. The
system call interface layer will return valid errno’s for each
corresponding call. The Pmemfile POSIX library will set errno as
appropriate and return errors as defined in the man page linked in
Section **<span style="font-variant:small-caps;">*libpmemfile-posix.3
manpage.*</span>**

1.  <span id="_Ref453008964" class="anchor"><span id="_Toc480386205" class="anchor"></span></span>Appendix A
    ========================================================================================================

    1.  <span id="_Ref475539283" class="anchor"><span id="_Toc480386206" class="anchor"></span></span>libpmemfile.1 manpage
        -------------------------------------------------------------------------------------------------------------------

[libpmemfile.1.txt](https://raw.githubusercontent.com/sarahjelinek/pmemfile/mpage/doc/generated/libpmemfile.1.txt)

<span id="_Ref475541027" class="anchor"><span id="_Toc480386207" class="anchor"></span></span>libpmemfile-posix.3 manpage
-------------------------------------------------------------------------------------------------------------------------

[libpmemfile-posix.3.txt](https://raw.githubusercontent.com/sarahjelinek/pmemfile/mpage/doc/generated/libpmemfile-posix.3.txt)
