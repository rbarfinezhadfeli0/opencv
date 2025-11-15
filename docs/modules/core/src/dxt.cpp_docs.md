# Documentation for `modules/core/src/dxt.cpp`

## File Metadata

- **Full Path**: `modules/core/src/dxt.cpp`
- **File Name**: `dxt.cpp`
- **File Size**: 156,954 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/src/dxt.cpp](../../../modules/core/src/dxt.cpp)

## Purpose and Role

This file is located in the `modules/core/src` directory and serves as part of the OpenCV library infrastructure.

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
//                        Intel License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000, Intel Corporation, all rights reserved.
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
//   * The name of Intel Corporation may not be used to endorse or promote products
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

#include "precomp.hpp"
#include "opencv2/core/opencl/runtime/opencl_clfft.hpp"
#include "opencv2/core/opencl/runtime/opencl_core.hpp"
#include "opencl_kernels_core.hpp"
#include <map>

namespace cv
{

// On Win64 optimized versions of DFT and DCT fail the tests (fixed in VS2010)
#if defined _MSC_VER && !defined CV_ICC && defined _M_X64 && _MSC_VER < 1600
# pragma optimize("", off)
# pragma warning(disable: 4748)
#endif

#if IPP_VERSION_X100 >= 710
#define USE_IPP_DFT 1
#else
#undef USE_IPP_DFT
#endif

#if defined USE_IPP_DFT
#if IPP_VERSION_X100 >= 202220
#define IPP_DISABLE_DFT32F ((depth == CV_32F) && (ippCPUID_AVX512F&cv::ipp::getIppFeatures()))
#else
#define IPP_DISABLE_DFT32F false
#endif
#endif

/****************************************************************************************\
                               Discrete Fourier Transform
\****************************************************************************************/

static unsigned char bitrevTab[] =
{
  0x00,0x80,0x40,0xc0,0x20,0xa0,0x60,0xe0,0x10,0x90,0x50,0xd0,0x30,0xb0,0x70,0xf0,
  0x08,0x88,0x48,0xc8,0x28,0xa8,0x68,0xe8,0x18,0x98,0x58,0xd8,0x38,0xb8,0x78,0xf8,
  0x04,0x84,0x44,0xc4,0x24,0xa4,0x64,0xe4,0x14,0x94,0x54,0xd4,0x34,0xb4,0x74,0xf4,
  0x0c,0x8c,0x4c,0xcc,0x2c,0xac,0x6c,0xec,0x1c,0x9c,0x5c,0xdc,0x3c,0xbc,0x7c,0xfc,
  0x02,0x82,0x42,0xc2,0x22,0xa2,0x62,0xe2,0x12,0x92,0x52,0xd2,0x32,0xb2,0x72,0xf2,
  0x0a,0x8a,0x4a,0xca,0x2a,0xaa,0x6a,0xea,0x1a,0x9a,0x5a,0xda,0x3a,0xba,0x7a,0xfa,
  0x06,0x86,0x46,0xc6,0x26,0xa6,0x66,0xe6,0x16,0x96,0x56,0xd6,0x36,0xb6,0x76,0xf6,
  0x0e,0x8e,0x4e,0xce,0x2e,0xae,0x6e,0xee,0x1e,0x9e,0x5e,0xde,0x3e,0xbe,0x7e,0xfe,
  0x01,0x81,0x41,0xc1,0x21,0xa1,0x61,0xe1,0x11,0x91,0x51,0xd1,0x31,0xb1,0x71,0xf1,
  0x09,0x89,0x49,0xc9,0x29,0xa9,0x69,0xe9,0x19,0x99,0x59,0xd9,0x39,0xb9,0x79,0xf9,
  0x05,0x85,0x45,0xc5,0x25,0xa5,0x65,0xe5,0x15,0x95,0x55,0xd5,0x35,0xb5,0x75,0xf5,
  0x0d,0x8d,0x4d,0xcd,0x2d,0xad,0x6d,0xed,0x1d,0x9d,0x5d,0xdd,0x3d,0xbd,0x7d,0xfd,
  0x03,0x83,0x43,0xc3,0x23,0xa3,0x63,0xe3,0x13,0x93,0x53,0xd3,0x33,0xb3,0x73,0xf3,
  0x0b,0x8b,0x4b,0xcb,0x2b,0xab,0x6b,0xeb,0x1b,0x9b,0x5b,0xdb,0x3b,0xbb,0x7b,0xfb,
  0x07,0x87,0x47,0xc7,0x27,0xa7,0x67,0xe7,0x17,0x97,0x57,0xd7,0x37,0xb7,0x77,0xf7,
  0x0f,0x8f,0x4f,0xcf,0x2f,0xaf,0x6f,0xef,0x1f,0x9f,0x5f,0xdf,0x3f,0xbf,0x7f,0xff
};

static const double DFTTab[][2] =
{
{ 1.00000000000000000, 0.00000000000000000 },
{-1.00000000000000000, 0.00000000000000000 },
{ 0.00000000000000000, 1.00000000000000000 },
{ 0.70710678118654757, 0.70710678118654746 },
{ 0.92387953251128674, 0.38268343236508978 },
{ 0.98078528040323043, 0.19509032201612825 },
{ 0.99518472667219693, 0.09801714032956060 },
{ 0.99879545620517241, 0.04906767432741802 },
{ 0.99969881869620425, 0.02454122852291229 },
{ 0.99992470183914450, 0.01227153828571993 },
{ 0.99998117528260111, 0.00613588464915448 },
{ 0.99999529380957619, 0.00306795676296598 },
{ 0.99999882345170188, 0.00153398018628477 },
{ 0.99999970586288223, 0.00076699031874270 },
{ 0.99999992646571789, 0.00038349518757140 },
{ 0.99999998161642933, 0.00019174759731070 },
{ 0.99999999540410733, 0.00009587379909598 },
{ 0.99999999885102686, 0.00004793689960307 },
{ 0.99999999971275666, 0.00002396844980842 },
{ 0.99999999992818922, 0.00001198422490507 },
{ 0.99999999998204725, 0.00000599211245264 },
{ 0.99999999999551181, 0.00000299605622633 },
{ 0.99999999999887801, 0.00000149802811317 },
{ 0.99999999999971945, 0.00000074901405658 },
{ 0.99999999999992983, 0.00000037450702829 },
{ 0.99999999999998246, 0.00000018725351415 },
{ 0.99999999999999567, 0.00000009362675707 },
{ 0.99999999999999889, 0.00000004681337854 },
{ 0.99999999999999978, 0.00000002340668927 },
{ 0.99999999999999989, 0.00000001170334463 },
{ 1.00000000000000000, 0.00000000585167232 },
{ 1.00000000000000000, 0.00000000292583616 }
};

namespace {
template <typename T>
struct Constants {
    static const T sin_120;
    static const T fft5_2;
    static const T fft5_3;
    static const T fft5_4;
    static const T fft5_5;
};

template <typename T>
const T Constants<T>::sin_120 = (T)0.86602540378443864676372317075294;

template <typename T>
const T Constants<T>::fft5_2 = (T)0.559016994374947424102293417182819;

template <typename T>
const T Constants<T>::fft5_3 = (T)-0.951056516295153572116439333379382;

template <typename T>
const T Constants<T>::fft5_4 = (T)-1.538841768587626701285145288018455;

template <typename T>
const T Constants<T>::fft5_5 = (T)0.363271264002680442947733378740309;

}  //namespace

#define BitRev(i,shift) \
   ((int)((((unsigned)bitrevTab[(i)&255] << 24)+ \
           ((unsigned)bitrevTab[((i)>> 8)&255] << 16)+ \
           ((unsigned)bitrevTab[((i)>>16)&255] <<  8)+ \
           ((unsigned)bitrevTab[((i)>>24)])) >> (shift)))

static int
DFTFactorize( int n, int* factors )
{
    int nf = 0, f, i, j;

    if( n <= 5 )
    {
        factors[0] = n;
        return 1;
    }

    f = (((n - 1)^n)+1) >> 1;
    if( f > 1 )
    {
        factors[nf++] = f;
        n = f == n ? 1 : n/f;
    }

    for( f = 3; n > 1; )
    {
        int d = n/f;
        if( d*f == n )
        {
            factors[nf++] = f;
            n = d;
        }
        else
        {
            f += 2;
            if( f*f > n )
                break;
        }
    }

    if( n > 1 )
        factors[nf++] = n;

    f = (factors[0] & 1) == 0;
    for( i = f; i < (nf+f)/2; i++ )
        CV_SWAP( factors[i], factors[nf-i-1+f], j );

    return nf;
}

static void
DFTInit( int n0, int nf, const int* factors, int* itab, int elem_size, void* _wave, int inv_itab )
{
    int digits[34], radix[34];
    int n = factors[0], m = 0;
    int* itab0 = itab;
    int i, j, k;
    Complex<double> w, w1;
    double t;

    if( n0 <= 5 )
    {
        itab[0] = 0;
        itab[n0-1] = n0-1;

        if( n0 != 4 )
        {
            for( i = 1; i < n0-1; i++ )
                itab[i] = i;
        }
        else
        {
            itab[1] = 2;
            itab[2] = 1;
        }
        if( n0 == 5 )
        {
            if( elem_size == sizeof(Complex<double>) )
                ((Complex<double>*)_wave)[0] = Complex<double>(1.,0.);
            else
                ((Complex<float>*)_wave)[0] = Complex<float>(1.f,0.f);
        }
        if( n0 != 4 )
            return;
        m = 2;
    }
    else
    {
        // radix[] is initialized from index 'nf' down to zero
        CV_Assert (nf < 34);
        radix[nf] = 1;
        digits[nf] = 0;
        for( i = 0; i < nf; i++ )
        {
            digits[i] = 0;
            radix[nf-i-1] = radix[nf-i]*factors[nf-i-1];
        }

        if( inv_itab && factors[0] != factors[nf-1] )
            itab = (int*)_wave;

        if( (n & 1) == 0 )
        {
            int a = radix[1], na2 = n*a>>1, na4 = na2 >> 1;
            for( m = 0; (unsigned)(1 << m) < (unsigned)n; m++ )
                ;
            if( n <= 2  )
            {
                itab[0] = 0;
                itab[1] = na2;
            }
            else if( n <= 256 )
            {
                int shift = 10 - m;
                for( i = 0; i <= n - 4; i += 4 )
                {
                    j = (bitrevTab[i>>2]>>shift)*a;
                    itab[i] = j;
                    itab[i+1] = j + na2;
                    itab[i+2] = j + na4;
                    itab[i+3] = j + na2 + na4;
                }
            }
            else
            {
                int shift = 34 - m;
                for( i = 0; i < n; i += 4 )
                {
                    int i4 = i >> 2;
                    j = BitRev(i4,shift)*a;
                    itab[i] = j;
                    itab[i+1] = j + na2;
                    itab[i+2] = j + na4;
                    itab[i+3] = j + na2 + na4;
                }
            }

            digits[1]++;

            if( nf >= 2 )
            {
                for( i = n, j = radix[2]; i < n0; )
                {
                    for( k = 0; k < n; k++ )
                        itab[i+k] = itab[k] + j;
                    if( (i += n) >= n0 )
                        break;
                    j += radix[2];
                    for( k = 1; ++digits[k] >= factors[k]; k++ )
                    {
                        digits[k] = 0;
                        j += radix[k+2] - radix[k];
                    }
                }
            }
        }
        else
        {
            for( i = 0, j = 0;; )
            {
                itab[i] = j;
                if( ++i >= n0 )
                    break;
                j += radix[1];
                for( k = 0; ++digits[k] >= factors[k]; k++ )
                {
                    digits[k] = 0;
                    j += radix[k+2] - radix[k];
                }
            }
        }

        if( itab != itab0 )
        {
            itab0[0] = 0;
            for( i = n0 & 1; i < n0; i += 2 )
            {
                int k0 = itab[i];
                int k1 = itab[i+1];
                itab0[k0] = i;
                itab0[k1] = i+1;
            }
        }
    }

    if( (n0 & (n0-1)) == 0 )
    {
        w.re = w1.re = DFTTab[m][0];
        w.im = w1.im = -DFTTab[m][1];
    }
    else
    {
        t = -CV_PI*2/n0;
        w.im = w1.im = sin(t);
        w.re = w1.re = std::sqrt(1. - w1.im*w1.im);
    }
    n = (n0+1)/2;

    if( elem_size == sizeof(Complex<double>) )
    {
        Complex<double>* wave = (Complex<double>*)_wave;

        wave[0].re = 1.;
        wave[0].im = 0.;

        if( (n0 & 1) == 0 )
        {
            wave[n].re = -1.;
            wave[n].im = 0;
        }

        for( i = 1; i < n; i++ )
        {
            wave[i] = w;
            wave[n0-i].re = w.re;
            wave[n0-i].im = -w.im;

            t = w.re*w1.re - w.im*w1.im;
            w.im = w.re*w1.im + w.im*w1.re;
            w.re = t;
        }
    }
    else
    {
        Complex<float>* wave = (Complex<float>*)_wave;
        CV_Assert( elem_size == sizeof(Complex<float>) );

        wave[0].re = 1.f;
        wave[0].im = 0.f;

        if( (n0 & 1) == 0 )
        {
            wave[n].re = -1.f;
            wave[n].im = 0.f;
        }

        for( i = 1; i < n; i++ )
        {
            wave[i].re = (float)w.re;
            wave[i].im = (float)w.im;
            wave[n0-i].re = (float)w.re;
            wave[n0-i].im = (float)-w.im;

            t = w.re*w1.re - w.im*w1.im;
            w.im = w.re*w1.im + w.im*w1.re;
            w.re = t;
        }
    }
}

// Reference radix-2 implementation.
template<typename T> struct DFT_R2
{
    void operator()(Complex<T>* dst, const int c_n, const int n, const int dw0, const Complex<T>* wave) const {
        const int nx = n/2;
        for(int i = 0 ; i < c_n; i += n)
        {
            Complex<T>* v = dst + i;
            T r0 = v[0].re + v[nx].re;
            T i0 = v[0].im + v[nx].im;
            T r1 = v[0].re - v[nx].re;
            T i1 = v[0].im - v[nx].im;
            v[0].re = r0; v[0].im = i0;
            v[nx].re = r1; v[nx].im = i1;

            for( int j = 1, dw = dw0; j < nx; j++, dw += dw0 )
            {
                v = dst + i + j;
                r1 = v[nx].re*wave[dw].re - v[nx].im*wave[dw].im;
                i1 = v[nx].im*wave[dw].re + v[nx].re*wave[dw].im;
                r0 = v[0].re; i0 = v[0].im;

                v[0].re = r0 + r1; v[0].im = i0 + i1;
                v[nx].re = r0 - r1; v[nx].im = i0 - i1;
            }
        }
    }
};

// Reference radix-3 implementation.
template<typename T> struct DFT_R3
{
    void operator()(Complex<T>* dst, const int c_n, const int n, const int dw0, const Complex<T>* wave) const {
        const int nx = n / 3;
        for(int i = 0; i < c_n; i += n )
        {
            {
                Complex<T>* v = dst + i;
                T r1 = v[nx].re + v[nx*2].re;
                T i1 = v[nx].im + v[nx*2].im;
                T r0 = v[0].re;
                T i0 = v[0].im;
                T r2 = Constants<T>::sin_120*(v[nx].im - v[nx*2].im);
                T i2 = Constants<T>::sin_120*(v[nx*2].re - v[nx].re);
                v[0].re = r0 + r1; v[0].im = i0 + i1;
                r0 -= (T)0.5*r1; i0 -= (T)0.5*i1;
                v[nx].re = r0 + r2; v[nx].im = i0 + i2;
                v[nx*2].re = r0 - r2; v[nx*2].im = i0 - i2;
            }

            for(int j = 1, dw = dw0; j < nx; j++, dw += dw0 )
            {
                Complex<T>* v = dst + i + j;
                T r0 = v[nx].re*wave[dw].re - v[nx].im*wave[dw].im;
                T i0 = v[nx].re*wave[dw].im + v[nx].im*wave[dw].re;
                T i2 = v[nx*2].re*wave[dw*2].re - v[nx*2].im*wave[dw*2].im;
                T r2 = v[nx*2].re*wave[dw*2].im + v[nx*2].im*wave[dw*2].re;
                T r1 = r0 + i2; T i1 = i0 + r2;

                r2 = Constants<T>::sin_120*(i0 - r2); i2 = Constants<T>::sin_120*(i2 - r0);
                r0 = v[0].re; i0 = v[0].im;
                v[0].re = r0 + r1; v[0].im = i0 + i1;
                r0 -= (T)0.5*r1; i0 -= (T)0.5*i1;
                v[nx].re = r0 + r2; v[nx].im = i0 + i2;
                v[nx*2].re = r0 - r2; v[nx*2].im = i0 - i2;
            }
        }
    }
};

// Reference radix-5 implementation.
template<typename T> struct DFT_R5
{
    void operator()(Complex<T>* dst, const int c_n, const int n, const int dw0, const Complex<T>* wave) const {
        const int nx = n / 5;
        for(int i = 0; i < c_n; i += n )
        {
            for(int j = 0, dw = 0; j < nx; j++, dw += dw0 )
            {
                Complex<T>* v0 = dst + i + j;
                Complex<T>* v1 = v0 + nx*2;
                Complex<T>* v2 = v1 + nx*2;

                T r0, i0, r1, i1, r2, i2, r3, i3, r4, i4, r5, i5;

                r3 = v0[nx].re*wave[dw].re - v0[nx].im*wave[dw].im;
                i3 = v0[nx].re*wave[dw].im + v0[nx].im*wave[dw].re;
                r2 = v2[0].re*wave[dw*4].re - v2[0].im*wave[dw*4].im;
                i2 = v2[0].re*wave[dw*4].im + v2[0].im*wave[dw*4].re;

                r1 = r3 + r2; i1 = i3 + i2;
                r3 -= r2; i3 -= i2;

                r4 = v1[nx].re*wave[dw*3].re - v1[nx].im*wave[dw*3].im;
                i4 = v1[nx].re*wave[dw*3].im + v1[nx].im*wave[dw*3].re;
                r0 = v1[0].re*wave[dw*2].re - v1[0].im*wave[dw*2].im;
                i0 = v1[0].re*wave[dw*2].im + v1[0].im*wave[dw*2].re;

                r2 = r4 + r0; i2 = i4 + i0;
                r4 -= r0; i4 -= i0;

                r0 = v0[0].re; i0 = v0[0].im;
                r5 = r1 + r2; i5 = i1 + i2;

                v0[0].re = r0 + r5; v0[0].im = i0 + i5;

                r0 -= (T)0.25*r5; i0 -= (T)0.25*i5;
                r1 = Constants<T>::fft5_2*(r1 - r2); i1 = Constants<T>::fft5_2*(i1 - i2);
                r2 = -Constants<T>::fft5_3*(i3 + i4); i2 = Constants<T>::fft5_3*(r3 + r4);

                i3 *= -Constants<T>::fft5_5; r3 *= Constants<T>::fft5_5;
                i4 *= -Constants<T>::fft5_4; r4 *= Constants<T>::fft5_4;

                r5 = r2 + i3; i5 = i2 + r3;
                r2 -= i4; i2 -= r4;

                r3 = r0 + r1; i3 = i0 + i1;
                r0 -= r1; i0 -= i1;

                v0[nx].re = r3 + r2; v0[nx].im = i3 + i2;
                v2[0].re = r3 - r2; v2[0].im = i3 - i2;

                v1[0].re = r0 + r5; v1[0].im = i0 + i5;
                v1[nx].re = r0 - r5; v1[nx].im = i0 - i5;
            }
        }
    }
};

template<typename T> struct DFT_VecR2
{
    void operator()(Complex<T>* dst, const int c_n, const int n, const int dw0, const Complex<T>* wave) const {
        DFT_R2<T>()(dst, c_n, n, dw0, wave);
    }
};

template<typename T> struct DFT_VecR3
{
    void operator()(Complex<T>* dst, const int c_n, const int n, const int dw0, const Complex<T>* wave) const {
        DFT_R3<T>()(dst, c_n, n, dw0, wave);
    }
};

template<typename T> struct DFT_VecR4
{
    int operator()(Complex<T>*, int, int, int&, const Complex<T>*) const { return 1; }
};

#if CV_SSE3

// multiplies *a and *b:
//  r_re + i*r_im = (a_re + i*a_im)*(b_re + i*b_im)
// r_re and r_im are placed respectively in bits 31:0 and 63:32 of the resulting
// vector register.
inline __m128 complexMul(const Complex<float>* const a, const Complex<float>* const b) {
    const __m128 z = _mm_setzero_ps();
    const __m128 neg_elem0 = _mm_set_ps(0.0f,0.0f,0.0f,-0.0f);
    // v_a[31:0] is a->re and v_a[63:32] is a->im.
    const __m128 v_a = _mm_loadl_pi(z, (const __m64*)a);
    const __m128 v_b = _mm_loadl_pi(z, (const __m64*)b);
    // x_1 = v[nx] * wave[dw].
    const __m128 v_a_riri = _mm_shuffle_ps(v_a, v_a, _MM_SHUFFLE(0, 1, 0, 1));
    const __m128 v_b_irri = _mm_shuffle_ps(v_b, v_b, _MM_SHUFFLE(1, 0, 0, 1));
    const __m128 mul = _mm_mul_ps(v_a_riri, v_b_irri);
    const __m128 xored = _mm_xor_ps(mul, neg_elem0);
    return _mm_hadd_ps(xored, z);
}

// optimized radix-2 transform
template<> struct DFT_VecR2<float> {
    void operator()(Complex<float>* dst, const int c_n, const int n, const int dw0, const Complex<float>* wave) const {
        const __m128 z = _mm_setzero_ps();
        const int nx = n/2;
        for(int i = 0 ; i < c_n; i += n)
        {
            {
                Complex<float>* v = dst + i;
                float r0 = v[0].re + v[nx].re;
                float i0 = v[0].im + v[nx].im;
                float r1 = v[0].re - v[nx].re;
                float i1 = v[0].im - v[nx].im;
                v[0].re = r0; v[0].im = i0;
                v[nx].re = r1; v[nx].im = i1;
            }

            for( int j = 1, dw = dw0; j < nx; j++, dw += dw0 )
            {
                Complex<float>* v = dst + i + j;
                const __m128 x_1 = complexMul(&v[nx], &wave[dw]);
                const __m128 v_0 = _mm_loadl_pi(z, (const __m64*)&v[0]);
                _mm_storel_pi((__m64*)&v[0], _mm_add_ps(v_0, x_1));
                _mm_storel_pi((__m64*)&v[nx], _mm_sub_ps(v_0, x_1));
            }
        }
    }
};

// Optimized radix-3 implementation.
template<> struct DFT_VecR3<float> {
    void operator()(Complex<float>* dst, const int c_n, const int n, const int dw0, const Complex<float>* wave) const {
        const int nx = n / 3;
        const __m128 z = _mm_setzero_ps();
        const __m128 neg_elem1 = _mm_set_ps(0.0f,0.0f,-0.0f,0.0f);
        const __m128 sin_120 = _mm_set1_ps(Constants<float>::sin_120);
        const __m128 one_half = _mm_set1_ps(0.5f);
        for(int i = 0; i < c_n; i += n )
        {
            {
                Complex<float>* v = dst + i;

                float r1 = v[nx].re + v[nx*2].re;
                float i1 = v[nx].im + v[nx*2].im;
                float r0 = v[0].re;
                float i0 = v[0].im;
                float r2 = Constants<float>::sin_120*(v[nx].im - v[nx*2].im);
                float i2 = Constants<float>::sin_120*(v[nx*2].re - v[nx].re);
                v[0].re = r0 + r1; v[0].im = i0 + i1;
                r0 -= (float)0.5*r1; i0 -= (float)0.5*i1;
                v[nx].re = r0 + r2; v[nx].im = i0 + i2;
                v[nx*2].re = r0 - r2; v[nx*2].im = i0 - i2;
            }

            for(int j = 1, dw = dw0; j < nx; j++, dw += dw0 )
            {
                Complex<float>* v = dst + i + j;
                const __m128 x_0 = complexMul(&v[nx], &wave[dw]);
                const __m128 x_2 = complexMul(&v[nx*2], &wave[dw*2]);
                const __m128 x_1 = _mm_add_ps(x_0, x_2);

                const __m128 v_0 = _mm_loadl_pi(z, (const __m64*)&v[0]);
                _mm_storel_pi((__m64*)&v[0], _mm_add_ps(v_0, x_1));

                const __m128 x_3 = _mm_mul_ps(sin_120, _mm_xor_ps(_mm_sub_ps(x_2, x_0), neg_elem1));
                const __m128 x_3s = _mm_shuffle_ps(x_3, x_3, _MM_SHUFFLE(0, 1, 0, 1));
                const __m128 x_4 = _mm_sub_ps(v_0, _mm_mul_ps(one_half, x_1));
                _mm_storel_pi((__m64*)&v[nx], _mm_add_ps(x_4, x_3s));
                _mm_storel_pi((__m64*)&v[nx*2], _mm_sub_ps(x_4, x_3s));
            }
        }
    }
};

// optimized radix-4 transform
template<> struct DFT_VecR4<float>
{
    int operator()(Complex<float>* dst, int N, int n0, int& _dw0, const Complex<float>* wave) const
    {
        int n = 1, i, j, nx, dw, dw0 = _dw0;
        __m128 z = _mm_setzero_ps(), x02=z, x13=z, w01=z, w23=z, y01, y23, t0, t1;
        Cv32suf t; t.i = 0x80000000;
        __m128 neg0_mask = _mm_load_ss(&t.f);
        __m128 neg3_mask = _mm_shuffle_ps(neg0_mask, neg0_mask, _MM_SHUFFLE(0,1,2,3));

        for( ; n*4 <= N; )
        {
            nx = n;
            n *= 4;
            dw0 /= 4;

            for( i = 0; i < n0; i += n )
            {
                Complexf *v0, *v1;

                v0 = dst + i;
                v1 = v0 + nx*2;

                x02 = _mm_loadl_pi(x02, (const __m64*)&v0[0]);
                x13 = _mm_loadl_pi(x13, (const __m64*)&v0[nx]);
                x02 = _mm_loadh_pi(x02, (const __m64*)&v1[0]);
                x13 = _mm_loadh_pi(x13, (const __m64*)&v1[nx]);

                y01 = _mm_add_ps(x02, x13);
                y23 = _mm_sub_ps(x02, x13);
                t1 = _mm_xor_ps(_mm_shuffle_ps(y01, y23, _MM_SHUFFLE(2,3,3,2)), neg3_mask);
                t0 = _mm_movelh_ps(y01, y23);
                y01 = _mm_add_ps(t0, t1);
                y23 = _mm_sub_ps(t0, t1);

                _mm_storel_pi((__m64*)&v0[0], y01);
                _mm_storeh_pi((__m64*)&v0[nx], y01);
                _mm_storel_pi((__m64*)&v1[0], y23);
                _mm_storeh_pi((__m64*)&v1[nx], y23);

                for( j = 1, dw = dw0; j < nx; j++, dw += dw0 )
                {
                    v0 = dst + i + j;
                    v1 = v0 + nx*2;

                    x13 = _mm_loadl_pi(x13, (const __m64*)&v0[nx]);
                    w23 = _mm_loadl_pi(w23, (const __m64*)&wave[dw*2]);
                    x13 = _mm_loadh_pi(x13, (const __m64*)&v1[nx]); // x1, x3 = r1 i1 r3 i3
                    w23 = _mm_loadh_pi(w23, (const __m64*)&wave[dw*3]); // w2, w3 = wr2 wi2 wr3 wi3

                    t0 = _mm_mul_ps(_mm_moveldup_ps(x13), w23);
                    t1 = _mm_mul_ps(_mm_movehdup_ps(x13), _mm_shuffle_ps(w23, w23, _MM_SHUFFLE(2,3,0,1)));
                    x13 = _mm_addsub_ps(t0, t1);
                    // re(x1*w2), im(x1*w2), re(x3*w3), im(x3*w3)
                    x02 = _mm_loadl_pi(x02, (const __m64*)&v1[0]); // x2 = r2 i2
                    w01 = _mm_loadl_pi(w01, (const __m64*)&wave[dw]); // w1 = wr1 wi1
                    x02 = _mm_shuffle_ps(x02, x02, _MM_SHUFFLE(0,0,1,1));
                    w01 = _mm_shuffle_ps(w01, w01, _MM_SHUFFLE(1,0,0,1));
                    x02 = _mm_mul_ps(x02, w01);
                    x02 = _mm_addsub_ps(x02, _mm_movelh_ps(x02, x02));
                    // re(x0) im(x0) re(x2*w1), im(x2*w1)
                    x02 = _mm_loadl_pi(x02, (const __m64*)&v0[0]);

                    y01 = _mm_add_ps(x02, x13);
                    y23 = _mm_sub_ps(x02, x13);
                    t1 = _mm_xor_ps(_mm_shuffle_ps(y01, y23, _MM_SHUFFLE(2,3,3,2)), neg3_mask);
                    t0 = _mm_movelh_ps(y01, y23);
                    y01 = _mm_add_ps(t0, t1);
                    y23 = _mm_sub_ps(t0, t1);

                    _mm_storel_pi((__m64*)&v0[0], y01);
                    _mm_storeh_pi((__m64*)&v0[nx], y01);
                    _mm_storel_pi((__m64*)&v1[0], y23);
                    _mm_storeh_pi((__m64*)&v1[nx], y23);
                }
            }
        }

        _dw0 = dw0;
        return n;
    }
};

#endif

#ifdef USE_IPP_DFT
static IppStatus ippsDFTFwd_CToC( const Complex<float>* src, Complex<float>* dst,
                             const void* spec, uchar* buf)
{
    return CV_INSTRUMENT_FUN_IPP(ippsDFTFwd_CToC_32fc, (const Ipp32fc*)src, (Ipp32fc*)dst,
                                 (const IppsDFTSpec_C_32fc*)spec, buf);
}

static IppStatus ippsDFTFwd_CToC( const Complex<double>* src, Complex<double>* dst,
                             const void* spec, uchar* buf)
{
    return CV_INSTRUMENT_FUN_IPP(ippsDFTFwd_CToC_64fc, (const Ipp64fc*)src, (Ipp64fc*)dst,
                                 (const IppsDFTSpec_C_64fc*)spec, buf);
}

static IppStatus ippsDFTInv_CToC( const Complex<float>* src, Complex<float>* dst,
                             const void* spec, uchar* buf)
{
    return CV_INSTRUMENT_FUN_IPP(ippsDFTInv_CToC_32fc, (const Ipp32fc*)src, (Ipp32fc*)dst,
                                 (const IppsDFTSpec_C_32fc*)spec, buf);
}

static IppStatus ippsDFTInv_CToC( const Complex<double>* src, Complex<double>* dst,
                                  const void* spec, uchar* buf)
{
    return CV_INSTRUMENT_FUN_IPP(ippsDFTInv_CToC_64fc, (const Ipp64fc*)src, (Ipp64fc*)dst,
                                 (const IppsDFTSpec_C_64fc*)spec, buf);
}

static IppStatus ippsDFTFwd_RToPack( const float* src, float* dst,
                                     const void* spec, uchar* buf)
{
    return CV_INSTRUMENT_FUN_IPP(ippsDFTFwd_RToPack_32f, src, dst, (const IppsDFTSpec_R_32f*)spec, buf);
}

static IppStatus ippsDFTFwd_RToPack( const double* src, double* dst,
                                     const void* spec, uchar* buf)
{
    return CV_INSTRUMENT_FUN_IPP(ippsDFTFwd_RToPack_64f, src, dst, (const IppsDFTSpec_R_64f*)spec, buf);
}

static IppStatus ippsDFTInv_PackToR( const float* src, float* dst,
                                     const void* spec, uchar* buf)
{
    return CV_INSTRUMENT_FUN_IPP(ippsDFTInv_PackToR_32f, src, dst, (const IppsDFTSpec_R_32f*)spec, buf);
}

static IppStatus ippsDFTInv_PackToR( const double* src, double* dst,
                                     const void* spec, uchar* buf)
{
    return CV_INSTRUMENT_FUN_IPP(ippsDFTInv_PackToR_64f, src, dst, (const IppsDFTSpec_R_64f*)spec, buf);
}
#endif

struct OcvDftOptions;

typedef void (*DFTFunc)(const OcvDftOptions & c, const void* src, void* dst);

struct OcvDftOptions {
    int nf;
    int *factors;
    double scale;

