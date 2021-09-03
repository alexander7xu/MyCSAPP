# C2 - Representing and Manipulating Information

In isolation, a single bit is not very useful. When we group bits together and apply some interpretation that gives meaning to the different possible bit patterns, however, we can represent the elements of any finite set. For example, using a binary number system, we can use groups of bits to encode nonnegative numbers. By using a standard character code, we can encode the letters and symbols in a document. We cover both of these encodings in this chapter, as well as encodings to represent negative numbers and to approximate real numbers.

## 2.1 Information Storage

Rather than accessing individual bits in memory, most computers use blocks of 8 bits, or bytes, as the smallest addressable unit of memory. A machine-level program views memory as a very large array of bytes, referred to as virtual memory. Every byte of memory is identified by a unique number, known as its address, and the set of all possible addresses is known as the virtual address space. As indicated by its name, this virtual address space is just a conceptual image presented to the machine-level program. The actual implementation (presented in Chapter 9) uses a combination of dynamic random access memory (DRAM), flash memory, disk storage, special hardware, and operating system software to provide the program with what appears to be a monolithic byte array.

For example, the value of a pointer in C—whether it points to an integer, a structure, or some other program object—is the virtual address of the first byte of some block of storage. The C compiler also associates type information with each pointer, so that it can generate different machine-level code to access the value stored at the location designated by the pointer depending on the type of that value. Although the C compiler maintains this type information, the actual machine-level program it generates has no information about data types. It simply treats each program object as a block of bytes and the program itself as a sequence of bytes.

### 2.1.1 Hexadecimal Notation

When a value x is a power of 2, that is, x = 2n for some nonnegative integer n, we can readily write x in hexadecimal form by remembering that the binary representation of x is simply 1 followed by n zeros. The hexadecimal digit 0 represents 4 binary zeros. So, for n written in the form i + 4j, where 0 ≤ i ≤ 3, we can write x with a leading hex digit of 1 (i = 0), 2 (i = 1), 4 (i = 2), or 8 (i = 3), followed by j hexadecimal 0s. As an example, for x = 2,048 = 211,we have n = 11= 3 + 4 . 2, giving hexadecimal representation 0x800.

### 2.1.3 Addressing and Byte Ordering

Many recent microprocessor chips are bi-endian, meaning that they can be configured to operate as either little- or big-endian machines. In practice, however, byte ordering becomes fixed once a particular operating system is chosen. For example, ARM microprocessors, used in many cell phones, have hardware that can operate in either little- or big-endian mode, but the two most common operating systems for these chips—Android (from Google) and IOS (from Apple)—operate only in little-endian mode.

A common problem is for data produced by a little-endian machine to be sent to a big-endian machine, or vice versa, leading to the bytes within the words being in reverse order for the receiving program. To avoid such problems, code written for networking applications must follow established conventions for byte ordering to make sure the sending machine converts its internal representation to the network standard, while the receiving machine converts the network standard to its internal representation.

```c
// gcc bulitin
#if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
puts("little endian machine");
#elif __BYTE_ORDER__ == __ORDER_BIG_ENDIAN__
puts("big endian machine");
#endif
uint32_t x = 0x12345678;
uint32_t y = __builtin_bswap32(x);
printf("%x\n%x\n", x, y);
```
```
little endian machine
0x12345678
0x78563412
```

### 2.1.4 Representing Strings

A string in C is encoded by an array of characters terminated by the null (having value 0) character. Each character is represented by some standard encoding, with the most common being the ASCII character code.

> Note: C standard library provides some functions in <string.h> to handle strings easily. However, it is easy to cause memory problems, especially for those programmers new to C.

```c
// The code that caused the heap overflow while handling CString

const char kHello[] = "hello";
printf("strlen: %d\nsizeof: %d", strlen(kHello), sizeof(kHello));

// strlen(str) return the length of str, without the null character
char *buf = (char *)malloc(strlen(kHello));
// strcpy(dst, src) will copy the null character from src to dst
strcpy(buf, kHello);
// to fix this bug, replace strlen(kHello) with strlen(kHello) + 1
```
```
strlen: 5
sizeof: 6
```

### 2.1.9 Shift Operations in C

