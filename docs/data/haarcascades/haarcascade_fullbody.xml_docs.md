# Documentation for `data/haarcascades/haarcascade_fullbody.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_fullbody.xml`
- **File Name**: `haarcascade_fullbody.xml`
- **File Size**: 476,827 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_fullbody.xml](../../data/haarcascades/haarcascade_fullbody.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
   14x28 fullbody detector (see the detailed description below). 

//////////////////////////////////////////////////////////////////////////
| Contributors License Agreement
| IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
|   By downloading, copying, installing or using the software you agree 
|   to this license.
|   If you do not agree to this license, do not download, install,
|   copy or use the software.
|
| Copyright (c) 2004, Hannes Kruppa and Bernt Schiele (ETH Zurich, Switzerland).
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

"Haar"-based Detectors For Pedestrian Detection
===============================================
by Hannes Kruppa and Bernt Schiele, ETH Zurich, Switzerland

This archive provides the following three detectors:
- upper body detector (most fun, useful in many scenarios!)
- lower body detector
- full body detector

These detectors have been successfully applied to pedestrian detection
in still images. They can be directly passed as parameters to the
program HaarFaceDetect.
NOTE: These detectors deal with frontal and backside views but not
with side views (also see "Known limitations" below).

RESEARCHERS:
If you are using any of the detectors or involved ideas please cite
this paper (available at www.vision.ethz.ch/publications/):

@InProceedings{Kruppa03-bmvc,
  author =       "Hannes Kruppa, Modesto Castrillon-Santana and Bernt Schiele",
  title =        "Fast and Robust Face Finding via Local Context."
  booktitle =    "Joint IEEE International Workshop on Visual Surveillance and Performance Evaluation of Tracking and Surveillance"
  year =         "2003",
  month =        "October"
}

COMMERCIAL:
If you have any commercial interest in this work please contact 
hkruppa@inf.ethz.ch


ADDITIONAL INFORMATION 
====================== 
Check out the demo movie, e.g. using mplayer or any (Windows/Linux-) player
that can play back .mpg movies.
Under Linux that's:
> ffplay demo.mpg
or:
> mplayer demo.mpg

The movie shows a person walking towards the camera in a realistic
indoor setting. Using ffplay or mplayer you can pause and continue the
movie by pressing the space bar.

Detections coming from the different detectors are visualized using
different line styles: 
upper body : dotted line
lower body : dashed line
full body  : solid line

You will notice that successful detections containing the target do
not sit tightly on the body but also include some of the background
left and right.  This is not a bug but accurately reflects the
employed training data which also includes portions of the background
to ensure proper silhouette representation. If you want to get a
feeling for the training data check out the CBCL data set:
http://www.ai.mit.edu/projects/cbcl/software-datasets/PedestrianData.html

There is also a small number of false alarms in this sequence.  
NOTE: This is per frame detection, not tracking (which is also one of
the reasons why it is not mislead by the person's shadow on the back
wall). 

On an Intel Xeon 1.7GHz machine the detectors operate at something
between 6Hz to 14 Hz (on 352 x 288 frames per second) depending on the
detector. The detectors work as well on much lower image resolutions
which is always an interesting possibility for speed-ups or
"coarse-to-fine" search strategies.

Additional information e.g. on training parameters, detector
combination, detecting other types of objects (e.g. cars) etc. is
available in my PhD thesis report (available end of June). Check out
www.vision.ethz.ch/kruppa/


KNOWN LIMITATIONS
==================
1) The detectors only support frontal and back views but not sideviews.
   Sideviews are trickier and it makes a lot of sense to include additional
   modalities for their detection, e.g. motion information. I recommend
   Viola and Jones' ICCV 2003 paper if this further interests you.

2) Don't expect these detectors to be as accurate as a frontal face detector.
   A frontal face as a pattern is pretty distinct with respect to other
   patterns occurring in the world (i.e. image "background"). This is not so
   for upper, lower and especially full bodies, because they have to rely
   on fragile silhouette information rather than internal (facial) features.
   Still, we found especially the upper body detector to perform amazingly well.
   In contrast to a face detector these detectors will also work at very low
   image resolutions 

