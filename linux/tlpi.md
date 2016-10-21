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

```c
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *pathname, int flags, .../* mode_t mode */);
```

* _flag_ argument: specifies the _access mode_ for the file
* _mode_ argument: specifies the permissions to be placed on the file. If the _open()_ call does not specify `O_CREAT`, _mode_ can be ommited

An example:

```c
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

```
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

```c
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

```c
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

```c
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

```c
#include <unistd.h>

int brk(void *end_data_segment);

void *sbrk(intptr_t increment);
```

The _brk()_ system call sets the program break to the location specified by _end\_data\_segment_. Since virtual memory is allocated in units of pages, _end\_data\_segment_ is effectively rounded up the the next page boundary. Attemps to set the program break below its initial value are likely to result in unexpected behavior, such as segment fault when trying to access data in now nonexistent parts of the initialized or uninitialized data segments.

A call to _sbrk()_ adjusts the program break by adding _increment_ to it. On success, it returns the previous address of the program break.

#### 7.1.2 Allocating Memory on the Heap: _malloc()_ and _free()_

```c
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

```c
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

```c
#include <pwd.h>

struct passwd *getpwnam(const char *name);
struct passwd *getpwuid(uid_t uid);

struct password {
    char *pw_name;        /* Login name (username) */
    char *pw_password;    /* Encrypted password */
    uid_t pw_uid;         /* User ID */
    gid_t pw_gid;         /* Group ID */
    char *pw_gecos;       /* Comment (user information) */
    char *pw_dir;         /* Initial working (home) directory */
    char *pw_shell;       /* Login shell */
}
```

#### Retrieving records from the group file

```c
#include <grp.h>

struct passwd *getgrnam(const char *name);
struct passwd *getgrgid(gid_t gid);

struct password {
    char  *gr_name;        /* Group name */
    char  *gr_passwd;      /* Encrypted password (if not password shadowing) */
    gid_t  gr_gid;         /* Group ID */
    char **gr_mem;         /* NULL-terminated array of pointers to names
                              of members listed in /etc/group */
}
```

#### Scanning all records in the password and group files

#### Retrieving records from the shadow password file

### 8.5 Password Encryption and User Authentication

For security reasons, UNIX systems encrypt passwords using a _one-way encryption_ algorithm, which means that there is no method of re-creating the original password from its encrypted form. Therefore the only way of validating a candidate password is to encrypt it using the name method and see if the encrypted result matches the value stored in `/etc/shadow`.


