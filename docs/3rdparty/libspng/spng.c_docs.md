# Documentation for `3rdparty/libspng/spng.c`

## File Metadata

- **Full Path**: `3rdparty/libspng/spng.c`
- **File Name**: `spng.c`
- **File Size**: 199,677 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libspng/spng.c](../../3rdparty/libspng/spng.c)

## Purpose and Role

This file is located in the `3rdparty/libspng` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* SPDX-License-Identifier: (BSD-2-Clause AND libpng-2.0) */

#define SPNG__BUILD

#include "spng.h"

#include <limits.h>
#include <string.h>
#include <stdio.h>
#include <math.h>

#define ZLIB_CONST

#ifdef __FRAMAC__
    #define SPNG_DISABLE_OPT
    #include "tests/framac_stubs.h"
#else
    #ifdef SPNG_USE_MINIZ
        #include <miniz.h>
    #else
        #include <zlib.h>
    #endif
#endif

#ifdef SPNG_MULTITHREADING
    #include <pthread.h>
#endif

/* Not build options, edit at your own risk! */
#define SPNG_READ_SIZE (8192)
#define SPNG_WRITE_SIZE SPNG_READ_SIZE
#define SPNG_MAX_CHUNK_COUNT (1000)

#define SPNG_TARGET_CLONES(x)

#ifndef SPNG_DISABLE_OPT

    #if defined(__i386__) || defined(__x86_64__) || defined(_M_IX86) || defined(_M_X64)
        #define SPNG_X86

        #if defined(__x86_64__) || defined(_M_X64)
            #define SPNG_X86_64
        #endif

    #elif defined(__aarch64__) || defined(_M_ARM64) /* || defined(__ARM_NEON) */
        #define SPNG_ARM /* NOTE: only arm64 builds are tested! */
    #else
        #pragma message "disabling SIMD optimizations for unknown target"
        #define SPNG_DISABLE_OPT
    #endif

    #if defined(SPNG_X86_64) && defined(SPNG_ENABLE_TARGET_CLONES)
        #undef SPNG_TARGET_CLONES
        #define SPNG_TARGET_CLONES(x) __attribute__((target_clones(x)))
    #else
        #define SPNG_TARGET_CLONES(x)
    #endif

    #ifndef SPNG_DISABLE_OPT
        static void defilter_sub3(size_t rowbytes, unsigned char *row);
        static void defilter_sub4(size_t rowbytes, unsigned char *row);
        static void defilter_avg3(size_t rowbytes, unsigned char *row, const unsigned char *prev);
        static void defilter_avg4(size_t rowbytes, unsigned char *row, const unsigned char *prev);
        static void defilter_paeth3(size_t rowbytes, unsigned char *row, const unsigned char *prev);
        static void defilter_paeth4(size_t rowbytes, unsigned char *row, const unsigned char *prev);

        #if defined(SPNG_ARM)
        static uint32_t expand_palette_rgba8_neon(unsigned char *row, const unsigned char *scanline, const unsigned char *plte, uint32_t width);
        static uint32_t expand_palette_rgb8_neon(unsigned char *row, const unsigned char *scanline, const unsigned char *plte, uint32_t width);
        #endif
    #endif
#endif

#if defined(_MSC_VER)
    #pragma warning(push)
    #pragma warning(disable: 4244)
#endif

#if (defined(__BYTE_ORDER__) && __BYTE_ORDER__ == __ORDER_BIG_ENDIAN__) || defined(__BIG_ENDIAN__)
    #define SPNG_BIG_ENDIAN
#else
    #define SPNG_LITTLE_ENDIAN
#endif

enum spng_state
{
    SPNG_STATE_INVALID = 0,
    SPNG_STATE_INIT = 1, /* No PNG buffer/stream is set */
    SPNG_STATE_INPUT, /* Decoder input PNG was set */
    SPNG_STATE_OUTPUT = SPNG_STATE_INPUT, /* Encoder output was set */
    SPNG_STATE_IHDR, /* IHDR was read/written */
    SPNG_STATE_FIRST_IDAT,  /* Encoded up to / reached first IDAT */
    SPNG_STATE_DECODE_INIT, /* Decoder is ready for progressive reads */
    SPNG_STATE_ENCODE_INIT = SPNG_STATE_DECODE_INIT,
    SPNG_STATE_EOI, /* Reached the last scanline/row */
    SPNG_STATE_LAST_IDAT, /* Reached last IDAT, set at end of decode_image() */
    SPNG_STATE_AFTER_IDAT, /*  */
    SPNG_STATE_IEND, /* Reached IEND */
};

enum spng__internal
{
    SPNG__IO_SIGNAL = 1 << 9,
    SPNG__CTX_FLAGS_ALL = (SPNG_CTX_IGNORE_ADLER32 | SPNG_CTX_ENCODER)
};

#define SPNG_STR(x) _SPNG_STR(x)
#define _SPNG_STR(x) #x

#define SPNG_VERSION_STRING SPNG_STR(SPNG_VERSION_MAJOR) "." \
                            SPNG_STR(SPNG_VERSION_MINOR) "." \
                            SPNG_STR(SPNG_VERSION_PATCH)

#define SPNG_GET_CHUNK_BOILERPLATE(chunk) \
    if(ctx == NULL) return 1; \
    int ret = read_chunks(ctx, 0); \
    if(ret) return ret; \
    if(!ctx->stored.chunk) return SPNG_ECHUNKAVAIL; \
    if(chunk == NULL) return 1

#define SPNG_SET_CHUNK_BOILERPLATE(chunk) \
    if(ctx == NULL || chunk == NULL) return 1; \
    if(ctx->data == NULL && !ctx->encode_only) return SPNG_ENOSRC; \
    int ret = read_chunks(ctx, 0); \
    if(ret) return ret

/* Determine if the spng_option can be overriden/optimized */
#define spng__optimize(option) (ctx->optimize_option & (1 << option))

struct spng_subimage
{
    uint32_t width;
    uint32_t height;
    size_t out_width; /* byte width based on output format */
    size_t scanline_width;
};

struct spng_text2
{
    int type;
    char *keyword;
    char *text;

    size_t text_length;

    uint8_t compression_flag; /* iTXt only */
    char *language_tag; /* iTXt only */
    char *translated_keyword; /* iTXt only */

    size_t cache_usage;
    char user_keyword_storage[80];
};

struct decode_flags
{
    unsigned apply_trns:  1;
    unsigned apply_gamma: 1;
    unsigned use_sbit:    1;
    unsigned indexed:     1;
    unsigned do_scaling:  1;
    unsigned interlaced:  1;
    unsigned same_layout: 1;
    unsigned zerocopy:    1;
    unsigned unpack:      1;
};

struct encode_flags
{
    unsigned interlace:      1;
    unsigned same_layout:    1;
    unsigned to_bigendian:   1;
    unsigned progressive:    1;
    unsigned finalize:       1;

    enum spng_filter_choice filter_choice;
};

struct spng_chunk_bitfield
{
    unsigned ihdr: 1;
    unsigned plte: 1;
    unsigned chrm: 1;
    unsigned iccp: 1;
    unsigned gama: 1;
    unsigned sbit: 1;
    unsigned srgb: 1;
    unsigned text: 1;
    unsigned bkgd: 1;
    unsigned hist: 1;
    unsigned trns: 1;
    unsigned phys: 1;
    unsigned splt: 1;
    unsigned time: 1;
    unsigned offs: 1;
    unsigned exif: 1;
    unsigned unknown: 1;
};

/* Packed sample iterator */
struct spng__iter
{
    const uint8_t mask;
    unsigned shift_amount;
    const unsigned initial_shift, bit_depth;
    const unsigned char *samples;
};

union spng__decode_plte
{
    struct spng_plte_entry rgba[256];
    unsigned char rgb[256 * 3];
    unsigned char raw[256 * 4];
    uint32_t align_this;
};

struct spng__zlib_options
{
    int compression_level;
    int window_bits;
    int mem_level;
    int strategy;
    int data_type;
};

typedef void spng__undo(spng_ctx *ctx);

struct spng_ctx
{
    size_t data_size;
    size_t bytes_read;
    size_t stream_buf_size;
    unsigned char *stream_buf;
    const unsigned char *data;

    /* User-defined pointers for streaming */
    spng_read_fn *read_fn;
    spng_write_fn *write_fn;
    void *stream_user_ptr;

    /* Used for buffer reads */
    const unsigned char *png_base;
    size_t bytes_left;
    size_t last_read_size;

    /* Used for encoding */
    int user_owns_out_png;
    unsigned char *out_png;
    unsigned char *write_ptr;
    size_t out_png_size;
    size_t bytes_encoded;

    /* These are updated by read/write_header()/read_chunk_bytes() */
    struct spng_chunk current_chunk;
    uint32_t cur_chunk_bytes_left;
    uint32_t cur_actual_crc;

    struct spng_alloc alloc;

    enum spng_ctx_flags flags;
    enum spng_format fmt;

    enum spng_state state;

    unsigned streaming: 1;
    unsigned internal_buffer: 1; /* encoding to internal buffer */

    unsigned inflate: 1;
    unsigned deflate: 1;
    unsigned encode_only: 1;
    unsigned strict: 1;
    unsigned discard: 1;
    unsigned skip_crc: 1;
    unsigned keep_unknown: 1;
    unsigned prev_was_idat: 1;

    struct spng__zlib_options image_options;
    struct spng__zlib_options text_options;

    spng__undo *undo;

    /* input file contains this chunk */
    struct spng_chunk_bitfield file;

    /* chunk was stored with spng_set_*() */
    struct spng_chunk_bitfield user;

    /* chunk was stored by reading or with spng_set_*() */
    struct spng_chunk_bitfield stored;

    /* used to reset the above in case of an error */
    struct spng_chunk_bitfield prev_stored;

    struct spng_chunk first_idat, last_idat;

    uint32_t max_width, max_height;

    size_t max_chunk_size;
    size_t chunk_cache_limit;
    size_t chunk_cache_usage;
    uint32_t chunk_count_limit;
    uint32_t chunk_count_total;

    int crc_action_critical;
    int crc_action_ancillary;

    uint32_t optimize_option;

    struct spng_ihdr ihdr;

    struct spng_plte plte;

    struct spng_chrm_int chrm_int;
    struct spng_iccp iccp;

    uint32_t gama;

    struct spng_sbit sbit;

    uint8_t srgb_rendering_intent;

    uint32_t n_text;
    struct spng_text2 *text_list;

    struct spng_bkgd bkgd;
    struct spng_hist hist;
    struct spng_trns trns;
    struct spng_phys phys;

    uint32_t n_splt;
    struct spng_splt *splt_list;

    struct spng_time time;
    struct spng_offs offs;
    struct spng_exif exif;

    uint32_t n_chunks;
    struct spng_unknown_chunk *chunk_list;

    struct spng_subimage subimage[7];

    z_stream zstream;
    unsigned char *scanline_buf, *prev_scanline_buf, *row_buf, *filtered_scanline_buf;
    unsigned char *scanline, *prev_scanline, *row, *filtered_scanline;

    /* based on fmt */
    size_t image_size; /* may be zero */
    size_t image_width;

    unsigned bytes_per_pixel; /* derived from ihdr */
    unsigned pixel_size; /* derived from spng_format+ihdr */
    int widest_pass;
    int last_pass; /* last non-empty pass */

    uint16_t *gamma_lut; /* points to either _lut8 or _lut16 */
    uint16_t *gamma_lut16;
    uint16_t gamma_lut8[256];
    unsigned char trns_px[8];
    union spng__decode_plte decode_plte;
    struct spng_sbit decode_sb;
    struct decode_flags decode_flags;
    struct spng_row_info row_info;

    struct encode_flags encode_flags;
};

static const uint32_t spng_u32max = INT32_MAX;

static const uint32_t adam7_x_start[7] = { 0, 4, 0, 2, 0, 1, 0 };
static const uint32_t adam7_y_start[7] = { 0, 0, 4, 0, 2, 0, 1 };
static const uint32_t adam7_x_delta[7] = { 8, 8, 4, 4, 2, 2, 1 };
static const uint32_t adam7_y_delta[7] = { 8, 8, 8, 4, 4, 2, 2 };

static const uint8_t spng_signature[8] = { 137, 80, 78, 71, 13, 10, 26, 10 };

static const uint8_t type_ihdr[4] = { 73, 72, 68, 82 };
static const uint8_t type_plte[4] = { 80, 76, 84, 69 };
static const uint8_t type_idat[4] = { 73, 68, 65, 84 };
static const uint8_t type_iend[4] = { 73, 69, 78, 68 };

static const uint8_t type_trns[4] = { 116, 82, 78, 83 };
static const uint8_t type_chrm[4] = { 99,  72, 82, 77 };
static const uint8_t type_gama[4] = { 103, 65, 77, 65 };
static const uint8_t type_iccp[4] = { 105, 67, 67, 80 };
static const uint8_t type_sbit[4] = { 115, 66, 73, 84 };
static const uint8_t type_srgb[4] = { 115, 82, 71, 66 };
static const uint8_t type_text[4] = { 116, 69, 88, 116 };
static const uint8_t type_ztxt[4] = { 122, 84, 88, 116 };
static const uint8_t type_itxt[4] = { 105, 84, 88, 116 };
static const uint8_t type_bkgd[4] = { 98,  75, 71, 68 };
static const uint8_t type_hist[4] = { 104, 73, 83, 84 };
static const uint8_t type_phys[4] = { 112, 72, 89, 115 };
static const uint8_t type_splt[4] = { 115, 80, 76, 84 };
static const uint8_t type_time[4] = { 116, 73, 77, 69 };

static const uint8_t type_offs[4] = { 111, 70, 70, 115 };
static const uint8_t type_exif[4] = { 101, 88, 73, 102 };

static inline void *spng__malloc(spng_ctx *ctx,  size_t size)
{
    return ctx->alloc.malloc_fn(size);
}

static inline void *spng__calloc(spng_ctx *ctx, size_t nmemb, size_t size)
{
    return ctx->alloc.calloc_fn(nmemb, size);
}

static inline void *spng__realloc(spng_ctx *ctx, void *ptr, size_t size)
{
    return ctx->alloc.realloc_fn(ptr, size);
}

static inline void spng__free(spng_ctx *ctx, void *ptr)
{
    ctx->alloc.free_fn(ptr);
}

