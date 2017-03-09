# Overview of Glibc Malloc
The glibc malloc is derived from ptmalloc (pthreads malloc), which is derived from dlmalloc (Doug Lea malloc)

## Features
- **"heap" style**: chunks of various sizes exist within a larger region of memory;
- **Chunk**: Chunk-oriented, devides a large region of memory(heap) into chunks of various sizes:
 - Each includes metadata about its size and the position of adjacent chunks;
 - A "chunk pointer" or *mchunkptr* does *not* point to the beginning of the chunk, but the last word in the previous chunk;
 - Chunks are adjacent to each other, all chunks **in the heap** can be iterated through by using the size information from the address of the first chunk in a heap;
 - In-used chunks are not tracked by the arena, free chunks are stored in **various lists based on size and history** called **"bins"**;
- **Arena**: A structure shared among one or more threads which contains references to one or more heaps, and the linked lists of chunks within those heaps which are "free":
 - Main arena" uses the application's heap, which owns a static variable in the malloc pointing to it, and other arenas use mmap'd heaps. Each of them has a *next* pointer to link additional arenas;
 - More than one region of memory are active at a time, to reduce contention;
 - Arena is created with *mmap*, the number of arenas is capped at eight times the number of CPUs in the system(modified by *mallopt*). **Heavily threaded application will still see some contention, but the trade-off is thar there will be less fragmentation**;
 - Mutex is used to control access to that arena. Some operations, such as **access to the fastbins** can be down with **atomic operations**.
- **Bins**:
 - **Fast**: Designed to make the operations fast. **Small chunks** are stored in **size-specific bins**. Chunks in fastbins are *not* coalesced, and stored in a **singly linked list**.
 - **Unsorted**: Chunks are initially stored in a single bin when they are just free'd. They are later sorted together.
 - **Small** & **Large**: Chunks in small-bins are of the same size, and in a range of size in large-bins. Coalesce with adjascent after chunks added in, and are doubly-linked. Pick the first chunk in small and use it, and find the "best" in large chunk.
 - The first chunk of each size contains the ptr and size information. To achieve better performance, when multiple chunk of a dozen size are present, the *second* is chosen or added, not to adjust the next-size linked list.

## Algorithm
- **Malloc**:
![glibc malloc algorithm](https://github.com/XingGaoY/Multi-core/raw/scripts/glibc-malloc.png)
- **Free**： 
Freeing memroy does not actually return it to the operating system, instead, marks a chunk of memory as "free to be reused".
![glibc malloc algorithm](https://github.com/XingGaoY/Multi-core/raw/scripts/glibc-free.png)
- **Realloc**
![glibc malloc algorithm](https://github.com/XingGaoY/Multi-core/raw/scripts/glibc-realloc.png)
