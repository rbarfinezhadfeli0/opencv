# Documentation for `docs/data/lbpcascades/lbpcascade_frontalcatface.xml_docs.md`

## File Metadata

- **Full Path**: `docs/data/lbpcascades/lbpcascade_frontalcatface.xml_docs.md`
- **File Name**: `lbpcascade_frontalcatface.xml_docs.md`
- **File Size**: 51,055 bytes
- **File Type**: .md
- **Link to Source**: [docs/data/lbpcascades/lbpcascade_frontalcatface.xml_docs.md](../../../docs/data/lbpcascades/lbpcascade_frontalcatface.xml_docs.md)

## Purpose and Role

This file is located in the `docs/data/lbpcascades` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `data/lbpcascades/lbpcascade_frontalcatface.xml`

## File Metadata

- **Full Path**: `data/lbpcascades/lbpcascade_frontalcatface.xml`
- **File Name**: `lbpcascade_frontalcatface.xml`
- **File Size**: 138,705 bytes
- **File Type**: .xml
- **Link to Source**: [data/lbpcascades/lbpcascade_frontalcatface.xml](../../data/lbpcascades/lbpcascade_frontalcatface.xml)

## Purpose and Role

This file is located in the `data/lbpcascades` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
 A frontal cat face detector using LBP features.

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
  <featureType>LBP</featureType>
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
    <maxCatCount>256</maxCatCount>
    <featSize>1</featSize></featureParams>
  <stageNum>20</stageNum>
  <stages>
    <!-- stage 0 -->
    <_>
      <maxWeakCount>9</maxWeakCount>
      <stageThreshold>-1.5429834127426147e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 104 1599056782 -3801105 836281000 -134348801 -10616883
            1430585292 -33555461 -170000676</internalNodes>
          <leafValues>
            -5.4490309953689575e-01 5.6734609603881836e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 294 204479732 1065172189 790634493 1060117940
            -289603585 -1075077121 -1092617217 -1073741825</internalNodes>
          <leafValues>
            -5.6601214408874512e-01 3.9540845155715942e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 71 -547356836 1566019389 -77856769 -551952545
            -539172900 -10560035 -52600869 994540299</internalNodes>
          <leafValues>
            -4.4183051586151123e-01 4.4477555155754089e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 148 -620691696 486545936 825237297 521165584 436288155
            536422827 1026760619 523727735</internalNodes>
          <leafValues>
            -4.9225500226020813e-01 3.4415841102600098e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 8 2065692655 -539494441 1911927792 -2108480 455801071
            67664201 -1084369169 1926622918</internalNodes>
          <leafValues>
            -4.0782809257507324e-01 3.7964683771133423e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 343 791362869 -1075233419 -573079553 -33596427
            -1610973955 -262147 1026375359 -1107337903</internalNodes>
          <leafValues>
            -4.4104734063148499e-01 3.3767992258071899e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 247 -770966446 -10095278 -717226475 1462226875
            1326402382 -70331239 1175244766 1326179231</internalNodes>
          <leafValues>
            -3.6573803424835205e-01 4.1576108336448669e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 137 -1898553987 -42050596 -1074372609 -38388009
            -50700804 -268436101 -352819484 -1163887504</internalNodes>
          <leafValues>
            -4.2117568850517273e-01 3.3924096822738647e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 188 -321131994 712703979 -1157694222 586350586
            1085998084 1281738876 -403229068 -925369374</internalNodes>
          <leafValues>
            -5.1358664035797119e-01 2.6718941330909729e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 1 -->
    <_>
      <maxWeakCount>12</maxWeakCount>
      <stageThreshold>-1.2918862104415894e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 54 1397073902 2013244398 -177098006 -201331986
            2002780074 1006612398 -1110 -67132758</internalNodes>
          <leafValues>
            -4.0280899405479431e-01 5.0070983171463013e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 80 -16823041 -41857 205806847 509368831 -1431699272
            -130942 167828670 -635806469</internalNodes>
          <leafValues>
            -3.7250527739524841e-01 4.8033073544502258e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 228 -866136833 419433907 182275839 536546047 201854172
            28121467 214781439 268402175</internalNodes>
          <leafValues>
            -4.5891678333282471e-01 3.6297130584716797e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 270 -1561337117 -1426066718 -1010829325 -38805262
            1202713555 -324545872 1441789815 -134233869</internalNodes>
          <leafValues>
            -3.5680472850799561e-01 4.3237748742103577e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 259 -597278700 268435458 -550058223 855800610 149425288
            775302951 -118555396 -272630117</internalNodes>
          <leafValues>
            -4.8106637597084045e-01 2.9466137290000916e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 327 -1111901448 1023973552 253008120 500562168 -5242881
            -1156710915 1340989437 -536879105</internalNodes>
          <leafValues>
            -3.2977777719497681e-01 4.3013024330139160e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 77 -537179120 521143296 -21635309 2134891643 1350441722
            -632386648 -1129300294 -263173</internalNodes>
          <leafValues>
            -3.6992338299751282e-01 3.5658127069473267e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 164 -1362153945 741281595 -1496274305 720813755
            2088729719 2146431455 -251775916 -185278474</internalNodes>
          <leafValues>
            -4.7965702414512634e-01 2.5329035520553589e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 129 2009068661 286465811 -1714126495 320044919
            -321273376 1216088341 -187041632 276299767</internalNodes>
          <leafValues>
            -4.0860968828201294e-01 3.0382931232452393e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 222 70785877 1041800816 -75508353 1071923196 2113290236
            -1074941012 -1075086339 -1351740304</internalNodes>
          <leafValues>
            -3.9227196574211121e-01 3.2063353061676025e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 52 -733166882 -44852481 -145568001 -15114497 -924915460
            -33678721 -88592641 1063732851</internalNodes>
          <leafValues>
            -3.7047380208969116e-01 3.1812894344329834e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 32 -2097737981 1148364791 -1210636301 -1330585805
            123014973 2113833983 -32925 -236603423</internalNodes>
          <leafValues>
            -4.2788907885551453e-01 2.8403493762016296e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 2 -->
    <_>
      <maxWeakCount>13</maxWeakCount>
      <stageThreshold>-1.3729060888290405e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 316 1058750395 -1080312336 -1078198785 -3 -1145389058
            -1078318408 -1079328769 -1077958496</internalNodes>
          <leafValues>
            -2.5900188088417053e-01 6.1616760492324829e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 151 -201887221 1915747883 -218104097 3145727 -92571030
            -117772190 -88408321 -1566377217</internalNodes>
          <leafValues>
            -3.7379077076911926e-01 4.5426961779594421e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 96 386868111 1607319551 2145227774 -81921 2136981469
            1564794828 -3277827 -171573810</internalNodes>
          <leafValues>
            -3.5450288653373718e-01 4.5893719792366028e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 2 -551362576 -73215120 -364212554 -11998256 71434372
            -1616176195 209715400 -840182021</internalNodes>
          <leafValues>
            -4.2596518993377686e-01 3.3351564407348633e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 82 247336735 586821439 -134217729 1056866239
            -1090551809 -9217 -33628162 -1430597650</internalNodes>
          <leafValues>
            -3.8904032111167908e-01 3.6951214075088501e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 249 -672137222 -74448921 -21891587 -136323721
            1322253791 -822281795 1565482863 1465906911</internalNodes>
          <leafValues>
            -3.2322180271148682e-01 4.2339357733726501e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 319 -1347239947 1069203248 -2097163 -1610678308
            -1879052295 -268437506 -1934079492 -1935110672</internalNodes>
          <leafValues>
            -4.0132158994674683e-01 3.3234956860542297e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 37 -1068709308 -1102522849 1644484342 -268379409
            1443710532 -11090094 -213845121 -12059649</internalNodes>
          <leafValues>
            -4.2390203475952148e-01 2.9791557788848877e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 202 -285495441 -1427182081 -290198603 -527241618
            -419467196 -855709209 -268509632 -922957372</internalNodes>
          <leafValues>
            -3.6988914012908936e-01 3.4229919314384460e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 43 -18449 1073711035 -1319386462 -203620626 2070804431
            854521820 -134545716 1442310948</internalNodes>
          <leafValues>
            -4.0420171618461609e-01 3.0070811510086060e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 205 225460085 1069170549 -1213441 -268681410 536639980
            -272859906 -1344397956 -1090621548</internalNodes>
          <leafValues>
            -4.0233635902404785e-01 3.1345960497856140e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 65 1347781250 -252185797 18161683 335127199 -918162925
            -68564231 1079360443 1080539939</internalNodes>
          <leafValues>
            -4.4784346222877502e-01 2.6452672481536865e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 40 138021973 914534260 -1628242049 179970033 2088664692
            -279028579 -285573707 -1431115148</internalNodes>
          <leafValues>
            -4.6739324927330017e-01 2.6337191462516785e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 3 -->
    <_>
      <maxWeakCount>15</maxWeakCount>
      <stageThreshold>-1.3434195518493652e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 312 1594941436 -539099137 1597374397 2136866813
            2133434303 -2359299 -180801 532360959</internalNodes>
          <leafValues>
            -3.9849624037742615e-01 4.3070858716964722e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 107 -1687737718 -136351793 819208354 -201466130
            -304104530 1430290438 -74187782 -254479670</internalNodes>
          <leafValues>
            -5.2847045660018921e-01 2.9759022593498230e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 106 1593187583 -57860 494558719 1568543997 -1430627078
            -17168194 -22085633 -17167126</internalNodes>
          <leafValues>
            -3.2349127531051636e-01 4.4297724962234497e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 233 -289936401 719515647 1894970098 -67961857
            1429694295 1079178739 1417049205 -235416077</internalNodes>
          <leafValues>
            -4.0733993053436279e-01 3.3705353736877441e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 338 -804225008 -1130847023 -1644863659 -626985985
            -1151011816 -67830287 -1090585640 -72811781</internalNodes>
          <leafValues>
            -4.0251138806343079e-01 3.1045806407928467e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 30 1325133735 -235668011 -4196493 -1297076240 37159747
            -1078198341 -5247041 -17338573</internalNodes>
          <leafValues>
            -3.5516351461410522e-01 3.3867400884628296e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 61 1563885570 990015234 1715350066 -1612515566
            2121997646 -562425 -1460130098 -2</internalNodes>
          <leafValues>
            -3.6495906114578247e-01 3.1801441311836243e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 35 -486805761 -30385 -164631585 -67393017 -662962177
            -269484302 -302258246 -298365693</internalNodes>
          <leafValues>
            -2.5626540184020996e-01 4.6705508232116699e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 242 582106997 -1375949832 -2097155 -1611859716
            -1078984707 -71372812 -1174471336 -1364685600</internalNodes>
          <leafValues>
            -3.4055373072624207e-01 3.2303497195243835e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 213 1857005319 651918339 -1159732176 -294652937
            2146914899 1349774535 -2097404 -50856010</internalNodes>
          <leafValues>
            -3.3969157934188843e-01 3.1695431470870972e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 292 -1364537606 -1925122 -36259590 -102826180
            -2098986320 243153064 -1515278601 -137380105</internalNodes>
          <leafValues>
            -3.3688172698020935e-01 3.2412472367286682e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 211 -824758745 174632251 -469831961 -1428162053
            1157038145 213857255 -186772236 -185273869</internalNodes>
          <leafValues>
            -4.0657189488410950e-01 2.5756362080574036e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 110 -16884641 -36276290 -160105525 -659501198 -652220
            -295070 -2176776 -15009184</internalNodes>
          <leafValues>
            -2.8538039326667786e-01 3.6766386032104492e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 90 -792331710 -1069030682 13697279 -2057320217
            -175702061 -60841468 -222841601 1783605027</internalNodes>
          <leafValues>
            -3.4717887639999390e-01 2.9574045538902283e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 234 749378744 935105716 -1397302147 1001388025
            255328920 -1799926224 -1397966593 -1074790917</internalNodes>
          <leafValues>
            -3.7167704105377197e-01 2.8397652506828308e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 4 -->
    <_>
      <maxWeakCount>15</maxWeakCount>
      <stageThreshold>-1.2288014888763428e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 271 -290784257 -542572545 -167807627 -2061 209652223
            -1093158913 1157580031 1173837303</internalNodes>
          <leafValues>
            -3.3414435386657715e-01 4.7178736329078674e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 317 -2098253 -1146358896 1476391185 -704777743
            -1140850693 -1415582030 -7696 -640683845</internalNodes>
          <leafValues>
            -2.5055918097496033e-01 5.2570897340774536e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 132 -100663299 272629759 -789195536 544341979
            -167781039 1966601201 -515 -234881025</internalNodes>
          <leafValues>
            -3.5798946022987366e-01 3.6239877343177795e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 48 -2105507696 -1366759952 -1126500363 -570630416
            -317934356 -33619969 -36643332 -2304812</internalNodes>
          <leafValues>
            -3.9822307229042053e-01 2.8633421659469604e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 74 -792677742 -101713185 7352369 298695163 -1761297729
            6029312 -25184257 1608465947</internalNodes>
          <leafValues>
            -4.2175698280334473e-01 2.4120111763477325e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 223 169615221 -1174487300 -281051137 -1090642276
            -4703305 -33558539 1073692156 -1359214592</internalNodes>
          <leafValues>
            -3.8317176699638367e-01 2.8932854533195496e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 75 -102806438 -9703431 1045568568 503377755 -547864488
            1534349686 1867669466 -82706561</internalNodes>
          <leafValues>
            -3.8330292701721191e-01 2.9036629199981689e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 332 784272181 -1073742539 -554188937 -67127948
            938153105 -1091568641 -1917947412 -2140908</internalNodes>
          <leafValues>
            -3.4002754092216492e-01 3.1559377908706665e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 214 256847367 707514298 -1090536922 -286331166
            1877943126 258930467 -52826284 -50858080</internalNodes>
          <leafValues>
            -4.0559005737304688e-01 2.5587162375450134e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 210 -176953385 839001603 -118293121 -46662093
            -928341523 783159107 1824812218 -294654113</internalNodes>
          <leafValues>
            -2.2742739319801331e-01 4.4131305813789368e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 291 -292556917 -268603344 -882511881 -142650156
            -1955672434 -84524073 -941787305 -807451567</internalNodes>
          <leafValues>
            -2.8523033857345581e-01 3.5285699367523193e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 86 -1006633234 -8464643 -1800132448 -126160132
            1597972294 2137374137 2130142532 -67635593</internalNodes>
          <leafValues>
            -3.6786875128746033e-01 2.6055160164833069e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 347 809435409 188951103 -1134973121 -1196245185
            926905404 -2113948 -1122058375 -93651077</internalNodes>
          <leafValues>
            -3.1239312887191772e-01 3.1785130500793457e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 47 -242549308 -1693329125 2124733233 857453163
            -233386788 -69871665 -1055992678 173495071</internalNodes>
          <leafValues>
            -3.7436348199844360e-01 2.9003840684890747e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 88 1543496623 -1277707265 1375580878 -1141900354
            -202408001 1073719014 -20973842 -454167574</internalNodes>
          <leafValues>
            -2.5309252738952637e-01 3.8837322592735291e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 5 -->
    <_>
      <maxWeakCount>17</maxWeakCount>
      <stageThreshold>-1.3014856576919556e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 56 -16777228 1343290737 -73730 1979579903 -167772964
            1145364445 -1 1969225695</internalNodes>
          <leafValues>
            -3.7386018037796021e-01 4.0434870123863220e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 329 -1078722575 -1078454128 991133557 -4369067
            -1079247105 -1145394016 522813437 -1080337223</internalNodes>
          <leafValues>
            -3.0502972006797791e-01 4.3909499049186707e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 252 -806361310 -492110674 -268445968 -1049642
            2002778951 1024456291 2012705892 -135792649</internalNodes>
          <leafValues>
            -4.1819226741790771e-01 2.9456692934036255e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 279 -52962084 805379072 -99820514 1040191240 -857961316
            -1164230721 -318841633 -1185</internalNodes>
          <leafValues>
            -3.2393226027488708e-01 3.5176092386245728e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 131 -148956144 992812800 463421370 920482344
            -1626551044 512506296 -8586248 -4198469</internalNodes>
          <leafValues>
            -4.0920355916023254e-01 2.6287314295768738e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 67 -1947229405 577929191 -1142967369 819199477
            1610560307 1002176493 -268443355 -185204747</internalNodes>
          <leafValues>
            -3.9257824420928955e-01 2.6880934834480286e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 173 2136997887 492685274 610548733 501067503 1428159743
            139549608 1843232763 1146223423</internalNodes>
          <leafValues>
            -2.6709714531898499e-01 3.5500597953796387e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 323 -173546237 2012723003 -208685715 1965110275
            -1648389341 -1207969861 -1532023285 -2068848597</internalNodes>
          <leafValues>
            -2.9938983917236328e-01 3.2542988657951355e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 72 -1025850814 -226835760 -10485921 -537564929
            1390123691 -201328836 -1152983169 -229685189</internalNodes>
          <leafValues>
            -2.8262135386466980e-01 3.3869415521621704e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 300 186646459 772435664 -277088299 -811293964 510621556
            -2003902767 768513520 -858982156</internalNodes>
          <leafValues>
            -3.5066390037536621e-01 2.6261052489280701e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 277 -1901527179 -1079583240 1608384509 -1214321156
            -1475544131 -1618224659 -1350140680 -1343693700</internalNodes>
          <leafValues>
            -3.1748643517494202e-01 2.9679971933364868e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 57 -1370792705 -151110594 137920254 505970139 -16908198
            -119123852 2113992958 -27118721</internalNodes>
          <leafValues>
            -2.9851835966110229e-01 3.1736648082733154e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 34 -421870864 -121460494 -69399301 -37695628 1854146044
            -572886810 -1662657067 -2754955</internalNodes>
          <leafValues>
            -3.1197559833526611e-01 2.9238173365592957e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 20 -395053825 -1172089085 -20972641 -36479168 16188668
            170858277 1862209791 -43004073</internalNodes>
          <leafValues>
            -2.6653704047203064e-01 3.6648163199424744e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 162 -876099570 -1028677841 346016622 -791218210
            -798236064 1141175622 -19929374 -733615388</internalNodes>
          <leafValues>
            -3.3825182914733887e-01 2.6917472481727600e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 314 268538656 1006620135 2070388735 2065690825
            -1156937026 -1545866585 -350495013 193696921</internalNodes>
          <leafValues>
            -3.5214114189147949e-01 2.6037210226058960e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 344 649287281 -1970393800 -524355 1006436977
            -1880457436 -1364230690 -575458832 -1711323724</internalNodes>
          <leafValues>
            -3.5447227954864502e-01 2.6205524802207947e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 6 -->
    <_>
      <maxWeakCount>19</maxWeakCount>
      <stageThreshold>-1.5306358337402344e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 325 -14630984 -4473090 -1074397189 -33964034 -72302598
            -1074020356 -1073741825 798731006</internalNodes>
          <leafValues>
            -3.5602328181266785e-01 4.2827311158180237e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 130 -551551009 486645509 1899748147 355677975 222889183
            -1073921969 -8405249 863993727</internalNodes>
          <leafValues>
            -3.4908288717269897e-01 3.5527125000953674e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 98 1599900924 -103183170 2077315071 -8544581 2147076347
            -348676 -6751554 1039993019</internalNodes>
          <leafValues>
            -4.1958260536193848e-01 2.6830750703811646e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 341 -551306566 -1928790024 409301104 -570567172
            155380464 201374866 1307966709 -840957953</internalNodes>
          <leafValues>
            -2.9362049698829651e-01 3.5112550854682922e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 159 -25168073 792404277 -1048577 -1077938185 -50397955
            -1125161865 -118697795 -17040197</internalNodes>
          <leafValues>
            -2.4668307602405548e-01 4.3156060576438904e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 102 -254483728 -134352193 1078993009 -438445569
            -145493001 236195840 2010188607 803207983</internalNodes>
          <leafValues>
            -4.0355175733566284e-01 2.2437272965908051e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 270 543941090 32109282 -158607600 -539249547 1450174403
            -2037673760 1430348867 -706789133</internalNodes>
          <leafValues>
            -3.8295698165893555e-01 2.4519409239292145e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 194 1820081525 1073508181 -237185025 -1627884305
            -1208341124 -1549315 786959612 -1360889776</internalNodes>
          <leafValues>
            -3.9130604267120361e-01 2.4020861089229584e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 123 -126746406 -16910241 1317990340 2140139227
            -152170248 -665943862 -494767108 -1308598001</internalNodes>
          <leafValues>
            -3.3991897106170654e-01 2.5616830587387085e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 169 -25173426 1157590007 -1462405486 9141882 1563902020
            1986233998 -121111822 1375206347</internalNodes>
          <leafValues>
            -3.9309167861938477e-01 2.3203048110008240e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 119 -2109765118 -1549312122 1974331919 -15794219
            -68435067 -10513465 -203437628 -207886460</internalNodes>
          <leafValues>
            -3.2824972271919250e-01 2.5953757762908936e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 293 532419764 -2413864 1004477621 931662384 -1970602497
            -1095996537 -839968773 -1411438117</internalNodes>
          <leafValues>
            -4.1732871532440186e-01 2.0762878656387329e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 139 1857947903 1492633971 -1101013025 975886606
            1350846468 -121757915 -130875788 -869224848</internalNodes>
          <leafValues>
            -3.4800508618354797e-01 2.4446432292461395e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 103 -2466598 -4662086 -978313286 -784675555 454565560
            487593980 1603337147 -7147781</internalNodes>
          <leafValues>
            -2.7047672867774963e-01 3.1779998540878296e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 109 -1561869790 120520547 1657708544 -1898513430
            -1614825521 2092432685 1538717248 -289931533</internalNodes>
          <leafValues>
            -4.0362039208412170e-01 2.1034663915634155e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 180 1084531215 -707808258 -1210116245 -439570458
            897553343 1907884029 -135266566 -188437888</internalNodes>
          <leafValues>
            -2.8848439455032349e-01 2.9848706722259521e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 70 -526324565 -2473047 -1752530953 -720765307
            -829754181 -88588561 725799882 -1423966207</internalNodes>
          <leafValues>
            -2.9072195291519165e-01 2.8631457686424255e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 186 -1035688188 -2075351177 1742138910 -1361646049
            2038259221 1281666845 -10016229 -927864043</internalNodes>
          <leafValues>
            -4.0738871693611145e-01 2.1433149278163910e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 111 -193049515 -579904651 -1078067337 -1125110406
            657196408 -308999 -269432520 -1626310360</internalNodes>
          <leafValues>
            -3.4779027104377747e-01 2.4815729260444641e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 7 -->
    <_>
      <maxWeakCount>19</maxWeakCount>
      <stageThreshold>-1.5291346311569214e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 251 -13959169 -28639242 -402655813 -34537805 256249855
            -568918017 1146377727 1413838847</internalNodes>
          <leafValues>
            -3.0813953280448914e-01 4.0832179784774780e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 58 -2117889 -6785 -88080641 -1078673830 1592918250
            -130866 -28588321 -67420082</internalNodes>
          <leafValues>
            -2.4101538956165314e-01 4.6791258454322815e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 104 63906446 1607320575 -243471634 -35783681
            -1106396194 1430561736 -201721409 -168502531</internalNodes>
          <leafValues>
            -4.4818142056465149e-01 2.3805019259452820e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 313 -201338603 -10642115 1604312349 -1120184061
            -570443331 -16641 -588542516 -1936978931</internalNodes>
          <leafValues>
            -2.7106782793998718e-01 3.9553779363632202e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 42 -1358982349 -1862229200 -537145985 -1122992815
            178914341 -1439598077 1593789951 -272634817</internalNodes>
          <leafValues>
            -3.2034155726432800e-01 3.0296289920806885e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 318 -1440743437 235588504 -10494472 -1887027976
            -1288767504 -1398099527 -843000652 -863175440</internalNodes>
          <leafValues>
            -3.0287060141563416e-01 3.1029415130615234e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 292 -555045126 -361514 762749686 -139778 235667694
            -1093395212 -11471106 -6309889</internalNodes>
          <leafValues>
            -3.0765682458877563e-01 3.1880700588226318e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 196 250872917 -68273685 -1078051329 -1082508571
            -1091283460 -1168712 -1417288 -1363666432</internalNodes>
          <leafValues>
            -3.3388796448707581e-01 2.7029579877853394e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 219 -290476795 -465649810 -35918852 -289735302
            -34138796 -84935682 -318770912 -857082906</internalNodes>
          <leafValues>
            -3.2501670718193054e-01 2.7079197764396667e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 351 255542577 -1383080455 -134234123 -35669132
            225794545 -1073809475 501579485 -43919</internalNodes>
          <leafValues>
            -3.1916373968124390e-01 2.6903003454208374e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 133 215553112 -1107551056 251485533 -1117840291
            -23183686 -1157685096 148380684 -1359280118</internalNodes>
          <leafValues>
            -3.4327763319015503e-01 2.6013481616973877e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 229 1781507627 11182978 -1532302414 -453707565
            2112190021 1230535507 1940017765 -723454538</internalNodes>
          <leafValues>
            -4.0361833572387695e-01 2.0124079287052155e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 35 1659037435 -1074583282 -585441281 -1142706330
            -487432961 -50689038 -752619781 2119369219</internalNodes>
          <leafValues>
            -2.8866782784461975e-01 2.8524476289749146e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 5 -277098501 -426275 -537657353 -80707839 148375995
            -1443515977 -1360028481 -290306189</internalNodes>
          <leafValues>
            -2.2325216233730316e-01 4.0045592188835144e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 342 -12845061 -1146450256 1643999573 -578956079
            -1430327361 -1364660568 1068039673 -826557192</internalNodes>
          <leafValues>
            -1.7625236511230469e-01 4.5335152745246887e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 246 -5334 16170343 -1563618650 -559235090 -213913050
            543944397 -82347060 -186319018</internalNodes>
          <leafValues>
            -3.3681878447532654e-01 2.5925871729850769e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 200 706711043 240052095 -1057817422 -288572133
            2104128000 107326306 -142977008 -184549550</internalNodes>
          <leafValues>
            -4.0993413329124451e-01 1.9274616241455078e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 207 678842645 -554033765 -299117059 -2195490 787765744
            -85783809 801398270 -1342553048</internalNodes>
          <leafValues>
            -3.0899345874786377e-01 2.7749294042587280e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 239 -352653541 -1464975560 -1711377924 1006431154
            -1914849861 -1997605505 -723791652 -2005992518</internalNodes>
          <leafValues>
            -2.7373707294464111e-01 3.1265005469322205e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 8 -->
    <_>
      <maxWeakCount>20</maxWeakCount>
      <stageThreshold>-1.2624083757400513e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 152 -536896171 528166667 -89134213 863190787 -10787
            -42746009 -135004165 861372191</internalNodes>
          <leafValues>
            -3.0395314097404480e-01 4.0150311589241028e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 316 1058061817 1069530432 1072045437 -779 -1079460102
            -1078220624 -1086607877 -1347745096</internalNodes>
          <leafValues>
            -3.1513056159019470e-01 3.4505581855773926e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 231 -681586977 321007414 1157254391 -135793561
            1280068831 961635295 1153779199 1107294463</internalNodes>
          <leafValues>
            -3.4076648950576782e-01 2.9044425487518311e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 281 13565963 -1605663761 -1068499969 -1551382549
            -957611569 -88150085 -486669105 -489698353</internalNodes>
          <leafValues>
            -2.9677531123161316e-01 3.0763381719589233e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 326 -1053826398 -721436430 -280231937 -975176521
            -409225541 -71593733 -6291717 -1413486341</internalNodes>
          <leafValues>
            -2.8983283042907715e-01 3.1503608822822571e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 183 1163896751 1977614331 671051643 -138674226
            2144851967 1470987751 -58725393 -188420220</internalNodes>
          <leafValues>
            -2.0130541920661926e-01 3.9263197779655457e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 38 -16788225 -25265109 -1172116865 974286447 -18980838
            -2269836 -349111681 -30245109</internalNodes>
          <leafValues>
            -2.6475778222084045e-01 3.2654273509979248e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 84 1433881682 1597703946 1291197240 -9438094 1526162906
            -394281474 -50917514 -414</internalNodes>
          <leafValues>
            -2.9093095660209656e-01 2.9709726572036743e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 49 -603945842 -1315963413 -414862337 -9040081
            -925350721 -117485633 -68928293 841171459</internalNodes>
          <leafValues>
            -3.6999693512916565e-01 2.1880353987216949e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 335 -235151561 906503990 -6316714 948108800 -319324675
            -16843053 -603980191 -117506534</internalNodes>
          <leafValues>
            -2.7334323525428772e-01 3.0180162191390991e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 66 -1413764333 724760368 601747389 -6095968 37694209
            -1342441059 92217889 -273154057</internalNodes>
          <leafValues>
            -2.6323935389518738e-01 2.9735177755355835e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 163 -335559766 -453021782 -323533214 1893203877
            -270539322 17588935 -186715916 -252379394</internalNodes>
          <leafValues>
            -3.0761030316352844e-01 2.5141632556915283e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 328 -13631567 -1212417327 403124577 89128053 905935803
            805306111 455804405 -1913655045</internalNodes>
          <leafValues>
            -2.4577388167381287e-01 3.1640377640724182e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 338 -1805103104 -204178216 -1109225537 -645859592
            -1179295532 -17539075 -1342640193 -1085605221</internalNodes>
          <leafValues>
            -2.7292105555534363e-01 2.8334984183311462e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 113 184427503 2077949879 2130628607 -1972228222
            267784191 -122913 -19677458 -87003834</internalNodes>
          <leafValues>
            -2.6322066783905029e-01 2.9420077800750732e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 81 105097999 1542832493 298852512 -33591146 -7371798
            1171296972 -159057160 -792331574</internalNodes>
          <leafValues>
            -3.1270226836204529e-01 2.4914608895778656e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 287 -1423199312 -1691103151 -1814751664 -1613770752
            990048439 1006682272 1472720818 -805482223</internalNodes>
          <leafValues>
            -3.1640163064002991e-01 2.3388214409351349e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 18 706367007 709894941 -1978089477 -1976242714
            1047418708 -9413867 -8999074 -94175241</internalNodes>
          <leafValues>
            -3.4782758355140686e-01 2.1951326727867126e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 320 -1074273477 -1349648528 -26397387 -305765168
            -1750336135 -1414551302 -820448067 -1996649316</internalNodes>
          <leafValues>
            -2.4355542659759521e-01 3.1015169620513916e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 151 -25443838 1937727008 -398471218 40370095 -231164274
            -1339821902 -97325926 -88867841</internalNodes>
          <leafValues>
            -2.6998028159141541e-01 2.8467071056365967e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 9 -->
    <_>
      <maxWeakCount>19</maxWeakCount>
      <stageThreshold>-1.3592376708984375e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 348 -995118944 -930416974 345887952 -570558340 88531328
            -1945218639 -587737611 -570425861</internalNodes>
          <leafValues>
            -2.9777532815933228e-01 4.2831858992576599e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 253 -67110358 1893122090 1893397120 -86249774
            1180660851 7567942 1349997680 -134220065</internalNodes>
          <leafValues>
            -3.9622232317924500e-01 2.9791569709777832e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 75 -539240230 -648023297 469764112 1593868287 -11661110
            1532777684 -552444965 -14942209</internalNodes>
          <leafValues>
            -2.9889813065528870e-01 3.6522966623306274e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 289 -67125368 -79955026 -2020348961 -35793731
            -1077342273 -616575495 -1089623397 497024223</internalNodes>
          <leafValues>
            -3.4431287646293640e-01 2.9128664731979370e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 166 1384071754 1113049038 -758590950 1895815884
            -763411890 1209352476 -89797814 -220206337</internalNodes>
          <leafValues>
            -4.1682410240173340e-01 2.1082815527915955e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 105 -254496094 -18879841 14000245 771751931 -11010061
            134742528 -8912901 929018859</internalNodes>
          <leafValues>
            -3.7486401200294495e-01 2.4162678420543671e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 179 -654313675 742329655 -262625 1060896639 -390366980
            74072179 -253568780 -289418469</internalNodes>
          <leafValues>
            -2.5083890557289124e-01 3.3719986677169800e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 315 -73668360 1565910264 2081358045 2146916605
            -1621517064 357571729 -1480983299 -24065</internalNodes>
          <leafValues>
            -2.4764552712440491e-01 3.5451245307922363e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 301 -811597901 -1364218069 -1648402160 -325060800
            -1976643936 -1364209682 -975732736 -856689421</internalNodes>
          <leafValues>
            -2.7585402131080627e-01 3.0668458342552185e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 285 -16801110 1942398638 2139043338 1884201710
            1565129044 1073771567 1927598335 1089267455</internalNodes>
          <leafValues>
            -3.5849529504776001e-01 2.3467011749744415e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 124 -552751172 -106362566 379098395 -619987142
            2056910831 -256982033 -448484726 991891214</internalNodes>
          <leafValues>
            -3.6114555597305298e-01 2.2252097725868225e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 28 -922792361 135407106 915543278 -1441729929
            -209763771 -33465743 -85987393 -79495181</internalNodes>
          <leafValues>
            -2.9683136940002441e-01 2.8010207414627075e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 174 -65566202 5422903 -1572145044 -617562398 -77657088
            1967600929 854335524 -8915084</internalNodes>
          <leafValues>
            -3.7435227632522583e-01 2.1536998450756073e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 22 -487912721 -800642866 2009588683 -1392279546
            1656090575 1071632022 -2052634693 -2044509181</internalNodes>
          <leafValues>
            -3.0010569095611572e-01 2.6017066836357117e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 226 -741397974 -774903581 1081934016 -260181765
            2068804966 -935049268 1908400875 -168901016</internalNodes>
          <leafValues>
            -2.7382543683052063e-01 2.9915502667427063e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 44 1817520043 -1431149805 -35670017 848351552
            1036462355 -1879155406 -302153961 -67637897</internalNodes>
          <leafValues>
            -3.3639410138130188e-01 2.2802397608757019e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 311 -1330405232 -1146452984 -920726316 -1147304716
            -1089883143 -1260027579 -1255145527 -1869873152</internalNodes>
          <leafValues>
            -3.6143729090690613e-01 2.2017471492290497e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 255 -488642849 338857182 1447921202 -691318883
            1085163095 23216146 1883555028 -209735809</internalNodes>
          <leafValues>
            -2.8023749589920044e-01 2.8233993053436279e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 6 -857296913 -38559749 -1123606723 -1326175183
            214765461 1826707963 1609522143 250940451</internalNodes>
          <leafValues>
            -2.9626613855361938e-01 2.7211683988571167e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 10 -->
    <_>
      <maxWeakCount>20</maxWeakCount>
      <stageThreshold>-1.2654424905776978e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 274 -152045826 -542116132 -671384098 -536871169
            71611552 4511120 1691862270 2144334079</internalNodes>
          <leafValues>
            -3.2745066285133362e-01 3.7415373325347900e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 55 -562049324 1347689172 838916858 323023775 -10528232
            1547698176 -79496197 -16257057</internalNodes>
          <leafValues>
            -3.2601267099380493e-01 3.1999495625495911e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 155 -150864973 823197205 855899696 1897004151
            1359478447 239799783 -1119879173 2071721983</internalNodes>
          <leafValues>
            -3.1169268488883972e-01 3.2441514730453491e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 131 -8122216 -1694493696 1544425874 1971917534
            1494268856 974434736 -47924226 -1081103685</internalNodes>
          <leafValues>
            -3.4951829910278320e-01 2.8645709156990051e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 2 -148702222 -39652368 -926160348 -412
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

