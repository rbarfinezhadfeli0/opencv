# Documentation for `docs/data/haarcascades/haarcascade_frontalcatface_extended.xml_docs.md`

## File Metadata

- **Full Path**: `docs/data/haarcascades/haarcascade_frontalcatface_extended.xml_docs.md`
- **File Name**: `haarcascade_frontalcatface_extended.xml_docs.md`
- **File Size**: 51,110 bytes
- **File Type**: .md
- **Link to Source**: [docs/data/haarcascades/haarcascade_frontalcatface_extended.xml_docs.md](../../../docs/data/haarcascades/haarcascade_frontalcatface_extended.xml_docs.md)

## Purpose and Role

This file is located in the `docs/data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `data/haarcascades/haarcascade_frontalcatface_extended.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_frontalcatface_extended.xml`
- **File Name**: `haarcascade_frontalcatface_extended.xml`
- **File Size**: 382,918 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_frontalcatface_extended.xml](../../data/haarcascades/haarcascade_frontalcatface_extended.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
 A frontal cat face detector using the full set of Haar features, i.e.
 horizontal, vertical, and diagonal features.

 Contributed by Joseph Howse (josephhowse@nummist.com).

 More information can be found in the following publications and
 presentations:

 Joseph Howse. OpenCV for Secret Agents (book). Packt Publishing, January
   2015.
 Joseph Howse. "Training Detectors and Recognizers in Python and OpenCV"
   (tutorial). ISMAR 2014. September 9, 2014.
   http://nummist.com/opencv/Howse_ISMAR_20140909.pdf
 Joseph Howse. "Training Intelligent Camera Systems with Python and OpenCV"
   (webcast). O’Reilly Media. June 17, 2014.
   http://www.oreilly.com/pub/e/3077

 Build scripts and demo applications can be found in the following repository:
 https://bitbucket.org/Joe_Howse/angora-blue

 KNOWN LIMITATIONS:

 An upright subject is assumed. In situations where the cat's face might be
 sideways or upside down (e.g. the cat is rolling over), try various rotations
 of the input image.

 CHANGELOG:

 2016-08-06: Re-trained with more negative samples and more stages. False
   positives are much rarer now. If you tailored your code for the cascade's
   previous version, now you should re-adjust the arguments of
   CascadeClassifier::detectMultiScale. For example, decrease the value of the
   minNeighbors argument. You do not need to use a human face detector to
   cross-check the positives anymore.
 2014-04-25: First release (at https://bitbucket.org/Joe_Howse/angora-blue)

 //////////////////////////////////////////////////////////////////////////
 | Contributors License Agreement
 | IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
 |   By downloading, copying, installing or using the software you agree
 |   to this license.
 |   If you do not agree to this license, do not download, install,
 |   copy or use the software.
 |
 | Copyright (c) 2014-2016, Joseph Howse (Nummist Media Corporation Limited,
 | Halifax, Nova Scotia, Canada). All rights reserved.
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
<cascade>
  <stageType>BOOST</stageType>
  <featureType>HAAR</featureType>
  <height>24</height>
  <width>24</width>
  <stageParams>
    <boostType>GAB</boostType>
    <minHitRate>9.9500000476837158e-01</minHitRate>
    <maxFalseAlarm>5.0000000000000000e-01</maxFalseAlarm>
    <weightTrimRate>9.4999999999999996e-01</weightTrimRate>
    <maxDepth>1</maxDepth>
    <maxWeakCount>100</maxWeakCount></stageParams>
  <featureParams>
    <maxCatCount>0</maxCatCount>
    <featSize>1</featSize>
    <mode>ALL</mode></featureParams>
  <stageNum>20</stageNum>
  <stages>
    <!-- stage 0 -->
    <_>
      <maxWeakCount>13</maxWeakCount>
      <stageThreshold>-1.4294912815093994e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 394 -1.5126220881938934e-02</internalNodes>
          <leafValues>
            7.5887596607208252e-01 -3.4230688214302063e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 737 3.9337221533060074e-03</internalNodes>
          <leafValues>
            -3.3288389444351196e-01 5.2361363172531128e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 757 -1.5044892206788063e-02</internalNodes>
          <leafValues>
            5.5565774440765381e-01 -2.2505992650985718e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 450 -1.5777055174112320e-02</internalNodes>
          <leafValues>
            7.2692525386810303e-01 -1.6206762194633484e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 443 3.0781796202063560e-02</internalNodes>
          <leafValues>
            -1.8173390626907349e-01 7.3483395576477051e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 220 1.8483418971300125e-02</internalNodes>
          <leafValues>
            -1.8690711259841919e-01 5.0116515159606934e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 681 1.3474167324602604e-02</internalNodes>
          <leafValues>
            -1.5681208670139313e-01 5.8611637353897095e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 554 5.3415738046169281e-02</internalNodes>
          <leafValues>
            -1.6418528556823730e-01 6.8128466606140137e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 741 5.4243900813162327e-03</internalNodes>
          <leafValues>
            -1.8231739103794098e-01 4.6716138720512390e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 336 1.7689792439341545e-02</internalNodes>
          <leafValues>
            -1.3713267445564270e-01 6.0434049367904663e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 187 2.2149257711134851e-04</internalNodes>
          <leafValues>
            -2.7738124132156372e-01 2.8165665268898010e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 288 -2.8517641127109528e-02</internalNodes>
          <leafValues>
            5.5257320404052734e-01 -1.2970162928104401e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 369 4.3854981660842896e-02</internalNodes>
          <leafValues>
            -1.9231440126895905e-01 4.2093500494956970e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 1 -->
    <_>
      <maxWeakCount>27</maxWeakCount>
      <stageThreshold>-1.5509251356124878e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 337 2.4014184251427650e-02</internalNodes>
          <leafValues>
            -2.1038578450679779e-01 7.3892170190811157e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 475 -5.5319909006357193e-03</internalNodes>
          <leafValues>
            4.4344031810760498e-01 -2.8907662630081177e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 4 2.7481060475111008e-02</internalNodes>
          <leafValues>
            -1.9128543138504028e-01 5.1661676168441772e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 457 -1.1628001928329468e-02</internalNodes>
          <leafValues>
            5.1978123188018799e-01 -1.7051684856414795e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 393 1.5159824397414923e-03</internalNodes>
          <leafValues>
            -2.9784303903579712e-01 3.9050224423408508e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 901 1.3662670738995075e-02</internalNodes>
          <leafValues>
            -1.4316783845424652e-01 4.4111710786819458e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 780 -3.6911026109009981e-03</internalNodes>
          <leafValues>
            3.2185173034667969e-01 -2.3853960633277893e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 769 3.3176485449075699e-02</internalNodes>
          <leafValues>
            -7.4603199958801270e-02 7.5860917568206787e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 317 -5.7046953588724136e-03</internalNodes>
          <leafValues>
            -7.5004047155380249e-01 1.0240622609853745e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 73 7.9660946503281593e-03</internalNodes>
          <leafValues>
            9.8882928490638733e-02 -7.3491615056991577e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 739 3.0965393409132957e-02</internalNodes>
          <leafValues>
            -1.6046196222305298e-01 4.5570060610771179e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 612 -4.0078125894069672e-03</internalNodes>
          <leafValues>
            -7.1539020538330078e-01 6.9276176393032074e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 647 -8.2283765077590942e-03</internalNodes>
          <leafValues>
            3.2576236128807068e-01 -1.8509653210639954e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 170 3.4253271296620369e-03</internalNodes>
          <leafValues>
            1.0964145511388779e-01 -5.8205413818359375e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 434 9.0980646200478077e-04</internalNodes>
          <leafValues>
            -2.0425215363502502e-01 2.7488732337951660e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 427 5.9772443026304245e-02</internalNodes>
          <leafValues>
            -1.3786207139492035e-01 4.0762668848037720e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 209 -4.1712004691362381e-02</internalNodes>
          <leafValues>
            4.9409377574920654e-01 -1.1713714897632599e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 248 -3.0311278998851776e-02</internalNodes>
          <leafValues>
            5.1191121339797974e-01 -1.0507214814424515e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 339 -6.5785087645053864e-03</internalNodes>
          <leafValues>
            -7.6472043991088867e-01 8.0923363566398621e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 37 1.1685060337185860e-02</internalNodes>
          <leafValues>
            5.0379037857055664e-02 -7.9744982719421387e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 423 6.5714016556739807e-02</internalNodes>
          <leafValues>
            -1.1398456245660782e-01 4.9489131569862366e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 755 9.7422497346997261e-03</internalNodes>
          <leafValues>
            -1.4347794651985168e-01 3.6561754345893860e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 870 4.9857441335916519e-03</internalNodes>
          <leafValues>
            7.9834438860416412e-02 -7.2391557693481445e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 735 -1.1547822505235672e-03</internalNodes>
          <leafValues>
            4.1867440938949585e-01 -1.2869183719158173e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 519 -4.4658007100224495e-03</internalNodes>
          <leafValues>
            -6.7933702468872070e-01 8.2867160439491272e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 862 3.6325352266430855e-03</internalNodes>
          <leafValues>
            6.6807270050048828e-02 -6.0182958841323853e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 127 7.4123376980423927e-03</internalNodes>
          <leafValues>
            -1.5108695626258850e-01 3.2046884298324585e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 2 -->
    <_>
      <maxWeakCount>26</maxWeakCount>
      <stageThreshold>-1.3890913724899292e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 619 1.7836617305874825e-02</internalNodes>
          <leafValues>
            -2.1508488059043884e-01 6.6796410083770752e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 457 -8.5781915113329887e-03</internalNodes>
          <leafValues>
            5.0962758064270020e-01 -2.2129471600055695e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 165 3.1586211174726486e-02</internalNodes>
          <leafValues>
            -2.1485456824302673e-01 4.2591696977615356e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 518 2.5690056383609772e-02</internalNodes>
          <leafValues>
            -1.5910078585147858e-01 6.7842948436737061e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 768 -2.2857591509819031e-02</internalNodes>
          <leafValues>
            5.7221925258636475e-01 -1.3710150122642517e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 741 4.7176675871014595e-03</internalNodes>
          <leafValues>
            -2.3617559671401978e-01 3.9870622754096985e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 615 -2.3281413596123457e-03</internalNodes>
          <leafValues>
            -7.0095318555831909e-01 1.3746888935565948e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 139 1.0266102617606521e-03</internalNodes>
          <leafValues>
            -2.6873087882995605e-01 2.6495781540870667e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 2 -7.6808528974652290e-03</internalNodes>
          <leafValues>
            3.6925876140594482e-01 -2.1339643001556396e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 454 6.4357556402683258e-02</internalNodes>
          <leafValues>
            -1.1779088526964188e-01 5.5030888319015503e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 296 8.9486092329025269e-02</internalNodes>
          <leafValues>
            -1.4395782351493835e-01 5.3468054533004761e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 253 -5.6334878318011761e-03</internalNodes>
          <leafValues>
            -6.5704786777496338e-01 1.3971389830112457e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 834 -8.0200601369142532e-03</internalNodes>
          <leafValues>
            3.6956611275672913e-01 -1.8284171819686890e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 732 8.3984360098838806e-03</internalNodes>
          <leafValues>
            -1.3507588207721710e-01 4.4903004169464111e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 246 -5.7764705270528793e-03</internalNodes>
          <leafValues>
            -6.5459579229354858e-01 1.1050829291343689e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 630 3.9896301925182343e-02</internalNodes>
          <leafValues>
            -1.5822732448577881e-01 3.6069712042808533e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 11 -6.8376958370208740e-02</internalNodes>
          <leafValues>
            6.2642019987106323e-01 -8.3647280931472778e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 696 -2.7075063437223434e-02</internalNodes>
          <leafValues>
            4.0549215674400330e-01 -1.4247153699398041e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 933 6.8107023835182190e-03</internalNodes>
          <leafValues>
            7.7754773199558258e-02 -6.4665120840072632e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 131 3.6659452598541975e-03</internalNodes>
          <leafValues>
            7.9356946051120758e-02 -5.4679936170578003e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 182 2.3308303207159042e-02</internalNodes>
          <leafValues>
            -1.4383231103420258e-01 3.4179633855819702e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 389 -3.2547116279602051e-02</internalNodes>
          <leafValues>
            3.6395668983459473e-01 -1.2551946938037872e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 471 1.6501296311616898e-02</internalNodes>
          <leafValues>
            -1.0674661397933960e-01 4.2714300751686096e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 616 -2.9296698048710823e-03</internalNodes>
          <leafValues>
            -5.7476091384887695e-01 8.5429534316062927e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 828 1.3306898763403296e-03</internalNodes>
          <leafValues>
            -1.2303277105093002e-01 3.7224721908569336e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 18 9.8933260887861252e-03</internalNodes>
          <leafValues>
            6.7675270140171051e-02 -6.7935848236083984e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 3 -->
    <_>
      <maxWeakCount>31</maxWeakCount>
      <stageThreshold>-1.4026626348495483e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 876 -1.4927964657545090e-02</internalNodes>
          <leafValues>
            6.3834953308105469e-01 -1.8698258697986603e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 467 -1.1759694665670395e-02</internalNodes>
          <leafValues>
            5.0763273239135742e-01 -2.0944127440452576e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 775 1.1289508081972599e-02</internalNodes>
          <leafValues>
            -1.4533838629722595e-01 5.3039866685867310e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 335 1.3691024854779243e-02</internalNodes>
          <leafValues>
            -1.3143934309482574e-01 5.9853446483612061e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 399 -8.6051290854811668e-03</internalNodes>
          <leafValues>
            3.1604155898094177e-01 -2.2497664391994476e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 898 1.1611104011535645e-02</internalNodes>
          <leafValues>
            -1.7180299758911133e-01 3.6340636014938354e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 919 5.4911419283598661e-04</internalNodes>
          <leafValues>
            -2.0625770092010498e-01 3.0243906378746033e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 448 -1.1997690424323082e-02</internalNodes>
          <leafValues>
            6.7541980743408203e-01 -1.0784135758876801e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 610 -2.0809918642044067e-03</internalNodes>
          <leafValues>
            -5.7404327392578125e-01 1.1769672483205795e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 277 6.8656861782073975e-02</internalNodes>
          <leafValues>
            -1.4633083343505859e-01 4.1269731521606445e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 215 -4.5645810663700104e-02</internalNodes>
          <leafValues>
            5.4341620206832886e-01 -1.1726979166269302e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 890 -1.8052812665700912e-02</internalNodes>
          <leafValues>
            3.6646232008934021e-01 -1.3256482779979706e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 897 9.2329997569322586e-03</internalNodes>
          <leafValues>
            9.1808989644050598e-02 -6.4987671375274658e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 142 -2.9587259050458670e-03</internalNodes>
          <leafValues>
            2.4805040657520294e-01 -2.0830279588699341e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 151 -7.1467030793428421e-03</internalNodes>
          <leafValues>
            -6.6564339399337769e-01 8.8065519928932190e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 756 -5.7738199830055237e-03</internalNodes>
          <leafValues>
            2.4252247810363770e-01 -2.1394193172454834e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 207 6.4636822789907455e-03</internalNodes>
          <leafValues>
            8.4821723401546478e-02 -6.4125812053680420e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 527 -2.8782974928617477e-02</internalNodes>
          <leafValues>
            3.5874211788177490e-01 -1.4370997250080109e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 715 -1.8174832221120596e-03</internalNodes>
          <leafValues>
            3.7480926513671875e-01 -1.2761794030666351e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 590 -1.9234847277402878e-03</internalNodes>
          <leafValues>
            -5.6678783893585205e-01 9.0299606323242188e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 588 2.8048637323081493e-03</internalNodes>
          <leafValues>
            8.5870750248432159e-02 -5.8541411161422729e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 178 7.0693701505661011e-02</internalNodes>
          <leafValues>
            -1.2318307906389236e-01 3.9827430248260498e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 554 6.2659628689289093e-02</internalNodes>
          <leafValues>
            -9.1229990124702454e-02 5.0639665126800537e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 321 -3.7420655135065317e-03</internalNodes>
          <leafValues>
            3.5059738159179688e-01 -1.2444343417882919e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 273 6.8388320505619049e-03</internalNodes>
          <leafValues>
            -1.0419095307588577e-01 4.5085826516151428e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 76 7.1193519979715347e-03</internalNodes>
          <leafValues>
            9.1205865144729614e-02 -5.2279585599899292e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 791 -9.8787562455981970e-04</internalNodes>
          <leafValues>
            2.8105542063713074e-01 -1.5169830620288849e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 639 1.8099821172654629e-03</internalNodes>
          <leafValues>
            6.5428622066974640e-02 -6.9196063280105591e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 726 -6.0212425887584686e-03</internalNodes>
          <leafValues>
            -6.2636482715606689e-01 5.1543414592742920e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 818 5.1644006744027138e-03</internalNodes>
          <leafValues>
            6.3040286302566528e-02 -6.3455927371978760e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 205 9.4506526365876198e-03</internalNodes>
          <leafValues>
            -1.3443979620933533e-01 3.1506177783012390e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 4 -->
    <_>
      <maxWeakCount>38</maxWeakCount>
      <stageThreshold>-1.4621645212173462e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 383 -1.5925668179988861e-02</internalNodes>
          <leafValues>
            6.2127149105072021e-01 -1.8520653247833252e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 648 1.0260052047669888e-02</internalNodes>
          <leafValues>
            -2.4736632406711578e-01 4.2336893081665039e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 3 5.7025998830795288e-03</internalNodes>
          <leafValues>
            -2.3670144379138947e-01 3.3228391408920288e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 264 9.3164276331663132e-03</internalNodes>
          <leafValues>
            -1.7946784198284149e-01 4.6311038732528687e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 830 -5.0438079051673412e-03</internalNodes>
          <leafValues>
            4.4613519310951233e-01 -1.6072992980480194e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 793 2.8381291776895523e-03</internalNodes>
          <leafValues>
            -1.8486896157264709e-01 3.5892590880393982e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 455 6.7377656698226929e-02</internalNodes>
          <leafValues>
            -1.7760114371776581e-01 3.9539518952369690e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 44 -8.7916189804673195e-03</internalNodes>
          <leafValues>
            -5.9182339906692505e-01 1.1145308613777161e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 874 1.3353329151868820e-02</internalNodes>
          <leafValues>
            -1.1993711441755295e-01 4.8862439393997192e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 324 -1.0008489713072777e-02</internalNodes>
          <leafValues>
            4.1768664121627808e-01 -1.2453128397464752e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 795 -1.4410717412829399e-03</internalNodes>
          <leafValues>
            3.4100320935249329e-01 -1.6849595308303833e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 123 1.1647527664899826e-01</internalNodes>
          <leafValues>
            -9.7596585750579834e-02 4.2289251089096069e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 301 -9.8112244158983231e-03</internalNodes>
          <leafValues>
            2.6155915856361389e-01 -2.0234876871109009e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 425 6.3042029738426208e-02</internalNodes>
          <leafValues>
            -1.2662252783775330e-01 3.6811619997024536e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 553 -1.7675247043371201e-02</internalNodes>
          <leafValues>
            4.1690909862518311e-01 -1.1987055838108063e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 105 4.0485346689820290e-03</internalNodes>
          <leafValues>
            7.0249855518341064e-02 -7.3556905984878540e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 675 8.2748252898454666e-03</internalNodes>
          <leafValues>
            -1.6168670356273651e-01 2.8835350275039673e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 313 -5.0843162462115288e-03</internalNodes>
          <leafValues>
            -5.8562570810317993e-01 8.9675068855285645e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 249 6.0826279222965240e-03</internalNodes>
          <leafValues>
            4.7766357660293579e-02 -6.8612217903137207e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 48 8.5826087743043900e-03</internalNodes>
          <leafValues>
            -1.6963686048984528e-01 2.6875671744346619e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 38 2.4908576160669327e-02</internalNodes>
          <leafValues>
            8.5034154355525970e-02 -5.7059210538864136e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 879 2.0448346622288227e-03</internalNodes>
          <leafValues>
            -1.8642950057983398e-01 2.3178242146968842e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 16 2.4130716919898987e-02</internalNodes>
          <leafValues>
            -1.2823060154914856e-01 3.4394741058349609e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 154 -4.7494415193796158e-03</internalNodes>
          <leafValues>
            -7.1827727556228638e-01 6.8053275346755981e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 199 -1.7751917243003845e-02</internalNodes>
          <leafValues>
            -5.5972510576248169e-01 5.2141726016998291e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 339 5.5826390162110329e-03</internalNodes>
          <leafValues>
            4.8266090452671051e-02 -5.9813541173934937e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 387 1.4416726771742105e-03</internalNodes>
          <leafValues>
            -9.2707693576812744e-02 4.1495534777641296e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 192 -2.1779362577944994e-03</internalNodes>
          <leafValues>
            2.7112621068954468e-01 -1.5071788430213928e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 607 3.0656920280307531e-03</internalNodes>
          <leafValues>
            6.0340058058500290e-02 -6.5465551614761353e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 469 1.9947460293769836e-01</internalNodes>
          <leafValues>
            -9.5098674297332764e-02 3.9016976952552795e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 857 -2.0255323499441147e-02</internalNodes>
          <leafValues>
            4.3044877052307129e-01 -8.8302992284297943e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 446 5.4685659706592560e-03</internalNodes>
          <leafValues>
            -8.7241113185882568e-02 3.9513549208641052e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 463 -1.0883151553571224e-03</internalNodes>
          <leafValues>
            2.9802373051643372e-01 -1.3696449995040894e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 655 -5.0911568105220795e-03</internalNodes>
          <leafValues>
            -6.2439930438995361e-01 6.2544539570808411e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 221 -5.2395770326256752e-03</internalNodes>
          <leafValues>
            -6.9036418199539185e-01 4.5142117887735367e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 955 4.0486194193363190e-02</internalNodes>
          <leafValues>
            -7.5753845274448395e-02 5.2426725625991821e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 300 4.1610337793827057e-03</internalNodes>
          <leafValues>
            6.6071115434169769e-02 -5.8079534769058228e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 272 -6.4253048039972782e-03</internalNodes>
          <leafValues>
            3.0481830239295959e-01 -1.1435022950172424e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 5 -->
    <_>
      <maxWeakCount>44</maxWeakCount>
      <stageThreshold>-1.4235107898712158e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 716 -2.2738082334399223e-03</internalNodes>
          <leafValues>
            5.9519726037979126e-01 -1.6779936850070953e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 457 -1.2204157188534737e-02</internalNodes>
          <leafValues>
            4.6985983848571777e-01 -1.7339397966861725e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 754 3.1242824625223875e-03</internalNodes>
          <leafValues>
            -2.2488421201705933e-01 3.4029743075370789e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 777 -3.9868438616394997e-03</internalNodes>
          <leafValues>
            3.8314539194107056e-01 -1.8952924013137817e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 538 -5.4737669415771961e-03</internalNodes>
          <leafValues>
            2.4583901464939117e-01 -2.3114782571792603e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 453 1.5154287219047546e-02</internalNodes>
          <leafValues>
            -1.0675037652254105e-01 5.8347207307815552e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 397 -1.4294658321887255e-03</internalNodes>
          <leafValues>
            3.8292840123176575e-01 -1.2911921739578247e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 750 -7.4405185878276825e-03</internalNodes>
          <leafValues>
            2.8356546163558960e-01 -1.7810684442520142e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 786 -4.0357224643230438e-03</internalNodes>
          <leafValues>
            2.6303085684776306e-01 -1.6862161457538605e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 618 -5.8342106640338898e-03</internalNodes>
          <leafValues>
            3.2040205597877502e-01 -1.4103877544403076e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 161 1.7279960215091705e-02</internalNodes>
          <leafValues>
            -1.7433850467205048e-01 2.7985212206840515e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 292 2.2125110030174255e-02</internalNodes>
          <leafValues>
            -1.1797516793012619e-01 4.0373948216438293e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 958 -4.4059187173843384e-02</internalNodes>
          <leafValues>
            5.2820503711700439e-01 -7.0916719734668732e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 194 -3.8316637277603149e-02</internalNodes>
          <leafValues>
            3.8833045959472656e-01 -1.0811555385589600e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 178 4.5704744756221771e-02</internalNodes>
          <leafValues>
            -1.7566929757595062e-01 3.4665411710739136e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 434 1.1523386929184198e-03</internalNodes>
          <leafValues>
            -1.7257389426231384e-01 2.5989890098571777e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 121 -1.0491746477782726e-02</internalNodes>
          <leafValues>
            -6.1285555362701416e-01 7.1230083703994751e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 395 -4.5014433562755585e-03</internalNodes>
          <leafValues>
            -5.7712453603744507e-01 5.8887075632810593e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 950 -3.7281280383467674e-03</internalNodes>
          <leafValues>
            -6.7359894514083862e-01 5.2957162261009216e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 331 3.4461893141269684e-02</internalNodes>
          <leafValues>
            -1.0375578701496124e-01 3.7974634766578674e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 462 -1.3906960375607014e-03</internalNodes>
          <leafValues>
            3.9171192049980164e-01 -1.0048408061265945e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 85 1.6332454979419708e-02</internalNodes>
          <leafValues>
            8.6256101727485657e-02 -4.5887523889541626e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 356 -6.0738036409020424e-03</internalNodes>
          <leafValues>
            -5.2265202999114990e-01 6.5308839082717896e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 486 -3.3630726393312216e-03</internalNodes>
          <leafValues>
            -5.6505429744720459e-01 5.5844355374574661e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 418 -1.5329496003687382e-02</internalNodes>
          <leafValues>
            3.4475114941596985e-01 -1.0086353123188019e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 587 -9.0496204793453217e-03</internalNodes>
          <leafValues>
            2.9553902149200439e-01 -1.1406829208135605e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 794 -3.1109917908906937e-03</internalNodes>
          <leafValues>
            -4.4897687435150146e-01 7.3615357279777527e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 939 3.3499556593596935e-03</internalNodes>
          <leafValues>
            5.4718658328056335e-02 -5.4810231924057007e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 188 1.8374501960352063e-03</internalNodes>
          <leafValues>
            -1.3522666692733765e-01 2.4655479192733765e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 908 2.6134990621358156e-03</internalNodes>
          <leafValues>
            6.6369861364364624e-02 -4.7342041134834290e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 65 -7.4155852198600769e-03</internalNodes>
          <leafValues>
            2.0866124331951141e-01 -1.5775154531002045e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 515 3.9352793246507645e-03</internalNodes>
          <leafValues>
            5.1660846918821335e-02 -6.2589824199676514e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 735 -1.0450070258229971e-03</internalNodes>
          <leafValues>
            3.3525371551513672e-01 -1.0084854811429977e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 784 1.2639444321393967e-03</internalNodes>
          <leafValues>
            -1.2103077769279480e-01 2.7691018581390381e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 479 7.7577251940965652e-03</internalNodes>
          <leafValues>
            4.6813234686851501e-02 -7.3385792970657349e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 18 -1.0632604360580444e-02</internalNodes>
          <leafValues>
            -7.1024382114410400e-01 3.3777639269828796e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 183 1.8631946295499802e-02</internalNodes>
          <leafValues>
            -1.4613701403141022e-01 2.1491082012653351e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 608 4.9128942191600800e-03</internalNodes>
          <leafValues>
            5.3445268422365189e-02 -6.3314527273178101e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 473 -9.8230186849832535e-03</internalNodes>
          <leafValues>
            2.6917773485183716e-01 -1.1376978456974030e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 910 -3.0754944309592247e-03</internalNodes>
          <leafValues>
            -5.0787961483001709e-01 6.1582125723361969e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 659 -6.7374799400568008e-03</internalNodes>
          <leafValues>
            2.3871047794818878e-01 -1.2552142143249512e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 507 -1.1759715154767036e-02</internalNodes>
          <leafValues>
            3.3646693825721741e-01 -9.4460532069206238e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 318 -4.1377237066626549e-03</internalNodes>
          <leafValues>
            -5.0522220134735107e-01 6.2668189406394958e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 320 1.7267453949898481e-03</internalNodes>
          <leafValues>
            -8.0607026815414429e-02 3.8304185867309570e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 6 -->
    <_>
      <maxWeakCount>47</maxWeakCount>
      <stageThreshold>-1.4313566684722900e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 882 -1.1920252814888954e-02</internalNodes>
          <leafValues>
            5.6617152690887451e-01 -1.5811842679977417e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 568 -4.3085627257823944e-03</internalNodes>
          <leafValues>
            4.4759327173233032e-01 -1.6846470534801483e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 883 1.1177745182067156e-03</internalNodes>
          <leafValues>
            -1.5351393818855286e-01 4.3508940935134888e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 798 3.5418532788753510e-02</internalNodes>
          <leafValues>
            -1.2973460555076599e-01 3.6943939328193665e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 393 2.2405586205422878e-03</internalNodes>
          <leafValues>
            -1.8800468742847443e-01 3.2498928904533386e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 265 -1.7982896417379379e-02</internalNodes>
          <leafValues>
            4.5607218146324158e-01 -1.0459473729133606e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 152 -4.9088716506958008e-02</internalNodes>
          <leafValues>
            3.4279289841651917e-01 -1.5114119648933411e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 275 7.1780886501073837e-03</internalNodes>
          <leafValues>
            6.3825756311416626e-02 -6.2449872493743896e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 849 3.9123920723795891e-03</internalNodes>
          <leafValues>
            7.1502417325973511e-02 -6.3956946134567261e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 689 -4.1980943642556667e-03</internalNodes>
          <leafValues>
            2.1998657286167145e-01 -1.9890366494655609e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 660 -4.5476644299924374e-03</internalNodes>
          <leafValues>
            2.1866278350353241e-01 -1.9852560758590698e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 944 -4.4158436357975006e-03</internalNodes>
          <leafValues>
            2.3959043622016907e-01 -1.7090958356857300e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 281 -4.7058244235813618e-03</internalNodes>
          <leafValues>
            -5.1537507772445679e-01 9.0310461819171906e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 116 -8.7488889694213867e-03</internalNodes>
          <leafValues>
            2.2937677800655365e-01 -1.8315380811691284e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 645 -3.1655649654567242e-03</internalNodes>
          <leafValues>
            -7.3091191053390503e-01 6.5193220973014832e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 267 6.4696683548390865e-03</internalNodes>
          <leafValues>
            -1.1077737808227539e-01 3.7207809090614319e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 615 2.2985613904893398e-03</internalNodes>
          <leafValues>
            7.7800542116165161e-02 -5.1104581356048584e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 359 4.5809363946318626e-03</internalNodes>
          <leafValues>
            5.7778771966695786e-02 -5.7898092269897461e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 188 1.1279166210442781e-03</internalNodes>
          <leafValues>
            -1.7981146275997162e-01 1.9939005374908447e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 347 -1.2820301577448845e-02</internalNodes>
          <leafValues>
            5.1867282390594482e-01 -6.9989629089832306e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 810 4.4866472482681274e-02</internalNodes>
          <leafValues>
            -1.4253044128417969e-01 3.0062338709831238e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 412 -3.5413210280239582e-03</internalNodes>
          <leafValues>
            -5.7618641853332520e-01 6.0328345745801926e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 362 -7.4678594246506691e-03</internalNodes>
          <leafValues>
            -5.0187259912490845e-01 6.1294022947549820e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 678 1.8058011308312416e-02</internalNodes>
          <leafValues>
            5.3603217005729675e-02 -5.8919399976730347e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 935 -6.8098572082817554e-03</internalNodes>
          <leafValues>
            -5.4100829362869263e-01 5.5898215621709824e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 307 3.6491458304226398e-03</internalNodes>
          <leafValues>
            4.7378763556480408e-02 -5.9323132038116455e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 284 1.4524955768138170e-03</internalNodes>
          <leafValues>
            -8.8994570076465607e-02 3.8729071617126465e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 219 -6.2408884987235069e-03</internalNodes>
          <leafValues>
            -6.6442847251892090e-01 5.1082015037536621e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 744 -9.9360430613160133e-04</internalNodes>
          <leafValues>
            3.2972389459609985e-01 -1.0494423657655716e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 285 3.9777760393917561e-03</internalNodes>
          <leafValues>
            5.4083213210105896e-02 -6.2114214897155762e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 380 -1.4884659089148045e-02</internalNodes>
          <leafValues>
            2.4066454172134399e-01 -1.2317410856485367e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 436 3.3154981210827827e-03</internalNodes>
          <leafValues>
            -1.1744727939367294e-01 2.9429042339324951e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 976 -4.7508114948868752e-03</internalNodes>
          <leafValues>
            -4.5763325691223145e-01 6.7066885530948639e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 779 -1.1973761022090912e-02</internalNodes>
          <leafValues>
            2.5750914216041565e-01 -1.1354148387908936e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 740 4.9072699621319771e-03</internalNodes>
          <leafValues>
            -1.1266437917947769e-01 3.0022394657135010e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 56 6.5630510449409485e-02</internalNodes>
          <leafValues>
            -1.0180503129959106e-01 3.0517497658729553e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 354 -2.3393325507640839e-02</internalNodes>
          <leafValues>
            3.2443770766258240e-01 -9.5363102853298187e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 834 -3.8902116939425468e-03</internalNodes>
          <leafValues>
            2.0148487389087677e-01 -1.4944279193878174e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 185 -2.5926973670721054e-02</internalNodes>
          <leafValues>
            -4.4917497038841248e-01 6.9752328097820282e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 173 -7.1825529448688030e-03</internalNodes>
          <leafValues>
            -5.6838059425354004e-01 4.9584377557039261e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 548 -9.9399685859680176e-03</internalNodes>
          <leafValues>
            3.0747908353805542e-01 -1.1064232140779495e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 978 -3.6286246031522751e-03</internalNodes>
          <leafValues>
            -6.0276371240615845e-01 5.2405584603548050e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 820 1.5756220091134310e-03</internalNodes>
          <leafValues>
            -1.1615782976150513e-01 2.6717522740364075e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 426 3.5662509500980377e-02</internalNodes>
          <leafValues>
            -1.0885569453239441e-01 2.9044550657272339e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 554 5.3282946348190308e-02</internalNodes>
          <leafValues>
            -8.1855505704879761e-02 4.0298762917518616e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 988 3.3901704009622335e-03</internalNodes>
          <leafValues>
            5.5047694593667984e-02 -5.4021596908569336e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 384 1.3204356655478477e-03</internalNodes>
          <leafValues>
            -9.4643965363502502e-02 3.0430349707603455e-01</leafValues></_></weakClassifi
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