    int* itab;
    void* wave;
    int tab_size;
    int n;

    bool isInverse;
    bool noPermute;
    bool isComplex;

    bool haveSSE3;

    DFTFunc dft_func;
    bool useIpp;

#ifdef USE_IPP_DFT
    uchar* ipp_spec;
    uchar* ipp_work;
#endif

    OcvDftOptions()
    {
        nf = 0;
        factors = 0;
        scale = 0;
        itab = 0;
        wave = 0;
        tab_size = 0;
        n = 0;
        isInverse = false;
        noPermute = false;
        isComplex = false;
        useIpp = false;
#ifdef USE_IPP_DFT
        ipp_spec = 0;
        ipp_work = 0;
#endif
        dft_func = 0;
        haveSSE3 = checkHardwareSupport(CV_CPU_SSE3);
    }
};

// mixed-radix complex discrete Fourier transform: double-precision version
template<typename T> static void
DFT(const OcvDftOptions & c, const Complex<T>* src, Complex<T>* dst)
{
    const Complex<T>* wave = (Complex<T>*)c.wave;
    const int * itab = c.itab;

    int n = c.n;
    int f_idx, nx;
    int inv = c.isInverse;
    int dw0 = c.tab_size, dw;
    int i, j, k;
    Complex<T> t;
    T scale = (T)c.scale;

    if(typeid(T) == typeid(float))
    {
        CALL_HAL(dft, cv_hal_dft, reinterpret_cast<const uchar*>(src), reinterpret_cast<uchar*>(dst), CV_32F,
                 c.nf, c.factors, c.scale, c.itab, c.wave, c.tab_size, c.n, c.isInverse, c.noPermute);
    }
    if(typeid(T) == typeid(double))
    {
        CALL_HAL(dft, cv_hal_dft, reinterpret_cast<const uchar*>(src), reinterpret_cast<uchar*>(dst), CV_64F,
                 c.nf, c.factors, c.scale, c.itab, c.wave, c.tab_size, c.n, c.isInverse, c.noPermute);
    }

    if( c.useIpp )
    {
#ifdef USE_IPP_DFT
        if( !inv )
        {
            if (ippsDFTFwd_CToC( src, dst, c.ipp_spec, c.ipp_work ) >= 0)
            {
                CV_IMPL_ADD(CV_IMPL_IPP);
                return;
            }
        }
        else
        {
            if (ippsDFTInv_CToC( src, dst, c.ipp_spec, c.ipp_work ) >= 0)
            {
                CV_IMPL_ADD(CV_IMPL_IPP);
                return;
            }
        }
        setIppErrorStatus();
#endif
    }

    int tab_step = c.tab_size == n ? 1 : c.tab_size == n*2 ? 2 : c.tab_size/n;

    // 0. shuffle data
    if( dst != src )
    {
        CV_Assert( !c.noPermute );
        if( !inv )
        {
            for( i = 0; i <= n - 2; i += 2, itab += 2*tab_step )
            {
                int k0 = itab[0], k1 = itab[tab_step];
                CV_Assert( (unsigned)k0 < (unsigned)n && (unsigned)k1 < (unsigned)n );
                dst[i] = src[k0]; dst[i+1] = src[k1];
            }

            if( i < n )
                dst[n-1] = src[n-1];
        }
        else
        {
            for( i = 0; i <= n - 2; i += 2, itab += 2*tab_step )
            {
                int k0 = itab[0], k1 = itab[tab_step];
                CV_Assert( (unsigned)k0 < (unsigned)n && (unsigned)k1 < (unsigned)n );
                t.re = src[k0].re; t.im = -src[k0].im;
                dst[i] = t;
                t.re = src[k1].re; t.im = -src[k1].im;
                dst[i+1] = t;
            }

            if( i < n )
            {
                t.re = src[n-1].re; t.im = -src[n-1].im;
                dst[i] = t;
            }
        }
    }
    else
    {
        if( !c.noPermute )
        {
            CV_Assert( c.factors[0] == c.factors[c.nf-1] );
            if( c.nf == 1 )
            {
                if( (n & 3) == 0 )
                {
                    int n2 = n/2;
                    Complex<T>* dsth = dst + n2;

                    for( i = 0; i < n2; i += 2, itab += tab_step*2 )
                    {
                        j = itab[0];
                        CV_Assert( (unsigned)j < (unsigned)n2 );

                        CV_SWAP(dst[i+1], dsth[j], t);
                        if( j > i )
                        {
                            CV_SWAP(dst[i], dst[j], t);
                            CV_SWAP(dsth[i+1], dsth[j+1], t);
                        }
                    }
                }
                // else do nothing
            }
            else
            {
                for( i = 0; i < n; i++, itab += tab_step )
                {
                    j = itab[0];
                    CV_Assert( (unsigned)j < (unsigned)n );
                    if( j > i )
                        CV_SWAP(dst[i], dst[j], t);
                }
            }
        }

        if( inv )
        {
            for( i = 0; i <= n - 2; i += 2 )
            {
                T t0 = -dst[i].im;
                T t1 = -dst[i+1].im;
                dst[i].im = t0; dst[i+1].im = t1;
            }

            if( i < n )
                dst[n-1].im = -dst[n-1].im;
        }
    }

    n = 1;
    // 1. power-2 transforms
    if( (c.factors[0] & 1) == 0 )
    {
        if( c.factors[0] >= 4 && c.haveSSE3)
        {
            DFT_VecR4<T> vr4;
            n = vr4(dst, c.factors[0], c.n, dw0, wave);
        }

        // radix-4 transform
        for( ; n*4 <= c.factors[0]; )
        {
            nx = n;
            n *= 4;
            dw0 /= 4;

            for( i = 0; i < c.n; i += n )
            {
                Complex<T> *v0, *v1;
                T r0, i0, r1, i1, r2, i2, r3, i3, r4, i4;

                v0 = dst + i;
                v1 = v0 + nx*2;

                r0 = v1[0].re; i0 = v1[0].im;
                r4 = v1[nx].re; i4 = v1[nx].im;

                r1 = r0 + r4; i1 = i0 + i4;
                r3 = i0 - i4; i3 = r4 - r0;

                r2 = v0[0].re; i2 = v0[0].im;
                r4 = v0[nx].re; i4 = v0[nx].im;

                r0 = r2 + r4; i0 = i2 + i4;
                r2 -= r4; i2 -= i4;

                v0[0].re = r0 + r1; v0[0].im = i0 + i1;
                v1[0].re = r0 - r1; v1[0].im = i0 - i1;
                v0[nx].re = r2 + r3; v0[nx].im = i2 + i3;
                v1[nx].re = r2 - r3; v1[nx].im = i2 - i3;

                for( j = 1, dw = dw0; j < nx; j++, dw += dw0 )
                {
                    v0 = dst + i + j;
                    v1 = v0 + nx*2;

                    r2 = v0[nx].re*wave[dw*2].re - v0[nx].im*wave[dw*2].im;
                    i2 = v0[nx].re*wave[dw*2].im + v0[nx].im*wave[dw*2].re;
                    r0 = v1[0].re*wave[dw].im + v1[0].im*wave[dw].re;
                    i0 = v1[0].re*wave[dw].re - v1[0].im*wave[dw].im;
                    r3 = v1[nx].re*wave[dw*3].im + v1[nx].im*wave[dw*3].re;
                    i3 = v1[nx].re*wave[dw*3].re - v1[nx].im*wave[dw*3].im;

                    r1 = i0 + i3; i1 = r0 + r3;
                    r3 = r0 - r3; i3 = i3 - i0;
                    r4 = v0[0].re; i4 = v0[0].im;

                    r0 = r4 + r2; i0 = i4 + i2;
                    r2 = r4 - r2; i2 = i4 - i2;

                    v0[0].re = r0 + r1; v0[0].im = i0 + i1;
                    v1[0].re = r0 - r1; v1[0].im = i0 - i1;
                    v0[nx].re = r2 + r3; v0[nx].im = i2 + i3;
                    v1[nx].re = r2 - r3; v1[nx].im = i2 - i3;
                }
            }
        }

        for( ; n < c.factors[0]; )
        {
            // do the remaining radix-2 transform
            n *= 2;
            dw0 /= 2;

            if(c.haveSSE3)
            {
                DFT_VecR2<T> vr2;
                vr2(dst, c.n, n, dw0, wave);
            }
            else
            {
                DFT_R2<T> vr2;
                vr2(dst, c.n, n, dw0, wave);
            }
        }
    }

    // 2. all the other transforms
    for( f_idx = (c.factors[0]&1) ? 0 : 1; f_idx < c.nf; f_idx++ )
    {
        int factor = c.factors[f_idx];
        nx = n;
        n *= factor;
        dw0 /= factor;

        if( factor == 3 )
        {
            if(c.haveSSE3)
            {
                DFT_VecR3<T> vr3;
                vr3(dst, c.n, n, dw0, wave);
            }
            else
            {
                DFT_R3<T> vr3;
                vr3(dst, c.n, n, dw0, wave);
            }
        }
        else if( factor == 5 )
        {
            DFT_R5<T> vr5;
            vr5(dst, c.n, n, dw0, wave);
        }
        else
        {
            // radix-"factor" - an odd number
            int p, q, factor2 = (factor - 1)/2;
            int d, dd, dw_f = c.tab_size/factor;
            AutoBuffer<Complex<T> > buf(factor2 * 2);
            Complex<T>* a = buf.data();
            Complex<T>* b = a + factor2;

            for( i = 0; i < c.n; i += n )
            {
                for( j = 0, dw = 0; j < nx; j++, dw += dw0 )
                {
                    Complex<T>* v = dst + i + j;
                    Complex<T> v_0 = v[0];
                    Complex<T> vn_0 = v_0;

                    if( j == 0 )
                    {
                        for( p = 1, k = nx; p <= factor2; p++, k += nx )
                        {
                            T r0 = v[k].re + v[n-k].re;
                            T i0 = v[k].im - v[n-k].im;
                            T r1 = v[k].re - v[n-k].re;
                            T i1 = v[k].im + v[n-k].im;

                            vn_0.re += r0; vn_0.im += i1;
                            a[p-1].re = r0; a[p-1].im = i0;
                            b[p-1].re = r1; b[p-1].im = i1;
                        }
                    }
                    else
                    {
                        const Complex<T>* wave_ = wave + dw*factor;
                        d = dw;

                        for( p = 1, k = nx; p <= factor2; p++, k += nx, d += dw )
                        {
                            T r2 = v[k].re*wave[d].re - v[k].im*wave[d].im;
                            T i2 = v[k].re*wave[d].im + v[k].im*wave[d].re;

                            T r1 = v[n-k].re*wave_[-d].re - v[n-k].im*wave_[-d].im;
                            T i1 = v[n-k].re*wave_[-d].im + v[n-k].im*wave_[-d].re;

                            T r0 = r2 + r1;
                            T i0 = i2 - i1;
                            r1 = r2 - r1;
                            i1 = i2 + i1;

                            vn_0.re += r0; vn_0.im += i1;
                            a[p-1].re = r0; a[p-1].im = i0;
                            b[p-1].re = r1; b[p-1].im = i1;
                        }
                    }

                    v[0] = vn_0;

                    for( p = 1, k = nx; p <= factor2; p++, k += nx )
                    {
                        Complex<T> s0 = v_0, s1 = v_0;
                        d = dd = dw_f*p;

                        for( q = 0; q < factor2; q++ )
                        {
                            T r0 = wave[d].re * a[q].re;
                            T i0 = wave[d].im * a[q].im;
                            T r1 = wave[d].re * b[q].im;
                            T i1 = wave[d].im * b[q].re;

                            s1.re += r0 + i0; s0.re += r0 - i0;
                            s1.im += r1 - i1; s0.im += r1 + i1;

                            d += dd;
                            d -= -(d >= c.tab_size) & c.tab_size;
                        }

                        v[k] = s0;
                        v[n-k] = s1;
                    }
                }
            }
        }
    }

    if( scale != 1 )
    {
        T re_scale = scale, im_scale = scale;
        if( inv )
            im_scale = -im_scale;

        for( i = 0; i < c.n; i++ )
        {
            T t0 = dst[i].re*re_scale;
            T t1 = dst[i].im*im_scale;
            dst[i].re = t0;
            dst[i].im = t1;
        }
    }
    else if( inv )
    {
        for( i = 0; i <= c.n - 2; i += 2 )
        {
            T t0 = -dst[i].im;
            T t1 = -dst[i+1].im;
            dst[i].im = t0;
            dst[i+1].im = t1;
        }

        if( i < c.n )
            dst[c.n-1].im = -dst[c.n-1].im;
    }
}


/* FFT of real vector
   output vector format:
     re(0), re(1), im(1), ... , re(n/2-1), im((n+1)/2-1) [, re((n+1)/2)] OR ...
     re(0), 0, re(1), im(1), ..., re(n/2-1), im((n+1)/2-1) [, re((n+1)/2), 0] */
template<typename T> static void
RealDFT(const OcvDftOptions & c, const T* src, T* dst)
{
    int n = c.n;
    int complex_output = c.isComplex;
    T scale = (T)c.scale;
    int j;
    dst += complex_output;

    if( c.useIpp )
    {
#ifdef USE_IPP_DFT
        if (ippsDFTFwd_RToPack( src, dst, c.ipp_spec, c.ipp_work ) >=0)
        {
            if( complex_output )
            {
                dst[-1] = dst[0];
                dst[0] = 0;
                if( (n & 1) == 0 )
                    dst[n] = 0;
            }
            CV_IMPL_ADD(CV_IMPL_IPP);
            return;
        }
        setIppErrorStatus();
#endif
    }
    CV_Assert( c.tab_size == n );

    if( n == 1 )
    {
        dst[0] = src[0]*scale;
    }
    else if( n == 2 )
    {
        T t = (src[0] + src[1])*scale;
        dst[1] = (src[0] - src[1])*scale;
        dst[0] = t;
    }
    else if( n & 1 )
    {
        dst -= complex_output;
        Complex<T>* _dst = (Complex<T>*)dst;
        _dst[0].re = src[0]*scale;
        _dst[0].im = 0;
        for( j = 1; j < n; j += 2 )
        {
            T t0 = src[c.itab[j]]*scale;
            T t1 = src[c.itab[j+1]]*scale;
            _dst[j].re = t0;
            _dst[j].im = 0;
            _dst[j+1].re = t1;
            _dst[j+1].im = 0;
        }
        OcvDftOptions sub_c = c;
        sub_c.isComplex = false;
        sub_c.isInverse = false;
        sub_c.noPermute = true;
        sub_c.scale = 1.;
        DFT(sub_c, _dst, _dst);
        if( !complex_output )
            dst[1] = dst[0];
    }
    else
    {
        T t0, t;
        T h1_re, h1_im, h2_re, h2_im;
        T scale2 = scale*(T)0.5;
        int n2 = n >> 1;

        c.factors[0] >>= 1;

        OcvDftOptions sub_c = c;
        sub_c.factors += (c.factors[0] == 1);
        sub_c.nf -= (c.factors[0] == 1);
        sub_c.isComplex = false;
        sub_c.isInverse = false;
        sub_c.noPermute = false;
        sub_c.scale = 1.;
        sub_c.n = n2;

        DFT(sub_c, (Complex<T>*)src, (Complex<T>*)dst);

        c.factors[0] <<= 1;

        t = dst[0] - dst[1];
        dst[0] = (dst[0] + dst[1])*scale;
        dst[1] = t*scale;

        t0 = dst[n2];
        t = dst[n-1];
        dst[n-1] = dst[1];

        const Complex<T> *wave = (const Complex<T>*)c.wave;

        for( j = 2, wave++; j < n2; j += 2, wave++ )
        {
            /* calc odd */
            h2_re = scale2*(dst[j+1] + t);
            h2_im = scale2*(dst[n-j] - dst[j]);

            /* calc even */
            h1_re = scale2*(dst[j] + dst[n-j]);
            h1_im = scale2*(dst[j+1] - t);

            /* rotate */
            t = h2_re*wave->re - h2_im*wave->im;
            h2_im = h2_re*wave->im + h2_im*wave->re;
            h2_re = t;
            t = dst[n-j-1];

            dst[j-1] = h1_re + h2_re;
            dst[n-j-1] = h1_re - h2_re;
            dst[j] = h1_im + h2_im;
            dst[n-j] = h2_im - h1_im;
        }

        if( j <= n2 )
        {
            dst[n2-1] = t0*scale;
            dst[n2] = -t*scale;
        }
    }

    if( complex_output && ((n & 1) == 0 || n == 1))
    {
        dst[-1] = dst[0];
        dst[0] = 0;
        if( n > 1 )
            dst[n] = 0;
    }
}

/* Inverse FFT of complex conjugate-symmetric vector
   input vector format:
      re[0], re[1], im[1], ... , re[n/2-1], im[n/2-1], re[n/2] OR
      re(0), 0, re(1), im(1), ..., re(n/2-1), im((n+1)/2-1) [, re((n+1)/2), 0] */
template<typename T> static void
CCSIDFT(const OcvDftOptions & c, const T* src, T* dst)
{
    int n = c.n;
    int complex_input = c.isComplex;
    int j, k;
    T scale = (T)c.scale;
    T save_s1 = 0.;
    T t0, t1, t2, t3, t;

    CV_Assert( c.tab_size == n );

    if( complex_input )
    {
        CV_Assert( src != dst );
        save_s1 = src[1];
        ((T*)src)[1] = src[0];
        src++;
    }
    if( c.useIpp )
    {
#ifdef USE_IPP_DFT
        if (ippsDFTInv_PackToR( src, dst, c.ipp_spec, c.ipp_work ) >=0)
        {
            if( complex_input )
                ((T*)src)[0] = (T)save_s1;
            CV_IMPL_ADD(CV_IMPL_IPP);
            return;
        }

        setIppErrorStatus();
#endif
    }
    if( n == 1 )
    {
        dst[0] = (T)(src[0]*scale);
    }
    else if( n == 2 )
    {
        t = (src[0] + src[1])*scale;
        dst[1] = (src[0] - src[1])*scale;
        dst[0] = t;
    }
    else if( n & 1 )
    {
        Complex<T>* _src = (Complex<T>*)(src-1);
        Complex<T>* _dst = (Complex<T>*)dst;

        _dst[0].re = src[0];
        _dst[0].im = 0;

        int n2 = (n+1) >> 1;

        for( j = 1; j < n2; j++ )
        {
            int k0 = c.itab[j], k1 = c.itab[n-j];
            t0 = _src[j].re; t1 = _src[j].im;
            _dst[k0].re = t0; _dst[k0].im = -t1;
            _dst[k1].re = t0; _dst[k1].im = t1;
        }

        OcvDftOptions sub_c = c;
        sub_c.isComplex = false;
        sub_c.isInverse = false;
        sub_c.noPermute = true;
        sub_c.scale = 1.;
        sub_c.n = n;

        DFT(sub_c, _dst, _dst);
        dst[0] *= scale;
        for( j = 1; j < n; j += 2 )
        {
            t0 = dst[j*2]*scale;
            t1 = dst[j*2+2]*scale;
            dst[j] = t0;
            dst[j+1] = t1;
        }
    }
    else
    {
        int inplace = src == dst;
        const Complex<T>* w = (const Complex<T>*)c.wave;

        t = src[1];
        t0 = (src[0] + src[n-1]);
        t1 = (src[n-1] - src[0]);
        dst[0] = t0;
        dst[1] = t1;

        int n2 = (n+1) >> 1;

        for( j = 2, w++; j < n2; j += 2, w++ )
        {
            T h1_re, h1_im, h2_re, h2_im;

            h1_re = (t + src[n-j-1]);
            h1_im = (src[j] - src[n-j]);

            h2_re = (t - src[n-j-1]);
            h2_im = (src[j] + src[n-j]);

            t = h2_re*w->re + h2_im*w->im;
            h2_im = h2_im*w->re - h2_re*w->im;
            h2_re = t;

            t = src[j+1];
            t0 = h1_re - h2_im;
            t1 = -h1_im - h2_re;
            t2 = h1_re + h2_im;
            t3 = h1_im - h2_re;

            if( inplace )
            {
                dst[j] = t0;
                dst[j+1] = t1;
                dst[n-j] = t2;
                dst[n-j+1]= t3;
            }
            else
            {
                int j2 = j >> 1;
                k = c.itab[j2];
                dst[k] = t0;
                dst[k+1] = t1;
                k = c.itab[n2-j2];
                dst[k] = t2;
                dst[k+1]= t3;
            }
        }

        if( j <= n2 )
        {
            t0 = t*2;
            t1 = src[n2]*2;

            if( inplace )
            {
                dst[n2] = t0;
                dst[n2+1] = t1;
            }
            else
            {
                k = c.itab[n2];
                dst[k*2] = t0;
                dst[k*2+1] = t1;
            }
        }

        c.factors[0] >>= 1;

        OcvDftOptions sub_c = c;
        sub_c.factors += (c.factors[0] == 1);
        sub_c.nf -= (c.factors[0] == 1);
        sub_c.isComplex = false;
        sub_c.isInverse = false;
        sub_c.noPermute = !inplace;
        sub_c.scale = 1.;
        sub_c.n = n2;

        DFT(sub_c, (Complex<T>*)dst, (Complex<T>*)dst);

        c.factors[0] <<= 1;

        for( j = 0; j < n; j += 2 )
        {
            t0 = dst[j]*scale;
            t1 = dst[j+1]*(-scale);
            dst[j] = t0;
            dst[j+1] = t1;
        }
    }
    if( complex_input )
        ((T*)src)[0] = (T)save_s1;
}

static void
CopyColumn( const uchar* _src, size_t src_step,
            uchar* _dst, size_t dst_step,
            int len, size_t elem_size )
{
    int i, t0, t1;
    const int* src = (const int*)_src;
    int* dst = (int*)_dst;
    src_step /= sizeof(src[0]);
    dst_step /= sizeof(dst[0]);

    if( elem_size == sizeof(int) )
    {
        for( i = 0; i < len; i++, src += src_step, dst += dst_step )
            dst[0] = src[0];
    }
    else if( elem_size == sizeof(int)*2 )
    {
        for( i = 0; i < len; i++, src += src_step, dst += dst_step )
        {
            t0 = src[0]; t1 = src[1];
            dst[0] = t0; dst[1] = t1;
        }
    }
    else if( elem_size == sizeof(int)*4 )
    {
        for( i = 0; i < len; i++, src += src_step, dst += dst_step )
        {
            t0 = src[0]; t1 = src[1];
            dst[0] = t0; dst[1] = t1;
            t0 = src[2]; t1 = src[3];
            dst[2] = t0; dst[3] = t1;
        }
    }
}


static void
CopyFrom2Columns( const uchar* _src, size_t src_step,
                  uchar* _dst0, uchar* _dst1,
                  int len, size_t elem_size )
{
    int i, t0, t1;
    const int* src = (const int*)_src;
    int* dst0 = (int*)_dst0;
    int* dst1 = (int*)_dst1;
    src_step /= sizeof(src[0]);

    if( elem_size == sizeof(int) )
    {
        for( i = 0; i < len; i++, src += src_step )
        {
            t0 = src[0]; t1 = src[1];
            dst0[i] = t0; dst1[i] = t1;
        }
    }
    else if( elem_size == sizeof(int)*2 )
    {
        for( i = 0; i < len*2; i += 2, src += src_step )
        {
            t0 = src[0]; t1 = src[1];
            dst0[i] = t0; dst0[i+1] = t1;
            t0 = src[2]; t1 = src[3];
            dst1[i] = t0; dst1[i+1] = t1;
        }
    }
    else if( elem_size == sizeof(int)*4 )
    {
        for( i = 0; i < len*4; i += 4, src += src_step )
        {
            t0 = src[0]; t1 = src[1];
            dst0[i] = t0; dst0[i+1] = t1;
            t0 = src[2]; t1 = src[3];
            dst0[i+2] = t0; dst0[i+3] = t1;
            t0 = src[4]; t1 = src[5];
            dst1[i] = t0; dst1[i+1] = t1;
            t0 = src[6]; t1 = src[7];
            dst1[i+2] = t0; dst1[i+3] = t1;
        }
    }
}


static void
CopyTo2Columns( const uchar* _src0, const uchar* _src1,
                uchar* _dst, size_t dst_step,
                int len, size_t elem_size )
{
    int i, t0, t1;
    const int* src0 = (const int*)_src0;
    const int* src1 = (const int*)_src1;
    int* dst = (int*)_dst;
    dst_step /= sizeof(dst[0]);

    if( elem_size == sizeof(int) )
    {
        for( i = 0; i < len; i++, dst += dst_step )
        {
            t0 = src0[i]; t1 = src1[i];
            dst[0] = t0; dst[1] = t1;
        }
    }
    else if( elem_size == sizeof(int)*2 )
    {
        for( i = 0; i < len*2; i += 2, dst += dst_step )
        {
            t0 = src0[i]; t1 = src0[i+1];
            dst[0] = t0; dst[1] = t1;
            t0 = src1[i]; t1 = src1[i+1];
            dst[2] = t0; dst[3] = t1;
        }
    }
    else if( elem_size == sizeof(int)*4 )
    {
        for( i = 0; i < len*4; i += 4, dst += dst_step )
        {
            t0 = src0[i]; t1 = src0[i+1];
            dst[0] = t0; dst[1] = t1;
            t0 = src0[i+2]; t1 = src0[i+3];
            dst[2] = t0; dst[3] = t1;
            t0 = src1[i]; t1 = src1[i+1];
            dst[4] = t0; dst[5] = t1;
            t0 = src1[i+2]; t1 = src1[i+3];
            dst[6] = t0; dst[7] = t1;
        }
    }
}


static void
ExpandCCS( uchar* _ptr, int n, int elem_size )
{
    int i;
    if( elem_size == (int)sizeof(float) )
    {
        float* p = (float*)_ptr;
        for( i = 1; i < (n+1)/2; i++ )
        {
            p[(n-i)*2] = p[i*2-1];
            p[(n-i)*2+1] = -p[i*2];
        }
        if( (n & 1) == 0 )
        {
            p[n] = p[n-1];
            p[n+1] = 0.f;
            n--;
        }
        for( i = n-1; i > 0; i-- )
            p[i+1] = p[i];
        p[1] = 0.f;
    }
    else
    {
        double* p = (double*)_ptr;
        for( i = 1; i < (n+1)/2; i++ )
        {
            p[(n-i)*2] = p[i*2-1];
            p[(n-i)*2+1] = -p[i*2];
        }
        if( (n & 1) == 0 )
        {
            p[n] = p[n-1];
            p[n+1] = 0.f;
            n--;
        }
        for( i = n-1; i > 0; i-- )
            p[i+1] = p[i];
        p[1] = 0.f;
    }
}

static void DFT_32f(const OcvDftOptions & c, const Complexf* src, Complexf* dst)
{
    DFT(c, src, dst);
}

static void DFT_64f(const OcvDftOptions & c, const Complexd* src, Complexd* dst)
{
    DFT(c, src, dst);
}


static void RealDFT_32f(const OcvDftOptions & c, const float* src, float* dst)
{
    RealDFT(c, src, dst);
}

static void RealDFT_64f(const OcvDftOptions & c, const double* src, double* dst)
{
    RealDFT(c, src, dst);
}

static void CCSIDFT_32f(const OcvDftOptions & c, const float* src, float* dst)
{
    CCSIDFT(c, src, dst);
}

static void CCSIDFT_64f(const OcvDftOptions & c, const double* src, double* dst)
{
    CCSIDFT(c, src, dst);
}

}

