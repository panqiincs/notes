# The Linux Programming Interface

## PREFACE

### Subject

This book describe the Linux programming interface--the system calls, library functions, and other low-level interfaces provided by Linux. This set of low-level interfaces is sometimes also know as the __system programming__ interface.

### About the author 

The author is a maintainer of the [__man-pages__](https://www.kernel.org/doc/man-pages/) project.


## 1: HISTORY AND STANDARDS

Unix, C, POSIX, SUS.


## 2: FUNDAMENTAL CONCEPTS

* Kernel
* Shell
* Users and Groups
* Single Directory Hierarchy, Directories, Links, and Files
* File I/O Model
* Programs
* Processes
* Memory Mappings
* Static and Shared Libraries
* IPC and Synchronization
* Signals
* Threads
* ...


## 3: SYSTEM PROGRAMMING CONCEPTS

### 3.1 System Calls

A _system call_ is a controlled entry point into the kernel, allowing a process to request that the kernel perform some actions on the process's behalf.

* A system call changes the processor state from user mode to kernel mode
* The set of system calls is fixed. Each system call is identified by a unique number.
* Each system call may have a set of arguments that specify information to be transferred from user space to kernel space and vice versa.

More details about the steps in the execution of a system call, see the book. **I want to write a post for it**.

### 3.2 Library Functions

Some library functions make use of system calls, some do not.

### 3.3 The standard C library; The GNU C Library(_glibc_)

Ways to etermine the version of _glibc_ on the system.

### 3.4 Handling Errors from System Calls and Library Functions

It is good practice to check the status value indicating whether a call succeeds or fails.

When a system call fails, it sets the global integer variable _errno_ to a positive value that identifies the specific error.

The _perror()_ and _strerror()_ library functions can print error messages based on the _errno_ value.

### 3.5 Notes on the Example Programs in This Book

### 3.6 Portability Issues


## 4: FILE I/O: THE UNIVERSAL I/O MODEL

### 4.1 Overview

All system calls for performing I/O refer to open files using a __file descriptor__. Everything is a file, including pipes, FIFOs, sockets, terminals, devices and regular files. Each **process** has its own set of file descriptors.

Three standard file descriptors: 0(stdin), 1(stdout), 2(stderr). The shell normally operates with these three file descriptors always open.

Four key system calls for performing file I/O:

* `fd = open(pathname, flags, mode)`
* `numread = read(fd, buffer, count)`
* `numwritten = write(fd, buffer, count)`
* `status = close(fd)`

### 4.2 Universality of I/O

Everything is a file.

### 4.3 Opening a file: _open()_

``` c
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *pathname, int flags, .../* mode_t mode */);
```

* _flag_ argument: specifies the _access mode_ for the file
* _mode_ argument: specifies the permissions to be placed on the file. If the _open()_ call does not specify `O_CREAT`, _mode_ can be ommited

An example:

``` c
/* Open new or existing file for reading and writing, truncating to zero bytes; 
   file permissions read+write for owner, nothing for all others */
fd = open("my file", O_RDWR | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR);

/* Open new or existing file for writing; 
   writes should always append to end of file */
fd = open("my file", O_WRONLY | O_CREAT | O_TRUNC | O_APPEND, S_IRUSR | S_IWUSR);
```

### 4.4 Reading from a File: _read()_

A call to _read()_ may read **less** than the requested number of bytes. For a **regular** file, the probable reason for this is that we were close to the end of the file. When applied to pipes, FIFOs, sockets, or terminals--there are also various circumstances where it may read **fewer** bytes than requested.

### 4.5 Writing to a File: _write()_

A call to _write()_ may write **less** than the requested number of bytes.

### 4.6 Closing a file: _close()_

Closes an open file descriptor, freeing it for subsequent reuse by the process. When a process terminates, all its open file descriptors are automatically closed.

### 4.7 Changing the File Offset: _lseek()_

For each open file, the kernel maintains a _File offset_, which determines the location at which the next _read()_ or _write()_ will occur.

``` c
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
```

_whence_ can be set to `SEEK_SET`,`SEEK_CUR` and `SEEK_END`.

#### File holes

Seek past the end of a file, a call to _read()_ returns 0(EOF), and a call to _write()_ creates a _file hole_. Reading from holes return a buffer of null bytes(0).

File holes does not take up disk space. The file system does not allocate any disk blocks for a hole until, data is written into it later.

### 4.8 Operations Outside the Universal I/O Model: _ioctl()_

A general-purpose mechanism for performing file and device operations that fall outside the universal I/O model.


## 5: FILE I/O: FURTHER DETAILS

### 5.1 Atomicity and Race Conditions

All **system calls** are executed atomically.

#### Creating a file exclusively.

Specifying `O_EXCL` in conjunction with `O_CREAT` causes _open()_ to return an error if the file already exists. This provides a way for a process to eusure that it is the creator of a file. An example shows a **race condition** will occur when `O_EXCL` is absent.

#### Appending data to a file

Race condition occurs when multiple processes append data to the same file. Open a file with `O_APPEND` flag guarantees the seek to the next byte past the end of the file and the write operation happen atomically, which can avoid the race condition.

### 5.2 File Control Operations: _fcntl()_

``` c
#include <fcntl.h>

int fcntl(int fd, int cmd, ...);
```

The third argument to _fcntl()_ can be of different types, or it can be ommited.

### 5.3 Open File Status Flags

You can use _fcntl()_ to retrieve or modify the access mode and open file status flags of an open file.(These are the values set by the _flags_ argument specified in the call to _open()_.)

* To retrieve, specify _cmd_ as `F_GETEL`
* To modify, firstly retrieve a copy of the existing flags, then modify the bits, and finally specify _cmd_ as `F_SETEL` to update the flags.

### 5.4 Relationship Between File Descriptors and Open Files

This section is **important**.

It is possible--and useful--to have multiple descriptors referring to the same open files. These file descriptors may be open in the same process or different processes.

...see more details in the book, try to understand Figure 5-2.

### 5.5 Duplicating File Descriptors

``` c
#include <unistd.h>

int dup(int oldfd);
int dup2(int oldfd, int newfd);
```

### 5.6 File I/O at a Specified Offset: _pread()_ and _pwrite()_

Calling _pread()_ is equivalent to _atomically_ perform _lseek()_ and _read()_ in a right order.

These system calls can be particularly useful in multithreaded applications. Using the two system calls, multiple threads can simultaneously perform I/O on the same file descriptor without being affected by changes made to the file offset by other threads.

### 5.7 Scatter-Gather I/O: _readv()_ and _writev()_

Instead of accepting a single buffer of data to be read or written, these functions transfer multiple buffers of data in a single system call.

### 5.8 Truncating a File: _truncate()_ and _ftruncate()_

They set the size of a file to the value specified by _length_.

### 5.9 Nonblocking I/O

Non blocking I/O mode can be used with devices, pipes, FIFOs, and sockets. `O_NONBLOCK` is generally ignored for regular files.

### 5.10 I/O on Large Files

### 5.11 The /dev/fd Directory

### 5.12 Creating Temporary Files


## 6: PROCESSES

### 6.1 Processes and Programs

A _process_ is an instance of an executing program.

A _program_ is a file containing a range of information that describes how to construct a process at run time.

One program may be used to construct many processes, many processes may be running the same program.

### 6.2 Process ID and Parent Process ID

``` c
#include <unistd.h>

pid_t getpid(void);
pid_t getppid(void);
```

If a child process becomes orphaned because its "birth" parent terminates, then the child is adopted by the _init_ process.

### 6.3 Memory Layout of a process 

Segments:

* _text segments_ contains machine-language instructions
* _initialized data segment_ contains global and static variables that are explicitly initialized
* _uninitialized segments_ contains global and static variables that are not explicitly initialized
* _stack_ stores function's local variables(so-called automatic variables), arguments, and return value.
* _heap_ dynamically allocated memory.

C program environment on most UNIX implementations provides three global symbols: _etext_, _edate_ and _eend_. The can be used to obtain the addresses of the next byte past, respectively, the end of the program text, the end of the initialized data segment, and the end of the uninitialized data segment. See Figure 6-1.

Please see Figure 6-1, important.

### 6.4 Virtual Memory Management

The previous discussion talks about the layout in _virtual memory_.

_Virtual memory management_ can make efficient use of both the CPU and RAM(physical memory) by exploiting a property that is typical of most programs: **loclity of reference**: _spatial locality_ and _temporal locality_.

Keywords: pages, page frames, resident set, swap area, page fault, page table, virtual address space...

Very important knowledge, see more details in a OS book.

### 6.5 The Stack and Stack Frames

_Stack pointer_, tracks the current top of the stack. 

The term _user stack_ is used to distinguish the stack from _kernel stack_. The kernel stack is a per-process memory region maintained in kernel memory that is used as the stack for execution of the functions called internally during the execution of a system call.

### 6.6 Command-Line Arguments (_argc_, _argv_)

### 6.7 Environment List

### 6.8 Performing a Nonlocal Goto: _setjmp()_ and _longjmp()_

The _setjmp()_ and _longjmp()_ functions are used to perform a _nonlocal goto_. The term _nonlocal_ refers to the fact that the target of the goto is a location somewhere outside the currently executing function.

One restriction of C's goto is that it is not possible to jump out of the current function into another function.

In some cases, coding would be simpler if we could jump from the middle of the nested function call back to one of the functions that called it. This is the functionality that _setjmp()_ and _longjmp()_ provide. They are useful for dealing with errors and interrupts encountered in a low-level subroutine of a program.

``` c
#include <setjmp.h>

int setjmp(jmp_buf env);

void longjmp(jmp_buf env, int val);
```

Calling _setjmp()_ establishes a target for later jump performed by _longjmp()_. _setjmp()_ saves the stack context/environment in _env_ for later use by _longjmp()_.

#### Restrictions on the use of _setjmp()_



## 7: MEMORY ALLOCATION

### 7.1 Allocating Memory on the Heap

The current limit of the heap is referred to as the _program break_(see Figure 6-1). C programs normally use the _malloc_ family functions, these functions are based on _brk()_ and _sbrk()_.

#### 7.1.1 Adjusting the Program Break: _brk()_ and _sbrk()_

Initially, the program break lies just past the end of the uninitialized data segment(the same location as _eend_, see Figure 6-1). After the program break is increased, the program may access any address in the newly allocated area, but no physical memory pages are allocated yet. The kernel automatically allocates new physical pages on the first attempt by the process to access addresses in those pages.

Use the following two system calls for manipulating the program break:

``` c
#include <unistd.h>

int brk(void *end_data_segment);

void *sbrk(intptr_t increment);
```

The _brk()_ system call sets the program break to the location specified by _end\_data\_segment_. Since virtual memory is allocated in units of pages, _end\_data\_segment_ is effectively rounded up the the next page boundary. Attemps to set the program break below its initial value are likely to result in unexpected behavior, such as segment fault when trying to access data in now nonexistent parts of the initialized or uninitialized data segments.

A call to _sbrk()_ adjusts the program break by adding _increment_ to it. On success, it returns the previous address of the program break.

#### 7.1.2 Allocating Memory on the Heap: _malloc()_ and _free()_

``` c
#include <stdlib.h>

void *malloc(size_t size);

void free(void *ptr);
```

Although the possibility of failure in allocating memory is small, all calls to _malloc()_, and the related functions that we describe later, should check for the error return. In general, _free()_ does not lower the program break, but instead adds the block of memory to a list of free blocks that are recycled by future calls to _malloc()_ for several reasons:

* The block memory being freed is typically somewhere in the middle of the heap, rather than at the end, so that lowering the program break is not possible.
* It minimizes the number of _sbrk()_ calls that the program must perform.
* In may cases, lowering the break would not help programs that allocate large amount of memory, since they typically tend to hold on to allocated memory or repeatedly release and reallocate memory, rather than release it all and then continue to run for an extended period of time.

Making any use of _ptr_ after the call to _free()_--for example, passing it to _free()_ a second time--is an error that can lead to unpredictable results.

#### 7.1.3 Implementation of _malloc()_ and _free()_

**Important!**Write your own version for _malloc()_ and _free()_(Exercise 7-2).

To avoid errors, observe the following rules:

* After we allocate a block of memory, we should be careful not to touch any bytes **outside** the range of that block.
* It is an error to free the same piece of allocated memory **more than once**.
* Never call _free()_ with a pointer value that wasn't obtained by a call to one of the functions in the _malloc_ package.
* If we are writing a **long-running** program that repeatedly allocates memory for various purposes, we should ensure that we deallocate any memory after we have finished using it.

Tools and libraries for _malloc_ debugging:

* _mtrace()_ and _muntrace()_
* _mcheck()_ and _mprobe()_
* The `MALLOC_CHECK_` environment variable

Controlling and monitoring the _malloc_ package: _mallopt()_, _mallinfo()_.

#### 7.1.4 Other Methods of Allocating Memory on the Heap

``` c
#include <stdlib.h>

void *calloc(size_t numitems, size_t size);
void *realloc(size_t numitems, size_t size);
```

The _calloc()_ function allocates memory for an array of identical items. The _numitems_ argument specifies how many items to allocate, and _size_ specifies their size. After allocating a block of memory of appropriate size, _calloc()_ returns a pointer to the start of the block(or NULL if failed), _calloc()_ initializes the allocated memory to 0.

The _realloc()_ function is used to resize(usually enlarge) a block of memory previously allocated by one of the functions in the _malloc_ package. The _ptr_ argument is a pointer to the block of memory that is to be resized. The _size_ argument specifies the desired new size of the block. On success _realloc()_ returns a pointer to the location of the resized block. This may be different from its location from its location before the call. On error, _realloc()_ returns NULL and leave the block pointed to by _ptr_ untouched.

Memory allocated using _calloc()_ or _realloc()_ should be deallocated with _free()_.

Allocating aligned memory: _memalign()_ and _posix\_memalign()_. They are designed to allocate memory starting at an address aligned at a specified power-of-two boundary.

### 7-2 Allocating Memory on the Stack: _alloca()_

The _alloca()_ function obtains memory from stack by increasing the size of the stack frame. This is possible because the calling function is the one whose stack frame is, by definition, on the top of the stack. Therefore, there is space above the frame for expansion, which can be accomplished by simply modifying the value of the stack pointer.

We must not call _free()_ to deallocate memory allocated with _alloca()_. Likewise, it is not possible to use _realloc()_ to resize a block of memory allocated by _alloca()_.

We can't use _alloca()_ within a function argument list. Because the stack space allocted by _alloca()_ would appear in the middle of the space.

Advantages over _malloc()_: faster, because it is implemented by the compiler as inline code that directly adjusts the stack pointer; the memory is automatically freed when the stack frame is removed, that is, when the function that called _alloca()_ returns.


## 8: USERS AND GROUPS

Every user has a unique login name and an associated numeric user identifier(UID). Users can belong to one or more groups. Each group also has a unique name and a group identifier(GID).

The primary purpose of user and group IDs is to determine ownership of various system resources and to control the permissions granted to processes accessing those resourses.

### 8.1 The Password File: /etc/passwd

### 8.2 The Shadow Password File: /etc/shadow

### 8.3 The Group File: /etc/group

### 8.4 Retrieving User and Group Information

#### Retrieving records from the password file

``` c
#include <pwd.h>

struct passwd *getpwnam(const char *name);
struct passwd *getpwuid(uid_t uid);

struct passwd {
    char *pw_name;        /* Login name (username) */
    char *pw_password;    /* Encrypted password */
    uid_t pw_uid;         /* User ID */
    gid_t pw_gid;         /* Group ID */
    char *pw_gecos;       /* Comment (user information) */
    char *pw_dir;         /* Initial working (home) directory */
    char *pw_shell;       /* Login shell */
};
```

#### Retrieving records from the group file

``` c
#include <grp.h>

struct passwd *getgrnam(const char *name);
struct passwd *getgrgid(gid_t gid);

struct group {
    char  *gr_name;        /* Group name */
    char  *gr_passwd;      /* Encrypted password (if not password shadowing) */
    gid_t  gr_gid;         /* Group ID */
    char **gr_mem;         /* NULL-terminated array of pointers to names
                              of members listed in /etc/group */
};
```

#### Scanning all records in the password and group files

#### Retrieving records from the shadow password file

### 8.5 Password Encryption and User Authentication

For security reasons, UNIX systems encrypt passwords using a _one-way encryption_ algorithm, which means that there is no method of re-creating the original password from its encrypted form. Therefore the only way of validating a candidate password is to encrypt it using the name method and see if the encrypted result matches the value stored in `/etc/shadow`.


## 9: PROCESS CREDENTIALS

### 9.1 Real User ID and Real Group ID

The real user ID and group ID identifies the user and group to which the process belongs. As part of the login process, a login shell gets its real user and group IDs from the third and fourth fields of the user's password record in the `/etc/passwd` file. When a new process is created, it inherits these identifiers from its parent.

### 9.2 Effective User ID and Effective Group ID

On most UNIX implementations, the effective user ID and group ID, in conjunction with the supplementary group IDS, are used to determine the permissions granted to a process when it tries to perform various operations.

A process with effective user ID 0 (the user ID of root) has all of the privileges of the superuser. Such a process is referred to as a _privileged process_. Certain system calls can be executed only by privileged processes.

Normally, the effective user and group IDs has the same values as the corresponding real IDs. Through the execution of set-user-ID and set-group-ID programs, they can be different.

### 9.3 Set-User-ID and Set-Group-ID Programs

A set-user-ID program allows a process to gain privileges it would not normally have, by setting the process's effective user ID to the same value as the user ID (owner) of the executable file. A set-group-ID program performs the analogous task for the process's effective group ID.

An executable file has two special permission bits: the set-user-ID and set-group-ID bits. (In fact, every file has these two permission bits, but it is their use with executable files that interests us here.) These permission bit are set using the _chmod_ command.

When the set-user-ID and set-group-ID bits are set, then the _x_ that is normally used to indicate that execute permission is set is replaced by an _s_.

Examples of commonly used set-user-ID programs on Linux include: _passwd(1)_, _mount(8)_, _umount(8)_ and so on.

### 9.4 Saved Set-User-ID and Saved Set-Group-ID

They allow programs to temporarily drop and then later reassume privileges.

### 9.5 File-System User ID and File-System Group ID

They are used for determining permissions for accessing files, while the effective IDs are used for other permission checks.

### 9.6 Supplementary Group IDs

A futher set of groups of which the process is considered to be a member for the purpose of permission checking.

### 9.7 Retrieving and Modifying Process Credentials

A lot of details, skip.


## 10: TIME

Within a program, we may be interested in two kinds of time:

* _Real time_: This is the time as measured either from some standard point(_calendar_ time) or from some fixed point (typically the start) in the life of a process(_elasped_ or _wall clock_ time). Measuring elapsed time is useful for a program that takes periodic actions or makes regular measurement from some external input device.

* _Process time_: This is the amount of CPU time used by a process. Measuring process time is useful for checking or optimizing the performance of a program or algorithm.

### 10.1 Calendar Time

UNIX systems represent time internally as a measure of seconds since the Epoch, that is since midnight on the morning of 1 January 1970, Universal Coordinated Time. This is approximately the date when the UNIX system came into being. Calendar time is stored in variables of type _time\_t_.

``` c
#include <sys/time.h>

int gettimeofday(struct timeval *tv, struct timezone *tz);

struct timeval {
    time_t      tv_sec;   /* Seconds since 00:00:00, 1 Jan 1970 UTC */
    suseconds_t tv_usec;  /* Additional microseconds (long it) */
};
```
The _gettimeofday()_ system call returns the calendar time in the buffer pointed by _tv_. The _tz_ argument is now obsolete and should always be specified as NULL.

``` c
#include <time.h>

time_t time(time_t *timep);
```

The _time()_ system call returns the number of seconds since the Epoch. If the _timep_ argument is not NULL, the number of seconds since the Epoch is also placed in the location to which _timep_ points. We often simply use the following call:

``` c
t = time(NULL);
```

### 10.2 Time-Convention Functions

There are a lot of functions used to convert between _time\_t_ values and other time formats, including printable representations. These functions shield us from the complexity brought to such conversions by timezones, daylight saving time(DST) regimes, and localization issues.

#### 10.2.1 Converting time_t to Printable Form

#### 10.2.2 Converting Between time_t and Broken-Down Time

#### 10.2.3 Converting Between Broken-Down Time and Printable Form

### 10.3 Timezones

Deal with different timezones.

### 10.4 Locales

Deal with different languages and different conventions for display information.

### 10.5 Updating the System Clock

Sets the system's calendar time.

### 10.6 The software Clock (Jiffies)

The accuracy of various time-related system calls described in this book is limited to the resolution of the system _software clock_, which measures time in units called _jiffies_. The size of a jiffy is defined by the constant HZ within the kernel source code. This the unit in which the kernel allocates the CPU to processes under the roundrobin time-sharing scheduling algorithm.

### 10.7 Process Time

Process time is the amount of CPU time used by a process since it was created. For recording purposes, the kernel separates CPU time into the following two components: 

* _User CPU Time_ is the amount of time spent executing in user mode. This is the time that it appears to the program that it has access to the CPU.

* _System CPU time_ is the amount of time spent executing in kernel mode. This is the time that the kernel spends executing system calls or performing other tasks on behalf of the program(e.g, servicing page faults).

When run a program from the shell, we can use the _time(1)_ command to obtain both process time values, as well as the real time required to run the program.


## 11: SYSTEM LIMITS AND OPTIONS

The C programming language standards and SUSv3 provides two principal avenues for an application to obtain system limits information:

* Some limits and options can be determined at compile time.
* Other limits and options may vary at run time. SUSv3 defines three functions--_sysconf()_, _pathconf()_, and _fpathconf()_--that an application can call to check these implementation limits and options.

### 11.1 System Limits

### 11.2 Retrieving System Limits (and Options) at Run Time

### 11.3 Retrieving File-Related Limits (and Options) at Run Time

### 11.4 Indeterminate Limits

### 11.5 System Options


## 12: SYSTEM AND PROCESS INFORMATION

### 12.1 The /proc File System

In order to provide easier access to kernel information, many modern UNIX implementations provide a `/proc` virtual file system. This file system resides under the `/proc` directory and contains various files that expose kernel information, allowing processes to conveniently read that information, and change it in some cases, using normal file I/O system calls. The `/proc` file system is said to be virtual because the files and subdirectories that it contains don't reside on a disk. Instead, the kernel creates them "on the fly" as process access them.

#### 12.1.1 Obtaining Information About a Process: /proc/_PID_

For each process on the system, the kernel provides a corresponding directory named `/proc/_PID_`, where _PID_ is the ID of the process.

##### The /proc/_PID_/fd direcotry

##### Threads: the /proc/_PID_/task directory

#### 12.1.2 System Information Under /proc

Various files and subdirectories under `/proc` provides access to system-wide information.

#### 12.1.3 Accessing /proc Files

### 12.2 System Indentification: _uname()_

The _uname()_ system call returns a range of identifying information about the host system on which an application is running, in the structure pointed to by _utsbuf_.

``` c
#include <sys/utsname.h>

int uname(struct utsname *ustbuf);
```


## 13: FILE I/O BUFFERING

### 13.1 Kernel Buffering of File I/O: The Buffer Cache

When working with disk files, the _read()_ and _write()_ system calls don't directly initiate disk access. Instead, they simply copy data between a user-space buffer and a buffer in the kernel _buffer_ cache. Please see the examples in the book.

The aim of this design is to allow _read()_ and _write()_ to be fast, since they don't need to wait on a (slow) disk operation. This design is also efficient, since it reduces the number of disk transfer that the kernel must perform.

#### Effect of buffer size on I/O system call performance

The kernel performs the same number of disk accesses, regardless of whether we perform 1000 writes of a single byte or a single write of a 1000 bytes. However, the latter is preferable, since it requires a single system call, while the former requires 1000. And system calls are expensive.

If we are transferring a large amount of data to or from a file, then by buffering data in large blocks, and thus performing fewer system calls, we can greatly improve I/O performance.

### 13.2 Buffering in the _stdio_ Library

Buffering of data into large blocks to reduce system calls is exactly what is done by the C library I/O functions(e.g., _fprintf()_, _fscanf()_, _fgets()_, _fputs()_, _fputc()_, _fgetc()_) when operating on disk files. Thus, using the _stdio_ library relieves us of task of buffering data for output with _write()_ or input via _read()_.

#### Setting the buffering mode of a _stdio_ stream

The _setvbuf()_ function controls the form of buffering employed by the _stdio_ library.

``` c
#include <stdio.h>

int setvbuf(FILE *stream, char *buf, int mode, size_t size);
```

The _stream_ argument identifies the file stream whose buffering is to be modified. After the stream has been opened, the _setvbuf()_ call must be made before calling any other _stdio_ function on the stream. The _setvbuf()_ call affects the behaviour of all subsequent _stdio_ operations on the specified stream.

The _buf_ and _size_ argument specify the buffer to be used for _stream_. These arguments may be specified in two ways:

* If _buf_ is non-NULL, then it points to a block of memory of _size_ bytes that is to be used as the buffer for _stream_. It should be either statically allocated of dynamically allocated on the heap.
* If _buf_ is NULL, then the _stdio_ library automatically allocates a buffer for use with _stream_.

The _mode_ argument specifies the type of buffering and has one of the following values:

* \_IONBF    Don't buffer I/O. Each _stdio_ library call results in an immediate _write()_ or _read()_ system call. The _buf_ and _size_ arguments are ignored, and can be specified as NULL or 0, respectively. This is the default for _stderr_, so that the error output is guaranteed to appear immediately.

* \_IOLBF    Employ line-buffered I/O. This flag is the default for stream refering to terminal devices. For output streams, data is buffered until a newline charactor is output (unless the buffer fills first). For input streams, data is read a line at a time.

* \_IOFBF    Emply fully buffered I/O. Data is read or written (via calls to _read()_ or _write()_) in units equal to the size of the buffer. This is the default for streams referring to disk files. 

#### Flushing a _stdio_ buffer

Regardless of the current buffering mode, at any time, we can force the dat in a _stdio_ output stream to be written using the _fflush()_ library function.

``` c
#include <stdio.h>

int fflush(FILE *stream);
```

If stream is NULL, _fflush()_ flushes all _stdio_ buffers. 

The _fflush()_ function can also be applied to an input stream. This causes any buffered input to be discarded.(The buffer will be refilled when the program next tries to read from the stream.)

A _stdio_ buffer is automacically flushed when the corresponding stream is closed.

### 13.3 Controlling Kernel Buffering of File I/O

It is possible to force flushing of kernel buffers for output files. Sometimes, this is necessary if an application must ensure that output really has been written to the disk before continuing.

#### Synchronized I/O data integrity and sychronized I/O file integrity

The difference between them involves the _metadata_ describing the file, which the kernel stores along with the data for a file.

_Synchronized I/O data integrity_, this is concerned with ensuring that a file data update transfers sufficient information to allow a later retrieval of that data to proceed.

_Synchronized I/O file integrity_, the difference with this mode of I/O completion is that during a file udpate, _all_ updated file metadata is transferred to disk, even if it is not necessary for the operation of a subsequent read of the file data.

#### System calls for controlling kernel buffering of file I/O

The _fsync()_ system call causes the buffered data and all metadata associated with the open file descriptor _fd_ to be flushed to disk. Calling _fsync()_ forces the file to the synchronized I/O file integrity completion state.

``` c
#include <unistd.h>

int fsync(int fd);
```

An _fsync()_ call returns only after the transfer to the disk device has completed.

The _fdatasync()_ system call operates similarly to _fsync()_, but only forces the file to the synchronized I/O data integrity completion state.

``` c
#include <unistd.h>

int fdatasync(int fd);
```

The _fdatasync()_ system call reduces the number of disk I/O operations than _fsync()_, because changes to file metadata attributes such as the last modification timestamp don't need to be transferred for synchronized I/O data completion. This make a considerable performance difference for applications that are making multiple file updates: because the file data and meta data normally reside on different parts of the disk, updating them both would require repeated seek operations backward and forward across the disk.

The _sync()_ system call causes all kernel buffers containing updated file information(i.e., data blocks, pointer blocks, metadata, and son on) to be flushed to disk.

``` c
#include <unistd.h>

void sync(void);
```

#### Making all writes synchronous: O_SYNC

Specifying the `O_SYNC` flag when calling _open()_ makes all subsequent output _synchronous_:

``` c
fd = open(pathname, O_WRONLY | O_SYNC);
```

After this _open_ call, every _write()_ to the file automacically flushes the file data and metadata to the disk (according to the synchronized I/O file integrity completion).

#### Performance impact of O_SYNC

Using the `O_SYNC` flag (or making frequent calls to _fsync()_, _fdatasync()_, or _sync()_) can strongly affect performance.

#### The O_DSYNC and O_RSYNC flags

The `O_DSYNC` flag causes writes to be performed according to the requirements of synchronized I/O data integrity completion(like _fdatasync()_). This contrasts with `O_SYNC`, which causes writes to be performed according to the requirements of synchronized I/O file integrity completion(like _fsync()_)

The `O_RSYNC` flag is specified in conjunction with either `O_SYNC` or `O_DSYNC`, and extends the write behaviors of these flags to read operations.

### 13.4 Summary of I/O Buffering

The transfer of user data by the _stdio_ library functions to the _stdio_ buffer, which is maintained in user memory space. When this buffer is filled, the _stdio_ library invokes the _write()_ system call, which transfers the data into the kernel buffer cache(maintained in kernel memory). Eventually, the kernel initiates a disk operation to transfer the data to the disk.

### 13.5 Advising the Kernel About I/O Patterns

The _posix\_fadvise()_ system call allows a process to inform the kernel about its likely pattern for accessing file data. The kernel may use this information to optimize the use of the buffer cache, thus improvint I/O performance.

### 13.6 Bypassing the Buffer Cache: Direct I/O

Starting with kernel 2.4, Linux allows an application to bypass the buffer cache when performing disk I/O, thus transferring data directly from user space to a file or disk device. This is sometimes termed _direct I/O_ or _raw I/O_.

For most applications, using direct I/O can considerably degrade performance. This is because the kernel applies a number of optimization to improve the performance of I/O done via the buffer cache. All of these optimizations are lost when we use direct I/O. Direct I/O is intended only for applications with specialized I/O requirements.

We can perform direct I/O either on an individual file or on a block device. To do this, we specify the `O_DIRECT` flag when opening the file or device with _open()_.

#### Alignment restrictions for direct I/O

Restrictions, related to block size.

### 13.7 Mixing Library Functions and System Calls for File I/O

It is possible to mix the use of system calls and the standard C library functions to perform I/O on the same file. The _fileno()_ and _fdopen()_ functions assist us with this task.

``` c
#include <stdio.h>

int fileno(FILE *stream);

FILE *fdopen(int fd, const char *mode);
```

Given a stream, _fileno()_ returns the corresponding file descriptor. This file descriptor can then be used in the usual way with I/O system calls such as _read()_, _write()_, _dup()_, and _fcntl()_.

The _fdopen()_ function is the converse of _fileno()_. Given a file descriptor, it creates a corresponding stream that uses this descriptor for its I/O.

The _fdopen()_ function is especially useful for descriptors referring to files other than regular files. E.g., sockets and pipes.

When using the _stdio_ library functions in conjunction with I/O system calls to perform I/O on disk files, we must keep buffering issues in mind.


## 14: FILE SYSTEMS

### 14.1 Device Special Files (Devices)

A device special file corresponds to a device on the system. Within the kernel, each device type has a corresponding device driver, which handles all I/O requests for the device. A _device driver_ is a unit of kernel code that implements a set of operations that (normally) correspond to input and output actions on an associated piece of hardware. The API provided by device drivers is fixed, and includes operations corresponding to the system calls _open()_, _close()_, _read()_, _write()_, _mmap()_, and _ioctl()_. The consistent interface allows for _universality of I/O_.

Two types of devices:

* _Character devices_ handle data on a charactor-by-character basis. E.g., terminals and keyboards.
* _Block devices_ handle data a block at a time. E.g., disks and tapes.

Device files appear within the file system, usually under the /dev directory.

#### Device IDs

Each device file has a _major ID number_ and a _minor ID number_. The major ID identifies the general class of device, and is used by the kernel to look up the appropriate driver for this type of device. The minor ID uniquely identifies a particular device within a general class.

### 14.2 Disks and Partitions

#### Disk drives

Information on the disk surface is located on a set of concentric circles called _tracks_. Tracks themselves are devided into a number of _sectors_, each of which consists of a series of _physical_ blocks, typically 512 bytes in size, represents the smallest unit of information that the drive can read or write.

Reading and writing information on the disk takes significant time.

#### Disk partitons

Each partition is treated by the kernel as a separate device residing under the /dev directory.

A disk partition may hold any type of information, but usually contains one of the following:

* a _file system_ holding regular files and directories.
* a _data area_ accessed as a raw-mode device, some database management system use this techniqure.
* _swap area_ used by the kernel for memory management.

### 14.3 File Systems

A file system is an organized collection of regular files and directories. Linux supports a wide range variety of file systems.

#### The _ext2_ file system

The use of _ext2_ has declined in favor of various journaling file systems.

#### File-system structure

The basic unit for allocating space in a file system is a _logical_ block, which is some multiple of contiguous physical blocks on the disk device on which the file systme resided.

A file system contains the followint parts:

* _Boot block_: This is always the first block in a file system. It is not used by the file system, rather, it contains information used to boot the operating system. Although only one boot block is needed by the operating system, all file systems have a boot block.
* _Superblock_: This is a single block, immediately following the boot block, which contains parameter information about the file system, including:
  -  the size of the i-node table;
  -  the size of logical blocks in this file system; and
  -  the size of the file system in logical blocks.
Different file systems residing on the same physical device can be of different types and sizes, and have different parameter settings.
* _I-node table_: Each file or directory in the file system has a unique entry in the i-node table. This entry records various information about the file.
* _Data blocks_: The great majority of space in a file system is used for the block of data that form the file and directories residing in the file system.

### 14.4 I-nodes

A file system's i-node table contains one i-node for each file residing in the file system. I-nodes are identified numerically by the sequential location in the i-node table. The information maintained in an i-node includes: file type, owner, group, access permissions, three timestamps, number of hard links to the file, size of the file in bytes and pointer to the data blocks of the file.

#### I-nodes and data block pointers in _ext2_

To locate the file data blocks, the kernel maintains a set of pointers in the i-node. Under _ext2_, each i-node contains 15 pointers. The first 12 point to the location in the file system of the first 12 blocks of the file. The next pointer is a _pointer to a block of pointers_. The 14th pointer is a _double indirect pointer_, and the 15th pointer is a _triple-indirect pointer_. See more details in the book.

### 14.5 The Virtual File Sytems (VFS)

### 14.6 Journaling File Systems

The limitation of _ext2_: after a system crash, a file-system consistency check must be performed on reboot in order to ensure the integrity of the file system, which requires a lot of time for a large file system.

Journaling file systems eliminate the need for lengthy file-system consistency checks after a system crash. A journaling file system records metadata updates to a log file before actual file updates performed. This means that in the event of a system crash, the log file can be replayed to quickly restore the file system to a consistent state.

### 14.7 Single Directory Hierarchy and Mount Points

On Linux, as on other UNIX systems, all files from all file systems reside under a single directory tree. Other file systems are _mounted_ under the root direcoty and appear as subtrees within the overall hierarchy.

The location at which a file system is mounted in the directory tree is called its mount point.

### 14.8 Mounting and Unmounting File Systems

Read this section when necessary, now skipped.

### 14.9 Advanced Mount Features

Read this section when necessary, now skipped.

### 14.10 A Virtual Memory File System: _tmpfs_

Linux supports the notion of _virtual file systems_ that reside in memory. Faster, since no disk access is involved.

The most sophiscated one is the _tmpfs_ file system. It uses not only RAM, but also the swap area.

### 14.11 Obtaining Information About a File System: _statvfs()_

The _statvfs()_ and _fstatvfs()_ library funtions obtain information about a mounted file system.


## 15: FILE ATTRIBUTES

### 15.1 Retrieving File Information: _stat()_

The _stat()_, _lstat()_, and _fstat()_ system calls retrieve information about a file, mostly drawn from the file i-node.

``` c
#include <sys/stat.h>

int stat(const char *pathname, struct stat *statbuf);
int lstat(const char *pathname, struct stat *statbuf);
int fstat(int fd, struct stat *statbuf);

struct stat {
    dev_t     st_dev;        /* IDs of device on which file resides */
    ino_t     st_ino;        /* I-node number of file */
    mode_t    st_mode;       /* File type and permissions */
    nlink_t   st_nlink;      /* Number of (hard) links to file */
    uid_t     st_uid;        /* User ID of file owner */
    gid_t     st_gid;        /* Group ID of file owner */
    dev_t     st_rdev;       /* IDs for device special file */
    off_t     st_size;       /* Total file size (bytes) */
    blksize_t st_blksize;    /* Optimal block size for I/O (bytes) */
    blkcnt_t  st_blocks;     /* Number of (512B) blocks allocated */
    time_t    st_atime;      /* Time of last file access */
    time_t    st_mtime;      /* Time of last file modification */
    time_t    st_ctime;      /* Time of last status change */
}
```

_lstat()_ is similar to _stat()_, except that if the named file is a symbolic link, informaton about the link itself is returned, rather than the file to which the link points.

The _stat()_ and _lstat()_ system calls don't require permissions on the file itself. However, execute(search) permission is required on all of the parent directories specified in _pathname_. The _fstat()_ system call always succeeds, if provided with a vailid file descriptor.

The _stat_ structure clearly shows all information related to a file.

### 15.2 File Timestamps

Various system calls and library functions can change the timestamp fields of files. The _open()_ `O_NOATIME` flag can prevent updates to the last access time of a file.

#### Changing File Timestamps with _utime()_ and _utimes()_

#### Changing File Timestamps with _utimensat()_ and _futimens()_

### 15.3 File Ownership

Each file has an associated user ID (UID) and group ID (GID). These IDs determine which user and group the file belongs to.

#### 15.3.1 Ownership of New Files

When a new file is created, its user ID is taken from the effective user ID of the process. The group ID of the new file may be taken from either the effective group ID of the process or the Group ID of the parent directory. The latter possibility is useful for creating project directories in which all files belong to a particular group and are accessible to the members of that group.

#### 15.3.2 Changint File Ownership: _chown()_, _fchown()_, and _lchown()_

The _chown()_, _fchown()_, and _lchown()_ system calls change the owner and group of a file.

Only a privileged process may use _chown()_ to change the user ID of a file. An unprivileged process can use _chown()_ to change the group ID of a file that it owns to any of the groups of which they are a member. A privileged process can change the group ID of a file to any value.

If the owner or group of a file is changed, then the set-user-ID and set-group-ID permission bits are both turned off. This is a security precaution to ensure that a normal user could not enable the SUID and SGID bit on an executable file and then somehow make it owned by some privileged user or group, thereby gaining that privileged identity when executing the file.

When changing the owner or group of a file, the set-group-ID permission bit is not turned off if the group-execute permission bit is already off or if we are changing the ownership of a directory.

### 15.4 File permissions

#### 15.4.1 Permissions on Regular Files

The file permissions mask divides the world into three categories: _Owner_(also know as _user_), _Group_ and _Other_. Three permissions may be granted to each user category: _Read_, _Write_ and _Execute_. Note that, in order to execute a script, you must have read and execute permissions.

#### 15.4.2 Permissions on Directories

The three permissions are interpreted differently in the case of a directory:

* _Read_: The contents of the directory may be listed.
* _Write_: Files may be created in and removed from the directory. Note that it is not necessary to have any permission on a file itself in order to be able to delete it.
* _Execute_: Files within the directory may be accessed. Execute permission on a directory is sometimes called _search_ permission.

When accessing a file, execute premission is required on all of the directories listed in the pathname.

Read permission on a directory only lets us view the list of filenames in the directory. We must have execute permission on the directory in order to access contents or the i-node information of files in the directory.

Conversely, if we have execute permission on a directory, but not read permission, then we can access a file in the directory if we know its name, but we can't list the contents (i.e., the other filenames in) of the directory. This is a simple and frequrently used technique to control access to the contents of a public directory.

To add or remove files in a directory, we need both execute and write permissions on the directory.

#### 15.4.3 Permission-Checking Algorithm

The checks against owner, group, and other permissions are done in order, and checking stops as soon as the applicable rule is found.

#### 15.4.4 Checking File Accessibility: _access()_

The _effective_ user and group IDs, as well as supplementary group IDs, are used to determine the permissions a process has when accessing a file. It is also possible for a program (e.g., a set-user-ID or set-group-ID program) to check file accessibility base on the _real_ user and group IDs of the process.

The _access()_ system call checks the accessibility of the file specified in _pathname_ based on a process's real user and group IDs (and supplementory group IDs).

**IMPORTANT**, from the man pages: this system call allows set-user-ID programs and capability-endowed programs to easily determine the invoking user's autority. In other words, _aceess()_ does not answer the question "can I rea/write/execute this file?" question. It answers a slightly different question: "(assuming I'm a setuid binary) can the user who invoked me read/write/execute this file?", which gives set-user-ID programs the possibility to prevent malicious users from causing them to read files which users shouldn't be able to read.

