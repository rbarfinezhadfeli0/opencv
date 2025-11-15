# Documentation for `docs/3rdparty/libpng/png.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libpng/png.c_docs.md`
- **File Name**: `png.c_docs.md`
- **File Size**: 107,376 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libpng/png.c_docs.md](../../../docs/3rdparty/libpng/png.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libpng` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libpng/png.c`

## File Metadata

- **Full Path**: `3rdparty/libpng/png.c`
- **File Name**: `png.c`
- **File Size**: 158,715 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libpng/png.c](../../3rdparty/libpng/png.c)

## Purpose and Role

This file is located in the `3rdparty/libpng` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* png.c - location for general purpose libpng functions
 *
 * Copyright (c) 2018-2025 Cosmin Truta
 * Copyright (c) 1998-2002,2004,2006-2018 Glenn Randers-Pehrson
 * Copyright (c) 1996-1997 Andreas Dilger
 * Copyright (c) 1995-1996 Guy Eric Schalnat, Group 42, Inc.
 *
 * This code is released under the libpng license.
 * For conditions of distribution and use, see the disclaimer
 * and license in png.h
 */

#include "pngpriv.h"

/* Generate a compiler error if there is an old png.h in the search path. */
typedef png_libpng_version_1_6_45 Your_png_h_is_not_version_1_6_45;

/* Tells libpng that we have already handled the first "num_bytes" bytes
 * of the PNG file signature.  If the PNG data is embedded into another
 * stream we can set num_bytes = 8 so that libpng will not attempt to read
 * or write any of the magic bytes before it starts on the IHDR.
 */

#ifdef PNG_READ_SUPPORTED
void PNGAPI
png_set_sig_bytes(png_structrp png_ptr, int num_bytes)
{
   unsigned int nb = (unsigned int)num_bytes;

   png_debug(1, "in png_set_sig_bytes");

   if (png_ptr == NULL)
      return;

   if (num_bytes < 0)
      nb = 0;

   if (nb > 8)
      png_error(png_ptr, "Too many bytes for PNG signature");

   png_ptr->sig_bytes = (png_byte)nb;
}

/* Checks whether the supplied bytes match the PNG signature.  We allow
 * checking less than the full 8-byte signature so that those apps that
 * already read the first few bytes of a file to determine the file type
 * can simply check the remaining bytes for extra assurance.  Returns
 * an integer less than, equal to, or greater than zero if sig is found,
 * respectively, to be less than, to match, or be greater than the correct
 * PNG signature (this is the same behavior as strcmp, memcmp, etc).
 */
int PNGAPI
png_sig_cmp(png_const_bytep sig, size_t start, size_t num_to_check)
{
   static const png_byte png_signature[8] = {137, 80, 78, 71, 13, 10, 26, 10};

   if (num_to_check > 8)
      num_to_check = 8;

   else if (num_to_check < 1)
      return -1;

   if (start > 7)
      return -1;

   if (start + num_to_check > 8)
      num_to_check = 8 - start;

   return memcmp(&sig[start], &png_signature[start], num_to_check);
}

#endif /* READ */

#if defined(PNG_READ_SUPPORTED) || defined(PNG_WRITE_SUPPORTED)
/* Function to allocate memory for zlib */
PNG_FUNCTION(voidpf /* PRIVATE */,
png_zalloc,(voidpf png_ptr, uInt items, uInt size),PNG_ALLOCATED)
{
   png_alloc_size_t num_bytes = size;

   if (png_ptr == NULL)
      return NULL;

   if (items >= (~(png_alloc_size_t)0)/size)
   {
      png_warning (png_voidcast(png_structrp, png_ptr),
          "Potential overflow in png_zalloc()");
      return NULL;
   }

   num_bytes *= items;
   return png_malloc_warn(png_voidcast(png_structrp, png_ptr), num_bytes);
}

/* Function to free memory for zlib */
void /* PRIVATE */
png_zfree(voidpf png_ptr, voidpf ptr)
{
   png_free(png_voidcast(png_const_structrp,png_ptr), ptr);
}

/* Reset the CRC variable to 32 bits of 1's.  Care must be taken
 * in case CRC is > 32 bits to leave the top bits 0.
 */
void /* PRIVATE */
png_reset_crc(png_structrp png_ptr)
{
   /* The cast is safe because the crc is a 32-bit value. */
   png_ptr->crc = (png_uint_32)crc32(0, Z_NULL, 0);
}

/* Calculate the CRC over a section of data.  We can only pass as
 * much data to this routine as the largest single buffer size.  We
 * also check that this data will actually be used before going to the
 * trouble of calculating it.
 */
void /* PRIVATE */
png_calculate_crc(png_structrp png_ptr, png_const_bytep ptr, size_t length)
{
   int need_crc = 1;

   if (PNG_CHUNK_ANCILLARY(png_ptr->chunk_name) != 0)
   {
      if ((png_ptr->flags & PNG_FLAG_CRC_ANCILLARY_MASK) ==
          (PNG_FLAG_CRC_ANCILLARY_USE | PNG_FLAG_CRC_ANCILLARY_NOWARN))
         need_crc = 0;
   }

   else /* critical */
   {
      if ((png_ptr->flags & PNG_FLAG_CRC_CRITICAL_IGNORE) != 0)
         need_crc = 0;
   }

   /* 'uLong' is defined in zlib.h as unsigned long; this means that on some
    * systems it is a 64-bit value.  crc32, however, returns 32 bits so the
    * following cast is safe.  'uInt' may be no more than 16 bits, so it is
    * necessary to perform a loop here.
    */
   if (need_crc != 0 && length > 0)
   {
      uLong crc = png_ptr->crc; /* Should never issue a warning */

      do
      {
         uInt safe_length = (uInt)length;
#ifndef __COVERITY__
         if (safe_length == 0)
            safe_length = (uInt)-1; /* evil, but safe */
#endif

         crc = crc32(crc, ptr, safe_length);

         /* The following should never issue compiler warnings; if they do the
          * target system has characteristics that will probably violate other
          * assumptions within the libpng code.
          */
         ptr += safe_length;
         length -= safe_length;
      }
      while (length > 0);

      /* And the following is always safe because the crc is only 32 bits. */
      png_ptr->crc = (png_uint_32)crc;
   }
}

/* Check a user supplied version number, called from both read and write
 * functions that create a png_struct.
 */
int
png_user_version_check(png_structrp png_ptr, png_const_charp user_png_ver)
{
   /* Libpng versions 1.0.0 and later are binary compatible if the version
    * string matches through the second '.'; we must recompile any
    * applications that use any older library version.
    */

   if (user_png_ver != NULL)
   {
      int i = -1;
      int found_dots = 0;

      do
      {
         i++;
         if (user_png_ver[i] != PNG_LIBPNG_VER_STRING[i])
            png_ptr->flags |= PNG_FLAG_LIBRARY_MISMATCH;
         if (user_png_ver[i] == '.')
            found_dots++;
      } while (found_dots < 2 && user_png_ver[i] != 0 &&
            PNG_LIBPNG_VER_STRING[i] != 0);
   }

   else
      png_ptr->flags |= PNG_FLAG_LIBRARY_MISMATCH;

   if ((png_ptr->flags & PNG_FLAG_LIBRARY_MISMATCH) != 0)
   {
#ifdef PNG_WARNINGS_SUPPORTED
      size_t pos = 0;
      char m[128];

      pos = png_safecat(m, (sizeof m), pos,
          "Application built with libpng-");
      pos = png_safecat(m, (sizeof m), pos, user_png_ver);
      pos = png_safecat(m, (sizeof m), pos, " but running with ");
      pos = png_safecat(m, (sizeof m), pos, PNG_LIBPNG_VER_STRING);
      PNG_UNUSED(pos)

      png_warning(png_ptr, m);
#endif

#ifdef PNG_ERROR_NUMBERS_SUPPORTED
      png_ptr->flags = 0;
#endif

      return 0;
   }

   /* Success return. */
   return 1;
}

/* Generic function to create a png_struct for either read or write - this
 * contains the common initialization.
 */
PNG_FUNCTION(png_structp /* PRIVATE */,
png_create_png_struct,(png_const_charp user_png_ver, png_voidp error_ptr,
    png_error_ptr error_fn, png_error_ptr warn_fn, png_voidp mem_ptr,
    png_malloc_ptr malloc_fn, png_free_ptr free_fn),PNG_ALLOCATED)
{
   png_struct create_struct;
#  ifdef PNG_SETJMP_SUPPORTED
      jmp_buf create_jmp_buf;
#  endif

   /* This temporary stack-allocated structure is used to provide a place to
    * build enough context to allow the user provided memory allocator (if any)
    * to be called.
    */
   memset(&create_struct, 0, (sizeof create_struct));

   /* Added at libpng-1.2.6 */
#  ifdef PNG_USER_LIMITS_SUPPORTED
      create_struct.user_width_max = PNG_USER_WIDTH_MAX;
      create_struct.user_height_max = PNG_USER_HEIGHT_MAX;

#     ifdef PNG_USER_CHUNK_CACHE_MAX
      /* Added at libpng-1.2.43 and 1.4.0 */
      create_struct.user_chunk_cache_max = PNG_USER_CHUNK_CACHE_MAX;
#     endif

#     ifdef PNG_USER_CHUNK_MALLOC_MAX
      /* Added at libpng-1.2.43 and 1.4.1, required only for read but exists
       * in png_struct regardless.
       */
      create_struct.user_chunk_malloc_max = PNG_USER_CHUNK_MALLOC_MAX;
#     endif
#  endif

   /* The following two API calls simply set fields in png_struct, so it is safe
    * to do them now even though error handling is not yet set up.
    */
#  ifdef PNG_USER_MEM_SUPPORTED
      png_set_mem_fn(&create_struct, mem_ptr, malloc_fn, free_fn);
#  else
      PNG_UNUSED(mem_ptr)
      PNG_UNUSED(malloc_fn)
      PNG_UNUSED(free_fn)
#  endif

   /* (*error_fn) can return control to the caller after the error_ptr is set,
    * this will result in a memory leak unless the error_fn does something
    * extremely sophisticated.  The design lacks merit but is implicit in the
    * API.
    */
   png_set_error_fn(&create_struct, error_ptr, error_fn, warn_fn);

#  ifdef PNG_SETJMP_SUPPORTED
      if (!setjmp(create_jmp_buf))
#  endif
      {
#  ifdef PNG_SETJMP_SUPPORTED
         /* Temporarily fake out the longjmp information until we have
          * successfully completed this function.  This only works if we have
          * setjmp() support compiled in, but it is safe - this stuff should
          * never happen.
          */
         create_struct.jmp_buf_ptr = &create_jmp_buf;
         create_struct.jmp_buf_size = 0; /*stack allocation*/
         create_struct.longjmp_fn = longjmp;
#  endif
         /* Call the general version checker (shared with read and write code):
          */
         if (png_user_version_check(&create_struct, user_png_ver) != 0)
         {
            png_structrp png_ptr = png_voidcast(png_structrp,
                png_malloc_warn(&create_struct, (sizeof *png_ptr)));

            if (png_ptr != NULL)
            {
               /* png_ptr->zstream holds a back-pointer to the png_struct, so
                * this can only be done now:
                */
               create_struct.zstream.zalloc = png_zalloc;
               create_struct.zstream.zfree = png_zfree;
               create_struct.zstream.opaque = png_ptr;

#              ifdef PNG_SETJMP_SUPPORTED
               /* Eliminate the local error handling: */
               create_struct.jmp_buf_ptr = NULL;
               create_struct.jmp_buf_size = 0;
               create_struct.longjmp_fn = 0;
#              endif

               *png_ptr = create_struct;

               /* This is the successful return point */
               return png_ptr;
            }
         }
      }

   /* A longjmp because of a bug in the application storage allocator or a
    * simple failure to allocate the png_struct.
    */
   return NULL;
}

/* Allocate the memory for an info_struct for the application. */
PNG_FUNCTION(png_infop,PNGAPI
png_create_info_struct,(png_const_structrp png_ptr),PNG_ALLOCATED)
{
   png_inforp info_ptr;

   png_debug(1, "in png_create_info_struct");

   if (png_ptr == NULL)
      return NULL;

   /* Use the internal API that does not (or at least should not) error out, so
    * that this call always returns ok.  The application typically sets up the
    * error handling *after* creating the info_struct because this is the way it
    * has always been done in 'example.c'.
    */
   info_ptr = png_voidcast(png_inforp, png_malloc_base(png_ptr,
       (sizeof *info_ptr)));

   if (info_ptr != NULL)
      memset(info_ptr, 0, (sizeof *info_ptr));

   return info_ptr;
}

/* This function frees the memory associated with a single info struct.
 * Normally, one would use either png_destroy_read_struct() or
 * png_destroy_write_struct() to free an info struct, but this may be
 * useful for some applications.  From libpng 1.6.0 this function is also used
 * internally to implement the png_info release part of the 'struct' destroy
 * APIs.  This ensures that all possible approaches free the same data (all of
 * it).
 */
void PNGAPI
png_destroy_info_struct(png_const_structrp png_ptr, png_infopp info_ptr_ptr)
{
   png_inforp info_ptr = NULL;

   png_debug(1, "in png_destroy_info_struct");

   if (png_ptr == NULL)
      return;

   if (info_ptr_ptr != NULL)
      info_ptr = *info_ptr_ptr;

   if (info_ptr != NULL)
   {
      /* Do this first in case of an error below; if the app implements its own
       * memory management this can lead to png_free calling png_error, which
       * will abort this routine and return control to the app error handler.
       * An infinite loop may result if it then tries to free the same info
       * ptr.
       */
      *info_ptr_ptr = NULL;

      png_free_data(png_ptr, info_ptr, PNG_FREE_ALL, -1);
      memset(info_ptr, 0, (sizeof *info_ptr));
      png_free(png_ptr, info_ptr);
   }
}