#ifdef USE_IPP_DFT
typedef IppStatus (CV_STDCALL* IppDFTGetSizeFunc)(int, int, IppHintAlgorithm, int*, int*, int*);
typedef IppStatus (CV_STDCALL* IppDFTInitFunc)(int, int, IppHintAlgorithm, void*, uchar*);
#endif

namespace cv
{
#if defined USE_IPP_DFT

typedef IppStatus (CV_STDCALL* ippiDFT_C_Func)(const Ipp32fc*, int, Ipp32fc*, int, const IppiDFTSpec_C_32fc*, Ipp8u*);
typedef IppStatus (CV_STDCALL* ippiDFT_R_Func)(const Ipp32f* , int, Ipp32f* , int, const IppiDFTSpec_R_32f* , Ipp8u*);

template <typename Dft>
class Dft_C_IPPLoop_Invoker : public ParallelLoopBody
{
public:

    Dft_C_IPPLoop_Invoker(const uchar * _src, size_t _src_step, uchar * _dst, size_t _dst_step, int _width,
                          const Dft& _ippidft, int _norm_flag, bool *_ok) :
        ParallelLoopBody(),
        src(_src), src_step(_src_step), dst(_dst), dst_step(_dst_step), width(_width),
        ippidft(_ippidft), norm_flag(_norm_flag), ok(_ok)
    {
        *ok = true;
    }

