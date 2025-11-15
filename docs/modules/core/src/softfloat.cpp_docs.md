# Documentation for `modules/core/src/softfloat.cpp`

## File Metadata

- **Full Path**: `modules/core/src/softfloat.cpp`
- **File Name**: `softfloat.cpp`
- **File Size**: 168,014 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/src/softfloat.cpp](../../../modules/core/src/softfloat.cpp)

## Purpose and Role

This file is located in the `modules/core/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

// This file is based on files from packages softfloat and fdlibm
// issued with the following licenses:

/*============================================================================

This C source file is part of the SoftFloat IEEE Floating-Point Arithmetic
Package, Release 3c, by John R. Hauser.

Copyright 2011, 2012, 2013, 2014, 2015, 2016, 2017 The Regents of the
University of California.  All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

 1. Redistributions of source code must retain the above copyright notice,
    this list of conditions, and the following disclaimer.

 2. Redistributions in binary form must reproduce the above copyright notice,
    this list of conditions, and the following disclaimer in the documentation
    and/or other materials provided with the distribution.

 3. Neither the name of the University nor the names of its contributors may
    be used to endorse or promote products derived from this software without
    specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS "AS IS", AND ANY
EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE, ARE
DISCLAIMED.  IN NO EVENT SHALL THE REGENTS OR CONTRIBUTORS BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

=============================================================================*/

// FDLIBM licenses:

/*
 * ====================================================
 * Copyright (C) 1993 by Sun Microsystems, Inc. All rights reserved.
 *
 * Developed at SunSoft, a Sun Microsystems, Inc. business.
 * Permission to use, copy, modify, and distribute this
 * software is freely granted, provided that this notice
 * is preserved.
 * ====================================================
 */

/*
 * ====================================================
 * Copyright (C) 2004 by Sun Microsystems, Inc. All rights reserved.
 *
 * Permission to use, copy, modify, and distribute this
 * software is freely granted, provided that this notice
 * is preserved.
 * ====================================================
 */

#include "precomp.hpp"

#include "opencv2/core/softfloat.hpp"

namespace cv
{

/*----------------------------------------------------------------------------
| Software floating-point underflow tininess-detection mode.
*----------------------------------------------------------------------------*/
enum {
    tininess_beforeRounding = 0,
    tininess_afterRounding  = 1
};
//fixed to make softfloat code stateless
static const uint_fast8_t globalDetectTininess = tininess_afterRounding;

/*----------------------------------------------------------------------------
| Software floating-point exception flags.
*----------------------------------------------------------------------------*/
enum {
    flag_inexact   =  1,
    flag_underflow =  2,
    flag_overflow  =  4,
    flag_infinite  =  8,
    flag_invalid   = 16
};

// Disabled to make softfloat code stateless
// This function may be changed in the future for better error handling
static inline void raiseFlags( uint_fast8_t /* flags */)
{
    //exceptionFlags |= flags;
}

/*----------------------------------------------------------------------------
| Software floating-point rounding mode.
*----------------------------------------------------------------------------*/
enum {
    round_near_even   = 0, // round to nearest, with ties to even
    round_minMag      = 1, // round to minimum magnitude (toward zero)
    round_min         = 2, // round to minimum (down)
    round_max         = 3, // round to maximum (up)
    round_near_maxMag = 4, // round to nearest, with ties to maximum magnitude (away from zero)
    round_odd         = 5  // round to odd (jamming)
};
/* What is round_odd (from SoftFloat manual):
 * If supported, mode round_odd first rounds a floating-point result to minimum magnitude,
 * the same as round_minMag, and then, if the result is inexact, the least-significant bit
 * of the result is set to 1. This rounding mode is also known as jamming.
 */

//fixed to make softfloat code stateless
static const uint_fast8_t globalRoundingMode = round_near_even;

/*----------------------------------------------------------------------------
*----------------------------------------------------------------------------*/

#define signF32UI( a ) (((uint32_t) (a)>>31) != 0)
#define expF32UI( a ) ((int_fast16_t) ((a)>>23) & 0xFF)
#define fracF32UI( a ) ((a) & 0x007FFFFF)
#define packToF32UI( sign, exp, sig ) (((uint32_t) (sign)<<31) + ((uint32_t) (exp)<<23) + (sig))

#define isNaNF32UI( a ) (((~(a) & 0x7F800000) == 0) && ((a) & 0x007FFFFF))

/*----------------------------------------------------------------------------
*----------------------------------------------------------------------------*/

#define signF64UI( a ) (((uint64_t) (a)>>63) != 0)
#define expF64UI( a ) ((int_fast16_t) ((a)>>52) & 0x7FF)
#define fracF64UI( a ) ((a) & UINT64_C( 0x000FFFFFFFFFFFFF ))
#define packToF64UI( sign, exp, sig ) ((uint64_t) (((uint_fast64_t) (sign)<<63) + ((uint_fast64_t) (exp)<<52) + (sig)))

#define isNaNF64UI( a ) (((~(a) & UINT64_C( 0x7FF0000000000000 )) == 0) && ((a) & UINT64_C( 0x000FFFFFFFFFFFFF )))

/*----------------------------------------------------------------------------
| Types used to pass 32-bit and 64-bit floating-point
| arguments and results to/from functions.  These types must be exactly
| 32 bits and 64 bits in size, respectively.  Where a
| platform has "native" support for IEEE-Standard floating-point formats,
| the types below may, if desired, be defined as aliases for the native types
| (typically 'float' and 'double').
*----------------------------------------------------------------------------*/
typedef softfloat float32_t;
typedef softdouble float64_t;

/*----------------------------------------------------------------------------
| Integer-to-floating-point conversion routines.
*----------------------------------------------------------------------------*/
static float32_t ui32_to_f32( uint32_t );
static float64_t ui32_to_f64( uint32_t );
static float32_t ui64_to_f32( uint64_t );
static float64_t ui64_to_f64( uint64_t );
static float32_t i32_to_f32( int32_t );
static float64_t i32_to_f64( int32_t );
static float32_t i64_to_f32( int64_t );
static float64_t i64_to_f64( int64_t );

/*----------------------------------------------------------------------------
| 32-bit (single-precision) floating-point operations.
*----------------------------------------------------------------------------*/
static int_fast32_t f32_to_i32( float32_t, uint_fast8_t, bool );
static int_fast32_t f32_to_i32_r_minMag( float32_t, bool );
static float64_t f32_to_f64( float32_t );
static float32_t f32_roundToInt( float32_t, uint_fast8_t, bool );
static float32_t f32_add( float32_t, float32_t );
static float32_t f32_sub( float32_t, float32_t );
static float32_t f32_mul( float32_t, float32_t );
static float32_t f32_mulAdd( float32_t, float32_t, float32_t );
static float32_t f32_div( float32_t, float32_t );
static float32_t f32_rem( float32_t, float32_t );
static float32_t f32_sqrt( float32_t );
static bool f32_eq( float32_t, float32_t );
static bool f32_le( float32_t, float32_t );
static bool f32_lt( float32_t, float32_t );

/*----------------------------------------------------------------------------
| 64-bit (double-precision) floating-point operations.
*----------------------------------------------------------------------------*/
static int_fast32_t f64_to_i32( float64_t, uint_fast8_t, bool );
static int_fast64_t f64_to_i64( float64_t, uint_fast8_t, bool );
static int_fast32_t f64_to_i32_r_minMag( float64_t, bool );
static float32_t f64_to_f32( float64_t );
static float64_t f64_roundToInt( float64_t, uint_fast8_t, bool );
static float64_t f64_add( float64_t, float64_t );
static float64_t f64_sub( float64_t, float64_t );
static float64_t f64_mul( float64_t, float64_t );
static float64_t f64_mulAdd( float64_t, float64_t, float64_t );
static float64_t f64_div( float64_t, float64_t );
static float64_t f64_rem( float64_t, float64_t );
static float64_t f64_sqrt( float64_t );
static bool f64_eq( float64_t, float64_t );
static bool f64_le( float64_t, float64_t );
static bool f64_lt( float64_t, float64_t );

/*----------------------------------------------------------------------------
| Ported from OpenCV and fdlibm and added for usability
*----------------------------------------------------------------------------*/

static float32_t f32_powi( float32_t x, int y);
static float64_t f64_powi( float64_t x, int y);
static float64_t f64_sin_kernel(float64_t x);
static float64_t f64_cos_kernel(float64_t x);
static void f64_sincos_reduce(const float64_t& x, float64_t& y, int& n);

static float32_t f32_exp( float32_t x);
static float64_t f64_exp(float64_t x);
static float32_t f32_log(float32_t x);
static float64_t f64_log(float64_t x);
static float32_t f32_cbrt(float32_t x);
static float32_t f32_pow( float32_t x, float32_t y);
static float64_t f64_pow( float64_t x, float64_t y);
static float64_t f64_sin( float64_t x );
static float64_t f64_cos( float64_t x );

/*----------------------------------------------------------------------------
| softfloat and softdouble methods and members
*----------------------------------------------------------------------------*/

softfloat::softfloat( const uint32_t a ) { *this = ui32_to_f32(a); }
softfloat::softfloat( const uint64_t a ) { *this = ui64_to_f32(a); }
softfloat::softfloat( const  int32_t a ) { *this =  i32_to_f32(a); }
softfloat::softfloat( const  int64_t a ) { *this =  i64_to_f32(a); }

softfloat::operator softdouble() const { return f32_to_f64(*this); }

softfloat softfloat::operator + (const softfloat& a) const { return f32_add(*this, a); }
softfloat softfloat::operator - (const softfloat& a) const { return f32_sub(*this, a); }
softfloat softfloat::operator * (const softfloat& a) const { return f32_mul(*this, a); }
softfloat softfloat::operator / (const softfloat& a) const { return f32_div(*this, a); }
softfloat softfloat::operator % (const softfloat& a) const { return f32_rem(*this, a); }

bool softfloat::operator == ( const softfloat& a ) const { return  f32_eq(*this, a); }
bool softfloat::operator != ( const softfloat& a ) const { return !f32_eq(*this, a); }
bool softfloat::operator >  ( const softfloat& a ) const { return  f32_lt(a, *this); }
bool softfloat::operator >= ( const softfloat& a ) const { return  f32_le(a, *this); }
bool softfloat::operator <  ( const softfloat& a ) const { return  f32_lt(*this, a); }
bool softfloat::operator <= ( const softfloat& a ) const { return  f32_le(*this, a); }

softdouble::softdouble( const uint32_t a ) { *this = ui32_to_f64(a); }
softdouble::softdouble( const uint64_t a ) { *this = ui64_to_f64(a); }
softdouble::softdouble( const  int32_t a ) { *this =  i32_to_f64(a); }
softdouble::softdouble( const  int64_t a ) { *this =  i64_to_f64(a); }

}

int cvTrunc(const cv::softfloat& a) { return cv::f32_to_i32_r_minMag(a, false); }
int cvRound(const cv::softfloat& a) { return cv::f32_to_i32(a, cv::round_near_even, false); }
int cvFloor(const cv::softfloat& a) { return cv::f32_to_i32(a, cv::round_min, false); }
int cvCeil (const cv::softfloat& a) { return cv::f32_to_i32(a, cv::round_max, false); }

int cvTrunc(const cv::softdouble& a) { return cv::f64_to_i32_r_minMag(a, false); }
int cvRound(const cv::softdouble& a) { return cv::f64_to_i32(a, cv::round_near_even, false); }
int cvFloor(const cv::softdouble& a) { return cv::f64_to_i32(a, cv::round_min, false); }
int cvCeil (const cv::softdouble& a) { return cv::f64_to_i32(a, cv::round_max, false); }

int64_t cvRound64(const cv::softdouble& a) { return cv::f64_to_i64(a, cv::round_near_even, false); }

namespace cv
{
softdouble::operator softfloat() const { return f64_to_f32(*this); }

softdouble softdouble::operator + (const softdouble& a) const { return f64_add(*this, a); }
softdouble softdouble::operator - (const softdouble& a) const { return f64_sub(*this, a); }
softdouble softdouble::operator * (const softdouble& a) const { return f64_mul(*this, a); }
softdouble softdouble::operator / (const softdouble& a) const { return f64_div(*this, a); }
softdouble softdouble::operator % (const softdouble& a) const { return f64_rem(*this, a); }

bool softdouble::operator == (const softdouble& a) const { return  f64_eq(*this, a); }
bool softdouble::operator != (const softdouble& a) const { return !f64_eq(*this, a); }
bool softdouble::operator >  (const softdouble& a) const { return  f64_lt(a, *this); }
bool softdouble::operator >= (const softdouble& a) const { return  f64_le(a, *this); }
bool softdouble::operator <  (const softdouble& a) const { return  f64_lt(*this, a); }
bool softdouble::operator <= (const softdouble& a) const { return  f64_le(*this, a); }

/*----------------------------------------------------------------------------
| Overloads for math functions
*----------------------------------------------------------------------------*/

softfloat  mulAdd( const softfloat&  a, const softfloat&  b, const softfloat & c) { return f32_mulAdd(a, b, c); }
softdouble mulAdd( const softdouble& a, const softdouble& b, const softdouble& c) { return f64_mulAdd(a, b, c); }

softfloat  sqrt( const softfloat&  a ) { return f32_sqrt(a); }
softdouble sqrt( const softdouble& a ) { return f64_sqrt(a); }

softfloat  exp( const softfloat&  a) { return f32_exp(a); }
softdouble exp( const softdouble& a) { return f64_exp(a); }

softfloat  log( const softfloat&  a ) { return f32_log(a); }
softdouble log( const softdouble& a ) { return f64_log(a); }

softfloat  pow( const softfloat&  a, const softfloat&  b) { return f32_pow(a, b); }
softdouble pow( const softdouble& a, const softdouble& b) { return f64_pow(a, b); }

softfloat cbrt(const softfloat& a) { return f32_cbrt(a); }

softdouble sin(const softdouble& a) { return f64_sin(a); }
softdouble cos(const softdouble& a) { return f64_cos(a); }

/*----------------------------------------------------------------------------
| The values to return on conversions to 32-bit integer formats that raise an
| invalid exception.
*----------------------------------------------------------------------------*/
#define i32_fromPosOverflow  0x7FFFFFFF
#define i32_fromNegOverflow  (-0x7FFFFFFF - 1)
#define i32_fromNaN          0x7FFFFFFF

/*----------------------------------------------------------------------------
| The values to return on conversions to 64-bit integer formats that raise an
| invalid exception.
*----------------------------------------------------------------------------*/
#define i64_fromPosOverflow  UINT64_C( 0x7FFFFFFFFFFFFFFF )
//fixed unsigned unary minus: -x == ~x + 1
//#define i64_fromNegOverflow (-UINT64_C( 0x7FFFFFFFFFFFFFFF ) - 1)
#define i64_fromNegOverflow  (~UINT64_C( 0x7FFFFFFFFFFFFFFF ) + 1 - 1)
#define i64_fromNaN          UINT64_C( 0x7FFFFFFFFFFFFFFF )

/*----------------------------------------------------------------------------
| "Common NaN" structure, used to transfer NaN representations from one format
| to another.
*----------------------------------------------------------------------------*/
struct commonNaN {
    bool sign;
#ifndef WORDS_BIGENDIAN
    uint64_t v0, v64;
#else
    uint64_t v64, v0;
#endif
};

/*----------------------------------------------------------------------------
| The bit pattern for a default generated 32-bit floating-point NaN.
*----------------------------------------------------------------------------*/
#define defaultNaNF32UI 0xFFC00000

/*----------------------------------------------------------------------------
| Returns true when 32-bit unsigned integer `uiA' has the bit pattern of a
| 32-bit floating-point signaling NaN.
| Note:  This macro evaluates its argument more than once.
*----------------------------------------------------------------------------*/
#define softfloat_isSigNaNF32UI( uiA ) ((((uiA) & 0x7FC00000) == 0x7F800000) && ((uiA) & 0x003FFFFF))

