# Documentation for `3rdparty/libtiff/ChangeLog`

## File Metadata

- **Full Path**: `3rdparty/libtiff/ChangeLog`
- **File Name**: `ChangeLog`
- **File Size**: 636,104 bytes
- **File Type**: no extension
- **Link to Source**: [3rdparty/libtiff/ChangeLog](../../3rdparty/libtiff/ChangeLog)

## Purpose and Role

This file is located in the `3rdparty/libtiff` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
2025-09-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'raw2tiff-742' into 'master'
	raw2tiff: close input file before exit; issue #742

	See merge request libtiff/libtiff!761

2025-09-08  Lee Howard  <faxguy@howardsilvan.com>

	raw2tiff: close input file before exit; issue #742.

2025-09-06  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'doc_add_LZW_BSD_license_to_LICENSE_md_issue_431' into 'master'
	Add BSD license for Lempel-Ziv & Welch compression (tif_lzw.c) to LICENSE.md

	Closes #431

	See merge request libtiff/libtiff!760

2025-09-06  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'formatting_issues' into 'master'
	Fix formatting issues

	See merge request libtiff/libtiff!759

2025-09-05  Even Rouault  <even.rouault@spatialys.com>

	Fix formatting issues.

2025-09-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiff2pdf-741' into 'master'
	avoid null pointer dereference, tiff2pdf #741

	See merge request libtiff/libtiff!758

2025-09-05  Lee Howard  <faxguy@howardsilvan.com>

	tiff2pdf: avoid null pointer dereference.
	Fixes #741

2025-09-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffcrop-740' into 'master'
	avoid buffer overflow in tiffcrop #740

	See merge request libtiff/libtiff!757

2025-09-05  Lee Howard  <faxguy@howardsilvan.com>

	tiffcrop: avoid buffer overflow.
	Fixes #740

2025-09-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffmedian-735' into 'master'
	close tiffmedian input file - #735

	See merge request libtiff/libtiff!756

2025-09-05  Lee Howard  <faxguy@howardsilvan.com>

	tiffmedian: close input file.
	Fixes #735

2025-09-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffcrop-734' into 'master'
	Tiffcrop 734

	See merge request libtiff/libtiff!755

2025-09-05  Lee Howard  <faxguy@howardsilvan.com>

	tiffcrop: avoid nullptr dereference.
	Fixes #734

2025-09-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffdither-733' into 'master'
	avoid out-of-bounds read identified in #733

	See merge request libtiff/libtiff!754

2025-09-05  Lee Howard  <faxguy@howardsilvan.com>

	tiffdither: avoid out-of-bounds read identified in #733.

2025-09-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffcrop-721' into 'master'
	fix double-free and memory leak exposed by issue #721

	See merge request libtiff/libtiff!753

2025-09-05  Lee Howard  <faxguy@howardsilvan.com>

	tiffcrop: fix double-free and memory leak exposed by issue #721.

2025-09-04  Su_Laus  <sulau@freenet.de>

	Add BSD license for Lempel-Ziv & Welch compression (tif_lzw.c) to LICENSE.md
	Closes #431

2025-09-04  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_54_read_CCITT_FAX_group3_without_EOL' into 'master'
	tif_fax3: For fax group 3 data if no EOL is detected, reading is retried without synchronisation for EOLs.

	Closes #54

	LibTIFF can write (encode) faxes of group 3 1d without EOL, but cannot read (decode) such files. This function has been added.

	See merge request libtiff/libtiff!752

2025-09-04  Su Laus  <sulau@freenet.de>

	tif_fax3: For fax group 3 data if no EOL is detected, reading is retried without synchronisation for EOLs.
	Closes #54

	LibTIFF can write (encode) faxes of group 3 1d without EOL, but cannot read (decode) such files. This function has been added.

2025-09-04  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'no-public-fix' into 'master'
	tiffio.h: fix compilation with LOGLUV_PUBLIC=0

	See merge request libtiff/libtiff!748

2025-08-19  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'doc_returnvalue_mapproc_issue_#12' into 'master'
	doc: update TIFFOpen.rst with the return values of mapproc and unmapproc.

	Closes #12

	See merge request libtiff/libtiff!749

2025-08-19  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffcp_preset_level' into 'master'
	tiffcp: fix setting compression level for lossless codecs

	Closes #730

	See merge request libtiff/libtiff!750

2025-08-19  Jan Musinsky  <musinsky@gmail.com>

	tiffcp: fix setting compression level for lossless codecs.
	Closes #730

2025-08-17  Su_Laus  <sulau@freenet.de>

	doc: update TIFFOpen.rst with the return values of mapproc and unmapproc.
	Closes #12

2025-08-12  Tim Blechmann  <tim@klingt.org>

	tiffio.h: fix compilation with LOGLUV_PUBLIC=0.
	LOGLUV_PUBLIC can be used to compile some functions with hidden
	visibility.
	However this functionality was broken (in clang), as LOGLUV_PUBLIC was
	defined to 1 if it wasn't defined, as the extern declarations were still
	visible by the compiler leading to "static declaration follows
	non-static declaration" errors.
	Changing the header from #ifdef to #if fixes the issue.

2025-08-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_728_729_tiffcmp_memory_leak' into 'master'
	tiffcmp: fix memory leak when second file cannot be opened. fix #728, #729

	Closes #728 and #729

	See merge request libtiff/libtiff!747

2025-08-08  Su_Laus  <sulau@freenet.de>

	tiffcmp: fix memory leak when second file cannot be opened.
	Closes #728, #729

2025-08-08  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix-issue-720' into 'master'
	add assert for TIFFReadCustomDirectory infoarray check

	See merge request libtiff/libtiff!744

2025-08-08  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_718_tiff2ps_defer_strile_load' into 'master'
	tiff2ps: check return of TIFFGetFiled()  to fix #718

	Closes #718

	See merge request libtiff/libtiff!746

2025-08-02  Su_Laus  <sulau@freenet.de>

	tiff2ps: check return of TIFFGetFiled() for TIFFTAG_STRIPBYTECOUNTS and TIFFTAG_TILEBYTECOUNTS to avoid NULL pointer dereference.
	Closes #718

2025-07-29  lxy  <854071997@qq.com>

	add assert for TIFFReadCustomDirectory infoarray check.

2025-07-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tif_jpeg_12_bit_bug_JPEGDecodeRaw_fix_717' into 'master'
	tif_jpeg: Fix bug in JPEGDecodeRaw() if JPEG_LIB_MK1_OR_12BIT is defined.

	Closes #717

	See merge request libtiff/libtiff!743

