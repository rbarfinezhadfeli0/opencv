# Documentation for `modules/videoio/misc/java/gen_dict.json`

## File Metadata

- **Full Path**: `modules/videoio/misc/java/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 1,447 bytes
- **File Type**: .json
- **Link to Source**: [modules/videoio/misc/java/gen_dict.json](../../../../modules/videoio/misc/java/gen_dict.json)

## Purpose and Role

This file is located in the `modules/videoio/misc/java` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "ManualFuncs" : {
        "IStreamReader" : {
            "IStreamReader" : {
                "j_code"   : [
                    "\n",
                    "/**",
                    " * Constructor of streaming callback object with abstract 'read' and 'seek' methods that should be implemented in Java code.<br>",
                    " * <b>NOTE</b>: Implemented callbacks should be called from the creation thread to avoid JNI performance degradation",
                    "*/",
                    "protected IStreamReader() { nativeObj = 0; }",
                    "\n"
                ],
                "jn_code": [],
                "cpp_code": []
            }
        }
    },
    "func_arg_fix" : {
        "read": { "buffer": {"ctype" : "byte[]"} }
    },
    "type_dict": {
        "Ptr_IStreamReader": {
            "j_type": "IStreamReader",
            "jn_type": "IStreamReader",
            "jni_name": "n_%(n)s",
            "jni_type": "jobject",
            "jni_var": "auto n_%(n)s = makePtr<JavaStreamReader>(env, source)",
            "j_import": "org.opencv.videoio.IStreamReader"
        },
        "vector_VideoCaptureAPIs": {
            "j_type": "List<Integer>",
            "jn_type": "List<Integer>",
            "jni_type": "jobject",
            "jni_var": "std::vector< cv::VideoCaptureAPIs > %(n)s",
            "suffix": "Ljava_util_List",
            "v_type": "vector_VideoCaptureAPIs"
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

