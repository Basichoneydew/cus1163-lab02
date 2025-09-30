# Lab 2: Operating System Interfaces and Process Information

#### Learning Objectives

- Understand how the operating system exposes kernel information through filesystem interfaces like /proc
- Implement directory traversal and file reading using system calls (opendir, readdir, open, read, close)
- Use strace to analyze system call patterns and understand the user/kernel boundary
- Parse and extract process information from kernel-provided data structures
- Compare library functions versus direct system calls for file operations

#### Prerequisites

- Basic C programming knowledge
- Familiarity with file operations from Lab 1
- Comfort with the Linux terminal and command line
- Understanding of processes and system calls

#### Introduction

The operating system kernel maintains detailed information about all running processes, system resources, and hardware.
Rather than keeping this information hidden, Linux exposes much of it through a special filesystem called /proc. This
lab explores how user programs can read kernel-provided information using the same file operations you learned in Lab 1.

You'll build a simple process information tool that reads from /proc, then use strace to see exactly what system calls
your program makes when accessing this kernel data.

#### Lab File Structure

**main.c (PROVIDED - Complete)**

This file contains the main entry point for your lab application. It presents a simple menu-driven interface for the
user, allowing them to select between:

- Listing processes
- Reading process information
- Showing system information
- Comparing file operation methods

The file handles user input, calls functions implemented in proc_reader.c, and prints results and status messages. This
file is fully implemented and requires no modifications.

**proc_reader.h (PROVIDED)**

This header file contains function prototypes for the process reading operations implemented in proc_reader.c. It
includes:

- All necessary system headers
- Function prototypes for directory traversal
- File reading function declarations
- Process information extraction interfaces

No changes are necessary for this file.

**proc_reader.c (TEMPLATE - TO COMPLETE)**

This file contains the core logic for your process information operations. You will implement:

- System calls to read directories
- File opening and reading operations
- Process data extraction
- System information display functions

The main functions are provided with TODO comments indicating where you must fill in the missing code. Follow the
instructions and hints to complete the required functionality using system calls.

#### Project Setup

- Create the lab directory and files
- Complete the TODO sections in proc_reader.c
- Compile the program using the command below

```
gcc -o lab2 main.c proc_reader.c
```

#### How to Run the Program

**Basic execution**

```
./lab2
```

**With system call tracing**

```
strace -e trace=openat,read,write,close ./lab2
```

**With detailed tracing**

```
strace -o trace.txt ./lab2
```

#### Expected Output

```
Operating System Interfaces Lab
This program explores the /proc filesystem interface

=== Process Information Lab ===
1. List all process directories
2. Read process information
3. Show system information
4. Compare file operation methods
5. Exit
Choose an option (1-5):
```

**Option 1: List Process Directories**

```
Listing all process directories in /proc...
Process directories in /proc:
PID      Type                
---      ----                
1        process
2        process
3        process
4        process
5        process
6        process
7        process
11       process
12       process
13       process
Found 156 process directories
SUCCESS: Process directories listed!
```

**Option 2: Read Process Information**

```
Enter process ID (PID) to examine: 1

Reading information for PID 1...

--- Process Information for PID 1 ---
Name:	systemd
Umask:	0000
State:	S (sleeping)
Tgid:	1
Ngid:	0
Pid:	1
PPid:	0
TracerPid:		0
Uid:	0	0	0	0
Gid:	0	0	0	0

--- Command Line ---
/sbin/init
SUCCESS: Process information read!
```

**Option 3: System Information**

```
Reading system information...

--- CPU Information (first 10 lines) ---
processor	: 0
BogoMIPS	: 48.00
Features	: fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics fphp asimdhp cpuid asimdrdm jscvt fcma lrcpc dcpop sha3 asimddp sha512 asimdfhm dit uscat ilrcpc flagm sb paca pacg dcpodp flagm2 frint

--- Memory Information (first 10 lines) ---
MemTotal:       2002716 kB
MemFree:         94384 kB
MemAvailable:   605152 kB
Buffers:          6928 kB
Cached:         788780 kB
SwapCached:     256572 kB
Active:         543752 kB
Inactive:       818144 kB
Active(anon):   316404 kB
Inactive(anon): 492524 kB
SUCCESS: System information displayed!
```