2025-07-11  Su_Laus  <sulau@freenet.de>

	tif_jpeg: Fix bug in JPEGDecodeRaw() if JPEG_LIB_MK1_OR_12BIT is defined for 8/12bit dual mode, introduced in libjpeg-turbo 2.2, which was actually released as 3.0.
	Closes #717

2025-07-10  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffdump_coverity_fix' into 'master'
	tiffdump: Fix coverity scan issue CID 1373365 tainted divisor.

	See merge request libtiff/libtiff!742

2025-07-06  Su_Laus  <sulau@freenet.de>

	tiffdump: Fix coverity scan issue CID 1373365: Passing tainted expression *datamem to PrintData, which uses it as a divisor or modulus.

2025-07-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffcrop_coverity_scan_update' into 'master'
	tiffcrop: Fix coverity scan issue CID 1655232

	See merge request libtiff/libtiff!741

2025-07-02  Su_Laus  <sulau@freenet.de>

	tiffcrop: Fix coverity scan issue CID 1655232: Uninitialized variables  (UNINIT) Using uninitialized value "crop.selections".

2025-07-02  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_716_tiffcrop_memory_leak_on_error_exit' into 'master'
	tiffcrop: close files and release memory on error exit, fix #716

	Closes #716

	See merge request libtiff/libtiff!740

2025-06-27  Su_Laus  <sulau@freenet.de>

	Close open TIFF files and release allocated buffers in tiffcrop before exiting in case of error to avoid memory leaks.

2025-06-26  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_714' into 'master'
	JPEGEncodeRaw(): error out if a previous scanline failed to be written, to...

	Closes #714

	See merge request libtiff/libtiff!739

2025-06-25  Even Rouault  <even.rouault@spatialys.com>

	JPEGEncodeRaw(): error out if a previous scanline failed to be written, to avoid out-of-bounds access
	Fixes #714

2025-06-25  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'improve_TIFFReadRGBAImage' into 'master'
	Improve TIFFReadRGBAImage and avoid buffer overflows.

	See merge request libtiff/libtiff!738

2025-06-25  Su Laus  <sulau@freenet.de>

	Improve TIFFReadRGBAImage and avoid buffer overflows.
	* Raster width can now be larger than image width.
	* Only image data are copied to raster buffer if tiles are padded.
	* Avoid buffer overflow if col_offset \> 0.
	* If row_offset \> 0 do not try to read after last row - avoiding warnings.
	* Feature "col_offset" and "row_offset" now works as expected.

2025-06-25  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'thumbnail-715' into 'master'
	Fix for thumbnail issue #715

	See merge request libtiff/libtiff!737

2025-06-25  Lee Howard  <faxguy@howardsilvan.com>

	Fix for thumbnail issue #715.

2025-06-25  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'issue-711' into 'master'
	Fix for issue #711

	See merge request libtiff/libtiff!735

2025-06-25  Lee Howard  <faxguy@howardsilvan.com>

	TIFFUnlinkDirectory() and TIFFWriteDirectorySec(): clear tif_rawcp when clearing tif_rawdata
	Fix for issue #711

2025-06-25  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiff_itrunc_rand' into 'master'
	tiff_itrunc(): don't use rand() to please Coverity Scan

	See merge request libtiff/libtiff!733

2025-06-12  Even Rouault  <even.rouault@spatialys.com>

	tiff_itrunc(): don't use rand() to please Coverity Scan.

2025-06-12  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiff-static' into 'master'
	Add build option tiff-static, fixes #709

	Closes #709

	See merge request libtiff/libtiff!731

2025-06-12  Demetrius flavious  <lahavlior@gmail.com>

	CMake: Add build option tiff-static, fixes #709.

2025-06-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_704_TIFFReadRGBAImage_crash' into 'master'
	tif_getimage.c: Fix buffer underflow crash for less raster rows at TIFFReadRGBAImageOriented()

	Closes #704

	See merge request libtiff/libtiff!732

2025-06-11  Su Laus  <sulau@freenet.de>

	tif_getimage.c: Fix buffer underflow crash for less raster rows at TIFFReadRGBAImageOriented()

2025-06-06  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'coverity_fixes' into 'master'
	tif_zip.c: changes to please Coverity Scan

	See merge request libtiff/libtiff!730

2025-06-06  Even Rouault  <even.rouault@spatialys.com>

	tif_zip.c: changes to please Coverity Scan.

2025-06-03  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'mkg3states' into 'master'
	Move mkg3states under libtiff/tools, fixes #708

	Closes #708

	See merge request libtiff/libtiff!728

2025-06-03  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_MSVC_warnings_in_some_tools' into 'master'
	Fixing MSVC compiler warnings in some tools.

	See merge request libtiff/libtiff!729

2025-06-03  Su Laus  <sulau@freenet.de>

	Fixing MSVC compiler warnings in some tools.

2025-05-27  Lior Lahav  <LahavLior@gmail.com>

	Move mkg3states under libtiff/tools, fixes #708.

2025-05-25  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'cmake3.10' into 'master'
	Fixed: CMake 4.0 warning when minimum required version is < 3.10

	See merge request libtiff/libtiff!726

2025-05-25  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_MSVC_warnings_in_some_test_progs' into 'master'
	Fixing MSVC compiler warnings in some test programs.

	See merge request libtiff/libtiff!725

2025-05-25  Su Laus  <sulau@freenet.de>

	Fixing MSVC compiler warnings in some test programs.

2025-05-25  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'predict_speedup' into 'master'
	tif_predict.c: speed-up decompression in some cases

	See merge request libtiff/libtiff!724

2025-05-25  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffmedian-707' into 'master'
	Tiffmedian 707

	See merge request libtiff/libtiff!727

2025-05-25  Lee Howard  <faxguy@howardsilvan.com>

	conflict resolution.

	Fix tiffmedian bug #707.

2025-05-24  Lior Lahav  <LahavLior@gmail.com>

	Fixed: CMake 4.0 warning when minimum required version is < 3.10.