#### 15.4.5 Set-User-ID, Set-Group-ID, and Sticky Bits

On older UNIX implementations, the sticky bit was provided as a way of making commonly user programs run faster. Modern UNIX implementations have more sophisticated memory-management systems, which have rendered this use of the sticky permissions bit obsolete.

In modern UNIX implementations, the sticky permissions bit serves differently. For directories, the sticky bit acts as the _restrited deletion_ flag. Setting the bit on a directory means that an unprivileged process can unlin and rename files in the directory only if it has write permission on the directory _and_ owns either the file or the directory. This makes it possible to create a own files in the directory but can't delete files owned by other users. The sticky permission bit is commonly set on the /tmp directory for this reason.

A file's sticky permission bit is set via the _chmod_ command or via the _chmod()_ system call.

#### 15.4.6 The Process File Mode Creation Mask: _umask()_

The umask is a process attribute that specifies which permission bits should always be turned _off_ when new files or directories are created by the process.

Often, a process just uses the umask it inherits from its parent shell, with the consequence that the user can control the umask of programs executed from the shell using the shell built-in command _umask_, which changes the umask of the shell process.

The initialization files for most shells set the default umask to the octal value 022 (----w--w-). This value specifies that write permission should always be turned off for group and other.

The _umask()_ system call changes a process's umask.

