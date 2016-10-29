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

