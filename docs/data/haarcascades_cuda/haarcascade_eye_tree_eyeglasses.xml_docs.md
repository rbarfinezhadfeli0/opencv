# Documentation for `data/haarcascades_cuda/haarcascade_eye_tree_eyeglasses.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_eye_tree_eyeglasses.xml`
- **File Name**: `haarcascade_eye_tree_eyeglasses.xml`
- **File Size**: 1,095,035 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_eye_tree_eyeglasses.xml](../../data/haarcascades_cuda/haarcascade_eye_tree_eyeglasses.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    Tree-based 20x20 frontal eye detector with better handling of eyeglasses.
    Created by Shameem Hameed (http://umich.edu/~shameem)

////////////////////////////////////////////////////////////////////////////////////////

  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.

  By downloading, copying, installing or using the software you agree to this license.
  If you do not agree to this license, do not download, install,
  copy or use the software.


                        Intel License Agreement
                For Open Source Computer Vision Library

 Copyright (C) 2000, Intel Corporation, all rights reserved.
 Third party copyrights are property of their respective owners.

 Redistribution and use in source and binary forms, with or without modification,
 are permitted provided that the following conditions are met:

   * Redistribution's of source code must retain the above copyright notice,
     this list of conditions and the following disclaimer.

   * Redistribution's in binary form must reproduce the above copyright notice,
     this list of conditions and the following disclaimer in the documentation
     and/or other materials provided with the distribution.

   * The name of Intel Corporation may not be used to endorse or promote products
     derived from this software without specific prior written permission.

 This software is provided by the copyright holders and contributors "as is" and
 any express or implied warranties, including, but not limited to, the implied
 warranties of merchantability and fitness for a particular purpose are disclaimed.
 In no event shall the Intel Corporation or contributors be liable for any direct,
 indirect, incidental, special, exemplary, or consequential damages
 (including, but not limited to, procurement of substitute goods or services;
 loss of use, data, or profits; or business interruption) however caused
 and on any theory of liability, whether in contract, strict liability,
 or tort (including negligence or otherwise) arising in any way out of
 the use of this software, even if advised of the possibility of such damage.
-->
<opencv_storage>
<haarcascade_eye_tree type_id="opencv-haar-classifier">
  <size>
    20 20</size>
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
                <_>
                  8 7 12 1 -1.</_>
                <_>
                  8 7 6 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0269871093332767</threshold>
            <left_node>2</left_node>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 7 8 6 -1.</_>
                <_>
                  6 7 4 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0506705306470394</threshold>
            <left_val>-0.8039547204971314</left_val>
            <right_val>0.6049140095710754</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  5 3 12 12 -1.</_>
                <_>
                  9 7 4 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1291539072990418</threshold>
            <left_val>0.9054458141326904</left_val>
            <right_val>0.0440708100795746</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 8 12 12 -1.</_>
                <_>
                  1 14 12 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0888277366757393</threshold>
            <left_node>2</left_node>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 9 9 5 -1.</_>
                <_>
                  8 9 3 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0203982405364513</threshold>
            <left_val>0.7921888232231140</left_val>
            <right_val>0.0406922996044159</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  5 7 9 6 -1.</_>
                <_>
                  8 7 3 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0612617582082748</threshold>
            <left_val>0.4258536100387573</left_val>
            <right_val>-0.7032520771026611</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 0 18 15 -1.</_>
                <_>
                  2 5 18 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.2049081027507782</threshold>
            <left_node>2</left_node>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  7 1 9 9 -1.</_>
                <_>
                  7 4 9 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0949330478906631</threshold>
            <left_val>-0.4401764869689941</left_val>
            <right_val>0.5364052057266235</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  8 19 3 1 -1.</_>
                <_>
                  9 19 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.2091030366718769e-003</threshold>
            <left_val>0.6877645850181580</left_val>
            <right_val>-0.5587934851646423</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 17 2 2 -1.</_>
                <_>
                  5 17 1 1 2.</_>
                <_>
                  6 18 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.2227972345426679e-004</threshold>
            <left_node>1</left_node>
            <right_val>-0.7268440127372742</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 17 2 2 -1.</_>
                <_>
                  5 17 1 1 2.</_>
                <_>
                  6 18 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.2678289143368602e-004</threshold>
            <left_val>-0.5802800059318543</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  10 18 3 1 -1.</_>
                <_>
                  11 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.8421510513871908e-004</threshold>
            <left_val>0.5617753267288208</left_val>
            <right_val>-0.2983418107032776</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 7 9 7 -1.</_>
                <_>
                  10 7 3 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0511505901813507</threshold>
            <left_val>0.5984076261520386</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  6 8 12 5 -1.</_>
                <_>
                  9 8 6 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0616220608353615</threshold>
            <left_node>2</left_node>
            <right_val>0.7474393248558044</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  13 1 6 7 -1.</_>
                <_>
                  13 1 3 7 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0728734731674194</threshold>
            <left_val>-0.4970377981662750</left_val>
            <right_val>0.2812925875186920</right_val></_></_></trees>
      <stage_threshold>-1.6473180055618286</stage_threshold>
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
                <_>
                  5 2 12 15 -1.</_>
                <_>
                  9 7 4 5 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.4199487864971161</threshold>
            <left_node>2</left_node>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  6 5 14 1 -1.</_>
                <_>
                  6 5 7 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0561862885951996</threshold>
            <left_val>0.2758620083332062</left_val>
            <right_val>-0.6462321877479553</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  9 9 10 1 -1.</_>
                <_>
                  9 9 5 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0237111095339060</threshold>
            <left_val>0.8524125218391419</left_val>
            <right_val>8.3703370764851570e-003</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 9 9 3 -1.</_>
                <_>
                  5 9 3 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0405234396457672</threshold>
            <left_node>1</left_node>
            <right_val>0.7427021861076355</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 8 20 12 -1.</_>
                <_>
                  0 14 20 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2738890051841736</threshold>
            <left_val>-0.4928669035434723</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  0 5 4 13 -1.</_>
                <_>
                  2 5 2 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0142938001081347</threshold>
            <left_val>0.7178478837013245</left_val>
            <right_val>-0.0422239787876606</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  11 18 3 2 -1.</_>
                <_>
                  12 18 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1144729107618332e-003</threshold>
            <left_val>-0.8019660115242004</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  11 18 3 1 -1.</_>
                <_>
                  12 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.0659949621185660e-003</threshold>
            <left_node>2</left_node>
            <right_val>-0.6602591276168823</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  11 19 3 1 -1.</_>
                <_>
                  12 19 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.0812469990924001e-003</threshold>
            <left_val>0.4791637063026428</left_val>
            <right_val>-0.5164529085159302</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 9 9 3 -1.</_>
                <_>
                  13 9 3 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0301982890814543</threshold>
            <left_node>1</left_node>
            <right_val>0.5132756233215332</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 8 8 7 -1.</_>
                <_>
                  7 8 4 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0405695512890816</threshold>
            <left_node>2</left_node>
            <right_val>0.6664149761199951</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  8 6 9 8 -1.</_>
                <_>
                  11 6 3 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0706797391176224</threshold>
            <left_val>-0.4529865980148315</left_val>
            <right_val>0.5548071861267090</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 18 2 2 -1.</_>
                <_>
                  4 18 1 1 2.</_>
                <_>
                  5 19 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.8928138827905059e-004</threshold>
            <left_val>-0.7252629995346069</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 18 2 2 -1.</_>
                <_>
                  4 18 1 1 2.</_>
                <_>
                  5 19 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.0574717139825225e-004</threshold>
            <left_node>2</left_node>
            <right_val>-0.5647987127304077</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  7 6 8 14 -1.</_>
                <_>
                  9 6 4 14 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0209765601903200</threshold>
            <left_val>0.6999353766441345</left_val>
            <right_val>0.0685004666447639</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 13 4 3 -1.</_>
                <_>
                  15 14 4 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0127949602901936</threshold>
            <left_node>1</left_node>
            <right_val>-0.8640956878662109</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 13 4 2 -1.</_>
                <_>
                  16 13 2 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-8.1120636314153671e-003</threshold>
            <left_val>0.4444836080074310</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  5 6 6 14 -1.</_>
                <_>
                  7 6 2 14 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0155065301805735</threshold>
            <left_val>0.3667531013488770</left_val>
            <right_val>-0.2918907105922699</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 7 8 11 -1.</_>
                <_>
                  2 7 4 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0129156503826380</threshold>
            <left_node>2</left_node>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 7 8 7 -1.</_>
                <_>
                  2 7 4 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.6297221928834915e-003</threshold>
            <left_val>-0.4756678044795990</left_val>
            <right_val>0.1035035029053688</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  2 16 3 1 -1.</_>
                <_>
                  3 17 1 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-3.6532930098474026e-003</threshold>
            <left_val>-0.6172305941581726</left_val>
            <right_val>0.5438253283500671</right_val></_></_></trees>
      <stage_threshold>-1.4257860183715820</stage_threshold>
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
                <_>
                  3 0 15 18 -1.</_>
                <_>
                  8 6 5 6 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.7873197197914124</threshold>
            <left_val>0.7126883864402771</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 6 20 14 -1.</_>
                <_>
                  0 13 20 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1690800935029984</threshold>
            <left_val>-0.7190899848937988</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  6 7 9 7 -1.</_>
                <_>
                  9 7 3 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0403696894645691</threshold>
            <left_val>0.4414893090724945</left_val>
            <right_val>-0.4225192964076996</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 9 6 2 -1.</_>
                <_>
                  5 9 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0191323608160019</threshold>
            <left_node>1</left_node>
            <right_val>0.6918622851371765</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  17 16 2 2 -1.</_>
                <_>
                  17 16 1 1 2.</_>
                <_>
                  18 17 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.4184539951384068e-004</threshold>
            <left_node>2</left_node>
            <right_val>-0.7611696720123291</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  17 16 2 2 -1.</_>
                <_>
                  17 16 1 1 2.</_>
                <_>
                  18 17 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.8941037645563483e-004</threshold>
            <left_val>-0.6814042925834656</left_val>
            <right_val>0.1600991934537888</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 8 6 5 -1.</_>
                <_>
                  16 8 2 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.1503049694001675e-003</threshold>
            <left_node>1</left_node>
            <right_node>2</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 18 4 2 -1.</_>
                <_>
                  16 19 4 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.3156129755079746e-003</threshold>
            <left_val>-0.5591660737991333</left_val>
            <right_val>0.5128449797630310</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  8 8 9 12 -1.</_>
                <_>
                  11 8 3 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0415212698280811</threshold>
            <left_val>0.2442256957292557</left_val>
            <right_val>-0.4688340127468109</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 18 3 1 -1.</_>
                <_>
                  9 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.1200548922643065e-004</threshold>
            <left_node>1</left_node>
            <right_val>-0.6952788829803467</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 18 3 2 -1.</_>
                <_>
                  9 18 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5798299573361874e-003</threshold>
            <left_val>-0.6350964903831482</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  0 8 4 11 -1.</_>
                <_>
                  2 8 2 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0115736499428749</threshold>
            <left_val>0.6468638181686401</left_val>
            <right_val>6.9198559504002333e-004</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 0 10 1 -1.</_>
                <_>
                  15 0 5 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.1843519061803818e-003</threshold>
            <left_node>2</left_node>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  13 1 3 3 -1.</_>
                <_>
                  14 1 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.9345690272748470e-003</threshold>
            <left_val>0.4563289880752564</left_val>
            <right_val>-0.5884143710136414</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  2 8 12 12 -1.</_>
                <_>
                  6 8 4 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0587881505489349</threshold>
            <left_val>0.2670420110225678</left_val>
            <right_val>-0.3834899067878723</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 17 1 3 -1.</_>
                <_>
                  18 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.5392808280885220e-004</threshold>
            <left_val>-0.4891336858272553</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  18 18 1 2 -1.</_>
                <_>
                  18 19 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.3035060409456491e-004</threshold>
            <left_val>-0.3842155039310455</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  8 10 6 5 -1.</_>
                <_>
                  10 10 2 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.8775108084082603e-003</threshold>
            <left_val>0.6684569716453552</left_val>
            <right_val>0.0931582599878311</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  13 17 3 2 -1.</_>
                <_>
                  14 17 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.6710379859432578e-003</threshold>
            <left_node>1</left_node>
            <right_val>-0.6036937236785889</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 4 6 12 -1.</_>
                <_>
                  0 8 6 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.4162790030241013e-003</threshold>
            <left_node>2</left_node>
            <right_val>-0.3041876852512360</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  0 8 5 4 -1.</_>
                <_>
                  0 9 5 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.7876187860965729e-003</threshold>
            <left_val>0.3969906866550446</left_val>
            <right_val>-0.6668758988380432</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  13 6 4 6 -1.</_>
                <_>
                  14 7 2 6 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0129167800769210</threshold>
            <left_node>1</left_node>
            <right_node>2</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 2 3 2 -1.</_>
                <_>
                  5 2 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.0156269203871489e-003</threshold>
            <left_val>-0.7123972773551941</left_val>
            <right_val>0.4625298976898193</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  11 2 8 17 -1.</_>
                <_>
                  13 2 4 17 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0197859406471252</threshold>
            <left_val>0.2833831906318665</left_val>
            <right_val>-0.3531793057918549</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 0 3 3 -1.</_>
                <_>
                  16 0 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.3207770902663469e-003</threshold>
            <left_node>1</left_node>
            <right_val>-0.7329139709472656</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  10 5 9 13 -1.</_>
                <_>
                  13 5 3 13 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0296062398701906</threshold>
            <left_node>2</left_node>
            <right_val>0.4953075945377350</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  5 8 8 6 -1.</_>
                <_>
                  7 8 4 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0446147881448269</threshold>
            <left_val>-0.1950280964374542</left_val>
            <right_val>0.7981641888618469</right_val></_></_></trees>
      <stage_threshold>-1.4711019992828369</stage_threshold>
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
                <_>
                  3 1 15 18 -1.</_>
                <_>
                  8 7 5 6 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.9236614108085632</threshold>
            <left_val>0.7691580057144165</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  6 7 9 8 -1.</_>
                <_>
                  9 7 3 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0481939390301704</threshold>
            <left_node>2</left_node>
            <right_val>-0.5136122703552246</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  0 6 20 14 -1.</_>
                <_>
                  0 13 20 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2866987884044647</threshold>
            <left_val>-0.2967190146446228</left_val>
            <right_val>0.6202818751335144</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 7 6 7 -1.</_>
                <_>
                  3 7 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0130381602793932</threshold>
            <left_node>1</left_node>
            <right_node>2</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  9 19 3 1 -1.</_>
                <_>
                  10 19 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.4749659458175302e-003</threshold>
            <left_val>-0.7129424810409546</left_val>
            <right_val>0.5911517739295960</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  4 6 9 7 -1.</_>
                <_>
                  7 6 3 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0469217486679554</threshold>
            <left_val>0.3130356073379517</left_val>
            <right_val>-0.3674969077110291</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 10 1 10 -1.</_>
                <_>
                  18 15 1 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.4459899868816137e-003</threshold>
            <left_val>-0.4693000018596649</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  12 16 2 4 -1.</_>
                <_>
                  12 16 1 2 2.</_>
                <_>
                  13 18 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.5321498978883028e-003</threshold>
            <left_val>-0.7745016217231751</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  12 19 4 1 -1.</_>
                <_>
                  13 19 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.4651260571554303e-003</threshold>
            <left_val>0.3641478121280670</left_val>
            <right_val>-0.5744588971138001</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 5 6 15 -1.</_>
                <_>
                  11 5 2 15 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0113074202090502</threshold>
            <left_node>1</left_node>
            <right_node>2</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  10 18 4 1 -1.</_>
                <_>
                  11 18 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.2048849603161216e-003</threshold>
            <left_val>-0.5572764873504639</left_val>
            <right_val>0.4787167012691498</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  1 0 12 16 -1.</_>
                <_>
                  5 0 4 16 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0627528727054596</threshold>
            <left_val>0.2278853058815002</left_val>
            <right_val>-0.4366796910762787</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 13 3 3 -1.</_>
                <_>
                  0 14 3 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.0173111483454704e-003</threshold>
            <left_val>-0.7356877923011780</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  1 13 1 3 -1.</_>
                <_>
                  1 14 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.5160309849306941e-003</threshold>
            <left_node>2</left_node>
            <right_val>-0.5848069787025452</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  14 0 6 1 -1.</_>
                <_>
                  17 0 3 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9954680465161800e-003</threshold>
            <left_val>0.0215440206229687</left_val>
            <right_val>0.5587568879127502</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  12 0 3 3 -1.</_>
                <_>
                  13 0 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.4435209818184376e-003</threshold>
            <left_node>1</left_node>
            <right_val>-0.7656589746475220</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  12 1 3 2 -1.</_>
                <_>
                  13 1 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.6550020556896925e-003</threshold>
            <left_val>-0.6544749736785889</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  14 2 6 13 -1.</_>
                <_>
                  16 2 2 13 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0114076901227236</threshold>
            <left_val>0.5363308191299439</left_val>
            <right_val>-0.0388491712510586</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  12 4 6 1 -1.</_>
                <_>
                  14 6 2 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-2.3805440869182348e-003</threshold>
            <left_node>1</left_node>
            <right_node>2</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  15 6 5 2 -1.</_>
                <_>
                  15 7 5 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.6475258208811283e-003</threshold>
            <left_val>0.3398441076278687</left_val>
            <right_val>-0.6502509117126465</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  9 0 5 12 -1.</_>
                <_>
                  9 4 5 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1401824057102203</threshold>
            <left_val>-0.3249109089374542</left_val>
            <right_val>0.7506706714630127</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 1 13 9 -1.</_>
                <_>
                  6 4 13 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0623583607375622</threshold>
            <left_val>0.4577716886997223</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 0 3 2 -1.</_>
                <_>
                  17 0 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.3628599699586630e-003</threshold>
            <left_node>2</left_node>
            <right_val>-0.6320266127586365</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  6 0 4 2 -1.</_>
                <_>
                  6 0 2 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-4.4609848409891129e-003</threshold>
            <left_val>0.4059796035289764</left_val>
            <right_val>-0.2085406929254532</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 2 3 3 -1.</_>
                <_>
                  3 3 3 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0100468397140503</threshold>
            <left_val>-0.7478982806205750</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  7 1 13 6 -1.</_>
                <_>
                  5 3 13 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0292748194187880</threshold>
            <left_node>2</left_node>
            <right_val>-0.1799547970294952</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  3 2 2 3 -1.</_>
                <_>
                  2 3 2 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>7.7389390207827091e-003</threshold>
            <left_val>0.4778284132480621</left_val>
            <right_val>-0.6511334180831909</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  17 0 3 1 -1.</_>
                <_>
                  18 0 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.4774020528420806e-003</threshold>
            <left_node>1</left_node>
            <right_val>-0.6626989841461182</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  1 12 5 6 -1.</_>
                <_>
                  1 15 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0149898203089833</threshold>
            <left_val>-0.1669555008411408</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  5 14 3 1 -1.</_>
                <_>
                  6 15 1 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>4.5073241926729679e-003</threshold>
            <left_val>0.3870205879211426</left_val>
            <right_val>-0.7340937256813049</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 7 7 3 -1.</_>
                <_>
                  0 8 7 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.4901049435138702e-003</threshold>
            <left_node>1</left_node>
            <right_val>-0.3428083956241608</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 8 2 4 -1.</_>
                <_>
                  0 9 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.9141662465408444e-004</threshold>
            <left_node>2</left_node>
            <right_val>-0.2803674042224884</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  7 2 4 3 -1.</_>
                <_>
                  6 3 4 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0115582197904587</threshold>
            <left_val>-0.4252395927906036</left_val>
            <right_val>0.4525966942310333</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 7 6 10 -1.</_>
                <_>
                  8 7 2 10 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0200119502842426</threshold>
            <left_val>0.4013311862945557</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  2 5 8 12 -1.</_>
                <_>
                  4 5 4 12 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0170923005789518</threshold>
            <left_val>0.3697001039981842</left_val>
            <right_node>2</right_node></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  4 0 12 4 -1.</_>
                <_>
                  4 2 12 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0676851719617844</threshold>
            <left_val>0.7443867921829224</left_val>
            <right_val>-0.3825584053993225</right_val></_></_></trees>
      <stage_threshold>-1.3850779533386230</stage_threshold>
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
                <_>
                  7 8 8 12 -1.</_>
                <_>
                  9 8 4 12 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0209111496806145</threshold>
            <left_node>1</left_node>
            <right_node>2</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 6 11 14 -1.</_>
                <_>
                  8 13 11 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1430570930242539</threshold>
            <left_val>-0.3496556878089905</left_val>
            <right_val>0.7013456225395203</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  16 9 4 9 -1.</_>
                <_>
                  18 9 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0119250295683742</threshold>
            <left_val>-0.6040462851524353</left_val>
            <right_val>0.0856159031391144</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  12 9 6 2 -1.</_>
                <_>
                  14 9 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0247420091181993</threshold>
            <left_node>1</left_node>
            <right_val>0.8536558747291565</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  6 1 10 6 -1.</_>
                <_>
                  6 3 10 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0457321181893349</threshold>
            <left_node>2</left_node>
            <right_val>0.4187641143798828</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  5 0 4 5 -1.</_>
                <_>
                  5 0 2 5 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0432044304907322</threshold>
            <left_val>-0.3909491896629334</left_val>
            <right_val>0.2738798856735230</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 17 1 3 -1.</_>
                <_>
                  2 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.2548422031104565e-004</threshold>
            <left_val>-0.6201112270355225</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  2 17 1 3 -1.</_>
                <_>
                  2 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.4243220211938024e-003</threshold>
            <left_node>2</left_node>
            <right_val>-0.6158943772315979</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  8 0 12 2 -1.</_>
                <_>
                  12 0 4 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.3335479460656643e-003</threshold>
            <left_val>0.6059644818305969</left_val>
            <right_val>0.0158404801040888</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 8 6 5 -1.</_>
                <_>
                  2 8 2 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.1891010738909245e-003</threshold>
            <left_node>1</left_node>
            <right_val>-0.2085282951593399</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 18 4 1 -1.</_>
                <_>
                  9 18 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.8233320442959666e-003</threshold>
            <left_node>2</left_node>
            <right_val>-0.8133838176727295</right_val></_>
          <_>
            <!-- node 2 -->
            <feature>
              <rects>
                <_>
                  10 18 2 1 -1.</_>
                <_>
                  11 18 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.6109029529616237e-003</threshold>
            <left_val>0.5678064823150635</left_val>
            <right_val>-0.8704625964164734</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
      
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

