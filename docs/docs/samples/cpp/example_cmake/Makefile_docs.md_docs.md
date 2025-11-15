# Documentation for `docs/samples/cpp/example_cmake/Makefile_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/example_cmake/Makefile_docs.md`
- **File Name**: `Makefile_docs.md`
- **File Size**: 857 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/example_cmake/Makefile_docs.md](../../../../docs/samples/cpp/example_cmake/Makefile_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/example_cmake` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/example_cmake/Makefile`

## File Metadata

- **Full Path**: `samples/cpp/example_cmake/Makefile`
- **File Name**: `Makefile`
- **File Size**: 281 bytes
- **File Type**: no extension
- **Link to Source**: [samples/cpp/example_cmake/Makefile](../../../samples/cpp/example_cmake/Makefile)

## Purpose and Role

This file is located in the `samples/cpp/example_cmake` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
CXX ?= g++

CXXFLAGS += -c -Wall $(shell pkg-config --cflags opencv)
LDFLAGS += $(shell pkg-config --libs --static opencv)

all: opencv_example

opencv_example: example.o; $(CXX) $< -o $@ $(LDFLAGS)

%.o: %.cpp; $(CXX) $< -o $@ $(CXXFLAGS)

clean: ; rm -f example.o opencv_example

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