#if defined(SPNG_USE_MINIZ)
static void *spng__zalloc(void *opaque, size_t items, size_t size)
#else
static void *spng__zalloc(void *opaque, uInt items, uInt size)
#endif
{
    spng_ctx *ctx = opaque;

    if(size > SIZE_MAX / items) return NULL;

    size_t len = (size_t)items * size;

    return spng__malloc(ctx, len);
}

static void spng__zfree(void *opqaue, void *ptr)
{
    spng_ctx *ctx = opqaue;
    spng__free(ctx, ptr);
}

static inline uint16_t read_u16(const void *src)
{
    const unsigned char *data = src;

    return (data[0] & 0xFFU) << 8 | (data[1] & 0xFFU);
}

static inline uint32_t read_u32(const void *src)
{
    const unsigned char *data = src;

    return (data[0] & 0xFFUL) << 24 | (data[1] & 0xFFUL) << 16 |
           (data[2] & 0xFFUL) << 8  | (data[3] & 0xFFUL);
}

static inline int32_t read_s32(const void *src)
{
    int32_t ret = (int32_t)read_u32(src);

    return ret;
}

static inline void write_u16(void *dest, uint16_t x)
{
    unsigned char *data = dest;

    data[0] = x >> 8;
    data[1] = x & 0xFF;
}

static inline void write_u32(void *dest, uint32_t x)
{
    unsigned char *data = dest;

    data[0] = (x >> 24);
    data[1] = (x >> 16) & 0xFF;
    data[2] = (x >> 8) & 0xFF;
    data[3] = x & 0xFF;
}

static inline void write_s32(void *dest, int32_t x)
{
    uint32_t n = x;
    write_u32(dest, n);
}

/* Returns an iterator for 1,2,4,8-bit samples */
static struct spng__iter spng__iter_init(unsigned bit_depth, const unsigned char *samples)
{
    struct spng__iter iter =
    {
        .mask = (uint32_t)(1 << bit_depth) - 1,
        .shift_amount = 8 - bit_depth,
        .initial_shift = 8 - bit_depth,
        .bit_depth = bit_depth,
        .samples = samples
    };

    return iter;
}

/* Returns the current sample unpacked, iterates to the next one */
static inline uint8_t get_sample(struct spng__iter *iter)
{
    uint8_t x = (iter->samples[0] >> iter->shift_amount) & iter->mask;

    iter->shift_amount -= iter->bit_depth;

    if(iter->shift_amount > 7)
    {
        iter->shift_amount = iter->initial_shift;
        iter->samples++;
    }

    return x;
}

static void u16_row_to_host(void *row, size_t size)
{
    uint16_t *px = row;
    size_t i, n = size / 2;

    for(i=0; i < n; i++)
    {
        px[i] = read_u16(&px[i]);
    }
}

static void u16_row_to_bigendian(void *row, size_t size)
{
    uint16_t *px = (uint16_t*)row;
    size_t i, n = size / 2;

    for(i=0; i < n; i++)
    {
        write_u16(&px[i], px[i]);
    }
}

static void rgb8_row_to_rgba8(const unsigned char *row, unsigned char *out, uint32_t n)
{
    uint32_t i;
    for(i=0; i < n; i++)
    {
        memcpy(out + i * 4, row + i * 3, 3);
        out[i*4+3] = 255;
    }
}

static unsigned num_channels(const struct spng_ihdr *ihdr)
{
    switch(ihdr->color_type)
    {
        case SPNG_COLOR_TYPE_TRUECOLOR: return 3;
        case SPNG_COLOR_TYPE_GRAYSCALE_ALPHA: return 2;
        case SPNG_COLOR_TYPE_TRUECOLOR_ALPHA: return 4;
        case SPNG_COLOR_TYPE_GRAYSCALE:
        case SPNG_COLOR_TYPE_INDEXED:
            return 1;
        default: return 0;
    }
}

/* Calculate scanline width in bits, round up to the nearest byte */
static int calculate_scanline_width(const struct spng_ihdr *ihdr, uint32_t width, size_t *scanline_width)
{
    if(ihdr == NULL || !width) return SPNG_EINTERNAL;

    size_t res = num_channels(ihdr) * ihdr->bit_depth;

    if(res > SIZE_MAX / width) return SPNG_EOVERFLOW;
    res = res * width;

    res += 15; /* Filter byte + 7 for rounding */

    if(res < 15) return SPNG_EOVERFLOW;

    res /= 8;

    if(res > UINT32_MAX) return SPNG_EOVERFLOW;

    *scanline_width = res;

    return 0;
}

static int calculate_subimages(struct spng_ctx *ctx)
{
    if(ctx == NULL) return SPNG_EINTERNAL;

    struct spng_ihdr *ihdr = &ctx->ihdr;
    struct spng_subimage *sub = ctx->subimage;

    if(ihdr->interlace_method == 1)
    {
        sub[0].width = (ihdr->width + 7) >> 3;
        sub[0].height = (ihdr->height + 7) >> 3;
        sub[1].width = (ihdr->width + 3) >> 3;
        sub[1].height = (ihdr->height + 7) >> 3;
        sub[2].width = (ihdr->width + 3) >> 2;
        sub[2].height = (ihdr->height + 3) >> 3;
        sub[3].width = (ihdr->width + 1) >> 2;
        sub[3].height = (ihdr->height + 3) >> 2;
        sub[4].width = (ihdr->width + 1) >> 1;
        sub[4].height = (ihdr->height + 1) >> 2;
        sub[5].width = ihdr->width >> 1;
        sub[5].height = (ihdr->height + 1) >> 1;
        sub[6].width = ihdr->width;
        sub[6].height = ihdr->height >> 1;
    }
    else
    {
        sub[0].width = ihdr->width;
        sub[0].height = ihdr->height;
    }

    int i;
    for(i=0; i < 7; i++)
    {
        if(sub[i].width == 0 || sub[i].height == 0) continue;

        int ret = calculate_scanline_width(ihdr, sub[i].width, &sub[i].scanline_width);
        if(ret) return ret;

        if(sub[ctx->widest_pass].scanline_width < sub[i].scanline_width) ctx->widest_pass = i;

        ctx->last_pass = i;
    }

    return 0;
}

static int check_decode_fmt(const struct spng_ihdr *ihdr, const int fmt)
{
    switch(fmt)
    {
        case SPNG_FMT_RGBA8:
        case SPNG_FMT_RGBA16:
        case SPNG_FMT_RGB8:
        case SPNG_FMT_PNG:
        case SPNG_FMT_RAW:
            return 0;
        case SPNG_FMT_G8:
        case SPNG_FMT_GA8:
            if(ihdr->color_type == SPNG_COLOR_TYPE_GRAYSCALE && ihdr->bit_depth <= 8) return 0;
            else return SPNG_EFMT;
        case SPNG_FMT_GA16:
            if(ihdr->color_type == SPNG_COLOR_TYPE_GRAYSCALE && ihdr->bit_depth == 16) return 0;
            else return SPNG_EFMT;
        default: return SPNG_EFMT;
    }
}

static int calculate_image_width(const struct spng_ihdr *ihdr, int fmt, size_t *len)
{
    if(ihdr == NULL || len == NULL) return SPNG_EINTERNAL;

    size_t res = ihdr->width;
    unsigned bytes_per_pixel;

    switch(fmt)
    {
        case SPNG_FMT_RGBA8:
        case SPNG_FMT_GA16:
            bytes_per_pixel = 4;
            break;
        case SPNG_FMT_RGBA16:
            bytes_per_pixel = 8;
            break;
        case SPNG_FMT_RGB8:
            bytes_per_pixel = 3;
            break;
        case SPNG_FMT_PNG:
        case SPNG_FMT_RAW:
        {
            int ret = calculate_scanline_width(ihdr, ihdr->width, &res);
            if(ret) return ret;

            res -= 1; /* exclude filter byte */
            bytes_per_pixel = 1;
            break;
        }
        case SPNG_FMT_G8:
            bytes_per_pixel = 1;
            break;
        case SPNG_FMT_GA8:
            bytes_per_pixel = 2;
            break;
        default: return SPNG_EINTERNAL;
    }

    if(res > SIZE_MAX / bytes_per_pixel) return SPNG_EOVERFLOW;
    res = res * bytes_per_pixel;

    *len = res;

    return 0;
}

static int calculate_image_size(const struct spng_ihdr *ihdr, int fmt, size_t *len)
{
    if(ihdr == NULL || len == NULL) return SPNG_EINTERNAL;

    size_t res = 0;

    int ret = calculate_image_width(ihdr, fmt, &res);
    if(ret) return ret;

    if(res > SIZE_MAX / ihdr->height) return SPNG_EOVERFLOW;
    res = res * ihdr->height;

    *len = res;

    return 0;
}

static int increase_cache_usage(spng_ctx *ctx, size_t bytes, int new_chunk)
{
    if(ctx == NULL || !bytes) return SPNG_EINTERNAL;

    if(new_chunk)
    {
        ctx->chunk_count_total++;
        if(ctx->chunk_count_total < 1) return SPNG_EOVERFLOW;

        if(ctx->chunk_count_total > ctx->chunk_count_limit) return SPNG_ECHUNK_LIMITS;
    }

    size_t new_usage = ctx->chunk_cache_usage + bytes;

    if(new_usage < ctx->chunk_cache_usage) return SPNG_EOVERFLOW;

    if(new_usage > ctx->chunk_cache_limit) return SPNG_ECHUNK_LIMITS;

    ctx->chunk_cache_usage = new_usage;

    return 0;
}

static int decrease_cache_usage(spng_ctx *ctx, size_t usage)
{
    if(ctx == NULL || !usage) return SPNG_EINTERNAL;
    if(usage > ctx->chunk_cache_usage) return SPNG_EINTERNAL;

    ctx->chunk_cache_usage -= usage;

    return 0;
}

static int is_critical_chunk(struct spng_chunk *chunk)
{
    if(chunk == NULL) return 0;
    if((chunk->type[0] & (1 << 5)) == 0) return 1;

    return 0;
}

static int decode_err(spng_ctx *ctx, int err)
{
    ctx->state = SPNG_STATE_INVALID;

    return err;
}

static int encode_err(spng_ctx *ctx, int err)
{
    ctx->state = SPNG_STATE_INVALID;

    return err;
}

static inline int read_data(spng_ctx *ctx, size_t bytes)
{
    if(ctx == NULL) return SPNG_EINTERNAL;
    if(!bytes) return 0;

    if(ctx->streaming && (bytes > SPNG_READ_SIZE)) return SPNG_EINTERNAL;

    int ret = ctx->read_fn(ctx, ctx->stream_user_ptr, ctx->stream_buf, bytes);

    if(ret)
    {
        if(ret > 0 || ret < SPNG_IO_ERROR) ret = SPNG_IO_ERROR;

        return ret;
    }

    ctx->bytes_read += bytes;
    if(ctx->bytes_read < bytes) return SPNG_EOVERFLOW;

    return 0;
}

/* Ensure there is enough space for encoding starting at ctx->write_ptr  */
static int require_bytes(spng_ctx *ctx, size_t bytes)
{
    if(ctx == NULL) return SPNG_EINTERNAL;

    if(ctx->streaming)
    {
        if(bytes > ctx->stream_buf_size)
        {
            size_t new_size = ctx->stream_buf_size;

            /* Start at default IDAT size + header + crc */
            if(new_size < (SPNG_WRITE_SIZE + 12)) new_size = SPNG_WRITE_SIZE + 12;

            if(new_size < bytes) new_size = bytes;

            void *temp = spng__realloc(ctx, ctx->stream_buf, new_size);

            if(temp == NULL) return encode_err(ctx, SPNG_EMEM);

            ctx->stream_buf = temp;
            ctx->stream_buf_size = bytes;
            ctx->write_ptr = ctx->stream_buf;
        }

        return 0;
    }

    if(!ctx->internal_buffer) return SPNG_ENODST;

    size_t required = ctx->bytes_encoded + bytes;
    if(required < bytes) return SPNG_EOVERFLOW;

    if(required > ctx->out_png_size)
    {
        size_t new_size = ctx->out_png_size;

        /* Start with a size that doesn't require a realloc() 100% of the time */
        if(new_size < (SPNG_WRITE_SIZE * 2)) new_size = SPNG_WRITE_SIZE * 2;

        /* Prefer the next power of two over the requested size */
        while(new_size < required)
        {
            if(new_size / SIZE_MAX > 2) return encode_err(ctx, SPNG_EOVERFLOW);

            new_size *= 2;
        }

        void *temp = spng__realloc(ctx, ctx->out_png, new_size);

        if(temp == NULL) return encode_err(ctx, SPNG_EMEM);

        ctx->out_png = temp;
        ctx->out_png_size = new_size;
        ctx->write_ptr = ctx->out_png + ctx->bytes_encoded;
    }

    return 0;
}

static int write_data(spng_ctx *ctx, const void *data, size_t bytes)
{
    if(ctx == NULL) return SPNG_EINTERNAL;
    if(!bytes) return 0;

    if(ctx->streaming)
    {
        if(bytes > SPNG_WRITE_SIZE) return SPNG_EINTERNAL;

        int ret = ctx->write_fn(ctx, ctx->stream_user_ptr, (void*)data, bytes);

        if(ret)
        {
            if(ret > 0 || ret < SPNG_IO_ERROR) ret = SPNG_IO_ERROR;

            return encode_err(ctx, ret);
        }
    }
    else
    {
        int ret = require_bytes(ctx, bytes);
        if(ret) return encode_err(ctx, ret);

        memcpy(ctx->write_ptr, data, bytes);

        ctx->write_ptr += bytes;
    }

    ctx->bytes_encoded += bytes;
    if(ctx->bytes_encoded < bytes) return SPNG_EOVERFLOW;

    return 0;
}

static int write_header(spng_ctx *ctx, const uint8_t chunk_type[4], size_t chunk_length, unsigned char **data)
{
    if(ctx == NULL || chunk_type == NULL) return SPNG_EINTERNAL;
    if(chunk_length > spng_u32max) return SPNG_EINTERNAL;

    size_t total = chunk_length + 12;

    int ret = require_bytes(ctx, total);
    if(ret) return ret;

    uint32_t crc = crc32(0, NULL, 0);
    ctx->current_chunk.crc = crc32(crc, chunk_type, 4);

    memcpy(&ctx->current_chunk.type, chunk_type, 4);
    ctx->current_chunk.length = (uint32_t)chunk_length;

    if(!data) return SPNG_EINTERNAL;

    if(ctx->streaming) *data = ctx->stream_buf + 8;
    else *data = ctx->write_ptr + 8;

    return 0;
}