/*----------------------------------------------------------------------------
| Assuming `uiA' has the bit pattern of a 32-bit floating-point NaN, converts
| this NaN to the common NaN form, and stores the resulting common NaN at the
| location pointed to by `zPtr'.  If the NaN is a signaling NaN, the invalid
| exception is raised.
*----------------------------------------------------------------------------*/
static void softfloat_f32UIToCommonNaN( uint_fast32_t uiA, struct commonNaN *zPtr );

/*----------------------------------------------------------------------------
| Converts the common NaN pointed to by `aPtr' into a 32-bit floating-point
| NaN, and returns the bit pattern of this value as an unsigned integer.
*----------------------------------------------------------------------------*/
static uint_fast32_t softfloat_commonNaNToF32UI( const struct commonNaN *aPtr );

/*----------------------------------------------------------------------------
| Interpreting `uiA' and `uiB' as the bit patterns of two 32-bit floating-
| point values, at least one of which is a NaN, returns the bit pattern of
| the combined NaN result.  If either `uiA' or `uiB' has the pattern of a
| signaling NaN, the invalid exception is raised.
*----------------------------------------------------------------------------*/
static uint_fast32_t softfloat_propagateNaNF32UI( uint_fast32_t uiA, uint_fast32_t uiB );

/*----------------------------------------------------------------------------
| The bit pattern for a default generated 64-bit floating-point NaN.
*----------------------------------------------------------------------------*/
#define defaultNaNF64UI UINT64_C( 0xFFF8000000000000 )

/*----------------------------------------------------------------------------
| Returns true when 64-bit unsigned integer `uiA' has the bit pattern of a
| 64-bit floating-point signaling NaN.
| Note:  This macro evaluates its argument more than once.
*----------------------------------------------------------------------------*/
#define softfloat_isSigNaNF64UI( uiA ) \
    ((((uiA) & UINT64_C( 0x7FF8000000000000 )) == UINT64_C( 0x7FF0000000000000 )) && \
      ((uiA) & UINT64_C( 0x0007FFFFFFFFFFFF )))

/*----------------------------------------------------------------------------
| Assuming `uiA' has the bit pattern of a 64-bit floating-point NaN, converts
| this NaN to the common NaN form, and stores the resulting common NaN at the
| location pointed to by `zPtr'.  If the NaN is a signaling NaN, the invalid
| exception is raised.
*----------------------------------------------------------------------------*/
static void softfloat_f64UIToCommonNaN( uint_fast64_t uiA, struct commonNaN *zPtr );

/*----------------------------------------------------------------------------
| Converts the common NaN pointed to by `aPtr' into a 64-bit floating-point
| NaN, and returns the bit pattern of this value as an unsigned integer.
*----------------------------------------------------------------------------*/
static uint_fast64_t softfloat_commonNaNToF64UI( const struct commonNaN *aPtr );

/*----------------------------------------------------------------------------
| Interpreting `uiA' and `uiB' as the bit patterns of two 64-bit floating-
| point values, at least one of which is a NaN, returns the bit pattern of
| the combined NaN result.  If either `uiA' or `uiB' has the pattern of a
| signaling NaN, the invalid exception is raised.
*----------------------------------------------------------------------------*/
static uint_fast64_t softfloat_propagateNaNF64UI( uint_fast64_t uiA, uint_fast64_t uiB );

/*----------------------------------------------------------------------------
*----------------------------------------------------------------------------*/

#ifndef WORDS_BIGENDIAN
struct uint128 { uint64_t v0, v64; };
struct uint64_extra { uint64_t extra, v; };
struct uint128_extra { uint64_t extra; struct uint128 v; };
#else
struct uint128 { uint64_t v64, v0; };
struct uint64_extra { uint64_t v, extra; };
struct uint128_extra { struct uint128 v; uint64_t extra; };
#endif

enum {
    softfloat_mulAdd_subC    = 1,
    softfloat_mulAdd_subProd = 2
};

/*----------------------------------------------------------------------------
*----------------------------------------------------------------------------*/
static int_fast32_t softfloat_roundToI32( bool, uint_fast64_t, uint_fast8_t, bool );

struct exp16_sig32 { int_fast16_t exp; uint_fast32_t sig; };
static struct exp16_sig32 softfloat_normSubnormalF32Sig( uint_fast32_t );

static float32_t softfloat_roundPackToF32( bool, int_fast16_t, uint_fast32_t );
static float32_t softfloat_normRoundPackToF32( bool, int_fast16_t, uint_fast32_t );

static float32_t softfloat_addMagsF32( uint_fast32_t, uint_fast32_t );
static float32_t softfloat_subMagsF32( uint_fast32_t, uint_fast32_t );
static float32_t softfloat_mulAddF32(uint_fast32_t, uint_fast32_t, uint_fast32_t, uint_fast8_t );

/*----------------------------------------------------------------------------
*----------------------------------------------------------------------------*/
static int_fast64_t softfloat_roundToI64( bool, uint_fast64_t, uint_fast64_t, uint_fast8_t, bool);

struct exp16_sig64 { int_fast16_t exp; uint_fast64_t sig; };
static struct exp16_sig64 softfloat_normSubnormalF64Sig( uint_fast64_t );

static float64_t softfloat_roundPackToF64( bool, int_fast16_t, uint_fast64_t );
static float64_t softfloat_normRoundPackToF64( bool, int_fast16_t, uint_fast64_t );

static float64_t softfloat_addMagsF64( uint_fast64_t, uint_fast64_t, bool );
static float64_t softfloat_subMagsF64( uint_fast64_t, uint_fast64_t, bool );
static float64_t softfloat_mulAddF64( uint_fast64_t, uint_fast64_t, uint_fast64_t, uint_fast8_t );

/*----------------------------------------------------------------------------
*----------------------------------------------------------------------------*/

/*----------------------------------------------------------------------------
| Shifts 'a' right by the number of bits given in 'dist', which must be in
| the range 1 to 63.  If any nonzero bits are shifted off, they are "jammed"
| into the least-significant bit of the shifted value by setting the least-
| significant bit to 1.  This shifted-and-jammed value is returned.
*----------------------------------------------------------------------------*/

static inline uint64_t softfloat_shortShiftRightJam64( uint64_t a, uint_fast8_t dist )
{ return a>>dist | ((a & (((uint_fast64_t) 1<<dist) - 1)) != 0); }

/*----------------------------------------------------------------------------
| Shifts 'a' right by the number of bits given in 'dist', which must not
| be zero.  If any nonzero bits are shifted off, they are "jammed" into the
| least-significant bit of the shifted value by setting the least-significant
| bit to 1.  This shifted-and-jammed value is returned.
|   The value of 'dist' can be arbitrarily large.  In particular, if 'dist' is
| greater than 32, the result will be either 0 or 1, depending on whether 'a'
| is zero or nonzero.
*----------------------------------------------------------------------------*/

static inline uint32_t softfloat_shiftRightJam32( uint32_t a, uint_fast16_t dist )
{
    //fixed unsigned unary minus: -x == ~x + 1
    return (dist < 31) ? a>>dist | ((uint32_t) (a<<((~dist + 1) & 31)) != 0) : (a != 0);
}

