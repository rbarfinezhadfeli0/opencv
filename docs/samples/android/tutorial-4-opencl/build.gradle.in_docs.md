# Documentation for `samples/android/tutorial-4-opencl/build.gradle.in`

## File Metadata

- **Full Path**: `samples/android/tutorial-4-opencl/build.gradle.in`
- **File Name**: `build.gradle.in`
- **File Size**: 2,356 bytes
- **File Type**: .in
- **Link to Source**: [samples/android/tutorial-4-opencl/build.gradle.in](../../../samples/android/tutorial-4-opencl/build.gradle.in)

## Purpose and Role

This file is located in the `samples/android/tutorial-4-opencl` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
apply plugin: 'com.android.application'

android {
    namespace 'org.opencv.samples.tutorial4'
    compileSdkVersion @ANDROID_COMPILE_SDK_VERSION@
    defaultConfig {
        applicationId "org.opencv.samples.tutorial4"
        minSdkVersion @ANDROID_MIN_SDK_VERSION@
        targetSdkVersion @ANDROID_TARGET_SDK_VERSION@
        versionCode 301
        versionName "3.01"

        externalNativeBuild {
            cmake {
                if (gradle.opencv_source == "sdk_path") {
                    arguments "-DOpenCV_DIR=" + project(':opencv').projectDir + "/@ANDROID_PROJECT_JNI_PATH@",
                              "-DOPENCV_FROM_SDK=TRUE",
                              "-DANDROID_SUPPORT_FLEXIBLE_PAGE_SIZES=ON",
                              "-DANDROID_OPENCL_SDK=@ANDROID_OPENCL_SDK@" @OPENCV_ANDROID_CMAKE_EXTRA_ARGS@

                } else {
                    arguments "-DOPENCV_VERSION_MAJOR=@OPENCV_VERSION_MAJOR@",
                              "-DOPENCV_FROM_SDK=FALSE",
                              "-DANDROID_SUPPORT_FLEXIBLE_PAGE_SIZES=ON",
                              "-DANDROID_OPENCL_SDK=@ANDROID_OPENCL_SDK@" @OPENCV_ANDROID_CMAKE_EXTRA_ARGS@
                }
                targets "JNIpart"
            }
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    sourceSets {
        main {
            java.srcDirs = @ANDROID_SAMPLE_JAVA_PATH@
            res.srcDirs = @ANDROID_SAMPLE_RES_PATH@
            manifest.srcFile '@ANDROID_SAMPLE_MANIFEST_PATH@'
        }
    }
    externalNativeBuild {
        cmake {
             path '@ANDROID_SAMPLE_JNI_PATH@/CMakeLists.txt'
        }
    }
    buildFeatures {
        if (gradle.opencv_source == "maven_local" || gradle.opencv_source == "maven_central") {
            prefab true
        }
    }
}

dependencies {
    //implementation fileTree(dir: 'libs', include: ['*.jar'])
    if (gradle.opencv_source == "sdk_path") {
        println 'Using OpenCV from SDK'
        implementation project(':opencv')
    } else if (gradle.opencv_source == "maven_local" || gradle.opencv_source == "maven_central") {
        println 'Using OpenCV from Maven repo'
        implementation 'org.opencv:opencv:@OPENCV_VERSION_PLAIN@'
    }
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

