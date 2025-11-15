# Documentation for `docs/data/haarcascades/haarcascade_eye_tree_eyeglasses.xml_docs.md`

## File Metadata

- **Full Path**: `docs/data/haarcascades/haarcascade_eye_tree_eyeglasses.xml_docs.md`
- **File Name**: `haarcascade_eye_tree_eyeglasses.xml_docs.md`
- **File Size**: 51,088 bytes
- **File Type**: .md
- **Link to Source**: [docs/data/haarcascades/haarcascade_eye_tree_eyeglasses.xml_docs.md](../../../docs/data/haarcascades/haarcascade_eye_tree_eyeglasses.xml_docs.md)

## Purpose and Role

This file is located in the `docs/data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `data/haarcascades/haarcascade_eye_tree_eyeglasses.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_eye_tree_eyeglasses.xml`
- **File Name**: `haarcascade_eye_tree_eyeglasses.xml`
- **File Size**: 601,661 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_eye_tree_eyeglasses.xml](../../data/haarcascades/haarcascade_eye_tree_eyeglasses.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    Tree-based 20x20 frontal eye detector with better handling of eyeglasses.
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
<cascade type_id="opencv-cascade-classifier"><stageType>BOOST</stageType>
  <featureType>HAAR</featureType>
  <height>20</height>
  <width>20</width>
  <stageParams>
    <maxWeakCount>47</maxWeakCount></stageParams>
  <featureParams>
    <maxCatCount>0</maxCatCount></featureParams>
  <stageNum>30</stageNum>
  <stages>
    <_>
      <maxWeakCount>5</maxWeakCount>
      <stageThreshold>-1.6473180055618286e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            2 1 0 -2.6987109333276749e-02 0 -1 1 5.0670530647039413e-02
            -2 -3 2 -1.2915390729904175e-01</internalNodes>
          <leafValues>
            -8.0395472049713135e-01 6.0491400957107544e-01
            9.0544581413269043e-01 4.4070810079574585e-02</leafValues></_>
        <_>
          <internalNodes>
            2 1 3 8.8827736675739288e-02 0 -1 4 -2.0398240536451340e-02
            -2 -3 5 -6.1261758208274841e-02</internalNodes>
          <leafValues>
            7.9218882322311401e-01 4.0692299604415894e-02
            4.2585361003875732e-01 -7.0325207710266113e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 6 -2.0490810275077820e-01 0 -1 7 9.4933047890663147e-02
            -2 -3 8 1.2091030366718769e-03</internalNodes>
          <leafValues>
            -4.4017648696899414e-01 5.3640520572662354e-01
            6.8776458501815796e-01 -5.5879348516464233e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 9 9.2227972345426679e-04 -1 2 10 -7.2678289143368602e-04
            -2 -3 11 6.8421510513871908e-04</internalNodes>
          <leafValues>
            -7.2684401273727417e-01 -5.8028000593185425e-01
            5.6177532672882080e-01 -2.9834181070327759e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 12 -5.1150590181350708e-02 2 -1 13
            6.1622060835361481e-02 -2 -3 14 7.2873473167419434e-02</internalNodes>
          <leafValues>
            5.9840762615203857e-01 7.4743932485580444e-01
            -4.9703779816627502e-01 2.8129258751869202e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>7</maxWeakCount>
      <stageThreshold>-1.4257860183715820e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            2 1 15 -4.1994878649711609e-01 0 -1 16
            -5.6186288595199585e-02 -2 -3 17 -2.3711109533905983e-02</internalNodes>
          <leafValues>
            2.7586200833320618e-01 -6.4623218774795532e-01
            8.5241252183914185e-01 8.3703370764851570e-03</leafValues></_>
        <_>
          <internalNodes>
            1 0 18 4.0523439645767212e-02 -1 2 19 2.7388900518417358e-01
            -2 -3 20 -1.4293800108134747e-02</internalNodes>
          <leafValues>
            7.4270218610763550e-01 -4.9286690354347229e-01
            7.1784788370132446e-01 -4.2223978787660599e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 21 -2.1144729107618332e-03 2 -1 22
            1.0659949621185660e-03 -2 -3 23 1.0812469990924001e-03</internalNodes>
          <leafValues>
            -8.0196601152420044e-01 -6.6025912761688232e-01
            4.7916370630264282e-01 -5.1645290851593018e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 24 3.0198289081454277e-02 2 -1 25 4.0569551289081573e-02
            -2 -3 26 7.0679739117622375e-02</internalNodes>
          <leafValues>
            5.1327562332153320e-01 6.6641497611999512e-01
            -4.5298659801483154e-01 5.5480718612670898e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 27 -7.8928138827905059e-04 2 -1 28
            8.0574717139825225e-04 -2 -3 29 -2.0976560190320015e-02</internalNodes>
          <leafValues>
            -7.2526299953460693e-01 -5.6479871273040771e-01
            6.9993537664413452e-01 6.8500466644763947e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 30 1.2794960290193558e-02 -1 2 31
            -8.1120636314153671e-03 -2 -3 32 -1.5506530180573463e-02</internalNodes>
          <leafValues>
            -8.6409568786621094e-01 4.4448360800743103e-01
            3.6675310134887695e-01 -2.9189071059226990e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 33 -1.2915650382637978e-02 0 -1 34
            6.6297221928834915e-03 -2 -3 35 -3.6532930098474026e-03</internalNodes>
          <leafValues>
            -4.7566780447959900e-01 1.0350350290536880e-01
            -6.1723059415817261e-01 5.4382532835006714e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>9</maxWeakCount>
      <stageThreshold>-1.4711019992828369e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 1 36 -7.8731971979141235e-01 -1 2 37
            1.6908009350299835e-01 -2 -3 38 -4.0369689464569092e-02</internalNodes>
          <leafValues>
            7.1268838644027710e-01 -7.1908998489379883e-01
            4.4148930907249451e-01 -4.2251929640769958e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 39 1.9132360816001892e-02 2 -1 40 6.4184539951384068e-04
            -2 -3 41 -7.8941037645563483e-04</internalNodes>
          <leafValues>
            6.9186228513717651e-01 -7.6116967201232910e-01
            -6.8140429258346558e-01 1.6009919345378876e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 42 -7.1503049694001675e-03 0 -1 43
            -2.3156129755079746e-03 -2 -3 44 -4.1521269828081131e-02</internalNodes>
          <leafValues>
            -5.5916607379913330e-01 5.1284497976303101e-01
            2.4422569572925568e-01 -4.6883401274681091e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 45 9.1200548922643065e-04 -1 2 46
            -1.5798299573361874e-03 -2 -3 47 -1.1573649942874908e-02</internalNodes>
          <leafValues>
            -6.9527888298034668e-01 -6.3509649038314819e-01
            6.4686381816864014e-01 6.9198559504002333e-04</leafValues></_>
        <_>
          <internalNodes>
            2 1 48 2.1843519061803818e-03 0 -1 49 2.9345690272748470e-03
            -2 -3 50 -5.8788150548934937e-02</internalNodes>
          <leafValues>
            4.5632898807525635e-01 -5.8841437101364136e-01
            2.6704201102256775e-01 -3.8348990678787231e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 51 -5.5392808280885220e-04 -1 2 52
            -5.3035060409456491e-04 -2 -3 53 -6.8775108084082603e-03</internalNodes>
          <leafValues>
            -4.8913368582725525e-01 -3.8421550393104553e-01
            6.6845697164535522e-01 9.3158259987831116e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 54 1.6710379859432578e-03 2 -1 55 1.4162790030241013e-03
            -2 -3 56 7.7876187860965729e-03</internalNodes>
          <leafValues>
            -6.0369372367858887e-01 -3.0418768525123596e-01
            3.9699068665504456e-01 -6.6687589883804321e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 57 -1.2916780076920986e-02 0 -1 58
            -3.0156269203871489e-03 -2 -3 59 -1.9785940647125244e-02</internalNodes>
          <leafValues>
            -7.1239727735519409e-01 4.6252989768981934e-01
            2.8338319063186646e-01 -3.5317930579185486e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 60 3.3207770902663469e-03 2 -1 61 2.9606239870190620e-02
            -2 -3 62 4.4614788144826889e-02</internalNodes>
          <leafValues>
            -7.3291397094726562e-01 4.9530759453773499e-01
            -1.9502809643745422e-01 7.9816418886184692e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>12</maxWeakCount>
      <stageThreshold>-1.3850779533386230e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 1 63 -9.2366141080856323e-01 2 -1 64
            -4.8193939030170441e-02 -2 -3 65 2.8669878840446472e-01</internalNodes>
          <leafValues>
            7.6915800571441650e-01 -5.1361227035522461e-01
            -2.9671901464462280e-01 6.2028187513351440e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 66 -1.3038160279393196e-02 0 -1 67
            -1.4749659458175302e-03 -2 -3 68 -4.6921748667955399e-02</internalNodes>
          <leafValues>
            -7.1294248104095459e-01 5.9115177392959595e-01
            3.1303560733795166e-01 -3.6749690771102905e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 69 2.4459899868816137e-03 -1 2 70
            -2.5321498978883028e-03 -2 -3 71 1.4651260571554303e-03</internalNodes>
          <leafValues>
            -4.6930000185966492e-01 -7.7450162172317505e-01
            3.6414781212806702e-01 -5.7445889711380005e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 72 -1.1307420209050179e-02 0 -1 73
            -1.2048849603161216e-03 -2 -3 74 -6.2752872705459595e-02</internalNodes>
          <leafValues>
            -5.5727648735046387e-01 4.7871670126914978e-01
            2.2788530588150024e-01 -4.3667969107627869e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 75 -4.0173111483454704e-03 2 -1 76
            1.5160309849306941e-03 -2 -3 77 1.9954680465161800e-03</internalNodes>
          <leafValues>
            -7.3568779230117798e-01 -5.8480697870254517e-01
            2.1544020622968674e-02 5.5875688791275024e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 78 3.4435209818184376e-03 -1 2 79
            -2.6550020556896925e-03 -2 -3 80 -1.1407690122723579e-02</internalNodes>
          <leafValues>
            -7.6565897464752197e-01 -6.5447497367858887e-01
            5.3633081912994385e-01 -3.8849171251058578e-02</leafValues></_>
        <_>
          <internalNodes>
            1 2 81 -2.3805440869182348e-03 0 -1 82
            6.6475258208811283e-03 -2 -3 83 1.4018240571022034e-01</internalNodes>
          <leafValues>
            3.3984410762786865e-01 -6.5025091171264648e-01
            -3.2491090893745422e-01 7.5067067146301270e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 84 -6.2358360737562180e-02 2 -1 85
            1.3628599699586630e-03 -2 -3 86 -4.4609848409891129e-03</internalNodes>
          <leafValues>
            4.5777168869972229e-01 -6.3202661275863647e-01
            4.0597960352897644e-01 -2.0854069292545319e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 87 -1.0046839714050293e-02 2 -1 88
            -2.9274819418787956e-02 -2 -3 89 7.7389390207827091e-03</internalNodes>
          <leafValues>
            -7.4789828062057495e-01 -1.7995479702949524e-01
            4.7782841324806213e-01 -6.5113341808319092e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 90 1.4774020528420806e-03 -1 2 91 1.4989820308983326e-02
            -2 -3 92 4.5073241926729679e-03</internalNodes>
          <leafValues>
            -6.6269898414611816e-01 -1.6695550084114075e-01
            3.8702058792114258e-01 -7.3409372568130493e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 93 1.4901049435138702e-03 2 -1 94 8.9141662465408444e-04
            -2 -3 95 -1.1558219790458679e-02</internalNodes>
          <leafValues>
            -3.4280839562416077e-01 -2.8036740422248840e-01
            -4.2523959279060364e-01 4.5259669423103333e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 96 -2.0011950284242630e-02 -1 2 97
            -1.7092300578951836e-02 -2 -3 98 -6.7685171961784363e-02</internalNodes>
          <leafValues>
            4.0133118629455566e-01 3.6970010399818420e-01
            7.4438679218292236e-01 -3.8255840539932251e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>12</maxWeakCount>
      <stageThreshold>-1.4432040452957153e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 2 99 -2.0911149680614471e-02 0 -1 100
            1.4305709302425385e-01 -2 -3 101 1.1925029568374157e-02</internalNodes>
          <leafValues>
            -3.4965568780899048e-01 7.0134562253952026e-01
            -6.0404628515243530e-01 8.5615903139114380e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 102 2.4742009118199348e-02 2 -1 103
            4.5732118189334869e-02 -2 -3 104 4.3204430490732193e-02</internalNodes>
          <leafValues>
            8.5365587472915649e-01 4.1876411437988281e-01
            -3.9094918966293335e-01 2.7387988567352295e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 105 -7.2548422031104565e-04 2 -1 106
            1.4243220211938024e-03 -2 -3 107 -5.3335479460656643e-03</internalNodes>
          <leafValues>
            -6.2011122703552246e-01 -6.1589437723159790e-01
            6.0596448183059692e-01 1.5840480104088783e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 108 -7.1891010738909245e-03 2 -1 109
            1.8233320442959666e-03 -2 -3 110 1.6109029529616237e-03</internalNodes>
          <leafValues>
            -2.0852829515933990e-01 -8.1338381767272949e-01
            5.6780648231506348e-01 -8.7046259641647339e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 111 -4.8350278288125992e-02 0 -1 112
            3.1746171414852142e-02 -2 -3 113 1.9233829807490110e-03</internalNodes>
          <leafValues>
            -3.5335820913314819e-01 4.4076570868492126e-01
            4.0730631351470947e-01 -5.9592568874359131e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 114 1.3614529743790627e-03 -1 2 115
            -3.6934199742972851e-03 -2 -3 116 -8.5378461517393589e-04</internalNodes>
          <leafValues>
            -5.5307251214981079e-01 -7.3163098096847534e-01
            4.3890678882598877e-01 -6.3009172677993774e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 117 -1.0950770229101181e-02 -1 2 118
            -7.2186449542641640e-03 -2 -3 119 1.8548289313912392e-02</internalNodes>
          <leafValues>
            3.9263078570365906e-01 2.7225250005722046e-01
            -4.1208618879318237e-01 6.3790637254714966e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 120 1.0859060566872358e-03 -1 2 121
            -6.5618362277746201e-03 -2 -3 122 -6.1777420341968536e-02</internalNodes>
          <leafValues>
            -5.0857210159301758e-01 3.5386729240417480e-01
            5.7568281888961792e-01 -2.8477248549461365e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 123 4.9480778397992253e-04 2 -1 124
            1.1606880463659763e-02 -2 -3 125 -1.6142609529197216e-03</internalNodes>
          <leafValues>
            -4.9583891034126282e-01 -5.1320201158523560e-01
            5.2665728330612183e-01 3.0917160212993622e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 126 2.0437680650502443e-03 -1 2 127
            -8.2394909113645554e-03 -2 -3 128 -3.9699211716651917e-02</internalNodes>
          <leafValues>
            -7.0948588848114014e-01 3.4189811348915100e-01
            4.7383341193199158e-01 -2.5060850381851196e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 129 -8.0377282574772835e-04 0 -1 130
            -5.4273242130875587e-03 -2 -3 131 -5.2662738598883152e-03</internalNodes>
          <leafValues>
            -5.1384007930755615e-01 2.9752710461616516e-01
            1.4577029645442963e-01 -4.6007528901100159e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 132 6.3841522205621004e-04 -1 2 133
            -1.5458120033144951e-03 -2 -3 134 1.1863360414281487e-03</internalNodes>
          <leafValues>
            -3.6412829160690308e-01 -5.8081609010696411e-01
            2.9298609495162964e-01 -5.1420718431472778e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>12</maxWeakCount>
      <stageThreshold>-1.5415630340576172e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 2 135 -2.7745011448860168e-01 0 -1 136
            -3.1200000084936619e-03 -2 -3 137 -8.0280922353267670e-02</internalNodes>
          <leafValues>
            8.3265638351440430e-01 1.0233189910650253e-01
            2.3773579299449921e-01 -6.4546662569046021e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 138 -6.9391548633575439e-02 2 -1 139
            5.3355181589722633e-03 -2 -3 140 -5.4189618676900864e-02</internalNodes>
          <leafValues>
            4.6008241176605225e-01 2.9137989878654480e-01
            4.7026729583740234e-01 -5.7723402976989746e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 141 1.8562959507107735e-02 -1 2 142
            4.6305730938911438e-02 -2 -3 143 -8.8262781500816345e-03</internalNodes>
          <leafValues>
            7.0555502176284790e-01 -5.2839881181716919e-01
            4.3953609466552734e-01 -1.3887490332126617e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 144 -2.8772179502993822e-03 -1 2 145
            -2.6457069907337427e-03 -2 -3 146 3.3441530540585518e-03</internalNodes>
          <leafValues>
            -2.7475830912590027e-01 -5.7746797800064087e-01
            3.6615240573883057e-01 -6.3586741685867310e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 147 -8.3742372691631317e-02 0 -1 148
            1.0164769738912582e-01 -2 -3 149 -2.1541758906096220e-03</internalNodes>
          <leafValues>
            -2.9664519429206848e-01 5.6140047311782837e-01
            -7.5446271896362305e-01 3.9601260423660278e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 150 -1.7133549554273486e-03 2 -1 151
            1.3899410143494606e-02 -2 -3 152 -2.8498120605945587e-02</internalNodes>
          <leafValues>
            -7.3741632699966431e-01 4.8247390985488892e-01
            4.1971048712730408e-01 -2.0021289587020874e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 153 -4.9728769809007645e-03 2 -1 154
            -3.4751880913972855e-02 -2 -3 155 -8.7171117775142193e-04</internalNodes>
          <leafValues>
            3.7631350755691528e-01 -4.4797790050506592e-01
            -6.9995099306106567e-01 1.5640909969806671e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 156 -3.3666230738162994e-03 -1 2 157
            -2.1378830075263977e-02 -2 -3 158 -1.1869249865412712e-02</internalNodes>
          <leafValues>
            -6.7721921205520630e-01 3.3951529860496521e-01
            5.4050672054290771e-01 -2.4071580171585083e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 159 -4.4268160127103329e-03 2 -1 160
            4.1405398398637772e-02 -2 -3 161 -3.7884410470724106e-02</internalNodes>
          <leafValues>
            -7.3965507745742798e-01 8.2905638217926025e-01
            1.7030739784240723e-01 -2.4498699605464935e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 162 3.7567419349215925e-04 -1 2 163
            -3.7140299100428820e-03 -2 -3 164 -6.1806719750165939e-03</internalNodes>
          <leafValues>
            -4.5103698968887329e-01 3.8348129391670227e-01
            3.6097520589828491e-01 -2.0644439756870270e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 165 -1.2373559875413775e-03 -1 2 166
            -2.1339580416679382e-03 -2 -3 167 2.8985869139432907e-03</internalNodes>
          <leafValues>
            -5.8166879415512085e-01 4.1669690608978271e-01
            -2.4721260368824005e-01 3.5056841373443604e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 168 -4.4636861421167850e-03 0 -1 169
            1.6411510296165943e-03 -2 -3 170 -7.3051019571721554e-03</internalNodes>
          <leafValues>
            3.5625410079956055e-01 -4.1040098667144775e-01
            2.0216129720211029e-01 -3.4234520792961121e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>13</maxWeakCount>
      <stageThreshold>-1.4762729406356812e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 2 171 -5.1942609250545502e-02 0 -1 172
            -4.7268528491258621e-02 -2 -3 173 -7.8969672322273254e-03</internalNodes>
          <leafValues>
            8.8198930025100708e-01 6.4829237759113312e-02
            8.8662758469581604e-02 -5.9007811546325684e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 174 9.0199249098077416e-04 2 -1 175
            -1.7289820313453674e-01 -2 -3 176 -2.3374119773507118e-03</internalNodes>
          <leafValues>
            5.9040898084640503e-01 -5.2029031515121460e-01
            5.2981728315353394e-01 -1.4985850453376770e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 177 -1.7534950748085976e-02 -1 2 178
            5.8875310060102493e-05 -2 -3 179 -3.2241028547286987e-01</internalNodes>
          <leafValues>
            5.3269028663635254e-01 -4.5709720253944397e-01
            5.7380169630050659e-01 -1.2866480648517609e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 180 8.3220787928439677e-05 0 -1 181
            -1.1180160072399303e-04 -2 -3 182 -1.0344980284571648e-02</internalNodes>
          <leafValues>
            9.0006209909915924e-02 -5.6352388858795166e-01
            6.3273417949676514e-01 5.0064269453287125e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 183 -9.4440882094204426e-04 2 -1 184
            -3.7474210839718580e-03 -2 -3 185 4.0574651211500168e-03</internalNodes>
          <leafValues>
            4.4386640191078186e-01 -3.4999918937683105e-01
            -4.5298218727111816e-01 3.0920198559761047e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 186 5.5205920943990350e-05 0 -1 187
            -7.5678288936614990e-02 -2 -3 188 -3.0975368618965149e-01</internalNodes>
          <leafValues>
            3.5544091463088989e-01 -3.6047360301017761e-01
            -6.4954018592834473e-01 3.0679279565811157e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 189 -7.9595847637392581e-05 0 -1 190
            4.0613119490444660e-03 -2 -3 191 4.3240871280431747e-02</internalNodes>
          <leafValues>
            3.3850470185279846e-01 -5.3271901607513428e-01
            -3.2592329382896423e-01 5.5076271295547485e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 192 -6.7015928216278553e-03 -1 2 193
            -1.0451120324432850e-03 -2 -3 194 8.3967261016368866e-03</internalNodes>
          <leafValues>
            5.0109171867370605e-01 -5.8881980180740356e-01
            -9.5237597823143005e-02 5.6516999006271362e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 195 -6.5531006839592010e-05 0 -1 196
            7.8218057751655579e-05 -2 -3 197 3.2988168299198151e-02</internalNodes>
          <leafValues>
            -4.6556711196899414e-01 5.4509781301021576e-02
            3.5248789191246033e-01 -5.2722948789596558e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 198 -1.4161449857056141e-02 2 -1 199
            3.1500440090894699e-02 -2 -3 200 -2.1956730633974075e-03</internalNodes>
          <leafValues>
            3.6811780929565430e-01 5.2040421962738037e-01
            1.1603529751300812e-01 -3.0985280871391296e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 201 -4.0099889039993286e-02 -1 2 202
            -3.2569639384746552e-02 -2 -3 203 -4.2014168575406075e-03</internalNodes>
          <leafValues>
            -4.5146378874778748e-01 -6.4392048120498657e-01
            -8.2594501972198486e-01 1.9259540736675262e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 204 2.0385689567774534e-03 0 -1 205
            -1.6212540213018656e-03 -2 -3 206 -8.6220083758234978e-03</internalNodes>
          <leafValues>
            -3.7723371386528015e-01 3.3918830752372742e-01
            4.8986920714378357e-01 -2.7532070875167847e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 207 9.2185800895094872e-05 2 -1 208
            -7.1932889113668352e-05 -2 -3 209 4.4952900498174131e-04</internalNodes>
          <leafValues>
            2.4223749339580536e-01 -4.2189198732376099e-01
            2.9407840967178345e-01 -4.4028049707412720e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>15</maxWeakCount>
      <stageThreshold>-1.4963719844818115e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 2 210 -1.9638450816273689e-02 0 -1 211
            1.1364299803972244e-01 -2 -3 212 -1.0112149640917778e-02</internalNodes>
          <leafValues>
            -3.2444450259208679e-01 7.4602019786834717e-01
            3.3333331346511841e-01 -5.6435650587081909e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 213 1.2130879797041416e-02 2 -1 214
            -1.5958850085735321e-01 -2 -3 215 -2.3524949792772532e-03</internalNodes>
          <leafValues>
            7.2214919328689575e-01 -3.9274591207504272e-01
            5.6152492761611938e-01 -1.3768480718135834e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 216 -4.1118920780718327e-03 -1 2 217
            -1.7832900583744049e-01 -2 -3 218 -7.8500732779502869e-03</internalNodes>
          <leafValues>
            6.3556081056594849e-01 3.3373141288757324e-01
            3.9536771178245544e-01 -3.3380430936813354e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 219 -4.6880490117473528e-05 0 -1 220
            5.2934719860786572e-05 -2 -3 221 2.0851430235779844e-05</internalNodes>
          <leafValues>
            -6.6118270158767700e-01 -4.8232190310955048e-02
            -9.8838359117507935e-02 4.4528418779373169e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 222 -1.8425289541482925e-02 -1 2 223
            -7.6133902184665203e-03 -2 -3 224 -6.0353721491992474e-03</internalNodes>
          <leafValues>
            -6.5690898895263672e-01 5.3413677215576172e-01
            3.6171048879623413e-01 -2.0478430390357971e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 225 4.3712720071198419e-05 0 -1 226
            -7.8823999501764774e-04 -2 -3 227 -4.5693209394812584e-03</internalNodes>
          <leafValues>
            -4.5326828956604004e-01 3.5517698526382446e-01
            6.1721032857894897e-01 -2.9707700014114380e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 228 -3.8058571517467499e-02 0 -1 229
            -1.1797689646482468e-01 -2 -3 230 4.6841651201248169e-03</internalNodes>
          <leafValues>
            3.5003998875617981e-01 -2.7257668972015381e-01
            -3.2559171319007874e-01 3.7737470865249634e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 231 -2.6372840511612594e-04 0 -1 232
            6.2580420635640621e-03 -2 -3 233 5.6767999922158197e-05</internalNodes>
          <leafValues>
            3.7421739101409912e-01 -5.8926701545715332e-01
            -4.8859021067619324e-01 -1.8623730167746544e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 234 9.2742107808589935e-03 2 -1 235
            -3.8514519110321999e-03 -2 -3 236 -5.3287498303689063e-05</internalNodes>
          <leafValues>
            3.0933541059494019e-01 -3.4513729810714722e-01
            5.2340328693389893e-01 -9.1159403324127197e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 237 9.8315975628793240e-04 2 -1 238
            8.2858657697215676e-04 -2 -3 239 1.1229789815843105e-02</internalNodes>
          <leafValues>
            -5.0185352563858032e-01 -3.0529549717903137e-01
            2.6219210028648376e-01 -4.7969821095466614e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 240 -1.0327639989554882e-02 -1 2 241
            -6.9197742268443108e-03 -2 -3 242 -5.0027170218527317e-03</internalNodes>
          <leafValues>
            -5.6315082311630249e-01 3.1225070357322693e-01
            1.7820779979228973e-01 -3.0091148614883423e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 243 -1.1156810069223866e-04 -1 2 244
            4.2464961297810078e-03 -2 -3 245 -4.7280951548600569e-05</internalNodes>
          <leafValues>
            1.8883679807186127e-01 -4.0101578831672668e-01
            4.6505901217460632e-01 -2.9863640666007996e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 246 -1.8891280051320791e-03 -1 2 247
            -5.8536308642942458e-05 -2 -3 248 2.0671950551331975e-05</internalNodes>
          <leafValues>
            5.6963747739791870e-01 1.8008249998092651e-01
            -5.8659601211547852e-01 -5.4875258356332779e-03</leafValues></_>
        <_>
          <internalNodes>
            0 1 249 -1.1267509544268250e-03 2 -1 250
            2.1378440782427788e-02 -2 -3 251 -1.2546040117740631e-02</internalNodes>
          <leafValues>
            -4.0261599421501160e-01 3.9230358600616455e-01
            4.9474561214447021e-01 -1.7322529852390289e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 252 -7.2257901774719357e-04 2 -1 253
            6.4563672058284283e-03 -2 -3 254 4.9086650833487511e-03</internalNodes>
          <leafValues>
            -3.0380329489707947e-01 4.7173491120338440e-01
            -1.6380549967288971e-01 3.7708491086959839e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>19</maxWeakCount>
      <stageThreshold>-1.5243699550628662e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            2 1 255 -7.2617560625076294e-02 0 -1 256
            -6.9059380330145359e-03 -2 -3 257 2.1727949380874634e-01</internalNodes>
          <leafValues>
            2.6602798700332642e-01 -4.9325171113014221e-01
            -1.0769230127334595e-01 8.2661122083663940e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 258 -2.0319509785622358e-03 0 -1 259
            2.8931589797139168e-02 -2 -3 260 -4.6076569706201553e-03</internalNodes>
          <leafValues>
            -3.7963140755891800e-02 8.0230438709259033e-01
            4.2468398809432983e-01 -2.9379379749298096e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 261 6.9408868439495564e-03 0 -1 262
            -5.9231962077319622e-03 -2 -3 263 5.1128160208463669e-02</internalNodes>
          <leafValues>
            4.1737049818038940e-01 -2.5552588701248169e-01
            -3.8619861006736755e-01 4.7076860070228577e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 264 1.5201330184936523e-02 -1 2 265
            -1.8096340820193291e-02 -2 -3 266 7.9378951340913773e-05</internalNodes>
          <leafValues>
            5.4354798793792725e-01 2.6651141047477722e-01
            -4.3927749991416931e-01 2.5831260718405247e-03</leafValues></_>
        <_>
          <internalNodes>
            0 1 267 -5.3462558425962925e-03 -1 2 268
            -6.9701080210506916e-03 -2 -3 269 8.4738981968257576e-05</internalNodes>
          <leafValues>
            -6.6308969259262085e-01 -7.0310682058334351e-01
            -1.7880809307098389e-01 2.5993299484252930e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 270 -2.8513800352811813e-03 2 -1 271
            2.2954840678721666e-03 -2 -3 272 -3.5036220215260983e-03</internalNodes>
          <leafValues>
            4.5053678750991821e-01 3.0560511350631714e-01
            1.5040870010852814e-01 -3.3283078670501709e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 273 -6.9570228457450867e-02 0 -1 274
            5.9261121350573376e-05 -2 -3 275 -5.9058349579572678e-02</internalNodes>
          <leafValues>
            -3.6899719387292862e-02 4.0927308797836304e-01
            1.3826370239257812e-01 -3.8214409351348877e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 276 -8.9645627886056900e-03 -1 2 277
            4.9211819714400917e-05 -2 -3 278 9.9640293046832085e-03</internalNodes>
          <leafValues>
            -5.8134728670120239e-01 -1.8481740355491638e-01
            8.7685473263263702e-02 5.8509802818298340e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 279 -1.9302699714899063e-02 -1 2 280
            -4.3869198998436332e-04 -2 -3 281 6.5669846662785858e-05</internalNodes>
          <leafValues>
            5.3263461589813232e-01 2.8891131281852722e-01
            -3.3493599295616150e-01 5.9566751122474670e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 282 -2.0224519073963165e-02 -1 2 283
            8.7082196841947734e-05 -2 -3 284 -1.6202719882130623e-02</internalNodes>
          <leafValues>
            -6.5536081790924072e-01 -1.2211789935827255e-01
            -4.7076839208602905e-01 3.0990770459175110e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 285 4.4353529810905457e-03 -1 2 286
            -9.0544822160154581e-04 -2 -3 287 -1.4297979651018977e-03</internalNodes>
          <leafValues>
            -5.4039931297302246e-01 4.2878800630569458e-01
            2.2322739660739899e-01 -1.8194420635700226e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 288 3.2359519973397255e-03 0 -1 289
            1.0716189717641100e-04 -2 -3 290 -5.8802281273528934e-04</internalNodes>
          <leafValues>
            -2.9218220710754395e-01 1.3910460472106934e-01
            -4.6926081180572510e-01 3.8085499405860901e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 291 -9.0546347200870514e-03 -1 2 292
            -8.6048766970634460e-03 -2 -3 293 -1.2719300575554371e-03</internalNodes>
          <leafValues>
            -5.0426542758941650e-01 -2.7559030055999756e-01
            3.6022108793258667e-01 -2.6484970003366470e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 294 -3.9098240085877478e-04 -1 2 295
            -3.6405251012183726e-04 -2 -3 296 -6.6685711499303579e-04</internalNodes>
          <leafValues>
            2.6651731133460999e-01 1.4721649885177612e-01
            -4.9719738960266113e-01 -6.1579849570989609e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 297 -2.4845570325851440e-02 -1 2 298
            -1.5436399728059769e-02 -2 -3 299 -5.6572312116622925e-01</internalNodes>
          <leafValues>
            -7.0820981264114380e-01 -4.7206890583038330e-01
            6.3965231180191040e-01 5.2069328725337982e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 300 -5.7480141520500183e-02 -1 2 301
            -1.4613820239901543e-02 -2 -3 302 -3.3993738889694214e-01</internalNodes>
          <leafValues>
            2.9297390580177307e-01 6.0129672288894653e-01
            1.9041299819946289e-02 -3.3254599571228027e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 303 -3.1427140347659588e-03 0 -1 304
            2.1966299973428249e-03 -2 -3 305 -2.4858590215444565e-02</internalNodes>
          <leafValues>
            -2.2972729802131653e-01 2.2367340326309204e-01
            -5.6212967634201050e-01 3.9542859792709351e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 306 -1.6135630430653691e-03 2 -1 307
            1.1416019697207958e-04 -2 -3 308 1.3170539750717580e-04</internalNodes>
          <leafValues>
            -4.8256790637969971e-01 2.6877319812774658e-01
            -3.9078921079635620e-01 1.7153440415859222e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 309 -8.5256207967177033e-05 -1 2 310
            6.4925159676931798e-05 -2 -3 311 -1.2689639814198017e-02</internalNodes>
          <leafValues>
            2.1754570305347443e-01 -4.7468620538711548e-01
            -6.6538578271865845e-01 1.2347090244293213e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>27</maxWeakCount>
      <stageThreshold>-1.3592849969863892e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            2 1 312 -2.9844639822840691e-02 0 -1 313
            -4.5487660169601440e-01 -2 -3 314 2.7445149607956409e-03</internalNodes>
          <leafValues>
            3.9222040772438049e-01 -3.9314880967140198e-01
            -1.5923570096492767e-01 8.2696700096130371e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 315 -1.0584670118987560e-02 0 -1 316
            -1.6308380290865898e-02 -2 -3 317 -4.8787441104650497e-02</internalNodes>
          <leafValues>
            4.5954689383506775e-01 -2.1620120108127594e-01
            7.5103652477264404e-01 7.4557967483997345e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 318 -2.9621229041367769e-03 2 -1 319
            1.7300529405474663e-02 -2 -3 320 -1.6731169074773788e-02</internalNodes>
          <leafValues>
            -2.4452270567417145e-01 -3.3090409636497498e-01
            5.3751850128173828e-01 2.9153820127248764e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 321 1.2326180003583431e-02 -1 2 322
            5.4928299039602280e-02 -2 -3 323 2.7763319667428732e-03</internalNodes>
          <leafValues>
            -5.4824811220169067e-01 -2.1952770650386810e-01
            3.6463689059019089e-02 5.0633782148361206e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 324 -4.5116998255252838e-02 2 -1 325
            1.1207940056920052e-02 -2 -3 326 -5.7006389833986759e-03</internalNodes>
          <leafValues>
            4.2339310050010681e-01 3.9984008669853210e-01
            -5.9729182720184326e-01 -9.8557651042938232e-02</leafValues></_>
        <_>
          <internalNodes>
            1 2 327 -5.3951311856508255e-03 0 -1 328
            7.8587066382169724e-03 -2 -3 329 1.0666639544069767e-02</internalNodes>
          <leafValues>
            3.4734690189361572e-01 -4.7281920909881592e-01
            -2.3315669596195221e-01 2.4360010027885437e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 330 2.8001810424029827e-03 0 -1 331
            -7.9198479652404785e-03 -2 -3 332 -2.3832279257476330e-03</internalNodes>
          <leafValues>
            -4.8354551196098328e-01 1.8321120738983154e-01
            3.2168481498956680e-02 -5.0476258993148804e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 333 -9.7674019634723663e-03 -1 2 334
            -1.3897259719669819e-02 -2 -3 335 -6.4803068526089191e-03</internalNodes>
          <leafValues>
            -7.4415212869644165e-01 4.5425128936767578e-01
            4.8292869329452515e-01 -1.0258570313453674e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 336 9.4482619315385818e-03 -1 2 337
            -7.0351187605410814e-04 -2 -3 338 -4.2770579457283020e-03</internalNodes>
          <leafValues>
            -5.3326022624969482e-01 2.9435831308364868e-01
            1.5501999855041504e-01 -3.0867969989776611e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 339 5.8752358891069889e-03 2 -1 340
            9.5629561692476273e-03 -2 -3 341 -6.8425266363192350e-05</internalNodes>
          <leafValues>
            -6.0491317510604858e-01 4.4039881229400635e-01
            1.0206270217895508e-01 -2.5624030828475952e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 342 5.4002371616661549e-03 2 -1 343
            2.9745819047093391e-03 -2 -3 344 -2.5536341127008200e-03</internalNodes>
          <leafValues>
            4.5371580123901367e-01 -6.0967987775802612e-01
            2.2111609578132629e-01 -1.2801170349121094e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 345 4.0425839833915234e-03 2 -1 346
            7.6407291926443577e-03 -2 -3 347 -1.0939979692921042e-03</internalNodes>
          <leafValues>
            -1.9264020025730133e-01 6.1178821325302124e-01
            -3.7973681092262268e-01 1.6438940167427063e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 348 -1.1377089685993269e-04 0 -1 349
            5.2979402244091034e-03 -2 -3 350 2.9510098975151777e-03</internalNodes>
          <leafValues>
            -2.7770480141043663e-02 4.3019628524780273e-01
            -3.7912338972091675e-01 1.0130850225687027e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 351 6.3235480338335037e-03 -1 2 352
            3.9955950342118740e-03 -2 -3 353 -5.3595582721754909e-04</internalNodes>
          <leafValues>
            4.0413460135459900e-01 -1.5097740292549133e-01
            5.9522801637649536e-01 -3.4380171447992325e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 354 3.6193430423736572e-03 2 -1 355
            3.4626820124685764e-03 -2 -3 356 2.9030859470367432e-02</internalNodes>
          <leafValues>
            -7.4454522132873535e-01 2.8504610061645508e-01
            -1.8565440177917480e-01 1.5829989314079285e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 357 6.0747697716578841e-04 2 -1 358
            9.4140451401472092e-03 -2 -3 359 -2.2230610251426697e-02</internalNodes>
          <leafValues>
            -3.3788970112800598e-01 -3.6750578880310059e-01
            -6.4205718040466309e-01 1.7526410520076752e-01</leafValues></_>
        <_>
          <internalNodes>
            2 1 360 -4.6881791204214096e-03 0 -1 361
            -3.9184167981147766e-03 -2 -3 362 -6.3269808888435364e-03</internalNodes>
          <leafValues>
            1.6476869583129883e-01 -2.2729560732841492e-01
            5.7388627529144287e-01 5.7931281626224518e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 363 -3.7428940413519740e-04 2 -1 364
            2.8672320768237114e-03 -2 -3 365 2.4337199283763766e-04</internalNodes>
          <leafValues>
            -3.5288140177726746e-01 -4.1419389843940735e-01
            2.0027640461921692e-01 -2.8263148665428162e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 366 -9.1555183753371239e-03 -1 2 367
            -1.2892490485683084e-03 -2 -3 368 -1.6453899443149567e-03</internalNodes>
          <leafValues>
            -5.4508739709854126e-01 2.5321239233016968e-01
            1.7635670304298401e-01 -2.3053619265556335e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 369 -7.6485536992549896e-02 2 -1 370
            3.8297360879369080e-04 -2 -3 371 -2.6448920834809542e-04</internalNodes>
          <leafValues>
            -7.0480287075042725e-01 2.2375050187110901e-01
            1.4251540601253510e-01 -2.4608950316905975e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 372 -7.9496540129184723e-03 -1 2 373
            -7.7398279681801796e-03 -2 -3 374 -1.0467980057001114e-02</internalNodes>
          <leafValues>
            -4.2123699188232422e-01 -4.6475729346275330e-01
            -4.7312980890274048e-01 1.3598929345607758e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 375 9.4248689711093903e-03 2 -1 376
            -3.7210211157798767e-03 -2 -3 377 -1.6539100557565689e-02</internalNodes>
          <leafValues>
            3.5587531328201294e-01 -1.5899239480495453e-01
            -6.1142671108245850e-01 3.3778318762779236e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 378 1.8258139491081238e-02 -1 2 379
            -6.1498139984905720e-03 -2 -3 380 1.4396630227565765e-02</internalNodes>
          <leafValues>
            -7.0120972394943237e-01 3.8414189219474792e-01
            2.2873559966683388e-02 -4.8029011487960815e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 381 -4.8927508294582367e-02 -1 2 382
            -4.9874751130118966e-04 -2 -3 383 -1.2338399887084961e-02</internalNodes>
          <leafValues>
            -1.2219530344009399e-01 4.4899681210517883e-01
            5.8306622505187988e-01 -1.5592460334300995e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 384 4.9237860366702080e-03 -1 2 385
            6.4515617850702256e-05 -2 -3 386 -9.0754460543394089e-03</internalNodes>
          <leafValues>
            5.7889437675476074e-01 -2.2252050042152405e-01
            2.5118181109428406e-01 -1.1915980279445648e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 387 -2.2913129068911076e-03 2 -1 388
            -1.1618229560554028e-02 -2 -3 389 -2.6231290772557259e-02</internalNodes>
          <leafValues>
            2.0203049480915070e-01 -2.4990449845790863e-01
            -7.2858989238739014e-01 2.2483369708061218e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 390 2.1525719785131514e-04 2 -1 391
            5.4147760383784771e-03 -2 -3 392 -6.8281739950180054e-03</internalNodes>
          <leafValues>
            -3.0237621068954468e-01 -3.4467801451683044e-01
            -5.1470118761062622e-01 1.8762029707431793e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>29</maxWeakCount>
      <stageThreshold>-1.3664239645004272e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 2 393 8.8577903807163239e-03 0 -1 394
            2.2660400718450546e-03 -2 -3 395 1.5509200282394886e-02</internalNodes>
          <leafValues>
            -3.6197811365127563e-01 3.4535628557205200e-01
            -2.2814500331878662e-01 8.0521601438522339e-01</leafValues></_>
        <_>
          <internalNodes>
            1 2 396 1.9730629399418831e-02 0 -1 397
            -5.2804131060838699e-02 -2 -3 398 -3.4123551100492477e-02</internalNodes>
          <leafValues>
            2.2162230312824249e-01 -2.6307260990142822e-01
            8.7687742710113525e-01 1.5147949755191803e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 399 -4.4995918869972229e-03 -1 2 400
            -3.8060150109231472e-03 -2 -3 401 -6.5935899328906089e-05</internalNodes>
          <leafValues>
            -5.1520478725433350e-01 3.1563198566436768e-01
            1.1052650213241577e-01 -3.0016160011291504e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 402 9.5838904380798340e-03 2 -1 403
            4.2877299711108208e-03 -2 -3 404 3.2141651026904583e-03</internalNodes>
          <leafValues>
            5.2808177471160889e-01 -6.3694041967391968e-01
            3.5910170525312424e-02 -5.4334390163421631e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 405 -7.9250690760090947e-04 2 -1 406
            -1.5514569822698832e-03 -2 -3 407 -1.7790550366044044e-02</internalNodes>
          <leafValues>
            -4.7867339849472046e-01 -9.1462276875972748e-02
            4.5612779259681702e-01 1.0628259740769863e-02</leafValues></_>
        <_>
          <internalNodes>
            2 1 408 -2.5881261099129915e-03 0 -1 409
            -2.7412150520831347e-03 -2 -3 410 4.4753181282430887e-04</internalNodes>
          <leafValues>
            1.6198949515819550e-01 -2.9113239049911499e-01
            -2.8482219576835632e-01 3.3902090787887573e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 411 -3.6593680270016193e-03 2 -1 412
            2.4432500358670950e-03 -2 -3 413 -1.3546410016715527e-02</internalNodes>
          <leafVal
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

