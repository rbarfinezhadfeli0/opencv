# Documentation for `data/haarcascades_cuda/haarcascade_frontalface_default.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_frontalface_default.xml`
- **File Name**: `haarcascade_frontalface_default.xml`
- **File Size**: 1,254,733 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_frontalface_default.xml](../../data/haarcascades_cuda/haarcascade_frontalface_default.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    Stump-based 24x24 discrete(?) adaboost frontal face detector.
    Created by Rainer Lienhart.

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
<haarcascade_frontalface_default type_id="opencv-haar-classifier">
  <size>24 24</size>
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
                <_>6 4 12 9 -1.</_>
                <_>6 7 12 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0315119996666908</threshold>
            <left_val>2.0875380039215088</left_val>
            <right_val>-2.2172100543975830</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 4 12 7 -1.</_>
                <_>10 4 4 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0123960003256798</threshold>
            <left_val>-1.8633940219879150</left_val>
            <right_val>1.3272049427032471</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 9 18 9 -1.</_>
                <_>3 12 18 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0219279993325472</threshold>
            <left_val>-1.5105249881744385</left_val>
            <right_val>1.0625729560852051</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 18 9 6 -1.</_>
                <_>8 20 9 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.7529998011887074e-003</threshold>
            <left_val>-0.8746389746665955</left_val>
            <right_val>1.1760339736938477</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 5 4 19 -1.</_>
                <_>5 5 2 19 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0150140002369881</threshold>
            <left_val>-0.7794569730758667</left_val>
            <right_val>1.2608419656753540</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 5 12 16 -1.</_>
                <_>6 13 12 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0993710011243820</threshold>
            <left_val>0.5575129985809326</left_val>
            <right_val>-1.8743000030517578</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 8 12 6 -1.</_>
                <_>5 11 12 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.7340000960975885e-003</threshold>
            <left_val>-1.6911929845809937</left_val>
            <right_val>0.4400970041751862</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 14 4 10 -1.</_>
                <_>11 19 4 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0188590008765459</threshold>
            <left_val>-1.4769539833068848</left_val>
            <right_val>0.4435009956359863</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 0 7 6 -1.</_>
                <_>4 3 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.9739998541772366e-003</threshold>
            <left_val>-0.8590919971466065</left_val>
            <right_val>0.8525559902191162</right_val></_></_></trees>
      <stage_threshold>-5.0425500869750977</stage_threshold>
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
                <_>6 6 12 6 -1.</_>
                <_>6 8 12 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0211100000888109</threshold>
            <left_val>1.2435649633407593</left_val>
            <right_val>-1.5713009834289551</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 4 12 7 -1.</_>
                <_>10 4 4 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0203559994697571</threshold>
            <left_val>-1.6204780340194702</left_val>
            <right_val>1.1817760467529297</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 8 19 12 -1.</_>
                <_>1 12 19 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0213089995086193</threshold>
            <left_val>-1.9415930509567261</left_val>
            <right_val>0.7006909847259522</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 2 24 3 -1.</_>
                <_>8 2 8 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0916600003838539</threshold>
            <left_val>-0.5567010045051575</left_val>
            <right_val>1.7284419536590576</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 9 6 15 -1.</_>
                <_>9 14 6 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0362880006432533</threshold>
            <left_val>0.2676379978656769</left_val>
            <right_val>-2.1831810474395752</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 14 10 -1.</_>
                <_>5 11 14 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0191099997609854</threshold>
            <left_val>-2.6730210781097412</left_val>
            <right_val>0.4567080140113831</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 0 14 9 -1.</_>
                <_>5 3 14 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.2539999857544899e-003</threshold>
            <left_val>-1.0852910280227661</left_val>
            <right_val>0.5356420278549194</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 11 9 6 -1.</_>
                <_>16 11 3 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0183550007641315</threshold>
            <left_val>-0.3520019948482513</left_val>
            <right_val>0.9333919882774353</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 5 6 10 -1.</_>
                <_>9 5 2 10 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.0569999516010284e-003</threshold>
            <left_val>0.9278209805488586</left_val>
            <right_val>-0.6634989976882935</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 8 6 10 -1.</_>
                <_>12 8 2 10 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.8770000040531158e-003</threshold>
            <left_val>1.1577470302581787</left_val>
            <right_val>-0.2977479994297028</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 5 4 9 -1.</_>
                <_>4 5 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0158140007406473</threshold>
            <left_val>-0.4196060001850128</left_val>
            <right_val>1.3576040267944336</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>18 0 6 11 -1.</_>
                <_>20 0 2 11 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0207000002264977</threshold>
            <left_val>1.4590020179748535</left_val>
            <right_val>-0.1973939985036850</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 6 24 13 -1.</_>
                <_>8 6 8 13 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1376080065965653</threshold>
            <left_val>1.1186759471893311</left_val>
            <right_val>-0.5291550159454346</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 6 6 9 -1.</_>
                <_>11 6 2 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0143189998343587</threshold>
            <left_val>-0.3512719869613648</left_val>
            <right_val>1.1440860033035278</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 18 10 6 -1.</_>
                <_>7 20 10 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0102530000731349</threshold>
            <left_val>-0.6085060238838196</left_val>
            <right_val>0.7709850072860718</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 7 14 12 -1.</_>
                <_>5 13 14 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0915080010890961</threshold>
            <left_val>0.3881779909133911</left_val>
            <right_val>-1.5122940540313721</right_val></_></_></trees>
      <stage_threshold>-4.9842400550842285</stage_threshold>
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
                <_>0 3 24 3 -1.</_>
                <_>8 3 8 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0697470009326935</threshold>
            <left_val>-1.0130879878997803</left_val>
            <right_val>1.4687349796295166</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 8 15 6 -1.</_>
                <_>5 11 15 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0315029993653297</threshold>
            <left_val>-1.6463639736175537</left_val>
            <right_val>1.0000629425048828</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 6 5 14 -1.</_>
                <_>9 13 5 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0142609998583794</threshold>
            <left_val>0.4648030102252960</left_val>
            <right_val>-1.5959889888763428</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 5 6 10 -1.</_>
                <_>11 5 2 10 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0144530003890395</threshold>
            <left_val>-0.6551190018653870</left_val>
            <right_val>0.8302180171012878</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 6 3 12 -1.</_>
                <_>6 12 3 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.0509999487549067e-003</threshold>
            <left_val>-1.3982310295104980</left_val>
            <right_val>0.4255059957504273</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 21 18 3 -1.</_>
                <_>9 21 6 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0327229984104633</threshold>
            <left_val>-0.5070260167121887</left_val>
            <right_val>1.0526109933853149</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 13 6 -1.</_>
                <_>5 8 13 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.2960001416504383e-003</threshold>
            <left_val>0.3635689914226532</left_val>
            <right_val>-1.3464889526367187</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>18 1 6 15 -1.</_>
                <_>18 1 3 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0504250004887581</threshold>
            <left_val>-0.3046140074729919</left_val>
            <right_val>1.4504129886627197</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 1 6 15 -1.</_>
                <_>4 1 3 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0468790009617805</threshold>
            <left_val>-0.4028620123863220</left_val>
            <right_val>1.2145609855651855</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 8 24 15 -1.</_>
                <_>8 8 8 15 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0693589970469475</threshold>
            <left_val>1.0539360046386719</left_val>
            <right_val>-0.4571970105171204</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 14 12 -1.</_>
                <_>5 6 7 6 2.</_>
                <_>12 12 7 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0490339994430542</threshold>
            <left_val>-1.6253089904785156</left_val>
            <right_val>0.1537899971008301</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 12 21 12 -1.</_>
                <_>2 16 21 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0848279967904091</threshold>
            <left_val>0.2840299904346466</left_val>
            <right_val>-1.5662059783935547</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 1 4 10 -1.</_>
                <_>10 1 2 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.7229999648407102e-003</threshold>
            <left_val>-1.0147459506988525</left_val>
            <right_val>0.2329480051994324</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 13 20 10 -1.</_>
                <_>2 13 10 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1156219989061356</threshold>
            <left_val>-0.1673289984464645</left_val>
            <right_val>1.2804069519042969</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 1 6 13 -1.</_>
                <_>2 1 2 13 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0512799993157387</threshold>
            <left_val>1.5162390470504761</left_val>
            <right_val>-0.3027110099792481</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>20 2 4 13 -1.</_>
                <_>20 2 2 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0427069999277592</threshold>
            <left_val>1.7631920576095581</left_val>
            <right_val>-0.0518320016562939</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 5 22 19 -1.</_>
                <_>11 5 11 19 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.3717809915542603</threshold>
            <left_val>-0.3138920068740845</left_val>
            <right_val>1.5357979536056519</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>18 4 6 9 -1.</_>
                <_>20 4 2 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0194129999727011</threshold>
            <left_val>-0.1001759991049767</left_val>
            <right_val>0.9365540146827698</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 3 6 11 -1.</_>
                <_>2 3 2 11 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0174390003085136</threshold>
            <left_val>-0.4037989974021912</left_val>
            <right_val>0.9629300236701965</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 1 4 9 -1.</_>
                <_>12 1 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0396389998495579</threshold>
            <left_val>0.1703909933567047</left_val>
            <right_val>-2.9602990150451660</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 6 19 3 -1.</_>
                <_>0 7 19 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.1469995677471161e-003</threshold>
            <left_val>0.8878679871559143</left_val>
            <right_val>-0.4381870031356812</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 1 4 9 -1.</_>
                <_>12 1 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.7219999572262168e-003</threshold>
            <left_val>-0.3721860051155090</left_val>
            <right_val>0.4001890122890472</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 1 4 9 -1.</_>
                <_>10 1 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0302310008555651</threshold>
            <left_val>0.0659240037202835</left_val>
            <right_val>-2.6469180583953857</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 14 14 -1.</_>
                <_>12 5 7 7 2.</_>
                <_>5 12 7 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0787959992885590</threshold>
            <left_val>-1.7491459846496582</left_val>
            <right_val>0.2847529947757721</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 10 18 2 -1.</_>
                <_>1 11 18 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.1110000088810921e-003</threshold>
            <left_val>-0.9390810132026672</left_val>
            <right_val>0.2320519983768463</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>17 13 4 11 -1.</_>
                <_>17 13 2 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0270910002291203</threshold>
            <left_val>-0.0526640005409718</left_val>
            <right_val>1.0756820440292358</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 4 6 9 -1.</_>
                <_>0 7 6 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0449649989604950</threshold>
            <left_val>-1.8294479846954346</left_val>
            <right_val>0.0995619967579842</right_val></_></_></trees>
      <stage_threshold>-4.6551899909973145</stage_threshold>
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
                <_>6 4 12 9 -1.</_>
                <_>6 7 12 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0657010003924370</threshold>
            <left_val>1.1558510065078735</left_val>
            <right_val>-1.0716359615325928</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 5 12 6 -1.</_>
                <_>10 5 4 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0158399995416403</threshold>
            <left_val>-1.5634720325469971</left_val>
            <right_val>0.7687709927558899</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 1 24 5 -1.</_>
                <_>8 1 8 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1457089930772781</threshold>
            <left_val>-0.5745009779930115</left_val>
            <right_val>1.3808720111846924</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 10 18 6 -1.</_>
                <_>4 12 18 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.1389999464154243e-003</threshold>
            <left_val>-1.4570560455322266</left_val>
            <right_val>0.5161030292510986</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 17 12 6 -1.</_>
                <_>2 17 6 3 2.</_>
                <_>8 20 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.7179999314248562e-003</threshold>
            <left_val>-0.8353360295295715</left_val>
            <right_val>0.5852220058441162</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>19 3 4 13 -1.</_>
                <_>19 3 2 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0185180008411407</threshold>
            <left_val>-0.3131209909915924</left_val>
            <right_val>1.1696679592132568</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 3 4 13 -1.</_>
                <_>3 3 2 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0199580006301403</threshold>
            <left_val>-0.4344260096549988</left_val>
            <right_val>0.9544690251350403</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 1 24 23 -1.</_>
                <_>8 1 8 23 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.2775500118732452</threshold>
            <left_val>1.4906179904937744</left_val>
            <right_val>-0.1381590068340302</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 7 8 12 -1.</_>
                <_>1 11 8 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.1859996318817139e-003</threshold>
            <left_val>-0.9636150002479553</left_val>
            <right_val>0.2766549885272980</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 7 3 14 -1.</_>
                <_>14 14 3 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0377379991114140</threshold>
            <left_val>-2.4464108943939209</left_val>
            <right_val>0.2361959964036942</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 12 16 6 -1.</_>
                <_>3 12 8 3 2.</_>
                <_>11 15 8 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0184630006551743</threshold>
            <left_val>0.1753920018672943</left_val>
            <right_val>-1.3423130512237549</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 6 12 6 -1.</_>
                <_>6 8 12 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0111149996519089</threshold>
            <left_val>0.4871079921722412</left_val>
            <right_val>-0.8985189795494080</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 7 6 12 -1.</_>
                <_>8 13 6 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0339279994368553</threshold>
            <left_val>0.1787420064210892</left_val>
            <right_val>-1.6342279911041260</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 15 9 6 -1.</_>
                <_>15 17 9 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0356490015983582</threshold>
            <left_val>-1.9607399702072144</left_val>
            <right_val>0.1810249984264374</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 17 18 3 -1.</_>
                <_>1 18 18 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0114380000159144</threshold>
            <left_val>0.9901069998741150</left_val>
            <right_val>-0.3810319900512695</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 4 16 12 -1.</_>
                <_>4 10 16 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0652360022068024</threshold>
            <left_val>-2.5794160366058350</left_val>
            <right_val>0.2475360035896301</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 1 4 20 -1.</_>
                <_>2 1 2 20 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0422720015048981</threshold>
            <left_val>1.4411840438842773</left_val>
            <right_val>-0.2950829863548279</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 0 18 2 -1.</_>
                <_>3 1 18 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9219999667257071e-003</threshold>
            <left_val>-0.4960860013961792</left_val>
            <right_val>0.6317359805107117</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 5 20 14 -1.</_>
                <_>1 5 10 7 2.</_>
                <_>11 12 10 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1292179971933365</threshold>
            <left_val>-2.3314270973205566</left_val>
            <right_val>0.0544969998300076</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 8 14 12 -1.</_>
                <_>5 12 14 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0229310002177954</threshold>
            <left_val>-0.8444709777832031</left_val>
            <right_val>0.3873809874057770</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 14 7 9 -1.</_>
                <_>3 17 7 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0341200008988380</threshold>
            <left_val>-1.4431500434875488</left_val>
            <right_val>0.0984229966998100</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 15 9 6 -1.</_>
                <_>14 17 9 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0262230001389980</threshold>
            <left_val>0.1822309941053391</left_val>
            <right_val>-1.2586519718170166</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 15 9 6 -1.</_>
                <_>1 17 9 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0222369991242886</threshold>
            <left_val>0.0698079988360405</left_val>
            <right_val>-2.3820950984954834</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 6 8 10 -1.</_>
                <_>15 6 4 5 2.</_>
                <_>11 11 4 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.8240001089870930e-003</threshold>
            <left_val>0.3933250010013580</left_val>
            <right_val>-0.2754279971122742</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 14 14 -1.</_>
                <_>5 5 7 7 2.</_>
                <_>12 12 7 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0436530001461506</threshold>
            <left_val>0.1483269929885864</left_val>
            <right_val>-1.1368780136108398</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 0 12 5 -1.</_>
                <_>10 0 4 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0572669990360737</threshold>
            <left_val>0.2462809979915619</left_val>
            <right_val>-1.2687400579452515</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 6 9 -1.</_>
                <_>9 3 6 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.3409998975694180e-003</threshold>
            <left_val>-0.7544890046119690</left_val>
            <right_val>0.2716380059719086</right_val></_></_>
        <_>
          <!-- tree 27 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 6 6 9 -1.</_>
                <_>11 6 2 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0129960002377629</threshold>
            <left_val>-0.3639490008354187</left_val>
            <right_val>0.7095919847488403</right_val></_></_>
        <_>
          <!-- tree 28 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 0 6 9 -1.</_>
                <_>9 0 2 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0265170000493526</threshold>
            <left_val>-2.3221859931945801</left_val>
            <right_val>0.0357440002262592</right_val></_></_>
        <_>
          <!-- tree 29 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 6 6 9 -1.</_>
                <_>12 6 2 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.8400002308189869e-003</threshold>
            <left_val>0.4219430088996887</left_val>
            <right_val>-0.0481849983334541</right_val></_></_>
        <_>
          <!-- tree 30 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 6 6 9 -1.</_>
                <_>10 6 2 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0165689997375011</threshold>
            <left_val>1.1099940538406372</left_val>
            <right_val>-0.3484970033168793</right_val></_></_>
        <_>
          <!-- tree 31 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 8 18 4 -1.</_>
                <_>9 8 6 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0681570023298264</threshold>
            <left_val>-3.3269989490509033</left_val>
            <right_val>0.2129900008440018</right_val></_></_></trees>
      <stage_threshold>-4.4531588554382324</stage_threshold>
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
                <_>6 0 12 9 -1.</_>
                <_>6 3 12 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0399740003049374</threshold>
            <left_val>-1.2173449993133545</left_val>
            <right_val>1.0826710462570190</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 0 24 6 -1.</_>
                <_>8 0 8 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1881950050592423</threshold>
            <left_val>-0.4828940033912659</left_val>
            <right_val>1.4045250415802002</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 7 16 12 -1.</_>
                <_>4 11 16 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0780270025134087</threshold>
            <left_val>-1.0782150030136108</left_val>
            <right_val>0.7404029965400696</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 6 6 6 -1.</_>
                <_>11 6 3 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.1899999663000926e-004</threshold>
            <left_val>-1.2019979953765869</left_val>
            <right_val>0.3774920105934143</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 20 24 3 -1.</_>
                <_>8 20 8 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0850569978356361</threshold>
            <left_val>-0.4393909871578217</left_val>
            <right_val>1.2647340297698975</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 6 4 9 -1.</_>
                <_>11 6 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.9720003306865692e-003</threshold>
            <left_val>-0.1844049990177155</left_val>
            <right_val>0.4572640061378479</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 13 15 4 -1.</_>
                <_>9 13 5 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.8120000436902046e-003</threshold>
            <left_val>0.3039669990539551</left_val>
            <right_val>-0.9599109888076782</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 6 4 9 -1.</_>
                <_>11 6 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0235079992562532</threshold>
            <left_val>1.2487529516220093</left_val>
            <right_val>0.0462279990315437</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 6 4 9 -1.</_>
                <_>11 6 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.0039997808635235e-003</threshold>
            <left_val>-0.5944210290908814</left_val>
            <right_val>0.5396329760551453</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 12 6 12 -1.</_>
                <_>9 18 6 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0338519997894764</threshold>
            <left_val>0.2849609851837158</left_val>
            <right_val>-1.4895249605178833</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 22 18 2 -1.</_>
                <_>1 23 18 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.2530000898987055e-003</threshold>
            <left_val>0.4812079966068268</left_val>
            <right_val>-0.5271239876747131</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 7 4 10 -1.</_>
                <_>10 12 4 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0290970001369715</threshold>
            <left_val>0.2674390077590942</left_val>
            <right_val>-1.6007850170135498</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 7 8 10 -1.</_>
                <_>6 12 8 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.4790000692009926e-003</threshold>
            <left_val>-1.3107639551162720</left_val>
            <right_val>0.1524309962987900</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 6 10 6 -1.</_>
                <_>7 8 10 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0107950000092387</threshold>
            <left_val>0.4561359882354736</left_val>
            <right_val>-0.7205089926719666</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 14 10 4 -1.</_>
                <_>0 16 10 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0246200002729893</threshold>
            <left_val>-1.7320619821548462</left_val>
            <right_val>0.0683630034327507</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 18 18 2 -1.</_>
                <_>6 19 18 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.7380000576376915e-003</threshold>
            <left_val>-0.1930329948663712</left_val>
            <right_val>0.6824349761009216</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 1 22 3 -1.</_>
                <_>1 2 22 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0122640002518892</threshold>
            <left_val>-1.6095290184020996</left_val>
            <right_val>0.0752680003643036</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 16 18 3 -1.</_>
                <_>6 17 18 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.8670000396668911e-003</threshold>
            <left_val>0.7428650259971619</left_val>
            <right_val>-0.2151020020246506</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 4 6 15 -1.</_>
                <_>5 4 3 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0767259970307350</threshold>
            <left_val>-0.2683509886264801</left_val>
            <right_val>1.3094140291213989</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>20 4 4 10 -1.</_>
                <_>20 4 2 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0285780001431704</threshold>
            <left_val>-0.0587930008769035</left_val>
            <right_val>1.2196329832077026</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 4 4 10 -1.</_>
                <_>2 4 2 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0196940004825592</threshold>
            <left_val>-0.3514289855957031</left_val>
            <right_val>0.8492699861526489</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 16 20 6 -1.</_>
                <_>12 16 10 3 2.</_>
                <_>2 19 10 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0290939994156361</threshold>
            <left_val>-1.0507299900054932</left_val>
            <right_val>0.2980630099773407</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 12 8 9 -1.</_>
                <_>4 12 4 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0291440002620220</threshold>
            <left_val>0.8254780173301697</left_val>
            <right_val>-0.3268719911575317</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 0 6 9 -1.</_>
                <_>14 0 2 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0197410006076097</threshold>
            <left_val>0.2045260071754456</left_val>
            <right_val>-0.8376020193099976</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 10 6 6 -1.</_>
                <_>8 10 3 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.3299999088048935e-003</threshold>
            <left_val>0.2057790011167526</left_val>
            <right_val>-0.6682980060577393</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 8 12 6 -1.</_>
                <_>17 8 6 3 2.</_>
                <_>11 11 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0355009995400906</threshold>
            <left_val>-1.2969900369644165</left_val>
            <right_val>0.1389749944210053</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 8 12 6 -1.</_>
                <_>0 8 6 3 2.</_>
                <_>6 11 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0161729995161295</threshold>
            <left_val>-1.3110569715499878</left_val>
            <right_val>0.07575199753046
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

