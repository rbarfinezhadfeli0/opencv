# Documentation for `3rdparty/zlib-ng/arch/s390/Makefile.in`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/arch/s390/Makefile.in`
- **File Name**: `Makefile.in`
- **File Size**: 1,073 bytes
- **File Type**: .in
- **Link to Source**: [3rdparty/zlib-ng/arch/s390/Makefile.in](../../../../3rdparty/zlib-ng/arch/s390/Makefile.in)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng/arch/s390` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
# Makefile for zlib-ng
# Copyright (C) 1995-2013 Jean-loup Gailly, Mark Adler
# For conditions of distribution and use, see copyright notice in zlib.h

CC=
CFLAGS=
SFLAGS=
INCLUDES=
SUFFIX=
VGFMAFLAG=
NOLTOFLAG=

SRCDIR=.
SRCTOP=../..
TOPDIR=$(SRCTOP)

s390_features.o:
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/s390_features.c

s390_features.lo:
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/s390_features.c

dfltcc_deflate.o:
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/dfltcc_deflate.c

dfltcc_deflate.lo:
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/dfltcc_deflate.c

dfltcc_inflate.o:
	$(CC) $(CFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/dfltcc_inflate.c

dfltcc_inflate.lo:
	$(CC) $(SFLAGS) $(INCLUDES) -c -o $@ $(SRCDIR)/dfltcc_inflate.c

crc32-vx.o:
	$(CC) $(CFLAGS) $(VGFMAFLAG) $(NOLTOFLAG) $(INCLUDES) -c -o $@ $(SRCDIR)/crc32-vx.c

crc32-vx.lo:
	$(CC) $(SFLAGS) $(VGFMAFLAG) $(NOLTOFLAG) $(INCLUDES) -c -o $@ $(SRCDIR)/crc32-vx.c

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

