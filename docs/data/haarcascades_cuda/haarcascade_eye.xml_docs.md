# Documentation for `data/haarcascades_cuda/haarcascade_eye.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_eye.xml`
- **File Name**: `haarcascade_eye.xml`
- **File Size**: 506,314 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_eye.xml](../../data/haarcascades_cuda/haarcascade_eye.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    Stump-based 20x20 frontal eye detector.
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
<haarcascade_frontaleye type_id="opencv-haar-classifier">
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
                  0 8 20 12 -1.</_>
                <_>
                  0 14 20 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1296395957469940</threshold>
            <left_val>-0.7730420827865601</left_val>
            <right_val>0.6835014820098877</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 1 4 15 -1.</_>
                <_>
                  9 6 4 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0463268086314201</threshold>
            <left_val>0.5735275149345398</left_val>
            <right_val>-0.4909768998622894</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 10 9 2 -1.</_>
                <_>
                  9 10 3 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0161730907857418</threshold>
            <left_val>0.6025434136390686</left_val>
            <right_val>-0.3161070942878723</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 0 10 9 -1.</_>
                <_>
                  7 3 10 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0458288416266441</threshold>
            <left_val>0.6417754888534546</left_val>
            <right_val>-0.1554504036903381</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  12 2 2 18 -1.</_>
                <_>
                  12 8 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0537596195936203</threshold>
            <left_val>0.5421931743621826</left_val>
            <right_val>-0.2048082947731018</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 6 8 6 -1.</_>
                <_>
                  8 9 8 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0341711901128292</threshold>
            <left_val>-0.2338819056749344</left_val>
            <right_val>0.4841090142726898</right_val></_></_></trees>
      <stage_threshold>-1.4562760591506958</stage_threshold>
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
                  2 0 17 18 -1.</_>
                <_>
                  2 6 17 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.2172762006521225</threshold>
            <left_val>0.7109889984130859</left_val>
            <right_val>-0.5936073064804077</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 10 1 8 -1.</_>
                <_>
                  10 14 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0120719699189067</threshold>
            <left_val>-0.2824048101902008</left_val>
            <right_val>0.5901355147361755</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 10 9 2 -1.</_>
                <_>
                  10 10 3 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0178541392087936</threshold>
            <left_val>0.5313752293586731</left_val>
            <right_val>-0.2275896072387695</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 1 6 6 -1.</_>
                <_>
                  5 3 6 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0223336108028889</threshold>
            <left_val>-0.1755609959363937</left_val>
            <right_val>0.6335613727569580</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 1 15 9 -1.</_>
                <_>
                  3 4 15 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0914200171828270</threshold>
            <left_val>0.6156309247016907</left_val>
            <right_val>-0.1689953058958054</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 3 9 6 -1.</_>
                <_>
                  6 5 9 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0289736501872540</threshold>
            <left_val>-0.1225007995963097</left_val>
            <right_val>0.7440117001533508</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 17 6 3 -1.</_>
                <_>
                  10 17 2 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.8203463926911354e-003</threshold>
            <left_val>0.1697437018156052</left_val>
            <right_val>-0.6544165015220642</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 10 9 1 -1.</_>
                <_>
                  12 10 3 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0203404892235994</threshold>
            <left_val>-0.1255664974451065</left_val>
            <right_val>0.8271045088768005</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 7 6 11 -1.</_>
                <_>
                  3 7 2 11 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0119261499494314</threshold>
            <left_val>0.3860568106174469</left_val>
            <right_val>-0.2099234014749527</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 18 3 1 -1.</_>
                <_>
                  10 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.7281101625412703e-004</threshold>
            <left_val>-0.6376119256019592</left_val>
            <right_val>0.1295239031314850</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 16 1 2 -1.</_>
                <_>
                  16 17 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.8322050891583785e-005</threshold>
            <left_val>-0.3463147878646851</left_val>
            <right_val>0.2292426973581314</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 17 6 3 -1.</_>
                <_>
                  11 17 2 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.0854417756199837e-003</threshold>
            <left_val>-0.6366580128669739</left_val>
            <right_val>0.1307865977287293</right_val></_></_></trees>
      <stage_threshold>-1.2550230026245117</stage_threshold>
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
                  8 0 5 18 -1.</_>
                <_>
                  8 6 5 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1181226968765259</threshold>
            <left_val>0.6784452199935913</left_val>
            <right_val>-0.5004578232765198</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 7 9 7 -1.</_>
                <_>
                  9 7 3 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0343327596783638</threshold>
            <left_val>0.6718636155128479</left_val>
            <right_val>-0.3574487864971161</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 6 6 10 -1.</_>
                <_>
                  16 6 2 10 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0215307995676994</threshold>
            <left_val>0.7222070097923279</left_val>
            <right_val>-0.1819241940975189</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 8 9 5 -1.</_>
                <_>
                  12 8 3 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0219099707901478</threshold>
            <left_val>0.6652938723564148</left_val>
            <right_val>-0.2751022875308991</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 7 9 6 -1.</_>
                <_>
                  6 7 3 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0287135392427444</threshold>
            <left_val>0.6995570063591003</left_val>
            <right_val>-0.1961558014154434</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 7 6 6 -1.</_>
                <_>
                  3 7 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0114674801006913</threshold>
            <left_val>0.5926734805107117</left_val>
            <right_val>-0.2209735065698624</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 0 4 18 -1.</_>
                <_>
                  16 6 4 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0226111691445112</threshold>
            <left_val>0.3448306918144226</left_val>
            <right_val>-0.3837955892086029</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 17 3 3 -1.</_>
                <_>
                  0 18 3 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.9308089977130294e-003</threshold>
            <left_val>-0.7944571971893311</left_val>
            <right_val>0.1562865972518921</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 0 2 1 -1.</_>
                <_>
                  17 0 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.6419910833938047e-005</threshold>
            <left_val>-0.3089601099491119</left_val>
            <right_val>0.3543108999729157</right_val></_></_></trees>
      <stage_threshold>-1.3728189468383789</stage_threshold>
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
                  0 8 20 12 -1.</_>
                <_>
                  0 14 20 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1988652050495148</threshold>
            <left_val>-0.5286070108413696</left_val>
            <right_val>0.3553672134876251</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 6 9 8 -1.</_>
                <_>
                  9 6 3 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0360089391469955</threshold>
            <left_val>0.4210968911647797</left_val>
            <right_val>-0.3934898078441620</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 3 12 9 -1.</_>
                <_>
                  5 6 12 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0775698497891426</threshold>
            <left_val>0.4799154102802277</left_val>
            <right_val>-0.2512216866016388</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 16 1 2 -1.</_>
                <_>
                  4 17 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.2630853285081685e-005</threshold>
            <left_val>-0.3847548961639404</left_val>
            <right_val>0.3184922039508820</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 10 2 1 -1.</_>
                <_>
                  19 10 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.2773229759186506e-004</threshold>
            <left_val>-0.2642731964588165</left_val>
            <right_val>0.3254724144935608</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 8 6 5 -1.</_>
                <_>
                  11 8 2 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0185748506337404</threshold>
            <left_val>0.4673658907413483</left_val>
            <right_val>-0.1506727039813995</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 2 1 -1.</_>
                <_>
                  1 0 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.0008762122597545e-005</threshold>
            <left_val>0.2931315004825592</left_val>
            <right_val>-0.2536509931087494</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 8 6 6 -1.</_>
                <_>
                  8 8 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0185521300882101</threshold>
            <left_val>0.4627366065979004</left_val>
            <right_val>-0.1314805001020432</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  11 7 6 7 -1.</_>
                <_>
                  13 7 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0130304200574756</threshold>
            <left_val>0.4162721931934357</left_val>
            <right_val>-0.1775148957967758</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  19 14 1 2 -1.</_>
                <_>
                  19 15 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.5694141085259616e-005</threshold>
            <left_val>-0.2803510129451752</left_val>
            <right_val>0.2668074071407318</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 17 1 2 -1.</_>
                <_>
                  6 18 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.7005260451696813e-004</threshold>
            <left_val>-0.2702724933624268</left_val>
            <right_val>0.2398165017366409</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 7 2 7 -1.</_>
                <_>
                  15 7 1 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.3129199873656034e-003</threshold>
            <left_val>0.4441143870353699</left_val>
            <right_val>-0.1442888975143433</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 8 2 4 -1.</_>
                <_>
                  7 8 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.7583490116521716e-003</threshold>
            <left_val>-0.1612619012594223</left_val>
            <right_val>0.4294076859951019</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 8 12 6 -1.</_>
                <_>
                  5 10 12 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0251947492361069</threshold>
            <left_val>0.4068729877471924</left_val>
            <right_val>-0.1820258051156998</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 17 1 3 -1.</_>
                <_>
                  2 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.4031709870323539e-003</threshold>
            <left_val>0.0847597867250443</left_val>
            <right_val>-0.8001856803894043</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 7 3 6 -1.</_>
                <_>
                  7 7 1 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.3991729877889156e-003</threshold>
            <left_val>0.5576609969139099</left_val>
            <right_val>-0.1184315979480743</right_val></_></_></trees>
      <stage_threshold>-1.2879480123519897</stage_threshold>
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
                  6 7 9 12 -1.</_>
                <_>
                  9 7 3 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0299430806189775</threshold>
            <left_val>0.3581081032752991</left_val>
            <right_val>-0.3848763108253479</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 2 11 12 -1.</_>
                <_>
                  6 6 11 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1256738007068634</threshold>
            <left_val>0.3931693136692047</left_val>
            <right_val>-0.3001225888729096</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 12 5 8 -1.</_>
                <_>
                  1 16 5 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.3635272197425365e-003</threshold>
            <left_val>-0.4390861988067627</left_val>
            <right_val>0.1925701051950455</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 7 6 7 -1.</_>
                <_>
                  16 7 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.0971820279955864e-003</threshold>
            <left_val>0.3990666866302490</left_val>
            <right_val>-0.2340787053108215</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 8 6 6 -1.</_>
                <_>
                  12 8 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0165979098528624</threshold>
            <left_val>0.4209528863430023</left_val>
            <right_val>-0.2267484068870544</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 18 4 2 -1.</_>
                <_>
                  16 19 4 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.0199299324303865e-003</threshold>
            <left_val>-0.7415673136711121</left_val>
            <right_val>0.1260118931531906</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 17 2 3 -1.</_>
                <_>
                  18 18 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5202340437099338e-003</threshold>
            <left_val>-0.7615460157394409</left_val>
            <right_val>0.0863736122846603</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 7 3 7 -1.</_>
                <_>
                  10 7 1 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.9663940444588661e-003</threshold>
            <left_val>0.4218223989009857</left_val>
            <right_val>-0.1790491938591003</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 6 6 8 -1.</_>
                <_>
                  7 6 2 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0192076005041599</threshold>
            <left_val>0.4689489901065826</left_val>
            <right_val>-0.1437875032424927</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  2 6 6 11 -1.</_>
                <_>
                  4 6 2 11 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0122226802632213</threshold>
            <left_val>0.3284207880496979</left_val>
            <right_val>-0.2180214971303940</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 10 12 8 -1.</_>
                <_>
                  8 14 12 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0575486682355404</threshold>
            <left_val>-0.3676880896091461</left_val>
            <right_val>0.2435711026191711</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 17 6 3 -1.</_>
                <_>
                  9 17 2 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.5794079825282097e-003</threshold>
            <left_val>-0.7224506735801697</left_val>
            <right_val>0.0636645630002022</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 9 3 3 -1.</_>
                <_>
                  11 9 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.9545740690082312e-003</threshold>
            <left_val>0.3584643900394440</left_val>
            <right_val>-0.1669632941484451</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 8 3 6 -1.</_>
                <_>
                  9 8 1 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.2017991654574871e-003</threshold>
            <left_val>0.3909480869770050</left_val>
            <right_val>-0.1204179003834724</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 0 6 5 -1.</_>
                <_>
                  9 0 2 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0136249903589487</threshold>
            <left_val>-0.5876771807670593</left_val>
            <right_val>0.0884047299623489</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 17 1 3 -1.</_>
                <_>
                  6 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.2853112467564642e-005</threshold>
            <left_val>-0.2634845972061157</left_val>
            <right_val>0.2141927927732468</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 18 4 2 -1.</_>
                <_>
                  0 19 4 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.6782939676195383e-003</threshold>
            <left_val>-0.7839016914367676</left_val>
            <right_val>0.0805269628763199</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 1 11 9 -1.</_>
                <_>
                  4 4 11 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0705971792340279</threshold>
            <left_val>0.4146926105022430</left_val>
            <right_val>-0.1398995965719223</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 1 14 9 -1.</_>
                <_>
                  3 4 14 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0920936465263367</threshold>
            <left_val>-0.1305518001317978</left_val>
            <right_val>0.5043578147888184</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 9 6 4 -1.</_>
                <_>
                  2 9 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.8004386052489281e-003</threshold>
            <left_val>0.3660975098609924</left_val>
            <right_val>-0.1403664946556091</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 13 1 2 -1.</_>
                <_>
                  18 14 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.5080977694597095e-005</threshold>
            <left_val>-0.2970443964004517</left_val>
            <right_val>0.2070294022560120</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  13 5 3 11 -1.</_>
                <_>
                  14 5 1 11 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.9870450962334871e-003</threshold>
            <left_val>0.3561570048332214</left_val>
            <right_val>-0.1544596999883652</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 18 8 2 -1.</_>
                <_>
                  0 18 4 1 2.</_>
                <_>
                  4 19 4 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.6441509835422039e-003</threshold>
            <left_val>-0.5435351729393005</left_val>
            <right_val>0.1029511019587517</right_val></_></_></trees>
      <stage_threshold>-1.2179850339889526</stage_threshold>
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
                  5 8 12 5 -1.</_>
                <_>
                  9 8 4 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0478624701499939</threshold>
            <left_val>0.4152823984622955</left_val>
            <right_val>-0.3418582081794739</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 7 11 10 -1.</_>
                <_>
                  4 12 11 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0873505324125290</threshold>
            <left_val>-0.3874978125095367</left_val>
            <right_val>0.2420420050621033</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 9 6 4 -1.</_>
                <_>
                  16 9 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0168494991958141</threshold>
            <left_val>0.5308247804641724</left_val>
            <right_val>-0.1728291064500809</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 7 6 8 -1.</_>
                <_>
                  3 7 3 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0288700293749571</threshold>
            <left_val>0.3584350943565369</left_val>
            <right_val>-0.2240259051322937</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 16 3 3 -1.</_>
                <_>
                  0 17 3 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.5679389946162701e-003</threshold>
            <left_val>0.1499049961566925</left_val>
            <right_val>-0.6560940742492676</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 11 12 1 -1.</_>
                <_>
                  11 11 4 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0241166595369577</threshold>
            <left_val>0.5588967800140381</left_val>
            <right_val>-0.1481028050184250</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 8 9 4 -1.</_>
                <_>
                  7 8 3 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0328266583383083</threshold>
            <left_val>0.4646868109703064</left_val>
            <right_val>-0.1078552976250649</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 16 6 4 -1.</_>
                <_>
                  7 16 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0152330603450537</threshold>
            <left_val>-0.7395442724227905</left_val>
            <right_val>0.0562368817627430</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 17 1 3 -1.</_>
                <_>
                  18 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.0209511169232428e-004</threshold>
            <left_val>-0.4554882049560547</left_val>
            <right_val>0.0970698371529579</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 17 1 3 -1.</_>
                <_>
                  18 18 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.5365108205005527e-004</threshold>
            <left_val>0.0951472967863083</left_val>
            <right_val>-0.5489501953125000</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 9 4 10 -1.</_>
                <_>
                  4 9 2 5 2.</_>
                <_>
                  6 14 2 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0106389503926039</threshold>
            <left_val>0.4091297090053558</left_val>
            <right_val>-0.1230840981006622</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 8 6 4 -1.</_>
                <_>
                  6 8 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.5217830017209053e-003</threshold>
            <left_val>0.4028914868831635</left_val>
            <right_val>-0.1604878008365631</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 2 2 18 -1.</_>
                <_>
                  10 8 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1067709997296333</threshold>
            <left_val>0.6175932288169861</left_val>
            <right_val>-0.0730911865830421</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 5 8 6 -1.</_>
                <_>
                  0 5 4 3 2.</_>
                <_>
                  4 8 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0162569191306829</threshold>
            <left_val>-0.1310368031263351</left_val>
            <right_val>0.3745365142822266</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 0 6 5 -1.</_>
                <_>
                  8 0 2 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0206793602555990</threshold>
            <left_val>-0.7140290737152100</left_val>
            <right_val>0.0523900091648102</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 0 2 14 -1.</_>
                <_>
                  18 7 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0170523691922426</threshold>
            <left_val>0.1282286047935486</left_val>
            <right_val>-0.3108068108558655</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 18 4 2 -1.</_>
                <_>
                  10 18 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.7122060097754002e-003</threshold>
            <left_val>-0.6055650711059570</left_val>
            <right_val>0.0818847566843033</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 17 6 3 -1.</_>
                <_>
                  1 18 6 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.0851430235779844e-005</threshold>
            <left_val>-0.2681298851966858</left_val>
            <right_val>0.1445384025573731</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  11 8 3 5 -1.</_>
                <_>
                  12 8 1 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.9284431412816048e-003</threshold>
            <left_val>-0.0787953510880470</left_val>
            <right_val>0.5676258206367493</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  11 8 3 4 -1.</_>
                <_>
                  12 8 1 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.5217379443347454e-003</threshold>
            <left_val>0.3706862926483154</left_val>
            <right_val>-0.1362057030200958</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  11 0 6 5 -1.</_>
                <_>
                  13 0 2 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0224261991679668</threshold>
            <left_val>-0.6870499849319458</left_val>
            <right_val>0.0510628595948219</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 7 6 7 -1.</_>
                <_>
                  3 7 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.6451441273093224e-003</threshold>
            <left_val>0.2349222004413605</left_val>
            <right_val>-0.1790595948696137</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 13 1 3 -1.</_>
                <_>
                  0 14 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.1175329564139247e-003</threshold>
            <left_val>-0.5986905097961426</left_val>
            <right_val>0.0743244364857674</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 2 9 6 -1.</_>
                <_>
                  3 4 9 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0192127898335457</threshold>
            <left_val>-0.1570255011320114</left_val>
            <right_val>0.2973746955394745</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 6 9 2 -1.</_>
                <_>
                  8 7 9 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.6293429806828499e-003</threshold>
            <left_val>-0.0997690185904503</left_val>
            <right_val>0.4213027060031891</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 14 3 6 -1.</_>
                <_>
                  0 16 3 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.5671862363815308e-003</threshold>
            <left_val>-0.6085879802703857</left_val>
            <right_val>0.0735062584280968</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 11 6 4 -1.</_>
                <_>
                  3 11 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0112179601565003</threshold>
            <left_val>-0.1032081022858620</left_val>
            <right_val>0.4190984964370728</right_val></_></_></trees>
      <stage_threshold>-1.2905240058898926</stage_threshold>
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
                  6 9 9 3 -1.</_>
                <_>
                  9 9 3 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0174864400178194</threshold>
            <left_val>0.3130728006362915</left_val>
            <right_val>-0.3368118107318878</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 0 9 6 -1.</_>
                <_>
                  6 2 9 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0307146497070789</threshold>
            <left_val>-0.1876619011163712</left_val>
            <right_val>0.5378080010414124</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 5 6 6 -1.</_>
                <_>
                  8 7 6 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0221887193620205</threshold>
            <left_val>0.3663788139820099</left_val>
            <right_val>-0.1612481027841568</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 12 2 1 -1.</_>
                <_>
                  2 12 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.0700771680567414e-005</threshold>
            <left_val>0.2124571055173874</left_val>
            <right_val>-0.2844462096691132</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 10 6 2 -1.</_>
                <_>
                  12 10 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.0170420221984386e-003</threshold>
            <left_val>0.3954311013221741</left_val>
            <right_val>-0.1317359060049057</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  13 8 6 6 -1.</_>
                <_>
                  15 8 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.8563609384000301e-003</threshold>
            <left_val>0.3037385940551758</left_val>
            <right_val>-0.2065781950950623</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 16 6 4 -1.</_>
                <_>
                  8 16 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0141292596235871</threshold>
            <left_val>-0.7650300860404968</left_val>
            <right_val>0.0982131883502007</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 0 9 9 -1.</_>
                <_>
                  8 3 9 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0479154810309410</threshold>
            <left_val>0.4830738902091980</left_val>
            <right_val>-0.1300680935382843</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 17 1 3 -1.</_>
    
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