    virtual void operator()(const Range& range) const CV_OVERRIDE
    {
        IppStatus status;
        Ipp8u* pBuffer = 0;
        Ipp8u* pMemInit= 0;
        int sizeBuffer=0;
        int sizeSpec=0;
        int sizeInit=0;

        IppiSize srcRoiSize = {width, 1};

        status = ippiDFTGetSize_C_32fc(srcRoiSize, norm_flag, ippAlgHintNone, &sizeSpec, &sizeInit, &sizeBuffer );
        if ( status < 0 )
        {
            *ok = false;
            return;
        }

        IppiDFTSpec_C_32fc* pDFTSpec = (IppiDFTSpec_C_32fc*)CV_IPP_MALLOC( sizeSpec );

        if ( sizeInit > 0 )
            pMemInit = (Ipp8u*)CV_IPP_MALLOC( sizeInit );

        if ( sizeBuffer > 0 )
            pBuffer = (Ipp8u*)CV_IPP_MALLOC( sizeBuffer );

        status = ippiDFTInit_C_32fc( srcRoiSize, norm_flag, ippAlgHintNone, pDFTSpec, pMemInit );

        if ( sizeInit > 0 )
            ippFree( pMemInit );

        if ( status < 0 )
        {
            ippFree( pDFTSpec );
            if ( sizeBuffer > 0 )
                ippFree( pBuffer );
            *ok = false;
            return;
        }

        for( int i = range.start; i < range.end; ++i)
            if(!ippidft((Ipp32fc*)(src + src_step * i), src_step, (Ipp32fc*)(dst + dst_step * i), dst_step,
                        pDFTSpec, (Ipp8u*)pBuffer))
            {
                *ok = false;
            }

        if ( sizeBuffer > 0 )
            ippFree( pBuffer );

        ippFree( pDFTSpec );
        CV_IMPL_ADD(CV_IMPL_IPP|CV_IMPL_MT);
    }

private:
    const uchar * src;
    size_t src_step;
    uchar * dst;
    size_t dst_step;
    int width;
    const Dft& ippidft;
    int norm_flag;
    bool *ok;

