# Documentation for `modules/features2d/src/agast_score.cpp`

## File Metadata

- **Full Path**: `modules/features2d/src/agast_score.cpp`
- **File Name**: `agast_score.cpp`
- **File Size**: 467,366 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/features2d/src/agast_score.cpp](../../../modules/features2d/src/agast_score.cpp)

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

#include "agast_score.hpp"

namespace cv
{

void makeAgastOffsets(int pixel[16], int rowStride, AgastFeatureDetector::DetectorType type)
{
    static const int offsets16[][2] =
    {
        {-3,  0}, {-3, -1}, {-2, -2}, {-1, -3}, {0, -3}, { 1, -3}, { 2, -2}, { 3, -1},
        { 3,  0}, { 3,  1}, { 2,  2}, { 1,  3}, {0,  3}, {-1,  3}, {-2,  2}, {-3,  1}
    };

    static const int offsets12d[][2] =
    {
        {-3,  0}, {-2, -1}, {-1, -2}, {0, -3}, { 1, -2}, { 2, -1},
        { 3,  0}, { 2,  1}, { 1,  2}, {0,  3}, {-1,  2}, {-2,  1}
    };

    static const int offsets12s[][2] =
    {
        {-2,  0}, {-2, -1}, {-1, -2}, {0, -2}, { 1, -2}, { 2, -1},
        { 2,  0}, { 2,  1}, { 1,  2}, {0,  2}, {-1,  2}, {-2,  1}
    };

    static const int offsets8[][2] =
    {
        {-1,  0}, {-1, -1}, {0, -1}, { 1, -1},
        { 1,  0}, { 1,  1}, {0,  1}, {-1,  1}
    };

    const int (*offsets)[2] = type == AgastFeatureDetector::OAST_9_16 ? offsets16 :
                              type == AgastFeatureDetector::AGAST_7_12d ? offsets12d :
                              type == AgastFeatureDetector::AGAST_7_12s ? offsets12s :
                              type == AgastFeatureDetector::AGAST_5_8 ? offsets8  : 0;

    const int offsets_len = type == AgastFeatureDetector::OAST_9_16 ? 16 :
                            type == AgastFeatureDetector::AGAST_7_12d ? 12 :
                            type == AgastFeatureDetector::AGAST_7_12s ? 12 :
                            type == AgastFeatureDetector::AGAST_5_8 ? 8 : 0;

    CV_Assert(pixel && offsets);

    int k = 0;
    for( ; k < offsets_len; k++ )
        pixel[k] = offsets[k][0] + offsets[k][1] * rowStride;
}

#if (defined __i386__ || defined(_M_IX86) || defined __x86_64__ || defined(_M_X64) || defined(_M_ARM64) || defined(__aarch64__) || defined(__arm__))
// 16 pixel mask
template<>
int agast_cornerScore<AgastFeatureDetector::OAST_9_16>(const uchar* ptr, const int pixel[], int threshold)
{
    int bmin = threshold;
    int bmax = 255;
    int b_test = (bmax + bmin) / 2;

    short offset0 = (short) pixel[0];
    short offset1 = (short) pixel[1];
    short offset2 = (short) pixel[2];
    short offset3 = (short) pixel[3];
    short offset4 = (short) pixel[4];
    short offset5 = (short) pixel[5];
    short offset6 = (short) pixel[6];
    short offset7 = (short) pixel[7];
    short offset8 = (short) pixel[8];
    short offset9 = (short) pixel[9];
    short offset10 = (short) pixel[10];
    short offset11 = (short) pixel[11];
    short offset12 = (short) pixel[12];
    short offset13 = (short) pixel[13];
    short offset14 = (short) pixel[14];
    short offset15 = (short) pixel[15];

    while(true)
    {
        const int cb = *ptr + b_test;
        const int c_b = *ptr - b_test;
        if(ptr[offset0] > cb)
          if(ptr[offset2] > cb)
            if(ptr[offset4] > cb)
              if(ptr[offset5] > cb)
                if(ptr[offset7] > cb)
                  if(ptr[offset3] > cb)
                    if(ptr[offset1] > cb)
                      if(ptr[offset6] > cb)
                        if(ptr[offset8] > cb)
                          goto is_a_corner;
                        else
                          if(ptr[offset15] > cb)
                            goto is_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset13] > cb)
                          if(ptr[offset14] > cb)
                            if(ptr[offset15] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset8] > cb)
                        if(ptr[offset9] > cb)
                          if(ptr[offset10] > cb)
                            if(ptr[offset6] > cb)
                              goto is_a_corner;
                            else
                              if(ptr[offset11] > cb)
                                if(ptr[offset12] > cb)
                                  if(ptr[offset13] > cb)
                                    if(ptr[offset14] > cb)
                                      if(ptr[offset15] > cb)
                                        goto is_a_corner;
                                      else
                                        goto is_not_a_corner;
                                    else
                                      goto is_not_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset10] > cb)
                      if(ptr[offset11] > cb)
                        if(ptr[offset12] > cb)
                          if(ptr[offset8] > cb)
                            if(ptr[offset9] > cb)
                              if(ptr[offset6] > cb)
                                goto is_a_corner;
                              else
                                if(ptr[offset13] > cb)
                                  if(ptr[offset14] > cb)
                                    if(ptr[offset15] > cb)
                                      goto is_a_corner;
                                    else
                                      goto is_not_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                            else
                              if(ptr[offset1] > cb)
                                if(ptr[offset13] > cb)
                                  if(ptr[offset14] > cb)
                                    if(ptr[offset15] > cb)
                                      goto is_a_corner;
                                    else
                                      goto is_not_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            if(ptr[offset1] > cb)
                              if(ptr[offset13] > cb)
                                if(ptr[offset14] > cb)
                                  if(ptr[offset15] > cb)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else if(ptr[offset7] < c_b)
                  if(ptr[offset14] > cb)
                    if(ptr[offset15] > cb)
                      if(ptr[offset1] > cb)
                        if(ptr[offset3] > cb)
                          if(ptr[offset6] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset13] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] > cb)
                            if(ptr[offset11] > cb)
                              if(ptr[offset12] > cb)
                                if(ptr[offset13] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset8] > cb)
                          if(ptr[offset9] > cb)
                            if(ptr[offset10] > cb)
                              if(ptr[offset11] > cb)
                                if(ptr[offset12] > cb)
                                  if(ptr[offset13] > cb)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else if(ptr[offset14] < c_b)
                    if(ptr[offset8] < c_b)
                      if(ptr[offset9] < c_b)
                        if(ptr[offset10] < c_b)
                          if(ptr[offset11] < c_b)
                            if(ptr[offset12] < c_b)
                              if(ptr[offset13] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto is_a_corner;
                                else
                                  if(ptr[offset15] < c_b)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  if(ptr[offset14] > cb)
                    if(ptr[offset15] > cb)
                      if(ptr[offset1] > cb)
                        if(ptr[offset3] > cb)
                          if(ptr[offset6] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset13] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] > cb)
                            if(ptr[offset11] > cb)
                              if(ptr[offset12] > cb)
                                if(ptr[offset13] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset8] > cb)
                          if(ptr[offset9] > cb)
                            if(ptr[offset10] > cb)
                              if(ptr[offset11] > cb)
                                if(ptr[offset12] > cb)
                                  if(ptr[offset13] > cb)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
              else if(ptr[offset5] < c_b)
                if(ptr[offset12] > cb)
                  if(ptr[offset13] > cb)
                    if(ptr[offset14] > cb)
                      if(ptr[offset15] > cb)
                        if(ptr[offset1] > cb)
                          if(ptr[offset3] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] > cb)
                              if(ptr[offset11] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset8] > cb)
                            if(ptr[offset9] > cb)
                              if(ptr[offset10] > cb)
                                if(ptr[offset11] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset6] > cb)
                          if(ptr[offset7] > cb)
                            if(ptr[offset8] > cb)
                              if(ptr[offset9] > cb)
                                if(ptr[offset10] > cb)
                                  if(ptr[offset11] > cb)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else if(ptr[offset12] < c_b)
                  if(ptr[offset7] < c_b)
                    if(ptr[offset8] < c_b)
                      if(ptr[offset9] < c_b)
                        if(ptr[offset10] < c_b)
                          if(ptr[offset11] < c_b)
                            if(ptr[offset13] < c_b)
                              if(ptr[offset6] < c_b)
                                goto is_a_corner;
                              else
                                if(ptr[offset14] < c_b)
                                  if(ptr[offset15] < c_b)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                if(ptr[offset12] > cb)
                  if(ptr[offset13] > cb)
                    if(ptr[offset14] > cb)
                      if(ptr[offset15] > cb)
                        if(ptr[offset1] > cb)
                          if(ptr[offset3] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] > cb)
                              if(ptr[offset11] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset8] > cb)
                            if(ptr[offset9] > cb)
                              if(ptr[offset10] > cb)
                                if(ptr[offset11] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset6] > cb)
                          if(ptr[offset7] > cb)
                            if(ptr[offset8] > cb)
                              if(ptr[offset9] > cb)
                                if(ptr[offset10] > cb)
                                  if(ptr[offset11] > cb)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else if(ptr[offset12] < c_b)
                  if(ptr[offset7] < c_b)
                    if(ptr[offset8] < c_b)
                      if(ptr[offset9] < c_b)
                        if(ptr[offset10] < c_b)
                          if(ptr[offset11] < c_b)
                            if(ptr[offset13] < c_b)
                              if(ptr[offset14] < c_b)
                                if(ptr[offset6] < c_b)
                                  goto is_a_corner;
                                else
                                  if(ptr[offset15] < c_b)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
            else if(ptr[offset4] < c_b)
              if(ptr[offset11] > cb)
                if(ptr[offset12] > cb)
                  if(ptr[offset13] > cb)
                    if(ptr[offset10] > cb)
                      if(ptr[offset14] > cb)
                        if(ptr[offset15] > cb)
                          if(ptr[offset1] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset8] > cb)
                              if(ptr[offset9] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset6] > cb)
                            if(ptr[offset7] > cb)
                              if(ptr[offset8] > cb)
                                if(ptr[offset9] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset5] > cb)
                          if(ptr[offset6] > cb)
                            if(ptr[offset7] > cb)
                              if(ptr[offset8] > cb)
                                if(ptr[offset9] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset1] > cb)
                        if(ptr[offset3] > cb)
                          if(ptr[offset14] > cb)
                            if(ptr[offset15] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else if(ptr[offset11] < c_b)
                if(ptr[offset7] < c_b)
                  if(ptr[offset8] < c_b)
                    if(ptr[offset9] < c_b)
                      if(ptr[offset10] < c_b)
                        if(ptr[offset6] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset3] < c_b)
                              goto is_a_corner;
                            else
                              if(ptr[offset12] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            if(ptr[offset12] < c_b)
                              if(ptr[offset13] < c_b)
                                if(ptr[offset14] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset12] < c_b)
                            if(ptr[offset13] < c_b)
                              if(ptr[offset14] < c_b)
                                if(ptr[offset15] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else
              if(ptr[offset11] > cb)
                if(ptr[offset12] > cb)
                  if(ptr[offset13] > cb)
                    if(ptr[offset10] > cb)
                      if(ptr[offset14] > cb)
                        if(ptr[offset15] > cb)
                          if(ptr[offset1] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset8] > cb)
                              if(ptr[offset9] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset6] > cb)
                            if(ptr[offset7] > cb)
                              if(ptr[offset8] > cb)
                                if(ptr[offset9] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset5] > cb)
                          if(ptr[offset6] > cb)
                            if(ptr[offset7] > cb)
                              if(ptr[offset8] > cb)
                                if(ptr[offset9] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset1] > cb)
                        if(ptr[offset3] > cb)
                          if(ptr[offset14] > cb)
                            if(ptr[offset15] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else if(ptr[offset11] < c_b)
                if(ptr[offset7] < c_b)
                  if(ptr[offset8] < c_b)
                    if(ptr[offset9] < c_b)
                      if(ptr[offset10] < c_b)
                        if(ptr[offset12] < c_b)
                          if(ptr[offset13] < c_b)
                            if(ptr[offset6] < c_b)
                              if(ptr[offset5] < c_b)
                                goto is_a_corner;
                              else
                                if(ptr[offset14] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                            else
                              if(ptr[offset14] < c_b)
                                if(ptr[offset15] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
          else if(ptr[offset2] < c_b)
            if(ptr[offset9] > cb)
              if(ptr[offset10] > cb)
                if(ptr[offset11] > cb)
                  if(ptr[offset8] > cb)
                    if(ptr[offset12] > cb)
                      if(ptr[offset13] > cb)
                        if(ptr[offset14] > cb)
                          if(ptr[offset15] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset5] > cb)
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset4] > cb)
                          if(ptr[offset5] > cb)
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset3] > cb)
                        if(ptr[offset4] > cb)
                          if(ptr[offset5] > cb)
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset1] > cb)
                      if(ptr[offset12] > cb)
                        if(ptr[offset13] > cb)
                          if(ptr[offset14] > cb)
                            if(ptr[offset15] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else if(ptr[offset9] < c_b)
              if(ptr[offset7] < c_b)
                if(ptr[offset8] < c_b)
                  if(ptr[offset6] < c_b)
                    if(ptr[offset5] < c_b)
                      if(ptr[offset4] < c_b)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset1] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] < c_b)
                            if(ptr[offset11] < c_b)
                              if(ptr[offset12] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset10] < c_b)
                          if(ptr[offset11] < c_b)
                            if(ptr[offset12] < c_b)
                              if(ptr[offset13] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset10] < c_b)
                        if(ptr[offset11] < c_b)
                          if(ptr[offset12] < c_b)
                            if(ptr[offset13] < c_b)
                              if(ptr[offset14] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset10] < c_b)
                      if(ptr[offset11] < c_b)
                        if(ptr[offset12] < c_b)
                          if(ptr[offset13] < c_b)
                            if(ptr[offset14] < c_b)
                              if(ptr[offset15] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else
              goto is_not_a_corner;
          else
            if(ptr[offset9] > cb)
              if(ptr[offset10] > cb)
                if(ptr[offset11] > cb)
                  if(ptr[offset8] > cb)
                    if(ptr[offset12] > cb)
                      if(ptr[offset13] > cb)
                        if(ptr[offset14] > cb)
                          if(ptr[offset15] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset5] > cb)
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset4] > cb)
                          if(ptr[offset5] > cb)
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset3] > cb)
                        if(ptr[offset4] > cb)
                          if(ptr[offset5] > cb)
                            if(ptr[offset6] > cb)
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset1] > cb)
                      if(ptr[offset12] > cb)
                        if(ptr[offset13] > cb)
                          if(ptr[offset14] > cb)
                            if(ptr[offset15] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else if(ptr[offset9] < c_b)
              if(ptr[offset7] < c_b)
                if(ptr[offset8] < c_b)
                  if(ptr[offset10] < c_b)
                    if(ptr[offset11] < c_b)
                      if(ptr[offset6] < c_b)
                        if(ptr[offset5] < c_b)
                          if(ptr[offset4] < c_b)
                            if(ptr[offset3] < c_b)
                              goto is_a_corner;
                            else
                              if(ptr[offset12] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            if(ptr[offset12] < c_b)
                              if(ptr[offset13] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset12] < c_b)
                            if(ptr[offset13] < c_b)
                              if(ptr[offset14] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset12] < c_b)
                          if(ptr[offset13] < c_b)
                            if(ptr[offset14] < c_b)
                              if(ptr[offset15] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else
              goto is_not_a_corner;
        else if(ptr[offset0] < c_b)
          if(ptr[offset2] > cb)
            if(ptr[offset9] > cb)
              if(ptr[offset7] > cb)
                if(ptr[offset8] > cb)
                  if(ptr[offset6] > cb)
                    if(ptr[offset5] > cb)
                      if(ptr[offset4] > cb)
                        if(ptr[offset3] > cb)
                          if(ptr[offset1] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] > cb)
                            if(ptr[offset11] > cb)
                              if(ptr[offset12] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset10] > cb)
                          if(ptr[offset11] > cb)
                            if(ptr[offset12] > cb)
                              if(ptr[offset13] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset10] > cb)
                        if(ptr[offset11] > cb)
                          if(ptr[offset12] > cb)
                            if(ptr[offset13] > cb)
                              if(ptr[offset14] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset10] > cb)
                      if(ptr[offset11] > cb)
                        if(ptr[offset12] > cb)
                          if(ptr[offset13] > cb)
                            if(ptr[offset14] > cb)
                              if(ptr[offset15] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else if(ptr[offset9] < c_b)
              if(ptr[offset10] < c_b)
                if(ptr[offset11] < c_b)
                  if(ptr[offset8] < c_b)
                    if(ptr[offset12] < c_b)
                      if(ptr[offset13] < c_b)
                        if(ptr[offset14] < c_b)
                          if(ptr[offset15] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset6] < c_b)
                              if(ptr[offset7] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset5] < c_b)
                            if(ptr[offset6] < c_b)
                              if(ptr[offset7] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset4] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset6] < c_b)
                              if(ptr[offset7] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset3] < c_b)
                        if(ptr[offset4] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset6] < c_b)
                              if(ptr[offset7] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset1] < c_b)
                      if(ptr[offset12] < c_b)
                        if(ptr[offset13] < c_b)
                          if(ptr[offset14] < c_b)
                            if(ptr[offset15] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else
              goto is_not_a_corner;
          else if(ptr[offset2] < c_b)
            if(ptr[offset4] > cb)
              if(ptr[offset11] > cb)
                if(ptr[offset7] > cb)
                  if(ptr[offset8] > cb)
                    if(ptr[offset9] > cb)
                      if(ptr[offset10] > cb)
                        if(ptr[offset6] > cb)
                          if(ptr[offset5] > cb)
                            if(ptr[offset3] > cb)
                              goto is_a_corner;
                            else
                              if(ptr[offset12] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            if(ptr[offset12] > cb)
                              if(ptr[offset13] > cb)
                                if(ptr[offset14] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset12] > cb)
                            if(ptr[offset13] > cb)
                              if(ptr[offset14] > cb)
                                if(ptr[offset15] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else if(ptr[offset11] < c_b)
                if(ptr[offset12] < c_b)
                  if(ptr[offset13] < c_b)
                    if(ptr[offset10] < c_b)
                      if(ptr[offset14] < c_b)
                        if(ptr[offset15] < c_b)
                          if(ptr[offset1] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset8] < c_b)
                              if(ptr[offset9] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset6] < c_b)
                            if(ptr[offset7] < c_b)
                              if(ptr[offset8] < c_b)
                                if(ptr[offset9] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset5] < c_b)
                          if(ptr[offset6] < c_b)
                            if(ptr[offset7] < c_b)
                              if(ptr[offset8] < c_b)
                                if(ptr[offset9] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset1] < c_b)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset14] < c_b)
                            if(ptr[offset15] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else if(ptr[offset4] < c_b)
              if(ptr[offset5] > cb)
                if(ptr[offset12] > cb)
                  if(ptr[offset7] > cb)
                    if(ptr[offset8] > cb)
                      if(ptr[offset9] > cb)
                        if(ptr[offset10] > cb)
                          if(ptr[offset11] > cb)
                            if(ptr[offset13] > cb)
                              if(ptr[offset6] > cb)
                                goto is_a_corner;
                              else
                                if(ptr[offset14] > cb)
                                  if(ptr[offset15] > cb)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else if(ptr[offset12] < c_b)
                  if(ptr[offset13] < c_b)
                    if(ptr[offset14] < c_b)
                      if(ptr[offset15] < c_b)
                        if(ptr[offset1] < c_b)
                          if(ptr[offset3] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] < c_b)
                              if(ptr[offset11] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset8] < c_b)
                            if(ptr[offset9] < c_b)
                              if(ptr[offset10] < c_b)
                                if(ptr[offset11] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset6] < c_b)
                          if(ptr[offset7] < c_b)
                            if(ptr[offset8] < c_b)
                              if(ptr[offset9] < c_b)
                                if(ptr[offset10] < c_b)
                                  if(ptr[offset11] < c_b)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else if(ptr[offset5] < c_b)
                if(ptr[offset7] > cb)
                  if(ptr[offset14] > cb)
                    if(ptr[offset8] > cb)
                      if(ptr[offset9] > cb)
                        if(ptr[offset10] > cb)
                          if(ptr[offset11] > cb)
                            if(ptr[offset12] > cb)
                              if(ptr[offset13] > cb)
                                if(ptr[offset6] > cb)
                                  goto is_a_corner;
                                else
                                  if(ptr[offset15] > cb)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else if(ptr[offset14] < c_b)
                    if(ptr[offset15] < c_b)
                      if(ptr[offset1] < c_b)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset6] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset13] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] < c_b)
                            if(ptr[offset11] < c_b)
                              if(ptr[offset12] < c_b)
                                if(ptr[offset13] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset8] < c_b)
                          if(ptr[offset9] < c_b)
                            if(ptr[offset10] < c_b)
                              if(ptr[offset11] < c_b)
                                if(ptr[offset12] < c_b)
                                  if(ptr[offset13] < c_b)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else if(ptr[offset7] < c_b)
                  if(ptr[offset3] < c_b)
                    if(ptr[offset1] < c_b)
                      if(ptr[offset6] < c_b)
                        if(ptr[offset8] < c_b)
                          goto is_a_corner;
                        else
                          if(ptr[offset15] < c_b)
                            goto is_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset13] < c_b)
                          if(ptr[offset14] < c_b)
                            if(ptr[offset15] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset8] < c_b)
                        if(ptr[offset9] < c_b)
                          if(ptr[offset10] < c_b)
                            if(ptr[offset6] < c_b)
                              goto is_a_corner;
                            else
                              if(ptr[offset11] < c_b)
                                if(ptr[offset12] < c_b)
                                  if(ptr[offset13] < c_b)
                                    if(ptr[offset14] < c_b)
                                      if(ptr[offset15] < c_b)
                                        goto is_a_corner;
                                      else
                                        goto is_not_a_corner;
                                    else
                                      goto is_not_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset10] < c_b)
                      if(ptr[offset11] < c_b)
                        if(ptr[offset12] < c_b)
                          if(ptr[offset8] < c_b)
                            if(ptr[offset9] < c_b)
                              if(ptr[offset6] < c_b)
                                goto is_a_corner;
                              else
                                if(ptr[offset13] < c_b)
                                  if(ptr[offset14] < c_b)
                                    if(ptr[offset15] < c_b)
                                      goto is_a_corner;
                                    else
                                      goto is_not_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                            else
                              if(ptr[offset1] < c_b)
                                if(ptr[offset13] < c_b)
                                  if(ptr[offset14] < c_b)
                                    if(ptr[offset15] < c_b)
                                      goto is_a_corner;
                                    else
                                      goto is_not_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            if(ptr[offset1] < c_b)
                              if(ptr[offset13] < c_b)
                                if(ptr[offset14] < c_b)
                                  if(ptr[offset15] < c_b)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  if(ptr[offset14] < c_b)
                    if(ptr[offset15] < c_b)
                      if(ptr[offset1] < c_b)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset6] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset13] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] < c_b)
                            if(ptr[offset11] < c_b)
                              if(ptr[offset12] < c_b)
                                if(ptr[offset13] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset8] < c_b)
                          if(ptr[offset9] < c_b)
                            if(ptr[offset10] < c_b)
                              if(ptr[offset11] < c_b)
                                if(ptr[offset12] < c_b)
                                  if(ptr[offset13] < c_b)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
              else
                if(ptr[offset12] > cb)
                  if(ptr[offset7] > cb)
                    if(ptr[offset8] > cb)
                      if(ptr[offset9] > cb)
                        if(ptr[offset10] > cb)
                          if(ptr[offset11] > cb)
                            if(ptr[offset13] > cb)
                              if(ptr[offset14] > cb)
                                if(ptr[offset6] > cb)
                                  goto is_a_corner;
                                else
                                  if(ptr[offset15] > cb)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else if(ptr[offset12] < c_b)
                  if(ptr[offset13] < c_b)
                    if(ptr[offset14] < c_b)
                      if(ptr[offset15] < c_b)
                        if(ptr[offset1] < c_b)
                          if(ptr[offset3] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] < c_b)
                              if(ptr[offset11] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset8] < c_b)
                            if(ptr[offset9] < c_b)
                              if(ptr[offset10] < c_b)
                                if(ptr[offset11] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset6] < c_b)
                          if(ptr[offset7] < c_b)
                            if(ptr[offset8] < c_b)
                              if(ptr[offset9] < c_b)
                                if(ptr[offset10] < c_b)
                                  if(ptr[offset11] < c_b)
                                    goto is_a_corner;
                                  else
                                    goto is_not_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
            else
              if(ptr[offset11] > cb)
                if(ptr[offset7] > cb)
                  if(ptr[offset8] > cb)
                    if(ptr[offset9] > cb)
                      if(ptr[offset10] > cb)
                        if(ptr[offset12] > cb)
                          if(ptr[offset13] > cb)
                            if(ptr[offset6] > cb)
                              if(ptr[offset5] > cb)
                                goto is_a_corner;
                              else
                                if(ptr[offset14] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                            else
                              if(ptr[offset14] > cb)
                                if(ptr[offset15] > cb)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else if(ptr[offset11] < c_b)
                if(ptr[offset12] < c_b)
                  if(ptr[offset13] < c_b)
                    if(ptr[offset10] < c_b)
                      if(ptr[offset14] < c_b)
                        if(ptr[offset15] < c_b)
                          if(ptr[offset1] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset8] < c_b)
                              if(ptr[offset9] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset6] < c_b)
                            if(ptr[offset7] < c_b)
                              if(ptr[offset8] < c_b)
                                if(ptr[offset9] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset5] < c_b)
                          if(ptr[offset6] < c_b)
                            if(ptr[offset7] < c_b)
                              if(ptr[offset8] < c_b)
                                if(ptr[offset9] < c_b)
                                  goto is_a_corner;
                                else
                                  goto is_not_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset1] < c_b)
                        if(ptr[offset3] < c_b)
                          if(ptr[offset14] < c_b)
                            if(ptr[offset15] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
          else
            if(ptr[offset9] > cb)
              if(ptr[offset7] > cb)
                if(ptr[offset8] > cb)
                  if(ptr[offset10] > cb)
                    if(ptr[offset11] > cb)
                      if(ptr[offset6] > cb)
                        if(ptr[offset5] > cb)
                          if(ptr[offset4] > cb)
                            if(ptr[offset3] > cb)
                              goto is_a_corner;
                            else
                              if(ptr[offset12] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            if(ptr[offset12] > cb)
                              if(ptr[offset13] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset12] > cb)
                            if(ptr[offset13] > cb)
                              if(ptr[offset14] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset12] > cb)
                          if(ptr[offset13] > cb)
                            if(ptr[offset14] > cb)
                              if(ptr[offset15] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else if(ptr[offset9] < c_b)
              if(ptr[offset10] < c_b)
                if(ptr[offset11] < c_b)
                  if(ptr[offset8] < c_b)
                    if(ptr[offset12] < c_b)
                      if(ptr[offset13] < c_b)
                        if(ptr[offset14] < c_b)
                          if(ptr[offset15] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset6] < c_b)
                              if(ptr[offset7] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset5] < c_b)
                            if(ptr[offset6] < c_b)
                              if(ptr[offset7] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset4] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset6] < c_b)
                              if(ptr[offset7] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset3] < c_b)
                        if(ptr[offset4] < c_b)
                          if(ptr[offset5] < c_b)
                            if(ptr[offset6] < c_b)
                              if(ptr[offset7] < c_b)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset1] < c_b)
                      if(ptr[offset12] < c_b)
                        if(ptr[offset13] < c_b)
                          if(ptr[offset14] < c_b)
                            if(ptr[offset15] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else
              goto is_not_a_corner;
        else
          if(ptr[offset7] > cb)
            if(ptr[offset8] > cb)
              if(ptr[offset9] > cb)
                if(ptr[offset6] > cb)
                  if(ptr[offset5] > cb)
                    if(ptr[offset4] > cb)
                      if(ptr[offset3] > cb)
                        if(ptr[offset2] > cb)
                          if(ptr[offset1] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] > cb)
                            if(ptr[offset11] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset10] > cb)
                          if(ptr[offset11] > cb)
                            if(ptr[offset12] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset10] > cb)
                        if(ptr[offset11] > cb)
                          if(ptr[offset12] > cb)
                            if(ptr[offset13] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset10] > cb)
                      if(ptr[offset11] > cb)
                        if(ptr[offset12] > cb)
                          if(ptr[offset13] > cb)
                            if(ptr[offset14] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  if(ptr[offset10] > cb)
                    if(ptr[offset11] > cb)
                      if(ptr[offset12] > cb)
                        if(ptr[offset13] > cb)
                          if(ptr[offset14] > cb)
                            if(ptr[offset15] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else
              goto is_not_a_corner;
          else if(ptr[offset7] < c_b)
            if(ptr[offset8] < c_b)
              if(ptr[offset9] < c_b)
                if(ptr[offset6] < c_b)
                  if(ptr[offset5] < c_b)
                    if(ptr[offset4] < c_b)
                      if(ptr[offset3] < c_b)
                        if(ptr[offset2] < c_b)
                          if(ptr[offset1] < c_b)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] < c_b)
                            if(ptr[offset11] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset10] < c_b)
                          if(ptr[offset11] < c_b)
                            if(ptr[offset12] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset10] < c_b)
                        if(ptr[offset11] < c_b)
                          if(ptr[offset12] < c_b)
                            if(ptr[offset13] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset10] < c_b)
                      if(ptr[offset11] < c_b)
                        if(ptr[offset12] < c_b)
                          if(ptr[offset13] < c_b)
                            if(ptr[offset14] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  if(ptr[offset10] < c_b)
                    if(ptr[offset11] < c_b)
                      if(ptr[offset12] < c_b)
                        if(ptr[offset13] < c_b)
                          if(ptr[offset14] < c_b)
                            if(ptr[offset15] < c_b)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
              else
                goto is_not_a_corner;
            else
              goto is_not_a_corner;
          else
            goto is_not_a_corner;

        is_a_corner:
            bmin = b_test;
            goto end;

        is_not_a_corner:
            bmax = b_test;
            goto end;

        end:

        if(bmin == bmax - 1 || bmin == bmax)
            return bmin;
        b_test = (bmin + bmax) / 2;
    }
}

