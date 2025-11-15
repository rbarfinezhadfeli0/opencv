# Documentation for `modules/objdetect/misc/java/gen_dict.json`

## File Metadata

- **Full Path**: `modules/objdetect/misc/java/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 3,268 bytes
- **File Type**: .json
- **Link to Source**: [modules/objdetect/misc/java/gen_dict.json](../../../../modules/objdetect/misc/java/gen_dict.json)

## Purpose and Role

This file is located in the `modules/objdetect/misc/java` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "ManualFuncs" : {
        "QRCodeEncoder" : {
            "QRCodeEncoder" : {
                "j_code"   : [
                    "\n",
                    "/** Generates QR code from input string.",
                    "@param encoded_info Input bytes to encode.",
                    "@param qrcode Generated QR code.",
                    "*/",
                    "public void encode(byte[] encoded_info, Mat qrcode) {",
                    "    encode_1(nativeObj, encoded_info, qrcode.nativeObj);",
                    "}",
                    "\n"
                ],
                "jn_code": [
                    "\n",
                    "private static native void encode_1(long nativeObj, byte[] encoded_info, long qrcode_nativeObj);",
                    "\n"
                ],
                "cpp_code": [
                    "//",
                    "//  void cv::QRCodeEncoder::encode(String encoded_info, Mat& qrcode)",
                    "//",
                    "\n",
                    "JNIEXPORT void JNICALL Java_org_opencv_objdetect_QRCodeEncoder_encode_11 (JNIEnv*, jclass, jlong, jbyteArray, jlong);",
                    "\n",
                    "JNIEXPORT void JNICALL Java_org_opencv_objdetect_QRCodeEncoder_encode_11",
                    "(JNIEnv* env, jclass , jlong self, jbyteArray encoded_info, jlong qrcode_nativeObj)",
                    "{",
                    "",
                    "    static const char method_name[] = \"objdetect::encode_11()\";",
                    "    try {",
                    "        LOGD(\"%s\", method_name);",
                    "        Ptr<cv::QRCodeEncoder>* me = (Ptr<cv::QRCodeEncoder>*) self; //TODO: check for NULL",
                    "        const char* n_encoded_info = reinterpret_cast<char*>(env->GetByteArrayElements(encoded_info, NULL));",
                    "        const jsize n_encoded_info_size = env->GetArrayLength(encoded_info);",
                    "        Mat& qrcode = *((Mat*)qrcode_nativeObj);",
                    "        (*me)->encode( std::string(n_encoded_info, n_encoded_info_size), qrcode );",
                    "    } catch(const std::exception &e) {",
                    "        throwJavaException(env, &e, method_name);",
                    "    } catch (...) {",
                    "        throwJavaException(env, 0, method_name);",
                    "    }",
                    "}",
                    "\n"
                ]
            }
        }
    },
    "type_dict": {
        "NativeByteArray": {
            "j_type" : "byte[]",
            "jn_type": "byte[]",
            "jni_type": "jbyteArray",
            "jni_name": "n_%(n)s",
            "jni_var": "jbyteArray n_%(n)s = env->NewByteArray(static_cast<jsize>(%(n)s.size())); env->SetByteArrayRegion(n_%(n)s, 0, static_cast<jsize>(%(n)s.size()), reinterpret_cast<const jbyte*>(%(n)s.c_str()));",
            "cast_from": "std::string"
        },
        "vector_NativeByteArray": {
            "j_type": "List<byte[]>",
            "jn_type": "List<byte[]>",
            "jni_type": "jobject",
            "jni_var": "std::vector< std::string > %(n)s",
            "suffix": "Ljava_util_List",
            "v_type": "vector_NativeByteArray"
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