/* Initialize the info structure.  This is now an internal function (0.89)
 * and applications using it are urged to use png_create_info_struct()
 * instead.  Use deprecated in 1.6.0, internal use removed (used internally it
 * is just a memset).
 *
 * NOTE: it is almost inconceivable that this API is used because it bypasses
 * the user-memory mechanism and the user error handling/warning mechanisms in
 * those cases where it does anything other than a memset.
 */
PNG_FUNCTION(void,PNGAPI
png_info_init_3,(png_infopp ptr_ptr, size_t png_info_struct_size),
    PNG_DEPRECATED)
{
   png_inforp info_ptr = *ptr_ptr;

   png_debug(1, "in png_info_init_3");

   if (info_ptr == NULL)
      return;

   if ((sizeof (png_info)) > png_info_struct_size)
   {
      *ptr_ptr = NULL;
      /* The following line is why this API should not be used: */
      free(info_ptr);
      info_ptr = png_voidcast(png_inforp, png_malloc_base(NULL,
          (sizeof *info_ptr)));
      if (info_ptr == NULL)
         return;
      *ptr_ptr = info_ptr;
   }

   /* Set everything to 0 */
   memset(info_ptr, 0, (sizeof *info_ptr));
}

void PNGAPI
png_data_freer(png_const_structrp png_ptr, png_inforp info_ptr,
    int freer, png_uint_32 mask)
{
   png_debug(1, "in png_data_freer");

   if (png_ptr == NULL || info_ptr == NULL)
      return;

   if (freer == PNG_DESTROY_WILL_FREE_DATA)
      info_ptr->free_me |= mask;

   else if (freer == PNG_USER_WILL_FREE_DATA)
      info_ptr->free_me &= ~mask;

   else
      png_error(png_ptr, "Unknown freer parameter in png_data_freer");
}

void PNGAPI
png_free_data(png_const_structrp png_ptr, png_inforp info_ptr, png_uint_32 mask,
    int num)
{
   png_debug(1, "in png_free_data");

   if (png_ptr == NULL || info_ptr == NULL)
      return;

#ifdef PNG_TEXT_SUPPORTED
   /* Free text item num or (if num == -1) all text items */
   if (info_ptr->text != NULL &&
       ((mask & PNG_FREE_TEXT) & info_ptr->free_me) != 0)
   {
      if (num != -1)
      {
         png_free(png_ptr, info_ptr->text[num].key);
         info_ptr->text[num].key = NULL;
      }

      else
      {
         int i;

         for (i = 0; i < info_ptr->num_text; i++)
            png_free(png_ptr, info_ptr->text[i].key);

         png_free(png_ptr, info_ptr->text);
         info_ptr->text = NULL;
         info_ptr->num_text = 0;
         info_ptr->max_text = 0;
      }
   }
#endif

#ifdef PNG_tRNS_SUPPORTED
   /* Free any tRNS entry */
   if (((mask & PNG_FREE_TRNS) & info_ptr->free_me) != 0)
   {
      info_ptr->valid &= ~PNG_INFO_tRNS;
      png_free(png_ptr, info_ptr->trans_alpha);
      info_ptr->trans_alpha = NULL;
      info_ptr->num_trans = 0;
   }
#endif

#ifdef PNG_sCAL_SUPPORTED
   /* Free any sCAL entry */
   if (((mask & PNG_FREE_SCAL) & info_ptr->free_me) != 0)
   {
      png_free(png_ptr, info_ptr->scal_s_width);
      png_free(png_ptr, info_ptr->scal_s_height);
      info_ptr->scal_s_width = NULL;
      info_ptr->scal_s_height = NULL;
      info_ptr->valid &= ~PNG_INFO_sCAL;
   }
#endif

#ifdef PNG_pCAL_SUPPORTED
   /* Free any pCAL entry */
   if (((mask & PNG_FREE_PCAL) & info_ptr->free_me) != 0)
   {
      png_free(png_ptr, info_ptr->pcal_purpose);
      png_free(png_ptr, info_ptr->pcal_units);
      info_ptr->pcal_purpose = NULL;
      info_ptr->pcal_units = NULL;

      if (info_ptr->pcal_params != NULL)
         {
            int i;

            for (i = 0; i < info_ptr->pcal_nparams; i++)
               png_free(png_ptr, info_ptr->pcal_params[i]);

            png_free(png_ptr, info_ptr->pcal_params);
            info_ptr->pcal_params = NULL;
         }
      info_ptr->valid &= ~PNG_INFO_pCAL;
   }
#endif

#ifdef PNG_iCCP_SUPPORTED
   /* Free any profile entry */
   if (((mask & PNG_FREE_ICCP) & info_ptr->free_me) != 0)
   {
      png_free(png_ptr, info_ptr->iccp_name);
      png_free(png_ptr, info_ptr->iccp_profile);
      info_ptr->iccp_name = NULL;
      info_ptr->iccp_profile = NULL;
      info_ptr->valid &= ~PNG_INFO_iCCP;
   }
#endif

#ifdef PNG_sPLT_SUPPORTED
   /* Free a given sPLT entry, or (if num == -1) all sPLT entries */
   if (info_ptr->splt_palettes != NULL &&
       ((mask & PNG_FREE_SPLT) & info_ptr->free_me) != 0)
   {
      if (num != -1)
      {
         png_free(png_ptr, info_ptr->splt_palettes[num].name);
         png_free(png_ptr, info_ptr->splt_palettes[num].entries);
         info_ptr->splt_palettes[num].name = NULL;
         info_ptr->splt_palettes[num].entries = NULL;
      }

      else
      {
         int i;

         for (i = 0; i < info_ptr->splt_palettes_num; i++)
         {
            png_free(png_ptr, info_ptr->splt_palettes[i].name);
            png_free(png_ptr, info_ptr->splt_palettes[i].entries);
         }

         png_free(png_ptr, info_ptr->splt_palettes);
         info_ptr->splt_palettes = NULL;
         info_ptr->splt_palettes_num = 0;
         info_ptr->valid &= ~PNG_INFO_sPLT;
      }
   }
#endif

#ifdef PNG_STORE_UNKNOWN_CHUNKS_SUPPORTED
   if (info_ptr->unknown_chunks != NULL &&
       ((mask & PNG_FREE_UNKN) & info_ptr->free_me) != 0)
   {
      if (num != -1)
      {
          png_free(png_ptr, info_ptr->unknown_chunks[num].data);
          info_ptr->unknown_chunks[num].data = NULL;
      }

      else
      {
         int i;

         for (i = 0; i < info_ptr->unknown_chunks_num; i++)
            png_free(png_ptr, info_ptr->unknown_chunks[i].data);

         png_free(png_ptr, info_ptr->unknown_chunks);
         info_ptr->unknown_chunks = NULL;
         info_ptr->unknown_chunks_num = 0;
      }
   }
#endif

#ifdef PNG_eXIf_SUPPORTED
   /* Free any eXIf entry */
   if (((mask & PNG_FREE_EXIF) & info_ptr->free_me) != 0)
   {
# ifdef PNG_READ_eXIf_SUPPORTED
      if (info_ptr->eXIf_buf)
      {
         png_free(png_ptr, info_ptr->eXIf_buf);
         info_ptr->eXIf_buf = NULL;
      }
# endif
      if (info_ptr->exif)
      {
         png_free(png_ptr, info_ptr->exif);
         info_ptr->exif = NULL;
      }
      info_ptr->valid &= ~PNG_INFO_eXIf;
   }
#endif

#ifdef PNG_hIST_SUPPORTED
   /* Free any hIST entry */
   if (((mask & PNG_FREE_HIST) & info_ptr->free_me) != 0)
   {
      png_free(png_ptr, info_ptr->hist);
      info_ptr->hist = NULL;
      info_ptr->valid &= ~PNG_INFO_hIST;
   }
#endif

   /* Free any PLTE entry that was internally allocated */
   if (((mask & PNG_FREE_PLTE) & info_ptr->free_me) != 0)
   {
      png_free(png_ptr, info_ptr->palette);
      info_ptr->palette = NULL;
      info_ptr->valid &= ~PNG_INFO_PLTE;
      info_ptr->num_palette = 0;
   }

#ifdef PNG_INFO_IMAGE_SUPPORTED
   /* Free any image bits attached to the info structure */
   if (((mask & PNG_FREE_ROWS) & info_ptr->free_me) != 0)
   {
      if (info_ptr->row_pointers != NULL)
      {
         png_uint_32 row;
         for (row = 0; row < info_ptr->height; row++)
            png_free(png_ptr, info_ptr->row_pointers[row]);

         png_free(png_ptr, info_ptr->row_pointers);
         info_ptr->row_pointers = NULL;
      }
      info_ptr->valid &= ~PNG_INFO_IDAT;
   }
#endif

   if (num != -1)
      mask &= ~PNG_FREE_MUL;

   info_ptr->free_me &= ~mask;
}
#endif /* READ || WRITE */

/* This function returns a pointer to the io_ptr associated with the user
 * functions.  The application should free any memory associated with this
 * pointer before png_write_destroy() or png_read_destroy() are called.
 */
png_voidp PNGAPI
png_get_io_ptr(png_const_structrp png_ptr)
{
   if (png_ptr == NULL)
      return NULL;

   return png_ptr->io_ptr;
}

#if defined(PNG_READ_SUPPORTED) || defined(PNG_WRITE_SUPPORTED)
#  ifdef PNG_STDIO_SUPPORTED
/* Initialize the default input/output functions for the PNG file.  If you
 * use your own read or write routines, you can call either png_set_read_fn()
 * or png_set_write_fn() instead of png_init_io().  If you have defined
 * PNG_NO_STDIO or otherwise disabled PNG_STDIO_SUPPORTED, you must use a
 * function of your own because "FILE *" isn't necessarily available.
 */
void PNGAPI
png_init_io(png_structrp png_ptr, png_FILE_p fp)
{
   png_debug(1, "in png_init_io");

   if (png_ptr == NULL)
      return;

   png_ptr->io_ptr = (png_voidp)fp;
}
#  endif

#  ifdef PNG_SAVE_INT_32_SUPPORTED
/* PNG signed integers are saved in 32-bit 2's complement format.  ANSI C-90
 * defines a cast of a signed integer to an unsigned integer either to preserve
 * the value, if it is positive, or to calculate:
 *
 *     (UNSIGNED_MAX+1) + integer
 *
 * Where UNSIGNED_MAX is the appropriate maximum unsigned value, so when the
 * negative integral value is added the result will be an unsigned value
 * corresponding to the 2's complement representation.
 */
void PNGAPI
png_save_int_32(png_bytep buf, png_int_32 i)
{
   png_save_uint_32(buf, (png_uint_32)i);
}
#  endif

#  ifdef PNG_TIME_RFC1123_SUPPORTED
/* Convert the supplied time into an RFC 1123 string suitable for use in
 * a "Creation Time" or other text-based time string.
 */
int PNGAPI
png_convert_to_rfc1123_buffer(char out[29], png_const_timep ptime)
{
   static const char short_months[12][4] =
        {"Jan", "Feb", "Mar", "Apr", "May", "Jun",
         "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"};

   if (out == NULL)
      return 0;

   if (ptime->year > 9999 /* RFC1123 limitation */ ||
       ptime->month == 0    ||  ptime->month > 12  ||
       ptime->day   == 0    ||  ptime->day   > 31  ||
       ptime->hour  > 23    ||  ptime->minute > 59 ||
       ptime->second > 60)
      return 0;

   {
      size_t pos = 0;
      char number_buf[5] = {0, 0, 0, 0, 0}; /* enough for a four-digit year */

#     define APPEND_STRING(string) pos = png_safecat(out, 29, pos, (string))
#     define APPEND_NUMBER(format, value)\
         APPEND_STRING(PNG_FORMAT_NUMBER(number_buf, format, (value)))
#     define APPEND(ch) if (pos < 28) out[pos++] = (ch)

      APPEND_NUMBER(PNG_NUMBER_FORMAT_u, (unsigned)ptime->day);
      APPEND(' ');
      APPEND_STRING(short_months[(ptime->month - 1)]);
      APPEND(' ');
      APPEND_NUMBER(PNG_NUMBER_FORMAT_u, ptime->year);
      APPEND(' ');
      APPEND_NUMBER(PNG_NUMBER_FORMAT_02u, (unsigned)ptime->hour);
      APPEND(':');
      APPEND_NUMBER(PNG_NUMBER_FORMAT_02u, (unsigned)ptime->minute);
      APPEND(':');
      APPEND_NUMBER(PNG_NUMBER_FORMAT_02u, (unsigned)ptime->second);
      APPEND_STRING(" +0000"); /* This reliably terminates the buffer */
      PNG_UNUSED (pos)

#     undef APPEND
#     undef APPEND_NUMBER
#     undef APPEND_STRING
   }

   return 1;
}

#    if PNG_LIBPNG_VER < 10700
/* To do: remove the following from libpng-1.7 */
/* Original API that uses a private buffer in png_struct.
 * Deprecated because it causes png_struct to carry a spurious temporary
 * buffer (png_struct::time_buffer), better to have the caller pass this in.
 */
png_const_charp PNGAPI
png_convert_to_rfc1123(png_structrp png_ptr, png_const_timep ptime)
{
   if (png_ptr != NULL)
   {
      /* The only failure above if png_ptr != NULL is from an invalid ptime */
      if (png_convert_to_rfc1123_buffer(png_ptr->time_buffer, ptime) == 0)
         png_warning(png_ptr, "Ignoring invalid time value");

      else
         return png_ptr->time_buffer;
   }

   return NULL;
}
#    endif /* LIBPNG_VER < 10700 */
#  endif /* TIME_RFC1123 */

#endif /* READ || WRITE */

