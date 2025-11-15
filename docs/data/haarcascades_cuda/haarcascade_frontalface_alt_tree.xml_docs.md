# Documentation for `data/haarcascades_cuda/haarcascade_frontalface_alt_tree.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_frontalface_alt_tree.xml`
- **File Name**: `haarcascade_frontalface_alt_tree.xml`
- **File Size**: 3,644,763 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_frontalface_alt_tree.xml](../../data/haarcascades_cuda/haarcascade_frontalface_alt_tree.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    Stump-based 20x20 gentle adaboost frontal face detector.
    This detector uses tree of stage classifiers instead of a cascade
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
<haarcascade_frontalface_tree_alt type_id="opencv-haar-classifier">
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
                <_>2 7 14 4 -1.</_>
                <_>2 9 14 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.7895569112151861e-003</threshold>
            <left_val>-0.9294580221176148</left_val>
            <right_val>0.6411985158920288</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 2 18 4 -1.</_>
                <_>7 2 6 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0120981102809310</threshold>
            <left_val>-0.7181009054183960</left_val>
            <right_val>0.4714100956916809</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 9 5 -1.</_>
                <_>8 5 3 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.2138449819758534e-003</threshold>
            <left_val>-0.7283161282539368</left_val>
            <right_val>0.3033069074153900</right_val></_></_></trees>
      <stage_threshold>-1.3442519903182983</stage_threshold>
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
                <_>3 6 14 9 -1.</_>
                <_>3 9 14 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.7510552257299423e-003</threshold>
            <left_val>-0.8594707250595093</left_val>
            <right_val>0.3688138127326965</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 1 18 5 -1.</_>
                <_>7 1 6 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0219867005944252</threshold>
            <left_val>-0.6018015146255493</left_val>
            <right_val>0.3289783000946045</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 6 12 8 -1.</_>
                <_>4 10 12 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.4913398819044232e-004</threshold>
            <left_val>-0.7943195104598999</left_val>
            <right_val>0.2549329996109009</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 5 6 10 -1.</_>
                <_>12 5 3 5 2.</_>
                <_>9 10 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.0192029876634479e-003</threshold>
            <left_val>0.2272932976484299</left_val>
            <right_val>-0.6362798213958740</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 0 11 9 -1.</_>
                <_>4 3 11 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.3674780493602157e-003</threshold>
            <left_val>-0.6001418232917786</left_val>
            <right_val>0.2411836981773377</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 5 4 8 -1.</_>
                <_>12 9 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.0245250305160880e-003</threshold>
            <left_val>-0.5854247212409973</left_val>
            <right_val>0.1255010962486267</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 5 10 10 -1.</_>
                <_>4 5 5 5 2.</_>
                <_>9 10 5 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0184658598154783</threshold>
            <left_val>0.1956356018781662</left_val>
            <right_val>-0.6763023138046265</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 5 6 7 -1.</_>
                <_>9 5 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.0901508182287216e-003</threshold>
            <left_val>-0.4491649866104126</left_val>
            <right_val>0.2667768895626068</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 8 5 12 -1.</_>
                <_>3 14 5 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0113580999895930</threshold>
            <left_val>0.1878322958946228</left_val>
            <right_val>-0.6137936115264893</right_val></_></_></trees>
      <stage_threshold>-1.6378560066223145</stage_threshold>
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
                <_>5 3 9 9 -1.</_>
                <_>5 6 9 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0115889497101307</threshold>
            <left_val>0.3456704020500183</left_val>
            <right_val>-0.7647898197174072</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 5 4 12 -1.</_>
                <_>8 11 4 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.1809530705213547e-003</threshold>
            <left_val>0.2410492002964020</left_val>
            <right_val>-0.6962355971336365</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 6 5 6 -1.</_>
                <_>3 9 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.1468549966812134e-003</threshold>
            <left_val>-0.8055366277694702</left_val>
            <right_val>0.1983861029148102</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 5 12 5 -1.</_>
                <_>8 5 4 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.6556499544531107e-003</threshold>
            <left_val>-0.7183313965797424</left_val>
            <right_val>0.1230567991733551</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 2 8 8 -1.</_>
                <_>1 2 4 4 2.</_>
                <_>5 6 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.9701640121638775e-003</threshold>
            <left_val>0.2277768999338150</left_val>
            <right_val>-0.4752016961574554</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 12 10 8 -1.</_>
                <_>13 12 5 4 2.</_>
                <_>8 16 5 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.3645539078861475e-003</threshold>
            <left_val>-0.4609504938125610</left_val>
            <right_val>0.2039465010166168</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 9 3 10 -1.</_>
                <_>4 14 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.4126059189438820e-005</threshold>
            <left_val>0.1821323931217194</left_val>
            <right_val>-0.4782927036285400</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 4 20 10 -1.</_>
                <_>0 9 20 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0175711102783680</threshold>
            <left_val>-0.7173755168914795</left_val>
            <right_val>0.1131113022565842</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 0 13 9 -1.</_>
                <_>3 3 13 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.3840472139418125e-003</threshold>
            <left_val>-0.4020568132400513</left_val>
            <right_val>0.2073028981685638</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 1 4 11 -1.</_>
                <_>10 1 2 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0147233996540308</threshold>
            <left_val>-0.6755877137184143</left_val>
            <right_val>0.0689730867743492</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 1 4 11 -1.</_>
                <_>8 1 2 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.2889222279191017e-003</threshold>
            <left_val>-0.6210517287254334</left_val>
            <right_val>0.1334936022758484</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 6 12 8 -1.</_>
                <_>10 6 6 4 2.</_>
                <_>4 10 6 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0277436301112175</threshold>
            <left_val>0.1176085025072098</left_val>
            <right_val>-0.5464112162590027</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 7 12 4 -1.</_>
                <_>4 9 12 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0394275598227978</threshold>
            <left_val>-0.2113427966833115</left_val>
            <right_val>0.3945299983024597</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 9 4 7 -1.</_>
                <_>11 9 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.6949411779642105e-003</threshold>
            <left_val>0.1258095055818558</left_val>
            <right_val>-0.4798910021781921</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 9 4 7 -1.</_>
                <_>7 9 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.8245279099792242e-003</threshold>
            <left_val>0.1965314000844955</left_val>
            <right_val>-0.4025667905807495</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 6 7 -1.</_>
                <_>11 0 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0289151892066002</threshold>
            <left_val>-0.8061652779579163</left_val>
            <right_val>0.0818822607398033</right_val></_></_></trees>
      <stage_threshold>-1.7317579984664917</stage_threshold>
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
                <_>0 7 20 6 -1.</_>
                <_>0 9 20 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.0171944573521614e-003</threshold>
            <left_val>-0.6898155212402344</left_val>
            <right_val>0.2413686066865921</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 3 8 6 -1.</_>
                <_>6 6 8 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.4478728882968426e-003</threshold>
            <left_val>0.2135320007801056</left_val>
            <right_val>-0.6414669156074524</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 2 6 7 -1.</_>
                <_>9 2 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.7917619552463293e-003</threshold>
            <left_val>-0.6144546866416931</left_val>
            <right_val>0.1923692971467972</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 7 5 9 -1.</_>
                <_>11 10 5 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.3905500206165016e-004</threshold>
            <left_val>-0.7536042928695679</left_val>
            <right_val>0.1569689065217972</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 6 8 8 -1.</_>
                <_>4 6 4 4 2.</_>
                <_>8 10 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.6769549478776753e-004</threshold>
            <left_val>0.1738051027059555</left_val>
            <right_val>-0.5840449929237366</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 5 6 8 -1.</_>
                <_>9 9 6 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.2802388779819012e-003</threshold>
            <left_val>-0.6696898937225342</left_val>
            <right_val>0.1128972992300987</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 10 5 6 -1.</_>
                <_>4 13 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.5238768905401230e-003</threshold>
            <left_val>0.1250194013118744</left_val>
            <right_val>-0.7329921722412109</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 0 6 5 -1.</_>
                <_>12 0 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.9299701610580087e-004</threshold>
            <left_val>-0.4496619999408722</left_val>
            <right_val>0.2159093022346497</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 14 10 6 -1.</_>
                <_>2 14 5 3 2.</_>
                <_>7 17 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.4371088733896613e-004</threshold>
            <left_val>-0.3890976905822754</left_val>
            <right_val>0.2118114978075028</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 2 17 2 -1.</_>
                <_>3 3 17 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.7145470958203077e-003</threshold>
            <left_val>-0.4671686887741089</left_val>
            <right_val>0.1503839939832687</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 4 8 -1.</_>
                <_>5 10 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.9272058317437768e-004</threshold>
            <left_val>-0.5859655141830444</left_val>
            <right_val>0.1171438023447990</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 3 6 9 -1.</_>
                <_>14 3 3 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0492618083953857</threshold>
            <left_val>-0.1380015015602112</left_val>
            <right_val>0.4936623871326447</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 0 9 5 -1.</_>
                <_>6 0 3 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0228375196456909</threshold>
            <left_val>-0.6374350786209106</left_val>
            <right_val>0.1232409030199051</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 2 4 9 -1.</_>
                <_>15 2 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.8372112214565277e-003</threshold>
            <left_val>-0.1239162981510162</left_val>
            <right_val>0.1062088981270790</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 2 4 9 -1.</_>
                <_>3 2 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0102562597021461</threshold>
            <left_val>-0.1876704990863800</left_val>
            <right_val>0.2982417047023773</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 8 6 12 -1.</_>
                <_>8 12 6 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0106186801567674</threshold>
            <left_val>0.1061246022582054</left_val>
            <right_val>-0.3324488103389740</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 13 16 4 -1.</_>
                <_>2 13 8 2 2.</_>
                <_>10 15 8 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0241131391376257</threshold>
            <left_val>0.0872006118297577</left_val>
            <right_val>-0.6684662103652954</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 5 8 6 -1.</_>
                <_>6 7 8 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.6754710599780083e-003</threshold>
            <left_val>0.1104328036308289</left_val>
            <right_val>-0.4458195865154266</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 11 8 6 -1.</_>
                <_>0 13 8 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0389962010085583</threshold>
            <left_val>-0.7022811174392700</left_val>
            <right_val>0.0818094909191132</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 18 20 2 -1.</_>
                <_>0 19 20 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.5777100343257189e-003</threshold>
            <left_val>0.1595419943332672</left_val>
            <right_val>-0.3286077082157135</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 12 7 6 -1.</_>
                <_>1 14 7 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.1089410707354546e-003</threshold>
            <left_val>0.1032636985182762</left_val>
            <right_val>-0.4440256059169769</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 1 17 3 -1.</_>
                <_>3 2 17 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0170516092330217</threshold>
            <left_val>-0.5585334897041321</left_val>
            <right_val>0.0627114996314049</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 6 5 6 -1.</_>
                <_>3 9 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.3652660418301821e-003</threshold>
            <left_val>-0.5393446087837219</left_val>
            <right_val>0.0708398967981339</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 5 12 7 -1.</_>
                <_>8 5 4 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0111861499026418</threshold>
            <left_val>-0.4726018011569977</left_val>
            <right_val>0.0810194164514542</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 4 14 4 -1.</_>
                <_>0 4 7 2 2.</_>
                <_>7 6 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0117052700370550</threshold>
            <left_val>0.2475008964538574</left_val>
            <right_val>-0.1777898967266083</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 11 12 9 -1.</_>
                <_>4 14 12 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0977369323372841</threshold>
            <left_val>-0.5617750883102417</left_val>
            <right_val>0.0809218212962151</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 2 14 16 -1.</_>
                <_>3 2 7 8 2.</_>
                <_>10 10 7 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0852280631661415</threshold>
            <left_val>-0.5223324894905090</left_val>
            <right_val>0.0728213936090469</right_val></_></_>
        <_>
          <!-- tree 27 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 0 18 4 -1.</_>
                <_>7 0 6 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0367334596812725</threshold>
            <left_val>0.4362357854843140</left_val>
            <right_val>-0.0993395075201988</right_val></_></_>
        <_>
          <!-- tree 28 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 1 10 16 -1.</_>
                <_>3 1 5 8 2.</_>
                <_>8 9 5 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.6704430822283030e-003</threshold>
            <left_val>0.1483422070741653</left_val>
            <right_val>-0.2711966931819916</right_val></_></_></trees>
      <stage_threshold>-1.9308480024337769</stage_threshold>
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
                <_>1 0 16 2 -1.</_>
                <_>1 1 16 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.1610370129346848e-003</threshold>
            <left_val>-0.5637788772583008</left_val>
            <right_val>0.2356878072023392</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 10 16 4 -1.</_>
                <_>2 12 16 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.1830299627035856e-003</threshold>
            <left_val>0.1572428047657013</left_val>
            <right_val>-0.6772817969322205</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 0 6 8 -1.</_>
                <_>9 0 2 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1273950114846230e-003</threshold>
            <left_val>-0.6615015268325806</left_val>
            <right_val>0.1494313925504684</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 3 10 9 -1.</_>
                <_>5 6 10 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1189346984028816</threshold>
            <left_val>0.5322582125663757</left_val>
            <right_val>-0.2296836972236633</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 0 6 7 -1.</_>
                <_>7 0 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0136248702183366</threshold>
            <left_val>-0.6063550114631653</left_val>
            <right_val>0.1700108945369721</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 6 8 12 -1.</_>
                <_>10 10 8 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.3198682619258761e-004</threshold>
            <left_val>-0.6897224187850952</left_val>
            <right_val>0.1158462986350060</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 8 15 3 -1.</_>
                <_>2 9 15 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.4108428992331028e-003</threshold>
            <left_val>-0.6296700239181519</left_val>
            <right_val>0.1243060007691383</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 6 9 12 -1.</_>
                <_>10 10 9 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0229822397232056</threshold>
            <left_val>-0.5049725174903870</left_val>
            <right_val>0.0166361201554537</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 6 6 8 -1.</_>
                <_>4 10 6 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.3721898905932903e-003</threshold>
            <left_val>-0.6246224045753479</left_val>
            <right_val>0.1379375010728836</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 8 4 12 -1.</_>
                <_>9 12 4 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.7364763021469116e-003</threshold>
            <left_val>0.1399662047624588</left_val>
            <right_val>-0.5482295155525208</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 0 6 18 -1.</_>
                <_>4 0 3 18 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0677370727062225</threshold>
            <left_val>-0.1917248070240021</left_val>
            <right_val>0.5470048785209656</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 2 13 2 -1.</_>
                <_>5 3 13 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.0138149634003639e-003</threshold>
            <left_val>-0.5542911887168884</left_val>
            <right_val>0.1451705992221832</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 6 5 -1.</_>
                <_>8 5 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.2857170077040792e-004</threshold>
            <left_val>-0.5103123784065247</left_val>
            <right_val>0.1102394014596939</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 0 8 12 -1.</_>
                <_>10 0 4 6 2.</_>
                <_>6 6 4 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0396889485418797</threshold>
            <left_val>-0.6183072924613953</left_val>
            <right_val>0.0966760963201523</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 1 6 10 -1.</_>
                <_>2 1 3 5 2.</_>
                <_>5 6 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.6646150033921003e-003</threshold>
            <left_val>0.1644988954067230</left_val>
            <right_val>-0.3718631863594055</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 12 7 6 -1.</_>
                <_>11 14 7 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.3499247878789902e-003</threshold>
            <left_val>0.1114505007863045</left_val>
            <right_val>-0.3744102120399475</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 12 18 4 -1.</_>
                <_>0 12 9 2 2.</_>
                <_>9 14 9 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0229040104895830</threshold>
            <left_val>-0.5809758901596069</left_val>
            <right_val>0.1107726022601128</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 15 6 -1.</_>
                <_>5 7 15 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0107034500688314</threshold>
            <left_val>0.0447332598268986</left_val>
            <right_val>-0.5811663269996643</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 6 5 9 -1.</_>
                <_>2 9 5 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.2331559234298766e-004</threshold>
            <left_val>-0.5442379117012024</left_val>
            <right_val>0.0870892927050591</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 8 10 6 -1.</_>
                <_>14 8 5 3 2.</_>
                <_>9 11 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0155544299632311</threshold>
            <left_val>0.0568843409419060</left_val>
            <right_val>-0.3764517009258270</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 10 10 -1.</_>
                <_>5 6 5 5 2.</_>
                <_>10 11 5 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0205394495278597</threshold>
            <left_val>-0.3871456980705261</left_val>
            <right_val>0.1183383986353874</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 4 12 4 -1.</_>
                <_>7 6 12 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.1234358903020620e-003</threshold>
            <left_val>0.0836354270577431</left_val>
            <right_val>-0.1986238956451416</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 10 16 4 -1.</_>
                <_>1 10 8 2 2.</_>
                <_>9 12 8 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0239328294992447</threshold>
            <left_val>0.0796005427837372</left_val>
            <right_val>-0.6537010073661804</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 17 18 3 -1.</_>
                <_>7 17 6 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0839204564690590</threshold>
            <left_val>-0.1065312996506691</left_val>
            <right_val>0.4877282083034515</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 0 3 17 -1.</_>
                <_>7 0 1 17 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0160031598061323</threshold>
            <left_val>0.0836432129144669</left_val>
            <right_val>-0.5920773148536682</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 4 4 16 -1.</_>
                <_>11 4 2 8 2.</_>
                <_>9 12 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.8071441017091274e-003</threshold>
            <left_val>0.0879975035786629</left_val>
            <right_val>-0.3327913880348206</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 0 4 20 -1.</_>
                <_>2 0 2 20 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0811044275760651</threshold>
            <left_val>0.6377518773078919</left_val>
            <right_val>-0.0676923617720604</right_val></_></_>
        <_>
          <!-- tree 27 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 2 6 13 -1.</_>
                <_>15 2 2 13 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0454030297696590</threshold>
            <left_val>-0.0515103898942471</left_val>
            <right_val>0.3022567033767700</right_val></_></_>
        <_>
          <!-- tree 28 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 1 6 18 -1.</_>
                <_>6 1 3 9 2.</_>
                <_>9 10 3 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0138772297650576</threshold>
            <left_val>0.0999676287174225</left_val>
            <right_val>-0.4652090966701508</right_val></_></_>
        <_>
          <!-- tree 29 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 0 4 13 -1.</_>
                <_>15 0 2 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0345907099545002</threshold>
            <left_val>-0.0976144373416901</left_val>
            <right_val>0.3467875123023987</right_val></_></_>
        <_>
          <!-- tree 30 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 3 14 -1.</_>
                <_>6 6 1 14 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0157045498490334</threshold>
            <left_val>0.0763441175222397</left_val>
            <right_val>-0.5335631966590881</right_val></_></_>
        <_>
          <!-- tree 31 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 2 6 13 -1.</_>
                <_>14 2 3 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1042054966092110</threshold>
            <left_val>0.6189097166061401</left_val>
            <right_val>-0.0442597605288029</right_val></_></_>
        <_>
          <!-- tree 32 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 2 18 3 -1.</_>
                <_>7 2 6 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1344318985939026</threshold>
            <left_val>-0.0598530210554600</left_val>
            <right_val>0.6363571286201477</right_val></_></_>
        <_>
          <!-- tree 33 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 11 8 -1.</_>
                <_>5 9 11 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.5646309368312359e-003</threshold>
            <left_val>-0.5360047221183777</left_val>
            <right_val>0.0731160268187523</right_val></_></_>
        <_>
          <!-- tree 34 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 0 20 3 -1.</_>
                <_>0 1 20 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0186470896005630</threshold>
            <left_val>0.0698561519384384</left_val>
            <right_val>-0.5687832236289978</right_val></_></_>
        <_>
          <!-- tree 35 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 4 7 4 -1.</_>
                <_>11 6 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0151595398783684</threshold>
            <left_val>0.0182063393294811</left_val>
            <right_val>-0.2766315937042236</right_val></_></_></trees>
      <stage_threshold>-2.0711259841918945</stage_threshold>
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
                <_>0 0 10 20 -1.</_>
                <_>5 0 5 20 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1477842926979065</threshold>
            <left_val>-0.8993312120437622</left_val>
            <right_val>0.5703592896461487</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 0 18 20 -1.</_>
                <_>7 0 6 20 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2998467087745667</threshold>
            <left_val>-0.6539415121078491</left_val>
            <right_val>0.3505445122718811</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 3 10 9 -1.</_>
                <_>5 6 10 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0790617167949677</threshold>
            <left_val>0.4408529102802277</left_val>
            <right_val>-0.6508756875991821</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 3 6 11 -1.</_>
                <_>14 3 3 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0584289617836475</threshold>
            <left_val>-0.4266535937786102</left_val>
            <right_val>0.5841056704521179</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 9 4 10 -1.</_>
                <_>3 14 4 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0146642802283168</threshold>
            <left_val>0.3243524134159088</left_val>
            <right_val>-0.5965961813926697</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 1 12 19 -1.</_>
                <_>8 1 6 19 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.3951719999313355</threshold>
            <left_val>-0.0757983475923538</left_val>
            <right_val>0.4865995049476624</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 1 12 19 -1.</_>
                <_>6 1 6 19 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1104058995842934</threshold>
            <left_val>-0.8455610275268555</left_val>
            <right_val>0.2137456983327866</right_val></_></_></trees>
      <stage_threshold>-2.1360809803009033</stage_threshold>
      <parent>4</parent>
      <next>6</next></_>
    <_>
      <!-- stage 6 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 4 4 16 -1.</_>
                <_>8 12 4 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.7777079269289970e-003</threshold>
            <left_val>0.1874440014362335</left_val>
            <right_val>-0.6535406112670898</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 8 4 12 -1.</_>
                <_>9 12 4 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.3003188222646713e-003</threshold>
            <left_val>0.0939518436789513</left_val>
            <right_val>-0.5691788792610169</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 2 8 12 -1.</_>
                <_>6 6 8 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.5426009930670261e-003</threshold>
            <left_val>0.1603170931339264</left_val>
            <right_val>-0.5182223916053772</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 7 6 13 -1.</_>
                <_>9 7 2 13 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.1971885412931442e-003</threshold>
            <left_val>-0.5742046236991882</left_val>
            <right_val>0.1479140073060989</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 6 7 6 -1.</_>
                <_>0 9 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.3701602155342698e-004</threshold>
            <left_val>-0.7044969797134399</left_val>
            <right_val>0.1075214967131615</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 8 19 3 -1.</_>
                <_>1 9 19 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.2125479299575090e-003</threshold>
            <left_val>-0.5087742805480957</left_val>
            <right_val>0.1136718988418579</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 0 3 14 -1.</_>
                <_>6 0 1 14 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0116757303476334</threshold>
            <left_val>0.0842586830258369</left_val>
            <right_val>-0.6738470196723938</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 3 10 6 -1.</_>
                <_>15 3 5 3 2.</_>
                <_>10 6 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.0404369570314884e-003</threshold>
            <left_val>0.1625111997127533</left_val>
            <right_val>-0.4143564999103546</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 1 8 8 -1.</_>
                <_>5 1 4 4 2.</_>
                <_>9 5 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.6540438458323479e-003</threshold>
            <left_val>-0.4283317923545837</left_val>
            <right_val>0.1306070983409882</ri
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

