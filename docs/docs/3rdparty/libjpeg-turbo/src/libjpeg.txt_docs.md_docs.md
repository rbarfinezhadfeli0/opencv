# Documentation for `docs/3rdparty/libjpeg-turbo/src/libjpeg.txt_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/src/libjpeg.txt_docs.md`
- **File Name**: `libjpeg.txt_docs.md`
- **File Size**: 51,016 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/src/libjpeg.txt_docs.md](../../../../docs/3rdparty/libjpeg-turbo/src/libjpeg.txt_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/src` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/src/libjpeg.txt`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/src/libjpeg.txt`
- **File Name**: `libjpeg.txt`
- **File Size**: 177,182 bytes
- **File Type**: .txt
- **Link to Source**: [3rdparty/libjpeg-turbo/src/libjpeg.txt](../../../3rdparty/libjpeg-turbo/src/libjpeg.txt)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/src` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
USING THE IJG JPEG LIBRARY

This file was part of the Independent JPEG Group's software:
Copyright (C) 1994-2013, Thomas G. Lane, Guido Vollbeding.
Lossless JPEG Modifications:
Copyright (C) 1999, Ken Murchison.
libjpeg-turbo Modifications:
Copyright (C) 2010, 2014-2018, 2020, 2022-2024, D. R. Commander.
Copyright (C) 2015, Google, Inc.
For conditions of distribution and use, see the accompanying README.ijg file.


This file describes how to use the IJG JPEG library within an application
program.  Read it if you want to write a program that uses the library.

The file example.c provides heavily commented code for calling the JPEG
library.  Also see jpeglib.h (the include file to be used by application
programs) for full details about data structures and function parameter lists.
The library source code, of course, is the ultimate reference.

Note that there have been *major* changes from the application interface
presented by IJG version 4 and earlier versions.  The old design had several
inherent limitations, and it had accumulated a lot of cruft as we added
features while trying to minimize application-interface changes.  We have
sacrificed backward compatibility in the version 5 rewrite, but we think the
improvements justify this.


TABLE OF CONTENTS
-----------------

Overview:
        Functions provided by the library
        Data Precision
        Outline of typical usage
Basic library usage:
        Data formats
        Compression details
        Decompression details
        Partial image decompression
        Mechanics of usage: include files, linking, etc
Advanced features:
        Compression parameter selection
        Decompression parameter selection
        Special color spaces
        Error handling
        Compressed data handling (source and destination managers)
        I/O suspension
        Progressive JPEG support
        Buffered-image mode
        Abbreviated datastreams and multiple images
        Special markers
        ICC profiles
        Raw (downsampled) image data
        Really raw data: DCT coefficients
        Progress monitoring
        Memory management
        Memory usage
        Library compile-time options
        Portability considerations

You should read at least the overview and basic usage sections before trying
to program with the library.  The sections on advanced features can be read
if and when you need them.


OVERVIEW
========

Functions provided by the library
---------------------------------

The IJG JPEG library provides C code to read and write JPEG-compressed image
files.  The surrounding application program receives or supplies image data a
scanline at a time, using a straightforward uncompressed image format.  All
details of color conversion and other preprocessing/postprocessing can be
handled by the library.

The library includes a substantial amount of code that is not covered by the
JPEG standard but is necessary for typical applications of JPEG.  These
functions preprocess the image before JPEG compression or postprocess it after
decompression.  They include colorspace conversion, downsampling/upsampling,
and color quantization.  The application indirectly selects use of this code
by specifying the format in which it wishes to supply or receive image data.
For example, if colormapped output is requested, then the decompression
library automatically invokes color quantization.

A wide range of quality vs. speed tradeoffs are possible in JPEG processing,
and even more so in decompression postprocessing.  The decompression library
provides multiple implementations that cover most of the useful tradeoffs,
ranging from very-high-quality down to fast-preview operation.  On the
compression side we have generally not provided low-quality choices, since
compression is normally less time-critical.  It should be understood that the
low-quality modes may not meet the JPEG standard's accuracy requirements;
nonetheless, they are useful for viewers.

A word about functions *not* provided by the library.  We handle a subset of
the ISO JPEG standard; most baseline, extended-sequential, and progressive
JPEG processes are supported.  (Our subset includes all features now in common
use.)  Unsupported ISO options include:
        * Hierarchical storage
        * DNL marker
        * Nonintegral subsampling ratios
We support 8-bit and 12-bit data precision in lossy mode and 2-bit through
16-bit data precision in lossless mode.

By itself, the library handles only interchange JPEG datastreams --- in
particular the widely used JFIF file format.  The library can be used by
surrounding code to process interchange or abbreviated JPEG datastreams that
are embedded in more complex file formats.  (For example, this library is
used by the free LIBTIFF library to support JPEG compression in TIFF.)


Data Precision
--------------

The data precision is the number of bits in the maximum sample value (which may
not be the same as the width of the data type used to store the sample.)  The
JPEG standard provides for, and this code supports, baseline (8-bit-per-sample)
and 12-bit-per-sample DCT processes as well as lossless (predictive) processes
with 2 to 16 bits of data precision per sample.  To create a JPEG image with a
particular data precision, set cinfo->data_precision accordingly.  Note that
9-bit and higher data precisions cause the sample size to be larger than a
char, which affects the surrounding application's image data.  The sample
application djpeg can support 12-bit data precision only for PPM, PGM, and GIF
file formats.  The sample applications cjpeg and djpeg can support 2-bit
through 7-bit and 9-bit through 16-bit data precision only for PPM and PGM file
formats.

Note that, when 12-bit data precision is enabled in lossy mode, the library
compresses in Huffman optimization mode by default, in order to generate valid
Huffman tables.  This is necessary because our default Huffman tables only
cover 8-bit data.  If you need to output 12-bit-per-sample JPEG files in one
pass, you'll have to supply suitable default Huffman tables.  You may also want
to supply your own DCT quantization tables; the existing quality-scaling code
has been developed for 8-bit data precision and probably doesn't generate
especially good tables for 12-bit data precision.

Functions that are specific to 9-bit through 12-bit data precision have a
prefix of "jpeg12_" instead of "jpeg_" and use the following data types and
macros:

  * J12SAMPLE instead of JSAMPLE
  * J12SAMPROW instead of JSAMPROW
  * J12SAMPARRAY instead of JSAMPARRAY
  * J12SAMPIMAGE instead of JSAMPIMAGE
  * MAXJ12SAMPLE instead of MAXJSAMPLE
  * CENTERJ12SAMPLE instead of CENTERJSAMPLE

Functions that are specific to 13-bit through 16-bit data precision have a
prefix of "jpeg16_" instead of "jpeg_" and use the following data types and
macros:

  * J16SAMPLE instead of JSAMPLE
  * J16SAMPROW instead of JSAMPROW
  * J16SAMPARRAY instead of JSAMPARRAY
  * J16SAMPIMAGE instead of JSAMPIMAGE
  * MAXJ16SAMPLE instead of MAXJSAMPLE
  * CENTERJ16SAMPLE instead of CENTERJSAMPLE

This allows multiple data precisions to be used in a single application.
(Refer to example.c).  Arithmetic coding and SIMD acceleration are currently
only implemented for lossy mode with 8-bit data precision.

Refer to the descriptions of the data_precision compression and decompression
parameters below for further information.

This documentation uses "J*SAMPLE", "J*SAMPROW", "J*SAMPARRAY", and
"J*SAMPIMAGE" to generically refer to the data types with 2 to 8 bits of data
precision per sample, 9 to 12 bits of data precision per sample, and 13 to 16
bits of data precision per sample.


Outline of typical usage
------------------------

The rough outline of a JPEG compression operation is:

        Allocate and initialize a JPEG compression object
        Specify the destination for the compressed data (eg, a file)
        Set parameters for compression, including image size & colorspace
        jpeg_start_compress(...);
        while (scan lines remain to be written)
                jpeg_write_scanlines(...);  /* Use jpeg12_write_scanlines() for
                                               9-bit through 12-bit data
                                               precision and
                                               jpeg16_write_scanlines() for
                                               13-bit through 16-bit data
                                               precision. */
        jpeg_finish_compress(...);
        Release the JPEG compression object

A JPEG compression object holds parameters and working state for the JPEG
library.  We make creation/destruction of the object separate from starting
or finishing compression of an image; the same object can be re-used for a
series of image compression operations.  This makes it easy to re-use the
same parameter settings for a sequence of images.  Re-use of a JPEG object
also has important implications for processing abbreviated JPEG datastreams,
as discussed later.

The image data to be compressed is supplied to jpeg*_write_scanlines() from
in-memory buffers.  If the application is doing file-to-file compression,
reading image data from the source file is the application's responsibility.
The library emits compressed data by calling a "data destination manager",
which typically will write the data into a file; but the application can
provide its own destination manager to do something else.

Similarly, the rough outline of a JPEG decompression operation is:

        Allocate and initialize a JPEG decompression object
        Specify the source of the compressed data (eg, a file)
        Call jpeg_read_header() to obtain image info
        Set parameters for decompression
        jpeg_start_decompress(...);
        while (scan lines remain to be read)
                jpeg_read_scanlines(...);  /* Use jpeg12_read_scanlines() for
                                              9-bit through 12-bit data
                                              precision and
                                              jpeg16_read_scanlines() for
                                              13-bit through 16-bit data
                                              precision. */
        jpeg_finish_decompress(...);
        Release the JPEG decompression object

This is comparable to the compression outline except that reading the
datastream header is a separate step.  This is helpful because information
about the image's size, colorspace, etc is available when the application
selects decompression parameters.  For example, the application can choose an
output scaling ratio that will fit the image into the available screen size.

The decompression library obtains compressed data by calling a data source
manager, which typically will read the data from a file; but other behaviors
can be obtained with a custom source manager.  Decompressed data is delivered
into in-memory buffers passed to jpeg*_read_scanlines().

It is possible to abort an incomplete compression or decompression operation
by calling jpeg_abort(); or, if you do not need to retain the JPEG object,
simply release it by calling jpeg_destroy().

JPEG compression and decompression objects are two separate struct types.
However, they share some common fields, and certain routines such as
jpeg_destroy() can work on either type of object.

The JPEG library has no static variables: all state is in the compression
or decompression object.  Therefore it is possible to process multiple
compression and decompression operations concurrently, using multiple JPEG
objects.

Both compression and decompression can be done in an incremental memory-to-
memory fashion, if suitable source/destination managers are used.  See the
section on "I/O suspension" for more details.


BASIC LIBRARY USAGE
===================

Data formats
------------

Before diving into procedural details, it is helpful to understand the
image data format that the JPEG library expects or returns.

The standard input image format is a rectangular array of pixels, with each
pixel having the same number of "component" or "sample" values (color
channels).  You must specify how many components there are and the colorspace
interpretation of the components.  Most applications will use RGB data
(three components per pixel) or grayscale data (one component per pixel).
PLEASE NOTE THAT RGB DATA IS THREE SAMPLES PER PIXEL, GRAYSCALE ONLY ONE.
A remarkable number of people manage to miss this, only to find that their
programs don't work with grayscale JPEG files.

There is no provision for colormapped input.  JPEG files are always full-color
or full grayscale (or sometimes another colorspace such as CMYK).  You can
feed in a colormapped image by expanding it to full-color format.  However
JPEG often doesn't work very well with source data that has been colormapped,
because of dithering noise.  This is discussed in more detail in the JPEG FAQ
and the other references mentioned in the README.ijg file.

Pixels are stored by scanlines, with each scanline running from left to
right.  The component values for each pixel are adjacent in the row; for
example, R,G,B,R,G,B,R,G,B,... for 24-bit RGB color.  Each scanline is an
array of data type JSAMPLE, J12SAMPLE, or J16SAMPLE --- which is typically
"unsigned char", "short", or "unsigned short" (respectively) unless you've
changed jmorecfg.h.

A 2-D array of pixels is formed by making a list of pointers to the starts of
scanlines; so the scanlines need not be physically adjacent in memory.  Even
if you process just one scanline at a time, you must make a one-element
pointer array to conform to this structure.  Pointers to J*SAMPLE rows are of
type J*SAMPROW, and the pointer to the pointer array is of type J*SAMPARRAY.

The library accepts or supplies one or more complete scanlines per call.
It is not possible to process part of a row at a time.  Scanlines are always
processed top-to-bottom.  You can process an entire image in one call if you
have it all in memory, but usually it's simplest to process one scanline at
a time.


The data format returned by the decompressor is the same in all details,
except that colormapped output is supported.  (Again, a JPEG file is never
colormapped.  But you can ask the decompressor to perform on-the-fly color
quantization to deliver colormapped output.)  If you request colormapped
output then the returned data array contains a single J*SAMPLE per pixel;
its value is an index into a color map.  The color map is represented as
a 2-D J*SAMPARRAY in which each row holds the values of one color component,
that is, colormap[i][j] is the value of the i'th color component for pixel
value (map index) j.  Note that since the colormap indexes are stored in
J*SAMPLEs, the maximum number of colors is limited by the size of J*SAMPLE
(ie, at most 256 colors for 8-bit data precision and 4096 colors for 12-bit
data precision).


Compression details
-------------------

Here we revisit the JPEG compression outline given in the overview.

1. Allocate and initialize a JPEG compression object.

A JPEG compression object is a "struct jpeg_compress_struct".  (It also has
a bunch of subsidiary structures which are allocated via malloc(), but the
application doesn't control those directly.)  This struct can be just a local
variable in the calling routine, if a single routine is going to execute the
whole JPEG compression sequence.  Otherwise it can be static or allocated
from malloc().

You will also need a structure representing a JPEG error handler.  The part
of this that the library cares about is a "struct jpeg_error_mgr".  If you
are providing your own error handler, you'll typically want to embed the
jpeg_error_mgr struct in a larger structure; this is discussed later under
"Error handling".  For now we'll assume you are just using the default error
handler.  The default error handler will print JPEG error/warning messages
on stderr, and it will call exit() if a fatal error occurs.

You must initialize the error handler structure, store a pointer to it into
the JPEG object's "err" field, and then call jpeg_create_compress() to
initialize the rest of the JPEG object.

Typical code for this step, if you are using the default error handler, is

        struct jpeg_compress_struct cinfo;
        struct jpeg_error_mgr jerr;
        ...
        cinfo.err = jpeg_std_error(&jerr);
        jpeg_create_compress(&cinfo);

jpeg_create_compress allocates a small amount of memory, so it could fail
if you are out of memory.  In that case it will exit via the error handler;
that's why the error handler must be initialized first.


2. Specify the destination for the compressed data (eg, a file).

As previously mentioned, the JPEG library delivers compressed data to a
"data destination" module.  The library includes one data destination
module which knows how to write to a stdio stream.  You can use your own
destination module if you want to do something else, as discussed later.

If you use the standard destination module, you must open the target stdio
stream beforehand.  Typical code for this step looks like:

        FILE *outfile;
        ...
        if ((outfile = fopen(filename, "wb")) == NULL) {
            fprintf(stderr, "can't open %s\n", filename);
            exit(1);
        }
        jpeg_stdio_dest(&cinfo, outfile);

where the last line invokes the standard destination module.

WARNING: it is critical that the binary compressed data be delivered to the
output file unchanged.  On non-Unix systems the stdio library may perform
newline translation or otherwise corrupt binary data.  To suppress this
behavior, you may need to use a "b" option to fopen (as shown above), or use
setmode() or another routine to put the stdio stream in binary mode.  See
cjpeg.c and djpeg.c for code that has been found to work on many systems.

You can select the data destination after setting other parameters (step 3),
if that's more convenient.  You may not change the destination between
calling jpeg_start_compress() and jpeg_finish_compress().


3. Set parameters for compression, including image size & colorspace.

You must supply information about the source image by setting the following
fields in the JPEG object (cinfo structure):

        image_width             Width of image, in pixels
        image_height            Height of image, in pixels
        input_components        Number of color channels (samples per pixel)
        in_color_space          Color space of source image

The image dimensions are, hopefully, obvious.  JPEG supports image dimensions
of 1 to 64K pixels in either direction.  The input color space is typically
RGB or grayscale, and input_components is 3 or 1 accordingly.  (See "Special
color spaces", later, for more info.)  The in_color_space field must be
assigned one of the J_COLOR_SPACE enum constants, typically JCS_RGB or
JCS_GRAYSCALE.

JPEG has a large number of compression parameters that determine how the
image is encoded.  Most applications don't need or want to know about all
these parameters.  You can set all the parameters to reasonable defaults by
calling jpeg_set_defaults(); then, if there are particular values you want
to change, you can do so after that.  The "Compression parameter selection"
section tells about all the parameters.

You must set in_color_space correctly before calling jpeg_set_defaults(),
because the defaults depend on the source image colorspace.  However the
other three source image parameters need not be valid until you call
jpeg_start_compress().  There's no harm in calling jpeg_set_defaults() more
than once, if that happens to be convenient.

Typical code for a 24-bit RGB source image is

        cinfo.image_width = Width;      /* image width and height, in pixels */
        cinfo.image_height = Height;
        cinfo.input_components = 3;     /* # of color components per pixel */
        cinfo.in_color_space = JCS_RGB; /* colorspace of input image */

        jpeg_set_defaults(&cinfo);
        /* Make optional parameter settings here */


4. jpeg_start_compress(...);

After you have established the data destination and set all the necessary
source image info and other parameters, call jpeg_start_compress() to begin
a compression cycle.  This will initialize internal state, allocate working
storage, and emit the first few bytes of the JPEG datastream header.

Typical code:

        jpeg_start_compress(&cinfo, TRUE);

The "TRUE" parameter ensures that a complete JPEG interchange datastream
will be written.  This is appropriate in most cases.  If you think you might
want to use an abbreviated datastream, read the section on abbreviated
datastreams, below.

Once you have called jpeg_start_compress(), you may not alter any JPEG
parameters or other fields of the JPEG object until you have completed
the compression cycle.


5. while (scan lines remain to be written)
        jpeg_write_scanlines(...);  /* Use jpeg12_write_scanlines() for 9-bit
                                       through 12-bit data precision and
                                       jpeg16_write_scanlines() for 13-bit
                                       through 16-bit data precision. */

Now write all the required image data by calling jpeg*_write_scanlines()
one or more times.  You can pass one or more scanlines in each call, up
to the total image height.  In most applications it is convenient to pass
just one or a few scanlines at a time.  The expected format for the passed
data is discussed under "Data formats", above.

Image data should be written in top-to-bottom scanline order.
Rec. ITU-T T.81 | ISO/IEC 10918-1 says, "Applications determine which edges of
a source image are defined as top, bottom, left, and right."  However, if you
want your files to be compatible with everyone else's, then top-to-bottom order
must be used.  If the source data must be read in bottom-to-top order, then you
can use the JPEG library's virtual array mechanism to invert the data
efficiently.  Examples of this can be found in the sample application cjpeg.

The library maintains a count of the number of scanlines written so far
in the next_scanline field of the JPEG object.  Usually you can just use
this variable as the loop counter, so that the loop test looks like
"while (cinfo.next_scanline < cinfo.image_height)".

Code for this step depends heavily on the way that you store the source data.
example.c shows the following code for the case of a full-size 2-D source
array containing 3-byte RGB pixels:

        JSAMPROW row_pointer[1];        /* pointer to a single row
                                           Use J12SAMPROW for 9-bit through
                                           12-bit data precision and J16SAMPROW
                                           for 13-bit through 16-bit data
                                           precision. */

        while (cinfo.next_scanline < cinfo.image_height) {
            row_pointer[0] = image_buffer[cinfo.next_scanline];
            jpeg_write_scanlines(&cinfo, row_pointer, 1);
                                        /* Use jpeg12_write_scanlines() for
                                           9-bit through 12-bit data precision
                                           and jpeg16_write_scanlines() for
                                           13-bit through 16-bit data
                                           precision. */
        }

jpeg*_write_scanlines() returns the number of scanlines actually written.
This will normally be equal to the number passed in, so you can usually
ignore the return value.  It is different in just two cases:
  * If you try to write more scanlines than the declared image height,
    the additional scanlines are ignored.
  * If you use a suspending data destination manager, output buffer overrun
    will cause the compressor to return before accepting all the passed lines.
    This feature is discussed under "I/O suspension", below.  The normal
    stdio destination manager will NOT cause this to happen.
In any case, the return value is the same as the change in the value of
next_scanline.


6. jpeg_finish_compress(...);

After all the image data has been written, call jpeg_finish_compress() to
complete the compression cycle.  This step is ESSENTIAL to ensure that the
last bufferload of data is written to the data destination.
jpeg_finish_compress() also releases working memory associated with the JPEG
object.

Typical code:

        jpeg_finish_compress(&cinfo);

If using the stdio destination manager, don't forget to close the output
stdio stream (if necessary) afterwards.

If you have requested a multi-pass operating mode, such as Huffman code
optimization, jpeg_finish_compress() will perform the additional passes using
data buffered by the first pass.  In this case jpeg_finish_compress() may take
quite a while to complete.  With the default compression parameters, this will
not happen.

It is an error to call jpeg_finish_compress() before writing the necessary
total number of scanlines.  If you wish to abort compression, call
jpeg_abort() as discussed below.

After completing a compression cycle, you may dispose of the JPEG object
as discussed next, or you may use it to compress another image.  In that case
return to step 2, 3, or 4 as appropriate.  If you do not change the
destination manager, the new datastream will be written to the same target.
If you do not change any JPEG parameters, the new datastream will be written
with the same parameters as before.  Note that you can change the input image
dimensions freely between cycles, but if you change the input colorspace, you
should call jpeg_set_defaults() to adjust for the new colorspace; and then
you'll need to repeat all of step 3.


7. Release the JPEG compression object.

When you are done with a JPEG compression object, destroy it by calling
jpeg_destroy_compress().  This will free all subsidiary memory (regardless of
the previous state of the object).  Or you can call jpeg_destroy(), which
works for either compression or decompression objects --- this may be more
convenient if you are sharing code between compression and decompression
cases.  (Actually, these routines are equivalent except for the declared type
of the passed pointer.  To avoid gripes from ANSI C compilers, jpeg_destroy()
should be passed a j_common_ptr.)

If you allocated the jpeg_compress_struct structure from malloc(), freeing
it is your responsibility --- jpeg_destroy() won't.  Ditto for the error
handler structure.

Typical code:

        jpeg_destroy_compress(&cinfo);


8. Aborting.

If you decide to abort a compression cycle before finishing, you can clean up
in either of two ways:

* If you don't need the JPEG object any more, just call
  jpeg_destroy_compress() or jpeg_destroy() to release memory.  This is
  legitimate at any point after calling jpeg_create_compress() --- in fact,
  it's safe even if jpeg_create_compress() fails.

* If you want to re-use the JPEG object, call jpeg_abort_compress(), or call
  jpeg_abort() which works on both compression and decompression objects.
  This will return the object to an idle state, releasing any working memory.
  jpeg_abort() is allowed at any time after successful object creation.

Note that cleaning up the data destination, if required, is your
responsibility; neither of these routines will call term_destination().
(See "Compressed data handling", below, for more about that.)

jpeg_destroy() and jpeg_abort() are the only safe calls to make on a JPEG
object that has reported an error by calling error_exit (see "Error handling"
for more info).  The internal state of such an object is likely to be out of
whack.  Either of these two routines will return the object to a known state.


Decompression details
---------------------

Here we revisit the JPEG decompression outline given in the overview.

1. Allocate and initialize a JPEG decompression object.

This is just like initialization for compression, as discussed above,
except that the object is a "struct jpeg_decompress_struct" and you
call jpeg_create_decompress().  Error handling is exactly the same.

Typical code:

        struct jpeg_decompress_struct cinfo;
        struct jpeg_error_mgr jerr;
        ...
        cinfo.err = jpeg_std_error(&jerr);
        jpeg_create_decompress(&cinfo);

(Both here and in the IJG code, we usually use variable name "cinfo" for
both compression and decompression objects.)


2. Specify the source of the compressed data (eg, a file).

As previously mentioned, the JPEG library reads compressed data from a "data
source" module.  The library includes one data source module which knows how
to read from a stdio stream.  You can use your own source module if you want
to do something else, as discussed later.

If you use the standard source module, you must open the source stdio stream
beforehand.  Typical code for this step looks like:

        FILE *infile;
        ...
        if ((infile = fopen(filename, "rb")) == NULL) {
            fprintf(stderr, "can't open %s\n", filename);
            exit(1);
        }
        jpeg_stdio_src(&cinfo, infile);

where the last line invokes the standard source module.

WARNING: it is critical that the binary compressed data be read unchanged.
On non-Unix systems the stdio library may perform newline translation or
otherwise corrupt binary data.  To suppress this behavior, you may need to use
a "b" option to fopen (as shown above), or use setmode() or another routine to
put the stdio stream in binary mode.  See cjpeg.c and djpeg.c for code that
has been found to work on many systems.

You may not change the data source between calling jpeg_read_header() and
jpeg_finish_decompress().  If you wish to read a series of JPEG images from
a single source file, you should repeat the jpeg_read_header() to
jpeg_finish_decompress() sequence without reinitializing either the JPEG
object or the data source module; this prevents buffered input data from
being discarded.


3. Call jpeg_read_header() to obtain image info.

Typical code for this step is just

        jpeg_read_header(&cinfo, TRUE);

This will read the source datastream header markers, up to the beginning
of the compressed data proper.  On return, the image dimensions and other
info have been stored in the JPEG object.  The application may wish to
consult this information before selecting decompression parameters.

More complex code is necessary if
  * A suspending data source is used --- in that case jpeg_read_header()
    may return before it has read all the header data.  See "I/O suspension",
    below.  The normal stdio source manager will NOT cause this to happen.
  * Abbreviated JPEG files are to be processed --- see the section on
    abbreviated datastreams.  Standard applications that deal only in
    interchange JPEG files need not be concerned with this case either.

It is permissible to stop at this point if you just wanted to find out the
image dimensions and other header info for a JPEG file.  In that case,
call jpeg_destroy() when you are done with the JPEG object, or call
jpeg_abort() to return it to an idle state before selecting a new data
source and reading another header.


4. Set parameters for decompression.

jpeg_read_header() sets appropriate default decompression parameters based on
the properties of the image (in particular, its colorspace).  However, you
may well want to alter these defaults before beginning the decompression.
For example, the default is to produce full color output from a color file.
If you want colormapped output you must ask for it.  Other options allow the
returned image to be scaled and allow various speed/quality tradeoffs to be
selected.  "Decompression parameter selection", below, gives details.

If the defaults are appropriate, nothing need be done at this step.

Note that all default values are set by each call to jpeg_read_header().
If you reuse a decompression object, you cannot expect your parameter
settings to be preserved across cycles, as you can for compression.
You must set desired parameter values each time.


5. jpeg_start_decompress(...);

Once the parameter values are satisfactory, call jpeg_start_decompress() to
begin decompression.  This will initialize internal state, allocate working
memory, and prepare for returning data.

Typical code is just

        jpeg_start_decompress(&cinfo);

If you have requested a multi-pass operating mode, such as 2-pass color
quantization, jpeg_start_decompress() will do everything needed before data
output can begin.  In this case jpeg_start_decompress() may take quite a while
to complete.  With a single-scan (non progressive) JPEG file and default
decompression parameters, this will not happen; jpeg_start_decompress() will
return quickly.

After this call, the final output image dimensions, including any requested
scaling, are available in the JPEG object; so is the selected colormap, if
colormapped output has been requested.  Useful fields include

        output_width            image width and height, as scaled
        output_height
        out_color_components    # of color components in out_color_space
        output_components       # of color components returned per pixel
        colormap                the selected colormap, if any
        actual_number_of_colors         number of entries in colormap

output_components is 1 (a colormap index) when quantizing colors; otherwise it
equals out_color_components.  It is the number of J*SAMPLE values that will be
emitted per pixel in the output arrays.

Typically you will need to allocate data buffers to hold the incoming image.
You will need output_width * output_components J*SAMPLEs per scanline in your
output buffer, and a total of output_height scanlines will be returned.

Note: if you are using the JPEG library's internal memory manager to allocate
data buffers (as djpeg does), then the manager's protocol requires that you
request large buffers *before* calling jpeg_start_decompress().  This is a
little tricky since the output_XXX fields are not normally valid then.  You
can make them valid by calling jpeg_calc_output_dimensions() after setting the
relevant parameters (scaling, output color space, and quantization flag).


6. while (scan lines remain to be read)
        jpeg_read_scanlines(...);  /* Use jpeg12_read_scanlines() for 9-bit
                                      through 12-bit data precision and
                                      jpeg16_read_scanlines() for 13-bit
                                      through 16-bit data precision. */

Now you can read the decompressed image data by calling jpeg*_read_scanlines()
one or more times.  At each call, you pass in the maximum number of scanlines
to be read (ie, the height of your working buffer); jpeg*_read_scanlines()
will return up to that many lines.  The return value is the number of lines
actually read.  The format of the returned data is discussed under "Data
formats", above.  Don't forget that grayscale and color JPEGs will return
different data formats!

Image data is returned in top-to-bottom scanline order.  If you must write
out the image in bottom-to-top order, you can use the JPEG library's virtual
array mechanism to invert the data efficiently.  Examples of this can be
found in the sample application djpeg.

The library maintains a count of the number of scanlines returned so far
in the output_scanline field of the JPEG object.  Usually you can just use
this variable as the loop counter, so that the loop test looks like
"while (cinfo.output_scanline < cinfo.output_height)".  (Note that the test
should NOT be against image_height, unless you never use scaling.  The
image_height field is the height of the original unscaled image.)
The return value always equals the change in the value of output_scanline.

If you don't use a suspending data source, it is safe to assume that
jpeg*_read_scanlines() reads at least one scanline per call, until the
bottom of the image has been reached.

If you use a buffer larger than one scanline, it is NOT safe to assume that
jpeg*_read_scanlines() fills it.  (The current implementation returns only a
few scanlines per call, no matter how large a buffer you pass.)  So you must
always provide a loop that calls jpeg*_read_scanlines() repeatedly until the
whole image has been read.


7. jpeg_finish_decompress(...);

After all the image data has been read, call jpeg_finish_decompress() to
complete the decompression cycle.  This causes working memory associated
with the JPEG object to be released.

Typical code:

        jpeg_finish_decompress(&cinfo);

If using the stdio source manager, don't forget to close the source stdio
stream if necessary.

It is an error to call jpeg_finish_decompress() before reading the correct
total number of scanlines.  If you wish to abort decompression, call
jpeg_abort() as discussed below.

After completing a decompression cycle, you may dispose of the JPEG object as
discussed next, or you may use it to decompress another image.  In that case
return to step 2 or 3 as appropriate.  If you do not change the source
manager, the next image will be read from the same source.


8. Release the JPEG decompression object.

When you are done with a JPEG decompression object, destroy it by calling
jpeg_destroy_decompress() or jpeg_destroy().  The previous discussion of
destroying compression objects applies here too.

Typical code:

        jpeg_destroy_decompress(&cinfo);


9. Aborting.

You can abort a decompression cycle by calling jpeg_destroy_decompress() or
jpeg_destroy() if you don't need the JPEG object any more, or
jpeg_abort_decompress() or jpeg_abort() if you want to reuse the object.
The previous discussion of aborting compression cycles applies here too.


Partial image decompression
---------------------------

Partial image decompression is convenient for performance-critical applications
that wish to view only a portion of a large JPEG image without decompressing
the whole thing.  It it also useful in memory-constrained environments (such as
on mobile devices.)  This library provides the following functions to support
partial image decompression:

1. Skipping rows when decompressing

        jpeg_skip_scanlines(j_decompress_ptr cinfo, JDIMENSION num_lines);
                /* Use jpeg12_skip_scanlines() for 12-bit data precision. */

This function provides application programmers with the ability to skip over
multiple rows in the JPEG image.

Suspending data sources are not supported by this function.  Calling
jpeg*_skip_scanlines() with a suspending data source will result in undefined
behavior.  Two-pass color quantization is also not supported by this function.
Calling jpeg*_skip_scanlines() with two-pass color quantization enabled will
result in an error.

jpeg*_skip_scanlines() will not allow skipping past the bottom of the image.
If the value of num_lines is large enough to skip past the bottom of the image,
then the function will skip to the end of the image instead.

If the value of num_lines is valid, then jpeg*_skip_scanlines() will always
skip all of the input rows requested.  There is no need to inspect the return
value of the function in that case.

Best results will be achieved by calling jpeg*_skip_scanlines() for large
chunks of rows.  The function should be viewed as a way to quickly jump to a
particular vertical offset in the JPEG image in order to decode a subset of the
image.  Used in this manner, it will provide significant performance
improvements.

Calling jpeg*_skip_scanlines() for small values of num_lines has several
potential drawbacks:
    1) JPEG decompression occurs in blocks, so if jpeg*_skip_scanlines() is
       called from the middle of a decompression block, then it is likely that
       much of the decompression work has already been done for the first
       couple of rows that need to be skipped.
    2) When this function returns, it must leave the decompressor in a state
       such that it is ready to read the next line.  This may involve
       decompressing a block that must be partially skipped.
These issues are especially tricky for cases in which upsampling requires
context rows.  In the worst case, jpeg*_skip_scanlines() will perform similarly
to jpeg*_read_scanlines() (since it will actually call jpeg*_read_scanlines().)

2. Decompressing partial scanlines

        jpeg_crop_scanline (j_decompress_ptr cinfo, JDIMENSION *xoffset,
                            JDIMENSION *width)
                /* Use jpeg12_crop_scanline() for 12-bit data precision. */

This function provides application programmers with the ability to decompress
only a portion of each row in the JPEG image.  It must be called after
jpeg_start_decompress() and before any calls to jpeg*_read_scanlines() or
jpeg*_skip_scanlines().

If xoffset and width do not form a valid subset of the image row, then this
function will generate an error.  Note that if the output image is scaled, then
xoffset and width are relative to the scaled image dimensions.

xoffset and width are passed by reference because xoffset must fall on an iMCU
boundary.  If it doesn't, then it will be moved left to the nearest iMCU
boundary, and width will be increased accordingly.  If the calling program does
not like the adjusted values of xoffset and width, then it can call
jpeg*_crop_scanline() again with new values (for instance, if it wants to move
xoffset to the nearest iMCU boundary to the right instead of to the left.)

After calling this function, cinfo->output_width will be set to the adjusted
width.  This value should be used when allocating an output buffer to pass to
jpeg*_read_scanlines().

The output image from a partial-width decompression will be identical to the
corresponding image region from a full decode, with one exception:  The "fancy"
(smooth) h2v2 (4:2:0) and h2v1 (4:2:2) upsampling algorithms fill in the
missing chroma components by averaging the chroma components from neighboring
pixels, except on the right and left edges of the image (where there are no
neighboring pixels.)  When performing a partial-width decompression, these
"fancy" upsampling algorithms may treat the left and right edges of the partial
image region as if they are the left and right edges of the image, meaning that
the upsampling algorithm may be simplified.  The result is that the pixels on
the left or right edge of the partial image may not be exactly identical to the
corresponding pixels in the original image.


Mechanics of usage: include files, linking, etc
-----------------------------------------------

Applications using the JPEG library should include the header file jpeglib.h
to obtain declarations of data types and routines.  Before including
jpeglib.h, include system headers that define at least the typedefs FILE and
size_t.  On ANSI-conforming systems, including <stdio.h> is sufficient; on
older Unix systems, you may need <sys/types.h> to define size_t.

If the application needs to refer to individual JPEG library error codes, also
include jerror.h to define those symbols.

jpeglib.h indirectly includes the files jconfig.h and jmorecfg.h.  If you are
installing the JPEG header files in a system directory, you will want to
install all four files: jpeglib.h, jerror.h, jconfig.h, jmorecfg.h.

The most convenient way to include the JPEG code into your executable program
is to prepare a library file ("libjpeg.a", or a corresponding name on non-Unix
machines) and reference it at your link step.  If you use only half of the
library (only compression or only decompression), only that much code will be
included from the library, unless your linker is hopelessly brain-damaged.
The supplied build system builds libjpeg.a automatically.

It may be worth pointing out that the core JPEG library does not actually
require the stdio library: only the default source/destination managers and
error handler need it.  You can use the library in a stdio-less environment
if you replace those modules and use jmemnobs.c (or another memory manager of
your own devising).  More info about the minimum system library requirements
may be found in jinclude.h.


ADVANCED FEATURES
=================

Compression parameter selection
-------------------------------

This section describes all the optional parameters you can set for JPEG
compression, as well as the "helper" routines provided to assist in this
task.  Proper setting of some parameters requires detailed understanding
of the JPEG standard; if you don't know what a parameter is for, it's best
not to mess with it!  See REFERENCES in the README.ijg file for pointers to
more info about JPEG.

It's a good idea to call jpeg_set_defaults() first, even if you plan to set
all the parameters; that way your code is more likely to work with future JPEG
libraries that have additional parameters.  For the same reason, we recommend
you use a helper routine where one is provided, in preference to twiddling
cinfo fields directly.

The helper routines are:

jpeg_set_defaults (j_compress_ptr cinfo)
        This routine sets all JPEG parameters to reasonable defaults, using
        only the input image's color space (field in_color_space, which must
        already be set in cinfo).  Many applications will only need to use
        this routine and perhaps jpeg_set_quality().