/*----------------------------------------------------------------------------
| Shifts 'a' right by the number of bits given in 'dist', which must not
| be zero.  If any nonzero bits are shifted off, they are "jammed" into the
| least-significant bit of the shifted value by setting the least-significant
| bit to 1.  This shifted-and-jammed value is returned.
|   The value of 'dist' can be arbitrarily large.  In particular, if 'dist' is
| greater than 64, the result will be either 0 or 1, depending on whether 'a'
| is zero or nonzero.
*----------------------------------------------------------------------------*/
static inline uint64_t softfloat_shiftRightJam64( uint64_t a, uint_fast32_t dist )
{
    //fixed unsigned unary minus: -x == ~x + 1
    return (dist < 63) ? a>>dist | ((uint64_t) (a<<((~dist + 1) & 63)) != 0) : (a != 0);
}

/*----------------------------------------------------------------------------
| A constant table that translates an 8-bit unsigned integer (the array index)
| into the number of leading 0 bits before the most-significant 1 of that
| integer.  For integer zero (index 0), the corresponding table element is 8.
*----------------------------------------------------------------------------*/
static const uint_least8_t softfloat_countLeadingZeros8[256] = {
    8, 7, 6, 6, 5, 5, 5, 5, 4, 4, 4, 4, 4, 4, 4, 4,
    3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3,
    2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2,
    2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2,
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
    1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
};

/*----------------------------------------------------------------------------
| Returns the number of leading 0 bits before the most-significant 1 bit of
| 'a'.  If 'a' is zero, 32 is returned.
*----------------------------------------------------------------------------*/
static inline uint_fast8_t softfloat_countLeadingZeros32( uint32_t a )
{
    uint_fast8_t count = 0;
    if ( a < 0x10000 ) {
        count = 16;
        a <<= 16;
    }
    if ( a < 0x1000000 ) {
        count += 8;
        a <<= 8;
    }
    count += softfloat_countLeadingZeros8[a>>24];
    return count;
}

/*----------------------------------------------------------------------------
| Returns the number of leading 0 bits before the most-significant 1 bit of
| 'a'.  If 'a' is zero, 64 is returned.
*----------------------------------------------------------------------------*/
static uint_fast8_t softfloat_countLeadingZeros64( uint64_t a );

/*----------------------------------------------------------------------------
| Returns an approximation to the reciprocal of the number represented by 'a',
| where 'a' is interpreted as an unsigned fixed-point number with one integer
| bit and 31 fraction bits.  The 'a' input must be "normalized", meaning that
| its most-significant bit (bit 31) must be 1.  Thus, if A is the value of
| the fixed-point interpretation of 'a', then 1 <= A < 2.  The returned value
| is interpreted as a pure unsigned fraction, having no integer bits and 32
| fraction bits.  The approximation returned is never greater than the true
| reciprocal 1/A, and it differs from the true reciprocal by at most 2.006 ulp
| (units in the last place).
*----------------------------------------------------------------------------*/
#define softfloat_approxRecip32_1( a ) ((uint32_t) (UINT64_C( 0x7FFFFFFFFFFFFFFF ) / (uint32_t) (a)))

/*----------------------------------------------------------------------------
| Returns an approximation to the reciprocal of the square root of the number
| represented by 'a', where 'a' is interpreted as an unsigned fixed-point
| number either with one integer bit and 31 fraction bits or with two integer
| bits and 30 fraction bits.  The format of 'a' is determined by 'oddExpA',
| which must be either 0 or 1.  If 'oddExpA' is 1, 'a' is interpreted as
| having one integer bit, and if 'oddExpA' is 0, 'a' is interpreted as having
| two integer bits.  The 'a' input must be "normalized", meaning that its
| most-significant bit (bit 31) must be 1.  Thus, if A is the value of the
| fixed-point interpretation of 'a', it follows that 1 <= A < 2 when 'oddExpA'
| is 1, and 2 <= A < 4 when 'oddExpA' is 0.
|   The returned value is interpreted as a pure unsigned fraction, having
| no integer bits and 32 fraction bits.  The approximation returned is never
| greater than the true reciprocal 1/sqrt(A), and it differs from the true
| reciprocal by at most 2.06 ulp (units in the last place).  The approximation
| returned is also always within the range 0.5 to 1; thus, the most-
| significant bit of the result is always set.
*----------------------------------------------------------------------------*/
static uint32_t softfloat_approxRecipSqrt32_1( unsigned int oddExpA, uint32_t a );

static const uint16_t softfloat_approxRecipSqrt_1k0s[16] = {
    0xB4C9, 0xFFAB, 0xAA7D, 0xF11C, 0xA1C5, 0xE4C7, 0x9A43, 0xDA29,
    0x93B5, 0xD0E5, 0x8DED, 0xC8B7, 0x88C6, 0xC16D, 0x8424, 0xBAE1
};
static const uint16_t softfloat_approxRecipSqrt_1k1s[16] = {
    0xA5A5, 0xEA42, 0x8C21, 0xC62D, 0x788F, 0xAA7F, 0x6928, 0x94B6,
    0x5CC7, 0x8335, 0x52A6, 0x74E2, 0x4A3E, 0x68FE, 0x432B, 0x5EFD
};

/*----------------------------------------------------------------------------
| Shifts the 128 bits formed by concatenating 'a64' and 'a0' left by the
| number of bits given in 'dist', which must be in the range 1 to 63.
*----------------------------------------------------------------------------*/
static inline struct uint128 softfloat_shortShiftLeft128( uint64_t a64, uint64_t a0, uint_fast8_t dist )
{
    struct uint128 z;
    z.v64 = a64<<dist | a0>>(-dist & 63);
    z.v0 = a0<<dist;
    return z;
}

/*----------------------------------------------------------------------------
| Shifts the 128 bits formed by concatenating 'a64' and 'a0' right by the
| number of bits given in 'dist', which must be in the range 1 to 63.  If any
| nonzero bits are shifted off, they are "jammed" into the least-significant
| bit of the shifted value by setting the least-significant bit to 1.  This
| shifted-and-jammed value is returned.
*----------------------------------------------------------------------------*/
static inline struct uint128 softfloat_shortShiftRightJam128(uint64_t a64, uint64_t a0, uint_fast8_t dist )
{
    uint_fast8_t negDist = -dist;
    struct uint128 z;
    z.v64 = a64>>dist;
    z.v0 =
        a64<<(negDist & 63) | a0>>dist
            | ((uint64_t) (a0<<(negDist & 63)) != 0);
    return z;
}

/*----------------------------------------------------------------------------
| Shifts the 128 bits formed by concatenating 'a64' and 'a0' right by the
| number of bits given in 'dist', which must not be zero.  If any nonzero bits
| are shifted off, they are "jammed" into the least-significant bit of the
| shifted value by setting the least-significant bit to 1.  This shifted-and-
| jammed value is returned.
|   The value of 'dist' can be arbitrarily large.  In particular, if 'dist' is
| greater than 128, the result will be either 0 or 1, depending on whether the
| original 128 bits are all zeros.
*----------------------------------------------------------------------------*/
static struct uint128 softfloat_shiftRightJam128( uint64_t a64, uint64_t a0, uint_fast32_t dist );

/*----------------------------------------------------------------------------
| Returns the sum of the 128-bit integer formed by concatenating 'a64' and
| 'a0' and the 128-bit integer formed by concatenating 'b64' and 'b0'.  The
| addition is modulo 2^128, so any carry out is lost.
*----------------------------------------------------------------------------*/
static inline struct uint128 softfloat_add128( uint64_t a64, uint64_t a0, uint64_t b64, uint64_t b0 )
{
    struct uint128 z;
    z.v0 = a0 + b0;
    z.v64 = a64 + b64 + (z.v0 < a0);
    return z;
}

/*----------------------------------------------------------------------------
| Returns the difference of the 128-bit integer formed by concatenating 'a64'
| and 'a0' and the 128-bit integer formed by concatenating 'b64' and 'b0'.
| The subtraction is modulo 2^128, so any borrow out (carry out) is lost.
*----------------------------------------------------------------------------*/
static inline struct uint128 softfloat_sub128( uint64_t a64, uint64_t a0, uint64_t b64, uint64_t b0 )
{
    struct uint128 z;
    z.v0 = a0 - b0;
    z.v64 = a64 - b64;
    z.v64 -= (a0 < b0);
    return z;
}

/*----------------------------------------------------------------------------
| Returns the 128-bit product of 'a' and 'b'.
*----------------------------------------------------------------------------*/
static struct uint128 softfloat_mul64To128( uint64_t a, uint64_t b );

/*----------------------------------------------------------------------------
*----------------------------------------------------------------------------*/
/*----------------------------------------------------------------------------
*----------------------------------------------------------------------------*/

static float32_t f32_add( float32_t a, float32_t b )
{
    uint_fast32_t uiA = a.v;
    uint_fast32_t uiB = b.v;

    if ( signF32UI( uiA ^ uiB ) ) {
        return softfloat_subMagsF32( uiA, uiB );
    } else {
        return softfloat_addMagsF32( uiA, uiB );
    }
}

static float32_t f32_div( float32_t a, float32_t b )
{
    uint_fast32_t uiA;
    bool signA;
    int_fast16_t expA;
    uint_fast32_t sigA;
    uint_fast32_t uiB;
    bool signB;
    int_fast16_t expB;
    uint_fast32_t sigB;
    bool signZ;
    struct exp16_sig32 normExpSig;
    int_fast16_t expZ;
    uint_fast64_t sig64A;
    uint_fast32_t sigZ;
    uint_fast32_t uiZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    signA = signF32UI( uiA );
    expA  = expF32UI( uiA );
    sigA  = fracF32UI( uiA );
    uiB = b.v;
    signB = signF32UI( uiB );
    expB  = expF32UI( uiB );
    sigB  = fracF32UI( uiB );
    signZ = signA ^ signB;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA == 0xFF ) {
        if ( sigA ) goto propagateNaN;
        if ( expB == 0xFF ) {
            if ( sigB ) goto propagateNaN;
            goto invalid;
        }
        goto infinity;
    }
    if ( expB == 0xFF ) {
        if ( sigB ) goto propagateNaN;
        goto zero;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! expB ) {
        if ( ! sigB ) {
            if ( ! (expA | sigA) ) goto invalid;
            raiseFlags( flag_infinite );
            goto infinity;
        }
        normExpSig = softfloat_normSubnormalF32Sig( sigB );
        expB = normExpSig.exp;
        sigB = normExpSig.sig;
    }
    if ( ! expA ) {
        if ( ! sigA ) goto zero;
        normExpSig = softfloat_normSubnormalF32Sig( sigA );
        expA = normExpSig.exp;
        sigA = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expZ = expA - expB + 0x7E;
    sigA |= 0x00800000;
    sigB |= 0x00800000;
    if ( sigA < sigB ) {
        --expZ;
        sig64A = (uint_fast64_t) sigA<<31;
    } else {
        sig64A = (uint_fast64_t) sigA<<30;
    }
    sigZ = (uint_fast32_t)(sig64A / sigB); // fixed warning on type cast
    if ( ! (sigZ & 0x3F) ) sigZ |= ((uint_fast64_t) sigB * sigZ != sig64A);
    return softfloat_roundPackToF32( signZ, expZ, sigZ );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 propagateNaN:
    uiZ = softfloat_propagateNaNF32UI( uiA, uiB );
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 invalid:
    raiseFlags( flag_invalid );
    uiZ = defaultNaNF32UI;
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 infinity:
    uiZ = packToF32UI( signZ, 0xFF, 0 );
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 zero:
    uiZ = packToF32UI( signZ, 0, 0 );
 uiZ:
    return float32_t::fromRaw(uiZ);
}

