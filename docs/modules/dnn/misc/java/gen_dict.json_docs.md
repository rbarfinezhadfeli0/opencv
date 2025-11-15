# Documentation for `modules/dnn/misc/java/gen_dict.json`

## File Metadata

- **Full Path**: `modules/dnn/misc/java/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 2,327 bytes
- **File Type**: .json
- **Link to Source**: [modules/dnn/misc/java/gen_dict.json](../../../../modules/dnn/misc/java/gen_dict.json)

## Purpose and Role

This file is located in the `modules/dnn/misc/java` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "type_dict": {
        "MatShape": {
            "j_type": "MatOfInt",
            "jn_type": "long",
            "jni_type": "jlong",
            "jni_var": "MatShape %(n)s",
            "suffix": "J",
            "v_type": "Mat",
            "j_import": "org.opencv.core.MatOfInt"
        },
        "vector_MatShape": {
            "j_type": "List<MatOfInt>",
            "jn_type": "long",
            "jni_type": "jlong",
            "jni_var": "std::vector< MatShape > %(n)s",
            "suffix": "J",
            "v_type": "Mat",
            "j_import": "org.opencv.core.MatOfInt"
        },
        "vector_vector_MatShape": {
            "j_type": "List<List<MatOfInt>>",
            "jn_type": "long",
            "jni_type": "jlong",
            "jni_var": "std::vector< std::vector<MatShape> > %(n)s",
            "suffix": "J",
            "v_type": "vector_Mat",
            "j_import": "org.opencv.core.MatOfInt"
        },
        "vector_size_t": {
            "j_type": "MatOfDouble",
            "jn_type": "long",
            "jni_type": "jlong",
            "jni_var": "std::vector<size_t> %(n)s",
            "suffix": "J",
            "v_type": "Mat",
            "j_import": "org.opencv.core.MatOfDouble"
        },
        "vector_Ptr_Layer": {
            "j_type": "List<Layer>",
            "jn_type": "List<Layer>",
            "jni_type": "jobject",
            "jni_var": "std::vector< Ptr<cv::dnn::Layer> > %(n)s",
            "suffix": "Ljava_util_List",
            "v_type": "vector_Layer",
            "j_import": "org.opencv.dnn.Layer"
        },
        "vector_Target": {
            "j_type": "List<Integer>",
            "jn_type": "List<Integer>",
            "jni_type": "jobject",
            "jni_var": "std::vector< cv::dnn::Target > %(n)s",
            "suffix": "Ljava_util_List",
            "v_type": "vector_Target"
        },
        "LayerId": {
            "j_type": "DictValue",
            "jn_type": "long",
            "jn_args": [
                [
                    "__int64",
                    ".getNativeObjAddr()"
                ]

            ],
            "jni_name": "(*(*(Ptr<cv::dnn::DictValue>*)%(n)s_nativeObj))",
            "jni_type": "jlong",
            "suffix": "J",
            "j_import": "org.opencv.dnn.DictValue"
        }
    }
}

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

