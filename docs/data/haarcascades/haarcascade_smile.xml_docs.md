# Documentation for `data/haarcascades/haarcascade_smile.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_smile.xml`
- **File Name**: `haarcascade_smile.xml`
- **File Size**: 188,506 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_smile.xml](../../data/haarcascades/haarcascade_smile.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

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
<cascade type_id="opencv-cascade-classifier"><stageType>BOOST</stageType>
  <featureType>HAAR</featureType>
  <height>18</height>
  <width>36</width>
  <stageParams>
    <maxWeakCount>53</maxWeakCount></stageParams>
  <featureParams>
    <maxCatCount>0</maxCatCount></featureParams>
  <stageNum>20</stageNum>
  <stages>
    <_>
      <maxWeakCount>11</maxWeakCount>
      <stageThreshold>-1.2678639888763428e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 0 -4.8783610691316426e-04</internalNodes>
          <leafValues>
            5.9219348430633545e-01 -4.4163608551025391e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1 -4.2209611274302006e-04</internalNodes>
          <leafValues>
            3.0318650603294373e-01 -3.2912918925285339e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 2 -4.9940118333324790e-04</internalNodes>
          <leafValues>
            4.8563310503959656e-01 -4.2923060059547424e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 3 3.7289198487997055e-02</internalNodes>
          <leafValues>
            -2.8667300939559937e-01 5.9979999065399170e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 4 1.4334049774333835e-03</internalNodes>
          <leafValues>
            -3.4893131256103516e-01 4.0482750535011292e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 5 -7.7213020995259285e-03</internalNodes>
          <leafValues>
            7.5714188814163208e-01 -1.2225949764251709e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 6 8.1067271530628204e-03</internalNodes>
          <leafValues>
            -1.6657720506191254e-01 7.5096148252487183e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 7 -7.7238711528480053e-03</internalNodes>
          <leafValues>
            6.2662792205810547e-01 -1.9127459824085236e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 8 4.4225031160749495e-04</internalNodes>
          <leafValues>
            -2.3944470286369324e-01 4.4840618968009949e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 9 -1.6867710510268807e-03</internalNodes>
          <leafValues>
            -1.8439069390296936e-01 9.1782413423061371e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 10 1.4625620096921921e-02</internalNodes>
          <leafValues>
            1.6168059408664703e-01 -8.1501179933547974e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>11</maxWeakCount>
      <stageThreshold>-1.5844069719314575e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 11 3.8141138851642609e-02</internalNodes>
          <leafValues>
            -3.3275881409645081e-01 7.7833342552185059e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 12 -1.3136120105627924e-04</internalNodes>
          <leafValues>
            3.6353090405464172e-01 -3.2043468952178955e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 13 -3.8757019210606813e-03</internalNodes>
          <leafValues>
            7.1352392435073853e-01 -3.5185989737510681e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 14 1.4266290236264467e-03</internalNodes>
          <leafValues>
            6.8100847303867340e-02 -6.1727327108383179e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 15 -2.4605958606116474e-04</internalNodes>
          <leafValues>
            5.7271498441696167e-01 -3.7860998511314392e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 16 -3.1822640448808670e-02</internalNodes>
          <leafValues>
            -6.3484561443328857e-01 1.1641839891672134e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 17 -1.7130950465798378e-02</internalNodes>
          <leafValues>
            -6.2793147563934326e-01 3.2479470968246460e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 18 -9.3903783708810806e-03</internalNodes>
          <leafValues>
            -2.7578958868980408e-01 2.2330729663372040e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 19 2.2802520543336868e-03</internalNodes>
          <leafValues>
            1.8977640569210052e-01 -6.8817621469497681e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 20 2.6840099599212408e-03</internalNodes>
          <leafValues>
            -2.2350500524044037e-01 1.3725799322128296e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 21 1.0604639537632465e-02</internalNodes>
          <leafValues>
            -2.1426230669021606e-01 5.6207871437072754e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>17</maxWeakCount>
      <stageThreshold>-1.3820559978485107e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 22 -3.1677199876867235e-04</internalNodes>
          <leafValues>
            4.6595481038093567e-01 -3.7425819039344788e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 23 -5.5120628327131271e-02</internalNodes>
          <leafValues>
            5.4179787635803223e-01 -2.2657650709152222e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 24 -6.4742640824988484e-04</internalNodes>
          <leafValues>
            3.7703070044517517e-01 -3.3486440777778625e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 25 3.9507839083671570e-01</internalNodes>
          <leafValues>
            -1.8144419789314270e-01 8.1325918436050415e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 26 4.0509410202503204e-02</internalNodes>
          <leafValues>
            -9.5369413495063782e-02 8.0595618486404419e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 27 4.8735421150922775e-03</internalNodes>
          <leafValues>
            -1.4023660123348236e-01 6.1643028259277344e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 28 1.0578040033578873e-02</internalNodes>
          <leafValues>
            1.2932670116424561e-01 -7.4823349714279175e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 29 9.2986393719911575e-03</internalNodes>
          <leafValues>
            5.8940600603818893e-02 -4.4107300043106079e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 30 -5.0301607698202133e-03</internalNodes>
          <leafValues>
            -6.6309732198715210e-01 1.8104769289493561e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 31 -1.0947990085696802e-04</internalNodes>
          <leafValues>
            2.2112590074539185e-01 -2.7309039235115051e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 32 -1.1685509979724884e-01</internalNodes>
          <leafValues>
            -7.7205967903137207e-01 1.2481659650802612e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 33 -4.3603649828583002e-05</internalNodes>
          <leafValues>
            1.3670609891414642e-01 -1.6127939522266388e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 34 -1.5056360280141234e-04</internalNodes>
          <leafValues>
            4.4860461354255676e-01 -2.1711289882659912e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 35 -1.6394609585404396e-02</internalNodes>
          <leafValues>
            -6.5827351808547974e-01 1.6745500266551971e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 36 -1.4482860453426838e-02</internalNodes>
          <leafValues>
            -6.8345147371292114e-01 1.3456159830093384e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 37 3.9269471017178148e-05</internalNodes>
          <leafValues>
            -1.4998139441013336e-01 1.6017720103263855e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 38 7.4323131702840328e-03</internalNodes>
          <leafValues>
            -1.6848459839820862e-01 5.3963989019393921e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>18</maxWeakCount>
      <stageThreshold>-1.3879380226135254e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 39 -4.3472499237395823e-04</internalNodes>
          <leafValues>
            4.3949240446090698e-01 -4.2248758673667908e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 40 3.2995320856571198e-02</internalNodes>
          <leafValues>
            -1.9798250496387482e-01 5.9534871578216553e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 41 -4.1011828579939902e-04</internalNodes>
          <leafValues>
            4.4403061270713806e-01 -3.0748468637466431e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 42 -8.1969738006591797e-02</internalNodes>
          <leafValues>
            -5.3334367275238037e-01 1.6718100011348724e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 43 1.7778700217604637e-02</internalNodes>
          <leafValues>
            -2.0450179278850555e-01 5.1444131135940552e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 44 2.2834699600934982e-02</internalNodes>
          <leafValues>
            -1.4846070110797882e-01 5.6242787837982178e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 45 3.8604341447353363e-02</internalNodes>
          <leafValues>
            -1.2731470167636871e-01 8.1494480371475220e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 46 -7.3286908445879817e-04</internalNodes>
          <leafValues>
            -3.7193441390991211e-01 6.7616499960422516e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 47 -2.3229040205478668e-02</internalNodes>
          <leafValues>
            7.1232062578201294e-01 -1.1589390039443970e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 48 -1.9575359299778938e-02</internalNodes>
          <leafValues>
            -6.8990731239318848e-01 1.3999509811401367e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 49 4.1991271427832544e-04</internalNodes>
          <leafValues>
            -1.8354649841785431e-01 4.9435558915138245e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 50 -5.7089749723672867e-02</internalNodes>
          <leafValues>
            6.2607848644256592e-01 -7.8576847910881042e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 51 2.5699699297547340e-02</internalNodes>
          <leafValues>
            1.1557140201330185e-01 -8.1935191154479980e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 52 3.2579619437456131e-02</internalNodes>
          <leafValues>
            -1.1767739802598953e-01 4.2776221036911011e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 53 -2.0592249929904938e-02</internalNodes>
          <leafValues>
            4.8685240745544434e-01 -2.1318539977073669e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 54 -1.7485279589891434e-02</internalNodes>
          <leafValues>
            -5.2287340164184570e-01 1.3397049903869629e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 55 8.9153228327631950e-04</internalNodes>
          <leafValues>
            9.6304491162300110e-02 -6.8863070011138916e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 56 5.7533901184797287e-02</internalNodes>
          <leafValues>
            -8.7080523371696472e-02 4.0480649471282959e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>25</maxWeakCount>
      <stageThreshold>-1.3538850545883179e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 57 -4.6606198884546757e-04</internalNodes>
          <leafValues>
            4.2773741483688354e-01 -3.5420769453048706e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 58 3.0554559826850891e-01</internalNodes>
          <leafValues>
            -1.6392810642719269e-01 8.6065232753753662e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 59 -1.1449400335550308e-02</internalNodes>
          <leafValues>
            5.9727329015731812e-01 -2.3234340548515320e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 60 6.3891541212797165e-03</internalNodes>
          <leafValues>
            -1.2915410101413727e-01 6.1052042245864868e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 61 -8.4334248676896095e-03</internalNodes>
          <leafValues>
            4.7928538918495178e-01 -1.9002729654312134e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 62 5.3808931261301041e-02</internalNodes>
          <leafValues>
            -1.1493770033121109e-01 5.3394538164138794e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 63 -4.7580219688825309e-04</internalNodes>
          <leafValues>
            -3.4598541259765625e-01 2.5488048791885376e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 64 -1.3450840197037905e-04</internalNodes>
          <leafValues>
            2.2414590418338776e-01 -1.9550070166587830e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 65 5.0016911700367928e-04</internalNodes>
          <leafValues>
            -1.9720549881458282e-01 4.9677640199661255e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 66 1.5063269995152950e-02</internalNodes>
          <leafValues>
            1.0630770027637482e-01 -4.1138210892677307e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 67 7.7588870190083981e-03</internalNodes>
          <leafValues>
            -1.5373119711875916e-01 4.8931619524955750e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 68 4.5410118997097015e-02</internalNodes>
          <leafValues>
            -7.3559306561946869e-02 2.7737921476364136e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 69 -1.4599669724702835e-02</internalNodes>
          <leafValues>
            -7.0966827869415283e-01 9.7515560686588287e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 70 1.7236070707440376e-02</internalNodes>
          <leafValues>
            1.6869539394974709e-02 -5.7388329505920410e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 71 1.4230710454285145e-02</internalNodes>
          <leafValues>
            9.4714500010013580e-02 -7.8395259380340576e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 72 -4.3706860393285751e-02</internalNodes>
          <leafValues>
            6.0979652404785156e-01 -1.5601889789104462e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 73 -6.2343222089111805e-04</internalNodes>
          <leafValues>
            3.4851190447807312e-01 -2.1704910695552826e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 74 1.9245050847530365e-02</internalNodes>
          <leafValues>
            -1.1710979789495468e-01 3.0701160430908203e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 75 2.7035778760910034e-01</internalNodes>
          <leafValues>
            -9.0096436440944672e-02 7.6656961441040039e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 76 -3.5394480801187456e-04</internalNodes>
          <leafValues>
            -2.0024789869785309e-01 1.2493360042572021e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 77 -3.6013960838317871e-02</internalNodes>
          <leafValues>
            6.7028558254241943e-01 -1.0571879893541336e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 78 9.2952791601419449e-03</internalNodes>
          <leafValues>
            -1.0574710369110107e-01 4.5093879103660583e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 79 -3.3304709359072149e-04</internalNodes>
          <leafValues>
            2.7933821082115173e-01 -2.4576769769191742e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 80 -2.9147620807634667e-05</internalNodes>
          <leafValues>
            8.5813812911510468e-02 -9.5469586551189423e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 81 4.4382669148035347e-04</internalNodes>
          <leafValues>
            -2.0220080018043518e-01 5.4543578624725342e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>23</maxWeakCount>
      <stageThreshold>-1.3707510232925415e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 82 7.9610757529735565e-03</internalNodes>
          <leafValues>
            -3.6722078919410706e-01 4.3154349923133850e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 83 6.3394829630851746e-02</internalNodes>
          <leafValues>
            -2.0739710330963135e-01 5.7426017522811890e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 84 -5.3193349391222000e-02</internalNodes>
          <leafValues>
            7.2550922632217407e-01 -1.4342020452022552e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 85 1.5460769645869732e-02</internalNodes>
          <leafValues>
            -9.6053816378116608e-02 7.5785237550735474e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 86 -1.7643140628933907e-02</internalNodes>
          <leafValues>
            6.6815620660781860e-01 -1.4176729321479797e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 87 9.5065636560320854e-03</internalNodes>
          <leafValues>
            -9.6259742975234985e-02 4.6996331214904785e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 88 4.0446049533784389e-03</internalNodes>
          <leafValues>
            -1.9732519984245300e-01 4.2838010191917419e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 89 3.2312041148543358e-03</internalNodes>
          <leafValues>
            1.1861690133810043e-01 -6.1039632558822632e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 90 -4.0159050375223160e-02</internalNodes>
          <leafValues>
            -4.1664341092109680e-01 2.1672329306602478e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 91 2.8524258732795715e-01</internalNodes>
          <leafValues>
            -1.0435750335454941e-01 8.5733968019485474e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 92 -4.9264221452176571e-03</internalNodes>
          <leafValues>
            4.7060468792915344e-01 -1.3997459411621094e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 93 1.3781700283288956e-02</internalNodes>
          <leafValues>
            -1.2713569402694702e-01 4.4618919491767883e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 94 -4.9873598618432879e-04</internalNodes>
          <leafValues>
            4.7026631236076355e-01 -1.5483739972114563e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 95 -1.5621389320585877e-04</internalNodes>
          <leafValues>
            1.8854810297489166e-01 -7.7839776873588562e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 96 -3.7597760092467070e-04</internalNodes>
          <leafValues>
            5.7697701454162598e-01 -1.3356220722198486e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 97 -1.0665910318493843e-02</internalNodes>
          <leafValues>
            -4.1065299510955811e-01 1.5562120079994202e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 98 -3.4135230816900730e-03</internalNodes>
          <leafValues>
            -7.6363432407379150e-01 1.0209649801254272e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 99 5.6471868447260931e-05</internalNodes>
          <leafValues>
            -1.6443930566310883e-01 2.2908419370651245e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 100 2.1611599368043244e-04</internalNodes>
          <leafValues>
            -1.6290329396724701e-01 4.5756360888481140e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 101 -1.0822719894349575e-02</internalNodes>
          <leafValues>
            -2.4462530016899109e-01 1.3888940215110779e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 102 -1.5084910206496716e-02</internalNodes>
          <leafValues>
            -5.7813477516174316e-01 1.1564119905233383e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 103 2.5715960189700127e-02</internalNodes>
          <leafValues>
            3.9631199091672897e-02 -6.5270012617111206e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 104 2.6093570049852133e-03</internalNodes>
          <leafValues>
            1.1421889811754227e-01 -5.6801080703735352e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>26</maxWeakCount>
      <stageThreshold>-1.3303329944610596e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 105 -5.1861900836229324e-02</internalNodes>
          <leafValues>
            7.0431172847747803e-01 -2.2143700718879700e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 106 -5.0341628491878510e-02</internalNodes>
          <leafValues>
            -4.6397829055786133e-01 2.8047460317611694e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 107 2.5709730386734009e-01</internalNodes>
          <leafValues>
            -1.3124279677867889e-01 8.2395941019058228e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 108 1.1031899601221085e-02</internalNodes>
          <leafValues>
            -1.4258140325546265e-01 6.3823902606964111e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 109 1.8565090373158455e-02</internalNodes>
          <leafValues>
            -1.5123879909515381e-01 5.9881192445755005e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 110 1.7502350732684135e-02</internalNodes>
          <leafValues>
            -1.2619799375534058e-01 3.8178038597106934e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 111 7.2723729535937309e-03</internalNodes>
          <leafValues>
            -1.5103289484977722e-01 5.8128422498703003e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 112 8.1504750996828079e-03</internalNodes>
          <leafValues>
            -6.5464757382869720e-02 5.6397551298141479e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 113 -1.8552739173173904e-02</internalNodes>
          <leafValues>
            5.3157097101211548e-01 -1.2526570260524750e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 114 -2.3101480677723885e-02</internalNodes>
          <leafValues>
            -6.7949390411376953e-01 1.1046259850263596e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 115 -1.8539339362177998e-04</internalNodes>
          <leafValues>
            3.0100038647651672e-01 -2.1206699311733246e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 116 1.7319120466709137e-02</internalNodes>
          <leafValues>
            -9.3738131225109100e-02 2.1008560061454773e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 117 1.4305620454251766e-02</internalNodes>
          <leafValues>
            1.8005949258804321e-01 -3.9776718616485596e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 118 2.5763340294361115e-02</internalNodes>
          <leafValues>
            8.7056998163461685e-03 -6.2894952297210693e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 119 -1.5383340418338776e-02</internalNodes>
          <leafValues>
            -5.3415471315383911e-01 1.0380730032920837e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 120 1.0605469578877091e-03</internalNodes>
          <leafValues>
            -9.0128518640995026e-02 1.6792120039463043e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 121 3.5230729263275862e-03</internalNodes>
          <leafValues>
            -1.7110690474510193e-01 3.2596540451049805e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 122 -1.0789279825985432e-02</internalNodes>
          <leafValues>
            3.6109921336174011e-01 -6.6339150071144104e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 123 2.7950939536094666e-01</internalNodes>
          <leafValues>
            -7.4605897068977356e-02 7.3369878530502319e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 124 3.8369540125131607e-03</internalNodes>
          <leafValues>
            4.4873539358377457e-02 -1.8602700531482697e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 125 1.6195949865505099e-03</internalNodes>
          <leafValues>
            -1.3922490179538727e-01 4.3437001109123230e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 126 1.1647949926555157e-02</internalNodes>
          <leafValues>
            -7.4357591569423676e-02 5.4201442003250122e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 127 -5.9066400863230228e-03</internalNodes>
          <leafValues>
            -7.0557588338851929e-01 8.6433619260787964e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 128 3.9686840772628784e-01</internalNodes>
          <leafValues>
            -7.4898369610309601e-02 9.4062858819961548e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 129 5.7663779705762863e-02</internalNodes>
          <leafValues>
            -9.6558406949043274e-02 5.4182428121566772e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 130 6.0319568961858749e-02</internalNodes>
          <leafValues>
            -6.6501073539257050e-02 6.4023548364639282e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>37</maxWeakCount>
      <stageThreshold>-1.5300060510635376e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 131 1.9050249829888344e-02</internalNodes>
          <leafValues>
            -4.4433408975601196e-01 4.3948569893836975e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 132 -2.0198300480842590e-02</internalNodes>
          <leafValues>
            -3.1706219911575317e-01 1.0432930290699005e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 133 2.1478030830621719e-02</internalNodes>
          <leafValues>
            -3.5024839639663696e-01 2.6355370879173279e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 134 -1.0187759995460510e-01</internalNodes>
          <leafValues>
            -5.9889578819274902e-01 1.7685799300670624e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 135 1.0974160395562649e-02</internalNodes>
          <leafValues>
            -1.4895239472389221e-01 6.0115218162536621e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 136 -1.1476710438728333e-02</internalNodes>
          <leafValues>
            4.0665709972381592e-01 -1.2404689937829971e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 137 -2.3431150242686272e-02</internalNodes>
          <leafValues>
            -7.1487832069396973e-01 1.4278119802474976e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 138 1.4963559806346893e-03</internalNodes>
          <leafValues>
            -1.7045859992504120e-01 1.7193080484867096e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 139 -5.4855772759765387e-04</internalNodes>
          <leafValues>
            3.1553238630294800e-01 -2.1444450318813324e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 140 7.4912630021572113e-02</internalNodes>
          <leafValues>
            9.1240562498569489e-02 -6.3951212167739868e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 141 6.8816398270428181e-03</internalNodes>
          <leafValues>
            -1.4904409646987915e-01 4.7952368855476379e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 142 -3.8212578743696213e-02</internalNodes>
          <leafValues>
            5.2887737751007080e-01 -6.1894729733467102e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 143 4.4051730073988438e-03</internalNodes>
          <leafValues>
            -1.1934129893779755e-01 5.0613421201705933e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 144 2.3966899141669273e-02</internalNodes>
          <leafValues>
            -8.9720509946346283e-02 3.3152779936790466e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 145 -3.4162990748882294e-02</internalNodes>
          <leafValues>
            5.3134781122207642e-01 -1.4666500687599182e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 146 1.9642219413071871e-03</internalNodes>
          <leafValues>
            9.0783588588237762e-02 -4.3032559752464294e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 147 9.6757910796441138e-05</internalNodes>
          <leafValues>
            2.2552539408206940e-01 -2.8220710158348083e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 148 -3.2862399239093065e-03</internalNodes>
          <leafValues>
            4.0515020489692688e-01 -1.1776199936866760e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 149 1.1688309721648693e-02</internalNodes>
          <leafValues>
            -9.1857127845287323e-02 6.2834888696670532e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 150 -6.0287420637905598e-03</internalNodes>
          <leafValues>
            3.9261808991432190e-01 -1.2287150323390961e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 151 -1.3721340335905552e-02</internalNodes>
          <leafValues>
            -5.5298799276351929e-01 9.1041281819343567e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 152 7.5626641511917114e-02</internalNodes>
          <leafValues>
            -4.4929590076208115e-02 1.7442759871482849e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 153 9.3434482812881470e-02</internalNodes>
          <leafValues>
            -8.4593951702117920e-02 6.0131162405014038e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 154 5.8748829178512096e-03</internalNodes>
          <leafValues>
            -4.4131498783826828e-02 3.9565709233283997e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 155 4.0064537897706032e-03</internalNodes>
          <leafValues>
            -1.1414399743080139e-01 3.7925380468368530e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 156 2.2945459932088852e-02</internalNodes>
          <leafValues>
            2.4673189967870712e-02 -4.1521999239921570e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 157 -1.2810460291802883e-02</internalNodes>
          <leafValues>
            -5.1557427644729614e-01 9.1319613158702850e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 158 2.0425529778003693e-01</internalNodes>
          <leafValues>
            -6.5927542746067047e-02 7.5942492485046387e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 159 4.9796327948570251e-03</internalNodes>
          <leafValues>
            1.0806279629468918e-01 -5.0016272068023682e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 160 2.8397630900144577e-02</internalNodes>
          <leafValues>
            -3.7152960896492004e-02 5.4010647535324097e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 161 6.0867150314152241e-03</internalNodes>
          <leafValues>
            -1.1978609859943390e-01 3.5692268610000610e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 162 -2.1456899412441999e-04</internalNodes>
          <leafValues>
            1.8740150332450867e-01 -8.8417202234268188e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 163 2.8941858909092844e-04</internalNodes>
          <leafValues>
            -1.2597979605197906e-01 3.9982271194458008e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 164 -1.3047619722783566e-03</internalNodes>
          <leafValues>
            1.5499970316886902e-01 -7.5386047363281250e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 165 -1.2975010089576244e-02</internalNodes>
          <leafValues>
            -5.5344110727310181e-01 8.2354247570037842e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 166 7.7442410401999950e-03</internalNodes>
          <leafValues>
            2.7699800208210945e-02 -3.4835991263389587e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 167 2.4850629270076752e-03</internalNodes>
          <leafValues>
            -1.2976129353046417e-01 3.7908831238746643e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>21</maxWeakCount>
      <stageThreshold>-1.4114329814910889e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 168 -4.0386881679296494e-02</internalNodes>
          <leafValues>
            5.9603548049926758e-01 -3.5741761326789856e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 169 -6.6068649175576866e-05</internalNodes>
          <leafValues>
            4.4628980755805969e-01 -3.5959470272064209e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 170 3.7622239906340837e-03</internalNodes>
          <leafValues>
            1.7947019636631012e-01 -7.5631511211395264e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 171 -3.0967719852924347e-02</internalNodes>
          <leafValues>
            -2.8847050666809082e-01 7.6870530843734741e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 172 3.0566560104489326e-02</internalNodes>
          <leafValues>
            1.4003600180149078e-01 -7.1755367517471313e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 173 9.9054910242557526e-04</internalNodes>
          <leafValues>
            8.2915589213371277e-02 -2.9197171330451965e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 174 1.2577700428664684e-02</internalNodes>
          <leafValues>
            1.5380719304084778e-01 -4.6882930397987366e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 175 1.2392920255661011e-01</internalNodes>
          <leafValues>
            -9.0823858976364136e-02 7.3837572336196899e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 176 3.7737488746643066e-01</internalNodes>
          <leafValues>
            -5.4232951253652573e-02 9.2291218042373657e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 177 1.0996370017528534e-01</internalNodes>
          <leafValues>
            9.1596268117427826e-02 -6.5977168083190918e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 178 -1.2721329694613814e-03</internalNodes>
          <leafValues>
            3.3475750684738159e-01 -1.8290689587593079e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 179 4.6906251460313797e-02</internalNodes>
          <leafValues>
            -8.3971053361892700e-02 6.9847589731216431e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 180 3.2869930146262050e-04</internalNodes>
          <leafValues>
            1.8794630467891693e-01 -2.9290059208869934e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 181 1.7333080177195370e-04</internalNodes>
          <leafValues>
            -2.6964160799980164e-01 3.4947571158409119e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 182 1.9800959154963493e-02</internalNodes>
          <leafValues>
            -1.4679229259490967e-01 4.3995618820190430e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 183 2.0056760695297271e-04</internalNodes>
          <leafValues>
            -1.3727410137653351e-01 2.2213310003280640e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 184 -1.4923149719834328e-03</internalNodes>
          <leafValues>
            3.4735259413719177e-01 -1.5948210656642914e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 185 -4.2736999603221193e-05</internalNodes>
          <leafValues>
            3.1527870893478394e-01 -2.3066949844360352e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 186 6.6625140607357025e-04</internalNodes>
          <leafValues>
            -2.0131100714206696e-01 2.8691890835762024e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 187 1.3850460163666867e-05</internalNodes>
          <leafValues>
            -2.0219239592552185e-01 2.3073309659957886e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 188 4.0972631424665451e-02</internalNodes>
          <leafValues>
            7.9543180763721466e-02 -8.0795639753341675e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>23</maxWeakCount>
      <stageThreshold>-1.3777890205383301e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 189 -4.6982929110527039e-02</internalNodes>
          <leafValues>
            7.0822530984878540e-01 -3.7034240365028381e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 190 -7.5753079727292061e-04</internalNodes>
          <leafValues>
            -1.2550309300422668e-01 1.3944420218467712e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 191 1.5327299945056438e-02</internalNodes>
          <leafValues>
            2.1613539755344391e-01 -5.6293952465057373e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 192 1.8147040158510208e-02</internalNodes>
          <leafValues>
            -3.2079648226499557e-02 3.2347559928894043e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 193 4.7347191721200943e-02</internalNodes>
          <leafValues>
            -1.7381580173969269e-01 5.7580447196960449e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 194 -5.9837941080331802e-02</internalNodes>
          <leafValues>
            4.7797870635986328e-01 -1.0260280221700668e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 195 -5.2796799689531326e-02</internalNodes>
          <leafValues>
            -4.7988489270210266e-01 1.8787759542465210e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 196 -2.4385429918766022e-02</internalNodes>
          <leafValues>
            -3.0841669440269470e-01 8.7605630978941917e-03</leafValues></_>
        <_>
          <internalNodes>
            0 -1 197 2.5288300588726997e-02</internalNodes>
          <leafValues>
            1.3914039731025696e-01 -7.1094942092895508e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 198 -2.1612450480461121e-02</internalNodes>
          <leafValues>
            -2.3282539844512939e-01 8.0994680523872375e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 199 3.4023479092866182e-03</internalNodes>
          <leafValues>
            -2.2989900410175323e-01 3.7889510393142700e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 200 1.1274600028991699e-01</internalNodes>
          <leafValues>
            -1.5474709682166576e-02 5.7030540704727173e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 201 3.4516870975494385e-02</internalNodes>
          <leafValues>
            -1.2300080060958862e-01 5.6775367259979248e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 202 7.8984811902046204e-02</internalNodes>
          <leafValues>
            -1.4242169260978699e-01 4.6941858530044556e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 203 -1.5377859584987164e-02</internalNodes>
          <leafValues>
            6.3946861028671265e-01 -1.1236190050840378e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 204 -2.2373620595317334e-04</internalNodes>
          <leafValues>
            5.5583298206329346e-01 -2.7247580885887146e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 205 -2.4762390181422234e-02</internalNodes>
          <leafValues>
            -5.0404858589172363e-01 1.4077790081501007e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 206 -9.4061157142277807e-05</internalNodes>
          <leafValues>
            3.7195280194282532e-01 -2.2502990067005157e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 207 -2.0256359130144119e-02</internalNodes>
          <leafValues>
            5.1051008701324463e-01 -1.4298759400844574e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 208 4.8122879117727280e-02</internalNodes>
          <leafValues>
            -6.6979512572288513e-02 3.6622309684753418e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 209 -2.3787800222635269e-02</internalNodes>
          <leafValues>
            5.0813251733779907e-01 -1.2908150255680084e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 210 -1.0520319920033216e-03</internalNodes>
          <leafValues>
            -1.5604670345783234e-01 6.6213317215442657e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 211 -2.6640200521796942e-03</internalNodes>
          <leafValues>
            -7.2545582056045532e-01 8.2365453243255615e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>25</maxWeakCount>
      <stageThreshold>-1.3266400098800659e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 212 -5.0224620848894119e-02</internalNodes>
          <leafValues>
            7.0845657587051392e-01 -2.5585499405860901e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 213 1.4072869904339314e-02</internalNodes>
          <leafValues>
            6.3033178448677063e-02 -5.9838529676198959e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 214 1.7804009839892387e-02</internalNodes>
          <leafValues>
            1.9414719939231873e-01 -5.8444267511367798e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 215 1.3046739995479584e-01</internalNodes>
          <leafValues>
            -1.1516980081796646e-01 8.5040301084518433e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 216 1.7506800591945648e-02</internalNodes>
          <leafValues>
            -2.0718969404697418e-01 4.6438288688659668e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 217 -7.4240020476281643e-03</internalNodes>
          <leafValues>
            -6.6565167903900146e-01 1.4034989476203918e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 218 -3.4571118652820587e-02</internalNodes>
          <leafValues>
            6.5112978219985962e-01 -1.4901919662952423e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 219 4.2270249687135220e-03</internalNodes>
          <leafValues>
            -1.6027219826355577e-03 3.8956061005592346e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 220 -5.0662040710449219e-02</internalNodes>
          <leafValues>
            5.8035767078399658e-01 -1.5141439437866211e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 221 -7.0715770125389099e-03</internalNodes>
          <leafValues>
            5.3008967638015747e-01 -1.4498309791088104e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 222 -1.1863510124385357e-02</internalNodes>
          <leafValues>
            6.7297422885894775e-01 -1.1063549667596817e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 223 -6.0520030558109283e-02</internalNodes>
          <leafValues>
            -3.3164489269256592e-01 2.1195560693740845e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 224 -7.7340779826045036e-03</internalNodes>
          <leafValues>
            -6.9414401054382324e-01 7.2705313563346863e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 225 -3.2486140727996826e-02</internalNodes>
          <leafValues>
            -5.1850819587707520e-01 5.9212621301412582e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 226 8.3279706537723541e-02</internalNodes>
          <leafValues>
            1.2067940086126328e-01 -5.3095632791519165e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 227 7.8782817581668496e-04</internalNodes>
          <leafValues>
            -2.7376559376716614e-01 2.7162519097328186e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 228 -1.7539180815219879e-02</internalNodes>
          <leafValues>
            -5.6902301311492920e-01 1.2287370115518570e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 229 -5.8226347900927067e-03</internalNodes>
          <leafValues>
            4.3865859508514404e-01 -1.4937420189380646e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 230 -1.0057560168206692e-02</internalNodes>
          <leafValues>
            -6.6168862581253052e-01 1.1445429921150208e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 231 9.0345427393913269e-02</internalNodes>
          <leafValues>
            -6.6665247082710266e-02 2.8706479072570801e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 232 -6.7587293684
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

