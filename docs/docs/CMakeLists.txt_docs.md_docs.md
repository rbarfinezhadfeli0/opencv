# Documentation for `docs/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 50,888 bytes
- **File Type**: .md
- **Link to Source**: [docs/CMakeLists.txt_docs.md](../docs/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `CMakeLists.txt`

## File Metadata

- **Full Path**: `CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 83,900 bytes
- **File Type**: .txt
- **Link to Source**: [CMakeLists.txt](CMakeLists.txt)

## Purpose and Role

This file is located in the `.` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# ----------------------------------------------------------------------------
#  Root CMake file for OpenCV
#
#    From the off-tree build directory, invoke:
#      $ cmake <PATH_TO_OPENCV_ROOT>
#
# ----------------------------------------------------------------------------
# Disable in-source builds to prevent source tree corruption.
if(" ${CMAKE_SOURCE_DIR}" STREQUAL " ${CMAKE_BINARY_DIR}")
  message(FATAL_ERROR "
FATAL: In-source builds are not allowed.
       You should create a separate directory for build files.
")
endif()

include(cmake/OpenCVMinDepVersions.cmake)

if(CMAKE_SYSTEM_NAME MATCHES WindowsPhone OR CMAKE_SYSTEM_NAME MATCHES WindowsStore)
  cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
  #Required to resolve linker error issues due to incompatibility with CMake v3.0+ policies.
  #CMake fails to find _fseeko() which leads to subsequent linker error.
  #See details here: http://www.cmake.org/Wiki/CMake/Policies
  cmake_policy(VERSION 2.8)
else()
  cmake_minimum_required(VERSION "${MIN_VER_CMAKE}" FATAL_ERROR)
endif()

#
# Configure CMake policies
#
if(POLICY CMP0026)
  cmake_policy(SET CMP0026 NEW)
endif()

if(POLICY CMP0042)
  cmake_policy(SET CMP0042 NEW)  # CMake 3.0+ (2.8.12): MacOS "@rpath" in target's install name
endif()

if(POLICY CMP0046)
  cmake_policy(SET CMP0046 NEW)  # warn about non-existed dependencies
endif()

if(POLICY CMP0051)
  cmake_policy(SET CMP0051 NEW)
endif()

if(POLICY CMP0054)  # CMake 3.1: Only interpret if() arguments as variables or keywords when unquoted.
  cmake_policy(SET CMP0054 NEW)
endif()

if(POLICY CMP0056)
  cmake_policy(SET CMP0056 NEW)  # try_compile(): link flags
endif()

if(POLICY CMP0057)
  cmake_policy(SET CMP0057 NEW)  # CMake 3.3: if(IN_LIST) support
endif()

if(POLICY CMP0066)
  cmake_policy(SET CMP0066 NEW)  # CMake 3.7: try_compile(): use per-config flags, like CMAKE_CXX_FLAGS_RELEASE
endif()

if(POLICY CMP0067)
  cmake_policy(SET CMP0067 NEW)  # CMake 3.8: try_compile(): honor language standard variables (like C++11)
endif()

if(POLICY CMP0068)
  cmake_policy(SET CMP0068 NEW)  # CMake 3.9+: `RPATH` settings on macOS do not affect `install_name`.
endif()

if(POLICY CMP0071)
  cmake_policy(SET CMP0071 NEW)  # CMake 3.10+: Let `AUTOMOC` and `AUTOUIC` process `GENERATED` files.
endif()

if(POLICY CMP0075)
  cmake_policy(SET CMP0075 NEW)  # CMake 3.12+: Include file check macros honor `CMAKE_REQUIRED_LIBRARIES`
endif()

if(POLICY CMP0077)
  cmake_policy(SET CMP0077 NEW)  # CMake 3.13+: option() honors normal variables.
endif()

if(POLICY CMP0091)
  cmake_policy(SET CMP0091 NEW) # CMake 3.15+: leave MSVC runtime selection out of default CMAKE_<LANG>_FLAGS_<CONFIG> flags
endif()

if(POLICY CMP0146)
  cmake_policy(SET CMP0146 OLD)  # CMake 3.27+: use CMake FindCUDA if available.
endif()

if(POLICY CMP0148)
  cmake_policy(SET CMP0148 OLD)  # CMake 3.27+: use CMake FindPythonInterp and FindPythonLib if available.
endif()

#
# Configure OpenCV CMake hooks
#
include(cmake/OpenCVUtils.cmake)
ocv_cmake_reset_hooks()
ocv_check_environment_variables(OPENCV_CMAKE_HOOKS_DIR)
if(DEFINED OPENCV_CMAKE_HOOKS_DIR)
  foreach(__dir ${OPENCV_CMAKE_HOOKS_DIR})
    get_filename_component(__dir "${__dir}" ABSOLUTE)
    ocv_cmake_hook_register_dir(${__dir})
  endforeach()
endif()

ocv_cmake_hook(CMAKE_INIT)

# must go before the project()/enable_language() commands
ocv_update(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "Configs" FORCE)
if(NOT DEFINED CMAKE_BUILD_TYPE
    AND NOT OPENCV_SKIP_DEFAULT_BUILD_TYPE
)
  message(STATUS "'Release' build type is used by default. Use CMAKE_BUILD_TYPE to specify build type (Release or Debug)")
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build")
endif()
if(DEFINED CMAKE_BUILD_TYPE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "${CMAKE_CONFIGURATION_TYPES}")
endif()

option(ENABLE_PIC "Generate position independent code (necessary for shared libraries)" TRUE)
set(CMAKE_POSITION_INDEPENDENT_CODE ${ENABLE_PIC})

ocv_cmake_hook(PRE_CMAKE_BOOTSTRAP)

# Bootstrap CMake system: setup CMAKE_SYSTEM_NAME and other vars

# workaround: https://gitlab.kitware.com/cmake/cmake/-/issues/20989
if(OPENCV_WORKAROUND_CMAKE_20989)
  set(CMAKE_SYSTEM_PROCESSOR_BACKUP ${CMAKE_SYSTEM_PROCESSOR})
endif()

project(OpenCV CXX C)

if(OPENCV_WORKAROUND_CMAKE_20989)
  set(CMAKE_SYSTEM_PROCESSOR ${CMAKE_SYSTEM_PROCESSOR_BACKUP})
endif()

enable_testing()

ocv_cmake_hook(POST_CMAKE_BOOTSTRAP)

if(NOT OPENCV_SKIP_CMAKE_SYSTEM_FILE)
  include("cmake/platforms/OpenCV-${CMAKE_SYSTEM_NAME}.cmake" OPTIONAL RESULT_VARIABLE "OPENCV_CMAKE_SYSTEM_FILE")
  if(NOT OPENCV_CMAKE_SYSTEM_FILE)
    message(STATUS "OpenCV: system-specific configuration file is not found: '${CMAKE_SYSTEM_NAME}'")
  endif()
endif()

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)  # https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT.html
  if(NOT CMAKE_CROSSCOMPILING)
    if(WIN32)
      set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE PATH "Installation Directory" FORCE)
    else()
      set(CMAKE_INSTALL_PREFIX "/usr/local" CACHE PATH "Installation Directory" FORCE)
    endif()
  else()
    # any cross-compiling
    set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE PATH "Installation Directory" FORCE)
  endif()
endif()

if(MSVC)
  set(CMAKE_USE_RELATIVE_PATHS ON CACHE INTERNAL "" FORCE)
endif()

ocv_cmake_eval(DEBUG_PRE ONCE)

ocv_clear_vars(OpenCVModules_TARGETS)

# ----------------------------------------------------------------------------
#  Autodetect if we are in a GIT repository
# ----------------------------------------------------------------------------
find_host_package(Git QUIET)

if(NOT DEFINED OPENCV_VCSVERSION AND GIT_FOUND)
  ocv_git_describe(OPENCV_VCSVERSION "${OpenCV_SOURCE_DIR}")
elseif(NOT DEFINED OPENCV_VCSVERSION)
  # We don't have git:
  set(OPENCV_VCSVERSION "unknown")
endif()

include(cmake/OpenCVDownload.cmake)

# ----------------------------------------------------------------------------
# Detect compiler and target platform architecture
# ----------------------------------------------------------------------------
include(cmake/OpenCVDetectCXXCompiler.cmake)
ocv_cmake_hook(POST_DETECT_COMPILER)

# ----------------------------------------------------------------------------
# OpenCV cmake options
# ----------------------------------------------------------------------------
set(BUILD_LIST "" CACHE STRING "Build only listed modules (comma-separated, e.g. 'videoio,dnn,ts')")
OCV_OPTION(OPENCV_ENABLE_NONFREE "Enable non-free algorithms" OFF)

# 3rd party libs
OCV_OPTION(OPENCV_FORCE_3RDPARTY_BUILD   "Force using 3rdparty code from source" OFF)
OCV_OPTION(BUILD_ZLIB               "Build zlib from source"             (WIN32 OR APPLE OR OPENCV_FORCE_3RDPARTY_BUILD) )
OCV_OPTION(BUILD_TIFF               "Build libtiff from source"          (WIN32 OR ANDROID OR APPLE OR OPENCV_FORCE_3RDPARTY_BUILD) )
OCV_OPTION(BUILD_OPENJPEG           "Build OpenJPEG from source"         (WIN32 OR ANDROID OR APPLE OR OPENCV_FORCE_3RDPARTY_BUILD) )
OCV_OPTION(BUILD_JASPER             "Build libjasper from source"        (WIN32 OR ANDROID OR APPLE OR OPENCV_FORCE_3RDPARTY_BUILD) )
OCV_OPTION(BUILD_JPEG               "Build libjpeg from source"          (WIN32 OR ANDROID OR APPLE OR OPENCV_FORCE_3RDPARTY_BUILD) )
OCV_OPTION(BUILD_PNG                "Build libpng from source"           (WIN32 OR ANDROID OR APPLE OR OPENCV_FORCE_3RDPARTY_BUILD) )
OCV_OPTION(BUILD_OPENEXR            "Build openexr from source"          (OPENCV_FORCE_3RDPARTY_BUILD) )
OCV_OPTION(BUILD_WEBP               "Build WebP from source"             (((WIN32 OR ANDROID OR APPLE) AND NOT WINRT) OR OPENCV_FORCE_3RDPARTY_BUILD) )
OCV_OPTION(BUILD_TBB                "Download and build TBB from source" (ANDROID OR OPENCV_FORCE_3RDPARTY_BUILD) )
OCV_OPTION(BUILD_IPP_IW             "Build IPP IW from source"           (NOT MINGW OR OPENCV_FORCE_3RDPARTY_BUILD) IF (X86_64 OR X86) AND NOT WINRT )
OCV_OPTION(BUILD_ITT                "Build Intel ITT from source"
    (NOT MINGW OR OPENCV_FORCE_3RDPARTY_BUILD)
    IF (X86_64 OR X86 OR ARM OR AARCH64 OR PPC64 OR PPC64LE) AND NOT WINRT AND NOT APPLE_FRAMEWORK
)

# Optional 3rd party components
# ===================================================
OCV_OPTION(WITH_1394 "Include IEEE1394 support" OFF
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_DC1394_2)
OCV_OPTION(WITH_AVFOUNDATION "Use AVFoundation for Video I/O (iOS/visionOS/Mac)" ON
  VISIBLE_IF APPLE
  VERIFY HAVE_AVFOUNDATION)
OCV_OPTION(WITH_AVIF "Enable AVIF support" ON
  VERIFY HAVE_AVIF)
OCV_OPTION(WITH_CAP_IOS "Enable iOS video capture" ON
  VISIBLE_IF IOS
  VERIFY HAVE_CAP_IOS)
OCV_OPTION(WITH_CAROTENE "Use NVidia carotene acceleration library for ARM platform" (NOT CV_DISABLE_OPTIMIZATION)
  VISIBLE_IF (ARM OR AARCH64) AND NOT IOS AND NOT XROS)
OCV_OPTION(WITH_KLEIDICV "Use KleidiCV library for ARM platforms" (ANDROID AND AARCH64 AND NOT CV_DISABLE_OPTIMIZATION)
  VISIBLE_IF (AARCH64 AND (ANDROID OR UNIX)))
OCV_OPTION(WITH_NDSRVP "Use Andes RVP extension" (NOT CV_DISABLE_OPTIMIZATION)
  VISIBLE_IF RISCV)
OCV_OPTION(WITH_HAL_RVV "Use HAL RVV optimizations" (NOT CV_DISABLE_OPTIMIZATION)
  VISIBLE_IF RISCV)
OCV_OPTION(WITH_FASTCV "Use Qualcomm FastCV acceleration library for ARM platform" OFF
  VISIBLE_IF ((ARM OR AARCH64) AND (ANDROID OR (UNIX AND NOT APPLE AND NOT IOS AND NOT XROS))))
OCV_OPTION(WITH_CPUFEATURES "Use cpufeatures Android library" ON
  VISIBLE_IF ANDROID
  VERIFY HAVE_CPUFEATURES)
OCV_OPTION(WITH_VTK "Include VTK library support (and build opencv_viz module eiher)" ON
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT AND NOT CMAKE_CROSSCOMPILING
  VERIFY HAVE_VTK)
OCV_OPTION(WITH_CUDA "Include NVidia Cuda Runtime support" OFF
  VISIBLE_IF NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_CUDA)
OCV_OPTION(WITH_CUFFT "Include NVidia Cuda Fast Fourier Transform (FFT) library support" WITH_CUDA
  VISIBLE_IF WITH_CUDA
  VERIFY HAVE_CUFFT)
OCV_OPTION(WITH_CUBLAS "Include NVidia Cuda Basic Linear Algebra Subprograms (BLAS) library support" WITH_CUDA
  VISIBLE_IF WITH_CUDA
  VERIFY HAVE_CUBLAS)
OCV_OPTION(WITH_CUDNN "Include NVIDIA CUDA Deep Neural Network (cuDNN) library support" WITH_CUDA
  VISIBLE_IF WITH_CUDA
  VERIFY HAVE_CUDNN)
OCV_OPTION(WITH_NVCUVID "Include NVidia Video Decoding library support" ON
  VISIBLE_IF WITH_CUDA
  VERIFY HAVE_NVCUVID)
OCV_OPTION(WITH_NVCUVENC "Include NVidia Video Encoding library support" ON
  VISIBLE_IF WITH_CUDA
  VERIFY HAVE_NVCUVENC)
OCV_OPTION(WITH_EIGEN "Include Eigen2/Eigen3 support" (NOT CV_DISABLE_OPTIMIZATION AND NOT CMAKE_CROSSCOMPILING)
  VISIBLE_IF NOT WINRT
  VERIFY HAVE_EIGEN)
OCV_OPTION(WITH_FFMPEG "Include FFMPEG support" (NOT ANDROID)
  VISIBLE_IF NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_FFMPEG)
OCV_OPTION(WITH_GSTREAMER "Include Gstreamer support" ON
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_GSTREAMER AND GSTREAMER_VERSION VERSION_GREATER "0.99")
OCV_OPTION(WITH_GTK "Include GTK support" ON
  VISIBLE_IF UNIX AND NOT APPLE AND NOT ANDROID
  VERIFY HAVE_GTK)
OCV_OPTION(WITH_GTK_2_X "Use GTK version 2" OFF
  VISIBLE_IF UNIX AND NOT APPLE AND NOT ANDROID
  VERIFY HAVE_GTK AND NOT HAVE_GTK3)
OCV_OPTION(WITH_FRAMEBUFFER "Include framebuffer support" OFF
  VISIBLE_IF UNIX AND NOT APPLE AND NOT ANDROID)
OCV_OPTION(WITH_FRAMEBUFFER_XVFB "Include virtual framebuffer support" OFF
  VISIBLE_IF UNIX AND NOT APPLE AND NOT ANDROID)
OCV_OPTION(WITH_WAYLAND "Include Wayland support" OFF
        VISIBLE_IF UNIX AND NOT APPLE AND NOT ANDROID
        VERIFY HAVE_WAYLAND)
OCV_OPTION(WITH_IPP "Include Intel IPP support" (NOT MINGW AND NOT CV_DISABLE_OPTIMIZATION)
  VISIBLE_IF (X86_64 OR X86) AND NOT WINRT AND NOT IOS AND NOT XROS
  VERIFY HAVE_IPP)
OCV_OPTION(WITH_HALIDE "Include Halide support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_HALIDE)
OCV_OPTION(WITH_VULKAN "Include Vulkan support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_VULKAN)
# replacement for deprecated options: WITH_INF_ENGINE, WITH_NGRAPH
OCV_OPTION(WITH_OPENVINO "Include Intel OpenVINO toolkit support" (WITH_INF_ENGINE)
  VISIBLE_IF TRUE
  VERIFY TARGET ocv.3rdparty.openvino)
OCV_OPTION(WITH_WEBNN "Include WebNN support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_WEBNN)
OCV_OPTION(WITH_JASPER "Include JPEG2K support (Jasper)" ON
  VISIBLE_IF NOT IOS AND NOT XROS
  VERIFY HAVE_JASPER)
OCV_OPTION(WITH_OPENJPEG "Include JPEG2K support (OpenJPEG)" ON
  VISIBLE_IF NOT IOS AND NOT XROS
  VERIFY HAVE_OPENJPEG)
OCV_OPTION(WITH_JPEG "Include JPEG support" ON
  VISIBLE_IF TRUE
  VERIFY HAVE_JPEG)
OCV_OPTION(WITH_JPEGXL "Include JPEG XL support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_JPEGXL)
OCV_OPTION(WITH_WEBP "Include WebP support" ON
  VISIBLE_IF NOT WINRT
  VERIFY HAVE_WEBP)
OCV_OPTION(WITH_OPENEXR "Include ILM support via OpenEXR" ((WIN32 OR ANDROID OR APPLE) OR BUILD_OPENEXR) OR NOT CMAKE_CROSSCOMPILING
  VISIBLE_IF NOT APPLE_FRAMEWORK AND NOT WINRT
  VERIFY HAVE_OPENEXR)
OCV_OPTION(WITH_OPENGL "Include OpenGL support" OFF
  VISIBLE_IF NOT ANDROID AND NOT WINRT
  VERIFY HAVE_OPENGL)
OCV_OPTION(WITH_OPENVX "Include OpenVX support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_OPENVX)
OCV_OPTION(WITH_OPENNI "Include OpenNI support" OFF
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_OPENNI)
OCV_OPTION(WITH_OPENNI2 "Include OpenNI2 support" OFF
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_OPENNI2)
OCV_OPTION(WITH_PNG "Include PNG support" ON
  VISIBLE_IF TRUE
  VERIFY HAVE_PNG)
OCV_OPTION(WITH_SPNG "Include SPNG support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_SPNG)
OCV_OPTION(WITH_GDCM "Include DICOM support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_GDCM)
OCV_OPTION(WITH_PVAPI "Include Prosilica GigE support" OFF
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_PVAPI)
OCV_OPTION(WITH_ARAVIS "Include Aravis GigE support" OFF
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT AND NOT WIN32
  VERIFY HAVE_ARAVIS_API)
OCV_OPTION(WITH_QT "Build with Qt Backend support" OFF
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_QT)
OCV_OPTION(WITH_WIN32UI "Build with Win32 UI Backend support" ON
  VISIBLE_IF WIN32 AND NOT WINRT
  VERIFY HAVE_WIN32UI)
OCV_OPTION(WITH_TBB "Include Intel TBB support" OFF
  VISIBLE_IF NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_TBB)
OCV_OPTION(WITH_HPX "Include Ste||ar Group HPX support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_HPX)
OCV_OPTION(WITH_OPENMP "Include OpenMP support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_OPENMP)
OCV_OPTION(WITH_PTHREADS_PF "Use pthreads-based parallel_for" ON
  VISIBLE_IF NOT WIN32 OR MINGW
  VERIFY HAVE_PTHREADS_PF)
OCV_OPTION(WITH_TIFF "Include TIFF support" ON
  VISIBLE_IF NOT IOS AND NOT XROS
  VERIFY HAVE_TIFF)
OCV_OPTION(WITH_V4L "Include Video 4 Linux support" ON
  VISIBLE_IF UNIX AND NOT ANDROID AND NOT APPLE
  VERIFY HAVE_CAMV4L OR HAVE_CAMV4L2 OR HAVE_VIDEOIO)
OCV_OPTION(WITH_DSHOW "Build VideoIO with DirectShow support" ON
  VISIBLE_IF WIN32 AND NOT ARM AND NOT WINRT
  VERIFY HAVE_DSHOW)
OCV_OPTION(WITH_MSMF "Build VideoIO with Media Foundation support" NOT MINGW
  VISIBLE_IF WIN32
  VERIFY HAVE_MSMF)
OCV_OPTION(WITH_MSMF_DXVA "Enable hardware acceleration in Media Foundation backend" WITH_MSMF
  VISIBLE_IF WIN32
  VERIFY HAVE_MSMF_DXVA)
OCV_OPTION(WITH_XIMEA "Include XIMEA cameras support" OFF
  VISIBLE_IF NOT ANDROID AND NOT WINRT
  VERIFY HAVE_XIMEA)
OCV_OPTION(WITH_UEYE "Include UEYE camera support" OFF
  VISIBLE_IF NOT ANDROID AND NOT APPLE AND NOT WINRT
  VERIFY HAVE_UEYE)
OCV_OPTION(WITH_XINE "Include Xine support (GPL)" OFF
  VISIBLE_IF UNIX AND NOT APPLE AND NOT ANDROID
  VERIFY HAVE_XINE)
OCV_OPTION(WITH_CLP "Include Clp support (EPL)" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_CLP)
OCV_OPTION(WITH_OPENCL "Include OpenCL Runtime support" (NOT ANDROID AND NOT CV_DISABLE_OPTIMIZATION)
  VISIBLE_IF NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_OPENCL)
OCV_OPTION(WITH_OPENCL_SVM "Include OpenCL Shared Virtual Memory support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_OPENCL_SVM) # experimental
OCV_OPTION(WITH_OPENCLAMDFFT "Include AMD OpenCL FFT library support" ON
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_CLAMDFFT)
OCV_OPTION(WITH_OPENCLAMDBLAS "Include AMD OpenCL BLAS library support" ON
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_CLAMDBLAS)
OCV_OPTION(WITH_DIRECTX "Include DirectX support" ON
  VISIBLE_IF WIN32 AND NOT WINRT
  VERIFY HAVE_DIRECTX)
OCV_OPTION(WITH_DIRECTML "Include DirectML support" ON
  VISIBLE_IF WIN32 AND NOT WINRT
  VERIFY HAVE_DIRECTML)
OCV_OPTION(WITH_OPENCL_D3D11_NV "Include NVIDIA OpenCL D3D11 support" WITH_DIRECTX
  VISIBLE_IF WIN32 AND NOT WINRT
  VERIFY HAVE_OPENCL_D3D11_NV)
OCV_OPTION(WITH_LIBREALSENSE "Include Intel librealsense support" OFF
  VISIBLE_IF NOT WITH_INTELPERC
  VERIFY HAVE_LIBREALSENSE)
OCV_OPTION(WITH_VA "Include VA support" (X86_64 OR X86)
  VISIBLE_IF UNIX AND NOT APPLE AND NOT ANDROID
  VERIFY HAVE_VA)
OCV_OPTION(WITH_VA_INTEL "Include Intel VA-API/OpenCL support" (X86_64 OR X86)
  VISIBLE_IF UNIX AND NOT APPLE AND NOT ANDROID
  VERIFY HAVE_VA_INTEL)
OCV_OPTION(WITH_MFX "Include Intel Media SDK support" OFF
  VISIBLE_IF (UNIX AND NOT ANDROID) OR (WIN32 AND NOT WINRT AND NOT MINGW)
  VERIFY HAVE_MFX)
OCV_OPTION(WITH_GDAL "Include GDAL Support" OFF
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS AND NOT WINRT
  VERIFY HAVE_GDAL)
OCV_OPTION(WITH_GPHOTO2 "Include gPhoto2 library support" OFF
  VISIBLE_IF UNIX AND NOT ANDROID AND NOT IOS AND NOT XROS
  VERIFY HAVE_GPHOTO2)
OCV_OPTION(WITH_LAPACK "Include Lapack library support" (NOT CV_DISABLE_OPTIMIZATION)
  VISIBLE_IF NOT ANDROID AND NOT IOS AND NOT XROS
  VERIFY HAVE_LAPACK)
OCV_OPTION(WITH_ITT "Include Intel ITT support" ON
  VISIBLE_IF NOT APPLE_FRAMEWORK
  VERIFY HAVE_ITT)
OCV_OPTION(WITH_PROTOBUF "Enable libprotobuf" ON
  VISIBLE_IF TRUE
  VERIFY HAVE_PROTOBUF)
OCV_OPTION(WITH_IMGCODEC_GIF "Include GIF support" ON
  VISIBLE_IF TRUE
  VERIFY HAVE_IMGCODEC_GIF)
OCV_OPTION(WITH_IMGCODEC_HDR "Include HDR support" ON
  VISIBLE_IF TRUE
  VERIFY HAVE_IMGCODEC_HDR)
OCV_OPTION(WITH_IMGCODEC_SUNRASTER "Include SUNRASTER support" ON
  VISIBLE_IF TRUE
  VERIFY HAVE_IMGCODEC_SUNRASTER)
OCV_OPTION(WITH_IMGCODEC_PXM "Include PNM (PBM,PGM,PPM) and PAM formats support" ON
  VISIBLE_IF TRUE
  VERIFY HAVE_IMGCODEC_PXM)
OCV_OPTION(WITH_IMGCODEC_PFM "Include PFM formats support" ON
  VISIBLE_IF TRUE
  VERIFY HAVE_IMGCODEC_PFM)
OCV_OPTION(WITH_QUIRC "Include library QR-code decoding" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_QUIRC)
OCV_OPTION(WITH_ANDROID_MEDIANDK "Use Android Media NDK for Video I/O (Android)" (ANDROID_NATIVE_API_LEVEL GREATER 20)
  VISIBLE_IF ANDROID
  VERIFY HAVE_ANDROID_MEDIANDK)
OCV_OPTION(WITH_ANDROID_NATIVE_CAMERA "Use Android NDK for Camera I/O (Android)" (ANDROID_NATIVE_API_LEVEL GREATER 23)
  VISIBLE_IF ANDROID
  VERIFY HAVE_ANDROID_NATIVE_CAMERA)
OCV_OPTION(WITH_ONNX "Include Microsoft ONNX Runtime support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_ONNX)
OCV_OPTION(WITH_TIMVX "Include Tim-VX support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_TIMVX)
# Attention when OBSENSOR_USE_ORBBEC_SDK set to off:
#   Astra2 cameras currently only support Windows and Linux kernel versions no higher than 4.15, and higher versions of Linux kernel may have exceptions.
OCV_OPTION(OBSENSOR_USE_ORBBEC_SDK "Use Orbbec SDK as backend to support more camera models and platforms (force to ON on MacOS)" OFF)
OCV_OPTION(WITH_OBSENSOR "Include obsensor support (Orbbec 3D Cameras)" ON
  VISIBLE_IF (WIN32 AND NOT ARM AND NOT WINRT AND NOT MINGW) OR ( UNIX AND NOT APPLE AND NOT ANDROID) OR (APPLE AND AARCH64 AND NOT IOS)
  VERIFY HAVE_OBSENSOR)
OCV_OPTION(WITH_CANN "Include CANN support" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_CANN)
OCV_OPTION(WITH_FLATBUFFERS "Include Flatbuffers support (required by DNN/TFLite importer)" ON
  VISIBLE_IF TRUE
  VERIFY HAVE_FLATBUFFERS)
OCV_OPTION(WITH_ZLIB_NG "Use zlib-ng instead of zlib" OFF
  VISIBLE_IF TRUE
  VERIFY HAVE_ZLIB_NG)

# OpenCV build components
# ===================================================
OCV_OPTION(BUILD_SHARED_LIBS        "Build shared libraries (.dll/.so) instead of static ones (.lib/.a)" NOT (ANDROID OR APPLE_FRAMEWORK) )
OCV_OPTION(BUILD_opencv_apps        "Build utility applications (used for example to train classifiers)" (NOT ANDROID AND NOT WINRT) IF (NOT APPLE_FRAMEWORK) )
OCV_OPTION(BUILD_opencv_js          "Build JavaScript bindings by Emscripten" OFF )
OCV_OPTION(BUILD_ANDROID_PROJECTS   "Build Android projects providing .apk files" ON  IF ANDROID )
OCV_OPTION(BUILD_ANDROID_EXAMPLES   "Build examples for Android platform"         ON  IF ANDROID )
OCV_OPTION(BUILD_DOCS               "Create build rules for OpenCV Documentation" OFF  IF (NOT WINRT AND NOT APPLE_FRAMEWORK))
OCV_OPTION(BUILD_EXAMPLES           "Build all examples"                          OFF )
OCV_OPTION(BUILD_PACKAGE            "Enables 'make package_source' command"       ON  IF NOT WINRT)
OCV_OPTION(BUILD_PERF_TESTS         "Build performance tests"                     NOT INSTALL_CREATE_DISTRIB  IF (NOT APPLE_FRAMEWORK) )
OCV_OPTION(BUILD_TESTS              "Build accuracy & regression tests"           NOT INSTALL_CREATE_DISTRIB  IF (NOT APPLE_FRAMEWORK) )
OCV_OPTION(BUILD_WITH_DEBUG_INFO    "Include debug info into release binaries ('OFF' means default settings)" OFF )
OCV_OPTION(BUILD_WITH_STATIC_CRT    "Enables use of statically linked CRT for statically linked OpenCV" ON IF MSVC )
OCV_OPTION(BUILD_WITH_DYNAMIC_IPP   "Enables dynamic linking of IPP (only for standalone IPP)" OFF )
OCV_OPTION(BUILD_FAT_JAVA_LIB       "Create Java wrapper exporting all functions of OpenCV library (requires static build of OpenCV modules)" ANDROID IF NOT BUILD_SHARED_LIBS)
OCV_OPTION(BUILD_ANDROID_SERVICE    "Build OpenCV Manager for Google Play" OFF IF ANDROID )
OCV_OPTION(BUILD_CUDA_STUBS         "Build CUDA modules stubs when no CUDA SDK" OFF  IF (NOT APPLE_FRAMEWORK) )
OCV_OPTION(BUILD_JAVA               "Enable Java support"                         (ANDROID OR NOT CMAKE_CROSSCOMPILING)  IF (ANDROID OR (NOT APPLE_FRAMEWORK AND NOT WINRT)) )
OCV_OPTION(BUILD_OBJC               "Enable Objective-C support"                  ON  IF APPLE_FRAMEWORK )
OCV_OPTION(BUILD_KOTLIN_EXTENSIONS  "Build Kotlin extensions (Android)"           ON  IF ANDROID )

# OpenCV installation options
# ===================================================
OCV_OPTION(INSTALL_CREATE_DISTRIB   "Change install rules to build the distribution package" OFF )
OCV_OPTION(INSTALL_BIN_EXAMPLES     "Install prebuilt examples" WIN32 IF BUILD_EXAMPLES)
OCV_OPTION(INSTALL_C_EXAMPLES       "Install C examples"        OFF )
OCV_OPTION(INSTALL_PYTHON_EXAMPLES  "Install Python examples"   OFF )
OCV_OPTION(INSTALL_ANDROID_EXAMPLES "Install Android examples"  OFF IF ANDROID )
OCV_OPTION(INSTALL_TO_MANGLED_PATHS "Enables mangled install paths, that help with side by side installs." OFF IF (UNIX AND NOT ANDROID AND NOT APPLE_FRAMEWORK AND BUILD_SHARED_LIBS) )
OCV_OPTION(INSTALL_TESTS            "Install accuracy and performance test binaries and test data" OFF)

# OpenCV build options
# ===================================================
OCV_OPTION(ENABLE_CCACHE              "Use ccache"                                               (UNIX AND (CMAKE_GENERATOR MATCHES "Makefile" OR CMAKE_GENERATOR MATCHES "Ninja" OR CMAKE_GENERATOR MATCHES "Xcode")) )
OCV_OPTION(ENABLE_PRECOMPILED_HEADERS "Use precompiled headers"                                  MSVC IF (MSVC OR (NOT IOS AND NOT XROS AND NOT CMAKE_CROSSCOMPILING) ) )
OCV_OPTION(ENABLE_DELAYLOAD           "Enable delayed loading of OpenCV DLLs"                    OFF VISIBLE_IF MSVC AND BUILD_SHARED_LIBS)
OCV_OPTION(ENABLE_SOLUTION_FOLDERS    "Solution folder in Visual Studio or in other IDEs"        (MSVC_IDE OR CMAKE_GENERATOR MATCHES Xcode) )
OCV_OPTION(ENABLE_PROFILING           "Enable profiling in the GCC compiler (Add flags: -g -pg)" OFF  IF CV_GCC )
OCV_OPTION(ENABLE_COVERAGE            "Enable coverage collection with  GCov"                    OFF  IF CV_GCC )
OCV_OPTION(OPENCV_ENABLE_MEMORY_SANITIZER "Better support for memory/address sanitizers"         OFF)
OCV_OPTION(ENABLE_OMIT_FRAME_POINTER  "Enable -fomit-frame-pointer for GCC"                      ON   IF CV_GCC )
OCV_OPTION(ENABLE_POWERPC             "Enable PowerPC for GCC"                                   ON   IF (CV_GCC AND CMAKE_SYSTEM_PROCESSOR MATCHES powerpc.*) )
OCV_OPTION(ENABLE_FAST_MATH           "Enable compiler options for fast math optimizations on FP computations (not recommended)" OFF)
OCV_OPTION(ENABLE_NOISY_WARNINGS      "Show all warnings even if they are too noisy"             OFF )
OCV_OPTION(OPENCV_WARNINGS_ARE_ERRORS "Treat warnings as errors"                                 OFF )
OCV_OPTION(ANDROID_EXAMPLES_WITH_LIBS "Build binaries of Android examples with native libraries" OFF  IF ANDROID )
OCV_OPTION(ENABLE_IMPL_COLLECTION     "Collect implementation data on function call"             OFF )
OCV_OPTION(ENABLE_INSTRUMENTATION     "Instrument functions to collect calls trace and performance" OFF )
OCV_OPTION(ENABLE_GNU_STL_DEBUG       "Enable GNU STL Debug mode (defines _GLIBCXX_DEBUG)"       OFF IF CV_GCC )
OCV_OPTION(ENABLE_BUILD_HARDENING     "Enable hardening of the resulting binaries (against security attacks, detects memory corruption, etc)" OFF)
OCV_OPTION(ENABLE_LTO                 "Enable Link Time Optimization" OFF IF CV_GCC OR MSVC)
OCV_OPTION(ENABLE_THIN_LTO            "Enable Thin LTO" OFF IF CV_CLANG)
OCV_OPTION(GENERATE_ABI_DESCRIPTOR    "Generate XML file for abi_compliance_checker tool" OFF IF UNIX)
OCV_OPTION(OPENCV_GENERATE_PKGCONFIG  "Generate .pc file for pkg-config build tool (deprecated)" OFF)
OCV_OPTION(CV_ENABLE_INTRINSICS       "Use intrinsic-based optimized code" ON )
OCV_OPTION(CV_DISABLE_OPTIMIZATION    "Disable explicit optimized code (dispatched code/intrinsics/loop unrolling/etc)" OFF )
OCV_OPTION(CV_TRACE                   "Enable OpenCV code trace" ON)
OCV_OPTION(OPENCV_GENERATE_SETUPVARS  "Generate setup_vars* scripts" ON IF (NOT ANDROID AND NOT APPLE_FRAMEWORK) )
OCV_OPTION(ENABLE_CONFIG_VERIFICATION "Fail build if actual configuration doesn't match requested (WITH_XXX != HAVE_XXX)" OFF)
OCV_OPTION(OPENCV_ENABLE_MEMALIGN     "Enable posix_memalign or memalign usage" ON)
OCV_OPTION(OPENCV_DISABLE_FILESYSTEM_SUPPORT "Disable filesystem support" OFF)
OCV_OPTION(OPENCV_DISABLE_THREAD_SUPPORT "Build the library without multi-threaded code." OFF)
OCV_OPTION(OPENCV_DISABLE_ENV_SUPPORT "Disable environment variables access (getenv)" (CMAKE_SYSTEM_NAME MATCHES "Windows(CE|Phone|Store)"))
OCV_OPTION(OPENCV_SEMIHOSTING         "Build the library for semihosting target (Arm). See https://developer.arm.com/documentation/100863/latest." OFF)
OCV_OPTION(ENABLE_CUDA_FIRST_CLASS_LANGUAGE "Enable CUDA as a first class language, if enabled dependant projects will need to use CMake >= 3.18" OFF
  VISIBLE_IF (WITH_CUDA AND NOT CMAKE_VERSION VERSION_LESS 3.18)
  VERIFY HAVE_CUDA)

OCV_OPTION(ENABLE_PYLINT              "Add target with Pylint checks"                            (BUILD_DOCS OR BUILD_EXAMPLES) IF (NOT CMAKE_CROSSCOMPILING AND NOT APPLE_FRAMEWORK) )
OCV_OPTION(ENABLE_FLAKE8              "Add target with Python flake8 checker"                    (BUILD_DOCS OR BUILD_EXAMPLES) IF (NOT CMAKE_CROSSCOMPILING AND NOT APPLE_FRAMEWORK) )

if(ENABLE_IMPL_COLLECTION)
  add_definitions(-DCV_COLLECT_IMPL_DATA)
endif()

if(OPENCV_DISABLE_FILESYSTEM_SUPPORT)
  add_definitions(-DOPENCV_HAVE_FILESYSTEM_SUPPORT=0)
endif()

# MathJax is used for math rendering by both Doxygen HTML and JavaDoc, so
# this var have to be defined before "modules" AND "doc" are processed
set(OPENCV_MATHJAX_RELPATH "https://cdn.jsdelivr.net/npm/mathjax@3.0.1" CACHE STRING "URI to a MathJax installation")

# ----------------------------------------------------------------------------
#  Get actual OpenCV version number from sources
# ----------------------------------------------------------------------------
include(cmake/OpenCVVersion.cmake)

ocv_cmake_hook(POST_OPTIONS)

# ----------------------------------------------------------------------------
#  Build & install layouts
# ----------------------------------------------------------------------------

if(OPENCV_TEST_DATA_PATH)
  get_filename_component(OPENCV_TEST_DATA_PATH ${OPENCV_TEST_DATA_PATH} ABSOLUTE)
endif()

# Save libs and executables in the same place
set(EXECUTABLE_OUTPUT_PATH "${CMAKE_BINARY_DIR}/bin" CACHE PATH "Output directory for applications")

if(ANDROID)
  set(LIBRARY_OUTPUT_PATH                "${OpenCV_BINARY_DIR}/lib/${ANDROID_NDK_ABI_NAME}")
  ocv_update(3P_LIBRARY_OUTPUT_PATH      "${OpenCV_BINARY_DIR}/3rdparty/lib/${ANDROID_NDK_ABI_NAME}")
else()
  set(LIBRARY_OUTPUT_PATH                "${OpenCV_BINARY_DIR}/lib")
  ocv_update(3P_LIBRARY_OUTPUT_PATH      "${OpenCV_BINARY_DIR}/3rdparty/lib")
endif()

if(ANDROID)
  if(ANDROID_ABI MATCHES "NEON")
    set(ENABLE_NEON ON)
  endif()
  if(ANDROID_ABI MATCHES "VFPV3")
    set(ENABLE_VFPV3 ON)
  endif()
endif()

if(WIN32)
  # Postfix of DLLs:
  ocv_update(OPENCV_DLLVERSION "${OPENCV_VERSION_MAJOR}${OPENCV_VERSION_MINOR}${OPENCV_VERSION_PATCH}")
  ocv_update(OPENCV_DEBUG_POSTFIX d)
else()
  # Postfix of so's:
  ocv_update(OPENCV_DLLVERSION "")
  ocv_update(OPENCV_DEBUG_POSTFIX "")
endif()

if(DEFINED CMAKE_DEBUG_POSTFIX)
  set(OPENCV_DEBUG_POSTFIX "${CMAKE_DEBUG_POSTFIX}")
endif()

if((INSTALL_CREATE_DISTRIB AND BUILD_SHARED_LIBS AND NOT DEFINED BUILD_opencv_world) OR APPLE_FRAMEWORK)
  set(BUILD_opencv_world ON CACHE INTERNAL "")
endif()

include(cmake/OpenCVInstallLayout.cmake)

# ----------------------------------------------------------------------------
#  Path for build/platform -specific headers
# ----------------------------------------------------------------------------
ocv_update(OPENCV_CONFIG_FILE_INCLUDE_DIR "${CMAKE_BINARY_DIR}/" CACHE PATH "Where to create the platform-dependant cvconfig.h")
ocv_include_directories(${OPENCV_CONFIG_FILE_INCLUDE_DIR})

# ----------------------------------------------------------------------------
#  Path for additional modules
# ----------------------------------------------------------------------------
set(OPENCV_EXTRA_MODULES_PATH "" CACHE PATH "Where to look for additional OpenCV modules (can be ;-separated list of paths)")

# ----------------------------------------------------------------------------
# OpenCV compiler and linker options
# ----------------------------------------------------------------------------

ocv_cmake_hook(POST_CMAKE_BUILD_OPTIONS)

# --- Python Support ---
if(NOT IOS AND NOT XROS)
  include(cmake/OpenCVDetectPython.cmake)
  include(cmake/OpenCVDetectDLPack.cmake)
endif()

include(cmake/OpenCVCompilerOptions.cmake)

ocv_cmake_hook(POST_COMPILER_OPTIONS)

# --- CUDA Support ---
if(ENABLE_CUDA_FIRST_CLASS_LANGUAGE)
  if(CMAKE_VERSION VERSION_LESS 3.18)
    message(WARNING "CUDA: First class language only supported for CMake versions >= 3.18, falling back to FindCUDA!")
    set(ENABLE_CUDA_FIRST_CLASS_LANGUAGE OFF CACHE BOOL "Enable CUDA as a first class language, if enabled dependant projects will need to use CMake >= 3.18" FORCE)
  else()

    # Check CUDA_PATH if supplied
    if(UNIX AND CUDA_PATH AND NOT ENV{CUDA_PATH})
      set(ENV{CUDA_PATH} ${CUDA_PATH})
    elseif(WIN32 AND CUDA_PATH)
      set(ENV{PATH} "${CUDA_PATH}\\bin\;$ENV{PATH}")
    endif()
    include(CheckLanguage)
    check_language(CUDA)

    # Fallback to checking default locations
    if(NOT CMAKE_CUDA_COMPILER)
      # Checking windows default search location isn't possible because the CUDA Toolkit is installed to C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/vXX.X
      if(WIN32)
        if(CMAKE_GENERATOR MATCHES "Visual Studio")
          message(STATUS "CUDA: Not detected, when using stand alone installations with the Visual Studio generator the path to the CUDA toolkit should be manually specified with -Tcuda=. e.g. -Tcuda=\"C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/vXX.X\"")
        else()
          message(STATUS "CUDA: Not detected, for stand alone installations the path to the CUDA toolkit should be manually specified with -DCUDA_PATH=. e.g. -DCUDA_PATH=\"C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/vXX.X\"")
        endif()
      elseif(UNIX)
        message(STATUS "CUDA: Not detected, make sure you have performed the mandatory Post-installation actions described in the CUDA installation guide.\n   For stand alone installations you can set the CUDA_PATH environmental or CMake variable. e.g. export CUDA_PATH=/usr/local/cuda-XX.X or -DCUDA_PATH=/usr/local/cuda-XX.X.")
        message(STATUS "CUDA: Falling back to searching for the CUDA compiler in its default location (/usr/local/cuda)")
        set(CUDA_PATH "/usr/local/cuda" CACHE INTERNAL "")
        set(ENV{CUDA_PATH} ${CUDA_PATH})
        unset(CMAKE_CUDA_COMPILER CACHE)
        unset(CMAKE_CUDA_COMPILER)
        check_language(CUDA)
      endif()
    endif()

    cmake_policy(SET CMP0092 NEW) # CMake 3.15+: leave warning flags out of default CMAKE_<LANG>_FLAGS flags.
    if(CMAKE_CUDA_COMPILER)
      #  CMake 3.18+: if CMAKE_CUDA_ARCHITECTURES is empty enable_language(CUDA) sets it to the default architecture chosen by the compiler, to trigger the OpenCV custom CUDA architecture search an empty value needs to be respected see https://github.com/opencv/opencv/pull/25941.
      if(CMAKE_CUDA_ARCHITECTURES)
        set(USER_DEFINED_CMAKE_CUDA_ARCHITECTURES TRUE)
      endif()
      enable_language(CUDA)
      if(NOT USER_DEFINED_CMAKE_CUDA_ARCHITECTURES)
        set(CMAKE_CUDA_ARCHITECTURES "")
      endif()
    elseif(UNIX)
      message(WARNING "CUDA: Not detected!  If you are not using the default host compiler (g++) then you need to specify both CMAKE_CUDA_HOST_COMPILER and CMAKE_CUDA_COMPILER. e.g. -DCMAKE_CUDA_HOST_COMPILER=/usr/bin/clang++ -DCMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc.")
    endif()
  endif()
endif()

# ----------------------------------------------------------------------------
#       CHECK FOR SYSTEM LIBRARIES, OPTIONS, ETC..
# ----------------------------------------------------------------------------
if(UNIX OR MINGW)
  if(NOT APPLE_FRAMEWORK OR OPENCV_ENABLE_PKG_CONFIG)
    if(CMAKE_CROSSCOMPILING AND NOT DEFINED ENV{PKG_CONFIG_LIBDIR} AND NOT DEFINED ENV{PKG_CONFIG_SYSROOT_DIR}
        AND NOT OPENCV_ENABLE_PKG_CONFIG
    )
      if(NOT PkgConfig_FOUND)
        message(STATUS "OpenCV disables pkg-config to avoid using of host libraries. Consider using PKG_CONFIG_LIBDIR to specify target SYSROOT")
      elseif(OPENCV_SKIP_PKG_CONFIG_WARNING)
        message(WARNING "pkg-config is enabled in cross-compilation mode without defining of PKG_CONFIG_LIBDIR environment variable. This may lead to misconfigured host-based dependencies.")
      endif()
    elseif(OPENCV_DISABLE_PKG_CONFIG)
      if(PkgConfig_FOUND)
        message(WARNING "OPENCV_DISABLE_PKG_CONFIG flag has no effect")
      endif()
    else()
      find_package(PkgConfig QUIET)
    endif()
  endif()
  include(CheckFunctionExists)
  include(CheckIncludeFile)
  include(CheckLibraryExists)
  include(CheckSymbolExists)

  if(NOT APPLE)
    CHECK_INCLUDE_FILE(pthread.h HAVE_PTHREAD)
    if(ANDROID)
      set(OPENCV_LINKER_LIBS ${OPENCV_LINKER_LIBS} dl m log)
    elseif(CMAKE_SYSTEM_NAME MATCHES "FreeBSD|NetBSD|DragonFly|OpenBSD|Haiku")
      set(OPENCV_LINKER_LIBS ${OPENCV_LINKER_LIBS} m pthread)
    elseif(EMSCRIPTEN)
      # no need to link to system libs with emscripten
    elseif(QNXNTO)
      set(OPENCV_LINKER_LIBS ${OPENCV_LINKER_LIBS} m)
      CHECK_LIBRARY_EXISTS(regex regexec "" HAVE_REGEX_LIBRARY)
      if(HAVE_REGEX_LIBRARY)
        set(OPENCV_LINKER_LIBS ${OPENCV_LINKER_LIBS} regex)
      endif()
    elseif(MINGW)
      set(OPENCV_LINKER_LIBS ${OPENCV_LINKER_LIBS} pthread)
    else()
      set(OPENCV_LINKER_LIBS ${OPENCV_LINKER_LIBS} dl m pthread rt)
    endif()
  else()
    set(HAVE_PTHREAD 1)
  endif()

  # Ensure that libpthread is not listed as one of the libraries to pass to the linker.
  if (OPENCV_DISABLE_THREAD_SUPPORT)
    list(REMOVE_ITEM OPENCV_LINKER_LIBS pthread)
  endif()

  if(OPENCV_ENABLE_MEMALIGN)
    CHECK_SYMBOL_EXISTS(posix_memalign stdlib.h HAVE_POSIX_MEMALIGN)
    CHECK_INCLUDE_FILE(malloc.h HAVE_MALLOC_H)
    if(HAVE_MALLOC_H)
      CHECK_SYMBOL_EXISTS(memalign malloc.h HAVE_MEMALIGN)
    endif()
    # TODO:
    # - std::aligned_alloc() C++17 / C11
  endif()

  CHECK_SYMBOL_EXISTS(getauxval sys/auxv.h HAVE_GETAUXVAL)
  CHECK_SYMBOL_EXISTS(elf_aux_info sys/auxv.h HAVE_ELF_AUX_INFO)
elseif(WIN32)
  include(CheckIncludeFile)
  include(CheckSymbolExists)

  if(OPENCV_ENABLE_MEMALIGN)
    CHECK_INCLUDE_FILE(malloc.h HAVE_MALLOC_H)
    if(HAVE_MALLOC_H)
      CHECK_SYMBOL_EXISTS(_aligned_malloc malloc.h HAVE_WIN32_ALIGNED_MALLOC)
    endif()
  endif()
endif()

if(DEFINED OPENCV_ALGO_HINT_DEFAULT)
  if(NOT OPENCV_ALGO_HINT_DEFAULT STREQUAL "ALGO_HINT_ACCURATE" AND
     NOT OPENCV_ALGO_HINT_DEFAULT STREQUAL "ALGO_HINT_APPROX")
    message(FATAL_ERROR "OPENCV_ALGO_HINT_DEFAULT should be one of ALGO_HINT_ACCURATE or ALGO_HINT_APPROX.")
  endif()
endif()

include(cmake/OpenCVPCHSupport.cmake)
include(cmake/OpenCVModule.cmake)

# ----------------------------------------------------------------------------
#  Detect endianness of build platform
# ----------------------------------------------------------------------------

if(IOS OR XROS)
  # test_big_endian needs try_compile, which doesn't work for iOS
  # http://public.kitware.com/Bug/view.php?id=12288
  set(WORDS_BIGENDIAN 0)
else()
  include(TestBigEndian)
  test_big_endian(WORDS_BIGENDIAN)
endif()

# ----------------------------------------------------------------------------
#  Detect 3rd-party libraries
# ----------------------------------------------------------------------------

if(ANDROID AND WITH_CPUFEATURES)
  add_subdirectory(3rdparty/cpufeatures)
  set(HAVE_CPUFEATURES 1)
endif()

include(cmake/OpenCVFindFrameworks.cmake)

include(cmake/OpenCVFindLibsGrfmt.cmake)
include(cmake/OpenCVFindLibsGUI.cmake)
include(cmake/OpenCVFindLibsVideo.cmake)
include(cmake/OpenCVFindLibsPerf.cmake)
include(cmake/OpenCVFindLAPACK.cmake)
include(cmake/OpenCVFindProtobuf.cmake)
include(cmake/OpenCVDetectFlatbuffers.cmake)
if(WITH_TIMVX)
  include(cmake/OpenCVFindTIMVX.cmake)
endif()
if(WITH_CANN)
  include(cmake/OpenCVFindCANN.cmake)
endif()

# ----------------------------------------------------------------------------
#  Detect other 3rd-party libraries/tools
# ----------------------------------------------------------------------------

# --- Java Support ---
if(BUILD_JAVA)
  if(ANDROID)
    include(cmake/android/OpenCVDetectAndroidSDK.cmake)
  else()
    include(cmake/OpenCVDetectApacheAnt.cmake)
    if(ANT_EXECUTABLE AND NOT OPENCV_JAVA_IGNORE_ANT)
      ocv_update(OPENCV_JAVA_SDK_BUILD_TYPE "ANT")
    elseif(NOT ANDROID)
      find_package(Java)
      if(Java_FOUND)
        include(UseJava)
        ocv_update(OPENCV_JAVA_SDK_BUILD_TYPE "JAVA")
      endif()
    endif()
    find_package(JNI)
  endif()
endif()

if(ENABLE_PYLINT AND PYTHON_DEFAULT_AVAILABLE)
  include(cmake/OpenCVPylint.cmake)
endif()
if(ENABLE_FLAKE8 AND PYTHON_DEFAULT_AVAILABLE)
  find_package(Flake8 QUIET)
  if(NOT FLAKE8_FOUND OR NOT FLAKE8_EXECUTABLE)
    include("${CMAKE_CURRENT_LIST_DIR}/cmake/FindFlake8.cmake")
  endif()
  if(FLAKE8_FOUND)
    list(APPEND OPENCV_FLAKE8_EXCLUDES ".git" "__pycache__" "config.py" "*.config.py" "config-*.py")
    list(APPEND OPENCV_FLAKE8_EXCLUDES "svgfig.py")  # 3rdparty
    if(NOT PYTHON3_VERSION_STRING VERSION_GREATER 3.6)
      # Python 3.6+ (PEP 526): variable annotations (type hints)
      list(APPEND OPENCV_FLAKE8_EXCLUDES "samples/dnn/dnn_model_runner/dnn_conversion/common/test/configs")
    endif()
    string(REPLACE ";" "," OPENCV_FLAKE8_EXCLUDES_STR "${OPENCV_FLAKE8_EXCLUDES}")
    add_custom_target(check_flake8
        COMMAND "${FLAKE8_EXECUTABLE}" . --count --select=E9,E901,E999,F821,F822,F823 --show-source --statistics --exclude='${OPENCV_FLAKE8_EXCLUDES_STR}'
        WORKING_DIRECTORY "${OpenCV_SOURCE_DIR}"
        COMMENT "Running flake8"
    )
  endif()
endif()


if(ANDROID AND ANDROID_EXECUTABLE AND ANT_EXECUTABLE AND (ANT_VERSION VERSION_GREATER 1.7) AND (ANDROID_TOOLS_Pkg_Revision GREATER 13))
  SET(CAN_BUILD_ANDROID_PROJECTS TRUE)
else()
  SET(CAN_BUILD_ANDROID_PROJECTS FALSE)
endif()

# --- OpenCL ---
if(WITH_OPENCL)
  include(cmake/OpenCVDetectOpenCL.cmake)
endif()

# --- Halide ---
if(WITH_HALIDE)
  include(cmake/OpenCVDetectHalide.cmake)
endif()

# --- VkCom ---
if(WITH_VULKAN)
  include(cmake/OpenCVDetectVulkan.cmake)
endif()

# --- WebNN ---
if(WITH_WEBNN)
  include(cmake/OpenCVDetectWebNN.cmake)
endif()

# --- Inference Engine ---
if(WITH_INF_ENGINE OR WITH_OPENVINO)
  include(cmake/OpenCVDetectInferenceEngine.cmake)
endif()

# --- DirectX ---
if(WITH_DIRECTX)
  include(cmake/OpenCVDetectDirectX.cmake)
endif()
# --- DirectML ---
if(WITH_DIRECTML)
  include(cmake/OpenCVDetectDirectML.cmake)
endif()

if(WITH_VTK)
  include(cmake/OpenCVDetectVTK.cmake)
endif()

if(WITH_OPENVX)
  include(cmake/FindOpenVX.cmake)
endif()

if(WITH_QUIRC)
  add_subdirectory(3rdparty/quirc)
  set(HAVE_QUIRC TRUE)
endif()

if(WITH_ONNX)
  include(cmake/FindONNX.cmake)
endif()

# ----------------------------------------------------------------------------
# OpenCV HAL
# ----------------------------------------------------------------------------
set(_hal_includes "")
macro(ocv_hal_register HAL_LIBRARIES_VAR HAL_HEADERS_VAR HAL_INCLUDE_DIRS_VAR)
  # 1. libraries
  foreach (l ${${HAL_LIBRARIES_VAR}})
    if(NOT TARGET ${l})
      get_filename_component(l "${l}" ABSOLUTE)
    endif()
    list(APPEND OPENCV_HAL_LINKER_LIBS ${l})
  endforeach()
  # 2. headers
  foreach (h ${${HAL_HEADERS_VAR}})
    set(_hal_includes "${_hal_includes}\n#include \"${h}\"")
  endforeach()
  # 3. include paths
  ocv_include_directories(${${HAL_INCLUDE_DIRS_VAR}})
endmacro()

if(NOT DEFINED OpenCV_HAL)
  set(OpenCV_HAL "OpenCV_HAL")
endif()

if(HAVE_IPP)
  ocv_debug_message(STATUS "Enable IPP acceleration")
  if(NOT ";${OpenCV_HAL};" MATCHES ";ipp;")
    set(OpenCV_HAL "ipp;${OpenCV_HAL}")
  endif()
endif()

if(HAVE_OPENVX)
  ocv_debug_message(STATUS "Enable OpenVX acceleration")
  if(NOT ";${OpenCV_HAL};" MATCHES ";openvx;")
    set(OpenCV_HAL "openvx;${OpenCV_HAL}")
  endif()
endif()

if(HAVE_FASTCV)
  ocv_debug_message(STATUS "Enable FastCV acceleration")
  if(NOT ";${OpenCV_HAL};" MATCHES ";fastcv;")
    set(OpenCV_HAL "fastcv;${OpenCV_HAL}")
  endif()
endif()

if(HAVE_KLEIDICV)
  ocv_debug_message(STATUS "Enable KleidiCV acceleration")
  if(NOT ";${OpenCV_HAL};" MATCHES ";kleidicv;")
    set(OpenCV_HAL "kleidicv;${OpenCV_HAL}")
  endif()
endif()

if(WITH_CAROTENE)
  ocv_debug_message(STATUS "Enable carotene acceleration")
  if(NOT ";${OpenCV_HAL};" MATCHES ";carotene;")
    set(OpenCV_HAL "carotene;${OpenCV_HAL}")
  endif()
endif()

if(WITH_NDSRVP)
  ocv_debug_message(STATUS "Andes RVP 3rdparty NDSRVP enabled")
  if(NOT ";${OpenCV_HAL};" MATCHES ";ndsrvp;")
    set(OpenCV_HAL "ndsrvp;${OpenCV_HAL}")
  endif()
endif()

if(WITH_HAL_RVV)
  ocv_debug_message(STATUS "Enable RVV HAL acceleration")
  if(NOT ";${OpenCV_HAL};" MATCHES ";rvvhal;")
    set(OpenCV_HAL "rvvhal;${OpenCV_HAL}")
  endif()
endif()

foreach(hal ${OpenCV_HAL})
  if(hal STREQUAL "carotene")
    if(";${CPU_BASELINE_FINAL};" MATCHES ";NEON;")
      add_subdirectory(hal/carotene/hal)
      ocv_hal_register(CAROTENE_HAL_LIBRARIES CAROTENE_HAL_HEADERS CAROTENE_HAL_INCLUDE_DIRS)
      list(APPEND OpenCV_USED_HAL "carotene (ver ${CAROTENE_HAL_VERSION})")
    else()
      message(STATUS "Carotene: NEON is not available, disabling carotene...")
    endif()
  elseif(hal STREQUAL "fastcv")
    if((ARM OR AARCH64) AND (ANDROID OR (UNIX AND NOT APPLE AND NOT IOS AND NOT XROS)))
      add_subdirectory(hal/fastcv)
      ocv_hal_register(FASTCV_HAL_LIBRARIES FASTCV_HAL_HEADERS FASTCV_HAL_INCLUDE_DIRS)
      list(APPEND OpenCV_USED_HAL "fastcv (ver ${FASTCV_HAL_VERSION})")
    else()
      message(STATUS "FastCV: fastcv is not available, disabling fastcv...")
    endif()
  elseif(hal STREQUAL "kleidicv")
    add_subdirectory(hal/kleidicv)
    ocv_hal_register(KLEIDICV_HAL_LIBRARIES KLEIDICV_HAL_HEADERS KLEIDICV_HAL_INCLUDE_DIRS)
    list(APPEND OpenCV_USED_HAL "KleidiCV (ver ${KLEIDICV_HAL_VERSION})")
  elseif(hal STREQUAL "ndsrvp")
    if(CMAKE_C_FLAGS MATCHES "-mext-dsp" AND CMAKE_CXX_FLAGS MATCHES "-mext-dsp" AND NOT ";${CPU_BASELINE_FINAL};" MATCHES ";RVV;")
      add_subdirectory(hal/ndsrvp)
      ocv_hal_register(NDSRVP_HAL_LIBRARIES NDSRVP_HAL_HEADERS NDSRVP_HAL_INCLUDE_DIRS)
      list(APPEND OpenCV_USED_HAL "ndsrvp (ver ${NDSRVP_HAL_VERSION})")
    else()
      message(STATUS "NDSRVP: Andes GNU Toolchain DSP extension is not enabled, disabling ndsrvp...")
    endif()
  elseif(hal STREQUAL "rvvhal")
    if(";${CPU_BASELINE_FINAL};" MATCHES ";RVV;")
      add_subdirectory(hal/riscv-rvv)
      ocv_hal_register(RVV_HAL_LIBRARIES RVV_HAL_HEADERS RVV_HAL_INCLUDE_DIRS)
      list(APPEND OpenCV_USED_HAL "RVV HAL (ver ${RVV_HAL_VERSION})")
    else()
      message(STATUS "RVV HAL: RVV is not available, disabling RVV HAL...")
    endif()
  elseif(hal STREQUAL "ipp")
    add_subdirectory(hal/ipp)
    ocv_hal_register(IPP_HAL_LIBRARIES IPP_HAL_HEADERS IPP_HAL_INCLUDE_DIRS)
    list(APPEND OpenCV_USED_HAL "ipp (ver ${IPP_HAL_VERSION})")
  elseif(hal STREQUAL "openvx")
    add_subdirectory(hal/openvx)
    ocv_hal_register(OPENVX_HAL_LIBRARIES OPENVX_HAL_HEADERS OPENVX_HAL_INCLUDE_DIRS)
    list(APPEND OpenCV_USED_HAL "openvx (ver ${OPENVX_HAL_VERSION})")
  else()
    ocv_debug_message(STATUS "OpenCV HAL: ${hal} ...")
    ocv_clear_vars(OpenCV_HAL_LIBRARIES OpenCV_HAL_HEADERS OpenCV_HAL_INCLUDE_DIRS)
    find_package(${hal} NO_MODULE QUIET)
    if(${hal}_FOUND)
      ocv_hal_register(OpenCV_HAL_LIBRARIES OpenCV_HAL_HEADERS OpenCV_HAL_INCLUDE_DIRS)
      list(APPEND OpenCV_USED_HAL "${hal} (ver ${${hal}_VERSION})")
    endif()
  endif()
endforeach()
configure_file("${OpenCV_SOURCE_DIR}/cmake/templates/custom_hal.hpp.in" "${OPENCV_CONFIG_FILE_INCLUDE_DIR}/custom_hal.hpp" @ONLY)
unset(_hal_includes)


# ----------------------------------------------------------------------------
# Code trace support
# ----------------------------------------------------------------------------
if(CV_TRACE)
  include(cmake/OpenCVDetectTrace.cmake)
endif()

ocv_cmake_hook(POST_DETECT_DEPENDECIES)  # typo, deprecated (2019-06)
ocv_cmake_hook(POST_DETECT_DEPENDENCIES)

# ----------------------------------------------------------------------------
# Solution folders:
# ----------------------------------------------------------------------------
if(ENABLE_SOLUTION_FOLDERS)
  set_property(GLOBAL PROPERTY USE_FOLDERS ON)
  set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMakeTargets")
endif()

# Extra OpenCV targets: uninstall, package_source, perf, etc.
include(cmake/OpenCVExtraTargets.cmake)

# ----------------------------------------------------------------------------
# Process subdirectories
# ----------------------------------------------------------------------------

# opencv.hpp and legacy headers
add_subdirectory(include)

# Enable compiler options for OpenCV modules/apps/samples only (ignore 3rdparty)
ocv_add_modules_compiler_options()

# OpenCV modules
ocv_register_modules()

# Generate targets for documentation
add_subdirectory(doc)

# various data that is used by cv libraries and/or demo applications.
add_subdirectory(data)

# extra applications
if(BUILD_opencv_apps)
  add_subdirectory(apps)
endif()

# examples
if(BUILD_EXAMPLES OR BUILD_ANDROID_EXAMPLES OR INSTALL_ANDROID_EXAMPLES OR INSTALL_PYTHON_EXAMPLES OR INSTALL_C_EXAMPLES)
  add_subdirectory(samples)
endif()

# ----------------------------------------------------------------------------
# Finalization: generate configuration-based files
# ----------------------------------------------------------------------------

ocv_cmake_hook(PRE_FINALIZE)

# Generate platform-dependent and configuration-dependent headers
include(cmake/OpenCVGenHeaders.cmake)

# Generate opencv.pc for pkg-config command
if(OPENCV_GENERATE_PKGCONFIG)
  include(cmake/OpenCVGenPkgconfig.cmake)
endif()

# Generate OpenCV.mk for ndk-build (Android build tool)
include(cmake/OpenCVGenAndroidMK.cmake)

# Generate OpenCVConfig.cmake and OpenCVConfig-version.cmake for cmake projects
include(cmake/OpenCVGenConfig.cmake)

# Generate Info.plist for the iOS/visionOS framework
if(APPLE_FRAMEWORK)
  include(cmake/OpenCVGenInfoPlist.cmake)
endif()

# Generate ABI descriptor
include(cmake/OpenCVGenABI.cmake)

# Generate environment setup file
if(INSTALL_TESTS AND OPENCV_TEST_DATA_PATH)
  if(ANDROID)
    get_filename_component(TEST_PATH ${OPENCV_TEST_INSTALL_PATH} DIRECTORY)
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/templates/opencv_run_all_tests_android.sh.in"
                   "${CMAKE_BINARY_DIR}/unix-install/opencv_run_all_tests.sh" @ONLY)
    install(PROGRAMS "${CMAKE_BINARY_DIR}/unix-install/opencv_run_all_tests.sh"
            DESTINATION ./ COMPONENT tests)
  elseif(WIN32)
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/templates/opencv_run_all_tests_windows.cmd.in"
                   "${CMAKE_BINARY_DIR}/win-install/opencv_run_all_tests.cmd" @ONLY)
    install(PROGRAMS "${CMAKE_BINARY_DIR}/win-install/opencv_run_all_tests.cmd"
            DESTINATION ${OPENCV_TEST_INSTALL_PATH} COMPONENT tests)
  elseif(UNIX)
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/templates/opencv_run_all_tests_unix.sh.in"
                   "${CMAK
... [truncated]
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

