# Documentation for `data/haarcascades_cuda/haarcascade_profileface.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_profileface.xml`
- **File Name**: `haarcascade_profileface.xml`
- **File Size**: 1,125,633 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_profileface.xml](../../data/haarcascades_cuda/haarcascade_profileface.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    20x20 profile face detector.
    Contributed by David Bradley from Princeton University.

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
<haarcascade_profileface type_id="opencv-haar-classifier">
  <size>20 20</size>
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
                <_>8 7 2 6 -1.</_>
                <_>8 10 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.1384399840608239e-003</threshold>
            <left_val>-0.8377197980880737</left_val>
            <right_val>0.7341383099555969</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 3 10 7 -1.</_>
                <_>13 3 5 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0113423503935337</threshold>
            <left_val>0.6270201802253723</left_val>
            <right_val>-0.7239630222320557</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 11 3 6 -1.</_>
                <_>10 14 3 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.1023089755326509e-003</threshold>
            <left_val>0.3760018944740295</left_val>
            <right_val>-0.6608840823173523</right_val></_></_></trees>
      <stage_threshold>-1.1856809854507446</stage_threshold>
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
                <_>10 4 8 8 -1.</_>
                <_>14 4 4 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0195538699626923</threshold>
            <left_val>0.4924583137035370</left_val>
            <right_val>-0.6339616775512695</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 7 5 4 -1.</_>
                <_>5 9 5 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.2794529795646667e-003</threshold>
            <left_val>-0.6460496783256531</left_val>
            <right_val>0.3581846058368683</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 4 6 6 -1.</_>
                <_>8 4 3 3 2.</_>
                <_>11 7 3 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.4270440917462111e-003</threshold>
            <left_val>-0.4725323021411896</left_val>
            <right_val>0.2849431037902832</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 14 5 2 -1.</_>
                <_>10 15 5 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9644061103463173e-003</threshold>
            <left_val>0.1699953973293304</left_val>
            <right_val>-0.7786815762519836</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 11 8 4 -1.</_>
                <_>7 13 8 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.2895270958542824e-003</threshold>
            <left_val>0.1555171012878418</left_val>
            <right_val>-0.6672509908676148</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 14 3 3 -1.</_>
                <_>11 15 3 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.0143910553306341e-003</threshold>
            <left_val>-0.6872130036354065</left_val>
            <right_val>0.1460456997156143</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 5 3 11 -1.</_>
                <_>4 5 1 11 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0173990093171597</threshold>
            <left_val>0.7252438068389893</left_val>
            <right_val>-0.1657290011644363</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 7 9 6 -1.</_>
                <_>8 10 9 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.0722442837432027e-004</threshold>
            <left_val>-0.4638808071613312</left_val>
            <right_val>0.2360499948263168</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 12 1 2 -1.</_>
                <_>13 13 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5043979510664940e-003</threshold>
            <left_val>-0.7595962882041931</left_val>
            <right_val>0.1143691986799240</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 3 6 17 -1.</_>
                <_>4 3 3 17 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1080468967556953</threshold>
            <left_val>-0.1286551952362061</left_val>
            <right_val>0.7909234166145325</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 12 1 3 -1.</_>
                <_>11 13 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.1923050042241812e-003</threshold>
            <left_val>-0.6240354776382446</left_val>
            <right_val>0.1484749019145966</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 9 6 9 -1.</_>
                <_>4 9 3 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0205713901668787</threshold>
            <left_val>0.4080848991870880</left_val>
            <right_val>-0.2128700017929077</right_val></_></_></trees>
      <stage_threshold>-1.4913179874420166</stage_threshold>
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
                <_>10 5 8 6 -1.</_>
                <_>14 5 4 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0368992090225220</threshold>
            <left_val>0.5330861806869507</left_val>
            <right_val>-0.4087265133857727</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 8 9 6 -1.</_>
                <_>7 10 9 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.4960909504443407e-003</threshold>
            <left_val>-0.6948931217193604</left_val>
            <right_val>0.2712517976760864</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 8 6 6 -1.</_>
                <_>5 8 3 3 2.</_>
                <_>8 11 3 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.4068039783742279e-004</threshold>
            <left_val>-0.5620825290679932</left_val>
            <right_val>0.2193035036325455</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 0 4 18 -1.</_>
                <_>4 0 2 18 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0580218285322189</threshold>
            <left_val>0.6906061768531799</left_val>
            <right_val>-0.1508214026689529</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 12 3 4 -1.</_>
                <_>10 14 3 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.1526979506015778e-003</threshold>
            <left_val>0.1392538994550705</left_val>
            <right_val>-0.6631165742874146</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 0 3 9 -1.</_>
                <_>7 3 3 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.4388440698385239e-003</threshold>
            <left_val>-0.3333317041397095</left_val>
            <right_val>0.3169938027858734</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 13 1 3 -1.</_>
                <_>11 14 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.4158539706841111e-003</threshold>
            <left_val>-0.6800730228424072</left_val>
            <right_val>0.1324332058429718</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 8 5 2 -1.</_>
                <_>4 9 5 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.8562711607664824e-004</threshold>
            <left_val>-0.3867216110229492</left_val>
            <right_val>0.1973295956850052</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 13 2 3 -1.</_>
                <_>11 14 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.5714060757309198e-003</threshold>
            <left_val>0.1203565970063210</left_val>
            <right_val>-0.7317706942558289</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 12 1 3 -1.</_>
                <_>12 13 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.8255549948662519e-003</threshold>
            <left_val>0.0779798403382301</left_val>
            <right_val>-0.7719609141349793</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 12 2 8 -1.</_>
                <_>9 16 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.1993020307272673e-003</threshold>
            <left_val>0.1682122945785523</left_val>
            <right_val>-0.4147912859916687</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 3 4 13 -1.</_>
                <_>8 3 2 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0231790803372860</threshold>
            <left_val>0.0753373205661774</left_val>
            <right_val>-0.7104706764221191</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 6 4 12 -1.</_>
                <_>4 6 2 12 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0465394183993340</threshold>
            <left_val>-0.1046483963727951</left_val>
            <right_val>0.6627069711685181</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 13 3 2 -1.</_>
                <_>12 13 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.7157640540972352e-003</threshold>
            <left_val>-0.4961821138858795</left_val>
            <right_val>0.1627524048089981</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 5 3 11 -1.</_>
                <_>4 5 1 11 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0127788297832012</threshold>
            <left_val>0.4625453948974609</left_val>
            <right_val>-0.1602790057659149</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 6 13 12 -1.</_>
                <_>3 12 13 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1521482020616531</threshold>
            <left_val>-0.7059270143508911</left_val>
            <right_val>0.1002250984311104</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 7 6 6 -1.</_>
                <_>7 7 3 3 2.</_>
                <_>10 10 3 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.1789899803698063e-003</threshold>
            <left_val>0.1234574988484383</left_val>
            <right_val>-0.3909341990947723</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 7 3 2 -1.</_>
                <_>5 7 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.2882770281285048e-003</threshold>
            <left_val>0.3708150088787079</left_val>
            <right_val>-0.1621042042970657</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 4 14 3 -1.</_>
                <_>12 4 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.9806189704686403e-003</threshold>
            <left_val>0.1808705925941467</left_val>
            <right_val>-0.3323985934257507</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 12 3 2 -1.</_>
                <_>11 12 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5072739915922284e-003</threshold>
            <left_val>-0.4947231113910675</left_val>
            <right_val>0.0982888564467430</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 10 2 3 -1.</_>
                <_>5 11 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9225040450692177e-003</threshold>
            <left_val>-0.1779111027717590</left_val>
            <right_val>0.3077332973480225</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 14 1 3 -1.</_>
                <_>12 15 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9025449873879552e-003</threshold>
            <left_val>0.0847949981689453</left_val>
            <right_val>-0.5902097225189209</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 6 3 3 -1.</_>
                <_>4 6 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.5421559587121010e-003</threshold>
            <left_val>0.3117577135562897</left_val>
            <right_val>-0.1439293026924133</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 4 3 2 -1.</_>
                <_>9 4 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.9751660767942667e-003</threshold>
            <left_val>-0.6364914178848267</left_val>
            <right_val>0.0826398879289627</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 3 3 13 -1.</_>
                <_>4 3 1 13 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0100032901391387</threshold>
            <left_val>-0.1169926002621651</left_val>
            <right_val>0.4238753020763397</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 4 2 3 -1.</_>
                <_>15 5 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.9193530315533280e-003</threshold>
            <left_val>-0.4711583852767944</left_val>
            <right_val>0.1103824004530907</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 8 4 4 -1.</_>
                <_>12 10 4 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0250706207007170</threshold>
            <left_val>0.0487759299576283</left_val>
            <right_val>-0.8035132884979248</right_val></_></_></trees>
      <stage_threshold>-1.9596290588378906</stage_threshold>
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
                <_>8 7 8 9 -1.</_>
                <_>8 10 8 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0142147997394204</threshold>
            <left_val>-0.6357787847518921</left_val>
            <right_val>0.3346172869205475</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 0 12 6 -1.</_>
                <_>8 0 6 3 2.</_>
                <_>14 3 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0125259095802903</threshold>
            <left_val>0.3276613056659699</left_val>
            <right_val>-0.4133152961730957</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 9 3 6 -1.</_>
                <_>5 12 3 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.2514370357384905e-005</threshold>
            <left_val>0.2310263067483902</left_val>
            <right_val>-0.5428205132484436</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 12 2 4 -1.</_>
                <_>12 12 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.8600060138851404e-003</threshold>
            <left_val>0.1793334931135178</left_val>
            <right_val>-0.6913194060325623</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 11 3 8 -1.</_>
                <_>11 11 1 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.8344792127609253e-003</threshold>
            <left_val>0.0910713002085686</left_val>
            <right_val>-0.7812684774398804</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 5 6 -1.</_>
                <_>5 7 5 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.2322301305830479e-003</threshold>
            <left_val>0.2065840959548950</left_val>
            <right_val>-0.4290603101253510</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 13 2 6 -1.</_>
                <_>10 16 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.5860600918531418e-004</threshold>
            <left_val>0.2073071002960205</left_val>
            <right_val>-0.4207031130790710</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 15 3 4 -1.</_>
                <_>11 15 1 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.5626380704343319e-003</threshold>
            <left_val>-0.6322708725929260</left_val>
            <right_val>0.1311862021684647</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 3 3 3 -1.</_>
                <_>8 3 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.9960161559283733e-003</threshold>
            <left_val>-0.7511237859725952</left_val>
            <right_val>0.0782033279538155</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 8 6 2 -1.</_>
                <_>8 8 3 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.3098740540444851e-003</threshold>
            <left_val>0.0934286415576935</left_val>
            <right_val>-0.6631010770797730</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 7 4 2 -1.</_>
                <_>10 7 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.2772040392737836e-004</threshold>
            <left_val>-0.3414882123470306</left_val>
            <right_val>0.2000820040702820</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 2 3 -1.</_>
                <_>6 6 1 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.3124160300940275e-004</threshold>
            <left_val>-0.2544816136360169</left_val>
            <right_val>0.2585771083831787</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 0 3 8 -1.</_>
                <_>9 0 1 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.5492179021239281e-003</threshold>
            <left_val>-0.6613898873329163</left_val>
            <right_val>0.0830044224858284</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 10 3 8 -1.</_>
                <_>5 14 3 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0380399487912655</threshold>
            <left_val>-0.8216357231140137</left_val>
            <right_val>0.0592315904796124</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 3 3 2 -1.</_>
                <_>13 3 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.8484580107033253e-003</threshold>
            <left_val>0.0897299572825432</left_val>
            <right_val>-0.5833374261856079</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 2 3 4 -1.</_>
                <_>9 2 1 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.8181698657572269e-003</threshold>
            <left_val>0.0939605608582497</left_val>
            <right_val>-0.5761976838111877</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 10 1 8 -1.</_>
                <_>14 14 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0111904898658395</threshold>
            <left_val>-0.6254429817199707</left_val>
            <right_val>0.0736088976264000</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 12 2 3 -1.</_>
                <_>6 13 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.4537129364907742e-003</threshold>
            <left_val>0.5512338876724243</left_val>
            <right_val>-0.1002079024910927</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 12 2 3 -1.</_>
                <_>6 13 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.3225629013031721e-003</threshold>
            <left_val>-0.1079789027571678</left_val>
            <right_val>0.5366494059562683</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 12 3 2 -1.</_>
                <_>10 12 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.6705761924386024e-003</threshold>
            <left_val>0.0883211269974709</left_val>
            <right_val>-0.6768360137939453</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 2 1 12 -1.</_>
                <_>12 6 1 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0116133103147149</threshold>
            <left_val>-0.5071188211441040</left_val>
            <right_val>0.0765566304326057</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 8 14 6 -1.</_>
                <_>2 8 7 3 2.</_>
                <_>9 11 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0375156104564667</threshold>
            <left_val>-0.7293627262115479</left_val>
            <right_val>0.0594486109912395</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 3 3 17 -1.</_>
                <_>12 3 1 17 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0230860300362110</threshold>
            <left_val>0.0507189594209194</left_val>
            <right_val>-0.7845978140830994</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 12 1 2 -1.</_>
                <_>12 13 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.1651988946541678e-006</threshold>
            <left_val>0.1668622046709061</left_val>
            <right_val>-0.2571322023868561</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 1 2 1 -1.</_>
                <_>14 1 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.1611627936363220e-004</threshold>
            <left_val>0.1063603013753891</left_val>
            <right_val>-0.4279364049434662</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 10 2 3 -1.</_>
                <_>5 11 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.1476460173726082e-003</threshold>
            <left_val>-0.1206965968012810</left_val>
            <right_val>0.4199318885803223</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 10 1 3 -1.</_>
                <_>5 11 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.5815099943429232e-003</threshold>
            <left_val>0.4871808886528015</left_val>
            <right_val>-0.1004581004381180</right_val></_></_>
        <_>
          <!-- tree 27 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 12 2 3 -1.</_>
                <_>12 13 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.7147070029750466e-003</threshold>
            <left_val>-0.4609631001949310</left_val>
            <right_val>0.1037511005997658</right_val></_></_></trees>
      <stage_threshold>-1.9849590063095093</stage_threshold>
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
                <_>8 2 10 10 -1.</_>
                <_>13 2 5 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0612027198076248</threshold>
            <left_val>0.3907910883426666</left_val>
            <right_val>-0.3940125107765198</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 13 3 1 -1.</_>
                <_>12 13 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.4643670292571187e-003</threshold>
            <left_val>-0.7369784116744995</left_val>
            <right_val>0.1566022038459778</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 10 1 4 -1.</_>
                <_>12 12 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.2080420795828104e-004</threshold>
            <left_val>0.2167553007602692</left_val>
            <right_val>-0.5801265835762024</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 7 2 6 -1.</_>
                <_>8 10 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.4895692048594356e-004</threshold>
            <left_val>-0.7230809926986694</left_val>
            <right_val>0.1278524994850159</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 11 1 3 -1.</_>
                <_>12 12 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.7158190021291375e-003</threshold>
            <left_val>-0.7710043191909790</left_val>
            <right_val>0.1021030992269516</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 12 3 3 -1.</_>
                <_>10 12 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.2490581031888723e-003</threshold>
            <left_val>-0.6062312722206116</left_val>
            <right_val>0.1242726966738701</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 0 8 6 -1.</_>
                <_>6 3 8 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0538419783115387</threshold>
            <left_val>-0.1716974973678589</left_val>
            <right_val>0.5335056781768799</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 0 8 19 -1.</_>
                <_>4 0 4 19 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1328897029161453</threshold>
            <left_val>0.5592436790466309</left_val>
            <right_val>-0.1895489990711212</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 4 9 -1.</_>
                <_>5 9 4 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.0965389972552657e-004</threshold>
            <left_val>-0.4716643095016480</left_val>
            <right_val>0.1692426055669785</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 14 1 2 -1.</_>
                <_>13 15 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.0799147468060255e-004</threshold>
            <left_val>0.1134722009301186</left_val>
            <right_val>-0.5984687805175781</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 3 8 15 -1.</_>
                <_>5 3 4 15 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1607262939214706</threshold>
            <left_val>-0.1029551997780800</left_val>
            <right_val>0.6648719906806946</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 14 2 3 -1.</_>
                <_>13 15 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.7097239615395665e-003</threshold>
            <left_val>-0.4727627933025360</left_val>
            <right_val>0.1339205056428909</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 7 3 2 -1.</_>
                <_>6 7 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.1734620202332735e-003</threshold>
            <left_val>-0.2279558926820755</left_val>
            <right_val>0.2613565027713776</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 5 3 1 -1.</_>
                <_>9 5 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5138329472392797e-003</threshold>
            <left_val>-0.5539500117301941</left_val>
            <right_val>0.1102833971381187</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 5 3 1 -1.</_>
                <_>10 5 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1774161141365767e-003</threshold>
            <left_val>-0.6222890019416809</left_val>
            <right_val>0.0784866735339165</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 11 1 3 -1.</_>
                <_>6 12 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.7727920096367598e-003</threshold>
            <left_val>0.4614112079143524</left_val>
            <right_val>-0.1349655985832214</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>18 4 1 2 -1.</_>
                <_>18 5 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.3199027469381690e-004</threshold>
            <left_val>0.1016277000308037</left_val>
            <right_val>-0.5163183808326721</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 11 2 3 -1.</_>
                <_>6 12 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.9746659565716982e-003</threshold>
            <left_val>-0.1299920976161957</left_val>
            <right_val>0.4211730062961578</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 10 3 4 -1.</_>
                <_>11 10 1 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.0399480387568474e-003</threshold>
            <left_val>-0.6370617151260376</left_val>
            <right_val>0.0776241272687912</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 5 2 14 -1.</_>
                <_>6 12 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0234148502349854</threshold>
            <left_val>0.0721827968955040</left_val>
            <right_val>-0.5983113050460815</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 8 3 4 -1.</_>
                <_>14 10 3 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.0927390540018678e-003</threshold>
            <left_val>-0.4166488051414490</left_val>
            <right_val>0.1182999014854431</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 5 3 6 -1.</_>
                <_>4 7 3 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.6441360348835588e-003</threshold>
            <left_val>0.1858306974172592</left_val>
            <right_val>-0.2755101919174194</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 10 2 8 -1.</_>
                <_>5 14 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0257362797856331</threshold>
            <left_val>-0.7514647841453552</left_val>
            <right_val>0.0639077499508858</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 1 3 2 -1.</_>
                <_>10 1 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.8924590442329645e-003</threshold>
            <left_val>-0.5678088068962097</left_val>
            <right_val>0.0732977390289307</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 1 3 3 -1.</_>
                <_>11 1 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.2889231592416763e-003</threshold>
            <left_val>-0.6373888850212097</left_val>
            <right_val>0.0686869472265244</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 12 8 8 -1.</_>
                <_>9 12 4 4 2.</_>
                <_>13 16 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.2964269630610943e-003</threshold>
            <left_val>-0.2506295144557953</left_val>
            <right_val>0.1598978042602539</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 13 6 4 -1.</_>
                <_>10 13 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0249144397675991</threshold>
            <left_val>0.0552609786391258</left_val>
            <right_val>-0.7620877027511597</right_val></_></_>
        <_>
          <!-- tree 27 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 6 3 12 -1.</_>
                <_>4 6 1 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0150885004550219</threshold>
            <left_val>0.3703337907791138</left_val>
            <right_val>-0.1200395971536636</right_val></_></_></trees>
      <stage_threshold>-1.8260079622268677</stage_threshold>
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
                <_>9 3 8 5 -1.</_>
                <_>13 3 4 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0118571799248457</threshold>
            <left_val>0.2942155897617340</left_val>
            <right_val>-0.5170331001281738</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 7 3 6 -1.</_>
                <_>7 10 3 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.0991980563849211e-003</threshold>
            <left_val>-0.6147174835205078</left_val>
            <right_val>0.2064850032329559</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 10 10 4 -1.</_>
                <_>5 12 10 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5772449842188507e-004</threshold>
            <left_val>0.2287074029445648</left_val>
            <right_val>-0.5525804758071899</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 12 1 6 -1.</_>
                <_>11 15 1 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.0669099467340857e-004</threshold>
            <left_val>0.1207000985741615</left_val>
            <right_val>-0.5492612719535828</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 8 6 2 -1.</_>
                <_>8 8 3 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.2675560321658850e-003</threshold>
            <left_val>0.1535481065511704</left_val>
            <right_val>-0.4607430100440979</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 0 8 4 -1.</_>
                <_>2 0 4 2 2.</_>
                <_>6 2 4 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0144694996997714</threshold>
            <left_val>-0.1897630989551544</left_val>
            <right_val>0.4207141101360321</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 7 3 5 -1.</_>
                <_>12 7 1 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.2127560330554843e-003</threshold>
            <left_val>-0.4513986110687256</left_val>
            <right_val>0.0994258671998978</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 13 2 3 -1.</_>
                <_>12 14 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.1505509503185749e-003</threshold>
            <left_val>0.1020087972283363</left_val>
            <right_val>-0.6206424236297607</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 12 1 2 -1.</_>
                <_>12 13 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.6638869419693947e-003</threshold>
            <left_val>-0.7036749124526978</left_val>
            <right_val>0.0772146806120873</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 11 6 3 -1.</_>
                <_>8 11 3 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.0530210565775633e-003</threshold>
            <left_val>-0.3245396018028259</left_val>
            <right_val>0.1761610954999924</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 6 3 9 -1.</_>
                <_>3 6 1 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0118364095687866</threshold>
            <left_val>-0.1350782066583633</left_val>
            <right_val>0.4264113008975983</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 12 1 3 -1.</_>
                <_>12 13 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.6512871095910668e-004</threshold>
            <left_val>0.0945027694106102</left_val>
            <right_val>-0.4854493141174316</right_val></_></_>
        <_>
          <!-- tree 12 -->
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