2025-05-23  Even Rouault  <even.rouault@spatialys.com>

	tif_predict.c: speed-up decompression in some cases.
	- Predictor=2 on Byte datatype with some loop unrolling
	- Predictor=3 on Float32 datatype with some loop unrolling and SSE2 in the interleaving phae

	Tested on:
	- https://rockyweb.usgs.gov/vdelivery/Datasets/Staged/Elevation/1m/Projects/WA_FEMAHQ_2018_D18/TIFF/USGS_1M_10_x56y512_WA_FEMAHQ_2018_D18.tif
	  USGS_1M_10_x56y512_WA_FEMAHQ_2018_D18.tif: 10012x10012, single band, Float32, LZW-compressed, Predictor=3, 316 MiB

	- and hillshade_predict.tif: 10012x10012, single band, Byte, ZSTD-compressed, Predictor=2, 63 MiB
	  (result of gdaldem hillshade USGS_1M_10_x56y512_WA_FEMAHQ_2018_D18.tif hillshade_predict.tif -co COMPRESS=ZSTD -co PREDICTOR=2)

	master:
	```
	$ hyperfine --warmup 2 -- "gdal raster convert hillshade_predict.tif foo --of mem -q"
	Benchmark #1: gdal raster convert hillshade_predict.tif foo --of mem -q
	  Time (mean ± σ):     512.1 ms ±   2.0 ms    [User: 413.1 ms, System: 98.3 ms]
	  Range (min … max):   509.6 ms … 516.4 ms

	$ hyperfine --warmup 2 --  "gdal raster convert USGS_1M_10_x56y512_WA_FEMAHQ_2018_D18.tif foo --of mem"
	Benchmark #1: gdal raster convert USGS_1M_10_x56y512_WA_FEMAHQ_2018_D18.tif foo --of mem
	  Time (mean ± σ):      2.126 s ±  0.027 s    [User: 1.844 s, System: 0.280 s]
	  Range (min … max):    2.098 s …  2.194 s
	```

	this PR:
	```
	$ hyperfine --warmup 2 -- "gdal raster convert hillshade_predict.tif foo --of mem"
	Benchmark #1: gdal raster convert hillshade_predict.tif foo --of mem
	  Time (mean ± σ):     384.3 ms ±   1.9 ms    [User: 285.1 ms, System: 98.3 ms]
	  Range (min … max):   381.6 ms … 387.4 ms

	$ hyperfine --warmup 2 --  "gdal raster convert USGS_1M_10_x56y512_WA_FEMAHQ_2018_D18.tif foo --of mem"
	Benchmark #1: gdal raster convert USGS_1M_10_x56y512_WA_FEMAHQ_2018_D18.tif foo --of mem
	  Time (mean ± σ):      1.489 s ±  0.004 s    [User: 1.201 s, System: 0.286 s]
	  Range (min … max):    1.484 s …  1.498 s
	```

2025-05-22  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffdither-703' into 'master'
	Don't skip the first line of the input image.  Addresses issue #703

	Closes #703

	See merge request libtiff/libtiff!723

2025-05-22  Lee Howard  <faxguy@howardsilvan.com>

	tiffdither/tiffmedian: Don't skip the first line of the input image.  Addresses issue #703

2025-05-22  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_682_tiff2rgba_size_t_overflow' into 'master'
	tiff2rgba: fix #682 buffer overflow

	Closes #682

	See merge request libtiff/libtiff!722

2025-05-22  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_MSVC_warnings_in_addtiffo' into 'master'
	Fixing MSVC compiler warnings in addtiffo.

	See merge request libtiff/libtiff!717

2025-05-22  Su Laus  <sulau@freenet.de>

	Fixing MSVC compiler warnings in addtiffo.

2025-05-19  Lee Howard  <faxguy@howardsilvan.com>

	Don't skip the first line of the input image.  Addresses issue #703.

2025-05-17  Su_Laus  <sulau@freenet.de>

	tiff2rgba: fix some "a partial expression can generate an overflow before it is assigned to a broader type" warnings.
	Closes #682

2025-05-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'master' into 'master'
	oss-fuzz: fix memory leak in fuzz target

	See merge request libtiff/libtiff!721

2025-05-11  Yuntong Zhang  <ang.unong@gmail.com>

	oss-fuzz: fix memory leak in fuzz target.
	Fixes: https://issues.oss-fuzz.com/issues/385176300

2025-05-02  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'spelling' into 'master'
	Spelling fixes.

	See merge request libtiff/libtiff!720

2025-05-02  Kurt Schwehr  <schwehr@google.com>

	Spelling fixes.
	Most found with codespell version 2.4.1.

2025-04-30  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'configure-ac-syntax-ac-2-71' into 'master'
	configure.ac: Syntax updates for Autoconf 2.71

	See merge request libtiff/libtiff!719

2025-04-29  Bob Friesenhahn  <graphicsmagick.project@gmail.com>

	Merge branch 'defines' into 'master'
	Move several defines into tif_config.h

	See merge request libtiff/libtiff!664

2025-04-29  Bob Friesenhahn  <graphicsmagick.project@gmail.com>

	Merge branch '672-simplify-and-soften-autogen-sh' into 'master'
	Resolve "Build fails when https://git.savannah.gnu.org/ is down"

	Closes #672

	See merge request libtiff/libtiff!718

2025-04-29  Bob Friesenhahn  <bobjfriesenhahn@gmail.com>

	autogen.sh: Enable verbose wget.

	configure.ac: Syntax updates for Autoconf 2.71.

	autogen.sh: Re-implement based on autoreconf. Failure to update config.guess/config.sub does not return error.

2025-04-19  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_698' into 'master'
	LZWDecode(): avoid nullptr dereference when trying to read again after...

	Closes #698

	See merge request libtiff/libtiff!716

2025-04-18  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix-684' into 'master'
	Fix for buffer underflow

	Closes #684

	See merge request libtiff/libtiff!713

2025-04-18  Timothy Lyanguzov  <theta682@gmail.com>

	Fix for buffer underflow.