**Option 4: Compare File Methods**

```
Comparing file operation methods...
Comparing file reading methods for: /proc/version

=== Method 1: Using System Calls ===
Linux version 6.8.0-40-generic (buildd@bos03-arm64-020) (aarch64-linux-gnu-gcc-13 (Ubuntu 13.2.0-23ubuntu4) 13.2.0, GNU ld (GNU Binutils for Ubuntu) 2.42) #40-Ubuntu SMP PREEMPT_DYNAMIC Fri Jul  5 11:20:03 UTC 2024

=== Method 2: Using Library Functions ===
Linux version 6.8.0-40-generic (buildd@bos03-arm64-020) (aarch64-linux-gnu-gcc-13 (Ubuntu 13.2.0-23ubuntu4) 13.2.0, GNU ld (GNU Binutils for Ubuntu) 2.42) #40-Ubuntu SMP PREEMPT_DYNAMIC Fri Jul  5 11:20:03 UTC 2024

NOTE: Run this program with strace to see the difference!
Example: strace -e trace=openat,read,write,close ./lab2
```

#### Implementation Hints

**Directory Operations**

```c
DIR *dir = opendir("/proc");
struct dirent *entry;
while ((entry = readdir(dir)) != NULL) {
    // Process each entry
}
closedir(dir);
```

**File Operations with System Calls**

```c
int fd = open(filename, O_RDONLY);
char buffer[1024];
ssize_t bytes_read;
while ((bytes_read = read(fd, buffer, sizeof(buffer) - 1)) > 0) {
    buffer[bytes_read] = '\0';
    printf("%s", buffer);
}
close(fd);
```

**File Operations with Library Functions**

```c
FILE *file = fopen(filename, "r");
char line[256];
while (fgets(line, sizeof(line), file) != NULL) {
    printf("%s", line);
}
fclose(file);
```

#### Analysis Activities

After completing the implementation:

- Run with strace and observe the system calls
- Compare the system calls made by both file reading methods
- Answer these questions:
- How many system calls does each method make?
- What's the difference between fopen() and open()?
- Why does the library method make different system calls?

### Analysis Activities Answers
#1 How many system calls does each method make?

Method 1 makes 3 system calls. These being openat(), read(), and close(). Method 2 makes 4 system calls, which are openat(), read(), close(), and newfstatat().

#2 What's the difference between fopen() and open()?

The difference between these two commands is that while open() is a direct system call that gives you access to a low-level file descriptor, it does not provide buffering, so you would have to manually use read/write operations. On the other hand, fopen() provides buffered I/O and returns a FILE *. This buffering improves performance by reducing the number of kernel transitions, as it groups many small reads or writes into fewer system calls. fopen() also gives you access to functions like fgets() and fprintf(), which makes file handling a lot easier.

#3 Why does the library method make different system calls?

The library method makes different system calls because, with method 2, the addition of the newfstatat() system call, we have access to file metadata such as permissions, type, and size information. This means that this method calls functions like fopen, fgets, and fclose, which are higher-level wrappers built on top of system calls and add features such as buffering and error checking. This buffering changes the syscall pattern by reducing the number of read() calls required for large files, which in turn improves performance.

**Commands for Analysis**

```
strace -e trace=openat,read,close ./lab2 2> syscalls.txt
```

#### Troubleshooting

- Permission denied errors: Some /proc files require special permissions
- File not found: Process directories appear and disappear as processes start/stop
- Compilation errors: Ensure all headers are included correctly
- Empty output: Check that you're null-terminating strings after read()

#### Submission

After completing your work:

```
git add .
git commit -m "completed lab 2 - process information reader"
git push origin main
```

Include:

- Complete implementation of all TODO sections
- Screenshots showing program output
- Text file with strace output showing system call differences
- Brief analysis of the system call patterns observed