static int trim_chunk(spng_ctx *ctx, uint32_t length)
{
    if(length > spng_u32max) return SPNG_EINTERNAL;
    if(length > ctx->current_chunk.length) return SPNG_EINTERNAL;

    ctx->current_chunk.length = length;

    return 0;
}

static int finish_chunk(spng_ctx *ctx)
{
    if(ctx == NULL) return SPNG_EINTERNAL;

    struct spng_chunk *chunk = &ctx->current_chunk;

    unsigned char *header;
    unsigned char *chunk_data;

    if(ctx->streaming)
    {
        chunk_data = ctx->stream_buf + 8;
        header = ctx->stream_buf;
    }
    else
    {
        chunk_data = ctx->write_ptr + 8;
        header = ctx->write_ptr;
    }

    write_u32(header, chunk->length);
    memcpy(header + 4, chunk->type, 4);

    chunk->crc = crc32(chunk->crc, chunk_data, chunk->length);

    write_u32(chunk_data + chunk->length, chunk->crc);

    if(ctx->streaming)
    {
        const unsigned char *ptr = ctx->stream_buf;
        uint32_t bytes_left = chunk->length + 12;
        uint32_t len = 0;

        while(bytes_left)
        {
            ptr += len;
            len = SPNG_WRITE_SIZE;

            if(len > bytes_left) len = bytes_left;

            int ret = write_data(ctx, ptr, len);
            if(ret) return ret;

            bytes_left -= len;
        }
    }
    else
    {
        ctx->bytes_encoded += chunk->length;
        if(ctx->bytes_encoded < chunk->length) return SPNG_EOVERFLOW;

        ctx->bytes_encoded += 12;
        if(ctx->bytes_encoded < 12) return SPNG_EOVERFLOW;

        ctx->write_ptr += chunk->length + 12;
    }

    return 0;
}

static int write_chunk(spng_ctx *ctx, const uint8_t type[4], const void *data, size_t length)
{
    if(ctx == NULL || type == NULL) return SPNG_EINTERNAL;
    if(length && data == NULL) return SPNG_EINTERNAL;

    unsigned char *write_ptr;

    int ret = write_header(ctx, type, length, &write_ptr);
    if(ret) return ret;

    if(length) memcpy(write_ptr, data, length);

    return finish_chunk(ctx);
}

static int write_iend(spng_ctx *ctx)
{
    unsigned char iend_chunk[12] = { 0, 0, 0, 0, 73, 69, 78, 68, 174, 66, 96, 130 };
    return write_data(ctx, iend_chunk, 12);
}

static int write_unknown_chunks(spng_ctx *ctx, enum spng_location location)
{
    if(!ctx->stored.unknown) return 0;

    const struct spng_unknown_chunk *chunk = ctx->chunk_list;

    uint32_t i;
    for(i=0; i < ctx->n_chunks; i++, chunk++)
    {
        if(chunk->location != location) continue;

        int ret = write_chunk(ctx, chunk->type, chunk->data, chunk->length);
        if(ret) return ret;
    }

    return 0;
}

/* Read and check the current chunk's crc,
   returns -SPNG_CRC_DISCARD if the chunk should be discarded */
static inline int read_and_check_crc(spng_ctx *ctx)
{
    if(ctx == NULL) return SPNG_EINTERNAL;

    int ret;
    ret = read_data(ctx, 4);
    if(ret) return ret;

    ctx->current_chunk.crc = read_u32(ctx->data);

    if(ctx->skip_crc) return 0;

    if(ctx->cur_actual_crc != ctx->current_chunk.crc)
    {
        if(is_critical_chunk(&ctx->current_chunk))
        {
            if(ctx->crc_action_critical == SPNG_CRC_USE) return 0;
        }
        else
        {
            if(ctx->crc_action_ancillary == SPNG_CRC_USE) return 0;
            if(ctx->crc_action_ancillary == SPNG_CRC_DISCARD) return -SPNG_CRC_DISCARD;
        }

        return SPNG_ECHUNK_CRC;
    }

    return 0;
}

/* Read and validate the current chunk's crc and the next chunk header */
static inline int read_header(spng_ctx *ctx)
{
    if(ctx == NULL) return SPNG_EINTERNAL;

    int ret;
    struct spng_chunk chunk = { 0 };

    ret = read_and_check_crc(ctx);
    if(ret)
    {
        if(ret == -SPNG_CRC_DISCARD)
        {
            ctx->discard = 1;
        }
        else return ret;
    }

    ret = read_data(ctx, 8);
    if(ret) return ret;

    chunk.offset = ctx->bytes_read - 8;

    chunk.length = read_u32(ctx->data);

    memcpy(&chunk.type, ctx->data + 4, 4);

    if(chunk.length > spng_u32max) return SPNG_ECHUNK_STDLEN;

    ctx->cur_chunk_bytes_left = chunk.length;

    if(is_critical_chunk(&chunk) && ctx->crc_action_critical == SPNG_CRC_USE) ctx->skip_crc = 1;
    else if(ctx->crc_action_ancillary == SPNG_CRC_USE) ctx->skip_crc = 1;
    else ctx->skip_crc = 0;

    if(!ctx->skip_crc)
    {
        ctx->cur_actual_crc = crc32(0, NULL, 0);
        ctx->cur_actual_crc = crc32(ctx->cur_actual_crc, chunk.type, 4);
    }

    ctx->current_chunk = chunk;

    return 0;
}

/* Read chunk bytes and update crc */
static int read_chunk_bytes(spng_ctx *ctx, uint32_t bytes)
{
    if(ctx == NULL) return SPNG_EINTERNAL;
    if(!ctx->cur_chunk_bytes_left || !bytes) return SPNG_EINTERNAL;
    if(bytes > ctx->cur_chunk_bytes_left) return SPNG_EINTERNAL; /* XXX: more specific error? */

    int ret;

    ret = read_data(ctx, bytes);
    if(ret) return ret;

    if(!ctx->skip_crc) ctx->cur_actual_crc = crc32(ctx->cur_actual_crc, ctx->data, bytes);

    ctx->cur_chunk_bytes_left -= bytes;

    return ret;
}

/* read_chunk_bytes() + read_data() with custom output buffer */
static int read_chunk_bytes2(spng_ctx *ctx, void *out, uint32_t bytes)
{
    if(ctx == NULL) return SPNG_EINTERNAL;
    if(!ctx->cur_chunk_bytes_left || !bytes) return SPNG_EINTERNAL;
    if(bytes > ctx->cur_chunk_bytes_left) return SPNG_EINTERNAL; /* XXX: more specific error? */

    int ret;
    uint32_t len = bytes;

    if(ctx->streaming && len > SPNG_READ_SIZE) len = SPNG_READ_SIZE;

    while(bytes)
    {
        if(len > bytes) len = bytes;

        ret = ctx->read_fn(ctx, ctx->stream_user_ptr, out, len);
        if(ret) return ret;

        if(!ctx->streaming) memcpy(out, ctx->data, len);

        ctx->bytes_read += len;
        if(ctx->bytes_read < len) return SPNG_EOVERFLOW;

        if(!ctx->skip_crc) ctx->cur_actual_crc = crc32(ctx->cur_actual_crc, out, len);

        ctx->cur_chunk_bytes_left -= len;

        out = (char*)out + len;
        bytes -= len;
        len = SPNG_READ_SIZE;
    }

    return 0;
}

static int discard_chunk_bytes(spng_ctx *ctx, uint32_t bytes)
{
    if(ctx == NULL) return SPNG_EINTERNAL;
    if(!bytes) return 0;

    int ret;

    if(ctx->streaming) /* Do small, consecutive reads */
    {
        while(bytes)
        {
            uint32_t len = SPNG_READ_SIZE;

            if(len > bytes) len = bytes;

            ret = read_chunk_bytes(ctx, len);
            if(ret) return ret;

            bytes -= len;
        }
    }
    else
    {
        ret = read_chunk_bytes(ctx, bytes);
        if(ret) return ret;
    }

    return 0;
}

static int spng__inflate_init(spng_ctx *ctx, int window_bits)
{
    if(ctx->zstream.state) inflateEnd(&ctx->zstream);

    ctx->inflate = 1;

    ctx->zstream.zalloc = spng__zalloc;
    ctx->zstream.zfree = spng__zfree;
    ctx->zstream.opaque = ctx;

    if(inflateInit2(&ctx->zstream, window_bits) != Z_OK) return SPNG_EZLIB_INIT;

#if ZLIB_VERNUM >= 0x1290 && !defined(SPNG_USE_MINIZ)

    int validate = 1;

    if(ctx->flags & SPNG_CTX_IGNORE_ADLER32) validate = 0;

    if(is_critical_chunk(&ctx->current_chunk))
    {
        if(ctx->crc_action_critical == SPNG_CRC_USE) validate = 0;
    }
    else /* ancillary */
    {
        if(ctx->crc_action_ancillary == SPNG_CRC_USE) validate = 0;
    }

    if(inflateValidate(&ctx->zstream, validate)) return SPNG_EZLIB_INIT;

#else /* This requires zlib >= 1.2.11 */
    #pragma message ("inflateValidate() not available, SPNG_CTX_IGNORE_ADLER32 will be ignored")
#endif

    return 0;
}

static int spng__deflate_init(spng_ctx *ctx, struct spng__zlib_options *options)
{
    if(ctx->zstream.state) deflateEnd(&ctx->zstream);

    ctx->deflate = 1;

    z_stream *zstream = &ctx->zstream;
    zstream->zalloc = spng__zalloc;
    zstream->zfree = spng__zfree;
    zstream->opaque = ctx;
    zstream->data_type = options->data_type;

    int ret = deflateInit2(zstream, options->compression_level, Z_DEFLATED, options->window_bits, options->mem_level, options->strategy);

    if(ret != Z_OK) return SPNG_EZLIB_INIT;

    return 0;
}

/* Inflate a zlib stream starting with start_buf if non-NULL,
   continuing from the datastream till an end marker,
   allocating and writing the inflated stream to *out,
   leaving "extra" bytes at the end, final buffer length is *len.

   Takes into account the chunk size and cache limits.
*/
static int spng__inflate_stream(spng_ctx *ctx, char **out, size_t *len, size_t extra, const void *start_buf, size_t start_len)
{
    int ret = spng__inflate_init(ctx, 15);
    if(ret) return ret;

    size_t max = ctx->chunk_cache_limit - ctx->chunk_cache_usage;

    if(ctx->max_chunk_size < max) max = ctx->max_chunk_size;

    if(extra > max) return SPNG_ECHUNK_LIMITS;
    max -= extra;

    uint32_t read_size;
    size_t size = 8 * 1024;
    void *t, *buf = spng__malloc(ctx, size);

    if(buf == NULL) return SPNG_EMEM;

    z_stream *stream = &ctx->zstream;

    if(start_buf != NULL && start_len)
    {
        stream->avail_in = (uInt)start_len;
        stream->next_in = start_buf;
    }
    else
    {
        stream->avail_in = 0;
        stream->next_in = NULL;
    }

    stream->avail_out = (uInt)size;
    stream->next_out = buf;

    while(ret != Z_STREAM_END)
    {
        ret = inflate(stream, Z_NO_FLUSH);

        if(ret == Z_STREAM_END) break;

        if(ret != Z_OK && ret != Z_BUF_ERROR)
        {
            ret = SPNG_EZLIB;
            goto err;
        }

        if(!stream->avail_out) /* Resize buffer */
        {
            /* overflow or reached chunk/cache limit */
            if( (2 > SIZE_MAX / size) || (size > max / 2) )
            {
                ret = SPNG_ECHUNK_LIMITS;
                goto err;
            }

            size *= 2;

            t = spng__realloc(ctx, buf, size);
            if(t == NULL) goto mem;

            buf = t;

            stream->avail_out = (uInt)size / 2;
            stream->next_out = (unsigned char*)buf + size / 2;
        }
        else if(!stream->avail_in) /* Read more chunk bytes */
        {
            read_size = ctx->cur_chunk_bytes_left;
            if(ctx->streaming && read_size > SPNG_READ_SIZE) read_size = SPNG_READ_SIZE;

            ret = read_chunk_bytes(ctx, read_size);

            if(ret)
            {
                if(!read_size) ret = SPNG_EZLIB;

                goto err;
            }

            stream->avail_in = read_size;
            stream->next_in = ctx->data;
        }
    }

    size = stream->total_out;

    if(!size)
    {
        ret = SPNG_EZLIB;
        goto err;
    }

    size += extra;
    if(size < extra) goto mem;

    t = spng__realloc(ctx, buf, size);
    if(t == NULL) goto mem;

    buf = t;

    (void)increase_cache_usage(ctx, size, 0);

    *out = buf;
    *len = size;

    return 0;

mem:
    ret = SPNG_EMEM;
err:
    spng__free(ctx, buf);
    return ret;
}

/* Read at least one byte from the IDAT stream */
static int read_idat_bytes(spng_ctx *ctx, uint32_t *bytes_read)
{
    if(ctx == NULL || bytes_read == NULL) return SPNG_EINTERNAL;
    if(memcmp(ctx->current_chunk.type, type_idat, 4)) return SPNG_EIDAT_TOO_SHORT;

    int ret;
    uint32_t len;

    while(!ctx->cur_chunk_bytes_left)
    {
        ret = read_header(ctx);
        if(ret) return ret;

        if(memcmp(ctx->current_chunk.type, type_idat, 4)) return SPNG_EIDAT_TOO_SHORT;
    }

    if(ctx->streaming)
    {/* TODO: estimate bytes to read for progressive reads */
        len = SPNG_READ_SIZE;
        if(len > ctx->cur_chunk_bytes_left) len = ctx->cur_chunk_bytes_left;
    }
    else len = ctx->current_chunk.length;

    ret = read_chunk_bytes(ctx, len);

    *bytes_read = len;

    return ret;
}