// 12 pixel mask in diamond format
template<>
int agast_cornerScore<AgastFeatureDetector::AGAST_7_12d>(const uchar* ptr, const int pixel[], int threshold)
{
    int bmin = threshold;
    int bmax = 255;
    int b_test = (bmax + bmin)/2;

    short offset0 = (short) pixel[0];
    short offset1 = (short) pixel[1];
    short offset2 = (short) pixel[2];
    short offset3 = (short) pixel[3];
    short offset4 = (short) pixel[4];
    short offset5 = (short) pixel[5];
    short offset6 = (short) pixel[6];
    short offset7 = (short) pixel[7];
    short offset8 = (short) pixel[8];
    short offset9 = (short) pixel[9];
    short offset10 = (short) pixel[10];
    short offset11 = (short) pixel[11];

    while(true)
    {
        const int cb = *ptr + b_test;
        const int c_b = *ptr - b_test;
        if(ptr[offset0] > cb)
          if(ptr[offset5] > cb)
            if(ptr[offset2] > cb)
              if(ptr[offset9] > cb)
                if(ptr[offset1] > cb)
                  if(ptr[offset6] > cb)
                    if(ptr[offset3] > cb)
                      if(ptr[offset4] > cb)
                        goto is_a_corner;
                      else
                        if(ptr[offset10] > cb)
                          if(ptr[offset11] > cb)
                            goto is_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset8] > cb)
                        if(ptr[offset10] > cb)
                          if(ptr[offset11] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset4] > cb)
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    if(ptr[offset11] > cb)
                      if(ptr[offset3] > cb)
                        if(ptr[offset4] > cb)
                          goto is_a_corner;
                        else
                          if(ptr[offset10] > cb)
                            goto is_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset8] > cb)
                          if(ptr[offset10] > cb)
                            goto is_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                else
                  if(ptr[offset6] > cb)
                    if(ptr[offset7] > cb)
                      if(ptr[offset8] > cb)
                        if(ptr[offset4] > cb)
                          if(ptr[offset3] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] > cb)
                            if(ptr[offset11] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                    else
                      goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
              else
                if(ptr[offset3] > cb)
                  if(ptr[offset4] > cb)
                    if(ptr[offset1] > cb)
                      if(ptr[offset6] > cb)
                        goto is_a_corner;
                      else
                        if(ptr[offset11] > cb)
                          goto is_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset6] > cb)
                        if(ptr[offset7] > cb)
                          if(ptr[offset8] > cb)
                            goto is_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
            else
              if(ptr[offset9] > cb)
                if(ptr[offset7] > cb)
                  if(ptr[offset8] > cb)
                    if(ptr[offset1] > cb)
                      if(ptr[offset10] > cb)
                        if(ptr[offset11] > cb)
                          goto is_a_corner;
                        else
                          if(ptr[offset6] > cb)
                            if(ptr[offset4] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        if(ptr[offset6] > cb)
                          if(ptr[offset3] > cb)
                            if(ptr[offset4] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                        else
                          goto is_not_a_corner;
                    else
                      if(ptr[offset6] > cb)
                        if(ptr[offset4] > cb)
                          if(ptr[offset3] > cb)
                            goto is_a_corner;
                          else
                            if(ptr[offset10] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                        else
                          if(ptr[offset10] > cb)
                            if(ptr[offset11] > cb)
                              goto is_a_corner;
                            else
                              goto is_not_a_corner;
                          else
                            goto is_not_a_corner;
                      else
                        goto is_not_a_corner;
                  else
                    goto is_not_a_corner;
                else
                  goto is_not_a_corner;
              else
                goto is_not_a_corner;
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
                              goto is_a_corner;
                            else
                              if(ptr[offset7] > cb)
                                goto is_a_corner;
                              else
                                goto is_not_a_corner;
                          else
                            goto
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `agast_score.hpp`

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

