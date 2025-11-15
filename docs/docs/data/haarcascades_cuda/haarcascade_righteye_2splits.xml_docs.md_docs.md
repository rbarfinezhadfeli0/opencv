# Documentation for `docs/data/haarcascades_cuda/haarcascade_righteye_2splits.xml_docs.md`

## File Metadata

- **Full Path**: `docs/data/haarcascades_cuda/haarcascade_righteye_2splits.xml_docs.md`
- **File Name**: `haarcascade_righteye_2splits.xml_docs.md`
- **File Size**: 51,098 bytes
- **File Type**: .md
- **Link to Source**: [docs/data/haarcascades_cuda/haarcascade_righteye_2splits.xml_docs.md](../../../docs/data/haarcascades_cuda/haarcascade_righteye_2splits.xml_docs.md)

## Purpose and Role

This file is located in the `docs/data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `data/haarcascades_cuda/haarcascade_righteye_2splits.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_righteye_2splits.xml`
- **File Name**: `haarcascade_righteye_2splits.xml`
- **File Size**: 324,585 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_righteye_2splits.xml](../../data/haarcascades_cuda/haarcascade_righteye_2splits.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    Tree-based 20x20 right eye detector.
    The detector is trained by 6665 positive samples from FERET, VALID and BioID face databases.
    Created by Shiqi Yu (http://yushiqi.cn/research/eyedetection).

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
<haarcascade_righteye type_id="opencv-haar-classifier">
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
                  8 7 3 12 -1.</_>
                <_>
                  8 11 3 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0482105500996113</threshold>
            <left_node>1</left_node>
            <right_val>-0.8614044785499573</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 7 8 3 -1.</_>
                <_>
                  10 9 4 3 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0415761992335320</threshold>
            <left_val>0.9176905751228333</left_val>
            <right_val>-0.2128400951623917</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 13 2 6 -1.</_>
                <_>
                  9 16 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.3528684228658676e-03</threshold>
            <left_val>-0.6978576779365540</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 2 12 8 -1.</_>
                <_>
                  11 2 6 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.2144919785205275e-04</threshold>
            <left_val>0.7952337265014648</left_val>
            <right_val>-0.4894809126853943</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 0 6 6 -1.</_>
                <_>
                  14 3 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0218533501029015</threshold>
            <left_val>0.7057464122772217</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 1 5 12 -1.</_>
                <_>
                  8 4 5 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0996729284524918</threshold>
            <left_val>-0.7066624164581299</left_val>
            <right_val>0.7921097874641418</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 8 3 12 -1.</_>
                <_>
                  1 12 3 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0216648206114769</threshold>
            <left_node>1</left_node>
            <right_val>-0.6089860796928406</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 11 2 7 -1.</_>
                <_>
                  1 11 1 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.5680727604776621e-04</threshold>
            <left_val>0.7168570160865784</left_val>
            <right_val>-0.3046456873416901</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 12 9 7 -1.</_>
                <_>
                  9 12 3 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0133330496028066</threshold>
            <left_node>1</left_node>
            <right_val>-0.4684469103813171</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  13 4 6 9 -1.</_>
                <_>
                  15 4 2 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.2925298959016800e-03</threshold>
            <left_val>0.6423593163490295</left_val>
            <right_val>-0.5118042826652527</right_val></_></_></trees>
      <stage_threshold>-2.2325520515441895</stage_threshold>
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
                  4 7 12 12 -1.</_>
                <_>
                  8 11 4 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.3394871950149536</threshold>
            <left_val>0.7791326045989990</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  15 0 4 20 -1.</_>
                <_>
                  15 5 4 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1367247998714447</threshold>
            <left_val>0.2642127871513367</left_val>
            <right_val>-0.8791009187698364</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 12 5 8 -1.</_>
                <_>
                  0 16 5 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0313945002853870</threshold>
            <left_val>-0.6995670199394226</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 2 12 8 -1.</_>
                <_>
                  12 2 4 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0108281401917338</threshold>
            <left_val>0.7650449275970459</left_val>
            <right_val>-0.4371921122074127</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  19 0 1 8 -1.</_>
                <_>
                  19 4 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.2506768368184566e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.5756158232688904</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  9 7 3 12 -1.</_>
                <_>
                  9 11 3 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0226754695177078</threshold>
            <left_val>0.7408059239387512</left_val>
            <right_val>-0.3667725026607513</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 2 8 8 -1.</_>
                <_>
                  1 6 8 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0391614809632301</threshold>
            <left_node>1</left_node>
            <right_val>0.6404516100883484</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 12 4 4 -1.</_>
                <_>
                  2 12 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.1934089493006468e-03</threshold>
            <left_val>0.1604758948087692</left_val>
            <right_val>-0.7101097702980042</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 7 6 8 -1.</_>
                <_>
                  9 7 3 4 2.</_>
                <_>
                  12 11 3 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0253219902515411</threshold>
            <left_node>1</left_node>
            <right_val>0.4957486093044281</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  13 18 7 2 -1.</_>
                <_>
                  13 19 7 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.7583367237821221e-04</threshold>
            <left_val>-0.7173789739608765</left_val>
            <right_val>-0.0185817703604698</right_val></_></_></trees>
      <stage_threshold>-2.1598019599914551</stage_threshold>
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
                  4 7 12 12 -1.</_>
                <_>
                  8 11 4 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.2655405998229980</threshold>
            <left_node>1</left_node>
            <right_val>-0.8471245169639587</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 8 5 12 -1.</_>
                <_>
                  0 12 5 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0225327797234058</threshold>
            <left_val>0.8797718882560730</left_val>
            <right_val>-0.3339469134807587</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 0 4 8 -1.</_>
                <_>
                  18 0 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.5310067515820265e-04</threshold>
            <left_val>-0.8203244805335999</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 12 1 8 -1.</_>
                <_>
                  16 16 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.5820249973330647e-04</threshold>
            <left_val>-0.7517635822296143</left_val>
            <right_val>0.6776971220970154</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 1 9 9 -1.</_>
                <_>
                  12 1 3 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.0837490117410198e-04</threshold>
            <left_node>1</left_node>
            <right_val>-0.8331400156021118</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 16 1 3 -1.</_>
                <_>
                  15 17 1 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>2.6810260023921728e-03</threshold>
            <left_val>0.5384474992752075</left_val>
            <right_val>-0.7653415799140930</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 14 2 4 -1.</_>
                <_>
                  2 16 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.5202371701598167e-04</threshold>
            <left_val>-0.7751489877700806</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  6 12 9 3 -1.</_>
                <_>
                  9 12 3 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0122417397797108</threshold>
            <left_val>0.6324015259742737</left_val>
            <right_val>-0.6339520812034607</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 18 5 2 -1.</_>
                <_>
                  0 19 5 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.2314196838997304e-05</threshold>
            <left_node>1</left_node>
            <right_val>0.4429041147232056</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  1 7 18 12 -1.</_>
                <_>
                  7 11 6 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.7191110849380493</threshold>
            <left_val>0.8013592958450317</left_val>
            <right_val>-0.5343109965324402</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 0 16 12 -1.</_>
                <_>
                  4 0 8 6 2.</_>
                <_>
                  12 6 8 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0242803394794464</threshold>
            <left_node>1</left_node>
            <right_val>-0.6779791712760925</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 3 2 5 -1.</_>
                <_>
                  9 3 1 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.4558640327304602e-03</threshold>
            <left_val>0.4903061091899872</left_val>
            <right_val>-0.8844798207283020</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  17 17 1 2 -1.</_>
                <_>
                  17 17 1 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-6.2993327446747571e-05</threshold>
            <left_node>1</left_node>
            <right_val>-0.5788341760635376</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  18 16 1 3 -1.</_>
                <_>
                  17 17 1 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-4.6443562023341656e-03</threshold>
            <left_val>-0.8587880730628967</left_val>
            <right_val>0.5245460271835327</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 9 2 6 -1.</_>
                <_>
                  1 9 1 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.0299328247783706e-05</threshold>
            <left_node>1</left_node>
            <right_val>-0.5271345973014832</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  3 3 3 4 -1.</_>
                <_>
                  4 3 1 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.7485519424080849e-03</threshold>
            <left_val>-0.8562619090080261</left_val>
            <right_val>0.4894461035728455</right_val></_></_></trees>
      <stage_threshold>-2.3451159000396729</stage_threshold>
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
                  4 7 12 12 -1.</_>
                <_>
                  8 11 4 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.3837707936763763</threshold>
            <left_val>0.7171502113342285</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  10 0 7 8 -1.</_>
                <_>
                  10 4 7 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1383703052997589</threshold>
            <left_val>0.3439235985279083</left_val>
            <right_val>-0.7993127703666687</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 0 2 9 -1.</_>
                <_>
                  19 0 1 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.3107071067206562e-04</threshold>
            <left_val>-0.6835243105888367</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 13 1 4 -1.</_>
                <_>
                  4 13 1 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-5.1273438148200512e-03</threshold>
            <left_val>0.5825061798095703</left_val>
            <right_val>-0.4095500111579895</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 8 6 2 -1.</_>
                <_>
                  12 10 2 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0261006802320480</threshold>
            <left_node>1</left_node>
            <right_val>-0.4371330142021179</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  14 11 4 7 -1.</_>
                <_>
                  15 11 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.0628979653120041e-03</threshold>
            <left_val>0.7068073749542236</left_val>
            <right_val>-0.2681793868541718</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 0 13 8 -1.</_>
                <_>
                  4 2 13 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0978548526763916</threshold>
            <left_val>0.7394003868103027</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  9 1 7 8 -1.</_>
                <_>
                  9 5 7 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1182982027530670</threshold>
            <left_val>0.6381418108940125</left_val>
            <right_val>-0.3872187137603760</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 0 12 9 -1.</_>
                <_>
                  10 0 6 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.5409049168229103e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.4880301952362061</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  14 3 4 4 -1.</_>
                <_>
                  15 3 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.6851659640669823e-03</threshold>
            <left_val>0.3908346891403198</left_val>
            <right_val>-0.6556153893470764</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 16 4 4 -1.</_>
                <_>
                  0 18 4 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.6870240215212107e-03</threshold>
            <left_val>-0.4989174902439117</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  3 17 2 1 -1.</_>
                <_>
                  3 17 1 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-3.8136160001158714e-03</threshold>
            <left_val>-0.6640558838844299</left_val>
            <right_val>0.4065074920654297</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  17 16 1 3 -1.</_>
                <_>
                  16 17 1 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>2.0289309322834015e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.6998921036720276</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  11 10 6 4 -1.</_>
                <_>
                  10 11 6 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-7.6308869756758213e-03</threshold>
            <left_val>0.4320684075355530</left_val>
            <right_val>-0.2966496944427490</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  19 0 1 4 -1.</_>
                <_>
                  19 2 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.3815231290645897e-04</threshold>
            <left_node>1</left_node>
            <right_val>-0.4680854082107544</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  17 0 3 3 -1.</_>
                <_>
                  18 1 1 1 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.5163291767239571e-03</threshold>
            <left_val>0.3652149140834808</left_val>
            <right_val>-0.7601454257965088</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 1 12 6 -1.</_>
                <_>
                  2 4 12 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0614795088768005</threshold>
            <left_node>1</left_node>
            <right_val>0.5699062943458557</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  19 2 1 16 -1.</_>
                <_>
                  15 6 1 8 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0462865792214870</threshold>
            <left_val>0.2262506037950516</left_val>
            <right_val>-0.4533078074455261</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  12 2 4 6 -1.</_>
                <_>
                  13 2 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.6903551556169987e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.7728670835494995</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  11 3 3 3 -1.</_>
                <_>
                  12 3 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.8803169950842857e-03</threshold>
            <left_val>0.2734912037849426</left_val>
            <right_val>-0.6666783094406128</right_val></_></_></trees>
      <stage_threshold>-2.3431489467620850</stage_threshold>
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
                  1 7 18 12 -1.</_>
                <_>
                  7 11 6 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.5542067289352417</threshold>
            <left_node>1</left_node>
            <right_val>-0.6062026023864746</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 1 12 9 -1.</_>
                <_>
                  12 1 4 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.9329799152910709e-03</threshold>
            <left_val>0.7854202985763550</left_val>
            <right_val>-0.3552212119102478</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 0 2 10 -1.</_>
                <_>
                  18 5 2 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0211699604988098</threshold>
            <left_val>0.5294768810272217</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 5 12 15 -1.</_>
                <_>
                  8 10 4 5 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.6742839813232422</threshold>
            <left_val>0.4606522023677826</left_val>
            <right_val>-0.7005820870399475</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 8 4 12 -1.</_>
                <_>
                  1 12 4 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0427250787615776</threshold>
            <left_node>1</left_node>
            <right_val>-0.5990480780601501</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  6 13 8 2 -1.</_>
                <_>
                  8 13 4 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0101093295961618</threshold>
            <left_val>0.6810922026634216</left_val>
            <right_val>-0.2073187977075577</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 0 4 15 -1.</_>
                <_>
                  18 0 2 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.5861130133271217e-03</threshold>
            <left_val>-0.5242084860801697</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  14 0 4 8 -1.</_>
                <_>
                  15 0 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.6380418613553047e-03</threshold>
            <left_val>-0.7016978263854980</left_val>
            <right_val>0.4410013854503632</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 0 8 9 -1.</_>
                <_>
                  5 3 8 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0976815819740295</threshold>
            <left_val>0.5770874023437500</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 0 6 6 -1.</_>
                <_>
                  10 0 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0101973600685596</threshold>
            <left_val>-0.0985185503959656</left_val>
            <right_val>-0.8811169862747192</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 17 3 3 -1.</_>
                <_>
                  11 17 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.5724549777805805e-03</threshold>
            <left_val>-0.8323333859443665</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  10 17 4 3 -1.</_>
                <_>
                  11 17 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.6594230439513922e-03</threshold>
            <left_val>0.3099535107612610</left_val>
            <right_val>-0.8160917758941650</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 12 4 4 -1.</_>
                <_>
                  15 12 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.0042720241472125e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.4355852007865906</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 18 4 2 -1.</_>
                <_>
                  9 18 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.6080000679939985e-03</threshold>
            <left_val>0.3356660008430481</left_val>
            <right_val>-0.8188933134078979</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 1 4 5 -1.</_>
                <_>
                  7 1 2 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.9724509008228779e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.7704818248748779</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  2 0 6 5 -1.</_>
                <_>
                  4 0 2 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0122432401403785</threshold>
            <left_val>0.2253420054912567</left_val>
            <right_val>-0.6869555115699768</right_val></_></_></trees>
      <stage_threshold>-2.1268370151519775</stage_threshold>
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
                <_>
                  8 7 8 3 -1.</_>
                <_>
                  10 9 4 3 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0577849298715591</threshold>
            <left_node>1</left_node>
            <right_val>-0.7051600813865662</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  14 12 4 3 -1.</_>
                <_>
                  15 12 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.7517809756100178e-03</threshold>
            <left_val>0.8565592169761658</left_val>
            <right_val>-0.0924034193158150</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 10 3 4 -1.</_>
                <_>
                  9 11 3 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0115223797038198</threshold>
            <left_node>1</left_node>
            <right_val>-0.4274964034557343</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  17 0 2 6 -1.</_>
                <_>
                  17 3 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.8323760963976383e-03</threshold>
            <left_val>0.7591353058815002</left_val>
            <right_val>-0.1089404970407486</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 9 6 9 -1.</_>
                <_>
                  3 12 2 3 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0809223875403404</threshold>
            <left_node>1</left_node>
            <right_val>-0.3136476874351501</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 11 8 4 -1.</_>
                <_>
                  9 11 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.2537011690437794e-03</threshold>
            <left_val>0.6999592185020447</left_val>
            <right_val>-0.1180569007992744</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 0 16 6 -1.</_>
                <_>
                  1 3 16 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1222786009311676</threshold>
            <left_val>0.5207250118255615</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  2 0 14 6 -1.</_>
                <_>
                  2 2 14 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0641681104898453</threshold>
            <left_val>0.3927274942398071</left_val>
            <right_val>-0.4219441115856171</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 11 2 9 -1.</_>
                <_>
                  1 11 1 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.3712888620793819e-04</threshold>
            <left_node>1</left_node>
            <right_val>-0.4952454864978790</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  18 11 1 8 -1.</_>
                <_>
                  18 11 1 4 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-2.8175620827823877e-03</threshold>
            <left_val>0.4135014116764069</left_val>
            <right_val>-0.3891927897930145</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 12 3 2 -1.</_>
                <_>
                  11 12 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.6368549335747957e-03</threshold>
            <left_val>0.6761502027511597</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  11 13 3 1 -1.</_>
                <_>
                  12 13 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.3223909772932529e-03</threshold>
            <left_val>0.4342699944972992</left_val>
            <right_val>-0.3764213025569916</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 0 4 8 -1.</_>
                <_>
                  17 0 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.7143539520911872e-04</threshold>
            <left_val>-0.5563088059425354</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  12 17 4 3 -1.</_>
                <_>
                  14 17 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.0255712121725082e-03</threshold>
            <left_val>-0.5232859253883362</left_val>
            <right_val>0.3464682102203369</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 17 1 2 -1.</_>
                <_>
                  15 17 1 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-9.2711612523999065e-05</threshold>
            <left_node>1</left_node>
            <right_val>-0.4965266883373260</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  15 16 1 3 -1.</_>
                <_>
                  14 17 1 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>1.9847028888761997e-03</threshold>
            <left_val>0.3340164124965668</left_val>
            <right_val>-0.6244689226150513</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 0 14 8 -1.</_>
                <_>
                  3 2 14 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0472034402191639</threshold>
            <left_node>1</left_node>
            <right_val>0.5756261944770813</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  18 1 1 2 -1.</_>
                <_>
                  18 2 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.8562600063160062e-05</threshold>
            <left_val>0.0261726602911949</left_val>
            <right_val>-0.6084907054901123</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 0 8 3 -1.</_>
                <_>
                  8 0 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.5034219771623611e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.6857675909996033</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  9 4 1 9 -1.</_>
                <_>
                  9 7 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.3834791071712971e-03</threshold>
            <left_val>-0.1731251031160355</left_val>
            <right_val>0.3856042921543121</right_val></_></_></trees>
      <stage_threshold>-2.0604379177093506</stage_threshold>
      <parent>4</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 6 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 13 9 2 -1.</_>
                <_>
                  9 13 3 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0155844502151012</threshold>
            <left_node>1</left_node>
            <right_val>-0.6664896011352539</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 13 5 6 -1.</_>
                <_>
                  0 16 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0145570198073983</threshold>
            <left_val>-0.4374513030052185</left_val>
            <right_val>0.7222781777381897</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  13 12 6 4 -1.</_>
                <_>
                  15 12 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.7889888994395733e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.4318324029445648</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 6 12 2 -1.</_>
                <_>
                  8 10 4 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0819367691874504</threshold>
            <left_val>0.6846765279769897</left_val>
            <right_val>-0.2254672944545746</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  19 0 1 8 -1.</_>
                <_>
                  19 4 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.2995368130505085e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.5240963101387024</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 2 12 8 -1.</_>
                <_>
                  11 2 6 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0137366401031613</threshold>
            <left_val>0.6162620782852173</left_val>
            <right_val>-0.3589316010475159</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 12 4 4 -1.</_>
                <_>
                  2 12 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.8069912008941174e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.4238238930702209</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  7 8 13 9 -1.</_>
                <_>
                  7 11 13 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0771310999989510</threshold>
            <left_val>0.6059936285018921</left_val>
            <right_val>-0.3155533075332642</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 1 2 6 -1.</_>
                <_>
                  19 1 1 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.4640208943746984e-04</threshold>
            <left_val>-0.4920611083507538</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  7 4 5 8 -1.</_>
                <_>
                  7 6 5 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0348415784537792</threshold>
            <left_val>-0.0410178899765015</left_val>
            <right_val>0.6133087873458862</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  11 18 9 2 -1.</_>
                <_>
                  11 19 9 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.2969048526138067e-04</threshold>
            <left_val>-0.4547941982746124</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  10 7 2 3 -1.</_>
                <_>
                  11 7 1 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.8510129242204130e-05</threshold>
            <left_val>0.4000732898712158</left_val>
            <right_val>-0.2088876962661743</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 18 6 2 -1.</_>
                <_>
                  6 18 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.6054688282310963e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.6793137788772583</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  6 13 6 7 -1.</_>
                <_>
                  8 13 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.1904482319951057e-03</threshold>
            <left_val>0.4706067144870758</left_val>
            <right_val>-0.1413861066102982</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 18 6 2 -1.</_>
                <_>
                  7 18 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.5724480189383030e-03</threshold>
            <left_val>-0.7052550911903381</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  18 5 2 2 -1.</_>
                <_>
                  18 6 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.0458237314596772e-04</threshold>
            <left_val>0.3609785139560699</left_val>
            <right_val>-0.1836154013872147</right_val></_></_>
        <_>
          <!-- tree 8 -->
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