static int read_scanline_bytes(spng_ctx *ctx, unsigned char *dest, size_t len)
{
    if(ctx == NULL || dest == NULL) return SPNG_EINTERNAL;

    int ret = Z_OK;
    uint32_t bytes_read;

    z_stream *zstream = &ctx->zstream;

    zstream->avail_out = (uInt)len;
    zstream->next_out = dest;

    while(zstream->avail_out != 0)
    {
        ret = inflate(zstream, Z_NO_FLUSH);

        if(ret == Z_OK) continue;

        if(ret == Z_STREAM_END) /* Reached an end-marker */
        {
            if(zstream->avail_out != 0) return SPNG_EIDAT_TOO_SHORT;
        }
        else if(ret == Z_BUF_ERROR) /* Read more IDAT bytes */
        {
            ret = read_idat_bytes(ctx, &bytes_read);
            if(ret) return ret;

            zstream->avail_in = bytes_read;
            zstream->next_in = ctx->data;
        }
        else return SPNG_EIDAT_STREAM;
    }

    return 0;
}

static uint8_t paeth(uint8_t a, uint8_t b, uint8_t c)
{
    int16_t p = a + b - c;
    int16_t pa = abs(p - a);
    int16_t pb = abs(p - b);
    int16_t pc = abs(p - c);

    if(pa <= pb && pa <= pc) return a;
    else if(pb <= pc) return b;

    return c;
}

SPNG_TARGET_CLONES("default,avx2")
static void defilter_up(size_t bytes, unsigned char *row, const unsigned char *prev)
{
    size_t i;
    for(i=0; i < bytes; i++)
    {
        row[i] += prev[i];
    }
}

/* Defilter *scanline in-place.
   *prev_scanline and *scanline should point to the first pixel,
   scanline_width is the width of the scanline including the filter byte.
*/
static int defilter_scanline(const unsigned char *prev_scanline, unsigned char *scanline,
                             size_t scanline_width, unsigned bytes_per_pixel, unsigned filter)
{
    if(prev_scanline == NULL || scanline == NULL || !scanline_width) return SPNG_EINTERNAL;

    size_t i;
    scanline_width--;

    if(filter == 0) return 0;

#ifndef SPNG_DISABLE_OPT
    if(filter == SPNG_FILTER_UP) goto no_opt;

    if(bytes_per_pixel == 4)
    {
        if(filter == SPNG_FILTER_SUB)
            defilter_sub4(scanline_width, scanline);
        else if(filter == SPNG_FILTER_AVERAGE)
            defilter_avg4(scanline_width, scanline, prev_scanline);
        else if(filter == SPNG_FILTER_PAETH)
            defilter_paeth4(scanline_width, scanline, prev_scanline);
        else return SPNG_EFILTER;

        return 0;
    }
    else if(bytes_per_pixel == 3)
    {
        if(filter == SPNG_FILTER_SUB)
            defilter_sub3(scanline_width, scanline);
        else if(filter == SPNG_FILTER_AVERAGE)
            defilter_avg3(scanline_width, scanline, prev_scanline);
        else if(filter == SPNG_FILTER_PAETH)
            defilter_paeth3(scanline_width, scanline, prev_scanline);
        else return SPNG_EFILTER;

        return 0;
    }
no_opt:
#endif

    if(filter == SPNG_FILTER_UP)
    {
        defilter_up(scanline_width, scanline, prev_scanline);
        return 0;
    }

    for(i=0; i < scanline_width; i++)
    {
        uint8_t x, a, b, c;

        if(i >= bytes_per_pixel)
        {
            a = scanline[i - bytes_per_pixel];
            b = prev_scanline[i];
            c = prev_scanline[i - bytes_per_pixel];
        }
        else /* First pixel in row */
        {
            a = 0;
            b = prev_scanline[i];
            c = 0;
        }

        x = scanline[i];

        switch(filter)
        {
            case SPNG_FILTER_SUB:
            {
                x = x + a;
                break;
            }
            case SPNG_FILTER_AVERAGE:
            {
                uint16_t avg = (a + b) / 2;
                x = x + avg;
                break;
            }
            case SPNG_FILTER_PAETH:
            {
                x = x + paeth(a,b,c);
                break;
            }
        }

        scanline[i] = x;
    }

    return 0;
}

static int filter_scanline(unsigned char *filtered, const unsigned char *prev_scanline, const unsigned char *scanline,
                           size_t scanline_width, unsigned bytes_per_pixel, const unsigned filter)
{
    if(prev_scanline == NULL || scanline == NULL || scanline_width <= 1) return SPNG_EINTERNAL;

    if(filter > 4) return SPNG_EFILTER;
    if(filter == 0) return 0;

    scanline_width--;

    uint32_t i;
    for(i=0; i < scanline_width; i++)
    {
        uint8_t x, a, b, c;

        if(i >= bytes_per_pixel)
        {
            a = scanline[i - bytes_per_pixel];
            b = prev_scanline[i];
            c = prev_scanline[i - bytes_per_pixel];
        }
        else /* first pixel in row */
        {
            a = 0;
            b = prev_scanline[i];
            c = 0;
        }

        x = scanline[i];

        switch(filter)
        {
            case SPNG_FILTER_SUB:
            {
                x = x - a;
                break;
            }
            case SPNG_FILTER_UP:
            {
                x = x - b;
                break;
            }
            case SPNG_FILTER_AVERAGE:
            {
                uint16_t avg = (a + b) / 2;
                x = x - avg;
                break;
            }
            case SPNG_FILTER_PAETH:
            {
                x = x - paeth(a,b,c);
                break;
            }
        }

        filtered[i] = x;
    }

    return 0;
}

static int32_t filter_sum(const unsigned char *prev_scanline, const unsigned char *scanline,
                          size_t size, unsigned bytes_per_pixel, const unsigned filter)
{
    /* prevent potential over/underflow, bails out at a width of ~8M pixels for RGBA8 */
    if(size > (INT32_MAX / 128)) return INT32_MAX;

    uint32_t i;
    int32_t sum = 0;
    uint8_t x, a, b, c;

    for(i=0; i < size; i++)
    {
        if(i >= bytes_per_pixel)
        {
            a = scanline[i - bytes_per_pixel];
            b = prev_scanline[i];
            c = prev_scanline[i - bytes_per_pixel];
        }
        else /* first pixel in row */
        {
            a = 0;
            b = prev_scanline[i];
            c = 0;
        }

        x = scanline[i];

        switch(filter)
        {
            case SPNG_FILTER_NONE:
            {
                break;
            }
            case SPNG_FILTER_SUB:
            {
                x = x - a;
                break;
            }
            case SPNG_FILTER_UP:
            {
                x = x - b;
                break;
            }
            case SPNG_FILTER_AVERAGE:
            {
                uint16_t avg = (a + b) / 2;
                x = x - avg;
                break;
            }
            case SPNG_FILTER_PAETH:
            {
                x = x - paeth(a,b,c);
                break;
            }
        }

        sum += 128 - abs((int)x - 128);
    }

    return sum;
}

static unsigned get_best_filter(const unsigned char *prev_scanline, const unsigned char *scanline,
                                size_t scanline_width, unsigned bytes_per_pixel, const int choices)
{
    if(!choices) return SPNG_FILTER_NONE;

    scanline_width--;

    int i;
    unsigned int best_filter = 0;
    enum spng_filter_choice flag;
    int32_t sum, best_score = INT32_MAX;
    int32_t filter_scores[5] = { INT32_MAX, INT32_MAX, INT32_MAX, INT32_MAX, INT32_MAX };

    if( !(choices & (choices - 1)) )
    {/* only one choice/bit is set */
        for(i=0; i < 5; i++)
        {
            if(choices == 1 << (i + 3)) return i;
        }
    }

    for(i=0; i < 5; i++)
    {
        flag = 1 << (i + 3);

        if(choices & flag) sum = filter_sum(prev_scanline, scanline, scanline_width, bytes_per_pixel, i);
        else continue;

        filter_scores[i] = abs(sum);

        if(filter_scores[i] < best_score)
        {
            best_score = filter_scores[i];
            best_filter = i;
        }
    }

    return best_filter;
}

/* Scale "sbits" significant bits in "sample" from "bit_depth" to "target"

   "bit_depth" must be a valid PNG depth
   "sbits" must be less than or equal to "bit_depth"
   "target" must be between 1 and 16
*/
static uint16_t sample_to_target(uint16_t sample, unsigned bit_depth, unsigned sbits, unsigned target)
{
    if(bit_depth == sbits)
    {
        if(target == sbits) return sample; /* No scaling */
    }/* bit_depth > sbits */
    else sample = sample >> (bit_depth - sbits); /* Shift significant bits to bottom */

    /* Downscale */
    if(target < sbits) return sample >> (sbits - target);

    /* Upscale using left bit replication */
    int8_t shift_amount = target - sbits;
    uint16_t sample_bits = sample;
    sample = 0;

    while(shift_amount >= 0)
    {
        sample = sample | (sample_bits << shift_amount);
        shift_amount -= sbits;
    }

    int8_t partial = shift_amount + (int8_t)sbits;

    if(partial != 0) sample = sample | (sample_bits >> abs(shift_amount));

    return sample;
}

static inline void gamma_correct_row(unsigned char *row, uint32_t pixels, int fmt, const uint16_t *gamma_lut)
{
    uint32_t i;

    if(fmt == SPNG_FMT_RGBA8)
    {
        unsigned char *px;
        for(i=0; i < pixels; i++)
        {
            px = row + i * 4;

            px[0] = gamma_lut[px[0]];
            px[1] = gamma_lut[px[1]];
            px[2] = gamma_lut[px[2]];
        }
    }
    else if(fmt == SPNG_FMT_RGBA16)
    {
        for(i=0; i < pixels; i++)
        {
            uint16_t px[4];
            memcpy(px, row + i * 8, 8);

            px[0] = gamma_lut[px[0]];
            px[1] = gamma_lut[px[1]];
            px[2] = gamma_lut[px[2]];

            memcpy(row + i * 8, px, 8);
        }
    }
    else if(fmt == SPNG_FMT_RGB8)
    {
        unsigned char *px;
        for(i=0; i < pixels; i++)
        {
            px = row + i * 3;

            px[0] = gamma_lut[px[0]];
            px[1] = gamma_lut[px[1]];
            px[2] = gamma_lut[px[2]];
        }
    }
}

/* Apply transparency to output row */
static inline void trns_row(unsigned char *row,
                            const unsigned char *scanline,
                            const unsigned char *trns,
                            unsigned scanline_stride,
                            struct spng_ihdr *ihdr,
                            uint32_t pixels,
                            int fmt)
{
    uint32_t i;
    unsigned row_stride;
    unsigned depth = ihdr->bit_depth;

    if(fmt == SPNG_FMT_RGBA8)
    {
        if(ihdr->color_type == SPNG_COLOR_TYPE_GRAYSCALE) return; /* already applied in the decoding loop */

        row_stride = 4;
        for(i=0; i < pixels; i++, scanline+=scanline_stride, row+=row_stride)
        {
            if(!memcmp(scanline, trns, scanline_stride)) row[3] = 0;
        }
    }
    else if(fmt == SPNG_FMT_RGBA16)
    {
        if(ihdr->color_type == SPNG_COLOR_TYPE_GRAYSCALE) return; /* already applied in the decoding loop */

        row_stride = 8;
        for(i=0; i < pixels; i++, scanline+=scanline_stride, row+=row_stride)
        {
            if(!memcmp(scanline, trns, scanline_stride)) memset(row + 6, 0, 2);
        }
    }
    else if(fmt == SPNG_FMT_GA8)
    {
        row_stride = 2;

        if(depth == 16)
        {
            for(i=0; i < pixels; i++, scanline+=scanline_stride, row+=row_stride)
            {
                if(!memcmp(scanline, trns, scanline_stride)) memset(row + 1, 0, 1);
            }
        }
        else /* depth <= 8 */
        {
            struct spng__iter iter = spng__iter_init(depth, scanline);

            for(i=0; i < pixels; i++, row+=row_stride)
            {
                if(trns[0] == get_sample(&iter)) row[1] = 0;
            }
        }
    }
    else if(fmt == SPNG_FMT_GA16)
    {
        row_stride = 4;

        if(depth == 16)
        {
            for(i=0; i< pixels; i++, scanline+=scanline_stride, row+=row_stride)
            {
                if(!memcmp(scanline, trns, 2)) memset(row + 2, 0, 2);
            }
        }
        else
        {
            struct spng__iter iter = spng__iter_init(depth, scanline);

            for(i=0; i< pixels; i++, row+=row_stride)
            {
                if(trns[0] == get_sample(&iter)) memset(row + 2, 0, 2);
            }
        }
    }
    else return;
}

static inline void scale_row(unsigned char *row, uint32_t pixels, int fmt, unsigned depth, const struct spng_sbit *sbit)
{
    uint32_t i;

    if(fmt == SPNG_FMT_RGBA8)
    {
        unsigned char px[4];
        for(i=0; i < pixels; i++)
        {
            memcpy(px, row + i * 4, 4);

            px[0] = sample_to_target(px[0], depth, sbit->red_bits, 8);
            px[1] = sample_to_target(px[1], depth, sbit->green_bits, 8);
            px[2] = sample_to_target(px[2], depth, sbit->blue_bits, 8);
            px[3] = sample_to_target(px[3], depth, sbit->alpha_bits, 8);

            memcpy(row + i * 4, px, 4);
        }
    }
    else if(fmt == SPNG_FMT_RGBA16)
    {
        uint16_t px[4];
        for(i=0; i < pixels; i++)
        {
            memcpy(px, row + i * 8, 8);

            px[0] = sample_to_target(px[0], depth, sbit->red_bits, 16);
            px[1] = sample_to_target(px[1], depth, sbit->green_bits, 16);
            px[2] = sample_to_target(px[2], depth, sbit->blue_bits, 16);
            px[3] = sample_to_target(px[3], depth, sbit->alpha_bits, 16);

            memcpy(row + i * 8, px, 8);
        }
    }
    else if(fmt == SPNG_FMT_RGB8)
    {
        unsigned char px[4];
        for(i=0; i < pixels; i++)
        {
            memcpy(px, row + i * 3, 3);

            px[0] = sample_to_target(px[0], depth, sbit->red_bits, 8);
            px[1] = sample_to_target(px[1], depth, sbit->green_bits, 8);
            px[2] = sample_to_target(px[2], depth, sbit->blue_bits, 8);

            memcpy(row + i * 3, px, 3);
        }
    }
    else if(fmt == SPNG_FMT_G8)
    {
        for(i=0; i < pixels; i++)
        {
            row[i] = sample_to_target(row[i], depth, sbit->grayscale_bits, 8);
        }
    }
    else if(fmt == SPNG_FMT_GA8)
    {
        for(i=0; i < pixels; i++)
        {
            row[i*2] = sample_to_target(row[i*2], depth, sbit->grayscale_bits, 8);
        }
    }
}