#### 15.4.7 Changing File Permissions: _chmod()_ and _fchmod()_

The _chmod()_ and _fchmod()_ system calls change the permission of a file.

### 15.5 I-node Flags (ext2 Extended File Attributes)

I-node flags controls the various behaviors of files and directories.


## 16: EXTENDED ATTRIBUTES

### 16.1 Overview

Extended attributes (EAs) allow arbitrary metadata, in the form of name-value pairs, to be associated with file i-nodes.

#### EA namespaces

EAs have names of the form _namespace.name_. The _namespace_ component serves to separate EAs into functionally distinct classes. The _name_ component uniquely identifies an EA within the given _namespace_.

Four values are supported for _namespace: user, trusted, system, and security_.

#### Creating and viewing EAs from the shell

We can use _setfattr(1)_ and _getfattr(1)_ commands to set and view the EAs on a file.

### 16.2 Extended Attribute Implementation Details

### 16.3 System Calls for Manipulating Extended Attributes


## 17: ACCESS CONTROL LISTS


## 18: DIRECTORIES AND LINKS

### 18.1 Directories and (Hard) Links

A _directory_ is stored in the file system in a similar way to a regular file. Two things distinguish a directory from a regular file:

* A directory is marked with a different file type in its i-node entry.
* A directory is a file with a special organization. Essentially, it is a table consisting of filenames and i-node numbers.