png_const_charp PNGAPI
png_get_copyright(png_const_structrp png_ptr)
{
   PNG_UNUSED(png_ptr)  /* Silence compiler warning about unused png_ptr */
#ifdef PNG_STRING_COPYRIGHT
   return PNG_STRING_COPYRIGHT
#else
   return PNG_STRING_NEWLINE \
      "libpng version 1.6.45" PNG_STRING_NEWLINE \
      "Copyright (c) 2018-2025 Cosmin Truta" PNG_STRING_NEWLINE \
      "Copyright (c) 1998-2002,2004,2006-2018 Glenn Randers-Pehrson" \
      PNG_STRING_NEWLINE \
      "Copyright (c) 1996-1997 Andreas Dilger" PNG_STRING_NEWLINE \
      "Copyright (c) 1995-1996 Guy Eric Schalnat, Group 42, Inc." \
      PNG_STRING_NEWLINE;
#endif
}

/* The following return the library version as a short string in the
 * format 1.0.0 through 99.99.99zz.  To get the version of *.h files
 * used with your application, print out PNG_LIBPNG_VER_STRING, which
 * is defined in png.h.
 * Note: now there is no difference between png_get_libpng_ver() and
 * png_get_header_ver().  Due to the version_nn_nn_nn typedef guard,
 * it is guaranteed that png.c uses the correct version of png.h.
 */
png_const_charp PNGAPI
png_get_libpng_ver(png_const_structrp png_ptr)
{
   /* Version of *.c files used when building libpng */
   return png_get_header_ver(png_ptr);
}

png_const_charp PNGAPI
png_get_header_ver(png_const_structrp png_ptr)
{
   /* Version of *.h files used when building libpng */
   PNG_UNUSED(png_ptr)  /* Silence compiler warning about unused png_ptr */
   return PNG_LIBPNG_VER_STRING;
}

png_const_charp PNGAPI
png_get_header_version(png_const_structrp png_ptr)
{
   /* Returns longer string containing both version and date */
   PNG_UNUSED(png_ptr)  /* Silence compiler warning about unused png_ptr */
#ifdef __STDC__
   return PNG_HEADER_VERSION_STRING
#  ifndef PNG_READ_SUPPORTED
      " (NO READ SUPPORT)"
#  endif
      PNG_STRING_NEWLINE;
#else
   return PNG_HEADER_VERSION_STRING;
#endif
}

#ifdef PNG_BUILD_GRAYSCALE_PALETTE_SUPPORTED
/* NOTE: this routine is not used internally! */
/* Build a grayscale palette.  Palette is assumed to be 1 << bit_depth
 * large of png_color.  This lets grayscale images be treated as
 * paletted.  Most useful for gamma correction and simplification
 * of code.  This API is not used internally.
 */
void PNGAPI
png_build_grayscale_palette(int bit_depth, png_colorp palette)
{
   int num_palette;
   int color_inc;
   int i;
   int v;

   png_debug(1, "in png_do_build_grayscale_palette");

   if (palette == NULL)
      return;

   switch (bit_depth)
   {
      case 1:
         num_palette = 2;
         color_inc = 0xff;
         break;

      case 2:
         num_palette = 4;
         color_inc = 0x55;
         break;

      case 4:
         num_palette = 16;
         color_inc = 0x11;
         break;

      case 8:
         num_palette = 256;
         color_inc = 1;
         break;

      default:
         num_palette = 0;
         color_inc = 0;
         break;
   }

   for (i = 0, v = 0; i < num_palette; i++, v += color_inc)
   {
      palette[i].red = (png_byte)(v & 0xff);
      palette[i].green = (png_byte)(v & 0xff);
      palette[i].blue = (png_byte)(v & 0xff);
   }
}
#endif

#ifdef PNG_SET_UNKNOWN_CHUNKS_SUPPORTED
int PNGAPI
png_handle_as_unknown(png_const_structrp png_ptr, png_const_bytep chunk_name)
{
   /* Check chunk_name and return "keep" value if it's on the list, else 0 */
   png_const_bytep p, p_end;

   if (png_ptr == NULL || chunk_name == NULL || png_ptr->num_chunk_list == 0)
      return PNG_HANDLE_CHUNK_AS_DEFAULT;

   p_end = png_ptr->chunk_list;
   p = p_end + png_ptr->num_chunk_list*5; /* beyond end */

   /* The code is the fifth byte after each four byte string.  Historically this
    * code was always searched from the end of the list, this is no longer
    * necessary because the 'set' routine handles duplicate entries correctly.
    */
   do /* num_chunk_list > 0, so at least one */
   {
      p -= 5;

      if (memcmp(chunk_name, p, 4) == 0)
         return p[4];
   }
   while (p > p_end);

   /* This means that known chunks should be processed and unknown chunks should
    * be handled according to the value of png_ptr->unknown_default; this can be
    * confusing because, as a result, there are two levels of defaulting for
    * unknown chunks.
    */
   return PNG_HANDLE_CHUNK_AS_DEFAULT;
}

#if defined(PNG_READ_UNKNOWN_CHUNKS_SUPPORTED) ||\
   defined(PNG_HANDLE_AS_UNKNOWN_SUPPORTED)
int /* PRIVATE */
png_chunk_unknown_handling(png_const_structrp png_ptr, png_uint_32 chunk_name)
{
   png_byte chunk_string[5];

   PNG_CSTRING_FROM_CHUNK(chunk_string, chunk_name);
   return png_handle_as_unknown(png_ptr, chunk_string);
}
#endif /* READ_UNKNOWN_CHUNKS || HANDLE_AS_UNKNOWN */
#endif /* SET_UNKNOWN_CHUNKS */

#ifdef PNG_READ_SUPPORTED
/* This function, added to libpng-1.0.6g, is untested. */
int PNGAPI
png_reset_zstream(png_structrp png_ptr)
{
   if (png_ptr == NULL)
      return Z_STREAM_ERROR;

   /* WARNING: this resets the window bits to the maximum! */
   return inflateReset(&png_ptr->zstream);
}
#endif /* READ */

/* This function was added to libpng-1.0.7 */
png_uint_32 PNGAPI
png_access_version_number(void)
{
   /* Version of *.c files used when building libpng */
   return (png_uint_32)PNG_LIBPNG_VER;
}

#if defined(PNG_READ_SUPPORTED) || defined(PNG_WRITE_SUPPORTED)
/* Ensure that png_ptr->zstream.msg holds some appropriate error message string.
 * If it doesn't 'ret' is used to set it to something appropriate, even in cases
 * like Z_OK or Z_STREAM_END where the error code is apparently a success code.
 */
void /* PRIVATE */
png_zstream_error(png_structrp png_ptr, int ret)
{
   /* Translate 'ret' into an appropriate error string, priority is given to the
    * one in zstream if set.  This always returns a string, even in cases like
    * Z_OK or Z_STREAM_END where the error code is a success code.
    */
   if (png_ptr->zstream.msg == NULL) switch (ret)
   {
      default:
      case Z_OK:
         png_ptr->zstream.msg = PNGZ_MSG_CAST("unexpected zlib return code");
         break;

      case Z_STREAM_END:
         /* Normal exit */
         png_ptr->zstream.msg = PNGZ_MSG_CAST("unexpected end of LZ stream");
         break;

      case Z_NEED_DICT:
         /* This means the deflate stream did not have a dictionary; this
          * indicates a bogus PNG.
          */
         png_ptr->zstream.msg = PNGZ_MSG_CAST("missing LZ dictionary");
         break;

      case Z_ERRNO:
         /* gz APIs only: should not happen */
         png_ptr->zstream.msg = PNGZ_MSG_CAST("zlib IO error");
         break;

      case Z_STREAM_ERROR:
         /* internal libpng error */
         png_ptr->zstream.msg = PNGZ_MSG_CAST("bad parameters to zlib");
         break;

      case Z_DATA_ERROR:
         png_ptr->zstream.msg = PNGZ_MSG_CAST("damaged LZ stream");
         break;

      case Z_MEM_ERROR:
         png_ptr->zstream.msg = PNGZ_MSG_CAST("insufficient memory");
         break;

      case Z_BUF_ERROR:
         /* End of input or output; not a problem if the caller is doing
          * incremental read or write.
          */
         png_ptr->zstream.msg = PNGZ_MSG_CAST("truncated");
         break;

      case Z_VERSION_ERROR:
         png_ptr->zstream.msg = PNGZ_MSG_CAST("unsupported zlib version");
         break;

      case PNG_UNEXPECTED_ZLIB_RETURN:
         /* Compile errors here mean that zlib now uses the value co-opted in
          * pngpriv.h for PNG_UNEXPECTED_ZLIB_RETURN; update the switch above
          * and change pngpriv.h.  Note that this message is "... return",
          * whereas the default/Z_OK one is "... return code".
          */
         png_ptr->zstream.msg = PNGZ_MSG_CAST("unexpected zlib return");
         break;
   }
}

/* png_convert_size: a PNGAPI but no longer in png.h, so deleted
 * at libpng 1.5.5!
 */

/* Added at libpng version 1.2.34 and 1.4.0 (moved from pngset.c) */
#ifdef PNG_GAMMA_SUPPORTED /* always set if COLORSPACE */
static int
png_colorspace_check_gamma(png_const_structrp png_ptr,
    png_colorspacerp colorspace, png_fixed_point gAMA, int from)
   /* This is called to check a new gamma value against an existing one.  The
    * routine returns false if the new gamma value should not be written.
    *
    * 'from' says where the new gamma value comes from:
    *
    *    0: the new gamma value is the libpng estimate for an ICC profile
    *    1: the new gamma value comes from a gAMA chunk
    *    2: the new gamma value comes from an sRGB chunk
    */
{
   png_fixed_point gtest;

   if ((colorspace->flags & PNG_COLORSPACE_HAVE_GAMMA) != 0 &&
       (png_muldiv(&gtest, colorspace->gamma, PNG_FP_1, gAMA) == 0  ||
      png_gamma_significant(gtest) != 0))
   {
      /* Either this is an sRGB image, in which case the calculated gamma
       * approximation should match, or this is an image with a profile and the
       * value libpng calculates for the gamma of the profile does not match the
       * value recorded in the file.  The former, sRGB, case is an error, the
       * latter is just a warning.
       */
      if ((colorspace->flags & PNG_COLORSPACE_FROM_sRGB) != 0 || from == 2)
      {
         png_chunk_report(png_ptr, "gamma value does not match sRGB",
             PNG_CHUNK_ERROR);
         /* Do not overwrite an sRGB value */
         return from == 2;
      }

      else /* sRGB tag not involved */
      {
         png_chunk_report(png_ptr, "gamma value does not match libpng estimate",
             PNG_CHUNK_WARNING);
         return from == 1;
      }
   }

   return 1;
}

void /* PRIVATE */
png_colorspace_set_gamma(png_const_structrp png_ptr,
    png_colorspacerp colorspace, png_fixed_point gAMA)
{
   /* Changed in libpng-1.5.4 to limit the values to ensure overflow can't
    * occur.  Since the fixed point representation is asymmetrical it is
    * possible for 1/gamma to overflow the limit of 21474 and this means the
    * gamma value must be at least 5/100000 and hence at most 20000.0.  For
    * safety the limits here are a little narrower.  The values are 0.00016 to
    * 6250.0, which are truly ridiculous gamma values (and will produce
    * displays that are all black or all white.)
    *
    * In 1.6.0 this test replaces the ones in pngrutil.c, in the gAMA chunk
    * handling code, which only required the value to be >0.
    */
   png_const_charp errmsg;

   if (gAMA < 16 || gAMA > 625000000)
      errmsg = "gamma value out of range";

#  ifdef PNG_READ_gAMA_SUPPORTED
   /* Allow the application to set the gamma value more than once */
   else if ((png_ptr->mode & PNG_IS_READ_STRUCT) != 0 &&
      (colorspace->flags & PNG_COLORSPACE_FROM_gAMA) != 0)
      errmsg = "duplicate";
#  endif

   /* Do nothing if the colorspace is already invalid */
   else if ((colorspace->flags & PNG_COLORSPACE_INVALID) != 0)
      return;

   else
   {
      if (png_colorspace_check_gamma(png_ptr, colorspace, gAMA,
          1/*from gAMA*/) != 0)
      {
         /* Store this gamma value. */
         colorspace->gamma = gAMA;
         colorspace->flags |=
            (PNG_COLORSPACE_HAVE_GAMMA | PNG_COLORSPACE_FROM_gAMA);
      }

      /* At present if the check_gamma test fails the gamma of the colorspace is
       * not updated however the colorspace is not invalidated.  This
       * corresponds to the case where the existing gamma comes from an sRGB
       * chunk or profile.  An error message has already been output.
       */
      return;
   }

   /* Error exit - errmsg has been set. */
   colorspace->flags |= PNG_COLORSPACE_INVALID;
   png_chunk_report(png_ptr, errmsg, PNG_CHUNK_WRITE_ERROR);
}