/* Expand to *row using 8-bit palette indices from *scanline */
static void expand_row(unsigned char *row,
                       const unsigned char *scanline,
                       const union spng__decode_plte *decode_plte,
                       uint32_t width,
                       int fmt)
{
    uint32_t i = 0;
    unsigned char *px;
    unsigned char entry;
    const struct spng_plte_entry *plte = decode_plte->rgba;

#if defined(SPNG_ARM)
    if(fmt == SPNG_FMT_RGBA8) i = expand_palette_rgba8_neon(row, scanline, decode_plte->raw, width);
    else if(fmt == SPNG_FMT_RGB8)
    {
        i = expand_palette_rgb8_neon(row, scanline, decode_plte->raw, width);

        for(; i < width; i++)
        {/* In this case the LUT is 3 bytes packed */
            px = row + i * 3;
            entry = scanline[i];
            px[0] = decode_plte->raw[entry * 3 + 0];
            px[1] = decode_plte->raw[entry * 3 + 1];
            px[2] = decode_plte->raw[entry * 3 + 2];
        }
        return;
    }
#endif

    if(fmt == SPNG_FMT_RGBA8)
    {
        for(; i < width; i++)
        {
            px = row + i * 4;
            entry = scanline[i];
            px[0] = plte[entry].red;
            px[1] = plte[entry].green;
            px[2] = plte[entry].blue;
            px[3] = plte[entry].alpha;
        }
    }
    else if(fmt == SPNG_FMT_RGB8)
    {
        for(; i < width; i++)
        {
            px = row + i * 3;
            entry = scanline[i];
            px[0] = plte[entry].red;
            px[1] = plte[entry].green;
            px[2] = plte[entry].blue;
        }
    }
}

/* Unpack 1/2/4/8-bit samples to G8/GA8/GA16 or G16 -> GA16 */
static void unpack_scanline(unsigned char *out, const unsigned char *scanline, uint32_t width, unsigned bit_depth, int fmt)
{
    struct spng__iter iter = spng__iter_init(bit_depth, scanline);
    uint32_t i;
    uint16_t sample, alpha = 65535;


    if(fmt == SPNG_FMT_GA8) goto ga8;
    else if(fmt == SPNG_FMT_GA16) goto ga16;

    /* 1/2/4-bit -> 8-bit */
    for(i=0; i < width; i++) out[i] = get_sample(&iter);

    return;

ga8:
    /* 1/2/4/8-bit -> GA8 */
    for(i=0; i < width; i++)
    {
        out[i*2] = get_sample(&iter);
        out[i*2 + 1] = 255;
    }

    return;

ga16:

    /* 16 -> GA16 */
    if(bit_depth == 16)
    {
        for(i=0; i < width; i++)
        {
            memcpy(out + i * 4, scanline + i * 2, 2);
            memcpy(out + i * 4 + 2, &alpha, 2);
        }
        return;
    }

     /* 1/2/4/8-bit -> GA16 */
    for(i=0; i < width; i++)
    {
        sample = get_sample(&iter);
        memcpy(out + i * 4, &sample, 2);
        memcpy(out + i * 4 + 2, &alpha, 2);
    }
}

static int check_ihdr(const struct spng_ihdr *ihdr, uint32_t max_width, uint32_t max_height)
{
    if(ihdr->width > spng_u32max || !ihdr->width) return SPNG_EWIDTH;
    if(ihdr->height > spng_u32max || !ihdr->height) return SPNG_EHEIGHT;

    if(ihdr->width > max_width) return SPNG_EUSER_WIDTH;
    if(ihdr->height > max_height) return SPNG_EUSER_HEIGHT;

    switch(ihdr->color_type)
    {
        case SPNG_COLOR_TYPE_GRAYSCALE:
        {
            if( !(ihdr->bit_depth == 1 || ihdr->bit_depth == 2 ||
                  ihdr->bit_depth == 4 || ihdr->bit_depth == 8 ||
                  ihdr->bit_depth == 16) )
                  return SPNG_EBIT_DEPTH;

            break;
        }
        case SPNG_COLOR_TYPE_TRUECOLOR:
        case SPNG_COLOR_TYPE_GRAYSCALE_ALPHA:
        case SPNG_COLOR_TYPE_TRUECOLOR_ALPHA:
        {
            if( !(ihdr->bit_depth == 8 || ihdr->bit_depth == 16) )
                return SPNG_EBIT_DEPTH;

            break;
        }
        case SPNG_COLOR_TYPE_INDEXED:
        {
            if( !(ihdr->bit_depth == 1 || ihdr->bit_depth == 2 ||
                  ihdr->bit_depth == 4 || ihdr->bit_depth == 8) )
                return SPNG_EBIT_DEPTH;

            break;
        }
        default: return SPNG_ECOLOR_TYPE;
    }

    if(ihdr->compression_method) return SPNG_ECOMPRESSION_METHOD;
    if(ihdr->filter_method) return SPNG_EFILTER_METHOD;

    if(ihdr->interlace_method > 1) return SPNG_EINTERLACE_METHOD;

    return 0;
}

static int check_plte(const struct spng_plte *plte, const struct spng_ihdr *ihdr)
{
    if(plte == NULL || ihdr == NULL) return 1;

    if(plte->n_entries == 0) return 1;
    if(plte->n_entries > 256) return 1;

    if(ihdr->color_type == SPNG_COLOR_TYPE_INDEXED)
    {
        if(plte->n_entries > (1U << ihdr->bit_depth)) return 1;
    }

    return 0;
}

static int check_sbit(const struct spng_sbit *sbit, const struct spng_ihdr *ihdr)
{
    if(sbit == NULL || ihdr == NULL) return 1;

    if(ihdr->color_type == 0)
    {
        if(sbit->grayscale_bits == 0) return SPNG_ESBIT;
        if(sbit->grayscale_bits > ihdr->bit_depth) return SPNG_ESBIT;
    }
    else if(ihdr->color_type == 2 || ihdr->color_type == 3)
    {
        if(sbit->red_bits == 0) return SPNG_ESBIT;
        if(sbit->green_bits == 0) return SPNG_ESBIT;
        if(sbit->blue_bits == 0) return SPNG_ESBIT;

        uint8_t bit_depth;
        if(ihdr->color_type == 3) bit_depth = 8;
        else bit_depth = ihdr->bit_depth;

        if(sbit->red_bits > bit_depth) return SPNG_ESBIT;
        if(sbit->green_bits > bit_depth) return SPNG_ESBIT;
        if(sbit->blue_bits > bit_depth) return SPNG_ESBIT;
    }
    else if(ihdr->color_type == 4)
    {
        if(sbit->grayscale_bits == 0) return SPNG_ESBIT;
        if(sbit->alpha_bits == 0) return SPNG_ESBIT;

        if(sbit->grayscale_bits > ihdr->bit_depth) return SPNG_ESBIT;
        if(sbit->alpha_bits > ihdr->bit_depth) return SPNG_ESBIT;
    }
    else if(ihdr->color_type == 6)
    {
        if(sbit->red_bits == 0) return SPNG_ESBIT;
        if(sbit->green_bits == 0) return SPNG_ESBIT;
        if(sbit->blue_bits == 0) return SPNG_ESBIT;
        if(sbit->alpha_bits == 0) return SPNG_ESBIT;

        if(sbit->red_bits > ihdr->bit_depth) return SPNG_ESBIT;
        if(sbit->green_bits > ihdr->bit_depth) return SPNG_ESBIT;
        if(sbit->blue_bits > ihdr->bit_depth) return SPNG_ESBIT;
        if(sbit->alpha_bits > ihdr->bit_depth) return SPNG_ESBIT;
    }

    return 0;
}

static int check_chrm_int(const struct spng_chrm_int *chrm_int)
{
    if(chrm_int == NULL) return 1;

    if(chrm_int->white_point_x > spng_u32max ||
       chrm_int->white_point_y > spng_u32max ||
       chrm_int->red_x > spng_u32max ||
       chrm_int->red_y > spng_u32max ||
       chrm_int->green_x  > spng_u32max ||
       chrm_int->green_y  > spng_u32max ||
       chrm_int->blue_x > spng_u32max ||
       chrm_int->blue_y > spng_u32max) return SPNG_ECHRM;

    return 0;
}

static int check_phys(const struct spng_phys *phys)
{
    if(phys == NULL) return 1;

    if(phys->unit_specifier > 1) return SPNG_EPHYS;

    if(phys->ppu_x > spng_u32max) return SPNG_EPHYS;
    if(phys->ppu_y > spng_u32max) return SPNG_EPHYS;

    return 0;
}

static int check_time(const struct spng_time *time)
{
    if(time == NULL) return 1;

    if(time->month == 0 || time->month > 12) return 1;
    if(time->day == 0 || time->day > 31) return 1;
    if(time->hour > 23) return 1;
    if(time->minute > 59) return 1;
    if(time->second > 60) return 1;

    return 0;
}

static int check_offs(const struct spng_offs *offs)
{
    if(offs == NULL) return 1;

    if(offs->unit_specifier > 1) return 1;

    return 0;
}

static int check_exif(const struct spng_exif *exif)
{
    if(exif == NULL) return 1;
    if(exif->data == NULL) return 1;

    if(exif->length < 4) return SPNG_ECHUNK_SIZE;
    if(exif->length > spng_u32max) return SPNG_ECHUNK_STDLEN;

    const uint8_t exif_le[4] = { 73, 73, 42, 0 };
    const uint8_t exif_be[4] = { 77, 77, 0, 42 };

    if(memcmp(exif->data, exif_le, 4) && memcmp(exif->data, exif_be, 4)) return 1;

    return 0;
}

/* Validate PNG keyword */
static int check_png_keyword(const char *str)
{
    if(str == NULL) return 1;
    size_t len = strlen(str);
    const char *end = str + len;

    if(!len) return 1;
    if(len > 79) return 1;
    if(str[0] == ' ') return 1; /* Leading space */
    if(end[-1] == ' ') return 1; /* Trailing space */
    if(strstr(str, "  ") != NULL) return 1; /* Consecutive spaces */

    uint8_t c;
    while(str != end)
    {
        memcpy(&c, str, 1);

        if( (c >= 32 && c <= 126) || (c >= 161) ) str++;
        else return 1; /* Invalid character */
    }

    return 0;
}

/* Validate PNG text *str up to 'len' bytes */
static int check_png_text(const char *str, size_t len)
{/* XXX: are consecutive newlines permitted? */
    if(str == NULL || len == 0) return 1;

    uint8_t c;
    size_t i = 0;
    while(i < len)
    {
        memcpy(&c, str + i, 1);

        if( (c >= 32 && c <= 126) || (c >= 161) || c == 10) i++;
        else return 1; /* Invalid character */
    }

    return 0;
}

/* Returns non-zero for standard chunks which are stored without allocating memory */
static int is_small_chunk(uint8_t type[4])
{
    if(!memcmp(type, type_plte, 4)) return 1;
    else if(!memcmp(type, type_chrm, 4)) return 1;
    else if(!memcmp(type, type_gama, 4)) return 1;
    else if(!memcmp(type, type_sbit, 4)) return 1;
    else if(!memcmp(type, type_srgb, 4)) return 1;
    else if(!memcmp(type, type_bkgd, 4)) return 1;
    else if(!memcmp(type, type_trns, 4)) return 1;
    else if(!memcmp(type, type_hist, 4)) return 1;
    else if(!memcmp(type, type_phys, 4)) return 1;
    else if(!memcmp(type, type_time, 4)) return 1;
    else if(!memcmp(type, type_offs, 4)) return 1;
    else return 0;
}

static int read_ihdr(spng_ctx *ctx)
{
    int ret;
    struct spng_chunk *chunk = &ctx->current_chunk;
    const unsigned char *data;

    chunk->offset = 8;
    chunk->length = 13;
    size_t sizeof_sig_ihdr = 29;

    ret = read_data(ctx, sizeof_sig_ihdr);
    if(ret) return ret;

    data = ctx->data;

    if(memcmp(data, spng_signature, sizeof(spng_signature))) return SPNG_ESIGNATURE;

    chunk->length = read_u32(data + 8);
    memcpy(&chunk->type, data + 12, 4);

    if(chunk->length != 13) return SPNG_EIHDR_SIZE;
    if(memcmp(chunk->type, type_ihdr, 4)) return SPNG_ENOIHDR;

    ctx->cur_actual_crc = crc32(0, NULL, 0);
    ctx->cur_actual_crc = crc32(ctx->cur_actual_crc, data + 12, 17);

    ctx->ihdr.width = read_u32(data + 16);
    ctx->ihdr.height = read_u32(data + 20);
    ctx->ihdr.bit_depth = data[24];
    ctx->ihdr.color_type = data[25];
    ctx->ihdr.compression_method = data[26];
    ctx->ihdr.filter_method = data[27];
    ctx->ihdr.interlace_method = data[28];

    ret = check_ihdr(&ctx->ihdr, ctx->max_width, ctx->max_height);
    if(ret) return ret;

    ctx->file.ihdr = 1;
    ctx->stored.ihdr = 1;

    if(ctx->ihdr.bit_depth < 8) ctx->bytes_per_pixel = 1;
    else ctx->bytes_per_pixel = num_channels(&ctx->ihdr) * (ctx->ihdr.bit_depth / 8);

    ret = calculate_subimages(ctx);
    if(ret) return ret;

    return 0;
}

static void splt_undo(spng_ctx *ctx)
{
    struct spng_splt *splt = &ctx->splt_list[ctx->n_splt - 1];

    spng__free(ctx, splt->entries);

    decrease_cache_usage(ctx, sizeof(struct spng_splt));
    decrease_cache_usage(ctx, splt->n_entries * sizeof(struct spng_splt_entry));

    splt->entries = NULL;

    ctx->n_splt--;
}

static void text_undo(spng_ctx *ctx)
{
    struct spng_text2 *text = &ctx->text_list[ctx->n_text - 1];

    spng__free(ctx, text->keyword);
    if(text->compression_flag) spng__free(ctx, text->text);

    decrease_cache_usage(ctx, text->cache_usage);
    decrease_cache_usage(ctx, sizeof(struct spng_text2));

    text->keyword = NULL;
    text->text = NULL;

    ctx->n_text--;
}

