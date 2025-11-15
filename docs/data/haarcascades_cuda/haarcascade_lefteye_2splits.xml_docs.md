# Documentation for `data/haarcascades_cuda/haarcascade_lefteye_2splits.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_lefteye_2splits.xml`
- **File Name**: `haarcascade_lefteye_2splits.xml`
- **File Size**: 323,226 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_lefteye_2splits.xml](../../data/haarcascades_cuda/haarcascade_lefteye_2splits.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    Tree-based 20x20 left eye detector.
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
<haarcascade_lefteye type_id="opencv-haar-classifier">
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
                  8 12 3 8 -1.</_>
                <_>
                  8 16 3 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0273259896785021</threshold>
            <left_val>-0.9060062170028687</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 11 8 9 -1.</_>
                <_>
                  7 11 4 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.0568458177149296e-03</threshold>
            <left_val>0.9338570833206177</left_val>
            <right_val>-0.4585995972156525</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 7 11 12 -1.</_>
                <_>
                  8 11 11 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1253869980573654</threshold>
            <left_val>0.7246372103691101</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  1 0 7 8 -1.</_>
                <_>
                  1 4 7 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1148729994893074</threshold>
            <left_val>0.5303416848182678</left_val>
            <right_val>-0.8322122097015381</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 7 6 6 -1.</_>
                <_>
                  7 9 6 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0583099387586117</threshold>
            <left_val>0.6540889143943787</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 0 7 4 -1.</_>
                <_>
                  0 2 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0176843702793121</threshold>
            <left_val>0.2948287129402161</left_val>
            <right_val>-0.7480958104133606</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 13 4 4 -1.</_>
                <_>
                  18 13 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.5937170032411814e-03</threshold>
            <left_val>-0.5030391812324524</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  17 15 2 3 -1.</_>
                <_>
                  17 15 1 3 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-1.3436110457405448e-03</threshold>
            <left_val>0.6599534153938293</left_val>
            <right_val>-0.5574085712432861</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 13 6 2 -1.</_>
                <_>
                  2 13 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1795940119773149e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.4201635122299194</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 0 6 6 -1.</_>
                <_>
                  7 0 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0115148704499006</threshold>
            <left_val>0.5969433188438416</left_val>
            <right_val>-0.8050804734230042</right_val></_></_></trees>
      <stage_threshold>-2.3924100399017334</stage_threshold>
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
                  5 7 9 12 -1.</_>
                <_>
                  8 11 3 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.2248556017875671</threshold>
            <left_node>1</left_node>
            <right_val>-0.8136320114135742</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 6 4 10 -1.</_>
                <_>
                  5 6 2 5 2.</_>
                <_>
                  7 11 2 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.6008004620671272e-03</threshold>
            <left_val>0.9086313843727112</left_val>
            <right_val>-0.3220897018909454</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 12 11 8 -1.</_>
                <_>
                  8 16 11 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0742191672325134</threshold>
            <left_val>-0.7532945275306702</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 0 1 8 -1.</_>
                <_>
                  0 4 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.3165741264820099e-03</threshold>
            <left_val>0.8633949756622314</left_val>
            <right_val>-0.0334635712206364</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 6 6 -1.</_>
                <_>
                  3 0 3 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1913449745625257e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.5572034716606140</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  14 14 6 6 -1.</_>
                <_>
                  14 17 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0118009597063065</threshold>
            <left_val>-0.3235968053340912</left_val>
            <right_val>0.6416382193565369</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 13 9 7 -1.</_>
                <_>
                  8 13 3 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.6179709285497665e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.5316786766052246</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  6 17 6 3 -1.</_>
                <_>
                  8 17 2 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.0587511658668518e-03</threshold>
            <left_val>-0.7361145019531250</left_val>
            <right_val>0.5566077232360840</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 4 4 -1.</_>
                <_>
                  0 2 4 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.9959779717028141e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.4147691130638123</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  1 0 3 3 -1.</_>
                <_>
                  2 1 1 1 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.0803930759429932e-03</threshold>
            <left_val>0.5927835702896118</left_val>
            <right_val>-0.6738492250442505</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 18 6 2 -1.</_>
                <_>
                  3 19 6 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9909010734409094e-03</threshold>
            <left_val>-0.4214592874050140</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  7 18 4 2 -1.</_>
                <_>
                  8 18 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.6845749923959374e-03</threshold>
            <left_val>0.5467922091484070</left_val>
            <right_val>-0.7509945034980774</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 10 12 2 -1.</_>
                <_>
                  6 11 12 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.0781872123479843e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.3989954888820648</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  15 8 3 1 -1.</_>
                <_>
                  16 9 1 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>2.6645609177649021e-03</threshold>
            <left_val>0.5894060134887695</left_val>
            <right_val>-0.4677804112434387</right_val></_></_></trees>
      <stage_threshold>-2.6498730182647705</stage_threshold>
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
                  5 7 9 12 -1.</_>
                <_>
                  8 11 3 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.2530143857002258</threshold>
            <left_node>1</left_node>
            <right_val>-0.7540258765220642</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 13 1 6 -1.</_>
                <_>
                  16 16 1 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.9663778841495514e-03</threshold>
            <left_val>-0.3527964949607849</left_val>
            <right_val>0.8799229860305786</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 7 5 6 -1.</_>
                <_>
                  7 9 5 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0471276491880417</threshold>
            <left_node>1</left_node>
            <right_val>-0.5223489999771118</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 12 4 6 -1.</_>
                <_>
                  18 12 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9500750349834561e-03</threshold>
            <left_val>-0.3037990927696228</left_val>
            <right_val>0.7520437836647034</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 6 8 -1.</_>
                <_>
                  0 4 6 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0714810267090797</threshold>
            <left_val>0.6584190130233765</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  3 1 15 12 -1.</_>
                <_>
                  3 5 15 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2218973040580750</threshold>
            <left_val>-0.6090720295906067</left_val>
            <right_val>0.5684216022491455</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  11 12 9 8 -1.</_>
                <_>
                  11 16 9 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0338428206741810</threshold>
            <left_val>-0.6431164741516113</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 0 12 9 -1.</_>
                <_>
                  4 0 4 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.1714561413973570e-04</threshold>
            <left_val>0.5462036132812500</left_val>
            <right_val>-0.3998414874076843</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 12 6 4 -1.</_>
                <_>
                  2 12 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.4458211157470942e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.4563683867454529</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  10 18 4 2 -1.</_>
                <_>
                  11 18 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.4395729415118694e-03</threshold>
            <left_val>0.4779818952083588</left_val>
            <right_val>-0.9124708771705627</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 2 3 3 -1.</_>
                <_>
                  6 2 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.1385070867836475e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.8361775875091553</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  12 18 3 2 -1.</_>
                <_>
                  13 18 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.8324409611523151e-03</threshold>
            <left_val>0.3346279859542847</left_val>
            <right_val>-0.7500854730606079</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 2 8 -1.</_>
                <_>
                  1 0 1 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.1167610064148903e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.6908379793167114</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 18 4 2 -1.</_>
                <_>
                  5 19 4 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.9106997367925942e-05</threshold>
            <left_val>-0.3456133008003235</left_val>
            <right_val>0.4118317961692810</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 11 6 6 -1.</_>
                <_>
                  17 11 3 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0154477702453732</threshold>
            <left_node>1</left_node>
            <right_val>0.3698019087314606</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  6 12 8 4 -1.</_>
                <_>
                  8 12 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0322449393570423</threshold>
            <left_val>0.6111283898353577</left_val>
            <right_val>-0.5568534135818481</right_val></_></_></trees>
      <stage_threshold>-2.3828399181365967</stage_threshold>
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
                  12 6 4 9 -1.</_>
                <_>
                  9 9 4 3 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.1225112974643707</threshold>
            <left_node>1</left_node>
            <right_val>-0.6702662706375122</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  11 9 4 7 -1.</_>
                <_>
                  12 10 2 7 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0142306098714471</threshold>
            <left_val>0.8780239224433899</left_val>
            <right_val>-0.1878418028354645</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 8 4 8 -1.</_>
                <_>
                  5 8 2 4 2.</_>
                <_>
                  7 12 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.9833219274878502e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.5812284946441650</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 12 11 8 -1.</_>
                <_>
                  8 16 11 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0770851373672485</threshold>
            <left_val>-0.5039535164833069</left_val>
            <right_val>0.6738736033439636</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 0 14 6 -1.</_>
                <_>
                  3 3 14 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1108618974685669</threshold>
            <left_val>0.6343203783035278</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  7 1 6 12 -1.</_>
                <_>
                  7 4 6 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0946047604084015</threshold>
            <left_val>-0.4972639083862305</left_val>
            <right_val>0.3878743946552277</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 18 7 2 -1.</_>
                <_>
                  0 19 7 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.7696130089461803e-04</threshold>
            <left_val>-0.6393880248069763</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 12 4 3 -1.</_>
                <_>
                  18 12 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.0120320841670036e-03</threshold>
            <left_val>-0.3531391024589539</left_val>
            <right_val>0.5153843760490417</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 4 8 -1.</_>
                <_>
                  2 0 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.6102839726954699e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.5191590189933777</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  3 0 4 1 -1.</_>
                <_>
                  5 0 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.6666069859638810e-03</threshold>
            <left_val>0.4047819077968597</left_val>
            <right_val>-0.6949635744094849</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 13 2 2 -1.</_>
                <_>
                  3 13 2 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-7.1480998303741217e-04</threshold>
            <left_node>1</left_node>
            <right_val>-0.4894518852233887</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 16 19 4 -1.</_>
                <_>
                  0 18 19 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.7647571191191673e-03</threshold>
            <left_val>-0.5003775954246521</left_val>
            <right_val>0.4079605937004089</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 13 8 2 -1.</_>
                <_>
                  11 13 4 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.8659597784280777e-03</threshold>
            <left_val>-0.3363642990589142</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 8 4 1 -1.</_>
                <_>
                  9 8 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.2938310392200947e-03</threshold>
            <left_val>-0.6762138009071350</left_val>
            <right_val>0.4701024889945984</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 1 1 4 -1.</_>
                <_>
                  0 3 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.6533139063976705e-04</threshold>
            <left_node>1</left_node>
            <right_val>-0.4707160890102386</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 0 1 4 -1.</_>
                <_>
                  0 1 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.0565679296851158e-03</threshold>
            <left_val>0.4132341146469116</left_val>
            <right_val>-0.5552641749382019</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 15 5 2 -1.</_>
                <_>
                  15 16 5 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.8385717642959207e-05</threshold>
            <left_val>-0.5152115821838379</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  7 18 3 2 -1.</_>
                <_>
                  8 18 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.7511800397187471e-03</threshold>
            <left_val>0.3341724872589111</left_val>
            <right_val>-0.7955815792083740</right_val></_></_></trees>
      <stage_threshold>-2.1312201023101807</stage_threshold>
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
                  13 7 3 8 -1.</_>
                <_>
                  11 9 3 4 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0646952390670776</threshold>
            <left_node>1</left_node>
            <right_val>-0.6132640242576599</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  15 12 2 8 -1.</_>
                <_>
                  15 16 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.5212170854210854e-03</threshold>
            <left_val>-0.5483155846595764</left_val>
            <right_val>0.7865244746208191</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 0 10 6 -1.</_>
                <_>
                  2 3 10 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0981097668409348</threshold>
            <left_val>0.6911330819129944</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 5 18 15 -1.</_>
                <_>
                  6 10 6 5 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.8593845963478088</threshold>
            <left_val>0.4536468088626862</left_val>
            <right_val>-0.5002614855766296</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 11 12 6 -1.</_>
                <_>
                  7 13 4 2 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0898361727595329</threshold>
            <left_node>1</left_node>
            <right_val>-0.5292878150939941</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 12 4 7 -1.</_>
                <_>
                  18 12 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.6945930439978838e-03</threshold>
            <left_val>-0.3819977939128876</left_val>
            <right_val>0.5782129764556885</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 18 4 2 -1.</_>
                <_>
                  9 18 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.5973599404096603e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.9192836880683899</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 17 4 3 -1.</_>
                <_>
                  9 17 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.0058110132813454e-03</threshold>
            <left_val>-0.8021379709243774</left_val>
            <right_val>0.2925927937030792</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 12 6 6 -1.</_>
                <_>
                  2 12 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.5496290549635887e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.4367895126342773</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 16 4 4 -1.</_>
                <_>
                  5 16 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.7376728616654873e-03</threshold>
            <left_val>0.4101088047027588</left_val>
            <right_val>-0.7269281148910522</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 0 4 6 -1.</_>
                <_>
                  4 0 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.6190437860786915e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.8489515185356140</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  1 0 4 7 -1.</_>
                <_>
                  2 0 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.5377281494438648e-03</threshold>
            <left_val>0.3012467920780182</left_val>
            <right_val>-0.7030177116394043</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 0 8 3 -1.</_>
                <_>
                  6 0 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.4952790699899197e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.4678474962711334</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 3 4 6 -1.</_>
                <_>
                  9 3 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.1753767766058445e-03</threshold>
            <left_val>-0.7453035116195679</left_val>
            <right_val>0.4001182019710541</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 10 3 2 -1.</_>
                <_>
                  10 11 3 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.2049742080271244e-03</threshold>
            <left_val>0.4866926968097687</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 3 7 6 -1.</_>
                <_>
                  4 6 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0878920033574104</threshold>
            <left_val>0.8349394798278809</left_val>
            <right_val>-0.3382771909236908</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 18 10 2 -1.</_>
                <_>
                  15 18 5 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.9997250102460384e-03</threshold>
            <left_val>-0.2903988957405090</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  9 13 6 1 -1.</_>
                <_>
                  9 13 3 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-9.0990252792835236e-03</threshold>
            <left_val>0.6231582164764404</left_val>
            <right_val>-0.3542473018169403</right_val></_></_></trees>
      <stage_threshold>-2.0176210403442383</stage_threshold>
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
                  10 8 4 6 -1.</_>
                <_>
                  8 10 4 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0557021014392376</threshold>
            <left_node>1</left_node>
            <right_val>-0.6984158158302307</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  14 12 6 8 -1.</_>
                <_>
                  14 16 6 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0340332910418510</threshold>
            <left_val>-0.3950918912887573</left_val>
            <right_val>0.8031312823295593</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 8 6 4 -1.</_>
                <_>
                  12 10 2 4 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0461990609765053</threshold>
            <left_node>1</left_node>
            <right_val>-0.4886038005352020</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 12 6 3 -1.</_>
                <_>
                  2 12 2 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.8061669804155827e-03</threshold>
            <left_val>0.8077561259269714</left_val>
            <right_val>-0.0744908228516579</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 11 2 6 -1.</_>
                <_>
                  19 11 1 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.8170489929616451e-03</threshold>
            <left_val>-0.3804352879524231</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 0 1 10 -1.</_>
                <_>
                  0 5 1 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.6162370815873146e-03</threshold>
            <left_val>0.6045172214508057</left_val>
            <right_val>-0.2258224040269852</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 4 8 12 -1.</_>
                <_>
                  7 4 4 12 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0157069507986307</threshold>
            <left_node>1</left_node>
            <right_val>-0.3757799863815308</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  1 3 9 8 -1.</_>
                <_>
                  4 3 3 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.3929950334131718e-03</threshold>
            <left_val>0.5421422123908997</left_val>
            <right_val>-0.3738824129104614</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 2 2 -1.</_>
                <_>
                  0 1 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.0047219984699041e-04</threshold>
            <left_node>1</left_node>
            <right_val>-0.4743340909481049</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  12 8 6 12 -1.</_>
                <_>
                  14 12 2 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0864751189947128</threshold>
            <left_val>0.5018631815910339</left_val>
            <right_val>-0.2113623023033142</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 2 14 6 -1.</_>
                <_>
                  4 4 14 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0779607668519020</threshold>
            <left_val>0.5733734965324402</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  3 0 12 8 -1.</_>
                <_>
                  3 4 12 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0985612869262695</threshold>
            <left_val>-0.3251555860042572</left_val>
            <right_val>0.5303598046302795</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 17 20 -1.</_>
                <_>
                  0 5 17 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.5435916781425476</threshold>
            <left_val>0.5946429967880249</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 0 13 6 -1.</_>
                <_>
                  4 2 13 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0441776998341084</threshold>
            <left_val>0.2967107892036438</left_val>
            <right_val>-0.3847483098506927</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 10 3 6 -1.</_>
                <_>
                  3 10 1 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.8016409426927567e-04</threshold>
            <left_node>1</left_node>
            <right_val>-0.3200058937072754</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  4 14 6 4 -1.</_>
                <_>
                  4 14 3 2 2.</_>
                <_>
                  7 16 3 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.6359390467405319e-03</threshold>
            <left_val>-0.1758614033460617</left_val>
            <right_val>0.4836035072803497</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 1 6 8 -1.</_>
                <_>
                  10 1 2 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0142036899924278</threshold>
            <left_val>-0.7788208723068237</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 1 2 6 -1.</_>
                <_>
                  1 1 1 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.3902818257920444e-05</threshold>
            <left_val>0.3061941862106323</left_val>
            <right_val>-0.3319604992866516</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 12 1 3 -1.</_>
                <_>
                  7 13 1 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>4.6157240867614746e-03</threshold>
            <left_node>1</left_node>
            <right_val>0.4968977868556976</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 4 8 4 -1.</_>
                <_>
                  5 4 8 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0111523102968931</threshold>
            <left_val>-0.5343589186668396</left_val>
            <right_val>0.0972294434905052</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 2 4 5 -1.</_>
                <_>
                  1 2 2 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.0547702014446259e-03</threshold>
            <left_val>-0.8381121754646301</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  5 12 3 2 -1.</_>
                <_>
                  6 12 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1118740551173687e-03</threshold>
            <left_val>0.6361703276634216</left_val>
            <right_val>-0.0482991896569729</right_val></_></_></trees>
      <stage_threshold>-2.2212049961090088</stage_threshold>
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
                  5 13 8 2 -1.</_>
                <_>
                  7 13 4 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0129568297415972</threshold>
            <left_node>1</left_node>
            <right_val>-0.6487473249435425</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  11 9 9 8 -1.</_>
                <_>
                  11 11 9 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0271410197019577</threshold>
            <left_val>0.7629305720329285</left_val>
            <right_val>-0.3394787013530731</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 12 4 3 -1.</_>
                <_>
                  18 12 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.5119998976588249e-03</threshold>
            <left_val>-0.5005983710289001</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  16 14 4 6 -1.</_>
                <_>
                  16 17 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0125166904181242</threshold>
            <left_val>-0.3687332868576050</left_val>
            <right_val>0.5988863110542297</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 12 6 3 -1.</_>
                <_>
                  2 12 2 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.0557941906154156e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.3894093036651611</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 6 7 6 -1.</_>
                <_>
                  6 8 7 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0469237491488457</threshold>
            <left_val>0.6326891183853149</left_val>
            <right_val>-0.2627002894878387</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 1 6 -1.</_>
                <_>
                  0 3 1 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.4018269032239914e-03</threshold>
            <left_node>1</left_node>
            <right_val>-0.5051792860031128</right_val></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  0 2 15 5 -1.</_>
                <_>
                  5 2 5 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0159360896795988</threshold>
            <left_val>0.6552600264549255</left_val>
            <right_val>-0.1730810999870300</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 11 10 3 -1.</_>
                <_>
                  13 11 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0140002900734544</threshold>
            <left_val>-0.4165323078632355</left_val>
            <right_node>1</right_node></_>
          <_>
            <!-- node 1 -->
            <feature>
              <rects>
                <_>
                  8 11 2 8 -1.</_>
                <_>
                  8 15 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0132027799263597</threshold>
            <left_val>-0.4912196993827820</left_val>
            <right_val>0.3739793896675110</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <
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

