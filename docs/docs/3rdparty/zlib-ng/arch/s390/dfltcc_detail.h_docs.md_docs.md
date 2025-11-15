# Documentation for `docs/3rdparty/zlib-ng/arch/s390/dfltcc_detail.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/zlib-ng/arch/s390/dfltcc_detail.h_docs.md`
- **File Name**: `dfltcc_detail.h_docs.md`
- **File Size**: 12,668 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/zlib-ng/arch/s390/dfltcc_detail.h_docs.md](../../../../../docs/3rdparty/zlib-ng/arch/s390/dfltcc_detail.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/zlib-ng/arch/s390` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/zlib-ng/arch/s390/dfltcc_detail.h`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/arch/s390/dfltcc_detail.h`
- **File Name**: `dfltcc_detail.h`
- **File Size**: 8,941 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/zlib-ng/arch/s390/dfltcc_detail.h](../../../../3rdparty/zlib-ng/arch/s390/dfltcc_detail.h)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng/arch/s390` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "zbuild.h"
#include <stdio.h>

#ifdef HAVE_SYS_SDT_H
#include <sys/sdt.h>
#endif

/*
   Tuning parameters.
 */
#ifndef DFLTCC_LEVEL_MASK
#define DFLTCC_LEVEL_MASK 0x2
#endif
#ifndef DFLTCC_BLOCK_SIZE
#define DFLTCC_BLOCK_SIZE 1048576
#endif
#ifndef DFLTCC_FIRST_FHT_BLOCK_SIZE
#define DFLTCC_FIRST_FHT_BLOCK_SIZE 4096
#endif
#ifndef DFLTCC_DHT_MIN_SAMPLE_SIZE
#define DFLTCC_DHT_MIN_SAMPLE_SIZE 4096
#endif
#ifndef DFLTCC_RIBM
#define DFLTCC_RIBM 0
#endif

#define static_assert(c, msg) __attribute__((unused)) static char static_assert_failed_ ## msg[c ? 1 : -1]

#define DFLTCC_SIZEOF_QAF 32
static_assert(sizeof(struct dfltcc_qaf_param) == DFLTCC_SIZEOF_QAF, qaf);

static inline int is_bit_set(const char *bits, int n) {
    return bits[n / 8] & (1 << (7 - (n % 8)));
}

static inline void clear_bit(char *bits, int n) {
    bits[n / 8] &= ~(1 << (7 - (n % 8)));
}

#define DFLTCC_FACILITY 151

static inline int is_dfltcc_enabled(void) {
    uint64_t facilities[(DFLTCC_FACILITY / 64) + 1];
    Z_REGISTER uint8_t r0 __asm__("r0");

    memset(facilities, 0, sizeof(facilities));
    r0 = sizeof(facilities) / sizeof(facilities[0]) - 1;
    /* STFLE is supported since z9-109 and only in z/Architecture mode. When
     * compiling with -m31, gcc defaults to ESA mode, however, since the kernel
     * is 64-bit, it's always z/Architecture mode at runtime.
     */
    __asm__ volatile(
#ifndef __clang__
                     ".machinemode push\n"
                     ".machinemode zarch\n"
#endif
                     "stfle %[facilities]\n"
#ifndef __clang__
                     ".machinemode pop\n"
#endif
                     : [facilities] "=Q" (facilities), [r0] "+r" (r0) :: "cc");
    return is_bit_set((const char *)facilities, DFLTCC_FACILITY);
}

#define DFLTCC_FMT0 0

#define CVT_CRC32 0
#define CVT_ADLER32 1
#define HTT_FIXED 0
#define HTT_DYNAMIC 1

#define DFLTCC_SIZEOF_GDHT_V0 384
#define DFLTCC_SIZEOF_CMPR_XPND_V0 1536
static_assert(offsetof(struct dfltcc_param_v0, csb) == DFLTCC_SIZEOF_GDHT_V0, gdht_v0);
static_assert(sizeof(struct dfltcc_param_v0) == DFLTCC_SIZEOF_CMPR_XPND_V0, cmpr_xpnd_v0);

static inline z_const char *oesc_msg(char *buf, int oesc) {
    if (oesc == 0x00)
        return NULL; /* Successful completion */
    else {
        sprintf(buf, "Operation-Ending-Supplemental Code is 0x%.2X", oesc);
        return buf;
    }
}

/*
   C wrapper for the DEFLATE CONVERSION CALL instruction.
 */
typedef enum {
    DFLTCC_CC_OK = 0,
    DFLTCC_CC_OP1_TOO_SHORT = 1,
    DFLTCC_CC_OP2_TOO_SHORT = 2,
    DFLTCC_CC_OP2_CORRUPT = 2,
    DFLTCC_CC_AGAIN = 3,
} dfltcc_cc;

#define DFLTCC_QAF 0
#define DFLTCC_GDHT 1
#define DFLTCC_CMPR 2
#define DFLTCC_XPND 4
#define HBT_CIRCULAR (1 << 7)
#define DFLTCC_FN_MASK ((1 << 7) - 1)

/* Return lengths of high (starting at param->ho) and low (starting at 0) fragments of the circular history buffer. */
static inline void get_history_lengths(struct dfltcc_param_v0 *param, size_t *hl_high, size_t *hl_low) {
    *hl_high = MIN(param->hl, HB_SIZE - param->ho);
    *hl_low = param->hl - *hl_high;
}

/* Notify instrumentation about an upcoming read/write access to the circular history buffer. */
static inline void instrument_read_write_hist(struct dfltcc_param_v0 *param, void *hist) {
    size_t hl_high, hl_low;

    get_history_lengths(param, &hl_high, &hl_low);
    instrument_read_write(hist + param->ho, hl_high);
    instrument_read_write(hist, hl_low);
}

/* Notify MSan about a completed write to the circular history buffer. */
static inline void msan_unpoison_hist(struct dfltcc_param_v0 *param, void *hist) {
    size_t hl_high, hl_low;

    get_history_lengths(param, &hl_high, &hl_low);
    __msan_unpoison(hist + param->ho, hl_high);
    __msan_unpoison(hist, hl_low);
}

static inline dfltcc_cc dfltcc(int fn, void *param,
                               unsigned char **op1, size_t *len1,
                               z_const unsigned char **op2, size_t *len2, void *hist) {
    unsigned char *t2 = op1 ? *op1 : NULL;
    unsigned char *orig_t2 = t2;
    size_t t3 = len1 ? *len1 : 0;
    z_const unsigned char *t4 = op2 ? *op2 : NULL;
    size_t t5 = len2 ? *len2 : 0;
    Z_REGISTER int r0 __asm__("r0");
    Z_REGISTER void *r1 __asm__("r1");
    Z_REGISTER unsigned char *r2 __asm__("r2");
    Z_REGISTER size_t r3 __asm__("r3");
    Z_REGISTER z_const unsigned char *r4 __asm__("r4");
    Z_REGISTER size_t r5 __asm__("r5");
    int cc;

    /* Insert pre-instrumentation for DFLTCC. */
    switch (fn & DFLTCC_FN_MASK) {
    case DFLTCC_QAF:
        instrument_write(param, DFLTCC_SIZEOF_QAF);
        break;
    case DFLTCC_GDHT:
        instrument_read_write(param, DFLTCC_SIZEOF_GDHT_V0);
        instrument_read(t4, t5);
        break;
    case DFLTCC_CMPR:
    case DFLTCC_XPND:
        instrument_read_write(param, DFLTCC_SIZEOF_CMPR_XPND_V0);
        instrument_read(t4, t5);
        instrument_write(t2, t3);
        instrument_read_write_hist(param, hist);
        break;
    }

    r0 = fn; r1 = param; r2 = t2; r3 = t3; r4 = t4; r5 = t5;
    __asm__ volatile(
#ifdef HAVE_SYS_SDT_H
                     STAP_PROBE_ASM(zlib, dfltcc_entry, STAP_PROBE_ASM_TEMPLATE(5))
#endif
                     ".insn rrf,0xb9390000,%[r2],%[r4],%[hist],0\n"
#ifdef HAVE_SYS_SDT_H
                     STAP_PROBE_ASM(zlib, dfltcc_exit, STAP_PROBE_ASM_TEMPLATE(5))
#endif
                     "ipm %[cc]\n"
                     : [r2] "+r" (r2)
                     , [r3] "+r" (r3)
                     , [r4] "+r" (r4)
                     , [r5] "+r" (r5)
                     , [cc] "=r" (cc)
                     : [r0] "r" (r0)
                     , [r1] "r" (r1)
                     , [hist] "r" (hist)
#ifdef HAVE_SYS_SDT_H
                     , STAP_PROBE_ASM_OPERANDS(5, r2, r3, r4, r5, hist)
#endif
                     : "cc", "memory");
    t2 = r2; t3 = r3; t4 = r4; t5 = r5;

    /* Insert post-instrumentation for DFLTCC. */
    switch (fn & DFLTCC_FN_MASK) {
    case DFLTCC_QAF:
        __msan_unpoison(param, DFLTCC_SIZEOF_QAF);
        break;
    case DFLTCC_GDHT:
        __msan_unpoison(param, DFLTCC_SIZEOF_GDHT_V0);
        break;
    case DFLTCC_CMPR:
        __msan_unpoison(param, DFLTCC_SIZEOF_CMPR_XPND_V0);
        __msan_unpoison(orig_t2, t2 - orig_t2 + (((struct dfltcc_param_v0 *)param)->sbb == 0 ? 0 : 1));
        msan_unpoison_hist(param, hist);
        break;
    case DFLTCC_XPND:
        __msan_unpoison(param, DFLTCC_SIZEOF_CMPR_XPND_V0);
        __msan_unpoison(orig_t2, t2 - orig_t2);
        msan_unpoison_hist(param, hist);
        break;
    }

    if (op1)
        *op1 = t2;
    if (len1)
        *len1 = t3;
    if (op2)
        *op2 = t4;
    if (len2)
        *len2 = t5;
    return (cc >> 28) & 3;
}

#define ALIGN_UP(p, size) (__typeof__(p))(((uintptr_t)(p) + ((size) - 1)) & ~((size) - 1))

static inline void dfltcc_reset_state(struct dfltcc_state *dfltcc_state) {
    /* Initialize available functions */
    if (is_dfltcc_enabled()) {
        dfltcc(DFLTCC_QAF, &dfltcc_state->param, NULL, NULL, NULL, NULL, NULL);
        memmove(&dfltcc_state->af, &dfltcc_state->param, sizeof(dfltcc_state->af));
    } else
        memset(&dfltcc_state->af, 0, sizeof(dfltcc_state->af));

    /* Initialize parameter block */
    memset(&dfltcc_state->param, 0, sizeof(dfltcc_state->param));
    dfltcc_state->param.nt = 1;
    dfltcc_state->param.ribm = DFLTCC_RIBM;
}

static inline void dfltcc_copy_state(void *dst, const void *src, uInt size, uInt extension_size) {
    memcpy(dst, src, ALIGN_UP(size, 8) + extension_size);
}

static inline void append_history(struct dfltcc_param_v0 *param, unsigned char *history,
                                  const unsigned char *buf, uInt count) {
    size_t offset;
    size_t n;

    /* Do not use more than 32K */
    if (count > HB_SIZE) {
        buf += count - HB_SIZE;
        count = HB_SIZE;
    }
    offset = (param->ho + param->hl) % HB_SIZE;
    if (offset + count <= HB_SIZE)
        /* Circular history buffer does not wrap - copy one chunk */
        memcpy(history + offset, buf, count);
    else {
        /* Circular history buffer wraps - copy two chunks */
        n = HB_SIZE - offset;
        memcpy(history + offset, buf, n);
        memcpy(history, buf + n, count - n);
    }
    n = param->hl + count;
    if (n <= HB_SIZE)
        /* All history fits into buffer - no need to discard anything */
        param->hl = n;
    else {
        /* History does not fit into buffer - discard extra bytes */
        param->ho = (param->ho + (n - HB_SIZE)) % HB_SIZE;
        param->hl = HB_SIZE;
    }
}

static inline void get_history(struct dfltcc_param_v0 *param, const unsigned char *history,
                               unsigned char *buf) {
    size_t hl_high, hl_low;

    get_history_lengths(param, &hl_high, &hl_low);
    memcpy(buf, history + param->ho, hl_high);
    memcpy(buf + hl_high, history, hl_low);
}
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **dfltcc_param_v0**: A class/struct defined in this file
- **dfltcc_qaf_param**: A class/struct defined in this file
- **dfltcc_state**: A class/struct defined in this file

### Functions and Methods

- **DFLTCC_BLOCK_SIZE()**: A function/method defined in this file
- **DFLTCC_FIRST_FHT_BLOCK_SIZE()**: A function/method defined in this file
- **enum()**: A function/method defined in this file
- **HAVE_SYS_SDT_H()**: A function/method defined in this file
- **DFLTCC_RIBM()**: A function/method defined in this file
- **__clang__()**: A function/method defined in this file
- **DFLTCC_LEVEL_MASK()**: A function/method defined in this file
- **DFLTCC_DHT_MIN_SAMPLE_SIZE()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `zbuild.h`
- `stdio.h`
- `sys/sdt.h`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