2025-04-18  Even Rouault  <even.rouault@spatialys.com>

	LZWDecode(): avoid nullptr dereference when trying to read again after end-of-information marker has been found with remaining output bytes (fixes #698)

2025-04-18  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'correct_not_HAVE_IEEEFP_issue_699' into 'master'
	Correct passing arguments to TIFFCvtIEEEFloatToNative().. if HAVE_IEEEFP is not defined.

	See merge request libtiff/libtiff!715

2025-04-17  Su_Laus  <sulau@freenet.de>

	Correct passing arguments to TIFFCvtIEEEFloatToNative() and TIFFCvtIEEEDoubleToNative() if HAVE_IEEEFP is not defined.
	See #699

2025-04-16  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'integer_overflow_gtTileContig_issue_79' into 'master'
	Update some integer overflow checks in tif_getimage.c

	Closes #79

	See merge request libtiff/libtiff!711

2025-04-14  Su_Laus  <sulau@freenet.de>

	Update some integer overflow checks in tif_getimage.c.
	Closes #79

2025-04-14  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_issue_5_write_buffer_overwriting_compression_with_prediction' into 'master'
	tif_predict.c: Don’t overwrite input buffer of TIFFWriteScanline() if prediction.

	Closes #5

	See merge request libtiff/libtiff!712

2025-04-13  Su_Laus  <sulau@freenet.de>

	tif_predict.c: Don’t overwrite input buffer of TIFFWriteScanline() if "prediction" is enabled. Use extra working buffer in PredictorEncodeRow().
	Closes #5

2025-04-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'doc_remove_dead_awaresystems_link' into 'master'
	doc: Remove dead links to no more existing Awaresystems web-site.

	See merge request libtiff/libtiff!705

2025-04-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'notes_for_allocating_TIFFYCbCrToRGB_structure_issue_681' into 'master'
	doc: Added hints for correct allocation of TIFFYCbCrtoRGB structure and its associated buffers.

	Closes #681

	See merge request libtiff/libtiff!709

2025-04-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'doc_add_multi_page_reference_issue_43' into 'master'
	doc: Added chapter to "Using the TIFF Library" for multi page

	Closes #43

	See merge request libtiff/libtiff!710

2025-04-10  Su_Laus  <sulau@freenet.de>

	doc: Added chapter to "Using the TIFF Library" with links to handling multi-page TIFF and custom directories.
	Closes #43

2025-04-09  Su_Laus  <sulau@freenet.de>

	doc: Added hints for correct allocation of TIFFYCbCrtoRGB structure and its associated buffers.
	Closes #681

2025-04-03  Su_Laus  <sulau@freenet.de>

	Replace some last links and remove last todos.

2025-04-02  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'CheckReachedCounters_static' into 'master'
	tif_fax3.c: add missing static qualifier in front of CheckReachedCounters() (master only)

	See merge request libtiff/libtiff!708

2025-04-02  Even Rouault  <even.rouault@spatialys.com>

	tif_fax3.c: add missing static qualifier in front of CheckReachedCounters() (master only)

2025-04-01  Su_Laus  <sulau@freenet.de>

	References to codec libraries updated and missing codec compile defines (tiffconf.h) added.

2025-04-01  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix-676' into 'master'
	Fix memory leak in _TIFFSetDefaultCompressionState

	Closes #676

	See merge request libtiff/libtiff!707

2025-04-01  Timothy Lyanguzov  <timothy.lyanguzov@sap.com>

	Fix memory leak in _TIFFSetDefaultCompressionState.
	`_TIFFSetDefaultCompressionState` does not clean up before setting default state

	follow up for !563
	fixes #676

2025-03-31  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffcp_integer_overflow_check_issue_546' into 'master'
	tiffcp: Improve non-secure integer overflow check. Closes #546

	Closes #546

	See merge request libtiff/libtiff!698

2025-03-31  Su_Laus  <sulau@freenet.de>

	Improve non-secure integer overflow check (comparison of division result with multiplicant) at compiler optimisation in tiffcp, rgb2ycbcr and tiff2rgba.
	Closes #546

2025-03-28  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'zlib_solo' into 'master'
	zip: Provide zlib allocation functions.

	See merge request libtiff/libtiff!706

2025-03-28  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_670_tif_fax3_EOL_error_count_reached' into 'master'
	tif_fax3.c: error out after a number of times end-of-line or unexpected code words

	Closes #670

	See merge request libtiff/libtiff!703

2025-03-28  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix-test-tiffcrop-tiffcp-32bpp-None-jpeg-race' into 'master'
	test: Fix race condition in {tiffcrop,tiffcp}-32bpp-None-jpeg.sh tests

	See merge request libtiff/libtiff!701

2025-03-28  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'doc_update_TIFFSetWriteOffset' into 'master'
	doc: Clarify TIFFSetWriteOffset() only sets offset for image data and not for IFD data.

	See merge request libtiff/libtiff!699

2025-03-28  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'coverity_fixes_in_tools' into 'master'
	Silence some Coverity Scan warnigs and mark already dismissed ones.

	See merge request libtiff/libtiff!696

2025-03-25  Nikolay Sivov  <nsivov@codeweavers.com>

	zip: Provide zlib allocation functions.
	Otherwise for zlib built with -DZ_SOLO inflating will fail.

2025-03-23  Su_Laus  <sulau@freenet.de>

	Updating BigTIFF specification and some miscelaneous editions.

2025-03-20  Su_Laus  <sulau@freenet.de>

	doc: Remove dead links to no more existing Awaresystems web-site.

2025-03-19  Roger Leigh  <rleigh@codelibre.net>

	Merge branch 'bugfix/issue_674-export_tiffxx_cmake_target' into 'master'
	Export tiffxx cmake target (issue #674)

	See merge request libtiff/libtiff!704

2025-03-18  Daniel Moreno  <daniel.moreno@cognex.com>

	Export tiffxx cmake target (issue #674)

2025-03-04  Su_Laus  <sulau@freenet.de>

	tif_fax3.c: error out after a number of times end-of-line or unexpected bad code words have been reached.
	Closes #670

2025-03-03  Bob Friesenhahn  <graphicsmagick.project@gmail.com>

	Merge branch 'fix_665_memory_leak' into 'master'
	Fix memory leak of issue # 665 (td_stripbytecount_p and td_stripoffset_p in TIFFSetupStrips())

	Closes #665

	See merge request libtiff/libtiff!697

2025-03-03  Su Laus  <sulau@freenet.de>

	Fix memory leak of issue # 665 (td_stripbytecount_p and td_stripoffset_p in TIFFSetupStrips())

2025-02-26  Gabi Falk  <gabifalk@gmx.com>

	test: Fix race condition in {tiffcrop,tiffcp}-32bpp-None-jpeg.sh tests.
	These tests used the same output path, which could cause failures
	when run in parallel.  These were the only tests with a conflicting
	outfile= parameter.

	Link: https://bugs.gentoo.org/943020

2025-02-25  Bob Friesenhahn  <graphicsmagick.project@gmail.com>

	Merge branch 'doc_re_entrancy_and_thread_safety_see_667' into 'master'
	doc: Update documentation on re-entrancy and thread safety.

	Closes #667

	See merge request libtiff/libtiff!700

2025-02-25  Su Laus  <sulau@freenet.de>

	doc: Update documentation on re-entrancy and thread safety.

2025-02-23  Su_Laus  <sulau@freenet.de>

	doc/functions/TIFFWriteDirectory.rst: Clarify TIFFSetWriteOffset() only sets offset for image data and not for IFD data.

2025-02-16  Su_Laus  <sulau@freenet.de>

	Silence some Coverity Scan warnigs and mark already dismissed ones. Guard all code related to CHUNKY_STRIP_READ_SUPPORT.

2025-02-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'doc_remove_deprecated_type_definitions' into 'master'
	doc: Update the documentation to reflect deprecated typedefs

	See merge request libtiff/libtiff!694

2025-02-11  Su Laus  <sulau@freenet.de>

	doc: Update the documentation to reflect deprecated typedefs.

2025-02-09  Su_Laus  <sulau@freenet.de>

	Silence some Coverity Scan warnigs and mark already dismissed ones.

2025-01-30  Su Laus  <sulau@freenet.de>

	Merge branch 'patch-1' into 'master'
	Doc: Fix return type of TIFFReadEncodedTile()

	See merge request libtiff/libtiff!693

2025-01-25  Mark Riehm  <imriehm2@gmail.com>

	Doc: Fix return type of TIFFReadEncodedTile()

2025-01-14  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'coverity_1638738' into 'master'
	TIFFSetSubDirectory(): check _TIFFCheckDirNumberAndOffset() return

	See merge request libtiff/libtiff!692

2025-01-12  Even Rouault  <even.rouault@spatialys.com>

	TIFFSetSubDirectory(): check _TIFFCheckDirNumberAndOffset() return.
	Fix Coverity Scan:
	```
	    CID 1638738:  Error handling issues  (CHECKED_RETURN)
	    Calling "_TIFFCheckDirNumberAndOffset" without checking return value (as is done elsewhere 4 out of 5 times).
	```

2025-01-12  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'remove_get_field_type_from_structure' into 'master'
	Remove get_field_type form TIFFField structure because it is not used anymore

	See merge request libtiff/libtiff!691

2025-01-09  Su_Laus  <sulau@freenet.de>

	Remove get_field_type form TIFFField structure because it is not used anymore and rename other parameter to set_get_field_type.

2025-01-06  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'doc_update_adding_new_tags_fix_642' into 'master'
	doc: Update "Defining New TIFF Tags" description. See also #642.

	Closes #642

	See merge request libtiff/libtiff!655

2025-01-06  Su Laus  <sulau@freenet.de>

	doc: Update "Defining New TIFF Tags" description. See also #642.

2025-01-01  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fax_encode_sample_per_pixel_warning' into 'master'
	tif_fax3.c: Error out for CCITT fax encoding if SamplesPerPixel is not equal 1 and PlanarConfiguration = Contiguous

	Closes #26

	See merge request libtiff/libtiff!687

2025-01-01  Su Laus  <sulau@freenet.de>

	tif_fax3.c: Error out for CCITT fax encoding if SamplesPerPixel is not equal 1 and PlanarConfiguration = Contiguous

2024-12-31  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_663' into 'master'
	_TIFFVSetField(): fix potential use of unallocated memory (out-of-bounds read...

	Closes #663

	See merge request libtiff/libtiff!689

2024-12-30  Even Rouault  <even.rouault@spatialys.com>

	_TIFFVSetField(): fix potential use of unallocated memory (out-of-bounds read / nullptr dereference) in case of out-of-memory situation when dealing with custom tags
	Fixes #663

2024-12-30  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'master' into 'master'
	Improved oss-fuzz fuzzer tiff_read_rgba_fuzzer.cc

	See merge request libtiff/libtiff!688

2024-12-30  Fabio Gritti  <degrigis@ucsb.edu>

	Improved oss-fuzz fuzzer tiff_read_rgba_fuzzer.cc.

2024-12-30  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'LINKER_LANGUAGE' into 'master'
	Set LINKER_LANGUAGE for C targets with C++ deps

	See merge request libtiff/libtiff!685

2024-12-30  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'merge_multiple_functions_implemented_twice_#154' into 'master'
	Merge several functions implemented twice in different modules, see #154

	Closes #154

	See merge request libtiff/libtiff!681

2024-12-30  Su Laus  <sulau@freenet.de>

	Merge several functions implemented twice in different modules, see #154

2024-12-19  Alyssa Ross  <hi@alyssa.is>

	Set LINKER_LANGUAGE for C targets with C++ deps.
	When C and C++ code are being linked together, a C++ compiler has to
	be used.  In my experience, this doesn't always cause problems, but it
	tends to when doing static builds.

	Without this change, I got build failures when I had lerc enabled:

	cd /build/source/build/tools && /nix/store/6941y72hx1k8rxiygm4iin16i4zhcdqr-cmake-3.30.5/bin/cmake -E cmake_link_script CMakeFiles/fax2ps.dir/link.txt --verbose=1
	/nix/store/gb7zjcg4fh1mnd5qp4pkrlib55w6j42l-aarch64-unknown-linux-musl-gcc-wrapper-13.3.0/bin/aarch64-unknown-linux-musl-gcc  -Wall -Winline -Wformat-security -Wpointer-arith -Wdisabled-optimization -Wno-unknown-pragmas -fstrict-aliasing -O3 -DNDEBUG CMakeFiles/fax2ps.dir/fax2ps.c.o -o fax2ps  ../libtiff/libtiff.a -Wl,-Bdynamic /nix/store/n49vspyhaga9pabpqqcwm9rlam7r4152-zlib-static-aarch64-unknown-linux-musl-1.3.1/lib/libz.a /nix/store/k71jx85iin8bvgqs1viq263qp186nb9h-libdeflate-static-aarch64-unknown-linux-musl-1.22/lib/libdeflate.a /nix/store/kp6hkjfjdmd94crz6claldi5hg9z8r87-libjpeg-turbo-static-aarch64-unknown-linux-musl-3.0.4/lib/libjpeg.a /nix/store/h2f1h8bri9d0mvdrvn5ad94hy31dan2s-lerc-static-aarch64-unknown-linux-musl-4.0.0/lib/libLerc.a /nix/store/hjf3h5prgfdadv3wsj4172sdxrnycgba-xz-static-aarch64-unknown-linux-musl-5.6.3/lib/liblzma.a /nix/store/qh8h4fdsvhj75xliiqb71qrynnng51p2-zstd-static-aarch64-unknown-linux-musl-1.5.6/lib/libzstd.a /nix/store/002ykljzr16pfxnijq7x8hr4nbicg745-libwebp-static-aarch64-unknown-linux-musl-1.4.0/lib/libwebp.a /nix/store/002ykljzr16pfxnijq7x8hr4nbicg745-libwebp-static-aarch64-unknown-linux-musl-1.4.0/lib/libsharpyuv.a /nix/store/dnm541qbzfanx6r0kqbc2p4xfjiwl4sf-musl-static-aarch64-unknown-linux-musl-1.2.5/lib/libm.a -Wl,-Bstatic
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: /nix/store/h2f1h8bri9d0mvdrvn5ad94hy31dan2s-lerc-static-aarch64-unknown-linux-musl-4.0.0/lib/libLerc.a(Lerc.cpp.o): in function `std::vector<unsigned char, std::allocator<unsigned char> >::operator=(std::vector<unsigned char, std::allocator<unsigned char> > const&) [clone .isra.0]':
	(.text+0x140): undefined reference to `operator new(unsigned long)'
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: (.text+0x16c): undefined reference to `operator delete(void*, unsigned long)'
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: (.text+0x26c): undefined reference to `std::__throw_bad_alloc()'
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: /nix/store/h2f1h8bri9d0mvdrvn5ad94hy31dan2s-lerc-static-aarch64-unknown-linux-musl-4.0.0/lib/libLerc.a(Lerc.cpp.o): in function `LercNS::Lerc::GetRanges(unsigned char const*, unsigned int, int, LercNS::Lerc2::HeaderInfo const&, double*, double*, unsigned long)':
	(.text+0x10a0): undefined reference to `operator delete(void*, unsigned long)'
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: (.text+0x10b4): undefined reference to `operator delete(void*, unsigned long)'
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: (.text+0x10c8): undefined reference to `operator delete(void*, unsigned long)'
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: (.text+0x10ec): undefined reference to `operator delete(void*, unsigned long)'
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: (.text+0x1100): undefined reference to `operator delete(void*, unsigned long)'
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: /nix/store/h2f1h8bri9d0mvdrvn5ad94hy31dan2s-lerc-static-aarch64-unknown-linux-musl-4.0.0/lib/libLerc.a(Lerc.cpp.o):(.text+0x1114): more undefined references to `operator delete(void*, unsigned long)' follow
	/nix/store/ncvpwrpay6vvh9wwc6fq28lb9hip7chc-aarch64-unknown-linux-musl-binutils-2.43.1/bin/aarch64-unknown-linux-musl-ld: /nix/store/h2f1h8bri9d0mvdrvn5ad94hy31dan2s-lerc-static-aarch64-unknown-linux-musl-4.0.0/lib/libLerc.a(Lerc.cpp.o): in function `bool LercNS::Lerc::Convert<signed char>(LercNS::CntZImage const&, signed char*, unsigned char*, bool)':
	[1808 more lines of this ommitted]
	collect2: error: ld returned 1 exit status

2024-12-17  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'read_only' into 'master'
	Take into account PACKBITS_READ_ONLY, LZW_READ_ONLY, and LERC_READ_ONLY macros

	See merge request libtiff/libtiff!683

2024-12-17  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'libtiff_predictor_3_inverted_endianness_write_fix' into 'master'
	fix writing a Predictor=3 file with non-native endianness

	See merge request libtiff/libtiff!684

2024-12-17  Even Rouault  <even.rouault@spatialys.com>

	fix writing a Predictor=3 file with non-native endianness.

	Take into account PACKBITS_READ_ONLY, LZW_READ_ONLY, and LERC_READ_ONLY macros
	to disable compiling write-support for those codecs.
	Also add missing casts in LERC codec.

2024-12-16  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tif_lzw_packbits_null' into 'master'
	tif_lzw.c / tif_packbits.c: use NULL instead of 0

	See merge request libtiff/libtiff!682

2024-12-16  Even Rouault  <even.rouault@spatialys.com>

	tif_lzw.c / tif_packbits.c: use NULL instead of 0.

2024-12-09  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'manpage_TIFFRGBAImage_#67_different_output_unassociated_alpha' into 'master'
	manpage: doc/functions/TIFFRGBAImage.rst note added about un-associated alpha handling

	Closes #67

	See merge request libtiff/libtiff!679

2024-12-09  Su Laus  <sulau@freenet.de>

	manpage: doc/functions/TIFFRGBAImage.rst note added about un-associated alpha handling

2024-12-09  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiff2pdf_division_by_zero_#654' into 'master'
	tiff2pdf: check h_samp and v_samp for range 1 to 4 to avoid division by zero.

	Closes #654

	See merge request libtiff/libtiff!680

2024-12-07  Su_Laus  <sulau@freenet.de>

	tiff2pdf: check h_samp and v_samp for range 1 to 4 to avoid division by zero.
	Closes #654

2024-12-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'master' into 'master'
	CMake CMP0074 new policy.

	See merge request libtiff/libtiff!678

2024-12-04  榆柳松  <wzhengsen@gmail.com>

	CMake CMP0074 new policy.

2024-12-04  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'manpage_TIFFReadRGBAImage_orientation_fault' into 'master'
	doc: TIFFReadRGBAImage.rst note about incorrect saving in raster for some TIFF orientations (5 to 8).

	Closes #76 and #77

	See merge request libtiff/libtiff!675

2024-12-04  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'TIFFRGBAImageOK_remove_unnecessary_calls' into 'master'
	Remove unnecessary calls to TIFFRGBAImageOK() in tif_getimage.c

	Closes #175

	See merge request libtiff/libtiff!677

2024-12-03  Su_Laus  <sulau@freenet.de>

	Remove unnecessary calls to TIFFRGBAImageOK() in tif_getimage.c.
	Closes #175

2024-11-30  Roger Leigh  <rleigh@codelibre.net>

	Merge branch 'master' into 'master'
	Rename cxx to tiff-cxx

	See merge request libtiff/libtiff!676

2024-11-30  榆柳松  <wzhengsen@gmail.com>

	Rename cxx to tiff-cxx,avoid overly mundane names.

2024-11-29  Su_Laus  <sulau@freenet.de>

	manpage: doc/functions/TIFFRGBAImage.rst note added for incorrect saving of images with TIFF orientation from 5 (LeftTop) to 8 (LeftBottom) in the raster.

2024-11-19  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_test_directory_for_big_endian_machines' into 'master'
	Update test/test_directory.c not to fail on big-endian machines. Fix memory leaks

	Closes #652 et #656

	See merge request libtiff/libtiff!673

2024-11-19  Su Laus  <sulau@freenet.de>

	Update test/test_directory.c not to fail on big-endian machines. Fix memory leaks
	Closes #652 et #656

2024-11-08  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tiffdither_multi_page' into 'master'
	tiffdither: process all pages in input TIFF file

	See merge request libtiff/libtiff!674

2024-11-08  Marco Rullo  <rullo.marco@gmail.com>

	tiffdither: process all pages in input TIFF file.

2024-10-30  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'unknown_tag_warnings' into 'master'
	No longer emit warnings about unknown tags by default, and add...

	See merge request libtiff/libtiff!463

2024-10-29  Even Rouault  <even.rouault@spatialys.com>

	tiffinfo: add a -W switch to warn about unknown tags.

	No longer emit warnings about unknown tags by default, and add TIFFOpenOptionsSetWarnAboutUnknownTags() for explicit control

2024-10-28  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'COMPRESSION_JXL_DNG_1_7' into 'master'
	tiff.h: add COMPRESSION_JXL_DNG_1_7=52546 as used for JPEGXL compression in...

	See merge request libtiff/libtiff!672

2024-10-27  Even Rouault  <even.rouault@spatialys.com>

	Merge branch '532_MergeFields_with_FIELD_IGNORE' into 'master'
	Issue #532: Updating TIFFMergeFieldInfo() with read_count=write_count=0 for FIELD_IGNORE.

	Closes #532

	See merge request libtiff/libtiff!668

2024-10-27  Su Laus  <sulau@freenet.de>

	Issue #532: Updating TIFFMergeFieldInfo() with read_count=write_count=0 for FIELD_IGNORE.
	Updating TIFFMergeFieldInfo() with read_count=write_count=0 for FIELD_IGNORE.
	Improving handling when field_name = NULL.

	Closes #532

	There was a regression in v4.7.0 related to re-opened issue https://gitlab.com/libtiff/libtiff/-/issues/532#note_2134433038 and following discussion.

2024-10-27  Even Rouault  <even.rouault@spatialys.com>

	tiff.h: add COMPRESSION_JXL_DNG_1_7=52546 as used for JPEGXL compression in the DNG 1.7 specification

2024-10-23  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_651_bis' into 'master'
	CMake: fix build with LLVM/Clang 17+

	Closes #651

	See merge request libtiff/libtiff!671

2024-10-21  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'remove_useless_cmake_minimum_required' into 'master'
	doc/CMakeLists.txt: remove useless cmake_minimum_required()

	See merge request libtiff/libtiff!670

2024-10-20  Daniel E @diizzyy  <unknown@example.com>

	CMake: fix build with LLVM/Clang 17+
	Fixes #651

2024-10-20  Even Rouault  <even.rouault@spatialys.com>

	doc/CMakeLists.txt: remove useless cmake_minimum_required()
	There's already one in top CMakeLists.txt (at 3.9.0 currently)

	Newer CMake versions throw this warning with current 3.2.0:
	```
	CMake Deprecation Warning at doc/CMakeLists.txt:27 (cmake_minimum_required):
	  Compatibility with CMake < 3.5 will be removed from a future version of
	  CMake.
	  Update the VERSION argument <min> value or use a ...<max> suffix to tell
	  CMake that the project does not need compatibility with older versions.
	```

2024-10-13  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'tif_jpeg_65535' into 'master'
	tif_jpeg.c: use JPEG_MAX_DIMENSION constant instead of hard-coded 65535 value

	See merge request libtiff/libtiff!658

2024-10-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_gdal_10875' into 'master'
	TIFFDefaultTransferFunction(): give up beyond td_bitspersample = 24

	See merge request libtiff/libtiff!665

2024-10-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'ppm2tiff_fix_#467_10bit_packwords' into 'master'
	ppm2tiff: Fix bug in `pack_words` trailing bytes

	Closes #467

	See merge request libtiff/libtiff!666

2024-10-05  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'master' into 'master'
	Fix issue #649 in fax2ps caused by regression in commit...

	Closes #649

	See merge request libtiff/libtiff!667

2024-10-05  Lee Howard  <faxguy@howardsilvan.com>

	Check TIFFTAG_TILELENGTH and TIFFTAGTILEWIDTH for valid input, addresses issue #650

2024-09-27  Lee Howard  <faxguy@howardsilvan.com>

	Fix issue #649 in fax2ps caused by regression in commit https://gitlab.com/libtiff/libtiff/-/commit/28c38d648b64a66c3218778c4745225fe3e3a06d where TIFFTAG_FAXFILLFUNC is being used rather than an output buffer.

2024-09-27  Su_Laus  <sulau@freenet.de>

	ppm2tiff: Fix bug in `pack_words` trailing bytes, where last two bytes of each line were written wrongly.
	Closes #467.

2024-09-26  Even Rouault  <even.rouault@spatialys.com>

	TIFFDefaultTransferFunction(): give up beyond td_bitspersample = 24.
	This function tries to allocate (1 << td_bitspersample) bytes which may
	results in denial of services if td_bitspersample is too large.

	Fixes https://github.com/OSGeo/gdal/issues/10875

2024-09-22  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'theta682-fix-Oi-GFS' into 'master'
	Fix parsing a private tag 0x80a6

	Closes #647

	See merge request libtiff/libtiff!661

2024-09-22  Timothy Lyanguzov  <theta682@gmail.com>

	Do not error out on a tag whose tag count value is zero, just issue a warning.
	Fix parsing a private tag 0x80a6

2024-09-21  Benjamin Gilbert  <bgilbert@backtick.net>

	Define HAVE_JPEGTURBO_DUAL_MODE_8_12 in tif_config.h.
	Remove special-case Autotools and CMake code to define it on the
	compiler command line.

	Define LERC_STATIC in tif_config.h.
	Remove special-case Autotools and CMake code to define it on the
	compiler command line.

	CMake: set WORDS_BIGENDIAN via #cmakedefine.
	Autotools sets WORDS_BIGENDIAN via tif_config.h, but the CMake equivalent
	in tif_config.h.cmake.in is a no-op; CMake passes the define on the
	compiler command line instead.  For consistency, have CMake set the define
	via tif_config.h.

2024-09-21  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_648_ASCII_length_at_WriteDirectorySec_codec_part' into 'master'
	Increment string length for ASCII tags in TIFFWriteDirectorySec() for codec...

	Closes #648

	See merge request libtiff/libtiff!663

2024-09-21  Su_Laus  <sulau@freenet.de>

	Increment string length for ASCII tags in TIFFWriteDirectorySec() for codec tags defined with FIELD_xxx bits, as it is done for FIELD_CUSTOM tags. Fixes #648

2024-09-11  Even Rouault  <even.rouault@spatialys.com>

	libtiff v4.7.0rc2 preparation

2024-09-11  Even Rouault  <even.rouault@spatialys.com>

	configure.ac: make a provision to look for up to python 3.16 ...
	hopefully we'll have ditched autoconf support at that point...

	Fix test dependency on tiff2rgba-32BPP that breaks 'make check -jX'

	test/tiff2ps*.sh scripts: try to use POSIX only diff flags.

2024-09-11  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'automake_doc_fix' into 'master'
	doc/Makefile.am: make sure that 'doc-html' and 'doc-man' targets have dependencies to avoid them to be rebuilt by successive make

	See merge request libtiff/libtiff!657

2024-09-10  Even Rouault  <even.rouault@spatialys.com>

	Merge branch '4_7_0_rst_rgb2ycbcr_thumbnail' into 'master'
	v4.7.0.rst: clarify that rgb2ycbcr and thumbnail are not installed

	See merge request libtiff/libtiff!659

2024-09-10  Even Rouault  <even.rouault@spatialys.com>

	v4.7.0.rst: clarify that rgb2ycbcr and thumbnail are not installed.

2024-09-08  Even Rouault  <even.rouault@spatialys.com>

	doc/Makefile.am: make sure that 'doc-html' and 'doc-man' targets have dependencies to avoid them to be rebuilt by successive make

	doc/Makefile.am: move rst_sources and EXTRA_DIST variables on top of file

2024-09-07  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'index_rst_formatting_issue' into 'master'
	index.rst: fix formatting issue

	See merge request libtiff/libtiff!656

2024-09-07  Even Rouault  <even.rouault@spatialys.com>

	index.rst: fix formatting issue.

2024-09-07  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'prepare_release_4_7_0' into 'master'
	Prepare release 4.7.0

	See merge request libtiff/libtiff!654

2024-09-07  Even Rouault  <even.rouault@spatialys.com>

	Doc: remove 'Master' terminlogy.

	v4.7.0.rst: remove mention about autoconf-archive since that change has been reverted

	Revert "autotools: allow pulling in updated macros from autoconf-archive"
	This reverts commit c820d16c30c27b3be6c3d10834b6d75607ae9d77.

	Revert "HOWTO-RELEASE: document the necessary dependencies for producing dist tarballs"
	This reverts commit f1a91e42b3f23641cb5559fb60e1d73d201275eb.

	Revert "configure.ac: fix HAVE_OPENGL determination due to recent changes"
	This reverts commit c370e67f7fd9320674544113218b7fbca4f65c2b.

	configure.ac: fix HAVE_OPENGL determination due to recent changes.

	Update version numbers for 4.7.0.

	Update ChangeLog and create release notes for v4.7.0.

2024-09-06  Even Rouault  <even.rouault@spatialys.com>

	typo fixes.

2024-09-06  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'autoconf-archive' into 'master'
	autotools: allow pulling in updated macros from autoconf-archive

	See merge request libtiff/libtiff!645

2024-09-06  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'remove_last_usage_of_get_field_type' into 'master'
	Change last usage of get_field_type at TIFFWriteDirectorySec() to set_field_type

	See merge request libtiff/libtiff!653

2024-09-06  Even Rouault  <even.rouault@spatialys.com>

	libtiff v4.7.0rc1 preparation

2024-09-06  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'autoconf-archive' into 'master'
	autotools: allow pulling in updated macros from autoconf-archive

	See merge request libtiff/libtiff!645

2024-09-06  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'remove_last_usage_of_get_field_type' into 'master'
	Change last usage of get_field_type at TIFFWriteDirectorySec() to set_field_type

	See merge request libtiff/libtiff!653

2024-09-05  Su_Laus  <sulau@freenet.de>

	Change last usage of get_field_type at TIFFWriteDirectorySec() for codec related tags to set_field_type.

2024-09-03  Eli Schwartz  <eschwartz93@gmail.com>

	HOWTO-RELEASE: document the necessary dependencies for producing dist tarballs

	autogen.sh: actually return failure if any step failed.
	Previously, it exited with failure if and only if the wget calls failed.
	But it is possible for any command to fail, e.g. if the autotools are
	broken, misconfigured, or missing dependencies.

2024-09-03  Eli Schwartz  <eschwartz93@gmail.com>

	autotools: allow pulling in updated macros from autoconf-archive.
	Instead of inlining a few macros in acinclude.m4 and forgetting about
	them, teach aclocal to install the macros to m4/ when autoreconf is run.
	They are no longer tracked in one big file, nor tracked at all in git,
	but producing dist tarballs will bundle them up just like `configure`
	and `Makefile.in`.

	Note: previous behavior of AX_CHECK_GL* was to simply ignore support and
	proceed, if it was asked for and not found. The updated macros expect to
	opt into this by specifying action-if-not-found.

	VL_* is obsolete. Upgrade to AX_CFLAGS_WARN_ALL.

2024-09-03  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'XZY2RGB' into 'master'
	Fixes #644 Index comparison as size_t to avoid overflow in TIFFXYZToRGB

	Closes #644

	See merge request libtiff/libtiff!649

2024-09-03  Nicolas Badoux  <n.badoux@hotmail.com>

	Fixes #644 Index comparison as size_t to avoid overflow in TIFFXYZToRGB.

2024-09-01  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'ci_werror' into 'master'
	CI: build with -Wall -Wextra -Werror

	See merge request libtiff/libtiff!652

2024-08-30  Even Rouault  <even.rouault@spatialys.com>

	configure.ac: avoid -Werror passed to CFLAGS to interfere with feature detection

	CI: build with -Wall -Wextra -Werror.

2024-08-30  Roger Leigh  <rleigh@codelibre.net>

	Merge branch 'ci-old-update' into 'master'
	ci: Update "old" build jobs to use Ubuntu 22.04

	See merge request libtiff/libtiff!651

2024-08-30  Roger Leigh  <rleigh@codelibre.net>

	ci: Update "old" build jobs to use Ubuntu 22.04.

2024-08-30  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'ci-ubuntu-22.04' into 'master'
	ci: Update to use Ubuntu 24.04 CI images

	See merge request libtiff/libtiff!650

2024-08-30  Roger Leigh  <rleigh@codelibre.net>

	ci: Update to use Ubuntu 24.04 CI images.

2024-08-28  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'uv_encode' into 'master'
	Fix #645 by using unsigned int for variable indexing an array in uv_decode() and uv_encode()

	Closes #645

	See merge request libtiff/libtiff!648

2024-08-28  Nicolas Badoux  <n.badoux@hotmail.com>

	Fix #645 by using unsigned int for variable indexing an array in uv_decode() and uv_encode()
	Change `vi` to unsigned int in `uv_decode()` and `uv_encode()` to ensure that upper bound `UV_NVS` cannot be satisfied by a negative value which will overflow when cast to an unsigned value at array access time. This fixes issue #645 where a segmentation fault was caused.

2024-08-27  Even Rouault  <even.rouault@spatialys.com>

	Merge branch 'fix_643_solitary_CustomDirectory_read' into 'master'
	Fix #643 by initializing pointer to TIFFSetField() and TIFFGetField() before TIFFReadGPSDirectory

	Closes #643

	See merge request libtiff/libtiff!647

2024-08-27  Su Laus  <sulau@freenet.de>

	Fix #643 by initializing pointer to TIFFSetField() and TIFFGetField() before TIFFReadGPSDire
... [truncated]
```

## General Information

This file is part of the OpenCV repository infrastructure.

