# Documentation for `data/haarcascades/haarcascade_frontalcatface.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_frontalcatface.xml`
- **File Name**: `haarcascade_frontalcatface.xml`
- **File Size**: 411,388 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_frontalcatface.xml](../../data/haarcascades/haarcascade_frontalcatface.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
 A frontal cat face detector using the basic set of Haar features, i.e.
 horizontal and vertical features but not diagonal features.

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
    <mode>BASIC</mode></featureParams>
  <stageNum>20</stageNum>
  <stages>
    <!-- stage 0 -->
    <_>
      <maxWeakCount>16</maxWeakCount>
      <stageThreshold>-1.4806525707244873e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 472 -1.5126220881938934e-02</internalNodes>
          <leafValues>
            7.5887596607208252e-01 -3.4230688214302063e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 839 3.9337221533060074e-03</internalNodes>
          <leafValues>
            -3.3288389444351196e-01 5.2361363172531128e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 858 -1.5044892206788063e-02</internalNodes>
          <leafValues>
            5.5565774440765381e-01 -2.2505992650985718e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 387 -1.2927042320370674e-02</internalNodes>
          <leafValues>
            5.7442700862884521e-01 -1.9708566367626190e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 137 5.5960696190595627e-03</internalNodes>
          <leafValues>
            -3.0430641770362854e-01 4.0241482853889465e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 207 1.5758406370878220e-02</internalNodes>
          <leafValues>
            -1.9767063856124878e-01 4.5033392310142517e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 678 2.4262722581624985e-02</internalNodes>
          <leafValues>
            -1.6931040585041046e-01 5.9707510471343994e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 267 -3.5242564976215363e-02</internalNodes>
          <leafValues>
            6.5973556041717529e-01 -1.4519356191158295e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 687 2.6568008586764336e-02</internalNodes>
          <leafValues>
            -1.3476610183715820e-01 5.4296624660491943e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 228 4.7154121100902557e-02</internalNodes>
          <leafValues>
            -1.7337851226329803e-01 4.6071702241897583e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 925 -5.3081759251654148e-03</internalNodes>
          <leafValues>
            5.4976856708526611e-01 -1.1913410574197769e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 608 5.3415738046169281e-02</internalNodes>
          <leafValues>
            -1.2382411211729050e-01 6.3972741365432739e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 671 -3.0798995867371559e-03</internalNodes>
          <leafValues>
            -8.2048600912094116e-01 1.0249497741460800e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 676 -2.3766520898789167e-03</internalNodes>
          <leafValues>
            -7.0665025711059570e-01 6.7025005817413330e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 180 1.1965663870796561e-03</internalNodes>
          <leafValues>
            -2.4753804504871368e-01 3.0198124051094055e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 830 -4.2106406763195992e-03</internalNodes>
          <leafValues>
            3.8455343246459961e-01 -1.8334107100963593e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 1 -->
    <_>
      <maxWeakCount>26</maxWeakCount>
      <stageThreshold>-1.4618960618972778e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 725 1.0133055038750172e-02</internalNodes>
          <leafValues>
            -2.8207325935363770e-01 6.2703561782836914e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 356 3.8468956947326660e-02</internalNodes>
          <leafValues>
            -1.4483113586902618e-01 7.4971008300781250e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 2 -3.7523733917623758e-03</internalNodes>
          <leafValues>
            4.2959973216056824e-01 -2.1445912122726440e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 844 9.9978316575288773e-04</internalNodes>
          <leafValues>
            -1.9259409606456757e-01 4.2325544357299805e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 387 -1.6786376014351845e-02</internalNodes>
          <leafValues>
            5.0582861900329590e-01 -1.8607729673385620e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 208 3.0330579727888107e-02</internalNodes>
          <leafValues>
            -2.1100421249866486e-01 4.2819553613662720e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 206 1.5150709077715874e-02</internalNodes>
          <leafValues>
            -2.1129198372364044e-01 3.6263525485992432e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 451 -3.6349350120872259e-03</internalNodes>
          <leafValues>
            3.9500275254249573e-01 -1.8650630116462708e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 270 -7.2061517275869846e-03</internalNodes>
          <leafValues>
            -7.2816300392150879e-01 1.1153221875429153e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 866 -2.0212728530168533e-02</internalNodes>
          <leafValues>
            5.6296736001968384e-01 -1.2056054919958115e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 265 2.5640423409640789e-03</internalNodes>
          <leafValues>
            -2.3753854632377625e-01 3.5794413089752197e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 230 -6.2726587057113647e-03</internalNodes>
          <leafValues>
            -6.7750877141952515e-01 1.2570948898792267e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 126 7.8710336238145828e-03</internalNodes>
          <leafValues>
            6.9211356341838837e-02 -7.6449161767959595e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 306 5.9134580194950104e-02</internalNodes>
          <leafValues>
            -1.7324967682361603e-01 3.3361187577247620e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 185 -2.8770491480827332e-03</internalNodes>
          <leafValues>
            3.6101511120796204e-01 -1.6122241318225861e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 388 -5.7046953588724136e-03</internalNodes>
          <leafValues>
            -6.7659336328506470e-01 8.4153175354003906e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 13 -7.8070178627967834e-02</internalNodes>
          <leafValues>
            6.0763663053512573e-01 -1.1037797480821609e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 321 6.5858578309416771e-03</internalNodes>
          <leafValues>
            9.3060031533241272e-02 -7.0068693161010742e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 796 -2.0920131355524063e-03</internalNodes>
          <leafValues>
            2.8173315525054932e-01 -1.8406434357166290e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 578 -2.1252598613500595e-02</internalNodes>
          <leafValues>
            3.9672371745109558e-01 -1.5127600729465485e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 770 -3.2937981188297272e-02</internalNodes>
          <leafValues>
            3.9487251639366150e-01 -1.3228580355644226e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1016 4.9491915851831436e-03</internalNodes>
          <leafValues>
            1.1234261840581894e-01 -4.7414371371269226e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 215 3.4271054901182652e-03</internalNodes>
          <leafValues>
            7.8623600304126740e-02 -5.7828009128570557e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 200 -6.0859560035169125e-03</internalNodes>
          <leafValues>
            -5.0091904401779175e-01 9.1926425695419312e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 990 1.2116413563489914e-02</internalNodes>
          <leafValues>
            -1.7154470086097717e-01 2.6759135723114014e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 456 8.2814376801252365e-03</internalNodes>
          <leafValues>
            -1.2938241660594940e-01 3.5665917396545410e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 2 -->
    <_>
      <maxWeakCount>26</maxWeakCount>
      <stageThreshold>-1.4103703498840332e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 532 -1.0988018475472927e-02</internalNodes>
          <leafValues>
            6.4358645677566528e-01 -2.3149165511131287e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 750 -7.8163212165236473e-03</internalNodes>
          <leafValues>
            5.4850798845291138e-01 -1.7881108820438385e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 289 7.1337133646011353e-02</internalNodes>
          <leafValues>
            -1.7631703615188599e-01 4.5873588323593140e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 549 5.2656695246696472e-02</internalNodes>
          <leafValues>
            -1.3836050033569336e-01 5.6253266334533691e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 8 1.5166129916906357e-02</internalNodes>
          <leafValues>
            -2.0990008115768433e-01 4.0483391284942627e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 970 -1.4538960531353951e-03</internalNodes>
          <leafValues>
            3.3692672848701477e-01 -2.1745139360427856e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 875 1.1136244982481003e-02</internalNodes>
          <leafValues>
            -1.5003634989261627e-01 5.2208083868026733e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 925 -3.3187635708600283e-03</internalNodes>
          <leafValues>
            3.9145255088806152e-01 -1.9418042898178101e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 485 4.9791105091571808e-02</internalNodes>
          <leafValues>
            -1.0192432254552841e-01 5.4612094163894653e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 828 4.3476112186908722e-02</internalNodes>
          <leafValues>
            -1.2768918275833130e-01 5.0825607776641846e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 719 -2.8149634599685669e-03</internalNodes>
          <leafValues>
            -7.0453292131423950e-01 1.2536850571632385e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 846 1.6101204091683030e-03</internalNodes>
          <leafValues>
            -2.6965174078941345e-01 2.2737979888916016e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 715 -1.5866891480982304e-03</internalNodes>
          <leafValues>
            -6.6891485452651978e-01 1.1686278134584427e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 677 -3.2338392920792103e-03</internalNodes>
          <leafValues>
            -6.7284232378005981e-01 6.6228114068508148e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 479 -9.9909156560897827e-03</internalNodes>
          <leafValues>
            3.6961549520492554e-01 -1.5993835031986237e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 350 4.8409838229417801e-02</internalNodes>
          <leafValues>
            -1.0068884491920471e-01 5.0648134946823120e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 273 8.0585200339555740e-03</internalNodes>
          <leafValues>
            -1.6782654821872711e-01 3.5382467508316040e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 338 -1.1718695983290672e-02</internalNodes>
          <leafValues>
            4.3832498788833618e-01 -1.2780784070491791e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 594 5.7147610932588577e-03</internalNodes>
          <leafValues>
            7.5814604759216309e-02 -7.2597140073776245e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 603 -2.0917234942317009e-03</internalNodes>
          <leafValues>
            -6.0916984081268311e-01 8.4811411798000336e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 855 5.7651996612548828e-03</internalNodes>
          <leafValues>
            -1.9243443012237549e-01 2.8976503014564514e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 565 -2.8093710541725159e-02</internalNodes>
          <leafValues>
            5.4229170083999634e-01 -1.0005526244640350e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 136 8.9291334152221680e-03</internalNodes>
          <leafValues>
            8.3808921277523041e-02 -6.3219338655471802e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 268 -5.1958961412310600e-03</internalNodes>
          <leafValues>
            -5.4964137077331543e-01 7.9588212072849274e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 95 9.2318728566169739e-03</internalNodes>
          <leafValues>
            -1.2818163633346558e-01 4.2056322097778320e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 964 -2.0556427538394928e-02</internalNodes>
          <leafValues>
            3.2048463821411133e-01 -1.3858842849731445e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 3 -->
    <_>
      <maxWeakCount>35</maxWeakCount>
      <stageThreshold>-1.4265209436416626e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 683 1.8821602687239647e-02</internalNodes>
          <leafValues>
            -1.7807419598102570e-01 5.9040957689285278e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 471 -9.5066539943218231e-03</internalNodes>
          <leafValues>
            5.0587177276611328e-01 -1.7767964303493500e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 884 1.3296608813107014e-03</internalNodes>
          <leafValues>
            -1.6886346042156219e-01 3.6326614022254944e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 473 3.5266026854515076e-02</internalNodes>
          <leafValues>
            -1.1824090778827667e-01 5.8951085805892944e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 340 1.7804209142923355e-02</internalNodes>
          <leafValues>
            -1.4211210608482361e-01 5.1762068271636963e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1001 4.7029324923641980e-04</internalNodes>
          <leafValues>
            -2.4296821653842926e-01 2.5087893009185791e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 182 7.1838246658444405e-03</internalNodes>
          <leafValues>
            9.2609666287899017e-02 -6.7694115638732910e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 390 -5.7565318420529366e-03</internalNodes>
          <leafValues>
            -7.3053181171417236e-01 8.2794629037380219e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 203 2.0850602537393570e-02</internalNodes>
          <leafValues>
            -1.7353208363056183e-01 3.3287450671195984e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 805 3.1848326325416565e-03</internalNodes>
          <leafValues>
            -2.0941653847694397e-01 2.6059800386428833e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 234 -7.5752258300781250e-02</internalNodes>
          <leafValues>
            5.1588213443756104e-01 -1.0057342052459717e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 5 2.8725115582346916e-02</internalNodes>
          <leafValues>
            -1.5012685954570770e-01 4.1436919569969177e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 175 -1.7325732856988907e-02</internalNodes>
          <leafValues>
            3.8678762316703796e-01 -1.3586300611495972e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 47 -3.2187681645154953e-03</internalNodes>
          <leafValues>
            -5.1590150594711304e-01 1.1511231958866119e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1020 -6.1595086008310318e-03</internalNodes>
          <leafValues>
            -7.0271849632263184e-01 5.5648274719715118e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 768 -8.7264683097600937e-03</internalNodes>
          <leafValues>
            2.6393634080886841e-01 -1.8446569144725800e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 57 8.1868227571249008e-03</internalNodes>
          <leafValues>
            8.0838531255722046e-02 -5.5512112379074097e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 139 -7.8468751162290573e-03</internalNodes>
          <leafValues>
            -5.7306796312332153e-01 8.3454042673110962e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 665 2.9962153639644384e-03</internalNodes>
          <leafValues>
            6.2645487487316132e-02 -5.8123600482940674e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 414 -4.3795984238386154e-03</internalNodes>
          <leafValues>
            2.2211562097072601e-01 -1.9649308919906616e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 908 -6.3172029331326485e-03</internalNodes>
          <leafValues>
            -6.6067039966583252e-01 6.4884319901466370e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 465 1.3302030274644494e-03</internalNodes>
          <leafValues>
            -1.0496762394905090e-01 4.2326071858406067e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 951 -4.3333107605576515e-03</internalNodes>
          <leafValues>
            -4.9972066283226013e-01 8.7225496768951416e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 244 -3.5346355289220810e-03</internalNodes>
          <leafValues>
            3.0818134546279907e-01 -1.4765550196170807e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 256 -8.7353587150573730e-03</internalNodes>
          <leafValues>
            -6.5214675664901733e-01 7.1881487965583801e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 491 -1.5620354562997818e-02</internalNodes>
          <leafValues>
            3.5721915960311890e-01 -1.1427627503871918e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 778 -3.9745438843965530e-03</internalNodes>
          <leafValues>
            -6.6090464591979980e-01 6.2067609280347824e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 689 -6.7040426656603813e-03</internalNodes>
          <leafValues>
            2.7337384223937988e-01 -1.4059108495712280e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 125 3.5359347239136696e-03</internalNodes>
          <leafValues>
            6.1201948672533035e-02 -6.0017114877700806e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 118 6.0818484053015709e-03</internalNodes>
          <leafValues>
            -1.5247075259685516e-01 2.4383027851581573e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 880 -7.2771648410707712e-04</internalNodes>
          <leafValues>
            3.0065426230430603e-01 -1.2037902325391769e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 643 4.6168416738510132e-03</internalNodes>
          <leafValues>
            5.5311698466539383e-02 -7.5343269109725952e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 676 2.5280299596488476e-03</internalNodes>
          <leafValues>
            5.7204965502023697e-02 -5.3993463516235352e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 878 1.5074670314788818e-02</internalNodes>
          <leafValues>
            -9.6106290817260742e-02 3.9084190130233765e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 831 -8.4932018071413040e-03</internalNodes>
          <leafValues>
            3.4130987524986267e-01 -1.4117397367954254e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 4 -->
    <_>
      <maxWeakCount>37</maxWeakCount>
      <stageThreshold>-1.3977209329605103e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 794 -2.5338861159980297e-03</internalNodes>
          <leafValues>
            5.7321399450302124e-01 -2.0396080613136292e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 588 -6.5112011507153511e-03</internalNodes>
          <leafValues>
            3.7378740310668945e-01 -2.5049039721488953e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 238 1.6318978741765022e-03</internalNodes>
          <leafValues>
            -2.1858637034893036e-01 3.5027471184730530e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 189 3.3452022820711136e-02</internalNodes>
          <leafValues>
            -1.4827065169811249e-01 4.7324529290199280e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 192 -1.1114047840237617e-02</internalNodes>
          <leafValues>
            4.1662359237670898e-01 -2.1660456061363220e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 527 -1.2996498262509704e-03</internalNodes>
          <leafValues>
            4.7613915801048279e-01 -1.6742442548274994e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 648 -3.2986078877002001e-03</internalNodes>
          <leafValues>
            -6.7662662267684937e-01 8.6653761565685272e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 4 6.6831205040216446e-03</internalNodes>
          <leafValues>
            -2.0158858597278595e-01 2.6189696788787842e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 482 2.1282089874148369e-03</internalNodes>
          <leafValues>
            -1.1156299710273743e-01 4.0097075700759888e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 682 -9.0472139418125153e-03</internalNodes>
          <leafValues>
            3.2078295946121216e-01 -1.6775439679622650e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 226 -5.3160609677433968e-03</internalNodes>
          <leafValues>
            -5.5567348003387451e-01 1.2950280308723450e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 205 7.9724024981260300e-03</internalNodes>
          <leafValues>
            -2.1466700732707977e-01 2.2514854371547699e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 920 -2.1980279125273228e-03</internalNodes>
          <leafValues>
            2.8711742162704468e-01 -1.6561916470527649e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 312 5.3897619247436523e-02</internalNodes>
          <leafValues>
            -1.4823001623153687e-01 3.4951418638229370e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 13 -7.6241128146648407e-02</internalNodes>
          <leafValues>
            6.0101884603500366e-01 -8.8328786194324493e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 129 -8.3202747628092766e-03</internalNodes>
          <leafValues>
            -7.2828358411788940e-01 8.7956465780735016e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 401 5.3778752684593201e-02</internalNodes>
          <leafValues>
            -1.0316975414752960e-01 5.0247919559478760e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 416 -1.2401826679706573e-02</internalNodes>
          <leafValues>
            2.7538898587226868e-01 -1.5569972991943359e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 986 1.3729928061366081e-02</internalNodes>
          <leafValues>
            -1.3373774290084839e-01 3.0739122629165649e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 905 -2.2788168862462044e-03</internalNodes>
          <leafValues>
            2.2555501759052277e-01 -1.9497908651828766e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 667 3.6288173869252205e-03</internalNodes>
          <leafValues>
            4.8981692641973495e-02 -7.9248648881912231e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 85 5.2453137934207916e-02</internalNodes>
          <leafValues>
            -1.3389803469181061e-01 3.2700663805007935e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 821 3.1685843132436275e-03</internalNodes>
          <leafValues>
            -1.4415425062179565e-01 2.8044179081916809e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 193 8.9051481336355209e-03</internalNodes>
          <leafValues>
            6.1227656900882721e-02 -7.0277702808380127e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 837 -1.3966157566756010e-03</internalNodes>
          <leafValues>
            4.2409667372703552e-01 -1.0888981819152832e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 271 -6.7695947363972664e-03</internalNodes>
          <leafValues>
            -5.1588076353073120e-01 8.3254821598529816e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 404 2.2157761268317699e-03</internalNodes>
          <leafValues>
            -1.3696527481079102e-01 2.8638482093811035e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 619 2.7808796148747206e-03</internalNodes>
          <leafValues>
            7.1316704154014587e-02 -6.0322999954223633e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 515 4.5836241915822029e-03</internalNodes>
          <leafValues>
            -1.2486589699983597e-01 3.2929363846778870e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1042 -5.1459800451993942e-03</internalNodes>
          <leafValues>
            -5.3781992197036743e-01 7.6631128787994385e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1043 2.4449056945741177e-03</internalNodes>
          <leafValues>
            8.5920669138431549e-02 -4.0670683979988098e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 71 -2.7756379917263985e-02</internalNodes>
          <leafValues>
            3.7449231743812561e-01 -1.0538945347070694e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 809 -1.8243372440338135e-02</internalNodes>
          <leafValues>
            3.4281516075134277e-01 -9.9502928555011749e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 372 3.8416781462728977e-03</internalNodes>
          <leafValues>
            7.3987491428852081e-02 -4.8903524875640869e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 376 -1.2322908267378807e-02</internalNodes>
          <leafValues>
            2.1036790311336517e-01 -1.5852701663970947e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 391 -4.1760304011404514e-03</internalNodes>
          <leafValues>
            3.1288132071495056e-01 -1.1697492748498917e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 859 -2.8026863932609558e-02</internalNodes>
          <leafValues>
            3.3711743354797363e-01 -1.2294299900531769e-01</leafValues></_></weakClassifiers></_>
    <!-- stage 5 -->
    <_>
      <maxWeakCount>42</maxWeakCount>
      <stageThreshold>-1.3775455951690674e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 725 1.3382414355874062e-02</internalNodes>
          <leafValues>
            -1.7922241985797882e-01 5.0368404388427734e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 967 1.9935802556574345e-03</internalNodes>
          <leafValues>
            -2.5249919295310974e-01 3.5295018553733826e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 891 -1.3569685397669673e-03</internalNodes>
          <leafValues>
            4.1222429275512695e-01 -1.8140394985675812e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 911 2.5418698787689209e-03</internalNodes>
          <leafValues>
            -2.3195247352123260e-01 2.5945317745208740e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 362 1.1867792345583439e-03</internalNodes>
          <leafValues>
            -1.1509010195732117e-01 4.0095508098602295e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 280 -4.0491363033652306e-03</internalNodes>
          <leafValues>
            -7.6275551319122314e-01 8.0663219094276428e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 264 2.4698153138160706e-02</internalNodes>
          <leafValues>
            -9.9053405225276947e-02 4.6469488739967346e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 832 1.3041709549725056e-02</internalNodes>
          <leafValues>
            -1.3049817085266113e-01 4.7066822648048401e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 257 -2.0927201956510544e-02</internalNodes>
          <leafValues>
            -7.2363191843032837e-01 7.5520738959312439e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 41 1.6108792275190353e-02</internalNodes>
          <leafValues>
            8.9385204017162323e-02 -5.0678378343582153e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 872 -8.6308103054761887e-03</internalNodes>
          <leafValues>
            3.1878158450126648e-01 -1.3526505231857300e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 347 1.2651814613491297e-03</internalNodes>
          <leafValues>
            -1.2344279885292053e-01 4.0271109342575073e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 735 -3.0170590616762638e-03</internalNodes>
          <leafValues>
            -5.6960099935531616e-01 7.0437252521514893e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 538 -3.5529488231986761e-03</internalNodes>
          <leafValues>
            2.0624065399169922e-01 -1.8426756560802460e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 735 2.8021419420838356e-03</internalNodes>
          <leafValues>
            7.2748780250549316e-02 -5.3796368837356567e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 447 -9.9331419914960861e-04</internalNodes>
          <leafValues>
            2.4827398359775543e-01 -1.5866567194461823e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 440 -7.1950745768845081e-03</internalNodes>
          <leafValues>
            -5.0943744182586670e-01 7.3041573166847229e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 906 -8.7737981230020523e-03</internalNodes>
          <leafValues>
            2.4838714301586151e-01 -1.5162147581577301e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 608 5.6750684976577759e-02</internalNodes>
          <leafValues>
            -8.4416143596172333e-02 4.4269657135009766e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 772 1.8110256642103195e-03</internalNodes>
          <leafValues>
            -1.7787678539752960e-01 2.2753682732582092e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 117 6.1733853071928024e-02</internalNodes>
          <leafValues>
            -1.4452947676181793e-01 2.6785543560981750e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 718 1.7999792471528053e-03</internalNodes>
          <leafValues>
            5.3869031369686127e-02 -7.0216673612594604e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 718 -1.7839821521192789e-03</internalNodes>
          <leafValues>
            -7.3474282026290894e-01 4.3809492141008377e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 795 -2.2269869223237038e-03</internalNodes>
          <leafValues>
            2.5256577134132385e-01 -1.4765015244483948e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 845 7.7408831566572189e-04</internalNodes>
          <leafValues>
            -1.6781617701053619e-01 2.5267890095710754e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 710 9.6316616982221603e-03</internalNodes>
          <leafValues>
            5.8525908738374710e-02 -6.3684886693954468e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 181 -1.1892126873135567e-02</internalNodes>
          <leafValues>
            2.6363542675971985e-01 -1.4106634259223938e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 326 4.8407237976789474e-02</internalNodes>
          <leafValues>
            -1.0837136209011078e-01 3.6018091440200806e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 572 -1.0315750539302826e-01</internalNodes>
          <leafValues>
            -7.3309695720672607e-01 6.4976803958415985e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 415 -2.6544972788542509e-03</internalNodes>
          <leafValues>
            2.7709859609603882e-01 -1.3764445483684540e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1033 -4.8850756138563156e-03</internalNodes>
          <leafValues>
            -5.0026285648345947e-01 6.8797707557678223e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 299 -1.1310833506286144e-02</internalNodes>
          <leafValues>
            2.5653550028800964e-01 -1.3755545020103455e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 152 -3.8394361734390259e-02</internalNodes>
          <leafValues>
            2.6404461264610291e-01 -1.3614650070667267e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 486 5.8298893272876740e-03</internalNodes>
          <leafValues>
            6.0382172465324402e-02 -5.9578329324722290e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 393 2.2631133906543255e-03</internalNodes>
          <leafValues>
            -1.0302778333425522e-01 3.4782779216766357e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 629 -1.8709234893321991e-02</internalNodes>
          <leafValues>
            -7.6758313179016113e-01 4.6181913465261459e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 67 3.7359733134508133e-02</internalNodes>
          <leafValues>
            -1.3407541811466217e-01 2.5607112050056458e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 504 -5.3099328652024269e-03</internalNodes>
          <leafValues>
            -6.9016355276107788e-01 4.7683756798505783e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 527 -1.5396323287859559e-03</internalNodes>
          <leafValues>
            3.7874689698219299e-01 -9.2663109302520752e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 470 -2.6333518326282501e-03</internalNodes>
          <leafValues>
            2.9358446598052979e-01 -1.2460695207118988e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 171 1.6515964642167091e-02</internalNodes>
          <leafValues>
            -1.4082725346088409e-01 2.3664724826812744e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 681 -4.4658156111836433e-03</internalNodes>
          <leafValues>
            -5.9253305196762085e-01 5.5994171649217606e-02</leafValues></_></weakClassifiers></_>
    <!-- stage 6 -->
    <_>
      <maxWeakCount>50</maxWeakCount>
      <stageThreshold>-1.3835698366165161e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 898 1.5156399458646774e-03</internalNodes>
          <leafValues>
            -1.0024535655975342e-01 5.8807808160781860e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 802 -3.5168868489563465e-03</internalNodes>
          <leafValues>
            4.0972998738288879e-01 -1.6088742017745972e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 180 2.3035616613924503e-03</internalNodes>
          <leafValues>
            -1.8985269963741302e-01 2.9883998632431030e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 254 4.5840561389923096e-02</internalNodes>
          <leafValues>
            -1.4383240044116974e-01 4.7528687119483948e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 405 5.5156396701931953e-03</internalNodes>
          <leafValues>
            -1.7356806993484497e-01 3.4583050012588501e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 436 3.9731184951961040e-03</internalNodes>
          <leafValues>
            7.8886620700359344e-02 -5.6442558765411377e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 412 -5.6995991617441177e-03</internalNodes>
          <leafValues>
            -4.7576662898063660e-01 9.4875656068325043e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 539 -9.6501735970377922e-03</internalNodes>
          <leafValues>
            2.3381656408309937e-01 -1.8310526013374329e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 209 6.1656545847654343e-02</internalNodes>
          <leafValues>
            -1.4697165787220001e-01 3.6247691512107849e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 398 1.1418928205966949e-01</internalNodes>
          <leafValues>
            -8.8033527135848999e-02 4.4633501768112183e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 3 -1.1903396807610989e-02</internalNodes>
          <leafValues>
            3.3496665954589844e-01 -1.2121009081602097e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 546 -4.1371315717697144e-02</internalNodes>
          <leafValues>
            4.1400006413459778e-01 -9.7229279577732086e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 380 7.8342631459236145e-03</internalNodes>
          <leafValues>
            -1.6631671786308289e-01 2.5738984346389771e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 304 -4.5139621943235397e-03</internalNodes>
          <leafValues>
            -4.6883803606033325e-01 8.7662570178508759e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 929 1.5914421528577805e-03</internalNodes>
          <leafValues>
            -1.1636006087064743e-01 3.2739594578742981e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 942 -5.2607608959078789e-03</internalNodes>
          <leafValues>
            -6.7755740880966187e-01 5.1752120256423950e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 941 3.1824512407183647e-03</internalNodes>
          <leafValues>
            5.2379645407199860e-02 -6.0918039083480835e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 939 -3.6813789047300816e-03</internalNodes>
          <leafValues>
            4.8251116275787354e-01 -9.2318780720233917e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 622 -4.3226117268204689e-03</internalNodes>
          <leafValues>
            -5.7561415433883667e-01 5.9672243893146515e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 250 -7.1843853220343590e-03</internalNodes>
          <leafValues>
            2.6631006598472595e-01 -1.4015418291091919e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 871 2.1028071641921997e-03</internalNodes>
          <leafValues>
            -1.1286304146051407e-01 3.5946926474571228e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 22 8.5248583927750587e-03</internalNodes>
          <leafValues>
            6.9424033164978027e-02 -5.2462881803512573e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 147 6.9785099476575851e-03</internalNodes>
          <leafValues>
            5.6668873876333237e-02 -5.6192052364349365e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 474 -5.2639590576291084e-03</internalNodes>
          <leafValues>
            -5.8648955821990967e-01 5.0352573394775391e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 406 2.8417459689080715e-03</internalNodes>
          <leafValues>
            -1.3425759971141815e-01 2.7325555682182312e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 394 -1.3187457807362080e-02</internalNodes>
          <leafValues>
            4.0453648567199707e-01 -9.1843754053115845e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 722 -6.7344801500439644e-03</internalNodes>
          <leafValues>
            -7.5647395849227905e-01 5.0157479941844940e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 187 2.1363141015172005e-02</internalNodes>
          <leafValues>
            4.7982390969991684e-02 -5.5388218164443970e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 623 1.6145884292200208e-03</internalNodes>
          <leafValues>
            7.9808227717876434e-02 -3.7233716249465942e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 525 -2.2595757618546486e-03</internalNodes>
          <leafValues>
            2.8343635797500610e-01 -1.1216876655817032e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 214 1.4407988637685776e-02</internalNodes>
          <leafValues>
            -1.0392460227012634e-01 3.1299999356269836e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 476 -1.4912552433088422e-03</internalNodes>
          <leafValues>
            2.8538599610328674e-01 -1.0644508898258209e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 195 9.8895151168107986e-03</internalNodes>
          <leafValues>
            5.0090074539184570e-02 -6.2053185701370239e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 115 4.2754956521093845e-03</internalNodes>
          <leafValues>
            6.5051443874835968e-02 -4.2582303285598755e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 754 -2.5489409454166889e-03</internalNodes>
          <leafValues>
            3.1278640031814575e-01 -9.9601686000823975e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 717 -6.0358326882123947e-03</internalNodes>
          <leafValues>
            2.2685267031192780e-01 -1.3849361240863800e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 875 1.1879121884703636e-02</internalNodes>
          <leafValues>
            -8.9687183499336243e-02 3.7642294168472290e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 111 1.2982923537492752e-02</internalNodes>
          <leafValues>
            4.3990727514028549e-02 -7.3371982574462891e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 993 -2.8599319048225880e-03</internalNodes>
          <leafValues>
            -4.3102917075157166e-01 5.9561621397733688e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 737 -3.5829999251291156e-04</internalNodes>
          <leafValues>
            1.7152757942676544e-01 -1.6511310636997223e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 27 2.5972571223974228e-02</internalNodes>
          <leafValues>
            -1.2855969369411469e-01 2.2820757329463959e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 516 4.2565623298287392e-03</internalNodes>
          <leafValues>
            5.7662181556224823e-02 -5.3734982013702393e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 50 -2.9159568250179291e-02</internalNodes>
          <leafValues>
            -6.3020753860473633e-01 4.0746636688709259e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 413 3.1341956928372383e-03</internalNodes>
          <leafValues>
            -8.1374719738960266e-02 4.1371321678161621e-01</leaf
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

