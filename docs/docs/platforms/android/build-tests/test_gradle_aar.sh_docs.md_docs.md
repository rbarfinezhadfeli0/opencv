# Documentation for `docs/platforms/android/build-tests/test_gradle_aar.sh_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/android/build-tests/test_gradle_aar.sh_docs.md`
- **File Name**: `test_gradle_aar.sh_docs.md`
- **File Size**: 2,274 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/android/build-tests/test_gradle_aar.sh_docs.md](../../../../docs/platforms/android/build-tests/test_gradle_aar.sh_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/android/build-tests` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/android/build-tests/test_gradle_aar.sh`

## File Metadata

- **Full Path**: `platforms/android/build-tests/test_gradle_aar.sh`
- **File Name**: `test_gradle_aar.sh`
- **File Size**: 1,635 bytes
- **File Type**: .sh
- **Link to Source**: [platforms/android/build-tests/test_gradle_aar.sh](../../../platforms/android/build-tests/test_gradle_aar.sh)

## Purpose and Role

This file is located in the `platforms/android/build-tests` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/bash -e
SDK_DIR=$1
LOCAL_MAVEN_REPO=$2
echo "OpenCV Android SDK path: ${SDK_DIR}"
echo "Use local maven repo from $LOCAL_MAVEN_REPO"

ANDROID_HOME=${ANDROID_HOME:-${ANDROID_SDK_ROOT:-${ANDROID_SDK?Required ANDROID_HOME/ANDROID_SDK/ANDROID_SDK_ROOT}}}
ANDROID_NDK=${ANDROID_NDK_HOME-${ANDROID_NDK:-${NDKROOT?Required ANDROID_NDK_HOME/ANDROID_NDK/NDKROOT}}}
OPENCV_GRADLE_VERBOSE_OPTIONS=${OPENCV_GRADLE_VERBOSE_OPTIONS:-'-i'}

echo "Android SDK: ${ANDROID_HOME}"
echo "Android NDK: ${ANDROID_NDK}"

if [ ! -d "${ANDROID_HOME}" ]; then
  echo "FATAL: Missing Android SDK directory"
  exit 1
fi
if [ ! -d "${ANDROID_NDK}" ]; then
  echo "FATAL: Missing Android NDK directory"
  exit 1
fi

export ANDROID_HOME=${ANDROID_HOME}
export ANDROID_SDK=${ANDROID_HOME}
export ANDROID_SDK_ROOT=${ANDROID_HOME}

export ANDROID_NDK=${ANDROID_NDK}
export ANDROID_NDK_HOME=${ANDROID_NDK}

echo "Cloning OpenCV Android SDK ..."
rm -rf "test-gradle-aar"
mkdir test-gradle-aar
cp -rp ${SDK_DIR}/samples/* test-gradle-aar/
echo "Cloning OpenCV Android SDK ... Done!"

# drop cmake bin name and "bin" folder from path
echo "ndk.dir=${ANDROID_NDK}" > "test-gradle-aar/local.properties"
echo "cmake.dir=$(dirname $(dirname $(which cmake)))" >> "test-gradle-aar/local.properties"

sed -i "s/opencv_source = 'sdk_path'/opencv_source = 'maven_local'/g" test-gradle-aar/settings.gradle
sed -i "s+opencv_maven_path = '<path_to_maven_repo>'+opencv_maven_path = 'file\\://$LOCAL_MAVEN_REPO'+g" test-gradle-aar/settings.gradle

echo "Run gradle ..."
(cd "test-gradle-aar"; ./gradlew ${OPENCV_GRADLE_VERBOSE_OPTIONS} assemble)

echo "#"
echo "# Done!"
echo "#"

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

