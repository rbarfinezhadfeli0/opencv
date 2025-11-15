# Documentation for `doc/tutorials/gpu/table_of_content_gpu.markdown`

## File Metadata

- **Full Path**: `doc/tutorials/gpu/table_of_content_gpu.markdown`
- **File Name**: `table_of_content_gpu.markdown`
- **File Size**: 919 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/tutorials/gpu/table_of_content_gpu.markdown](../../../doc/tutorials/gpu/table_of_content_gpu.markdown)

## Purpose and Role

This file is located in the `doc/tutorials/gpu` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
@cond CUDA_MODULES
GPU-Accelerated Computer Vision (cuda module) {#tutorial_table_of_content_gpu}
=============================================

Squeeze out every little computation power from your system by using the power of your video card to
run the OpenCV algorithms.

-   @subpage tutorial_gpu_basics_similarity

    *Languages:* C++

    *Compatibility:* \> OpenCV 2.0

    *Author:* Bernát Gábor

    This will give a good grasp on how to approach coding on the GPU module, once you already know
    how to handle the other modules. As a test case it will port the similarity methods from the
    tutorial @ref tutorial_video_input_psnr_ssim to the GPU.

-   @subpage tutorial_gpu_thrust_interop

    *Languages:* C++

    *Compatibility:* \>= OpenCV 3.0

    This tutorial will show you how to wrap a GpuMat into a thrust iterator in order to be able to
    use the functions in the thrust library.
@endcond

```

## General Information

This file is part of the OpenCV repository infrastructure.

