# Documentation for `docs/3rdparty/zlib-ng/arch/generic/Makefile.in_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/zlib-ng/arch/generic/Makefile.in_docs.md`
- **File Name**: `Makefile.in_docs.md`
- **File Size**: 3,498 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/zlib-ng/arch/generic/Makefile.in_docs.md](../../../../../docs/3rdparty/zlib-ng/arch/generic/Makefile.in_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/zlib-ng/arch/generic` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/zlib-ng/arch/generic/Makefile.in`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/arch/generic/Makefile.in`
- **File Name**: `Makefile.in`
- **File Size**: 2,891 bytes
- **File Type**: .in
- **Link to Source**: [3rdparty/zlib-ng/arch/generic/Makefile.in](../../../../3rdparty/zlib-ng/arch/generic/Makefile.in)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng/arch/generic` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
# Makefile for zlib-ng
# Copyright (C) 1995-2013 Jean-loup Gailly, Mark Adler
# Copyright (C) 2024 Hans Kristian Rosbach
# For conditions of distribution and use, see copyright notice in zlib.h

CC=
CFLAGS=
SFLAGS=
INCLUDES=

SRCDIR=.
SRCTOP=../..
TOPDIR=$(SRCTOP)

all: \
 adler32_c.o adler32_c.lo \
 adler32_fold_c.o adler32_fold_c.lo \
 chunkset_c.o chunkset_c.lo \
 compare256_c.o compare256_c.lo \
 crc32_braid_c.o crc32_braid_c.lo \
 crc32_fold_c.o crc32_fold_c.lo \
 slide_hash_c.o slide_hash_c.lo


adler32_c.o: $(SRCDIR)/adler32_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/adler32_p.h
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/adler32_c.c

adler32_c.lo: $(SRCDIR)/adler32_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/adler32_p.h
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/adler32_c.c

adler32_fold_c.o: $(SRCDIR)/adler32_fold_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/functable.h
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/adler32_fold_c.c

adler32_fold_c.lo: $(SRCDIR)/adler32_fold_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/functable.h
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/adler32_fold_c.c

chunkset_c.o: $(SRCDIR)/chunkset_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/chunkset_tpl.h $(SRCTOP)/inffast_tpl.h
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/chunkset_c.c

chunkset_c.lo: $(SRCDIR)/chunkset_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/chunkset_tpl.h $(SRCTOP)/inffast_tpl.h
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/chunkset_c.c

compare256_c.o: $(SRCDIR)/compare256_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/zutil_p.h $(SRCTOP)/deflate.h $(SRCTOP)/fallback_builtins.h
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/compare256_c.c

compare256_c.lo: $(SRCDIR)/compare256_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/zutil_p.h $(SRCTOP)/deflate.h $(SRCTOP)/fallback_builtins.h
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/compare256_c.c

crc32_braid_c.o: $(SRCDIR)/crc32_braid_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/crc32_braid_p.h $(SRCTOP)/crc32_braid_tbl.h
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/crc32_braid_c.c

crc32_braid_c.lo: $(SRCDIR)/crc32_braid_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/crc32_braid_p.h $(SRCTOP)/crc32_braid_tbl.h
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/crc32_braid_c.c

crc32_fold_c.o: $(SRCDIR)/crc32_fold_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/functable.h
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/crc32_fold_c.c

crc32_fold_c.lo: $(SRCDIR)/crc32_fold_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/functable.h
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/crc32_fold_c.c

slide_hash_c.o: $(SRCDIR)/slide_hash_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/deflate.h
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/slide_hash_c.c

slide_hash_c.lo: $(SRCDIR)/slide_hash_c.c  $(SRCTOP)/zbuild.h $(SRCTOP)/deflate.h
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/slide_hash_c.c


mostlyclean: clean
clean:
	rm -f *.o *.lo *~
	rm -rf objs
	rm -f *.gcda *.gcno *.gcov

distclean: clean
	rm -f Makefile

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