The i-node table is numbered starting at 1, 0 in the i-node field of a directory entry indicates that the entry is unused. I-node 1 is used to record bad blocks in the file system. The root directory (/) of a file system is always stored in i-node entry 2, so that the kernel knows where to start when resolving a pathname.

The i-node doesn't contain a filename, it is only the mapping within a directory list that defines the name of a file. This has a useful consequence: we can create multiple names--in the same or in different directories--each of which refers to the same i-node. These multiple names are know as _links_, or sometimes as _hard links_ to distinguish them from symbolic links.

Files refer to the same i-node entry, refer to the same file, the link count will be greater than 1. If one of these filenames is removed, the other name, and the file itself, continue to exist. The i-node entry and data blocks for the file are removed only when the i-node's link count falls to 0--that is, when all of the names for the file have been removed.

Hard links have two limitations, both of which can be circumvented by the use of symbolic links:

* I-node numbers are unique only within a file system, a hard link must reside on the same file system as the file to which it refers.
* A hard link can't be made to a directory, this prevents the creation of circular links.

### 18.3 Symbolic (Soft) Links

A _symbolic links_, also sometimes called a _soft link_, is a special file type whose data is the name of another file.

Symbolic links don't have the same status as hard links. In particular, a symbolic link is not included in the link count of the file to which it refers. Therefore, if the filename to which the symbolic link refers is removed, the symbolic link itself continues to exist, even though it can no longer be dereferenced, we say it has become a _dangling link_. It is even possible to create a symbolic link to a filename that doesn't exist at the time the link is created.

