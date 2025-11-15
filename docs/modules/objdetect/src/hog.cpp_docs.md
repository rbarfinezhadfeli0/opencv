# Documentation for `modules/objdetect/src/hog.cpp`

## File Metadata

- **Full Path**: `modules/objdetect/src/hog.cpp`
- **File Name**: `hog.cpp`
- **File Size**: 166,825 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/objdetect/src/hog.cpp](../../../modules/objdetect/src/hog.cpp)

## Purpose and Role

This file is located in the `modules/objdetect/src` directory and serves as part of the OpenCV library infrastructure.

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

#include "precomp.hpp"
#include "cascadedetect.hpp"
#include "opencv2/core/hal/intrin.hpp"
#include "opencl_kernels_objdetect.hpp"

#include <cstdio>
#include <iterator>
#include <limits>

/****************************************************************************************\
      The code below is implementation of HOG (Histogram-of-Oriented Gradients)
      descriptor and object detection, introduced by Navneet Dalal and Bill Triggs.

      The computed feature vectors are compatible with the
      INRIA Object Detection and Localization Toolkit
      (http://pascal.inrialpes.fr/soft/olt/)
\****************************************************************************************/

namespace cv
{

#define NTHREADS 256

static int numPartsWithin(int size, int part_size, int stride)
{
    CV_Assert(stride != 0);
    return (size - part_size + stride) / stride;
}

static Size numPartsWithin(cv::Size size, cv::Size part_size,
                                                cv::Size stride)
{
    return Size(numPartsWithin(size.width, part_size.width, stride.width),
        numPartsWithin(size.height, part_size.height, stride.height));
}

static size_t getBlockHistogramSize(Size block_size, Size cell_size, int nbins)
{
    CV_Assert(!cell_size.empty());
    Size cells_per_block = Size(block_size.width / cell_size.width,
                                block_size.height / cell_size.height);
    return (size_t)(nbins * cells_per_block.area());
}

size_t HOGDescriptor::getDescriptorSize() const
{
    CV_Assert(!cellSize.empty());
    CV_Assert(!blockStride.empty());

    CV_Assert(blockSize.width % cellSize.width == 0 &&
        blockSize.height % cellSize.height == 0);
    CV_Assert((winSize.width - blockSize.width) % blockStride.width == 0 &&
        (winSize.height - blockSize.height) % blockStride.height == 0 );

    return (size_t)nbins*
        (blockSize.width/cellSize.width)*
        (blockSize.height/cellSize.height)*
        ((winSize.width - blockSize.width)/blockStride.width + 1)*
        ((winSize.height - blockSize.height)/blockStride.height + 1);
}

double HOGDescriptor::getWinSigma() const
{
    return winSigma > 0 ? winSigma : (blockSize.width + blockSize.height)/8.;
}

bool HOGDescriptor::checkDetectorSize() const
{
    size_t detectorSize = svmDetector.size(), descriptorSize = getDescriptorSize();
    return detectorSize == 0 ||
        detectorSize == descriptorSize ||
        detectorSize == descriptorSize + 1;
}

void HOGDescriptor::setSVMDetector(InputArray _svmDetector)
{
    _svmDetector.getMat().convertTo(svmDetector, CV_32F);
    CV_Assert(checkDetectorSize());

    if (_svmDetector.empty())
    {
        oclSvmDetector = UMat();
        return;
    }

    Mat detector_reordered(1, (int)svmDetector.size(), CV_32FC1);

    size_t block_hist_size = getBlockHistogramSize(blockSize, cellSize, nbins);
    cv::Size blocks_per_img = numPartsWithin(winSize, blockSize, blockStride);

    for (int i = 0; i < blocks_per_img.height; ++i)
        for (int j = 0; j < blocks_per_img.width; ++j)
        {
            const float *src = &svmDetector[0] + (j * blocks_per_img.height + i) * block_hist_size;
            float *dst = detector_reordered.ptr<float>() + (i * blocks_per_img.width + j) * block_hist_size;
            for (size_t k = 0; k < block_hist_size; ++k)
                dst[k] = src[k];
        }
    size_t descriptor_size = getDescriptorSize();
    free_coef = svmDetector.size() > descriptor_size ? svmDetector[descriptor_size] : 0;
    detector_reordered.copyTo(oclSvmDetector);
}

#define CV_TYPE_NAME_HOG_DESCRIPTOR "opencv-object-detector-hog"

bool HOGDescriptor::read(FileNode& obj)
{
    CV_Assert(!obj["winSize"].empty());

    if( !obj.isMap() )
        return false;
    FileNodeIterator it = obj["winSize"].begin();
    it >> winSize.width >> winSize.height; CV_Assert(!winSize.empty());
    it = obj["blockSize"].begin();
    it >> blockSize.width >> blockSize.height; CV_Assert(!blockSize.empty());
    it = obj["blockStride"].begin();
    it >> blockStride.width >> blockStride.height; CV_Assert(!blockStride.empty());
    it = obj["cellSize"].begin();
    it >> cellSize.width >> cellSize.height; CV_Assert(!cellSize.empty());
    obj["nbins"] >> nbins; CV_Assert(nbins > 0);
    obj["derivAperture"] >> derivAperture;
    obj["winSigma"] >> winSigma;
    obj["histogramNormType"] >> histogramNormType;
    obj["L2HysThreshold"] >> L2HysThreshold;
    obj["gammaCorrection"] >> gammaCorrection;
    obj["nlevels"] >> nlevels; CV_Assert(nlevels > 0);
    if (obj["signedGradient"].empty())
        signedGradient = false;
    else
        obj["signedGradient"] >> signedGradient;

    FileNode vecNode = obj["SVMDetector"];
    if( vecNode.isSeq() )
    {
        std::vector<float> _svmDetector;
        vecNode >> _svmDetector;
        setSVMDetector(_svmDetector);
    }
    return true;
}

void HOGDescriptor::write(FileStorage& fs, const String& objName) const
{
    if( !objName.empty() )
        fs << objName;

    fs << "{" CV_TYPE_NAME_HOG_DESCRIPTOR
       << "winSize" << winSize
       << "blockSize" << blockSize
       << "blockStride" << blockStride
       << "cellSize" << cellSize
       << "nbins" << nbins
       << "derivAperture" << derivAperture
       << "winSigma" << getWinSigma()
       << "histogramNormType" << histogramNormType
       << "L2HysThreshold" << L2HysThreshold
       << "gammaCorrection" << gammaCorrection
       << "nlevels" << nlevels
       << "signedGradient" << signedGradient;
    if( !svmDetector.empty() )
        fs << "SVMDetector" << svmDetector;
    fs << "}";
}

bool HOGDescriptor::load(const String& filename, const String& objname)
{
    FileStorage fs(filename, FileStorage::READ);
    FileNode obj = !objname.empty() ? fs[objname] : fs.getFirstTopLevelNode();
    return read(obj);
}

void HOGDescriptor::save(const String& filename, const String& objName) const
{
    FileStorage fs(filename, FileStorage::WRITE);
    write(fs, !objName.empty() ? objName : FileStorage::getDefaultObjectName(filename));
}

void HOGDescriptor::copyTo(HOGDescriptor& c) const
{
    c.winSize = winSize;
    c.blockSize = blockSize;
    c.blockStride = blockStride;
    c.cellSize = cellSize;
    c.nbins = nbins;
    c.derivAperture = derivAperture;
    c.winSigma = winSigma;
    c.histogramNormType = histogramNormType;
    c.L2HysThreshold = L2HysThreshold;
    c.gammaCorrection = gammaCorrection;
    c.setSVMDetector(svmDetector);
    c.nlevels = nlevels;
    c.signedGradient = signedGradient;
}

void HOGDescriptor::computeGradient(InputArray _img, InputOutputArray _grad, InputOutputArray _qangle,
    Size paddingTL, Size paddingBR) const
{
    CV_INSTRUMENT_REGION();

    Mat img = _img.getMat();
    CV_Assert(!img.empty());
    CV_Assert( img.type() == CV_8U || img.type() == CV_8UC3 );

    Size gradsize(img.cols + paddingTL.width + paddingBR.width,
        img.rows + paddingTL.height + paddingBR.height);
    _grad.create(gradsize, CV_32FC2);  // <magnitude*(1-alpha), magnitude*alpha>
    _qangle.create(gradsize, CV_8UC2); // [0..nbins-1] - quantized gradient orientation
    Mat grad = _grad.getMat();
    Mat qangle = _qangle.getMat();

    Size wholeSize;
    Point roiofs;
    img.locateROI(wholeSize, roiofs);

    int i, x, y;
    int cn = img.channels();

    Mat_<float> _lut(1, 256);
    const float* const lut = &_lut(0,0);
#if CV_SIMD128
    v_float32x4 idx(0.0f, 1.0f, 2.0f, 3.0f);
    v_float32x4 ifour = v_setall_f32(4.0);

    float* const _data = &_lut(0, 0);
    if ( gammaCorrection )
        for ( i = 0; i < 256; i += 4)
        {
            v_store(_data + i, v_sqrt(idx));
            idx = v_add(idx, ifour);
        }
    else
        for ( i = 0; i < 256; i += 4)
        {
            v_store(_data + i, idx);
            idx = v_add(idx, ifour);
        }
#else
    if( gammaCorrection )
        for( i = 0; i < 256; i++ )
            _lut(0,i) = std::sqrt((float)i);
    else
        for( i = 0; i < 256; i++ )
            _lut(0,i) = (float)i;
#endif

    AutoBuffer<int> mapbuf(gradsize.width + gradsize.height + 4);
    int* xmap = mapbuf.data() + 1;
    int* ymap = xmap + gradsize.width + 2;

    const int borderType = (int)BORDER_REFLECT_101;

    for( x = -1; x < gradsize.width + 1; x++ )
        xmap[x] = borderInterpolate(x - paddingTL.width + roiofs.x,
        wholeSize.width, borderType) - roiofs.x;
    for( y = -1; y < gradsize.height + 1; y++ )
        ymap[y] = borderInterpolate(y - paddingTL.height + roiofs.y,
        wholeSize.height, borderType) - roiofs.y;

    // x- & y- derivatives for the whole row
    int width = gradsize.width;
    AutoBuffer<float> _dbuf(width*4);
    float* const dbuf = _dbuf.data();
    Mat Dx(1, width, CV_32F, dbuf);
    Mat Dy(1, width, CV_32F, dbuf + width);
    Mat Mag(1, width, CV_32F, dbuf + width*2);
    Mat Angle(1, width, CV_32F, dbuf + width*3);
#if CV_SIMD128
    int widthP2 = width+2;
    AutoBuffer<float> _lutBuf(9*widthP2);
    float* const lutBuf = _lutBuf.data();
#endif

    if (cn == 3)
    {
        int end = gradsize.width + 2;
        xmap -= 1, x = 0;
#if CV_SIMD128
        for ( ; x <= end - 4; x += 4)
        {
            v_int32x4 mul_res = v_load(xmap + x);
            mul_res = v_add(mul_res, v_add(mul_res, mul_res));
            v_store(xmap + x, mul_res);
        }
#endif
        for ( ; x < end; ++x)
            xmap[x] *= 3;
        xmap += 1;
    }

#if CV_SIMD128
    typedef const uchar* const T;
    float *lutPrev, *lutCurr, *lutNext;
    {
        y = 0;
        const uchar* imgPtr  = img.ptr(ymap[y]);
        const uchar* prevPtr = img.data + img.step*ymap[y-1];

        lutPrev = lutBuf+widthP2*0;
        lutCurr = lutBuf+widthP2*3;

        {
            int x0 = xmap[-1], x1 = xmap[0];
            T p02 = imgPtr + x0, p12 = imgPtr + x1;

            lutPrev[0+widthP2*0] = lut[prevPtr[x0+0]];
            lutPrev[0+widthP2*1] = lut[prevPtr[x0+1]];
            lutPrev[0+widthP2*2] = lut[prevPtr[x0+2]];
            lutCurr[0+widthP2*0] = lut[p02[0]]; lutCurr[1+widthP2*0] = lut[p12[0]];
            lutCurr[0+widthP2*1] = lut[p02[1]]; lutCurr[1+widthP2*1] = lut[p12[1]];
            lutCurr[0+widthP2*2] = lut[p02[2]]; lutCurr[1+widthP2*2] = lut[p12[2]];
        }

        for( x = 0; x <= width - 4; x += 4 )
        {
            int x0 = xmap[x], x1 = xmap[x+1], x2 = xmap[x+2], x3 = xmap[x+3];
            T p02 = imgPtr + xmap[x+1];
            T p12 = imgPtr + xmap[x+2];
            T p22 = imgPtr + xmap[x+3];
            T p32 = imgPtr + xmap[x+4];

            v_float32x4 _dx00 = v_float32x4(lut[p02[0]], lut[p12[0]], lut[p22[0]], lut[p32[0]]);
            v_float32x4 _dx10 = v_float32x4(lut[p02[1]], lut[p12[1]], lut[p22[1]], lut[p32[1]]);
            v_float32x4 _dx20 = v_float32x4(lut[p02[2]], lut[p12[2]], lut[p22[2]], lut[p32[2]]);

            v_store(lutCurr+x+widthP2*0+2, _dx00);
            v_store(lutCurr+x+widthP2*1+2, _dx10);
            v_store(lutCurr+x+widthP2*2+2, _dx20);

            v_float32x4 _dy00 = v_float32x4(lut[prevPtr[x0+0]], lut[prevPtr[x1+0]], lut[prevPtr[x2+0]], lut[prevPtr[x3+0]]);
            v_float32x4 _dy10 = v_float32x4(lut[prevPtr[x0+1]], lut[prevPtr[x1+1]], lut[prevPtr[x2+1]], lut[prevPtr[x3+1]]);
            v_float32x4 _dy20 = v_float32x4(lut[prevPtr[x0+2]], lut[prevPtr[x1+2]], lut[prevPtr[x2+2]], lut[prevPtr[x3+2]]);

            v_store(lutPrev+x+widthP2*0+1, _dy00);
            v_store(lutPrev+x+widthP2*1+1, _dy10);
            v_store(lutPrev+x+widthP2*2+1, _dy20);
        }
        {
            int x0 = xmap[x];

            lutPrev[x+widthP2*0+1] = lut[prevPtr[x0+0]];
            lutPrev[x+widthP2*1+1] = lut[prevPtr[x0+1]];
            lutPrev[x+widthP2*2+1] = lut[prevPtr[x0+2]];
        }
    }
#endif

    float angleScale = signedGradient ? (float)(nbins/(2.0*CV_PI)) : (float)(nbins/CV_PI);
    for( y = 0; y < gradsize.height; y++ )
    {
        const uchar* imgPtr  = img.ptr(ymap[y]);
        //In case subimage is used ptr() generates an assert for next and prev rows
        //(see http://code.opencv.org/issues/4149)
        const uchar* prevPtr = img.data + img.step*ymap[y-1];
        const uchar* nextPtr = img.data + img.step*ymap[y+1];

        float* gradPtr = grad.ptr<float>(y);
        uchar* qanglePtr = qangle.ptr(y);

        if( cn == 1 )
        {
            for( x = 0; x < width; x++ )
            {
                int x1 = xmap[x];
                dbuf[x] = (float)(lut[imgPtr[xmap[x+1]]] - lut[imgPtr[xmap[x-1]]]);
                dbuf[width + x] = (float)(lut[nextPtr[x1]] - lut[prevPtr[x1]]);
            }
        }
        else
        {
            x = 0;
#if CV_SIMD128
            int yMod = y%3;

            // Circular lut history buffer
            if (yMod == 0)
            {
                lutPrev = lutBuf+widthP2*0;
                lutCurr = lutBuf+widthP2*3;
                lutNext = lutBuf+widthP2*6;
            }
            else if (yMod == 1)
            {
                lutPrev = lutBuf+widthP2*3;
                lutCurr = lutBuf+widthP2*6;
                lutNext = lutBuf+widthP2*0;
            }
            else
            {
                lutPrev = lutBuf+widthP2*6;
                lutCurr = lutBuf+widthP2*0;
                lutNext = lutBuf+widthP2*3;
            }

            {
                int x0 = xmap[-1];

                lutNext[0+widthP2*0] = lut[nextPtr[x0+0]];
                lutNext[0+widthP2*1] = lut[nextPtr[x0+1]];
                lutNext[0+widthP2*2] = lut[nextPtr[x0+2]];
            }
            for( ; x <= width - 4; x += 4 )
            {
                int x0 = xmap[x], x1 = xmap[x+1], x2 = xmap[x+2], x3 = xmap[x+3];

                v_float32x4 _dx0 = v_sub(v_load(lutCurr + x + widthP2 * 0 + 2), v_load(lutCurr + x + widthP2 * 0));
                v_float32x4 _dx1 = v_sub(v_load(lutCurr + x + widthP2 * 1 + 2), v_load(lutCurr + x + widthP2 * 1));
                v_float32x4 _dx2 = v_sub(v_load(lutCurr + x + widthP2 * 2 + 2), v_load(lutCurr + x + widthP2 * 2));

                v_float32x4 _dy00 = v_float32x4(lut[nextPtr[x0+0]], lut[nextPtr[x1+0]], lut[nextPtr[x2+0]], lut[nextPtr[x3+0]]);
                v_float32x4 _dy0 = v_sub(_dy00, v_load(lutPrev + x + widthP2 * 0 + 1));

                v_store(lutNext+x+widthP2*0+1, _dy00);

                v_float32x4 _dy10 = v_float32x4(lut[nextPtr[x0+1]], lut[nextPtr[x1+1]], lut[nextPtr[x2+1]], lut[nextPtr[x3+1]]);
                v_float32x4 _dy1 = v_sub(_dy10, v_load(lutPrev + x + widthP2 * 1 + 1));

                v_store(lutNext+x+widthP2*1+1, _dy10);

                v_float32x4 _dy20 = v_float32x4(lut[nextPtr[x0+2]], lut[nextPtr[x1+2]], lut[nextPtr[x2+2]], lut[nextPtr[x3+2]]);
                v_float32x4 _dy2 = v_sub(_dy20, v_load(lutPrev + x + widthP2 * 2 + 1));

                v_store(lutNext+x+widthP2*2+1, _dy20);

                v_float32x4 _mag0 = v_add(v_mul(_dx0, _dx0), v_mul(_dy0, _dy0));
                v_float32x4 _mag1 = v_add(v_mul(_dx1, _dx1), v_mul(_dy1, _dy1));
                v_float32x4 _mag2 = v_add(v_mul(_dx2, _dx2), v_mul(_dy2, _dy2));

                v_float32x4 mask = v_reinterpret_as_f32(v_gt(_mag2, _mag1));
                _dx2 = v_select(mask, _dx2, _dx1);
                _dy2 = v_select(mask, _dy2, _dy1);

                mask = v_reinterpret_as_f32(v_gt(v_max(_mag2, _mag1), _mag0));
                _dx2 = v_select(mask, _dx2, _dx0);
                _dy2 = v_select(mask, _dy2, _dy0);

                v_store(dbuf + x, _dx2);
                v_store(dbuf + x + width, _dy2);
            }
            {
                int x0 = xmap[x];

                lutNext[x+widthP2*0+1] = lut[nextPtr[x0+0]];
                lutNext[x+widthP2*1+1] = lut[nextPtr[x0+1]];
                lutNext[x+widthP2*2+1] = lut[nextPtr[x0+2]];
            }
#endif
            for( ; x < width; x++ )
            {
                int x1 = xmap[x];
                float dx0, dy0, dx, dy, mag0, mag;
                const uchar* p2 = imgPtr + xmap[x+1];
                const uchar* p0 = imgPtr + xmap[x-1];

                dx0 = lut[p2[2]] - lut[p0[2]];
                dy0 = lut[nextPtr[x1+2]] - lut[prevPtr[x1+2]];
                mag0 = dx0*dx0 + dy0*dy0;

                dx = lut[p2[1]] - lut[p0[1]];
                dy = lut[nextPtr[x1+1]] - lut[prevPtr[x1+1]];
                mag = dx*dx + dy*dy;
                if( mag0 < mag )
                {
                    dx0 = dx;
                    dy0 = dy;
                    mag0 = mag;
                }

                dx = lut[p2[0]] - lut[p0[0]];
                dy = lut[nextPtr[x1]] - lut[prevPtr[x1]];
                mag = dx*dx + dy*dy;
                if( mag0 < mag )
                {
                    dx0 = dx;
                    dy0 = dy;
                    mag0 = mag;
                }

                dbuf[x] = dx0;
                dbuf[x+width] = dy0;
            }
        }

        // computing angles and magnidutes
        cartToPolar( Dx, Dy, Mag, Angle, false );

        // filling the result matrix
        x = 0;
#if CV_SIMD128
        v_float32x4 fhalf = v_setall_f32(0.5f);
        v_float32x4 _angleScale = v_setall_f32(angleScale), fone = v_setall_f32(1.0f);
        v_int32x4 ione = v_setall_s32(1), _nbins = v_setall_s32(nbins), izero = v_setzero_s32();

        for ( ; x <= width - 4; x += 4)
        {
            int x2 = x << 1;
            v_float32x4 _mag = v_load(dbuf + x + (width << 1));
            v_float32x4 _angle = v_load(dbuf + x + width * 3);
            _angle = v_sub(v_mul(_angleScale, _angle), fhalf);

            v_int32x4 _hidx = v_floor(_angle);
            _angle = v_sub(_angle, v_cvt_f32(_hidx));

            v_float32x4 ft0 = v_mul(_mag, v_sub(fone, _angle));
            v_float32x4 ft1 = v_mul(_mag, _angle);

            v_store_interleave(gradPtr + x2, ft0, ft1);

            v_int32x4 mask0 = v_shr<31>(_hidx);
            v_int32x4 it0 = v_and(mask0, _nbins);
            mask0 = (v_ge(_hidx, _nbins));
            v_int32x4 it1 = v_and(mask0, _nbins);
            _hidx = v_add(_hidx, v_sub(it0, it1));

            it0 = v_reinterpret_as_s32(v_pack(v_pack(_hidx, izero), v_reinterpret_as_s16(izero)));
            _hidx = v_add(_hidx, ione);
            _hidx = v_and(_hidx, v_lt(_hidx, _nbins));
            it1 = v_reinterpret_as_s32(v_pack(v_pack(_hidx, izero), v_reinterpret_as_s16(izero)));
            v_uint8x16 it2, it3;
            v_zip(v_reinterpret_as_u8(it0), v_reinterpret_as_u8(it1), it2, it3);

            v_store_low(qanglePtr + x2, it2);
        }
#endif
        for( ; x < width; x++ )
        {
            float mag = dbuf[x+width*2], angle = dbuf[x+width*3]*angleScale - 0.5f;
            int hidx = cvFloor(angle);
            angle -= hidx;
            gradPtr[x*2] = mag*(1.f - angle);
            gradPtr[x*2+1] = mag*angle;

            if( hidx < 0 )
                hidx += nbins;
            else if( hidx >= nbins )
                hidx -= nbins;

            CV_Assert( (unsigned)hidx < (unsigned)nbins );

            qanglePtr[x*2] = (uchar)hidx;
            hidx++;
            hidx &= hidx < nbins ? -1 : 0;
            qanglePtr[x*2+1] = (uchar)hidx;
        }
    }
}

struct HOGCache
{
    struct BlockData
    {
        BlockData() :
            histOfs(0), imgOffset()
        { }

