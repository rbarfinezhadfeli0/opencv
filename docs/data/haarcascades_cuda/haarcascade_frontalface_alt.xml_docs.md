# Documentation for `data/haarcascades_cuda/haarcascade_frontalface_alt.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_frontalface_alt.xml`
- **File Name**: `haarcascade_frontalface_alt.xml`
- **File Size**: 919,871 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_frontalface_alt.xml](../../data/haarcascades_cuda/haarcascade_frontalface_alt.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    Stump-based 20x20 gentle adaboost frontal face detector.
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
<haarcascade_frontalface_alt type_id="opencv-haar-classifier">
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
                <_>3 7 14 4 -1.</_>
                <_>3 9 14 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.0141958743333817e-003</threshold>
            <left_val>0.0337941907346249</left_val>
            <right_val>0.8378106951713562</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 2 18 4 -1.</_>
                <_>7 2 6 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0151513395830989</threshold>
            <left_val>0.1514132022857666</left_val>
            <right_val>0.7488812208175659</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 7 15 9 -1.</_>
                <_>1 10 15 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.2109931819140911e-003</threshold>
            <left_val>0.0900492817163467</left_val>
            <right_val>0.6374819874763489</right_val></_></_></trees>
      <stage_threshold>0.8226894140243530</stage_threshold>
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
                <_>5 6 2 6 -1.</_>
                <_>5 9 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.6227109590545297e-003</threshold>
            <left_val>0.0693085864186287</left_val>
            <right_val>0.7110946178436279</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 5 6 3 -1.</_>
                <_>9 5 2 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.2906649392098188e-003</threshold>
            <left_val>0.1795803010463715</left_val>
            <right_val>0.6668692231178284</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 0 12 9 -1.</_>
                <_>4 3 12 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.0025708042085171e-003</threshold>
            <left_val>0.1693672984838486</left_val>
            <right_val>0.6554006934165955</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 9 10 8 -1.</_>
                <_>6 13 10 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.9659894108772278e-003</threshold>
            <left_val>0.5866332054138184</left_val>
            <right_val>0.0914145186543465</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 6 14 8 -1.</_>
                <_>3 10 14 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.5227010957896709e-003</threshold>
            <left_val>0.1413166970014572</left_val>
            <right_val>0.6031895875930786</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 1 6 10 -1.</_>
                <_>14 1 3 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0366676896810532</threshold>
            <left_val>0.3675672113895416</left_val>
            <right_val>0.7920318245887756</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 8 5 12 -1.</_>
                <_>7 12 5 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.3361474573612213e-003</threshold>
            <left_val>0.6161385774612427</left_val>
            <right_val>0.2088509947061539</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 1 18 3 -1.</_>
                <_>7 1 6 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.6961314082145691e-003</threshold>
            <left_val>0.2836230993270874</left_val>
            <right_val>0.6360273957252502</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 8 17 2 -1.</_>
                <_>1 9 17 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.1488880263641477e-003</threshold>
            <left_val>0.2223580926656723</left_val>
            <right_val>0.5800700783729553</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>16 6 4 2 -1.</_>
                <_>16 7 4 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1484689787030220e-003</threshold>
            <left_val>0.2406464070081711</left_val>
            <right_val>0.5787054896354675</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 17 2 2 -1.</_>
                <_>5 18 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.1219060290604830e-003</threshold>
            <left_val>0.5559654831886292</left_val>
            <right_val>0.1362237036228180</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 2 6 12 -1.</_>
                <_>14 2 3 12 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0939491465687752</threshold>
            <left_val>0.8502737283706665</left_val>
            <right_val>0.4717740118503571</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 0 4 12 -1.</_>
                <_>4 0 2 6 2.</_>
                <_>6 6 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.3777789426967502e-003</threshold>
            <left_val>0.5993673801422119</left_val>
            <right_val>0.2834529876708984</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 11 18 8 -1.</_>
                <_>8 11 6 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0730631574988365</threshold>
            <left_val>0.4341886043548584</left_val>
            <right_val>0.7060034275054932</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 7 10 2 -1.</_>
                <_>5 8 10 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.6767389974556863e-004</threshold>
            <left_val>0.3027887940406799</left_val>
            <right_val>0.6051574945449829</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 11 5 3 -1.</_>
                <_>15 12 5 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.0479710809886456e-003</threshold>
            <left_val>0.1798433959484100</left_val>
            <right_val>0.5675256848335266</right_val></_></_></trees>
      <stage_threshold>6.9566087722778320</stage_threshold>
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
                <_>5 3 10 9 -1.</_>
                <_>5 6 10 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0165106896311045</threshold>
            <left_val>0.6644225120544434</left_val>
            <right_val>0.1424857974052429</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 4 2 14 -1.</_>
                <_>9 11 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.7052499353885651e-003</threshold>
            <left_val>0.6325352191925049</left_val>
            <right_val>0.1288477033376694</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 5 4 12 -1.</_>
                <_>3 9 4 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.8069869149476290e-003</threshold>
            <left_val>0.1240288019180298</left_val>
            <right_val>0.6193193197250366</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 5 12 5 -1.</_>
                <_>8 5 4 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5402400167658925e-003</threshold>
            <left_val>0.1432143002748489</left_val>
            <right_val>0.5670015811920166</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 6 10 8 -1.</_>
                <_>5 10 10 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.6386279175058007e-004</threshold>
            <left_val>0.1657433062791824</left_val>
            <right_val>0.5905207991600037</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 0 6 9 -1.</_>
                <_>8 3 6 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9253729842603207e-003</threshold>
            <left_val>0.2695507109165192</left_val>
            <right_val>0.5738824009895325</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 12 1 8 -1.</_>
                <_>9 16 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.0214841030538082e-003</threshold>
            <left_val>0.1893538981676102</left_val>
            <right_val>0.5782774090766907</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 7 20 6 -1.</_>
                <_>0 9 20 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.6365420781075954e-003</threshold>
            <left_val>0.2309329062700272</left_val>
            <right_val>0.5695425868034363</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 0 6 17 -1.</_>
                <_>9 0 2 17 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5127769438549876e-003</threshold>
            <left_val>0.2759602069854736</left_val>
            <right_val>0.5956642031669617</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 6 4 -1.</_>
                <_>11 0 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0101574398577213</threshold>
            <left_val>0.1732538044452667</left_val>
            <right_val>0.5522047281265259</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 1 6 4 -1.</_>
                <_>7 1 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0119536602869630</threshold>
            <left_val>0.1339409947395325</left_val>
            <right_val>0.5559014081954956</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 1 6 16 -1.</_>
                <_>14 1 2 16 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.8859491944313049e-003</threshold>
            <left_val>0.3628703951835632</left_val>
            <right_val>0.6188849210739136</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 5 18 8 -1.</_>
                <_>0 5 9 4 2.</_>
                <_>9 9 9 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0801329165697098</threshold>
            <left_val>0.0912110507488251</left_val>
            <right_val>0.5475944876670837</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 15 10 4 -1.</_>
                <_>13 15 5 2 2.</_>
                <_>8 17 5 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.0643280111253262e-003</threshold>
            <left_val>0.3715142905712128</left_val>
            <right_val>0.5711399912834168</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 1 4 8 -1.</_>
                <_>3 1 2 4 2.</_>
                <_>5 5 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.3419450260698795e-003</threshold>
            <left_val>0.5953313708305359</left_val>
            <right_val>0.3318097889423370</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 6 14 10 -1.</_>
                <_>10 6 7 5 2.</_>
                <_>3 11 7 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0546011403203011</threshold>
            <left_val>0.1844065934419632</left_val>
            <right_val>0.5602846145629883</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 1 6 16 -1.</_>
                <_>4 1 2 16 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.9071690514683723e-003</threshold>
            <left_val>0.3594244122505188</left_val>
            <right_val>0.6131715178489685</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 18 20 2 -1.</_>
                <_>0 19 20 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.4718717951327562e-004</threshold>
            <left_val>0.5994353294372559</left_val>
            <right_val>0.3459562957286835</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 13 4 3 -1.</_>
                <_>8 14 4 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.3013808317482471e-003</threshold>
            <left_val>0.4172652065753937</left_val>
            <right_val>0.6990845203399658</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 14 2 3 -1.</_>
                <_>9 15 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.5017572119832039e-003</threshold>
            <left_val>0.4509715139865875</left_val>
            <right_val>0.7801457047462463</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 12 9 6 -1.</_>
                <_>0 14 9 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0241385009139776</threshold>
            <left_val>0.5438212752342224</left_val>
            <right_val>0.1319826990365982</right_val></_></_></trees>
      <stage_threshold>9.4985427856445313</stage_threshold>
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
                <_>5 7 3 4 -1.</_>
                <_>5 9 3 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9212230108678341e-003</threshold>
            <left_val>0.1415266990661621</left_val>
            <right_val>0.6199870705604553</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 3 2 16 -1.</_>
                <_>9 11 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.2748669541906565e-004</threshold>
            <left_val>0.6191074252128601</left_val>
            <right_val>0.1884928941726685</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 6 13 8 -1.</_>
                <_>3 10 13 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.1409931620582938e-004</threshold>
            <left_val>0.1487396955490112</left_val>
            <right_val>0.5857927799224854</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 3 8 2 -1.</_>
                <_>12 3 4 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.1878609918057919e-003</threshold>
            <left_val>0.2746909856796265</left_val>
            <right_val>0.6359239816665649</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 8 4 12 -1.</_>
                <_>8 12 4 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.1015717908740044e-003</threshold>
            <left_val>0.5870851278305054</left_val>
            <right_val>0.2175628989934921</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 3 8 6 -1.</_>
                <_>15 3 4 3 2.</_>
                <_>11 6 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1448440384119749e-003</threshold>
            <left_val>0.5880944728851318</left_val>
            <right_val>0.2979590892791748</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 1 6 19 -1.</_>
                <_>9 1 2 19 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.8977119363844395e-003</threshold>
            <left_val>0.2373327016830444</left_val>
            <right_val>0.5876647233963013</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 6 4 -1.</_>
                <_>11 0 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0216106791049242</threshold>
            <left_val>0.1220654994249344</left_val>
            <right_val>0.5194202065467835</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 1 9 3 -1.</_>
                <_>6 1 3 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.6299318782985210e-003</threshold>
            <left_val>0.2631230950355530</left_val>
            <right_val>0.5817409157752991</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 15 10 4 -1.</_>
                <_>13 15 5 2 2.</_>
                <_>8 17 5 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.9393711853772402e-004</threshold>
            <left_val>0.3638620078563690</left_val>
            <right_val>0.5698544979095459</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 3 6 10 -1.</_>
                <_>3 3 3 10 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0538786612451077</threshold>
            <left_val>0.4303531050682068</left_val>
            <right_val>0.7559366226196289</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 4 15 15 -1.</_>
                <_>3 9 15 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.8887349870055914e-003</threshold>
            <left_val>0.2122603058815002</left_val>
            <right_val>0.5613427162170410</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 5 8 6 -1.</_>
                <_>6 7 8 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.3635339457541704e-003</threshold>
            <left_val>0.5631849169731140</left_val>
            <right_val>0.2642767131328583</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 4 12 10 -1.</_>
                <_>10 4 6 5 2.</_>
                <_>4 9 6 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0240177996456623</threshold>
            <left_val>0.5797107815742493</left_val>
            <right_val>0.2751705944538117</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 4 4 4 -1.</_>
                <_>8 4 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.0543030404951423e-004</threshold>
            <left_val>0.2705242037773132</left_val>
            <right_val>0.5752568840980530</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 11 1 2 -1.</_>
                <_>15 12 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.4790197433903813e-004</threshold>
            <left_val>0.5435624718666077</left_val>
            <right_val>0.2334876954555512</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 11 2 2 -1.</_>
                <_>3 12 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.4091329649090767e-003</threshold>
            <left_val>0.5319424867630005</left_val>
            <right_val>0.2063155025243759</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>16 11 1 3 -1.</_>
                <_>16 12 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.4642629539594054e-003</threshold>
            <left_val>0.5418980717658997</left_val>
            <right_val>0.3068861067295075</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 15 6 4 -1.</_>
                <_>3 15 3 2 2.</_>
                <_>6 17 3 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.6352549428120255e-003</threshold>
            <left_val>0.3695372939109802</left_val>
            <right_val>0.6112868189811707</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 7 8 2 -1.</_>
                <_>6 8 8 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.3172752056270838e-004</threshold>
            <left_val>0.3565036952495575</left_val>
            <right_val>0.6025236248970032</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 11 1 3 -1.</_>
                <_>3 12 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.0998890977352858e-003</threshold>
            <left_val>0.1913982033729553</left_val>
            <right_val>0.5362827181816101</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 0 12 2 -1.</_>
                <_>6 1 12 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.4213981861248612e-004</threshold>
            <left_val>0.3835555016994476</left_val>
            <right_val>0.5529310107231140</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 14 2 3 -1.</_>
                <_>9 15 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.2655049581080675e-003</threshold>
            <left_val>0.4312896132469177</left_val>
            <right_val>0.7101895809173584</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 15 6 2 -1.</_>
                <_>7 16 6 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.9134991867467761e-004</threshold>
            <left_val>0.3984830975532532</left_val>
            <right_val>0.6391963958740234</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 5 4 6 -1.</_>
                <_>0 7 4 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0152841797098517</threshold>
            <left_val>0.2366732954978943</left_val>
            <right_val>0.5433713793754578</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 12 12 2 -1.</_>
                <_>8 12 4 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.8381411470472813e-003</threshold>
            <left_val>0.5817500948905945</left_val>
            <right_val>0.3239189088344574</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 3 1 9 -1.</_>
                <_>6 6 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.1093179071322083e-004</threshold>
            <left_val>0.5540593862533569</left_val>
            <right_val>0.2911868989467621</right_val></_></_>
        <_>
          <!-- tree 27 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 17 3 2 -1.</_>
                <_>11 17 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.1275060288608074e-003</threshold>
            <left_val>0.1775255054235458</left_val>
            <right_val>0.5196629166603088</right_val></_></_>
        <_>
          <!-- tree 28 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 9 2 2 -1.</_>
                <_>9 10 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.4576259097084403e-004</threshold>
            <left_val>0.3024170100688934</left_val>
            <right_val>0.5533593893051148</right_val></_></_>
        <_>
          <!-- tree 29 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 6 6 4 -1.</_>
                <_>9 6 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0226465407758951</threshold>
            <left_val>0.4414930939674377</left_val>
            <right_val>0.6975377202033997</right_val></_></_>
        <_>
          <!-- tree 30 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 17 3 2 -1.</_>
                <_>8 17 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.8804960418492556e-003</threshold>
            <left_val>0.2791394889354706</left_val>
            <right_val>0.5497952103614807</right_val></_></_>
        <_>
          <!-- tree 31 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 17 3 3 -1.</_>
                <_>11 17 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.0889107882976532e-003</threshold>
            <left_val>0.5263199210166931</left_val>
            <right_val>0.2385547012090683</right_val></_></_>
        <_>
          <!-- tree 32 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 12 3 2 -1.</_>
                <_>8 13 3 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.7318050377070904e-003</threshold>
            <left_val>0.4319379031658173</left_val>
            <right_val>0.6983600854873657</right_val></_></_>
        <_>
          <!-- tree 33 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 3 6 2 -1.</_>
                <_>11 3 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.8482700735330582e-003</threshold>
            <left_val>0.3082042932510376</left_val>
            <right_val>0.5390920042991638</right_val></_></_>
        <_>
          <!-- tree 34 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 11 14 4 -1.</_>
                <_>3 13 14 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5062530110299122e-005</threshold>
            <left_val>0.5521922111511231</left_val>
            <right_val>0.3120366036891937</right_val></_></_>
        <_>
          <!-- tree 35 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 10 18 4 -1.</_>
                <_>10 10 9 2 2.</_>
                <_>1 12 9 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0294755697250366</threshold>
            <left_val>0.5401322841644287</left_val>
            <right_val>0.1770603060722351</right_val></_></_>
        <_>
          <!-- tree 36 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 10 3 3 -1.</_>
                <_>0 11 3 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.1387329846620560e-003</threshold>
            <left_val>0.5178617835044861</left_val>
            <right_val>0.1211019009351730</right_val></_></_>
        <_>
          <!-- tree 37 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 1 6 6 -1.</_>
                <_>11 1 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0209429506212473</threshold>
            <left_val>0.5290294289588928</left_val>
            <right_val>0.3311221897602081</right_val></_></_>
        <_>
          <!-- tree 38 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 7 3 6 -1.</_>
                <_>9 7 1 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.5665529370307922e-003</threshold>
            <left_val>0.7471994161605835</left_val>
            <right_val>0.4451968967914581</right_val></_></_></trees>
      <stage_threshold>18.4129695892333980</stage_threshold>
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
                <_>1 0 18 9 -1.</_>
                <_>1 3 18 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.8206960996612906e-004</threshold>
            <left_val>0.2064086049795151</left_val>
            <right_val>0.6076732277870178</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 10 2 6 -1.</_>
                <_>12 13 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.6790600493550301e-003</threshold>
            <left_val>0.5851997137069702</left_val>
            <right_val>0.1255383938550949</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 5 19 8 -1.</_>
                <_>0 9 19 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.9827912375330925e-004</threshold>
            <left_val>0.0940184295177460</left_val>
            <right_val>0.5728961229324341</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 0 6 9 -1.</_>
                <_>9 0 2 9 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.8959012171253562e-004</threshold>
            <left_val>0.1781987994909287</left_val>
            <right_val>0.5694308876991272</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 3 6 1 -1.</_>
                <_>7 3 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.8560499195009470e-003</threshold>
            <left_val>0.1638399064540863</left_val>
            <right_val>0.5788664817810059</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 3 6 1 -1.</_>
                <_>13 3 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.8122469559311867e-003</threshold>
            <left_val>0.2085440009832382</left_val>
            <right_val>0.5508564710617065</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 10 4 6 -1.</_>
                <_>5 13 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.5896620461717248e-003</threshold>
            <left_val>0.5702760815620422</left_val>
            <right_val>0.1857215017080307</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 3 6 1 -1.</_>
                <_>13 3 2 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0100783398374915</threshold>
            <left_val>0.5116943120956421</left_val>
            <right_val>0.2189770042896271</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 4 12 6 -1.</_>
                <_>4 6 12 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0635263025760651</threshold>
            <left_val>0.7131379842758179</left_val>
            <right_val>0.4043813049793243</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 12 2 6 -1.</_>
                <_>15 14 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.1031491756439209e-003</threshold>
            <left_val>0.2567181885242462</left_val>
            <right_val>0.5463973283767700</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 3 2 2 -1.</_>
                <_>10 3 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.4035000242292881e-003</threshold>
            <left_val>0.1700665950775147</left_val>
            <right_val>0.5590974092483521</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 3 3 1 -1.</_>
                <_>10 3 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.5226360410451889e-003</threshold>
            <left_val>0.5410556793212891</left_val>
            <right_val>0.2619054019451141</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 1 4 14 -1.</_>
                <_>3 1 2 14 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0179974399507046</threshold>
            <left_val>0.3732436895370483</left_val>
            <right_val>0.6535220742225647</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 4 4 -1.</_>
                <_>11 0 2 2 2.</_>
                <_>9 2 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.4538191072642803e-003</threshold>
            <left_val>0.2626481950283051</left_val>
            <right_val>0.5537446141242981</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 5 1 14 -1.</_>
                <_>7 12 1 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0118807600811124</threshold>
            <left_val>0.2003753930330277</left_val>
            <right_val>0.5544745922088623</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>19 0 1 4 -1.</_>
                <_>19 2 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.2713660253211856e-003</threshold>
            <left_val>0.5591902732849121</left_val>
            <right_val>0.3031975924968720</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 6 4 -1.</_>
                <_>8 5 3 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.1376109905540943e-003</threshold>
            <left_val>0.2730407118797302</left_val>
            <right_val>0.5646508932113648</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 18 3 2 -1.</_>
                <_>10 18 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.2651998810470104e-003</threshold>
            <left_val>0.1405909061431885</left_val>
            <right_val>0.5461820960044861</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 18 3 2 -1.</_>
                <_>9 18 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.9602861031889915e-003</threshold>
            <left_val>0.1795035004615784</left_val>
            <right_val>0.5459290146827698</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 5 12 6 -1.</_>
                <_>4 7 12 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.8448226451873779e-003</threshold>
            <left_val>0.5736783146858215</left_val>
            <right_val>0.2809219956398010</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 12 2 6 -1.</_>
                <_>3 14 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.6430689767003059e-003</threshold>
            <left_val>0.2370675951242447</left_val>
            <right_val>0.5503826141357422</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 8 2 12 -1.</_>
                <_>10 12 2 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.9997808635234833e-003</threshold>
            <left_val>0.5608199834823608</left_val>
            <right_val>0.3304282128810883</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 18 3 2 -1.</_>
                <_>8 18 1 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.1221720166504383e-003</threshold>
            <left_val>0.1640105992555618</left_val>
            <right_val>0.5378993153572083</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 6 2 -1.</_>
                <_>11 0 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0156249096617103</threshold>
            <left_val>0.5227649211883545</left_val>
            <right_val>0.2288603931665421</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 11 9 3 -1.</_>
                <_>5 12 9 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0103564197197557</threshold>
            <left_val>0.7016193866729736</left_val>
            <right_val>0.4252927899360657</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 6 2 -1.</_>
                <_>11 0 2 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.7960809469223022e-003</threshold>
            <left_val>0.2767347097396851</left_val>
            <right_val>0.5355830192565918</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 1 18 5 -1.</_>
                <_>7 1 6 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1622693985700607</threshold>
            <left_val>0.4342240095138550</left_val>
            <right_val>0.7442579269409180</right_val></_></_>
        <_>
          <!-- tree 27 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 0 4 4 -1.</_>
                <_>10 0 2 2 2.</_>
                <_>8 2 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.5542530715465546e-003</threshold>
            <left_val>0.5726485848426819</left_val>
            <right_val>0.2582125067710877</right_val></_></_>
        <_>
          <!-- tree 28 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 12 1 3 -1.</_>
                <_>3 13 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1309209987521172e-003</threshold>
            <left_val>0.2106848061084747</left_val>
            <right_val>0.5361018776893616</right_val></_></_>
        <_>
          <!-- tree 29 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 14 5 3 -1.</_>
                <_>8 15 5 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0132084200158715</threshold>
            <left_val>0.7593790888786316</left_val>
            <right_val>0.4552468061447144</right_val></_></_>
        <_>
          <!-- tree 30 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 4 10 12 -1.</_>
                <_>5 4 5 6 2.</_>
                <_>10 10 5 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0659966766834259</threshold>
            <left_val>0.1252475976943970</left_val>
            <right_val>0.5344039797782898</right_val></_></_>
        <_>
          <!-- tree 31 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 6 9 12 -1.</_>
                <_>9 10 9 4 3.</_></rects>
      
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