void /* PRIVATE */
png_colorspace_sync_info(png_const_structrp png_ptr, png_inforp info_ptr)
{
   if ((info_ptr->colorspace.flags & PNG_COLORSPACE_INVALID) != 0)
   {
      /* Everything is invalid */
      info_ptr->valid &= ~(PNG_INFO_gAMA|PNG_INFO_cHRM|PNG_INFO_sRGB|
         PNG_INFO_iCCP);

#     ifdef PNG_COLORSPACE_SUPPORTED
      /* Clean up the iCCP profile now if it won't be used. */
      png_free_data(png_ptr, info_ptr, PNG_FREE_ICCP, -1/*not used*/);
#     else
      PNG_UNUSED(png_ptr)
#     endif
   }

   else
   {
#     ifdef PNG_COLORSPACE_SUPPORTED
      /* Leave the INFO_iCCP flag set if the pngset.c code has already set
       * it; this allows a PNG to contain a profile which matches sRGB and
       * yet still have that profile retrievable by the application.
       */
      if ((info_ptr->colorspace.flags & PNG_COLORSPACE_MATCHES_sRGB) != 0)
         info_ptr->valid |= PNG_INFO_sRGB;

      else
         info_ptr->valid &= ~PNG_INFO_sRGB;

      if ((info_ptr->colorspace.flags & PNG_COLORSPACE_HAVE_ENDPOINTS) != 0)
         info_ptr->valid |= PNG_INFO_cHRM;

      else
         info_ptr->valid &= ~PNG_INFO_cHRM;
#     endif

      if ((info_ptr->colorspace.flags & PNG_COLORSPACE_HAVE_GAMMA) != 0)
         info_ptr->valid |= PNG_INFO_gAMA;

      else
         info_ptr->valid &= ~PNG_INFO_gAMA;
   }
}

#ifdef PNG_READ_SUPPORTED
void /* PRIVATE */
png_colorspace_sync(png_const_structrp png_ptr, png_inforp info_ptr)
{
   if (info_ptr == NULL) /* reduce code size; check here not in the caller */
      return;

   info_ptr->colorspace = png_ptr->colorspace;
   png_colorspace_sync_info(png_ptr, info_ptr);
}
#endif
#endif /* GAMMA */

#ifdef PNG_COLORSPACE_SUPPORTED
static png_int_32
png_fp_add(png_int_32 addend0, png_int_32 addend1, int *error)
{
   /* Safely add two fixed point values setting an error flag and returning 0.5
    * on overflow.
    * IMPLEMENTATION NOTE: ANSI requires signed overflow not to occur, therefore
    * relying on addition of two positive values producing a negative one is not
    * safe.
    */
   if (addend0 > 0)
   {
      if (0x7fffffff - addend0 >= addend1)
         return addend0+addend1;
   }
   else if (addend0 < 0)
   {
      if (-0x7fffffff - addend0 <= addend1)
         return addend0+addend1;
   }
   else
      return addend1;

   *error = 1;
   return PNG_FP_1/2;
}

static png_int_32
png_fp_sub(png_int_32 addend0, png_int_32 addend1, int *error)
{
   /* As above but calculate addend0-addend1. */
   if (addend1 > 0)
   {
      if (-0x7fffffff + addend1 <= addend0)
         return addend0-addend1;
   }
   else if (addend1 < 0)
   {
      if (0x7fffffff + addend1 >= addend0)
         return addend0-addend1;
   }
   else
      return addend0;

   *error = 1;
   return PNG_FP_1/2;
}

static int
png_safe_add(png_int_32 *addend0_and_result, png_int_32 addend1,
      png_int_32 addend2)
{
   /* Safely add three integers.  Returns 0 on success, 1 on overflow.  Does not
    * set the result on overflow.
    */
   int error = 0;
   int result = png_fp_add(*addend0_and_result,
                           png_fp_add(addend1, addend2, &error),
                           &error);
   if (!error) *addend0_and_result = result;
   return error;
}

/* Added at libpng-1.5.5 to support read and write of true CIEXYZ values for
 * cHRM, as opposed to using chromaticities.  These internal APIs return
 * non-zero on a parameter error.  The X, Y and Z values are required to be
 * positive and less than 1.0.
 */
static int
png_xy_from_XYZ(png_xy *xy, const png_XYZ *XYZ)
{
   png_int_32 d, dred, dgreen, dblue, dwhite, whiteX, whiteY;

   /* 'd' in each of the blocks below is just X+Y+Z for each component,
    * x, y and z are X,Y,Z/(X+Y+Z).
    */
   d = XYZ->red_X;
   if (png_safe_add(&d, XYZ->red_Y, XYZ->red_Z))
      return 1;
   dred = d;
   if (png_muldiv(&xy->redx, XYZ->red_X, PNG_FP_1, dred) == 0)
      return 1;
   if (png_muldiv(&xy->redy, XYZ->red_Y, PNG_FP_1, dred) == 0)
      return 1;

   d = XYZ->green_X;
   if (png_safe_add(&d, XYZ->green_Y, XYZ->green_Z))
      return 1;
   dgreen = d;
   if (png_muldiv(&xy->greenx, XYZ->green_X, PNG_FP_1, dgreen) == 0)
      return 1;
   if (png_muldiv(&xy->greeny, XYZ->green_Y, PNG_FP_1, dgreen) == 0)
      return 1;

   d = XYZ->blue_X;
   if (png_safe_add(&d, XYZ->blue_Y, XYZ->blue_Z))
      return 1;
   dblue = d;
   if (png_muldiv(&xy->bluex, XYZ->blue_X, PNG_FP_1, dblue) == 0)
      return 1;
   if (png_muldiv(&xy->bluey, XYZ->blue_Y, PNG_FP_1, dblue) == 0)
      return 1;

   /* The reference white is simply the sum of the end-point (X,Y,Z) vectors so
    * the fillowing calculates (X+Y+Z) of the reference white (media white,
    * encoding white) itself:
    */
   d = dblue;
   if (png_safe_add(&d, dred, dgreen))
      return 1;
   dwhite = d;

   /* Find the white X,Y values from the sum of the red, green and blue X,Y
    * values.
    */
   d = XYZ->red_X;
   if (png_safe_add(&d, XYZ->green_X, XYZ->blue_X))
      return 1;
   whiteX = d;

   d = XYZ->red_Y;
   if (png_safe_add(&d, XYZ->green_Y, XYZ->blue_Y))
      return 1;
   whiteY = d;

   if (png_muldiv(&xy->whitex, whiteX, PNG_FP_1, dwhite) == 0)
      return 1;
   if (png_muldiv(&xy->whitey, whiteY, PNG_FP_1, dwhite) == 0)
      return 1;

   return 0;
}

static int
png_XYZ_from_xy(png_XYZ *XYZ, const png_xy *xy)
{
   png_fixed_point red_inverse, green_inverse, blue_scale;
   png_fixed_point left, right, denominator;

   /* Check xy and, implicitly, z.  Note that wide gamut color spaces typically
    * have end points with 0 tristimulus values (these are impossible end
    * points, but they are used to cover the possible colors).  We check
    * xy->whitey against 5, not 0, to avoid a possible integer overflow.
    *
    * The limits here will *not* accept ACES AP0, where bluey is -7700
    * (-0.0770) because the PNG spec itself requires the xy values to be
    * unsigned.  whitey is also required to be 5 or more to avoid overflow.
    *
    * Instead the upper limits have been relaxed to accomodate ACES AP1 where
    * redz ends up as -600 (-0.006).  ProPhotoRGB was already "in range."
    * The new limit accomodates the AP0 and AP1 ranges for z but not AP0 redy.
    */
   const png_fixed_point fpLimit = PNG_FP_1+(PNG_FP_1/10);
   if (xy->redx   < 0 || xy->redx > fpLimit) return 1;
   if (xy->redy   < 0 || xy->redy > fpLimit-xy->redx) return 1;
   if (xy->greenx < 0 || xy->greenx > fpLimit) return 1;
   if (xy->greeny < 0 || xy->greeny > fpLimit-xy->greenx) return 1;
   if (xy->bluex  < 0 || xy->bluex > fpLimit) return 1;
   if (xy->bluey  < 0 || xy->bluey > fpLimit-xy->bluex) return 1;
   if (xy->whitex < 0 || xy->whitex > fpLimit) return 1;
   if (xy->whitey < 5 || xy->whitey > fpLimit-xy->whitex) return 1;

   /* The reverse calculation is more difficult because the original tristimulus
    * value had 9 independent values (red,green,blue)x(X,Y,Z) however only 8
    * derived values were recorded in the cHRM chunk;
    * (red,green,blue,white)x(x,y).  This loses one degree of freedom and
    * therefore an arbitrary ninth value has to be introduced to undo the
    * original transformations.
    *
    * Think of the original end-points as points in (X,Y,Z) space.  The
    * chromaticity values (c) have the property:
    *
    *           C
    *   c = ---------
    *       X + Y + Z
    *
    * For each c (x,y,z) from the corresponding original C (X,Y,Z).  Thus the
    * three chromaticity values (x,y,z) for each end-point obey the
    * relationship:
    *
    *   x + y + z = 1
    *
    * This describes the plane in (X,Y,Z) space that intersects each axis at the
    * value 1.0; call this the chromaticity plane.  Thus the chromaticity
    * calculation has scaled each end-point so that it is on the x+y+z=1 plane
    * and chromaticity is the intersection of the vector from the origin to the
    * (X,Y,Z) value with the chromaticity plane.
    *
    * To fully invert the chromaticity calculation we would need the three
    * end-point scale factors, (red-scale, green-scale, blue-scale), but these
    * were not recorded.  Instead we calculated the reference white (X,Y,Z) and
    * recorded the chromaticity of this.  The reference white (X,Y,Z) would have
    * given all three of the scale factors since:
    *
    *    color-C = color-c * color-scale
    *    white-C = red-C + green-C + blue-C
    *            = red-c*red-scale + green-c*green-scale + blue-c*blue-scale
    *
    * But cHRM records only white-x and white-y, so we have lost the white scale
    * factor:
    *
    *    white-C = white-c*white-scale
    *
    * To handle this the inverse transformation makes an arbitrary assumption
    * about white-scale:
    *
    *    Assume: white-Y = 1.0
    *    Hence:  white-scale = 1/white-y
    *    Or:     red-Y + green-Y + blue-Y = 1.0
    *
    * Notice the last statement of the assumption gives an equation in three of
    * the nine values we want to calculate.  8 more equations come from the
    * above routine as summarised at the top above (the chromaticity
    * calculation):
    *
    *    Given: color-x = color-X / (color-X + color-Y + color-Z)
    *    Hence: (color-x - 1)*color-X + color.x*color-Y + color.x*color-Z = 0
    *
    * This is 9 simultaneous equations in the 9 variables "color-C" and can be
    * solved by Cramer's rule.  Cramer's rule requires calculating 10 9x9 matrix
    * determinants, however this is not as bad as it seems because only 28 of
    * the total of 90 terms in the various matrices are non-zero.  Nevertheless
    * Cramer's rule is notoriously numerically unstable because the determinant
    * calculation involves the difference of large, but similar, numbers.  It is
    * difficult to be sure that the calculation is stable for real world values
    * and it is certain that it becomes unstable where the end points are close
    * together.
    *
    * So this code uses the perhaps slightly less optimal but more
    * understandable and totally obvious approach of calculating color-scale.
    *
    * This algorithm depends on the precision in white-scale and that is
    * (1/white-y), so we can immediately see that as white-y approaches 0 the
    * accuracy inherent in the cHRM chunk drops off substantially.
    *
    * libpng arithmetic: a simple inversion of the above equations
    * ------------------------------------------------------------
    *
    *    white_scale = 1/white-y
    *    white-X = white-x * white-scale
    *    white-Y = 1.0
    *    white-Z = (1 - white-x - white-y) * white_scale
    *
    *    white-C = red-C + green-C + blue-C
    *            = red-c*red-scale + green-c*green-scale + blue-c*blue-scale
    *
    * This gives us three equations in (red-scale,green-scale,blue-scale) where
    * all the coefficients are now known:
    *
    *    red-x*red-scale + green-x*green-scale + blue-x*blue-scale
    *       = white-x/white-y
    *    red-y*red-scale + green-y*green-scale + blue-y*blue-scale = 1
    *    red-z*red-scale + green-z*green-scale + blue-z*blue-scale
    *       = (1 - white-x - white-y)/white-y
    *
    * In the last equation color-z is (1 - color-x - color-y) so we can add all
    * three equations together to get an alternative third:
    *
    *    red-scale + green-scale + blue-scale = 1/white-y = white-scale
    *
    * So now we have a Cramer's rule solution where the determinants are just
    * 3x3 - far more tractible.  Unfortunately 3x3 determinants still involve
    * multiplication of three coefficients so we can't guarantee to avoid
    * overflow in the libpng fixed point representation.  Using Cramer's rule in
    * floating point is probably a good choice here, but it's not an option for
    * fixed point.  Instead proceed to simplify the first two equations by
    * eliminating what is likely to be the largest value, blue-scale:
    *
    *    blue-scale = white-scale - red-scale - green-scale
    *
    * Hence:
    *
    *    (red-x - blue-x)*red-scale + (green-x - blue-x)*green-scale =
    *                (white-x - blue-x)*white-scale
    *
    *    (red-y - blue-y)*red-scale + (green-y - blue-y)*green-scale =
    *                1 - blue-y*white-scale
    *
    * And now we can trivially solve for (red-scale,green-scale):
    *
    *    green-scale =
    *                (white-x - blue-x)*white-scale - (red-x - blue-x)*red-scale
    *                -----------------------------------------------------------
    *                                  green-x - blue-x
    *
    *    red-scale =
    *                1 - blue-y*white-scale - (green-y - blue-y) * green-scale
    *                ---------------------------------------------------------
    *                                  red-y - blue-y
    *
    * Hence:
    *
    *    red-scale =
    *          ( (green-x - blue-x) * (white-y - blue-y) -
    *            (green-y - blue-y) * (white-x - blue-x) ) / white-y
    * -------------------------------------------------------------------------
    *  (green-x - blue-x)*(red-y - blue-y)-(green-y - blue-y)*(red-x - blue-x)
    *
    *    green-scale =
    *          ( (red-y - blue-y) * (white-x - blue-x) -
    *            (red-x - blue-x) * (white-y - blue-y) ) / white-y
    * -------------------------------------------------------------------------
    *  (green-x - blue-x)*(red-y - blue-y)-(green-y - blue-y)*(red-x - blue-x)
    *
    * Accuracy:
    * The input values have 5 decimal digits of accuracy.
    *
    * In the previous implementation the values were all in the range 0 < value
    * < 1, so simple products are in the same range but may need up to 10
    * decimal digits to preserve the original precision and avoid underflow.
    * Because we are using a 32-bit signed representation we cannot match this;
    * the best is a little over 9 decimal digits, less than 10.
    *
    * This range has now been extended to allow values up to 1.1, or 110,000 in
    * fixed point.
    *
    * The approach used here is to preserve the maximum precision within the
    * signed representation.  Because the red-scale calculation above uses the
    * difference between two products of values that must be in the range
    * -1.1..+1.1 it is sufficient to divide the product by 8;
    * ceil(121,000/32767*2).  The factor is irrelevant in the calculation
    * because it is applied to both numerator and denominator.
    *
    * Note that the values of the differences of the products of the
    * chromaticities in the above equations tend to be small, for example for
    * the sRGB chromaticities they are:
    *
    * red numerator:    -0.04751
    * green numerator:  -0.08788
    * denominator:      -0.2241 (without white-y multiplication)
    *
    *  The resultant Y coefficients from the chromaticities of some widely used
    *  color space definitions are (to 15 decimal places):
    *
    *  sRGB
    *    0.212639005871510 0.715168678767756 0.072192315360734
    *  Kodak ProPhoto
    *    0.288071128229293 0.711843217810102 0.000085653960605
    *  Adobe RGB
    *    0.297344975250536 0.627363566255466 0.075291458493998
    *  Adobe Wide Gamut RGB
    *    0.258728243040113 0.724682314948566 0.016589442011321
    */
   int error = 0;

   /* By the argument above overflow should be impossible here, however the
    * code now simply returns a failure code.  The xy subtracts in the arguments
    * to png_muldiv are *not* checked for overflow because the checks at the
    * start guarantee they are in the range 0..110000 and png_fixed_point is a
    * 32-bit signed number.
    */
   if (png_muldiv(&left, xy->greenx-xy->bluex, xy->redy - xy->bluey, 8) == 0)
      return 1;
   if (png_muldiv(&right, xy->greeny-xy->bluey, xy->redx - xy->bluex, 8) == 0)
      return 1;
   denominator = png_fp_sub(left, right, &error);
   if (error) return 1;

   /* Now find the red numerator. */
   if (png_muldiv(&left, xy->greenx-xy->bluex, xy->whitey-xy->bluey, 8) == 0)
      return 1;
   if (png_muldiv(&right, xy->greeny-xy->bluey, xy->whitex-xy->bluex, 8) == 0)
      return 1;

   /* Overflow is possible here and it indicates an extreme set of PNG cHRM
    * chunk values.  This calculation actually returns the reciprocal of the
    * scale value because this allows us to delay the multiplication of white-y
    * into the denominator, which tends to produce a small number.
    */
   if (png_muldiv(&red_inverse, xy->whitey, denominator,
                  png_fp_sub(left, right, &error)) == 0 || error ||
       red_inverse <= xy->whitey /* r+g+b scales = white scale */)
      return 1;

   /* Similarly for green_inverse: */
   if (png_muldiv(&left, xy->redy-xy->bluey, xy->whitex-xy->bluex, 8) == 0)
      return 1;
   if (png_muldiv(&right, xy->redx-xy->bluex, xy->whitey-xy->bluey, 8) == 0)
      return 1;
   if (png_muldiv(&green_inverse, xy->whitey, denominator,
                  png_fp_sub(left, right, &error)) == 0 || error ||
       green_inverse <= xy->whitey)
      return 1;

   /* And the blue scale, the checks above guarantee this can't overflow but it
    * can still produce 0 for extreme cHRM values.
    */
   blue_scale = png_fp_sub(png_fp_sub(png_reciprocal(xy->whitey),
                                      png_reciprocal(red_inverse), &error),
                           png_reciprocal(green_inverse), &error);
   if (error || blue_scale <= 0)
      return 1;


   /* And fill in the png_XYZ.  Again the subtracts are safe because of the
    * checks on the xy values at the start (the subtracts just calculate the
    * corresponding z values.)
    */
   if (png_muldiv(&XYZ->red_X, xy->redx, PNG_FP_1, red_inverse) == 0)
      return 1;
   if (png_muldiv(&XYZ->red_Y, xy->redy, PNG_FP_1, red_inverse) == 0)
      return 1;
   if (png_muldiv(&XYZ->red_Z, PNG_FP_1 - xy->redx - xy->redy, PNG_FP_1,
       red_inverse) == 0)
      return 1;

   if (png_muldiv(&XYZ->green_X, xy->greenx, PNG_FP_1, green_inverse) == 0)
      return 1;
   if (png_muldiv(&XYZ->green_Y, xy->greeny, PNG_FP_1, green_inverse) == 0)
      return 1;
   if (png_muldiv(&XYZ->green_Z, PNG_FP_1 - xy->greenx - xy->greeny, PNG_FP_1,
       green_inverse) == 0)
      return 1;

   if (png_muldiv(&XYZ->blue_X, xy->bluex, blue_scale, PNG_FP_1) == 0)
      return 1;
   if (png_muldiv(&XYZ->blue_Y, xy->bluey, blue_scale, PNG_FP_1) == 0)
      return 1;
   if (png_muldiv(&XYZ->blue_Z, PNG_FP_1 - xy->bluex - xy->bluey, blue_scale,
       PNG_FP_1) == 0)
      return 1;

   return 0; /*success*/
}