Since a symbolic link refers to a filename, rather than an i-node number, it can be used to link to a file in a different file system. Symbolic links also do not suffer the other limitation of hard links: we can create symbolic links to directories. It is possible to chain symbolic links.

#### Interpretation of symbolic links by system calls

Many system calls dereferences symbolic links and thus work on the file to which the link refers. Some system calls don't dereference symbolic links, but instead operate directly on the link itself.

System calls: _stat()_ and _lstat()_.

#### File permissions and ownership for symbolic links

### 18.3 Creating and Removing (Hard) Links: _link()_ and _unlink()_

The _link()_ and _unlink()_ system calls create and remove hard links.

On Linux, the _link()_ system call does not dereference symbolic links.

The _unlink()_ system call removes a link (deletes a filename) and, if that is the last link to the file, also removes the file itself. We can't use _unlink()_ to remove a directory, that task requires _rmdir()_ or _remove()_. The _unlink()_ system call does not dereference symbolic links.

#### An open file is deleted only when all file descriptors are closed

In addition to maintaining a link count for each i-node, the kernel also counts open file descriptions for the file. If the lask link to a file is removed and any processes hold open file descriptors referring to the file, the file won't actually be deleted until all of the descriptors are closed.

### 18.4 Changing the Name of a File: _rename()_