static void chunk_undo(spng_ctx *ctx)
{
    struct spng_unknown_chunk *chunk = &ctx->chunk_list[ctx->n_chunks - 1];

    spng__free(ctx, chunk->data);

    decrease_cache_usage(ctx, chunk->length);
    decrease_cache_usage(ctx, sizeof(struct spng_unknown_chunk));

    chunk->data = NULL;

    ctx->n_chunks--;
}

static int read_non_idat_chunks(spng_ctx *ctx)
{
    int ret;
    struct spng_chunk chunk;
    const unsigned char *data;

    ctx->discard = 0;
    ctx->undo = NULL;
    ctx->prev_stored = ctx->stored;

    while( !(ret = read_header(ctx)))
    {
        if(ctx->discard)
        {
            if(ctx->undo) ctx->undo(ctx);

            ctx->stored = ctx->prev_stored;
        }

        ctx->discard = 0;
        ctx->undo = NULL;

        ctx->prev_stored = ctx->stored;
        chunk = ctx->current_chunk;

        if(!memcmp(chunk.type, type_idat, 4))
        {
            if(ctx->state < SPNG_STATE_FIRST_IDAT)
            {
                if(ctx->ihdr.color_type == 3 && !ctx->stored.plte) return SPNG_ENOPLTE;

                ctx->first_idat = chunk;
                return 0;
            }

            if(ctx->prev_was_idat)
            {
                /* Ignore extra IDAT's */
                ret = discard_chunk_bytes(ctx, chunk.length);
                if(ret) return ret;

                continue;
            }
            else return SPNG_ECHUNK_POS; /* IDAT chunk not at the end of the IDAT sequence */
        }

        ctx->prev_was_idat = 0;

        if(is_small_chunk(chunk.type))
        {
            /* None of the known chunks can be zero length */
            if(!chunk.length) return SPNG_ECHUNK_SIZE;

            /* The largest of these chunks is PLTE with 256 entries */
            ret = read_chunk_bytes(ctx, chunk.length > 768 ? 768 : chunk.length);
            if(ret) return ret;
        }

        data = ctx->data;

        if(is_critical_chunk(&chunk))
        {
            if(!memcmp(chunk.type, type_plte, 4))
            {
                if(ctx->file.trns || ctx->file.hist || ctx->file.bkgd) return SPNG_ECHUNK_POS;
                if(chunk.length % 3 != 0) return SPNG_ECHUNK_SIZE;

                ctx->plte.n_entries = chunk.length / 3;

                if(check_plte(&ctx->plte, &ctx->ihdr)) return SPNG_ECHUNK_SIZE; /* XXX: EPLTE? */

                size_t i;
                for(i=0; i < ctx->plte.n_entries; i++)
                {
                    ctx->plte.entries[i].red   = data[i * 3];
                    ctx->plte.entries[i].green = data[i * 3 + 1];
                    ctx->plte.entries[i].blue  = data[i * 3 + 2];
                }

                ctx->file.plte = 1;
                ctx->stored.plte = 1;
            }
            else if(!memcmp(chunk.type, type_iend, 4))
            {
                if(ctx->state == SPNG_STATE_AFTER_IDAT)
                {
                    if(chunk.length) return SPNG_ECHUNK_SIZE;

                    ret = read_and_check_crc(ctx);
                    if(ret == -SPNG_CRC_DISCARD) ret = 0;

                    return ret;
                }
                else return SPNG_ECHUNK_POS;
            }
            else if(!memcmp(chunk.type, type_ihdr, 4)) return SPNG_ECHUNK_POS;
            else return SPNG_ECHUNK_UNKNOWN_CRITICAL;
        }
        else if(!memcmp(chunk.type, type_chrm, 4)) /* Ancillary chunks */
        {
            if(ctx->file.plte) return SPNG_ECHUNK_POS;
            if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
            if(ctx->file.chrm) return SPNG_EDUP_CHRM;

            if(chunk.length != 32) return SPNG_ECHUNK_SIZE;

            ctx->chrm_int.white_point_x = read_u32(data);
            ctx->chrm_int.white_point_y = read_u32(data + 4);
            ctx->chrm_int.red_x = read_u32(data + 8);
            ctx->chrm_int.red_y = read_u32(data + 12);
            ctx->chrm_int.green_x = read_u32(data + 16);
            ctx->chrm_int.green_y = read_u32(data + 20);
            ctx->chrm_int.blue_x = read_u32(data + 24);
            ctx->chrm_int.blue_y = read_u32(data + 28);

            if(check_chrm_int(&ctx->chrm_int)) return SPNG_ECHRM;

            ctx->file.chrm = 1;
            ctx->stored.chrm = 1;
        }
        else if(!memcmp(chunk.type, type_gama, 4))
        {
            if(ctx->file.plte) return SPNG_ECHUNK_POS;
            if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
            if(ctx->file.gama) return SPNG_EDUP_GAMA;

            if(chunk.length != 4) return SPNG_ECHUNK_SIZE;

            ctx->gama = read_u32(data);

            if(!ctx->gama) return SPNG_EGAMA;
            if(ctx->gama > spng_u32max) return SPNG_EGAMA;

            ctx->file.gama = 1;
            ctx->stored.gama = 1;
        }
        else if(!memcmp(chunk.type, type_sbit, 4))
        {
            if(ctx->file.plte) return SPNG_ECHUNK_POS;
            if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
            if(ctx->file.sbit) return SPNG_EDUP_SBIT;

            if(ctx->ihdr.color_type == 0)
            {
                if(chunk.length != 1) return SPNG_ECHUNK_SIZE;

                ctx->sbit.grayscale_bits = data[0];
            }
            else if(ctx->ihdr.color_type == 2 || ctx->ihdr.color_type == 3)
            {
                if(chunk.length != 3) return SPNG_ECHUNK_SIZE;

                ctx->sbit.red_bits = data[0];
                ctx->sbit.green_bits = data[1];
                ctx->sbit.blue_bits = data[2];
            }
            else if(ctx->ihdr.color_type == 4)
            {
                if(chunk.length != 2) return SPNG_ECHUNK_SIZE;

                ctx->sbit.grayscale_bits = data[0];
                ctx->sbit.alpha_bits = data[1];
            }
            else if(ctx->ihdr.color_type == 6)
            {
                if(chunk.length != 4) return SPNG_ECHUNK_SIZE;

                ctx->sbit.red_bits = data[0];
                ctx->sbit.green_bits = data[1];
                ctx->sbit.blue_bits = data[2];
                ctx->sbit.alpha_bits = data[3];
            }

            if(check_sbit(&ctx->sbit, &ctx->ihdr)) return SPNG_ESBIT;

            ctx->file.sbit = 1;
            ctx->stored.sbit = 1;
        }
        else if(!memcmp(chunk.type, type_srgb, 4))
        {
            if(ctx->file.plte) return SPNG_ECHUNK_POS;
            if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
            if(ctx->file.srgb) return SPNG_EDUP_SRGB;

            if(chunk.length != 1) return SPNG_ECHUNK_SIZE;

            ctx->srgb_rendering_intent = data[0];

            if(ctx->srgb_rendering_intent > 3) return SPNG_ESRGB;

            ctx->file.srgb = 1;
            ctx->stored.srgb = 1;
        }
        else if(!memcmp(chunk.type, type_bkgd, 4))
        {
            if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
            if(ctx->file.bkgd) return SPNG_EDUP_BKGD;

            if(ctx->ihdr.color_type == 0 || ctx->ihdr.color_type == 4)
            {
                if(chunk.length != 2) return SPNG_ECHUNK_SIZE;

                ctx->bkgd.gray = read_u16(data);
            }
            else if(ctx->ihdr.color_type == 2 || ctx->ihdr.color_type == 6)
            {
                if(chunk.length != 6) return SPNG_ECHUNK_SIZE;

                ctx->bkgd.red = read_u16(data);
                ctx->bkgd.green = read_u16(data + 2);
                ctx->bkgd.blue = read_u16(data + 4);
            }
            else if(ctx->ihdr.color_type == 3)
            {
                if(chunk.length != 1) return SPNG_ECHUNK_SIZE;
                if(!ctx->file.plte) return SPNG_EBKGD_NO_PLTE;

                ctx->bkgd.plte_index = data[0];
                if(ctx->bkgd.plte_index >= ctx->plte.n_entries) return SPNG_EBKGD_PLTE_IDX;
            }

            ctx->file.bkgd = 1;
            ctx->stored.bkgd = 1;
        }
        else if(!memcmp(chunk.type, type_trns, 4))
        {
            if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
            if(ctx->file.trns) return SPNG_EDUP_TRNS;
            if(!chunk.length) return SPNG_ECHUNK_SIZE;

            if(ctx->ihdr.color_type == 0)
            {
                if(chunk.length != 2) return SPNG_ECHUNK_SIZE;

                ctx->trns.gray = read_u16(data);
            }
            else if(ctx->ihdr.color_type == 2)
            {
                if(chunk.length != 6) return SPNG_ECHUNK_SIZE;

                ctx->trns.red = read_u16(data);
                ctx->trns.green = read_u16(data + 2);
                ctx->trns.blue = read_u16(data + 4);
            }
            else if(ctx->ihdr.color_type == 3)
            {
                if(chunk.length > ctx->plte.n_entries) return SPNG_ECHUNK_SIZE;
                if(!ctx->file.plte) return SPNG_ETRNS_NO_PLTE;

                memcpy(ctx->trns.type3_alpha, data, chunk.length);
                ctx->trns.n_type3_entries = chunk.length;
            }

            if(ctx->ihdr.color_type == 4 || ctx->ihdr.color_type == 6)  return SPNG_ETRNS_COLOR_TYPE;

            ctx->file.trns = 1;
            ctx->stored.trns = 1;
        }
        else if(!memcmp(chunk.type, type_hist, 4))
        {
            if(!ctx->file.plte) return SPNG_EHIST_NO_PLTE;
            if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
            if(ctx->file.hist) return SPNG_EDUP_HIST;

            if( (chunk.length / 2) != (ctx->plte.n_entries) ) return SPNG_ECHUNK_SIZE;

            size_t k;
            for(k=0; k < (chunk.length / 2); k++)
            {
                ctx->hist.frequency[k] = read_u16(data + k*2);
            }

            ctx->file.hist = 1;
            ctx->stored.hist = 1;
        }
        else if(!memcmp(chunk.type, type_phys, 4))
        {
            if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
            if(ctx->file.phys) return SPNG_EDUP_PHYS;

            if(chunk.length != 9) return SPNG_ECHUNK_SIZE;

            ctx->phys.ppu_x = read_u32(data);
            ctx->phys.ppu_y = read_u32(data + 4);
            ctx->phys.unit_specifier = data[8];

            if(check_phys(&ctx->phys)) return SPNG_EPHYS;

            ctx->file.phys = 1;
            ctx->stored.phys = 1;
        }
        else if(!memcmp(chunk.type, type_time, 4))
        {
            if(ctx->file.time) return SPNG_EDUP_TIME;

            if(chunk.length != 7) return SPNG_ECHUNK_SIZE;

            struct spng_time time;

            time.year = read_u16(data);
            time.month = data[2];
            time.day = data[3];
            time.hour = data[4];
            time.minute = data[5];
            time.second = data[6];

            if(check_time(&time)) return SPNG_ETIME;

            ctx->file.time = 1;

            if(!ctx->user.time) ctx->time = time;

            ctx->stored.time = 1;
        }
        else if(!memcmp(chunk.type, type_offs, 4))
        {
            if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
            if(ctx->file.offs) return SPNG_EDUP_OFFS;

            if(chunk.length != 9) return SPNG_ECHUNK_SIZE;

            ctx->offs.x = read_s32(data);
            ctx->offs.y = read_s32(data + 4);
            ctx->offs.unit_specifier = data[8];

            if(check_offs(&ctx->offs)) return SPNG_EOFFS;

            ctx->file.offs = 1;
            ctx->stored.offs = 1;
        }
        else /* Arbitrary-length chunk */
        {

            if(!memcmp(chunk.type, type_exif, 4))
            {
                if(ctx->file.exif) return SPNG_EDUP_EXIF;
                if(!chunk.length) return SPNG_EEXIF;

                ctx->file.exif = 1;

                if(ctx->user.exif) goto discard;

                if(increase_cache_usage(ctx, chunk.length, 1)) return SPNG_ECHUNK_LIMITS;

                struct spng_exif exif;

                exif.length = chunk.length;

                exif.data = spng__malloc(ctx, chunk.length);
                if(exif.data == NULL) return SPNG_EMEM;

                ret = read_chunk_bytes2(ctx, exif.data, chunk.length);
                if(ret)
                {
                    spng__free(ctx, exif.data);
                    return ret;
                }

                if(check_exif(&exif))
                {
                    spng__free(ctx, exif.data);
                    return SPNG_EEXIF;
                }

                ctx->exif = exif;

                ctx->stored.exif = 1;
            }
            else if(!memcmp(chunk.type, type_iccp, 4))
            {/* TODO: add test file with color profile */
                if(ctx->file.plte) return SPNG_ECHUNK_POS;
                if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
                if(ctx->file.iccp) return SPNG_EDUP_ICCP;
                if(!chunk.length) return SPNG_ECHUNK_SIZE;

                ctx->file.iccp = 1;

                uint32_t peek_bytes =  81 > chunk.length ? chunk.length : 81;

                ret = read_chunk_bytes(ctx, peek_bytes);
                if(ret) return ret;

                unsigned char *keyword_nul = memchr(ctx->data, '\0', peek_bytes);
                if(keyword_nul == NULL) return SPNG_EICCP_NAME;

                uint32_t keyword_len = keyword_nul - ctx->data;

                if(keyword_len > 79) return SPNG_EICCP_NAME;

                memcpy(ctx->iccp.profile_name, ctx->data, keyword_len);

                if(check_png_keyword(ctx->iccp.profile_name)) return SPNG_EICCP_NAME;

                if(chunk.length < (keyword_len + 2)) return SPNG_ECHUNK_SIZE;

                if(ctx->data[keyword_len + 1] != 0) return SPNG_EICCP_COMPRESSION_METHOD;

                ret = spng__inflate_stream(ctx, &ctx->iccp.profile, &ctx->iccp.profile_len, 0, ctx->data + keyword_len + 2, peek_bytes - (keyword_len + 2));

                if(ret) return ret;

                ctx->stored.iccp = 1;
            }
             else if(!memcmp(chunk.type, type_text, 4) ||
                     !memcmp(chunk.type, type_ztxt, 4) ||
                     !memcmp(chunk.type, type_itxt, 4))
            {
                if(!chunk.length) return SPNG_ECHUNK_SIZE;

                ctx->file.text = 1;

                if(ctx->user.text) goto discard;

                if(increase_cache_usage(ctx, sizeof(struct spng_text2), 1)) return SPNG_ECHUNK_LIMITS;

                ctx->n_text++;
                if(ctx->n_text < 1) return SPNG_EOVERFLOW;
                if(sizeof(struct spng_text2) > SIZE_MAX / ctx->n_text) return SPNG_EOVERFLOW;

                void *buf = spng__realloc(ctx, ctx->text_list, ctx->n_text * sizeof(struct spng_text2));
                if(buf == NULL) return SPNG_EMEM;
                ctx->text_list = buf;

                struct spng_text2 *text = &ctx->text_list[ctx->n_text - 1];
                memset(text, 0, sizeof(struct spng_text2));

                ctx->undo = text_undo;

                uint32_t text_offset = 0, language_tag_offset = 0, translated_keyword_offset = 0;
                uint32_t peek_bytes = 256; /* enough for 3 80-byte keywords and some text bytes */
                uint32_t keyword_len;

                if(peek_bytes > chunk.length) peek_bytes = chunk.length;

                ret = read_chunk_bytes(ctx, peek_bytes);
                if(ret) return ret;

                data = ctx->data;

                const unsigned char *zlib_stream = NULL;
                const unsigned char *peek_end = data + peek_bytes;
                const unsigned char *keyword_nul = memchr(data, 0, chunk.length > 80 ? 80 : chunk.length);

                if(keyword_nul == NULL) return SPNG_ETEXT_KEYWORD;

                keyword_len = keyword_nul - data;

                if(!memcmp(chunk.type, type_text, 4))
                {
                    text->type = SPNG_TEXT;

                    text->text_length = chunk.length - keyword_len - 1;

                    text_offset = keyword_len;

                    /* increment past nul if there is a text field */
                    if(text->text_length) text_offset++;
                }
                else if(!memcmp(chunk.type, type_ztxt, 4))
                {
                    text->type = SPNG_ZTXT;

                    if((peek_bytes - keyword_len) <= 2) return SPNG_EZTXT;

                    if(keyword_nul[1]) return SPNG_EZTXT_COMPRESSION_METHOD;

                    text->compression_flag = 1;

                    text_offset = keyword_len + 2;
                }
                else if(!memcmp(chunk.type, type_itxt, 4))
                {
                    text->type = SPNG_ITXT;

                    /* at least two 1-byte fields, two >=0 length strings, and one byte of (compressed) text */
                    if((peek_bytes - keyword_len) < 5) return SPNG_EITXT;

                    text->compression_flag = keyword_nul[1];

                    if(text->compression_flag > 1) return SPNG_EITXT_COMPRESSION_FLAG;

                    if(keyword_nul[2]) return SPNG_EITXT_COMPRESSION_METHOD;

                    language_tag_offset = keyword_len + 3;

                    const unsigned char *term;
                    term = memchr(data + language_tag_offset, 0, peek_bytes - language_tag_offset);
                    if(term == NULL) return SPNG_EITXT_LANG_TAG;

                    if((peek_end - term) < 2) return SPNG_EITXT;

                    translated_keyword_offset = term - data + 1;

                    zlib_stream = memchr(data + translated_keyword_offset, 0, peek_bytes - translated_keyword_offset);
                    if(zlib_stream == NULL) return SPNG_EITXT;
                    if(zlib_stream == peek_end) return SPNG_EITXT;

                    text_offset = zlib_stream - data + 1;
                    text->text_length = chunk.length - text_offset;
                }
                else return SPNG_EINTERNAL;


                if(text->compression_flag)
                {
                    /* cache usage = peek_bytes + decompressed text size + nul */
                    if(increase_cache_usage(ctx, peek_bytes, 0)) return SPNG_ECHUNK_LIMITS;

                    text->keyword = spng__calloc(ctx, 1, peek_bytes);
                    if(text->keyword == NULL) return SPNG_EMEM;

                    memcpy(text->keyword, data, peek_bytes);

                    zlib_stream = ctx->data + text_offset;

                    ret = spng__inflate_stream(ctx, &text->text, &text->text_length, 1, zlib_stream, peek_bytes - text_offset);

                    if(ret) return ret;

                    text->text[text->text_length - 1] = '\0';
                    text->cache_usage = text->text_length + peek_bytes;
                }
                else
                {
                    if(increase_cache_usage(ctx, chunk.length + 1, 0)) return SPNG_ECHUNK_LIMITS;

                    text->keyword = spng__malloc(ctx, chunk.length + 1);
                    if(text->keyword == NULL) return SPNG_EMEM;

                    memcpy(text->keyword, data, peek_bytes);

                    if(chunk.length > peek_bytes)
                    {
                        ret = read_chunk_bytes2(ctx, text->keyword + peek_bytes, chunk.length - peek_bytes);
                        if(ret) return ret;
                    }

                    text->text = text->keyword + text_offset;

                    text->text_length = chunk.length - text_offset;

                    text->text[text->text_length] = '\0';
                    text->cache_usage = chunk.length + 1;
                }

                if(check_png_keyword(text->keyword)) return SPNG_ETEXT_KEYWORD;

                text->text_length = strlen(text->text);

                if(text->type != SPNG_ITXT)
                {
                    language_tag_offset = keyword_len;
                    translated_keyword_offset = keyword_len;

                    if(ctx->strict && check_png_text(text->text, text->text_length))
                    {
                        if(text->type == SPNG_ZTXT) return SPNG_EZTXT;
                        else return SPNG_ETEXT;
                    }
                }

                text->language_tag = text->keyword + language_tag_offset;
                text->translated_keyword = text->keyword + translated_keyword_offset;

                ctx->stored.text = 1;
            }
            else if(!memcmp(chunk.type, type_splt, 4))
            {
                if(ctx->state == SPNG_STATE_AFTER_IDAT) return SPNG_ECHUNK_POS;
                if(ctx->user.splt) goto discard; /* XXX: could check profile names for uniqueness */
                if(!chunk.length) return SPNG_ECHUNK_SIZE;

                ctx->file.splt = 1;

                /* chunk.length + sizeof(struct spng_splt) + splt->n_entries * sizeof(struct spng_splt_entry) */
                if(increase_cache_usage(ctx, chunk.length + sizeof(struct spng_splt), 1)) return SPNG_ECHUNK_LIMITS;

                ctx->n_splt++;
                if(ctx->n_splt < 1) return SPNG_EOVERFLOW;
                if(sizeof(struct spng_splt) > SIZE_MAX / ctx->n_splt) return SPNG_EOVERFLOW;

                void *buf = spng__realloc(ctx, ctx->splt_list, ctx->n_splt * sizeof(struct spng_splt));
                if(buf == NULL) return SPNG_EMEM;
                ctx->splt_list = buf;

                struct spng_splt *splt = &ctx->splt_list[ctx->n_splt - 1];

                memset(splt, 0, sizeof(struct spng_splt));

                ctx->undo = splt_undo;

                void *t = spng__malloc(ctx, chunk.length);
                if(t == NULL) return SPNG_EMEM;

                splt->entries = t; /* simplifies error handling */
                data = t;

                ret = read_chunk_bytes2(ctx, t, chunk.length);
                if(ret) return ret;

                uint32_t keyword_len = chunk.length < 80 ? chunk.length : 80;

                const unsigned char *keyword_nul = memchr(data, 0, keyword_len);
                if(keyword_nul == NULL) return SPNG_ESPLT_NAME;

                keyword_len = keyword_nul - data;

                memcpy(splt->name, data, keyword_len);

                if(check_png_keyword(splt->name)) return SPNG_ESPLT_NAME;

                uint32_t j;
                for(j=0; j < (ctx->n_splt - 1); j++)
                {
                    if(!strcmp(ctx->splt_list[j].name, splt->name)) return SPNG_ESPLT_DUP_NAME;
                }

                if( (chunk.length - keyword_len) <= 2) return SPNG_ECHUNK_SIZE;

                splt->sample_depth = data[keyword_len + 1];

                uint32_t entries_len = chunk.length - keyword_len - 2;
                if(!entries_len) return SPNG_ECHUNK_SIZE;

                if(splt->sample_depth == 16)
                {
                    if(entries_len % 10 != 0) return SPNG_ECHUNK_SIZE;
                    splt->n_entries = entries_len / 10;
                }
                else if(splt->sample_depth == 8)
                {
                    if(entries_len % 6 != 0) return SPNG_ECHUNK_SIZE;
                    splt->n_entries = entries_len / 6;
                }
                else return SPNG_ESPLT_DEPTH;

                if(!splt->n_entries) return SPNG_ECHUNK_SIZE;

                size_t list_size = splt->n_entries;

                if(list_size > SIZE_MAX / sizeof(struct spng_splt_entry)) return SPNG_EOVERFLOW;

                list_size *= sizeof(struct spng_splt_entry);

                if(increase_cache_usage(ctx, list_size, 0)) return SPNG_ECHUNK_LIMITS;

                splt->entries = spng__malloc(ctx, list_size);
                if(splt->entries == NULL)
                {
                    spng__free(ctx, t);
                    return SPNG_EMEM;
                }

                data = (unsigned char*)t + keyword_len + 2;

                uint32_t k;
                if(splt->sample_depth == 16)
                {
                    for(k=0; k < splt->n_entries; k++)
                    {
                        splt->entries[k].red =   read_u16(data + k * 10);
                        splt->entries[k].green = read_u16(data + k * 10 + 2);
                        splt->entries[k].blue =  read_u16(data + k * 10 + 4);
                        splt->entries[k].alpha = read_u16(data + k * 10 + 6);
                        splt->entries[k].frequency = read_u16(data + k * 10 + 8);
                    }
                }
                else if(splt->sample_depth == 8)
                {
                    for(k=0; k < splt->n_entries; k++)
                    {
                        splt->entries[k].red =   data[k * 6];
                        splt->entries[k].green = data[k * 6 + 1];
                        splt->entries[k].blue =  data[k * 6 + 2];
                        splt->entries[k].alpha = data[k * 6 + 3];
                        splt->entries[k].frequency = read_u16(data + k * 6 + 4);
                    }
                }

                spng__free(ctx, t);
                decrease_cache_usage(ctx, chunk.length);

                ctx->stored.splt = 1;
            }
            else /* Unknown chunk */
            {
                ctx->file.unknown = 1;

                if(!ctx->keep_unknown) goto discard;
                if(ctx->user.unknown) goto discard;

                if(increase_cache_usage(ctx, chunk.length + sizeof(struct spng_unknown_chunk), 1)) return SPNG_ECHUNK_LIMITS;

                ctx->n_chunks++;
                if(ctx->n_chunks < 1) return SPNG_EOVERFLOW;
                if(sizeof(struct spng_unknown_chunk) > SIZE_MAX / ctx->n_chunks) return SPNG_EOVERFLOW;

                void *buf = spng__realloc(ctx, ctx->chunk_list, ctx->n_chunks * sizeof(struct spng_unknown_chunk));
                if(buf == NULL) return SPNG_EMEM;
                ctx->chunk_list = buf;

                struct spng_unknown_chunk *chunkp = &ctx->chunk_list[ctx->n_chunks - 1];

                memset(chunkp, 0, sizeof(struct spng_unknown_chunk));

                ctx->undo = chunk_undo;

                memcpy(chunkp->type, chunk.type, 4);

                if(ctx->state < SPNG_STATE_FIRST_IDAT)
                {
                    if(ctx->file.plte) chunkp->location = SPNG_AFTER_PLTE;
                    else chunkp->location = SPNG_AFTER_IHDR;
                }
                else if(ctx->state >= SPNG_STATE_AFTER_IDAT) chunkp->location = SPNG_AFTER_IDAT;

                if(chunk.length > 0)
                {
                    void *t = spng__malloc(ctx, chunk.length);
                    if(t == NULL) return SPNG_EMEM;

                    ret = read_chunk_bytes2(ctx, t, chunk.length);
                    if(ret)
                    {
                        spng__free(ctx, t);
                        return ret;
                    }

                    chunkp->length = chunk.length;
                    chunkp->data = t;
                }

                ctx->stored.unknown = 1;
            }

discard:
            ret = discard_chunk_bytes(ctx, ctx->cur_chunk_bytes_left);
            if(ret) return ret;
        }

    }

    return ret;
}