    const Dft_C_IPPLoop_Invoker& operator= (const Dft_C_IPPLoop_Invoker&);
};

template <typename Dft>
class Dft_R_IPPLoop_Invoker : public ParallelLoopBody
{
public:

    Dft_R_IPPLoop_Invoker(const uchar * _src, size_t _src_step, uchar * _dst, size_t _dst_step, int _width,
                          const Dft& _ippidft, int _norm_flag, bool *_ok) :
        ParallelLoopBody(),
        src(_src), src_step(_src_step), dst(_dst), dst_step(_dst_step), width(_width),
        ippidft(_ippidft), norm_flag(_norm_flag), ok(_ok)
    {
        *ok = true;
    }

    virtual void operator()(const Range& range) const CV_OVERRIDE
    {
        IppStatus status;
        Ipp8u* pBuffer = 0;
        Ipp8u* pMemInit= 0;
        int sizeBuffer=0;
        int sizeSpec=0;
        int sizeInit=0;

        IppiSize srcRoiSize = {width, 1};

        status = ippiDFTGetSize_R_32f(srcRoiSize, norm_flag, ippAlgHintNone, &sizeSpec, &sizeInit, &sizeBuffer );
        if ( status < 0 )
        {
            *ok = false;
            return;
        }

        IppiDFTSpec_R_32f* pDFTSpec = (IppiDFTSpec_R_32f*)CV_IPP_MALLOC( sizeSpec );

        if ( sizeInit > 0 )
            pMemInit = (Ipp8u*)CV_IPP_MALLOC( sizeInit );

        if ( sizeBuffer > 0 )
            pBuffer = (Ipp8u*)CV_IPP_MALLOC( sizeBuffer );

        status = ippiDFTInit_R_32f( srcRoiSize, norm_flag, ippAlgHintNone, pDFTSpec, pMemInit );

        if ( sizeInit > 0 )
            ippFree( pMemInit );

        if ( status < 0 )
        {
            ippFree( pDFTSpec );
            if ( sizeBuffer > 0 )
                ippFree( pBuffer );
            *ok = false;
            return;
        }

        for( int i = range.start; i < range.end; ++i)
            if(!ippidft((float*)(src + src_step * i), src_step, (float*)(dst + dst_step * i), dst_step,
                        pDFTSpec, (Ipp8u*)pBuffer))
            {
                *ok = false;
            }

        if ( sizeBuffer > 0 )
            ippFree( pBuffer );

        ippFree( pDFTSpec );
        CV_IMPL_ADD(CV_IMPL_IPP|CV_IMPL_MT);
    }

private:
    const uchar * src;
    size_t src_step;
    uchar * dst;
    size_t dst_step;
    int width;
    const Dft& ippidft;
    int norm_flag;
    bool *ok;

    const Dft_R_IPPLoop_Invoker& operator= (const Dft_R_IPPLoop_Invoker&);
};

template <typename Dft>
bool Dft_C_IPPLoop(const uchar * src, size_t src_step, uchar * dst, size_t dst_step, int width, int height, const Dft& ippidft, int norm_flag)
{
    bool ok;
    parallel_for_(Range(0, height), Dft_C_IPPLoop_Invoker<Dft>(src, src_step, dst, dst_step, width, ippidft, norm_flag, &ok), (width * height)/(double)(1<<16) );
    return ok;
}

template <typename Dft>
bool Dft_R_IPPLoop(const uchar * src, size_t src_step, uchar * dst, size_t dst_step, int width, int height, const Dft& ippidft, int norm_flag)
{
    bool ok;
    parallel_for_(Range(0, height), Dft_R_IPPLoop_Invoker<Dft>(src, src_step, dst, dst_step, width, ippidft, norm_flag, &ok), (width * height)/(double)(1<<16) );
    return ok;
}

struct IPPDFT_C_Functor
{
    IPPDFT_C_Functor(ippiDFT_C_Func _func) : ippiDFT_CToC_32fc_C1R(_func){}

    bool operator()(const Ipp32fc* src, size_t srcStep, Ipp32fc* dst, size_t dstStep, const IppiDFTSpec_C_32fc* pDFTSpec, Ipp8u* pBuffer) const
    {
        return ippiDFT_CToC_32fc_C1R ? CV_INSTRUMENT_FUN_IPP(ippiDFT_CToC_32fc_C1R, src, static_cast<int>(srcStep), dst, static_cast<int>(dstStep), pDFTSpec, pBuffer) >= 0 : false;
    }
private:
    ippiDFT_C_Func ippiDFT_CToC_32fc_C1R;
};

struct IPPDFT_R_Functor
{
    IPPDFT_R_Functor(ippiDFT_R_Func _func) : ippiDFT_PackToR_32f_C1R(_func){}

    bool operator()(const Ipp32f* src, size_t srcStep, Ipp32f* dst, size_t dstStep, const IppiDFTSpec_R_32f* pDFTSpec, Ipp8u* pBuffer) const
    {
        return ippiDFT_PackToR_32f_C1R ? CV_INSTRUMENT_FUN_IPP(ippiDFT_PackToR_32f_C1R, src, static_cast<int>(srcStep), dst, static_cast<int>(dstStep), pDFTSpec, pBuffer) >= 0 : false;
    }
private:
    ippiDFT_R_Func ippiDFT_PackToR_32f_C1R;
};

static bool ippi_DFT_C_32F(const uchar * src, size_t src_step, uchar * dst, size_t dst_step, int width, int height, bool inv, int norm_flag)
{
    CV_INSTRUMENT_REGION_IPP();

    IppStatus status;
    Ipp8u* pBuffer = 0;
    Ipp8u* pMemInit= 0;
    int sizeBuffer=0;
    int sizeSpec=0;
    int sizeInit=0;

    IppiSize srcRoiSize = {width, height};

    status = ippiDFTGetSize_C_32fc(srcRoiSize, norm_flag, ippAlgHintNone, &sizeSpec, &sizeInit, &sizeBuffer );
    if ( status < 0 )
        return false;

    IppiDFTSpec_C_32fc* pDFTSpec = (IppiDFTSpec_C_32fc*)CV_IPP_MALLOC( sizeSpec );

    if ( sizeInit > 0 )
        pMemInit = (Ipp8u*)CV_IPP_MALLOC( sizeInit );

    if ( sizeBuffer > 0 )
        pBuffer = (Ipp8u*)CV_IPP_MALLOC( sizeBuffer );

    status = ippiDFTInit_C_32fc( srcRoiSize, norm_flag, ippAlgHintNone, pDFTSpec, pMemInit );

    if ( sizeInit > 0 )
        ippFree( pMemInit );

    if ( status < 0 )
    {
        ippFree( pDFTSpec );
        if ( sizeBuffer > 0 )
            ippFree( pBuffer );
        return false;
    }

    if (!inv)
        status = CV_INSTRUMENT_FUN_IPP(ippiDFTFwd_CToC_32fc_C1R, (Ipp32fc*)src, static_cast<int>(src_step), (Ipp32fc*)dst, static_cast<int>(dst_step), pDFTSpec, pBuffer);
    else
        status = CV_INSTRUMENT_FUN_IPP(ippiDFTInv_CToC_32fc_C1R, (Ipp32fc*)src, static_cast<int>(src_step), (Ipp32fc*)dst, static_cast<int>(dst_step), pDFTSpec, pBuffer);

    if ( sizeBuffer > 0 )
        ippFree( pBuffer );

    ippFree( pDFTSpec );

    if(status >= 0)
    {
        CV_IMPL_ADD(CV_IMPL_IPP);
        return true;
    }
    return false;
}

static bool ippi_DFT_R_32F(const uchar * src, size_t src_step, uchar * dst, size_t dst_step, int width, int height, bool inv, int norm_flag)
{
    CV_INSTRUMENT_REGION_IPP();

    IppStatus status;
    Ipp8u* pBuffer = 0;
    Ipp8u* pMemInit= 0;
    int sizeBuffer=0;
    int sizeSpec=0;
    int sizeInit=0;

    IppiSize srcRoiSize = {width, height};

    status = ippiDFTGetSize_R_32f(srcRoiSize, norm_flag, ippAlgHintNone, &sizeSpec, &sizeInit, &sizeBuffer );
    if ( status < 0 )
        return false;

    IppiDFTSpec_R_32f* pDFTSpec = (IppiDFTSpec_R_32f*)CV_IPP_MALLOC( sizeSpec );

    if ( sizeInit > 0 )
        pMemInit = (Ipp8u*)CV_IPP_MALLOC( sizeInit );

    if ( sizeBuffer > 0 )
        pBuffer = (Ipp8u*)CV_IPP_MALLOC( sizeBuffer );

    status = ippiDFTInit_R_32f( srcRoiSize, norm_flag, ippAlgHintNone, pDFTSpec, pMemInit );

    if ( sizeInit > 0 )
        ippFree( pMemInit );

    if ( status < 0 )
    {
        ippFree( pDFTSpec );
        if ( sizeBuffer > 0 )
            ippFree( pBuffer );
        return false;
    }

    if (!inv)
        status = CV_INSTRUMENT_FUN_IPP(ippiDFTFwd_RToPack_32f_C1R, (float*)src, static_cast<int>(src_step), (float*)dst, static_cast<int>(dst_step), pDFTSpec, pBuffer);
    else
        status = CV_INSTRUMENT_FUN_IPP(ippiDFTInv_PackToR_32f_C1R, (float*)src, static_cast<int>(src_step), (float*)dst, static_cast<int>(dst_step), pDFTSpec, pBuffer);

    if ( sizeBuffer > 0 )
        ippFree( pBuffer );

    ippFree( pDFTSpec );

    if(status >= 0)
    {
        CV_IMPL_ADD(CV_IMPL_IPP);
        return true;
    }
    return false;
}

#endif
}