The _rename()_ system call can be used both to rename a file and to move it into another directory on the same file system.

The _rename()_ call just manipulates directory entries, it does not remove file data. Renaming a file does not affect other hard links to the file, nor does it affect any processes that hold open descriptors for the file, since these descriptors refer to open file descriptions, which have no connection with filenames.

### 18.5 Working with Symbolic Links: _symlink()_ and _readlink()_

The _symlink()_ system call creates a new symbolic link. The _readlink()_ system call retieve the content of the link itself--that is, the pathname to which it refers.

### 18.6 Creating and Removing Directories: _mkdir()_ and _rmdir()_

The _mkdir()_ system call creates a new directory. The _rmdir()_ system call removes a directory. In order for _rmdir()_ to succeed, the directory must be empty.

### 18.7 Removing a File or Directory: _remove()_

The _remove()_ library function removes a file or an empty directory. If _pathname_ argument is a file, _remove()_ calls _unlink()_; if _pathname_ argument is a directory, _remove()_ calls _rmdir()_.

### 18.8 Reading Directories: _opendir()_ and _readdir()_

The library functions described in this section can be used to open a directory and retrieve the names of the files it contains one by one.

``` c
#include <dirent.h>

DIR *opendir(const char *dirpath);
DIR *fdopendir(int fd);
strunct dirent *readdir(DIR *dirp);
int closedir(DIR *dirp);

struct dirent {
    ino_t d_ino;     /* File i-node number */
    char  d_name[];  /* Null-terminated name of file */
};
```

