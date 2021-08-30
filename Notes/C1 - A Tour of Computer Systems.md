# CHAPTER 1 - A Tour of Computer Systems

## 1.1 Information Is Bits + Context

All information in a system—including disk files, programs stored in memory, user data stored in memory, and data transferred across a network—is represented as a bunch of bits. The only thing that distinguishes different data objects is the context in which we view them. For example, in different contexts, the same sequence of bytes might represent an integer, floating-point number, character string, or machine instruction.

```c
// view the same data 0x41424300 in different types

const char bytes[] = {0x41, 0x42, 0x43, 0};
printf("cstring  %s\n", bytes);
printf("float    %f\n", *(float *)bytes);
printf("int32_t  %d\n", *(int32_t *)bytes);

/* output
cstring  ABC
float    0.000000
int32_t  4407873
*/
```

## 1.4 Processors Read and Interpret Instructions Stored in Memory

### 1.4.1 Hardware Organization of a System

#### Buses

Buses are typically designed to transfer fixed-size chunks of bytes known as words.The number of bytes in a word (the word size) is a fundamental system parameter that varies across systems. Most machines today have word sizes of either 4 bytes (32 bits) or 8 bytes (64 bits).

```c
// smaller is not necessarily better

const int64_t kMaxCount = 0x87654321;
printf("WORD_SIZE  %d\n", sizeof(void*));

int beg = clock();
int16_t a16 = 0x1234, b16 = 0x5678;
for (register int64_t i = 0; i < kMaxCount; ++i) a16 += b16, b16 += a16;
printf("int16_t    %d\n", clock() - beg);

beg = clock();
int32_t a32 = 0x1234, b32 = 0x5678;
for (register int64_t i = 0; i < kMaxCount; ++i) a32 += b32, b32 += a32;
printf("int32_t    %d\n", clock() - beg);

beg = clock();
int64_t a64 = 0x1234, b64 = 0x5678;
for (register int64_t i = 0; i < kMaxCount; ++i) a64 += b64, b64 += a64;
printf("int64_t    %d\n", clock() - beg);

/* output
WORD_SIZE  8
int16_t    10264
int32_t    9880
int64_t    9733
*/
```

#### Main Memory

The main memory is a temporary storage device that holds both a program and the data it manipulates while the processor is executing the program. 

```c
// executable is copied to main memory while running

void SayHello() { puts("Hello"); }
void SayBye() { puts("Bye"); }

void ViewMemory(const uint8_t *beg, const uint8_t *end) {
  if (((uint64_t)beg & 0xF) != 0) printf("%X\t", (uint64_t)beg & ~0xF);
  printf("%*s", (uint64_t)(beg - ((uint64_t)beg & ~0xF)) * 3, "");

  for (; beg < end; ++beg) {
    if (((uint64_t)beg & 0xF) == 0) printf("%X\t", beg);
    printf("%02X%c", *beg, ((uint64_t)beg & 0xF) == 0xF ? '\n' : ' ');
  }
}

// In main function
ViewMemory((uint8_t *)&SayHello, (uint8_t *)&SayBye);

/* Output
401550  55 48 89 E5 48 83 EC 20 48 8D 0D A1 2A 00 00 E8
401560  FC 15 00 00 90 48 83 C4 20 5D C3
*/

/* Found in executable
950     55 48 89 E5 48 83 EC 20 48 8D 0D A1 2A 00 00 E8
960     FC 15 00 00 90 48 83 C4 20 5D C3 55 48 89 E5 48
*/
```

## 1.5 Caches Matter

To deal with the processor–memory gap, system designers include smaller, faster storage devices called cache memories (or simply caches) that serve as temporary staging areas for information that the processor is likely to need in the near future.

## 1.6 Storage Devices Form a Hierarchy

The main idea of a memory hierarchy is that storage at one level serves as a cache for storage at the next lower level. Thus, the register file is a cache for the L1 cache. Caches L1 and L2 are caches for L2 and L3, respectively. The L3 cache is a cache for the main memory, which is a cache for the disk.

## 1.7 The Operating System Manages the Hardware

The operating system has two primary purposes: (1) to protect the hardware from misuse by runaway applications and (2) to provide applications with simple and uniform mechanisms for manipulating complicated and often wildly different low-level hardware devices.

### 1.7.1 Processes

When a program such as hello runs on a modern system, the operating system provides the illusion that the program is the only one running on the system. The program appears to have exclusive use of both the processor, main memory, and I/O devices. The processor appears to execute the instructions in the program, one after the other, without interruption.Andthe codeand data ofthe programappear to be the only objects in the system’s memory.

### 1.7.2 Threads

it is easier to share data between multiple threads than between multiple processes, and because threads are typically more efficient than processes.

### 1.7.4 Files

Every I/O device, including disks, keyboards, displays, and even networks, is modeled as a file. All input and output in the system is performed by reading and writing files, using a small set of system calls known as Unix I/O.

## 1.9 Important Themes

### 1.9.1 Amdahl’s Law

The main idea is that when we speed up one part of a system, the effect on the overall system performance depends on both how significant this part was and how much it sped up.

$T_{new} = (1− α)T_{old} + (αT_{old})/k = T_{old}[(1− α) + α/k]$
$S = T_{old}/T_{new} = \frac{1}{(1− α) + α/k}$

Even though we made a substantial improvement to a major part of the system, our net speedup was significantly less than the speedup for the one part. This is the major insight of Amdahl’s law— to significantly speed up the entire system, we must improve the speed of a very large fraction of the overall system.

### 1.9.2 Concurrency and Parallelism

We use the term concurrency to refer to the general concept of a system with multiple, simultaneous activities, and the term parallelism to refer to the use of concurrency to make a system run faster.

#### Thread-Level Concurrency

Multi-core processors have several CPUs (referred to as “cores”) integrated onto a single integrated-circuit chip.

(CPU cores) each with its own L1 and L2 caches, and with each L1 cache split into two parts—one to hold recently fetched instructions and one to hold data. The cores share higher levels of cache as well as the interface to main memory.

(Hyperthreading) It involves having multiple copies of some of the CPU hardware, such as program counters and register files, while having only single copies of other parts of the hardware, such as the units that perform floating-point arithmetic.

### 1.9.3 The Importance of Abstractions in Computer Systems

the virtual machine, providing an abstraction of the entire computer, including the operating system, the processor, and the programs.