Acknowledgements
================
Thanks to Martin Spengler, ETH Zurich, for providing the demo movie.
-->
<opencv_storage>
<cascade type_id="opencv-cascade-classifier"><stageType>BOOST</stageType>
  <featureType>HAAR</featureType>
  <height>28</height>
  <width>14</width>
  <stageParams>
    <maxWeakCount>107</maxWeakCount></stageParams>
  <featureParams>
    <maxCatCount>0</maxCatCount></featureParams>
  <stageNum>30</stageNum>
  <stages>
    <_>
      <maxWeakCount>9</maxWeakCount>
      <stageThreshold>-1.2288980484008789e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 0 -5.5820569396018982e-02</internalNodes>
          <leafValues>
            5.8697921037673950e-01 -6.2811422348022461e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1 -3.8861181586980820e-02</internalNodes>
          <leafValues>
            -7.0916819572448730e-01 2.6821210980415344e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 2 -2.6740878820419312e-01</internalNodes>
          <leafValues>
            8.3082962036132812e-01 -2.2599589824676514e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 3 9.6419736742973328e-02</internalNodes>
          <leafValues>
            -1.1697849631309509e-01 8.7254559993743896e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 4 -1.0798710398375988e-02</internalNodes>
          <leafValues>
            -5.7219749689102173e-01 2.5325658917427063e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 5 1.1365639977157116e-02</internalNodes>
          <leafValues>
            1.9650830328464508e-01 -7.2744637727737427e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 6 -5.0216919044032693e-04</internalNodes>
          <leafValues>
            2.4435159564018250e-01 -5.1973581314086914e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 7 -2.8462480753660202e-02</internalNodes>
          <leafValues>
            -8.3607292175292969e-01 1.1158040165901184e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 8 1.3473170110955834e-03</internalNodes>
          <leafValues>
            -3.8406538963317871e-01 2.6767989993095398e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>15</maxWeakCount>
      <stageThreshold>-1.0969949960708618e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 9 -1.0743220336735249e-02</internalNodes>
          <leafValues>
            4.7747328877449036e-01 -6.2392932176589966e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 10 -1.3188569573685527e-03</internalNodes>
          <leafValues>
            2.1242660284042358e-01 -2.4162709712982178e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 11 -5.5571161210536957e-03</internalNodes>
          <leafValues>
            3.6147859692573547e-01 -3.7251719832420349e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 12 -1.3893410563468933e-01</internalNodes>
          <leafValues>
            -6.7900502681732178e-01 1.1280310153961182e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 13 2.6465829461812973e-02</internalNodes>
          <leafValues>
            1.2474969774484634e-01 -8.2852339744567871e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 14 -8.9386843144893646e-02</internalNodes>
          <leafValues>
            7.4271762371063232e-01 -1.7019319534301758e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 15 -2.1335419267416000e-02</internalNodes>
          <leafValues>
            -7.1750187873840332e-01 1.5566180646419525e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 16 5.5709101259708405e-02</internalNodes>
          <leafValues>
            -1.5310040116310120e-01 7.1804767847061157e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 17 -6.9709950685501099e-01</internalNodes>
          <leafValues>
            8.1154191493988037e-01 -1.0886389762163162e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 18 2.0205999910831451e-01</internalNodes>
          <leafValues>
            7.6398417353630066e-02 -7.3011511564254761e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 19 -7.1882657706737518e-02</internalNodes>
          <leafValues>
            -7.1488589048385620e-01 1.6517649590969086e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 20 -1.9228760153055191e-02</internalNodes>
          <leafValues>
            -3.9868369698524475e-01 4.0557239204645157e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 21 1.1500229593366385e-03</internalNodes>
          <leafValues>
            -3.8260778784751892e-01 3.1855079531669617e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 22 2.3252779617905617e-02</internalNodes>
          <leafValues>
            5.4390400648117065e-02 -7.0669990777969360e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 23 -3.2618120894767344e-04</internalNodes>
          <leafValues>
            2.2610600292682648e-01 -4.0709879994392395e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>14</maxWeakCount>
      <stageThreshold>-1.2285970449447632e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 24 -1.2910200655460358e-01</internalNodes>
          <leafValues>
            7.6003128290176392e-01 -2.3405790328979492e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 25 6.7449256777763367e-02</internalNodes>
          <leafValues>
            1.7179529368877411e-01 -8.4364777803421021e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 26 1.2663270346820354e-02</internalNodes>
          <leafValues>
            2.2913210093975067e-01 -7.3072457313537598e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 27 -4.2741331271827221e-03</internalNodes>
          <leafValues>
            6.2420479953289032e-02 -4.0985938906669617e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 28 -2.3143950849771500e-02</internalNodes>
          <leafValues>
            -8.3971828222274780e-01 2.0115749537944794e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 29 -5.5371038615703583e-04</internalNodes>
          <leafValues>
            1.5369419753551483e-01 -4.4038110971450806e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 30 -9.5239803194999695e-03</internalNodes>
          <leafValues>
            -6.3186800479888916e-01 1.6250230371952057e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 31 2.8307670727372169e-02</internalNodes>
          <leafValues>
            -7.2599969804286957e-02 3.7919989228248596e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 32 -4.5148018747568130e-02</internalNodes>
          <leafValues>
            7.4493628740310669e-01 -1.5581710636615753e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 33 1.0014739632606506e-01</internalNodes>
          <leafValues>
            1.7949639260768890e-01 -6.4644080400466919e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 34 7.3245721869170666e-03</internalNodes>
          <leafValues>
            1.7763899266719818e-01 -5.7654058933258057e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 35 1.1875670403242111e-02</internalNodes>
          <leafValues>
            -3.1129720807075500e-01 1.6321399807929993e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 36 -2.5479039177298546e-02</internalNodes>
          <leafValues>
            6.2692481279373169e-01 -1.1333750188350677e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 37 -7.9196523874998093e-03</internalNodes>
          <leafValues>
            -7.7624428272247314e-01 1.5427610278129578e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>22</maxWeakCount>
      <stageThreshold>-1.1200269460678101e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 38 -8.5809278488159180e-01</internalNodes>
          <leafValues>
            7.8796839714050293e-01 -2.2135549783706665e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 39 -1.6491119749844074e-03</internalNodes>
          <leafValues>
            2.5673401355743408e-01 -4.3194240331649780e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 40 -2.5882309302687645e-02</internalNodes>
          <leafValues>
            -8.7551230192184448e-01 8.8385626673698425e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 41 -4.7666151076555252e-03</internalNodes>
          <leafValues>
            -4.7022369503974915e-01 2.2800800204277039e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 42 -8.3729699254035950e-02</internalNodes>
          <leafValues>
            6.3385730981826782e-01 -1.4888319373130798e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 43 -4.0685739368200302e-02</internalNodes>
          <leafValues>
            -9.3931788206100464e-01 1.0598939843475819e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 44 -5.0759920850396156e-03</internalNodes>
          <leafValues>
            -4.5554420351982117e-01 1.7864370346069336e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 45 2.3427829146385193e-03</internalNodes>
          <leafValues>
            -2.1434280276298523e-01 1.5531420707702637e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 46 2.7649151161313057e-04</internalNodes>
          <leafValues>
            -3.3348160982131958e-01 2.2780239582061768e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 47 1.6941839829087257e-02</internalNodes>
          <leafValues>
            7.4140816926956177e-02 -5.6262052059173584e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 48 4.7558981180191040e-01</internalNodes>
          <leafValues>
            -1.0861130058765411e-01 8.2985258102416992e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 49 5.8000627905130386e-03</internalNodes>
          <leafValues>
            1.3249030709266663e-01 -5.1620399951934814e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 50 -7.4477560818195343e-02</internalNodes>
          <leafValues>
            -5.5545568466186523e-01 1.2344320118427277e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 51 -3.5143009154126048e-04</internalNodes>
          <leafValues>
            6.8190753459930420e-02 -1.3616859912872314e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 52 7.3454021476209164e-03</internalNodes>
          <leafValues>
            1.3678510487079620e-01 -5.3645122051239014e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 53 -1.5471279621124268e-02</internalNodes>
          <leafValues>
            2.6180639863014221e-01 -1.0545810312032700e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 54 5.6055500172078609e-03</internalNodes>
          <leafValues>
            -2.5746351480484009e-01 2.8795930743217468e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 55 -2.4552858667448163e-04</internalNodes>
          <leafValues>
            1.0099930316209793e-01 -2.6119679212570190e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 56 -3.3138900995254517e-02</internalNodes>
          <leafValues>
            -8.3779567480087280e-01 1.1327689886093140e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 57 3.5591889172792435e-02</internalNodes>
          <leafValues>
            8.2336090505123138e-02 -6.2505662441253662e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 58 2.0834030210971832e-01</internalNodes>
          <leafValues>
            6.9524437189102173e-02 -8.6881148815155029e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 59 -2.8165400028228760e-02</internalNodes>
          <leafValues>
            -5.9799849987030029e-01 8.0329902470111847e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>25</maxWeakCount>
      <stageThreshold>-1.0664960145950317e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 60 -2.6740709319710732e-02</internalNodes>
          <leafValues>
            3.8912421464920044e-01 -4.9827679991722107e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 61 -1.2516999850049615e-03</internalNodes>
          <leafValues>
            1.3123430311679840e-01 -3.6368998885154724e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 62 -4.1634511202573776e-02</internalNodes>
          <leafValues>
            5.7444751262664795e-01 -1.3932879269123077e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 63 1.0096579790115356e-02</internalNodes>
          <leafValues>
            9.9073797464370728e-02 -2.2956989705562592e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 64 -1.9090399146080017e-02</internalNodes>
          <leafValues>
            -5.5153107643127441e-01 1.5110069513320923e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 65 -3.1481068581342697e-02</internalNodes>
          <leafValues>
            -4.5884269475936890e-01 1.7579549551010132e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 66 -1.7687549814581871e-02</internalNodes>
          <leafValues>
            4.4711831212043762e-01 -1.5292930603027344e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 67 -4.3685659766197205e-03</internalNodes>
          <leafValues>
            1.2185490131378174e-01 -1.6688570380210876e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 68 8.9326845481991768e-03</internalNodes>
          <leafValues>
            -1.3333690166473389e-01 6.3753342628479004e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 69 -5.0706309266388416e-03</internalNodes>
          <leafValues>
            -1.1220289766788483e-01 6.9824352860450745e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 70 -5.9803090989589691e-03</internalNodes>
          <leafValues>
            -5.1842898130416870e-01 1.6099199652671814e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 71 2.9967839363962412e-03</internalNodes>
          <leafValues>
            4.1065338999032974e-02 -1.9455850124359131e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 72 3.8641549181193113e-03</internalNodes>
          <leafValues>
            1.6673240065574646e-01 -4.3569779396057129e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 73 6.8349428474903107e-03</internalNodes>
          <leafValues>
            -1.7162640392780304e-01 1.4818060398101807e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 74 4.3158490210771561e-02</internalNodes>
          <leafValues>
            8.3203509449958801e-02 -7.7821850776672363e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 75 7.6560080051422119e-03</internalNodes>
          <leafValues>
            8.4740802645683289e-02 -4.9738150835037231e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 76 -3.1110988929867744e-03</internalNodes>
          <leafValues>
            2.5827148556709290e-01 -2.5552031397819519e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 77 1.1870309710502625e-01</internalNodes>
          <leafValues>
            -9.0944238007068634e-02 7.2286212444305420e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 78 1.6875969246029854e-02</internalNodes>
          <leafValues>
            1.2629170715808868e-01 -5.5205297470092773e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 79 -1.0887029930017889e-04</internalNodes>
          <leafValues>
            8.1648796796798706e-02 -1.6937020421028137e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 80 2.8222990222275257e-03</internalNodes>
          <leafValues>
            1.6411300003528595e-01 -3.5218268632888794e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 81 -5.2425849437713623e-01</internalNodes>
          <leafValues>
            4.8906171321868896e-01 -1.2674759328365326e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 82 3.6927509307861328e-01</internalNodes>
          <leafValues>
            8.6115993559360504e-02 -6.7184638977050781e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 83 -1.6883780062198639e-01</internalNodes>
          <leafValues>
            -8.4915691614151001e-01 5.4833348840475082e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 84 -1.9279260188341141e-02</internalNodes>
          <leafValues>
            -7.8011512756347656e-01 6.2202680855989456e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>22</maxWeakCount>
      <stageThreshold>-1.2319500446319580e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 85 -2.0901350677013397e-01</internalNodes>
          <leafValues>
            6.9808167219161987e-01 -3.4573590755462646e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 86 -4.8061009147204459e-04</internalNodes>
          <leafValues>
            2.0923900604248047e-01 -2.4147640168666840e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 87 -2.4844119325280190e-03</internalNodes>
          <leafValues>
            2.7636009454727173e-01 -4.1990399360656738e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 88 -2.1536289714276791e-03</internalNodes>
          <leafValues>
            2.4710460007190704e-01 -3.0677899718284607e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 89 5.8911990374326706e-02</internalNodes>
          <leafValues>
            -7.0834763348102570e-02 7.1133142709732056e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 90 -2.3095219512470067e-04</internalNodes>
          <leafValues>
            1.7148600518703461e-01 -3.6168378591537476e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 91 -3.1396400183439255e-02</internalNodes>
          <leafValues>
            -8.0131882429122925e-01 1.0042560100555420e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 92 -3.5601970739662647e-03</internalNodes>
          <leafValues>
            9.9432766437530518e-02 -1.4848260581493378e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 93 -4.3389322236180305e-03</internalNodes>
          <leafValues>
            -5.6621241569519043e-01 1.4096799492835999e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 94 2.1326710283756256e-01</internalNodes>
          <leafValues>
            4.8158209770917892e-02 -7.4858909845352173e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 95 1.0042529553174973e-02</internalNodes>
          <leafValues>
            1.0428400337696075e-01 -5.5387377738952637e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 96 -2.6825280860066414e-02</internalNodes>
          <leafValues>
            5.7281607389450073e-01 -8.2537978887557983e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 97 8.3760882262140512e-04</internalNodes>
          <leafValues>
            -2.5626900792121887e-01 2.5898420810699463e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 98 -7.6051978394389153e-03</internalNodes>
          <leafValues>
            -5.8677357435226440e-01 5.1210779696702957e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 99 -1.1935640126466751e-01</internalNodes>
          <leafValues>
            -4.5530828833580017e-01 1.2570330500602722e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 100 6.6083478741347790e-03</internalNodes>
          <leafValues>
            -1.6316379606723785e-01 4.6659541130065918e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 101 1.7303509637713432e-02</internalNodes>
          <leafValues>
            -1.2391400337219238e-01 5.9755408763885498e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 102 5.4382272064685822e-03</internalNodes>
          <leafValues>
            1.3838729262351990e-01 -5.5069202184677124e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 103 2.4591449182480574e-03</internalNodes>
          <leafValues>
            -3.9927339553833008e-01 1.5387089550495148e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 104 3.5056238994002342e-03</internalNodes>
          <leafValues>
            -1.6146700084209442e-01 1.6086600720882416e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 105 -2.3172689543571323e-04</internalNodes>
          <leafValues>
            1.7059360444545746e-01 -3.5409420728683472e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 106 1.1914529837667942e-02</internalNodes>
          <leafValues>
            1.6265639662742615e-01 -4.1463181376457214e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>18</maxWeakCount>
      <stageThreshold>-1.1912549734115601e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 107 -4.5429700985550880e-03</internalNodes>
          <leafValues>
            4.2964971065521240e-01 -5.6915849447250366e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 108 4.6804840676486492e-03</internalNodes>
          <leafValues>
            -1.0380080342292786e-01 2.5453719496726990e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 109 3.5870380233973265e-03</internalNodes>
          <leafValues>
            -3.6577078700065613e-01 3.9343339204788208e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 110 -3.4428331255912781e-01</internalNodes>
          <leafValues>
            7.3125761747360229e-01 -1.5060240030288696e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 111 3.3054459840059280e-02</internalNodes>
          <leafValues>
            1.7657589912414551e-01 -5.1060509681701660e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 112 -2.1190310362726450e-03</internalNodes>
          <leafValues>
            8.6859323084354401e-02 -1.7733760178089142e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 113 1.3780740089714527e-02</internalNodes>
          <leafValues>
            -1.2247169762849808e-01 6.6472941637039185e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 114 2.4847950786352158e-02</internalNodes>
          <leafValues>
            2.3976799845695496e-01 -3.2456618547439575e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 115 -1.3126630336046219e-02</internalNodes>
          <leafValues>
            4.9461808800697327e-01 -2.0954379439353943e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 116 -1.6886189579963684e-02</internalNodes>
          <leafValues>
            -1.3973990082740784e-01 7.5013160705566406e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 117 -5.2776751108467579e-03</internalNodes>
          <leafValues>
            -3.8919359445571899e-01 1.8921519815921783e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 118 -2.0325549412518740e-03</internalNodes>
          <leafValues>
            2.4965450167655945e-01 -1.7960360646247864e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 119 -1.8056800588965416e-02</internalNodes>
          <leafValues>
            -5.3683072328567505e-01 1.0615479946136475e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 120 -2.8815109282732010e-02</internalNodes>
          <leafValues>
            5.3303200006484985e-01 -7.8712686896324158e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 121 -6.0971658676862717e-02</internalNodes>
          <leafValues>
            -8.5663092136383057e-01 8.1721447408199310e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 122 -6.2022160738706589e-02</internalNodes>
          <leafValues>
            -6.7228960990905762e-01 8.2316987216472626e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 123 -6.2961759977042675e-03</internalNodes>
          <leafValues>
            2.7192309498786926e-01 -2.3713490366935730e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 124 4.9608140252530575e-03</internalNodes>
          <leafValues>
            -1.4295519888401031e-01 2.9380369186401367e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>30</maxWeakCount>
      <stageThreshold>-1.1750839948654175e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 125 -8.7001353502273560e-02</internalNodes>
          <leafValues>
            6.3087427616119385e-01 -2.6264131069183350e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 126 -4.5627020299434662e-03</internalNodes>
          <leafValues>
            1.4641839265823364e-01 -5.2321881055831909e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 127 -4.1381991468369961e-03</internalNodes>
          <leafValues>
            2.1747599542140961e-01 -3.2107940316200256e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 128 -1.9443330529611558e-04</internalNodes>
          <leafValues>
            1.4305000007152557e-01 -4.4748461246490479e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 129 -2.6125069707632065e-03</internalNodes>
          <leafValues>
            -3.5936230421066284e-01 2.0934499800205231e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 130 -3.5238351672887802e-02</internalNodes>
          <leafValues>
            -5.5879557132720947e-01 1.1818339675664902e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 131 2.3880550637841225e-02</internalNodes>
          <leafValues>
            -1.2345419824123383e-01 6.4505738019943237e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 132 -3.5878319758921862e-03</internalNodes>
          <leafValues>
            2.3340910673141479e-01 -2.9905730485916138e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 133 -3.4388148784637451e-01</internalNodes>
          <leafValues>
            6.3334107398986816e-01 -8.6101479828357697e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 134 -2.5634190533310175e-03</internalNodes>
          <leafValues>
            -3.0992001295089722e-01 8.8213436305522919e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 135 4.7002349048852921e-02</internalNodes>
          <leafValues>
            7.3533393442630768e-02 -7.5965261459350586e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 136 7.1428148075938225e-03</internalNodes>
          <leafValues>
            -1.6981430351734161e-01 4.1982281208038330e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 137 -3.7736629601567984e-03</internalNodes>
          <leafValues>
            -5.5664837360382080e-01 1.0060050338506699e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 138 2.2179849445819855e-02</internalNodes>
          <leafValues>
            -7.6009899377822876e-02 6.3711041212081909e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 139 2.9807379178237170e-05</internalNodes>
          <leafValues>
            -2.7143061161041260e-01 2.1503789722919464e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 140 -1.4308329809864517e-05</internalNodes>
          <leafValues>
            1.3090610504150391e-01 -2.8089499473571777e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 141 -1.1500260233879089e-01</internalNodes>
          <leafValues>
            -7.1986222267150879e-01 7.6884172856807709e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 142 -2.5318590924143791e-02</internalNodes>
          <leafValues>
            4.5250499248504639e-01 -9.0481691062450409e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 143 -4.8698320984840393e-02</internalNodes>
          <leafValues>
            -7.4177128076553345e-01 6.7692406475543976e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 144 -5.0045289099216461e-03</internalNodes>
          <leafValues>
            1.3680170476436615e-01 -1.1860919743776321e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 145 7.5120502151548862e-03</internalNodes>
          <leafValues>
            9.1260991990566254e-02 -5.6960678100585938e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 146 -5.4631778039038181e-03</internalNodes>
          <leafValues>
            1.1702360212802887e-01 -1.4761230349540710e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 147 1.5256009995937347e-02</internalNodes>
          <leafValues>
            -1.0768359899520874e-01 6.4716261625289917e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 148 -2.1900620311498642e-02</internalNodes>
          <leafValues>
            -6.0776418447494507e-01 6.4449213445186615e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 149 2.1267218980938196e-03</internalNodes>
          <leafValues>
            -2.3115469515323639e-01 2.1813300251960754e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 150 -3.1501919031143188e-02</internalNodes>
          <leafValues>
            -1.3678109645843506e-01 6.6003270447254181e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 151 1.8107969313859940e-02</internalNodes>
          <leafValues>
            1.0865720361471176e-01 -4.4673460721969604e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 152 -1.1059570312500000e-01</internalNodes>
          <leafValues>
            4.6954178810119629e-01 -1.1268380284309387e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 153 2.2349569480866194e-03</internalNodes>
          <leafValues>
            -2.9884970188140869e-01 1.8147529661655426e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 154 4.6504188328981400e-02</internalNodes>
          <leafValues>
            1.2846769392490387e-01 -2.6609849929809570e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>27</maxWeakCount>
      <stageThreshold>-1.1861419677734375e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 155 -4.8820599913597107e-02</internalNodes>
          <leafValues>
            4.2807990312576294e-01 -5.5154949426651001e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 156 1.4779040357097983e-03</internalNodes>
          <leafValues>
            -1.8688060343265533e-01 1.9038289785385132e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 157 -1.0012290440499783e-02</internalNodes>
          <leafValues>
            3.8451421260833740e-01 -2.1723049879074097e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 158 -5.1000278443098068e-02</internalNodes>
          <leafValues>
            -7.6136952638626099e-01 1.3625900261104107e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 159 5.2959132008254528e-03</internalNodes>
          <leafValues>
            -2.3021429777145386e-01 2.8536239266395569e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 160 -4.8654139041900635e-02</internalNodes>
          <leafValues>
            7.0992070436477661e-01 -4.9203149974346161e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 161 8.8448636233806610e-03</internalNodes>
          <leafValues>
            -3.1505361199378967e-01 2.0899020135402679e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 162 1.0062800347805023e-01</internalNodes>
          <leafValues>
            6.6908989101648331e-03 6.7013871669769287e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 163 -7.0256260223686695e-03</internalNodes>
          <leafValues>
            -3.9408329129219055e-01 1.7433549463748932e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 164 -2.1224319934844971e-03</internalNodes>
          <leafValues>
            1.6996310651302338e-01 -3.0237409472465515e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 165 9.9532064050436020e-03</internalNodes>
          <leafValues>
            -1.4202840626239777e-01 4.5167461037635803e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 166 1.2565069831907749e-02</internalNodes>
          <leafValues>
            7.3175877332687378e-02 -6.1700421571731567e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 167 -1.7854310572147369e-03</internalNodes>
          <leafValues>
            1.4909860491752625e-01 -3.2865241169929504e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 168 -4.0306518785655499e-03</internalNodes>
          <leafValues>
            -4.5713710784912109e-01 1.0815720260143280e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 169 -7.3099560104310513e-03</internalNodes>
          <leafValues>
            -6.5592771768569946e-01 6.5615788102149963e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 170 -3.3843431621789932e-02</internalNodes>
          <leafValues>
            5.0412368774414062e-01 -6.1626069247722626e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 171 3.8319290615618229e-04</internalNodes>
          <leafValues>
            -2.5153478980064392e-01 2.0271340012550354e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 172 -2.6169361080974340e-03</internalNodes>
          <leafValues>
            2.2497959434986115e-01 -2.1958619356155396e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 173 -4.5606079511344433e-03</internalNodes>
          <leafValues>
            -4.6598041057586670e-01 1.2348009645938873e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 174 1.0822789743542671e-02</internalNodes>
          <leafValues>
            -9.6618972718715668e-02 4.6412429213523865e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 175 -5.3171347826719284e-03</internalNodes>
          <leafValues>
            -5.5634248256683350e-01 9.4623282551765442e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 176 -9.3140971148386598e-04</internalNodes>
          <leafValues>
            1.0143929719924927e-01 -1.0564240068197250e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 177 8.4296840941533446e-04</internalNodes>
          <leafValues>
            -1.3243100047111511e-01 3.5351079702377319e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 178 -2.7806960046291351e-02</internalNodes>
          <leafValues>
            -6.5050601959228516e-01 3.3153589814901352e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 179 6.9245469057932496e-04</internalNodes>
          <leafValues>
            -2.6702880859375000e-01 2.1129630506038666e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 180 -1.2787230312824249e-02</internalNodes>
          <leafValues>
            2.1593640744686127e-01 -8.6767077445983887e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 181 -6.1678601196035743e-04</internalNodes>
          <leafValues>
            1.6959980130195618e-01 -2.9248940944671631e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>21</maxWeakCount>
      <stageThreshold>-1.0550270080566406e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 182 -5.1706928759813309e-02</internalNodes>
          <leafValues>
            4.6942698955535889e-01 -5.1280671358108521e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 183 5.5232150480151176e-03</internalNodes>
          <leafValues>
            -2.4982389807701111e-01 6.3005810976028442e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 184 -9.2110745608806610e-03</internalNodes>
          <leafValues>
            3.7530669569969177e-01 -2.2910380363464355e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 185 4.1729960590600967e-02</internalNodes>
          <leafValues>
            -1.1262010037899017e-01 6.7508697509765625e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 186 4.5255841687321663e-03</internalNodes>
          <leafValues>
            -2.6939728856086731e-01 2.4889509379863739e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 187 -8.5208792006596923e-04</internalNodes>
          <leafValues>
            2.0098550617694855e-01 -2.3001730442047119e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 188 -3.4569639246910810e-03</internalNodes>
          <leafValues>
            -3.6372348666191101e-01 2.7142500877380371e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 189 -8.8200360536575317e-02</internalNodes>
          <leafValues>
            -7.5951957702636719e-01 -7.2166309691965580e-03</leafValues></_>
        <_>
          <internalNodes>
            0 -1 190 -2.3253160179592669e-04</internalNodes>
          <leafValues>
            1.4738219976425171e-01 -4.2548701167106628e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 191 1.9258400425314903e-02</internalNodes>
          <leafValues>
            -8.4830872714519501e-02 5.9487771987915039e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 192 -3.1915740109980106e-03</internalNodes>
          <leafValues>
            -4.2638280987739563e-01 1.3357159495353699e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 193 -2.2229040041565895e-02</internalNodes>
          <leafValues>
            -4.2298269271850586e-01 3.6127958446741104e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 194 -5.3123440593481064e-03</internalNodes>
          <leafValues>
            2.9349780082702637e-01 -2.2197869420051575e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 195 5.6796981953084469e-03</internalNodes>
          <leafValues>
            8.0412790179252625e-02 -1.9725289940834045e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 196 3.2511178869754076e-03</internalNodes>
          <leafValues>
            -1.6628390550613403e-01 3.3107280731201172e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 197 2.5559039786458015e-03</internalNodes>
          <leafValues>
            6.7350171506404877e-02 -2.4642370641231537e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 198 3.1239999458193779e-02</internalNodes>
          <leafValues>
            -6.7393511533737183e-02 8.2851767539978027e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 199 -4.4333371333777905e-03</internalNodes>
          <leafValues>
            -3.8048321008682251e-01 1.4248619973659515e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 200 -3.9497618563473225e-03</internalNodes>
          <leafValues>
            -3.5660448670387268e-01 1.8685440719127655e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 201 -1.4043290168046951e-02</internalNodes>
          <leafValues>
            5.3222888708114624e-01 -7.8980803489685059e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 202 4.2212791740894318e-03</internalNodes>
          <leafValues>
            -1.9841830432415009e-01 3.1367298960685730e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>43</maxWeakCount>
      <stageThreshold>-1.1214250326156616e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 203 -1.5278789401054382e-01</internalNodes>
          <leafValues>
            5.4140037298202515e-01 -1.8756979703903198e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 204 -7.0655636489391327e-02</internalNodes>
          <leafValues>
            3.4003350138664246e-01 -1.4459669589996338e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 205 -2.1033229306340218e-02</internalNodes>
          <leafValues>
            -5.5878472328186035e-01 1.1598149687051773e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 206 -9.5666358247399330e-03</internalNodes>
          <leafValues>
            1.0890080034732819e-01 -2.0365689694881439e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 207 -4.2720541357994080e-02</internalNodes>
          <leafValues>
            -9.4030022621154785e-01 6.3606321811676025e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 208 -4.5477859675884247e-03</internalNodes>
          <leafValues>
            3.4227019548416138e-01 -1.7053720355033875e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 209 3.7029080558568239e-03</internalNodes>
          <leafValues>
            8.3720892667770386e-02 -4.6139541268348694e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 210 -1.1458870023488998e-01</internalNodes>
          <leafValues>
            6.0027849674224854e-01 -1.7764480784535408e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 211 5.7319342158734798e-03</internalNodes>
          <leafValues>
            -2.5590109825134277e-01 2.0062319934368134e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 212 -7.0237793028354645e-02</internalNodes>
          <leafValues>
            2.53
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