The _opendir()_ function opens a directory and returns a handle that can be used to refer to the directory in later calls. The structure of type DIR is a so-called _directory stream_.

The _fdopendir()_ function is like _opendir()_, except that the directory for which a stream is to be created is specified via the open file descriptor _fd_.

The _readdir()_ function reads successive entries from a directory stream. Each call to _readdir()_ reads the next directory from the directory stream referred to by _dirp_ and returns a pointer to a statically allocated structure of type _dirent_. This structure is overwritten on each call to _readdir()_. 

Further information about the file referred to by _d\_name_ can be obtained by calling _stat()_ on the pathname contructed using the _dirpath_ argument that was specified to _opendir()_ concatenated with the value returned in the _d\_name_ field.

The _closedir()_ function closes the open directory stream referred to by _dirp_, freeing the resources used by the stream.

#### Directory streams and file descriptors

A directory stream has an associated file descriptor. The _dirfd()_ function returns the file descriptor associated with the directory stream.

#### The _readdir\_r()_ funtion

The _readdir\_r()_ function is a variation on _readdir()_. The key semantic difference between _readdir\_r()_ and _readdir()_ is that the former is reentrant, while the latter is not.

### 18.9 File Tree Walking: _nftw()_

The _nftw()_ function allows a program to recursively walk through an entire directory subtree performing some operation for each file in the subtree.

### 18.10 The Current Working Directory of a Process

A process's _current working directory_ defines the starting point for the resolution of relative pathnames referred to by the process. A new process inherits its current working directory from its parent.

#### Retrieving the current working directory

A process can retrieve its current working directory using _getcwd()_.

#### Changing the current working directory

The _chdir()_ system call changes the calling process's current working directory to the relative or absolute pathname.

### 18.11 Operating Relative to a Directory File Descriptor

### 18.12 Changing the Root Directory of a Process: _chroot()_

Every process has a _root directory_, which is the point from which absolute pathnames are interpreted. By default, this is the real root directory of the file system.

The _chroot()_ system call changes the process's root directory. Thereafter, all absolute pathnames are interpreted as starting from that location in the file system.

### 18.13 Resolving a Pathname: _realpath()_

The _realpath()_ library function dereferences all symbolic links and resolves all reference to `/.` and `/..` to produce a null-terminated string containing the corresponding absolute pathname.

### 18.14 Parsing Pathname Strings: _dirname()_ and _basename()_

The _dirname()_ and _basename()_ functions break a pathname string into directory and filename parts.


## 19: MONITORING FILE EVENTS


## 20: SIGNALS: FUNDAMENTAL CONCEPTS

### 20.1 Concepts and Overview

A _signal_ is a notification to a process that an event has occurred. Signals are sometimes described as _software interrupts_.

One process can send a signal to another process or itself. However, the usual source of many signals sent to a process is the kernel: 

* A hardware exception
* The user typed one of the terminal special charactors that generate signals
* A software event occurred

_Standard_ signals are used by the kernel to notify processes of events, on Linux, the standard signals are numbered from 1 to 31. The other type is _realtime_ signals.

A signal is said to be _generated_ by some event. Once generated, a signal is later _delivered_ to a process, which then takes some action in response to the signal. Between the time it is generated and the time it is delivered, a signal is said to be _pending_. If you don't want to be interrupted by the delivery of a signal, add a signal to the process's _signal mask_--a set of signals whose delivery is currently _blocked_. If a signal is generated while it is blocked, it remains pending until it is later unblocked (removed from the signal mask).

Default actions:

* The signal is _ignored_
* The process is _terminated_ (killed)
* A _core dump file_ is generated, and the process is terminated. A core dump file contains an image of the virtual memory of the process, which can be loaded into a debugger in order to inspect the state of the process at the time that it terminated
* The process is _stopped_--execution of the process is suspended
* Execution of the process is _resumed_ after previous being stopped

Instead of accepting the default for a particular signal, a program can change the action that occur when the signal is delivered, setting the _disposition_ of the signal:

* The _default action_ should occur
* The signal is _ignored_
* A _signal handler_ is executed

### 20.2 Signal Types and Default Actions

### 20.3 Changing Signal Dispositions: _signal()_

UNIX systems provide two ways of changing the disposition of a signal: _signal()_ and _sigaction()_. _Signal()_ may cause portable issues, _sigaction()_ is the preferred API for a signal handler.

``` c
#include <signal.h>

void ( *signal(int sig, void (*handler)(int)) ) (int);
```

A more readable form:

``` c
typedef void (*sighandler_t)(int);

sighandler_t signal(int sig, sighandler_t handler);
```

### 20.4 Introduction to Signal Handlers

A _signal handler_ is a function that is called when specified signal is delivered to a process.

### 20.5 Sending Signals: _kill()_

One process can send a signal to another process using the _kill()_ system call, which is the analog of the _kill_ shell command.

``` c
#include <signal.h>

int kill(pid_t pid, int sig);
```

### 20.6 Checking for the Existence of a Process

Specifying the _sig_ argument as 0 can be used to check for the existence of a process.

Various ways can be used to check whether a particular process is running.

### 20.7 Other Ways of Sending Signals: _raise()_ and _killpg()_

The _raise()_ function can send a signal to process itself.

The _killpg()_ function sends a signal to all of the members of a process group.

### 20.8 Displaying Signal Descriptions

Each signal has an associated printable descriptions. The _strsignal()_ function can be used to get the description of signals.

### 20.9 Signal Sets

Multiple signals are represented using a data structure called a _signal set_.