/* Read chunks before or after the IDAT chunks depending on state */
static int read_chunks(spng_ctx *ctx, int only_ihdr)
{
    if(ctx == NULL) return SPNG_EINTERNAL;
    if(!ctx->state) return SPNG_EBADSTATE;
    if(ctx->data == NULL)
    {
        if(ctx->encode_only) return 0;
        else return SPNG_EINTERNAL;
    }

    int ret = 0;

    if(ctx->state == SPNG_STATE_INPUT)
    {
        ret = read_ihdr(ctx);

        if(ret) return decode_err(ctx, ret);

        ctx->state = SPNG_STATE_IHDR;
    }

    if(only_ihdr) return 0;

    if(ctx->state == SPNG_STATE_EOI)
    {
        ctx->state = SPNG_STATE_AFTER_IDAT;
        ctx->prev_was_idat = 1;
    }

    while(ctx->state < SPNG_STATE_FIRST_IDAT || ctx->state == SPNG_STATE_AFTER_IDAT)
    {
        ret = read_non_idat_chunks(ctx);

        if(!ret)
        {
            if(ctx->state < SPNG_STATE_FIRST_IDAT) ctx->state = SPNG_STATE_FIRST_IDAT;
            else if(ctx->state == SPNG_STATE_AFTER_IDAT) ctx->state = SPNG_STATE_IEND;
        }
        else
        {
            switch(ret)
            {
                case SPNG_ECHUNK_POS:
                case SPNG_ECHUNK_SIZE: /* size != expected size, SPNG_ECHUNK_STDLEN = invalid size */
                case SPNG_EDUP_PLTE:
                case SPNG_EDUP_CHRM:
                case SPNG_EDUP_GAMA:
                case SPNG_EDUP_ICCP:
                case SPNG_EDUP_SBIT:
                case SPNG_EDUP_SRGB:
                case SPNG_EDUP_BKGD:
                case SPNG_EDUP_HIST:
                case SPNG_EDUP_TRNS:
                case SPNG_EDUP_PHYS:
                case SPNG_EDUP_TIME:
                case SPNG_EDUP_OFFS:
                case SPNG_EDUP_EXIF:
                case SPNG_ECHRM:
                case SPNG_ETRNS_COLOR_TYPE:
                case SPNG_ETRNS_NO_PLTE:
                case SPNG_EGAMA:
                case SPNG_EICCP_NAME:
                case SPNG_EICCP_COMPRESSION_METHOD:
                case SPNG_ESBIT:
                case SPNG_ESRGB:
                case SPNG_ETEXT:
                case SPNG_ETEXT_KEYWORD:
                case SPNG_EZTXT:
                case SPNG_EZTXT_COMPRESSION_METHOD:
                case SPNG_EITXT:
                case SPNG_EITXT_COMPRESSION_FLAG:
                case SPNG_EITXT_COMPRESSION_METHOD:
                case SPNG_EITXT_LANG_TAG:
                case SPNG_EITXT_TRANSLATED_KEY:
                case SPNG_EBKGD_NO_PLTE:
                case SPNG_EBKGD_PLTE_IDX:
                case SPNG_EHIST_NO_PLTE:
                case SPNG_EPHYS:
                case SPNG_ESPLT_NAME:
                case SPNG_ESPLT_DUP_NAME:
                case SPNG_ESPLT_DEPTH:
                case SPNG_ETIME:
                case SPNG_EOFFS:
                case SPNG_EEXIF:
                case SPNG_EZLIB:
                {
                    if(!ctx->strict && !is_critical_chunk(&ctx->current_chunk))
                    {
                        ret = discard_chunk_bytes(ctx, ctx->cur_chunk_bytes_left);
                        if(ret) return decode_err(ctx, ret);

                        if(ctx->undo) ctx->undo(ctx);

                        ctx->stored = ctx->prev_stored;

                        ctx->discard = 0;
                        ctx->undo = NULL;

                        continue;
                    }
                    else return decode_err(ctx, ret);

                    break;
                }
                default: return decode_err(ctx, ret);
            }
        }
    }

    return ret;
}

