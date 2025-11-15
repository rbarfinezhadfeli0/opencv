# Documentation for `docs/3rdparty/libjpeg-turbo/src/jddiffct.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/src/jddiffct.c_docs.md`
- **File Name**: `jddiffct.c_docs.md`
- **File Size**: 17,102 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/src/jddiffct.c_docs.md](../../../../docs/3rdparty/libjpeg-turbo/src/jddiffct.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/src` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/src/jddiffct.c`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/src/jddiffct.c`
- **File Name**: `jddiffct.c`
- **File Size**: 13,681 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjpeg-turbo/src/jddiffct.c](../../../3rdparty/libjpeg-turbo/src/jddiffct.c)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * jddiffct.c
 *
 * This file was part of the Independent JPEG Group's software:
 * Copyright (C) 1994-1997, Thomas G. Lane.
 * Lossless JPEG Modifications:
 * Copyright (C) 1999, Ken Murchison.
 * libjpeg-turbo Modifications:
 * Copyright (C) 2022, 2024, D. R. Commander.
 * For conditions of distribution and use, see the accompanying README.ijg
 * file.
 *
 * This file contains the [un]difference buffer controller for decompression.
 * This controller is the top level of the lossless JPEG decompressor proper.
 * The difference buffer lies between the entropy decoding and
 * prediction/undifferencing steps.  The undifference buffer lies between the
 * prediction/undifferencing and scaling steps.
 *
 * In buffered-image mode, this controller is the interface between
 * input-oriented processing and output-oriented processing.
 */

#define JPEG_INTERNALS
#include "jinclude.h"
#include "jpeglib.h"
#include "jlossls.h"            /* Private declarations for lossless codec */


#ifdef D_LOSSLESS_SUPPORTED

/* Private buffer controller object */

typedef struct {
  struct jpeg_d_coef_controller pub; /* public fields */

  /* These variables keep track of the current location of the input side. */
  /* cinfo->input_iMCU_row is also used for this. */
  JDIMENSION MCU_ctr;           /* counts MCUs processed in current row */
  unsigned int restart_rows_to_go;      /* MCU rows left in this restart
                                           interval */
  unsigned int MCU_vert_offset;         /* counts MCU rows within iMCU row */
  unsigned int MCU_rows_per_iMCU_row;   /* number of such rows needed */

  /* The output side's location is represented by cinfo->output_iMCU_row. */

  JDIFFARRAY diff_buf[MAX_COMPONENTS];  /* iMCU row of differences */
  JDIFFARRAY undiff_buf[MAX_COMPONENTS]; /* iMCU row of undiff'd samples */

#ifdef D_MULTISCAN_FILES_SUPPORTED
  /* In multi-pass modes, we need a virtual sample array for each component. */
  jvirt_sarray_ptr whole_image[MAX_COMPONENTS];
#endif
} my_diff_controller;

typedef my_diff_controller *my_diff_ptr;

/* Forward declarations */
METHODDEF(int) decompress_data(j_decompress_ptr cinfo, _JSAMPIMAGE output_buf);
#ifdef D_MULTISCAN_FILES_SUPPORTED
METHODDEF(int) output_data(j_decompress_ptr cinfo, _JSAMPIMAGE output_buf);
#endif


LOCAL(void)
start_iMCU_row(j_decompress_ptr cinfo)
/* Reset within-iMCU-row counters for a new row (input side) */
{
  my_diff_ptr diff = (my_diff_ptr)cinfo->coef;

  /* In an interleaved scan, an MCU row is the same as an iMCU row.
   * In a noninterleaved scan, an iMCU row has v_samp_factor MCU rows.
   * But at the bottom of the image, process only what's left.
   */
  if (cinfo->comps_in_scan > 1) {
    diff->MCU_rows_per_iMCU_row = 1;
  } else {
    if (cinfo->input_iMCU_row < (cinfo->total_iMCU_rows-1))
      diff->MCU_rows_per_iMCU_row = cinfo->cur_comp_info[0]->v_samp_factor;
    else
      diff->MCU_rows_per_iMCU_row = cinfo->cur_comp_info[0]->last_row_height;
  }

  diff->MCU_ctr = 0;
  diff->MCU_vert_offset = 0;
}


/*
 * Initialize for an input processing pass.
 */

METHODDEF(void)
start_input_pass(j_decompress_ptr cinfo)
{
  my_diff_ptr diff = (my_diff_ptr)cinfo->coef;

  /* Because it is hitching a ride on the jpeg_inverse_dct struct,
   * start_pass_lossless() will be called at the start of the output pass.
   * This ensures that it will be called at the start of the input pass as
   * well.
   */
  (*cinfo->idct->start_pass) (cinfo);

  /* Check that the restart interval is an integer multiple of the number
   * of MCUs in an MCU row.
   */
  if (cinfo->restart_interval % cinfo->MCUs_per_row != 0)
    ERREXIT2(cinfo, JERR_BAD_RESTART,
             cinfo->restart_interval, cinfo->MCUs_per_row);

  /* Initialize restart counter */
  diff->restart_rows_to_go = cinfo->restart_interval / cinfo->MCUs_per_row;

  cinfo->input_iMCU_row = 0;
  start_iMCU_row(cinfo);
}


/*
 * Check for a restart marker & resynchronize decoder, undifferencer.
 * Returns FALSE if must suspend.
 */

METHODDEF(boolean)
process_restart(j_decompress_ptr cinfo)
{
  my_diff_ptr diff = (my_diff_ptr)cinfo->coef;

  if (!(*cinfo->entropy->process_restart) (cinfo))
    return FALSE;

  (*cinfo->idct->start_pass) (cinfo);

  /* Reset restart counter */
  diff->restart_rows_to_go = cinfo->restart_interval / cinfo->MCUs_per_row;

  return TRUE;
}


/*
 * Initialize for an output processing pass.
 */

METHODDEF(void)
start_output_pass(j_decompress_ptr cinfo)
{
  cinfo->output_iMCU_row = 0;
}


/*
 * Decompress and return some data in the supplied buffer.
 * Always attempts to emit one fully interleaved MCU row ("iMCU" row).
 * Input and output must run in lockstep since we have only a one-MCU buffer.
 * Return value is JPEG_ROW_COMPLETED, JPEG_SCAN_COMPLETED, or JPEG_SUSPENDED.
 *
 * NB: output_buf contains a plane for each component in image,
 * which we index according to the component's SOF position.
 */

METHODDEF(int)
decompress_data(j_decompress_ptr cinfo, _JSAMPIMAGE output_buf)
{
  my_diff_ptr diff = (my_diff_ptr)cinfo->coef;
  lossless_decomp_ptr losslessd = (lossless_decomp_ptr)cinfo->idct;
  JDIMENSION MCU_col_num;       /* index of current MCU within row */
  JDIMENSION MCU_count;         /* number of MCUs decoded */
  JDIMENSION last_iMCU_row = cinfo->total_iMCU_rows - 1;
  int ci, compi, row, prev_row;
  unsigned int yoffset;
  jpeg_component_info *compptr;

  /* Loop to process as much as one whole iMCU row */
  for (yoffset = diff->MCU_vert_offset; yoffset < diff->MCU_rows_per_iMCU_row;
       yoffset++) {

    /* Process restart marker if needed; may have to suspend */
    if (cinfo->restart_interval) {
      if (diff->restart_rows_to_go == 0)
        if (!process_restart(cinfo))
          return JPEG_SUSPENDED;
    }

    MCU_col_num = diff->MCU_ctr;
    /* Try to fetch an MCU row (or remaining portion of suspended MCU row). */
    MCU_count =
      (*cinfo->entropy->decode_mcus) (cinfo,
                                      diff->diff_buf, yoffset, MCU_col_num,
                                      cinfo->MCUs_per_row - MCU_col_num);
    if (MCU_count != cinfo->MCUs_per_row - MCU_col_num) {
      /* Suspension forced; update state counters and exit */
      diff->MCU_vert_offset = yoffset;
      diff->MCU_ctr += MCU_count;
      return JPEG_SUSPENDED;
    }

    /* Account for restart interval (no-op if not using restarts) */
    if (cinfo->restart_interval)
      diff->restart_rows_to_go--;

    /* Completed an MCU row, but perhaps not an iMCU row */
    diff->MCU_ctr = 0;
  }

  /*
   * Undifference and scale each scanline of the disassembled MCU row
   * separately.  We do not process dummy samples at the end of a scanline
   * or dummy rows at the end of the image.
   */
  for (ci = 0; ci < cinfo->comps_in_scan; ci++) {
    compptr = cinfo->cur_comp_info[ci];
    compi = compptr->component_index;
    for (row = 0, prev_row = compptr->v_samp_factor - 1;
         row < (cinfo->input_iMCU_row == last_iMCU_row ?
                compptr->last_row_height : compptr->v_samp_factor);
         prev_row = row, row++) {
      (*losslessd->predict_undifference[compi])
        (cinfo, compi, diff->diff_buf[compi][row],
          diff->undiff_buf[compi][prev_row], diff->undiff_buf[compi][row],
          compptr->width_in_blocks);
      (*losslessd->scaler_scale) (cinfo, diff->undiff_buf[compi][row],
                                  output_buf[compi][row],
                                  compptr->width_in_blocks);
    }
  }

  /* Completed the iMCU row, advance counters for next one.
   *
   * NB: output_data will increment output_iMCU_row.
   * This counter is not needed for the single-pass case
   * or the input side of the multi-pass case.
   */
  if (++(cinfo->input_iMCU_row) < cinfo->total_iMCU_rows) {
    start_iMCU_row(cinfo);
    return JPEG_ROW_COMPLETED;
  }
  /* Completed the scan */
  (*cinfo->inputctl->finish_input_pass) (cinfo);
  return JPEG_SCAN_COMPLETED;
}


/*
 * Dummy consume-input routine for single-pass operation.
 */

METHODDEF(int)
dummy_consume_data(j_decompress_ptr cinfo)
{
  return JPEG_SUSPENDED;        /* Always indicate nothing was done */
}


#ifdef D_MULTISCAN_FILES_SUPPORTED

/*
 * Consume input data and store it in the full-image sample buffer.
 * We read as much as one fully interleaved MCU row ("iMCU" row) per call,
 * ie, v_samp_factor rows for each component in the scan.
 * Return value is JPEG_ROW_COMPLETED, JPEG_SCAN_COMPLETED, or JPEG_SUSPENDED.
 */

METHODDEF(int)
consume_data(j_decompress_ptr cinfo)
{
  my_diff_ptr diff = (my_diff_ptr)cinfo->coef;
  int ci, compi;
  _JSAMPARRAY buffer[MAX_COMPS_IN_SCAN];
  jpeg_component_info *compptr;

  /* Align the virtual buffers for the components used in this scan. */
  for (ci = 0; ci < cinfo->comps_in_scan; ci++) {
    compptr = cinfo->cur_comp_info[ci];
    compi = compptr->component_index;
    buffer[compi] = (_JSAMPARRAY)(*cinfo->mem->access_virt_sarray)
      ((j_common_ptr)cinfo, diff->whole_image[compi],
       cinfo->input_iMCU_row * compptr->v_samp_factor,
       (JDIMENSION)compptr->v_samp_factor, TRUE);
  }

  return decompress_data(cinfo, buffer);
}


/*
 * Output some data from the full-image sample buffer in the multi-pass case.
 * Always attempts to emit one fully interleaved MCU row ("iMCU" row).
 * Return value is JPEG_ROW_COMPLETED, JPEG_SCAN_COMPLETED, or JPEG_SUSPENDED.
 *
 * NB: output_buf contains a plane for each component in image.
 */

METHODDEF(int)
output_data(j_decompress_ptr cinfo, _JSAMPIMAGE output_buf)
{
  my_diff_ptr diff = (my_diff_ptr)cinfo->coef;
  JDIMENSION last_iMCU_row = cinfo->total_iMCU_rows - 1;
  int ci, samp_rows, row;
  _JSAMPARRAY buffer;
  jpeg_component_info *compptr;

  /* Force some input to be done if we are getting ahead of the input. */
  while (cinfo->input_scan_number < cinfo->output_scan_number ||
         (cinfo->input_scan_number == cinfo->output_scan_number &&
          cinfo->input_iMCU_row <= cinfo->output_iMCU_row)) {
    if ((*cinfo->inputctl->consume_input) (cinfo) == JPEG_SUSPENDED)
      return JPEG_SUSPENDED;
  }

  /* OK, output from the virtual arrays. */
  for (ci = 0, compptr = cinfo->comp_info; ci < cinfo->num_components;
       ci++, compptr++) {
    /* Align the virtual buffer for this component. */
    buffer = (_JSAMPARRAY)(*cinfo->mem->access_virt_sarray)
      ((j_common_ptr)cinfo, diff->whole_image[ci],
       cinfo->output_iMCU_row * compptr->v_samp_factor,
       (JDIMENSION)compptr->v_samp_factor, FALSE);

    if (cinfo->output_iMCU_row < last_iMCU_row)
      samp_rows = compptr->v_samp_factor;
    else {
      /* NB: can't use last_row_height here; it is input-side-dependent! */
      samp_rows = (int)(compptr->height_in_blocks % compptr->v_samp_factor);
      if (samp_rows == 0) samp_rows = compptr->v_samp_factor;
    }

    for (row = 0; row < samp_rows; row++) {
      memcpy(output_buf[ci][row], buffer[row],
             compptr->width_in_blocks * sizeof(_JSAMPLE));
    }
  }

  if (++(cinfo->output_iMCU_row) < cinfo->total_iMCU_rows)
    return JPEG_ROW_COMPLETED;
  return JPEG_SCAN_COMPLETED;
}

#endif /* D_MULTISCAN_FILES_SUPPORTED */


/*
 * Initialize difference buffer controller.
 */

GLOBAL(void)
_jinit_d_diff_controller(j_decompress_ptr cinfo, boolean need_full_buffer)
{
  my_diff_ptr diff;
  int ci;
  jpeg_component_info *compptr;

#if BITS_IN_JSAMPLE == 8
  if (cinfo->data_precision > BITS_IN_JSAMPLE || cinfo->data_precision < 2)
#else
  if (cinfo->data_precision > BITS_IN_JSAMPLE ||
      cinfo->data_precision < BITS_IN_JSAMPLE - 3)
#endif
    ERREXIT1(cinfo, JERR_BAD_PRECISION, cinfo->data_precision);

  diff = (my_diff_ptr)
    (*cinfo->mem->alloc_small) ((j_common_ptr)cinfo, JPOOL_IMAGE,
                                sizeof(my_diff_controller));
  cinfo->coef = (struct jpeg_d_coef_controller *)diff;
  diff->pub.start_input_pass = start_input_pass;
  diff->pub.start_output_pass = start_output_pass;

  /* Create the [un]difference buffers. */
  for (ci = 0, compptr = cinfo->comp_info; ci < cinfo->num_components;
       ci++, compptr++) {
    diff->diff_buf[ci] =
      ALLOC_DARRAY(JPOOL_IMAGE,
                   (JDIMENSION)jround_up((long)compptr->width_in_blocks,
                                         (long)compptr->h_samp_factor),
                   (JDIMENSION)compptr->v_samp_factor);
    diff->undiff_buf[ci] =
      ALLOC_DARRAY(JPOOL_IMAGE,
                   (JDIMENSION)jround_up((long)compptr->width_in_blocks,
                                         (long)compptr->h_samp_factor),
                   (JDIMENSION)compptr->v_samp_factor);
  }

  if (need_full_buffer) {
#ifdef D_MULTISCAN_FILES_SUPPORTED
    /* Allocate a full-image virtual array for each component. */
    int access_rows;

    for (ci = 0, compptr = cinfo->comp_info; ci < cinfo->num_components;
         ci++, compptr++) {
      access_rows = compptr->v_samp_factor;
      diff->whole_image[ci] = (*cinfo->mem->request_virt_sarray)
        ((j_common_ptr)cinfo, JPOOL_IMAGE, FALSE,
         (JDIMENSION)jround_up((long)compptr->width_in_blocks,
                               (long)compptr->h_samp_factor),
         (JDIMENSION)jround_up((long)compptr->height_in_blocks,
                               (long)compptr->v_samp_factor),
         (JDIMENSION)access_rows);
    }
    diff->pub.consume_data = consume_data;
    diff->pub._decompress_data = output_data;
#else
    ERREXIT(cinfo, JERR_NOT_COMPILED);
#endif
  } else {
    diff->pub.consume_data = dummy_consume_data;
    diff->pub._decompress_data = decompress_data;
    diff->whole_image[0] = NULL; /* flag for no virtual arrays */
  }
}

#endif /* D_LOSSLESS_SUPPORTED */
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

- **jpeg_d_coef_controller**: A class/struct defined in this file
- **between**: A class/struct defined in this file

### Functions and Methods

- **struct()**: A function/method defined in this file
- **my_diff_controller()**: A function/method defined in this file
- **D_LOSSLESS_SUPPORTED()**: A function/method defined in this file
- **D_MULTISCAN_FILES_SUPPORTED()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `jlossls.h`
- `jinclude.h`
- `jpeglib.h`

**Python Imports:**
- `the`


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