static bool f32_eq( float32_t a, float32_t b )
{
    uint_fast32_t uiA;
    uint_fast32_t uiB;

    uiA = a.v;
    uiB = b.v;
    if ( isNaNF32UI( uiA ) || isNaNF32UI( uiB ) )
    {
        if (softfloat_isSigNaNF32UI( uiA ) || softfloat_isSigNaNF32UI( uiB ) )
            raiseFlags( flag_invalid );
        return false;
    }
    return (uiA == uiB) || ! (uint32_t) ((uiA | uiB)<<1);
}

static bool f32_le( float32_t a, float32_t b )
{
    uint_fast32_t uiA;
    uint_fast32_t uiB;
    bool signA, signB;

    uiA = a.v;
    uiB = b.v;
    if ( isNaNF32UI( uiA ) || isNaNF32UI( uiB ) )
    {
        raiseFlags( flag_invalid );
        return false;
    }
    signA = signF32UI( uiA );
    signB = signF32UI( uiB );
    return (signA != signB) ? signA || ! (uint32_t) ((uiA | uiB)<<1)
                            : (uiA == uiB) || (signA ^ (uiA < uiB));
}

static bool f32_lt( float32_t a, float32_t b )
{
    uint_fast32_t uiA;
    uint_fast32_t uiB;
    bool signA, signB;

    uiA = a.v; uiB = b.v;
    if ( isNaNF32UI( uiA ) || isNaNF32UI( uiB ) )
    {
        raiseFlags( flag_invalid );
        return false;
    }
    signA = signF32UI( uiA );
    signB = signF32UI( uiB );
    return (signA != signB) ? signA && ((uint32_t) ((uiA | uiB)<<1) != 0)
                            : (uiA != uiB) && (signA ^ (uiA < uiB));
}

static float32_t f32_mulAdd( float32_t a, float32_t b, float32_t c )
{
    uint_fast32_t uiA;
    uint_fast32_t uiB;
    uint_fast32_t uiC;

    uiA = a.v;
    uiB = b.v;
    uiC = c.v;
    return softfloat_mulAddF32( uiA, uiB, uiC, 0 );
}

static float32_t f32_mul( float32_t a, float32_t b )
{
    uint_fast32_t uiA;
    bool signA;
    int_fast16_t expA;
    uint_fast32_t sigA;
    uint_fast32_t uiB;
    bool signB;
    int_fast16_t expB;
    uint_fast32_t sigB;
    bool signZ;
    uint_fast32_t magBits;
    struct exp16_sig32 normExpSig;
    int_fast16_t expZ;
    uint_fast32_t sigZ, uiZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    signA = signF32UI( uiA );
    expA  = expF32UI( uiA );
    sigA  = fracF32UI( uiA );
    uiB = b.v;
    signB = signF32UI( uiB );
    expB  = expF32UI( uiB );
    sigB  = fracF32UI( uiB );
    signZ = signA ^ signB;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA == 0xFF ) {
        if ( sigA || ((expB == 0xFF) && sigB) ) goto propagateNaN;
        magBits = expB | sigB;
        goto infArg;
    }
    if ( expB == 0xFF ) {
        if ( sigB ) goto propagateNaN;
        magBits = expA | sigA;
        goto infArg;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! expA ) {
        if ( ! sigA ) goto zero;
        normExpSig = softfloat_normSubnormalF32Sig( sigA );
        expA = normExpSig.exp;
        sigA = normExpSig.sig;
    }
    if ( ! expB ) {
        if ( ! sigB ) goto zero;
        normExpSig = softfloat_normSubnormalF32Sig( sigB );
        expB = normExpSig.exp;
        sigB = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expZ = expA + expB - 0x7F;
    sigA = (sigA | 0x00800000)<<7;
    sigB = (sigB | 0x00800000)<<8;
    sigZ = (uint_fast32_t)softfloat_shortShiftRightJam64( (uint_fast64_t) sigA * sigB, 32 ); //fixed warning on type cast
    if ( sigZ < 0x40000000 ) {
        --expZ;
        sigZ <<= 1;
    }
    return softfloat_roundPackToF32( signZ, expZ, sigZ );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 propagateNaN:
    uiZ = softfloat_propagateNaNF32UI( uiA, uiB );
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 infArg:
    if ( ! magBits ) {
        raiseFlags( flag_invalid );
        uiZ = defaultNaNF32UI;
    } else {
        uiZ = packToF32UI( signZ, 0xFF, 0 );
    }
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 zero:
    uiZ = packToF32UI( signZ, 0, 0 );
 uiZ:
    return float32_t::fromRaw(uiZ);
}

