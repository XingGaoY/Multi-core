# Overview of Malloc
The glibc malloc is derived from ptmalloc (pthreads malloc), which is derived from dlmalloc (Doug Lea malloc)

## Features
- **"heap" style**: chunks of various sizes exist within a larger region of memory;
- **Chunk**: Chunk-oriented, devides a large region of memory(heap) into chunks of various sizes:
 - Each includes metadata about its size and the position of adjacent chunks;
 - A "chunk pointer" or *mchunkptr* does *not* point to the beginning of the chunk, but the last word in the previous chunk;
 - Chunks are adjacent to each other, all chunks **in the heap** can be iterated through by using the size information from the address of the first chunk in a heap;
- **Arena**: A structure shared among one or more threads which contains references to one or more heaps, and the linked lists of chunks within those heaps which are "free":
 - **Main arena" uses the application's heap, which owns a static variable in the malloc pointing to it, and other arenas use mmap'd heaps. Each of them has a *next* pointer to link additional arenas;
 - More than one region of memory are active at a time, to reduce contention;
 - Arena is created with *mmap*, the number of arenas is capped at eight times the number of CPUs in the system(modified by *mallopt*). **Heavily threaded application will still see some contention, but the trade-off is thar there will be less fragmentation**;