#ifdef HAVE_OPENCL

namespace cv
{

enum FftType
{
    R2R = 0, // real to CCS in case forward transform, CCS to real otherwise
    C2R = 1, // complex to real in case inverse transform
    R2C = 2, // real to complex in case forward transform
    C2C = 3  // complex to complex
};

struct OCL_FftPlan
{
private:
    UMat twiddles;
    String buildOptions;
    int thread_count;
    int dft_size;
    int dft_depth;
    bool status;

public:
    OCL_FftPlan(int _size, int _depth) : dft_size(_size), dft_depth(_depth), status(true)
    {
        CV_Assert( dft_depth == CV_32F || dft_depth == CV_64F );

        int min_radix;
        std::vector<int> radixes, blocks;
        ocl_getRadixes(dft_size, radixes, blocks, min_radix);
        thread_count = dft_size / min_radix;

        if (thread_count > (int) ocl::Device::getDefault().maxWorkGroupSize())
        {
            status = false;
            return;
        }

        // generate string with radix calls
        String radix_processing;
        int n = 1, twiddle_size = 0;
        for (size_t i=0; i<radixes.size(); i++)
        {
            int radix = radixes[i], block = blocks[i];
            if (block > 1)
                radix_processing += format("fft_radix%d_B%d(smem,twiddles+%d,ind,%d,%d);", radix, block, twiddle_size, n, dft_size/radix);
            else
                radix_processing += format("fft_radix%d(smem,twiddles+%d,ind,%d,%d);", radix, twiddle_size, n, dft_size/radix);
            twiddle_size += (radix-1)*n;
            n *= radix;
        }

        twiddles.create(1, twiddle_size, CV_MAKE_TYPE(dft_depth, 2));
        if (dft_depth == CV_32F)
            fillRadixTable<float>(twiddles, radixes);
        else
            fillRadixTable<double>(twiddles, radixes);

        buildOptions = format("-D LOCAL_SIZE=%d -D kercn=%d -D FT=%s -D CT=%s%s -D RADIX_PROCESS=%s",
                              dft_size, min_radix, ocl::typeToStr(dft_depth), ocl::typeToStr(CV_MAKE_TYPE(dft_depth, 2)),
                              dft_depth == CV_64F ? " -D DOUBLE_SUPPORT" : "", radix_processing.c_str());
    }

    bool enqueueTransform(InputArray _src, OutputArray _dst, int num_dfts, int flags, int fftType, bool rows = true) const
    {
        if (!status)
            return false;

        UMat src = _src.getUMat();
        UMat dst = _dst.getUMat();

        size_t globalsize[2];
        size_t localsize[2];
        String kernel_name;

        bool is1d = (flags & DFT_ROWS) != 0 || num_dfts == 1;
        bool inv = (flags & DFT_INVERSE) != 0;
        String options = buildOptions;

        if (rows)
        {
            globalsize[0] = thread_count; globalsize[1] = src.rows;
            localsize[0] = thread_count; localsize[1] = 1;
            kernel_name = !inv ? "fft_multi_radix_rows" : "ifft_multi_radix_rows";
            if ((is1d || inv) && (flags & DFT_SCALE))
                options += " -D DFT_SCALE";
        }
        else
        {
            globalsize[0] = num_dfts; globalsize[1] = thread_count;
            localsize[0] = 1; localsize[1] = thread_count;
            kernel_name = !inv ? "fft_multi_radix_cols" : "ifft_multi_radix_cols";
            if (flags & DFT_SCALE)
                options += " -D DFT_SCALE";
        }

        options += src.channels() == 1 ? " -D REAL_INPUT" : " -D COMPLEX_INPUT";
        options += dst.channels() == 1 ? " -D REAL_OUTPUT" : " -D COMPLEX_OUTPUT";
        options += is1d ? " -D IS_1D" : "";

        if (!inv)
        {
            if ((is1d && src.channels() == 1) || (rows && (fftType == R2R)))
                options += " -D NO_CONJUGATE";
        }
        else
        {
            if (rows && (fftType == C2R || fftType == R2R))
                options += " -D NO_CONJUGATE";
            if (dst.cols % 2 == 0)
                options += " -D EVEN";
        }

        ocl::Kernel k(kernel_name.c_str(), ocl::core::fft_oclsrc, options);
        if (k.empty())
            return false;

        k.args(ocl::KernelArg::ReadOnly(src), ocl::KernelArg::WriteOnly(dst), ocl::KernelArg::ReadOnlyNoSize(twiddles), thread_count, num_dfts);
        return k.run(2, globalsize, localsize, false);
    }

private:
    static void ocl_getRadixes(int cols, std::vector<int>& radixes, std::vector<int>& blocks, int& min_radix)
    {
        int factors[34];
        int nf = DFTFactorize(cols, factors);

        int n = 1;
        int factor_index = 0;
        min_radix = INT_MAX;

        // 2^n transforms
        if ((factors[factor_index] & 1) == 0)
        {
            for( ; n < factors[factor_index];)
            {
                int radix = 2, block = 1;
                if (8*n <= factors[0])
                    radix = 8;
                else if (4*n <= factors[0])
                {
                    radix = 4;
                    if (cols % 12 == 0)
                        block = 3;
                    else if (cols % 8 == 0)
                        block = 2;
                }
                else
                {
                    if (cols % 10 == 0)
                        block = 5;
                    else if (cols % 8 == 0)
                        block = 4;
                    else if (cols % 6 == 0)
                        block = 3;
                    else if (cols % 4 == 0)
                        block = 2;
                }

                radixes.push_back(radix);
                blocks.push_back(block);
                min_radix = min(min_radix, block*radix);
                n *= radix;
            }
            factor_index++;
        }

        // all the other transforms
        for( ; factor_index < nf; factor_index++)
        {
            int radix = factors[factor_index], block = 1;
            if (radix == 3)
            {
                if (cols % 12 == 0)
                    block = 4;
                else if (cols % 9 == 0)
                    block = 3;
                else if (cols % 6 == 0)
                    block = 2;
            }
            else if (radix == 5)
            {
                if (cols % 10 == 0)
                    block = 2;
            }
            radixes.push_back(radix);
            blocks.push_back(block);
            min_radix = min(min_radix, block*radix);
        }
    }

    template <typename T>
    static void fillRadixTable(UMat twiddles, const std::vector<int>& radixes)
    {
        Mat tw = twiddles.getMat(ACCESS_WRITE);
        T* ptr = tw.ptr<T>();
        int ptr_index = 0;

        int n = 1;
        for (size_t i=0; i<radixes.size(); i++)
        {
            int radix = radixes[i];
            n *= radix;

            for (int j=1; j<radix; j++)
            {
                double theta = -CV_2PI*j/n;

                for (int k=0; k<(n/radix); k++)
                {
                    ptr[ptr_index++] = (T) cos(k*theta);
                    ptr[ptr_index++] = (T) sin(k*theta);
                }
            }
        }
    }
};

class OCL_FftPlanCache
{
public:
    static OCL_FftPlanCache & getInstance()
    {
        CV_SINGLETON_LAZY_INIT_REF(OCL_FftPlanCache, new OCL_FftPlanCache())
    }

    Ptr<OCL_FftPlan> getFftPlan(int dft_size, int depth)
    {
        int key = (dft_size << 16) | (depth & 0xFFFF);
        std::map<int, Ptr<OCL_FftPlan> >::iterator f = planStorage.find(key);
        if (f != planStorage.end())
        {
            return f->second;
        }
        else
        {
            Ptr<OCL_FftPlan> newPlan = Ptr<OCL_FftPlan>(new OCL_FftPlan(dft_size, depth));
            planStorage[key] = newPlan;
            return newPlan;
        }
    }

    ~OCL_FftPlanCache()
    {
        planStorage.clear();
    }

protected:
    OCL_FftPlanCache() :
        planStorage()
    {
    }
    std::map<int, Ptr<OCL_FftPlan> > planStorage;
};

static bool ocl_dft_rows(InputArray _src, OutputArray _dst, int nonzero_rows, int flags, int fftType)
{
    int type = _src.type(), depth = CV_MAT_DEPTH(type);
    Ptr<OCL_FftPlan> plan = OCL_FftPlanCache::getInstance().getFftPlan(_src.cols(), depth);
    return plan->enqueueTransform(_src, _dst, nonzero_rows, flags, fftType, true);
}

static bool ocl_dft_cols(InputArray _src, OutputArray _dst, int nonzero_cols, int flags, int fftType)
{
    int type = _src.type(), depth = CV_MAT_DEPTH(type);
    Ptr<OCL_FftPlan> plan = OCL_FftPlanCache::getInstance().getFftPlan(_src.rows(), depth);
    return plan->enqueueTransform(_src, _dst, nonzero_cols, flags, fftType, false);
}

inline FftType determineFFTType(bool real_input, bool complex_input, bool real_output, bool complex_output, bool inv)
{
    // output format is not specified
    if (!real_output && !complex_output)
        complex_output = true;

    // input or output format is ambiguous
    if (real_input == complex_input || real_output == complex_output)
        CV_Error(Error::StsBadArg, "Invalid FFT input or output format");

    FftType result = real_input ? (real_output ? R2R : R2C) : (real_output ? C2R : C2C);

    // Forward Complex to CCS not supported
    if (result == C2R && !inv)
        result = C2C;

    // Inverse CCS to Complex not supported
    if (result == R2C && inv)
        result = R2R;

    return result;
}

static bool ocl_dft(InputArray _src, OutputArray _dst, int flags, int nonzero_rows)
{
    int type = _src.type(), cn = CV_MAT_CN(type), depth = CV_MAT_DEPTH(type);
    Size ssize = _src.size();
    bool doubleSupport = ocl::Device::getDefault().doubleFPConfig() > 0;

    if (!(cn == 1 || cn == 2)
        || !(depth == CV_32F || (depth == CV_64F && doubleSupport))
        || ((flags & DFT_REAL_OUTPUT) && (flags & DFT_COMPLEX_OUTPUT)))
        return false;

    // if is not a multiplication of prime numbers { 2, 3, 5 }
    if (ssize.area() != getOptimalDFTSize(ssize.area()))
        return false;

    UMat src = _src.getUMat();
    bool inv = (flags & DFT_INVERSE) != 0 ? 1 : 0;

    if( nonzero_rows <= 0 || nonzero_rows > _src.rows() )
        nonzero_rows = _src.rows();
    bool is1d = (flags & DFT_ROWS) != 0 || nonzero_rows == 1;

    FftType fftType = determineFFTType(cn == 1, cn == 2,
        (flags & DFT_REAL_OUTPUT) != 0, (flags & DFT_COMPLEX_OUTPUT) != 0, inv);

    UMat output;
    if (fftType == C2C || fftType == R2C)
    {
        // complex output
        _dst.create(src.size(), CV_MAKETYPE(depth, 2));
        output = _dst.getUMat();
    }
    else
    {
        // real output
        if (is1d)
        {
            _dst.create(src.size(), CV_MAKETYPE(depth, 1));
            output = _dst.getUMat();
        }
        else
        {
            _dst.create(src.size(), CV_MAKETYPE(depth, 1));
            output.create(src.size(), CV_MAKETYPE(depth, 2));
        }
    }

    bool result = false;
    if (!inv)
    {
        int nonzero_cols = fftType == R2R ? output.cols/2 + 1 : output.cols;
        result = ocl_dft_rows(src, output, nonzero_rows, flags, fftType);
        if (!is1d)
            result = result && ocl_dft_cols(output, _dst, nonzero_cols, flags, fftType);
    }
    else
    {
        if (fftType == C2C)
        {
            // complex output
            result = ocl_dft_rows(src, output, nonzero_rows, flags, fftType);
            if (!is1d)
                result = result && ocl_dft_cols(output, output, output.cols, flags, fftType);
        }
        else
        {
            if (is1d)
            {
                result = ocl_dft_rows(src, output, nonzero_rows, flags, fftType);
            }
            else
            {
                int nonzero_cols = src.cols/2 + 1;
                result = ocl_dft_cols(src, output, nonzero_cols, flags, fftType);
                result = result && ocl_dft_rows(output, _dst, nonzero_rows, flags, fftType);
            }
        }
    }
    return result;
}

} // namespace cv;

#endif

#ifdef HAVE_CLAMDFFT