static float32_t f32_rem( float32_t a, float32_t b )
{
    uint_fast32_t uiA;
    bool signA;
    int_fast16_t expA;
    uint_fast32_t sigA;
    uint_fast32_t uiB;
    int_fast16_t expB;
    uint_fast32_t sigB;
    struct exp16_sig32 normExpSig;
    uint32_t rem;
    int_fast16_t expDiff;
    uint32_t q, recip32, altRem, meanRem;
    bool signRem;
    uint_fast32_t uiZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    signA = signF32UI( uiA );
    expA  = expF32UI( uiA );
    sigA  = fracF32UI( uiA );
    uiB = b.v;
    expB = expF32UI( uiB );
    sigB = fracF32UI( uiB );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA == 0xFF ) {
        if ( sigA || ((expB == 0xFF) && sigB) ) goto propagateNaN;
        goto invalid;
    }
    if ( expB == 0xFF ) {
        if ( sigB ) goto propagateNaN;
        return a;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! expB ) {
        if ( ! sigB ) goto invalid;
        normExpSig = softfloat_normSubnormalF32Sig( sigB );
        expB = normExpSig.exp;
        sigB = normExpSig.sig;
    }
    if ( ! expA ) {
        if ( ! sigA ) return a;
        normExpSig = softfloat_normSubnormalF32Sig( sigA );
        expA = normExpSig.exp;
        sigA = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    rem = sigA | 0x00800000;
    sigB |= 0x00800000;
    expDiff = expA - expB;
    if ( expDiff < 1 ) {
        if ( expDiff < -1 ) return a;
        sigB <<= 6;
        if ( expDiff ) {
            rem <<= 5;
            q = 0;
        } else {
            rem <<= 6;
            q = (sigB <= rem);
            if ( q ) rem -= sigB;
        }
    } else {
        recip32 = softfloat_approxRecip32_1( sigB<<8 );
        /*--------------------------------------------------------------------
        | Changing the shift of `rem' here requires also changing the initial
        | subtraction from `expDiff'.
        *--------------------------------------------------------------------*/
        rem <<= 7;
        expDiff -= 31;
        /*--------------------------------------------------------------------
        | The scale of `sigB' affects how many bits are obtained during each
        | cycle of the loop.  Currently this is 29 bits per loop iteration,
        | which is believed to be the maximum possible.
        *--------------------------------------------------------------------*/
        sigB <<= 6;
        for (;;) {
            q = (rem * (uint_fast64_t) recip32)>>32;
            if ( expDiff < 0 ) break;
            //fixed unsigned unary minus: -x == ~x + 1
            rem = ~(q * (uint32_t) sigB) + 1;
            expDiff -= 29;
        }
        /*--------------------------------------------------------------------
        | (`expDiff' cannot be less than -30 here.)
        *--------------------------------------------------------------------*/
        q >>= ~expDiff & 31;
        rem = (rem<<(expDiff + 30)) - q * (uint32_t) sigB;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    do {
        altRem = rem;
        ++q;
        rem -= sigB;
    } while ( ! (rem & 0x80000000) );
    meanRem = rem + altRem;
    if ( (meanRem & 0x80000000) || (! meanRem && (q & 1)) ) rem = altRem;
    signRem = signA;
    if ( 0x80000000 <= rem ) {
        signRem = ! signRem;
        //fixed unsigned unary minus: -x == ~x + 1
        rem = ~rem + 1;
    }
    return softfloat_normRoundPackToF32( signRem, expB, rem );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 propagateNaN:
    uiZ = softfloat_propagateNaNF32UI( uiA, uiB );
    goto uiZ;
 invalid:
    raiseFlags( flag_invalid );
    uiZ = defaultNaNF32UI;
 uiZ:
    return float32_t::fromRaw(uiZ);
}

static float32_t f32_roundToInt( float32_t a, uint_fast8_t roundingMode, bool exact )
{
    uint_fast32_t uiA;
    int_fast16_t exp;
    uint_fast32_t uiZ, lastBitMask, roundBitsMask;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    exp = expF32UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( exp <= 0x7E ) {
        if ( ! (uint32_t) (uiA<<1) ) return a;
        if ( exact ) raiseFlags(flag_inexact);
        uiZ = uiA & packToF32UI( 1, 0, 0 );
        switch ( roundingMode ) {
         case round_near_even:
            if ( ! fracF32UI( uiA ) ) break;
            /* fallthrough */
         case round_near_maxMag:
            if ( exp == 0x7E ) uiZ |= packToF32UI( 0, 0x7F, 0 );
            break;
         case round_min:
            if ( uiZ ) uiZ = packToF32UI( 1, 0x7F, 0 );
            break;
         case round_max:
            if ( ! uiZ ) uiZ = packToF32UI( 0, 0x7F, 0 );
            break;
        }
        goto uiZ;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( 0x96 <= exp ) {
        if ( (exp == 0xFF) && fracF32UI( uiA ) ) {
            uiZ = softfloat_propagateNaNF32UI( uiA, 0 );
            goto uiZ;
        }
        return a;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiZ = uiA;
    lastBitMask = (uint_fast32_t) 1<<(0x96 - exp);
    roundBitsMask = lastBitMask - 1;
    if ( roundingMode == round_near_maxMag ) {
        uiZ += lastBitMask>>1;
    } else if ( roundingMode == round_near_even ) {
        uiZ += lastBitMask>>1;
        if ( ! (uiZ & roundBitsMask) ) uiZ &= ~lastBitMask;
    } else if (
        roundingMode
            == (signF32UI( uiZ ) ? round_min : round_max)
    ) {
        uiZ += roundBitsMask;
    }
    uiZ &= ~roundBitsMask;
    if ( exact && (uiZ != uiA) ) {
        raiseFlags(flag_inexact);
    }
 uiZ:
    return float32_t::fromRaw(uiZ);
}

static float32_t f32_sqrt( float32_t a )
{
    uint_fast32_t uiA;
    bool signA;
    int_fast16_t expA;
    uint_fast32_t sigA, uiZ;
    struct exp16_sig32 normExpSig;
    int_fast16_t expZ;
    uint_fast32_t sigZ, shiftedSigZ;
    uint32_t negRem;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    signA = signF32UI( uiA );
    expA  = expF32UI( uiA );
    sigA  = fracF32UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA == 0xFF ) {
        if ( sigA ) {
            uiZ = softfloat_propagateNaNF32UI( uiA, 0 );
            goto uiZ;
        }
        if ( ! signA ) return a;
        goto invalid;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( signA ) {
        if ( ! (expA | sigA) ) return a;
        goto invalid;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! expA ) {
        if ( ! sigA ) return a;
        normExpSig = softfloat_normSubnormalF32Sig( sigA );
        expA = normExpSig.exp;
        sigA = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expZ = ((expA - 0x7F)>>1) + 0x7E;
    expA &= 1;
    sigA = (sigA | 0x00800000)<<8;
    sigZ =
        ((uint_fast64_t) sigA * softfloat_approxRecipSqrt32_1( expA, sigA ))
            >>32;
    if ( expA ) sigZ >>= 1;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    sigZ += 2;
    if ( (sigZ & 0x3F) < 2 ) {
        shiftedSigZ = sigZ>>2;
        negRem = shiftedSigZ * shiftedSigZ;
        sigZ &= ~3;
        if ( negRem & 0x80000000 ) {
            sigZ |= 1;
        } else {
            if ( negRem ) --sigZ;
        }
    }
    return softfloat_roundPackToF32( 0, expZ, sigZ );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 invalid:
    raiseFlags( flag_invalid );
    uiZ = defaultNaNF32UI;
 uiZ:
    return float32_t::fromRaw(uiZ);
}

static float32_t f32_sub( float32_t a, float32_t b )
{
    uint_fast32_t uiA;
    uint_fast32_t uiB;

    uiA = a.v;
    uiB = b.v;
    if ( signF32UI( uiA ^ uiB ) ) {
        return softfloat_addMagsF32( uiA, uiB );
    } else {
        return softfloat_subMagsF32( uiA, uiB );
    }
}

static float64_t f32_to_f64( float32_t a )
{
    uint_fast32_t uiA;
    bool sign;
    int_fast16_t exp;
    uint_fast32_t frac;
    struct commonNaN commonNaN;
    uint_fast64_t uiZ;
    struct exp16_sig32 normExpSig;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    sign = signF32UI( uiA );
    exp  = expF32UI( uiA );
    frac = fracF32UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( exp == 0xFF ) {
        if ( frac ) {
            softfloat_f32UIToCommonNaN( uiA, &commonNaN );
            uiZ = softfloat_commonNaNToF64UI( &commonNaN );
        } else {
            uiZ = packToF64UI( sign, 0x7FF, 0 );
        }
        goto uiZ;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! exp ) {
        if ( ! frac ) {
            uiZ = packToF64UI( sign, 0, 0 );
            goto uiZ;
        }
        normExpSig = softfloat_normSubnormalF32Sig( frac );
        exp = normExpSig.exp - 1;
        frac = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiZ = packToF64UI( sign, exp + 0x380, (uint_fast64_t) frac<<29 );
 uiZ:
    return float64_t::fromRaw(uiZ);
}

static int_fast32_t f32_to_i32( float32_t a, uint_fast8_t roundingMode, bool exact )
{
    uint_fast32_t uiA;
    bool sign;
    int_fast16_t exp;
    uint_fast32_t sig;
    uint_fast64_t sig64;
    int_fast16_t shiftDist;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    sign = signF32UI( uiA );
    exp  = expF32UI( uiA );
    sig  = fracF32UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
#if (i32_fromNaN != i32_fromPosOverflow) || (i32_fromNaN != i32_fromNegOverflow)
    if ( (exp == 0xFF) && sig ) {
#if (i32_fromNaN == i32_fromPosOverflow)
        sign = 0;
#elif (i32_fromNaN == i32_fromNegOverflow)
        sign = 1;
#else
        raiseFlags( flag_invalid );
        return i32_fromNaN;
#endif
    }
#endif
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( exp ) sig |= 0x00800000;
    sig64 = (uint_fast64_t) sig<<32;
    shiftDist = 0xAA - exp;
    if ( 0 < shiftDist ) sig64 = softfloat_shiftRightJam64( sig64, shiftDist );
    return softfloat_roundToI32( sign, sig64, roundingMode, exact );
}

static int_fast32_t f32_to_i32_r_minMag( float32_t a, bool exact )
{
    uint_fast32_t uiA;
    int_fast16_t exp;
    uint_fast32_t sig;
    int_fast16_t shiftDist;
    bool sign;
    int_fast32_t absZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    exp = expF32UI( uiA );
    sig = fracF32UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    shiftDist = 0x9E - exp;
    if ( 32 <= shiftDist ) {
        if ( exact && (exp | sig) ) {
            raiseFlags(flag_inexact);
        }
        return 0;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    sign = signF32UI( uiA );
    if ( shiftDist <= 0 ) {
        if ( uiA == packToF32UI( 1, 0x9E, 0 ) ) return -0x7FFFFFFF - 1;
        raiseFlags( flag_invalid );
        return
            (exp == 0xFF) && sig ? i32_fromNaN
                : sign ? i32_fromNegOverflow : i32_fromPosOverflow;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    sig = (sig | 0x00800000)<<8;
    absZ = sig>>shiftDist;
    if ( exact && ((uint_fast32_t) absZ<<shiftDist != sig) ) {
        raiseFlags(flag_inexact);
    }
    return sign ? -absZ : absZ;
}

static float64_t f64_add( float64_t a, float64_t b )
{
    uint_fast64_t uiA;
    bool signA;
    uint_fast64_t uiB;
    bool signB;

    uiA = a.v;
    signA = signF64UI( uiA );
    uiB = b.v;
    signB = signF64UI( uiB );
    if ( signA == signB ) {
        return softfloat_addMagsF64( uiA, uiB, signA );
    } else {
        return softfloat_subMagsF64( uiA, uiB, signA );
    }
}

static float64_t f64_div( float64_t a, float64_t b )
{
    uint_fast64_t uiA;
    bool signA;
    int_fast16_t expA;
    uint_fast64_t sigA;
    uint_fast64_t uiB;
    bool signB;
    int_fast16_t expB;
    uint_fast64_t sigB;
    bool signZ;
    struct exp16_sig64 normExpSig;
    int_fast16_t expZ;
    uint32_t recip32, sig32Z, doubleTerm;
    uint_fast64_t rem;
    uint32_t q;
    uint_fast64_t sigZ;
    uint_fast64_t uiZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    signA = signF64UI( uiA );
    expA  = expF64UI( uiA );
    sigA  = fracF64UI( uiA );
    uiB = b.v;
    signB = signF64UI( uiB );
    expB  = expF64UI( uiB );
    sigB  = fracF64UI( uiB );
    signZ = signA ^ signB;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA == 0x7FF ) {
        if ( sigA ) goto propagateNaN;
        if ( expB == 0x7FF ) {
            if ( sigB ) goto propagateNaN;
            goto invalid;
        }
        goto infinity;
    }
    if ( expB == 0x7FF ) {
        if ( sigB ) goto propagateNaN;
        goto zero;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! expB ) {
        if ( ! sigB ) {
            if ( ! (expA | sigA) ) goto invalid;
            raiseFlags( flag_infinite );
            goto infinity;
        }
        normExpSig = softfloat_normSubnormalF64Sig( sigB );
        expB = normExpSig.exp;
        sigB = normExpSig.sig;
    }
    if ( ! expA ) {
        if ( ! sigA ) goto zero;
        normExpSig = softfloat_normSubnormalF64Sig( sigA );
        expA = normExpSig.exp;
        sigA = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expZ = expA - expB + 0x3FE;
    sigA |= UINT64_C( 0x0010000000000000 );
    sigB |= UINT64_C( 0x0010000000000000 );
    if ( sigA < sigB ) {
        --expZ;
        sigA <<= 11;
    } else {
        sigA <<= 10;
    }
    sigB <<= 11;
    recip32 = softfloat_approxRecip32_1( sigB>>32 ) - 2;
    sig32Z = ((uint32_t) (sigA>>32) * (uint_fast64_t) recip32)>>32;
    doubleTerm = sig32Z<<1;
    rem =
        ((sigA - (uint_fast64_t) doubleTerm * (uint32_t) (sigB>>32))<<28)
            - (uint_fast64_t) doubleTerm * ((uint32_t) sigB>>4);
    q = (((uint32_t) (rem>>32) * (uint_fast64_t) recip32)>>32) + 4;
    sigZ = ((uint_fast64_t) sig32Z<<32) + ((uint_fast64_t) q<<4);
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( (sigZ & 0x1FF) < 4<<4 ) {
        q &= ~7;
        sigZ &= ~(uint_fast64_t) 0x7F;
        doubleTerm = q<<1;
        rem =
            ((rem - (uint_fast64_t) doubleTerm * (uint32_t) (sigB>>32))<<28)
                - (uint_fast64_t) doubleTerm * ((uint32_t) sigB>>4);
        if ( rem & UINT64_C( 0x8000000000000000 ) ) {
            sigZ -= 1<<7;
        } else {
            if ( rem ) sigZ |= 1;
        }
    }
    return softfloat_roundPackToF64( signZ, expZ, sigZ );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 propagateNaN:
    uiZ = softfloat_propagateNaNF64UI( uiA, uiB );
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 invalid:
    raiseFlags( flag_invalid );
    uiZ = defaultNaNF64UI;
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 infinity:
    uiZ = packToF64UI( signZ, 0x7FF, 0 );
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 zero:
    uiZ = packToF64UI( signZ, 0, 0 );
 uiZ:
    return float64_t::fromRaw(uiZ);
}

static bool f64_eq( float64_t a, float64_t b )
{
    uint_fast64_t uiA;
    uint_fast64_t uiB;

    uiA = a.v;
    uiB = b.v;
    if ( isNaNF64UI( uiA ) || isNaNF64UI( uiB ) )
    {
        if ( softfloat_isSigNaNF64UI( uiA ) || softfloat_isSigNaNF64UI( uiB ) )
            raiseFlags( flag_invalid );
        return false;
    }
    return (uiA == uiB) || ! ((uiA | uiB) & UINT64_C( 0x7FFFFFFFFFFFFFFF ));
}

static bool f64_le( float64_t a, float64_t b )
{
    uint_fast64_t uiA;
    uint_fast64_t uiB;
    bool signA, signB;

    uiA = a.v;
    uiB = b.v;
    if ( isNaNF64UI( uiA ) || isNaNF64UI( uiB ) ) {
        raiseFlags( flag_invalid );
        return false;
    }
    signA = signF64UI( uiA );
    signB = signF64UI( uiB );
    return (signA != signB) ? signA || ! ((uiA | uiB) & UINT64_C( 0x7FFFFFFFFFFFFFFF ))
                            : (uiA == uiB) || (signA ^ (uiA < uiB));
}

static bool f64_lt( float64_t a, float64_t b )
{
    uint_fast64_t uiA;
    uint_fast64_t uiB;
    bool signA, signB;

    uiA = a.v;
    uiB = b.v;
    if ( isNaNF64UI( uiA ) || isNaNF64UI( uiB ) ) {
        raiseFlags( flag_invalid );
        return false;
    }
    signA = signF64UI( uiA );
    signB = signF64UI( uiB );
    return (signA != signB) ? signA && ((uiA | uiB) & UINT64_C( 0x7FFFFFFFFFFFFFFF ))
                            : (uiA != uiB) && (signA ^ (uiA < uiB));
}

static float64_t f64_mulAdd( float64_t a, float64_t b, float64_t c )
{
    uint_fast64_t uiA;
    uint_fast64_t uiB;
    uint_fast64_t uiC;

    uiA = a.v;
    uiB = b.v;
    uiC = c.v;
    return softfloat_mulAddF64( uiA, uiB, uiC, 0 );
}

static float64_t f64_mul( float64_t a, float64_t b )
{
    uint_fast64_t uiA;
    bool signA;
    int_fast16_t expA;
    uint_fast64_t sigA;
    uint_fast64_t uiB;
    bool signB;
    int_fast16_t expB;
    uint_fast64_t sigB;
    bool signZ;
    uint_fast64_t magBits;
    struct exp16_sig64 normExpSig;
    int_fast16_t expZ;
    struct uint128 sig128Z;
    uint_fast64_t sigZ, uiZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    signA = signF64UI( uiA );
    expA  = expF64UI( uiA );
    sigA  = fracF64UI( uiA );
    uiB = b.v;
    signB = signF64UI( uiB );
    expB  = expF64UI( uiB );
    sigB  = fracF64UI( uiB );
    signZ = signA ^ signB;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA == 0x7FF ) {
        if ( sigA || ((expB == 0x7FF) && sigB) ) goto propagateNaN;
        magBits = expB | sigB;
        goto infArg;
    }
    if ( expB == 0x7FF ) {
        if ( sigB ) goto propagateNaN;
        magBits = expA | sigA;
        goto infArg;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! expA ) {
        if ( ! sigA ) goto zero;
        normExpSig = softfloat_normSubnormalF64Sig( sigA );
        expA = normExpSig.exp;
        sigA = normExpSig.sig;
    }
    if ( ! expB ) {
        if ( ! sigB ) goto zero;
        normExpSig = softfloat_normSubnormalF64Sig( sigB );
        expB = normExpSig.exp;
        sigB = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expZ = expA + expB - 0x3FF;
    sigA = (sigA | UINT64_C( 0x0010000000000000 ))<<10;
    sigB = (sigB | UINT64_C( 0x0010000000000000 ))<<11;
    sig128Z = softfloat_mul64To128( sigA, sigB );
    sigZ = sig128Z.v64 | (sig128Z.v0 != 0);

    if ( sigZ < UINT64_C( 0x4000000000000000 ) ) {
        --expZ;
        sigZ <<= 1;
    }
    return softfloat_roundPackToF64( signZ, expZ, sigZ );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 propagateNaN:
    uiZ = softfloat_propagateNaNF64UI( uiA, uiB );
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 infArg:
    if ( ! magBits ) {
        raiseFlags( flag_invalid );
        uiZ = defaultNaNF64UI;
    } else {
        uiZ = packToF64UI( signZ, 0x7FF, 0 );
    }
    goto uiZ;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 zero:
    uiZ = packToF64UI( signZ, 0, 0 );
 uiZ:
    return float64_t::fromRaw(uiZ);
}

static float64_t f64_rem( float64_t a, float64_t b )
{
    uint_fast64_t uiA;
    bool signA;
    int_fast16_t expA;
    uint_fast64_t sigA;
    uint_fast64_t uiB;
    int_fast16_t expB;
    uint_fast64_t sigB;
    struct exp16_sig64 normExpSig;
    uint64_t rem;
    int_fast16_t expDiff;
    uint32_t q, recip32;
    uint_fast64_t q64;
    uint64_t altRem, meanRem;
    bool signRem;
    uint_fast64_t uiZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    signA = signF64UI( uiA );
    expA  = expF64UI( uiA );
    sigA  = fracF64UI( uiA );
    uiB = b.v;
    expB = expF64UI( uiB );
    sigB = fracF64UI( uiB );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA == 0x7FF ) {
        if ( sigA || ((expB == 0x7FF) && sigB) ) goto propagateNaN;
        goto invalid;
    }
    if ( expB == 0x7FF ) {
        if ( sigB ) goto propagateNaN;
        return a;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA < expB - 1 ) return a;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! expB ) {
        if ( ! sigB ) goto invalid;
        normExpSig = softfloat_normSubnormalF64Sig( sigB );
        expB = normExpSig.exp;
        sigB = normExpSig.sig;
    }
    if ( ! expA ) {
        if ( ! sigA ) return a;
        normExpSig = softfloat_normSubnormalF64Sig( sigA );
        expA = normExpSig.exp;
        sigA = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    rem = sigA | UINT64_C( 0x0010000000000000 );
    sigB |= UINT64_C( 0x0010000000000000 );
    expDiff = expA - expB;
    if ( expDiff < 1 ) {
        if ( expDiff < -1 ) return a;
        sigB <<= 9;
        if ( expDiff ) {
            rem <<= 8;
            q = 0;
        } else {
            rem <<= 9;
            q = (sigB <= rem);
            if ( q ) rem -= sigB;
        }
    } else {
        recip32 = softfloat_approxRecip32_1( sigB>>21 );
        /*--------------------------------------------------------------------
        | Changing the shift of `rem' here requires also changing the initial
        | subtraction from `expDiff'.
        *--------------------------------------------------------------------*/
        rem <<= 9;
        expDiff -= 30;
        /*--------------------------------------------------------------------
        | The scale of `sigB' affects how many bits are obtained during each
        | cycle of the loop.  Currently this is 29 bits per loop iteration,
        | the maximum possible.
        *--------------------------------------------------------------------*/
        sigB <<= 9;
        for (;;) {
            q64 = (uint32_t) (rem>>32) * (uint_fast64_t) recip32;
            if ( expDiff < 0 ) break;
            q = (q64 + 0x80000000)>>32;
            rem <<= 29;
            rem -= q * (uint64_t) sigB;
            if ( rem & UINT64_C( 0x8000000000000000 ) ) rem += sigB;
            expDiff -= 29;
        }
        /*--------------------------------------------------------------------
        | (`expDiff' cannot be less than -29 here.)
        *--------------------------------------------------------------------*/
        q = (uint32_t) (q64>>32)>>(~expDiff & 31);
        rem = (rem<<(expDiff + 30)) - q * (uint64_t) sigB;
        if ( rem & UINT64_C( 0x8000000000000000 ) ) {
            altRem = rem + sigB;
            goto selectRem;
        }
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    do {
        altRem = rem;
        ++q;
        rem -= sigB;
    } while ( ! (rem & UINT64_C( 0x8000000000000000 )) );
 selectRem:
    meanRem = rem + altRem;
    if (
        (meanRem & UINT64_C( 0x8000000000000000 )) || (! meanRem && (q & 1))
    ) {
        rem = altRem;
    }
    signRem = signA;
    if ( rem & UINT64_C( 0x8000000000000000 ) ) {
        signRem = ! signRem;
        //fixed unsigned unary minus: -x == ~x + 1
        rem = ~rem + 1;
    }
    return softfloat_normRoundPackToF64( signRem, expB, rem );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 propagateNaN:
    uiZ = softfloat_propagateNaNF64UI( uiA, uiB );
    goto uiZ;
 invalid:
    raiseFlags( flag_invalid );
    uiZ = defaultNaNF64UI;
 uiZ:
    return float64_t::fromRaw(uiZ);
}

static float64_t f64_roundToInt( float64_t a, uint_fast8_t roundingMode, bool exact )
{
    uint_fast64_t uiA;
    int_fast16_t exp;
    uint_fast64_t uiZ, lastBitMask, roundBitsMask;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    exp = expF64UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( exp <= 0x3FE ) {
        if ( ! (uiA & UINT64_C( 0x7FFFFFFFFFFFFFFF )) ) return a;
        if ( exact ) raiseFlags(flag_inexact);
        uiZ = uiA & packToF64UI( 1, 0, 0 );
        switch ( roundingMode ) {
         case round_near_even:
            if ( ! fracF64UI( uiA ) ) break;
            /* fallthrough */
         case round_near_maxMag:
            if ( exp == 0x3FE ) uiZ |= packToF64UI( 0, 0x3FF, 0 );
            break;
         case round_min:
            if ( uiZ ) uiZ = packToF64UI( 1, 0x3FF, 0 );
            break;
         case round_max:
            if ( ! uiZ ) uiZ = packToF64UI( 0, 0x3FF, 0 );
            break;
        }
        goto uiZ;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( 0x433 <= exp ) {
        if ( (exp == 0x7FF) && fracF64UI( uiA ) ) {
            uiZ = softfloat_propagateNaNF64UI( uiA, 0 );
            goto uiZ;
        }
        return a;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiZ = uiA;
    lastBitMask = (uint_fast64_t) 1<<(0x433 - exp);
    roundBitsMask = lastBitMask - 1;
    if ( roundingMode == round_near_maxMag ) {
        uiZ += lastBitMask>>1;
    } else if ( roundingMode == round_near_even ) {
        uiZ += lastBitMask>>1;
        if ( ! (uiZ & roundBitsMask) ) uiZ &= ~lastBitMask;
    } else if (
        roundingMode
            == (signF64UI( uiZ ) ? round_min : round_max)
    ) {
        uiZ += roundBitsMask;
    }
    uiZ &= ~roundBitsMask;
    if ( exact && (uiZ != uiA) ) {
        raiseFlags(flag_inexact);
    }
 uiZ:
    return float64_t::fromRaw(uiZ);
}

static float64_t f64_sqrt( float64_t a )
{
    uint_fast64_t uiA;
    bool signA;
    int_fast16_t expA;
    uint_fast64_t sigA, uiZ;
    struct exp16_sig64 normExpSig;
    int_fast16_t expZ;
    uint32_t sig32A, recipSqrt32, sig32Z;
    uint_fast64_t rem;
    uint32_t q;
    uint_fast64_t sigZ, shiftedSigZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    signA = signF64UI( uiA );
    expA  = expF64UI( uiA );
    sigA  = fracF64UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA == 0x7FF ) {
        if ( sigA ) {
            uiZ = softfloat_propagateNaNF64UI( uiA, 0 );
            goto uiZ;
        }
        if ( ! signA ) return a;
        goto invalid;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( signA ) {
        if ( ! (expA | sigA) ) return a;
        goto invalid;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! expA ) {
        if ( ! sigA ) return a;
        normExpSig = softfloat_normSubnormalF64Sig( sigA );
        expA = normExpSig.exp;
        sigA = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    | (`sig32Z' is guaranteed to be a lower bound on the square root of
    | `sig32A', which makes `sig32Z' also a lower bound on the square root of
    | `sigA'.)
    *------------------------------------------------------------------------*/
    expZ = ((expA - 0x3FF)>>1) + 0x3FE;
    expA &= 1;
    sigA |= UINT64_C( 0x0010000000000000 );
    sig32A = (uint32_t)(sigA>>21); //fixed warning on type cast
    recipSqrt32 = softfloat_approxRecipSqrt32_1( expA, sig32A );
    sig32Z = ((uint_fast64_t) sig32A * recipSqrt32)>>32;
    if ( expA ) {
        sigA <<= 8;
        sig32Z >>= 1;
    } else {
        sigA <<= 9;
    }
    rem = sigA - (uint_fast64_t) sig32Z * sig32Z;
    q = ((uint32_t) (rem>>2) * (uint_fast64_t) recipSqrt32)>>32;
    sigZ = ((uint_fast64_t) sig32Z<<32 | 1<<5) + ((uint_fast64_t) q<<3);
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( (sigZ & 0x1FF) < 0x22 ) {
        sigZ &= ~(uint_fast64_t) 0x3F;
        shiftedSigZ = sigZ>>6;
        rem = (sigA<<52) - shiftedSigZ * shiftedSigZ;
        if ( rem & UINT64_C( 0x8000000000000000 ) ) {
            --sigZ;
        } else {
            if ( rem ) sigZ |= 1;
        }
    }
    return softfloat_roundPackToF64( 0, expZ, sigZ );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 invalid:
    raiseFlags( flag_invalid );
    uiZ = defaultNaNF64UI;
 uiZ:
    return float64_t::fromRaw(uiZ);
}

static float64_t f64_sub( float64_t a, float64_t b )
{
    uint_fast64_t uiA;
    bool signA;
    uint_fast64_t uiB;
    bool signB;

    uiA = a.v;
    signA = signF64UI( uiA );
    uiB = b.v;
    signB = signF64UI( uiB );

    if ( signA == signB ) {
        return softfloat_subMagsF64( uiA, uiB, signA );
    } else {
        return softfloat_addMagsF64( uiA, uiB, signA );
    }
}

static float32_t f64_to_f32( float64_t a )
{
    uint_fast64_t uiA;
    bool sign;
    int_fast16_t exp;
    uint_fast64_t frac;
    struct commonNaN commonNaN;
    uint_fast32_t uiZ, frac32;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    sign = signF64UI( uiA );
    exp  = expF64UI( uiA );
    frac = fracF64UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( exp == 0x7FF ) {
        if ( frac ) {
            softfloat_f64UIToCommonNaN( uiA, &commonNaN );
            uiZ = softfloat_commonNaNToF32UI( &commonNaN );
        } else {
            uiZ = packToF32UI( sign, 0xFF, 0 );
        }
        goto uiZ;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    frac32 = (uint_fast32_t)softfloat_shortShiftRightJam64( frac, 22 ); //fixed warning on type cast
    if ( ! (exp | frac32) ) {
        uiZ = packToF32UI( sign, 0, 0 );
        goto uiZ;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    return softfloat_roundPackToF32( sign, exp - 0x381, frac32 | 0x40000000 );
 uiZ:
    return float32_t::fromRaw(uiZ);
}

static int_fast32_t f64_to_i32( float64_t a, uint_fast8_t roundingMode, bool exact )
{
    uint_fast64_t uiA;
    bool sign;
    int_fast16_t exp;
    uint_fast64_t sig;
    int_fast16_t shiftDist;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    sign = signF64UI( uiA );
    exp  = expF64UI( uiA );
    sig  = fracF64UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
#if (i32_fromNaN != i32_fromPosOverflow) || (i32_fromNaN != i32_fromNegOverflow)
    if ( (exp == 0x7FF) && sig ) {
#if (i32_fromNaN == i32_fromPosOverflow)
        sign = 0;
#elif (i32_fromNaN == i32_fromNegOverflow)
        sign = 1;
#else
        raiseFlags( flag_invalid );
        return i32_fromNaN;
#endif
    }
#endif
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( exp ) sig |= UINT64_C( 0x0010000000000000 );
    shiftDist = 0x427 - exp;
    if ( 0 < shiftDist ) sig = softfloat_shiftRightJam64( sig, shiftDist );
    return softfloat_roundToI32( sign, sig, roundingMode, exact );
}

static int_fast64_t f64_to_i64(float64_t a, uint_fast8_t roundingMode, bool exact )
{
    uint_fast64_t uiA;
    bool sign;
    int_fast16_t exp;
    uint_fast64_t sig;
    int_fast16_t shiftDist;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    sign = signF64UI(uiA);
    exp = expF64UI(uiA);
    sig = fracF64UI(uiA);
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
#if (i64_fromNaN != i64_fromPosOverflow) || (i64_fromNaN != i64_fromNegOverflow)
    if ((exp == 0x7FF) && sig) {
#if (i64_fromNaN == i64_fromPosOverflow)
        sign = 0;
#elif (i64_fromNaN == i64_fromNegOverflow)
        sign = 1;
#else
        raiseFlags(flag_invalid);
        return i64_fromNaN;
#endif
    }
#endif
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if (exp) sig |= UINT64_C(0x0010000000000000);
    shiftDist = 0x433 - exp;
    if (shiftDist <= 0) {
        bool isValid = shiftDist >= -11;
        if (isValid)
        {
            uint_fast64_t z = sig << -shiftDist;
            if (0 == (z & UINT64_C(0x8000000000000000)))
            {
                return sign ? -(int_fast64_t)z : (int_fast64_t)z;
            }
        }
        raiseFlags(flag_invalid);
        return sign ? i64_fromNegOverflow : i64_fromPosOverflow;
    }
    else {
        if (shiftDist < 64)
            return
                softfloat_roundToI64(
                    sign, sig >> shiftDist, sig << (-shiftDist & 63), roundingMode, exact);
        else
            return
                softfloat_roundToI64(
                    sign, 0, (shiftDist == 64) ? sig : (sig != 0), roundingMode, exact);
    }
}

static int_fast32_t f64_to_i32_r_minMag( float64_t a, bool exact )
{
    uint_fast64_t uiA;
    int_fast16_t exp;
    uint_fast64_t sig;
    int_fast16_t shiftDist;
    bool sign;
    int_fast32_t absZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    uiA = a.v;
    exp = expF64UI( uiA );
    sig = fracF64UI( uiA );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    shiftDist = 0x433 - exp;
    if ( 53 <= shiftDist ) {
        if ( exact && (exp | sig) ) {
            raiseFlags(flag_inexact);
        }
        return 0;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    sign = signF64UI( uiA );
    if ( shiftDist < 22 ) {
        if (
            sign && (exp == 0x41E) && (sig < UINT64_C( 0x0000000000200000 ))
        ) {
            if ( exact && sig ) {
                raiseFlags(flag_inexact);
            }
            return -0x7FFFFFFF - 1;
        }
        raiseFlags( flag_invalid );
        return
            (exp == 0x7FF) && sig ? i32_fromNaN
                : sign ? i32_fromNegOverflow : i32_fromPosOverflow;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    sig |= UINT64_C( 0x0010000000000000 );
    absZ = (int_fast32_t)(sig>>shiftDist); //fixed warning on type cast
    if ( exact && ((uint_fast64_t) (uint_fast32_t) absZ<<shiftDist != sig) ) {
        raiseFlags(flag_inexact);
    }
    return sign ? -absZ : absZ;
}

static float32_t i32_to_f32( int32_t a )
{
    bool sign;
    uint_fast32_t absA;

    sign = (a < 0);
    if ( ! (a & 0x7FFFFFFF) ) {
        return float32_t::fromRaw(sign ? packToF32UI( 1, 0x9E, 0 ) : 0);
    }
    //fixed unsigned unary minus: -x == ~x + 1
    absA = sign ? (~(uint_fast32_t) a + 1) : (uint_fast32_t) a;
    return softfloat_normRoundPackToF32( sign, 0x9C, absA );
}

static float64_t i32_to_f64( int32_t a )
{
    uint_fast64_t uiZ;
    bool sign;
    uint_fast32_t absA;
    int_fast8_t shiftDist;

    if ( ! a ) {
        uiZ = 0;
    } else {
        sign = (a < 0);
        //fixed unsigned unary minus: -x == ~x + 1
        absA = sign ? (~(uint_fast32_t) a + 1) : (uint_fast32_t) a;
        shiftDist = softfloat_countLeadingZeros32( absA ) + 21;
        uiZ =
            packToF64UI(
                sign, 0x432 - shiftDist, (uint_fast64_t) absA<<shiftDist );
    }
    return float64_t::fromRaw(uiZ);
}

static float32_t i64_to_f32( int64_t a )
{
    bool sign;
    uint_fast64_t absA;
    int_fast8_t shiftDist;
    uint_fast32_t sig;

    sign = (a < 0);
    //fixed unsigned unary minus: -x == ~x + 1
    absA = sign ? (~(uint_fast64_t) a + 1) : (uint_fast64_t) a;
    shiftDist = softfloat_countLeadingZeros64( absA ) - 40;
    if ( 0 <= shiftDist ) {
        return float32_t::fromRaw(a ? packToF32UI(sign, 0x95 - shiftDist, (uint_fast32_t) absA<<shiftDist ) : 0);
    } else {
        shiftDist += 7;
        sig =
            (shiftDist < 0)
                ? (uint_fast32_t) softfloat_shortShiftRightJam64( absA, -shiftDist ) //fixed warning on type cast
                : (uint_fast32_t) absA<<shiftDist;
        return softfloat_roundPackToF32( sign, 0x9C - shiftDist, sig );
    }
}

static float64_t i64_to_f64( int64_t a )
{
    bool sign;
    uint_fast64_t absA;

    sign = (a < 0);
    if ( ! (a & UINT64_C( 0x7FFFFFFFFFFFFFFF )) ) {
        return float64_t::fromRaw(sign ? packToF64UI( 1, 0x43E, 0 ) : 0);
    }
    //fixed unsigned unary minus: -x == ~x + 1
    absA = sign ? (~(uint_fast64_t) a + 1) : (uint_fast64_t) a;
    return softfloat_normRoundPackToF64( sign, 0x43C, absA );
}

static float32_t softfloat_addMagsF32( uint_fast32_t uiA, uint_fast32_t uiB )
{
    int_fast16_t expA;
    uint_fast32_t sigA;
    int_fast16_t expB;
    uint_fast32_t sigB;
    int_fast16_t expDiff;
    uint_fast32_t uiZ;
    bool signZ;
    int_fast16_t expZ;
    uint_fast32_t sigZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expA = expF32UI( uiA );
    sigA = fracF32UI( uiA );
    expB = expF32UI( uiB );
    sigB = fracF32UI( uiB );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expDiff = expA - expB;
    if ( ! expDiff ) {
        /*--------------------------------------------------------------------
        *--------------------------------------------------------------------*/
        if ( ! expA ) {
            uiZ = uiA + sigB;
            goto uiZ;
        }
        if ( expA == 0xFF ) {
            if ( sigA | sigB ) goto propagateNaN;
            uiZ = uiA;
            goto uiZ;
        }
        signZ = signF32UI( uiA );
        expZ = expA;
        sigZ = 0x01000000 + sigA + sigB;
        if ( ! (sigZ & 1) && (expZ < 0xFE) ) {
            uiZ = packToF32UI( signZ, expZ, sigZ>>1 );
            goto uiZ;
        }
        sigZ <<= 6;
    } else {
        /*--------------------------------------------------------------------
        *--------------------------------------------------------------------*/
        signZ = signF32UI( uiA );
        sigA <<= 6;
        sigB <<= 6;
        if ( expDiff < 0 ) {
            if ( expB == 0xFF ) {
                if ( sigB ) goto propagateNaN;
                uiZ = packToF32UI( signZ, 0xFF, 0 );
                goto uiZ;
            }
            expZ = expB;
            sigA += expA ? 0x20000000 : sigA;
            sigA = softfloat_shiftRightJam32( sigA, -expDiff );
        } else {
            if ( expA == 0xFF ) {
                if ( sigA ) goto propagateNaN;
                uiZ = uiA;
                goto uiZ;
            }
            expZ = expA;
            sigB += expB ? 0x20000000 : sigB;
            sigB = softfloat_shiftRightJam32( sigB, expDiff );
        }
        sigZ = 0x20000000 + sigA + sigB;
        if ( sigZ < 0x40000000 ) {
            --expZ;
            sigZ <<= 1;
        }
    }
    return softfloat_roundPackToF32( signZ, expZ, sigZ );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 propagateNaN:
    uiZ = softfloat_propagateNaNF32UI( uiA, uiB );
 uiZ:
    return float32_t::fromRaw(uiZ);
}

static float64_t
 softfloat_addMagsF64( uint_fast64_t uiA, uint_fast64_t uiB, bool signZ )
{
    int_fast16_t expA;
    uint_fast64_t sigA;
    int_fast16_t expB;
    uint_fast64_t sigB;
    int_fast16_t expDiff;
    uint_fast64_t uiZ;
    int_fast16_t expZ;
    uint_fast64_t sigZ;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expA = expF64UI( uiA );
    sigA = fracF64UI( uiA );
    expB = expF64UI( uiB );
    sigB = fracF64UI( uiB );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expDiff = expA - expB;
    if ( ! expDiff ) {
        /*--------------------------------------------------------------------
        *--------------------------------------------------------------------*/
        if ( ! expA ) {
            uiZ = uiA + sigB;
            goto uiZ;
        }
        if ( expA == 0x7FF ) {
            if ( sigA | sigB ) goto propagateNaN;
            uiZ = uiA;
            goto uiZ;
        }
        expZ = expA;
        sigZ = UINT64_C( 0x0020000000000000 ) + sigA + sigB;
        sigZ <<= 9;
    } else {
        /*--------------------------------------------------------------------
        *--------------------------------------------------------------------*/
        sigA <<= 9;
        sigB <<= 9;
        if ( expDiff < 0 ) {
            if ( expB == 0x7FF ) {
                if ( sigB ) goto propagateNaN;
                uiZ = packToF64UI( signZ, 0x7FF, 0 );
                goto uiZ;
            }
            expZ = expB;
            if ( expA ) {
                sigA += UINT64_C( 0x2000000000000000 );
            } else {
                sigA <<= 1;
            }
            sigA = softfloat_shiftRightJam64( sigA, -expDiff );
        } else {
            if ( expA == 0x7FF ) {
                if ( sigA ) goto propagateNaN;
                uiZ = uiA;
                goto uiZ;
            }
            expZ = expA;
            if ( expB ) {
                sigB += UINT64_C( 0x2000000000000000 );
            } else {
                sigB <<= 1;
            }
            sigB = softfloat_shiftRightJam64( sigB, expDiff );
        }
        sigZ = UINT64_C( 0x2000000000000000 ) + sigA + sigB;
        if ( sigZ < UINT64_C( 0x4000000000000000 ) ) {
            --expZ;
            sigZ <<= 1;
        }
    }
    return softfloat_roundPackToF64( signZ, expZ, sigZ );
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
 propagateNaN:
    uiZ = softfloat_propagateNaNF64UI( uiA, uiB );
 uiZ:
    return float64_t::fromRaw(uiZ);
}

static uint32_t softfloat_approxRecipSqrt32_1( unsigned int oddExpA, uint32_t a )
{
    int index;
    uint16_t eps, r0;
    uint_fast32_t ESqrR0;
    uint32_t sigma0;
    uint_fast32_t r;
    uint32_t sqrSigma0;

    index = (a>>27 & 0xE) + oddExpA;
    eps = (uint16_t) (a>>12);
    r0 = softfloat_approxRecipSqrt_1k0s[index]
             - ((softfloat_approxRecipSqrt_1k1s[index] * (uint_fast32_t) eps)
                    >>20);
    ESqrR0 = (uint_fast32_t) r0 * r0;
    if ( ! oddExpA ) ESqrR0 <<= 1;
    sigma0 = ~(uint_fast32_t) (((uint32_t) ESqrR0 * (uint_fast64_t) a)>>23);
    r = (uint_fast32_t)(((uint_fast32_t) r0<<16) + ((r0 * (uint_fast64_t) sigma0)>>25)); //fixed warning on type cast
    sqrSigma0 = ((uint_fast64_t) sigma0 * sigma0)>>32;
    r += ((uint32_t) ((r>>1) + (r>>3) - ((uint_fast32_t) r0<<14))
              * (uint_fast64_t) sqrSigma0)
             >>48;
    if ( ! (r & 0x80000000) ) r = 0x80000000;
    return r;
}

/*----------------------------------------------------------------------------
| Converts the common NaN pointed to by `aPtr' into a 32-bit floating-point
| NaN, and returns the bit pattern of this value as an unsigned integer.
*----------------------------------------------------------------------------*/
static uint_fast32_t softfloat_commonNaNToF32UI( const struct commonNaN *aPtr )
{
    return (uint_fast32_t) aPtr->sign<<31 | 0x7FC00000 | aPtr->v64>>41;
}

/*----------------------------------------------------------------------------
| Converts the common NaN pointed to by `aPtr' into a 64-bit floating-point
| NaN, and returns the bit pattern of this value as an unsigned integer.
*----------------------------------------------------------------------------*/
static uint_fast64_t softfloat_commonNaNToF64UI( const struct commonNaN *aPtr )
{
    return
        (uint_fast64_t) aPtr->sign<<63 | UINT64_C( 0x7FF8000000000000 )
            | aPtr->v64>>12;
}

static uint_fast8_t softfloat_countLeadingZeros64( uint64_t a )
{
    uint_fast8_t count;
    uint32_t a32;

    count = 0;
    a32 = a>>32;
    if ( ! a32 ) {
        count = 32;
        a32 = (uint32_t) a; //fixed warning on type cast
    }
    /*------------------------------------------------------------------------
    | From here, result is current count + count leading zeros of `a32'.
    *------------------------------------------------------------------------*/
    if ( a32 < 0x10000 ) {
        count += 16;
        a32 <<= 16;
    }
    if ( a32 < 0x1000000 ) {
        count += 8;
        a32 <<= 8;
    }
    count += softfloat_countLeadingZeros8[a32>>24];
    return count;
}

/*----------------------------------------------------------------------------
| Assuming `uiA' has the bit pattern of a 32-bit floating-point NaN, converts
| this NaN to the common NaN form, and stores the resulting common NaN at the
| location pointed to by `zPtr'.  If the NaN is a signaling NaN, the invalid
| exception is raised.
*----------------------------------------------------------------------------*/
static void softfloat_f32UIToCommonNaN( uint_fast32_t uiA, struct commonNaN *zPtr )
{
    if ( softfloat_isSigNaNF32UI( uiA ) ) {
        raiseFlags( flag_invalid );
    }
    zPtr->sign = (uiA>>31) != 0;
    zPtr->v64  = (uint_fast64_t) uiA<<41;
    zPtr->v0   = 0;
}

/*----------------------------------------------------------------------------
| Assuming `uiA' has the bit pattern of a 64-bit floating-point NaN, converts
| this NaN to the common NaN form, and stores the resulting common NaN at the
| location pointed to by `zPtr'.  If the NaN is a signaling NaN, the invalid
| exception is raised.
*----------------------------------------------------------------------------*/
static void softfloat_f64UIToCommonNaN( uint_fast64_t uiA, struct commonNaN *zPtr )
{
    if ( softfloat_isSigNaNF64UI( uiA ) ) {
        raiseFlags( flag_invalid );
    }
    zPtr->sign = (uiA>>63) != 0;
    zPtr->v64  = uiA<<12;
    zPtr->v0   = 0;
}

static struct uint128 softfloat_mul64To128( uint64_t a, uint64_t b )
{
    uint32_t a32, a0, b32, b0;
    struct uint128 z;
    uint64_t mid1, mid;

    a32 = a>>32;
    a0 = (uint32_t)a; //fixed warning on type cast
    b32 = b>>32;
    b0 = (uint32_t) b; //fixed warning on type cast
    z.v0 = (uint_fast64_t) a0 * b0;
    mid1 = (uint_fast64_t) a32 * b0;
    mid = mid1 + (uint_fast64_t) a0 * b32;
    z.v64 = (uint_fast64_t) a32 * b32;
    z.v64 += (uint_fast64_t) (mid < mid1)<<32 | mid>>32;
    mid <<= 32;
    z.v0 += mid;
    z.v64 += (z.v0 < mid);
    return z;
}

static float32_t
 softfloat_mulAddF32(
     uint_fast32_t uiA, uint_fast32_t uiB, uint_fast32_t uiC, uint_fast8_t op )
{
    bool signA;
    int_fast16_t expA;
    uint_fast32_t sigA;
    bool signB;
    int_fast16_t expB;
    uint_fast32_t sigB;
    bool signC;
    int_fast16_t expC;
    uint_fast32_t sigC;
    bool signProd;
    uint_fast32_t magBits, uiZ;
    struct exp16_sig32 normExpSig;
    int_fast16_t expProd;
    uint_fast64_t sigProd;
    bool signZ;
    int_fast16_t expZ;
    uint_fast32_t sigZ;
    int_fast16_t expDiff;
    uint_fast64_t sig64Z, sig64C;
    int_fast8_t shiftDist;

    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    signA = signF32UI( uiA );
    expA  = expF32UI( uiA );
    sigA  = fracF32UI( uiA );
    signB = signF32UI( uiB );
    expB  = expF32UI( uiB );
    sigB  = fracF32UI( uiB );
    signC = signF32UI( uiC ) ^ (op == softfloat_mulAdd_subC);
    expC  = expF32UI( uiC );
    sigC  = fracF32UI( uiC );
    signProd = signA ^ signB ^ (op == softfloat_mulAdd_subProd);
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( expA == 0xFF ) {
        if ( sigA || ((expB == 0xFF) && sigB) ) goto propagateNaN_ABC;
        magBits = expB | sigB;
        goto infProdArg;
    }
    if ( expB == 0xFF ) {
        if ( sigB ) goto propagateNaN_ABC;
        magBits = expA | sigA;
        goto infProdArg;
    }
    if ( expC == 0xFF ) {
        if ( sigC ) {
            uiZ = 0;
            goto propagateNaN_ZC;
        }
        uiZ = uiC;
        goto uiZ;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    if ( ! expA ) {
        if ( ! sigA ) goto zeroProd;
        normExpSig = softfloat_normSubnormalF32Sig( sigA );
        expA = normExpSig.exp;
        sigA = normExpSig.sig;
    }
    if ( ! expB ) {
        if ( ! sigB ) goto zeroProd;
        normExpSig = softfloat_normSubnormalF32Sig( sigB );
        expB = normExpSig.exp;
        sigB = normExpSig.sig;
    }
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expProd = expA + expB - 0x7E;
    sigA = (sigA | 0x00800000)<<7;
    sigB = (sigB | 0x00800000)<<7;
    sigProd = (uint_fast64_t) sigA * sigB;
    if ( sigProd < UINT64_C( 0x2000000000000000 ) ) {
        --expProd;
        sigProd <<= 1;
    }
    signZ = signProd;
    if ( ! expC ) {
        if ( ! sigC ) {
            expZ = expProd - 1;
            sigZ = (uint_fast32_t) softfloat_shortShiftRightJam64( sigProd, 31 ); //fixed warning on type cast
            goto roundPack;
        }
        normExpSig = softfloat_normSubnormalF32Sig( sigC );
        expC = normExpSig.exp;
        sigC = normExpSig.sig;
    }
    sigC = (sigC | 0x00800000)<<6;
    /*------------------------------------------------------------------------
    *------------------------------------------------------------------------*/
    expDiff = expProd - expC;
    if ( signProd == signC ) {
        /*--------------------------------------------------------------------
        *--------------------------------------------------------------------*/
        if ( expDiff <= 0 ) {
            expZ = expC;
            sigZ = sigC + (uint_fast32_t) softfloat_shiftRightJam64( sigProd, 32 - expDiff ); //fixed warning on type cast
        } else {
            expZ = expProd;
            sig64Z =
                sigProd
                    + softfloat_shiftRightJam64(
                          (uint_fast64_t) sigC<<32, expDiff );
            sigZ = (uint_fast32_t) softfloat_shortShiftRightJam64( sig64Z, 32 ); //fixed warning on type cast
        }
        if ( sigZ < 0x40000000 ) {
            --expZ;
            sigZ <<= 1;
        }
    } else {
        /*--------------------------------------------------------------------
        *--------------------------------------------------------------------*/
        sig64C = (uint_fast64_t) sigC<<32;
        if ( expDiff < 0 ) {
            signZ = signC;
            expZ = expC;
            sig64Z = sig64C - softfloat_shiftRightJam64( sigProd, -expDiff );
        } else if ( ! expDiff ) {
            expZ = expProd;
            sig64Z = sigProd - sig64C;
            if ( ! sig64Z ) goto completeCancellation;
            if ( sig64Z & UINT64_C( 0x8000000000000000 ) ) {

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

- **uint128_extra**: A class/struct defined in this file
- **uint128**: A class/struct defined in this file
- **exp16_sig32**: A class/struct defined in this file
- **uint64_extra**: A class/struct defined in this file
- **exp16_sig64**: A class/struct defined in this file
- **commonNaN**: A class/struct defined in this file

### Functions and Methods

- **softfloat()**: A function/method defined in this file
- **EXPTAB_SCALE()**: A function/method defined in this file
- **may()**: A function/method defined in this file
- **EXPTAB_MASK()**: A function/method defined in this file
- **EXPPOLY_32F_A0()**: A function/method defined in this file
- **softdouble()**: A function/method defined in this file
- **WORDS_BIGENDIAN()**: A function/method defined in this file
- **of()**: A function/method defined in this file
- **on()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/softfloat.hpp`
- `precomp.hpp`

**Python Imports:**
- `the`
- `SoftFloat`
- `one`
- `zero`
- `OpenCV.`
- `packages`
- `functions.`
- `this`
- `OpenCV`
- `fdlibm`


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

