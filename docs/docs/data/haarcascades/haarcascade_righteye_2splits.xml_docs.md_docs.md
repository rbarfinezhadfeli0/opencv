# Documentation for `docs/data/haarcascades/haarcascade_righteye_2splits.xml_docs.md`

## File Metadata

- **Full Path**: `docs/data/haarcascades/haarcascade_righteye_2splits.xml_docs.md`
- **File Name**: `haarcascade_righteye_2splits.xml_docs.md`
- **File Size**: 51,073 bytes
- **File Type**: .md
- **Link to Source**: [docs/data/haarcascades/haarcascade_righteye_2splits.xml_docs.md](../../../docs/data/haarcascades/haarcascade_righteye_2splits.xml_docs.md)

## Purpose and Role

This file is located in the `docs/data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `data/haarcascades/haarcascade_righteye_2splits.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_righteye_2splits.xml`
- **File Name**: `haarcascade_righteye_2splits.xml`
- **File Size**: 196,170 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_righteye_2splits.xml](../../data/haarcascades/haarcascade_righteye_2splits.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
    Tree-based 20x20 right eye detector.
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
    <maxWeakCount>34</maxWeakCount></stageParams>
  <featureParams>
    <maxCatCount>0</maxCatCount></featureParams>
  <stageNum>20</stageNum>
  <stages>
    <_>
      <maxWeakCount>5</maxWeakCount>
      <stageThreshold>-2.2325520515441895e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 0 -4.8210550099611282e-02 -1 -2 1
            -4.1576199233531952e-02</internalNodes>
          <leafValues>
            -8.6140447854995728e-01 9.1769057512283325e-01
            -2.1284009516239166e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 2 9.3528684228658676e-03 -1 -2 3 -2.2144919785205275e-04</internalNodes>
          <leafValues>
            -6.9785767793655396e-01 7.9523372650146484e-01
            -4.8948091268539429e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 4 -2.1853350102901459e-02 -1 -2 5 9.9672928452491760e-02</internalNodes>
          <leafValues>
            7.0574641227722168e-01 -7.0666241645812988e-01
            7.9210978746414185e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 6 -2.1664820611476898e-02 -1 -2 7
            -7.5680727604776621e-04</internalNodes>
          <leafValues>
            -6.0898607969284058e-01 7.1685701608657837e-01
            -3.0464568734169006e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 8 -1.3333049602806568e-02 -1 -2 9 9.2925298959016800e-03</internalNodes>
          <leafValues>
            -4.6844691038131714e-01 6.4235931634902954e-01
            -5.1180428266525269e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>5</maxWeakCount>
      <stageThreshold>-2.1598019599914551e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 1 10 -3.3948719501495361e-01 -1 -2 11
            -1.3672479987144470e-01</internalNodes>
          <leafValues>
            7.7913260459899902e-01 2.6421278715133667e-01
            -8.7910091876983643e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 12 3.1394500285387039e-02 -1 -2 13
            -1.0828140191733837e-02</internalNodes>
          <leafValues>
            -6.9956701993942261e-01 7.6504492759704590e-01
            -4.3719211220741272e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 14 -4.2506768368184566e-03 -1 -2 15
            -2.2675469517707825e-02</internalNodes>
          <leafValues>
            -5.7561582326889038e-01 7.4080592393875122e-01
            -3.6677250266075134e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 16 3.9161480963230133e-02 -1 -2 17
            -3.1934089493006468e-03</internalNodes>
          <leafValues>
            6.4045161008834839e-01 1.6047589480876923e-01
            -7.1010977029800415e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 18 2.5321990251541138e-02 -1 -2 19
            7.7583367237821221e-04</internalNodes>
          <leafValues>
            4.9574860930442810e-01 -7.1737897396087646e-01
            -1.8581770360469818e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>8</maxWeakCount>
      <stageThreshold>-2.3451159000396729e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 20 -2.6554059982299805e-01 -1 -2 21
            -2.2532779723405838e-02</internalNodes>
          <leafValues>
            -8.4712451696395874e-01 8.7977188825607300e-01
            -3.3394691348075867e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 22 8.5310067515820265e-04 -1 -2 23
            1.5820249973330647e-04</internalNodes>
          <leafValues>
            -8.2032448053359985e-01 -7.5176358222961426e-01
            6.7769712209701538e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 24 -1.0837490117410198e-04 -1 -2 25
            2.6810260023921728e-03</internalNodes>
          <leafValues>
            -8.3314001560211182e-01 5.3844749927520752e-01
            -7.6534157991409302e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 26 8.5202371701598167e-04 -1 -2 27
            -1.2241739779710770e-02</internalNodes>
          <leafValues>
            -7.7514898777008057e-01 6.3240152597427368e-01
            -6.3395208120346069e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 28 6.2314196838997304e-05 -1 -2 29
            -7.1911108493804932e-01</internalNodes>
          <leafValues>
            4.4290411472320557e-01 8.0135929584503174e-01
            -5.3431099653244019e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 30 -2.4280339479446411e-02 -1 -2 31
            3.4558640327304602e-03</internalNodes>
          <leafValues>
            -6.7797917127609253e-01 4.9030610918998718e-01
            -8.8447982072830200e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 32 -6.2993327446747571e-05 -1 -2 33
            -4.6443562023341656e-03</internalNodes>
          <leafValues>
            -5.7883417606353760e-01 -8.5878807306289673e-01
            5.2454602718353271e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 34 -4.0299328247783706e-05 -1 -2 35
            -3.7485519424080849e-03</internalNodes>
          <leafValues>
            -5.2713459730148315e-01 -8.5626190900802612e-01
            4.8944610357284546e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>10</maxWeakCount>
      <stageThreshold>-2.3431489467620850e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 1 36 -3.8377079367637634e-01 -1 -2 37
            -1.3837030529975891e-01</internalNodes>
          <leafValues>
            7.1715021133422852e-01 3.4392359852790833e-01
            -7.9931277036666870e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 38 3.3107071067206562e-04 -1 -2 39
            -5.1273438148200512e-03</internalNodes>
          <leafValues>
            -6.8352431058883667e-01 5.8250617980957031e-01
            -4.0955001115798950e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 40 -2.6100680232048035e-02 -1 -2 41
            -1.0628979653120041e-03</internalNodes>
          <leafValues>
            -4.3713301420211792e-01 7.0680737495422363e-01
            -2.6817938685417175e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 42 -9.7854852676391602e-02 -1 -2 43
            -1.1829820275306702e-01</internalNodes>
          <leafValues>
            7.3940038681030273e-01 6.3814181089401245e-01
            -3.8721871376037598e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 44 -7.5409049168229103e-03 -1 -2 45
            2.6851659640669823e-03</internalNodes>
          <leafValues>
            -4.8803019523620605e-01 3.9083468914031982e-01
            -6.5561538934707642e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 46 1.6870240215212107e-03 -1 -2 47
            -3.8136160001158714e-03</internalNodes>
          <leafValues>
            -4.9891749024391174e-01 -6.6405588388442993e-01
            4.0650749206542969e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 48 2.0289309322834015e-03 -1 -2 49
            -7.6308869756758213e-03</internalNodes>
          <leafValues>
            -6.9989210367202759e-01 4.3206840753555298e-01
            -2.9664969444274902e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 50 -3.3815231290645897e-04 -1 -2 51
            7.5163291767239571e-03</internalNodes>
          <leafValues>
            -4.6808540821075439e-01 3.6521491408348083e-01
            -7.6014542579650879e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 52 6.1479508876800537e-02 -1 -2 53
            -4.6286579221487045e-02</internalNodes>
          <leafValues>
            5.6990629434585571e-01 2.2625060379505157e-01
            -4.5330780744552612e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 54 4.6903551556169987e-03 -1 -2 55
            1.8803169950842857e-03</internalNodes>
          <leafValues>
            -7.7286708354949951e-01 2.7349120378494263e-01
            -6.6667830944061279e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>8</maxWeakCount>
      <stageThreshold>-2.1268370151519775e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 56 -5.5420672893524170e-01 -1 -2 57
            -6.9329799152910709e-03</internalNodes>
          <leafValues>
            -6.0620260238647461e-01 7.8542029857635498e-01
            -3.5522121191024780e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 58 -2.1169960498809814e-02 -1 -2 59
            -6.7428398132324219e-01</internalNodes>
          <leafValues>
            5.2947688102722168e-01 4.6065220236778259e-01
            -7.0058208703994751e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 60 -4.2725078761577606e-02 -1 -2 61
            -1.0109329596161842e-02</internalNodes>
          <leafValues>
            -5.9904807806015015e-01 6.8109220266342163e-01
            -2.0731879770755768e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 62 6.5861130133271217e-03 -1 -2 63
            -7.6380418613553047e-03</internalNodes>
          <leafValues>
            -5.2420848608016968e-01 -7.0169782638549805e-01
            4.4100138545036316e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 64 -9.7681581974029541e-02 -1 -2 65
            1.0197360068559647e-02</internalNodes>
          <leafValues>
            5.7708740234375000e-01 -9.8518550395965576e-02
            -8.8111698627471924e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 66 -2.5724549777805805e-03 -1 -2 67
            2.6594230439513922e-03</internalNodes>
          <leafValues>
            -8.3233338594436646e-01 3.0995351076126099e-01
            -8.1609177589416504e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 68 -1.0042720241472125e-03 -1 -2 69
            2.6080000679939985e-03</internalNodes>
          <leafValues>
            -4.3558520078659058e-01 3.3566600084304810e-01
            -8.1889331340789795e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 70 4.9724509008228779e-03 -1 -2 71
            1.2243240140378475e-02</internalNodes>
          <leafValues>
            -7.7048182487487793e-01 2.2534200549125671e-01
            -6.8695551156997681e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>10</maxWeakCount>
      <stageThreshold>-2.0604379177093506e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 72 -5.7784929871559143e-02 -1 -2 73
            -1.7517809756100178e-03</internalNodes>
          <leafValues>
            -7.0516008138656616e-01 8.5655921697616577e-01
            -9.2403419315814972e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 74 -1.1522379703819752e-02 -1 -2 75
            -3.8323760963976383e-03</internalNodes>
          <leafValues>
            -4.2749640345573425e-01 7.5913530588150024e-01
            -1.0894049704074860e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 76 -8.0922387540340424e-02 -1 -2 77
            -6.2537011690437794e-03</internalNodes>
          <leafValues>
            -3.1364768743515015e-01 6.9995921850204468e-01
            -1.1805690079927444e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 78 -1.2227860093116760e-01 -1 -2 79
            -6.4168110489845276e-02</internalNodes>
          <leafValues>
            5.2072501182556152e-01 3.9272749423980713e-01
            -4.2194411158561707e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 80 -5.3712888620793819e-04 -1 -2 81
            -2.8175620827823877e-03</internalNodes>
          <leafValues>
            -4.9524548649787903e-01 4.1350141167640686e-01
            -3.8919278979301453e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 82 -3.6368549335747957e-03 -1 -2 83
            -1.3223909772932529e-03</internalNodes>
          <leafValues>
            6.7615020275115967e-01 4.3426999449729919e-01
            -3.7642130255699158e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 84 3.7143539520911872e-04 -1 -2 85
            -5.0255712121725082e-03</internalNodes>
          <leafValues>
            -5.5630880594253540e-01 -5.2328592538833618e-01
            3.4646821022033691e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 86 -9.2711612523999065e-05 -1 -2 87
            1.9847028888761997e-03</internalNodes>
          <leafValues>
            -4.9652668833732605e-01 3.3401641249656677e-01
            -6.2446892261505127e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 88 4.7203440219163895e-02 -1 -2 89
            -6.8562600063160062e-05</internalNodes>
          <leafValues>
            5.7562619447708130e-01 2.6172660291194916e-02
            -6.0849070549011230e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 90 7.5034219771623611e-03 -1 -2 91
            6.3834791071712971e-03</internalNodes>
          <leafValues>
            -6.8576759099960327e-01 -1.7312510311603546e-01
            3.8560429215431213e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>12</maxWeakCount>
      <stageThreshold>-2.3187489509582520e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 92 -1.5584450215101242e-02 -1 -2 93
            1.4557019807398319e-02</internalNodes>
          <leafValues>
            -6.6648960113525391e-01 -4.3745130300521851e-01
            7.2227817773818970e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 94 -5.7889888994395733e-03 -1 -2 95
            -8.1936769187450409e-02</internalNodes>
          <leafValues>
            -4.3183240294456482e-01 6.8467652797698975e-01
            -2.2546729445457458e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 96 -4.2995368130505085e-03 -1 -2 97
            -1.3736640103161335e-02</internalNodes>
          <leafValues>
            -5.2409631013870239e-01 6.1626207828521729e-01
            -3.5893160104751587e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 98 -4.8069912008941174e-03 -1 -2 99
            -7.7131099998950958e-02</internalNodes>
          <leafValues>
            -4.2382389307022095e-01 6.0599362850189209e-01
            -3.1555330753326416e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 100 4.4640208943746984e-04 -1 -2 101
            3.4841578453779221e-02</internalNodes>
          <leafValues>
            -4.9206110835075378e-01 -4.1017889976501465e-02
            6.1330878734588623e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 102 8.2969048526138067e-04 -1 -2 103
            -7.8510129242204130e-05</internalNodes>
          <leafValues>
            -4.5479419827461243e-01 4.0007328987121582e-01
            -2.0888769626617432e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 104 4.6054688282310963e-03 -1 -2 105
            -7.1904482319951057e-03</internalNodes>
          <leafValues>
            -6.7931377887725830e-01 4.7060671448707581e-01
            -1.4138610661029816e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 106 -5.5724480189383030e-03 -1 -2 107
            -7.0458237314596772e-04</internalNodes>
          <leafValues>
            -7.0525509119033813e-01 3.6097851395606995e-01
            -1.8361540138721466e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 108 1.8595060333609581e-02 -1 -2 109
            5.0072550773620605e-02</internalNodes>
          <leafValues>
            4.1765761375427246e-01 -4.1869449615478516e-01
            2.8186509013175964e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 110 -2.0355919376015663e-02 -1 -2 111
            -2.8686519712209702e-02</internalNodes>
          <leafValues>
            -3.6494150757789612e-01 -5.3867787122726440e-01
            3.4767881035804749e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 112 -7.1101690991781652e-05 -1 -2 113
            2.0686469506472349e-03</internalNodes>
          <leafValues>
            -4.0156790614128113e-01 3.2963660359382629e-01
            -7.0951050519943237e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 114 1.1430920567363501e-03 -1 -2 115
            -8.8636036962270737e-03</internalNodes>
          <leafValues>
            4.4172981381416321e-01 1.8426130712032318e-01
            -4.1275170445442200e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>15</maxWeakCount>
      <stageThreshold>-2.2203750610351562e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 116 -7.7637642621994019e-02 -1 -2 117
            -8.4830820560455322e-03</internalNodes>
          <leafValues>
            -4.9321529269218445e-01 7.8138542175292969e-01
            -3.6062291264533997e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 118 -1.7180460272356868e-03 -1 -2 119
            2.4740949273109436e-02</internalNodes>
          <leafValues>
            -4.7690048813819885e-01 -3.2420080900192261e-01
            5.9280002117156982e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 120 3.3028100151568651e-03 -1 -2 121
            -3.4622039645910263e-02</internalNodes>
          <leafValues>
            -5.3991597890853882e-01 5.2076727151870728e-01
            -3.3530798554420471e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 122 -7.1505777304992080e-04 -1 -2 123
            -9.0145105496048927e-03</internalNodes>
          <leafValues>
            -4.8981699347496033e-01 -7.7969801425933838e-01
            3.6586359143257141e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 124 -1.0250939521938562e-03 -1 -2 125
            -5.5693178437650204e-03</internalNodes>
          <leafValues>
            -4.6970510482788086e-01 -6.9695621728897095e-01
            3.5025438666343689e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 126 1.3235070509836078e-03 -1 -2 127
            -3.3737940248101950e-03</internalNodes>
          <leafValues>
            -4.4707980751991272e-01 -5.6195151805877686e-01
            3.1833809614181519e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 128 -6.4095242123585194e-05 -1 -2 129
            -2.7294119354337454e-03</internalNodes>
          <leafValues>
            -3.5473638772964478e-01 4.1285240650177002e-01
            -3.1416821479797363e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 130 6.3087652961257845e-05 -1 -2 131
            -1.5436099842190742e-02</internalNodes>
          <leafValues>
            -3.5946568846702576e-01 -6.1329078674316406e-01
            3.4301999211311340e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 132 -2.1025019232183695e-03 -1 -2 133
            -1.6849569976329803e-02</internalNodes>
          <leafValues>
            -7.6962250471115112e-01 3.6569809913635254e-01
            -2.1210379898548126e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 134 5.6847798987291753e-05 -1 -2 135
            5.9984489344060421e-03</internalNodes>
          <leafValues>
            -4.0466558933258057e-01 2.8503778576850891e-01
            -5.8756178617477417e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 136 6.1389962211251259e-03 -1 -2 137
            -2.8117469628341496e-04</internalNodes>
          <leafValues>
            -8.7189829349517822e-01 2.5182509422302246e-01
            -3.1868219375610352e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 138 -4.5429798774421215e-03 -1 -2 139
            -3.2167110592126846e-02</internalNodes>
          <leafValues>
            -3.6724218726158142e-01 -7.9481202363967896e-01
            2.8887200355529785e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 140 5.0912089645862579e-03 -1 -2 141
            -1.5173070132732391e-03</internalNodes>
          <leafValues>
            -7.1477490663528442e-01 4.4514629244804382e-01
            -9.5207341015338898e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 142 -6.0079508693888783e-04 -1 -2 143
            4.4868541881442070e-03</internalNodes>
          <leafValues>
            -3.6021450161933899e-01 2.8276360034942627e-01
            -7.2084128856658936e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 144 -3.7957848981022835e-03 -1 -2 145
            -9.1829998418688774e-03</internalNodes>
          <leafValues>
            -2.8717440366744995e-01 5.0479042530059814e-01
            -7.0781037211418152e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>17</maxWeakCount>
      <stageThreshold>-2.1757249832153320e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 146 -5.5760249495506287e-02 -1 -2 147
            -5.9436690062284470e-02</internalNodes>
          <leafValues>
            -5.5854648351669312e-01 6.8943697214126587e-01
            -3.7195080518722534e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 148 -5.4637178778648376e-02 -1 -2 149
            2.3608359694480896e-01</internalNodes>
          <leafValues>
            5.3040331602096558e-01 -4.7355309128761292e-01
            4.6322488784790039e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 150 -9.4560505822300911e-03 -1 -2 151
            -5.3182709962129593e-02</internalNodes>
          <leafValues>
            -3.2544779777526855e-01 6.3468569517135620e-01
            -2.8268361091613770e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 152 -1.0638199746608734e-02 -1 -2 153
            -2.1207019686698914e-02</internalNodes>
          <leafValues>
            -5.5776351690292358e-01 3.9049190282821655e-01
            -4.2111930251121521e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 154 -5.6731878430582583e-05 -1 -2 155
            -4.4976451317779720e-04</internalNodes>
          <leafValues>
            -4.1803309321403503e-01 3.7355789542198181e-01
            -3.9199641346931458e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 156 2.7574670966714621e-03 -1 -2 157
            2.5649419985711575e-03</internalNodes>
          <leafValues>
            -7.9104632139205933e-01 1.9258180260658264e-01
            -7.5344461202621460e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 158 -9.4359368085861206e-03 -1 -2 159
            1.4136210083961487e-03</internalNodes>
          <leafValues>
            4.4834750890731812e-01 -3.3878430724143982e-01
            4.4291919469833374e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 160 3.9976350963115692e-03 -1 -2 161
            -1.5278969658538699e-03</internalNodes>
          <leafValues>
            -6.6637581586837769e-01 3.1292399764060974e-01
            -2.8027990460395813e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 162 -3.2376639865105972e-05 -1 -2 163
            1.6323389718309045e-03</internalNodes>
          <leafValues>
            -4.6672090888023376e-01 2.7995559573173523e-01
            -6.1321508884429932e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 164 7.7096219174563885e-03 -1 -2 165
            -7.8599318861961365e-02</internalNodes>
          <leafValues>
            2.0352549850940704e-01 7.2726912796497345e-02
            -6.8677097558975220e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 166 -3.6581400781869888e-03 -1 -2 167
            -4.2612198740243912e-02</internalNodes>
          <leafValues>
            -6.8079459667205811e-01 -8.4551781415939331e-01
            1.5990570187568665e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 168 -4.8822778626345098e-04 -1 -2 169
            -4.6951142139732838e-03</internalNodes>
          <leafValues>
            -4.7945699095726013e-01 -8.2234281301498413e-01
            2.0431579649448395e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 170 6.1706348787993193e-05 -1 -2 171
            1.3809910044074059e-02</internalNodes>
          <leafValues>
            -3.1742820143699646e-01 3.0769300460815430e-01
            -4.3544968962669373e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 172 -4.2187729850411415e-03 -1 -2 173
            -3.9540808647871017e-03</internalNodes>
          <leafValues>
            6.2499982118606567e-01 1.3225209712982178e-01
            -3.9745101332664490e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 174 2.2203531116247177e-03 -1 -2 175
            6.2806582718621939e-05</internalNodes>
          <leafValues>
            -6.0045331716537476e-01 -2.2429980337619781e-01
            2.9768520593643188e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 176 2.3292789701372385e-03 -1 -2 177
            -5.3711822256445885e-03</internalNodes>
          <leafValues>
            -7.5982081890106201e-01 2.6484918594360352e-01
            -2.6005539298057556e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 178 6.4782587287481874e-05 -1 -2 179
            7.6606678776443005e-03</internalNodes>
          <leafValues>
            -3.2119300961494446e-01 2.4176409840583801e-01
            -8.3822727203369141e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>19</maxWeakCount>
      <stageThreshold>-2.2618789672851562e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            1 0 180 -1.4848279766738415e-02 -1 -2 181
            -1.6066679963842034e-03</internalNodes>
          <leafValues>
            -5.3391128778457642e-01 7.6002711057662964e-01
            -2.1091739833354950e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 182 -1.5651920437812805e-01 -1 -2 183
            -5.5439779534935951e-03</internalNodes>
          <leafValues>
            -4.2818549275398254e-01 6.5620750188827515e-01
            -2.2949840128421783e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 184 -1.9448339939117432e-02 -1 -2 185
            7.6653067953884602e-03</internalNodes>
          <leafValues>
            -4.4212520122528076e-01 -3.3950591087341309e-01
            4.6587219834327698e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 186 -2.1142010390758514e-01 -1 -2 187
            -1.0628429800271988e-01</internalNodes>
          <leafValues>
            5.5007970333099365e-01 6.8280947208404541e-01
            -3.0987739562988281e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 188 -5.2653599530458450e-02 -1 -2 189
            -5.3522300731856376e-05</internalNodes>
          <leafValues>
            -3.4818819165229797e-01 5.0566762685775757e-01
            -2.5229519605636597e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 190 -5.7972650974988937e-03 -1 -2 191
            -3.7428899668157101e-03</internalNodes>
          <leafValues>
            3.0238011479377747e-01 2.2873230278491974e-01
            -4.8366579413414001e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 192 -5.2694038458866999e-05 -1 -2 193
            -1.1983739677816629e-03</internalNodes>
          <leafValues>
            -3.7988960742950439e-01 -6.7442452907562256e-01
            2.8611260652542114e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 194 2.2544799372553825e-02 -1 -2 195
            3.1783939339220524e-03</internalNodes>
          <leafValues>
            4.7565719485282898e-01 -2.8893348574638367e-01
            5.5509638786315918e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 196 3.4742769785225391e-03 -1 -2 197
            -8.1408787518739700e-03</internalNodes>
          <leafValues>
            -5.9826552867889404e-01 -5.5933791399002075e-01
            2.2349210083484650e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 198 -3.0238809995353222e-03 -1 -2 199
            -5.9159598313271999e-03</internalNodes>
          <leafValues>
            4.5917978882789612e-01 6.2234902381896973e-01
            -2.4468150734901428e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 200 2.3184430319815874e-03 -1 -2 201
            7.7198208309710026e-03</internalNodes>
          <leafValues>
            -6.0478079319000244e-01 2.1004509925842285e-01
            -6.4331281185150146e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 202 -5.5973320268094540e-03 -1 -2 203
            2.0320380281191319e-04</internalNodes>
          <leafValues>
            -7.1625810861587524e-01 -3.8018029928207397e-01
            2.1336899697780609e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 204 -3.8205389864742756e-03 -1 -2 205
            4.8883338458836079e-03</internalNodes>
          <leafValues>
            -3.5957258939743042e-01 2.6471930742263794e-01
            -5.8996689319610596e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 206 -1.3334590476006269e-03 -1 -2 207
            -1.5447080368176103e-03</internalNodes>
          <leafValues>
            3.2258489727973938e-01 3.6971050500869751e-01
            -3.1308570504188538e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 208 7.5150746852159500e-05 -1 -2 209
            -1.1108840117231011e-03</internalNodes>
          <leafValues>
            -3.4674531221389771e-01 -5.7477539777755737e-01
            2.9201140999794006e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 210 -1.6881119518075138e-04 -1 -2 211
            -1.2814450019504875e-04</internalNodes>
          <leafValues>
            -3.6041781306266785e-01 3.5043209791183472e-01
            -2.2014050185680389e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 212 1.9546970725059509e-02 -1 -2 213
            -1.1061180382966995e-02</internalNodes>
          <leafValues>
            4.1295918822288513e-01 2.5962719321250916e-01
            -3.4875950217247009e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 214 1.8147419905290008e-03 -1 -2 215
            -7.1724010631442070e-03</internalNodes>
          <leafValues>
            -5.2019888162612915e-01 2.7452668547630310e-01
            -2.6828849315643311e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 216 2.2158189676702023e-03 -1 -2 217
            -9.6856858581304550e-03</internalNodes>
          <leafValues>
            -5.7340908050537109e-01 -5.8028572797775269e-01
            1.8564410507678986e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>19</maxWeakCount>
      <stageThreshold>-2.0994780063629150e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 1 218 -1.2065219692885876e-02 -1 -2 219
            -4.9067771434783936e-01</internalNodes>
          <leafValues>
            6.1679571866989136e-01 1.4063939452171326e-01
            -5.5357742309570312e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 220 -6.6585717722773552e-03 -1 -2 221
            1.5827560797333717e-02</internalNodes>
          <leafValues>
            -5.1332288980484009e-01 -3.6301520466804504e-01
            4.3343341350555420e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 222 -1.4081180095672607e-02 -1 -2 223
            -1.2139449827373028e-02</internalNodes>
          <leafValues>
            5.4223722219467163e-01 4.4281288981437683e-01
            -3.4171119332313538e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 224 7.8055798076093197e-03 -1 -2 225
            -7.0759910158813000e-05</internalNodes>
          <leafValues>
            -4.8659759759902954e-01 3.4818679094314575e-01
            -3.2806739211082458e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 226 -1.8199630081653595e-02 -1 -2 227
            -2.5289389304816723e-03</internalNodes>
          <leafValues>
            5.6594151258468628e-01 1.1310060322284698e-01
            -4.0772381424903870e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 228 1.0156990028917789e-03 -1 -2 229
            2.9432660085149109e-04</internalNodes>
          <leafValues>
            -5.9842979907989502e-01 2.8439450263977051e-01
            -3.2190230488777161e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 230 2.0865290425717831e-03 -1 -2 231
            -1.7371569992974401e-03</internalNodes>
          <leafValues>
            -7.8285712003707886e-01 3.3585301041603088e-01
            -2.0582370460033417e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 232 -7.0026202592998743e-05 -1 -2 233
            -1.4891549944877625e-03</internalNodes>
          <leafValues>
            -3.9109349250793457e-01 -4.6953418850898743e-01
            2.7609241008758545e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 234 -1.1788429692387581e-02 -1 -2 235
            -1.5155089786276221e-03</internalNodes>
          <leafValues>
            -4.0114149451255798e-01 -7.4290478229522705e-01
            2.7695629000663757e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 236 6.8396717309951782e-02 -1 -2 237
            -7.6441407203674316e-02</internalNodes>
          <leafValues>
            4.5235648751258850e-01 4.2848169803619385e-01
            -3.1636309623718262e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 238 6.8310201168060303e-02 -1 -2 239
            -6.4508013427257538e-02</internalNodes>
          <leafValues>
            5.1404279470443726e-01 1.8081870675086975e-01
            -3.4217950701713562e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 240 -2.8335719835013151e-03 -1 -2 241
            -9.9732237868010998e-04</internalNodes>
          <leafValues>
            -6.9509768486022949e-01 -4.3724590539932251e-01
            2.0226080715656281e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 242 -2.2869910299777985e-01 -1 -2 243
            2.9855249449610710e-03</internalNodes>
          <leafValues>
            6.4662200212478638e-01 8.1149758771061897e-03
            -6.0210299491882324e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 244 -2.9535989742726088e-03 -1 -2 245
            -2.1225619129836559e-03</internalNodes>
          <leafValues>
            -7.2013127803802490e-01 5.0875622034072876e-01
            -5.9366609901189804e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 246 -2.9382819775491953e-03 -1 -2 247
            -5.8961478061974049e-03</internalNodes>
          <leafValues>
            3.9287531375885010e-01 4.1866040229797363e-01
            -2.5405511260032654e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 248 2.5730929337441921e-03 -1 -2 249
            1.6647739335894585e-02</internalNodes>
          <leafValues>
            -5.8707278966903687e-01 1.9208480417728424e-01
            -6.0388940572738647e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 250 2.4041840806603432e-03 -1 -2 251
            -9.0452830772846937e-04</internalNodes>
          <leafValues>
            -5.7192337512969971e-01 3.4860768914222717e-01
            -1.3049240410327911e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 252 4.0814210660755634e-03 -1 -2 253
            3.3811479806900024e-03</internalNodes>
          <leafValues>
            5.1778018474578857e-01 -6.3828541897237301e-03
            -6.1447817087173462e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 254 -2.7499340940266848e-03 -1 -2 255
            -4.8207710497081280e-03</internalNodes>
          <leafValues>
            -6.5407788753509521e-01 -6.0029619932174683e-01
            1.4374589920043945e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>21</maxWeakCount>
      <stageThreshold>-2.1254189014434814e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 1 256 7.9710120335221291e-03 -1 -2 257
            -9.7160867881029844e-04</internalNodes>
          <leafValues>
            -6.1992239952087402e-01 5.4877161979675293e-01
            -4.0606960654258728e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 258 -1.0945869609713554e-02 -1 -2 259
            -6.1174821108579636e-02</internalNodes>
          <leafValues>
            4.6936869621276855e-01 3.0570849776268005e-01
            -4.4459891319274902e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 260 -2.3100150283426046e-03 -1 -2 261
            -4.7585051506757736e-02</internalNodes>
          <leafValues>
            -3.7816441059112549e-01 4.8865839838981628e-01
            -2.9728868603706360e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 262 -2.5944279041141272e-03 -1 -2 263
            -3.9469371549785137e-03</internalNodes>
          <leafValues>
            -5.4405367374420166e-01 3.6382490396499634e-01
            -3.0469849705696106e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 264 3.1871569808572531e-04 -1 -2 265
            -2.6655721012502909e-03</internalNodes>
          <leafValues>
            -4.6822971105575562e-01 3.3131968975067139e-01
            -2.9918238520622253e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 266 -3.9534650743007660e-02 -1 -2 267
            -9.4085611635819077e-04</internalNodes>
          <leafValues>
            -3.5316830873489380e-01 4.4447100162506104e-01
            -1.1088660359382629e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 268 6.9526307925116271e-05 -1 -2 269
            -9.6976682543754578e-03</internalNodes>
          <leafValues>
            -3.9403268694877625e-01 5.7181888818740845e-01
            -1.6370950266718864e-02</leafValues></_>
        <_>
          <internalNodes>
            1 0 270 3.9469040930271149e-02 -1 -2 271
            -8.2811042666435242e-03</internalNodes>
          <leafValues>
            6.9152122735977173e-01 1.3349990546703339e-01
            -4.7064480185508728e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 272 -4.3219728395342827e-03 -1 -2 273
            -5.5436040274798870e-03</internalNodes>
          <leafValues>
            3.8239258527755737e-01 1.5645879507064819e-01
            -4.1088208556175232e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 274 -5.9953341406071559e-05 -1 -2 275
            -5.9089371934533119e-03</internalNodes>
          <leafValues>
            -3.9221799373626709e-01 -5.9083867073059082e-01
            2.7924481034278870e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 276 -4.4721391052007675e-02 -1 -2 277
            4.1267018765211105e-02</internalNodes>
          <leafValues>
            4.1454491019248962e-01 -3.2242009043693542e-01
            3.7849879264831543e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 278 5.6728709751041606e-05 -1 -2 279
            -6.2427870929241180e-02</internalNodes>
          <leafValues>
            -3.2228040695190430e-01 -5.9666448831558228e-01
            2.8915780782699585e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 280 -5.6994128972291946e-03 -1 -2 281
            7.5202910229563713e-03</internalNodes>
          <leafValues>
            3.7499341368675232e-01 -2.8132459521293640e-01
            5.0988858938217163e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 282 -3.3640549518167973e-03 -1 -2 283
            -6.8076648749411106e-03</internalNodes>
          <leafValues>
            -6.3978207111358643e-01 -7.3105818033218384e-01
            1.4475250244140625e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 284 1.2633459642529488e-02 -1 -2 285
            -2.9199919663369656e-03</internalNodes>
          <leafValues>
            -7.7725297212600708e-01 2.3258599638938904e-01
            -2.0490600168704987e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 286 -3.0582249164581299e-02 -1 -2 287
            -2.7796169742941856e-03</internalNodes>
          <leafValues>
            -6.5738821029663086e-01 -5.4888349771499634e-01
            1.3837890326976776e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 288 -7.6163080520927906e-03 -1 -2 289
            -1.8409560434520245e-03</internalNodes>
          <leafValues>
            -3.5912349820137024e-01 2.2404469549655914e-01
            -3.7881860136985779e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 290 -3.9200261235237122e-02 -1 -2 291
            -2.2543789818882942e-03</internalNodes>
          <leafValues>
            5.0090551376342773e-01 3.1364008784294128e-01
            -2.2131860256195068e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 292 2.3894659243524075e-03 -1 -2 293
            -1.0725490283221006e-03</internalNodes>
          <leafValues>
            -5.8699512481689453e-01 4.7141209244728088e-01
            -3.2570488750934601e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 294 8.9095337898470461e-05 -1 -2 295
            1.6920049674808979e-03</internalNodes>
          <leafValues>
            -3.0444309115409851e-01 3.0280891060829163e-01
            -3.8902729749679565e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 296 1.1784000322222710e-02 -1 -2 297
            3.9335917681455612e-03</internalNodes>
          <leafValues>
            -6.8993437290191650e-01 -6.7763939499855042e-02
            4.6499788761138916e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>22</maxWeakCount>
      <stageThreshold>-2.0614759922027588e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 1 298 1.1430840007960796e-02 -1 -2 299
            -3.2242920249700546e-02</internalNodes>
          <leafValues>
            -3.9274570345878601e-01 6.5568798780441284e-01
            -3.1068810820579529e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 300 -1.8382760463282466e-03 -1 -2 301
            -1.0764399915933609e-01</internalNodes>
          <leafValues>
            -4.0825068950653076e-01 4.3280079960823059e-01
            -4.2263451218605042e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 302 -2.3866090923547745e-03 -1 -2 303
            8.6586214601993561e-03</internalNodes>
          <leafValues>
            -4.6435201168060303e-01 -4.0673071146011353e-01
            4.1267868876457214e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 304 -1.6437229933217168e-03 -1 -2 305
            -9.8511137068271637e-02</internalNodes>
          <leafValues>
            -2.1344049274921417e-01 6.8432319164276123e-01
            -9.7035013139247894e-02</leafValues></_>
        <_>
          <internalNodes>
            0 1 306 4.4292360544204712e-03 -1 -2 307
            4.6966210938990116e-03</internalNodes>
          <leafValues>
            -3.9498910307884216e-01 -1.1345980316400528e-01
            4.9681991338729858e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 308 -8.8480701670050621e-03 -1 -2 309
            -6.7258379422128201e-03</internalNodes>
          <leafValues>
            -3.1293100118637085e-01 -6.1635792255401611e-01
            3.1764769554138184e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 310 2.0052040927112103e-03 -1 -2 311
            -1.3407340273261070e-02</internalNodes>
          <leafValues>
            3.1724271178245544e-01 1.9735060632228851e-01
            -3.7199181318283081e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 312 -4.4199679978191853e-03 -1 -2 313
            -3.2800938934087753e-02</internalNodes>
          <leafValues>
            -5.7164478302001953e-01 3.0599930882453918e-01
            -1.7397969961166382e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 314 4.9407979531679302e-05 -1 -2 315
            4.1550169698894024e-03</internalNodes>
          <leafValues>
            -2.8270530700683594e-01 2.9686808586120605e-01
            -4.8494309186935425e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 316 -7.5589967309497297e-05 -1 -2 317
            -3.2147730235010386e-03</internalNodes>
          <leafValues>
            -3.8531139492988586e-01 -6.3306808471679688e-01
            2.3434750735759735e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 318 1.6021779738366604e-03 -1 -2 319
            -1.9478019326925278e-02</internalNodes>
          <leafValues>
            -2.9579049348831177e-01 -4.9625208973884583e-01
            2.6092579960823059e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 320 -2.5193750858306885e-02 -1 -2 321
            -4.6487729996442795e-02</internalNodes>
          <leafValues>
            3.9384880661964417e-01 2.2168830037117004e-01
            -2.9691740870475769e-01</leafValues></_>
        <_>
          <internalNodes>
            1 0 322 4.3414267711341381e-03 -1 -2 323
            -2.4886759929358959e-03</internalNodes>
          <leafValues>
            -6.7661178112030029e-01 2.0509929955005646e-01
            -2.9771140217781067e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 324 -5.8827269822359085e-03 -1 -2 325
            9.0498890494927764e-04</internalNodes>
          <leafValues>
            -6.1301797628402710e-01 -3.4023219347000122e-01
            1.8168209493160248e-01</leafValues></_>
        <_>
          <internalNodes>
            0 1 326 -9.8338901996612549e-02 -1 -2 327
            5.6141808629035950e-02</internalNodes>
          <leafValues>
            4.7729569673538208e-01 -2.2904439270496368e-01
            3.441008925437927
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