static int
png_XYZ_normalize(png_XYZ *XYZ)
{
   png_int_32 Y, Ytemp;

   /* Normalize by scaling so the sum of the end-point Y values is PNG_FP_1. */
   Ytemp = XYZ->red_Y;
   if (png_safe_add(&Ytemp, XYZ->green_Y, XYZ->blue_Y))
      return 1;

   Y = Ytemp;

   if (Y != PNG_FP_1)
   {
      if (png_muldiv(&XYZ->red_X, XYZ->red_X, PNG_FP_1, Y) == 0)
         return 1;
      if (png_muldiv(&XYZ->red_Y, XYZ->red_Y, PNG_FP_1, Y) == 0)
         return 1;
      if (png_muldiv(&XYZ->red_Z, XYZ->red_Z, PNG_FP_1, Y) == 0)
         return 1;

      if (png_muldiv(&XYZ->green_X, XYZ->green_X, PNG_FP_1, Y) == 0)
         return 1;
      if (png_muldiv(&XYZ->green_Y, XYZ->green_Y, PNG_FP_1, Y) == 0)
         return 1;
      if (png_muldiv(&XYZ->green_Z, XYZ->green_Z, PNG_FP_1, Y) == 0)
         return 1;

      if (png_muldiv(&XYZ->blue_X, XYZ->blue_X, PNG_FP_1, Y) == 0)
         return 1;
      if (png_muldiv(&XYZ->blue_Y, XYZ->blue_Y, PNG_FP_1, Y) == 0)
         return 1;
      if (png_muldiv(&XYZ->blue_Z, XYZ->blue_Z, PNG_FP_1, Y) == 0)
         return 1;
   }

   return 0;
}

static int
png_colorspace_endpoints_match(const png_xy *xy1, const png_xy *xy2, int delta)
{
   /* Allow an error of +/-0.01 (absolute value) on each chromaticity */
   if (PNG_OUT_OF_RANGE(xy1->whitex, xy2->whitex,delta) ||
       PNG_OUT_OF_RANGE(xy1->whitey, xy2->whitey,delta) ||
       PNG_OUT_OF_RANGE(xy1->redx,   xy2->redx,  delta) ||
       PNG_OUT_OF_RANGE(xy1->redy,   xy2->redy,  delta) ||
       PNG_OUT_OF_RANGE(xy1->greenx, xy2->greenx,delta) ||
       PNG_OUT_OF_RANGE(xy1->greeny, xy2->greeny,delta) ||
       PNG_OUT_OF_RANGE(xy1->bluex,  xy2->bluex, delta) ||
       PNG_OUT_OF_RANGE(xy1->bluey,  xy2->bluey, delta))
      return 0;
   return 1;
}

/* Added in libpng-1.6.0, a different check for the validity of a set of cHRM
 * chunk chromaticities.  Earlier checks used to simply look for the overflow
 * condition (where the determinant of the matrix to solve for XYZ ends up zero
 * because the chromaticity values are not all distinct.)  Despite this it is
 * theoretically possible to produce chromaticities that are apparently valid
 * but that rapidly degrade to invalid, potentially crashing, sets because of
 * arithmetic inaccuracies when calculations are performed on them.  The new
 * check is to round-trip xy -> XYZ -> xy and then check that the result is
 * within a small percentage of the original.
 */
static int
png_colorspace_check_xy(png_XYZ *XYZ, const png_xy *xy)
{
   int result;
   png_xy xy_test;

   /* As a side-effect this routine also returns the XYZ endpoints. */
   result = png_XYZ_from_xy(XYZ, xy);
   if (result != 0)
      return result;

   result = png_xy_from_XYZ(&xy_test, XYZ);
   if (result != 0)
      return result;

   if (png_colorspace_endpoints_match(xy, &xy_test,
       5/*actually, the math is pretty accurate*/) != 0)
      return 0;

   /* Too much slip */
   return 1;
}

/* This is the check going the other way.  The XYZ is modified to normalize it
 * (another side-effect) and the xy chromaticities are returned.
 */
static int
png_colorspace_check_XYZ(png_xy *xy, png_XYZ *XYZ)
{
   int result;
   png_XYZ XYZtemp;

   result = png_XYZ_normalize(XYZ);
   if (result != 0)
      return result;

   result = png_xy_from_XYZ(xy, XYZ);
   if (result != 0)
      return result;

   XYZtemp = *XYZ;
   return png_colorspace_check_xy(&XYZtemp, xy);
}

/* Used to check for an endpoint match against sRGB */
static const png_xy sRGB_xy = /* From ITU-R BT.709-3 */
{
   /* color      x       y */
   /* red   */ 64000, 33000,
   /* green */ 30000, 60000,
   /* blue  */ 15000,  6000,
   /* white */ 31270, 32900
};

static int
png_colorspace_set_xy_and_XYZ(png_const_structrp png_ptr,
    png_colorspacerp colorspace, const png_xy *xy, const png_XYZ *XYZ,
    int preferred)
{
   if ((colorspace->flags & PNG_COLORSPACE_INVALID) != 0)
      return 0;

   /* The consistency check is performed on the chromaticities; this factors out
    * variations because of the normalization (or not) of the end point Y
    * values.
    */
   if (preferred < 2 &&
       (colorspace->flags & PNG_COLORSPACE_HAVE_ENDPOINTS) != 0)
   {
      /* The end points must be reasonably close to any we already have.  The
       * following allows an error of up to +/-.001
       */
      if (png_colorspace_endpoints_match(xy, &colorspace->end_points_xy,
          100) == 0)
      {
         colorspace->flags |= PNG_COLORSPACE_INVALID;
         png_benign_error(png_ptr, "inconsistent chromaticities");
         return 0; /* failed */
      }

      /* Only overwrite with preferred values */
      if (preferred == 0)
         return 1; /* ok, but no change */
   }

   colorspace->end_points_xy = *xy;
   colorspace->end_points_XYZ = *XYZ;
   colorspace->flags |= PNG_COLORSPACE_HAVE_ENDPOINTS;

   /* The end points are normally quoted to two decimal digits, so allow +/-0.01
    * on this test.
    */
   if (png_colorspace_endpoints_match(xy, &sRGB_xy, 1000) != 0)
      colorspace->flags |= PNG_COLORSPACE_ENDPOINTS_MATCH_sRGB;

   else
      colorspace->flags &= PNG_COLORSPACE_CANCEL(
         PNG_COLORSPACE_ENDPOINTS_MATCH_sRGB);

   return 2; /* ok and changed */
}

int /* PRIVATE */
png_colorspace_set_chromaticities(png_const_structrp png_ptr,
    png_colorspacerp colorspace, const png_xy *xy, int preferred)
{
   /* We must check the end points to ensure they are reasonable - in the past
    * color management systems have crashed as a result of getting bogus
    * colorant values, while this isn't the fault of libpng it is the
    * responsibility of libpng because PNG carries the bomb and libpng is in a
    * position to protect against it.
    */
   png_XYZ XYZ;

   switch (png_colorspace_check_xy(&XYZ, xy))
   {
      case 0: /* success */
         return png_colorspace_set_xy_and_XYZ(png_ptr, colorspace, xy, &XYZ,
             preferred);

      case 1:
         /* We can't invert the chromaticities so we can't produce value XYZ
          * values.  Likely as not a color management system will fail too.
          */
         colorspace->flags |= PNG_COLORSPACE_INVALID;
         png_benign_error(png_ptr, "invalid chromaticities");
         break;

      default:
         /* libpng is broken; this should be a warning but if it happens we
          * want error reports so for the moment it is an error.
          */
         colorspace->flags |= PNG_COLORSPACE_INVALID;
         png_error(png_ptr, "internal error checking chromaticities");
   }

   return 0; /* failed */
}

int /* PRIVATE */
png_colorspace_set_endpoints(png_const_structrp png_ptr,
    png_colorspacerp colorspace, const png_XYZ *XYZ_in, int preferred)
{
   png_XYZ XYZ = *XYZ_in;
   png_xy xy;

   switch (png_colorspace_check_XYZ(&xy, &XYZ))
   {
      case 0:
         return png_colorspace_set_xy_and_XYZ(png_ptr, colorspace, &xy, &XYZ,
             preferred);

      case 1:
         /* End points are invalid. */
         colorspace->flags |= PNG_COLORSPACE_INVALID;
         png_benign_error(png_ptr, "invalid end points");
         break;

      default:
         colorspace->flags |= PNG_COLORSPACE_INVALID;
         png_error(png_ptr, "internal error checking chromaticities");
   }

   return 0; /* failed */
}

