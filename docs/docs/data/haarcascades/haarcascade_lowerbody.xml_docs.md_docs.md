# Documentation for `docs/data/haarcascades/haarcascade_lowerbody.xml_docs.md`

## File Metadata

- **Full Path**: `docs/data/haarcascades/haarcascade_lowerbody.xml_docs.md`
- **File Name**: `haarcascade_lowerbody.xml_docs.md`
- **File Size**: 51,038 bytes
- **File Type**: .md
- **Link to Source**: [docs/data/haarcascades/haarcascade_lowerbody.xml_docs.md](../../../docs/data/haarcascades/haarcascade_lowerbody.xml_docs.md)

## Purpose and Role

This file is located in the `docs/data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `data/haarcascades/haarcascade_lowerbody.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_lowerbody.xml`
- **File Name**: `haarcascade_lowerbody.xml`
- **File Size**: 395,322 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_lowerbody.xml](../../data/haarcascades/haarcascade_lowerbody.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
   19x23 lowerbody detector (see the detailed description below). 

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
  <height>23</height>
  <width>19</width>
  <stageParams>
    <maxWeakCount>89</maxWeakCount></stageParams>
  <featureParams>
    <maxCatCount>0</maxCatCount></featureParams>
  <stageNum>27</stageNum>
  <stages>
    <_>
      <maxWeakCount>17</maxWeakCount>
      <stageThreshold>-1.4308550357818604e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 0 -1.6869869083166122e-02</internalNodes>
          <leafValues>
            5.4657417535781860e-01 -6.3678038120269775e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1 2.5349899660795927e-03</internalNodes>
          <leafValues>
            -3.7605491280555725e-01 3.2378101348876953e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 2 -2.4709459394216537e-02</internalNodes>
          <leafValues>
            -6.7979127168655396e-01 2.0501059293746948e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 3 8.2436859607696533e-02</internalNodes>
          <leafValues>
            2.0588639378547668e-01 -8.4938430786132812e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 4 -8.2128931535407901e-04</internalNodes>
          <leafValues>
            3.1891921162605286e-01 -4.6469458937644958e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 5 2.3016959428787231e-02</internalNodes>
          <leafValues>
            1.8670299649238586e-01 -7.0330899953842163e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 6 6.6386149264872074e-03</internalNodes>
          <leafValues>
            1.6370490193367004e-01 -8.4604722261428833e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 7 7.6682120561599731e-04</internalNodes>
          <leafValues>
            -3.9852690696716309e-01 2.3113329708576202e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 8 1.1731679737567902e-01</internalNodes>
          <leafValues>
            1.0445039719343185e-01 -8.8510942459106445e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 9 1.5421230345964432e-02</internalNodes>
          <leafValues>
            -2.7859508991241455e-01 2.8921920061111450e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 10 3.4018948674201965e-02</internalNodes>
          <leafValues>
            -1.4287669956684113e-01 7.7801531553268433e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 11 3.4638870507478714e-02</internalNodes>
          <leafValues>
            1.8644079566001892e-01 -6.0324841737747192e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 12 -3.7503659725189209e-01</internalNodes>
          <leafValues>
            9.2781841754913330e-01 -1.5421600639820099e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 13 -5.6011971086263657e-02</internalNodes>
          <leafValues>
            -5.8591067790985107e-01 1.9547510147094727e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 14 -1.4878909569233656e-03</internalNodes>
          <leafValues>
            2.8139349818229675e-01 -4.1853010654449463e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 15 -1.4495699666440487e-02</internalNodes>
          <leafValues>
            -7.2273969650268555e-01 9.4288460910320282e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 16 -5.6178281083703041e-03</internalNodes>
          <leafValues>
            -5.9551960229873657e-01 1.5202650427818298e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>13</maxWeakCount>
      <stageThreshold>-1.1907930374145508e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 17 -3.1839120201766491e-03</internalNodes>
          <leafValues>
            4.0025138854980469e-01 -6.8473160266876221e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 18 3.5989920143038034e-03</internalNodes>
          <leafValues>
            -5.1895952224731445e-01 3.0101141333580017e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 19 1.8804630264639854e-02</internalNodes>
          <leafValues>
            1.5554919838905334e-01 -8.0477172136306763e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 20 5.2497140131890774e-03</internalNodes>
          <leafValues>
            1.3780809938907623e-01 -6.0767507553100586e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 21 -1.4204799663275480e-03</internalNodes>
          <leafValues>
            3.2319429516792297e-01 -4.3407461047172546e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 22 -2.5174349546432495e-02</internalNodes>
          <leafValues>
            -7.0780879259109497e-01 9.3106329441070557e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 23 3.2285219058394432e-03</internalNodes>
          <leafValues>
            -3.2510471343994141e-01 3.3571699261665344e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 24 9.4993412494659424e-02</internalNodes>
          <leafValues>
            8.2439087331295013e-02 -8.7549537420272827e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 25 -6.5919090993702412e-03</internalNodes>
          <leafValues>
            -7.3804199695587158e-01 1.3853749632835388e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 26 -1.1146620381623507e-03</internalNodes>
          <leafValues>
            1.7917269468307495e-01 -2.7955859899520874e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 27 1.3349019922316074e-02</internalNodes>
          <leafValues>
            1.3057829439640045e-01 -6.9802671670913696e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 28 -3.5181451588869095e-02</internalNodes>
          <leafValues>
            4.6535360813140869e-01 -1.0698779672384262e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 29 3.1874589622020721e-02</internalNodes>
          <leafValues>
            -1.3565389811992645e-01 7.9047888517379761e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>19</maxWeakCount>
      <stageThreshold>-1.3129220008850098e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 30 -1.0647430084645748e-02</internalNodes>
          <leafValues>
            3.8079029321670532e-01 -5.8672338724136353e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 31 -7.3214493691921234e-02</internalNodes>
          <leafValues>
            -7.9550951719284058e-01 1.7223259806632996e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 32 6.0464427806437016e-03</internalNodes>
          <leafValues>
            1.6532160341739655e-01 -6.9376647472381592e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 33 7.3225022060796618e-04</internalNodes>
          <leafValues>
            -3.3247160911560059e-01 2.3669970035552979e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 34 -1.0990080423653126e-02</internalNodes>
          <leafValues>
            -6.9136887788772583e-01 2.1058270335197449e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 35 -1.5282750246115029e-04</internalNodes>
          <leafValues>
            2.0305849611759186e-01 -4.6551659703254700e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 36 2.4822261184453964e-04</internalNodes>
          <leafValues>
            -4.2122921347618103e-01 2.7335309982299805e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 37 -8.4205856546759605e-03</internalNodes>
          <leafValues>
            -4.3744468688964844e-01 5.8831848204135895e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 38 -3.6992791295051575e-01</internalNodes>
          <leafValues>
            9.1070818901062012e-01 -8.7207540869712830e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 39 6.1259930953383446e-03</internalNodes>
          <leafValues>
            1.1886730045080185e-01 -1.8520170450210571e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 40 -6.0144090093672276e-03</internalNodes>
          <leafValues>
            -6.3057059049606323e-01 1.4577180147171021e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 41 8.5623031482100487e-03</internalNodes>
          <leafValues>
            -2.9369381070137024e-01 3.2411348819732666e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 42 -1.3966850005090237e-02</internalNodes>
          <leafValues>
            -8.0650371313095093e-01 1.1267790198326111e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 43 -4.1734468191862106e-02</internalNodes>
          <leafValues>
            7.7495330572128296e-01 -7.8866302967071533e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 44 -2.7996799326501787e-04</internalNodes>
          <leafValues>
            2.7783310413360596e-01 -3.5196089744567871e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 45 1.9588569179177284e-02</internalNodes>
          <leafValues>
            -6.5759636461734772e-02 5.2414137125015259e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 46 9.2163113877177238e-03</internalNodes>
          <leafValues>
            -1.5525479614734650e-01 5.4835391044616699e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 47 -2.1458569914102554e-02</internalNodes>
          <leafValues>
            -5.2255308628082275e-01 8.2208268344402313e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 48 3.6805770359933376e-03</internalNodes>
          <leafValues>
            -2.4434129893779755e-01 3.6122488975524902e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>23</maxWeakCount>
      <stageThreshold>-1.3777279853820801e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 49 -8.3544738590717316e-03</internalNodes>
          <leafValues>
            2.8173181414604187e-01 -4.9728131294250488e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 50 -5.5724289268255234e-03</internalNodes>
          <leafValues>
            -6.5505301952362061e-01 1.9406059384346008e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 51 -5.7714767754077911e-03</internalNodes>
          <leafValues>
            -6.2230938673019409e-01 2.7622398734092712e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 52 2.2995889186859131e-02</internalNodes>
          <leafValues>
            1.9798569381237030e-02 -7.8324538469314575e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 53 -1.1443760013207793e-03</internalNodes>
          <leafValues>
            2.8108718991279602e-01 -4.8214849829673767e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 54 -2.5917509198188782e-01</internalNodes>
          <leafValues>
            -6.8214958906173706e-01 -3.3729869755916297e-04</leafValues></_>
        <_>
          <internalNodes>
            0 -1 55 -3.0133039690554142e-03</internalNodes>
          <leafValues>
            -6.5704411268234253e-01 1.3693599402904510e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 56 5.4540671408176422e-03</internalNodes>
          <leafValues>
            8.6931817233562469e-02 -7.0567971467971802e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 57 6.6230311058461666e-03</internalNodes>
          <leafValues>
            1.6634289920330048e-01 -5.1772958040237427e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 58 -1.2561669573187828e-02</internalNodes>
          <leafValues>
            9.0290471911430359e-02 -1.6850970685482025e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 59 4.2890738695859909e-02</internalNodes>
          <leafValues>
            1.2977810204029083e-01 -5.8218061923980713e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 60 -1.3341030571609735e-03</internalNodes>
          <leafValues>
            1.3694329559803009e-01 -1.9437809288501740e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 61 -4.1247460991144180e-02</internalNodes>
          <leafValues>
            6.8543851375579834e-01 -1.3039450347423553e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 62 -9.1503392904996872e-03</internalNodes>
          <leafValues>
            -1.1895430088043213e-01 6.7576698958873749e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 63 -1.7151240026578307e-03</internalNodes>
          <leafValues>
            2.6475539803504944e-01 -3.0487450957298279e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 64 2.0843200385570526e-01</internalNodes>
          <leafValues>
            1.2401489913463593e-01 -4.7014111280441284e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 65 7.2393968701362610e-02</internalNodes>
          <leafValues>
            9.6924379467964172e-02 -7.7347749471664429e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 66 -1.5335980569943786e-03</internalNodes>
          <leafValues>
            1.7991219460964203e-01 -2.5788331031799316e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 67 4.8640500754117966e-03</internalNodes>
          <leafValues>
            1.1392980068922043e-01 -5.5173867940902710e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 68 -1.6523050144314766e-03</internalNodes>
          <leafValues>
            1.5154689550399780e-01 -2.2901679575443268e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 69 7.5348757207393646e-02</internalNodes>
          <leafValues>
            -1.4630889892578125e-01 6.8105882406234741e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 70 -8.2630068063735962e-03</internalNodes>
          <leafValues>
            -7.2783601284027100e-01 1.0281019657850266e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 71 -5.5124741047620773e-03</internalNodes>
          <leafValues>
            -6.3059347867965698e-01 9.3257799744606018e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>15</maxWeakCount>
      <stageThreshold>-1.0618749856948853e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 72 -9.3849105760455132e-03</internalNodes>
          <leafValues>
            5.2500581741333008e-01 -4.3231061100959778e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 73 -1.3772470410913229e-03</internalNodes>
          <leafValues>
            2.0698480308055878e-01 -4.2718759179115295e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 74 2.6320109143853188e-02</internalNodes>
          <leafValues>
            1.5825170278549194e-01 -6.5509521961212158e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 75 -4.5488759875297546e-02</internalNodes>
          <leafValues>
            -4.9510109424591064e-01 1.7998820543289185e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 76 -4.7006201930344105e-03</internalNodes>
          <leafValues>
            3.3971160650253296e-01 -3.6917701363563538e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 77 -1.3270860072225332e-03</internalNodes>
          <leafValues>
            3.0907860398292542e-01 -1.9771750271320343e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 78 9.3802614137530327e-03</internalNodes>
          <leafValues>
            9.4488449394702911e-02 -7.3198097944259644e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 79 4.3565612286329269e-03</internalNodes>
          <leafValues>
            1.1520200222730637e-01 -5.4008102416992188e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 80 8.1178937107324600e-03</internalNodes>
          <leafValues>
            -1.5956309437751770e-01 5.3777867555618286e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 81 -8.7829083204269409e-03</internalNodes>
          <leafValues>
            5.6634718179702759e-01 -1.3279379904270172e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 82 2.1944850683212280e-02</internalNodes>
          <leafValues>
            1.5901289880275726e-01 -5.1751822233200073e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 83 4.9510098993778229e-02</internalNodes>
          <leafValues>
            1.1067640036344528e-02 -4.9972468614578247e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 84 -2.1175360307097435e-03</internalNodes>
          <leafValues>
            2.6490759849548340e-01 -2.4565629661083221e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 85 1.0379469953477383e-02</internalNodes>
          <leafValues>
            1.2624099850654602e-01 -4.0877240896224976e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 86 2.4977258872240782e-03</internalNodes>
          <leafValues>
            -1.9723020493984222e-01 3.8866749405860901e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>18</maxWeakCount>
      <stageThreshold>-9.5461457967758179e-01</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 87 -6.1489548534154892e-03</internalNodes>
          <leafValues>
            4.0187481045722961e-01 -5.2397370338439941e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 88 5.0464540719985962e-02</internalNodes>
          <leafValues>
            1.3049679994583130e-01 -5.8651441335678101e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 89 -5.5906269699335098e-02</internalNodes>
          <leafValues>
            -5.1229542493820190e-01 2.4392889440059662e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 90 1.4281509816646576e-01</internalNodes>
          <leafValues>
            -1.5180160291492939e-02 -6.9593918323516846e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 91 4.1162770241498947e-02</internalNodes>
          <leafValues>
            1.3673730194568634e-01 -6.4158838987350464e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 92 -1.6468750312924385e-02</internalNodes>
          <leafValues>
            2.6339039206504822e-01 -2.2083680331707001e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 93 2.4763140827417374e-02</internalNodes>
          <leafValues>
            1.0897739976644516e-01 -6.5213900804519653e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 94 4.3008858337998390e-03</internalNodes>
          <leafValues>
            -1.8299630284309387e-01 4.3614229559898376e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 95 3.4035290591418743e-03</internalNodes>
          <leafValues>
            -2.4363580346107483e-01 2.8224369883537292e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 96 -2.2210620343685150e-02</internalNodes>
          <leafValues>
            -5.4645758867263794e-01 1.3542969524860382e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 97 -2.6968019083142281e-02</internalNodes>
          <leafValues>
            6.5300947427749634e-01 -1.4297309517860413e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 98 -3.4927908331155777e-02</internalNodes>
          <leafValues>
            -5.2346628904342651e-01 1.0084570199251175e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 99 3.6263581365346909e-02</internalNodes>
          <leafValues>
            1.5110149979591370e-01 -5.4185849428176880e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 100 -3.8526788353919983e-02</internalNodes>
          <leafValues>
            -8.6942279338836670e-01 3.7176769226789474e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 101 2.5399168953299522e-03</internalNodes>
          <leafValues>
            -2.6125881075859070e-01 2.7278441190719604e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 102 -1.2931150384247303e-02</internalNodes>
          <leafValues>
            -4.9501579999923706e-01 9.1383516788482666e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 103 1.1981350369751453e-02</internalNodes>
          <leafValues>
            -1.2059610337018967e-01 6.3848638534545898e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 104 -7.4320413172245026e-02</internalNodes>
          <leafValues>
            4.6591779589653015e-01 -4.0265668183565140e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>14</maxWeakCount>
      <stageThreshold>-1.1777880191802979e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 105 -6.9070039317011833e-03</internalNodes>
          <leafValues>
            4.3197679519653320e-01 -5.1717847585678101e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 106 -8.1628039479255676e-03</internalNodes>
          <leafValues>
            2.7116540074348450e-01 -3.2803410291671753e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 107 1.8852509558200836e-02</internalNodes>
          <leafValues>
            1.5548799932003021e-01 -5.5243927240371704e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 108 3.4079391509294510e-02</internalNodes>
          <leafValues>
            1.5272259712219238e-01 -6.5318012237548828e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 109 -3.2038250938057899e-03</internalNodes>
          <leafValues>
            3.4725460410118103e-01 -2.7734228968620300e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 110 2.1410689223557711e-03</internalNodes>
          <leafValues>
            -6.8888276815414429e-02 2.4079489707946777e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 111 1.4620450139045715e-01</internalNodes>
          <leafValues>
            1.5766879916191101e-01 -5.4515862464904785e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 112 -6.2386798672378063e-03</internalNodes>
          <leafValues>
            3.2899579405784607e-01 -1.6970640420913696e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 113 7.7623138204216957e-03</internalNodes>
          <leafValues>
            1.6352510452270508e-01 -5.1879328489303589e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 114 3.7800080608576536e-03</internalNodes>
          <leafValues>
            -1.8464370071887970e-01 4.8660078644752502e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 115 2.2303969599306583e-03</internalNodes>
          <leafValues>
            -1.7057199776172638e-01 4.7744798660278320e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 116 2.4544890038669109e-03</internalNodes>
          <leafValues>
            -3.3550649881362915e-01 2.5369268655776978e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 117 -2.1707419306039810e-02</internalNodes>
          <leafValues>
            -4.8321890830993652e-01 1.6075029969215393e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 118 1.7421970143914223e-02</internalNodes>
          <leafValues>
            7.9877912998199463e-02 -7.5137257575988770e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>34</maxWeakCount>
      <stageThreshold>-1.2834340333938599e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 119 8.8802073150873184e-03</internalNodes>
          <leafValues>
            -4.4682410359382629e-01 2.6062530279159546e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 120 -3.0198058811947703e-04</internalNodes>
          <leafValues>
            1.5258400142192841e-01 -3.5206508636474609e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 121 6.7998501472175121e-03</internalNodes>
          <leafValues>
            1.2259320169687271e-01 -6.8427437543869019e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 122 2.7802670374512672e-03</internalNodes>
          <leafValues>
            -3.3681631088256836e-01 1.8518559634685516e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 123 -1.1553820222616196e-02</internalNodes>
          <leafValues>
            -6.9871348142623901e-01 1.3079600036144257e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 124 -2.6563290506601334e-02</internalNodes>
          <leafValues>
            -7.0277881622314453e-01 1.7791330814361572e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 125 -2.5158381322398782e-04</internalNodes>
          <leafValues>
            2.4779480695724487e-01 -3.9787930250167847e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 126 3.5748310387134552e-02</internalNodes>
          <leafValues>
            -3.8043439388275146e-02 4.7976261377334595e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 127 -1.9973930902779102e-03</internalNodes>
          <leafValues>
            2.5774869322776794e-01 -3.1990098953247070e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 128 -1.1007110029459000e-01</internalNodes>
          <leafValues>
            -4.9102869629859924e-01 2.3104630410671234e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 129 -2.2225650027394295e-03</internalNodes>
          <leafValues>
            2.3825299739837646e-01 -2.8415530920028687e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 130 -7.7874241396784782e-03</internalNodes>
          <leafValues>
            -3.8951370120048523e-01 5.5762890726327896e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 131 5.6415859609842300e-02</internalNodes>
          <leafValues>
            -9.3521721661090851e-02 7.2561162710189819e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 132 -3.5978010855615139e-03</internalNodes>
          <leafValues>
            1.9452190399169922e-01 -1.9651280343532562e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 133 -7.2716898284852505e-03</internalNodes>
          <leafValues>
            3.4169870615005493e-01 -2.2851559519767761e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 134 7.1941758506000042e-03</internalNodes>
          <leafValues>
            7.2148866951465607e-02 -4.5313501358032227e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 135 -4.1034761816263199e-03</internalNodes>
          <leafValues>
            -5.1336747407913208e-01 1.3323569297790527e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 136 -3.4210970625281334e-03</internalNodes>
          <leafValues>
            -4.2383781075477600e-01 8.4852807223796844e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 137 4.1890922002494335e-03</internalNodes>
          <leafValues>
            -1.3398550450801849e-01 4.3749558925628662e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 138 1.1827970156446099e-03</internalNodes>
          <leafValues>
            -2.9739010334014893e-01 2.2126840054988861e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 139 -4.1196551173925400e-02</internalNodes>
          <leafValues>
            -5.0735759735107422e-01 1.3243959844112396e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 140 2.9593890067189932e-03</internalNodes>
          <leafValues>
            -1.4052620530128479e-01 6.1360880732536316e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 141 -5.0226859748363495e-03</internalNodes>
          <leafValues>
            -4.7495970129966736e-01 1.2069150060415268e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 142 -1.5097860246896744e-02</internalNodes>
          <leafValues>
            2.7555391192436218e-01 -5.3780451416969299e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 143 -2.7190970256924629e-02</internalNodes>
          <leafValues>
            7.5995457172393799e-01 -7.4793189764022827e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 144 1.9893879070878029e-02</internalNodes>
          <leafValues>
            -6.7238640040159225e-03 7.3972767591476440e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 145 7.7208830043673515e-03</internalNodes>
          <leafValues>
            9.3071162700653076e-02 -6.5780252218246460e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 146 -1.1565990280359983e-03</internalNodes>
          <leafValues>
            9.4645917415618896e-02 -1.6407909989356995e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 147 2.6069190353155136e-03</internalNodes>
          <leafValues>
            -1.3877980411052704e-01 4.7349870204925537e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 148 -5.3586110472679138e-02</internalNodes>
          <leafValues>
            -3.7349641323089600e-01 2.5728559121489525e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 149 1.5184599906206131e-03</internalNodes>
          <leafValues>
            -2.2478710114955902e-01 2.3574599623680115e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 150 -3.7061560899019241e-02</internalNodes>
          <leafValues>
            -6.1827117204666138e-01 8.2348063588142395e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 151 -2.6311799883842468e-02</internalNodes>
          <leafValues>
            -6.0057657957077026e-01 7.7768869698047638e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 152 -8.7947428226470947e-02</internalNodes>
          <leafValues>
            3.8841038942337036e-01 -8.1545598804950714e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>20</maxWeakCount>
      <stageThreshold>-1.2891789674758911e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 153 -2.9038030654191971e-02</internalNodes>
          <leafValues>
            5.0635957717895508e-01 -4.3462699651718140e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 154 3.9044669829308987e-03</internalNodes>
          <leafValues>
            -1.9009789824485779e-01 5.1840317249298096e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 155 2.9162769205868244e-03</internalNodes>
          <leafValues>
            -3.4351310133934021e-01 2.4016310274600983e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 156 -8.9670084416866302e-03</internalNodes>
          <leafValues>
            -4.2667150497436523e-01 1.2316550314426422e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 157 -2.4935540277510881e-03</internalNodes>
          <leafValues>
            3.6086550354957581e-01 -1.8381460011005402e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 158 -4.8912568017840385e-03</internalNodes>
          <leafValues>
            -6.4749848842620850e-01 1.0856709629297256e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 159 -4.0970719419419765e-03</internalNodes>
          <leafValues>
            2.2143830358982086e-01 -3.1505578756332397e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 160 4.3956499546766281e-02</internalNodes>
          <leafValues>
            -1.0780169814825058e-01 7.1893501281738281e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 161 1.9277370302006602e-03</internalNodes>
          <leafValues>
            2.0247739553451538e-01 -4.0381088852882385e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 162 9.4976946711540222e-03</internalNodes>
          <leafValues>
            4.3494019657373428e-02 -2.9908061027526855e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 163 3.5389279946684837e-03</internalNodes>
          <leafValues>
            -1.5109489858150482e-01 5.1864242553710938e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 164 -2.2064079530537128e-03</internalNodes>
          <leafValues>
            2.3006440699100494e-01 -3.3191001415252686e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 165 3.9085410535335541e-03</internalNodes>
          <leafValues>
            -3.4253311157226562e-01 2.2951880097389221e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 166 2.6973709464073181e-03</internalNodes>
          <leafValues>
            1.1976680159568787e-01 -3.5321989655494690e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 167 -2.1321459207683802e-03</internalNodes>
          <leafValues>
            1.8206289410591125e-01 -2.8434100747108459e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 168 2.6955150533467531e-03</internalNodes>
          <leafValues>
            7.4593842029571533e-02 -3.0896648764610291e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 169 -6.0222679749131203e-03</internalNodes>
          <leafValues>
            1.8041500449180603e-01 -2.7531668543815613e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 170 -8.9143458753824234e-03</internalNodes>
          <leafValues>
            2.4166099727153778e-01 -1.4506129920482635e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 171 2.3474939167499542e-02</internalNodes>
          <leafValues>
            -1.2354619801044464e-01 6.5625041723251343e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 172 -5.6602950207889080e-03</internalNodes>
          <leafValues>
            -3.3785250782966614e-01 1.1194559931755066e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>20</maxWeakCount>
      <stageThreshold>-1.0202569961547852e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 173 -6.9699093699455261e-02</internalNodes>
          <leafValues>
            5.0786459445953369e-01 -4.7562688589096069e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 174 2.1672779694199562e-02</internalNodes>
          <leafValues>
            -2.9134199023246765e-01 3.4561529755592346e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 175 -4.7600260004401207e-03</internalNodes>
          <leafValues>
            3.6477440595626831e-01 -1.9551509618759155e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 176 -4.6418169513344765e-03</internalNodes>
          <leafValues>
            -5.6445592641830444e-01 9.8486669361591339e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 177 -6.0006938874721527e-03</internalNodes>
          <leafValues>
            -6.3645982742309570e-01 1.4379170536994934e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 178 1.9073469564318657e-02</internalNodes>
          <leafValues>
            -3.4218288958072662e-02 5.5043292045593262e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 179 4.7993380576372147e-02</internalNodes>
          <leafValues>
            -8.5889510810375214e-02 7.6790231466293335e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 180 -3.6511209327727556e-03</internalNodes>
          <leafValues>
            2.0186069607734680e-01 -2.9832679033279419e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 181 -1.4485770370811224e-03</internalNodes>
          <leafValues>
            -5.1293247938156128e-01 1.3695690035820007e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 182 -3.3748829737305641e-03</internalNodes>
          <leafValues>
            -4.0975129604339600e-01 1.1581440269947052e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 183 2.3586750030517578e-03</internalNodes>
          <leafValues>
            1.7582429945468903e-01 -4.5439630746841431e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 184 -2.2074829787015915e-02</internalNodes>
          <leafValues>
            4.6775639057159424e-01 -4.6358831226825714e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 185 7.0953248068690300e-03</internalNodes>
          <leafValues>
            -3.2100531458854675e-01 2.2119350731372833e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 186 -2.0119780674576759e-03</internalNodes>
          <leafValues>
            5.4601740092039108e-02 -9.7853101789951324e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 187 4.9847508780658245e-03</internalNodes>
          <leafValues>
            -1.3063269853591919e-01 5.2815079689025879e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 188 -5.3485459648072720e-03</internalNodes>
          <leafValues>
            -4.2115539312362671e-01 1.1927159875631332e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 189 2.5243330746889114e-03</internalNodes>
          <leafValues>
            1.2105660140514374e-01 -4.5177119970321655e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 190 -2.4893151130527258e-03</internalNodes>
          <leafValues>
            1.2249600142240524e-01 -1.1200980097055435e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 191 4.3740491382777691e-03</internalNodes>
          <leafValues>
            -1.0549320280551910e-01 6.0806149244308472e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 192 -7.3214988224208355e-03</internalNodes>
          <leafValues>
            4.7615110874176025e-01 -6.8390920758247375e-02</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>24</maxWeakCount>
      <stageThreshold>-1.0336159467697144e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 193 -4.2286239564418793e-02</internalNodes>
          <leafValues>
            3.6749860644340515e-01 -4.3680980801582336e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 194 3.8884699344635010e-02</internalNodes>
          <leafValues>
            -3.5438889265060425e-01 2.7009218931198120e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 195 1.5983959892764688e-03</internalNodes>
          <leafValues>
            -3.2200628519058228e-01 2.5404900312423706e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 196 3.9249849505722523e-03</internalNodes>
          <leafValues>
            1.6477300226688385e-01 -4.2043879628181458e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 197 1.5850430354475975e-03</internalNodes>
          <leafValues>
            -2.5503370165824890e-01 3.1559389829635620e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 198 -3.4282119013369083e-03</internalNodes>
          <leafValues>
            -4.0074288845062256e-01 1.1993350088596344e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 199 -3.3538821153342724e-03</internalNodes>
          <leafValues>
            3.0459630489349365e-01 -2.2311030328273773e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 200 -6.7664748057723045e-03</internalNodes>
          <leafValues>
            3.2396519184112549e-01 -9.2932380735874176e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 201 -6.7180307814851403e-04</internalNodes>
          <leafValues>
            -3.2457518577575684e-01 2.1808999776840210e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 202 2.8931829147040844e-03</internalNodes>
          <leafValues>
            1.2530609965324402e-01 -4.8582470417022705e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 203 -3.3115309197455645e-03</internalNodes>
          <leafValues>
            4.0534108877182007e-01 -2.2432869672775269e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 204 8.8509041815996170e-03</internalNodes>
          <leafValues>
            1.2155570089817047e-01 -6.0243481397628784e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 205 5.4662628099322319e-03</internalNodes>
          <leafValues>
            -1.6978119313716888e-01 4.0752619504928589e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 206 4.7559391707181931e-02</internalNodes>
          <leafValues>
            -8.1737041473388672e-02 6.9865119457244873e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 207 3.1745019368827343e-03</internalNodes>
          <leafValues>
            1.7419810593128204e-01 -3.7237030267715454e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 208 -5.1520839333534241e-03</internalNodes>
          <leafValues>
            2.7799358963966370e-01 -2.5311779975891113e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 209 -4.8141111619770527e-03</internalNodes>
          <leafValues>
            -5.8466029167175293e-01 1.5894299745559692e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 210 2.1967150270938873e-02</internalNodes>
          <leafValues>
            -1.0052759945392609e-01 4.7374871373176575e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 211 -6.0128211043775082e-03</internalNodes>
          <leafValues>
            1.9820199906826019e-01 -4.2172819375991821e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 212 4.5052049681544304e-03</internalNodes>
          <leafValues>
            1.7064809799194336e
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

