# Documentation for `data/haarcascades_cuda/haarcascade_lowerbody.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_lowerbody.xml`
- **File Name**: `haarcascade_lowerbody.xml`
- **File Size**: 531,485 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_lowerbody.xml](../../data/haarcascades_cuda/haarcascade_lowerbody.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
   19x23 lowerbody detector (see the detailed description below).

//////////////////////////////////////////////////////////////////////////
| Contributors License Agreement
| IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
|   By downloading, copying, installing or using the software you agree
|   to this license.
|   If you do not agree to this license, do not download, install,
|   copy or use the software.
|
| Copyright (c) 2004, Hannes Kruppa and Bernt Schiele (ETH Zurich, Switzerland).
|  All rights reserved.
|
| Redistribution and use in source and binary forms, with or without
| modification, are permitted provided that the following conditions are
| met:
|
|    * Redistributions of source code must retain the above copyright
|       notice, this list of conditions and the following disclaimer.
|    * Redistributions in binary form must reproduce the above
|      copyright notice, this list of conditions and the following
|      disclaimer in the documentation and/or other materials provided
|      with the distribution.
|    * The name of Contributor may not used to endorse or promote products
|      derived from this software without specific prior written permission.
|
| THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
| "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
| LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
| A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
| CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
| EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
| PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
| PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
| LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
| NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
| SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.  Back to
| Top
//////////////////////////////////////////////////////////////////////////

"Haar"-based Detectors For Pedestrian Detection
===============================================
by Hannes Kruppa and Bernt Schiele, ETH Zurich, Switzerland

This archive provides the following three detectors:
- upper body detector (most fun, useful in many scenarios!)
- lower body detector
- full body detector

These detectors have been successfully applied to pedestrian detection
in still images. They can be directly passed as parameters to the
program HaarFaceDetect.
NOTE: These detectors deal with frontal and backside views but not
with side views (also see "Known limitations" below).

RESEARCHERS:
If you are using any of the detectors or involved ideas please cite
this paper (available at www.vision.ethz.ch/publications/):

@InProceedings{Kruppa03-bmvc,
  author =       "Hannes Kruppa, Modesto Castrillon-Santana and Bernt Schiele",
  title =        "Fast and Robust Face Finding via Local Context."
  booktitle =    "Joint IEEE International Workshop on Visual Surveillance and Performance Evaluation of Tracking and Surveillance"
  year =         "2003",
  month =        "October"
}

COMMERCIAL:
If you have any commercial interest in this work please contact
hkruppa@inf.ethz.ch


ADDITIONAL INFORMATION
======================
Check out the demo movie, e.g. using mplayer or any (Windows/Linux-) player
that can play back .mpg movies.
Under Linux that's:
> ffplay demo.mpg
or:
> mplayer demo.mpg

The movie shows a person walking towards the camera in a realistic
indoor setting. Using ffplay or mplayer you can pause and continue the
movie by pressing the space bar.

Detections coming from the different detectors are visualized using
different line styles:
upper body : dotted line
lower body : dashed line
full body  : solid line

You will notice that successful detections containing the target do
not sit tightly on the body but also include some of the background
left and right.  This is not a bug but accurately reflects the
employed training data which also includes portions of the background
to ensure proper silhouette representation. If you want to get a
feeling for the training data check out the CBCL data set:
http://www.ai.mit.edu/projects/cbcl/software-datasets/PedestrianData.html

There is also a small number of false alarms in this sequence.
NOTE: This is per frame detection, not tracking (which is also one of
the reasons why it is not mislead by the person's shadow on the back
wall).

On an Intel Xeon 1.7GHz machine the detectors operate at something
between 6Hz to 14 Hz (on 352 x 288 frames per second) depending on the
detector. The detectors work as well on much lower image resolutions
which is always an interesting possibility for speed-ups or
"coarse-to-fine" search strategies.

Additional information e.g. on training parameters, detector
combination, detecting other types of objects (e.g. cars) etc. is
available in my PhD thesis report (available end of June). Check out
www.vision.ethz.ch/kruppa/


KNOWN LIMITATIONS
=================
1) The detectors only support frontal and back views but not sideviews.
   Sideviews are trickier and it makes a lot of sense to include additional
   modalities for their detection, e.g. motion information. I recommend
   Viola and Jones' ICCV 2003 paper if this further interests you.

