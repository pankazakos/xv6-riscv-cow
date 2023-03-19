# Xv6 Copy on Write Implementation

## Description
Current implementation of fork system call of xv6 copies the user-space of a parent process to its child. Not only this can be pointless when a fork is followed by an exec, but also if the child and parent processes have to write on the same page they would do it on different copies of the page, thus wasting memory. To solve this issue, fork should copy the pages to child only when it is necessary. This is called Copy on Write (Cow).

## Copy on Write
Fork system call creates a table of entries (Page table entries or PTEs) for all child processes that reference the pages of parent. They are defined as read-only in both parent and child processes. When a process tries to change one of the referenced read-only pages, a callback will create a copy of this page and assign its corresponding PTE to read-write.

## Brief Docs
### kernel/ricv.h
- **PTE_COW**: this flag indicates that a page is shareable between processes and should be read-only as well. 8th bit is used for this flag 
### kernel/kalloc.c
- **kmem.refc**: this is a reference counter, meaning that it counts how many processes are using this page. Once the counter gets 0 the page can be deleted by kfree().
- **increment_refc()**: increments the reference counter.
- **kfree()**: the reference counter is decremented if it is above 0.
### kernel/vm.c
- **uvmcopy()**: the part that allocates new page is deleted and instead, the reference counter is incremented. The page is marked as COW with PTE_COW flag and the PTE references the parent page.
- **pfault_handle()**: this handler checks for available virtual address and whether a page is valid and user. If the page is also marked as COW, then it copies the shared page that needs to be written from a process.
- **copyout()**: the call for the cow handler (pfault_handle()) is also needed here.
### kernel/trap.c
- **usertrap()**: I also call the cow_handler whenever there is a page fault code of 15, 13 and 12. Code 15 means that a process tried to write on a read-only page, 13 indicates an error while reading the page and 12 indicates an error while executing a command regarding the page.


## Build and run project
### Install dependencies
```
sudo apt install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

### Compile
```
make
```

### Run xv6 with qemu
```
make qemu
```
To exit press `ctrl + a + x`

## Testing
Tests can be found under `user` directory. To run the tests you should run them as a command from xv6 prompt.
```
$ cowtest
simple: ok
simple: ok
three: ok
three: ok
three: ok
file: ok
ALL COW TESTS PASSED

$ usertests
usertests starting
test MAXVAplus: OK
test manywrites: OK
test execout: OK
test copyin: OK
test copyout: OK
test copyinstr1: OK
test copyinstr2: OK
test copyinstr3: OK
test rwsbrk: OK
test truncate1: OK
test truncate2: OK
test truncate3: OK
test reparent2: OK
test pgbug: OK
test sbrkbugs: OK
test badarg: OK
test reparent: OK
test twochildren: OK
test forkfork: OK
test forkforkfork: OK
test argptest: OK
test createdelete: OK
test linkunlink: OK
test linktest: OK
test unlinkread: OK
test concreate: OK
test subdir: OK
test fourfiles: OK
test sharedfd: OK
test dirtest: OK
test exectest: OK
test bigargtest: OK
test bigwrite: OK
test bsstest: OK
test sbrkbasic: OK
test sbrkmuch: OK
test kernmem: OK
test sbrkfail: OK
test sbrkarg: OK
test sbrklast: OK
test sbrk8000: OK
test validatetest: OK
test stacktest: OK
test opentest: OK
test writetest: OK
test writebig: OK
test createtest: OK
test openiput: OK
test exitiput: OK
test iput: OK
test mem: OK
test pipe1: OK
test killstatus: OK
test preempt: kill... wait... OK
test exitwait: OK
test rmdot: OK
test fourteen: OK
test bigfile: OK
test dirfile: OK
test iref: OK
test forktest: OK
test bigdir: OK
ALL TESTS PASSED
```
