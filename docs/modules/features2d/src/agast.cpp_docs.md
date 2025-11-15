# Documentation for `modules/features2d/src/agast.cpp`

## File Metadata

- **Full Path**: `modules/features2d/src/agast.cpp`
- **File Name**: `agast.cpp`
- **File Size**: 391,733 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/features2d/src/agast.cpp](../../../modules/features2d/src/agast.cpp)

## Purpose and Role

This file is located in the `modules/features2d/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* This is AGAST and OAST, an optimal and accelerated corner detector
              based on the accelerated segment tests
   Below is the original copyright and the references */

/*
Copyright (C) 2010  Elmar Mair
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:

    *Redistributions of source code must retain the above copyright
     notice, this list of conditions and the following disclaimer.

    *Redistributions in binary form must reproduce the above copyright
     notice, this list of conditions and the following disclaimer in the
     documentation and/or other materials provided with the distribution.

    *Neither the name of the University of Cambridge nor the names of
     its contributors may be used to endorse or promote products derived
     from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
*/

/*
The references are:
 * Adaptive and Generic Corner Detection Based on the Accelerated Segment Test,
   Elmar Mair and Gregory D. Hager and Darius Burschka
   and Michael Suppa and Gerhard Hirzinger ECCV 2010
   URL: http://www6.in.tum.de/Main/ResearchAgast
*/

#include "precomp.hpp"
#include "agast_score.hpp"

namespace cv
{

#if (defined __i386__ || defined(_M_IX86) || defined __x86_64__ || defined(_M_X64) || defined(_M_ARM64) || defined(__aarch64__) || defined(__arm__))

static void AGAST_5_8(InputArray _img, std::vector<KeyPoint>& keypoints, int threshold)
{

    cv::Mat img;
    if(!_img.getMat().isContinuous())
      img = _img.getMat().clone();
    else
      img = _img.getMat();

    size_t total = 0;
    int xsize = img.cols;
    int ysize = img.rows;
    size_t nExpectedCorners = keypoints.capacity();
    int x, y;
    int xsizeB = xsize - 2;
    int ysizeB = ysize - 1;
    int width;

    keypoints.resize(0);

    int pixel_5_8_[16];
    makeAgastOffsets(pixel_5_8_, (int)img.step, AgastFeatureDetector::AGAST_5_8);

    short offset0 = (short) pixel_5_8_[0];
    short offset1 = (short) pixel_5_8_[1];
    short offset2 = (short) pixel_5_8_[2];
    short offset3 = (short) pixel_5_8_[3];
    short offset4 = (short) pixel_5_8_[4];
    short offset5 = (short) pixel_5_8_[5];
    short offset6 = (short) pixel_5_8_[6];
    short offset7 = (short) pixel_5_8_[7];

    width = xsize;

    for(y = 1; y < ysizeB; y++)
    {
        x = 0;
        while(true)
        {
          homogeneous:
          {
            x++;
            if(x > xsizeB)
                break;
            else
            {
                const unsigned char* const ptr = img.ptr() + y*width + x;
                const int cb = *ptr + threshold;
                const int c_b = *ptr - threshold;
                if(ptr[offset0] > cb)
                  if(ptr[offset2] > cb)
                    if(ptr[offset3] > cb)
                      if(ptr[offset5] > cb)
                        if(ptr[offset1] > cb)
                          if(ptr[offset4] > cb)
                            goto success_structured;
                          else
                            if(ptr[offset7] > cb)
                              goto success_structured;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset4] > cb)
                            if(ptr[offset6] > cb)
                              goto success_structured;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        if(ptr[offset1] > cb)
                          if(ptr[offset4] > cb)
                            goto success_homogeneous;
                          else
                            if(ptr[offset7] > cb)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                        else
                          goto homogeneous;
                    else
                      if(ptr[offset7] > cb)
                        if(ptr[offset6] > cb)
                          if(ptr[offset5] > cb)
                            if(ptr[offset1] > cb)
                              goto success_structured;
                            else
                              if(ptr[offset4] > cb)
                                goto success_structured;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset1] > cb)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        if(ptr[offset5] < c_b)
                          if(ptr[offset3] < c_b)
                            if(ptr[offset7] < c_b)
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_structured;
                                else
                                  goto structured;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                  else
                    if(ptr[offset5] > cb)
                      if(ptr[offset7] > cb)
                        if(ptr[offset6] > cb)
                          if(ptr[offset1] > cb)
                            goto success_homogeneous;
                          else
                            if(ptr[offset4] > cb)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        goto homogeneous;
                    else
                      if(ptr[offset5] < c_b)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset2] < c_b)
                            if(ptr[offset1] < c_b)
                              if(ptr[offset4] < c_b)
                                goto success_structured;
                              else
                                goto homogeneous;
                            else
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_structured;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset7] < c_b)
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_structured;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        goto homogeneous;
                else
                if(ptr[offset0] < c_b)
                  if(ptr[offset2] < c_b)
                    if(ptr[offset7] > cb)
                      if(ptr[offset3] < c_b)
                        if(ptr[offset5] < c_b)
                          if(ptr[offset1] < c_b)
                            if(ptr[offset4] < c_b)
                              goto success_structured;
                            else
                              goto structured;
                          else
                            if(ptr[offset4] < c_b)
                              if(ptr[offset6] < c_b)
                                goto success_structured;
                              else
                                goto structured;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset1] < c_b)
                            if(ptr[offset4] < c_b)
                              goto success_structured;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        if(ptr[offset5] > cb)
                          if(ptr[offset3] > cb)
                            if(ptr[offset4] > cb)
                              if(ptr[offset6] > cb)
                                goto success_structured;
                              else
                                goto structured;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                    else
                      if(ptr[offset7] < c_b)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset1] < c_b)
                              goto success_structured;
                            else
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_structured;
                                else
                                  goto structured;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset1] < c_b)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset6] < c_b)
                            if(ptr[offset5] < c_b)
                              if(ptr[offset1] < c_b)
                                goto success_structured;
                              else
                                if(ptr[offset4] < c_b)
                                  goto success_structured;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset1] < c_b)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        if(ptr[offset3] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset1] < c_b)
                              if(ptr[offset4] < c_b)
                                goto success_structured;
                              else
                                goto homogeneous;
                            else
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_structured;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset1] < c_b)
                              if(ptr[offset4] < c_b)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          goto homogeneous;
                  else
                    if(ptr[offset5] > cb)
                      if(ptr[offset3] > cb)
                        if(ptr[offset2] > cb)
                          if(ptr[offset1] > cb)
                            if(ptr[offset4] > cb)
                              goto success_structured;
                            else
                              goto homogeneous;
                          else
                            if(ptr[offset4] > cb)
                              if(ptr[offset6] > cb)
                                goto success_structured;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset7] > cb)
                            if(ptr[offset4] > cb)
                              if(ptr[offset6] > cb)
                                goto success_structured;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        goto homogeneous;
                    else
                      if(ptr[offset5] < c_b)
                        if(ptr[offset7] < c_b)
                          if(ptr[offset6] < c_b)
                            if(ptr[offset1] < c_b)
                              goto success_homogeneous;
                            else
                              if(ptr[offset4] < c_b)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        goto homogeneous;
                else
                  if(ptr[offset3] > cb)
                    if(ptr[offset5] > cb)
                      if(ptr[offset2] > cb)
                        if(ptr[offset1] > cb)
                          if(ptr[offset4] > cb)
                            goto success_homogeneous;
                          else
                            goto homogeneous;
                        else
                          if(ptr[offset4] > cb)
                            if(ptr[offset6] > cb)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        if(ptr[offset7] > cb)
                          if(ptr[offset4] > cb)
                            if(ptr[offset6] > cb)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                    else
                      goto homogeneous;
                  else
                    if(ptr[offset3] < c_b)
                      if(ptr[offset5] < c_b)
                        if(ptr[offset2] < c_b)
                          if(ptr[offset1] < c_b)
                            if(ptr[offset4] < c_b)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                          else
                            if(ptr[offset4] < c_b)
                              if(ptr[offset6] < c_b)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset7] < c_b)
                            if(ptr[offset4] < c_b)
                              if(ptr[offset6] < c_b)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        goto homogeneous;
                    else
                      goto homogeneous;
            }
          }
          structured:
          {
            x++;
            if(x > xsizeB)
                break;
            else
            {
                const unsigned char* const ptr = img.ptr() + y*width + x;
                const int cb = *ptr + threshold;
                const int c_b = *ptr - threshold;
                if(ptr[offset0] > cb)
                  if(ptr[offset2] > cb)
                    if(ptr[offset3] > cb)
                      if(ptr[offset5] > cb)
                        if(ptr[offset7] > cb)
                          if(ptr[offset1] > cb)
                            goto success_structured;
                          else
                            if(ptr[offset4] > cb)
                              if(ptr[offset6] > cb)
                                goto success_structured;
                              else
                                goto structured;
                            else
                              goto structured;
                        else
                          if(ptr[offset1] > cb)
                            if(ptr[offset4] > cb)
                              goto success_structured;
                            else
                              goto structured;
                          else
                            if(ptr[offset4] > cb)
                              if(ptr[offset6] > cb)
                                goto success_structured;
                              else
                                goto structured;
                            else
                              goto structured;
                      else
                        if(ptr[offset7] > cb)
                          if(ptr[offset1] > cb)
                            goto success_structured;
                          else
                            goto structured;
                        else
                          if(ptr[offset1] > cb)
                            if(ptr[offset4] > cb)
                              goto success_structured;
                            else
                              goto structured;
                          else
                            goto structured;
                    else
                      if(ptr[offset7] > cb)
                        if(ptr[offset6] > cb)
                          if(ptr[offset5] > cb)
                            if(ptr[offset1] > cb)
                              goto success_structured;
                            else
                              if(ptr[offset4] > cb)
                                goto success_structured;
                              else
                                goto structured;
                          else
                            if(ptr[offset1] > cb)
                              goto success_structured;
                            else
                              goto structured;
                        else
                          goto structured;
                      else
                        if(ptr[offset5] < c_b)
                          if(ptr[offset3] < c_b)
                            if(ptr[offset7] < c_b)
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_structured;
                                else
                                  goto structured;
                              else
                                goto structured;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto structured;
                  else
                    if(ptr[offset5] > cb)
                      if(ptr[offset7] > cb)
                        if(ptr[offset6] > cb)
                          if(ptr[offset1] > cb)
                            goto success_structured;
                          else
                            if(ptr[offset4] > cb)
                              goto success_structured;
                            else
                              goto structured;
                        else
                          goto structured;
                      else
                        goto structured;
                    else
                      if(ptr[offset5] < c_b)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset2] < c_b)
                            if(ptr[offset1] < c_b)
                              if(ptr[offset4] < c_b)
                                goto success_structured;
                              else
                                goto structured;
                            else
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_structured;
                                else
                                  goto structured;
                              else
                                goto structured;
                          else
                            if(ptr[offset7] < c_b)
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          goto structured;
                      else
                        goto homogeneous;
                else
                if(ptr[offset0] < c_b)
                  if(ptr[offset2] < c_b)
                    if(ptr[offset7] > cb)
                      if(ptr[offset3] < c_b)
                        if(ptr[offset5] < c_b)
                          if(ptr[offset1] < c_b)
                            if(ptr[offset4] < c_b)
                              goto success_structured;
                            else
                              goto structured;
                          else
                            if(ptr[offset4] < c_b)
                              if(ptr[offset6] < c_b)
                                goto success_structured;
                              else
                                goto structured;
                            else
                              goto structured;
                        else
                          if(ptr[offset1] < c_b)
                            if(ptr[offset4] < c_b)
                              goto success_structured;
                            else
                              goto structured;
                          else
                            goto structured;
                      else
                        if(ptr[offset5] > cb)
                          if(ptr[offset3] > cb)
                            if(ptr[offset4] > cb)
                              if(ptr[offset6] > cb)
                                goto success_structured;
                              else
                                goto structured;
                            else
                              goto structured;
                          else
                            goto homogeneous;
                        else
                          goto structured;
                    else
                      if(ptr[offset7] < c_b)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset1] < c_b)
                              goto success_structured;
                            else
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_structured;
                                else
                                  goto structured;
                              else
                                goto structured;
                          else
                            if(ptr[offset1] < c_b)
                              goto success_structured;
                            else
                              goto structured;
                        else
                          if(ptr[offset6] < c_b)
                            if(ptr[offset5] < c_b)
                              if(ptr[offset1] < c_b)
                                goto success_structured;
                              else
                                if(ptr[offset4] < c_b)
                                  goto success_structured;
                                else
                                  goto structured;
                            else
                              if(ptr[offset1] < c_b)
                                goto success_structured;
                              else
                                goto structured;
                          else
                            goto structured;
                      else
                        if(ptr[offset3] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset1] < c_b)
                              if(ptr[offset4] < c_b)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                            else
                              if(ptr[offset4] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto success_homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset1] < c_b)
                              if(ptr[offset4] < c_b)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          goto homogeneous;
                  else
                    if(ptr[offset5] > cb)
                      if(ptr[offset3] > cb)
                        if(ptr[offset2] > cb)
                          if(ptr[offset1] > cb)
                            if(ptr[offset4] > cb)
                              goto success_structured;
                            else
                              goto structured;
                          else
                            if(ptr[offset4] > cb)
                              if(ptr[offset6] > cb)
                                goto success_structured;
                              else
                                goto structured;
                            else
                              goto structured;
                        else
                          if(ptr[offset7] > cb)
                            if(ptr[offset4] > cb)
                              if(ptr[offset6] > cb)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        goto structured;
                    else
                      if(ptr[offset5] < c_b)
                        if(ptr[offset7] < c_b)
                          if(ptr[offset6] < c_b)
                            if(ptr[offset1] < c_b)
                              goto success_structured;
                            else
                              if(ptr[offset4] < c_b)
                                goto success_structured;
                              else
                                goto structured;
                          else
                            goto structured;
                        else
                          goto structured;
                      else
                        goto homogeneous;
                else
                  if(ptr[offset3] > cb)
                    if(ptr[offset5] > cb)
                      if(ptr[offset2] > cb)
                        if(ptr[offset1] > cb)
                          if(ptr[offset4] > cb)
                            goto success_homogeneous;
                          else
                            goto homogeneous;
                        else
                          if(ptr[offset4] > cb)
                            if(ptr[offset6] > cb)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        if(ptr[offset7] > cb)
                          if(ptr[offset4] > cb)
                            if(ptr[offset6] > cb)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                    else
                      goto homogeneous;
                  else
                    if(ptr[offset3] < c_b)
                      if(ptr[offset5] < c_b)
                        if(ptr[offset2] < c_b)
                          if(ptr[offset1] < c_b)
                            if(ptr[offset4] < c_b)
                              goto success_homogeneous;
                            else
                              goto homogeneous;
                          else
                            if(ptr[offset4] < c_b)
                              if(ptr[offset6] < c_b)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset7] < c_b)
                            if(ptr[offset4] < c_b)
                              if(ptr[offset6] < c_b)
                                goto success_homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        goto homogeneous;
                    else
                      goto homogeneous;
            }
          }
            success_homogeneous:
            if(total == nExpectedCorners)
            {
                if(nExpectedCorners == 0)
                {
                    nExpectedCorners = 512;
                    keypoints.reserve(nExpectedCorners);
                }
                else
                {
                    nExpectedCorners *= 2;
                    keypoints.reserve(nExpectedCorners);
                }
            }
            keypoints.push_back(KeyPoint(Point2f((float)x, (float)y), 7.0f));
            total++;
            goto homogeneous;
            success_structured:
            if(total == nExpectedCorners)
            {
                if(nExpectedCorners == 0)
                {
                    nExpectedCorners = 512;
                    keypoints.reserve(nExpectedCorners);
                }
                else
                {
                    nExpectedCorners *= 2;
                    keypoints.reserve(nExpectedCorners);
                }
            }
            keypoints.push_back(KeyPoint(Point2f((float)x, (float)y), 7.0f));
            total++;
            goto structured;
        }
    }
}

static void AGAST_7_12d(InputArray _img, std::vector<KeyPoint>& keypoints, int threshold)
{
    cv::Mat img;
    if(!_img.getMat().isContinuous())
      img = _img.getMat().clone();
    else
      img = _img.getMat();

    size_t total = 0;
    int xsize = img.cols;
    int ysize = img.rows;
    size_t nExpectedCorners = keypoints.capacity();
    int x, y;
    int xsizeB = xsize - 4;
    int ysizeB = ysize - 3;
    int width;

    keypoints.resize(0);

    int pixel_7_12d_[16];
    makeAgastOffsets(pixel_7_12d_, (int)img.step, AgastFeatureDetector::AGAST_7_12d);

    short offset0 = (short) pixel_7_12d_[0];
    short offset1 = (short) pixel_7_12d_[1];
    short offset2 = (short) pixel_7_12d_[2];
    short offset3 = (short) pixel_7_12d_[3];
    short offset4 = (short) pixel_7_12d_[4];
    short offset5 = (short) pixel_7_12d_[5];
    short offset6 = (short) pixel_7_12d_[6];
    short offset7 = (short) pixel_7_12d_[7];
    short offset8 = (short) pixel_7_12d_[8];
    short offset9 = (short) pixel_7_12d_[9];
    short offset10 = (short) pixel_7_12d_[10];
    short offset11 = (short) pixel_7_12d_[11];

    width = xsize;

    for(y = 3; y < ysizeB; y++)
    {
        x = 2;
        while(true)
        {
          homogeneous:
          {
            x++;
            if(x > xsizeB)
                break;
            else
            {
                const unsigned char* const ptr = img.ptr() + y*width + x;
                const int cb = *ptr + threshold;
                const int c_b = *ptr - threshold;
                if(ptr[offset0] > cb)
                  if(ptr[offset5] > cb)
                    if(ptr[offset2] > cb)
                      if(ptr[offset9] > cb)
                        if(ptr[offset1] > cb)
                          if(ptr[offset6] > cb)
                            if(ptr[offset3] > cb)
                              if(ptr[offset4] > cb)
                                goto success_homogeneous;
                              else
                                if(ptr[offset10] > cb)
                                  if(ptr[offset11] > cb)
                                    goto success_structured;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset8] > cb)
                                if(ptr[offset10] > cb)
                                  if(ptr[offset11] > cb)
                                    goto success_structured;
                                  else
                                    if(ptr[offset4] > cb)
                                      if(ptr[offset7] > cb)
                                        goto success_structured;
                                      else
                                        goto structured;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset11] > cb)
                              if(ptr[offset3] > cb)
                                if(ptr[offset4] > cb)
                                  goto success_homogeneous;
                                else
                                  if(ptr[offset10] > cb)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset8] > cb)
                                  if(ptr[offset10] > cb)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset6] > cb)
                            if(ptr[offset7] > cb)
                              if(ptr[offset8] > cb)
                                if(ptr[offset4] > cb)
                                  if(ptr[offset3] > cb)
                                    goto success_structured;
                                  else
                                    if(ptr[offset10] > cb)
                                      goto success_structured;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset10] > cb)
                                    if(ptr[offset11] > cb)
                                      goto success_structured;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        if(ptr[offset3] > cb)
                          if(ptr[offset4] > cb)
                            if(ptr[offset1] > cb)
                              if(ptr[offset6] > cb)
                                goto success_homogeneous;
                              else
                                if(ptr[offset11] > cb)
                                  goto success_homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset6] > cb)
                                if(ptr[offset7] > cb)
                                  if(ptr[offset8] > cb)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                    else
                      if(ptr[offset9] > cb)
                        if(ptr[offset7] > cb)
                          if(ptr[offset8] > cb)
                            if(ptr[offset1] > cb)
                              if(ptr[offset10] > cb)
                                if(ptr[offset11] > cb)
                                  goto success_homogeneous;
                                else
                                  if(ptr[offset6] > cb)
                                    if(ptr[offset4] > cb)
                                      goto success_structured;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset6] > cb)
                                  if(ptr[offset3] > cb)
                                    if(ptr[offset4] > cb)
                                      goto success_structured;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset6] > cb)
                                if(ptr[offset4] > cb)
                                  if(ptr[offset3] > cb)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset10] > cb)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset10] > cb)
                                    if(ptr[offset11] > cb)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        goto homogeneous;
                  else
                    if(ptr[offset5] < c_b)
                      if(ptr[offset9] > cb)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset4] < c_b)
                            if(ptr[offset11] > cb)
                              if(ptr[offset1] > cb)
                                if(ptr[offset8] > cb)
                                  if(ptr[offset10] > cb)
                                    if(ptr[offset2] > cb)
                                      goto success_structured;
                                    else
                                      if(ptr[offset7] > cb)
                                        goto success_structured;
                                      else
                                        goto structured;
                                  else
                                    goto homogeneous;
                                else
                                  if(ptr[offset6] < c_b)
                                    if(ptr[offset2] < c_b)
                                      if(ptr[offset7] < c_b)
                                        if(ptr[offset8] < c_b)
                                          goto success_structured;
                                        else
                                          goto structured;
                                      else
                                        goto structured;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset6] > cb)
                                  if(ptr[offset7] > cb)
                                    if(ptr[offset8] > cb)
                                      if(ptr[offset10] > cb)
                                        goto success_structured;
                                      else
                                        goto structured;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  if(ptr[offset6] < c_b)
                                    if(ptr[offset2] < c_b)
                                      if(ptr[offset7] < c_b)
                                        if(ptr[offset1] < c_b)
                                          goto success_structured;
                                        else
                                          if(ptr[offset8] < c_b)
                                            goto success_structured;
                                          else
                                            goto structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                            else
                              if(ptr[offset2] < c_b)
                                if(ptr[offset7] < c_b)
                                  if(ptr[offset1] < c_b)
                                    if(ptr[offset6] < c_b)
                                      goto success_structured;
                                    else
                                      goto homogeneous;
                                  else
                                    if(ptr[offset6] < c_b)
                                      if(ptr[offset8] < c_b)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset11] > cb)
                              if(ptr[offset8] > cb)
                                if(ptr[offset10] > cb)
                                  if(ptr[offset1] > cb)
                                    if(ptr[offset2] > cb)
                                      goto success_structured;
                                    else
                                      if(ptr[offset7] > cb)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                  else
                                    if(ptr[offset6] > cb)
                                      if(ptr[offset7] > cb)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset11] > cb)
                            if(ptr[offset10] > cb)
                              if(ptr[offset3] > cb)
                                if(ptr[offset1] > cb)
                                  if(ptr[offset2] > cb)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset7] > cb)
                                      if(ptr[offset8] > cb)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset6] > cb)
                                    if(ptr[offset7] > cb)
                                      if(ptr[offset8] > cb)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset8] > cb)
                                  if(ptr[offset1] > cb)
                                    if(ptr[offset2] > cb)
                                      goto success_homogeneous;
                                    else
                                      if(ptr[offset7] > cb)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                  else
                                    if(ptr[offset6] > cb)
                                      if(ptr[offset7] > cb)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        if(ptr[offset9] < c_b)
                          if(ptr[offset2] > cb)
                            if(ptr[offset1] > cb)
                              if(ptr[offset4] > cb)
                                if(ptr[offset10] > cb)
                                  if(ptr[offset3] > cb)
                                    if(ptr[offset11] > cb)
                                      goto success_structured;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  if(ptr[offset6] < c_b)
                                    if(ptr[offset7] < c_b)
                                      if(ptr[offset8] < c_b)
                                        if(ptr[offset11] < c_b)
                                          if(ptr[offset10] < c_b)
                                            goto success_structured;
                                          else
                                            goto structured;
                                        else
                                          goto structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset6] < c_b)
                                  if(ptr[offset7] < c_b)
                                    if(ptr[offset8] < c_b)
                                      if(ptr[offset10] < c_b)
                                        if(ptr[offset4] < c_b)
                                          goto success_structured;
                                        else
                                          if(ptr[offset11] < c_b)
                                            goto success_structured;
                                          else
                                            goto structured;
                                      else
                                        if(ptr[offset3] < c_b)
                                          if(ptr[offset4] < c_b)
                                            goto success_structured;
                                          else
                                            goto structured;
                                        else
                                          goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset6] < c_b)
                                if(ptr[offset7] < c_b)
                                  if(ptr[offset8] < c_b)
                                    if(ptr[offset4] < c_b)
                                      if(ptr[offset3] < c_b)
                                        goto success_structured;
                                      else
                                        if(ptr[offset10] < c_b)
                                          goto success_structured;
                                        else
                                          goto homogeneous;
                                    else
                                      if(ptr[offset10] < c_b)
                                        if(ptr[offset11] < c_b)
                                          goto success_structured;
                                        else
                                          goto homogeneous;
                                      else
                                        goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset6] < c_b)
                              if(ptr[offset7] < c_b)
                                if(ptr[offset8] < c_b)
                                  if(ptr[offset4] < c_b)
                                    if(ptr[offset3] < c_b)
                                      goto success_homogeneous;
                                    else
                                      if(ptr[offset10] < c_b)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                  else
                                    if(ptr[offset10] < c_b)
                                      if(ptr[offset11] < c_b)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset2] < c_b)
                                    if(ptr[offset1] < c_b)
                                      if(ptr[offset3] < c_b)
                                        if(ptr[offset4] < c_b)
                                          goto success_structured;
                                        else
                                          goto homogeneous;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset2] > cb)
                            if(ptr[offset1] > cb)
                              if(ptr[offset3] > cb)
                                if(ptr[offset4] > cb)
                                  if(ptr[offset10] > cb)
                                    if(ptr[offset11] > cb)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            if(ptr[offset2] < c_b)
                              if(ptr[offset3] < c_b)
                                if(ptr[offset4] < c_b)
                                  if(ptr[offset7] < c_b)
                                    if(ptr[offset1] < c_b)
                                      if(ptr[offset6] < c_b)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                    else
                                      if(ptr[offset6] < c_b)
                                        if(ptr[offset8] < c_b)
                                          goto success_homogeneous;
                                        else
                                          goto homogeneous;
                                      else
                                        goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                    else
                      if(ptr[offset2] > cb)
                        if(ptr[offset10] > cb)
                          if(ptr[offset11] > cb)
                            if(ptr[offset9] > cb)
                              if(ptr[offset1] > cb)
                                if(ptr[offset3] > cb)
                                  goto success_homogeneous;
                                else
                                  if(ptr[offset8] > cb)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset6] > cb)
                                  if(ptr[offset7] > cb)
                                    if(ptr[offset8] > cb)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset1] > cb)
                                if(ptr[offset3] > cb)
                                  if(ptr[offset4] > cb)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        if(ptr[offset9] > cb)
                          if(ptr[offset7] > cb)
                            if(ptr[offset8] > cb)
                              if(ptr[offset10] > cb)
                                if(ptr[offset11] > cb)
                                  if(ptr[offset1] > cb)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset6] > cb)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                else if(ptr[offset0] < c_b)
                  if(ptr[offset2] > cb)
                    if(ptr[offset5] > cb)
                      if(ptr[offset7] > cb)
                        if(ptr[offset6] > cb)
                          if(ptr[offset4] > cb)
                            if(ptr[offset3] > cb)
                              if(ptr[offset1] > cb)
                                goto success_homogeneous;
                              else
                                if(ptr[offset8] > cb)
                                  goto success_homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset9] > cb)
                                if(ptr[offset8] > cb)
                                  if(ptr[offset10] > cb)
                                    goto success_structured;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset9] > cb)
                              if(ptr[offset8] > cb)
                                if(ptr[offset10] > cb)
                                  if(ptr[offset11] > cb)
                                    goto success_structured;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        if(ptr[offset9] < c_b)
                          if(ptr[offset8] < c_b)
                            if(ptr[offset10] < c_b)
                              if(ptr[offset11] < c_b)
                                if(ptr[offset7] < c_b)
                                  if(ptr[offset1] < c_b)
                                    goto success_structured;
                                  else
                                    if(ptr[offset6] < c_b)
                                      goto success_structured;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                    else
                      if(ptr[offset9] < c_b)
                        if(ptr[offset7] < c_b)
                          if(ptr[offset8] < c_b)
                            if(ptr[offset5] < c_b)
                              if(ptr[offset1] < c_b)
                                if(ptr[offset10] < c_b)
                                  if(ptr[offset11] < c_b)
                                    goto success_structured;
                                  else
                                    if(ptr[offset6] < c_b)
                                      if(ptr[offset4] < c_b)
                                        goto success_structured;
                                      else
                                        goto structured;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset6] < c_b)
                                    if(ptr[offset3] < c_b)
                                      if(ptr[offset4] < c_b)
                                        goto success_structured;
                                      else
                                        goto structured;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset6] < c_b)
                                  if(ptr[offset4] < c_b)
                                    if(ptr[offset3] < c_b)
                                      goto success_structured;
                                    else
                                      if(ptr[offset10] < c_b)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                  else
                                    if(ptr[offset10] < c_b)
                                      if(ptr[offset11] < c_b)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset10] < c_b)
                                if(ptr[offset11] < c_b)
                                  if(ptr[offset1] < c_b)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset6] < c_b)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        goto homogeneous;
                  else
                    if(ptr[offset2] < c_b)
                      if(ptr[offset9] > cb)
                        if(ptr[offset5] > cb)
                          if(ptr[offset1] < c_b)
                            if(ptr[offset4] < c_b)
                              if(ptr[offset10] < c_b)
                                if(ptr[offset3] < c_b)
                                  if(ptr[offset11] < c_b)
                                    goto success_structured;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                if(ptr[offset6] > cb)
                                  if(ptr[offset7] > cb)
                                    if(ptr[offset8] > cb)
                                      if(ptr[offset11] > cb)
                                        if(ptr[offset10] > cb)
                                          goto success_structured;
                                        else
                                          goto structured;
                                      else
                                        goto structured;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset6] > cb)
                                if(ptr[offset7] > cb)
                                  if(ptr[offset8] > cb)
                                    if(ptr[offset10] > cb)
                                      if(ptr[offset4] > cb)
                                        goto success_structured;
                                      else
                                        if(ptr[offset11] > cb)
                                          goto success_structured;
                                        else
                                          goto structured;
                                    else
                                      if(ptr[offset3] > cb)
                                        if(ptr[offset4] > cb)
                                          goto success_structured;
                                        else
                                          goto structured;
                                      else
                                        goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                if(ptr[offset8] > cb)
                                  if(ptr[offset4] > cb)
                                    if(ptr[offset3] > cb)
                                      goto success_structured;
                                    else
                                      if(ptr[offset10] > cb)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                  else
                                    if(ptr[offset10] > cb)
                                      if(ptr[offset11] > cb)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset3] < c_b)
                            if(ptr[offset4] < c_b)
                              if(ptr[offset5] < c_b)
                                if(ptr[offset1] < c_b)
                                  if(ptr[offset6] < c_b)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset11] < c_b)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset6] < c_b)
                                    if(ptr[offset7] < c_b)
                                      if(ptr[offset8] < c_b)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset1] < c_b)
                                  if(ptr[offset10] < c_b)
                                    if(ptr[offset11] < c_b)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                      else
                        if(ptr[offset9] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset1] < c_b)
                              if(ptr[offset6] < c_b)
                                if(ptr[offset3] < c_b)
                                  if(ptr[offset4] < c_b)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset10] < c_b)
                                      if(ptr[offset11] < c_b)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset8] < c_b)
                                    if(ptr[offset10] < c_b)
                                      if(ptr[offset11] < c_b)
                                        goto success_structured;
                                      else
                                        if(ptr[offset4] < c_b)
                                          if(ptr[offset7] < c_b)
                                            goto success_structured;
                                          else
                                            goto structured;
                                        else
                                          goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset11] < c_b)
                                  if(ptr[offset3] < c_b)
                                    if(ptr[offset4] < c_b)
                                      goto success_homogeneous;
                                    else
                                      if(ptr[offset10] < c_b)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                  else
                                    if(ptr[offset8] < c_b)
                                      if(ptr[offset10] < c_b)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset6] < c_b)
                                if(ptr[offset7] < c_b)
                                  if(ptr[offset8] < c_b)
                                    if(ptr[offset4] < c_b)
                                      if(ptr[offset3] < c_b)
                                        goto success_structured;
                                      else
                                        if(ptr[offset10] < c_b)
                                          goto success_structured;
                                        else
                                          goto homogeneous;
                                    else
                                      if(ptr[offset10] < c_b)
                                        if(ptr[offset11] < c_b)
                                          goto success_structured;
                                        else
                                          goto homogeneous;
                                      else
                                        goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset10] < c_b)
                              if(ptr[offset11] < c_b)
                                if(ptr[offset1] < c_b)
                                  if(ptr[offset3] < c_b)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset8] < c_b)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset6] < c_b)
                                    if(ptr[offset7] < c_b)
                                      if(ptr[offset8] < c_b)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          if(ptr[offset3] < c_b)
                            if(ptr[offset4] < c_b)
                              if(ptr[offset5] < c_b)
                                if(ptr[offset1] < c_b)
                                  if(ptr[offset6] < c_b)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset11] < c_b)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset6] < c_b)
                                    if(ptr[offset7] < c_b)
                                      if(ptr[offset8] < c_b)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset1] < c_b)
                                  if(ptr[offset10] < c_b)
                                    if(ptr[offset11] < c_b)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                    else
                      if(ptr[offset9] < c_b)
                        if(ptr[offset7] < c_b)
                          if(ptr[offset8] < c_b)
                            if(ptr[offset5] < c_b)
                              if(ptr[offset1] < c_b)
                                if(ptr[offset10] < c_b)
                                  if(ptr[offset11] < c_b)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset6] < c_b)
                                      if(ptr[offset4] < c_b)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  if(ptr[offset6] < c_b)
                                    if(ptr[offset3] < c_b)
                                      if(ptr[offset4] < c_b)
                                        goto success_structured;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset6] < c_b)
                                  if(ptr[offset4] < c_b)
                                    if(ptr[offset3] < c_b)
                                      goto success_homogeneous;
                                    else
                                      if(ptr[offset10] < c_b)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                  else
                                    if(ptr[offset10] < c_b)
                                      if(ptr[offset11] < c_b)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset10] < c_b)
                                if(ptr[offset11] < c_b)
                                  if(ptr[offset1] < c_b)
                                    goto success_homogeneous;
                                  else
                                    if(ptr[offset6] < c_b)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        if(ptr[offset5] > cb)
                          if(ptr[offset9] > cb)
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                if(ptr[offset8] > cb)
                                  if(ptr[offset4] > cb)
                                    if(ptr[offset3] > cb)
                                      goto success_homogeneous;
                                    else
                                      if(ptr[offset10] > cb)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                  else
                                    if(ptr[offset10] > cb)
                                      if(ptr[offset11] > cb)
                                        goto success_homogeneous;
                                      else
                                        goto homogeneous;
                                    else
                                      goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                else
                  if(ptr[offset5] > cb)
                    if(ptr[offset9] > cb)
                      if(ptr[offset6] > cb)
                        if(ptr[offset7] > cb)
                          if(ptr[offset4] > cb)
                            if(ptr[offset3] > cb)
                              if(ptr[offset8] > cb)
                                goto success_homogeneous;
                              else
                                if(ptr[offset1] > cb)
                                  if(ptr[offset2] > cb)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset8] > cb)
                                if(ptr[offset10] > cb)
                                  goto success_homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            if(ptr[offset11] > cb)
                              if(ptr[offset8] > cb)
                                if(ptr[offset10] > cb)
                                  goto success_homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        goto homogeneous;
                    else
                      if(ptr[offset2] > cb)
                        if(ptr[offset3] > cb)
                          if(ptr[offset4] > cb)
                            if(ptr[offset7] > cb)
                              if(ptr[offset1] > cb)
                                if(ptr[offset6] > cb)
                                  goto success_homogeneous;
                                else
                                  goto homogeneous;
                              else
                                if(ptr[offset6] > cb)
                                  if(ptr[offset8] > cb)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        goto homogeneous;
                  else
                    if(ptr[offset5] < c_b)
                      if(ptr[offset9] < c_b)
                        if(ptr[offset6] < c_b)
                          if(ptr[offset7] < c_b)
                            if(ptr[offset4] < c_b)
                              if(ptr[offset3] < c_b)
                                if(ptr[offset8] < c_b)
                                  goto success_homogeneous;
                                else
                                  if(ptr[offset1] < c_b)
                                    if(ptr[offset2] < c_b)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                if(ptr[offset8] < c_b)
                                  if(ptr[offset10] < c_b)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                            else
                              if(ptr[offset11] < c_b)
                                if(ptr[offset8] < c_b)
                                  if(ptr[offset10] < c_b)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  goto homogeneous;
                              else
                                goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                      else
                        if(ptr[offset2] < c_b)
                          if(ptr[offset3] < c_b)
                            if(ptr[offset4] < c_b)
                              if(ptr[offset7] < c_b)
                                if(ptr[offset1] < c_b)
                                  if(ptr[offset6] < c_b)
                                    goto success_homogeneous;
                                  else
                                    goto homogeneous;
                                else
                                  if(ptr[offset6] < c_b)
                                    if(ptr[offset8] < c_b)
                                      goto success_homogeneous;
                                    else
                                      goto homogeneous;
                                  else
                                    goto homogeneous;
                              else
                                goto homogeneous;
                            else
                              goto homogeneous;
                          else
                            goto homogeneous;
                        else
                          goto homogeneous;
                    else
                      goto homogeneous;
            }
          }
          structured:
          {
            x++;
            if(x > xsizeB)
                break;
            else
            {
                const unsigned char* const ptr = img.ptr() + y*width + x;
                const int cb = *ptr + threshold;
                const int c_b = *ptr - threshold;
                if(ptr[offset0] > cb)
                  if(ptr[offset5] > cb)
                    if(ptr[offset2] > cb)
                      if(ptr[offset9] > cb)
                        if(ptr[offset1] > cb)
                          if(ptr[offset6] > cb)
                            if(ptr[offset3] > cb)
                              if(ptr[offset4] > cb)
                                goto success_structured;
                              else
                                if(ptr[offset10] > cb)
                                  if(ptr[offset11] > cb)
                                    goto success_structured;
                                  else
                                    goto structured;
                                else
                                  goto structured;
                            else
                              if(ptr[offset8] > cb)
                                if(ptr[offset10] > cb)
                                  if(ptr[offset11] > cb)
                                    goto success_structured;
                                  else
                                    if(ptr[offset4] > cb)
                                      if(ptr[offset7] > cb)
                                        goto success_structured;
                                      else
                                        goto structured;
                                    else
                                      goto structured;
                                else
                                  goto structured;
                              else
                                goto structured;
                          else
                            if(ptr[offset11] > cb)
                              if(ptr[offset3] > cb)
                                if(ptr[offset4] > cb)
                                  goto success_structured;
                                else
                                  if(ptr[offset10] > cb)
                                    goto success_structured;
                                  else
                                    goto structured;
                              else
                                if(ptr[offset8] > cb)
                                  if(ptr[offset10] > cb)
                                    goto success_structured;
                                  else
                                    goto structured;
                                else
                                  goto structured;
                            else
                              goto structured;
                        else
                          if(ptr[offset6] > cb)
                            if(ptr[offset7] > cb)
                              if(ptr[offset8] > cb)
                                if(ptr[offset4] > cb)
                                  if(ptr[offset3] > cb)
                                    goto success_structured;
                                  else
                                    if(ptr[offset10] > cb)
                                      goto success_structured;
                                    else
                                      goto structured;
                                else
                                  if(ptr[offset10] > cb)
                                    if(ptr[offset11] > cb)
                                      goto success_structured;
                                    else
                                      goto structured;
                                  else
                                    goto structured;
                              else
                                goto structured;
                            else
                              goto structured;
                          else
                            goto structured;
                      else
                        if(ptr[offset3] > cb)
                          if(ptr[offset4] > cb)
                            if(ptr[offset1] > cb)
                              if(ptr[offset6] > cb)
                                goto success_structured;
                              else
                                if(ptr[offset11] > cb)
                                  goto success_structured;
                                else
                                  goto structured;
                            else
                              if(ptr[offset6] > cb)
                                if(ptr[offset7] > cb)
                                  if(ptr[offset8] > cb)
                                    goto success_structured;
                                  else
                                    goto structured;
                                else
                                  goto structured;
                              else
                                goto structured;
                          else
                            goto structured;
                        else
                          goto structured;
                    else
                      if(ptr[offset9] > cb)
                        if(ptr[offset7] > cb)
                          if(ptr[offset8] > cb)
                            if(ptr[offset1] > cb)
                              if(ptr[offset10] > cb)
                                if(ptr[offset11] > cb)
                                  goto success_structured;
                                else
                                  if(ptr[offset6] > cb)
                                    if(ptr[offset4] > cb)
                                      goto success_structured;
                                    else
                                      goto structured;
                                  else
                                    goto structured;
                              else
                                if(ptr[offset6] > cb)
                                  if(ptr[offset3] > cb)
                                    if(ptr[offset4] > cb)
                                      goto success_structured;
                                    else
                                      goto structured;
                                  else
                                    goto structured;
                                else
                                  goto structured;
                        
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

- **AgastFeatureDetector_Impl**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `agast_score.hpp`
- `precomp.hpp`

**Python Imports:**
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

