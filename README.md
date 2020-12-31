# xv6-tracking
Repository to record the progress for self-learning MIT 6.S081. Out of the respect to MIT for making all the great course resource public, the code implementation will only be kept in a separate private repository.
- [Util Lab](#util-lab)
    - [Lab Result](#lab-result)
- [Syscall Lab](#syscall-lab)
    - [Lab Result](#lab-result-2)
- [Page Table Lab](#page-table-lab)
    - [Lab Result](#lab-result-3)

## Util Lab
This lab requires to write some basic user space functions utilizing some library functions and system call functions.

Most of the problems in this lab are not very hard to implement. The most interesting one is the `primes.c` program. The tricky part is every process will have two pipes to communicate with (one to communicate with its parent, one to communicate with its child), so how to use the `fork` and `pipe` to create the process? The way I approach this problem is:
- For a process, it will have two sets of file descriptors (corresponding to two pipes). It also keeps a `flag` to indicate which pipes is the one it should use to communicate with its parent.
- Main process will initialize a pipe and set the `flag` to correspond pipe. Then after the `fork`, the child process will get the same memory image as the parent. Since the `flag` will be the same as its parent, it knows which set of file descriptors to use to read the input, and the other one will be the one it uses to communicate with its child. Then it can `fork` to create the child, write values to the child pipe, and `wait` for child to return.
- For a child process:
    - If `read` returns 0, it can `exit`.
    - Otherwise, it will print the value and be responsible for filter out all the multiples of this value. It will initialize a pipe with another file descriptors set to write output to its child. Note here the `flag` will need to be change to correspond pipe.
    - `fork` to create the child, write filtered value to its child pipe, and then `wait` for child.

### Lab Result
```sh
== Test sleep, no arguments ==
$ make qemu-gdb
sleep, no arguments: OK (5.6s)
== Test sleep, returns ==
$ make qemu-gdb
sleep, returns: OK (1.8s)
== Test sleep, makes syscall ==
$ make qemu-gdb
sleep, makes syscall: OK (1.6s)
== Test pingpong ==
$ make qemu-gdb
pingpong: OK (1.0s)
== Test primes ==
$ make qemu-gdb
primes: OK (2.3s)
== Test find, in current directory ==
$ make qemu-gdb
find, in current directory: OK (1.4s)
== Test find, recursive ==
$ make qemu-gdb
find, recursive: OK (1.1s)
== Test xargs ==
$ make qemu-gdb
xargs: OK (2.2s)
== Test time ==
time: OK
Score: 100/100
```

## Syscall Lab

### Lab Result
```sh
== Test trace 32 grep ==
$ make qemu-gdb
trace 32 grep: OK (4.8s)
== Test trace all grep ==
$ make qemu-gdb
trace all grep: OK (1.7s)
== Test trace nothing ==
$ make qemu-gdb
trace nothing: OK (1.0s)
== Test trace children ==
$ make qemu-gdb
trace children: OK (11.5s)
== Test sysinfotest ==
$ make qemu-gdb
sysinfotest: OK (2.1s)
== Test time ==
time: OK
Score: 35/35
```

## Page Table Lab
This is the first lab involving much kernel programming. To get the kernel page mapping correct is tricky. The lab Q&A lecture is pretty helpful, which eventually saves me after I spent a few days of debugging.

**A few things to note:**
- kernel page table and user page table are completely separate. `walk` is the bridge to get kernel space and user space work together. Before adding virtual address of user space to kernel page table, kernel mode and user mode are essentially using physical address to communicate.
- I eventually followed the approach from the lab Q&A lecture to finish this lab. While to create a new kernel page for each process in `allocproc` is not impossible, the approach may cause some performance issue as you will need to free all the 3-level page tables when the process is done. I also tried to reuse everything in global kernel page table except allocating each kernel stack dynamically, but I was not able to get this working.
- In the third problem, I was also think about to add user virtual address mappings only to the level 2 kernel page table such that level 1 and level 0 user page table can be reused. However, the problem here is kernel cannot access PTE with `PTE_U` flag, so I eventually gave up this approach.

### Lab Result
```sh
== Test pte printout ==
$ make qemu-gdb
pte printout: OK (5.2s)
== Test answers-pgtbl.txt == answers-pgtbl.txt: OK
== Test count copyin ==
$ make qemu-gdb
count copyin: OK (1.3s)
== Test usertests ==
$ make qemu-gdb
(230.3s)
== Test   usertests: copyin ==
  usertests: copyin: OK
== Test   usertests: copyinstr1 ==
  usertests: copyinstr1: OK
== Test   usertests: copyinstr2 ==
  usertests: copyinstr2: OK
== Test   usertests: copyinstr3 ==
  usertests: copyinstr3: OK
== Test   usertests: sbrkmuch ==
  usertests: sbrkmuch: OK
== Test   usertests: all tests ==
  usertests: all tests: OK
== Test time ==
time: OK
Score: 66/66
```

