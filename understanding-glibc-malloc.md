# Understanding glibc malloc
**per thread arena**: Maintaining separate heap and freelist data structures for each thread.

The author present a test program to show that:
- **malloc**: a heap segment(called main arena or thread arena) is created by increasing program brk location with syscall *brk* in main thread, or *mmap* in thread arena. And a memory size of 132 KB(larger than need) is allocated. It will grow when run out of free space by increasing brk location or mmap, and shrink reversely;
- **free**: the free block is release to *free bin*;

Some terms:
- **Arena**: The number of arenas is irrespective of number of threads
```
For 32 bit systems:
	Number of arena = 2 * number of cores.
For 64 bit systems:
	Number of arena = 8 * number of cores.
```