static int read_scanline(spng_ctx *ctx)
{
    int ret, pass = ctx->row_info.pass;
    struct spng_row_info *ri = &ctx->row_info;
    const struct spng_subimage *sub = ctx->subimage;
    size_t scanline_width = sub[pass].scanline_width;
    uint32_t scanline_idx = ri->scanline_idx;

    uint8_t next_filter = 0;

    if(scanline_idx == (sub[pass].height - 1) && ri->pass == ctx->last_pass)
    {
        ret = read_scanline_bytes(ctx, ctx->scanline, scanline_width - 1);
    }
    else
    {
        ret = read_scanline_bytes(ctx, ctx->scanline, scanline_width);
        if(ret) return ret;

        next_filter = ctx->scanline[scanline_width - 1];
        if(next_filter > 4) ret = SPNG_EFILTER;
    }

    if(ret) return ret;

    if(!scanline_idx && ri->filter > 1)
    {
        /* prev_scanline is all zeros for the first scanline */
        memset(ctx->prev_scanline, 0, scanline_width);
    }

    if(ctx->ihdr.bit_depth == 16 && ctx->fmt != SPNG_FMT_RAW) u16_row_to_host(ctx->scanline, scanline_width - 1);

    ret = defilter_scanline(ctx->prev_scanline, ctx->scanline, scanline_width, ctx->bytes_per_pixel, ri->filter);
    if(ret) return ret;

    ri->filter = next_filter;

    return 0;
}

static int update_row_info(spng_ctx *ctx)
{
    int interlacing = ctx->ihdr.interlace_method;
    struct spng_row_info *ri = &ctx->row_info;
    const struct spng_subimage *sub = ctx->subimage;

    if(ri->scanline_idx == (sub[ri->pass].height - 1)) /* Last scanline */
    {
        if(ri->pass == ctx->last_pass)
        {
            ctx->state = SPNG_STATE_EOI;

            return SPNG_EOI;
        }

        ri->scanline_idx = 0;
        ri->pass++;

        /* Skip empty passes */
        while( (!sub[ri->pass].width || !sub[ri->pass].height) && (ri->pass < ctx->last_pass)) ri->pass++;
    }
    else
    {
        ri->row_num++;
        ri->scanline_idx++;
    }

    if(interlacing) ri->row_num = adam7_y_start[ri->pass] + ri->scanline_idx * adam7_y_delta[ri->pass];

    return 0;
}

int spng_decode_scanline(spng_ctx *ctx, void *out, size_t len)
{
    if(ctx == NULL || out == NULL) return 1;

    if(ctx->state >= SPNG_STATE_EOI) return SPNG_EOI;

    struct decode_flags f = ctx->decode_flags;

    struct spng_row_info *ri = &ctx->row_info;
    const struct spng_subimage *sub = ctx->subimage;

    const struct spng_ihdr *ihdr = &ctx->ihdr;
    const uint16_t *gamma_lut = ctx->gamma_lut;
    unsigned char *trns_px = ctx->trns_px;
    const struct spng_sbit *sb = &ctx->decode_sb;
    const struct spng_plte_entry *plte = ctx->decode_plte.rgba;
    struct spng__iter iter = spng__iter_init(ihdr->bit_depth, ctx->scanline);

    const unsigned char *scanline;

    const int pass = ri->pass;
    const int fmt = ctx->fmt;
    const size_t scanline_width = sub[pass].scanline_width;
    const uint32_t width = sub[pass].width;
    uint32_t k;
    uint8_t r_8, g_8, b_8, a_8, gray_8;
    uint16_t r_16, g_16, b_16, a_16, gray_16;
    r_8=0; g_8=0; b_8=0; a_8=0; gray_8=0;
    r_16=0; g_16=0; b_16=0; a_16=0; gray_16=0;
    size_t pixel_size = 4; /* SPNG_FMT_RGBA8 */
    size_t pixel_offset = 0;
    unsigned char *pixel;
    unsigned processing_depth = ihdr->bit_depth;

    if(f.indexed) processing_depth = 8;

    if(fmt == SPNG_FMT_RGBA16) pixel_size = 8;
    else if(fmt == SPNG_FMT_RGB8) pixel_size = 3;

    if(len < sub[pass].out_width) return SPNG_EBUFSIZ;

    int ret = read_scanline(ctx);

    if(ret) return decode_err(ctx, ret);

    scanline = ctx->scanline;

    for(k=0; k < width; k++)
    {
        pixel = (unsigned char*)out + pixel_offset;
        pixel_offset += pixel_size;

        if(f.same_layout)
        {
            if(f.zerocopy) break;

            memcpy(out, scanline, scanline_width - 1);
            break;
        }

        if(f.unpack)
        {
            unpack_scanline(out, scanline, width, ihdr->bit_depth, fmt);
            break;
        }

        if(ihdr->color_type == SPNG_COLOR_TYPE_TRUECOLOR)
        {
            if(ihdr->bit_depth == 16)
            {
                memcpy(&r_16, scanline + (k * 6), 2);
                memcpy(&g_16, scanline + (k * 6) + 2, 2);
                memcpy(&b_16, scanline + (k * 6) + 4, 2);

                a_16 = 65535;
            }
            else /* == 8 */
            {
                if(fmt == SPNG_FMT_RGBA8)
                {
                    rgb8_row_to_rgba8(scanline, out, width);
                    break;
                }

                r_8 = scanline[k * 3];
                g_8 = scanline[k * 3 + 1];
                b_8 = scanline[k * 3 + 2];

                a_8 = 255;
            }
        }
        else if(ihdr->color_type == SPNG_COLOR_TYPE_INDEXED)
        {
            uint8_t entry = 0;

            if(ihdr->bit_depth == 8)
            {
                if(fmt & (SPNG_FMT_RGBA8 | SPNG_FMT_RGB8))
                {
                    expand_row(out, scanline, &ctx->decode_plte, width, fmt);
                    break;
                }

                entry = scanline[k];
            }
            else /* < 8 */
            {
                entry = get_sample(&iter);
            }

            if(fmt & (SPNG_FMT_RGBA8 | SPNG_FMT_RGB8))
            {
                pixel[0] = plte[entry].red;
                pixel[1] = plte[entry].green;
                pixel[2] = plte[entry].blue;
                if(fmt == SPNG_FMT_RGBA8) pixel[3] = plte[entry].alpha;

                continue;
            }
            else /* RGBA16 */
            {
                r_16 = plte[entry].red;
                g_16 = plte[entry].green;
                b_16 = plte[entry].blue;
                a_16 = plte[entry].alpha;

                r_16 = (r_16 << 8) | r_16;
                g_16 = (g_16 << 8) | g_16;
                b_16 = (b_16 << 8) | b_16;
                a_16 = (a_16 << 8) | a_16;

                memcpy(pixel, &r_16, 2);
                memcpy(pixel + 2, &g_16, 2);
                memcpy(pixel + 4, &b_16, 2);
                memcpy(pixel + 6, &a_16, 2);

                continue;
            }
        }
        else if(ihdr->color_type == SPNG_COLOR_TYPE_TRUECOLOR_ALPHA)
        {
            if(ihdr->bit_depth == 16)
            {
                memcpy(&r_16, scanline + (k * 8), 2);
                memcpy(&g_16, scanline + (k * 8) + 2, 2);
                memcpy(&b_16, scanline + (k * 8) + 4, 2);
                memcpy(&a_16, scanline + (k * 8) + 6, 2);
            }
            else /* == 8 */
            {
                r_8 = scanline[k * 4];
                g_8 = scanline[k * 4 + 1];
                b_8 = scanline[k * 4 + 2];
                a_8 = scanline[k * 4 + 3];
            }
        }
        else if(ihdr->color_type == SPNG_COLOR_TYPE_GRAYSCALE)
        {
            if(ihdr->bit_depth == 16)
            {
                memcpy(&gray_16, scanline + k * 2, 2);

                if(f.apply_trns && ctx->trns.gray == gray_16) a_16 = 0;
                else a_16 = 65535;

                r_16 = gray_16;
                g_16 = gray_16;
                b_16 = gray_16;
            }
            else /* <= 8 */
            {
                gray_8 = get_sample(&iter);

                if(f.apply_trns && ctx->trns.gray == gray_8) a_8 = 0;
                else a_8 = 255;

                r_8 = gray_8; g_8 = gray_8; b_8 = gray_8;
            }
        }
        else if(ihdr->color_type == SPNG_COLOR_TYPE_GRAYSCALE_ALPHA)
        {
            if(ihdr->bit_depth == 16)
            {
                memcpy(&gray_16, scanline + (k * 4), 2);
                memcpy(&a_16, scanline + (k * 4) + 2, 2);

                r_16 = gray_16;
                g_16 = gray_16;
                b_16 = gray_16;
            }
            else /* == 8 */
            {
                gray_8 = scanline[k * 2];
                a_8 = scanline[k * 2 + 1];

                r_8 = gray_8;
                g_8 = gray_8;
                b_8 = gray_8;
            }
        }


        if(fmt & (SPNG_FMT_RGBA8 | SPNG_FMT_RGB8))
        {
            if(ihdr->bit_depth == 16)
            {
                r_8 = r_16 >> 8;
                g_8 = g_16 >> 8;
                b_8 = b_16 >> 8;
                a_8 = a_16 >> 8;
            }

            pixel[0] = r_8;
            pixel[1] = g_8;
            pixel[2] = b_8;

            if(fmt == SPNG_FMT_RGBA8) pixel[3] = a_8;
        }
        else if(fmt == SPNG_FMT_RGBA16)
        {
            if(ihdr->bit_depth != 16)
            {
                r_16 = r_8;
                g_16 = g_8;
                b_16 = b_8;
                a_16 = a_8;
            }

            memcpy(pixel, &r_16, 2);
            memcpy(pixel + 2, &g_16, 2);
            memcpy(pixel + 4, &b_16, 2);
        
... [Content truncated - file is very large]
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **spng_plte**: A class/struct defined in this file
- **spng_chrm_int**: A class/struct defined in this file
- **encode_flags**: A class/struct defined in this file
- **decode_flags**: A class/struct defined in this file
- **spng_chunk**: A class/struct defined in this file
- **spng_bkgd**: A class/struct defined in this file
- **spng_ihdr**: A class/struct defined in this file
- **spng_time**: A class/struct defined in this file
- **spng_text2**: A class/struct defined in this file
- **spng_plte_entry**: A class/struct defined in this file
- **spng__zlib_options**: A class/struct defined in this file
- **spng_trns**: A class/struct defined in this file
- **spng_chunk_bitfield**: A class/struct defined in this file
- **spng__iter**: A class/struct defined in this file
- **spng_subimage**: A class/struct defined in this file
- **spng_splt**: A class/struct defined in this file
- **spng_row_info**: A class/struct defined in this file
- **spng_exif**: A class/struct defined in this file
- **spng_ctx**: A class/struct defined in this file
- **spng_unknown_chunk**: A class/struct defined in this file
- **spng_sbit**: A class/struct defined in this file
- **spng_iccp**: A class/struct defined in this file
- **spng_alloc**: A class/struct defined in this file
- **spng_offs**: A class/struct defined in this file
- **spng_phys**: A class/struct defined in this file
- **spng_hist**: A class/struct defined in this file

### Functions and Methods

- **__FRAMAC__()**: A function/method defined in this file
- **void()**: A function/method defined in this file
- **predicts()**: A function/method defined in this file
- **SPNG_MULTITHREADING()**: A function/method defined in this file
- **SPNG_DISABLE_OPT()**: A function/method defined in this file
- **SPNG_USE_MINIZ()**: A function/method defined in this file
- **SPNG_SSE()**: A function/method defined in this file
- **SPNG_TARGET_CLONES()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `immintrin.h`
- `stdio.h`
- `inttypes.h`
- `spng.h`
- `miniz.h`
- `arm64_neon.h`
- `pthread.h`
- `arm_neon.h`
- `limits.h`
- `tests/framac_stubs.h`
- `math.h`
- `string.h`
- `zlib.h`

**Python Imports:**
- `any`
- `the`
- `arm`
- `ihdr`
- `filter_neon_intrinsics.c`
- `filtering`
- `libpng`
- `spng_format`
- `palette_neon_intrinsics.c`


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