2) Don't expect these detectors to be as accurate as a frontal face detector.
   A frontal face as a pattern is pretty distinct with respect to other
   patterns occurring in the world (i.e. image "background"). This is not so
   for upper, lower and especially full bodies, because they have to rely
   on fragile silhouette information rather than internal (facial) features.
   Still, we found especially the upper body detector to perform amazingly well.
   In contrast to a face detector these detectors will also work at very low
   image resolutions

Acknowledgements
================
Thanks to Martin Spengler, ETH Zurich, for providing the demo movie.
-->
<opencv_storage>
<haarcascade_lowerbody type_id="opencv-haar-classifier">
  <size>19 23</size>
  <stages>
    <_>
      <!-- stage 0 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 4 12 16 -1.</_>
                <_>7 4 4 16 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0168698690831661</threshold>
            <left_val>0.5465741753578186</left_val>
            <right_val>-0.6367803812026978</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 0 2 20 -1.</_>
                <_>11 10 2 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.5349899660795927e-003</threshold>
            <left_val>-0.3760549128055573</left_val>
            <right_val>0.3237810134887695</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 1 4 22 -1.</_>
                <_>4 12 4 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0247094593942165</threshold>
            <left_val>-0.6797912716865540</left_val>
            <right_val>0.2050105929374695</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 8 7 12 -1.</_>
                <_>9 14 7 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0824368596076965</threshold>
            <left_val>0.2058863937854767</left_val>
            <right_val>-0.8493843078613281</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 0 6 10 -1.</_>
                <_>6 0 3 5 2.</_>
                <_>9 5 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.2128931535407901e-004</threshold>
            <left_val>0.3189192116260529</left_val>
            <right_val>-0.4646945893764496</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 18 18 5 -1.</_>
                <_>1 18 9 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0230169594287872</threshold>
            <left_val>0.1867029964923859</left_val>
            <right_val>-0.7033089995384216</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 20 10 3 -1.</_>
                <_>9 20 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.6386149264872074e-003</threshold>
            <left_val>0.1637049019336700</left_val>
            <right_val>-0.8460472226142883</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 17 10 6 -1.</_>
                <_>6 20 10 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.6682120561599731e-004</threshold>
            <left_val>-0.3985269069671631</left_val>
            <right_val>0.2311332970857620</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 0 4 20 -1.</_>
                <_>0 10 4 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1173167973756790</threshold>
            <left_val>0.1044503971934319</left_val>
            <right_val>-0.8851094245910645</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 0 16 14 -1.</_>
                <_>3 7 16 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0154212303459644</threshold>
            <left_val>-0.2785950899124146</left_val>
            <right_val>0.2892192006111145</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 1 4 13 -1.</_>
                <_>7 1 2 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0340189486742020</threshold>
            <left_val>-0.1428766995668411</left_val>
            <right_val>0.7780153155326843</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 8 18 12 -1.</_>
                <_>10 8 9 6 2.</_>
                <_>1 14 9 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0346388705074787</threshold>
            <left_val>0.1864407956600189</left_val>
            <right_val>-0.6032484173774719</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 0 15 21 -1.</_>
                <_>7 0 5 21 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.3750365972518921</threshold>
            <left_val>0.9278184175491333</left_val>
            <right_val>-0.1542160063982010</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 5 18 18 -1.</_>
                <_>10 5 9 9 2.</_>
                <_>1 14 9 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0560119710862637</threshold>
            <left_val>-0.5859106779098511</left_val>
            <right_val>0.1954751014709473</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 19 15 3 -1.</_>
                <_>7 19 5 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.4878909569233656e-003</threshold>
            <left_val>0.2813934981822968</left_val>
            <right_val>-0.4185301065444946</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 20 12 3 -1.</_>
                <_>7 20 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0144956996664405</threshold>
            <left_val>-0.7227396965026856</left_val>
            <right_val>0.0942884609103203</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 21 14 2 -1.</_>
                <_>8 21 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.6178281083703041e-003</threshold>
            <left_val>-0.5955196022987366</left_val>
            <right_val>0.1520265042781830</right_val></_></_></trees>
      <stage_threshold>-1.4308550357818604</stage_threshold>
      <parent>-1</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 1 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 16 18 6 -1.</_>
                <_>6 16 6 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.1839120201766491e-003</threshold>
            <left_val>0.4002513885498047</left_val>
            <right_val>-0.6847316026687622</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 3 4 20 -1.</_>
                <_>8 13 4 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.5989920143038034e-003</threshold>
            <left_val>-0.5189595222473145</left_val>
            <right_val>0.3010114133358002</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 19 18 3 -1.</_>
                <_>9 19 9 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0188046302646399</threshold>
            <left_val>0.1555491983890533</left_val>
            <right_val>-0.8047717213630676</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 21 14 2 -1.</_>
                <_>5 21 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.2497140131890774e-003</threshold>
            <left_val>0.1378080993890762</left_val>
            <right_val>-0.6076750755310059</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 0 9 5 -1.</_>
                <_>5 0 3 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.4204799663275480e-003</threshold>
            <left_val>0.3231942951679230</left_val>
            <right_val>-0.4340746104717255</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 20 15 3 -1.</_>
                <_>8 20 5 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0251743495464325</threshold>
            <left_val>-0.7078087925910950</left_val>
            <right_val>0.0931063294410706</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 9 6 14 -1.</_>
                <_>5 9 2 14 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.2285219058394432e-003</threshold>
            <left_val>-0.3251047134399414</left_val>
            <right_val>0.3357169926166534</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 3 3 18 -1.</_>
                <_>12 12 3 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0949934124946594</threshold>
            <left_val>0.0824390873312950</left_val>
            <right_val>-0.8754953742027283</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 14 4 9 -1.</_>
                <_>3 14 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.5919090993702412e-003</threshold>
            <left_val>-0.7380419969558716</left_val>
            <right_val>0.1385374963283539</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 15 11 8 -1.</_>
                <_>7 17 11 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.1146620381623507e-003</threshold>
            <left_val>0.1791726946830750</left_val>
            <right_val>-0.2795585989952087</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 7 6 10 -1.</_>
                <_>0 7 3 5 2.</_>
                <_>3 12 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0133490199223161</threshold>
            <left_val>0.1305782943964005</left_val>
            <right_val>-0.6980267167091370</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 6 4 13 -1.</_>
                <_>10 6 2 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0351814515888691</threshold>
            <left_val>0.4653536081314087</left_val>
            <right_val>-0.1069877967238426</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 4 13 -1.</_>
                <_>7 6 2 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0318745896220207</threshold>
            <left_val>-0.1356538981199265</left_val>
            <right_val>0.7904788851737976</right_val></_></_></trees>
      <stage_threshold>-1.1907930374145508</stage_threshold>
      <parent>0</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 2 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 2 6 8 -1.</_>
                <_>8 2 6 4 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0106474300846457</threshold>
            <left_val>0.3807902932167053</left_val>
            <right_val>-0.5867233872413635</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 11 19 12 -1.</_>
                <_>0 17 19 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0732144936919212</threshold>
            <left_val>-0.7955095171928406</left_val>
            <right_val>0.1722325980663300</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 18 6 5 -1.</_>
                <_>3 18 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.0464427806437016e-003</threshold>
            <left_val>0.1653216034173966</left_val>
            <right_val>-0.6937664747238159</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 17 9 6 -1.</_>
                <_>12 17 3 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.3225022060796618e-004</threshold>
            <left_val>-0.3324716091156006</left_val>
            <right_val>0.2366997003555298</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 20 15 3 -1.</_>
                <_>5 20 5 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0109900804236531</threshold>
            <left_val>-0.6913688778877258</left_val>
            <right_val>0.2105827033519745</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 19 8 4 -1.</_>
                <_>9 19 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5282750246115029e-004</threshold>
            <left_val>0.2030584961175919</left_val>
            <right_val>-0.4655165970325470</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 17 9 6 -1.</_>
                <_>3 17 3 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.4822261184453964e-004</threshold>
            <left_val>-0.4212292134761810</left_val>
            <right_val>0.2733530998229981</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 17 5 6 -1.</_>
                <_>14 20 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.4205856546759605e-003</threshold>
            <left_val>-0.4374446868896484</left_val>
            <right_val>0.0588318482041359</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 2 15 14 -1.</_>
                <_>7 2 5 14 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.3699279129505158</threshold>
            <left_val>0.9107081890106201</left_val>
            <right_val>-0.0872075408697128</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 17 5 6 -1.</_>
                <_>14 20 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.1259930953383446e-003</threshold>
            <left_val>0.1188673004508019</left_val>
            <right_val>-0.1852017045021057</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 17 5 6 -1.</_>
                <_>0 20 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.0144090093672276e-003</threshold>
            <left_val>-0.6305705904960632</left_val>
            <right_val>0.1457718014717102</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 0 13 8 -1.</_>
                <_>3 4 13 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.5623031482100487e-003</threshold>
            <left_val>-0.2936938107013702</left_val>
            <right_val>0.3241134881973267</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 21 14 2 -1.</_>
                <_>7 21 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0139668500050902</threshold>
            <left_val>-0.8065037131309509</left_val>
            <right_val>0.1126779019832611</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 4 4 15 -1.</_>
                <_>9 4 2 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0417344681918621</threshold>
            <left_val>0.7749533057212830</left_val>
            <right_val>-0.0788663029670715</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 18 8 5 -1.</_>
                <_>5 18 4 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.7996799326501787e-004</threshold>
            <left_val>0.2778331041336060</left_val>
            <right_val>-0.3519608974456787</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 4 4 15 -1.</_>
                <_>9 4 2 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0195885691791773</threshold>
            <left_val>-0.0657596364617348</left_val>
            <right_val>0.5241413712501526</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 4 4 15 -1.</_>
                <_>8 4 2 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.2163113877177238e-003</threshold>
            <left_val>-0.1552547961473465</left_val>
            <right_val>0.5483539104461670</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 11 8 8 -1.</_>
                <_>15 11 4 4 2.</_>
                <_>11 15 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0214585699141026</threshold>
            <left_val>-0.5225530862808228</left_val>
            <right_val>0.0822082683444023</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 13 6 7 -1.</_>
                <_>6 13 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.6805770359933376e-003</threshold>
            <left_val>-0.2443412989377976</left_val>
            <right_val>0.3612248897552490</right_val></_></_></trees>
      <stage_threshold>-1.3129220008850098</stage_threshold>
      <parent>1</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 3 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 1 8 13 -1.</_>
                <_>7 1 4 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.3544738590717316e-003</threshold>
            <left_val>0.2817318141460419</left_val>
            <right_val>-0.4972813129425049</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 21 14 2 -1.</_>
                <_>5 21 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.5724289268255234e-003</threshold>
            <left_val>-0.6550530195236206</left_val>
            <right_val>0.1940605938434601</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 21 18 2 -1.</_>
                <_>9 21 9 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.7714767754077911e-003</threshold>
            <left_val>-0.6223093867301941</left_val>
            <right_val>0.2762239873409271</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 18 8 5 -1.</_>
                <_>7 18 4 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0229958891868591</threshold>
            <left_val>0.0197985693812370</left_val>
            <right_val>-0.7832453846931458</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 17 8 6 -1.</_>
                <_>8 17 4 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.1443760013207793e-003</threshold>
            <left_val>0.2810871899127960</left_val>
            <right_val>-0.4821484982967377</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 2 7 10 -1.</_>
                <_>10 2 7 5 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.2591750919818878</threshold>
            <left_val>-0.6821495890617371</left_val>
            <right_val>-3.3729869755916297e-004</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 9 2 14 -1.</_>
                <_>3 9 1 14 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.0133039690554142e-003</threshold>
            <left_val>-0.6570441126823425</left_val>
            <right_val>0.1369359940290451</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 7 2 16 -1.</_>
                <_>15 7 1 16 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.4540671408176422e-003</threshold>
            <left_val>0.0869318172335625</left_val>
            <right_val>-0.7056797146797180</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 8 4 15 -1.</_>
                <_>3 8 2 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.6230311058461666e-003</threshold>
            <left_val>0.1663428992033005</left_val>
            <right_val>-0.5177295804023743</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 0 3 14 -1.</_>
                <_>14 0 3 7 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0125616695731878</threshold>
            <left_val>0.0902904719114304</left_val>
            <right_val>-0.1685097068548203</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 6 8 9 -1.</_>
                <_>9 6 4 9 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0428907386958599</threshold>
            <left_val>0.1297781020402908</left_val>
            <right_val>-0.5821806192398071</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 15 11 8 -1.</_>
                <_>8 17 11 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.3341030571609735e-003</threshold>
            <left_val>0.1369432955980301</left_val>
            <right_val>-0.1943780928850174</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 7 4 10 -1.</_>
                <_>7 7 2 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0412474609911442</threshold>
            <left_val>0.6854385137557983</left_val>
            <right_val>-0.1303945034742355</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 15 9 8 -1.</_>
                <_>10 17 9 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.1503392904996872e-003</threshold>
            <left_val>-0.1189543008804321</left_val>
            <right_val>0.0675766989588737</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 15 9 8 -1.</_>
                <_>0 17 9 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.7151240026578307e-003</threshold>
            <left_val>0.2647553980350494</left_val>
            <right_val>-0.3048745095729828</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 1 17 18 -1.</_>
                <_>2 10 17 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2084320038557053</threshold>
            <left_val>0.1240148991346359</left_val>
            <right_val>-0.4701411128044128</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 0 16 2 -1.</_>
                <_>2 0 8 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0723939687013626</threshold>
            <left_val>0.0969243794679642</left_val>
            <right_val>-0.7734774947166443</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 0 9 5 -1.</_>
                <_>11 0 3 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5335980569943786e-003</threshold>
            <left_val>0.1799121946096420</left_val>
            <right_val>-0.2578833103179932</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 0 6 10 -1.</_>
                <_>6 0 3 5 2.</_>
                <_>9 5 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.8640500754117966e-003</threshold>
            <left_val>0.1139298006892204</left_val>
            <right_val>-0.5517386794090271</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 6 4 7 -1.</_>
                <_>10 6 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.6523050144314766e-003</threshold>
            <left_val>0.1515468955039978</left_val>
            <right_val>-0.2290167957544327</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 4 15 11 -1.</_>
                <_>7 4 5 11 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0753487572073936</threshold>
            <left_val>-0.1463088989257813</left_val>
            <right_val>0.6810588240623474</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 15 4 8 -1.</_>
                <_>15 15 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.2630068063735962e-003</threshold>
            <left_val>-0.7278360128402710</left_val>
            <right_val>0.1028101965785027</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 15 4 8 -1.</_>
                <_>2 15 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.5124741047620773e-003</threshold>
            <left_val>-0.6305934786796570</left_val>
            <right_val>0.0932577997446060</right_val></_></_></trees>
      <stage_threshold>-1.3777279853820801</stage_threshold>
      <parent>2</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 4 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 4 11 -1.</_>
                <_>7 6 2 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.3849105760455132e-003</threshold>
            <left_val>0.5250058174133301</left_val>
            <right_val>-0.4323106110095978</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 17 16 4 -1.</_>
                <_>7 17 8 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.3772470410913229e-003</threshold>
            <left_val>0.2069848030805588</left_val>
            <right_val>-0.4271875917911530</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 3 10 8 -1.</_>
                <_>9 3 5 8 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0263201091438532</threshold>
            <left_val>0.1582517027854919</left_val>
            <right_val>-0.6550952196121216</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 6 7 10 -1.</_>
                <_>12 6 7 5 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0454887598752975</threshold>
            <left_val>-0.4951010942459106</left_val>
            <right_val>0.1799882054328919</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 0 6 5 -1.</_>
                <_>5 0 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.7006201930344105e-003</threshold>
            <left_val>0.3397116065025330</left_val>
            <right_val>-0.3691770136356354</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 18 14 3 -1.</_>
                <_>4 19 14 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.3270860072225332e-003</threshold>
            <left_val>0.3090786039829254</left_val>
            <right_val>-0.1977175027132034</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 20 14 3 -1.</_>
                <_>9 20 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.3802614137530327e-003</threshold>
            <left_val>0.0944884493947029</left_val>
            <right_val>-0.7319809794425964</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 21 14 2 -1.</_>
                <_>4 21 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.3565612286329269e-003</threshold>
            <left_val>0.1152020022273064</left_val>
            <right_val>-0.5400810241699219</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 8 3 14 -1.</_>
                <_>9 8 1 14 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.1178937107324600e-003</threshold>
            <left_val>-0.1595630943775177</left_val>
            <right_val>0.5377786755561829</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 9 3 14 -1.</_>
                <_>9 9 1 14 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.7829083204269409e-003</threshold>
            <left_val>0.5663471817970276</left_val>
            <right_val>-0.1327937990427017</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 7 9 16 -1.</_>
                <_>5 11 9 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0219448506832123</threshold>
            <left_val>0.1590128988027573</left_val>
            <right_val>-0.5175182223320007</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 13 6 8 -1.</_>
                <_>11 17 6 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0495100989937782</threshold>
            <left_val>0.0110676400363445</left_val>
            <right_val>-0.4997246861457825</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 17 7 6 -1.</_>
                <_>4 19 7 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1175360307097435e-003</threshold>
            <left_val>0.2649075984954834</left_val>
            <right_val>-0.2456562966108322</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 13 16 8 -1.</_>
                <_>10 13 8 4 2.</_>
                <_>2 17 8 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0103794699534774</threshold>
            <left_val>0.1262409985065460</left_val>
            <right_val>-0.4087724089622498</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 18 15 3 -1.</_>
                <_>2 19 15 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.4977258872240782e-003</threshold>
            <left_val>-0.1972302049398422</left_val>
            <right_val>0.3886674940586090</right_val></_></_></trees>
      <stage_threshold>-1.0618749856948853</stage_threshold>
      <parent>3</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 5 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 13 15 3 -1.</_>
                <_>7 13 5 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.1489548534154892e-003</threshold>
            <left_val>0.4018748104572296</left_val>
            <right_val>-0.5239737033843994</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 0 11 16 -1.</_>
                <_>8 4 11 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0504645407199860</threshold>
            <left_val>0.1304967999458313</left_val>
            <right_val>-0.5865144133567810</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 0 19 18 -1.</_>
                <_>0 6 19 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0559062696993351</threshold>
            <left_val>-0.5122954249382019</left_val>
            <right_val>0.2439288944005966</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 0 11 16 -1.</_>
                <_>8 4 11 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1428150981664658</threshold>
            <left_val>-0.0151801602914929</left_val>
            <right_val>-0.6959391832351685</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 1 4 20 -1.</_>
                <_>0 6 4 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0411627702414989</threshold>
            <left_val>0.1367373019456863</left_val>
            <right_val>-0.6415883898735046</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 6 15 4 -1.</_>
                <_>8 6 5 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0164687503129244</threshold>
            <left_val>0.2633903920650482</left_val>
            <right_val>-0.2208368033170700</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 9 18 6 -1.</_>
                <_>0 9 9 3 2.</_>
                <_>9 12 9 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0247631408274174</threshold>
            <left_val>0.1089773997664452</left_val>
            <right_val>-0.6521390080451965</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 5 3 14 -1.</_>
                <_>9 5 1 14 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.3008858337998390e-003</threshold>
            <left_val>-0.1829963028430939</left_val>
            <right_val>0.4361422955989838</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 0 6 8 -1.</_>
                <_>3 0 2 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.4035290591418743e-003</threshold>
            <left_val>-0.2436358034610748</left_val>
            <right_val>0.2822436988353729</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 6 18 6 -1.</_>
                <_>10 6 9 3 2.</_>
                <_>1 9 9 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0222106203436852</threshold>
            <left_val>-0.5464575886726379</left_val>
            <right_val>0.1354296952486038</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 7 4 15 -1.</_>
                <_>8 7 2 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0269680190831423</threshold>
            <left_val>0.6530094742774963</left_val>
            <right_val>-0.1429730951786041</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 5 8 10 -1.</_>
                <_>11 10 8 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0349279083311558</threshold>
            <left_val>-0.5234662890434265</left_val>
            <right_val>0.1008457019925118</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 5 8 10 -1.</_>
                <_>0 10 8 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0362635813653469</threshold>
            <left_val>0.1511014997959137</left_val>
            <right_val>-0.5418584942817688</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 20 15 3 -1.</_>
                <_>8 20 5 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0385267883539200</threshold>
            <left_val>-0.8694227933883667</left_val>
            <right_val>0.0371767692267895</right_val></_></_>
        <_>
          <!-- t
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