#if defined(PNG_sRGB_SUPPORTED) || defined(PNG_iCCP_SUPPORTED)
/* Error message generation */
static char
png_icc_tag_char(png_uint_32 byte)
{
   byte &= 0xff;
   if (byte >= 32 && byte <= 126)
      return (char)byte;
   else
      return '?';
}

static void
png_icc_tag_name(char *name, png_uint_32 tag)
{
   name[0] = '\'';
   name[1] = png_icc_tag_char(tag >> 24);
   name[2] = png_icc_tag_char(tag >> 16);
   name[3] = png_icc_tag_char(tag >>  8);
   name[4] = png_icc_tag_char(tag      );
   name[5] = '\'';
}

static int
is_ICC_signature_char(png_alloc_size_t it)
{
   return it == 32 || (it >= 48 && it <= 57) || (it >= 65 && it <= 90) ||
      (it >= 97 && it <= 122);
}

static int
is_ICC_signature(png_alloc_size_t it)
{
   return is_ICC_signature_char(it >> 24) /* checks all the top bits */ &&
      is_ICC_signature_char((it >> 16) & 0xff) &&
      is_ICC_signature_char((it >> 8) & 0xff) &&
      is_ICC_signature_char(it & 0xff);
}

static int
png_icc_profile_error(png_const_structrp png_ptr, png_colorspacerp colorspace,
    png_const_charp name, png_alloc_size_t value, png_const_charp reason)
{
   size_t pos;
   char message[196]; /* see below for calculation */

   if (colorspace != NULL)
      colorspace->flags |= PNG_COLORSPACE_INVALID;

   pos = png_safecat(message, (sizeof message), 0, "profile '"); /* 9 chars */
   pos = png_safecat(message, pos+79, pos, name); /* Truncate to 79 chars */
   pos = png_safecat(message, (sizeof message), pos, "': "); /* +2 = 90 */
   if (is_ICC_signature(value) != 0)
   {
      /* So 'value' is at most 4 bytes and the following cast is safe */
      png_icc_tag_name(message+pos, (png_uint_32)value);
      pos += 6; /* total +8; less than the else clause */
      message[pos++] = ':';
      message[pos++] = ' ';
   }
#  ifdef PNG_WARNINGS_SUPPORTED
   else
   {
      char number[PNG_NUMBER_BUFFER_SIZE]; /* +24 = 114 */

      pos = png_safecat(message, (sizeof message), pos,
          png_format_number(number, number+(sizeof number),
          PNG_NUMBER_FORMAT_x, value));
      pos = png_safecat(message, (sizeof message), pos, "h: "); /* +2 = 116 */
   }
#  endif
   /* The 'reason' is an arbitrary message, allow +79 maximum 195 */
   pos = png_safecat(message, (sizeof message), pos, reason);
   PNG_UNUSED(pos)

   /* This is recoverable, but make it unconditionally an app_error on write to
    * avoid writing invalid ICC profiles into PNG files (i.e., we handle them
    * on read, with a warning, but on write unless the app turns off
    * application errors the PNG won't be written.)
    */
   png_chunk_report(png_ptr, message,
       (colorspace != NULL) ? PNG_CHUNK_ERROR : PNG_CHUNK_WRITE_ERROR);

   return 0;
}
#endif /* sRGB || iCCP */

#ifdef PNG_sRGB_SUPPORTED
int /* PRIVATE */
png_colorspace_set_sRGB(png_const_structrp png_ptr, png_colorspacerp colorspace,
    int intent)
{
   /* sRGB sets known gamma, end points and (from the chunk) intent. */
   /* IMPORTANT: these are not necessarily the values found in an ICC profile
    * because ICC profiles store values adapted to a D50 environment; it is
    * expected that the ICC profile mediaWhitePointTag will be D50; see the
    * checks and code elsewhere to understand this better.
    *
    * These XYZ values, which are accurate to 5dp, produce rgb to gray
    * coefficients of (6968,23435,2366), which are reduced (because they add up
    * to 32769 not 32768) to (6968,23434,2366).  These are the values that
    * libpng has traditionally used (and are the best values given the 15bit
    * algorithm used by the rgb to gray code.)
    */
   static const png_XYZ sRGB_XYZ = /* D65 XYZ (*not* the D50 adapted values!) */
   {
      /* color      X      Y      Z */
      /* red   */ 41239, 21264,  1933,
      /* green */ 35758, 71517, 11919,
      /* blue  */ 18048,  7219, 95053
   };

   /* Do nothing if the colorspace is already invalidated. */
   if ((colorspace->flags & PNG_COLORSPACE_INVALID) != 0)
      return 0;

   /* Check the intent, then check for existing settings.  It is valid for the
    * PNG file to have cHRM or gAMA chunks along with sRGB, but the values must
    * be consistent with the correct values.  If, however, this function is
    * called below because an iCCP chunk matches sRGB then it is quite
    * conceivable that an older app recorded incorrect gAMA and cHRM because of
    * an incorrect calculation based on the values in the profile - this does
    * *not* invalidate the profile (though it still produces an error, which can
    * be ignored.)
    */
   if (intent < 0 || intent >= PNG_sRGB_INTENT_LAST)
      return png_icc_profile_error(png_ptr, colorspace, "sRGB",
          (png_alloc_size_t)intent, "invalid sRGB rendering intent");

   if ((colorspace->flags & PNG_COLORSPACE_HAVE_INTENT) != 0 &&
       colorspace->rendering_intent != intent)
      return png_icc_profile_error(png_ptr, colorspace, "sRGB",
         (png_alloc_size_t)intent, "inconsistent rendering intents");

   if ((colorspace->flags & PNG_COLORSPACE_FROM_sRGB) != 0)
   {
      png_benign_error(png_ptr, "duplicate sRGB information ignored");
      return 0;
   }

   /* If the standard sRGB cHRM chunk does not match the one from the PNG file
    * warn but overwrite the value with the correct one.
    */
   if ((colorspace->flags & PNG_COLORSPACE_HAVE_ENDPOINTS) != 0 &&
       !png_colorspace_endpoints_match(&sRGB_xy, &colorspace->end_points_xy,
       100))
      png_chunk_report(png_ptr, "cHRM chunk does not match sRGB",
         PNG_CHUNK_ERROR);

   /* This check is just done for the error reporting - the routine always
    * returns true when the 'from' argument corresponds to sRGB (2).
    */
   (void)png_colorspace_check_gamma(png_ptr, colorspace, PNG_GAMMA_sRGB_INVERSE,
       2/*from sRGB*/);

   /* intent: bugs in GCC force 'int' to be used as the parameter type. */
   colorspace->rendering_intent = (png_uint_16)intent;
   colorspace->flags |= PNG_COLORSPACE_HAVE_INTENT;

   /* endpoints */
   colorspace->end_points_xy = sRGB_xy;
   colorspace->end_points_XYZ = sRGB_XYZ;
   colorspace->flags |=
      (PNG_COLORSPACE_HAVE_ENDPOINTS|PNG_COLORSPACE_ENDPOINTS_MATCH_sRGB);

   /* gamma */
   colorspace->gamma = PNG_GAMMA_sRGB_INVERSE;
   colorspace->flags |= PNG_COLORSPACE_HAVE_GAMMA;

   /* Finally record that we have an sRGB profile */
   colorspace->flags |=
      (PNG_COLORSPACE_MATCHES_sRGB|PNG_COLORSPACE_FROM_sRGB);

   return 1; /* set */
}
#endif /* sRGB */

#ifdef PNG_iCCP_SUPPORTED
/* Encoded value of D50 as an ICC XYZNumber.  From the ICC 2010 spec the value
 * is XYZ(0.9642,1.0,0.8249), which scales to:
 *
 *    (63189.8112, 65536, 54060.6464)
 */
static const png_byte D50_nCIEXYZ[12] =
   { 0x00, 0x00, 0xf6, 0xd6, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0xd3, 0x2d };

static int /* bool */
icc_check_length(png_const_structrp png_ptr, png_colorspacerp colorspace,
    png_const_charp name, png_uint_32 profile_length)
{
   if (profile_length < 132)
      return png_icc_profile_error(png_ptr, colorspace, name, profile_length,
          "too short");
   return 1;
}

#ifdef PNG_READ_iCCP_SUPPORTED
int /* PRIVATE */
png_icc_check_length(png_const_structrp png_ptr, png_colorspacerp colorspace,
    png_const_charp name, png_uint_32 profile_length)
{
   if (!icc_check_length(png_ptr, colorspace, name, profile_length))
      return 0;

   /* This needs to be here because the 'normal' check is in
    * png_decompress_chunk, yet this happens after the attempt to
    * png_malloc_base the required data.  We only need this on read; on write
    * the caller supplies the profile buffer so libpng doesn't allocate it.  See
    * the call to icc_check_length below (the write case).
    */
#  ifdef PNG_SET_USER_LIMITS_SUPPORTED
      else if (png_ptr->user_chunk_malloc_max > 0 &&
               png_ptr->user_chunk_malloc_max < profile_length)
         return png_icc_profile_error(png_ptr, colorspace, name, profile_length,
             "exceeds application limits");
#  elif PNG_USER_CHUNK_MALLOC_MAX > 0
      else if (PNG_USER_CHUNK_MALLOC_MAX < profile_length)
         return png_icc_profile_error(png_ptr, colorspace, name, profile_length,
             "exceeds libpng limits");
#  else /* !SET_USER_LIMITS */
      /* This will get compiled out on all 32-bit and better systems. */
      else if (PNG_SIZE_MAX < profile_length)
         return png_icc_profile_error(png_ptr, colorspace, name, profile_length,
             "exceeds system limits");
#  endif /* !SET_USER_LIMITS */

   return 1;
}
#endif /* READ_iCCP */

int /* PRIVATE */
png_icc_check_header(png_const_structrp png_ptr, png_colorspacerp colorspace,
    png_const_charp name, png_uint_32 profile_length,
    png_const_bytep profile/* first 132 bytes only */, int color_type)
{
   png_uint_32 temp;

   /* Length check; this cannot be ignored in this code because profile_length
    * is used later to check the tag table, so even if the profile seems over
    * long profile_length from the caller must be correct.  The caller can fix
    * this up on read or write by just passing in the profile header length.
    */
   temp = png_get_uint_32(profile);
   if (temp != profile_length)
      return png_icc_profile_error(png_ptr, colorspace, name, temp,
          "length does not match profile");

   temp = (png_uint_32) (*(profile+8));
   if (temp > 3 && (profile_length & 3))
      return png_icc_profile_error(png_ptr, colorspace, name, profile_length,
          "invalid length");

   temp = png_get_uint_32(profile+128); /* tag count: 12 bytes/tag */
   if (temp > 357913930 || /* (2^32-4-132)/12: maximum possible tag count */
      profile_length < 132+12*temp) /* truncated tag table */
      return png_icc_profile_error(png_ptr, colorspace, name, temp,
          "tag count too large");

   /* The 'intent' must be valid or we can't store it, ICC limits the intent to
    * 16 bits.
    */
   temp = png_get_uint_32(profile+64);
   if (temp >= 0xffff) /* The ICC limit */
      return png_icc_profile_error(png_ptr, colorspace, name, temp,
          "invalid rendering intent");

   /* This is just a warning because the profile may be valid in future
    * versions.
    */
   if (temp >= PNG_sRGB_INTENT_LAST)
      (void)png_icc_profile_error(png_ptr, NULL, name, temp,
          "intent outside defined range");

   /* At this point the tag table can't be checked because it hasn't necessarily
    * been loaded; however, various header fields can be checked.  These checks
    * are for values permitted by the PNG spec in an ICC profile; the PNG spec
    * restricts the profiles that can be passed in an iCCP chunk (they must be
    * appropriate to processing PNG data!)
    */

   /* Data checks (could be skipped).  These checks must be independent of the
    * version number; however, the version number doesn't accommodate changes in
    * the header fields (just the known tags and the interpretation of the
    * data.)
    */
   temp = png_get_uint_32(profile+36); /* signature 'ascp' */
   if (temp != 0x61637370)
      return png_icc_profile_error(png_ptr, colorspace, name, temp,
          "invalid signature");

   /* Currently the PCS illuminant/adopted white point (the computational
    * white point) are required to be D50,
    * however the profile contains a record of the illuminant so perhaps ICC
    * expects to be able to change this in the future (despite the rationale in
    * the introduction for using a fixed PCS adopted white.)  Consequently the
    * following is just a warning.
    */
   if (memcmp(profile+68, D50_nCIEXYZ, 12) != 0)
      (void)png_icc_profile_error(png_ptr, NULL, name, 0/*no tag value*/,
          "PCS illuminant is not D50");

   /* The PNG spec requires this:
    * "If the iCCP chunk is present, the image samples conform to the colour
    * space represented by the embedded ICC profile as defined by the
    * International Color Consortium [ICC]. The colour space of the ICC profile
    * shall be an RGB colour space for colour images (PNG colour types 2, 3, and
    * 6), or a greyscale colour space for greyscale images (PNG colour types 0
    * and 4)."
    *
    * This checking code ensures the embedded profile (on either read or write)
    * conforms to the specification requirements.  Notice that an ICC 'gray'
    * color-space profile contains the information to transform the monochrome
    * data to XYZ or L*a*b (according to which PCS the profile uses) and this
    * should be used in preference to the standard libpng K channel replication
    * into R, G and B channels.
    *
    * Previously it was suggested that an RGB profile on grayscale data could be
    * handled.  However it it is clear that using an RGB profile in this context
    * must be an error - there is no specification of what it means.  Thus it is
    * almost certainly more correct to ignore the profile.
    */
   temp = png_get_uint_32(profile+16); /* data colour space field */
   switch (temp)
   {
      case 0x52474220: /* 'RGB ' */
         if ((color_type & PNG_COLOR_MASK_COLOR) == 0)
            return png_icc_profile_error(png_ptr, colorspace, name, temp,
                "RGB color space not permitted on grayscale PNG");
         break;

      case 0x47524159: /* 'GRAY' */
         if ((color_type & PNG_COLOR_MASK_COLOR) != 0)
            return png_icc_profile_error(png_ptr, colorspace, name, temp,
                "Gray color space not permitted on RGB PNG");
         break;

      default:
         return png_icc_profile_error(png_ptr, colorspace, name, temp,
             "invalid ICC profile color space");
   }

   /* It is up to the application to check that the profile class matches the
    * application requirements; the spec provides no guidance, but it's pretty
    * weird if the profile is not scanner ('scnr'), monitor ('mntr'), printer
    * ('prtr') or 'spac' (for generic color spaces).  Issue a warning in these
    * cases.  Issue an error for device link or abstract profiles - these don't
    * contain the records necessary to transform the color-space to anything
    * other than the target device (and not even that for an abstract profile).
    * Profiles of these classes may not be embedded in images.
    */
   temp = png_get_uint_32(profile+12); /* profile/device class */
   switch (temp)
   {
      case 0x73636e72: /* 'scnr' */
      case 0x6d6e7472: /* 'mntr' */
      case 0x70727472: /* 'prtr' */
      case 0x73706163: /* 'spac' */
         /* All supported */
         break;

      case 0x61627374: /* 'abst' */
         /* May not be embedded in an image */
         return png_icc_profile_error(png_ptr, colorspace, name, temp,
             "invalid embedded Abstract ICC profile");

      case 0x6c696e6b: /* 'link' */
         /* DeviceLink profiles cannot be interpreted in a non-device specific
          * fashion, if an app uses the AToB0Tag in the profile the results are
          * undefined unless the result is sent to the intended device,
          * therefore a DeviceLink profile should not be found embedded in a
          * PNG.
          */
         return png_icc_profile_error(png_ptr, colorspace, name, temp,
             "unexpected DeviceLink ICC profile class");

      case 0x6e6d636c: /* 'nmcl' */
         /* A NamedColor profile is also device specific, however it doesn't
          * contain an AToB0 tag that is open to misinterpretation.  Almost
          * certainly it will fail the tests below.
          */
         (void)png_icc_profile_error(png_ptr, NULL, name, temp,
             "unexpected NamedColor ICC profile class");
         break;

      default:
         /* To allow for future enhancements to the profile accept unrecognized
          * profile classes with a warning, these then hit the test below on the
          * tag content to ensure they are backward compatible with one of the
          * understood profiles.
          */
         (void)png_icc_profile_error(png_ptr, NULL, name, temp,
             "unrecognized ICC profile class");
         break;
   }

   /* For any profile other than a device link one the PCS must be encoded
    * either in XYZ or Lab.
    */
   temp = png_get_uint_32(profile+20);
   switch (temp)
   {
      case 0x58595a20: /* 'XYZ ' */
      case 0x4c616220: /* 'Lab ' */
         break;

      default:
         return png_icc_profile_error(png_ptr, colorspace, name, temp,
             "unexpected ICC PCS encoding");
   }

   return 1;
}