The C standards do not precisely define which type of right shift should be used with signed numbers—either arithmetic or logical shifts may be used. This unfortunately means that any code assuming one form or the other will potentially encounter portability problems. In practice, however, **almost all compiler/machine combinations use arithmetic right shifts for signed data, and many programmers assume this to be the case. For unsigned data, on the other hand, right shifts must be logical.**

```c
printf("0x%X\n", (int16_t)0x8086 >> 8);
printf("0x%X\n", (uint16_t)0x8086 >> 8);
```
```
0xFFFFFF80
0x80
```

## 2.2 Integer Representations

### 2.2.4 Conversions between Signed and Unsigned

The effect of casting is to keep the bit values identical but change how these bits are interpreted. The numeric values might change, but the bit patterns do not.

PRINCIPLE: Conversion from two’s complement to unsigned

$$
For x such that TMin_w\le x\le TMax_w:\\\\
T2U(x) =
\begin{cases}
x+2^w, & x<0\\\\
x, & x\ge 0
\end{cases}
$$

PRINCIPLE: Unsigned to two’s-complement conversion

$$
For u such that 0\le u\le UMax_w:\\\\
U2T(u) =
\begin{cases}
u, & u\le TMax_w\\\\
u-2^w, & u> TMax_w
\end{cases}
$$

### 2.2.6 Expanding the Bit Representation of a Number

Expanding the bit representation of a number will however, keep the numeric values identical.

To convert an unsigned number to a larger data type, we can simply add leading zeros to the representation; this operation is known as zero extension.

For converting a two’s-complement number to a larger data type, the rule is to perform a sign extension, adding copies of the most significant bit to the representation, expressed by the following principle. **It can be proved by MI because $-2^{w-1}=-2^w+2^{w-1}$**

The relative order of conversion from one data size to another and between unsigned and signed: first changes the size and then the type. i.e.`(uint64_t) s32` is equivalent to `(unsigned)(int64_t) s32`.

### 2.2.7 Truncating Numbers

**When truncating a w-bit number to a k-bit number, we simply drop the high-order w − k bits.**

## 2.3 Integer Arithmetic

### 2.3.1 Unsigned Addition

PRINCIPLE: Detecting overflow of unsigned addition

For $x$ and $y$ in the range $0\le x, y\le UMax_w$, let $s = x +^u_w y$. Then the computation of $s$ overflowed if and only if $s<x$ (or equivalently, $s<y$).

```c
int IsIntAddOverflow(int x, int y) { return x + y < x; }
```

### 2.3.2 Two’s-Complement Addition

The w-bit two’s-complement sum of two numbers has the exact same bit-level representation as the unsigned sum. In fact, most computers use the same machine instruction to perform either unsigned or signed addition.

```c
int IsUintAddOverflow(unsigned int x, unsigned int y) {
  return !((x < 0 && y < 0 && x + y > 0) || (x > 0 && y > 0 && x + y < 0));
}
```

**Integer addition forms an abelian group.**

### 2.3.3 Two’s-Complement Negation

**-x=~x+1$, Convert subtraction to addition: $y-x=y+(-x)=y+~x+1**

$TMin_{w} = -TMin_{w}$


### 2.3.5 Two’s-Complement Multiplication

Although the bit-level representations of the full products may differ, **those of the truncated products are identical.**

No matter unsigned or two’s-complement, addition or multiplication, we could **do the corresponding operaterion then discard overflowed bits in bit-level representation**. In numeric representation, because discarding bits is equivalent to set those bits as zero, it looks like $Mod 2^w$ in unsigned case.

## 2.4 Floating Point

### 2.4.2 IEEE Floating-Point Representation

| type | bit width | exp | frac | precision | postive abs range |
|-|-|-|-|-|-|
| single | 32 | 8 | 23 | 6 | 1.401298e-45 ~ 3.402823e+38 |
| double | 64 | 11 | 52 | 15 | 4.940656e-324 ~ 1.797693e+308 |

precision: $\lfloor log_{10}(2^{frac+1})\rfloor$

postive abs range: $2^{-frac}*2^{2-2^{exp-1}}$ ~ $(2-2^{-frac})*2^{2^{exp-1}-1}$

