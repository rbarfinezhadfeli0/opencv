# Documentation for `data/haarcascades_cuda/haarcascade_smile.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_smile.xml`
- **File Name**: `haarcascade_smile.xml`
- **File Size**: 281,795 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_smile.xml](../../data/haarcascades_cuda/haarcascade_smile.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
  Smile detector
  Contributed by Oscar Deniz Suarez
  More information can be found at http://visilab.etsii.uclm.es/personas/oscar/oscar.html

//////////////////////////////////////////////////////////////////////////
| Contributors License Agreement
| IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
|   By downloading, copying, installing or using the software you agree
|   to this license.
|   If you do not agree to this license, do not download, install,
|   copy or use the software.
|
| Copyright (c) 2011, Modesto Castrillon-Santana (IUSIANI, Universidad de
| Las Palmas de Gran Canaria, Spain).
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

-->
<opencv_storage>
<!-- Automatically converted from data/classifier, window size = 36x18 -->
<SmileDetector type_id="opencv-haar-classifier">
  <size>
    36 18</size>
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
                  0 0 2 4 -1.</_>
                <_>
                  0 2 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.8783610691316426e-004</threshold>
            <left_val>0.5921934843063355</left_val>
            <right_val>-0.4416360855102539</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  34 10 2 8 -1.</_>
                <_>
                  34 14 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.2209611274302006e-004</threshold>
            <left_val>0.3031865060329437</left_val>
            <right_val>-0.3291291892528534</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 10 2 8 -1.</_>
                <_>
                  0 14 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.9940118333324790e-004</threshold>
            <left_val>0.4856331050395966</left_val>
            <right_val>-0.4292306005954742</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 0 18 10 -1.</_>
                <_>
                  24 0 9 5 2.</_>
                <_>
                  15 5 9 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0372891984879971</threshold>
            <left_val>-0.2866730093955994</left_val>
            <right_val>0.5997999906539917</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 0 4 4 -1.</_>
                <_>
                  7 0 2 4 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>1.4334049774333835e-003</threshold>
            <left_val>-0.3489313125610352</left_val>
            <right_val>0.4048275053501129</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 5 6 4 -1.</_>
                <_>
                  15 6 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.7213020995259285e-003</threshold>
            <left_val>0.7571418881416321</left_val>
            <right_val>-0.1222594976425171</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  13 6 8 3 -1.</_>
                <_>
                  13 7 8 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.1067271530628204e-003</threshold>
            <left_val>-0.1665772050619125</left_val>
            <right_val>0.7509614825248718</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 6 8 4 -1.</_>
                <_>
                  14 7 8 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.7238711528480053e-003</threshold>
            <left_val>0.6266279220581055</left_val>
            <right_val>-0.1912745982408524</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 10 2 8 -1.</_>
                <_>
                  0 14 2 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.4225031160749495e-004</threshold>
            <left_val>-0.2394447028636932</left_val>
            <right_val>0.4484061896800995</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  34 0 2 16 -1.</_>
                <_>
                  35 0 1 8 2.</_>
                <_>
                  34 8 1 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.6867710510268807e-003</threshold>
            <left_val>-0.1843906939029694</left_val>
            <right_val>0.0917824134230614</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 0 4 7 -1.</_>
                <_>
                  3 0 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0146256200969219</threshold>
            <left_val>0.1616805940866470</left_val>
            <right_val>-0.8150117993354797</right_val></_></_></trees>
      <stage_threshold>-1.2678639888763428</stage_threshold>
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
                  4 7 28 3 -1.</_>
                <_>
                  11 7 14 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0381411388516426</threshold>
            <left_val>-0.3327588140964508</left_val>
            <right_val>0.7783334255218506</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  34 0 2 2 -1.</_>
                <_>
                  34 1 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.3136120105627924e-004</threshold>
            <left_val>0.3635309040546417</left_val>
            <right_val>-0.3204346895217896</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 12 4 6 -1.</_>
                <_>
                  0 15 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.8757019210606813e-003</threshold>
            <left_val>0.7135239243507385</left_val>
            <right_val>-0.3518598973751068</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  34 0 2 2 -1.</_>
                <_>
                  34 1 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.4266290236264467e-003</threshold>
            <left_val>0.0681008473038673</left_val>
            <right_val>-0.6172732710838318</right_val></_></_>
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
            <threshold>-2.4605958606116474e-004</threshold>
            <left_val>0.5727149844169617</left_val>
            <right_val>-0.3786099851131439</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  17 5 9 12 -1.</_>
                <_>
                  20 5 3 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0318226404488087</threshold>
            <left_val>-0.6348456144332886</left_val>
            <right_val>0.1164183989167213</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 5 9 12 -1.</_>
                <_>
                  13 5 3 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0171309504657984</threshold>
            <left_val>-0.6279314756393433</left_val>
            <right_val>0.3247947096824646</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 0 32 1 -1.</_>
                <_>
                  4 0 16 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.3903783708810806e-003</threshold>
            <left_val>-0.2757895886898041</left_val>
            <right_val>0.2233072966337204</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 3 3 -1.</_>
                <_>
                  1 0 1 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.2802520543336868e-003</threshold>
            <left_val>0.1897764056921005</left_val>
            <right_val>-0.6881762146949768</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  32 7 4 7 -1.</_>
                <_>
                  33 8 2 7 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>2.6840099599212408e-003</threshold>
            <left_val>-0.2235050052404404</left_val>
            <right_val>0.1372579932212830</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 0 8 6 -1.</_>
                <_>
                  7 0 4 3 2.</_>
                <_>
                  11 3 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0106046395376325</threshold>
            <left_val>-0.2142623066902161</left_val>
            <right_val>0.5620787143707275</right_val></_></_></trees>
      <stage_threshold>-1.5844069719314575</stage_threshold>
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
                  0 0 2 2 -1.</_>
                <_>
                  0 1 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.1677199876867235e-004</threshold>
            <left_val>0.4659548103809357</left_val>
            <right_val>-0.3742581903934479</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  27 1 8 9 -1.</_>
                <_>
                  29 3 4 9 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0551206283271313</threshold>
            <left_val>0.5417978763580322</left_val>
            <right_val>-0.2265765070915222</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  1 10 1 8 -1.</_>
                <_>
                  1 14 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.4742640824988484e-004</threshold>
            <left_val>0.3770307004451752</left_val>
            <right_val>-0.3348644077777863</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 6 30 9 -1.</_>
                <_>
                  13 9 10 3 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.3950783908367157</threshold>
            <left_val>-0.1814441978931427</left_val>
            <right_val>0.8132591843605042</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  12 5 8 6 -1.</_>
                <_>
                  12 7 8 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0405094102025032</threshold>
            <left_val>-0.0953694134950638</left_val>
            <right_val>0.8059561848640442</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 4 6 3 -1.</_>
                <_>
                  16 5 6 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.8735421150922775e-003</threshold>
            <left_val>-0.1402366012334824</left_val>
            <right_val>0.6164302825927734</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 2 18 -1.</_>
                <_>
                  0 0 1 9 2.</_>
                <_>
                  1 9 1 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0105780400335789</threshold>
            <left_val>0.1293267011642456</left_val>
            <right_val>-0.7482334971427918</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  34 2 2 14 -1.</_>
                <_>
                  35 2 1 7 2.</_>
                <_>
                  34 9 1 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.2986393719911575e-003</threshold>
            <left_val>0.0589406006038189</left_val>
            <right_val>-0.4410730004310608</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 2 2 14 -1.</_>
                <_>
                  0 2 1 7 2.</_>
                <_>
                  1 9 1 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.0301607698202133e-003</threshold>
            <left_val>-0.6630973219871521</left_val>
            <right_val>0.1810476928949356</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  35 0 1 4 -1.</_>
                <_>
                  35 2 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.0947990085696802e-004</threshold>
            <left_val>0.2211259007453919</left_val>
            <right_val>-0.2730903923511505</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  5 0 24 18 -1.</_>
                <_>
                  5 0 12 9 2.</_>
                <_>
                  17 9 12 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1168550997972488</threshold>
            <left_val>-0.7720596790313721</left_val>
            <right_val>0.1248165965080261</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  35 16 1 2 -1.</_>
                <_>
                  35 17 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.3603649828583002e-005</threshold>
            <left_val>0.1367060989141464</left_val>
            <right_val>-0.1612793952226639</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 16 1 2 -1.</_>
                <_>
                  0 17 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5056360280141234e-004</threshold>
            <left_val>0.4486046135425568</left_val>
            <right_val>-0.2171128988265991</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  17 6 8 12 -1.</_>
                <_>
                  19 6 4 12 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0163946095854044</threshold>
            <left_val>-0.6582735180854797</left_val>
            <right_val>0.1674550026655197</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  11 5 8 13 -1.</_>
                <_>
                  13 5 4 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0144828604534268</threshold>
            <left_val>-0.6834514737129211</left_val>
            <right_val>0.1345615983009338</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  35 16 1 2 -1.</_>
                <_>
                  35 17 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.9269471017178148e-005</threshold>
            <left_val>-0.1499813944101334</left_val>
            <right_val>0.1601772010326386</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 9 12 3 -1.</_>
                <_>
                  10 10 12 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.4323131702840328e-003</threshold>
            <left_val>-0.1684845983982086</left_val>
            <right_val>0.5396398901939392</right_val></_></_></trees>
      <stage_threshold>-1.3820559978485107</stage_threshold>
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
                  0 10 1 8 -1.</_>
                <_>
                  0 14 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.3472499237395823e-004</threshold>
            <left_val>0.4394924044609070</left_val>
            <right_val>-0.4224875867366791</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  20 0 10 10 -1.</_>
                <_>
                  25 0 5 5 2.</_>
                <_>
                  20 5 5 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0329953208565712</threshold>
            <left_val>-0.1979825049638748</left_val>
            <right_val>0.5953487157821655</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 1 4 -1.</_>
                <_>
                  0 2 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.1011828579939902e-004</threshold>
            <left_val>0.4440306127071381</left_val>
            <right_val>-0.3074846863746643</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  19 0 13 18 -1.</_>
                <_>
                  19 9 13 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0819697380065918</threshold>
            <left_val>-0.5333436727523804</left_val>
            <right_val>0.1671810001134872</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 0 14 6 -1.</_>
                <_>
                  4 0 7 3 2.</_>
                <_>
                  11 3 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0177787002176046</threshold>
            <left_val>-0.2045017927885056</left_val>
            <right_val>0.5144413113594055</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 5 6 6 -1.</_>
                <_>
                  16 7 6 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0228346996009350</threshold>
            <left_val>-0.1484607011079788</left_val>
            <right_val>0.5624278783798218</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  13 7 7 8 -1.</_>
                <_>
                  13 9 7 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0386043414473534</threshold>
            <left_val>-0.1273147016763687</left_val>
            <right_val>0.8149448037147522</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  33 0 3 1 -1.</_>
                <_>
                  34 0 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.3286908445879817e-004</threshold>
            <left_val>-0.3719344139099121</left_val>
            <right_val>0.0676164999604225</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 1 10 4 -1.</_>
                <_>
                  6 2 10 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0232290402054787</threshold>
            <left_val>0.7123206257820129</left_val>
            <right_val>-0.1158939003944397</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 2 6 16 -1.</_>
                <_>
                  18 2 3 8 2.</_>
                <_>
                  15 10 3 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0195753592997789</threshold>
            <left_val>-0.6899073123931885</left_val>
            <right_val>0.1399950981140137</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 10 1 8 -1.</_>
                <_>
                  0 14 1 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.1991271427832544e-004</threshold>
            <left_val>-0.1835464984178543</left_val>
            <right_val>0.4943555891513825</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  27 4 6 6 -1.</_>
                <_>
                  29 6 2 6 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0570897497236729</threshold>
            <left_val>0.6260784864425659</left_val>
            <right_val>-0.0785768479108810</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 5 8 8 -1.</_>
                <_>
                  16 5 4 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0256996992975473</threshold>
            <left_val>0.1155714020133019</left_val>
            <right_val>-0.8193519115447998</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  27 5 6 6 -1.</_>
                <_>
                  29 7 2 6 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0325796194374561</threshold>
            <left_val>-0.1176773980259895</left_val>
            <right_val>0.4277622103691101</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 5 6 6 -1.</_>
                <_>
                  7 7 6 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0205922499299049</threshold>
            <left_val>0.4868524074554443</left_val>
            <right_val>-0.2131853997707367</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  12 5 12 9 -1.</_>
                <_>
                  15 5 6 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0174852795898914</threshold>
            <left_val>-0.5228734016418457</left_val>
            <right_val>0.1339704990386963</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 3 1 -1.</_>
                <_>
                  1 0 1 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.9153228327631950e-004</threshold>
            <left_val>0.0963044911623001</left_val>
            <right_val>-0.6886307001113892</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 4 18 6 -1.</_>
                <_>
                  15 6 18 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0575339011847973</threshold>
            <left_val>-0.0870805233716965</left_val>
            <right_val>0.4048064947128296</right_val></_></_></trees>
      <stage_threshold>-1.3879380226135254</stage_threshold>
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
                  0 10 1 6 -1.</_>
                <_>
                  0 13 1 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.6606198884546757e-004</threshold>
            <left_val>0.4277374148368835</left_val>
            <right_val>-0.3542076945304871</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  3 6 30 6 -1.</_>
                <_>
                  13 8 10 2 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.3055455982685089</threshold>
            <left_val>-0.1639281064271927</left_val>
            <right_val>0.8606523275375366</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  11 7 12 4 -1.</_>
                <_>
                  11 8 12 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0114494003355503</threshold>
            <left_val>0.5972732901573181</left_val>
            <right_val>-0.2323434054851532</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 8 9 3 -1.</_>
                <_>
                  14 9 9 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.3891541212797165e-003</threshold>
            <left_val>-0.1291541010141373</left_val>
            <right_val>0.6105204224586487</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 8 7 4 -1.</_>
                <_>
                  14 9 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.4334248676896095e-003</threshold>
            <left_val>0.4792853891849518</left_val>
            <right_val>-0.1900272965431213</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  12 7 18 6 -1.</_>
                <_>
                  12 9 18 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0538089312613010</threshold>
            <left_val>-0.1149377003312111</left_val>
            <right_val>0.5339453816413879</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 8 3 10 -1.</_>
                <_>
                  7 13 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.7580219688825309e-004</threshold>
            <left_val>-0.3459854125976563</left_val>
            <right_val>0.2548804879188538</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  35 10 1 6 -1.</_>
                <_>
                  35 13 1 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.3450840197037905e-004</threshold>
            <left_val>0.2241459041833878</left_val>
            <right_val>-0.1955007016658783</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 10 1 6 -1.</_>
                <_>
                  0 13 1 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.0016911700367928e-004</threshold>
            <left_val>-0.1972054988145828</left_val>
            <right_val>0.4967764019966126</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 13 9 5 -1.</_>
                <_>
                  21 13 3 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0150632699951530</threshold>
            <left_val>0.1063077002763748</left_val>
            <right_val>-0.4113821089267731</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 9 6 4 -1.</_>
                <_>
                  15 10 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.7588870190083981e-003</threshold>
            <left_val>-0.1537311971187592</left_val>
            <right_val>0.4893161952495575</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 4 18 8 -1.</_>
                <_>
                  16 6 18 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0454101189970970</threshold>
            <left_val>-0.0735593065619469</left_val>
            <right_val>0.2773792147636414</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 14 9 3 -1.</_>
                <_>
                  12 14 3 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0145996697247028</threshold>
            <left_val>-0.7096682786941528</left_val>
            <right_val>0.0975155606865883</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  32 0 4 6 -1.</_>
                <_>
                  32 0 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0172360707074404</threshold>
            <left_val>0.0168695393949747</left_val>
            <right_val>-0.5738832950592041</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 4 6 -1.</_>
                <_>
                  2 0 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0142307104542851</threshold>
            <left_val>0.0947145000100136</left_val>
            <right_val>-0.7839525938034058</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  27 0 6 7 -1.</_>
                <_>
                  29 2 2 7 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0437068603932858</threshold>
            <left_val>0.6097965240478516</left_val>
            <right_val>-0.1560188978910446</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 1 4 -1.</_>
                <_>
                  0 2 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-6.2343222089111805e-004</threshold>
            <left_val>0.3485119044780731</left_val>
            <right_val>-0.2170491069555283</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  27 8 6 4 -1.</_>
                <_>
                  29 10 2 4 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0192450508475304</threshold>
            <left_val>-0.1171097978949547</left_val>
            <right_val>0.3070116043090820</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 9 27 6 -1.</_>
                <_>
                  13 11 9 2 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2703577876091003</threshold>
            <left_val>-0.0900964364409447</left_val>
            <right_val>0.7665696144104004</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  31 14 2 3 -1.</_>
                <_>
                  31 14 1 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.5394480801187456e-004</threshold>
            <left_val>-0.2002478986978531</left_val>
            <right_val>0.1249336004257202</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  10 0 5 6 -1.</_>
                <_>
                  8 2 5 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0360139608383179</threshold>
            <left_val>0.6702855825424194</left_val>
            <right_val>-0.1057187989354134</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 7 11 3 -1.</_>
                <_>
                  14 8 11 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.2952791601419449e-003</threshold>
            <left_val>-0.1057471036911011</left_val>
            <right_val>0.4509387910366058</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 12 2 6 -1.</_>
                <_>
                  0 15 2 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.3304709359072149e-004</threshold>
            <left_val>0.2793382108211517</left_val>
            <right_val>-0.2457676976919174</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  34 13 2 4 -1.</_>
                <_>
                  34 15 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.9147620807634667e-005</threshold>
            <left_val>0.0858138129115105</left_val>
            <right_val>-0.0954695865511894</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 13 2 4 -1.</_>
                <_>
                  0 15 2 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.4382669148035347e-004</threshold>
            <left_val>-0.2022008001804352</left_val>
            <right_val>0.5454357862472534</right_val></_></_></trees>
      <stage_threshold>-1.3538850545883179</stage_threshold>
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
                  3 6 4 12 -1.</_>
                <_>
                  3 10 4 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.9610757529735565e-003</threshold>
            <left_val>-0.3672207891941071</left_val>
            <right_val>0.4315434992313385</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  14 0 22 12 -1.</_>
                <_>
                  25 0 11 6 2.</_>
                <_>
                  14 6 11 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0633948296308517</threshold>
            <left_val>-0.2073971033096314</left_val>
            <right_val>0.5742601752281189</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  8 1 7 6 -1.</_>
                <_>
                  6 3 7 2 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0531933493912220</threshold>
            <left_val>0.7255092263221741</left_val>
            <right_val>-0.1434202045202255</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  12 5 14 3 -1.</_>
                <_>
                  12 6 14 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0154607696458697</threshold>
            <left_val>-0.0960538163781166</left_val>
            <right_val>0.7578523755073547</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  7 6 7 4 -1.</_>
                <_>
                  6 7 7 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0176431406289339</threshold>
            <left_val>0.6681562066078186</left_val>
            <right_val>-0.1417672932147980</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  18 3 6 4 -1.</_>
                <_>
                  18 4 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.5065636560320854e-003</threshold>
            <left_val>-0.0962597429752350</left_val>
            <right_val>0.4699633121490479</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  4 5 5 6 -1.</_>
                <_>
                  4 7 5 2 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.0446049533784389e-003</threshold>
            <left_val>-0.1973251998424530</left_val>
            <right_val>0.4283801019191742</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  33 0 3 4 -1.</_>
                <_>
                  34 0 1 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.2312041148543358e-003</threshold>
            <left_val>0.1186169013381004</left_val>
            <right_val>-0.6103963255882263</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  9 0 6 18 -1.</_>
                <_>
                  9 9 6 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0401590503752232</threshold>
            <left_val>-0.4166434109210968</left_val>
            <right_val>0.2167232930660248</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  6 6 24 6 -1.</_>
                <_>
                  14 8 8 2 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2852425873279572</threshold>
            <left_val>-0.1043575033545494</left_val>
            <right_val>0.8573396801948547</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  16 8 4 4 -1.</_>
                <_>
                  16 9 4 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.9264221452176571e-003</threshold>
            <left_val>0.4706046879291534</left_val>
            <right_val>-0.1399745941162109</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  13 8 13 4 -1.</_>
                <_>
                  13 9 13 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0137817002832890</threshold>
            <left_val>-0.1271356940269470</left_val>
            <right_val>0.4461891949176788</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 16 2 2 -1.</_>
                <_>
                  0 17 2 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.9873598618432879e-004</threshold>
            <left_val>0.4702663123607636</left_val>
            <right_val>-0.1548373997211456</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  35 14 1 4 -1.</_>
                <_>
                  35 15 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.5621389320585877e-004</threshold>
            <left_val>0.1885481029748917</left_val>
            <right_val>-0.0778397768735886</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 14 1 4 -1.</_>
                <_>
                  0 15 1 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.7597760092467070e-004</threshold>
            <left_val>0.5769770145416260</left_val>
            <right_val>-0.1335622072219849</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  15 6 9 7 -1.</_>
                <_>
                  18 6 3 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0106659103184938</threshold>
            <left_val>-0.4106529951095581</left_val>
            <right_val>0.1556212007999420</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  0 0 3 4 -1.</_>
                <_>
                  1 0 1 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.4135230816900730e-003</threshold>
            <left_val>-0.7636343240737915</left_val>
            <right_val>0.1020964980125427</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>
                  34 16 2 2 -1.</_>
                <_>
                  35 16 1 1 2.</_>
                <_>
                  34 17 1 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.6471868447260931e-005</threshold>
            <left_val>-0.1644393056631088</left_val>
            <right_val>0.2290841937065125</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
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

