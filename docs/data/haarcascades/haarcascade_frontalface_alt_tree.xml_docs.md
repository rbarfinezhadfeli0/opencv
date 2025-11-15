# Documentation for `data/haarcascades/haarcascade_frontalface_alt_tree.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_frontalface_alt_tree.xml`
- **File Name**: `haarcascade_frontalface_alt_tree.xml`
- **File Size**: 2,689,040 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_frontalface_alt_tree.xml](../../data/haarcascades/haarcascade_frontalface_alt_tree.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

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
<cascade type_id="opencv-cascade-classifier"><stageType>BOOST</stageType>
  <featureType>HAAR</featureType>
  <height>20</height>
  <width>20</width>
  <stageParams>
    <maxWeakCount>406</maxWeakCount></stageParams>
  <featureParams>
    <maxCatCount>0</maxCatCount></featureParams>
  <stageNum>47</stageNum>
  <stages>
    <_>
      <maxWeakCount>3</maxWeakCount>
      <stageThreshold>-1.3442519903182983e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 0 3.7895569112151861e-03</internalNodes>
          <leafValues>
            -9.2945802211761475e-01 6.4119851589202881e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1 1.2098110280930996e-02</internalNodes>
          <leafValues>
            -7.1810090541839600e-01 4.7141009569168091e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 2 1.2138449819758534e-03</internalNodes>
          <leafValues>
            -7.2831612825393677e-01 3.0330690741539001e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>9</maxWeakCount>
      <stageThreshold>-1.6378560066223145e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 3 8.7510552257299423e-03</internalNodes>
          <leafValues>
            -8.5947072505950928e-01 3.6881381273269653e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 4 2.1986700594425201e-02</internalNodes>
          <leafValues>
            -6.0180151462554932e-01 3.2897830009460449e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 5 6.4913398819044232e-04</internalNodes>
          <leafValues>
            -7.9431951045989990e-01 2.5493299961090088e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 6 -1.0192029876634479e-03</internalNodes>
          <leafValues>
            2.2729329764842987e-01 -6.3627982139587402e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 7 1.3674780493602157e-03</internalNodes>
          <leafValues>
            -6.0014182329177856e-01 2.4118369817733765e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 8 1.0245250305160880e-03</internalNodes>
          <leafValues>
            -5.8542472124099731e-01 1.2550109624862671e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 9 1.8465859815478325e-02</internalNodes>
          <leafValues>
            1.9563560187816620e-01 -6.7630231380462646e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 10 4.0901508182287216e-03</internalNodes>
          <leafValues>
            -4.4916498661041260e-01 2.6677688956260681e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 11 1.1358099989593029e-02</internalNodes>
          <leafValues>
            1.8783229589462280e-01 -6.1379361152648926e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>16</maxWeakCount>
      <stageThreshold>-1.7317579984664917e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 12 -1.1588949710130692e-02</internalNodes>
          <leafValues>
            3.4567040205001831e-01 -7.6478981971740723e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 13 5.1809530705213547e-03</internalNodes>
          <leafValues>
            2.4104920029640198e-01 -6.9623559713363647e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 14 2.1468549966812134e-03</internalNodes>
          <leafValues>
            -8.0553662776947021e-01 1.9838610291481018e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 15 -3.6556499544531107e-03</internalNodes>
          <leafValues>
            -7.1833139657974243e-01 1.2305679917335510e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 16 -1.9701640121638775e-03</internalNodes>
          <leafValues>
            2.2777689993381500e-01 -4.7520169615745544e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 17 -3.3645539078861475e-03</internalNodes>
          <leafValues>
            -4.6095049381256104e-01 2.0394650101661682e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 18 -7.4126059189438820e-05</internalNodes>
          <leafValues>
            1.8213239312171936e-01 -4.7829270362854004e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 19 -1.7571110278367996e-02</internalNodes>
          <leafValues>
            -7.1737551689147949e-01 1.1311130225658417e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 20 6.3840472139418125e-03</internalNodes>
          <leafValues>
            -4.0205681324005127e-01 2.0730289816856384e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 21 -1.4723399654030800e-02</internalNodes>
          <leafValues>
            -6.7558771371841431e-01 6.8973086774349213e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 22 -5.2889222279191017e-03</internalNodes>
          <leafValues>
            -6.2105172872543335e-01 1.3349360227584839e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 23 2.7743630111217499e-02</internalNodes>
          <leafValues>
            1.1760850250720978e-01 -5.4641121625900269e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 24 3.9427559822797775e-02</internalNodes>
          <leafValues>
            -2.1134279668331146e-01 3.9452999830245972e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 25 8.6949411779642105e-03</internalNodes>
          <leafValues>
            1.2580950558185577e-01 -4.7989100217819214e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 26 2.8245279099792242e-03</internalNodes>
          <leafValues>
            1.9653140008449554e-01 -4.0256679058074951e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 27 -2.8915189206600189e-02</internalNodes>
          <leafValues>
            -8.0616527795791626e-01 8.1882260739803314e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>29</maxWeakCount>
      <stageThreshold>-1.9308480024337769e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 28 8.0171944573521614e-03</internalNodes>
          <leafValues>
            -6.8981552124023438e-01 2.4136860668659210e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 29 -2.4478728882968426e-03</internalNodes>
          <leafValues>
            2.1353200078010559e-01 -6.4146691560745239e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 30 1.7917619552463293e-03</internalNodes>
          <leafValues>
            -6.1445468664169312e-01 1.9236929714679718e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 31 4.3905500206165016e-04</internalNodes>
          <leafValues>
            -7.5360429286956787e-01 1.5696890652179718e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 32 -3.6769549478776753e-04</internalNodes>
          <leafValues>
            1.7380510270595551e-01 -5.8404499292373657e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 33 -4.2802388779819012e-03</internalNodes>
          <leafValues>
            -6.6968989372253418e-01 1.1289729923009872e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 34 3.5238768905401230e-03</internalNodes>
          <leafValues>
            1.2501940131187439e-01 -7.3299217224121094e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 35 7.9299701610580087e-04</internalNodes>
          <leafValues>
            -4.4966199994087219e-01 2.1590930223464966e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 36 4.4371088733896613e-04</internalNodes>
          <leafValues>
            -3.8909769058227539e-01 2.1181149780750275e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 37 -2.7145470958203077e-03</internalNodes>
          <leafValues>
            -4.6716868877410889e-01 1.5038399398326874e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 38 -6.9272058317437768e-04</internalNodes>
          <leafValues>
            -5.8596551418304443e-01 1.1714380234479904e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 39 4.9261808395385742e-02</internalNodes>
          <leafValues>
            -1.3800150156021118e-01 4.9366238713264465e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 40 -2.2837519645690918e-02</internalNodes>
          <leafValues>
            -6.3743507862091064e-01 1.2324090301990509e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 41 4.8372112214565277e-03</internalNodes>
          <leafValues>
            -1.2391629815101624e-01 1.0620889812707901e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 42 1.0256259702146053e-02</internalNodes>
          <leafValues>
            -1.8767049908638000e-01 2.9824170470237732e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 43 1.0618680156767368e-02</internalNodes>
          <leafValues>
            1.0612460225820541e-01 -3.3244881033897400e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 44 2.4113139137625694e-02</internalNodes>
          <leafValues>
            8.7200611829757690e-02 -6.6846621036529541e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 45 -3.6754710599780083e-03</internalNodes>
          <leafValues>
            1.1043280363082886e-01 -4.4581958651542664e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 46 -3.8996201008558273e-02</internalNodes>
          <leafValues>
            -7.0228111743927002e-01 8.1809490919113159e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 47 1.5777100343257189e-03</internalNodes>
          <leafValues>
            1.5954199433326721e-01 -3.2860770821571350e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 48 9.1089410707354546e-03</internalNodes>
          <leafValues>
            1.0326369851827621e-01 -4.4402560591697693e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 49 -1.7051609233021736e-02</internalNodes>
          <leafValues>
            -5.5853348970413208e-01 6.2711499631404877e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 50 1.3652660418301821e-03</internalNodes>
          <leafValues>
            -5.3934460878372192e-01 7.0839896798133850e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 51 -1.1186149902641773e-02</internalNodes>
          <leafValues>
            -4.7260180115699768e-01 8.1019416451454163e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 52 -1.1705270037055016e-02</internalNodes>
          <leafValues>
            2.4750089645385742e-01 -1.7778989672660828e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 53 -9.7736932337284088e-02</internalNodes>
          <leafValues>
            -5.6177508831024170e-01 8.0921821296215057e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 54 -8.5228063166141510e-02</internalNodes>
          <leafValues>
            -5.2233248949050903e-01 7.2821393609046936e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 55 -3.6733459681272507e-02</internalNodes>
          <leafValues>
            4.3623578548431396e-01 -9.9339507520198822e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 56 -3.6704430822283030e-03</internalNodes>
          <leafValues>
            1.4834220707416534e-01 -2.7119669318199158e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>36</maxWeakCount>
      <stageThreshold>-2.0711259841918945e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 57 -1.1610370129346848e-03</internalNodes>
          <leafValues>
            -5.6377887725830078e-01 2.3568780720233917e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 58 1.1830299627035856e-03</internalNodes>
          <leafValues>
            1.5724280476570129e-01 -6.7728179693222046e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 59 -2.1273950114846230e-03</internalNodes>
          <leafValues>
            -6.6150152683258057e-01 1.4943139255046844e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 60 -1.1893469840288162e-01</internalNodes>
          <leafValues>
            5.3225821256637573e-01 -2.2968369722366333e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 61 -1.3624870218336582e-02</internalNodes>
          <leafValues>
            -6.0635501146316528e-01 1.7001089453697205e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 62 -6.3198682619258761e-04</internalNodes>
          <leafValues>
            -6.8972241878509521e-01 1.1584629863500595e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 63 -4.4108428992331028e-03</internalNodes>
          <leafValues>
            -6.2967002391815186e-01 1.2430600076913834e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 64 -2.2982239723205566e-02</internalNodes>
          <leafValues>
            -5.0497251749038696e-01 1.6636120155453682e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 65 -2.3721898905932903e-03</internalNodes>
          <leafValues>
            -6.2462240457534790e-01 1.3793750107288361e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 66 8.7364763021469116e-03</internalNodes>
          <leafValues>
            1.3996620476245880e-01 -5.4822951555252075e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 67 6.7737072706222534e-02</internalNodes>
          <leafValues>
            -1.9172480702400208e-01 5.4700487852096558e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 68 -4.0138149634003639e-03</internalNodes>
          <leafValues>
            -5.5429118871688843e-01 1.4517059922218323e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 69 1.2857170077040792e-04</internalNodes>
          <leafValues>
            -5.1031237840652466e-01 1.1023940145969391e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 70 -3.9688948541879654e-02</internalNodes>
          <leafValues>
            -6.1830729246139526e-01 9.6676096320152283e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 71 -1.6646150033921003e-03</internalNodes>
          <leafValues>
            1.6449889540672302e-01 -3.7186318635940552e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 72 5.3499247878789902e-03</internalNodes>
          <leafValues>
            1.1145050078630447e-01 -3.7441021203994751e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 73 -2.2904010489583015e-02</internalNodes>
          <leafValues>
            -5.8097589015960693e-01 1.1077260226011276e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 74 1.0703450068831444e-02</internalNodes>
          <leafValues>
            4.4733259826898575e-02 -5.8116632699966431e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 75 -4.2331559234298766e-04</internalNodes>
          <leafValues>
            -5.4423791170120239e-01 8.7089292705059052e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 76 1.5554429963231087e-02</internalNodes>
          <leafValues>
            5.6884340941905975e-02 -3.7645170092582703e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 77 -2.0539449527859688e-02</internalNodes>
          <leafValues>
            -3.8714569807052612e-01 1.1833839863538742e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 78 -3.1234358903020620e-03</internalNodes>
          <leafValues>
            8.3635427057743073e-02 -1.9862389564514160e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 79 2.3932829499244690e-02</internalNodes>
          <leafValues>
            7.9600542783737183e-02 -6.5370100736618042e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 80 8.3920456469058990e-02</internalNodes>
          <leafValues>
            -1.0653129965066910e-01 4.8772820830345154e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 81 1.6003159806132317e-02</internalNodes>
          <leafValues>
            8.3643212914466858e-02 -5.9207731485366821e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 82 5.8071441017091274e-03</internalNodes>
          <leafValues>
            8.7997503578662872e-02 -3.3279138803482056e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 83 -8.1104427576065063e-02</internalNodes>
          <leafValues>
            6.3775187730789185e-01 -6.7692361772060394e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 84 4.5403029769659042e-02</internalNodes>
          <leafValues>
            -5.1510389894247055e-02 3.0225670337677002e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 85 1.3877229765057564e-02</internalNodes>
          <leafValues>
            9.9967628717422485e-02 -4.6520909667015076e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 86 3.4590709954500198e-02</internalNodes>
          <leafValues>
            -9.7614437341690063e-02 3.4678751230239868e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 87 1.5704549849033356e-02</internalNodes>
          <leafValues>
            7.6344117522239685e-02 -5.3356319665908813e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 88 -1.0420549660921097e-01</internalNodes>
          <leafValues>
            6.1890971660614014e-01 -4.4259760528802872e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 89 1.3443189859390259e-01</internalNodes>
          <leafValues>
            -5.9853021055459976e-02 6.3635712862014771e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 90 -2.5646309368312359e-03</internalNodes>
          <leafValues>
            -5.3600472211837769e-01 7.3116026818752289e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 91 1.8647089600563049e-02</internalNodes>
          <leafValues>
            6.9856151938438416e-02 -5.6878322362899780e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 92 1.5159539878368378e-02</internalNodes>
          <leafValues>
            1.8206339329481125e-02 -2.7663159370422363e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>7</maxWeakCount>
      <stageThreshold>-2.1360809803009033e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 93 1.4778429269790649e-01</internalNodes>
          <leafValues>
            -8.9933121204376221e-01 5.7035928964614868e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 94 2.9984670877456665e-01</internalNodes>
          <leafValues>
            -6.5394151210784912e-01 3.5054451227188110e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 95 -7.9061716794967651e-02</internalNodes>
          <leafValues>
            4.4085291028022766e-01 -6.5087568759918213e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 96 5.8428961783647537e-02</internalNodes>
          <leafValues>
            -4.2665359377861023e-01 5.8410567045211792e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 97 -1.4664280228316784e-02</internalNodes>
          <leafValues>
            3.2435241341590881e-01 -5.9659618139266968e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 98 3.9517199993133545e-01</internalNodes>
          <leafValues>
            -7.5798347592353821e-02 4.8659950494766235e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 99 1.1040589958429337e-01</internalNodes>
          <leafValues>
            -8.4556102752685547e-01 2.1374569833278656e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>50</maxWeakCount>
      <stageThreshold>-1.8755869865417480e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 100 3.7777079269289970e-03</internalNodes>
          <leafValues>
            1.8744400143623352e-01 -6.5354061126708984e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 101 5.3003188222646713e-03</internalNodes>
          <leafValues>
            9.3951843678951263e-02 -5.6917887926101685e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 102 -5.5426009930670261e-03</internalNodes>
          <leafValues>
            1.6031709313392639e-01 -5.1822239160537720e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 103 -9.1971885412931442e-03</internalNodes>
          <leafValues>
            -5.7420462369918823e-01 1.4791400730609894e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 104 5.3701602155342698e-04</internalNodes>
          <leafValues>
            -7.0449697971343994e-01 1.0752149671316147e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 105 -2.2125479299575090e-03</internalNodes>
          <leafValues>
            -5.0877428054809570e-01 1.1367189884185791e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 106 1.1675730347633362e-02</internalNodes>
          <leafValues>
            8.4258683025836945e-02 -6.7384701967239380e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 107 -2.0404369570314884e-03</internalNodes>
          <leafValues>
            1.6251119971275330e-01 -4.1435649991035461e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 108 -7.6540438458323479e-03</internalNodes>
          <leafValues>
            -4.2833179235458374e-01 1.3060709834098816e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 109 2.9370479285717010e-02</internalNodes>
          <leafValues>
            5.4651051759719849e-02 -3.4795379638671875e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 110 -9.5828901976346970e-03</internalNodes>
          <leafValues>
            -4.8620718717575073e-01 1.1706890165805817e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 111 6.0666278004646301e-03</internalNodes>
          <leafValues>
            -3.6553880572319031e-01 8.7813600897789001e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 112 1.7992249922826886e-03</internalNodes>
          <leafValues>
            1.6035990417003632e-01 -3.0859109759330750e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 113 -1.0092309676110744e-02</internalNodes>
          <leafValues>
            -3.9505869150161743e-01 1.1514779925346375e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 114 2.5171819142997265e-03</internalNodes>
          <leafValues>
            -3.0043110251426697e-01 1.8256050348281860e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 115 -1.7089240252971649e-02</internalNodes>
          <leafValues>
            -5.2173590660095215e-01 9.7457267343997955e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 116 -5.5856268852949142e-02</internalNodes>
          <leafValues>
            5.3540021181106567e-01 -8.9221552014350891e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 117 -2.3930610623210669e-03</internalNodes>
          <leafValues>
            -4.7012439370155334e-01 8.6141407489776611e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 118 3.6918919067829847e-03</internalNodes>
          <leafValues>
            -2.7755591273307800e-01 1.5186099708080292e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 119 2.1945969201624393e-03</internalNodes>
          <leafValues>
            -1.6867069900035858e-01 1.1952520161867142e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 120 2.9675459954887629e-03</internalNodes>
          <leafValues>
            -3.8940680027008057e-01 1.0388910025358200e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 121 1.9976729527115822e-03</internalNodes>
          <leafValues>
            9.1141343116760254e-02 -4.1050049662590027e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 122 -2.0369699224829674e-02</internalNodes>
          <leafValues>
            -5.9968769550323486e-01 6.9301806390285492e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 123 2.3318571038544178e-03</internalNodes>
          <leafValues>
            6.1892550438642502e-02 -3.2886800169944763e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 124 -4.2863588780164719e-02</internalNodes>
          <leafValues>
            -7.3844969272613525e-01 5.7071659713983536e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 125 1.1471749749034643e-03</internalNodes>
          <leafValues>
            -5.1379621028900146e-01 7.1196496486663818e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 126 -1.3735669665038586e-02</internalNodes>
          <leafValues>
            -5.3785508871078491e-01 6.5542042255401611e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 127 4.7165591269731522e-02</internalNodes>
          <leafValues>
            4.5389361679553986e-02 -6.8944799900054932e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 128 -1.1204879730939865e-02</internalNodes>
          <leafValues>
            1.6932639479637146e-01 -2.3061719536781311e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 129 -1.5478420257568359e-01</internalNodes>
          <leafValues>
            -7.7705371379852295e-01 1.2142470106482506e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 130 5.8086342178285122e-03</internalNodes>
          <leafValues>
            1.1318100243806839e-01 -3.3206319808959961e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 131 -2.8529569506645203e-02</internalNodes>
          <leafValues>
            -5.6747281551361084e-01 4.8734560608863831e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 132 -3.8758948445320129e-02</internalNodes>
          <leafValues>
            5.9423100948333740e-01 -7.5139336287975311e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 133 3.1037809327244759e-02</internalNodes>
          <leafValues>
            5.1973540335893631e-02 -5.8552652597427368e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 134 7.4786080404010136e-06</internalNodes>
          <leafValues>
            -2.7623200416564941e-01 1.4088490605354309e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 135 3.1000260263681412e-02</internalNodes>
          <leafValues>
            3.1331729143857956e-02 -5.6860172748565674e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 136 -4.9860659986734390e-02</internalNodes>
          <leafValues>
            -8.2924622297286987e-01 3.8801580667495728e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 137 -4.2323280125856400e-02</internalNodes>
          <leafValues>
            -4.3062108755111694e-01 1.6579480841755867e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 138 9.1987219639122486e-04</internalNodes>
          <leafValues>
            -2.1154449880123138e-01 1.5517529845237732e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 139 2.0559869706630707e-01</internalNodes>
          <leafValues>
            -6.2403179705142975e-02 3.2229611277580261e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 140 2.9118418693542480e-01</internalNodes>
          <leafValues>
            3.9228469133377075e-02 -9.4128221273422241e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 141 7.8337509185075760e-03</internalNodes>
          <leafValues>
            -1.4806599915027618e-01 1.7849209904670715e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 142 1.1393319815397263e-02</internalNodes>
          <leafValues>
            7.7987723052501678e-02 -4.2424258589744568e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 143 -9.1807022690773010e-02</internalNodes>
          <leafValues>
            3.3689481019973755e-01 -5.6174129247665405e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 144 -1.6038250178098679e-02</internalNodes>
          <leafValues>
            -2.4954010546207428e-01 1.4570869505405426e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 145 5.4830290377140045e-02</internalNodes>
          <leafValues>
            -1.5496000647544861e-01 2.0329600572586060e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 146 2.4449700489640236e-02</internalNodes>
          <leafValues>
            6.0974378138780594e-02 -6.3072341680526733e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 147 2.9260670766234398e-02</internalNodes>
          <leafValues>
            4.6833608299493790e-02 -3.7985381484031677e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 148 3.9965552277863026e-03</internalNodes>
          <leafValues>
            -1.6927300393581390e-01 1.9100320339202881e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 149 -6.9938853383064270e-02</internalNodes>
          <leafValues>
            5.4655587673187256e-01 -5.4965749382972717e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>25</maxWeakCount>
      <stageThreshold>-1.9646480083465576e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 150 4.5835621654987335e-02</internalNodes>
          <leafValues>
            -4.9982848763465881e-01 4.0961080789566040e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 151 2.6363100856542587e-02</internalNodes>
          <leafValues>
            -3.9193201065063477e-01 5.1567757129669189e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 152 1.5189830213785172e-02</internalNodes>
          <leafValues>
            -5.2216362953186035e-01 3.1368219852447510e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 153 -2.0805280655622482e-02</internalNodes>
          <leafValues>
            3.7614479660987854e-01 -4.7375538945198059e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 154 -7.4902721680700779e-03</internalNodes>
          <leafValues>
            1.6283489763736725e-01 -7.0384472608566284e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 155 2.7719369530677795e-01</internalNodes>
          <leafValues>
            -1.6404120624065399e-01 3.3481580018997192e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 156 6.4188443124294281e-02</internalNodes>
          <leafValues>
            -8.0176621675491333e-01 1.2763829529285431e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 157 4.0668170899152756e-02</internalNodes>
          <leafValues>
            -3.3386930823326111e-01 2.8456181287765503e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 158 7.4888020753860474e-03</internalNodes>
          <leafValues>
            -3.7188920378684998e-01 2.5932261347770691e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 159 6.4942672848701477e-02</internalNodes>
          <leafValues>
            1.0372909903526306e-01 -7.1671068668365479e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 160 -2.1149769891053438e-03</internalNodes>
          <leafValues>
            -7.5683927536010742e-01 7.9019591212272644e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 161 -4.8293141298927367e-04</internalNodes>
          <leafValues>
            -4.9852079153060913e-01 8.1111326813697815e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 162 1.3996459543704987e-01</internalNodes>
          <leafValues>
            8.7497599422931671e-02 -7.6389372348785400e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 163 5.2211988717317581e-02</internalNodes>
          <leafValues>
            3.1640481203794479e-02 -5.3281372785568237e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 164 3.0680459458380938e-03</internalNodes>
          <leafValues>
            -6.2458527088165283e-01 1.3869540393352509e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 165 5.0478860735893250e-02</internalNodes>
          <leafValues>
            7.9063497483730316e-02 -7.4017041921615601e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 166 -8.5122063755989075e-03</internalNodes>
          <leafValues>
            -4.9971660971641541e-01 1.1132259666919708e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 167 7.0091806352138519e-02</internalNodes>
          <leafValues>
            9.7081907093524933e-02 -6.1879187822341919e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 168 -2.7261190116405487e-03</internalNodes>
          <leafValues>
            9.7546629607677460e-02 -5.7760041952133179e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 169 1.0676559992134571e-02</internalNodes>
          <leafValues>
            -2.9058128595352173e-01 1.8426120281219482e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 170 6.3848652644082904e-04</internalNodes>
          <leafValues>
            1.3869750499725342e-01 -4.2546540498733521e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 171 -4.7957260161638260e-02</internalNodes>
          <leafValues>
            -7.3249137401580811e-01 4.1188109666109085e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 172 1.7140049487352371e-02</internalNodes>
          <leafValues>
            -3.1973451375961304e-01 1.6840089857578278e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 173 7.8544542193412781e-02</internalNodes>
          <leafValues>
            5.0053231418132782e-02 -7.1410048007965088e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 174 -1.1342849582433701e-02</internalNodes>
          <leafValues>
            -3.8810971379280090e-01 1.2976409494876862e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>53</maxWeakCount>
      <stageThreshold>-2.1222629547119141e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 175 -8.6751781054772437e-05</internalNodes>
          <leafValues>
            2.5179910659790039e-01 -6.7723119258880615e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 176 2.0550179481506348e-01</internalNodes>
          <leafValues>
            2.0217150449752808e-02 -3.3618199825286865e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 177 1.3893260061740875e-01</internalNodes>
          <leafValues>
            1.0678269714117050e-01 -8.6710119247436523e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 178 2.6432450395077467e-03</internalNodes>
          <leafValues>
            -4.1057088971138000e-01 2.5603920221328735e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 179 -1.6145260306075215e-03</internalNodes>
          <leafValues>
            1.7448160052299500e-01 -5.0290131568908691e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 180 -4.6492749825119972e-03</internalNodes>
          <leafValues>
            -8.3960932493209839e-01 1.0409969836473465e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 181 -5.5983918718993664e-03</internalNodes>
          <leafValues>
            -5.2673357725143433e-01 1.2114489823579788e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 182 2.1482799202203751e-03</internalNodes>
          <leafValues>
            8.6831927299499512e-02 -5.2384740114212036e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 183 -2.2942349314689636e-03</internalNodes>
          <leafValues>
            1.5666730701923370e-01 -3.9387580752372742e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 184 -1.0809659725055099e-03</internalNodes>
          <leafValues>
            9.4777546823024750e-02 -5.7967597246170044e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 185 -1.8739879131317139e-02</internalNodes>
          <leafValues>
            -4.3780770897865295e-01 1.2754319608211517e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 186 -2.0956669468432665e-03</internalNodes>
          <leafValues>
            2.1275860071182251e-01 -1.7645539343357086e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 187 -6.1370119452476501e-02</internalNodes>
          <leafValues>
            -6.7007988691329956e-01 8.5291177034378052e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 188 -4.5074969530105591e-02</internalNodes>
          <leafValues>
            -4.7614151239395142e-01 3.8384389132261276e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 189 4.5961341820657253e-03</internalNodes>
          <leafValues>
            9.0776696801185608e-02 -5.3642177581787109e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 190 -5.6205179542303085e-02</internalNodes>
          <leafValues>
            -4.4128128886222839e-01 2.6340639218688011e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 191 -1.7070030793547630e-02</internalNodes>
          <leafValues>
            3.1962528824806213e-01 -1.5699079632759094e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 192 1.3778540305793285e-02</internalNodes>
          <leafValues>
            -4.1468238830566406e-01 1.0832040011882782e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 193 5.6932470761239529e-03</internalNodes>
          <leafValues>
            1.0973270237445831e-01 -4.1420969367027283e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 194 1.1573060182854533e-03</internalNodes>
          <leafValues>
            -4.6996459364891052e-01 1.4088229835033417e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 195 -4.3259391532046720e-05</internalNodes>
          <leafValues>
            -5.9117478132247925e-01 7.2208836674690247e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 196 -1.4467669825535268e-04</internalNodes>
          <leafValues>
            1.4340500533580780e-01 -2.0809020102024078e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 197 -3.0667539685964584e-02</internalNodes>
          <leafValues>
            -6.4181727170944214e-01 7.6316222548484802e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 198 6.4002368599176407e-03</internalNodes>
          <leafValues>
            -1.5426200628280640e-01 2.0618820190429688e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 199 2.7318780776113272e-03</internalNodes>
          <leafValues>
            -1.8429130315780640e-01 2.2046269476413727e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 200 -4.1759859770536423e-02</internalNodes>
          <leafValues>
            5.1284658908843994e-01 -4.3097220361232758e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 201 -3.0174419283866882e-02</internalNodes>
          <leafValues>
            -3.6134809255599976e-01 1.1633390188217163e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 202 6.8081771023571491e-03</internalNodes>
          <leafValues>
            -2.5953280925750732e-01 1.4927390217781067e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 203 4.3430369347333908e-02</internalNodes>
          <leafValues>
            6.8601243197917938e-02 -5.8221191167831421e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 204 2.1121300756931305e-02</internalNodes>
          <leafValues>
            -8.5372917354106903e-02 8.0498583614826202e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 205 9.9840283393859863e-02</internalNodes>
          <leafValues>
            5.3292520344257355e-02 -7.1819657087326050e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 206 5.6953770108520985e-03</internalNodes>
          <leafValues>
            -8.8976107537746429e-02 1.3483940064907074e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 207 -5.9984568506479263e-02</internalNodes>
          <leafValues>
            6.8324291706085205e-01 -5.1916271448135376e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 208 5.9353262186050415e-03</internalNodes>
          <leafValues>
            1.0305190086364746e-01 -2.5361439585685730e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 209 -7.4867930379696190e-05</internalNodes>
          <leafValues>
            1.3340729475021362e-01 -2.9323559999465942e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 210 -2.5437519070692360e-04</internalNodes>
          <leafValues>
            1.5335780382156372e-01 -1.9387570023536682e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 211 7.7576987678185105e-04</internalNodes>
          <leafValues>
            -3.1155571341514587e-01 1.0632509738206863e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 212 5.4478500038385391e-02</internalNodes>
          <leafValues>
            2.6277480646967888e-02 -6.6687411069869995e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 213 1.2692850083112717e-02</internalNodes>
          <leafValues>
            9.3613043427467346e-02 -3.9152190089225769e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 214 -3.0766960233449936e-02</internalNodes>
          <leafValues>
            -5.9238088130950928e-01 4.8314999788999557e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 215 -1.9366150721907616e-02</internalNodes>
          <leafValues>
            4.3661609292030334e-01 -8.8672943413257599e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 216 -2.8705620206892490e-03</internalNodes>
          <leafValues>
            1.5244780480861664e-01 -1.3861170411109924e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 217 4.0003698319196701e-02</internalNodes>
          <leafValues>
            5.8748051524162292e-02 -6.9119709730148315e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 218 -8.1130467355251312e-02</internalNodes>
          <leafValues>
            -7.8684318065643311e-01 2.0421498920768499e-03</leafValues></_>
        <_>
          <internalNodes>
            0 -1 219 -2.1017501130700111e-03</internalNodes>
          <leafValues>
            1.9100449979305267e-01 -1.9659680128097534e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 220 8.6481617763638496e-03</internalNodes>
          <leafValues>
            8.8689289987087250e-02 -3.7414151430130005e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 221 -5.2429020404815674e-02</internalNodes>
          <leafValues>
            -7.2615998983383179e-01 3.9465688169002533e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 222 3.4464800264686346e-03</internalNodes>
          <leafValues>
            -1.1640899628400803e-01 2.7386268973350525e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 223 -7.0581152103841305e-03</internalNodes>
          <leafValues>
            -3.6283940076828003e-01 9.2023678123950958e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 224 -5.7412259280681610e-02</internalNodes>
          <leafValues>
            -8.8839381933212280e-01 2.6647759601473808e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 225 3.3479030244052410e-03</internalNodes>
          <leafValues>
            -1.4884050190448761e-01 1.8366430699825287e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 226 -5.3958419710397720e-02</internalNodes>
          <leafValues>
            3.8098138570785522e-01 -4.4046580791473389e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 227 -2.5719689205288887e-02</internalNodes>
          <leafValues>
            3.2570821046829224e-01 -1.0078220069408417e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>44</maxWeakCount>
      <stageThreshold>-2.1038460731506348e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 228 1.2441220134496689e-01</internalNodes>
          <leafValues>
            -3.8573729991912842e-01 3.9273661375045776e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 229 3.7802878767251968e-02</internalNodes>
          <leafValues>
            -4.7028678655624390e-01 3.5786831378936768e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 230 3.0441429466009140e-02</internalNodes>
          <leafValues>
            -3.9460399746894836e-01 3.2518500089645386e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 231 3.9223438943736255e-04</internalNodes>
          <leafValues>
            -4.5166510343551636e-01 1.9672380387783051e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 232 3.9077710360288620e-02</internalNodes>
          <leafValues>
            -2.1073329448699951e-01 4.3864768743515015e-01</leafValues></_>
        <_>
          <internalNodes>
       
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

