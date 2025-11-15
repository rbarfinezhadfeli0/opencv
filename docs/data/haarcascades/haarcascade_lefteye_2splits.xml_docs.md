# Documentation for `data/haarcascades/haarcascade_lefteye_2splits.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_lefteye_2splits.xml`
- **File Name**: `haarcascade_lefteye_2splits.xml`
- **File Size**: 195,369 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_lefteye_2splits.xml](../../data/haarcascades/haarcascade_lefteye_2splits.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

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
<cascade type_id="opencv-cascade-classifier"><stageType>BOOST</stageType>
  <featureType>HAAR</featureType>
  <height>20</height>
  <width>20</width>
  <stageParams>
    <maxWeakCount>33</maxWeakCount></stageParams>
  <featureParams>
    <maxCatCount>0</maxCatCount></featureParams>
  <stageNum>20</stageNum>
  <stages>
    <_>
      <maxWeakCount>5</maxWeakCount>
      <stageThreshold>-2.3924100399017334e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 1 0 2.7325989678502083e-02 -1 -2 1 -7.0568458177149296e-03</internalNodes>
          <leafValues>
            -9.0600621700286865e-01 9.3385708332061768e-01
            -4.5859959721565247e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 2 -1.2538699805736542e-01 -1 -2 3
            -1.1487299948930740e-01</internalNodes>
          <leafValues>
            7.2463721036911011e-01 5.3034168481826782e-01
            -8.3221220970153809e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 4 -5.8309938758611679e-02 -1 -2 5
            -1.7684370279312134e-02</internalNodes>
          <leafValues>
            6.5408891439437866e-01 2.9482871294021606e-01
            -7.4809581041336060e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 6 3.5937170032411814e-03 -1 -2 7 -1.3436110457405448e-03</internalNodes>
          <leafValues>
            -5.0303918123245239e-01 6.5995341539382935e-01
            -5.5740857124328613e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 8 -2.1795940119773149e-03 -1 -2 9 1.1514870449900627e-02</internalNodes>
          <leafValues>
            -4.2016351222991943e-01 5.9694331884384155e-01
            -8.0508047342300415e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>7</maxWeakCount>
      <stageThreshold>-2.6498730182647705e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 10 -2.2485560178756714e-01 -1 -2 11
            -9.6008004620671272e-03</internalNodes>
          <leafValues>
            -8.1363201141357422e-01 9.0863138437271118e-01
            -3.2208970189094543e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 12 7.4219167232513428e-02 -1 -2 13
            -5.3165741264820099e-03</internalNodes>
          <leafValues>
            -7.5329452753067017e-01 8.6339497566223145e-01
            -3.3463571220636368e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 14 -2.1913449745625257e-03 -1 -2 15
            1.1800959706306458e-02</internalNodes>
          <leafValues>
            -5.5720347166061401e-01 -3.2359680533409119e-01
            6.4163821935653687e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 16 -7.6179709285497665e-03 -1 -2 17
            -9.0587511658668518e-03</internalNodes>
          <leafValues>
            -5.3167867660522461e-01 -7.3611450195312500e-01
            5.5660772323608398e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 18 -4.9959779717028141e-03 -1 -2 19
            8.0803930759429932e-03</internalNodes>
          <leafValues>
            -4.1476911306381226e-01 5.9278357028961182e-01
            -6.7384922504425049e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 20 1.9909010734409094e-03 -1 -2 21
            1.6845749923959374e-03</internalNodes>
          <leafValues>
            -4.2145928740501404e-01 5.4679220914840698e-01
            -7.5099450349807739e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 22 -5.0781872123479843e-03 -1 -2 23
            2.6645609177649021e-03</internalNodes>
          <leafValues>
            -3.9899548888206482e-01 5.8940601348876953e-01
            -4.6778041124343872e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>8</maxWeakCount>
      <stageThreshold>-2.3828399181365967e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 24 -2.5301438570022583e-01 -1 -2 25
            2.9663778841495514e-03</internalNodes>
          <leafValues>
            -7.5402587652206421e-01 -3.5279649496078491e-01
            8.7992298603057861e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 26 -4.7127649188041687e-02 -1 -2 27
            1.9500750349834561e-03</internalNodes>
          <leafValues>
            -5.2234899997711182e-01 -3.0379909276962280e-01
            7.5204378366470337e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 28 -7.1481026709079742e-02 -1 -2 29
            2.2189730405807495e-01</internalNodes>
          <leafValues>
            6.5841901302337646e-01 -6.0907202959060669e-01
            5.6842160224914551e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 30 3.3842820674180984e-02 -1 -2 31
            -5.1714561413973570e-04</internalNodes>
          <leafValues>
            -6.4311647415161133e-01 5.4620361328125000e-01
            -3.9984148740768433e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 32 -3.4458211157470942e-03 -1 -2 33
            2.4395729415118694e-03</internalNodes>
          <leafValues>
            -4.5636838674545288e-01 4.7798189520835876e-01
            -9.1247087717056274e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 34 2.1385070867836475e-03 -1 -2 35
            1.8324409611523151e-03</internalNodes>
          <leafValues>
            -8.3617758750915527e-01 3.3462798595428467e-01
            -7.5008547306060791e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 36 1.1167610064148903e-03 -1 -2 37
            9.9106997367925942e-05</internalNodes>
          <leafValues>
            -6.9083797931671143e-01 -3.4561330080032349e-01
            4.1183179616928101e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 38 1.5447770245373249e-02 -1 -2 39
            -3.2244939357042313e-02</internalNodes>
          <leafValues>
            3.6980190873146057e-01 6.1112838983535767e-01
            -5.5685341358184814e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>9</maxWeakCount>
      <stageThreshold>-2.1312201023101807e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 40 -1.2251129746437073e-01 -1 -2 41
            -1.4230609871447086e-02</internalNodes>
          <leafValues>
            -6.7026627063751221e-01 8.7802392244338989e-01
            -1.8784180283546448e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 42 -5.9833219274878502e-03 -1 -2 43
            7.7085137367248535e-02</internalNodes>
          <leafValues>
            -5.8122849464416504e-01 -5.0395351648330688e-01
            6.7387360334396362e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 44 -1.1086189746856689e-01 -1 -2 45
            9.4604760408401489e-02</internalNodes>
          <leafValues>
            6.3432037830352783e-01 -4.9726390838623047e-01
            3.8787439465522766e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 46 1.7696130089461803e-04 -1 -2 47
            2.0120320841670036e-03</internalNodes>
          <leafValues>
            -6.3938802480697632e-01 -3.5313910245895386e-01
            5.1538437604904175e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 48 -1.6102839726954699e-03 -1 -2 49
            1.6666069859638810e-03</internalNodes>
          <leafValues>
            -5.1915901899337769e-01 4.0478190779685974e-01
            -6.9496357440948486e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 50 -7.1480998303741217e-04 -1 -2 51
            -4.7647571191191673e-03</internalNodes>
          <leafValues>
            -4.8945188522338867e-01 -5.0037759542465210e-01
            4.0796059370040894e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 52 7.8659597784280777e-03 -1 -2 53
            -1.2938310392200947e-03</internalNodes>
          <leafValues>
            -3.3636429905891418e-01 -6.7621380090713501e-01
            4.7010248899459839e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 54 -3.6533139063976705e-04 -1 -2 55
            2.0565679296851158e-03</internalNodes>
          <leafValues>
            -4.7071608901023865e-01 4.1323411464691162e-01
            -5.5526417493820190e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 56 7.8385717642959207e-05 -1 -2 57
            1.7511800397187471e-03</internalNodes>
          <leafValues>
            -5.1521158218383789e-01 3.3417248725891113e-01
            -7.9558157920837402e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>9</maxWeakCount>
      <stageThreshold>-2.0176210403442383e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 58 -6.4695239067077637e-02 -1 -2 59
            9.5212170854210854e-03</internalNodes>
          <leafValues>
            -6.1326402425765991e-01 -5.4831558465957642e-01
            7.8652447462081909e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 60 -9.8109766840934753e-02 -1 -2 61
            -8.5938459634780884e-01</internalNodes>
          <leafValues>
            6.9113308191299438e-01 4.5364680886268616e-01
            -5.0026148557662964e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 62 -8.9836172759532928e-02 -1 -2 63
            2.6945930439978838e-03</internalNodes>
          <leafValues>
            -5.2928781509399414e-01 -3.8199779391288757e-01
            5.7821297645568848e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 64 2.5973599404096603e-03 -1 -2 65
            -3.0058110132813454e-03</internalNodes>
          <leafValues>
            -9.1928368806838989e-01 -8.0213797092437744e-01
            2.9259279370307922e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 66 -4.5496290549635887e-03 -1 -2 67
            4.7376728616654873e-03</internalNodes>
          <leafValues>
            -4.3678951263427734e-01 4.1010880470275879e-01
            -7.2692811489105225e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 68 4.6190437860786915e-03 -1 -2 69
            4.5377281494438648e-03</internalNodes>
          <leafValues>
            -8.4895151853561401e-01 3.0124679207801819e-01
            -7.0301771163940430e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 70 -2.4952790699899197e-03 -1 -2 71
            -5.1753767766058445e-03</internalNodes>
          <leafValues>
            -4.6784749627113342e-01 -7.4530351161956787e-01
            4.0011820197105408e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 72 -5.2049742080271244e-03 -1 -2 73
            -8.7892003357410431e-02</internalNodes>
          <leafValues>
            4.8669269680976868e-01 8.3493947982788086e-01
            -3.3827719092369080e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 74 6.9997250102460384e-03 -1 -2 75
            -9.0990252792835236e-03</internalNodes>
          <leafValues>
            -2.9039889574050903e-01 6.2315821647644043e-01
            -3.5424730181694031e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>11</maxWeakCount>
      <stageThreshold>-2.2212049961090088e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 76 -5.5702101439237595e-02 -1 -2 77
            3.4033291041851044e-02</internalNodes>
          <leafValues>
            -6.9841581583023071e-01 -3.9509189128875732e-01
            8.0313128232955933e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 78 -4.6199060976505280e-02 -1 -2 79
            -4.8061669804155827e-03</internalNodes>
          <leafValues>
            -4.8860380053520203e-01 8.0775612592697144e-01
            -7.4490822851657867e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 80 1.8170489929616451e-03 -1 -2 81
            -3.6162370815873146e-03</internalNodes>
          <leafValues>
            -3.8043528795242310e-01 6.0451722145080566e-01
            -2.2582240402698517e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 82 -1.5706950798630714e-02 -1 -2 83
            4.3929950334131718e-03</internalNodes>
          <leafValues>
            -3.7577998638153076e-01 5.4214221239089966e-01
            -3.7388241291046143e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 84 -1.0047219984699041e-04 -1 -2 85
            -8.6475118994712830e-02</internalNodes>
          <leafValues>
            -4.7433409094810486e-01 5.0186318159103394e-01
            -2.1136230230331421e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 86 -7.7960766851902008e-02 -1 -2 87
            9.8561286926269531e-02</internalNodes>
          <leafValues>
            5.7337349653244019e-01 -3.2515558600425720e-01
            5.3035980463027954e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 88 -5.4359167814254761e-01 -1 -2 89
            -4.4177699834108353e-02</internalNodes>
          <leafValues>
            5.9464299678802490e-01 2.9671078920364380e-01
            -3.8474830985069275e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 90 -8.8016409426927567e-04 -1 -2 91
            2.6359390467405319e-03</internalNodes>
          <leafValues>
            -3.2000589370727539e-01 -1.7586140334606171e-01
            4.8360350728034973e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 92 -1.4203689992427826e-02 -1 -2 93
            -7.3902818257920444e-05</internalNodes>
          <leafValues>
            -7.7882087230682373e-01 3.0619418621063232e-01
            -3.3196049928665161e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 94 4.6157240867614746e-03 -1 -2 95
            1.1152310296893120e-02</internalNodes>
          <leafValues>
            4.9689778685569763e-01 -5.3435891866683960e-01
            9.7229443490505219e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 96 -6.0547702014446259e-03 -1 -2 97
            -2.1118740551173687e-03</internalNodes>
          <leafValues>
            -8.3811217546463013e-01 6.3617032766342163e-01
            -4.8299189656972885e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>13</maxWeakCount>
      <stageThreshold>-2.1328830718994141e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 98 -1.2956829741597176e-02 -1 -2 99
            -2.7141019701957703e-02</internalNodes>
          <leafValues>
            -6.4874732494354248e-01 7.6293057203292847e-01
            -3.3947870135307312e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 100 4.5119998976588249e-03 -1 -2 101
            1.2516690418124199e-02</internalNodes>
          <leafValues>
            -5.0059837102890015e-01 -3.6873328685760498e-01
            5.9888631105422974e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 102 -6.0557941906154156e-03 -1 -2 103
            -4.6923749148845673e-02</internalNodes>
          <leafValues>
            -3.8940930366516113e-01 6.3268911838531494e-01
            -2.6270028948783875e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 104 -2.4018269032239914e-03 -1 -2 105
            -1.5936089679598808e-02</internalNodes>
          <leafValues>
            -5.0517928600311279e-01 6.5526002645492554e-01
            -1.7308109998703003e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 106 1.4000290073454380e-02 -1 -2 107
            1.3202779926359653e-02</internalNodes>
          <leafValues>
            -4.1653230786323547e-01 -4.9121969938278198e-01
            3.7397938966751099e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 108 -2.7658580802381039e-04 -1 -2 109
            -4.8634149134159088e-03</internalNodes>
          <leafValues>
            -4.5382869243621826e-01 -5.9796881675720215e-01
            3.1217721104621887e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 110 2.7654920704662800e-03 -1 -2 111
            2.5534769892692566e-01</internalNodes>
          <leafValues>
            -7.6476567983627319e-01 -3.4267220646142960e-02
            7.0786577463150024e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 112 4.6812961809337139e-03 -1 -2 113
            6.5162130631506443e-03</internalNodes>
          <leafValues>
            -7.8790861368179321e-01 1.8877579271793365e-01
            -7.9132258892059326e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 114 5.7325329631567001e-02 -1 -2 115
            -1.2718330137431622e-02</internalNodes>
          <leafValues>
            6.2349188327789307e-01 3.0860608816146851e-01
            -3.2784330844879150e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 116 -6.7374261561781168e-04 -1 -2 117
            5.6564649567008018e-03</internalNodes>
          <leafValues>
            -4.5451548695564270e-01 2.7431339025497437e-01
            -7.8447937965393066e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 118 3.1134090386331081e-03 -1 -2 119
            2.4249779526144266e-03</internalNodes>
          <leafValues>
            3.9738771319389343e-01 -3.5198271274566650e-01
            3.0490091443061829e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 120 -5.5641461163759232e-02 -1 -2 121
            4.3548129498958588e-02</internalNodes>
          <leafValues>
            4.5575490593910217e-01 -3.3370929956436157e-01
            2.9501429200172424e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 122 8.0783379962667823e-04 -1 -2 123
            1.8713270546868443e-03</internalNodes>
          <leafValues>
            2.2460040450096130e-01 -6.6048407554626465e-01
            1.5031670033931732e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>13</maxWeakCount>
      <stageThreshold>-1.9884539842605591e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 124 -4.3516629934310913e-01 -1 -2 125
            6.2595037743449211e-03</internalNodes>
          <leafValues>
            -4.9959290027618408e-01 -2.3639589548110962e-01
            7.9975378513336182e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 126 -6.6518150269985199e-03 -1 -2 127
            -5.7092090137302876e-03</internalNodes>
          <leafValues>
            -5.4752808809280396e-01 6.4273327589035034e-01
            -2.1511809527873993e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 128 1.9450180232524872e-02 -1 -2 129
            -5.4476498626172543e-03</internalNodes>
          <leafValues>
            -5.3605002164840698e-01 5.5794501304626465e-01
            -2.1474960446357727e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 130 -1.6347589553333819e-04 -1 -2 131
            7.1614650078117847e-03</internalNodes>
          <leafValues>
            -5.5962842702865601e-01 -1.6604369878768921e-01
            4.6805259585380554e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 132 -1.3145170174539089e-02 -1 -2 133
            -1.1436809785664082e-02</internalNodes>
          <leafValues>
            -4.1279909014701843e-01 3.7901800870895386e-01
            -4.1791579127311707e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 134 -7.2912001051008701e-03 -1 -2 135
            -5.2170921117067337e-04</internalNodes>
          <leafValues>
            -7.6089668273925781e-01 3.2527619600296021e-01
            -3.0110970139503479e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 136 3.3754010219126940e-03 -1 -2 137
            2.5100160855799913e-03</internalNodes>
          <leafValues>
            -7.8373962640762329e-01 1.8525449931621552e-01
            -5.8084958791732788e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 138 -1.2884209863841534e-03 -1 -2 139
            -1.8726480193436146e-03</internalNodes>
          <leafValues>
            2.7339500188827515e-01 1.6819879412651062e-01
            -5.1986902952194214e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 140 2.4010189808905125e-03 -1 -2 141
            4.8938081599771976e-03</internalNodes>
          <leafValues>
            -8.2964670658111572e-01 1.6796599328517914e-01
            -6.5530872344970703e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 142 3.1223020050674677e-03 -1 -2 143
            5.0366491079330444e-02</internalNodes>
          <leafValues>
            -4.3521308898925781e-01 -5.8327801525592804e-03
            7.0878309011459351e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 144 3.6151800304651260e-02 -1 -2 145
            -1.3426589965820312e-01</internalNodes>
          <leafValues>
            4.4979161024093628e-01 3.9472430944442749e-01
            -3.7588629126548767e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 146 -2.7791369706392288e-02 -1 -2 147
            -1.2712170369923115e-02</internalNodes>
          <leafValues>
            -2.9488721489906311e-01 -7.2011739015579224e-01
            3.6595028638839722e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 148 -3.8276749546639621e-04 -1 -2 149
            -6.1330529861152172e-03</internalNodes>
          <leafValues>
            -4.0581339597702026e-01 -5.2725958824157715e-01
            3.6040499806404114e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>16</maxWeakCount>
      <stageThreshold>-2.0902318954467773e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 150 -4.7748669981956482e-02 -1 -2 151
            4.6201851218938828e-03</internalNodes>
          <leafValues>
            -5.9902387857437134e-01 -2.4887490272521973e-01
            6.9201582670211792e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 152 -8.5353456437587738e-02 -1 -2 153
            -7.0110969245433807e-03</internalNodes>
          <leafValues>
            -5.1715832948684692e-01 5.6950652599334717e-01
            -2.4749420583248138e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 154 -7.6567470096051693e-03 -1 -2 155
            -3.5919491201639175e-02</internalNodes>
          <leafValues>
            -3.7316519021987915e-01 4.9438580870628357e-01
            -3.9586681127548218e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 156 -7.4326626956462860e-02 -1 -2 157
            9.0118587017059326e-02</internalNodes>
          <leafValues>
            5.6755977869033813e-01 -3.8921171426773071e-01
            3.1079098582267761e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 158 1.6736460849642754e-02 -1 -2 159
            1.8592580454424024e-03</internalNodes>
          <leafValues>
            -3.6674138903617859e-01 3.4875720739364624e-01
            -5.7483112812042236e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 160 7.5264140032231808e-03 -1 -2 161
            -3.5309391096234322e-03</internalNodes>
          <leafValues>
            6.7878991365432739e-01 4.8617920279502869e-01
            -2.5660640001296997e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 162 -4.9510748795000836e-05 -1 -2 163
            -6.8923248909413815e-03</internalNodes>
          <leafValues>
            -4.5661240816116333e-01 -5.7134729623794556e-01
            3.2921048998832703e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 164 6.1156069859862328e-03 -1 -2 165
            -5.5014882236719131e-03</internalNodes>
          <leafValues>
            -7.1315360069274902e-01 -5.9139078855514526e-01
            1.9805949926376343e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 166 -4.2378060519695282e-02 -1 -2 167
            2.2011259570717812e-03</internalNodes>
          <leafValues>
            -3.8239300251007080e-01 3.3457010984420776e-01
            -4.3032339215278625e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 168 2.1217379253357649e-03 -1 -2 169
            6.4385468140244484e-03</internalNodes>
          <leafValues>
            -6.8310022354125977e-01 2.0478610694408417e-01
            -6.1793941259384155e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 170 3.1177410855889320e-03 -1 -2 171
            4.2230269173160195e-04</internalNodes>
          <leafValues>
            5.1137161254882812e-01 -3.6440208554267883e-01
            2.1073049306869507e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 172 -6.5657291561365128e-03 -1 -2 173
            2.5686610024422407e-03</internalNodes>
          <leafValues>
            -6.4581501483917236e-01 2.7643561363220215e-01
            -3.4198498725891113e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 174 -6.2437567976303399e-05 -1 -2 175
            -3.6269261036068201e-03</internalNodes>
          <leafValues>
            -3.1758078932762146e-01 -8.1051957607269287e-01
            2.7218630909919739e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 176 -3.4638389479368925e-03 -1 -2 177
            -7.4930191040039062e-02</internalNodes>
          <leafValues>
            -3.9515769481658936e-01 -5.4353868961334229e-01
            2.6106119155883789e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 178 -9.7247250378131866e-03 -1 -2 179
            4.5450199395418167e-03</internalNodes>
          <leafValues>
            4.1124871373176575e-01 -3.1576550006866455e-01
            3.9046970009803772e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 180 -2.7354240883141756e-03 -1 -2 181
            -1.6969470307230949e-02</internalNodes>
          <leafValues>
            -7.4906748533248901e-01 -6.2437218427658081e-01
            1.8387380242347717e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>15</maxWeakCount>
      <stageThreshold>-1.9407310485839844e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 182 -2.4978699162602425e-02 -1 -2 183
            -5.8007869869470596e-02</internalNodes>
          <leafValues>
            -6.0697889328002930e-01 7.1478021144866943e-01
            -2.9943239688873291e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 184 -5.1753749139606953e-03 -1 -2 185
            -8.9618662605062127e-04</internalNodes>
          <leafValues>
            -3.5297989845275879e-01 5.4417461156845093e-01
            -3.9789950847625732e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 186 -2.8718139219563454e-05 -1 -2 187
            4.7620530240237713e-03</internalNodes>
          <leafValues>
            -4.8898181319236755e-01 -3.1144559383392334e-01
            4.6786791086196899e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 188 1.9751280546188354e-02 -1 -2 189
            -1.2683609966188669e-03</internalNodes>
          <leafValues>
            -4.3020489811897278e-01 -5.4090851545333862e-01
            3.9797520637512207e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 190 -4.5749718992738053e-05 -1 -2 191
            2.4090509396046400e-03</internalNodes>
          <leafValues>
            -4.4518938660621643e-01 2.8822308778762817e-01
            -5.4514312744140625e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 192 -4.5728669501841068e-03 -1 -2 193
            8.9018214493989944e-03</internalNodes>
          <leafValues>
            5.5039870738983154e-01 -4.1598889231681824e-01
            1.7468899488449097e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 194 -1.2056449800729752e-01 -1 -2 195
            4.6919930726289749e-02</internalNodes>
          <leafValues>
            6.8890577554702759e-01 -4.2266309261322021e-01
            1.7010940611362457e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 196 -4.2390259914100170e-03 -1 -2 197
            3.2174249645322561e-03</internalNodes>
          <leafValues>
            -6.3045340776443481e-01 -3.6097949743270874e-01
            2.4933730065822601e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 198 -8.5738790221512318e-04 -1 -2 199
            -1.8432449549436569e-02</internalNodes>
          <leafValues>
            3.0993479490280151e-01 9.7758449614048004e-02
            -5.0742352008819580e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 200 5.8692828752100468e-03 -1 -2 201
            -6.8751699291169643e-03</internalNodes>
          <leafValues>
            -7.4556058645248413e-01 -6.7458391189575195e-01
            1.5918810665607452e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 202 -6.8542227381840348e-05 -1 -2 203
            -1.0658579878509045e-02</internalNodes>
          <leafValues>
            -4.1279420256614685e-01 3.7002709507942200e-01
            -2.1731729805469513e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 204 -1.8811509944498539e-03 -1 -2 205
            -2.2309130057692528e-02</internalNodes>
          <leafValues>
            5.7902830839157104e-01 1.9725680351257324e-01
            -3.2475191354751587e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 206 6.5826578065752983e-04 -1 -2 207
            -5.0781588070094585e-03</internalNodes>
          <leafValues>
            -6.0630238056182861e-01 -7.7123302221298218e-01
            1.8186129629611969e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 208 5.6215081363916397e-02 -1 -2 209
            -3.7720590829849243e-02</internalNodes>
          <leafValues>
            5.0561398267745972e-01 3.6052110791206360e-01
            -3.2743760943412781e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 210 3.9480631239712238e-03 -1 -2 211
            -2.4269670248031616e-03</internalNodes>
          <leafValues>
            -7.5788182020187378e-01 5.2076101303100586e-01
            -6.1021361500024796e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>19</maxWeakCount>
      <stageThreshold>-2.1061589717864990e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 212 -1.6906699165701866e-02 -1 -2 213
            2.5327840819954872e-02</internalNodes>
          <leafValues>
            -4.7501268982887268e-01 -4.4016760587692261e-01
            6.0885351896286011e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 214 -1.5663320198655128e-02 -1 -2 215
            -1.6101899743080139e-01</internalNodes>
          <leafValues>
            5.7100051641464233e-01 4.0989148616790771e-01
            -3.8142371177673340e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 216 1.6885380318854004e-04 -1 -2 217
            -3.0552360694855452e-03</internalNodes>
          <leafValues>
            -4.7958490252494812e-01 4.2852300405502319e-01
            -2.8252631425857544e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 218 4.8042940907180309e-03 -1 -2 219
            -5.0092511810362339e-03</internalNodes>
          <leafValues>
            -6.8659138679504395e-01 -5.9033542871475220e-01
            1.9732500612735748e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 220 -3.7119518965482712e-02 -1 -2 221
            3.7857799325138330e-03</internalNodes>
          <leafValues>
            -4.3130961060523987e-01 3.3596190810203552e-01
            -3.7401720881462097e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 222 -1.0869850404560566e-02 -1 -2 223
            4.0577541221864522e-04</internalNodes>
          <leafValues>
            5.4841208457946777e-01 -5.0022697448730469e-01
            5.1423858851194382e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 224 5.0201490521430969e-03 -1 -2 225
            2.5601210072636604e-03</internalNodes>
          <leafValues>
            -5.9016227722167969e-01 1.9469800591468811e-01
            -6.4648360013961792e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 226 -1.2395749799907207e-03 -1 -2 227
            -5.1075750961899757e-03</internalNodes>
          <leafValues>
            -2.7762159705162048e-01 -6.1149162054061890e-01
            3.5250389575958252e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 228 -6.4853738876990974e-05 -1 -2 229
            2.3282810579985380e-03</internalNodes>
          <leafValues>
            -3.4008860588073730e-01 2.7134749293327332e-01
            -6.6915398836135864e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 230 -1.5571110416203737e-03 -1 -2 231
            2.3992219939827919e-03</internalNodes>
          <leafValues>
            -4.1144248843193054e-01 2.5939700007438660e-01
            -4.0380299091339111e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 232 7.7784422319382429e-04 -1 -2 233
            3.2334199640899897e-03</internalNodes>
          <leafValues>
            2.9523921012878418e-01 -5.8436852693557739e-01
            -1.7936639487743378e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 234 -5.6113858590833843e-05 -1 -2 235
            1.9111000001430511e-03</internalNodes>
          <leafValues>
            -3.5021650791168213e-01 2.6312610507011414e-01
            -6.1549347639083862e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 236 -3.4321150742471218e-03 -1 -2 237
            -1.4541969634592533e-02</internalNodes>
          <leafValues>
            3.7493300437927246e-01 4.3788930773735046e-01
            -3.0131611227989197e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 238 -2.5027070194482803e-02 -1 -2 239
            -3.1183639075607061e-03</internalNodes>
          <leafValues>
            -5.2829748392105103e-01 -8.1336849927902222e-01
            1.7928420007228851e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 240 2.9415208846330643e-03 -1 -2 241
            -2.4807679001241922e-03</internalNodes>
          <leafValues>
            -4.7243058681488037e-01 -6.0058331489562988e-01
            2.1497109532356262e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 242 -4.2498838156461716e-03 -1 -2 243
            7.6959328725934029e-03</internalNodes>
          <leafValues>
            -3.3230608701705933e-01 2.1247069537639618e-01
            -8.1967252492904663e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 244 -6.1426039785146713e-02 -1 -2 245
            5.3176790475845337e-02</internalNodes>
          <leafValues>
            5.2200448513031006e-01 -2.9851761460304260e-01
            2.8654190897941589e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 246 2.5695779186207801e-05 -1 -2 247
            2.4311970919370651e-03</internalNodes>
          <leafValues>
            -3.4719291329383850e-01 -1.2133490294218063e-01
            3.8965350389480591e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 248 5.6956289336085320e-03 -1 -2 249
            -6.6630227956920862e-04</internalNodes>
          <leafValues>
            -6.6364032030105591e-01 2.7921909093856812e-01
            -2.1624849736690521e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>20</maxWeakCount>
      <stageThreshold>-2.0051579475402832e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 250 -2.8509549796581268e-02 -1 -2 251
            -1.6429109498858452e-02</internalNodes>
          <leafValues>
            -5.5133241415023804e-01 6.0328769683837891e-01
            -3.0009600520133972e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 252 -5.8078952133655548e-03 -1 -2 253
            -1.4670349657535553e-02</internalNodes>
          <leafValues>
            -4.8640519380569458e-01 4.4786658883094788e-01
            -3.5448360443115234e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 254 -1.0694459779188037e-03 -1 -2 255
            -5.0697539001703262e-02</internalNodes>
          <leafValues>
            -3.8593119382858276e-01 4.3865600228309631e-01
            -3.1134051084518433e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 256 -7.2318017482757568e-02 -1 -2 257
            -1.6740759834647179e-02</internalNodes>
          <leafValues>
            5.5695492029190063e-01 3.4036931395530701e-01
            -3.7713068723678589e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 258 1.2923260219395161e-02 -1 -2 259
            -2.0832989830523729e-03</internalNodes>
          <leafValues>
            2.6987180113792419e-01 7.2217263281345367e-02
            -5.0617259740829468e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 260 2.9217539122328162e-04 -1 -2 261
            4.6477448195219040e-03</internalNodes>
          <leafValues>
            -4.7199469804763794e-01 -2.0233640074729919e-01
            3.6684620380401611e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 262 1.6355320112779737e-03 -1 -2 263
            6.0143060982227325e-03</internalNodes>
          <leafValues>
            -3.3369150757789612e-01 2.6335370540618896e-01
            -7.5315129756927490e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 264 -1.9768040627241135e-02 -1 -2 265
            5.0995801575481892e-03</internalNodes>
          <leafValues>
            -7.3396641016006470e-01 -1.0626330226659775e-01
            3.7877479195594788e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 266 2.1737320348620415e-03 -1 -2 267
            2.3621059954166412e-02</internalNodes>
          <leafValues>
            -4.5873621106147766e-01 -3.7341989576816559e-02
            5.0312960147857666e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 268 4.7070439904928207e-02 -1 -2 269
            4.8429161310195923e-02</internalNodes>
          <leafValues>
            3.9159670472145081e-01 -2.7507638931274414e-01
            3.6923450231552124e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 270 7.1763257437851280e-05 -1 -2 271
            -4.0031517855823040e-03</internalNodes>
          <leafValues>
            -2.6133701205253601e-01 -4.6118479967117310e-01
            3.4101578593254089e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 272 2.5536299217492342e-03 -1 -2 273
            -2.5720898993313313e-03</internalNodes>
          <leafValues>
            4.4237849116325378e-01 4.3066531419754028e-01
            -2.8360688686370850e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 274 8.7512210011482239e-03 -1 -2 275
            5.7346918620169163e-03</internalNodes>
          <leafValues>
            -7.7647632360458374e-01 1.4551159739494324e-01
            -7.5074160099029541e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 276 -6.6438838839530945e-03 -1 -2 277
            -3.4590701106935740e-03</internalNodes>
          <leafValues>
            4.0350550413131714e-01 2.8769719600677490e-01
            -2.8021600842475891e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 278 9.9742468446493149e-03 -1 -2 279
            1.3233659788966179e-02</internalNodes>
          <leafValues>
            -6.0677021741867065e-01 1.5478080511093140e-01
            -7.0759147405624390e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 280 -5.0271311774849892e-03 -1 -2 281
            -1.2092100223526359e-04</internalNodes>
          <leafValues>
            -7.3897778987884521e-01 2.3473000526428223e-01
            -2.4400579929351807e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 282 -1.2881499715149403e-03 -1 -2 283
            6.2854858115315437e-03</internalNodes>
          <leafValues>
            -2.8901669383049011e-01 2.8100869059562683e-01
            -5.6933850049972534e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 284 5.6929360143840313e-03 -1 -2 285
            -5.3880861960351467e-03</internalNodes>
          <leafValues>
            -7.8456932306289673e-01 2.6201328635215759e-01
            -2.2232030332088470e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 286 4.8205819912254810e-03 -1 -2 287
            3.4279188513755798e-01</internalNodes>
          <leafValues>
            5.6795972585678101e-01 -1.8314230442047119e-01
            5.4108071327209473e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 288 5.1370919682085514e-03 -1 -2 289
            -9.1285221278667450e-03</internalNodes>
          <leafValues>
            -3.9116761088371277e-01 5.3076338768005371e-01
            -3.0019309371709824e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>21</maxWeakCount>
      <stageThreshold>-2.1121981143951416e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 290 -5.1386129111051559e-02 -1 -2 291
            5.1850839518010616e-03</internalNodes>
          <leafValues>
            -5.3148782253265381e-01 -2.4744540452957153e-01
            6.1181622743606567e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 292 -1.5259400010108948e-02 -1 -2 293
            2.5995150208473206e-02</internalNodes>
          <leafValues>
            -4.3303629755973816e-01 4.3979901820421219e-02
            7.3829138278961182e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 294 -3.2312370836734772e-02 -1 -2 295
            1.3700700365006924e-02</internalNodes>
          <leafValues>
            -3.9609751105308533e-01 -2.7643880248069763e-01
            4.2535358667373657e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 296 -2.2647869773209095e-03 -1 -2 297
            -6.8290620110929012e-03</internalNodes>
          <leafValues>
            -3.2005569338798523e-01 -5.1682972908020020e-01
            3.6975708603858948e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 298 -2.2481549531221390e-03 -1 -2 299
            4.5944549143314362e-02</internalNodes>
          <leafValues>
            -3.6244350671768188e-01 -1.3187309959903359e-03
            6.3217681646347046e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 300 1.8755620112642646e-03 -1 -2 301
            -1.9700559787452221e-03</internalNodes>
          <leafValues>
            -7.1403390169143677e-01 -5.8730661869049072e-01
            1.7592810094356537e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 302 -6.5721389837563038e-03 -1 -2 303
            -1.1746180243790150e-02</internalNodes>
          <leafValues>
            -3.6347511410713196e-01 3.1440791487693787e-01
            -4.0111118555068970e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 304 -1.6494120063725859e-04 -1 -2 305
            -7.2169408667832613e-05</internalNodes>
          <leafValues>
            -3.7792590260505676e-01 5.2791112661361694e-01
            -1.0790319740772247e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 306 1.9697639800142497e-04 -1 -2 307
            -1.1423509567975998e-02</internalNodes>
          <leafValues>
            -4.7097641229629517e-01 -8.5209292173385620e-01
            1.7662869393825531e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 308 -4.5562228187918663e-03 -1 -2 309
            -4.4720191508531570e-03</internalNodes>
          <leafValues>
            -8.0601161718368530e-01 -6.1500209569931030e-01
            1.2908309698104858e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 310 -1.7765410011634231e-03 -1 -2 311
            -7.8799277544021606e-03</internalNodes>
          <leafValues>
            3.1382599472999573e-01 3.0394628643989563e-01
            -3.7204921245574951e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 312 -1.4284689677879214e-03 -1 -2 313
            -1.8939910223707557e-03</internalNodes>
          <leafValues>
            5.0413030385971069e-01 3.4823760390281677e-01
            -2.3673820495605469e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 314 -3.1496640294790268e-03 -1 -2 315
            -1.0716119781136513e-02</internalNodes>
          <leafValues>
            -6.6812378168106079e-01 -4.8515519499778748e-01
            1.9036419689655304e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 316 -6.8033537827432156e-03 -1 -2 317
            1.4902319759130478e-02</internalNodes>
          <leafValues>
            -5.6979268789291382e-01 1.3098250329494476e-01
            -7.1448272466659546e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 318 -3.4170228987932205e-02 -1 -2 319
            -1.4779250323772430e-01</internalNodes>
          <leafValues>
            5.0575131177902222e-01 2.8233268857002258e-01
            -2.7205321192741394e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 320 -5.5842810979811475e-05 -1 -2 321
            3.9885081350803375e-02</internalNodes>
          <leafValues>
            -2.6936730742454529e-01 5.6696129031479359e-03
            6.3975161314010620e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 322 1.2483130209147930e-02 -1 -2 323
            -3.2864511013031006e-04</internalNodes>
          <leafValues>
            -7.4533742666244507e-01 3.6449620127677917e-01
            -9.6498817205429077e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 324 -1.4710469986312091e-04 -1 -2 325
            -2.7814340591430664e-01</internalNodes>
          <leafValues>
            1.4060440659523010e-01 5.7002830505371094e-01
            -4.8755478858947754e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 326 -1.3452640268951654e-03 -1 -2 327
            9.1500842245295644e-04</internalNodes>
          <leafValues>
            3.9255830645561218e-01 -3.0215170979499817e-01
            3.6698031425476074e-01</leafValues></_>
        
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