namespace cv {

#define CLAMDDFT_Assert(func) \
    { \
        clfftStatus s = (func); \
        CV_Assert(s == CLFFT_SUCCESS); \
    }

class PlanCache
{
    struct FftPlan
    {
        FftPlan(const Size & _dft_size, int _src_step, int _dst_step, bool _doubleFP, bool _inplace, int _flags, FftType _fftType) :
            dft_size(_dft_size), src_step(_src_step), dst_step(_dst_step),
            doubleFP(_doubleFP), inplace(_inplace), flags(_flags), fftType(_fftType),
            context((cl_context)ocl::Context::getDefault().ptr()), plHandle(0)
        {
            bool dft_inverse = (flags & DFT_INVERSE) != 0;
            bool dft_scale = (flags & DFT_SCALE) != 0;
            bool dft_rows = (flags & DFT_ROWS) != 0;

            clfftLayout inLayout = CLFFT_REAL, outLayout = CLFFT_REAL;
            clfftDim dim = dft_size.height == 1 || dft_rows ? CLFFT_1D : CLFFT_2D;

            size_t batchSize = dft_rows ? dft_size.height : 1;
            size_t clLengthsIn[3] = { (size_t)dft_size.width, dft_rows ? 1 : (size_t)dft_size.height, 1 };
            size_t clStridesIn[3] = { 1, 1, 1 };
            size_t clStridesOut[3]  = { 1, 1, 1 };
            int elemSize = doubleFP ? sizeof(double) : sizeof(float);

            switch (fftType)
            {
            case C2C:
                inLayout = CLFFT_COMPLEX_INTERLEAVED;
                outLayout = CLFFT_COMPLEX_INTERLEAVED;
                clStridesIn[1] = src_step / (elemSize << 1);
                clStridesOut[1] = dst_step / (elemSize << 1);
                break;
            case R2C:
                inLayout = CLFFT_REAL;
                outLayout = CLFFT_HERMITIAN_INTERLEAVED;
                clStridesIn[1] = src_step / elemSize;
                clStridesOut[1] = dst_step / (elemSize << 1);
                break;
            case C2R:
                inLayout = CLFFT_HERMITIAN_INTERLEAVED;
                outLayout = CLFFT_REAL;
                clStridesIn[1] = src_step / (elemSize << 1);
                clStridesOut[1] = dst_step / elemSize;
                break;
            case R2R:
            default:
                CV_Error(Error::StsNotImplemented, "AMD Fft does not support this type");
                break;
            }

            clStridesIn[2] = dft_rows ? clStridesIn[1] : dft_size.width * clStridesIn[1];
            clStridesOut[2] = dft_rows ? clStridesOut[1] : dft_size.width * clStridesOut[1];

            CLAMDDFT_Assert(clfftCreateDefaultPlan(&plHandle, (cl_context)ocl::Context::getDefault().ptr(), dim, clLengthsIn))

            // setting plan properties
            CLAMDDFT_Assert(clfftSetPlanPrecision(plHandle, doubleFP ? CLFFT_DOUBLE : CLFFT_SINGLE));
            CLAMDDFT_Assert(clfftSetResultLocation(plHandle, inplace ? CLFFT_INPLACE : CLFFT_OUTOFPLACE))
            CLAMDDFT_Assert(clfftSetLayout(plHandle, inLayout, outLayout))
            CLAMDDFT_Assert(clfftSetPlanBatchSize(plHandle, batchSize))
            CLAMDDFT_Assert(clfftSetPlanInStride(plHandle, dim, clStridesIn))
            CLAMDDFT_Assert(clfftSetPlanOutStride(plHandle, dim, clStridesOut))
            CLAMDDFT_Assert(clfftSetPlanDistance(plHandle, clStridesIn[dim], clStridesOut[dim]))

            float scale = dft_scale ? 1.0f / (dft_rows ? dft_size.width : dft_size.area()) : 1.0f;
            CLAMDDFT_Assert(clfftSetPlanScale(plHandle, dft_inverse ? CLFFT_BACKWARD : CLFFT_FORWARD, scale))

            // ready to bake
            cl_command_queue queue = (cl_command_queue)ocl::Queue::getDefault().ptr();
            CLAMDDFT_Assert(clfftBakePlan(plHandle, 1, &queue, NULL, NULL))
        }

        ~FftPlan()
        {
            // Do not tear down clFFT.
            // The user application may still use clFFT even after OpenCV is unloaded.
            /*clfftDestroyPlan(&plHandle);*/
        }

        friend class PlanCache;

    private:
        Size dft_size;
        int src_step, dst_step;
        bool doubleFP;
        bool inplace;
        int flags;
        FftType fftType;

        cl_context context;
        clfftPlanHandle plHandle;
    };

public:
    static PlanCache & getInstance()
    {
        CV_SINGLETON_LAZY_INIT_REF(PlanCache, new PlanCache())
    }

    clfftPlanHandle getPlanHandle(const Size & dft_size, int src_step, int dst_step, bool doubleFP,
                                  bool inplace, int flags, FftType fftType)
    {
        cl_context currentContext = (cl_context)ocl::Context::getDefault().ptr();

        for (size_t i = 0, size = planStorage.size(); i < size; ++i)
        {
            const FftPlan * const plan = planStorage[i];

            if (plan->dft_size == dft_size &&
                plan->flags == flags &&
                plan->src_step == src_step &&
                plan->dst_step == dst_step &&
                plan->doubleFP == doubleFP &&
                plan->fftType == fftType &&
                plan->inplace == inplace)
            {
                if (plan->context != currentContext)
                {
                    planStorage.erase(planStorage.begin() + i);
                    break;
                }

                return plan->plHandle;
            }
        }

        // no baked plan is found, so let's create a new one
        Ptr<FftPlan> newPlan = Ptr<FftPlan>(new FftPlan(dft_size, src_step, dst_step, doubleFP, inplace, flags, fftType));
        planStorage.push_back(newPlan);

        return newPlan->plHandle;
    }

    ~PlanCache()
    {
        planStorage.clear();
    }

protected:
    PlanCache() :
        planStorage()
    {
    }

    std::vector<Ptr<FftPlan> > planStorage;
};

extern "C" {

static void CL_CALLBACK oclCleanupCallback(cl_event e, cl_int, void *p)
{
    UMatData * u = (UMatData *)p;

    if( u && CV_XADD(&u->urefcount, -1) == 1 )
        u->currAllocator->deallocate(u);
    u = 0;

    clReleaseEvent(e), e = 0;
}

}

static bool ocl_dft_amdfft(InputArray _src, OutputArray _dst, int flags)
{
    int type = _src.type(), depth = CV_MAT_DEPTH(type), cn = CV_MAT_CN(type);
    Size ssize = _src.size();

    bool doubleSupport = ocl::Device::getDefault().doubleFPConfig() > 0;
    if ( (!doubleSupport && depth == CV_64F) ||
         !(type == CV_32FC1 || type == CV_32FC2 || type == CV_64FC1 || type == CV_64FC2) ||
         _src.offset() != 0)
        return false;

    // if is not a multiplication of prime numbers { 2, 3, 5 }
    if (ssize.area() != getOptimalDFTSize(ssize.area()))
        return false;

    int dst_complex_input = cn == 2 ? 1 : 0;
    bool dft_inverse = (flags & DFT_INVERSE) != 0 ? 1 : 0;
    int dft_complex_output = (flags & DFT_COMPLEX_OUTPUT) != 0;
    bool dft_real_output = (flags & DFT_REAL_OUTPUT) != 0;

    CV_Assert(dft_complex_output + dft_real_output < 2);
    FftType fftType = (FftType)(dst_complex_input << 0 | dft_complex_output << 1);

    switch (fftType)
    {
    case C2C:
        _dst.create(ssize.height, ssize.width, CV_MAKE_TYPE(depth, 2));
        break;
    case R2C: // TODO implement it if possible
    case C2R: // TODO implement it if possible
    case R2R: // AMD Fft does not support this type
    default:
        return false;
    }

    UMat src = _src.getUMat(), dst = _dst.getUMat();
    bool inplace = src.u == dst.u;

    clfftPlanHandle plHandle = PlanCache::getInstance().
            getPlanHandle(ssize, (int)src.step, (int)dst.step,
                          depth == CV_64F, inplace, flags, fftType);

    // get the bufferSize
    size_t bufferSize = 0;
    CLAMDDFT_Assert(clfftGetTmpBufSize(plHandle, &bufferSize))
    UMat tmpBuffer(1, (int)bufferSize, CV_8UC1);

    cl_mem srcarg = (cl_mem)src.handle(ACCESS_READ);
    cl_mem dstarg = (cl_mem)dst.handle(ACCESS_RW);

    cl_command_queue queue = (cl_command_queue)ocl::Queue::getDefault().ptr();
    cl_event e = 0;

    CLAMDDFT_Assert(clfftEnqueueTransform(plHandle, dft_inverse ? CLFFT_BACKWARD : CLFFT_FORWARD,
                                          1, &queue, 0, NULL, &e,
                                          &srcarg, &dstarg, (cl_mem)tmpBuffer.handle(ACCESS_RW)))

    tmpBuffer.addref();
    clSetEventCallback(e, CL_COMPLETE, oclCleanupCallback, tmpBuffer.u);
    return true;
}

#undef DFT_ASSERT

}

#endif // HAVE_CLAMDFFT

namespace cv
{

template <typename T>
static void complementComplex(T * ptr, size_t step, int n, int len, int dft_dims)
{
    T* p0 = (T*)ptr;
    size_t dstep = step/sizeof(p0[0]);
    for(int i = 0; i < len; i++ )
    {
        T* p = p0 + dstep*i;
        T* q = dft_dims == 1 || i == 0 || i*2 == len ? p : p0 + dstep*(len-i);

        for( int j = 1; j < (n+1)/2; j++ )
        {
            p[(n-j)*2] = q[j*2];
            p[(n-j)*2+1] = -q[j*2+1];
        }
    }
}

static void complementComplexOutput(int depth, uchar * ptr, size_t step, int count, int len, int dft_dims)
{
    if( depth == CV_32F )
        complementComplex((float*)ptr, step, count, len, dft_dims);
    else
        complementComplex((double*)ptr, step, count, len, dft_dims);
}

enum DftMode {
    InvalidDft = 0,
    FwdRealToCCS,
    FwdRealToComplex,
    FwdComplexToComplex,
    InvCCSToReal,
    InvComplexToReal,
    InvComplexToComplex,
};

enum DftDims {
    InvalidDim = 0,
    OneDim,
    OneDimColWise,
    TwoDims
};

inline const char * modeName(DftMode m)
{
    switch (m)
    {
    case InvalidDft: return "InvalidDft";
    case FwdRealToCCS: return "FwdRealToCCS";
    case FwdRealToComplex: return "FwdRealToComplex";
    case FwdComplexToComplex: return "FwdComplexToComplex";
    case InvCCSToReal: return "InvCCSToReal";
    case InvComplexToReal: return "InvComplexToReal";
    case InvComplexToComplex: return "InvComplexToComplex";
    }
    return 0;
}

inline const char * dimsName(DftDims d)
{
    switch (d)
    {
    case InvalidDim: return "InvalidDim";
    case OneDim: return "OneDim";
    case OneDimColWise: return "OneDimColWise";
    case TwoDims: return "TwoDims";
    };
    return 0;
}

template <typename T>
inline bool isInv(T mode)
{
    switch ((DftMode)mode)
    {
        case InvCCSToReal:
        case InvComplexToReal:
        case InvComplexToComplex: return true;
        default: return false;
    }
}

inline DftMode determineMode(bool inv, int cn1, int cn2)
{
    if (!inv)
    {
        if (cn1 == 1 && cn2 == 1)
            return FwdRealToCCS;
        else if (cn1 == 1 && cn2 == 2)
            return FwdRealToComplex;
        else if (cn1 == 2 && cn2 == 2)
            return FwdComplexToComplex;
    }
    else
    {
        if (cn1 == 1 && cn2 == 1)
            return InvCCSToReal;
        else if (cn1 == 2 && cn2 == 1)
            return InvComplexToReal;
        else if (cn1 == 2 && cn2 == 2)
            return InvComplexToComplex;
    }
    return InvalidDft;
}


inline DftDims determineDims(int rows, int cols, bool isRowWise, bool isContinuous)
{
    // printf("%d x %d (%d, %d)\n", rows, cols, isRowWise, isContinuous);
    if (isRowWise)
        return OneDim;
    if (cols == 1 && rows > 1) // one-column-shaped input
    {
        if (isContinuous)
            return OneDim;
        else
            return OneDimColWise;
    }
    if (rows == 1)
        return OneDim;
    if (cols > 1 && rows > 1)
        return TwoDims;
    return InvalidDim;
}

class OcvDftImpl CV_FINAL : public hal::DFT2D
{
protected:
    Ptr<hal::DFT1D> contextA;
    Ptr<hal::DFT1D> contextB;
    bool needBufferA;
    bool needBufferB;
    bool inv;
    int width;
    int height;
    DftMode mode;
    int elem_size;
    int complex_elem_size;
    int depth;
    bool real_transform;
    int nonzero_rows;
    bool isRowTransform;
    bool isScaled;
    std::vector<int> stages;
    bool useIpp;
    int src_channels;
    int dst_channels;

    AutoBuffer<uchar> tmp_bufA;
    AutoBuffer<uchar> tmp_bufB;
    AutoBuffer<uchar> buf0;
    AutoBuffer<uchar> buf1;

public:
    OcvDftImpl()
    {
        needBufferA = false;
        needBufferB = false;
        inv = false;
        width = 0;
        height = 0;
        mode = InvalidDft;
        elem_size = 0;
        complex_elem_size = 0;
        depth = 0;
        real_transform = false;
        nonzero_rows = 0;
        isRowTransform = false;
        isScaled = false;
        useIpp = false;
        src_channels = 0;
        dst_channels = 0;
    }

