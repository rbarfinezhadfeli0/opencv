# Documentation for `modules/core/include/opencv2/core/cuda/detail/color_detail.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/cuda/detail/color_detail.hpp`
- **File Name**: `color_detail.hpp`
- **File Size**: 223,992 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/cuda/detail/color_detail.hpp](../../../../../../../modules/core/include/opencv2/core/cuda/detail/color_detail.hpp)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core/cuda/detail` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#ifndef OPENCV_CUDA_COLOR_DETAIL_HPP
#define OPENCV_CUDA_COLOR_DETAIL_HPP

#include "../common.hpp"
#include "../vec_traits.hpp"
#include "../saturate_cast.hpp"
#include "../limits.hpp"
#include "../functional.hpp"

//! @cond IGNORED

namespace cv { namespace cuda { namespace device
{
    #ifndef CV_DESCALE
        #define CV_DESCALE(x, n) (((x) + (1 << ((n)-1))) >> (n))
    #endif

    namespace color_detail
    {
        template<typename T> struct ColorChannel
        {
            typedef float worktype_f;
            static __device__ __forceinline__ T max() { return numeric_limits<T>::max(); }
            static __device__ __forceinline__ T half() { return (T)(max()/2 + 1); }
        };

        template<> struct ColorChannel<float>
        {
            typedef float worktype_f;
            static __device__ __forceinline__ float max() { return 1.f; }
            static __device__ __forceinline__ float half() { return 0.5f; }
        };

        template <typename T> static __device__ __forceinline__ void setAlpha(typename TypeVec<T, 3>::vec_type& vec, T val)
        {
        }

        template <typename T> static __device__ __forceinline__ void setAlpha(typename TypeVec<T, 4>::vec_type& vec, T val)
        {
            vec.w = val;
        }

        template <typename T> static __device__ __forceinline__ T getAlpha(const typename TypeVec<T, 3>::vec_type& vec)
        {
            return ColorChannel<T>::max();
        }

        template <typename T> static __device__ __forceinline__ T getAlpha(const typename TypeVec<T, 4>::vec_type& vec)
        {
            return vec.w;
        }

        //constants for conversion from/to RGB and Gray, YUV, YCrCb according to BT.601
        constexpr float B2YF = 0.114f;
        constexpr float G2YF = 0.587f;
        constexpr float R2YF = 0.299f;

        //to YCbCr
        constexpr float YCBF = 0.564f; // == 1/2/(1-B2YF)
        constexpr float YCRF = 0.713f; // == 1/2/(1-R2YF)
        const     int   YCBI = 9241;  // == YCBF*16384
        const     int   YCRI = 11682; // == YCRF*16384
        //to YUV
        constexpr float B2UF = 0.492f;
        constexpr float R2VF = 0.877f;
        const     int   B2UI = 8061;  // == B2UF*16384
        const     int   R2VI = 14369; // == R2VF*16384
        //from YUV
        constexpr float U2BF = 2.032f;
        constexpr float U2GF = -0.395f;
        constexpr float V2GF = -0.581f;
        constexpr float V2RF = 1.140f;
        const     int   U2BI = 33292;
        const     int   U2GI = -6472;
        const     int   V2GI = -9519;
        const     int   V2RI = 18678;
        //from YCrCb
        constexpr float CB2BF = 1.773f;
        constexpr float CB2GF = -0.344f;
        constexpr float CR2GF = -0.714f;
        constexpr float CR2RF = 1.403f;
        const     int   CB2BI = 29049;
        const     int   CB2GI = -5636;
        const     int   CR2GI = -11698;
        const     int   CR2RI = 22987;

        enum
        {
            yuv_shift  = 14,
            xyz_shift  = 12,
            gray_shift = 15,
            R2Y        = 4899,
            G2Y        = 9617,
            B2Y        = 1868,
            RY15 =  9798, // == R2YF*32768 + 0.5
            GY15 = 19235, // == G2YF*32768 + 0.5
            BY15 =  3735, // == B2YF*32768 + 0.5
            BLOCK_SIZE = 256
        };
    }

////////////////// Various 3/4-channel to 3/4-channel RGB transformations /////////////////

    namespace color_detail
    {
        template <typename T, int scn, int dcn, int bidx> struct RGB2RGB
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ typename TypeVec<T, dcn>::vec_type operator()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                dst.x = (&src.x)[bidx];
                dst.y = src.y;
                dst.z = (&src.x)[bidx^2];
                setAlpha(dst, getAlpha<T>(src));

                return dst;
            }

            __host__ __device__ __forceinline__ RGB2RGB() {}
            __host__ __device__ __forceinline__ RGB2RGB(const RGB2RGB&) {}
        };

        template <> struct RGB2RGB<uchar, 4, 4, 2> : unary_function<uint, uint>
        {
            __device__ uint operator()(uint src) const
            {
                uint dst = 0;

                dst |= (0xffu & (src >> 16));
                dst |= (0xffu & (src >> 8)) << 8;
                dst |= (0xffu & (src)) << 16;
                dst |= (0xffu & (src >> 24)) << 24;

                return dst;
            }

            __host__ __device__ __forceinline__ RGB2RGB() {}
            __host__ __device__ __forceinline__ RGB2RGB(const RGB2RGB&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB2RGB_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2RGB<T, scn, dcn, bidx> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

/////////// Transforming 16-bit (565 or 555) RGB to/from 24/32-bit (888[8]) RGB //////////

    namespace color_detail
    {
        template <int green_bits, int bidx> struct RGB2RGB5x5Converter;
        template<int bidx> struct RGB2RGB5x5Converter<6, bidx>
        {
            static __device__ __forceinline__ ushort cvt(const uchar3& src)
            {
                return (ushort)(((&src.x)[bidx] >> 3) | ((src.y & ~3) << 3) | (((&src.x)[bidx^2] & ~7) << 8));
            }

            static __device__ __forceinline__ ushort cvt(uint src)
            {
                uint b = 0xffu & (src >> (bidx * 8));
                uint g = 0xffu & (src >> 8);
                uint r = 0xffu & (src >> ((bidx ^ 2) * 8));
                return (ushort)((b >> 3) | ((g & ~3) << 3) | ((r & ~7) << 8));
            }
        };

        template<int bidx> struct RGB2RGB5x5Converter<5, bidx>
        {
            static __device__ __forceinline__ ushort cvt(const uchar3& src)
            {
                return (ushort)(((&src.x)[bidx] >> 3) | ((src.y & ~7) << 2) | (((&src.x)[bidx^2] & ~7) << 7));
            }

            static __device__ __forceinline__ ushort cvt(uint src)
            {
                uint b = 0xffu & (src >> (bidx * 8));
                uint g = 0xffu & (src >> 8);
                uint r = 0xffu & (src >> ((bidx ^ 2) * 8));
                uint a = 0xffu & (src >> 24);
                return (ushort)((b >> 3) | ((g & ~7) << 2) | ((r & ~7) << 7) | (a * 0x8000));
            }
        };

        template<int scn, int bidx, int green_bits> struct RGB2RGB5x5;

        template<int bidx, int green_bits> struct RGB2RGB5x5<3, bidx,green_bits> : unary_function<uchar3, ushort>
        {
            __device__ __forceinline__ ushort operator()(const uchar3& src) const
            {
                return RGB2RGB5x5Converter<green_bits, bidx>::cvt(src);
            }

            __host__ __device__ __forceinline__ RGB2RGB5x5() {}
            __host__ __device__ __forceinline__ RGB2RGB5x5(const RGB2RGB5x5&) {}
        };

        template<int bidx, int green_bits> struct RGB2RGB5x5<4, bidx,green_bits> : unary_function<uint, ushort>
        {
            __device__ __forceinline__ ushort operator()(uint src) const
            {
                return RGB2RGB5x5Converter<green_bits, bidx>::cvt(src);
            }

            __host__ __device__ __forceinline__ RGB2RGB5x5() {}
            __host__ __device__ __forceinline__ RGB2RGB5x5(const RGB2RGB5x5&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB2RGB5x5_TRAITS(name, scn, bidx, green_bits) \
    struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2RGB5x5<scn, bidx, green_bits> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

    namespace color_detail
    {
        template <int green_bits, int bidx> struct RGB5x52RGBConverter;

        template <int bidx> struct RGB5x52RGBConverter<5, bidx>
        {
            static __device__ __forceinline__ void cvt(uint src, uchar3& dst)
            {
                (&dst.x)[bidx] = src << 3;
                dst.y = (src >> 2) & ~7;
                (&dst.x)[bidx ^ 2] = (src >> 7) & ~7;
            }

            static __device__ __forceinline__ void cvt(uint src, uint& dst)
            {
                dst = 0;

                dst |= (0xffu & (src << 3)) << (bidx * 8);
                dst |= (0xffu & ((src >> 2) & ~7)) << 8;
                dst |= (0xffu & ((src >> 7) & ~7)) << ((bidx ^ 2) * 8);
                dst |= ((src & 0x8000) * 0xffu) << 24;
            }
        };

        template <int bidx> struct RGB5x52RGBConverter<6, bidx>
        {
            static __device__ __forceinline__ void cvt(uint src, uchar3& dst)
            {
                (&dst.x)[bidx] = src << 3;
                dst.y = (src >> 3) & ~3;
                (&dst.x)[bidx ^ 2] = (src >> 8) & ~7;
            }

            static __device__ __forceinline__ void cvt(uint src, uint& dst)
            {
                dst = 0xffu << 24;

                dst |= (0xffu & (src << 3)) << (bidx * 8);
                dst |= (0xffu &((src >> 3) & ~3)) << 8;
                dst |= (0xffu & ((src >> 8) & ~7)) << ((bidx ^ 2) * 8);
            }
        };

        template <int dcn, int bidx, int green_bits> struct RGB5x52RGB;

        template <int bidx, int green_bits> struct RGB5x52RGB<3, bidx, green_bits> : unary_function<ushort, uchar3>
        {
            __device__ __forceinline__ uchar3 operator()(ushort src) const
            {
                uchar3 dst;
                RGB5x52RGBConverter<green_bits, bidx>::cvt(src, dst);
                return dst;
            }
            __host__ __device__ __forceinline__ RGB5x52RGB() {}
            __host__ __device__ __forceinline__ RGB5x52RGB(const RGB5x52RGB&) {}

        };

        template <int bidx, int green_bits> struct RGB5x52RGB<4, bidx, green_bits> : unary_function<ushort, uint>
        {
            __device__ __forceinline__ uint operator()(ushort src) const
            {
                uint dst;
                RGB5x52RGBConverter<green_bits, bidx>::cvt(src, dst);
                return dst;
            }
            __host__ __device__ __forceinline__ RGB5x52RGB() {}
            __host__ __device__ __forceinline__ RGB5x52RGB(const RGB5x52RGB&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB5x52RGB_TRAITS(name, dcn, bidx, green_bits) \
    struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB5x52RGB<dcn, bidx, green_bits> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

///////////////////////////////// Grayscale to Color ////////////////////////////////

    namespace color_detail
    {
        template <typename T, int dcn> struct Gray2RGB : unary_function<T, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator()(T src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                dst.z = dst.y = dst.x = src;
                setAlpha(dst, ColorChannel<T>::max());

                return dst;
            }
            __host__ __device__ __forceinline__ Gray2RGB() {}
            __host__ __device__ __forceinline__ Gray2RGB(const Gray2RGB&) {}
        };

        template <> struct Gray2RGB<uchar, 4> : unary_function<uchar, uint>
        {
            __device__ __forceinline__ uint operator()(uint src) const
            {
                uint dst = 0xffu << 24;

                dst |= src;
                dst |= src << 8;
                dst |= src << 16;

                return dst;
            }
            __host__ __device__ __forceinline__ Gray2RGB() {}
            __host__ __device__ __forceinline__ Gray2RGB(const Gray2RGB&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_GRAY2RGB_TRAITS(name, dcn) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::Gray2RGB<T, dcn> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

    namespace color_detail
    {
        template <int green_bits> struct Gray2RGB5x5Converter;
        template<> struct Gray2RGB5x5Converter<6>
        {
            static __device__ __forceinline__ ushort cvt(uint t)
            {
                return (ushort)((t >> 3) | ((t & ~3) << 3) | ((t & ~7) << 8));
            }
        };

        template<> struct Gray2RGB5x5Converter<5>
        {
            static __device__ __forceinline__ ushort cvt(uint t)
            {
                t >>= 3;
                return (ushort)(t | (t << 5) | (t << 10));
            }
        };

        template<int green_bits> struct Gray2RGB5x5 : unary_function<uchar, ushort>
        {
            __device__ __forceinline__ ushort operator()(uint src) const
            {
                return Gray2RGB5x5Converter<green_bits>::cvt(src);
            }

            __host__ __device__ __forceinline__ Gray2RGB5x5() {}
            __host__ __device__ __forceinline__ Gray2RGB5x5(const Gray2RGB5x5&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_GRAY2RGB5x5_TRAITS(name, green_bits) \
    struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::Gray2RGB5x5<green_bits> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

///////////////////////////////// Color to Grayscale ////////////////////////////////

    namespace color_detail
    {
        template <int green_bits> struct RGB5x52GrayConverter;
        template <> struct RGB5x52GrayConverter<6>
        {
            static __device__ __forceinline__ uchar cvt(uint t)
            {
                return (uchar)CV_DESCALE(((t << 3) & 0xf8) * BY15 + ((t >> 3) & 0xfc) * GY15 + ((t >> 8) & 0xf8) * RY15, gray_shift);
            }
        };

        template <> struct RGB5x52GrayConverter<5>
        {
            static __device__ __forceinline__ uchar cvt(uint t)
            {
                return (uchar)CV_DESCALE(((t << 3) & 0xf8) * BY15 + ((t >> 2) & 0xf8) * GY15 + ((t >> 7) & 0xf8) * RY15, gray_shift);
            }
        };

        template<int green_bits> struct RGB5x52Gray : unary_function<ushort, uchar>
        {
            __device__ __forceinline__ uchar operator()(uint src) const
            {
                return RGB5x52GrayConverter<green_bits>::cvt(src);
            }
            __host__ __device__ __forceinline__ RGB5x52Gray() {}
            __host__ __device__ __forceinline__ RGB5x52Gray(const RGB5x52Gray&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB5x52GRAY_TRAITS(name, green_bits) \
    struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB5x52Gray<green_bits> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

    namespace color_detail
    {
        template <int bidx, typename T> static __device__ __forceinline__ T RGB2GrayConvert(const T* src)
        {
            return (T)CV_DESCALE((unsigned)(src[bidx] * BY15 + src[1] * GY15 + src[bidx^2] * RY15), gray_shift);
        }

        template <int bidx> static __device__ __forceinline__ uchar RGB2GrayConvert(uint src)
        {
            uint b = 0xffu & (src >> (bidx * 8));
            uint g = 0xffu & (src >> 8);
            uint r = 0xffu & (src >> ((bidx ^ 2) * 8));
            return CV_DESCALE((uint)(b * BY15 + g * GY15 + r * RY15), gray_shift);
        }

        template <int bidx> static __device__ __forceinline__ float RGB2GrayConvert(const float* src)
        {
            return src[bidx] * B2YF + src[1] * G2YF + src[bidx^2] * R2YF;
        }

        template <typename T, int scn, int bidx> struct RGB2Gray : unary_function<typename TypeVec<T, scn>::vec_type, T>
        {
            __device__ __forceinline__ T operator()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                return RGB2GrayConvert<bidx>(&src.x);
            }
            __host__ __device__ __forceinline__ RGB2Gray() {}
            __host__ __device__ __forceinline__ RGB2Gray(const RGB2Gray&) {}
        };

        template <int bidx> struct RGB2Gray<uchar, 4, bidx> : unary_function<uint, uchar>
        {
            __device__ __forceinline__ uchar operator()(uint src) const
            {
                return RGB2GrayConvert<bidx>(src);
            }
            __host__ __device__ __forceinline__ RGB2Gray() {}
            __host__ __device__ __forceinline__ RGB2Gray(const RGB2Gray&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB2GRAY_TRAITS(name, scn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2Gray<T, scn, bidx> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

///////////////////////////////////// RGB <-> YUV //////////////////////////////////////

    namespace color_detail
    {
        __constant__ float c_RGB2YUVCoeffs_f[5] = { B2YF, G2YF, R2YF, B2UF, R2VF };
        __constant__ int   c_RGB2YUVCoeffs_i[5] = { B2Y, G2Y, R2Y, B2UI, R2VI };

        template <int bidx, typename T, typename D> static __device__ void RGB2YUVConvert(const T* src, D& dst)
        {
            const int delta = ColorChannel<T>::half() * (1 << yuv_shift);

            const int Y = CV_DESCALE(src[0] * c_RGB2YUVCoeffs_i[bidx^2] + src[1] * c_RGB2YUVCoeffs_i[1] + src[2] * c_RGB2YUVCoeffs_i[bidx], yuv_shift);
            const int Cr = CV_DESCALE((src[bidx^2] - Y) * c_RGB2YUVCoeffs_i[3] + delta, yuv_shift);
            const int Cb = CV_DESCALE((src[bidx] - Y) * c_RGB2YUVCoeffs_i[4] + delta, yuv_shift);

            dst.x = saturate_cast<T>(Y);
            dst.y = saturate_cast<T>(Cr);
            dst.z = saturate_cast<T>(Cb);
        }

        template <int bidx, typename D> static __device__ __forceinline__ void RGB2YUVConvert(const float* src, D& dst)
        {
            dst.x = src[0] * c_RGB2YUVCoeffs_f[bidx^2] + src[1] * c_RGB2YUVCoeffs_f[1] + src[2] * c_RGB2YUVCoeffs_f[bidx];
            dst.y = (src[bidx^2] - dst.x) * c_RGB2YUVCoeffs_f[3] + ColorChannel<float>::half();
            dst.z = (src[bidx] - dst.x) * c_RGB2YUVCoeffs_f[4] + ColorChannel<float>::half();
        }

        template <typename T, int scn, int dcn, int bidx> struct RGB2YUV
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator ()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;
                RGB2YUVConvert<bidx>(&src.x, dst);
                return dst;
            }
            __host__ __device__ __forceinline__ RGB2YUV() {}
            __host__ __device__ __forceinline__ RGB2YUV(const RGB2YUV&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB2YUV_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2YUV<T, scn, dcn, bidx> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

    namespace color_detail
    {
        __constant__ float c_YUV2RGBCoeffs_f[5] = { U2BF, U2GF, V2GF, V2RF };
        __constant__ int   c_YUV2RGBCoeffs_i[5] = { U2BI, U2GI, V2GI, V2RI };

        template <int bidx, typename T, typename D> static __device__ void YUV2RGBConvert(const T& src, D* dst)
        {
            const int b = src.x + CV_DESCALE((src.z - ColorChannel<D>::half()) * c_YUV2RGBCoeffs_i[3], yuv_shift);

            const int g = src.x + CV_DESCALE((src.z - ColorChannel<D>::half()) * c_YUV2RGBCoeffs_i[2]
                                             + (src.y - ColorChannel<D>::half()) * c_YUV2RGBCoeffs_i[1], yuv_shift);

            const int r = src.x + CV_DESCALE((src.y - ColorChannel<D>::half()) * c_YUV2RGBCoeffs_i[0], yuv_shift);

            dst[bidx] = saturate_cast<D>(b);
            dst[1] = saturate_cast<D>(g);
            dst[bidx^2] = saturate_cast<D>(r);
        }

        template <int bidx> static __device__ uint YUV2RGBConvert(uint src)
        {
            const int x = 0xff & (src);
            const int y = 0xff & (src >> 8);
            const int z = 0xff & (src >> 16);

            const int b = x + CV_DESCALE((z - ColorChannel<uchar>::half()) * c_YUV2RGBCoeffs_i[3], yuv_shift);

            const int g = x + CV_DESCALE((z - ColorChannel<uchar>::half()) * c_YUV2RGBCoeffs_i[2]
                                         + (y - ColorChannel<uchar>::half()) * c_YUV2RGBCoeffs_i[1], yuv_shift);

            const int r = x + CV_DESCALE((y - ColorChannel<uchar>::half()) * c_YUV2RGBCoeffs_i[0], yuv_shift);

            uint dst = 0xffu << 24;

            dst |= saturate_cast<uchar>(b) << (bidx * 8);
            dst |= saturate_cast<uchar>(g) << 8;
            dst |= saturate_cast<uchar>(r) << ((bidx ^ 2) * 8);

            return dst;
        }

        template <int bidx, typename T> static __device__ __forceinline__ void YUV2RGBConvert(const T& src, float* dst)
        {
            dst[bidx] = src.x + (src.z - ColorChannel<float>::half()) * c_YUV2RGBCoeffs_f[3];

            dst[1] = src.x + (src.z - ColorChannel<float>::half()) * c_YUV2RGBCoeffs_f[2]
                     + (src.y - ColorChannel<float>::half()) * c_YUV2RGBCoeffs_f[1];

            dst[bidx^2] = src.x + (src.y - ColorChannel<float>::half()) * c_YUV2RGBCoeffs_f[0];
        }

        template <typename T, int scn, int dcn, int bidx> struct YUV2RGB
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator ()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                YUV2RGBConvert<bidx>(src, &dst.x);
                setAlpha(dst, ColorChannel<T>::max());

                return dst;
            }
            __host__ __device__ __forceinline__ YUV2RGB() {}
            __host__ __device__ __forceinline__ YUV2RGB(const YUV2RGB&) {}
        };

        template <int bidx> struct YUV2RGB<uchar, 4, 4, bidx> : unary_function<uint, uint>
        {
            __device__ __forceinline__ uint operator ()(uint src) const
            {
                return YUV2RGBConvert<bidx>(src);
            }
            __host__ __device__ __forceinline__ YUV2RGB() {}
            __host__ __device__ __forceinline__ YUV2RGB(const YUV2RGB&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_YUV2RGB_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::YUV2RGB<T, scn, dcn, bidx> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

///////////////////////////////////// RGB <-> YCrCb //////////////////////////////////////

    namespace color_detail
    {
        __constant__ float c_RGB2YCrCbCoeffs_f[5] = {R2YF, G2YF, B2YF, YCRF, YCBF};
        __constant__ int   c_RGB2YCrCbCoeffs_i[5] = {R2Y, G2Y, B2Y, YCRI, YCBI};

        template <int bidx, typename T, typename D> static __device__ void RGB2YCrCbConvert(const T* src, D& dst)
        {
            const int delta = ColorChannel<T>::half() * (1 << yuv_shift);

            const int Y = CV_DESCALE(src[0] * c_RGB2YCrCbCoeffs_i[bidx^2] + src[1] * c_RGB2YCrCbCoeffs_i[1] + src[2] * c_RGB2YCrCbCoeffs_i[bidx], yuv_shift);
            const int Cr = CV_DESCALE((src[bidx^2] - Y) * c_RGB2YCrCbCoeffs_i[3] + delta, yuv_shift);
            const int Cb = CV_DESCALE((src[bidx] - Y) * c_RGB2YCrCbCoeffs_i[4] + delta, yuv_shift);

            dst.x = saturate_cast<T>(Y);
            dst.y = saturate_cast<T>(Cr);
            dst.z = saturate_cast<T>(Cb);
        }

        template <int bidx> static __device__ uint RGB2YCrCbConvert(uint src)
        {
            const int delta = ColorChannel<uchar>::half() * (1 << yuv_shift);

            const int Y = CV_DESCALE((0xffu & src) * c_RGB2YCrCbCoeffs_i[bidx^2] + (0xffu & (src >> 8)) * c_RGB2YCrCbCoeffs_i[1] + (0xffu & (src >> 16)) * c_RGB2YCrCbCoeffs_i[bidx], yuv_shift);
            const int Cr = CV_DESCALE(((0xffu & (src >> ((bidx ^ 2) * 8))) - Y) * c_RGB2YCrCbCoeffs_i[3] + delta, yuv_shift);
            const int Cb = CV_DESCALE(((0xffu & (src >> (bidx * 8))) - Y) * c_RGB2YCrCbCoeffs_i[4] + delta, yuv_shift);

            uint dst = 0;

            dst |= saturate_cast<uchar>(Y);
            dst |= saturate_cast<uchar>(Cr) << 8;
            dst |= saturate_cast<uchar>(Cb) << 16;

            return dst;
        }

        template <int bidx, typename D> static __device__ __forceinline__ void RGB2YCrCbConvert(const float* src, D& dst)
        {
            dst.x = src[0] * c_RGB2YCrCbCoeffs_f[bidx^2] + src[1] * c_RGB2YCrCbCoeffs_f[1] + src[2] * c_RGB2YCrCbCoeffs_f[bidx];
            dst.y = (src[bidx^2] - dst.x) * c_RGB2YCrCbCoeffs_f[3] + ColorChannel<float>::half();
            dst.z = (src[bidx] - dst.x) * c_RGB2YCrCbCoeffs_f[4] + ColorChannel<float>::half();
        }

        template <typename T, int scn, int dcn, int bidx> struct RGB2YCrCb
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator ()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;
                RGB2YCrCbConvert<bidx>(&src.x, dst);
                return dst;
            }
            __host__ __device__ __forceinline__ RGB2YCrCb() {}
            __host__ __device__ __forceinline__ RGB2YCrCb(const RGB2YCrCb&) {}
        };

        template <int bidx> struct RGB2YCrCb<uchar, 4, 4, bidx> : unary_function<uint, uint>
        {
            __device__ __forceinline__ uint operator ()(uint src) const
            {
                return RGB2YCrCbConvert<bidx>(src);
            }

            __host__ __device__ __forceinline__ RGB2YCrCb() {}
            __host__ __device__ __forceinline__ RGB2YCrCb(const RGB2YCrCb&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB2YCrCb_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2YCrCb<T, scn, dcn, bidx> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

    namespace color_detail
    {
        __constant__ float c_YCrCb2RGBCoeffs_f[5] = {CR2RF, CR2GF, CB2GF, CB2BF};
        __constant__ int   c_YCrCb2RGBCoeffs_i[5] = {CR2RI, CR2GI, CB2GI, CB2BI};

        template <int bidx, typename T, typename D> static __device__ void YCrCb2RGBConvert(const T& src, D* dst)
        {
            const int b = src.x + CV_DESCALE((src.z - ColorChannel<D>::half()) * c_YCrCb2RGBCoeffs_i[3], yuv_shift);
            const int g = src.x + CV_DESCALE((src.z - ColorChannel<D>::half()) * c_YCrCb2RGBCoeffs_i[2] + (src.y - ColorChannel<D>::half()) * c_YCrCb2RGBCoeffs_i[1], yuv_shift);
            const int r = src.x + CV_DESCALE((src.y - ColorChannel<D>::half()) * c_YCrCb2RGBCoeffs_i[0], yuv_shift);

            dst[bidx] = saturate_cast<D>(b);
            dst[1] = saturate_cast<D>(g);
            dst[bidx^2] = saturate_cast<D>(r);
        }

        template <int bidx> static __device__ uint YCrCb2RGBConvert(uint src)
        {
            const int x = 0xff & (src);
            const int y = 0xff & (src >> 8);
            const int z = 0xff & (src >> 16);

            const int b = x + CV_DESCALE((z - ColorChannel<uchar>::half()) * c_YCrCb2RGBCoeffs_i[3], yuv_shift);
            const int g = x + CV_DESCALE((z - ColorChannel<uchar>::half()) * c_YCrCb2RGBCoeffs_i[2] + (y - ColorChannel<uchar>::half()) * c_YCrCb2RGBCoeffs_i[1], yuv_shift);
            const int r = x + CV_DESCALE((y - ColorChannel<uchar>::half()) * c_YCrCb2RGBCoeffs_i[0], yuv_shift);

            uint dst = 0xffu << 24;

            dst |= saturate_cast<uchar>(b) << (bidx * 8);
            dst |= saturate_cast<uchar>(g) << 8;
            dst |= saturate_cast<uchar>(r) << ((bidx ^ 2) * 8);

            return dst;
        }

        template <int bidx, typename T> __device__ __forceinline__ void YCrCb2RGBConvert(const T& src, float* dst)
        {
            dst[bidx] = src.x + (src.z - ColorChannel<float>::half()) * c_YCrCb2RGBCoeffs_f[3];
            dst[1] = src.x + (src.z - ColorChannel<float>::half()) * c_YCrCb2RGBCoeffs_f[2] + (src.y - ColorChannel<float>::half()) * c_YCrCb2RGBCoeffs_f[1];
            dst[bidx^2] = src.x + (src.y - ColorChannel<float>::half()) * c_YCrCb2RGBCoeffs_f[0];
        }

        template <typename T, int scn, int dcn, int bidx> struct YCrCb2RGB
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator ()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                YCrCb2RGBConvert<bidx>(src, &dst.x);
                setAlpha(dst, ColorChannel<T>::max());

                return dst;
            }
            __host__ __device__ __forceinline__ YCrCb2RGB() {}
            __host__ __device__ __forceinline__ YCrCb2RGB(const YCrCb2RGB&) {}
        };

        template <int bidx> struct YCrCb2RGB<uchar, 4, 4, bidx> : unary_function<uint, uint>
        {
            __device__ __forceinline__ uint operator ()(uint src) const
            {
                return YCrCb2RGBConvert<bidx>(src);
            }
            __host__ __device__ __forceinline__ YCrCb2RGB() {}
            __host__ __device__ __forceinline__ YCrCb2RGB(const YCrCb2RGB&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_YCrCb2RGB_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::YCrCb2RGB<T, scn, dcn, bidx> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

////////////////////////////////////// RGB <-> XYZ ///////////////////////////////////////

    namespace color_detail
    {
        __constant__ float c_RGB2XYZ_D65f[9] = { 0.412453f, 0.357580f, 0.180423f, 0.212671f, 0.715160f, 0.072169f, 0.019334f, 0.119193f, 0.950227f };
        __constant__ int   c_RGB2XYZ_D65i[9] = { 1689, 1465, 739, 871, 2929, 296, 79, 488, 3892 };

        template <int bidx, typename T, typename D> static __device__ __forceinline__ void RGB2XYZConvert(const T* src, D& dst)
        {
            dst.z = saturate_cast<T>(CV_DESCALE(src[bidx^2] * c_RGB2XYZ_D65i[6] + src[1] * c_RGB2XYZ_D65i[7] + src[bidx] * c_RGB2XYZ_D65i[8], xyz_shift));
            dst.x = saturate_cast<T>(CV_DESCALE(src[bidx^2] * c_RGB2XYZ_D65i[0] + src[1] * c_RGB2XYZ_D65i[1] + src[bidx] * c_RGB2XYZ_D65i[2], xyz_shift));
            dst.y = saturate_cast<T>(CV_DESCALE(src[bidx^2] * c_RGB2XYZ_D65i[3] + src[1] * c_RGB2XYZ_D65i[4] + src[bidx] * c_RGB2XYZ_D65i[5], xyz_shift));
        }

        template <int bidx> static __device__ __forceinline__ uint RGB2XYZConvert(uint src)
        {
            const uint b = 0xffu & (src >> (bidx * 8));
            const uint g = 0xffu & (src >> 8);
            const uint r = 0xffu & (src >> ((bidx ^ 2) * 8));

            const uint x = saturate_cast<uchar>(CV_DESCALE(r * c_RGB2XYZ_D65i[0] + g * c_RGB2XYZ_D65i[1] + b * c_RGB2XYZ_D65i[2], xyz_shift));
            const uint y = saturate_cast<uchar>(CV_DESCALE(r * c_RGB2XYZ_D65i[3] + g * c_RGB2XYZ_D65i[4] + b * c_RGB2XYZ_D65i[5], xyz_shift));
            const uint z = saturate_cast<uchar>(CV_DESCALE(r * c_RGB2XYZ_D65i[6] + g * c_RGB2XYZ_D65i[7] + b * c_RGB2XYZ_D65i[8], xyz_shift));

            uint dst = 0;

            dst |= x;
            dst |= y << 8;
            dst |= z << 16;

            return dst;
        }

        template <int bidx, typename D> static __device__ __forceinline__ void RGB2XYZConvert(const float* src, D& dst)
        {
            dst.x = src[bidx^2] * c_RGB2XYZ_D65f[0] + src[1] * c_RGB2XYZ_D65f[1] + src[bidx] * c_RGB2XYZ_D65f[2];
            dst.y = src[bidx^2] * c_RGB2XYZ_D65f[3] + src[1] * c_RGB2XYZ_D65f[4] + src[bidx] * c_RGB2XYZ_D65f[5];
            dst.z = src[bidx^2] * c_RGB2XYZ_D65f[6] + src[1] * c_RGB2XYZ_D65f[7] + src[bidx] * c_RGB2XYZ_D65f[8];
        }

        template <typename T, int scn, int dcn, int bidx> struct RGB2XYZ
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                RGB2XYZConvert<bidx>(&src.x, dst);

                return dst;
            }
            __host__ __device__ __forceinline__ RGB2XYZ() {}
            __host__ __device__ __forceinline__ RGB2XYZ(const RGB2XYZ&) {}
        };

        template <int bidx> struct RGB2XYZ<uchar, 4, 4, bidx> : unary_function<uint, uint>
        {
            __device__ __forceinline__ uint operator()(uint src) const
            {
                return RGB2XYZConvert<bidx>(src);
            }
            __host__ __device__ __forceinline__ RGB2XYZ() {}
            __host__ __device__ __forceinline__ RGB2XYZ(const RGB2XYZ&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB2XYZ_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2XYZ<T, scn, dcn, bidx> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

    namespace color_detail
    {
        __constant__ float c_XYZ2sRGB_D65f[9] = { 3.240479f, -1.53715f, -0.498535f, -0.969256f, 1.875991f, 0.041556f, 0.055648f, -0.204043f, 1.057311f };
        __constant__ int   c_XYZ2sRGB_D65i[9] = { 13273, -6296, -2042, -3970, 7684, 170, 228, -836, 4331 };

        template <int bidx, typename T, typename D> static __device__ __forceinline__ void XYZ2RGBConvert(const T& src, D* dst)
        {
            dst[bidx^2] = saturate_cast<D>(CV_DESCALE(src.x * c_XYZ2sRGB_D65i[0] + src.y * c_XYZ2sRGB_D65i[1] + src.z * c_XYZ2sRGB_D65i[2], xyz_shift));
            dst[1]      = saturate_cast<D>(CV_DESCALE(src.x * c_XYZ2sRGB_D65i[3] + src.y * c_XYZ2sRGB_D65i[4] + src.z * c_XYZ2sRGB_D65i[5], xyz_shift));
            dst[bidx]   = saturate_cast<D>(CV_DESCALE(src.x * c_XYZ2sRGB_D65i[6] + src.y * c_XYZ2sRGB_D65i[7] + src.z * c_XYZ2sRGB_D65i[8], xyz_shift));
        }

        template <int bidx> static __device__ __forceinline__ uint XYZ2RGBConvert(uint src)
        {
            const int x = 0xff & src;
            const int y = 0xff & (src >> 8);
            const int z = 0xff & (src >> 16);

            const uint r = saturate_cast<uchar>(CV_DESCALE(x * c_XYZ2sRGB_D65i[0] + y * c_XYZ2sRGB_D65i[1] + z * c_XYZ2sRGB_D65i[2], xyz_shift));
            const uint g = saturate_cast<uchar>(CV_DESCALE(x * c_XYZ2sRGB_D65i[3] + y * c_XYZ2sRGB_D65i[4] + z * c_XYZ2sRGB_D65i[5], xyz_shift));
            const uint b = saturate_cast<uchar>(CV_DESCALE(x * c_XYZ2sRGB_D65i[6] + y * c_XYZ2sRGB_D65i[7] + z * c_XYZ2sRGB_D65i[8], xyz_shift));

            uint dst = 0xffu << 24;

            dst |= b << (bidx * 8);
            dst |= g << 8;
            dst |= r << ((bidx ^ 2) * 8);

            return dst;
        }

        template <int bidx, typename T> static __device__ __forceinline__ void XYZ2RGBConvert(const T& src, float* dst)
        {
            dst[bidx^2] = src.x * c_XYZ2sRGB_D65f[0] + src.y * c_XYZ2sRGB_D65f[1] + src.z * c_XYZ2sRGB_D65f[2];
            dst[1]      = src.x * c_XYZ2sRGB_D65f[3] + src.y * c_XYZ2sRGB_D65f[4] + src.z * c_XYZ2sRGB_D65f[5];
            dst[bidx]   = src.x * c_XYZ2sRGB_D65f[6] + src.y * c_XYZ2sRGB_D65f[7] + src.z * c_XYZ2sRGB_D65f[8];
        }

        template <typename T, int scn, int dcn, int bidx> struct XYZ2RGB
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                XYZ2RGBConvert<bidx>(src, &dst.x);
                setAlpha(dst, ColorChannel<T>::max());

                return dst;
            }
            __host__ __device__ __forceinline__ XYZ2RGB() {}
            __host__ __device__ __forceinline__ XYZ2RGB(const XYZ2RGB&) {}
        };

        template <int bidx> struct XYZ2RGB<uchar, 4, 4, bidx> : unary_function<uint, uint>
        {
            __device__ __forceinline__ uint operator()(uint src) const
            {
                return XYZ2RGBConvert<bidx>(src);
            }
            __host__ __device__ __forceinline__ XYZ2RGB() {}
            __host__ __device__ __forceinline__ XYZ2RGB(const XYZ2RGB&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_XYZ2RGB_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::XYZ2RGB<T, scn, dcn, bidx> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

////////////////////////////////////// RGB <-> HSV ///////////////////////////////////////

    namespace color_detail
    {
        __constant__ int c_HsvDivTable   [256] = {0, 1044480, 522240, 348160, 261120, 208896, 174080, 149211, 130560, 116053, 104448, 94953, 87040, 80345, 74606, 69632, 65280, 61440, 58027, 54973, 52224, 49737, 47476, 45412, 43520, 41779, 40172, 38684, 37303, 36017, 34816, 33693, 32640, 31651, 30720, 29842, 29013, 28229, 27486, 26782, 26112, 25475, 24869, 24290, 23738, 23211, 22706, 22223, 21760, 21316, 20890, 20480, 20086, 19707, 19342, 18991, 18651, 18324, 18008, 17703, 17408, 17123, 16846, 16579, 16320, 16069, 15825, 15589, 15360, 15137, 14921, 14711, 14507, 14308, 14115, 13926, 13743, 13565, 13391, 13221, 13056, 12895, 12738, 12584, 12434, 12288, 12145, 12006, 11869, 11736, 11605, 11478, 11353, 11231, 11111, 10995, 10880, 10768, 10658, 10550, 10445, 10341, 10240, 10141, 10043, 9947, 9854, 9761, 9671, 9582, 9495, 9410, 9326, 9243, 9162, 9082, 9004, 8927, 8852, 8777, 8704, 8632, 8561, 8492, 8423, 8356, 8290, 8224, 8160, 8097, 8034, 7973, 7913, 7853, 7795, 7737, 7680, 7624, 7569, 7514, 7461, 7408, 7355, 7304, 7253, 7203, 7154, 7105, 7057, 7010, 6963, 6917, 6872, 6827, 6782, 6739, 6695, 6653, 6611, 6569, 6528, 6487, 6447, 6408, 6369, 6330, 6292, 6254, 6217, 6180, 6144, 6108, 6073, 6037, 6003, 5968, 5935, 5901, 5868, 5835, 5803, 5771, 5739, 5708, 5677, 5646, 5615, 5585, 5556, 5526, 5497, 5468, 5440, 5412, 5384, 5356, 5329, 5302, 5275, 5249, 5222, 5196, 5171, 5145, 5120, 5095, 5070, 5046, 5022, 4998, 4974, 4950, 4927, 4904, 4881, 4858, 4836, 4813, 4791, 4769, 4748, 4726, 4705, 4684, 4663, 4642, 4622, 4601, 4581, 4561, 4541, 4522, 4502, 4483, 4464, 4445, 4426, 4407, 4389, 4370, 4352, 4334, 4316, 4298, 4281, 4263, 4246, 4229, 4212, 4195, 4178, 4161, 4145, 4128, 4112, 4096};
        __constant__ int c_HsvDivTable180[256] = {0, 122880, 61440, 40960, 30720, 24576, 20480, 17554, 15360, 13653, 12288, 11171, 10240, 9452, 8777, 8192, 7680, 7228, 6827, 6467, 6144, 5851, 5585, 5343, 5120, 4915, 4726, 4551, 4389, 4237, 4096, 3964, 3840, 3724, 3614, 3511, 3413, 3321, 3234, 3151, 3072, 2997, 2926, 2858, 2793, 2731, 2671, 2614, 2560, 2508, 2458, 2409, 2363, 2318, 2276, 2234, 2194, 2156, 2119, 2083, 2048, 2014, 1982, 1950, 1920, 1890, 1862, 1834, 1807, 1781, 1755, 1731, 1707, 1683, 1661, 1638, 1617, 1596, 1575, 1555, 1536, 1517, 1499, 1480, 1463, 1446, 1429, 1412, 1396, 1381, 1365, 1350, 1336, 1321, 1307, 1293, 1280, 1267, 1254, 1241, 1229, 1217, 1205, 1193, 1182, 1170, 1159, 1148, 1138, 1127, 1117, 1107, 1097, 1087, 1078, 1069, 1059, 1050, 1041, 1033, 1024, 1016, 1007, 999, 991, 983, 975, 968, 960, 953, 945, 938, 931, 924, 917, 910, 904, 897, 890, 884, 878, 871, 865, 859, 853, 847, 842, 836, 830, 825, 819, 814, 808, 803, 798, 793, 788, 783, 778, 773, 768, 763, 759, 754, 749, 745, 740, 736, 731, 727, 723, 719, 714, 710, 706, 702, 698, 694, 690, 686, 683, 679, 675, 671, 668, 664, 661, 657, 654, 650, 647, 643, 640, 637, 633, 630, 627, 624, 621, 617, 614, 611, 608, 605, 602, 599, 597, 594, 591, 588, 585, 582, 580, 577, 574, 572, 569, 566, 564, 561, 559, 556, 554, 551, 549, 546, 544, 541, 539, 537, 534, 532, 530, 527, 525, 523, 521, 518, 516, 514, 512, 510, 508, 506, 504, 502, 500, 497, 495, 493, 492, 490, 488, 486, 484, 482};
        __constant__ int c_HsvDivTable256[256] = {0, 174763, 87381, 58254, 43691, 34953, 29127, 24966, 21845, 19418, 17476, 15888, 14564, 13443, 12483, 11651, 10923, 10280, 9709, 9198, 8738, 8322, 7944, 7598, 7282, 6991, 6722, 6473, 6242, 6026, 5825, 5638, 5461, 5296, 5140, 4993, 4855, 4723, 4599, 4481, 4369, 4263, 4161, 4064, 3972, 3884, 3799, 3718, 3641, 3567, 3495, 3427, 3361, 3297, 3236, 3178, 3121, 3066, 3013, 2962, 2913, 2865, 2819, 2774, 2731, 2689, 2648, 2608, 2570, 2533, 2497, 2461, 2427, 2394, 2362, 2330, 2300, 2270, 2241, 2212, 2185, 2158, 2131, 2106, 2081, 2056, 2032, 2009, 1986, 1964, 1942, 1920, 1900, 1879, 1859, 1840, 1820, 1802, 1783, 1765, 1748, 1730, 1713, 1697, 1680, 1664, 1649, 1633, 1618, 1603, 1589, 1574, 1560, 1547, 1533, 1520, 1507, 1494, 1481, 1469, 1456, 1444, 1432, 1421, 1409, 1398, 1387, 1376, 1365, 1355, 1344, 1334, 1324, 1314, 1304, 1295, 1285, 1276, 1266, 1257, 1248, 1239, 1231, 1222, 1214, 1205, 1197, 1189, 1181, 1173, 1165, 1157, 1150, 1142, 1135, 1128, 1120, 1113, 1106, 1099, 1092, 1085, 1079, 1072, 1066, 1059, 1053, 1046, 1040, 1034, 1028, 1022, 1016, 1010, 1004, 999, 993, 987, 982, 976, 971, 966, 960, 955, 950, 945, 940, 935, 930, 925, 920, 915, 910, 906, 901, 896, 892, 887, 883, 878, 874, 869, 865, 861, 857, 853, 848, 844, 840, 836, 832, 828, 824, 820, 817, 813, 809, 805, 802, 798, 794, 791, 787, 784, 780, 777, 773, 770, 767, 763, 760, 757, 753, 750, 747, 744, 741, 737, 734, 731, 728, 725, 722, 719, 716, 713, 710, 708, 705, 702, 699, 696, 694, 691, 688, 685};

        template <int bidx, int hr, typename D> static __device__ void RGB2HSVConvert(const uchar* src, D& dst)
        {
            const int hsv_shift = 12;
            const int* hdiv_table = hr == 180 ? c_HsvDivTable180 : c_HsvDivTable256;

            int b = src[bidx], g = src[1], r = src[bidx^2];
            int h, s, v = b;
            int vmin = b, diff;
            int vr, vg;

            v = ::max(v, g);
            v = ::max(v, r);
            vmin = ::min(vmin, g);
            vmin = ::min(vmin, r);

            diff = v - vmin;
            vr = (v == r) * -1;
            vg = (v == g) * -1;

            s = (diff * c_HsvDivTable[v] + (1 << (hsv_shift-1))) >> hsv_shift;
            h = (vr & (g - b)) + (~vr & ((vg & (b - r + 2 * diff)) + ((~vg) & (r - g + 4 * diff))));
            h = (h * hdiv_table[diff] + (1 << (hsv_shift-1))) >> hsv_shift;
            h += (h < 0) * hr;

            dst.x = saturate_cast<uchar>(h);
            dst.y = (uchar)s;
            dst.z = (uchar)v;
        }

        template <int bidx, int hr> static __device__ uint RGB2HSVConvert(uint src)
        {
            const int hsv_shift = 12;
            const int* hdiv_table = hr == 180 ? c_HsvDivTable180 : c_HsvDivTable256;

            const int b = 0xff & (src >> (bidx * 8));
            const int g = 0xff & (src >> 8);
            const int r = 0xff & (src >> ((bidx ^ 2) * 8));

            int h, s, v = b;
            int vmin = b, diff;
            int vr, vg;

            v = ::max(v, g);
            v = ::max(v, r);
            vmin = ::min(vmin, g);
            vmin = ::min(vmin, r);

            diff = v - vmin;
            vr = (v == r) * -1;
            vg = (v == g) * -1;

            s = (diff * c_HsvDivTable[v] + (1 << (hsv_shift-1))) >> hsv_shift;
            h = (vr & (g - b)) + (~vr & ((vg & (b - r + 2 * diff)) + ((~vg) & (r - g + 4 * diff))));
            h = (h * hdiv_table[diff] + (1 << (hsv_shift-1))) >> hsv_shift;
            h += (h < 0) * hr;

            uint dst = 0;

            dst |= saturate_cast<uchar>(h);
            dst |= (0xffu & s) << 8;
            dst |= (0xffu & v) << 16;

            return dst;
        }

        template <int bidx, int hr, typename D> static __device__ void RGB2HSVConvert(const float* src, D& dst)
        {
            const float hscale = hr * (1.f / 360.f);

            float b = src[bidx], g = src[1], r = src[bidx^2];
            float h, s, v;

            float vmin, diff;

            v = vmin = r;
            v = fmax(v, g);
            v = fmax(v, b);
            vmin = fmin(vmin, g);
            vmin = fmin(vmin, b);

            diff = v - vmin;
            s = diff / (float)(::fabs(v) + numeric_limits<float>::epsilon());
            diff = (float)(60. / (diff + numeric_limits<float>::epsilon()));

            h  = (v == r) * (g - b) * diff;
            h += (v != r && v == g) * ((b - r) * diff + 120.f);
            h += (v != r && v != g) * ((r - g) * diff + 240.f);
            h += (h < 0) * 360.f;

            dst.x = h * hscale;
            dst.y = s;
            dst.z = v;
        }

        template <typename T, int scn, int dcn, int bidx, int hr> struct RGB2HSV
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                RGB2HSVConvert<bidx, hr>(&src.x, dst);

                return dst;
            }
            __host__ __device__ __forceinline__ RGB2HSV() {}
            __host__ __device__ __forceinline__ RGB2HSV(const RGB2HSV&) {}
        };

        template <int bidx, int hr> struct RGB2HSV<uchar, 4, 4, bidx, hr> : unary_function<uint, uint>
        {
            __device__ __forceinline__ uint operator()(uint src) const
            {
                return RGB2HSVConvert<bidx, hr>(src);
            }
            __host__ __device__ __forceinline__ RGB2HSV() {}
            __host__ __device__ __forceinline__ RGB2HSV(const RGB2HSV&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB2HSV_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2HSV<T, scn, dcn, bidx, 180> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <typename T> struct name ## _full_traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2HSV<T, scn, dcn, bidx, 256> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <> struct name ## _traits<float> \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2HSV<float, scn, dcn, bidx, 360> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <> struct name ## _full_traits<float> \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2HSV<float, scn, dcn, bidx, 360> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

    namespace color_detail
    {
        __constant__ int c_HsvSectorData[6][3] = { {1,3,0}, {1,0,2}, {3,0,1}, {0,2,1}, {0,1,3}, {2,1,0} };

        template <int bidx, int hr, typename T> static __device__ void HSV2RGBConvert(const T& src, float* dst)
        {
            const float hscale = 6.f / hr;

            float h = src.x, s = src.y, v = src.z;
            float b = v, g = v, r = v;

            if (s != 0)
            {
                h *= hscale;

                if( h < 0 )
                    do h += 6; while( h < 0 );
                else if( h >= 6 )
                    do h -= 6; while( h >= 6 );

                int sector = __float2int_rd(h);
                h -= sector;

                if ( (unsigned)sector >= 6u )
                {
                    sector = 0;
                    h = 0.f;
                }

                float tab[4];
                tab[0] = v;
                tab[1] = v * (1.f - s);
                tab[2] = v * (1.f - s * h);
                tab[3] = v * (1.f - s * (1.f - h));

                b = tab[c_HsvSectorData[sector][0]];
                g = tab[c_HsvSectorData[sector][1]];
                r = tab[c_HsvSectorData[sector][2]];
            }

            dst[bidx] = b;
            dst[1] = g;
            dst[bidx^2] = r;
        }

        template <int bidx, int HR, typename T> static __device__ void HSV2RGBConvert(const T& src, uchar* dst)
        {
            float3 buf;

            buf.x = src.x;
            buf.y = src.y * (1.f / 255.f);
            buf.z = src.z * (1.f / 255.f);

            HSV2RGBConvert<bidx, HR>(buf, &buf.x);

            dst[0] = saturate_cast<uchar>(buf.x * 255.f);
            dst[1] = saturate_cast<uchar>(buf.y * 255.f);
            dst[2] = saturate_cast<uchar>(buf.z * 255.f);
        }

        template <int bidx, int hr> static __device__ uint HSV2RGBConvert(uint src)
        {
            float3 buf;

            buf.x = src & 0xff;
            buf.y = ((src >> 8) & 0xff) * (1.f/255.f);
            buf.z = ((src >> 16) & 0xff) * (1.f/255.f);

            HSV2RGBConvert<bidx, hr>(buf, &buf.x);

            uint dst = 0xffu << 24;

            dst |= saturate_cast<uchar>(buf.x * 255.f);
            dst |= saturate_cast<uchar>(buf.y * 255.f) << 8;
            dst |= saturate_cast<uchar>(buf.z * 255.f) << 16;

            return dst;
        }

        template <typename T, int scn, int dcn, int bidx, int hr> struct HSV2RGB
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                HSV2RGBConvert<bidx, hr>(src, &dst.x);
                setAlpha(dst, ColorChannel<T>::max());

                return dst;
            }
            __host__ __device__ __forceinline__ HSV2RGB() {}
            __host__ __device__ __forceinline__ HSV2RGB(const HSV2RGB&) {}
        };

        template <int bidx, int hr> struct HSV2RGB<uchar, 4, 4, bidx, hr> : unary_function<uint, uint>
        {
            __device__ __forceinline__ uint operator()(uint src) const
            {
                return HSV2RGBConvert<bidx, hr>(src);
            }
            __host__ __device__ __forceinline__ HSV2RGB() {}
            __host__ __device__ __forceinline__ HSV2RGB(const HSV2RGB&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_HSV2RGB_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::HSV2RGB<T, scn, dcn, bidx, 180> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <typename T> struct name ## _full_traits \
    { \
        typedef ::cv::cuda::device::color_detail::HSV2RGB<T, scn, dcn, bidx, 255> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <> struct name ## _traits<float> \
    { \
        typedef ::cv::cuda::device::color_detail::HSV2RGB<float, scn, dcn, bidx, 360> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <> struct name ## _full_traits<float> \
    { \
        typedef ::cv::cuda::device::color_detail::HSV2RGB<float, scn, dcn, bidx, 360> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

/////////////////////////////////////// RGB <-> HLS ////////////////////////////////////////

    namespace color_detail
    {
        template <int bidx, int hr, typename D> static __device__ void RGB2HLSConvert(const float* src, D& dst)
        {
            const float hscale = hr * (1.f / 360.f);

            float b = src[bidx], g = src[1], r = src[bidx^2];
            float h = 0.f, s = 0.f, l;
            float vmin, vmax, diff;

            vmax = vmin = r;
            vmax = fmax(vmax, g);
            vmax = fmax(vmax, b);
            vmin = fmin(vmin, g);
            vmin = fmin(vmin, b);

            diff = vmax - vmin;
            l = (vmax + vmin) * 0.5f;

            if (diff > numeric_limits<float>::epsilon())
            {
                s = (l < 0.5f) * diff / (vmax + vmin);
                s += (l >= 0.5f) * diff / (2.0f - vmax - vmin);

                diff = 60.f / diff;

                h  = (vmax == r) * (g - b) * diff;
                h += (vmax != r && vmax == g) * ((b - r) * diff + 120.f);
                h += (vmax != r && vmax != g) * ((r - g) * diff + 240.f);
                h += (h < 0.f) * 360.f;
            }

            dst.x = h * hscale;
            dst.y = l;
            dst.z = s;
        }

        template <int bidx, int hr, typename D> static __device__ void RGB2HLSConvert(const uchar* src, D& dst)
        {
            float3 buf;

            buf.x = src[0] * (1.f / 255.f);
            buf.y = src[1] * (1.f / 255.f);
            buf.z = src[2] * (1.f / 255.f);

            RGB2HLSConvert<bidx, hr>(&buf.x, buf);

            dst.x = saturate_cast<uchar>(buf.x);
            dst.y = saturate_cast<uchar>(buf.y*255.f);
            dst.z = saturate_cast<uchar>(buf.z*255.f);
        }

        template <int bidx, int hr> static __device__ uint RGB2HLSConvert(uint src)
        {
            float3 buf;

            buf.x = (0xff & src) * (1.f / 255.f);
            buf.y = (0xff & (src >> 8)) * (1.f / 255.f);
            buf.z = (0xff & (src >> 16)) * (1.f / 255.f);

            RGB2HLSConvert<bidx, hr>(&buf.x, buf);

            uint dst = 0xffu << 24;

            dst |= saturate_cast<uchar>(buf.x);
            dst |= saturate_cast<uchar>(buf.y * 255.f) << 8;
            dst |= saturate_cast<uchar>(buf.z * 255.f) << 16;

            return dst;
        }

        template <typename T, int scn, int dcn, int bidx, int hr> struct RGB2HLS
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                RGB2HLSConvert<bidx, hr>(&src.x, dst);

                return dst;
            }
            __host__ __device__ __forceinline__ RGB2HLS() {}
            __host__ __device__ __forceinline__ RGB2HLS(const RGB2HLS&) {}
        };

        template <int bidx, int hr> struct RGB2HLS<uchar, 4, 4, bidx, hr> : unary_function<uint, uint>
        {
            __device__ __forceinline__ uint operator()(uint src) const
            {
                return RGB2HLSConvert<bidx, hr>(src);
            }
            __host__ __device__ __forceinline__ RGB2HLS() {}
            __host__ __device__ __forceinline__ RGB2HLS(const RGB2HLS&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_RGB2HLS_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2HLS<T, scn, dcn, bidx, 180> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <typename T> struct name ## _full_traits \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2HLS<T, scn, dcn, bidx, 256> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <> struct name ## _traits<float> \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2HLS<float, scn, dcn, bidx, 360> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <> struct name ## _full_traits<float> \
    { \
        typedef ::cv::cuda::device::color_detail::RGB2HLS<float, scn, dcn, bidx, 360> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

    namespace color_detail
    {
        __constant__ int c_HlsSectorData[6][3] = { {1,3,0}, {1,0,2}, {3,0,1}, {0,2,1}, {0,1,3}, {2,1,0} };

        template <int bidx, int hr, typename T> static __device__ void HLS2RGBConvert(const T& src, float* dst)
        {
            const float hscale = 6.0f / hr;

            float h = src.x, l = src.y, s = src.z;
            float b = l, g = l, r = l;

            if (s != 0)
            {
                float p2  = (l <= 0.5f) * l * (1 + s);
                      p2 += (l > 0.5f) * (l + s - l * s);
                float p1 = 2 * l - p2;

                h *= hscale;

                if( h < 0 )
                    do h += 6; while( h < 0 );
                else if( h >= 6 )
                    do h -= 6; while( h >= 6 );

                int sector;
                sector = __float2int_rd(h);

                h -= sector;

                float tab[4];
                tab[0] = p2;
                tab[1] = p1;
                tab[2] = p1 + (p2 - p1) * (1 - h);
                tab[3] = p1 + (p2 - p1) * h;

                b = tab[c_HlsSectorData[sector][0]];
                g = tab[c_HlsSectorData[sector][1]];
                r = tab[c_HlsSectorData[sector][2]];
            }

            dst[bidx] = b;
            dst[1] = g;
            dst[bidx^2] = r;
        }

        template <int bidx, int hr, typename T> static __device__ void HLS2RGBConvert(const T& src, uchar* dst)
        {
            float3 buf;

            buf.x = src.x;
            buf.y = src.y * (1.f / 255.f);
            buf.z = src.z * (1.f / 255.f);

            HLS2RGBConvert<bidx, hr>(buf, &buf.x);

            dst[0] = saturate_cast<uchar>(buf.x * 255.f);
            dst[1] = saturate_cast<uchar>(buf.y * 255.f);
            dst[2] = saturate_cast<uchar>(buf.z * 255.f);
        }

        template <int bidx, int hr> static __device__ uint HLS2RGBConvert(uint src)
        {
            float3 buf;

            buf.x = 0xff & src;
            buf.y = (0xff & (src >> 8)) * (1.f / 255.f);
            buf.z = (0xff & (src >> 16)) * (1.f / 255.f);

            HLS2RGBConvert<bidx, hr>(buf, &buf.x);

            uint dst = 0xffu << 24;

            dst |= saturate_cast<uchar>(buf.x * 255.f);
            dst |= saturate_cast<uchar>(buf.y * 255.f) << 8;
            dst |= saturate_cast<uchar>(buf.z * 255.f) << 16;

            return dst;
        }

        template <typename T, int scn, int dcn, int bidx, int hr> struct HLS2RGB
            : unary_function<typename TypeVec<T, scn>::vec_type, typename TypeVec<T, dcn>::vec_type>
        {
            __device__ __forceinline__ typename TypeVec<T, dcn>::vec_type operator()(const typename TypeVec<T, scn>::vec_type& src) const
            {
                typename TypeVec<T, dcn>::vec_type dst;

                HLS2RGBConvert<bidx, hr>(src, &dst.x);
                setAlpha(dst, ColorChannel<T>::max());

                return dst;
            }
            __host__ __device__ __forceinline__ HLS2RGB() {}
            __host__ __device__ __forceinline__ HLS2RGB(const HLS2RGB&) {}
        };

        template <int bidx, int hr> struct HLS2RGB<uchar, 4, 4, bidx, hr> : unary_function<uint, uint>
        {
            __device__ __forceinline__ uint operator()(uint src) const
            {
                return HLS2RGBConvert<bidx, hr>(src);
            }
            __host__ __device__ __forceinline__ HLS2RGB() {}
            __host__ __device__ __forceinline__ HLS2RGB(const HLS2RGB&) {}
        };
    }

#define OPENCV_CUDA_IMPLEMENT_HLS2RGB_TRAITS(name, scn, dcn, bidx) \
    template <typename T> struct name ## _traits \
    { \
        typedef ::cv::cuda::device::color_detail::HLS2RGB<T, scn, dcn, bidx, 180> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <typename T> struct name ## _full_traits \
    { \
        typedef ::cv::cuda::device::color_detail::HLS2RGB<T, scn, dcn, bidx, 255> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <> struct name ## _traits<float> \
    { \
        typedef ::cv::cuda::device::color_detail::HLS2RGB<float, scn, dcn, bidx, 360> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    }; \
    template <> struct name ## _full_traits<float> \
    { \
        typedef ::cv::cuda::device::color_detail::HLS2RGB<float, scn, dcn, bidx, 360> functor_type; \
        static __host__ __device__ __forceinline__ functor_type create_functor() \
        { \
            return functor_type(); \
        } \
    };

///////////////////////////////////// RGB <-> Lab /////////////////////////////////////

    namespace color_detail
    {
        enum
        {
            LAB_CBRT_TAB_SIZE = 1024,
            GAMMA_TAB_SIZE = 1024,
            lab_shift = xyz_shift,
            gamma_shift = 3,
            lab_shift2 = (lab_shift + gamma_shift),
            LAB_CBRT_TAB_SIZE_B = (256 * 3 / 2 * (1 << gamma_shift))
        };

        __constant__ ushort c_sRGBGammaTab_b[] = {0,1,1,2,2,3,4,4,5,6,6,7,8,8,9,10,11,11,12,13,14,15,16,17,19,20,21,22,24,25,26,28,29,31,33,34,36,38,40,41,43,45,47,49,51,54,56,58,60,63,65,68,70,73,75,78,81,83,86,89,92,95,98,101,105,108,111,115,118,121,125,129,132,136,140,144,147,151,155,160,164,168,172,176,181,185,190,194,199,204,209,213,218,223,228,233,239,244,249,255,260,265,271,277,282,288,294,300,306,312,318,324,331,337,343,350,356,363,370,376,383,390,397,404,411,418,426,433,440,448,455,463,471,478,486,494,502,510,518,527,535,543,552,560,569,578,586,595,604,613,622,631,641,650,659,669,678,688,698,707,717,727,737,747,757,768,778,788,799,809,820,831,842,852,863,875,886,897,908,920,931,943,954,966,978,990,1002,1014,1026,1038,1050,1063,1075,1088,1101,1113,1126,1139,1152,1165,1178,1192,1205,1218,1232,1245,1259,1273,1287,1301,1315,1329,1343,1357,1372,1386,1401,1415,1430,1445,1460,1475,1490,1505,1521,1536,1551,1567,1583,1598,1614,1630,1646,1662,1678,1695,1711,1728,1744,1761,1778,1794,1811,1828,1846,1863,1880,1897,1915,1933,1950,1968,1986,2004,2022,2040};

        __device__ __forceinline__ int LabCbrt_b(int i)
        {
            float x = i * (1.f / (255.f * (1 << gamma_shift)));
            return (1 << lab_shift2) * (x < 0.008856f ? x * 7.787f + 0.13793103448275862f : ::cbrtf(x));
        }

        template <bool srgb, int blueIdx, typename T, typename D>
        __device__ __forceinline__ void RGB2LabConvert_b(const T& src, D& dst)
        {
            const int Lscale = (116 * 255 + 50) / 100;
            const int Lshift = -((16 * 255 * (1 << lab_shift2) + 50) / 100);

            int B = blueIdx == 0 ? src.x : src.z;
            int G = src.y;
            int R = blueIdx == 0 ? src.z : src.x;

            if (srgb)
            {
                B = c_sRGBGammaTab_b[B];
                G = c_sRGBGammaTab_b[G];
                R = c_sRGBGammaTab_b[R];
            }
            else
            {
                B <<= 3;
                G <<= 3;
                R <<= 3;
            }

            int fX = LabCbrt_b(CV_DESCALE(B * 778 + G * 1541 + R * 1777, lab_shift));
            int fY = LabCbrt_b(CV_DESCALE(B * 296 + G * 2929 + R * 871, lab_shift));
            int fZ = LabCbrt_b(CV_DESCALE(B * 3575 + G * 448 + R * 73, lab_shift));

            int L = CV_DESCALE(Lscale * fY + Lshift, lab_shift2);
            int a = CV_DESCALE(500 * (fX - fY) + 128 * (1 << lab_shift2), lab_shift2);
            int b = CV_DESCALE(200 * (fY - fZ) + 128 * (1 << lab_shift2), lab_shift2);

            dst.x = saturate_cast<uchar>(L);
            dst.y = saturate_cast<uchar>(a);
            dst.z = saturate_cast<uchar>(b);
        }

        __device__ __forceinline__ float splineInterpolate(float x, const float* tab, int n)
        {
            int ix = ::min(::max(int(x), 0), n-1);
            x -= ix;
            tab += ix * 4;
            return ((tab[3] * x + tab[2]) * x + tab[1]) * x + tab[0];
        }

        __constant__ float c_sRGBGammaTab[] = {0,7.55853e-05,0.,-7.51331e-13,7.55853e-05,7.55853e-05,-2.25399e-12,3.75665e-12,0.000151171,7.55853e-05,9.01597e-12,-6.99932e-12,0.000226756,7.55853e-05,-1.1982e-11,2.41277e-12,0.000302341,7.55853e-05,-4.74369e-12,1.19001e-11,0.000377927,7.55853e-05,3.09568e-11,-2.09095e-11,0.000453512,7.55853e-05,-3.17718e-11,1.35303e-11,0.000529097,7.55853e-05,8.81905e-12,-4.10782e-12,0.000604683,7.55853e-05,-3.50439e-12,2.90097e-12,0.000680268,7.55853e-05,5.19852e-12,-7.49607e-12,0.000755853,7.55853e-05,-1.72897e-11,2.70833e-11,0.000831439,7.55854e-05,6.39602e-11,-4.26295e-11,0.000907024,7.55854e-05,-6.39282e-11,2.70193e-11,0.000982609,7.55853e-05,1.71298e-11,-7.24017e-12,0.00105819,7.55853e-05,-4.59077e-12,1.94137e-12,0.00113378,7.55853e-05,1.23333e-12,-5.25291e-13,0.00120937,7.55853e-05,-3.42545e-13,1.59799e-13,0.00128495,7.55853e-05,1.36852e-13,-1.13904e-13,0.00136054,7.55853e-05,-2.04861e-13,2.95818e-13,0.00143612,7.55853e-05,6.82594e-13,-1.06937e-12,0.00151171,7.55853e-05,-2.52551e-12,3.98166e-12,0.00158729,7.55853e-05,9.41946e-12,-1.48573e-11,0.00166288,7.55853e-05,-3.51523e-11,5.54474e-11,0.00173846,7.55854e-05,1.3119e-10,-9.0517e-11,0.00181405,7.55854e-05,-1.40361e-10,7.37899e-11,0.00188963,7.55853e-05,8.10085e-11,-8.82272e-11,0.00196522,7.55852e-05,-1.83673e-10,1.62704e-10,0.0020408,7.55853e-05,3.04438e-10,-2.13341e-10,0.00211639,7.55853e-05,-3.35586e-10,2.25e-10,0.00219197,7.55853e-05,3.39414e-10,-2.20997e-10,0.00226756,7.55853e-05,-3.23576e-10,1.93326e-10,0.00234315,7.55853e-05,2.564e-10,-8.66446e-11,0.00241873,7.55855e-05,-3.53328e-12,-7.9578e-11,0.00249432,7.55853e-05,-2.42267e-10,1.72126e-10,0.0025699,7.55853e-05,2.74111e-10,-1.43265e-10,0.00264549,7.55854e-05,-1.55683e-10,-6.47292e-11,0.00272107,7.55849e-05,-3.4987e-10,8.67842e-10,0.00279666,7.55868e-05,2.25366e-09,-3.8723e-09,0.00287224,7.55797e-05,-9.36325e-09,1.5087e-08,0.00294783,7.56063e-05,3.58978e-08,-5.69415e-08,0.00302341,7.55072e-05,-1.34927e-07,2.13144e-07,0.003099,7.58768e-05,5.04507e-07,1.38713e-07,0.00317552,7.7302e-05,9.20646e-07,-1.55186e-07,0.00325359,7.86777e-05,4.55087e-07,4.26813e-08,0.00333276,7.97159e-05,5.83131e-07,-1.06495e-08,0.00341305,8.08502e-05,5.51182e-07,3.87467e-09,0.00349446,8.19642e-05,5.62806e-07,-1.92586e-10,0.00357698,8.30892e-05,5.62228e-07,1.0866e-09,0.00366063,8.4217e-05,5.65488e-07,5.02818e-10,0.00374542,8.53494e-05,5.66997e-07,8.60211e-10,0.00383133,8.6486e-05,5.69577e-07,7.13044e-10,0.00391839,8.76273e-05,5.71716e-07,4.78527e-10,0.00400659,8.87722e-05,5.73152e-07,1.09818e-09,0.00409594,8.99218e-05,5.76447e-07,2.50964e-10,0.00418644,9.10754e-05,5.772e-07,1.15762e-09,0.00427809,9.22333e-05,5.80672e-07,2.40865e-10,0.0043709,9.33954e-05,5.81395e-07,1.13854e-09,0.00446488,9.45616e-05,5.84811e-07,3.27267e-10,0.00456003,9.57322e-05,5.85792e-07,8.1197e-10,0.00465635,9.69062e-05,5.88228e-07,6.15823e-10,0.00475384,9.80845e-05,5.90076e-07,9.15747e-10,0.00485252,9.92674e-05,5.92823e-07,3.778e-10,0.00495238,0.000100454,5.93956e-07,8.32623e-10,0.00505343,0.000101645,5.96454e-07,4.82695e-10,0.00515567,0.000102839,5.97902e-07,9.61904e-10,0.00525911,0.000104038,6.00788e-07,3.26281e-10,0.00536375,0.00010524,6.01767e-07,9.926e-10,0.00546959,0.000106447,6.04745e-07,3.59933e-10,0.00557664,0.000107657,6.05824e-07,8.2728e-10,0.0056849,0.000108871,6.08306e-07,5.21898e-10,0.00579438,0.00011009,6.09872e-07,8.10492e-10,0.00590508,0.000111312,6.12303e-07,4.27046e-10,0.00601701,0.000112538,6.13585e-07,7.40878e-10,0.00613016,0.000113767,6.15807e-07,8.00469e-10,0.00624454,0.000115001,6.18209e-07,2.48178e-10,0.00636016,0.000116238,6.18953e-07,1.00073e-09,0.00647702,0.000117479,6.21955e-07,4.05654e-10,0.00659512,0.000118724,6.23172e-07,6.36192e-10,0.00671447,0.000119973,6.25081e-07,7.74927e-10,0.00683507,0.000121225,6.27406e-07,4.54975e-10,0.00695692,0.000122481,6.28771e-07,6.64841e-10,0.00708003,0.000123741,6.30765e-07,6.10972e-10,0.00720441,0.000125004,6.32598e-07,6.16543e-10,0.00733004,0.000126271,6.34448e-07,6.48204e-10,0.00745695,0.000127542,6.36392e-07,5.15835e-10,0.00758513,0.000128816,6.3794e-07,5.48103e-10,0.00771458,0.000130094,6.39584e-07,1.01706e-09,0.00784532,0.000131376,6.42635e-07,4.0283e-11,0.00797734,0.000132661,6.42756e-07,6.84471e-10,0.00811064,0.000133949,6.4481e-07,9.47144e-10,0.00824524,0.000135241,6.47651e-07,1.83472e-10,0.00838112,0.000136537,6.48201e-07,1.11296e-09,0.00851831,0.000137837,6.5154e-07,2.13163e-11,0.0086568,0.00013914,6.51604e-07,6.64462e-10,0.00879659,0.000140445,6.53598e-07,1.04613e-09,0.00893769,0.000141756,6.56736e-07,-1.92377e-10,0.0090801,0.000143069,6.56159e-07,1.58601e-09,0.00922383,0.000144386,6.60917e-07,-5.63754e-10,0.00936888,0.000145706,6.59226e-07,1.60033e-09,0.00951524,0.000147029,6.64027e-07,-2.49543e-10,0.00966294,0.000148356,6.63278e-07,1.26043e-09,0.00981196,0.000149687,6.67059e-07,-1.35572e-10,0.00996231,0.00015102,6.66653e-07,1.14458e-09,0.010114,0.000152357,6.70086e-07,2.13864e-10,0.010267,0.000153698,6.70728e-07,7.93856e-10,0.0104214,0.000155042,6.73109e-07,3.36077e-10,0.0105771,0.000156389,6.74118e-07,6.55765e-10,0.0107342,0.000157739,6.76085e-07,7.66211e-10,0.0108926,0.000159094,6.78384e-07,4.66116e-12,0.0110524,0.000160451,6.78398e-07,1.07775e-09,0.0112135,0.000161811,6.81631e-07,3.41023e-10,0.011376,0.000163175,6.82654e-07,3.5205e-10,0.0115398,0.000164541,6.8371e-07,1.04473e-09,0.0117051,0.000165912,6.86844e-07,1.25757e-10,0.0118717,0.000167286,6.87222e-07,3.14818e-10,0.0120396,0.000168661,6.88166e-07,1.40886e-09,0.012209,0.000170042,6.92393e-07,-3.62244e-10,0.0123797,0.000171425,6.91306e-07,9.71397e-10,0.0125518,0.000172811,6.9422e-07,2.02003e-10,0.0127253,0.0001742,6.94826e-07,1.01448e-09,0.0129002,0.000175593,6.97869e-07,3.96653e-10,0.0130765,0.00017699,6.99059e-07,1.92927e-10,0.0132542,0.000178388,6.99638e-07,6.94305e-10,0.0134333,0.00017979,7.01721e-07,7.55108e-10,0.0136138,0.000181195,7.03986e-07,1.05918e-11,0.0137957,0.000182603,7.04018e-07,1.06513e-09,0.013979,0.000184015,7.07214e-07,3.85512e-10,0.0141637,0.00018543,7.0837e-07,1.86769e-10,0.0143499,0.000186848,7.0893e-07,7.30116e-10,0.0145374,0.000188268,7.11121e-07,6.17983e-10,0.0147264,0.000189692,7.12975e-07,5.23282e-10,0.0149168,0.000191119,7.14545e-07,8.28398e-11,0.0151087,0.000192549,7.14793e-07,1.0081e-09,0.0153019,0.000193981,7.17817e-07,5.41244e-10,0.0154966,0.000195418,7.19441e-07,-3.7907e-10,0.0156928,0.000196856,7.18304e-07,1.90641e-09,0.0158903,0.000198298,7.24023e-07,-7.27387e-10,0.0160893,0.000199744,7.21841e-07,1.00317e-09,0.0162898,0.000201191,7.24851e-07,4.39949e-10,0.0164917,0.000202642,7.2617e-07,9.6234e-10,0.0166951,0.000204097,7.29057e-07,-5.64019e-10,0.0168999,0.000205554,7.27365e-07,1.29374e-09,0.0171062,0.000207012,7.31247e-07,9.77025e-10,0.017314,0.000208478,7.34178e-07,-1.47651e-09,0.0175232,0.000209942,7.29748e-07,3.06636e-09,0.0177338,0.00021141,7.38947e-07,-1.47573e-09,0.017946,0.000212884,7.3452e-07,9.7386e-10,0.0181596,0.000214356,7.37442e-07,1.30562e-09,0.0183747,0.000215835,7.41358e-07,-6.08376e-10,0.0185913,0.000217315,7.39533e-07,1.12785e-09,0.0188093,0.000218798,7.42917e-07,-1.77711e-10,0.0190289,0.000220283,7.42384e-07,1.44562e-09,0.0192499,0.000221772,7.46721e-07,-1.68825e-11,0.0194724,0.000223266,7.4667e-07,4.84533e-10,0.0196964,0.000224761,7.48124e-07,-5.85298e-11,0.0199219,0.000226257,7.47948e-07,1.61217e-09,0.0201489,0.000227757,7.52785e-07,-8.02136e-10,0.0203775,0.00022926,7.50378e-07,1.59637e-09,0.0206075,0.000230766,7.55167e-07,4.47168e-12,0.020839,0.000232276,7.55181e-07,2.48387e-10,0.021072,0.000233787,7.55926e-07,8.6474e-10,0.0213066,0.000235302,7.5852e-07,1.78299e-11,0.0215426,0.000236819,7.58573e-07,9.26567e-10,0.0217802,0.000238339,7.61353e-07,1.34529e-12,0.0220193,0.000239862,7.61357e-07,9.30659e-10,0.0222599,0.000241387,7.64149e-07,1.34529e-12,0.0225021,0.000242915,7.64153e-07,9.26567e-10,0.0227458,0.000244447,7.66933e-07,1.76215e-11,0.022991,0.00024598,7.66986e-07,8.65536e-10,0.0232377,0.000247517,7.69582e-07,2.45677e-10,0.023486,0.000249057,7.70319e-07,1.44193e-11,0.0237358,0.000250598,7.70363e-07,1.55918e-09,0.0239872,0.000252143,7.7504e-07,-6.63173e-10,0.0242401,0.000253691,7.73051e-07,1.09357e-09,0.0244946,0.000255241,7.76331e-07,1.41919e-11,0.0247506,0.000256793,7.76374e-07,7.12248e-10,0.0250082,0.000258348,7.78511e-07,8.62049e-10,0.0252673,0.000259908,7.81097e-07,-4.35061e-10,0.025528,0.000261469,7.79792e-07,8.7825e-10,0.0257902,0.000263031,7.82426e-07,6.47181e-10,0.0260541,0.000264598,7.84368e-07,2.58448e-10,0.0263194,0.000266167,7.85143e-07,1.81558e-10,0.0265864,0.000267738,7.85688e-07,8.78041e-10,0.0268549,0.000269312,7.88322e-07,3.15102e-11,0.027125,0.000270889,7.88417e-07,8.58525e-10,0.0273967,0.000272468,7.90992e-07,2.59812e-10,0.02767,0.000274051,7.91772e-07,-3.5224e-11,0.0279448,0.000275634,7.91666e-07,1.74377e-09,0.0282212,0.000277223,7.96897e-07,-1.35196e-09,0.0284992,0.000278813,7.92841e-07,1.80141e-09,0.0287788,0.000280404,7.98246e-07,-2.65629e-10,0.0290601,0.000281999,7.97449e-07,1.12374e-09,0.0293428,0.000283598,8.0082e-07,-5.04106e-10,0.0296272,0.000285198,7.99308e-07,8.92764e-10,0.0299132,0.000286799,8.01986e-07,6.58379e-10,0.0302008,0.000288405,8.03961e-07,1.98971e-10,0.0304901,0.000290014,8.04558e-07,4.08382e-10,0.0307809,0.000291624,8.05783e-07,3.01839e-11,0.0310733,0.000293236,8.05874e-07,1.33343e-09,0.0313673,0.000294851,8.09874e-07,2.2419e-10,0.031663,0.000296472,8.10547e-07,-3.67606e-10,0.0319603,0.000298092,8.09444e-07,1.24624e-09,0.0322592,0.000299714,8.13182e-07,-8.92025e-10,0.0325597,0.000301338,8.10506e-07,2.32183e-09,0.0328619,0.000302966,8.17472e-07,-9.44719e-10,0.0331657,0.000304598,8.14638e-07,1.45703e-09,0.0334711,0.000306232,8.19009e-07,-1.15805e-09,0.0337781,0.000307866,8.15535e-07,3.17507e-09,0.0340868,0.000309507,8.2506e-07,-4.09161e-09,0.0343971,0.000311145,8.12785e-07,5.74079e-09,0.0347091,0.000312788,8.30007e-07,-3.97034e-09,0.0350227,0.000314436,8.18096e-07,2.68985e-09,0.035338,0.00031608,8.26166e-07,6.61676e-10,0.0356549,0.000317734,8.28151e-07,-1.61123e-09,0.0359734,0.000319386,8.23317e-07,2.05786e-09,0.0362936,0.000321038,8.29491e-07,8.30388e-10,0.0366155,0.0003227,8.31982e-07,-1.65424e-09,0.036939,0.000324359,8.27019e-07,2.06129e-09,0.0372642,0.000326019,8.33203e-07,8.59719e-10,0.0375911,0.000327688,8.35782e-07,-1.77488e-09,0.0379196,0.000329354,8.30458e-07,2.51464e-09,0.0382498,0.000331023,8.38002e-07,-8.33135e-10,0.0385817,0.000332696,8.35502e-07,8.17825e-10,0.0389152,0.00033437,8.37956e-07,1.28718e-09,0.0392504,0.00033605,8.41817e-07,-2.2413e-09,0.0395873,0.000337727,8.35093e-07,3.95265e-09,0.0399258,0.000339409,8.46951e-07,-2.39332e-09,0.0402661,0.000341095,8.39771e-07,1.89533e-09,0.040608,0.000342781,8.45457e-07,-1.46271e-09,0.0409517,0.000344467,8.41069e-07,3.95554e-09,0.041297,0.000346161,8.52936e-07,-3.18369e-09,0.041644,0.000347857,8.43385e-07,1.32873e-09,0.0419927,0.000349548,8.47371e-07,1.59402e-09,0.0423431,0.000351248,8.52153e-07,-2.54336e-10,0.0426952,0.000352951,8.5139e-07,-5.76676e-10,0.043049,0.000354652,8.4966e-07,2.56114e-09,0.0434045,0.000356359,8.57343e-07,-2.21744e-09,0.0437617,0.000358067,8.50691e-07,2.58344e-09,0.0441206,0.000359776,8.58441e-07,-6.65826e-10,0.0444813,0.000361491,8.56444e-07,7.99218e-11,0.0448436,0.000363204,8.56684e-07,3.46063e-10,0.0452077,0.000364919,8.57722e-07,2.26116e-09,0.0455734,0.000366641,8.64505e-07,-1.94005e-09,0.045941,0.000368364,8.58685e-07,1.77384e-09,0.0463102,0.000370087,8.64007e-07,-1.43005e-09,0.0466811,0.000371811,8.59717e-07,3.94634e-09,0.0470538,0.000373542,8.71556e-07,-3.17946e-09,0.0474282,0.000375276,8.62017e-07,1.32104e-09,0.0478043,0.000377003,8.6598e-07,1.62045e-09,0.0481822,0.00037874,8.70842e-07,-3.52297e-10,0.0485618,0.000380481,8.69785e-07,-2.11211e-10,0.0489432,0.00038222,8.69151e-07,1.19716e-09,0.0493263,0.000383962,8.72743e-07,-8.52026e-10,0.0497111,0.000385705,8.70187e-07,2.21092e-09,0.0500977,0.000387452,8.76819e-07,-5.41339e-10,0.050486,0.000389204,8.75195e-07,-4.5361e-11,0.0508761,0.000390954,8.75059e-07,7.22669e-10,0.0512679,0.000392706,8.77227e-07,8.79936e-10,0.0516615,0.000394463,8.79867e-07,-5.17048e-10,0.0520568,0.000396222,8.78316e-07,1.18833e-09,0.0524539,0.000397982,8.81881e-07,-5.11022e-10,0.0528528,0.000399744,8.80348e-07,8.55683e-10,0.0532534,0.000401507,8.82915e-07,8.13562e-10,0.0536558,0.000403276,8.85356e-07,-3.84603e-10,0.05406,0.000405045,8.84202e-07,7.24962e-10,0.0544659,0.000406816,8.86377e-07,1.20986e-09,0.0548736,0.000408592,8.90006e-07,-1.83896e-09,0.0552831,0.000410367,8.84489e-07,2.42071e-09,0.0556944,0.000412143,8.91751e-07,-3.93413e-10,0.0561074,0.000413925,8.90571e-07,-8.46967e-10,0.0565222,0.000415704,8.8803e-07,3.78122e-09,0.0569388,0.000417491,8.99374e-07,-3.1021e-09,0.0573572,0.000419281,8.90068e-07,1.17658e-09,0.0577774,0.000421064,8.93597e-07,2.12117e-09,0.0581993,0.000422858,8.99961e-07,-2.21068e-09,0.0586231,0.000424651,8.93329e-07,2.9961e-09,0.0590486,0.000426447,9.02317e-07,-2.32311e-09,0.059476,0.000428244,8.95348e-07,2.57122e-09,0.0599051,0.000430043,9.03062e-07,-5.11098e-10,0.0603361,0.000431847,9.01528e-07,-5.27166e-10,0.0607688,0.000433649,8.99947e-07,2.61984e-09,0.0612034,0.000435457,9.07806e-07,-2.50141e-09,0.0616397,0.000437265,9.00302e-07,3.66045e-09,0.0620779,0.000439076,9.11283e-07,-4.68977e-09,0.0625179,0.000440885,8.97214e-07,7.64783e-09,0.0629597,0.000442702,9.20158e-07,-7.27499e-09,0.0634033,0.000444521,8.98333e-07,6.55113e-09,0.0638487,0.000446337,9.17986e-07,-4.02844e-09,0.0642959,0.000448161,9.05901e-07,2.11196e-09,0.064745,0.000449979,9.12236e-07,3.03125e-09,0.0651959,0.000451813,9.2133e-07,-6.78648e-09,0.0656486,0.000453635,9.00971e-07,9.21375e-09,0.0661032,0.000455464,9.28612e-07,-7.71684e-09,0.0665596,0.000457299,9.05462e-07,6.7522e-09,0.0670178,0.00045913,9.25718e-07,-4.3907e-09,0.0674778,0.000460968,9.12546e-07,3.36e-09,0.0679397,0.000462803,9.22626e-07,-1.59876e-09,0.0684034,0.000464644,9.1783e-07,3.0351e-09,0.068869,0.000466488,9.26935e-07,-3.09101e-09,0.0693364,0.000468333,9.17662e-07,1.8785e-09,0.0698057,0.000470174,9.23298e-07,3.02733e-09,0.0702768,0.00047203,9.3238e-07,-6.53722e-09,0.0707497,0.000473875,9.12768e-07,8.22054e-09,0.0712245,0.000475725,9.37429e-07,-3.99325e-09,0.0717012,0.000477588,9.2545e-07,3.01839e-10,0.0721797,0.00047944,9.26355e-07,2.78597e-09,0.0726601,0.000481301,9.34713e-07,-3.99507e-09,0.0731423,0.000483158,9.22728e-07,5.7435e-09,0.0736264,0.000485021,9.39958e-07,-4.07776e-09,0.0741123,0.000486888,9.27725e-07,3.11695e-09,0.0746002,0.000488753,9.37076e-07,-9.39394e-10,0.0750898,0.000490625,9.34258e-07,6.4055e-10,0.0755814,0.000492495,9.3618e-07,-1.62265e-09,0.0760748,0.000494363,9.31312e-07,5.84995e-09,0.0765701,0.000496243,9.48861e-07,-6.87601e-09,0.0770673,0.00049812,9.28233e-07,6.75296e-09,0.0775664,0.000499997,9.48492e-07,-5.23467e-09,0.0780673,0.000501878,9.32788e-07,6.73523e-09,0.0785701,0.000503764,9.52994e-07,-6.80514e-09,0.0790748,0.000505649,9.32578e-07,5.5842e-09,0.0795814,0.000507531,9.49331e-07,-6.30583e-10,0.0800899,0.000509428,9.47439e-07,-3.0618e-09,0.0806003,0.000511314,9.38254e-07,5.4273e-09,0.0811125,0.000513206,9.54536e-07,-3.74627e-09,0.0816267,0.000515104,9.43297e-07,2.10713e-09,0.0821427,0.000516997,9.49618e-07,2.76839e-09,0.0826607,0.000518905,9.57924e-07,-5.73006e-09,0.0831805,0.000520803,9.40733e-07,5.25072e-09,0.0837023,0.0005227,9.56486e-07,-3.71718e-10,0.084226,0.000524612,9.5537e-07,-3.76404e-09,0.0847515,0.000526512,9.44078e-07,7.97735e-09,0.085279,0.000528424,9.6801e-07,-5.79367e-09,0.0858084,0.000530343,9.50629e-07,2.96268e-10,0.0863397,0.000532245,9.51518e-07,4.6086e-09,0.0868729,0.000534162,9.65344e-07,-3.82947e-09,0.087408,0.000536081,9.53856e-07,3.25861e-09,0.087945,0.000537998,9.63631e-07,-1.7543e-09,0.088484,0.00053992,9.58368e-07,3.75849e-09,0.0890249,0.000541848,9.69644e-07,-5.82891e-09,0.0895677,0.00054377,9.52157e-07,4.65593e-09,0.0901124,0.000545688,9.66125e-07,2.10643e-09,0.0906591,0.000547627,9.72444e-07,-5.63099e-09,0.0912077,0.000549555,9.55551e-07,5.51627e-09,0.0917582,0.000551483,9.721e-07,-1.53292e-09,0.0923106,0.000553422,9.67501e-07,6.15311e-10,0.092865,0.000555359,9.69347e-07,-9.28291e-10,0.0934213,0.000557295,9.66562e-07,3.09774e-09,0.0939796,0.000559237,9.75856e-07,-4.01186e-09,0.0945398,0.000561177,9.6382e-07,5.49892e-09,0.095102,0.000563121,9.80317e-07,-3.08258e-09,0.0956661,0.000565073,9.71069e-07,-6.19176e-10,0.0962321,0.000567013,9.69212e-07,5.55932e-09,0.0968001,0.000568968,9.8589e-07,-6.71704e-09,0.09737,0.00057092,9.65738e-07,6.40762e-09,0.0979419,0.00057287,9.84961e-07,-4.0122e-09,0.0985158,0.000574828,9.72925e-07,2.19059e-09,0.0990916,0.000576781,9.79496e-07,2.70048e-09,0.0996693,0.000578748,9.87598e-07,-5.54193e-09,0.100249,0.000580706,9.70972e-07,4.56597e-09,0.100831,0.000582662,9.8467e-07,2.17923e-09,0.101414,0.000584638,9.91208e-07,-5.83232e-09,0.102,0.000586603,9.73711e-07,6.24884e-09,0.102588,0.000588569,9.92457e-07,-4.26178e-09,0.103177,0.000590541,9.79672e-07,3.34781e-09,0.103769,0.00059251,9.89715e-07,-1.67904e-09,0.104362,0.000594485,9.84678e-07,3.36839e-09,0.104958,0.000596464,9.94783e-07,-4.34397e-09,0.105555,0.000598441,9.81751e-07,6.55696e-09,0.106155,0.000600424,1.00142e-06,-6.98272e-09,0.106756,0.000602406,9.80474e-07,6.4728e-09,0.107359,0.000604386,9.99893e-07,-4.00742e-09,0.107965,0.000606374,9.8787e-07,2.10654e-09,0.108572,0.000608356,9.9419e-07,3.0318e-09,0.109181,0.000610353,1.00329e-06,-6.7832e-09,0.109793,0.00061234,9.82936e-07,9.1998e-09,0.110406,0.000614333,1.01054e-06,-7.6642e-09,0.111021,0.000616331,9.87543e-07,6.55579e-09,0.111639,0.000618326,1.00721e-06,-3.65791e-09,0.112258,0.000620329,9.96236e-07,6.25467e-10,0.112879,0.000622324,9.98113e-07,1.15593e-09,0.113503,0.000624323,1.00158e-06,2.20158e-09,0.114128,0.000626333,1.00819e-06,-2.51191e-09,0.114755,0.000628342,1.00065e-06,3.95517e-10,0.115385,0.000630345,1.00184e-06,9.29807e-10,0.116016,0.000632351,1.00463e-06,3.33599e-09,0.116649,0.00063437,1.01463e-06,-6.82329e-09,0.117285,0.000636379,9.94163e-07,9.05595e-09,0.117922,0.000638395,1.02133e-06,-7.04862e-09,0.118562,0.000640416,1.00019e-06,4.23737e-09,0.119203,0.000642429,1.0129e-06,-2.45033e-09,0.119847,0.000644448,1.00555e-06,5.56395e-09,0.120492,0.000646475,1.02224e-06,-4.9043e-09,0.121139,0.000648505,1.00753e-06,-8.47952e-10,0.121789,0.000650518,1.00498e-06,8.29622e-09,0.122441,0.000652553,1.02987e-06,-9.98538e-09,0.123094,0.000654582,9.99914e-07,9.2936e-09,0.12375,0.00065661,1.02779e-06,-4.83707e-09,0.124407,0.000658651,1.01328e-06,2.60411e-09,0.125067,0.000660685,1.0211e-06,-5.57945e-09,0.125729,0.000662711,1.00436e-06,1.22631e-08,0.126392,0.000664756,1.04115e-06,-1.36704e-08,0.127058,0.000666798,1.00014e-06,1.26161e-08,0.127726,0.000668836,1.03798e-06,-6.99155e-09,0.128396,0.000670891,1.01701e-06,4.48836e-10,0.129068,0.000672926,1.01836e-06,5.19606e-09,0.129742,0.000674978,1.03394e-06,-6.3319e-09,0.130418,0.000677027,1.01495e-06,5.2305e-09,0.131096,0.000679073,1.03064e-06,3.11123e-10,0.131776,0.000681135,1.03157e-06,-6.47511e-09,0.132458,0.000683179,1.01215e-06,1.06882e-08,0.133142,0.000685235,1.04421e-06,-6.47519e-09,0.133829,0.000687304,1.02479e-06,3.11237e-10,0.134517,0.000689355,1.02572e-06,5.23035e-09,0.135207,0.000691422,1.04141e-06,-6.3316e-09,0.1359,0.000693486,1.02242e-06,5.19484e-09,0.136594,0.000695546,1.038e-06,4.53497e-10,0.137291,0.000697623,1.03936e-06,-7.00891e-09,0.137989,0.000699681,1.01834e-06,1.2681e-08,0.13869,0.000701756,1.05638e-06,-1.39128e-08,0.139393,0.000703827,1.01464e-06,1.31679e-08,0.140098,0.000705896,1.05414e-06,-8.95659e-09,0.140805,0.000707977,1.02727e-06,7.75742e-09,0.141514,0.000710055,1.05055e-06,-7.17182e-09,0.142225,0.000712135,1.02903e-06,6.02862e-09,0.142938,0.000714211,1.04712e-06,-2.04163e-09,0.143653,0.000716299,1.04099e-06,2.13792e-09,0.144371,0.000718387,1.04741e-06,-6.51009e-09,0.14509,0.000720462,1.02787e-06,9.00123e-09,0.145812,0.000722545,1.05488e-06,3.07523e-10,0.146535,0.000724656,1.0558e-06,-1.02312e-08,0.147261,0.000726737,1.02511e-06,1.0815e-08,0.147989,0.000728819,1.05755e-06,-3.22681e-09,0.148719,0.000730925,1.04787e-06,2.09244e-09,0.14945,0.000733027,1.05415e-06,-5.143e-09,0.150185,0.00073512,1.03872e-06,3.57844e-09,0.150921,0.000737208,1.04946e-06,5.73027e-09,0.151659,0.000739324,1.06665e-06,-1.15983e-08,0.152399,0.000741423,1.03185e-06,1.08605e-08,0.153142,0.000743519,1.06443e-06,-2.04106e-09,0.153886,0.000745642,1.05831e-06,-2.69642e-09,0.154633,0.00074775,1.05022e-06,-2.07425e-09,0.155382,0.000749844,1.044e-06,1.09934e-08,0.156133,0.000751965,1.07698e-06,-1.20972e-08,0.156886,0.000754083,1.04069e-06,7.59288e-09,0.157641,0.000756187,1.06347e-06,-3.37305e-09,0.158398,0.000758304,1.05335e-06,5.89921e-09,0.159158,0.000760428,1.07104e-06,-5.32248e-09,0.159919,0.000762554,1.05508e-06,4.8927e-10,0.160683,0.000764666,1.05654e-06,3.36547e-09,0.161448,0.000766789,1.06664e-06,9.50081e-10,0.162216,0.000768925,1.06949e-06,-7.16568e-09,0.162986,0.000771043,1.04799e-06,1.28114e-08,0.163758,0.000773177,1.08643e-06,-1.42774e-08,0.164533,0.000775307,1.0436e-06,1.44956e-08,0.165309,0.000777438,1.08708e-06,-1.39025e-08,0.166087,0.00077957,1.04538e-06,1.13118e-08,0.166868,0.000781695,1.07931e-06,-1.54224e-09,0.167651,0.000783849,1.07468e-06,-5.14312e-09,0.168436,0.000785983,1.05925e-06,7.21381e-09,0.169223,0.000788123,1.0809e-06,-8.81096e-09,0.170012,0.000790259,1.05446e-06,1.31289e-08,0.170803,0.000792407,1.09385e-06,-1.39022e-08,0.171597,0.000794553,1.05214e-06,1.26775e-08,0.172392,0.000796695,1.09018e-06,-7.00557e-09,0.17319,0.000798855,1.06916e-06,4.43796e-10,0.17399,0.000800994,1.07049e-06,5.23031e-09,0.174792,0.000803151,1.08618e-06,-6.46397e-09,0.175596,0.000805304,1.06679e-06,5.72444e-09,0.176403,0.000807455,1.08396e-06,-1.53254e-09,0.177211,0.000809618,1.07937e-06,4.05673e-10,0.178022,0.000811778,1.08058e-06,-9.01916e-11,0.178835,0.000813939,1.08031e-06,-4.49821e-11,0.17965,0.000816099,1.08018e-06,2.70234e-10,0.180467,0.00081826,1.08099e-06,-1.03603e-09,0.181286,0.000820419,1.07788e-06,3.87392e-09,0.182108,0.000822587,1.0895e-06,4.41522e-10,0.182932,0.000824767,1.09083e-06,-5.63997e-09,0.183758,0.000826932,1.07391e-06,7.21707e-09,0.184586,0.000829101,1.09556e-06,-8.32718e-09,0.185416,0.000831267,1.07058e-06,1.11907e-08,0.186248,0.000833442,1.10415e-06,-6.63336e-09,0.187083,0.00083563,1.08425e-06,4.41484e-10,0.187919,0.0008378,1.08557e-06,4.86754e-09,0.188758,0.000839986,1.10017e-06,-5.01041e-09,0.189599,0.000842171,1.08514e-06,2.72811e-10,0.190443,0.000844342,1.08596e-06,3.91916e-09,0.191288,0.000846526,1.09772e-06,-1.04819e-09,0.192136,0.000848718,1.09457e-06,2.73531e-10,0.192985,0.000850908,1.0954e-06,-4.58916e-11,0.193837,0.000853099,1.09526e-06,-9.01158e-11,0.194692,0.000855289,1.09499e-06,4.06506e-10,0.195548,0.00085748,1.09621e-06,-1.53595e-09,0.196407,0.000859668,1.0916e-06,5.73717e-09,0.197267,0.000861869,1.10881e-06,-6.51164e-09,0.19813,0.000864067,1.08928e-06,5.40831e-09,0.198995,0.000866261,1.1055e-06,-2.20401e-10,0.199863,0.000868472,1.10484e-06,-4.52652e-09,0.200732,0.000870668,1.09126e-06,3.42508e-09,0.201604,0.000872861,1.10153e-06,5.72762e-09,0.202478,0.000875081,1.11872e-06,-1.14344e-08,0.203354,0.000877284,1.08441e-06,1.02076e-08,0.204233,0.000879484,1.11504e-06,4.06355e-10,0.205113,0.000881715,1.11626e-06,-1.18329e-08,0.205996,0.000883912,1.08076e-06,1.71227e-08,0.206881,0.000886125,1.13213e-06,-1.19546e-08,0.207768,0.000888353,1.09626e-06,8.93465e-10,0.208658,0.000890548,1.09894e-06,8.38062e-09,0.209549,0.000892771,1.12408e-06,-4.61353e-09,0.210443,0.000895006,1.11024e-06,-4.82756e-09,0.211339,0.000897212,1.09576e-06,9.02245e-09,0.212238,0.00089943,1.12283e-06,-1.45997e-09,0.213138,0.000901672,1.11845e-06,-3.18255e-09,0.214041,0.000903899,1.1089e-06,-7.11073e-10,0.214946,0.000906115,1.10677e-06,6.02692e-09,0.215853,0.000908346,1.12485e-06,-8.49548e-09,0.216763,0.00091057,1.09936e-06,1.30537e-08,0.217675,0.000912808,1.13852e-06,-1.3917e-08,0.218588,0.000915044,1.09677e-06,1.28121e-08,0.219505,0.000917276,1.13521e-06,-7.5288e-09,0.220423,0.000919523,1.11262e-06,2.40205e-09,0.221344,0.000921756,1.11983e-06,-2.07941e-09,0.222267,0.000923989,1.11359e-06,5.91551e-09,0.223192,0.000926234,1.13134e-06,-6.68149e-09,0.224119,0.000928477,1.11129e-06,5.90929e-09,0.225049,0.000930717,1.12902e-06,-2.05436e-09,0.22598,0.000932969,1.12286e-06,2.30807e-09,0.226915,0.000935222,1.12978e-06,-7.17796e-09,0.227851,0.00093746,1.10825e-06,1.15028e-08,0.228789,0.000939711,1.14276e-06,-9.03083e-09,0.22973,0.000941969,1.11566e-06,9.71932e-09,0.230673,0.00094423,1.14482e-06,-1.49452e-08,0.231619,0.000946474,1.09998e-06,2.02591e-08,0.232566,0.000948735,1.16076e-06,-2.13879e-08,0.233516,0.000950993,1.0966e-06,2.05888e-08,0.234468,0.000953247,1.15837e-06,-1.62642e-08,0.235423,0.000955515,1.10957e-06,1.46658e-08,0.236379,0.000957779,1.15357e-06,-1.25966e-08,0.237338,0.000960048,1.11578e-06,5.91793e-09,0.238299,0.000962297,1.13353e-06,3.82602e-09,0.239263,0.000964576,1.14501e-06,-6.3208e-09,0.240229,0.000966847,1.12605e-06,6.55613e-09,0.241197,0.000969119,1.14572e-06,-5.00268e-09,0.242167,0.000971395,1.13071e-06,-1.44659e-09,0.243139,0.000973652,1.12637e-06,1.07891e-08,0.244114,0.000975937,1.15874e-06,-1.19073e-08,0.245091,0.000978219,1.12302e-06,7.03782e-09,0.246071,0.000980486,1.14413e-06,-1.34276e-09,0.247052,0.00098277,1.1401e-06,-1.66669e-09,0.248036,0.000985046,1.1351e-06,8.00935e-09,0.249022,0.00098734,1.15913e-06,-1.54694e-08,0.250011,0.000989612,1.11272e-06,2.4066e-08,0.251002,0.000991909,1.18492e-06,-2.11901e-08,0.251995,0.000994215,1.12135e-06,1.08973e-09,0.25299,0.000996461,1.12462e-06,1.68311e-08,0.253988,0.000998761,1.17511e-06,-8.8094e-09,0.254987,0.00100109,1.14868e-06,-1.13958e-08,0.25599,0.00100335,1.1145e-06,2.45902e-08,0.256994,0.00100565,1.18827e-06,-2.73603e-08,0.258001,0.00100795,1.10618e-06,2.52464e-08,0.25901,0.00101023,1.18192e-06,-1.40207e-08,0.260021,0.00101256,1.13986e-06,1.03387e-09,0.261035,0.00101484,1.14296e-06,9.8853e-09,0.262051,0.00101715,1.17262e-06,-1.07726e-08,0.263069,0.00101947,1.1403e-06,3.40272e-09,0.26409,0.00102176,1.15051e-06,-2.83827e-09,0.265113,0.00102405,1.142e-06,7.95039e-09,0.266138,0.00102636,1.16585e-06,8.39047e-10,0.267166,0.00102869,1.16836e-06,-1.13066e-08,0.268196,0.00103099,1.13444e-06,1.4585e-08,0.269228,0.00103331,1.1782e-06,-1.72314e-08,0.270262,0.00103561,1.1265e-06,2.45382e-08,0.271299,0.00103794,1.20012e-06,-2.13166e-08,0.272338,0.00104028,1.13617e-06,1.12364e-09,0.273379,0.00104255,1.13954e-06,1.68221e-08,0.274423,0.00104488,1.19001e-06,-8.80736e-09,0.275469,0.00104723,1.16358e-06,-1.13948e-08,0.276518,0.00104953,1.1294e-06,2.45839e-08,0.277568,0.00105186,1.20315e-06,-2.73361e-08,0.278621,0.00105418,1.12114e-06,2.51559e-08,0.279677,0.0010565,1.19661e-06,-1.36832e-08,0.280734,0.00105885,1.15556e-06,-2.25706e-10,0.281794,0.00106116,1.15488e-06,1.45862e-08,0.282857,0.00106352,1.19864e-06,-2.83167e-08,0.283921,0.00106583,1.11369e-06,3.90759e-08,0.284988,0.00106817,1.23092e-06,-3.85801e-08,0.286058,0.00107052,1.11518e-06,2.58375e-08,0.287129,0.00107283,1.19269e-06,-5.16498e-09,0.288203,0.0010752,1.1772e-06,-5.17768e-09,0.28928,0.00107754,1.16167e-06,-3.92671e-09,0.290358,0.00107985,1.14988e-06,2.08846e-08,0.29144,0.00108221,1.21254e-06,-2.00072e-08,0.292523,0.00108458,1.15252e-06,-4.60659e-10,0.293609,0.00108688,1.15114e-06,2.18499e-08,0.294697,0.00108925,1.21669e-06,-2.73343e-08,0.295787,0.0010916,1.13468e-06,2.78826e-08,0.29688,0.00109395,1.21833e-06,-2.45915e-08,0.297975,0.00109632,1.14456e-06,1.08787e-08,0.299073,0.00109864,1.17719e-06,1.08788e-08,0.300172,0.00110102,1.20983e-06,-2.45915e-08,0.301275,0.00110337,1.13605e-06,2.78828e-08,0.302379,0.00110573,1.2197e-06,-2.73348e-08,0.303486,0.00110808,1.1377e-06,2.18518e-08,0.304595,0.00111042,1.20325e-06,-4.67556e-10,0.305707,0.00111283,1.20185e-06,-1.99816e-08,0.306821,0.00111517,1.14191e-06,2.07891e-08,0.307937,0.00111752,1.20427e-06,-3.57026e-09,0.309056,0.00111992,1.19356e-06,-6.50797e-09,0.310177,0.00112228,1.17404e-06,-2.00165e-10,0.3113,0.00112463,1.17344e-06,7.30874e-09,0.312426,0.001127,1.19536e-06,7.67424e-10,0.313554,0.00112939,1.19767e-06,-1.03784e-08,0.314685,0.00113176,1.16653e-06,1.09437e-08,0.315818,0.00113412,1.19936e-06,-3.59406e-09,0.316953,0.00113651,1.18858e-06,3.43251e-09,0.318091,0.0011389,1.19888e-06,-1.0136e-08,0.319231,0.00114127,1.16847e-06,7.30915e-09,0.320374,0.00114363,1.1904e-06,1.07018e-08,0.321518,0.00114604,1.2225e-06,-2.03137e-08,0.322666,0.00114842,1.16156e-06,1.09484e-08,0.323815,0.00115078,1.19441e-06,6.32224e-09,0.324967,0.00115319,1.21337e-06,-6.43509e-09,0.326122,0.00115559,1.19407e-06,-1.03842e-08,0.327278,0.00115795,1.16291e-06,1.81697e-08,0.328438,0.00116033,1.21742e-06,-2.6901e-09,0.329599,0.00116276,1.20935e-06,-7.40939e-09,0.330763,0.00116515,1.18713e-06,2.52533e-09,0.331929,0.00116754,1.1947e-06,-2.69191e-09,0.333098,0.00116992,1.18663e-06,8.24218e-09,0.334269,0.00117232,1.21135e-06,-4.74377e-10,0.335443,0.00117474,1.20993e-06,-6.34471e-09,0.336619,0.00117714,1.1909e-06,-3.94922e-09,0.337797,0.00117951,1.17905e-06,2.21417e-08,0.338978,0.00118193,1.24547e-06,-2.50128e-08,0.340161,0.00118435,1.17043e-06,1.8305e-08,0.341346,0.00118674,1.22535e-06,-1.84048e-08,0.342534,0.00118914,1.17013e-06,2.55121e-08,0.343725,0.00119156,1.24667e-06,-2.40389e-08,0.344917,0.00119398,1.17455e-06,1.10389e-08,0.346113,0.00119636,1.20767e-06,9.68574e-09,0.34731,0.0011988,1.23673e-06,-1.99797e-08,0.34851,0.00120122,1.17679e-06,1.06284e-08,0.349713,0.0012036,1.20867e-06,7.26868e-09,0.350917,0.00120604,1.23048e-06,-9.90072e-09,0.352125,0.00120847,1.20078e-06,2.53177e-09,0.353334,0.00121088,1.20837e-06,-2.26199e-10,0.354546,0.0012133,1.20769e-06,-1.62705e-09,0.355761,0.00121571,1.20281e-06,6.73435e-09,0.356978,0.00121813,1.22302e-06,4.49207e-09,0.358197,0.00122059,1.23649e-06,-2.47027e-08,0.359419,0.00122299,1.16238e-06,3.47142e-08,0.360643,0.00122542,1.26653e-06,-2.47472e-08,0.36187,0.00122788,1.19229e-06,4.66965e-09,0.363099,0.00123028,1.20629e-06,6.06872e-09,0.36433,0.00123271,1.2245e-06,8.57729e-10,0.365564,0.00123516,1.22707e-06,-9.49952e-09,0.366801,0.00123759,1.19858e-06,7.33792e-09,0.36804,0.00124001,1.22059e-06,9.95025e-09,0.369281,0.00124248,1.25044e-06,-1.73366e-08,0.370525,0.00124493,1.19843e-06,-2.08464e-10,0.371771,0.00124732,1.1978e-06,1.81704e-08,0.373019,0.00124977,1.25232e-06,-1.28683e-08,0.37427,0.00125224,1.21371e-06,3.50042e-09,0.375524,0.00125468,1.22421e-06,-1.1335e-09,0.37678,0.00125712,1.22081e-06,1.03345e-09,0.378038,0.00125957,1.22391e-06,-3.00023e-09,0.379299,0.00126201,1.21491e-06,1.09676e-08,0.380562,0.00126447,1.24781e-06,-1.10676e-08,0.381828,0.00126693,1.21461e-06,3.50042e-09,0.383096,0.00126937,1.22511e-06,-2.93403e-09,0.384366,0.00127181,1.21631e-06,8.23574e-09,0.385639,0.00127427,1.24102e-06,-2.06607e-10,0.386915,0.00127675,1.2404e-06,-7.40935e-09,0.388193,0.00127921,1.21817e-06,4.1761e-11,0.389473,0.00128165,1.21829e-06,7.24223e-09,0.390756,0.0012841,1.24002e-06,7.91564e-10,0.392042,0.00128659,1.2424e-06,-1.04086e-08,0.393329,0.00128904,1.21117e-06,1.10405e-08,0.39462,0.0012915,1.24429e-06,-3.951e-09,0.395912,0.00129397,1.23244e-06,4.7634e-09,0.397208,0.00129645,1.24673e-06,-1.51025e-08,0.398505,0.0012989,1.20142e-06,2.58443e-08,0.399805,0.00130138,1.27895e-06,-2.86702e-08,0.401108,0.00130385,1.19294e-06,2.92318e-08,0.402413,0.00130632,1.28064e-06,-2.86524e-08,0.403721,0.0013088,1.19468e-06,2.5
... [Content truncated - file is very large]
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **RGB2XYZ**: A class/struct defined in this file
- **RGB5x52GrayConverter**: A class/struct defined in this file
- **RGB5x52RGBConverter**: A class/struct defined in this file
- **Gray2RGB5x5Converter**: A class/struct defined in this file
- **YUV2RGB**: A class/struct defined in this file
- **Gray2RGB5x5**: A class/struct defined in this file
- **RGB2YUV**: A class/struct defined in this file
- **ColorChannel**: A class/struct defined in this file
- **Gray2RGB**: A class/struct defined in this file
- **YCrCb2RGB**: A class/struct defined in this file
- **RGB2RGB5x5Converter**: A class/struct defined in this file
- **XYZ2RGB**: A class/struct defined in this file
- **RGB5x52RGB**: A class/struct defined in this file
- **name**: A class/struct defined in this file
- **RGB2YCrCb**: A class/struct defined in this file
- **RGB2RGB5x5**: A class/struct defined in this file
- **RGB2RGB**: A class/struct defined in this file
- **RGB2Gray**: A class/struct defined in this file
- **RGB5x52Gray**: A class/struct defined in this file

### Functions and Methods

- **CV_DESCALE()**: A function/method defined in this file
- **float()**: A function/method defined in this file
- **OPENCV_CUDA_COLOR_DETAIL_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../saturate_cast.hpp`
- `../vec_traits.hpp`
- `../common.hpp`
- `../functional.hpp`
- `../limits.hpp`

**Python Imports:**
- `YUV`
- `24`
- `YCrCb`
- `this`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.