int /* PRIVATE */
png_icc_check_tag_table(png_const_structrp png_ptr, png_colorspacerp colorspace,
    png_const_charp name, png_uint_32 profile_length,
    png_const_bytep profile /* header plus whole tag table */)
{
   png_uint_32 tag_count = png_get_uint_32(profile+128);
   png_uint_32 itag;
   png_const_bytep tag = profile+132; /* The first tag */

   /* First scan all the tags in the table and add bits to the icc_info value
    * (temporarily in 'tags').
    */
   for (itag=0; itag < tag_count; ++itag, tag += 12)
   {
      png_uint_32 tag_id = png_get_uint_32(tag+0);
      png_uint_32 tag_start = png_get_uint_32(tag+4); /* must be aligned */
      png_uint_32 tag_length = png_get_uint_32(tag+8);/* not padded */

      /* The ICC specification does not exclude zero length tags, therefore the
       * start might actually be anywhere if there is no data, but this would be
       * a clear abuse of the intent of the standard so the start is checked for
       * being in range.  All defined tag types have an 8 byte header - a 4 byte
       * type signature then 0.
       */

      /* This is a hard error; potentially it can cause read outside the
       * profile.
       */
      if (tag_start > profile_length || tag_length > profile_length - tag_start)
         return png_icc_profile_error(png_ptr, colorspace, name, tag_id,
             "ICC profile tag outside profile");

      if ((tag_start & 3) != 0)
      {
         /* CNHP730S.icc shipped with Microsoft Windows 64 violates this; it is
          * only a warning here because libpng does not care about the
          * alignment.
          */
         (void)png_icc_profile_error(png_ptr, NULL, name, tag_id,
             "ICC profile tag start not a multiple of 4");
      }
   }

   return 1; /* success, maybe with warnings */
}

#ifdef PNG_sRGB_SUPPORTED
#if PNG_sRGB_PROFILE_CHECKS >= 0
/* Information about the known ICC sRGB profiles */
static const struct
{
   png_uint_32 adler, crc, length;
   png_uint_32 md5[4];
   png_byte    have_md5;
   png_byte    is_broken;
   png_uint_16 intent;

#  define PNG_MD5(a,b,c,d) { a, b, c, d }, (a!=0)||(b!=0)||(c!=0)||(d!=0)
#  define PNG_ICC_CHECKSUM(adler, crc, md5, intent, broke, date, length, fname)\
      { adler, crc, length, md5, broke, intent },

} png_sRGB_checks[] =
{
   /* This data comes from contrib/tools/checksum-icc run on downloads of
    * all four ICC sRGB profiles from www.color.org.
    */
   /* adler32, crc32, MD5[4], intent, date, length, file-name */
   PNG_ICC_CHECKSUM(0x0a3fd9f6, 0x3b8772b9,
       PNG_MD5(0x29f83dde, 0xaff255ae, 0x7842fae4, 0xca83390d), 0, 0,
       "2009/03/27 21:36:31", 3048, "sRGB_IEC61966-2-1_black_scaled.icc")

   /* ICC sRGB v2 perceptual no black-compensation: */
   PNG_ICC_CHECKSUM(0x4909e5e1, 0x427ebb21,
       PNG_MD5(0xc95bd637, 0xe95d8a3b, 0x0df38f99, 0xc1320389), 1, 0,
       "2009/03/27 21:37:45", 3052, "sRGB_IEC61966-2-1_no_black_scaling.icc")

   PNG_ICC_CHECKSUM(0xfd2144a1, 0x306fd8ae,
       PNG_MD5(0xfc663378, 0x37e2886b, 0xfd72e983, 0x8228f1b8), 0, 0,
       "2009/08/10 17:28:01", 60988, "sRGB_v4_ICC_preference_displayclass.icc")

   /* ICC sRGB v4 perceptual */
   PNG_ICC_CHECKSUM(0x209c35d2, 0xbbef7812,
       PNG_MD5(0x34562abf, 0x994ccd06, 0x6d2c5721, 0xd0d68c5d), 0, 0,
       "2007/07/25 00:05:37", 60960, "sRGB_v4_ICC_preference.icc")

   /* The following profiles have no known MD5 checksum. If there is a match
    * on the (empty) MD5 the other fields are used to attempt a match and
    * a warning is produced.  The first two of these profiles have a 'cprt' tag
    * which suggests that they were also made by Hewlett Packard.
    */
   PNG_ICC_CHECKSUM(0xa054d762, 0x5d5129ce,
       PNG_MD5(0x00000000, 0x00000000, 0x00000000, 0x00000000), 1, 0,
       "2004/07/21 18:57:42", 3024, "sRGB_IEC61966-2-1_noBPC.icc")

   /* This is a 'mntr' (display) profile with a mediaWhitePointTag that does not
    * match the D50 PCS illuminant in the header (it is in fact the D65 values,
    * so the white point is recorded as the un-adapted value.)  The profiles
    * below only differ in one byte - the intent - and are basically the same as
    * the previous profile except for the mediaWhitePointTag error and a missing
    * chromaticAdaptationTag.
    */
   PNG_ICC_CHECKSUM(0xf784f3fb, 0x182ea552,
       PNG_MD5(0x00000000, 0x00000000, 0x00000000, 0x00000000), 0, 1/*broken*/,
       "1998/02/09 06:49:00", 3144, "HP-Microsoft sRGB v2 perceptual")

   PNG_ICC_CHECKSUM(0x0398f3fc, 0xf29e526d,
       PNG_MD5(0x00000000, 0x00000000, 0x00000000, 0x00000000), 1, 1/*broken*/,
       "1998/02/09 06:49:00", 3144, "HP-Microsoft sRGB v2 media-relative")
};

static int
png_compare_ICC_profile_with_sRGB(png_const_structrp png_ptr,
    png_const_bytep profile, uLong adler)
{
   /* The quick check is to verify just the MD5 signature and trust the
    * rest of the data.  Because the profile has already been verified for
    * correctness this is safe.  png_colorspace_set_sRGB will check the 'intent'
    * field too, so if the profile has been edited with an intent not defined
    * by sRGB (but maybe defined by a later ICC specification) the read of
    * the profile will fail at that point.
    */

   png_uint_32 length = 0;
   png_uint_32 intent = 0x10000; /* invalid */
#if PNG_sRGB_PROFILE_CHECKS > 1
   uLong crc = 0; /* the value for 0 length data */
#endif
   unsigned int i;

#ifdef PNG_SET_OPTION_SUPPORTED
   /* First see if PNG_SKIP_sRGB_CHECK_PROFILE has been set to "on" */
   if (((png_ptr->options >> PNG_SKIP_sRGB_CHECK_PROFILE) & 3) ==
               PNG_OPTION_ON)
      return 0;
#endif

   for (i=0; i < (sizeof png_sRGB_checks) / (sizeof png_sRGB_checks[0]); ++i)
   {
      if (png_get_uint_32(profile+84) == png_sRGB_checks[i].md5[0] &&
         png_get_uint_32(profile+88) == png_sRGB_checks[i].md5[1] &&
         png_get_uint_32(profile+92) == png_sRGB_checks[i].md5[2] &&
         png_get_uint_32(profile+96) == png_sRGB_checks[i].md5[3])
      {
         /* This may be one of the old HP profiles without an MD5, in that
          * case we can only use the length and Adler32 (note that these
          * are not used by default if there is an MD5!)
          */
#        if PNG_sRGB_PROFILE_CHECKS == 0
            if (png_sRGB_checks[i].have_md5 != 0)
               return 1+png_sRGB_checks[i].is_broken;
#        endif

         /* Profile is unsigned or more checks have been configured in. */
         if (length == 0)
         {
            length = png_get_uint_32(profile);
            intent = png_get_uint_32(profile+64);
         }

         /* Length *and* intent must match */
         if (length == (png_uint_32) png_sRGB_checks[i].length &&
            intent == (png_uint_32) png_sRGB_checks[i].intent)
         {
            /* Now calculate the adler32 if not done already. */
            if (adler == 0)
            {
               adler = adler32(0, NULL, 0);
               adler = adler32(adler, profile, length);
            }

            if (adler == png_sRGB_checks[i].adler)
            {
               /* These basic checks suggest that the data has not been
                * modified, but if the check level is more than 1 perform
                * our own crc32 checksum on the data.
                */
#              if PNG_sRGB_PROFILE_CHECKS > 1
                  if (crc == 0)
                  {
                     crc = crc32(0, NULL, 0);
                     crc = crc32(crc, profile, length);
                  }

                  /* So this check must pass for the 'return' below to happen.
                   */
                  if (crc == png_sRGB_checks[i].crc)
#              endif
               {
                  if (png_sRGB_checks[i].is_broken != 0)
                  {
                     /* These profiles are known to have bad data that may cause
                      * problems if they are used, therefore attempt to
                      * discourage their use, skip the 'have_md5' warning below,
                      * which is made irrelevant by this error.
                      */
                     png_chunk_report(png_ptr, "known incorrect sRGB profile",
                         PNG_CHUNK_ERROR);
                  }

                  /* Warn that this being done; this isn't even an error since
                   * the profile is perfectly valid, but it would be nice if
                   * people used the up-to-date ones.
                   */
                  else if (png_sRGB_checks[i].have_md5 == 0)
                  {
                     png_chunk_report(png_ptr,
                         "out-of-date sRGB profile with no signature",
                         PNG_CHUNK_WARNING);
                  }

                  return 1+png_sRGB_checks[i].is_broken;
               }
            }

# if PNG_sRGB_PROFILE_CHECKS > 0
         /* The signature matched, but the profile had been changed in some
          * way.  This probably indicates a data error or uninformed hacking.
          * Fall through to "no match".
          */
         png_chunk_report(png_ptr,
             "Not recognizing known sRGB profile that has been edited",
             PNG_CHUNK_WARNING);
         break;
# endif
         }
      }
   }

   return 0; /* no match */
}

void /* PRIVATE */
png_icc_set_sRGB(png_const_structrp png_ptr,
    png_colorspacerp colorspace, png_const_bytep profile, uLong adler)
{
   /* Is this profile one of the known ICC sRGB profiles?  If it is, just set
    * the sRGB information.
    */
   if (png_compare_ICC_profile_with_sRGB(png_ptr, profile, adler) != 0)
      (void)png_colorspace_set_sRGB(png_ptr, colorspace,
         (int)/*already checked*/png_get_uint_32(profile+64));
}
#endif /* PNG_sRGB_PROFILE_CHECKS >= 0 */
#endif /* sRGB */