        int histOfs;
        Point imgOffset;
    };

    struct PixData
    {
        size_t gradOfs, qangleOfs;
        int histOfs[4];
        float histWeights[4];
        float gradWeight;
    };

    HOGCache();
    HOGCache(const HOGDescriptor* descriptor,
        const Mat& img, const Size& paddingTL, const Size& paddingBR,
        bool useCache, const Size& cacheStride);
    virtual ~HOGCache() { }
    virtual void init(const HOGDescriptor* descriptor,
        const Mat& img, const Size& paddingTL, const Size& paddingBR,
        bool useCache, const Size& cacheStride);

    Size windowsInImage(const Size& imageSize, const Size& winStride) const;
    Rect getWindow(const Size& imageSize, const Size& winStride, int idx) const;

    const float* getBlock(Point pt, float* buf);
    virtual void normalizeBlockHistogram(float* histogram) const;

    std::vector<PixData> pixData;
    std::vector<BlockData> blockData;

    bool useCache;
    std::vector<int> ymaxCached;
    Size winSize;
    Size cacheStride;
    Size nblocks, ncells;
    int blockHistogramSize;
    int count1, count2, count4;
    Point imgoffset;
    Mat_<float> blockCache;
    Mat_<uchar> blockCacheFlags;

    Mat grad, qangle;
    const HOGDescriptor* descriptor;
};

HOGCache::HOGCache() :
    blockHistogramSize(), count1(), count2(), count4()
{
    useCache = false;
    descriptor = 0;
}

HOGCache::HOGCache(const HOGDescriptor* _descriptor,
    const Mat& _img, const Size& _paddingTL, const Size& _paddingBR,
    bool _useCache, const Size& _cacheStride)
{
    init(_descriptor, _img, _paddingTL, _paddingBR, _useCache, _cacheStride);
}

void HOGCache::init(const HOGDescriptor* _descriptor,
    const Mat& _img, const Size& _paddingTL, const Size& _paddingBR,
    bool _useCache, const Size& _cacheStride)
{
    descriptor = _descriptor;
    cacheStride = _cacheStride;
    useCache = _useCache;

    descriptor->computeGradient(_img, grad, qangle, _paddingTL, _paddingBR);
    imgoffset = _paddingTL;

    winSize = descriptor->winSize;
    Size blockSize = descriptor->blockSize;
    Size blockStride = descriptor->blockStride;
    Size cellSize = descriptor->cellSize;
    int i, j, nbins = descriptor->nbins;
    int rawBlockSize = blockSize.width*blockSize.height;

    nblocks = Size((winSize.width - blockSize.width)/blockStride.width + 1,
        (winSize.height - blockSize.height)/blockStride.height + 1);
    ncells = Size(blockSize.width/cellSize.width, blockSize.height/cellSize.height);
    blockHistogramSize = ncells.width*ncells.height*nbins;

    if( useCache )
    {
        Size cacheSize((grad.cols - blockSize.width)/cacheStride.width+1,
            (winSize.height/cacheStride.height)+1);

        blockCache.create(cacheSize.height, cacheSize.width*blockHistogramSize);
        blockCacheFlags.create(cacheSize);

        size_t cacheRows = blockCache.rows;
        ymaxCached.resize(cacheRows);
        for(size_t ii = 0; ii < cacheRows; ii++ )
            ymaxCached[ii] = -1;
    }

    Mat_<float> weights(blockSize);
    float sigma = (float)descriptor->getWinSigma();
    float scale = 1.f/(sigma*sigma*2);

    {
        AutoBuffer<float> di(blockSize.height), dj(blockSize.width);
        float* _di = di.data(), *_dj = dj.data();
        float bh = blockSize.height * 0.5f, bw = blockSize.width * 0.5f;

        i = 0;
    #if CV_SIMD128
        v_float32x4 idx(0.0f, 1.0f, 2.0f, 3.0f);
        v_float32x4 _bw = v_setall_f32(bw), _bh = v_setall_f32(bh);
        v_float32x4 ifour = v_setall_f32(4.0);

        for (; i <= blockSize.height - 4; i += 4)
        {
            v_float32x4 t = v_sub(idx, _bh);
            t = v_mul(t, t);
            idx = v_add(idx, ifour);
            v_store(_di + i, t);
        }
    #endif
        for ( ; i < blockSize.height; ++i)
        {
            _di[i] = i - bh;
            _di[i] *= _di[i];
        }

        j = 0;
    #if CV_SIMD128
        idx = v_float32x4(0.0f, 1.0f, 2.0f, 3.0f);

        for (; j <= blockSize.height - 4; j += 4)
        {
            v_float32x4 t = v_sub(idx, _bw);
            t = v_mul(t, t);
            idx = v_add(idx, ifour);
            v_store(_dj + j, t);
        }
    #endif
        for ( ; j < blockSize.width; ++j)
        {
            _dj[j] = j - bw;
            _dj[j] *= _dj[j];
        }

        for(i = 0; i < blockSize.height; i++)
            for(j = 0; j < blockSize.width; j++)
                weights(i,j) = std::exp(-(_di[i] + _dj[j])*scale);
    }

    blockData.resize(nblocks.width*nblocks.height);
    pixData.resize(rawBlockSize*3);

    // Initialize 2 lookup tables, pixData & blockData.
    // Here is why:
    //
    // The detection algorithm runs in 4 nested loops (at each pyramid layer):
    //  loop over the windows within the input image
    //    loop over the blocks within each window
    //      loop over the cells within each block
    //        loop over the pixels in each cell
    //
    // As each of the loops runs over a 2-dimensional array,
    // we could get 8(!) nested loops in total, which is very-very slow.
    //
    // To speed the things up, we do the following:
    //   1. loop over windows is unrolled in the HOGDescriptor::{compute|detect} methods;
    //         inside we compute the current search window using getWindow() method.
    //         Yes, it involves some overhead (function call + couple of divisions),
    //         but it's tiny in fact.
    //   2. loop over the blocks is also unrolled. Inside we use pre-computed blockData[j]
    //         to set up gradient and histogram pointers.
    //   3. loops over cells and pixels in each cell are merged
    //       (since there is no overlap between cells, each pixel in the block is processed once)
    //      and also unrolled. Inside we use PixData[k] to access the gradient values and
    //      update the histogram
    //

    count1 = count2 = count4 = 0;
    for( j = 0; j < blockSize.width; j++ )
        for( i = 0; i < blockSize.height; i++ )
        {
            PixData* data = 0;
            float cellX = (j+0.5f)/cellSize.width - 0.5f;
            float cellY = (i+0.5f)/cellSize.height - 0.5f;
            int icellX0 = cvFloor(cellX);
            int icellY0 = cvFloor(cellY);
            int icellX1 = icellX0 + 1, icellY1 = icellY0 + 1;
            cellX -= icellX0;
            cellY -= icellY0;

            if( (unsigned)icellX0 < (unsigned)ncells.width &&
               (unsigned)icellX1 < (unsigned)ncells.width )
            {
                if( (unsigned)icellY0 < (unsigned)ncells.height &&
                   (unsigned)icellY1 < (unsigned)ncells.height )
                {
                    data = &pixData[rawBlockSize*2 + (count4++)];
                    data->histOfs[0] = (icellX0*ncells.height + icellY0)*nbins;
                    data->histWeights[0] = (1.f - cellX)*(1.f - cellY);
                    data->histOfs[1] = (icellX1*ncells.height + icellY0)*nbins;
                    data->histWeights[1] = cellX*(1.f - cellY);
                    data->histOfs[2] = (icellX0*ncells.height + icellY1)*nbins;
                    data->histWeights[2] = (1.f - cellX)*cellY;
                    data->histOfs[3] = (icellX1*ncells.height + icellY1)*nbins;
                    data->histWeights[3] = cellX*cellY;
                }
                else
                {
                    data = &pixData[rawBlockSize + (count2++)];
                    if( (unsigned)icellY0 < (unsigned)ncells.height )
                    {
                        icellY1 = icellY0;
                        cellY = 1.f - cellY;
                    }
                    data->histOfs[0] = (icellX0*ncells.height + icellY1)*nbins;
                    data->histWeights[0] = (1.f - cellX)*cellY;
                    data->histOfs[1] = (icellX1*ncells.height + icellY1)*nbins;
                    data->histWeights[1] = cellX*cellY;
                    data->histOfs[2] = data->histOfs[3] = 0;
                    data->histWeights[2] = data->histWeights[3] = 0;
                }
            }
            else
            {
                if( (unsigned)icellX0 < (unsigned)ncells.width )
                {
                    icellX1 = icellX0;
                    cellX = 1.f - cellX;
                }

                if( (unsigned)icellY0 < (unsigned)ncells.height &&
                   (unsigned)icellY1 < (unsigned)ncells.height )
                {
                    data = &pixData[rawBlockSize + (count2++)];
                    data->histOfs[0] = (icellX1*ncells.height + icellY0)*nbins;
                    data->histWeights[0] = cellX*(1.f - cellY);
                    data->histOfs[1] = (icellX1*ncells.height + icellY1)*nbins;
                    data->histWeights[1] = cellX*cellY;
                    data->histOfs[2] = data->histOfs[3] = 0;
                    data->histWeights[2] = data->histWeights[3] = 0;
                }
                else
                {
                    data = &pixData[count1++];
                    if( (unsigned)icellY0 < (unsigned)ncells.height )
                    {
                        icellY1 = icellY0;
                        cellY = 1.f - cellY;
                    }
                    data->histOfs[0] = (icellX1*ncells.height + icellY1)*nbins;
                    data->histWeights[0] = cellX*cellY;
                    data->histOfs[1] = data->histOfs[2] = data->histOfs[3] = 0;
                    data->histWeights[1] = data->histWeights[2] = data->histWeights[3] = 0;
                }
            }
            data->gradOfs = (grad.cols*i + j)*2;
            data->qangleOfs = (qangle.cols*i + j)*2;
            data->gradWeight = weights(i,j);
        }

    CV_Assert( count1 + count2 + count4 == rawBlockSize );
    // defragment pixData
    for( j = 0; j < count2; j++ )
        pixData[j + count1] = pixData[j + rawBlockSize];
    for( j = 0; j < count4; j++ )
        pixData[j + count1 + count2] = pixData[j + rawBlockSize*2];
    count2 += count1;
    count4 += count2;

    // initialize blockData
    for( j = 0; j < nblocks.width; j++ )
        for( i = 0; i < nblocks.height; i++ )
        {
            BlockData& data = blockData[j*nblocks.height + i];
            data.histOfs = (j*nblocks.height + i)*blockHistogramSize;
            data.imgOffset = Point(j*blockStride.width,i*blockStride.height);
        }
}

const float* HOGCache::getBlock(Point pt, float* buf)
{
    float* blockHist = buf;
    CV_Assert(descriptor != 0);

//    Size blockSize = descriptor->blockSize;
    pt += imgoffset;

//    CV_Assert( (unsigned)pt.x <= (unsigned)(grad.cols - blockSize.width) &&
//        (unsigned)pt.y <= (unsigned)(grad.rows - blockSize.height) );

    if( useCache )
    {
        CV_Assert( pt.x % cacheStride.width == 0 &&
                   pt.y % cacheStride.height == 0 );
        Point cacheIdx(pt.x/cacheStride.width,
                       (pt.y/cacheStride.height) % blockCache.rows);
        if( pt.y != ymaxCached[cacheIdx.y] )
        {
            Mat_<uchar> cacheRow = blockCacheFlags.row(cacheIdx.y);
            cacheRow = (uchar)0;
            ymaxCached[cacheIdx.y] = pt.y;
        }

        blockHist = &blockCache[cacheIdx.y][cacheIdx.x*blockHistogramSize];
        uchar& computedFlag = blockCacheFlags(cacheIdx.y, cacheIdx.x);
        if( computedFlag != 0 )
            return blockHist;
        computedFlag = (uchar)1; // set it at once, before actual computing
    }

    int k, C1 = count1, C2 = count2, C4 = count4;
    const float* gradPtr = grad.ptr<float>(pt.y) + pt.x*2;
    const uchar* qanglePtr = qangle.ptr(pt.y) + pt.x*2;

//    CV_Assert( blockHist != 0 );
    memset(blockHist, 0, sizeof(float) * blockHistogramSize);

    const PixData* _pixData = &pixData[0];

    for( k = 0; k < C1; k++ )
    {
        const PixData& pk = _pixData[k];
        const float* const a = gradPtr + pk.gradOfs;
        float w = pk.gradWeight*pk.histWeights[0];
        const uchar* h = qanglePtr + pk.qangleOfs;
        int h0 = h[0], h1 = h[1];

        float* hist = blockHist + pk.histOfs[0];
        float t0 = hist[h0] + a[0]*w;
        float t1 = hist[h1] + a[1]*w;
        hist[h0] = t0; hist[h1] = t1;
    }

#if CV_SIMD128
    float hist0[4], hist1[4];
    for( ; k < C2; k++ )
    {
        const PixData& pk = _pixData[k];
        const float* const a = gradPtr + pk.gradOfs;
        const uchar* const h = qanglePtr + pk.qangleOfs;
        int h0 = h[0], h1 = h[1];

        v_float32x4 _a0 = v_setall_f32(a[0]), _a1 = v_setall_f32(a[1]);
        v_float32x4 w = v_mul(v_setall_f32(pk.gradWeight), v_load(pk.histWeights));
        v_float32x4 _t0 = v_mul(_a0, w), _t1 = v_mul(_a1, w);

        v_store(hist0, _t0);
        v_store(hist1, _t1);

        float* hist = blockHist + pk.histOfs[0];
        float t0 = hist[h0] + hist0[0];
        float t1 = hist[h1] + hist1[0];
        hist[h0] = t0; hist[h1] = t1;

        hist = blockHist + pk.histOfs[1];
        t0 = hist[h0] + hist0[1];
        t1 = hist[h1] + hist1[1];
        hist[h0] = t0; hist[h1] = t1;
    }
#else
    for( ; k < C2; k++ )
    {
        const PixData& pk = _pixData[k];
        const float* const a = gradPtr + pk.gradOfs;
        float w, t0, t1, a0 = a[0], a1 = a[1];
        const uchar* const h = qanglePtr + pk.qangleOfs;
        int h0 = h[0], h1 = h[1];

        float* hist = blockHist + pk.histOfs[0];
        w = pk.gradWeight*pk.histWeights[0];
        t0 = hist[h0] + a0*w;
        t1 = hist[h1] + a1*w;
        hist[h0] = t0; hist[h1] = t1;

        hist = blockHist + pk.histOfs[1];
        w = pk.gradWeight*pk.histWeights[1];
        t0 = hist[h0] + a0*w;
        t1 = hist[h1] + a1*w;
        hist[h0] = t0; hist[h1] = t1;
    }
#endif

#if CV_SIMD128
    for( ; k < C4; k++ )
    {
        const PixData& pk = _pixData[k];
        const float* const a = gradPtr + pk.gradOfs;
        const uchar* const h = qanglePtr + pk.qangleOfs;
        int h0 = h[0], h1 = h[1];

        v_float32x4 _a0 = v_setall_f32(a[0]), _a1 = v_setall_f32(a[1]);
        v_float32x4 w = v_mul(v_setall_f32(pk.gradWeight), v_load(pk.histWeights));
        v_float32x4 _t0 = v_mul(_a0, w), _t1 = v_mul(_a1, w);

        v_store(hist0, _t0);
        v_store(hist1, _t1);

        float* hist = blockHist + pk.histOfs[0];
        float t0 = hist[h0] + hist0[0];
        float t1 = hist[h1] + hist1[0];
        hist[h0] = t0; hist[h1] = t1;

        hist = blockHist + pk.histOfs[1];
        t0 = hist[h0] + hist0[1];
        t1 = hist[h1] + hist1[1];
        hist[h0] = t0; hist[h1] = t1;

        hist = blockHist + pk.histOfs[2];
        t0 = hist[h0] + hist0[2];
        t1 = hist[h1] + hist1[2];
        hist[h0] = t0; hist[h1] = t1;

        hist = blockHist + pk.histOfs[3];
        t0 = hist[h0] + hist0[3];
        t1 = hist[h1] + hist1[3];
        hist[h0] = t0; hist[h1] = t1;
    }
#else
    for( ; k < C4; k++ )
    {
        const PixData& pk = _pixData[k];
        const float* a = gradPtr + pk.gradOfs;
        float w, t0, t1, a0 = a[0], a1 = a[1];
        const uchar* h = qanglePtr + pk.qangleOfs;
        int h0 = h[0], h1 = h[1];

        float* hist = blockHist + pk.histOfs[0];
        w = pk.gradWeight*pk.histWeights[0];
        t0 = hist[h0] + a0*w;
        t1 = hist[h1] + a1*w;
        hist[h0] = t0; hist[h1] = t1;

        hist = blockHist + pk.histOfs[1];
        w = pk.gradWeight*pk.histWeights[1];
        t0 = hist[h0] + a0*w;
        t1 = hist[h1] + a1*w;
        hist[h0] = t0; hist[h1] = t1;

        hist = blockHist + pk.histOfs[2];
        w = pk.gradWeight*pk.histWeights[2];
        t0 = hist[h0] + a0*w;
        t1 = hist[h1] + a1*w;
        hist[h0] = t0; hist[h1] = t1;

        hist = blockHist + pk.histOfs[3];
        w = pk.gradWeight*pk.histWeights[3];
        t0 = hist[h0] + a0*w;
        t1 = hist[h1] + a1*w;
        hist[h0] = t0; hist[h1] = t1;
    }
#endif

    normalizeBlockHistogram(blockHist);

    return blockHist;
}

void HOGCache::normalizeBlockHistogram(float* _hist) const
{
    float* hist = &_hist[0], sum = 0.0f, partSum[4];
    size_t i = 0, sz = blockHistogramSize;

#if CV_SIMD128
    v_float32x4 p0 = v_load(hist);
    v_float32x4 s = v_mul(p0, p0);

    for (i = 4; i <= sz - 4; i += 4)
    {
        p0 = v_load(hist + i);
        s = v_add(s, v_mul(p0, p0));
    }
    v_store(partSum, s);
#else
    partSum[0] = 0.0f;
    partSum[1] = 0.0f;
    partSum[2] = 0.0f;
    partSum[3] = 0.0f;
    for ( ; i <= sz - 4; i += 4)
    {
        partSum[0] += hist[i] * hist[i];
        partSum[1] += hist[i+1] * hist[i+1];
        partSum[2] += hist[i+2] * hist[i+2];
        partSum[3] += hist[i+3] * hist[i+3];
    }
#endif
    float t0 = partSum[0] + partSum[1];
    float t1 = partSum[2] + partSum[3];
    sum = t0 + t1;
    for ( ; i < sz; ++i)
        sum += hist[i]*hist[i];

    float scale = 1.f/(std::sqrt(sum)+sz*0.1f), thresh = (float)descriptor->L2HysThreshold;
    i = 0, sum = 0.0f;

#if CV_SIMD128
    v_float32x4 _scale = v_setall_f32(scale);
    static v_float32x4 _threshold = v_setall_f32(thresh);

    v_float32x4 p = v_mul(_scale, v_load(hist));
    p = v_min(p, _threshold);
    s = v_mul(p, p);
    v_store(hist, p);

    for(i = 4 ; i <= sz - 4; i += 4)
    {
        p = v_load(hist + i);
        p = v_mul(p, _scale);
        p = v_min(p, _threshold);
        s = v_add(s, v_mul(p, p));
        v_store(hist + i, p);
    }

    v_store(partSum, s);
#else
    partSum[0] = 0.0f;
    partSum[1] = 0.0f;
    partSum[2] = 0.0f;
    partSum[3] = 0.0f;
    for( ; i <= sz - 4; i += 4)
    {
        hist[i] = std::min(hist[i]*scale, thresh);
        hist[i+1] = std::min(hist[i+1]*scale, thresh);
        hist[i+2] = std::min(hist[i+2]*scale, thresh);
        hist[i+3] = std::min(hist[i+3]*scale, thresh);
        partSum[0] += hist[i]*hist[i];
        partSum[1] += hist[i+1]*hist[i+1];
        partSum[2] += hist[i+2]*hist[i+2];
        partSum[3] += hist[i+3]*hist[i+3];
    }
#endif
    t0 = partSum[0] + partSum[1];
    t1 = partSum[2] + partSum[3];
    sum = t0 + t1;
    for( ; i < sz; ++i)
    {
        hist[i] = std::min(hist[i]*scale, thresh);
        sum += hist[i]*hist[i];
    }

    scale = 1.f/(std::sqrt(sum)+1e-3f), i = 0;
#if CV_SIMD128
    v_float32x4 _scale2 = v_setall_f32(scale);
    for ( ; i <= sz - 4; i += 4)
    {
        v_float32x4 t = v_mul(_scale2, v_load(hist + i));
        v_store(hist + i, t);
    }
#endif
    for ( ; i < sz; ++i)
        hist[i] *= scale;
}

Size HOGCache::windowsInImage(const Size& imageSize, const Size& winStride) const
{
    return Size((imageSize.width - winSize.width)/winStride.width + 1,
        (imageSize.height - winSize.height)/winStride.height + 1);
}

Rect HOGCache::getWindow(const Size& imageSize, const Size& winStride, int idx) const
{
    int nwindowsX = (imageSize.width - winSize.width)/winStride.width + 1;
    int y = idx / nwindowsX;
    int x = idx - nwindowsX*y;
    return Rect( x*winStride.width, y*winStride.height, winSize.width, winSize.height );
}

static inline int gcd(int a, int b)
{
    if( a < b )
        std::swap(a, b);
    while( b > 0 )
    {
        int r = a % b;
        a = b;
        b = r;
    }
    return a;
}

#ifdef HAVE_OPENCL

static bool ocl_compute_gradients_8UC1(int height, int width, InputArray _img, float angle_scale,
                                       UMat grad, UMat qangle, bool correct_gamma, int nbins)
{
    ocl::Kernel k("compute_gradients_8UC1_kernel", ocl::objdetect::objdetect_hog_oclsrc);
    if(k.empty())
        return false;

    UMat img = _img.getUMat();

    size_t localThreads[3] = { NTHREADS, 1, 1 };
    size_t globalThreads[3] = { (size_t)width, (size_t)height, 1 };
    char correctGamma = (correct_gamma) ? 1 : 0;
    int grad_quadstep = (int)grad.step >> 3;
    int qangle_elem_size = CV_ELEM_SIZE1(qangle.type());
    int qangle_step = (int)qangle.step / (2 * qangle_elem_size);

    int idx = 0;
    idx = k.set(idx, height);
    idx = k.set(idx, width);
    idx = k.set(idx, (int)img.step1());
    idx = k.set(idx, grad_quadstep);
    idx = k.set(idx, qangle_step);
    idx = k.set(idx, ocl::KernelArg::PtrReadOnly(img));
    idx = k.set(idx, ocl::KernelArg::PtrWriteOnly(grad));
    idx = k.set(idx, ocl::KernelArg::PtrWriteOnly(qangle));
    idx = k.set(idx, angle_scale);
    idx = k.set(idx, correctGamma);
    idx = k.set(idx, nbins);

    return k.run(2, globalThreads, localThreads, false);
}

static bool ocl_computeGradient(InputArray img, UMat grad, UMat qangle, int nbins, Size effect_size, bool gamma_correction, bool signedGradient)
{
    float angleScale = signedGradient ? (float)(nbins/(2.0*CV_PI)) : (float)(nbins/CV_PI);

    return ocl_compute_gradients_8UC1(effect_size.height, effect_size.width, img,
         angleScale, grad, qangle, gamma_correction, nbins);
}

#define CELL_WIDTH 8
#define CELL_HEIGHT 8
#define CELLS_PER_BLOCK_X 2
#define CELLS_PER_BLOCK_Y 2

static bool ocl_compute_hists(int nbins, int block_stride_x, int block_stride_y, int height, int width,
                              UMat grad, UMat qangle, UMat gauss_w_lut, UMat block_hists, size_t block_hist_size)
{
    ocl::Kernel k("compute_hists_lut_kernel", ocl::objdetect::objdetect_hog_oclsrc);
    if(k.empty())
        return false;

    int img_block_width = (width - CELLS_PER_BLOCK_X * CELL_WIDTH + block_stride_x)/block_stride_x;
    int img_block_height = (height - CELLS_PER_BLOCK_Y * CELL_HEIGHT + block_stride_y)/block_stride_y;
    int blocks_total = img_block_width * img_block_height;

    int qangle_elem_size = CV_ELEM_SIZE1(qangle.type());
    int grad_quadstep = (int)grad.step >> 2;
    int qangle_step = (int)qangle.step / qangle_elem_size;

    int blocks_in_group = 4;
    size_t localThreads[3] = { (size_t)blocks_in_group * 24, 2, 1 };
    size_t globalThreads[3] = {((img_block_width * img_block_height + blocks_in_group - 1)/blocks_in_group) * localThreads[0], 2, 1 };

    int hists_size = (nbins * CELLS_PER_BLOCK_X * CELLS_PER_BLOCK_Y * 12) * sizeof(float);
    int final_hists_size = (nbins * CELLS_PER_BLOCK_X * CELLS_PER_BLOCK_Y) * sizeof(float);

    int smem = (hists_size + final_hists_size) * blocks_in_group;

    int idx = 0;
    idx = k.set(idx, block_stride_x);
    idx = k.set(idx, block_stride_y);
    idx = k.set(idx, nbins);
    idx = k.set(idx, (int)block_hist_size);
    idx = k.set(idx, img_block_width);
    idx = k.set(idx, blocks_in_group);
    idx = k.set(idx, blocks_total);
    idx = k.set(idx, grad_quadstep);
    idx = k.set(idx, qangle_step);
    idx = k.set(idx, ocl::KernelArg::PtrReadOnly(grad));
    idx = k.set(idx, ocl::KernelArg::PtrReadOnly(qangle));
    idx = k.set(idx, ocl::KernelArg::PtrReadOnly(gauss_w_lut));
    idx = k.set(idx, ocl::KernelArg::PtrWriteOnly(block_hists));
    idx = k.set(idx, (void*)NULL, (size_t)smem);

    return k.run(2, globalThreads, localThreads, false);
}

static int power_2up(unsigned int n)
{
    for(unsigned int i = 1; i<=1024; i<<=1)
        if(n < i)
            return i;
    return -1; // Input is too big
}

static bool ocl_normalize_hists(int nbins, int block_stride_x, int block_stride_y,
                                int height, int width, UMat block_hists, float threshold)
{
    int block_hist_size = nbins * CELLS_PER_BLOCK_X * CELLS_PER_BLOCK_Y;
    int img_block_width = (width - CELLS_PER_BLOCK_X * CELL_WIDTH + block_stride_x)
        / block_stride_x;
    int img_block_height = (height - CELLS_PER_BLOCK_Y * CELL_HEIGHT + block_stride_y)
        / block_stride_y;
    int nthreads;
    size_t globalThreads[3] = { 1, 1, 1  };
    size_t localThreads[3] = { 1, 1, 1  };

    int idx = 0;
    ocl::Kernel k;
    if ( nbins == 9 )
    {
        k.create("normalize_hists_36_kernel", ocl::objdetect::objdetect_hog_oclsrc, "");
        if(k.empty())
            return false;

        int blocks_in_group = NTHREADS / block_hist_size;
        nthreads = blocks_in_group * block_hist_size;
        int num_groups = (img_block_width * img_block_height + blocks_in_group - 1)/blocks_in_group;
        globalThreads[0] = nthreads * num_groups;
        localThreads[0] = nthreads;
    }
    else
    {
        k.create("normalize_hists_kernel", ocl::objdetect::objdetect_hog_oclsrc, "");
        if(k.empty())
            return false;

        nthreads = power_2up(block_hist_size);
        globalThreads[0] = img_block_width * nthreads;
        globalThreads[1] = img_block_height;
        localThreads[0] = nthreads;

        if ((nthreads < 32) || (nthreads > 512) )
            return false;

        idx = k.set(idx, nthreads);
        idx = k.set(idx, block_hist_size);
        idx = k.set(idx, img_block_width);
    }
    idx = k.set(idx, ocl::KernelArg::PtrReadWrite(block_hists));
    idx = k.set(idx, threshold);
    idx = k.set(idx, (void*)NULL,  nthreads * sizeof(float));

    return k.run(2, globalThreads, localThreads, false);
}

static bool ocl_extract_descrs_by_rows(int win_height, int win_width, int block_stride_y, int block_stride_x, int win_stride_y, int win_stride_x,
                                       int height, int width, UMat block_hists, UMat descriptors,
                                       int block_hist_size, int descr_size, int descr_width)
{
    ocl::Kernel k("extract_descrs_by_rows_kernel", ocl::objdetect::objdetect_hog_oclsrc);
    if(k.empty())
        return false;

    int win_block_stride_x = win_stride_x / block_stride_x;
    int win_block_stride_y = win_stride_y / block_stride_y;
    int img_win_width = (width - win_width + win_stride_x) / win_stride_x;
    int img_win_height = (height - win_height + win_stride_y) / win_stride_y;
    int img_block_width = (width - CELLS_PER_BLOCK_X * CELL_WIDTH + block_stride_x) /
        block_stride_x;

    int descriptors_quadstep = (int)descriptors.step >> 2;

    size_t globalThreads[3] = { (size_t)img_win_width * NTHREADS, (size_t)img_win_height, 1 };
    size_t localThreads[3] = { NTHREADS, 1, 1 };

    int idx = 0;
    idx = k.set(idx, block_hist_size);
    idx = k.set(idx, descriptors_quadstep);
    idx = k.set(idx, descr_size);
    idx = k.set(idx, descr_width);
    idx = k.set(idx, img_block_width);
    idx = k.set(idx, win_block_stride_x);
    idx = k.set(idx, win_block_stride_y);
    idx = k.set(idx, ocl::KernelArg::PtrReadOnly(block_hists));
    idx = k.set(idx, ocl::KernelArg::PtrWriteOnly(descriptors));

    return k.run(2, globalThreads, localThreads, false);
}

static bool ocl_extract_descrs_by_cols(int win_height, int win_width, int block_stride_y, int block_stride_x, int win_stride_y, int win_stride_x,
                                       int height, int width, UMat block_hists, UMat descriptors,
                                       int block_hist_size, int descr_size, int nblocks_win_x, int nblocks_win_y)
{
    ocl::Kernel k("extract_descrs_by_cols_kernel", ocl::objdetect::objdetect_hog_oclsrc);
    if(k.empty())
        return false;

    int win_block_stride_x = win_stride_x / block_stride_x;
    int win_block_stride_y = win_stride_y / block_stride_y;
    int img_win_width = (width - win_width + win_stride_x) / win_stride_x;
    int img_win_height = (height - win_height + win_stride_y) / win_stride_y;
    int img_block_width = (width - CELLS_PER_BLOCK_X * CELL_WIDTH + block_stride_x) /
        block_stride_x;

    int descriptors_quadstep = (int)descriptors.step >> 2;

    size_t globalThreads[3] = { (size_t)img_win_width * NTHREADS, (size_t)img_win_height, 1 };
    size_t localThreads[3] = { NTHREADS, 1, 1 };

    int idx = 0;
    idx = k.set(idx, block_hist_size);
    idx = k.set(idx, descriptors_quadstep);
    idx = k.set(idx, descr_size);
    idx = k.set(idx, nblocks_win_x);
    idx = k.set(idx, nblocks_win_y);
    idx = k.set(idx, img_block_width);
    idx = k.set(idx, win_block_stride_x);
    idx = k.set(idx, win_block_stride_y);
    idx = k.set(idx, ocl::KernelArg::PtrReadOnly(block_hists));
    idx = k.set(idx, ocl::KernelArg::PtrWriteOnly(descriptors));

    return k.run(2, globalThreads, localThreads, false);
}

static bool ocl_compute(InputArray _img, Size win_stride, std::vector<float>& _descriptors, HOGDescriptor::DescriptorStorageFormat descr_format, Size blockSize,
                        Size cellSize, int nbins, Size blockStride, Size winSize, float sigma, bool gammaCorrection, double L2HysThreshold, bool signedGradient)
{
    Size imgSize = _img.size();
    Size effect_size = imgSize;

    UMat grad(imgSize, CV_32FC2);
    int qangle_type = ocl::Device::getDefault().isIntel() ? CV_32SC2 : CV_8UC2;
    UMat qangle(imgSize, qangle_type);

    const size_t block_hist_size = getBlockHistogramSize(blockSize, cellSize, nbins);
    const Size blocks_per_img = numPartsWithin(imgSize, blockSize, blockStride);
    UMat block_hists(1, static_cast<int>(block_hist_size * blocks_per_img.area()) + 256, CV_32F);

    Size wins_per_img = numPartsWithin(imgSize, winSize, win_stride);
    UMat labels(1, wins_per_img.area(), CV_8U);

    float scale = 1.f / (2.f * sigma * sigma);
    Mat gaussian_lut(1, 512, CV_32FC1);
    int idx = 0;
    for(int i=-8; i<8; i++)
        for(int j=-8; j<8; j++)
            gaussian_lut.at<float>(idx++) = std::exp(-(j * j + i * i) * scale);
    for(int i=-8; i<8; i++)
        for(int j=-8; j<8; j++)
            gaussian_lut.at<float>(idx++) = (8.f - fabs(j + 0.5f)) * (8.f - fabs(i + 0.5f)) / 64.f;

    if(!ocl_computeGradient(_img, grad, qangle, nbins, effect_size, gammaCorrection, signedGradient))
        return false;

    UMat gauss_w_lut;
    gaussian_lut.copyTo(gauss_w_lut);
    if(!ocl_compute_hists(nbins, blockStride.width, blockStride.height, effect_size.height,
        effect_size.width, grad, qangle, gauss_w_lut, block_hists, block_hist_size))
        return false;

    if(!ocl_normalize_hists(nbins, blockStride.width, blockStride.height, effect_size.height,
        effect_size.width, block_hists, (float)L2HysThreshold))
        return false;

    Size blocks_per_win = numPartsWithin(winSize, blockSize, blockStride);
    wins_per_img = numPartsWithin(effect_size, winSize, win_stride);

    int descr_size = blocks_per_win.area()*(int)block_hist_size;
    int descr_width = (int)block_hist_size*blocks_per_win.width;

    UMat descriptors(wins_per_img.area(), static_cast<int>(blocks_per_win.area() * block_hist_size), CV_32F);
    switch (descr_format)
    {
    case HOGDescriptor::DESCR_FORMAT_ROW_BY_ROW:
        if(!ocl_extract_descrs_by_rows(winSize.height, winSize.width,
            blockStride.height, blockStride.width, win_stride.height, win_stride.width, effect_size.height,
            effect_size.width, block_hists, descriptors, (int)block_hist_size, descr_size, descr_width))
            return false;
        break;
    case HOGDescriptor::DESCR_FORMAT_COL_BY_COL:
        if(!ocl_extract_descrs_by_cols(winSize.height, winSize.width,
            blockStride.height, blockStride.width, win_stride.height, win_stride.width, effect_size.height, effect_size.width,
            block_hists, descriptors, (int)block_hist_size, descr_size, blocks_per_win.width, blocks_per_win.height))
            return false;
        break;
    default:
        return false;
    }
    descriptors.reshape(1, (int)descriptors.total()).getMat(ACCESS_READ).copyTo(_descriptors);
    return true;
}
#endif //HAVE_OPENCL

void HOGDescriptor::compute(InputArray _img, std::vector<float>& descriptors,
    Size winStride, Size padding, const std::vector<Point>& locations) const
{
    CV_INSTRUMENT_REGION();

    if( winStride == Size() )
        winStride = cellSize;
    Size cacheStride(gcd(winStride.width, blockStride.width),
                     gcd(winStride.height, blockStride.height));

    Size imgSize = _img.size();

    size_t nwindows = locations.size();
    padding.width = (int)alignSize(std::max(padding.width, 0), cacheStride.width);
    padding.height = (int)alignSize(std::max(padding.height, 0), cacheStride.height);
    Size paddedImgSize(imgSize.width + padding.width*2, imgSize.height + padding.height*2);

    CV_OCL_RUN(_img.dims() <= 2 && _img.type() == CV_8UC1 && _img.isUMat(),
        ocl_compute(_img, winStride, descriptors, HOGDescriptor::DESCR_FORMAT_COL_BY_COL, blockSize,
        cellSize, nbins, blockStride, winSize, (float)getWinSigma(), gammaCorrection, L2HysThreshold, signedGradient))

    Mat img = _img.getMat();
    HOGCache cache(this, img, padding, padding, nwindows == 0, cacheStride);

    if( !nwindows )
        nwindows = cache.windowsInImage(paddedImgSize, winStride).area();

    const HOGCache::BlockData* blockData = &cache.blockData[0];

    int nblocks = cache.nblocks.area();
    int blockHistogramSize = cache.blockHistogramSize;
    size_t dsize = getDescriptorSize();
    descriptors.resize(dsize*nwindows);

    // for each window
    for( size_t i = 0; i < nwindows; i++ )
    {
        float* descriptor = &descriptors[i*dsize];

        Point pt0;
        if( !locations.empty() )
        {
            pt0 = locations[i];
            if( pt0.x < -padding.width || pt0.x > img.cols + padding.width - winSize.width ||
                pt0.y < -padding.height || pt0.y > img.rows + padding.height - winSize.height )
                continue;
        }
        else
        {
            pt0 = cache.getWindow(paddedImgSize, winStride, (int)i).tl() - Point(padding);
//            CV_Assert(pt0.x % cacheStride.width == 0 && pt0.y % cacheStride.height == 0);
        }

        for( int j = 0; j < nblocks; j++ )
        {
            const HOGCache::BlockData& bj = blockData[j];
            Point pt = pt0 + bj.imgOffset;

            float* dst = descriptor + bj.histOfs;
            const float* src = cache.getBlock(pt, dst);
            if( src != dst )
                memcpy(dst, src, blockHistogramSize * sizeof(float));
        }
    }
}

void HOGDescriptor::detect(InputArray _img,
    std::vector<Point>& hits, std::vector<double>& weights, double hitThreshold,
    Size winStride, Size padding, const std::vector<Point>& locations) const
{
    CV_INSTRUMENT_REGION();

    Mat img = _img.getMat();
    hits.clear();
    weights.clear();
    if( svmDetector.empty() )
        return;

    if( winStride == Size() )
        winStride = cellSize;
    Size cacheStride(gcd(winStride.width, blockStride.width),
        gcd(winStride.height, blockStride.height));

    size_t nwindows = locations.size();
    padding.width = (int)alignSize(std::max(padding.width, 0), cacheStride.width);
    padding.height = (int)alignSize(std::max(padding.height, 0), cacheStride.height);
    Size paddedImgSize(img.cols + padding.width*2, img.rows + padding.height*2);

    HOGCache cache(this, img, padding, padding, nwindows == 0, cacheStride);

    if( !nwindows )
        nwindows = cache.windowsInImage(paddedImgSize, winStride).area();

    const HOGCache::BlockData* blockData = &cache.blockData[0];

    int nblocks = cache.nblocks.area();
    int blockHistogramSize = cache.blockHistogramSize;
    size_t dsize = getDescriptorSize();

    double rho = svmDetector.size() > dsize ? svmDetector[dsize] : 0;
    std::vector<float> blockHist(blockHistogramSize);

#if CV_SIMD128
    float partSum[4];
#endif

    for( size_t i = 0; i < nwindows; i++ )
    {
        Point pt0;
        if( !locations.empty() )
        {
            pt0 = locations[i];
            if( pt0.x < -padding.width || pt0.x > img.cols + padding.width - winSize.width ||
                    pt0.y < -padding.height || pt0.y > img.rows + padding.height - winSize.height )
                continue;
        }
        else
        {
            pt0 = cache.getWindow(paddedImgSize, winStride, (int)i).tl() - Point(padding);
            CV_Assert(pt0.x % cacheStride.width == 0 && pt0.y % cacheStride.height == 0);
        }
        double s = rho;
        const float* svmVec = &svmDetector[0];

        int j, k;
        for( j = 0; j < nblocks; j++, svmVec += blockHistogramSize )
        {
            const HOGCache::BlockData& bj = blockData[j];
            Point pt = pt0 + bj.imgOffset;

            const float* vec = cache.getBlock(pt, &blockHist[0]);
#if CV_SIMD128
            v_float32x4 _vec = v_load(vec);
            v_float32x4 _svmVec = v_load(svmVec);
            v_float32x4 sum = v_mul(_svmVec, _vec);

            for( k = 4; k <= blockHistogramSize - 4; k += 4 )
            {
                _vec = v_load(vec + k);
                _svmVec = v_load(svmVec + k);

                sum = v_add(sum, v_mul(_vec, _svmVec));
            }

            v_store(partSum, sum);
            double t0 = partSum[0] + partSum[1];
            double t1 = partSum[2] + partSum[3];
            s += t0 + t1;
#else
            for( k = 0; k <= blockHistogramSize - 4; k += 4 )
                s += vec[k]*svmVec[k] + vec[k+1]*svmVec[k+1] +
                    vec[k+2]*svmVec[k+2] + vec[k+3]*svmVec[k+3];
#endif
            for( ; k < blockHistogramSize; k++ )
                s += vec[k]*svmVec[k];
        }
        if( s >= hitThreshold )
        {
            hits.push_back(pt0);
            weights.push_back(s);
        }
    }
}

void HOGDescriptor::detect(InputArray img, std::vector<Point>& hits, double hitThreshold,
    Size winStride, Size padding, const std::vector<Point>& locations) const
{
    CV_INSTRUMENT_REGION();

    std::vector<double> weightsV;
    detect(img, hits, weightsV, hitThreshold, winStride, padding, locations);
}

class HOGInvoker :
    public ParallelLoopBody
{
public:
    HOGInvoker( const HOGDescriptor* _hog, const Mat& _img,
        double _hitThreshold, const Size& _winStride, const Size& _padding,
        const double* _levelScale, std::vector<Rect> * _vec, Mutex* _mtx,
        std::vector<double>* _weights=0, std::vector<double>* _scales=0 )
    {
        hog = _hog;
        img = _img;
        hitThreshold = _hitThreshold;
        winStride = _winStride;
        padding = _padding;
        levelScale = _levelScale;
        vec = _vec;
        weights = _weights;
        scales = _scales;
        mtx = _mtx;
    }

    void operator()(const Range& range) const CV_OVERRIDE
    {
        int i, i1 = range.start, i2 = range.end;
        double minScale = i1 > 0 ? levelScale[i1] : i2 > 1 ? levelScale[i1+1] : std::max(img.cols, img.rows);
        Size maxSz(cvCeil(img.cols/minScale), cvCeil(img.rows/minScale));
        Mat smallerImgBuf(maxSz, img.type());
        std::vector<Point> locations;
        std::vector<double> hitsWeights;

        for( i = i1; i < i2; i++ )
        {
            double scale = levelScale[i];
            Size sz(cvRound(img.cols/scale), cvRound(img.rows/scale));
            Mat smallerImg(sz, img.type(), smallerImgBuf.ptr());
            if( sz == img.size() )
            {
                smallerImg = Mat(sz, img.type(), img.data, img.step);
            }
            else
            {
                if(getDefaultAlgorithmHint() == ALGO_HINT_APPROX)
                    resize(img, smallerImg, sz, 0, 0, INTER_LINEAR);
                else
                    resize(img, smallerImg, sz, 0, 0, INTER_LINEAR_EXACT);
            }
            hog->detect(smallerImg, locations, hitsWeights, hitThreshold, winStride, padding);
            Size scaledWinSize = Size(cvRound(hog->winSize.width*scale), cvRound(hog->winSize.height*scale));

            mtx->lock();
            for( size_t j = 0; j < locations.size(); j++ )
            {
                vec->push_back(Rect(cvRound(locations[j].x*scale),
                                    cvRound(locations[j].y*scale),
                                    scaledWinSize.width, scaledWinSize.height));
                if (scales)
                    scales->push_back(scale);
            }
            mtx->unlock();

            if (weights && (!hitsWeights.empty()))
            {
                mtx->lock();
                for (size_t j = 0; j < locations.size(); j++)
                    weights->push_back(hitsWeights[j]);
                mtx->unlock();
            }
        }
    }

private:
    const HOGDescriptor* hog;
    Mat img;
    double hitThreshold;
    Size winStride;
    Size padding;
    const double* levelScale;
    std::vector<Rect>* vec;
    std::vector<double>* weights;
    std::vector<double>* scales;
    Mutex* mtx;
};

#ifdef HAVE_OPENCL

static bool ocl_classify_hists(int win_height, int win_width, int block_stride_y, int block_stride_x,
                               int win_stride_y, int win_stride_x, int height, int width,
                               const UMat& block_hists, UMat detector,
                               float free_coef, float threshold, UMat& labels, Size descr_size, int block_hist_size)
{
    int nthreads;
    cv::String opts;

    ocl::Kernel k;
    int idx = 0;
    switch (descr_size.width)
    {
    case 180:
        nthreads = 180;
        k.create("classify_hists_180_kernel", ocl::objdetect::objdetect_hog_oclsrc, "");
        if(k.empty())
            return false;
        idx = k.set(idx, descr_size.width);
        idx = k.set(idx, descr_size.height);
        break;

    case 252:
        nthreads = 256;
        k.create("classify_hists_252_kernel", ocl::objdetect::objdetect_hog_oclsrc, "");
        if(k.empty())
            return false;
        idx = k.set(idx, descr_size.width);
        idx = k.set(idx, descr_size.height);
        break;

    default:
        nthreads = 256;
        k.create("classify_hists_kernel", ocl::objdetect::objdetect_hog_oclsrc, "");
        if(k.empty())
            return false;
        idx = k.set(idx, descr_size.area());
        idx = k.set(idx, descr_size.height);
    }

    int win_block_stride_x = win_stride_x / block_stride_x;
    int win_block_stride_y = win_stride_y / block_stride_y;
    int img_win_width = (width - win_width + win_stride_x) / win_stride_x;
    int img_win_height = (height - win_height + win_stride_y) / win_stride_y;
    int img_block_width = (width - CELLS_PER_BLOCK_X * CELL_WIDTH + block_stride_x) /
        block_stride_x;

    size_t globalThreads[3] = { (size_t)img_win_width * nthreads, (size_t)img_win_height, 1 };
    size_t localThreads[3] = { (size_t)nthreads, 1, 1 };

    idx = k.set(idx, block_hist_size);
    idx = k.set(idx, img_win_width);
    idx = k.set(idx, img_block_width);
    idx = k.set(idx, win_block_stride_x);
    idx = k.set(idx, win_block_stride_y);
    idx = k.set(idx, ocl::KernelArg::PtrReadOnly(block_hists));
    idx = k.set(idx, ocl::KernelArg::PtrReadOnly(detector));
    idx = k.set(idx, free_coef);
    idx = k.set(idx, threshold);
    idx = k.set(idx, ocl::KernelArg::PtrWriteOnly(labels));

    return k.run(2, globalThreads, localThreads, false);
}

static bool ocl_detect(InputArray img, std::vector<Point> &hits, double hit_threshold, Size win_stride,
                       const UMat& oclSvmDetector, Size blockSize, Size cellSize, int nbins, Size blockStride, Size winSize,
                       bool gammaCorrection, double L2HysThreshold, float sigma, float free_coef, bool signedGradient)
{
    hits.clear();
    if (oclSvmDetector.empty())
        return false;

    Size imgSize = img.size();
    Size effect_size = imgSize;
    UMat grad(imgSize, CV_32FC2);
    int qangle_type = ocl::Device::getDefault().isIntel() ? CV_32SC2 : CV_8UC2;
    UMat qangle(imgSize, qangle_type);

    const size_t block_hist_size = getBlockHistogramSize(blockSize, cellSize, nbins);
    const Size blocks_per_img = numPartsWithin(imgSize, blockSize, blockStride);
    UMat block_hists(1, static_cast<int>(block_hist_size * blocks_per_img.area()) + 256, CV_32F);

    Size wins_per_img = numPartsWithin(imgSize, winSize, win_stride);
    UMat labels(1, wins_per_img.area(), CV_8U);

    float scale = 1.f / (2.f * sigma * sigma);
    Mat gaussian_lut(1, 512, CV_32FC1);
    int idx = 0;
    for(int i=-8; i<8; i++)
        for(int j=-8; j<8; j++)
            gaussian_lut.at<float>(idx++) = std::exp(-(j * j + i * i) * scale);
    for(int i=-8; i<8; i++)
        for(int j=-8; j<8; j++)
            gaussian_lut.at<float>(idx++) = (8.f - fabs(j + 0.5f)) * (8.f - fabs(i + 0.5f)) / 64.f;

    if(!ocl_computeGradient(img, grad, qangle, nbins, effect_size, gammaCorrection, signedGradient))
        return false;

    UMat gauss_w_lut;
    gaussian_lut.copyTo(gauss_w_lut);
    if(!ocl_compute_hists(nbins, blockStride.width, blockStride.height, effect_size.height,
        effect_size.width, grad, qangle, gauss_w_lut, block_hists, block_hist_size))
        return false;

    if(!ocl_normalize_hists(nbins, blockStride.width, blockStride.height, effect_size.height,
        effect_size.width, block_hists, (float)L2HysThreshold))
        return false;

    Size blocks_per_win = numPartsWithin(winSize, blockSize, blockStride);

    Size descr_size((int)block_hist_size*blocks_per_win.width, blocks_per_win.height);

    if(!ocl_classify_hists(winSize.height, winSize.width, blockStride.height,
        blockStride.width, win_stride.height, win_stride.width,
        effect_size.height, effect_size.width, block_hists, oclSvmDetector,
        free_coef, (float)hit_threshold, labels, descr_size, (int)block_hist_size))
        return false;

    Mat labels_host = labels.getMat(ACCESS_READ);
    unsigned char *vec = labels_host.ptr();
    for (int i = 0; i < wins_per_img.area(); i++)
    {
        int y = i / wins_per_img.width;
        int x = i - wins_per_img.width * y;
        if (vec[i])
        {
            hits.push_back(Point(x * win_stride.width, y * win_stride.height));
        }
    }
    return true;
}

static bool ocl_detectMultiScale(InputArray _img, std::vector<Rect> &found_locations, std::vector<double>& level_scale,
                                              double hit_threshold, Size win_stride, double group_threshold,
                                              const UMat& oclSvmDetector, Size blockSize, Size cellSize,
                                              int nbins, Size blockStride, Size winSize, bool gammaCorrection,
                                              double L2HysThreshold, float sigma, float free_coef, bool signedGradient)
{
    std::vector<Rect> all_candidates;
    std::vector<Point> locations;
    UMat image_scale;
    Size imgSize = _img.size();
    image_scale.create(imgSize, _img.type());

    for (size_t i = 0; i<level_scale.size() ; i++)
    {
        double scale = level_scale[i];
        Size effect_size = Size(cvRound(imgSize.width / scale), cvRound(imgSize.height / scale));
        if (effect_size == imgSize)
        {
            if(!ocl_detect(_img, locations, hit_threshold, win_stride, oclSvmDetector, blockSize, cellSize, nbins,
                blockStride, winSize, gammaCorrection, L2HysThreshold, sigma, free_coef, signedGradient))
                return false;
        }
        else
        {
            resize(_img, image_scale, effect_size, 0, 0, INTER_LINEAR_EXACT);
            if(!ocl_detect(image_scale, locations, hit_threshold, win_stride, oclSvmDetector, blockSize, cellSize, nbins,
                blockStride, winSize, gammaCorrection, L2HysThreshold, sigma, free_coef, signedGradient))
                return false;
        }
        Size scaled_win_size(cvRound(winSize.width * scale),
            cvRound(winSize.height * scale));
        for (size_t j = 0; j < locations.size(); j++)
            all_candidates.push_back(Rect(Point2d(locations[j]) * scale, scaled_win_size));
    }
    found_locations.assign(all_candidates.begin(), all_candidates.end());
    groupRectangles(found_locations, (int)group_threshold, 0.2);
    clipObjects(imgSize, found_locations, 0, 0);

    return true;
}
#endif //HAVE_OPENCL

void HOGDescriptor::detectMultiScale(
    InputArray _img, std::vector<Rect>& foundLocations, std::vector<double>& foundWeights,
    double hitThreshold, Size winStride, Size padding,
    double scale0, double groupThreshold, bool useMeanshiftGrouping) const
{
    CV_INSTRUMENT_REGION();

    double scale = 1.;
    int levels = 0;

    Size imgSize = _img.size();
    std::vector<double> levelScale;
    for( levels = 0; levels < nlevels; levels++ )
    {
        levelScale.push_back(scale);
        if( cvRound(imgSize.width/scale) < winSize.width ||
            cvRound(imgSize.height/scale) < winSize.height ||
                scale0 <= 1 )
            break;
        scale *= scale0;
    }
    levels = std::max(levels, 1);
    levelScale.resize(levels);

    if(winStride == Size())
        winStride = blockStride;

    CV_OCL_RUN(_img.dims() <= 2 && _img.type() == CV_8UC1 && scale0 > 1 && winStride.width % blockStride.width == 0 &&
        winStride.height % blockStride.height == 0 && padding == Size(0,0) && _img.isUMat(),
        ocl_detectMultiScale(_img, foundLocations, levelScale, hitThreshold, winStride, groupThreshold, oclSvmDetector,
        blockSize, cellSize, nbins, blockStride, winSize, gammaCorrection, L2HysThreshold, (float)getWinSigma(), free_coef, signedGradient));

    std::vector<Rect> allCandidates;
    std::vector<double> tempScales;
    std::vector<double> tempWeights;
    std::vector<double> foundScales;

    Mutex mtx;
    Mat img = _img.getMat();
    Range range(0, (int)levelScale.size());
    HOGInvoker invoker(this, img, hitThreshold, winStride, padding, &levelScale[0], &allCandidates, &mtx, &tempWeights, &tempScales);
    parallel_for_(range, invoker);

    std::copy(tempScales.begin(), tempScales.end(), back_inserter(foundScales));
    foundLocations.clear();
    std::copy(allCandidates.begin(), allCandidates.end(), back_inserter(foundLocations));
    foundWeights.clear();
    std::copy(tempWeights.begin(), tempWeights.end(), back_inserter(foundWeights));

    if ( useMeanshiftGrouping )
        groupRectangles_meanshift(foundLocations, foundWeights, foundScales, groupThreshold, winSize);
    else
        groupRectangles(foundLocations, foundWeights, (int)groupThreshold, 0.2);
    clipObjects(imgSize, foundLocations, 0, &foundWeights);
}

void HOGDescriptor::detectMultiScale(InputArray img, std::vector<Rect>& foundLocations,
    double hitThreshold, Size winStride, Size padding,
    double scale0, double groupThreshold, bool useMeanshiftGrouping) const
{
    CV_INSTRUMENT_REGION();

    std::vector<double> foundWeights;
    detectMultiScale(img, foundLocations, foundWeights, hitThreshold, winStride,
                padding, scale0, groupThreshold, useMeanshiftGrouping);
}

std::vector<float> HOGDescriptor::getDefaultPeopleDetector()
{
    static const float detector[] = {
        0.05359386f, -0.14721455f, -0.05532170f, 0.05077307f,
        0.11547081f, -0.04268804f, 0.04635834f, -0.05468199f, 0.08232084f,
        0.10424068f, -0.02294518f, 0.01108519f, 0.01378693f, 0.11193510f,
        0.01268418f, 0.08528346f, -0.06309239f, 0.13054633f, 0.08100729f,
        -0.05209739f, -0.04315529f, 0.09341384f, 0.11035026f, -0.07596218f,
        -0.05517511f, -0.04465296f, 0.02947334f, 0.04555536f,
        -3.55954492e-003f, 0.07818956f, 0.07730991f, 0.07890715f, 0.06222893f,
        0.09001380f, -0.03574381f, 0.03414327f, 0.05677258f, -0.04773581f,
        0.03746637f, -0.03521175f, 0.06955440f, -0.03849038f, 0.01052293f,
        0.01736112f, 0.10867710f, 0.08748853f, 3.29739624e-003f, 0.10907028f,
        0.07913758f, 0.10393070f, 0.02091867f, 0.11594022f, 0.13182420f,
        0.09879354f, 0.05362710f, -0.06745391f, -7.01260753e-003f,
        5.24702156e-003f, 0.03236255f, 0.01407916f, 0.02207983f, 0.02537322f,
        0.04547948f, 0.07200756f, 0.03129894f, -0.06274468f, 0.02107014f,
        0.06035208f, 0.08636236f, 4.53164103e-003f, 0.02193363f, 0.02309801f,
        0.05568166f, -0.02645093f, 0.04448695f, 0.02837519f, 0.08975694f,
        0.04461516f, 0.08975355f, 0.07514391f, 0.02306982f, 0.10410084f,
        0.06368385f, 0.05943464f, 4.58420580e-003f, 0.05220337f, 0.06675851f,
        0.08358569f, 0.06712101f, 0.06559004f, -0.03930482f, -9.15936660e-003f,
        -0.05897915f, 0.02816453f, 0.05032348f, 0.06780671f, 0.03377650f,
        -6.09417039e-004f, -0.01795146f, -0.03083684f, -0.01302475f,
        -0.02972313f, 7.88706727e-003f, -0.03525961f, -2.50397739e-003f,
        0.05245084f, 0.11791293f, -0.02167498f, 0.05299332f, 0.06640524f,
        0.05190265f, -8.27316567e-003f, 0.03033127f, 0.05842173f,
        -4.01050318e-003f, -6.25105947e-003f, 0.05862958f, -0.02465461f,
        0.05546781f, -0.08228195f, -0.07234028f, 0.04640540f, -0.01308254f,
        -0.02506191f, 0.03100746f, -0.04665651f, -0.04591486f, 0.02949927f,
        0.06035462f, 0.02244646f, -0.01698639f, 0.01040041f, 0.01131170f,
        0.05419579f, -0.02130277f, -0.04321722f, -0.03665198f, 0.01126490f,
        -0.02606488f, -0.02228328f, -0.02255680f, -0.03427236f,
        -7.75165204e-003f, -0.06195229f, 8.21638294e-003f, 0.09535975f,
        -0.03709979f, -0.06942501f, 0.14579427f, -0.05448192f, -0.02055904f,
        0.05747357f, 0.02781788f, -0.07077577f, -0.05178314f, -0.10429011f,
        -0.11235505f, 0.07529039f, -0.07559302f, -0.08786739f, 0.02983843f,
        0.02667585f, 0.01382199f, -0.01797496f, -0.03141199f, -0.02098101f,
        0.09029204f, 0.04955018f, 0.13718739f, 0.11379953f, 1.80019124e-003f,
        -0.04577610f, -1.11108483e-003f, -0.09470536f, -0.11596080f,
        0.04489342f, 0.01784211f, 3.06850672e-003f, 0.10781866f,
        3.36498418e-003f, -0.10842580f, -0.07436839f, -0.10535070f,
        -0.01866805f, 0.16057891f, -5.07316366e-003f, -0.04295658f,
        -5.90488780e-003f, 8.82003549e-003f, -0.01492646f, -0.05029279f,
        -0.12875880f, 8.78831954e-004f, -0.01297184f, -0.07592774f,
        -0.02668831f, -6.93787413e-004f, 0.02406698f, -0.01773298f,
        -0.03855745f, -0.05877856f, 0.03259695f, 0.12826584f, 0.06292590f,
        -4.10733931e-003f, 0.10996531f, 0.01332991f, 0.02088735f, 0.04037504f,
        -0.05210760f, 0.07760046f, 0.06399347f, -0.05751930f, -0.10053057f,
        0.07505023f, -0.02139782f, 0.01796176f, 2.34400877e-003f, -0.04208319f,
        0.07355055f, 0.05093350f, -0.02996780f, -0.02219072f, 0.03355330f,
        0.04418742f, -0.05580705f, -0.05037573f, -0.04548179f, 0.01379514f,
        0.02150671f, -0.02194211f, -0.13682702f, 0.05464972f, 0.01608082f,
        0.05309116f, 0.04701022f, 1.33690401e-003f, 0.07575664f, 0.09625306f,
        8.92647635e-003f, -0.02819123f, 0.10866830f, -0.03439325f,
        -0.07092371f, -0.06004780f, -0.02712298f, -7.07467366e-003f,
        -0.01637020f, 0.01336790f, -0.10313606f, 0.04906582f, -0.05732445f,
        -0.02731079f, 0.01042235f, -0.08340668f, 0.03686501f, 0.06108340f,
        0.01322748f, -0.07809529f, 0.03774724f, -0.03413248f, -0.06096525f,
        -0.04212124f, -0.07982176f, -1.25973229e-003f, -0.03045501f,
        -0.01236493f, -0.06312395f, 0.04789570f, -0.04602066f, 0.08576570f,
        0.02521080f, 0.02988098f, 0.10314583f, 0.07060035f, 0.04520544f,
        -0.04426654f, 0.13146530f, 0.08386490f, 0.02164590f, -2.12280243e-003f,
        -0.03686353f, -0.02074944f, -0.03829959f, -0.01530596f, 0.02689708f,
        0.11867401f, -0.06043470f, -0.02785023f, -0.04775074f, 0.04878745f,
        0.06350956f, 0.03494788f, 0.01467400f, 1.17890188e-003f, 0.04379614f,
        2.03681854e-003f, -0.03958609f, -0.01072688f, 6.43705716e-003f,
        0.02996500f, -0.03418507f, -0.01960307f, -0.01219154f,
        -4.37000440e-003f, -0.02549453f, 0.02646318f, -0.01632513f,
        6.46516960e-003f, -0.01929734f, 4.78711911e-003f, 0.04962371f,
        0.03809111f, 0.07265724f, 0.05758125f, -0.03741554f, 0.01648608f,
        -8.45285598e-003f, 0.03996826f, -0.08185477f, 0.02638875f,
        -0.04026615f, -0.02744674f, -0.04071517f, 1.05096330e-003f,
        -0.04741232f, -0.06733172f, 8.70434940e-003f, -0.02192543f,
        1.35350740e-003f, -0.03056974f, -0.02975521f, -0.02887780f,
        -0.01210713f, -0.04828526f, -0.09066251f, -0.09969629f, -0.03665164f,
        -8.88111943e-004f, -0.06826669f, -0.01866150f, -0.03627640f,
        -0.01408288f, 0.01874239f, -0.02075835f, 0.09145175f, -0.03547291f,
        0.05396780f, 0.04198981f, 0.01301925f, -0.03384354f, -0.12201976f,
        0.06830920f, -0.03715654f, 9.55848210e-003f, 5.05685573e-003f,
        0.05659294f, 3.90764466e-003f, 0.02808490f, -0.05518097f, -0.03711621f,
        -0.02835565f, -0.04420464f, -0.01031947f, 0.01883466f,
        -8.49525444e-003f, -0.09419250f, -0.01269387f, -0.02133371f,
        -0.10190815f, -0.07844430f, 2.43644323e-003f, -4.09610150e-003f,
        0.01202551f, -0.06452291f, -0.10593818f, -0.02464746f, -0.02199699f,
        -0.07401930f, 0.07285886f, 8.87513801e-004f, 9.97662079e-003f,
        8.46779719e-003f, 0.03730333f, -0.02905126f, 0.03573337f, -0.04393689f,
        -0.12014472f, 0.03176554f, -2.76015815e-003f, 0.10824566f, 0.05090732f,
        -3.30179278e-003f, -0.05123822f, 5.04784798e-003f, -0.05664124f,
        -5.99415926e-003f, -0.05341901f, -0.01221393f, 0.01291318f,
        9.91760660e-003f, -7.56987557e-003f, -0.06193124f, -2.24549137e-003f,
        0.01987562f, -0.02018840f, -0.06975540f, -0.06601523f, -0.03349112f,
        -0.08910118f, -0.03371435f, -0.07406893f, -0.02248047f, -0.06159951f,
        2.77751544e-003f, -0.05723337f, -0.04792468f, 0.07518548f,
        2.77279224e-003f, 0.04211938f, 0.03100502f, 0.05278448f, 0.03954679f,
        -0.03006846f, -0.03851741f, -0.02792403f, -0.02875333f, 0.01531280f,
        0.02186953f, -0.01989829f, 2.50679464e-003f, -0.10258728f,
        -0.04785743f, -0.02887216f, 3.85063468e-003f, 0.01112236f,
        8.29218887e-003f, -0.04822981f, -0.04503597f, -0.03713100f,
        -0.06988008f, -0.11002295f, -2.69209221e-003f, 1.85383670e-003f,
        -0.05921049f, -0.06105053f, -0.08458050f, -0.04527602f,
        8.90329306e-004f, -0.05875023f, -2.68602883e-003f, -0.01591195f,
        0.03631859f, 0.05493166f, 0.07300330f, 5.53333294e-003f, 0.06400407f,
        0.01847740f, -5.76280477e-003f, -0.03210877f, 4.25160583e-003f,
        0.01166520f, -1.44864211e-003f, 0.02253744f, -0.03367080f, 0.06983195f,
        -4.22323542e-003f, -8.89401045e-003f, -0.07943393f, 0.05199728f,
        0.06065201f, 0.04133492f, 1.44032843e-003f, -0.09585235f, -0.03964731f,
        0.04232114f, 0.01750465f, -0.04487902f, -7.59733608e-003f, 0.02011171f,
        0.04673622f, 0.09011173f, -0.07869188f, -0.04682482f, -0.05080139f,
        -3.99383716e-003f, -0.05346331f, 0.01085723f, -0.03599333f,
        -0.07097908f, 0.03551549f, 0.02680387f, 0.03471529f, 0.01790393f,
        0.05471273f, 9.62048303e-003f, -0.03180215f, 0.05864431f, 0.02330614f,
        0.01633144f, -0.05616681f, -0.10245429f, -0.08302189f, 0.07291322f,
        -0.01972590f, -0.02619633f, -0.02485327f, -0.04627592f,
        1.48853404e-003f, 0.05514185f, -0.01270860f, -0.01948900f, 0.06373586f,
        0.05002292f, -0.03009798f, 8.76216311e-003f, -0.02474238f,
        -0.05504891f, 1.74034527e-003f, -0.03333667f, 0.01524987f, 0.11663762f,
        -1.32344989e-003f, -0.06608453f, 0.05687166f, -6.89525274e-004f,
        -0.04402352f, 0.09450210f, -0.04222684f, -0.05360983f, 0.01779531f,
        0.02561388f, -0.11075410f, -8.77790991e-003f, -0.01099504f,
        -0.10380266f, 0.03103457f, -0.02105741f, -0.07371717f, 0.05146710f,
        0.10581432f, -0.08617968f, -0.02892107f, 0.01092199f, 0.14551543f,
        -2.24320893e-003f, -0.05818033f, -0.07390742f, 0.05701261f,
        0.12937020f, -0.04986651f, 0.10182415f, 0.05028650f, 0.12515625f,
        0.09175041f, 0.06404983f, 0.01523394f, 0.09460562f, 0.06106631f,
        -0.14266998f, -0.02926703f, 0.02762171f, 0.02164151f,
        -9.58488265e-004f, -0.04231362f, -0.09866509f, 0.04322244f,
        0.05872034f, -0.04838847f, 0.06319253f, 0.02443798f, -0.03606876f,
        9.38737206e-003f, 0.04289991f, -0.01027411f, 0.08156885f, 0.08751175f,
        -0.13191354f, 8.16054735e-003f, -0.01452161f, 0.02952677f, 0.03615945f,
        -2.09128903e-003f, 0.02246693f, 0.09623287f, 0.09412123f, -0.02924758f,
        -0.07815186f, -0.02203079f, -2.02566991e-003f, 0.01094733f,
        -0.01442332f, 0.02838561f, 0.11882371f, 7.28798332e-003f, -0.10345965f,
        0.07561217f, -0.02049661f, 4.44177445e-003f, 0.01609347f, -0.04893158f,
        -0.08758243f, -7.67420698e-003f, 0.08862378f, 0.06098121f, 0.06565887f,
        7.32981879e-003f, 0.03558407f, -0.03874352f, -0.02490055f,
        -0.06771075f, 0.09939223f, -0.01066077f, 0.01382995f, -0.07289080f,
        7.47184316e-003f, 0.10621431f, -0.02878659f, 0.02383525f, -0.03274646f,
        0.02137008f, 0.03837290f, 0.02450992f, -0.04296818f, -0.02895143f,
        0.05327370f, 0.01499020f, 0.04998732f, 0.12938657f, 0.09391870f,
        0.04292390f, -0.03359194f, -0.06809492f, 0.01125796f, 0.17290455f,
        -0.03430733f, -0.06255233f, -0.01813114f, 0.11726857f, -0.06127599f,
        -0.08677909f, -0.03429872f, 0.04684938f, 0.08161420f, 0.03538774f,
        0.01833884f, 0.11321855f, 0.03261845f, -0.04826299f, 0.01752407f,
        -0.01796414f, -0.10464549f, -3.30041884e-003f, 2.29343961e-004f,
        0.01457292f, -0.02132982f, -0.02602923f, -9.87351313e-003f,
        0.04273872f, -0.02103316f, -0.07994065f, 0.02614958f, -0.02111666f,
        -0.06964913f, -0.13453490f, -0.06861878f, -6.09341264e-003f,
        0.08251446f, 0.15612499f, 2.46531400e-003f, 8.88424646e-003f,
        -0.04152999f, 0.02054853f, 0.05277953f, -0.03087788f, 0.02817579f,
        0.13939077f, 0.07641046f, -0.03627627f, -0.03015098f, -0.04041540f,
        -0.01360690f, -0.06227205f, -0.02738223f, 0.13577610f, 0.15235767f,
        -0.05392922f, -0.11175954f, 0.02157129f, 0.01146481f, -0.05264937f,
        -0.06595174f, -0.02749175f, 0.11812254f, 0.17404149f, -0.06137035f,
        -0.11003478f, -0.01351621f, -0.01745916f, -0.08577441f, -0.04469909f,
        -0.06106115f, 0.10559758f, 0.20806813f, -0.09174948f, 7.09621934e-004f,
        0.03579374f, 0.07215115f, 0.02221742f, 0.01827742f, -7.90785067e-003f,
        0.01489554f, 0.14519960f, -0.06425831f, 0.02990399f, -1.80181325e-003f,
        -0.01401528f, -0.04171134f, -3.70530109e-003f, -0.09090481f,
        0.09520713f, 0.08845516f, -0.02651753f, -0.03016730f, 0.02562448f,
        0.03563816f, -0.03817881f, 0.01433385f, 0.02256983f, 0.02872120f,
        0.01001934f, -0.06332260f, 0.04338406f, 0.07001807f, -0.04705722f,
        -0.07318907f, 0.02630457f, 0.03106382f, 0.06648342f, 0.10913180f,
        -0.01630815f, 0.02910308f, 0.02895109f, 0.08040254f, 0.06969310f,
        0.06797734f, 6.08639978e-003f, 4.16588830e-003f, 0.08926726f,
        -0.03123648f, 0.02700146f, 0.01168734f, -0.01631594f, 4.61015804e-003f,
        8.51359498e-003f, -0.03544224f, 0.03571994f, 4.29766066e-003f,
        -0.01970077f, -8.79793242e-003f, 0.09607988f, 0.01544222f,
        -0.03923707f, 0.07308586f, 0.06061262f, 1.31683104e-004f,
        -7.98222050e-003f, 0.02399261f, -0.06084389f, -0.02743429f,
        -0.05475523f, -0.04131311f, 0.03559756f, 0.03055342f, 0.02981433f,
        0.14860515f, 0.01766787f, 0.02945257f, 0.04898238f, 0.01026922f,
        0.02811658f, 0.08267091f, 0.02732154f, -0.01237693f, 0.11760156f,
        0.03802063f, -0.03309754f, 5.24957618e-003f, -0.02460510f, 0.02691451f,
        0.05399988f, -0.10133506f, 0.06385437f, -0.01818005f, 0.02259503f,
        0.03573135f, 0.01042848f, -0.04153402f, -0.04043029f, 0.01643575f,
        0.08326677f, 4.61383024e-004f, -0.05308095f, -0.08536223f,
        -1.61011645e-003f, -0.02163720f, -0.01783352f, 0.03859637f,
        0.08498885f, -0.01725216f, 0.08625131f, 0.10995087f, 0.09177644f,
        0.08498347f, 0.07646490f, 0.05580502f, 0.02693516f, 0.09996913f,
        0.09070327f, 0.06667200f, 0.05873008f, -0.02247842f, 0.07772321f,
        0.12408436f, 0.12629253f, -8.41997913e-004f, 0.01477783f, 0.09165990f,
        -2.98401713e-003f, -0.06466447f, -0.07057302f, 2.09516948e-004f,
        0.02210209f, -0.02158809f, -0.08602506f, -0.02284836f,
        4.01876355e-003f, 9.56660323e-003f, -0.02073978f, -0.04635138f,
        -7.59423291e-003f, -0.01377393f, -0.04559359f, -0.13284740f,
        -0.08671406f, -0.03654395f, 0.01142869f, 0.03287891f, -0.04392983f,
        0.06142959f, 0.17710890f, 0.10385257f, 0.01329137f, 0.10067633f,
        0.12450829f, -0.04476709f, 0.09049144f, 0.04589312f, 0.11167907f,
        0.08587538f, 0.04767583f, 1.67188141e-003f, 0.02359802f, -0.03808852f,
        0.03126272f, -0.01919029f, -0.05698918f, -0.02365112f, -0.06519032f,
        -0.05599358f, -0.07097308f, -0.03301812f, -0.04719102f, -0.02566297f,
        0.01324074f, -0.09230672f, -0.05518232f, -0.04712864f, -0.03380903f,
        -0.06719479f, 0.01183908f, -0.09326738f, 0.01642865f, 0.03789867f,
        -6.61567831e-003f, 0.07796386f, 0.07246574f, 0.04706347f, -0.02523437f,
        -0.01696830f, -0.08068866f, 0.06030888f, 0.10527060f, -0.06611756f,
        0.02977346f, 0.02621830f, 0.01913855f, -0.08479366f, -0.06322418f,
        -0.13570616f, -0.07644490f, 9.31900274e-003f, -0.08095149f,
        -0.10197903f, -0.05204025f, 0.01413151f, -0.07800411f, -0.01885122f,
        -0.07509381f, -0.10136326f, -0.05212355f, -0.09944065f,
        -1.33606605e-003f, -0.06342617f, -0.04178550f, -0.12373723f,
        -0.02832736f, -0.06057501f, 0.05830070f, 0.07604282f, -0.06462587f,
        8.02447461e-003f, 0.11580125f, 0.12332212f, 0.01978462f,
        -2.72378162e-003f, 0.05850752f, -0.04674481f, 0.05148062f,
        -2.62542837e-003f, 0.11253355f, 0.09893716f, 0.09785093f, -0.04659257f,
        -0.01102429f, -0.07002308f, 0.03088913f, -0.02565549f, -0.07671449f,
        3.17443861e-003f, -0.10783514f, -0.02314270f, -0.11089555f,
        -0.01024768f, 0.03116021f, -0.04964825f, 0.02281825f, 5.50005678e-003f,
        -0.08427856f, -0.14685495f, -0.07719755f, -0.13342668f, -0.04525511f,
        -0.09914210f, 0.02588859f, 0.03469279f, 0.04664020f, 0.11688190f,
        0.09647275f, 0.10857815f, -0.01448726f, 0.04299758f, -0.06763151f,
        1.33257592e-003f, 0.14331576f, 0.07574340f, 0.09166205f, 0.05674926f,
        0.11325553f, -0.01106494f, 0.02062161f, -0.11484840f, -0.07492137f,
        -0.02864293f, -0.01275638f, -0.06946032f, -0.10101652f, -0.04113498f,
        -0.02214783f, -0.01273942f, -0.07480393f, -0.10556041f, -0.07622112f,
        -0.09988393f, -0.11453961f, -0.12073903f, -0.09412795f, -0.07146588f,
        -0.04054537f, -0.06127083f, 0.04221122f, 0.07688113f, 0.04099256f,
        0.12663734f, 0.14683802f, 0.21761774f, 0.12525328f, 0.18431792f,
        -1.66402373e-003f, 2.37777247e-003f, 0.01445475f, 0.03509416f,
        0.02654697f, 0.01716739f, 0.05374011f, 0.02944174f, 0.11323927f,
        -0.01485456f, -0.01611330f, -1.85554172e-003f, -0.01708549f,
        -0.05435753f, -0.05302101f, 0.05260378f, -0.03582945f,
        -3.42867890e-004f, 1.36076682e-003f, -0.04436073f, -0.04228432f,
        0.03281291f, -0.05480836f, -0.10197772f, -0.07206279f, -0.10741059f,
        -0.02366946f, 0.10278475f, -2.74783419e-003f, -0.03242477f,
        0.02308955f, 0.02835869f, 0.10348799f, 0.19580358f, 0.10252027f,
        0.08039929f, 0.05525554f, -0.13250865f, -0.14395352f, 3.13586881e-003f,
        -0.03387071f, 8.94669443e-003f, 0.05406157f, -4.97324532e-003f,
        -0.01189114f, 2.82919413e-004f, -0.03901557f, -0.04898705f,
        0.02164520f, -0.01382906f, -0.01850416f, 0.01869347f, -0.02450060f,
        0.02291678f, 0.08196463f, 0.03309153f, -0.10629974f, 0.02473924f,
        0.05344394f, -0.02404823f, -0.03243643f, -5.55244600e-003f,
        -0.08009996f, 0.02811539f, 0.04235742f, 0.01859004f, 0.04902123f,
        -0.01438252f, -0.01526853f, 0.02044195f, -0.05008660f, 0.04244113f,
        0.07611816f, 0.04950470f, -0.06020549f, -4.26026015e-003f, 0.13133512f,
        -0.01438738f, -0.01958807f, -0.04044152f, -0.12425045f,
        2.84353318e-003f, -0.05042776f, -0.09121484f, 7.34345755e-003f,
        0.09388847f, 0.11800314f, 4.72295098e-003f, 4.44378285e-003f,
        -0.07984917f, -0.03613737f, 0.04490915f, -0.02246483f, 0.04681071f,
        0.05240871f, 0.02157206f, -0.04603431f, -0.01197929f, -0.02748779f,
        0.13621049f, 0.08812155f, -0.07802048f, 4.86458559e-003f, -0.01598836f,
        0.01024450f, -0.03463517f, -0.02304239f, -0.08692665f, 0.06655128f,
        0.05785803f, -0.12640759f, 0.02307472f, 0.07337402f, 0.07525434f,
        0.04943763f, -0.02241034f, -0.09978238f, 0.14487994f, -0.06570521f,
        -0.07855482f, 0.02830222f, -5.29603509e-004f, -0.04669895f,
        -0.11822784f, -0.12246452f, -0.15365660f, -0.02969127f, 0.08078201f,
        0.13512598f, 0.11505685f, 0.04740673f, 0.01376022f, -0.05852978f,
        -0.01537809f, -0.05541119f, 0.02491065f, -0.02870786f, 0.02760978f,
        0.23836176f, 0.22347429f, 0.10306466f, -0.06919070f, -0.10132039f,
        -0.20198342f, -0.05040560f, 0.27163076f, 0.36987007f, 0.34540465f,
        0.29095781f, 0.05649706f, 0.04125737f, 0.07505883f, -0.02737836f,
        -8.43431335e-003f, 0.07368195f, 0.01653876f, -0.09402955f,
        -0.09574359f, 0.01474337f, -0.07128561f, -0.03460737f, 0.11438941f,
        0.13752601f, -0.06385452f, -0.06310338f, 8.19548313e-003f, 0.11622470f,
        5.05133113e-003f, -0.07602754f, 0.06695660f, 0.25723928f, 0.09037900f,
        0.28826267f, 0.13165380f, -0.05312614f, -0.02137198f, -0.03442232f,
        -0.06255679f, 0.03899667f, 0.18391028f, 0.26016650f, 0.03374462f,
        0.01860465f, 0.19077586f, 0.18160543f, 3.43634398e-003f, -0.03036782f,
        0.19683038f, 0.35378191f, 0.24968483f, -0.03222649f, 0.28972381f,
        0.43091634f, 0.30778357f, 0.02335266f, -0.09877399f, -6.85245218e-003f,
        0.08945240f, -0.08150686f, 0.02792493f, 0.24806842f, 0.17338486f,
        0.06231801f, -0.10432383f, -0.16653322f, -0.13197899f, -0.08531576f,
        -0.19271527f, -0.13536365f, 0.22240199f, 0.39219588f, 0.26597717f,
        -0.01231649f, 0.01016179f, 0.13379875f, 0.12018334f, -0.04852953f,
        -0.07915270f, 0.07036012f, 3.87723115e-003f, -0.06126805f,
        -0.15015170f, -0.11406515f, -0.08556531f, -0.07429333f, -0.16115491f,
        0.13214062f, 0.25691369f, 0.05697750f, 0.06861912f, -6.02903729e-003f,
        -7.94562511e-003f, 0.04799571f, 0.06695165f, -0.01926842f, 0.06206308f,
        0.13450983f, -0.06381495f, -2.98370165e-003f, -0.03482971f,
        7.53991678e-003f, 0.03895611f, 0.11464261f, 0.01669971f,
        8.27818643e-003f, -7.49160210e-003f, -0.11712562f, -0.10650621f,
        -0.10353880f, -0.04994106f, -7.65618810e-004f, 0.03023767f,
        -0.04759270f, -0.07302686f, -0.05825012f, -0.13156348f, -0.10639747f,
        -0.19393684f, -0.09973683f, -0.07918908f, 4.63177625e-004f,
        -6.61382044e-004f, 0.15853868f, 0.08561199f, -0.07660093f,
        -0.08015265f, -0.06164073f, 0.01882577f, -7.29908410e-004f,
        0.06840892f, 0.03843764f, 0.20274927f, 0.22028814f, -5.26101235e-003f,
        0.01452435f, -0.06331623f, 0.02865064f, 0.05673740f, 0.12171564f,
        0.03837196f, 0.03555467f, -0.02662914f, -0.10280123f, -0.06526285f,
        -0.11066351f, -0.08988424f, -0.10103678f, 8.10526591e-003f,
        5.95238712e-003f, 0.02617721f, -0.01705742f, -0.10897956f,
        -0.08004991f, -0.11271993f, -0.06185647f, -0.06103712f, 0.01597041f,
        -0.05923606f, 0.09410726f, 0.22858568f, 0.03263380f, 0.06772990f,
        -0.09003516f, 0.01017870f, 0.01931688f, 0.08628357f, -0.01430009f,
        0.10954945f, 0.16612452f, -0.02434544f, -0.03310068f, -0.04236627f,
        0.01212392f, -6.15046406e-003f, 0.06954194f, 0.03015283f, 0.01787957f,
        0.02781667f, -0.05561153f, -8.96244217e-003f, -0.04971489f,
        0.07510284f, 0.01775282f, 0.05889897f, -0.07981427f, 0.03647643f,
        -3.73833324e-003f, -0.08894575f, -0.06429435f, -0.08068276f,
        0.03567704f, -0.07131936f, -7.21910037e-003f, -0.09566668f,
        0.17886090f, 0.14911725f, 0.02070032f, -0.05017120f, -0.04992622f,
        0.01570143f, -0.09906903f, 0.06456193f, 0.15329507f, 0.18820767f,
        0.11689861f, -0.01178513f, -0.02225163f, -0.01905318f, 0.10271224f,
        -7.27029052e-003f, 0.11664233f, 0.14796902f, 0.07771893f, 0.02400013f,
        -0.05361797f, -0.01972888f, 0.01376177f, 0.06740040f, -0.06525395f,
        0.05726178f, -0.02404981f, -0.14018567f, -0.02074987f, -0.04621970f,
        -0.04688627f, -0.01842059f, 0.07722727f, -0.04852883f, 0.01529004f,
        -0.19639495f, 0.10817073f, 0.03795860f, -0.09435206f, -0.07984378f,
        -0.03383440f, 0.11081333f, 0.02237366f, 0.12703256f, 0.21613893f,
        0.02918790f, 4.66472283e-003f, -0.10274266f, -0.04854131f,
        -3.46305710e-003f, 0.08652268f, 0.02251546f, 0.09636052f, 0.17180754f,
        -0.09272388f, 4.59174305e-004f, -0.11723048f, -0.12210111f,
        -0.15547538f, 0.07218186f, -0.05297846f, 0.03779940f, 0.05150875f,
        -0.03802310f, 0.03870645f, -0.15250699f, -0.08696499f, -0.02021560f,
        0.04118926f, -0.15177974f, 0.01577647f, 0.10249301f, 7.50041893e-003f,
        0.01721806f, -0.06828983f, -0.02397596f, -0.06598977f, -0.04317593f,
        -0.08064980f, 6.66632550e-003f, 0.03333484f, 0.07093620f, 0.08231064f,
        -0.06577903f, -0.06698844f, -0.06984019f, -0.06508023f, -0.14145090f,
        -0.02393239f, 0.06485303f, 8.83263443e-003f, 0.09251080f, -0.07557579f,
        -0.05067699f, -0.09798748f, -0.06703258f, -0.14056294f, 0.03245994f,
        0.12554143f, 0.01761621f, 0.12980327f, -0.04081950f, -0.11906909f,
        -0.14813015f, -0.08376863f, -0.12200681f, 0.04988137f, 0.05424247f,
        -3.90952639e-003f, 0.03255733f, -0.12717837f, -0.07461493f,
        -0.05703964f, -0.01736189f, -0.08026433f, -0.05433894f, -0.01719359f,
        0.02886275f, 0.01772653f, -0.09163518f, 3.57789593e-003f, -0.10129993f,
        -0.02653764f, -0.08131415f, -0.03847986f, -7.62157550e-004f,
        0.06486648f, 0.19675669f, -0.04919156f, -0.07059129f, -0.04857785f,
        -0.01042383f, -0.08328653f, 0.03660302f, -0.03696846f, 0.04969259f,
        0.08241162f, -0.12514858f, -0.06122676f, -0.03750202f,
        6.52989605e-003f, -0.10247213f, 0.02568346f, 4.51781414e-003f,
        -0.03734229f, -0.01131264f, -0.05412074f, 8.89345480e-004f,
        -0.12388977f, -0.05959237f, -0.12418608f, -0.06151643f, -0.07310260f,
        0.02441575f, 0.07023528f, -0.07548289f, -7.57147965e-004f,
        -0.09061348f, -0.08112976f, -0.06920306f, 9.54394229e-003f,
        -0.01219902f, 1.21273217e-003f, -8.88989680e-003f, -0.08309301f,
        -0.04552661f, -0.10739882f, -0.05691034f, -0.13928030f, 0.09027749f,
        0.15123098f, 0.03175976f, 0.17763577f, 3.29913251e-004f, 0.05151888f,
        -0.09844074f, -0.09475287f, -0.08571247f, 0.16241577f, 0.19336018f,
        8.57454538e-003f, 0.11474732f, -0.01493934f, 0.03352379f, -0.08966240f,
        -0.02322310f, 0.02663568f, 0.05448750f, -0.03536883f, -0.07210463f,
        -0.06807277f, -0.03121621f, -0.05932408f, -0.17282860f, -0.15873498f,
        -0.04956378f, 0.01603377f, -0.12385946f, 0.13878587f, 0.21468069f,
        0.13510075f, 0.20992437f, 0.08845878f, 0.08104013f, 0.03754176f,
        0.12173114f, 0.11103114f, 0.10643122f, 0.13941477f, 0.11640384f,
        0.14786847f, 0.01218238f, 0.01160753f, 0.03547940f, 0.08794311f,
        -0.01695384f, -0.07692261f, -0.08236158f, 6.79194089e-003f,
        -0.02458403f, 0.13022894f, 0.10953187f, 0.09857773f, 0.04735930f,
        -0.04353498f, -0.15173385f, -0.17904443f, -0.10450364f, -0.13418166f,
        -0.06633098f, -0.03170381f, -0.06839000f, -0.11350126f, -0.06983913f,
        0.19083543f, 0.17604128f, 0.07730632f, 0.10022651f, 0.36428109f,
        0.28291923f, 0.12688625f, 0.15942036f, 0.14064661f, -0.11201853f,
        -0.13969108f, -0.09088077f, -0.14107047f, 0.05117374f,
        -2.63348082e-003f, -0.10794610f, -0.09715455f, -0.05284977f,
        0.01565668f, 0.05031200f, 0.07021113f, -0.02963028f, 0.01766960f,
        0.08333644f, -0.03211382f, 4.90096770e-003f, 0.05186674f, -0.05045737f,
        -0.09624767f, -0.02525997f, 0.06916669f, 0.01213916f, 0.05333899f,
        -0.03443280f, -0.10055527f, -0.06291115f, 5.42851724e-003f,
        -6.30360236e-003f, 0.02270257f, -0.01769792f, 0.03273688f, 0.07746078f,
        7.77099328e-003f, 0.05041346f, 0.01648103f, -0.02321534f, -0.09930186f,
        -0.02293853f, 0.02034990f, -0.08324204f, 0.08510064f, -0.03732836f,
        -0.06465405f, -0.06086946f, 0.13680504f, -0.11469388f, -0.03896406f,
        -0.07142810f, 2.67581246e-003f, -0.03639632f, -0.09849060f,
        -0.11014334f, 0.17489147f, 0.17610909f, -0.16091567f, -0.07248894f,
        0.01567141f, 0.23742996f, 0.07552249f, -0.06270349f, -0.07303379f,
        0.25442186f, 0.16903116f, -0.08168741f, -0.05913896f, -0.03954096f,
        6.81776879e-003f, -0.05615319f, -0.07303037f, -0.12176382f,
        0.12385108f, 0.22084464f, -0.05543206f, -0.03310431f, 0.05731593f,
        0.19481890f, 0.04016430f, -0.06480758f, -0.12353460f, 0.18733442f,
        -0.09631214f, -0.11192076f, 0.12404587f, 0.15671748f, 0.19256128f,
        0.10895617f, 0.03391477f, -0.13032004f, -0.05626907f, -0.09025607f,
        0.23485197f, 0.27812332f, 0.26725492f, 0.07255980f, 0.16565137f,
        0.22388470f, 0.07441066f, -0.21003133f, -0.08075339f, -0.15031935f,
        0.07023834f, 0.10872041f, 0.18156518f, 0.20037253f, 0.13571967f,
        -0.11915682f, -0.11131983f, -0.18878011f, 0.06074620f, 0.20578890f,
        0.12413109f, 0.03930207f, 0.29176015f, 0.29502738f, 0.27856228f,
        -0.01803601f, 0.16646385f, 0.19268319f, 0.01900682f, 0.06026287f,
        2.35868432e-003f, 0.01558199f, 0.02707230f, 0.11383014f, 0.12103992f,
        0.03907350f, 0.04637353f, 0.09020995f, 0.11919726f, -3.63007211e-003f,
        0.02220155f, 0.10336831f, 0.17351882f, 0.12259731f, 0.18983354f,
        0.15736865f, 0.01160725f, -0.01690723f, -9.69582412e-004f, 0.07213813f,
        0.01161613f, 0.17864859f, 0.24486147f, 0.18208991f, 0.20177495f,
        0.05972528f, -8.93934630e-003f, -0.02316955f, 0.14436610f, 0.14114498f,
        0.05520950f, 0.06353590f, -0.19124921f, 0.10174713f, 0.29414919f,
        0.26448128f, 0.09344960f, 0.15284036f, 0.19797507f, 0.11369792f,
        -0.12722753f, -0.21396367f, -0.02008235f, -0.06566695f, -0.01662150f,
        -0.03937003f, 0.04778343f, 0.05017274f, -0.02299062f, -0.20208496f,
        -0.06395898f, 0.13721776f, 0.22544557f, 0.14888357f, 0.08687132f,
        0.27088094f, 0.32206613f, 0.09782200f, -0.18523243f, -0.17232181f,
        -0.01041531f, 0.04008654f, 0.04199702f, -0.08081299f, -0.03755421f,
        -0.04809646f, -0.05222081f, -0.21709201f, -0.06622940f, 0.02945281f,
        -0.04600435f, -0.05256077f, -0.08432942f, 0.02848100f, 0.03490564f,
        8.28621630e-003f, -0.11051246f, -0.11210597f, -0.01998289f,
        -0.05369405f, -0.08869293f, -0.18799506f, -0.05436598f, -0.05011634f,
        -0.05419716f, -0.06151857f, -0.10827805f, 0.04346
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

- **PixData**: A class/struct defined in this file
- **HOGConfInvoker**: A class/struct defined in this file
- **HOGInvoker**: A class/struct defined in this file
- **BlockData**: A class/struct defined in this file
- **HOGCache**: A class/struct defined in this file

### Functions and Methods

- **call()**: A function/method defined in this file
- **renurn()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **const()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/hal/intrin.hpp`
- `cascadedetect.hpp`
- `opencl_kernels_objdetect.hpp`
- `iterator`
- `limits`
- `precomp.hpp`
- `cstdio`

**Python Imports:**
- `daimler`
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