view postive abs range as int: $1$ ~ $2^{exp+frac-1}$

```c
// floating-point in C

#ifdef __MINGW64__
#define printf __mingw_printf  // long double work wrong without this line
#endif                         // __MINGW_GCC_VERSION
printf("Type\t\tsize\tprec\tabs range\n");
printf("float\t\t%d\t%d\t%-14e ~ %e\n",
       sizeof(float), FLT_DIG, FLT_TRUE_MIN, FLT_MAX);
printf("double\t\t%d\t%d\t%-14e ~ %e\n",
       sizeof(double), DBL_DIG, DBL_TRUE_MIN, DBL_MAX);
printf("long double\t%d\t%d\t%Le ~ %Le\n",
       sizeof(long double), LDBL_DIG, LDBL_TRUE_MIN, LDBL_MAX);
```
```
Type            size    prec    abs range
float           4       6       1.175494e-38   ~ 3.402823e+38
double          8       15      2.225074e-308  ~ 1.797693e+308
long double     16      18      3.362103e-4932 ~ 1.189731e+4932
```

#### Case 1: Normalized Values

When the bit pattern of exp is neither all zeros nor all ones, the exponent field is interpreted as representing a signed integer in biased form. That is, the exponent value is $E=e−Bias$, where $e$ is the unsigned number having bit representation $e_{k−1}...e_1e_0$ and **Bias is a bias value equal to $2^{k−1}−1$. (So that E range form $2-2^{k-1}$ to $2^{k-1}-1$)**

The significand is defined to be $M=1+f$. This is sometimes called an implied leading 1 representation, because we can viewM to be the number with binary representation $1.f_{n−1}f_{n−2}...f_0$. **This representation is a trick for getting an additional bit of precision for free**, since we can always adjust the exponent E so that significand M is in the range $1\le M<2$ (assuming there is no overflow). We therefore do not need to explicitly represent the leading bit, since it always equals 1.

#### Case 2: Denormalized Values

When the exponent field is all zeros, the represented number is in denormalized form. In this case, the exponent value is $E=1−Bias$, and the significand value is $M=f$, that is, the value of the fraction field without an implied leading 1.

Denormalized numbers serve two purposes. First, they **provide a way to represent numeric value 0**, since with a normalized number we must always have $M\ge 1$, and hence we cannot represent 0. The floating-point representation of +0.0 has a bit pattern of all zeros. Curiously, when the sign bit is 1, but the other fields are all zeros, we get the value −0.0.

A second function of denormalized numbers is to represent numbers that are very close to 0.0. They **provide a property known as gradual underflow in which possible numeric values are spaced evenly near 0.0**.

#### Case 3: Special Values

A final category of values occurs when the exponent field is all ones. When the fraction field is all zeros, the resulting values represent infinity, either $+\infty$ when $s=0$ or $−\infty$ when $s=1$. **Infinity can represent results that overflow**. When the fraction field is nonzero, the resulting value is called a $NaN$, short for “not a number.”

**the IEEE format was designed so that floating-point numbers could be sorted using an integer sorting routine.**

- If we write some postive floating point in ascending order then interpret the bit representations of the values as unsigned integers, they occur in ascending order.
- However, when dealing with negative numbers, they have a leading 1 and occur in descending order

```c
int Float32Cmp(float x, float y) {
  uint32_t ux = *(int *)&x;
  uint32_t uy = *(int *)&y;
  uint32_t sx = ux >> 31;
  uint32_t sy = uy >> 31;

  if (isnan(x) || isnan(y)) return (1 << ((2 << sizeof(int)) - 1));
  if (((ux << 1) == 0 && (uy << 1) == 0) || ux == uy) return 0;
  if ((sx & !sy) || (!sx && !sy && ux < uy) || (sx && sy && ux > uy)) return -1;
  return 1;
}
```

### 2.4.4 Rounding

Round-to-even mode adopts the convention that it rounds the number either upward or downward such that the least significant digit of the result is even. Thus, it rounds both $1.50 and $2.50 to $2.

Rounding toward even numbers avoids this statistical bias in most real-life situations. It will round upward about50% of the time and round downward about 50% of the time.


