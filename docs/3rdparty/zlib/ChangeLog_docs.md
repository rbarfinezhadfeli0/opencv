# Documentation for `3rdparty/zlib/ChangeLog`

## File Metadata

- **Full Path**: `3rdparty/zlib/ChangeLog`
- **File Name**: `ChangeLog`
- **File Size**: 83,837 bytes
- **File Type**: no extension
- **Link to Source**: [3rdparty/zlib/ChangeLog](../../3rdparty/zlib/ChangeLog)

## Purpose and Role

This file is located in the `3rdparty/zlib` directory and serves as part of the OpenCV library infrastructure.

## File Content

```

                ChangeLog file for zlib

Changes in 1.3.1 (22 Jan 2024)
- Reject overflows of zip header fields in minizip
- Fix bug in inflateSync() for data held in bit buffer
- Add LIT_MEM define to use more memory for a small deflate speedup
- Fix decision on the emission of Zip64 end records in minizip
- Add bounds checking to ERR_MSG() macro, used by zError()
- Neutralize zip file traversal attacks in miniunz
- Fix a bug in ZLIB_DEBUG compiles in check_match()
- Various portability and appearance improvements

Changes in 1.3 (18 Aug 2023)
- Remove K&R function definitions and zlib2ansi
- Fix bug in deflateBound() for level 0 and memLevel 9
- Fix bug when gzungetc() is used immediately after gzopen()
- Fix bug when using gzflush() with a very small buffer
- Fix crash when gzsetparams() attempted for transparent write
- Fix test/example.c to work with FORCE_STORED
- Rewrite of zran in examples (see zran.c version history)
- Fix minizip to allow it to open an empty zip file
- Fix reading disk number start on zip64 files in minizip
- Fix logic error in minizip argument processing
- Add minizip testing to Makefile
- Read multiple bytes instead of byte-by-byte in minizip unzip.c
- Add memory sanitizer to configure (--memory)
- Various portability improvements
- Various documentation improvements
- Various spelling and typo corrections

Changes in 1.2.13 (13 Oct 2022)
- Fix configure issue that discarded provided CC definition
- Correct incorrect inputs provided to the CRC functions
- Repair prototypes and exporting of new CRC functions
- Fix inflateBack to detect invalid input with distances too far
- Have infback() deliver all of the available output up to any error
- Fix a bug when getting a gzip header extra field with inflate()
- Fix bug in block type selection when Z_FIXED used
- Tighten deflateBound bounds
- Remove deleted assembler code references
- Various portability and appearance improvements

Changes in 1.2.12 (27 Mar 2022)
- Cygwin does not have _wopen(), so do not create gzopen_w() there
- Permit a deflateParams() parameter change as soon as possible
- Limit hash table inserts after switch from stored deflate
- Fix bug when window full in deflate_stored()
- Fix CLEAR_HASH macro to be usable as a single statement
- Avoid a conversion error in gzseek when off_t type too small
- Have Makefile return non-zero error code on test failure
- Avoid some conversion warnings in gzread.c and gzwrite.c
- Update use of errno for newer Windows CE versions
- Small speedup to inflate [psumbera]
- Return an error if the gzputs string length can't fit in an int
- Add address checking in clang to -w option of configure
- Don't compute check value for raw inflate if asked to validate
- Handle case where inflateSync used when header never processed
- Avoid the use of ptrdiff_t
- Avoid an undefined behavior of memcpy() in gzappend()
- Avoid undefined behaviors of memcpy() in gz*printf()
- Avoid an undefined behavior of memcpy() in _tr_stored_block()
- Make the names in functions declarations identical to definitions
- Remove old assembler code in which bugs have manifested
- Fix deflateEnd() to not report an error at start of raw deflate
- Add legal disclaimer to README
- Emphasize the need to continue decompressing gzip members
- Correct the initialization requirements for deflateInit2()
- Fix a bug that can crash deflate on some input when using Z_FIXED
- Assure that the number of bits for deflatePrime() is valid
- Use a structure to make globals in enough.c evident
- Use a macro for the printf format of big_t in enough.c
- Clean up code style in enough.c, update version
- Use inline function instead of macro for index in enough.c
- Clarify that prefix codes are counted in enough.c
- Show all the codes for the maximum tables size in enough.c
- Add gznorm.c example, which normalizes gzip files
- Fix the zran.c example to work on a multiple-member gzip file
- Add tables for crc32_combine(), to speed it up by a factor of 200
- Add crc32_combine_gen() and crc32_combine_op() for fast combines
- Speed up software CRC-32 computation by a factor of 1.5 to 3
- Use atomic test and set, if available, for dynamic CRC tables
- Don't bother computing check value after successful inflateSync()
- Correct comment in crc32.c
- Add use of the ARMv8 crc32 instructions when requested
- Use ARM crc32 instructions if the ARM architecture has them
- Explicitly note that the 32-bit check values are 32 bits
- Avoid adding empty gzip member after gzflush with Z_FINISH
- Fix memory leak on error in gzlog.c
- Fix error in comment on the polynomial representation of a byte
- Clarify gz* function interfaces, referring to parameter names
- Change macro name in inflate.c to avoid collision in VxWorks
- Correct typo in blast.c
- Improve portability of contrib/minizip
- Fix indentation in minizip's zip.c
- Replace black/white with allow/block. (theresa-m)
- minizip warning fix if MAXU32 already defined. (gvollant)
- Fix unztell64() in minizip to work past 4GB. (Daniël Hörchner)
- Clean up minizip to reduce warnings for testing
- Add fallthrough comments for gcc
- Eliminate use of ULL constants
- Separate out address sanitizing from warnings in configure
- Remove destructive aspects of make distclean
- Check for cc masquerading as gcc or clang in configure
- Fix crc32.c to compile local functions only if used

Changes in 1.2.11 (15 Jan 2017)
- Fix deflate stored bug when pulling last block from window
- Permit immediate deflateParams changes before any deflate input

Changes in 1.2.10 (2 Jan 2017)
- Avoid warnings on snprintf() return value
- Fix bug in deflate_stored() for zero-length input
- Fix bug in gzwrite.c that produced corrupt gzip files
- Remove files to be installed before copying them in Makefile.in
- Add warnings when compiling with assembler code

Changes in 1.2.9 (31 Dec 2016)
- Fix contrib/minizip to permit unzipping with desktop API [Zouzou]
- Improve contrib/blast to return unused bytes
- Assure that gzoffset() is correct when appending
- Improve compress() and uncompress() to support large lengths
- Fix bug in test/example.c where error code not saved
- Remedy Coverity warning [Randers-Pehrson]
- Improve speed of gzprintf() in transparent mode
- Fix inflateInit2() bug when windowBits is 16 or 32
- Change DEBUG macro to ZLIB_DEBUG
- Avoid uninitialized access by gzclose_w()
- Allow building zlib outside of the source directory
- Fix bug that accepted invalid zlib header when windowBits is zero
- Fix gzseek() problem on MinGW due to buggy _lseeki64 there
- Loop on write() calls in gzwrite.c in case of non-blocking I/O
- Add --warn (-w) option to ./configure for more compiler warnings
- Reject a window size of 256 bytes if not using the zlib wrapper
- Fix bug when level 0 used with Z_HUFFMAN or Z_RLE
- Add --debug (-d) option to ./configure to define ZLIB_DEBUG
- Fix bugs in creating a very large gzip header
- Add uncompress2() function, which returns the input size used
- Assure that deflateParams() will not switch functions mid-block
- Dramatically speed up deflation for level 0 (storing)
- Add gzfread(), duplicating the interface of fread()
- Add gzfwrite(), duplicating the interface of fwrite()
- Add deflateGetDictionary() function
- Use snprintf() for later versions of Microsoft C
- Fix *Init macros to use z_ prefix when requested
- Replace as400 with os400 for OS/400 support [Monnerat]
- Add crc32_z() and adler32_z() functions with size_t lengths
- Update Visual Studio project files [AraHaan]

Changes in 1.2.8 (28 Apr 2013)
- Update contrib/minizip/iowin32.c for Windows RT [Vollant]
- Do not force Z_CONST for C++
- Clean up contrib/vstudio [Roß]
- Correct spelling error in zlib.h
- Fix mixed line endings in contrib/vstudio

Changes in 1.2.7.3 (13 Apr 2013)
- Fix version numbers and DLL names in contrib/vstudio/*/zlib.rc

Changes in 1.2.7.2 (13 Apr 2013)
- Change check for a four-byte type back to hexadecimal
- Fix typo in win32/Makefile.msc
- Add casts in gzwrite.c for pointer differences

Changes in 1.2.7.1 (24 Mar 2013)
- Replace use of unsafe string functions with snprintf if available
- Avoid including stddef.h on Windows for Z_SOLO compile [Niessink]
- Fix gzgetc undefine when Z_PREFIX set [Turk]
- Eliminate use of mktemp in Makefile (not always available)
- Fix bug in 'F' mode for gzopen()
- Add inflateGetDictionary() function
- Correct comment in deflate.h
- Use _snprintf for snprintf in Microsoft C
- On Darwin, only use /usr/bin/libtool if libtool is not Apple
- Delete "--version" file if created by "ar --version" [Richard G.]
- Fix configure check for veracity of compiler error return codes
- Fix CMake compilation of static lib for MSVC2010 x64
- Remove unused variable in infback9.c
- Fix argument checks in gzlog_compress() and gzlog_write()
- Clean up the usage of z_const and respect const usage within zlib
- Clean up examples/gzlog.[ch] comparisons of different types
- Avoid shift equal to bits in type (caused endless loop)
- Fix uninitialized value bug in gzputc() introduced by const patches
- Fix memory allocation error in examples/zran.c [Nor]
- Fix bug where gzopen(), gzclose() would write an empty file
- Fix bug in gzclose() when gzwrite() runs out of memory
- Check for input buffer malloc failure in examples/gzappend.c
- Add note to contrib/blast to use binary mode in stdio
- Fix comparisons of differently signed integers in contrib/blast
- Check for invalid code length codes in contrib/puff
- Fix serious but very rare decompression bug in inftrees.c
- Update inflateBack() comments, since inflate() can be faster
- Use underscored I/O function names for WINAPI_FAMILY
- Add _tr_flush_bits to the external symbols prefixed by --zprefix
- Add contrib/vstudio/vc10 pre-build step for static only
- Quote --version-script argument in CMakeLists.txt
- Don't specify --version-script on Apple platforms in CMakeLists.txt
- Fix casting error in contrib/testzlib/testzlib.c
- Fix types in contrib/minizip to match result of get_crc_table()
- Simplify contrib/vstudio/vc10 with 'd' suffix
- Add TOP support to win32/Makefile.msc
- Support i686 and amd64 assembler builds in CMakeLists.txt
- Fix typos in the use of _LARGEFILE64_SOURCE in zconf.h
- Add vc11 and vc12 build files to contrib/vstudio
- Add gzvprintf() as an undocumented function in zlib
- Fix configure for Sun shell
- Remove runtime check in configure for four-byte integer type
- Add casts and consts to ease user conversion to C++
- Add man pages for minizip and miniunzip
- In Makefile uninstall, don't rm if preceding cd fails
- Do not return Z_BUF_ERROR if deflateParam() has nothing to write

Changes in 1.2.7 (2 May 2012)
- Replace use of memmove() with a simple copy for portability
- Test for existence of strerror
- Restore gzgetc_ for backward compatibility with 1.2.6
- Fix build with non-GNU make on Solaris
- Require gcc 4.0 or later on Mac OS X to use the hidden attribute
- Include unistd.h for Watcom C
- Use __WATCOMC__ instead of __WATCOM__
- Do not use the visibility attribute if NO_VIZ defined
- Improve the detection of no hidden visibility attribute
- Avoid using __int64 for gcc or solo compilation
- Cast to char * in gzprintf to avoid warnings [Zinser]
- Fix make_vms.com for VAX [Zinser]
- Don't use library or built-in byte swaps
- Simplify test and use of gcc hidden attribute
- Fix bug in gzclose_w() when gzwrite() fails to allocate memory
- Add "x" (O_EXCL) and "e" (O_CLOEXEC) modes support to gzopen()
- Fix bug in test/minigzip.c for configure --solo
- Fix contrib/vstudio project link errors [Mohanathas]
- Add ability to choose the builder in make_vms.com [Schweda]
- Add DESTDIR support to mingw32 win32/Makefile.gcc
- Fix comments in win32/Makefile.gcc for proper usage
- Allow overriding the default install locations for cmake
- Generate and install the pkg-config file with cmake
- Build both a static and a shared version of zlib with cmake
- Include version symbols for cmake builds
- If using cmake with MSVC, add the source directory to the includes
- Remove unneeded EXTRA_CFLAGS from win32/Makefile.gcc [Truta]
- Move obsolete emx makefile to old [Truta]
- Allow the use of -Wundef when compiling or using zlib
- Avoid the use of the -u option with mktemp
- Improve inflate() documentation on the use of Z_FINISH
- Recognize clang as gcc
- Add gzopen_w() in Windows for wide character path names
- Rename zconf.h in CMakeLists.txt to move it out of the way
- Add source directory in CMakeLists.txt for building examples
- Look in build directory for zlib.pc in CMakeLists.txt
- Remove gzflags from zlibvc.def in vc9 and vc10
- Fix contrib/minizip compilation in the MinGW environment
- Update ./configure for Solaris, support --64 [Mooney]
- Remove -R. from Solaris shared build (possible security issue)
- Avoid race condition for parallel make (-j) running example
- Fix type mismatch between get_crc_table() and crc_table
- Fix parsing of version with "-" in CMakeLists.txt [Snider, Ziegler]
- Fix the path to zlib.map in CMakeLists.txt
- Force the native libtool in Mac OS X to avoid GNU libtool [Beebe]
- Add instructions to win32/Makefile.gcc for shared install [Torri]

Changes in 1.2.6.1 (12 Feb 2012)
- Avoid the use of the Objective-C reserved name "id"
- Include io.h in gzguts.h for Microsoft compilers
- Fix problem with ./configure --prefix and gzgetc macro
- Include gz_header definition when compiling zlib solo
- Put gzflags() functionality back in zutil.c
- Avoid library header include in crc32.c for Z_SOLO
- Use name in GCC_CLASSIC as C compiler for coverage testing, if set
- Minor cleanup in contrib/minizip/zip.c [Vollant]
- Update make_vms.com [Zinser]
- Remove unnecessary gzgetc_ function
- Use optimized byte swap operations for Microsoft and GNU [Snyder]
- Fix minor typo in zlib.h comments [Rzesniowiecki]

Changes in 1.2.6 (29 Jan 2012)
- Update the Pascal interface in contrib/pascal
- Fix function numbers for gzgetc_ in zlibvc.def files
- Fix configure.ac for contrib/minizip [Schiffer]
- Fix large-entry detection in minizip on 64-bit systems [Schiffer]
- Have ./configure use the compiler return code for error indication
- Fix CMakeLists.txt for cross compilation [McClure]
- Fix contrib/minizip/zip.c for 64-bit architectures [Dalsnes]
- Fix compilation of contrib/minizip on FreeBSD [Marquez]
- Correct suggested usages in win32/Makefile.msc [Shachar, Horvath]
- Include io.h for Turbo C / Borland C on all platforms [Truta]
- Make version explicit in contrib/minizip/configure.ac [Bosmans]
- Avoid warning for no encryption in contrib/minizip/zip.c [Vollant]
- Minor cleanup up contrib/minizip/unzip.c [Vollant]
- Fix bug when compiling minizip with C++ [Vollant]
- Protect for long name and extra fields in contrib/minizip [Vollant]
- Avoid some warnings in contrib/minizip [Vollant]
- Add -I../.. -L../.. to CFLAGS for minizip and miniunzip
- Add missing libs to minizip linker command
- Add support for VPATH builds in contrib/minizip
- Add an --enable-demos option to contrib/minizip/configure
- Add the generation of configure.log by ./configure
- Exit when required parameters not provided to win32/Makefile.gcc
- Have gzputc return the character written instead of the argument
- Use the -m option on ldconfig for BSD systems [Tobias]
- Correct in zlib.map when deflateResetKeep was added

Changes in 1.2.5.3 (15 Jan 2012)
- Restore gzgetc function for binary compatibility
- Do not use _lseeki64 under Borland C++ [Truta]
- Update win32/Makefile.msc to build test/*.c [Truta]
- Remove old/visualc6 given CMakefile and other alternatives
- Update AS400 build files and documentation [Monnerat]
- Update win32/Makefile.gcc to build test/*.c [Truta]
- Permit stronger flushes after Z_BLOCK flushes
- Avoid extraneous empty blocks when doing empty flushes
- Permit Z_NULL arguments to deflatePending
- Allow deflatePrime() to insert bits in the middle of a stream
- Remove second empty static block for Z_PARTIAL_FLUSH
- Write out all of the available bits when using Z_BLOCK
- Insert the first two strings in the hash table after a flush

Changes in 1.2.5.2 (17 Dec 2011)
- fix ld error: unable to find version dependency 'ZLIB_1.2.5'
- use relative symlinks for shared libs
- Avoid searching past window for Z_RLE strategy
- Assure that high-water mark initialization is always applied in deflate
- Add assertions to fill_window() in deflate.c to match comments
- Update python link in README
- Correct spelling error in gzread.c
- Fix bug in gzgets() for a concatenated empty gzip stream
- Correct error in comment for gz_make()
- Change gzread() and related to ignore junk after gzip streams
- Allow gzread() and related to continue after gzclearerr()
- Allow gzrewind() and gzseek() after a premature end-of-file
- Simplify gzseek() now that raw after gzip is ignored
- Change gzgetc() to a macro for speed (~40% speedup in testing)
- Fix gzclose() to return the actual error last encountered
- Always add large file support for windows
- Include zconf.h for windows large file support
- Include zconf.h.cmakein for windows large file support
- Update zconf.h.cmakein on make distclean
- Merge vestigial vsnprintf determination from zutil.h to gzguts.h
- Clarify how gzopen() appends in zlib.h comments
- Correct documentation of gzdirect() since junk at end now ignored
- Add a transparent write mode to gzopen() when 'T' is in the mode
- Update python link in zlib man page
- Get inffixed.h and MAKEFIXED result to match
- Add a ./config --solo option to make zlib subset with no library use
- Add undocumented inflateResetKeep() function for CAB file decoding
- Add --cover option to ./configure for gcc coverage testing
- Add #define ZLIB_CONST option to use const in the z_stream interface
- Add comment to gzdopen() in zlib.h to use dup() when using fileno()
- Note behavior of uncompress() to provide as much data as it can
- Add files in contrib/minizip to aid in building libminizip
- Split off AR options in Makefile.in and configure
- Change ON macro to Z_ARG to avoid application conflicts
- Facilitate compilation with Borland C++ for pragmas and vsnprintf
- Include io.h for Turbo C / Borland C++
- Move example.c and minigzip.c to test/
- Simplify incomplete code table filling in inflate_table()
- Remove code from inflate.c and infback.c that is impossible to execute
- Test the inflate code with full coverage
- Allow deflateSetDictionary, inflateSetDictionary at any time (in raw)
- Add deflateResetKeep and fix inflateResetKeep to retain dictionary
- Fix gzwrite.c to accommodate reduced memory zlib compilation
- Have inflate() with Z_FINISH avoid the allocation of a window
- Do not set strm->adler when doing raw inflate
- Fix gzeof() to behave just like feof() when read is not past end of file
- Fix bug in gzread.c when end-of-file is reached
- Avoid use of Z_BUF_ERROR in gz* functions except for premature EOF
- Document gzread() capability to read concurrently written files
- Remove hard-coding of resource compiler in CMakeLists.txt [Blammo]

Changes in 1.2.5.1 (10 Sep 2011)
- Update FAQ entry on shared builds (#13)
- Avoid symbolic argument to chmod in Makefile.in
- Fix bug and add consts in contrib/puff [Oberhumer]
- Update contrib/puff/zeros.raw test file to have all block types
- Add full coverage test for puff in contrib/puff/Makefile
- Fix static-only-build install in Makefile.in
- Fix bug in unzGetCurrentFileInfo() in contrib/minizip [Kuno]
- Add libz.a dependency to shared in Makefile.in for parallel builds
- Spell out "number" (instead of "nb") in zlib.h for total_in, total_out
- Replace $(...) with `...` in configure for non-bash sh [Bowler]
- Add darwin* to Darwin* and solaris* to SunOS\ 5* in configure [Groffen]
- Add solaris* to Linux* in configure to allow gcc use [Groffen]
- Add *bsd* to Linux* case in configure [Bar-Lev]
- Add inffast.obj to dependencies in win32/Makefile.msc
- Correct spelling error in deflate.h [Kohler]
- Change libzdll.a again to libz.dll.a (!) in win32/Makefile.gcc
- Add test to configure for GNU C looking for gcc in output of $cc -v
- Add zlib.pc generation to win32/Makefile.gcc [Weigelt]
- Fix bug in zlib.h for _FILE_OFFSET_BITS set and _LARGEFILE64_SOURCE not
- Add comment in zlib.h that adler32_combine with len2 < 0 makes no sense
- Make NO_DIVIDE option in adler32.c much faster (thanks to John Reiser)
- Make stronger test in zconf.h to include unistd.h for LFS
- Apply Darwin patches for 64-bit file offsets to contrib/minizip [Slack]
- Fix zlib.h LFS support when Z_PREFIX used
- Add updated as400 support (removed from old) [Monnerat]
- Avoid deflate sensitivity to volatile input data
- Avoid division in adler32_combine for NO_DIVIDE
- Clarify the use of Z_FINISH with deflateBound() amount of space
- Set binary for output file in puff.c
- Use u4 type for crc_table to avoid conversion warnings
- Apply casts in zlib.h to avoid conversion warnings
- Add OF to prototypes for adler32_combine_ and crc32_combine_ [Miller]
- Improve inflateSync() documentation to note indeterminacy
- Add deflatePending() function to return the amount of pending output
- Correct the spelling of "specification" in FAQ [Randers-Pehrson]
- Add a check in configure for stdarg.h, use for gzprintf()
- Check that pointers fit in ints when gzprint() compiled old style
- Add dummy name before $(SHAREDLIBV) in Makefile [Bar-Lev, Bowler]
- Delete line in configure that adds -L. libz.a to LDFLAGS [Weigelt]
- Add debug records in assembler code [Londer]
- Update RFC references to use http://tools.ietf.org/html/... [Li]
- Add --archs option, use of libtool to configure for Mac OS X [Borstel]

Changes in 1.2.5 (19 Apr 2010)
- Disable visibility attribute in win32/Makefile.gcc [Bar-Lev]
- Default to libdir as sharedlibdir in configure [Nieder]
- Update copyright dates on modified source files
- Update trees.c to be able to generate modified trees.h
- Exit configure for MinGW, suggesting win32/Makefile.gcc
- Check for NULL path in gz_open [Homurlu]

Changes in 1.2.4.5 (18 Apr 2010)
- Set sharedlibdir in configure [Torok]
- Set LDFLAGS in Makefile.in [Bar-Lev]
- Avoid mkdir objs race condition in Makefile.in [Bowler]
- Add ZLIB_INTERNAL in front of internal inter-module functions and arrays
- Define ZLIB_INTERNAL to hide internal functions and arrays for GNU C
- Don't use hidden attribute when it is a warning generator (e.g. Solaris)

Changes in 1.2.4.4 (18 Apr 2010)
- Fix CROSS_PREFIX executable testing, CHOST extract, mingw* [Torok]
- Undefine _LARGEFILE64_SOURCE in zconf.h if it is zero, but not if empty
- Try to use bash or ksh regardless of functionality of /bin/sh
- Fix configure incompatibility with NetBSD sh
- Remove attempt to run under bash or ksh since have better NetBSD fix
- Fix win32/Makefile.gcc for MinGW [Bar-Lev]
- Add diagnostic messages when using CROSS_PREFIX in configure
- Added --sharedlibdir option to configure [Weigelt]
- Use hidden visibility attribute when available [Frysinger]

Changes in 1.2.4.3 (10 Apr 2010)
- Only use CROSS_PREFIX in configure for ar and ranlib if they exist
- Use CROSS_PREFIX for nm [Bar-Lev]
- Assume _LARGEFILE64_SOURCE defined is equivalent to true
- Avoid use of undefined symbols in #if with && and ||
- Make *64 prototypes in gzguts.h consistent with functions
- Add -shared load option for MinGW in configure [Bowler]
- Move z_off64_t to public interface, use instead of off64_t
- Remove ! from shell test in configure (not portable to Solaris)
- Change +0 macro tests to -0 for possibly increased portability

Changes in 1.2.4.2 (9 Apr 2010)
- Add consistent carriage returns to readme.txt's in masmx86 and masmx64
- Really provide prototypes for *64 functions when building without LFS
- Only define unlink() in minigzip.c if unistd.h not included
- Update README to point to contrib/vstudio project files
- Move projects/vc6 to old/ and remove projects/
- Include stdlib.h in minigzip.c for setmode() definition under WinCE
- Clean up assembler builds in win32/Makefile.msc [Rowe]
- Include sys/types.h for Microsoft for off_t definition
- Fix memory leak on error in gz_open()
- Symbolize nm as $NM in configure [Weigelt]
- Use TEST_LDSHARED instead of LDSHARED to link test programs [Weigelt]
- Add +0 to _FILE_OFFSET_BITS and _LFS64_LARGEFILE in case not defined
- Fix bug in gzeof() to take into account unused input data
- Avoid initialization of structures with variables in puff.c
- Updated win32/README-WIN32.txt [Rowe]

Changes in 1.2.4.1 (28 Mar 2010)
- Remove the use of [a-z] constructs for sed in configure [gentoo 310225]
- Remove $(SHAREDLIB) from LIBS in Makefile.in [Creech]
- Restore "for debugging" comment on sprintf() in gzlib.c
- Remove fdopen for MVS from gzguts.h
- Put new README-WIN32.txt in win32 [Rowe]
- Add check for shell to configure and invoke another shell if needed
- Fix big fat stinking bug in gzseek() on uncompressed files
- Remove vestigial F_OPEN64 define in zutil.h
- Set and check the value of _LARGEFILE_SOURCE and _LARGEFILE64_SOURCE
- Avoid errors on non-LFS systems when applications define LFS macros
- Set EXE to ".exe" in configure for MINGW [Kahle]
- Match crc32() in crc32.c exactly to the prototype in zlib.h [Sherrill]
- Add prefix for cross-compilation in win32/makefile.gcc [Bar-Lev]
- Add DLL install in win32/makefile.gcc [Bar-Lev]
- Allow Linux* or linux* from uname in configure [Bar-Lev]
- Allow ldconfig to be redefined in configure and Makefile.in [Bar-Lev]
- Add cross-compilation prefixes to configure [Bar-Lev]
- Match type exactly in gz_load() invocation in gzread.c
- Match type exactly of zcalloc() in zutil.c to zlib.h alloc_func
- Provide prototypes for *64 functions when building zlib without LFS
- Don't use -lc when linking shared library on MinGW
- Remove errno.h check in configure and vestigial errno code in zutil.h

Changes in 1.2.4 (14 Mar 2010)
- Fix VER3 extraction in configure for no fourth subversion
- Update zlib.3, add docs to Makefile.in to make .pdf out of it
- Add zlib.3.pdf to distribution
- Don't set error code in gzerror() if passed pointer is NULL
- Apply destination directory fixes to CMakeLists.txt [Lowman]
- Move #cmakedefine's to a new zconf.in.cmakein
- Restore zconf.h for builds that don't use configure or cmake
- Add distclean to dummy Makefile for convenience
- Update and improve INDEX, README, and FAQ
- Update CMakeLists.txt for the return of zconf.h [Lowman]
- Update contrib/vstudio/vc9 and vc10 [Vollant]
- Change libz.dll.a back to libzdll.a in win32/Makefile.gcc
- Apply license and readme changes to contrib/asm686 [Raiter]
- Check file name lengths and add -c option in minigzip.c [Li]
- Update contrib/amd64 and contrib/masmx86/ [Vollant]
- Avoid use of "eof" parameter in trees.c to not shadow library variable
- Update make_vms.com for removal of zlibdefs.h [Zinser]
- Update assembler code and vstudio projects in contrib [Vollant]
- Remove outdated assembler code contrib/masm686 and contrib/asm586
- Remove old vc7 and vc8 from contrib/vstudio
- Update win32/Makefile.msc, add ZLIB_VER_SUBREVISION [Rowe]
- Fix memory leaks in gzclose_r() and gzclose_w(), file leak in gz_open()
- Add contrib/gcc_gvmat64 for longest_match and inflate_fast [Vollant]
- Remove *64 functions from win32/zlib.def (they're not 64-bit yet)
- Fix bug in void-returning vsprintf() case in gzwrite.c
- Fix name change from inflate.h in contrib/inflate86/inffas86.c
- Check if temporary file exists before removing in make_vms.com [Zinser]
- Fix make install and uninstall for --static option
- Fix usage of _MSC_VER in gzguts.h and zutil.h [Truta]
- Update readme.txt in contrib/masmx64 and masmx86 to assemble

Changes in 1.2.3.9 (21 Feb 2010)
- Expunge gzio.c
- Move as400 build information to old
- Fix updates in contrib/minizip and contrib/vstudio
- Add const to vsnprintf test in configure to avoid warnings [Weigelt]
- Delete zconf.h (made by configure) [Weigelt]
- Change zconf.in.h to zconf.h.in per convention [Weigelt]
- Check for NULL buf in gzgets()
- Return empty string for gzgets() with len == 1 (like fgets())
- Fix description of gzgets() in zlib.h for end-of-file, NULL return
- Update minizip to 1.1 [Vollant]
- Avoid MSVC loss of data warnings in gzread.c, gzwrite.c
- Note in zlib.h that gzerror() should be used to distinguish from EOF
- Remove use of snprintf() from gzlib.c
- Fix bug in gzseek()
- Update contrib/vstudio, adding vc9 and vc10 [Kuno, Vollant]
- Fix zconf.h generation in CMakeLists.txt [Lowman]
- Improve comments in zconf.h where modified by configure

Changes in 1.2.3.8 (13 Feb 2010)
- Clean up text files (tabs, trailing whitespace, etc.) [Oberhumer]
- Use z_off64_t in gz_zero() and gz_skip() to match state->skip
- Avoid comparison problem when sizeof(int) == sizeof(z_off64_t)
- Revert to Makefile.in from 1.2.3.6 (live with the clutter)
- Fix missing error return in gzflush(), add zlib.h note
- Add *64 functions to zlib.map [Levin]
- Fix signed/unsigned comparison in gz_comp()
- Use SFLAGS when testing shared linking in configure
- Add --64 option to ./configure to use -m64 with gcc
- Fix ./configure --help to correctly name options
- Have make fail if a test fails [Levin]
- Avoid buffer overrun in contrib/masmx64/gvmat64.asm [Simpson]
- Remove assembler object files from contrib

Changes in 1.2.3.7 (24 Jan 2010)
- Always gzopen() with O_LARGEFILE if available
- Fix gzdirect() to work immediately after gzopen() or gzdopen()
- Make gzdirect() more precise when the state changes while reading
- Improve zlib.h documentation in many places
- Catch memory allocation failure in gz_open()
- Complete close operation if seek forward in gzclose_w() fails
- Return Z_ERRNO from gzclose_r() if close() fails
- Return Z_STREAM_ERROR instead of EOF for gzclose() being passed NULL
- Return zero for gzwrite() errors to match zlib.h description
- Return -1 on gzputs() error to match zlib.h description
- Add zconf.in.h to allow recovery from configure modification [Weigelt]
- Fix static library permissions in Makefile.in [Weigelt]
- Avoid warnings in configure tests that hide functionality [Weigelt]
- Add *BSD and DragonFly to Linux case in configure [gentoo 123571]
- Change libzdll.a to libz.dll.a in win32/Makefile.gcc [gentoo 288212]
- Avoid access of uninitialized data for first inflateReset2 call [Gomes]
- Keep object files in subdirectories to reduce the clutter somewhat
- Remove default Makefile and zlibdefs.h, add dummy Makefile
- Add new external functions to Z_PREFIX, remove duplicates, z_z_ -> z_
- Remove zlibdefs.h completely -- modify zconf.h instead

Changes in 1.2.3.6 (17 Jan 2010)
- Avoid void * arithmetic in gzread.c and gzwrite.c
- Make compilers happier with const char * for gz_error message
- Avoid unused parameter warning in inflate.c
- Avoid signed-unsigned comparison warning in inflate.c
- Indent #pragma's for traditional C
- Fix usage of strwinerror() in glib.c, change to gz_strwinerror()
- Correct email address in configure for system options
- Update make_vms.com and add make_vms.com to contrib/minizip [Zinser]
- Update zlib.map [Brown]
- Fix Makefile.in for Solaris 10 make of example64 and minizip64 [Torok]
- Apply various fixes to CMakeLists.txt [Lowman]
- Add checks on len in gzread() and gzwrite()
- Add error message for no more room for gzungetc()
- Remove zlib version check in gzwrite()
- Defer compression of gzprintf() result until need to
- Use snprintf() in gzdopen() if available
- Remove USE_MMAP configuration determination (only used by minigzip)
- Remove examples/pigz.c (available separately)
- Update examples/gun.c to 1.6

Changes in 1.2.3.5 (8 Jan 2010)
- Add space after #if in zutil.h for some compilers
- Fix relatively harmless bug in deflate_fast() [Exarevsky]
- Fix same problem in deflate_slow()
- Add $(SHAREDLIBV) to LIBS in Makefile.in [Brown]
- Add deflate_rle() for faster Z_RLE strategy run-length encoding
- Add deflate_huff() for faster Z_HUFFMAN_ONLY encoding
- Change name of "write" variable in inffast.c to avoid library collisions
- Fix premature EOF from gzread() in gzio.c [Brown]
- Use zlib header window size if windowBits is 0 in inflateInit2()
- Remove compressBound() call in deflate.c to avoid linking compress.o
- Replace use of errno in gz* with functions, support WinCE [Alves]
- Provide alternative to perror() in minigzip.c for WinCE [Alves]
- Don't use _vsnprintf on later versions of MSVC [Lowman]
- Add CMake build script and input file [Lowman]
- Update contrib/minizip to 1.1 [Svensson, Vollant]
- Moved nintendods directory from contrib to root
- Replace gzio.c with a new set of routines with the same functionality
- Add gzbuffer(), gzoffset(), gzclose_r(), gzclose_w() as part of above
- Update contrib/minizip to 1.1b
- Change gzeof() to return 0 on error instead of -1 to agree with zlib.h

Changes in 1.2.3.4 (21 Dec 2009)
- Use old school .SUFFIXES in Makefile.in for FreeBSD compatibility
- Update comments in configure and Makefile.in for default --shared
- Fix test -z's in configure [Marquess]
- Build examplesh and minigzipsh when not testing
- Change NULL's to Z_NULL's in deflate.c and in comments in zlib.h
- Import LDFLAGS from the environment in configure
- Fix configure to populate SFLAGS with discovered CFLAGS options
- Adapt make_vms.com to the new Makefile.in [Zinser]
- Add zlib2ansi script for C++ compilation [Marquess]
- Add _FILE_OFFSET_BITS=64 test to make test (when applicable)
- Add AMD64 assembler code for longest match to contrib [Teterin]
- Include options from $SFLAGS when doing $LDSHARED
- Simplify 64-bit file support by introducing z_off64_t type
- Make shared object files in objs directory to work around old Sun cc
- Use only three-part version number for Darwin shared compiles
- Add rc option to ar in Makefile.in for when ./configure not run
- Add -WI,-rpath,. to LDFLAGS for OSF 1 V4*
- Set LD_LIBRARYN32_PATH for SGI IRIX shared compile
- Protect against _FILE_OFFSET_BITS being defined when compiling zlib
- Rename Makefile.in targets allstatic to static and allshared to shared
- Fix static and shared Makefile.in targets to be independent
- Correct error return bug in gz_open() by setting state [Brown]
- Put spaces before ;;'s in configure for better sh compatibility
- Add pigz.c (parallel implementation of gzip) to examples/
- Correct constant in crc32.c to UL [Leventhal]
- Reject negative lengths in crc32_combine()
- Add inflateReset2() function to work like inflateEnd()/inflateInit2()
- Include sys/types.h for _LARGEFILE64_SOURCE [Brown]
- Correct typo in doc/algorithm.txt [Janik]
- Fix bug in adler32_combine() [Zhu]
- Catch missing-end-of-block-code error in all inflates and in puff
    Assures that random input to inflate eventually results in an error
- Added enough.c (calculation of ENOUGH for inftrees.h) to examples/
- Update ENOUGH and its usage to reflect discovered bounds
- Fix gzerror() error report on empty input file [Brown]
- Add ush casts in trees.c to avoid pedantic runtime errors
- Fix typo in zlib.h uncompress() description [Reiss]
- Correct inflate() comments with regard to automatic header detection
- Remove deprecation comment on Z_PARTIAL_FLUSH (it stays)
- Put new version of gzlog (2.0) in examples with interruption recovery
- Add puff compile option to permit invalid distance-too-far streams
- Add puff TEST command options, ability to read piped input
- Prototype the *64 functions in zlib.h when _FILE_OFFSET_BITS == 64, but
  _LARGEFILE64_SOURCE not defined
- Fix Z_FULL_FLUSH to truly erase the past by resetting s->strstart
- Fix deflateSetDictionary() to use all 32K for output consistency
- Remove extraneous #define MIN_LOOKAHEAD in deflate.c (in deflate.h)
- Clear bytes after deflate lookahead to avoid use of uninitialized data
- Change a limit in inftrees.c to be more transparent to Coverity Prevent
- Update win32/zlib.def with exported symbols from zlib.h
- Correct spelling errors in zlib.h [Willem, Sobrado]
- Allow Z_BLOCK for deflate() to force a new block
- Allow negative bits in inflatePrime() to delete existing bit buffer
- Add Z_TREES flush option to inflate() to return at end of trees
- Add inflateMark() to return current state information for random access
- Add Makefile for NintendoDS to contrib [Costa]
- Add -w in configure compile tests to avoid spurious warnings [Beucler]
- Fix typos in zlib.h comments for deflateSetDictionary()
- Fix EOF detection in transparent gzread() [Maier]

Changes in 1.2.3.3 (2 October 2006)
- Make --shared the default for configure, add a --static option
- Add compile option to permit invalid distance-too-far streams
- Add inflateUndermine() function which is required to enable above
- Remove use of "this" variable name for C++ compatibility [Marquess]
- Add testing of shared library in make test, if shared library built
- Use ftello() and fseeko() if available instead of ftell() and fseek()
- Provide two versions of all functions that use the z_off_t type for
  binary compatibility -- a normal version and a 64-bit offset version,
  per the Large File Support Extension when _LARGEFILE64_SOURCE is
  defined; use the 64-bit versions by default when _FILE_OFFSET_BITS
  is defined to be 64
- Add a --uname= option to configure to perhaps help with cross-compiling

Changes in 1.2.3.2 (3 September 2006)
- Turn off silly Borland warnings [Hay]
- Use off64_t and define _LARGEFILE64_SOURCE when present
- Fix missing dependency on inffixed.h in Makefile.in
- Rig configure --shared to build both shared and static [Teredesai, Truta]
- Remove zconf.in.h and instead create a new zlibdefs.h file
- Fix contrib/minizip/unzip.c non-encrypted after encrypted [Vollant]
- Add treebuild.xml (see http://treebuild.metux.de/) [Weigelt]

Changes in 1.2.3.1 (16 August 2006)
- Add watcom directory with OpenWatcom make files [Daniel]
- Remove #undef of FAR in zconf.in.h for MVS [Fedtke]
- Update make_vms.com [Zinser]
- Use -fPIC for shared build in configure [Teredesai, Nicholson]
- Use only major version number for libz.so on IRIX and OSF1 [Reinholdtsen]
- Use fdopen() (not _fdopen()) for Interix in zutil.h [Bäck]
- Add some FAQ entries about the contrib directory
- Update the MVS question in the FAQ
- Avoid extraneous reads after EOF in gzio.c [Brown]
- Correct spelling of "successfully" in gzio.c [Randers-Pehrson]
- Add comments to zlib.h about gzerror() usage [Brown]
- Set extra flags in gzip header in gzopen() like deflate() does
- Make configure options more compatible with double-dash conventions
  [Weigelt]
- Clean up compilation under Solaris SunStudio cc [Rowe, Reinholdtsen]
- Fix uninstall target in Makefile.in [Truta]
- Add pkgconfig support [Weigelt]
- Use $(DESTDIR) macro in Makefile.in [Reinholdtsen, Weigelt]
- Replace set_data_type() with a more accurate detect_data_type() in
  trees.c, according to the txtvsbin.txt document [Truta]
- Swap the order of #include <stdio.h> and #include "zlib.h" in
  gzio.c, example.c and minigzip.c [Truta]
- Shut up annoying VS2005 warnings about standard C deprecation [Rowe,
  Truta] (where?)
- Fix target "clean" from win32/Makefile.bor [Truta]
- Create .pdb and .manifest files in win32/makefile.msc [Ziegler, Rowe]
- Update zlib www home address in win32/DLL_FAQ.txt [Truta]
- Update contrib/masmx86/inffas32.asm for VS2005 [Vollant, Van Wassenhove]
- Enable browse info in the "Debug" and "ASM Debug" configurations in
  the Visual C++ 6 project, and set (non-ASM) "Debug" as default [Truta]
- Add pkgconfig support [Weigelt]
- Add ZLIB_VER_MAJOR, ZLIB_VER_MINOR and ZLIB_VER_REVISION in zlib.h,
  for use in win32/zlib1.rc [Polushin, Rowe, Truta]
- Add a document that explains the new text detection scheme to
  doc/txtvsbin.txt [Truta]
- Add rfc1950.txt, rfc1951.txt and rfc1952.txt to doc/ [Truta]
- Move algorithm.txt into doc/ [Truta]
- Synchronize FAQ with website
- Fix compressBound(), was low for some pathological cases [Fearnley]
- Take into account wrapper variations in deflateBound()
- Set examples/zpipe.c input and output to binary mode for Windows
- Update examples/zlib_how.html with new zpipe.c (also web site)
- Fix some warnings in examples/gzlog.c and examples/zran.c (it seems
  that gcc became pickier in 4.0)
- Add zlib.map for Linux: "All symbols from zlib-1.1.4 remain
  un-versioned, the patch adds versioning only for symbols introduced in
  zlib-1.2.0 or later.  It also declares as local those symbols which are
  not designed to be exported." [Levin]
- Update Z_PREFIX list in zconf.in.h, add --zprefix option to configure
- Do not initialize global static by default in trees.c, add a response
  NO_INIT_GLOBAL_POINTERS to initialize them if needed [Marquess]
- Don't use strerror() in gzio.c under WinCE [Yakimov]
- Don't use errno.h in zutil.h under WinCE [Yakimov]
- Move arguments for AR to its usage to allow replacing ar [Marot]
- Add HAVE_VISIBILITY_PRAGMA in zconf.in.h for Mozilla [Randers-Pehrson]
- Improve inflateInit() and inflateInit2() documentation
- Fix structure size comment in inflate.h
- Change configure help option from --h* to --help [Santos]

Changes in 1.2.3 (18 July 2005)
- Apply security vulnerability fixes to contrib/infback9 as well
- Clean up some text files (carriage returns, trailing space)
- Update testzlib, vstudio, masmx64, and masmx86 in contrib [Vollant]

Changes in 1.2.2.4 (11 July 2005)
- Add inflatePrime() function for starting inflation at bit boundary
- Avoid some Visual C warnings in deflate.c
- Avoid more silly Visual C warnings in inflate.c and inftrees.c for 64-bit
  compile
- Fix some spelling errors in comments [Betts]
- Correct inflateInit2() error return documentation in zlib.h
- Add zran.c example of compressed data random access to examples
  directory, shows use of inflatePrime()
- Fix cast for assignments to strm->state in inflate.c and infback.c
- Fix zlibCompileFlags() in zutil.c to use 1L for long shifts [Oberhumer]
- Move declarations of gf2 functions to right place in crc32.c [Oberhumer]
- Add cast in trees.c t avoid a warning [Oberhumer]
- Avoid some warnings in fitblk.c, gun.c, gzjoin.c in examples [Oberhumer]
- Update make_vms.com [Zinser]
- Initialize state->write in inflateReset() since copied in inflate_fast()
- Be more strict on incomplete code sets in inflate_table() and increase
  ENOUGH and MAXD -- this repairs a possible security vulnerability for
  invalid inflate input.  Thanks to Tavis Ormandy and Markus Oberhumer for
  discovering the vulnerability and providing test cases
- Add ia64 support to configure for HP-UX [Smith]
- Add error return to gzread() for format or i/o error [Levin]
- Use malloc.h for OS/2 [Necasek]

Changes in 1.2.2.3 (27 May 2005)
- Replace 1U constants in inflate.c and inftrees.c for 64-bit compile
- Typecast fread() return values in gzio.c [Vollant]
- Remove trailing space in minigzip.c outmode (VC++ can't deal with it)
- Fix crc check bug in gzread() after gzungetc() [Heiner]
- Add the deflateTune() function to adjust internal compression parameters
- Add a fast gzip decompressor, gun.c, to examples (use of inflateBack)
- Remove an incorrect assertion in examples/zpipe.c
- Add C++ wrapper in infback9.h [Donais]
- Fix bug in inflateCopy() when decoding fixed codes
- Note in zlib.h how much deflateSetDictionary() actually uses
- Remove USE_DICT_HEAD in deflate.c (would mess up inflate if used)
- Add _WIN32_WCE to define WIN32 in zconf.in.h [Spencer]
- Don't include stderr.h or errno.h for _WIN32_WCE in zutil.h [Spencer]
- Add gzdirect() function to indicate transparent reads
- Update contrib/minizip [Vollant]
- Fix compilation of deflate.c when both ASMV and FASTEST [Oberhumer]
- Add casts in crc32.c to avoid warnings [Oberhumer]
- Add contrib/masmx64 [Vollant]
- Update contrib/asm586, asm686, masmx86, testzlib, vstudio [Vollant]

Changes in 1.2.2.2 (30 December 2004)
- Replace structure assignments in deflate.c and inflate.c with zmemcpy to
  avoid implicit memcpy calls (portability for no-library compilation)
- Increase sprintf() buffer size in gzdopen() to allow for large numbers
- Add INFLATE_STRICT to check distances against zlib header
- Improve WinCE errno handling and comments [Chang]
- Remove comment about no gzip header processing in FAQ
- Add Z_FIXED strategy option to deflateInit2() to force fixed trees
- Add updated make_vms.com [Coghlan], update README
- Create a new "examples" directory, move gzappend.c there, add zpipe.c,
  fitblk.c, gzlog.[ch], gzjoin.c, and zlib_how.html
- Add FAQ entry and comments in deflate.c on uninitialized memory access
- Add Solaris 9 make options in configure [Gilbert]
- Allow strerror() usage in gzio.c for STDC
- Fix DecompressBuf in contrib/delphi/ZLib.pas [ManChesTer]
- Update contrib/masmx86/inffas32.asm and gvmat32.asm [Vollant]
- Use z_off_t for adler32_combine() and crc32_combine() lengths
- Make adler32() much faster for small len
- Use OS_CODE in deflate() default gzip header

Changes in 1.2.2.1 (31 October 2004)
- Allow inflateSetDictionary() call for raw inflate
- Fix inflate header crc check bug for file names and comments
- Add deflateSetHeader() and gz_header structure for custom gzip headers
- Add inflateGetheader() to retrieve gzip headers
- Add crc32_combine() and adler32_combine() functions
- Add alloc_func, free_func, in_func, out_func to Z_PREFIX list
- Use zstreamp consistently in zlib.h (inflate_back functions)
- Remove GUNZIP condition from definition of inflate_mode in inflate.h
  and in contrib/inflate86/inffast.S [Truta, Anderson]
- Add support for AMD64 in contrib/inflate86/inffas86.c [Anderson]
- Update projects/README.projects and projects/visualc6 [Truta]
- Update win32/DLL_FAQ.txt [Truta]
- Avoid warning under NO_GZCOMPRESS in gzio.c; fix typo [Truta]
- Deprecate Z_ASCII; use Z_TEXT instead [Truta]
- Use a new algorithm for setting strm->data_type in trees.c [Truta]
- Do not define an exit() prototype in zutil.c unless DEBUG defined
- Remove prototype of exit() from zutil.c, example.c, minigzip.c [Truta]
- Add comment in zlib.h for Z_NO_FLUSH parameter to deflate()
- Fix Darwin build version identification [Peterson]

Changes in 1.2.2 (3 October 2004)
- Update zlib.h comments on gzip in-memory processing
- Set adler to 1 in inflateReset() to support Java test suite [Walles]
- Add contrib/dotzlib [Ravn]
- Update win32/DLL_FAQ.txt [Truta]
- Update contrib/minizip [Vollant]
- Move contrib/visual-basic.txt to old/ [Truta]
- Fix assembler builds in projects/visualc6/ [Truta]

Changes in 1.2.1.2 (9 September 2004)
- Update INDEX file
- Fix trees.c to update strm->data_type (no one ever noticed!)
- Fix bug in error case in inflate.c, infback.c, and infback9.c [Brown]
- Add "volatile" to crc table flag declaration (for DYNAMIC_CRC_TABLE)
- Add limited multitasking protection to DYNAMIC_CRC_TABLE
- Add NO_vsnprintf for VMS in zutil.h [Mozilla]
- Don't declare strerror() under VMS [Mozilla]
- Add comment to DYNAMIC_CRC_TABLE to use get_crc_table() to initialize
- Update contrib/ada [Anisimkov]
- Update contrib/minizip [Vollant]
- Fix configure to not hardcode directories for Darwin [Peterson]
- Fix gzio.c to not return error on empty files [Brown]
- Fix indentation; update version in contrib/delphi/ZLib.pas and
  contrib/pascal/zlibpas.pas [Truta]
- Update mkasm.bat in contrib/masmx86 [Truta]
- Update contrib/untgz [Truta]
- Add projects/README.projects [Truta]
- Add project for MS Visual C++ 6.0 in projects/visualc6 [Cadieux, Truta]
- Update win32/DLL_FAQ.txt [Truta]
- Update list of Z_PREFIX symbols in zconf.h [Randers-Pehrson, Truta]
- Remove an unnecessary assignment to curr in inftrees.c [Truta]
- Add OS/2 to exe builds in configure [Poltorak]
- Remove err dummy parameter in zlib.h [Kientzle]

Changes in 1.2.1.1 (9 January 2004)
- Update email address in README
- Several FAQ updates
- Fix a big fat bug in inftrees.c that prevented decoding valid
  dynamic blocks with only literals and no distance codes --
  Thanks to "Hot Emu" for the bug report and sample file
- Add a note to puff.c on no distance codes case

Changes in 1.2.1 (17 November 2003)
- Remove a tab in contrib/gzappend/gzappend.c
- Update some interfaces in contrib for new zlib functions
- Update zlib version number in some contrib entries
- Add Windows CE definition for ptrdiff_t in zutil.h [Mai, Truta]
- Support shared libraries on Hurd and KFreeBSD [Brown]
- Fix error in NO_DIVIDE option of adler32.c

Changes in 1.2.0.8 (4 November 2003)
- Update version in contrib/delphi/ZLib.pas and contrib/pascal/zlibpas.pas
- Add experimental NO_DIVIDE #define in adler32.c
    - Possibly faster on some processors (let me know if it is)
- Correct Z_BLOCK to not return on first inflate call if no wrap
- Fix strm->data_type on inflate() return to correctly indicate EOB
- Add deflatePrime() function for appending in the middle of a byte
- Add contrib/gzappend for an example of appending to a stream
- Update win32/DLL_FAQ.txt [Truta]
- Delete Turbo C comment in README [Truta]
- Improve some indentation in zconf.h [Truta]
- Fix infinite loop on bad input in configure script [Church]
- Fix gzeof() for concatenated gzip files [Johnson]
- Add example to contrib/visual-basic.txt [Michael B.]
- Add -p to mkdir's in Makefile.in [vda]
- Fix configure to properly detect presence or lack of printf functions
- Add AS400 support [Monnerat]
- Add a little Cygwin support [Wilson]

Changes in 1.2.0.7 (21 September 2003)
- Correct some debug formats in contrib/infback9
- Cast a type in a debug statement in trees.c
- Change search and replace delimiter in configure from % to # [Beebe]
- Update contrib/untgz to 0.2 with various fixes [Truta]
- Add build support for Amiga [Nikl]
- Remove some directories in old that have been updated to 1.2
- Add dylib building for Mac OS X in configure and Makefile.in
- Remove old distribution stuff from Makefile
- Update README to point to DLL_FAQ.txt, and add comment on Mac OS X
- Update links in README

Changes in 1.2.0.6 (13 September 2003)
- Minor FAQ updates
- Update contrib/minizip to 1.00 [Vollant]
- Remove test of gz functions in example.c when GZ_COMPRESS defi
... [truncated]
```

## General Information

This file is part of the OpenCV repository infrastructure.