jpeg_set_colorspace (j_compress_ptr cinfo, J_COLOR_SPACE colorspace)
        Sets the JPEG file's colorspace (field jpeg_color_space) as specified,
        and sets other color-space-dependent parameters appropriately.  See
        "Special color spaces", below, before using this.  A large number of
        parameters, including all per-component parameters, are set by this
        routine; if you want to twiddle individual parameters you should call
        jpeg_set_colorspace() before rather than after.

jpeg_default_colorspace (j_compress_ptr cinfo)
        Selects an appropriate JPEG colorspace based on cinfo->in_color_space,
        and calls jpeg_set_colorspace().  This is actually a subroutine of
        jpeg_set_defaults().  It's broken out in case you want to change
        just the colorspace-dependent JPEG parameters.

jpeg_set_quality (j_compress_ptr cinfo, int quality, boolean force_baseline)
        Constructs JPEG quantization tables appropriate for the indicated
        quality setting.  The quality value is expressed on the 0..100 scale
        recommended by IJG (cjpeg's "-quality" switch uses this routine).
        Note that the exact mapping from quality values to tables may change
        in future IJG releases as more is learned about DCT quantization.
        If the force_baseline parameter is TRUE, then the quantization table
        entries are constrained to the range 1..255 for full JPEG baseline
        compatibility.  In the current implementation, this only makes a
        difference for quality settings below 25, and it effectively prevents
        very small/low-quality files from being generated.  The IJG decoder
        is capable of reading the non-baseline files generated at low quality
        settings when force_baseline is FALSE, but other decoders may not be.

jpeg_set_linear_quality (j_compress_ptr cinfo, int scale_factor,
                         boolean force_baseline)
        Same as jpeg_set_quality() except that the generated tables are the
        sample tables given in Annex K (Clause K.1) of
        Rec. ITU-T T.81 (1992) | ISO/IEC 10918-1:1994, multiplied by the
        specified scale factor (which is expressed as a percentage; thus
        scale_factor = 100 reproduces the spec's tables).  Note that larger
        scale factors give lower quality.  This entry point is useful for
        conforming to the Adobe PostScript DCT conventions, but we do not
        recommend linear scaling as a user-visible quality scale otherwise.
        force_baseline again constrains the computed table entries to 1..255.

int jpeg_quality_scaling (int quality)
        Converts a value on the IJG-recommended quality scale to a linear
        scaling percentage.  Note that this routine may change or go away
        in future releases --- IJG may choose to adopt a scaling method that
        can't be expressed as a simple scalar multiplier, in which case the
        premise of this routine collapses.  Caveat user.

jpeg_default_qtables (j_compress_ptr cinfo, boolean force_baseline)
        [libjpeg v7+ API/ABI emulation only]
        Set default quantization tables with linear q_scale_factor[] values
        (see below).

jpeg_add_quant_table (j_compress_ptr cinfo, int which_tbl,
                      const unsigned int *basic_table,
                      int scale_factor, boolean force_baseline)
        Allows an arbitrary quantization table to be created.  which_tbl
        indicates which table slot to fill.  basic_table points to an array
        of 64 unsigned ints given in normal array order.  These values are
        multiplied by scale_factor/100 and then clamped to the range 1..65535
        (or to 1..255 if force_baseline is TRUE).
        CAUTION: prior to library version 6a, jpeg_add_quant_table expected
        the basic table to be given in JPEG zigzag order.  If you need to
        write code that works with either older or newer versions of this
        routine, you must check the library version number.  Something like
        "#if JPEG_LIB_VERSION >= 61" is the right test.

jpeg_simple_progression (j_compress_ptr cinfo)
        Generates a default scan script for writing a progressive-JPEG file.
        This is the recommended method of creating a progressive file,
        unless you want to make a custom scan sequence.  You must ensure that
        the JPEG color space is set correctly before calling this routine.

jpeg_enable_lossless (j_compress_ptr cinfo, int predictor_selection_value,
                      int point_transform)
        Enables lossless mode with the 
... [truncated]
```

## Purpose

This configuration file is used to control build settings, dependencies, or runtime behavior of the OpenCV library.

## Key Settings

Configuration files in OpenCV typically control:
- Build system configuration (CMake)
- Compiler flags and options
- Feature enablement/disablement
- Path specifications
- Version information
- Dependency management

## Usage

This file is processed during the build configuration phase or at runtime to customize OpenCV behavior.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