int /* PRIVATE */
png_colorspace_set_ICC(png_const_structrp png_ptr, png_colorspacerp colorspace,
    png_const_charp name, png_uint_32 profile_length, png_const_bytep profile,
    int color_type)
{
   if ((colorspace->flags & PNG_COLORSPACE_INVALID) != 0)
      return 0;

   if (icc_check_length(png_ptr, colorspace, name, profile_length) != 0 &&
       png_icc_check_header(png_ptr, colorspace, name, profile_length, profile,
           color_type) != 0 &&
       png_icc_check_tag_table(png_ptr, colorspace, name, profile_length,
           profile) != 0)
   {
#     if defined(PNG_sRGB_SUPPORTED) && PNG_sRGB_PROFILE_CHECKS >= 0
         /* If no sRGB support, don't try storing sRGB information */
         png_icc_set_sRGB(png_ptr, colorspace, profile, 0);
#     endif
      return 1;
   }

   /* Failure case */
   return 0;
}
#endif /* iCCP */

#ifdef PNG_READ_RGB_TO_GRAY_SUPPORTED
void /* PRIVATE */
png_colorspace_set_rgb_coefficients(png_structrp png_ptr)
{
   /* Set the rgb_to_gray coefficients from the colorspace. */
   if (png_ptr->rgb_to_gray_coefficients_set == 0 &&
      (png_ptr->colorspace.flags & PNG_COLORSPACE_HAVE_ENDPOINTS) != 0)
   {
      /* png_set_background has not been called, get the coefficients from the Y
       * values of the colorspace colorants.
       */
      png_fixed_point r = png_ptr->colorspace.end_points_XYZ.red_Y;
      png_fixed_point g = png_ptr->colorspace.end_points_XYZ.green_Y;
      png_fixed_point b = png_ptr->colorspace.end_points_XYZ.blue_Y;
      png_fixed_point total = r+g+b;

      if (total > 0 &&
         r >= 0 && png_muldiv(&r, r, 32768, total) && r >= 0 && r <= 32768 &&
         g >= 0 && png_muldiv(&g, g, 32768, total) && g >= 0 && g <= 32768 &&
         b >= 0 && png_muldiv(&b, b, 32768, total) && b >= 0 && b <= 32768 &&
         r+g+b <= 32769)
      {
         /* We allow 0 coefficients here.  r+g+b may be 32769 if two or
          * all of the coefficients were rounded up.  Handle this by
          * reducing the *largest* coefficient by 1; this matches the
          * approach used for the default coefficients in pngrtran.c
          */
         int add = 0;

         if (r+g+b > 32768)
            add = -1;
         else if (r+g+b < 32768)
            add = 1;

         if (add != 0)
         {
            if (g >= r && g >= b)
               g += add;
            else if (r >= g && r >= b)
               r += add;
            else
               b += add;
         }

         /* Check for an internal error. */
         if (r+g+b != 32768)
            png_error(png_ptr,
                "internal error handling cHRM coefficients");

         else
         {
            png_ptr->rgb_to_gray_red_coeff   = (png_uint_16)r;
            png_ptr->rgb_to_gray_green_coeff = (png_uint_16)g;
         }
      }

      /* This is a png_error at present even though it could be ignored -
       * it should never happen, but it is important that if it does, the
       * bug is fixed.
       */
      else
         png_error(png_ptr, "internal error handling cHRM->XYZ");
   }
}
#endif /* READ_RGB_TO_GRAY */

#endif /* COLORSPACE */

void /* PRIVATE */
png_check_IHDR(png_const_structrp png_ptr,
    png_uint_32 width, png_uint_32 height, int bit_depth,
    int color_type, int interlace_type, int compression_type,
    int filter_type)
{
   int error = 0;

   /* Check for width and height valid values */
   if (width == 0)
   {
      png_warning(png_ptr, "Image width is zero in IHDR");
      error = 1;
   }

   if (width > PNG_UINT_31_MAX)
   {
      png_warning(png_ptr, "Invalid image width in IHDR");
      error = 1;
   }

   /* The bit mask on the first line below must be at least as big as a
    * png_uint_32.  "~7U" is not adequate on 16-bit systems because it will
    * be an unsigned 16-bit value.  Casting to (png_alloc_size_t) makes the
    * type of the result at least as bit (in bits) as the RHS of the > operator
    * which also avoids a common warning on 64-bit systems that the comparison
    * of (png_uint_32) against the constant value on the RHS will always be
    * false.
    */
   if (((width + 7) & ~(png_alloc_size_t)7) >
       (((PNG_SIZE_MAX
           - 48        /* big_row_buf hack */
           - 1)        /* filter byte */
           / 8)        /* 8-byte RGBA pixels */
           - 1))       /* extra max_pixel_depth pad */
   {
      /* The size of the row must be within the limits of this architecture.
       * Because the read code can perform arbitrary transformations the
       * maximum size is checked here.  Because the code in png_read_start_row
       * adds extra space "for safety's sake" in several places a conservative
       * limit is used here.
       *
       * NOTE: it would be far better to check the size that is actually used,
       * but the effect in the real world is minor and the changes are more
       * extensive, therefore much more dangerous and much more difficult to
       * write in a way that avoids compiler warnings.
       */
      png_warning(png_ptr, "Image width is too large for this architecture");
      error = 1;
   }

#ifdef PNG_SET_USER_LIMITS_SUPPORTED
   if (width > png_ptr->user_width_max)
#else
   if (width > PNG_USER_WIDTH_MAX)
#endif
   {
      png_warning(png_ptr, "Image width exceeds user limit in IHDR");
      error = 1;
   }

   if (height == 0)
   {
      png_warning(png_ptr, "Image height is zero in IHDR");
      error = 1;
   }

   if (height > PNG_UINT_31_MAX)
   {
      png_warning(png_ptr, "Invalid image height in IHDR");
      error = 1;
   }

#ifdef PNG_SET_USER_LIMITS_SUPPORTED
   if (height > png_ptr->user_height_max)
#else
   if (height > PNG_USER_HEIGHT_MAX)
#endif
   {
      png_warning(png_ptr, "Image height exceeds user limit in IHDR");
      error = 1;
   }

   /* Check other values */
   if (bit_depth != 1 && bit_depth != 2 && bit_depth != 4 &&
       bit_depth != 8 && bit_depth != 16)
   {
      png_warning(png_ptr, "Invalid bit depth in IHDR");
      error = 1;
   }

   if (color_type < 0 || color_type == 1 ||
       color_type == 5 || color_type > 6)
   {
      png_warning(png_ptr, "Invalid color type in IHDR");
      error = 1;
   }

   if (((color_type == PNG_COLOR_TYPE_PALETTE) && bit_depth > 8) ||
       ((color_type == PNG_COLOR_TYPE_RGB ||
         color_type == PNG_COLOR_TYPE_GRAY_ALPHA ||
         color_type == PNG_COLOR_TYPE_RGB_ALPHA) && bit_depth < 8))
   {
      png_warning(png_ptr, "Invalid color type/bit depth combination in IHDR");
      error = 1;
   }

   if (interlace_type >= PNG_INTERLACE_LAST)
   {
      png_warning(png_ptr, "Unknown interlace method in IHDR");
      error = 1;
   }

   if (compression_type != PNG_COMPRESSION_TYPE_BASE)
   {
      png_warning(png_ptr, "Unknown compression method in IHDR");
      error = 1;
   }

#ifdef PNG_MNG_FEATURES_SUPPORTED
   /* Accept filter_method 64 (intrapixel differencing) only if
    * 1. Libpng was compiled with PNG_MNG_FEATURES_SUPPORTED and
    * 2. Libpng did not read a PNG signature (this filter_method is only
    *    used in PNG datastreams that are embedded in MNG datastreams) and
    * 3. The application called png_permit_mng_features with a mask that
    *    included PNG_FLAG_MNG_FILTER_64 and
    * 4. The filter_method is 64 and
    * 5. The color_type is RGB or RGBA
    */
   if ((png_ptr->mode & PNG_HAVE_PNG_SIGNATURE) != 0 &&
       png_ptr->mng_features_permitted != 0)
      png_warning(png_ptr, "MNG features are not allowed in a PNG datastream");

   if (filter_type != PNG_FILTER_TYPE_BASE)
   {
      if (!((png_ptr->mng_features_permitted & PNG_FLAG_MNG_FILTER_64) != 0 &&
          (filter_type == PNG_INTRAPIXEL_DIFFERENCING) &&
          ((png_ptr->mode & PNG_HAVE_PNG_SIGNATURE) == 0) &&
          (color_type == PNG_COLOR_TYPE_RGB ||
          color_type == PNG_COLOR_TYPE_RGB_ALPHA)))
      {
         png_warning(png_ptr, "Unknown filter method in IHDR");
         error = 1;
      }

      if ((png_ptr->mode & PNG_HAVE_PNG_SIGNATURE) != 0)
      {
         png_warning(png_ptr, "Invalid filter method in IHDR");
         error = 1;
      }
   }

#else
   if (filter_type != PNG_FILTER_TYPE_BASE)
   {
      png_warning(png_ptr, "Unknown filter method in IHDR");
      error = 1;
   }
#endif

   if (error == 1)
      png_error(png_ptr, "Invalid IHDR data");
}

#if defined(PNG_sCAL_SUPPORTED) || defined(PNG_pCAL_SUPPORTED)
/* ASCII to fp functions */
/* Check an ASCII formatted floating point value, see the more detailed
 * comments in pngpriv.h
 */
/* The following is used internally to preserve the sticky flags */
#define png_fp_add(state, flags) ((state) |= (flags))
#define png_fp_set(state, value) ((state) = (value) | ((state) & PNG_FP_STICKY))

int /* PRIVATE */
png_check_fp_number(png_const_charp string, size_t size, int *statep,
    size_t *whereami)
{
   int state = *statep;
   size_t i = *whereami;

   while (i < size)
   {
      int type;
      /* First find the type of the next character */
      switch (string[i])
      {
      case 43:  type = PNG_FP_SAW_SIGN;                   break;
      case 45:  type = PNG_FP_SAW_SIGN + PNG_FP_NEGATIVE; break;
      case 46:  type = PNG_FP_SAW_DOT;                    break;
      case 48:  type = PNG_FP_SAW_DIGIT;                  break;
      case 49: case 50: case 51: case 52:
      case 53: case 54: case 55: case 56:
      case 57:  type = PNG_FP_SAW_DIGIT + PNG_FP_NONZERO; break;
      case 69:
      case 101: type = PNG_FP_SAW_E;                      break;
      default:  goto PNG_FP_End;
      }

      /* Now deal with this type according to the current
       * state, the type is arranged to not overlap the
       * bits of the PNG_FP_STATE.
       */
      switch ((state & PNG_FP_STATE) + (type & PNG_FP_SAW_ANY))
      {
      case PNG_FP_INTEGER + PNG_FP_SAW_SIGN:
         if ((state & PNG_FP_SAW_ANY) != 0)
            goto PNG_FP_End; /* not a part of the number */

         png_fp_add(state, type);
         break;

      case PNG_FP_INTEGER + PNG_FP_SAW_DOT:
         /* Ok as trailer, ok as lead of fraction. */
         if ((state & PNG_FP_SAW_DOT) != 0) /* two dots */
            goto PNG_FP_End;

         else if ((state & PNG_FP_SAW_DIGIT) != 0) /* trailing dot? */
            png_fp_add(state, type);

         else
            png_fp_set(state, PNG_FP_FRACTION | type);

         break;

      case PNG_FP_INTEGER + PNG_FP_SAW_DIGIT:
         if ((state & PNG_FP_SAW_DOT) != 0) /* delayed fraction */
            png_fp_set(state, PNG_FP_FRACTION | PNG_FP_SAW_DOT);

         png_fp_add(state, type | PNG_FP_WAS_VALID);

         break;

      case PNG_FP_INTEGER + PNG_FP_SAW_E:
         if ((state & PNG_FP_SAW_DIGIT) == 0)
            goto PNG_FP_End;

         png_fp_set(state, PNG_FP_EXPONENT);

         break;

   /* case PNG_FP_FRACTION + PNG_FP_SAW_SIGN:
         goto PNG_FP_End; ** no sign in fraction */

   /* case PNG_FP_FRACTION + PNG_FP_SAW_DOT:
         goto PNG_FP_End; ** Because SAW_DOT is always set */

      case PNG_FP_FRACTION + PNG_FP_SAW_DIGIT:
         png_fp_add(state, type | PNG_FP_WAS_VALID);
         break;

      case PNG_FP_FRACTION + PNG_FP_SAW_E:
         /* This is correct because the trailing '.' on an
          * integer is handled above - so we can only get here
          * with the sequence ".E" (with no preceding digits).
          */
         if ((state & PNG_FP_SAW_DIGIT) == 0)
            goto PNG_FP_End;

         png_fp_set(state, PNG_FP_EXPONENT);

         break;

      case PNG_FP_EXPONENT + PNG_FP_SAW_SIGN:
         if ((state & PNG_FP_SAW_ANY) != 0)
            goto PNG_FP_End; /* not a part of the number */

         png_fp_add(state, PNG_FP_SAW_SIGN);

         break;

   /* case PNG_FP_EXPONENT + PNG_FP_SAW_DOT:
         goto PNG_FP_End; */

      case PNG_FP_EXPONENT + PNG_FP_SAW_DIGIT:
         png_fp_add(state, PNG_FP_SAW_DIGIT | PNG_FP_WAS_VALID);

         break;

   /* case PNG_FP_EXPONEXT + PNG_FP_SAW_E:
         goto PNG_FP_End; */

      default: goto PNG_FP_End; /* I.e. break 2 */
      }

      /* The character seems ok, continue. */
      ++i;
   }

PNG_FP_End:
   /* Here at the end, update the state and return the correct
    * return code.
    */
   *statep = state;
   *whereami = i;

   return (state & PNG_FP_SAW_DIGIT) != 0;
}


/* The same but for a complete stri
... [truncated]

## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