    void init(int _width, int _height, int _depth, int _src_channels, int _dst_channels, int flags, int _nonzero_rows)
    {
        bool isComplex = _src_channels != _dst_channels;
        nonzero_rows = _nonzero_rows;
        width = _width;
        height = _height;
        depth = _depth;
        src_channels = _src_channels;
        dst_channels = _dst_channels;
        bool isInverse = (flags & CV_HAL_DFT_INVERSE) != 0;
        bool isInplace = (flags & CV_HAL_DFT_IS_INPLACE) != 0;
        bool isContinuous = (flags & CV_HAL_DFT_IS_CONTINUOUS) != 0;
        mode = determineMode(isInverse, _src_channels, _dst_channels);
        inv = isInverse;
        isRowTransform = (flags & CV_HAL_DFT_ROWS) != 0;
        isScaled = (flags & CV_HAL_DFT_SCALE) != 0;
        needBufferA = false;
        needBufferB = false;
        real_transform = (mode != FwdComplexToComplex && mode != InvComplexToComplex);

        elem_size = (depth == CV_32F) ? sizeof(float) : sizeof(double);
        complex_elem_size = elem_size * 2;
        if( !real_transform )
            elem_size = complex_elem_size;

#if defined USE_IPP_DFT
        CV_IPP_CHECK()
        {
            if (nonzero_rows == 0 && depth == CV_32F && ((width * height)>(int)(1<<6)))
            {
                if (mode == FwdComplexToComplex || mode == InvComplexToComplex || mode == FwdRealToCCS || mode == InvCCSToReal)
                {
                    useIpp = true;
                    return;
                }
            }
        }
#endif

        DftDims dims = determineDims(height, width, isRowTransform, isContinuous);
        if (dims == TwoDims)
        {
            stages.resize(2);
            if (mode == InvCCSToReal || mode == InvComplexToReal)
            {
                stages[0] = 1;
                stages[1] = 0;
            }
            else
            {
                stages[0] = 0;
                stages[1] = 1;
            }
        }
        else
        {
            stages.resize(1);
            if (dims == OneDimColWise)
                stages[0] = 1;
            else
                stages[0] = 0;
        }

        for(uint stageIndex = 0; stageIndex < stages.size(); ++stageIndex)
        {
            if (stageIndex == 1)
            {
                isInplace = true;
                isComplex = false;
            }

            int stage = stages[stageIndex];
            bool isLastStage = (stageIndex + 1 == stages.size());

            int len, count;

            int f = 0;
            if (inv)
                f |= CV_HAL_DFT_INVERSE;
            if (isScaled)
                f |= CV_HAL_DFT_SCALE;
            if (isRowTransform)
                f |= CV_HAL_DFT_ROWS;
            if (isComplex)
                f |= CV_HAL_DFT_COMPLEX_OUTPUT;
            if (real_transform)
                f |= CV_HAL_DFT_REAL_OUTPUT;
            if (!isLastStage)
                f |= CV_HAL_DFT_TWO_STAGE;

            if( stage == 0 ) // row-wise transform
            {
                if (width == 1 && !isRowTransform )
                {
                    len = height;
                    count = width;
                }
                else
                {
                    len = width;
                    count = height;
                }
                needBufferA = isInplace;
                contextA = hal::DFT1D::create(len, count, depth, f, &needBufferA);
                if (needBufferA)
                    tmp_bufA.allocate(len * complex_elem_size);
            }
            else
            {
                len = height;
                count = width;
                f |= CV_HAL_DFT_STAGE_COLS;
                needBufferB = isInplace;
                contextB = hal::DFT1D::create(len, count, depth, f, &needBufferB);
                if (needBufferB)
                    tmp_bufB.allocate(len * complex_elem_size);

                buf0.allocate(len * complex_elem_size);
                buf1.allocate(len * complex_elem_size);
            }
        }
    }

    void apply(const uchar * src, size_t src_step, uchar * dst, size_t dst_step) CV_OVERRIDE
    {
#if defined USE_IPP_DFT
        if (useIpp)
        {
            int ipp_norm_flag = !isScaled ? 8 : inv ? 2 : 1;
            if (!isRowTransform)
            {
                if (mode == FwdComplexToComplex || mode == InvComplexToComplex)
                {
                    if (ippi_DFT_C_32F(src, src_step, dst, dst_step, width, height, inv, ipp_norm_flag))
                    {
                        CV_IMPL_ADD(CV_IMPL_IPP);
                        return;
                    }
                    setIppErrorStatus();
                }
                else if (mode == FwdRealToCCS || mode == InvCCSToReal)
                {
                    if (ippi_DFT_R_32F(src, src_step, dst, dst_step, width, height, inv, ipp_norm_flag))
                    {
                        CV_IMPL_ADD(CV_IMPL_IPP);
                        return;
                    }
                    setIppErrorStatus();
                }
            }
            else
            {
                if (mode == FwdComplexToComplex || mode == InvComplexToComplex)
                {
                    ippiDFT_C_Func ippiFunc = inv ? (ippiDFT_C_Func)ippiDFTInv_CToC_32fc_C1R : (ippiDFT_C_Func)ippiDFTFwd_CToC_32fc_C1R;
                    if (Dft_C_IPPLoop(src, src_step, dst, dst_step, width, height, IPPDFT_C_Functor(ippiFunc),ipp_norm_flag))
                    {
                        CV_IMPL_ADD(CV_IMPL_IPP|CV_IMPL_MT);
                        return;
                    }
                    setIppErrorStatus();
                }
                else if (mode == FwdRealToCCS || mode == InvCCSToReal)
                {
                    ippiDFT_R_Func ippiFunc = inv ? (ippiDFT_R_Func)ippiDFTInv_PackToR_32f_C1R : (ippiDFT_R_Func)ippiDFTFwd_RToPack_32f_C1R;
                    if (Dft_R_IPPLoop(src, src_step, dst, dst_step, width, height, IPPDFT_R_Functor(ippiFunc),ipp_norm_flag))
                    {
                        CV_IMPL_ADD(CV_IMPL_IPP|CV_IMPL_MT);
                        return;
                    }
                    setIppErrorStatus();
                }
            }
            return;
        }
#endif

        for(uint stageIndex = 0; stageIndex < stages.size(); ++stageIndex)
        {
            int stage_src_channels = src_channels;
            int stage_dst_channels = dst_channels;

            if (stageIndex == 1)
            {
                src = dst;
                src_step = dst_step;
                stage_src_channels = stage_dst_channels;
            }

            int stage = stages[stageIndex];
            bool isLastStage = (stageIndex + 1 == stages.size());
            bool isComplex = stage_src_channels != stage_dst_channels;

            if( stage == 0 )
                rowDft(src, src_step, dst, dst_step, isComplex, isLastStage);
            else
                colDft(src, src_step, dst, dst_step, stage_src_channels, stage_dst_channels, isLastStage);
        }
    }

protected:

    void rowDft(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step, bool isComplex, bool isLastStage)
    {
        int len, count;
        if (width == 1 && !isRowTransform )
        {
            len = height;
            count = width;
        }
        else
        {
            len = width;
            count = height;
        }
        int dptr_offset = 0;
        int dst_full_len = len*elem_size;

        if( needBufferA )
        {
            if (mode == FwdRealToCCS && (len & 1) && len > 1)
                dptr_offset = elem_size;
        }

        if( !inv && isComplex )
            dst_full_len += (len & 1) ? elem_size : complex_elem_size;

        int nz = nonzero_rows;
        if( nz <= 0 || nz > count )
            nz = count;

        int i;
        for( i = 0; i < nz; i++ )
        {
            const uchar* sptr = src_data + src_step * i;
            uchar* dptr0 = dst_data + dst_step * i;
            uchar* dptr = dptr0;

            if( needBufferA )
                dptr = tmp_bufA.data();

            contextA->apply(sptr, dptr);

            if( needBufferA )
                memcpy( dptr0, dptr + dptr_offset, dst_full_len );
        }

        for( ; i < count; i++ )
        {
            uchar* dptr0 = dst_data + dst_step * i;
            memset( dptr0, 0, dst_full_len );
        }
        if(isLastStage &&  mode == FwdRealToComplex)
            complementComplexOutput(depth, dst_data, dst_step, len, nz, 1);
    }

    void colDft(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step, int stage_src_channels, int stage_dst_channels, bool isLastStage)
    {
        int len = height;
        int count = width;
        int a = 0, b = count;
        uchar *dbuf0, *dbuf1;
        const uchar* sptr0 = src_data;
        uchar* dptr0 = dst_data;

        dbuf0 = buf0.data(), dbuf1 = buf1.data();

        if( needBufferB )
        {
            dbuf1 = tmp_bufB.data();
            dbuf0 = buf1.data();
        }

        if( real_transform )
        {
            int even;
            a = 1;
            even = (count & 1) == 0;
            b = (count+1)/2;
            if( !inv )
            {
                memset( buf0.data(), 0, len*complex_elem_size );
                CopyColumn( sptr0, src_step, buf0.data(), complex_elem_size, len, elem_size );
                sptr0 += stage_dst_channels*elem_size;
                if( even )
                {
                    memset( buf1.data(), 0, len*complex_elem_size );
                    CopyColumn( sptr0 + (count-2)*elem_size, src_step,
                                buf1.data(), complex_elem_size, len, elem_size );
                }
            }
            else if( stage_src_channels == 1 )
            {
                CopyColumn( sptr0, src_step, buf0.data(), elem_size, len, elem_size );
                ExpandCCS( buf0.data(), len, elem_size );
                if( even )
                {
                    CopyColumn( sptr0 + (count-1)*elem_size, src_step,
                                buf1.data(), elem_size, len, elem_size );
                    ExpandCCS( buf1.data(), len, elem_size );
                }
                sptr0 += elem_size;
            }
            else
            {
                CopyColumn( sptr0, src_step, buf0.data(), complex_elem_size, len, complex_elem_size );
                if( even )
                {
                    CopyColumn( sptr0 + b*complex_elem_size, src_step,
                                   buf1.data(), complex_elem_size, len, complex_elem_size );
                }
                sptr0 += complex_elem_size;
            }

            if( even )
                contextB->apply(buf1.data(), dbuf1);
            contextB->apply(buf0.data(), dbuf0);

            if( stage_dst_channels == 1 )
            {
                if( !inv )
                {
                    // copy the half of output vector to the first/last column.
                    // before doing that, defgragment the vector
                    memcpy( dbuf0 + elem_size, dbuf0, elem_size );
                    CopyColumn( dbuf0 + elem_size, elem_size, dptr0,
                                   dst_step, len, elem_size );
                    if( even )
                    {
                        memcpy( dbuf1 + elem_size, dbuf1, elem_size );
                        CopyColumn( dbuf1 + elem_size, elem_size,
                                       dptr0 + (count-1)*elem_size,
                                       dst_step, len, elem_size );
                    }
                    dptr0 += elem_size;
                }
                else
                {
                    // copy the real part of the complex vector to the first/last column
                    CopyColumn( dbuf0, complex_elem_size, dptr0, dst_step, len, elem_size );
                    if( even )
                        CopyColumn( dbuf1, complex_elem_size, dptr0 + (count-1)*elem_size,
                                       dst_step, len, elem_size );
                    dptr0 += elem_size;
                }
            }
            else
            {
                CV_Assert( !inv );
                CopyColumn( dbuf0, complex_elem_size, dptr0,
                               dst_step, len, complex_elem_size );
                if( even )
                    CopyColumn( dbuf1, complex_elem_size,
                                   dptr0 + b*complex_elem_size,
                                   dst_step, len, complex_elem_size );
                dptr0 += complex_elem_size;
            }
        }

        for(int i = a; i < b; i += 2 )
        {
            if( i+1 < b )
            {
                CopyFrom2Columns( sptr0, src_step, buf0.data(), buf1.data(), len, complex_elem_size );
                contextB->apply(buf1.data(), dbuf1);
            }
            else
                CopyColumn( sptr0, src_step, buf0.data(), complex_elem_size, len, complex_elem_size );

            contextB->apply(buf0.data(), dbuf0);

            if( i+1 < b )
                CopyTo2Columns( dbuf0, dbuf1, dptr0, dst_step, len, complex_elem_size );
            else
                CopyColumn( dbuf0, complex_elem_size, dptr0, dst_step, len, complex_elem_size );
            sptr0 += 2*complex_elem_size;
            dptr0 += 2*complex_elem_size;
        }
        if(isLastStage && mode == FwdRealToComplex)
            complementComplexOutput(depth, dst_data, dst_step, count, len, 2);
    }
};

class OcvDftBasicImpl CV_FINAL : public hal::DFT1D
{
public:
    OcvDftOptions opt;
    int _factors[34];
    AutoBuffer<uchar> wave_buf;
    AutoBuffer<int>
... [Content truncated - file is very large]
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **FftPlan**: A class/struct defined in this file
- **Dft_C_IPPLoop_Invoker**: A class/struct defined in this file
- **IPPDFT_C_Functor**: A class/struct defined in this file
- **DFT_VecR4**: A class/struct defined in this file
- **PlanCache**: A class/struct defined in this file
- **OcvDftBasicImpl**: A class/struct defined in this file
- **OCL_FftPlan**: A class/struct defined in this file
- **IPPDFT_R_Functor**: A class/struct defined in this file
- **OCL_FftPlanCache**: A class/struct defined in this file
- **OcvDftOptions**: A class/struct defined in this file
- **DFT_VecR2**: A class/struct defined in this file
- **ReplacementDCT2D**: A class/struct defined in this file
- **DctIPPLoop_Invoker**: A class/struct defined in this file
- **ReplacementDFT1D**: A class/struct defined in this file
- **DFT_R5**: A class/struct defined in this file
- **Dft_R_IPPLoop_Invoker**: A class/struct defined in this file
- **ReplacementDFT2D**: A class/struct defined in this file
- **OcvDctImpl**: A class/struct defined in this file
- **Constants**: A class/struct defined in this file
- **OcvDftImpl**: A class/struct defined in this file
- **DFT_R3**: A class/struct defined in this file
- **DFT_R2**: A class/struct defined in this file
- **DFT_VecR3**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_EXCLUDE_C_API()**: A function/method defined in this file
- **IppStatus()**: A function/method defined in this file
- **USE_IPP_DFT()**: A function/method defined in this file
- **IPP_RETURN()**: A function/method defined in this file
- **void()**: A function/method defined in this file
- **IPP_RELEASE()**: A function/method defined in this file
- **MUL_SPECTRUMS_COL()**: A function/method defined in this file
- **DFT_ASSERT()**: A function/method defined in this file
- **HAVE_CLAMDFFT()**: A function/method defined in this file
- **HAVE_IPP()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **MUL_SPECTRUMS_ROW()**: A function/method defined in this file
- **VAL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencl_kernels_core.hpp`
- `opencv2/core/opencl/runtime/opencl_clfft.hpp`
- `opencv2/core/opencl/runtime/opencl_core.hpp`
- `precomp.hpp`
- `map`

**Python Imports:**
- `index`
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