### 20.10 The Signal Mask (Blocking Signal Delivery)

For each process, the kernel maintains a _signal mask_--a set of signals whose delivery to the process is currently blocked. If a signal that is blocked is sent to a process, delivery of that signal is delayed until it is unblocked by being removed from the process signal mask.

The _sigprocmask()_ system call can be used at any time to explicitly add signals to, and remove signals from, the signal mask.

Attempts to block `SIGKILL` and `SIGSTOP` are silently ignored.

### 20.11 Pending Signals

If a process receives a signal that is currently blocking, that signal is added to the process's set of pending signals. When and if the signal is later unblocked, it is then delivered to the process. To determine which signals are pending for a process, we can call _sigpending()_.

### 20.12 Signals Are Not Queued

The set of pending signals is only a mask; it indicates whether or not a signal has occurred, but not how many times it has occurred. In other word, if the same signal is generated multiple times while it is blocked, then it is recorded in the set of pending signals, and later delivered, just once.

### 20.13 Changing Signal Dispositions: _sigaction()_

The _sigaction()_ system call is an alternative to _signal()_ for setting the disposition of a signal, it is more complex and flexibale than _signal()_. In particular, _sigaction()_ allows us to retrieve the disposition of a signal without changing it, and to set various attributes controlling precisely what happens when a signal handler is invoked.

``` c
#include <signal.h>

int sigaction(int sig, const struct sigaction *act, struct sigaction *oldact);

struct sigaction {
    void   (*sa_handler)(int);    /* Address of handler */
    sigset_t sa_mask;             /* Signals blocked during handler
                                     invocation */
    int      sa_flags;            /* Flags controlling handler invocation */
    void   (*sa_restorer)(void);  /* Not for application use */
};
```

### 20.14 Waiting for a Signal: _pause()_

Calling _pause()_ suspends execution of the process until the call is interrupted by a signal handler (or until an unhandled signal terminates the process).


## 21: SIGNALS: SIGNAL HANDLERS

### 21.1 Designing Signal Handlers

In general, it is preferable to write simple signal handlers. One important reason for this is to reduce the risk of creating race conditions.

#### 21.1.1 Signals Are Not Queued (Revisited)

#### 21.1.2 Reentrant and Async-Signal-Safe Functions

##### Reentrant and nonreentrant functions

A function is said to be _reentrant_ if it can safely be simultaneously executed by multiple threads of execution in the same process. In this context, "safe" means that the function achieves its expected result, regardless of the state of execution of any other thread of execution.

A function may be _nonreentrant_ if it updates global or static data structures.(A function that employs only local variables is guaranteed to be reentrant.) 

##### Standard async-signal-safe functions

An _async-signal-safe_ function is one that the implementation guarantees to be safe when called from a signal handler. A function is async-signal-safe either because it is reentrant or because it is not interruptible by a signal handler.

A function is unsafe only when invocation of a signal handler interrupts the execution of an unsafe function, and the handler itself also calls an unsafe function. In other words, when writing signal handlers, we have two choices:

* Ensure that the code of the signal handler itself is reentrant and that it calls only async-signal-safe functions.
* Block delivery of signals while executing code in the main program that calls unsafe functions or works with global data structure also updated by the signal handler.

We must not call unsafe functions from within a signal handler.

##### Use of _errno_ inside signal handlers

#### 21.1.3 Global Variables and the _sig\_atomic\_t_ Data Type

Reading and writing global variables may involve more than one machine language instruction, and a signal hanlder may interrupt the main program in the middle of such an instruction sequence. The integer data type, _sig\_atomic\_t_, guarantes reads and writes are atomic.

### 21.2 Other methods of Terminating a Signal Handler

#### Performing a Nonlocal Goto from a signal handler

#### Terminating a Process Abnormally: _abort()_

The _abort()_ function terminates the calling process and causes it to produce a core dump. It terminates the calling process by raising a `SIGABRT` signal.

#### 21.3 Handling a Signal on an Alternate Stack: _sigaltstack()_

An alternate signal stack is useful in cases when the standard stack has been exhausted by growing too large.

#### 21.4 The `SA_SIGINFO` Flag

Setting the `SA\_SIGINFO` flag when establishing a handler with _sigaction()_ allows the handler to obtain addtional information about a signal when it is delivered.

#### 21.5 Interruption and Restarting of System Calls

When a signal handler interrupts a blocked system call, the system call fails with the error `EINTR`.

Specifying the `SA\_RESTART` flag when establishing the signal handler with _sigaction()_, system calls are automatically restarted by the kernel on the process's behalf.


## 22: SIGNALS: ADVANCED FEATURES


## 23: TIMERS AND SLEEPING


## 24: PROCESS CREATION

### 24.1 Overview of _fork()_, _exit()_, _wait()_, and _execve()_

The `SIGCHLD` signal is generated by the kernel for a parent process when one of its children terminates.

### 24.2 Creating a New Process: _fork()_

The key point to understanding _fork()_ is to realize that after it has completed its work, two processes exist, and, in each process, execution continues from the point where _fork()_ returns.

It is indeterminate which of the two processes is next scheduled to use the CPU.

#### 24.2.1 File Sharing Between Parent and Child

When a _fork()_ is performed, the child receives duplicates of all of the parent's file descriptors, they refer to the samme open file description. The parent and child share these attributes in the open file descripiton.

#### 24.2.2 Memory Semantics of _fork()_

Avoid wasteful copying:

* The parent and child can share the same text segment(read only)

* _copy-on-write_

### 24.3 The _vfork()_ System Call

Like _fork()_, _vfork()_ is used by the calling process to create a new child process. However, _vfork()_ is expressly designed to be used in programs where the child performs an immediate _exec()_ call.

* No duplication of virtual memory pages or page tables is done for the child process. Instead, the child shares the parent's memory until it either performs a successful _exec()_ or calls _\_exit()_ to terminate

* Execution of the parent process is suspended until the child has performed an _exec()_ or _\_exit()_

The semantics of _vfork()_ mean that after the call, the child is guaranteed to be scheduled for the CPU before the parent.

Except where speed is absolutely critical, new programs should avoid the use of _vfork()_ in favor of _fork()_. Where it is used, _vfork()_ should generally be immediately followed by a call to _exec()_. If the _exec()_ call fails, the child process should terminate using _\_exit()_.

### 24.4 Race Conditions After _fork()_

We can't assume a particular order of execution for the parent and child after a _fork()_. If we need to guarantee a particular order, we must use some kind of synchronization technique.

### 24.5 Avoiding Race Conditions by Synchronizing With Signals

After a _fork()_, if either process needs to wait for the other to complete an action, then the active process can send a signal after completing the action, the other process wait for the signal.


## 25: PROCESS TERMINATION


### 25.1 Terminating a Process: _\_exit()_ and _exit()_

The following actions are performed by _exit()_:

* Exit handlers are called, in reverse order of their registration
* The _stdio_ stream buffers are flushed
* The _\_exit()_ system call is invoked

Performing an explicit _return n_ is generally equivalent to calling _exit(n)_.

Performing a return without specifying a value, or falling off the end of the _main()_ function, also results in the caller of _main()_ invoking _exit()_, but with results that vary depending on the version of the C standard supported and the compilation options employed.

### 25.2 Details of Process Termination

### 25.3 Exit Handlers

An exit handler is a programmer-supplied function that is registered at some point during the life of the process and is then automatically called during _normal_ process termination via _exit()_. Exit hanlders are not called if a program calls _\_exit()_ directly or if the process is terminated abnormally by a signal.

A child process created via _fork()_ inherits a copy of its parent's exit handler registrations, when a process performs an _exec()_, all exit handler registrations are removed.

### 25.4 Interactions Between _fork()_, _stdio_ Buffers, and _\_exit()_

A very good example in the book.
