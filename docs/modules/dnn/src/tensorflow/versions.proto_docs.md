# Documentation for `modules/dnn/src/tensorflow/versions.proto`

## File Metadata

- **Full Path**: `modules/dnn/src/tensorflow/versions.proto`
- **File Name**: `versions.proto`
- **File Size**: 961 bytes
- **File Type**: .proto
- **Link to Source**: [modules/dnn/src/tensorflow/versions.proto](../../../../modules/dnn/src/tensorflow/versions.proto)

## Purpose and Role

This file is located in the `modules/dnn/src/tensorflow` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
syntax = "proto3";

package opencv_tensorflow;
option cc_enable_arenas = true;
option java_outer_classname = "VersionsProtos";
option java_multiple_files = true;
option java_package = "org.tensorflow.framework";

// Version information for a piece of serialized data
//
// There are different types of versions for each type of data
// (GraphDef, etc.), but they all have the same common shape
// described here.
//
// Each consumer has "consumer" and "min_producer" versions (specified
// elsewhere).  A consumer is allowed to consume this data if
//
//   producer >= min_producer
//   consumer >= min_consumer
//   consumer not in bad_consumers
//
message VersionDef {
  // The version of the code that produced this data.
  int32 producer = 1;

  // Any consumer below this version is not allowed to consume this data.
  int32 min_consumer = 2;

  // Specific consumer versions which are disallowed (e.g. due to bugs).
  repeated int32 bad_consumers = 3;
};

```

## General Information

This file is part of the OpenCV repository infrastructure.

