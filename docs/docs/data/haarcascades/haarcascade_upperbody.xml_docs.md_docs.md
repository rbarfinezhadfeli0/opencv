# Documentation for `docs/data/haarcascades/haarcascade_upperbody.xml_docs.md`

## File Metadata

- **Full Path**: `docs/data/haarcascades/haarcascade_upperbody.xml_docs.md`
- **File Name**: `haarcascade_upperbody.xml_docs.md`
- **File Size**: 51,038 bytes
- **File Type**: .md
- **Link to Source**: [docs/data/haarcascades/haarcascade_upperbody.xml_docs.md](../../../docs/data/haarcascades/haarcascade_upperbody.xml_docs.md)

## Purpose and Role

This file is located in the `docs/data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `data/haarcascades/haarcascade_upperbody.xml`

## File Metadata

- **Full Path**: `data/haarcascades/haarcascade_upperbody.xml`
- **File Name**: `haarcascade_upperbody.xml`
- **File Size**: 785,819 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades/haarcascade_upperbody.xml](../../data/haarcascades/haarcascade_upperbody.xml)

## Purpose and Role

This file is located in the `data/haarcascades` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<!--
   22x18 upperbody detector (see the detailed description below). 

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
  <height>18</height>
  <width>22</width>
  <stageParams>
    <maxWeakCount>152</maxWeakCount></stageParams>
  <featureParams>
    <maxCatCount>0</maxCatCount></featureParams>
  <stageNum>30</stageNum>
  <stages>
    <_>
      <maxWeakCount>20</maxWeakCount>
      <stageThreshold>-1.1264339685440063e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 0 -1.3696029782295227e-02</internalNodes>
          <leafValues>
            4.5076468586921692e-01 -4.2179030179977417e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 1 1.2441449798643589e-02</internalNodes>
          <leafValues>
            1.6493250429630280e-01 -7.4793487787246704e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 2 -2.7094660326838493e-03</internalNodes>
          <leafValues>
            3.1004700064659119e-01 -3.7617141008377075e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 3 -1.0008010268211365e-01</internalNodes>
          <leafValues>
            7.6182198524475098e-01 -7.4556976556777954e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 4 -2.5114119052886963e-01</internalNodes>
          <leafValues>
            -6.4154028892517090e-01 1.5139220654964447e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 5 -1.0510650277137756e-01</internalNodes>
          <leafValues>
            7.1459370851516724e-01 -1.4498579502105713e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 6 -8.8448017835617065e-02</internalNodes>
          <leafValues>
            7.5773179531097412e-01 -6.8586893379688263e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 7 1.0874910280108452e-02</internalNodes>
          <leafValues>
            1.4610609412193298e-01 -5.4263710975646973e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 8 1.2690570205450058e-02</internalNodes>
          <leafValues>
            1.1674589663743973e-01 -4.9649459123611450e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 9 -3.2198399305343628e-02</internalNodes>
          <leafValues>
            -3.8529390096664429e-01 9.8437972366809845e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 10 -3.4077179152518511e-03</internalNodes>
          <leafValues>
            2.5200870633125305e-01 -2.2382549941539764e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 11 3.0324390158057213e-02</internalNodes>
          <leafValues>
            -1.0534449666738510e-01 6.5735417604446411e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 12 4.1930507868528366e-03</internalNodes>
          <leafValues>
            1.2872399389743805e-01 -5.3160661458969116e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 13 8.0501407384872437e-02</internalNodes>
          <leafValues>
            4.1696660220623016e-02 -7.2123032808303833e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 14 -3.4822080284357071e-02</internalNodes>
          <leafValues>
            -4.9751108884811401e-01 1.3959939777851105e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 15 7.5519368983805180e-03</internalNodes>
          <leafValues>
            -9.2147678136825562e-02 1.1294340342283249e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 16 -1.7572140321135521e-02</internalNodes>
          <leafValues>
            -5.6784427165985107e-01 9.3572810292243958e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 17 5.2012042142450809e-03</internalNodes>
          <leafValues>
            -7.9238079488277435e-02 6.1878960579633713e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 18 -3.0798919498920441e-02</internalNodes>
          <leafValues>
            -5.6658512353897095e-01 9.5271490514278412e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 19 -1.3465429656207561e-03</internalNodes>
          <leafValues>
            2.4011470377445221e-01 -2.6026639342308044e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>33</maxWeakCount>
      <stageThreshold>-1.1226719617843628e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 20 1.9108939450234175e-03</internalNodes>
          <leafValues>
            -4.6240958571434021e-01 3.0612170696258545e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 21 9.5464065670967102e-03</internalNodes>
          <leafValues>
            9.1956138610839844e-02 -5.3501170873641968e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 22 -4.3402809649705887e-02</internalNodes>
          <leafValues>
            5.6817841529846191e-01 -1.1284930258989334e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 23 5.0386030226945877e-02</internalNodes>
          <leafValues>
            -8.0316931009292603e-02 7.3521858453750610e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 24 -6.8480317713692784e-04</internalNodes>
          <leafValues>
            2.5798648595809937e-01 -2.8049409389495850e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 25 1.1548049747943878e-01</internalNodes>
          <leafValues>
            9.2065572738647461e-02 -7.5556892156600952e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 26 -1.9348369678482413e-03</internalNodes>
          <leafValues>
            2.9440790414810181e-01 -2.4102710187435150e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 27 -4.3528810143470764e-02</internalNodes>
          <leafValues>
            4.9202969670295715e-01 -3.9650101214647293e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 28 -3.0218150466680527e-02</internalNodes>
          <leafValues>
            7.7227920293807983e-01 -8.6786523461341858e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 29 2.4536589160561562e-02</internalNodes>
          <leafValues>
            9.5944821834564209e-02 -4.8642969131469727e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 30 2.3958990350365639e-02</internalNodes>
          <leafValues>
            1.0437840223312378e-01 -5.1219838857650757e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 31 -2.5370830669999123e-02</internalNodes>
          <leafValues>
            -3.1981548666954041e-01 9.1486573219299316e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 32 -1.8606419907882810e-03</internalNodes>
          <leafValues>
            2.2783969342708588e-01 -2.4307970702648163e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 33 2.2550800815224648e-02</internalNodes>
          <leafValues>
            6.9207556545734406e-02 -3.0054280161857605e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 34 -4.9752090126276016e-02</internalNodes>
          <leafValues>
            -6.1078047752380371e-01 9.4472773373126984e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 35 -2.6602389290928841e-02</internalNodes>
          <leafValues>
            5.9581768512725830e-01 -9.2046052217483521e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 36 1.0760000348091125e-01</internalNodes>
          <leafValues>
            1.0278519988059998e-01 -5.4303371906280518e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 37 1.7690699547529221e-02</internalNodes>
          <leafValues>
            6.6057138144969940e-02 -6.3213908672332764e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 38 -6.2409918755292892e-02</internalNodes>
          <leafValues>
            6.8724197149276733e-01 -6.7070558667182922e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 39 -1.9801619928330183e-03</internalNodes>
          <leafValues>
            9.4411551952362061e-02 -8.7819486856460571e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 40 6.3668429851531982e-02</internalNodes>
          <leafValues>
            1.1531739681959152e-01 -4.8129761219024658e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 41 -3.0797829851508141e-02</internalNodes>
          <leafValues>
            3.5854768753051758e-01 -1.2593799829483032e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 42 -1.8353419727645814e-04</internalNodes>
          <leafValues>
            1.4788399636745453e-01 -2.8546810150146484e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 43 1.7074620118364692e-03</internalNodes>
          <leafValues>
            7.9929657280445099e-02 -2.5233370065689087e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 44 -1.5325199812650681e-02</internalNodes>
          <leafValues>
            -5.7711857557296753e-01 9.8908327519893646e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 45 4.1389189660549164e-02</internalNodes>
          <leafValues>
            -6.5550796687602997e-02 5.7363802194595337e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 46 -4.5577771379612386e-04</internalNodes>
          <leafValues>
            2.2593089938163757e-01 -1.9105580449104309e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 47 -1.3455689884722233e-02</internalNodes>
          <leafValues>
            -4.0233930945396423e-01 8.6477622389793396e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 48 -3.7978399544954300e-02</internalNodes>
          <leafValues>
            5.5257588624954224e-01 -8.1541016697883606e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 49 -1.7197500914335251e-02</internalNodes>
          <leafValues>
            -1.8363009393215179e-01 5.1999870687723160e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 50 -1.2581580085679889e-03</internalNodes>
          <leafValues>
            1.8830040097236633e-01 -2.5726661086082458e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 51 6.7725107073783875e-02</internalNodes>
          <leafValues>
            -8.0956451594829559e-02 7.1803241968154907e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 52 3.5489428788423538e-02</internalNodes>
          <leafValues>
            1.0068070143461227e-01 -5.3774142265319824e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>29</maxWeakCount>
      <stageThreshold>-1.0127470493316650e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 53 -5.3695798851549625e-03</internalNodes>
          <leafValues>
            2.7479499578475952e-01 -3.4178960323333740e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 54 6.2695867381989956e-04</internalNodes>
          <leafValues>
            -9.8646633327007294e-02 1.0728420317173004e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 55 -1.6484269872307777e-02</internalNodes>
          <leafValues>
            -6.4972907304763794e-01 9.6037752926349640e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 56 -2.2104099392890930e-02</internalNodes>
          <leafValues>
            -4.5984488725662231e-01 1.6304630041122437e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 57 1.1904139816761017e-01</internalNodes>
          <leafValues>
            -9.9600397050380707e-02 7.3729759454727173e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 58 -2.0222070161253214e-03</internalNodes>
          <leafValues>
            2.1029269695281982e-01 -2.4577130377292633e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 59 6.7500352859497070e-02</internalNodes>
          <leafValues>
            -1.2467789649963379e-01 5.7654231786727905e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 60 -1.9655939936637878e-01</internalNodes>
          <leafValues>
            -6.0891747474670410e-01 9.9672056734561920e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 61 4.9431171268224716e-02</internalNodes>
          <leafValues>
            1.3752749562263489e-01 -4.5580869913101196e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 62 2.3380089551210403e-02</internalNodes>
          <leafValues>
            4.7141890972852707e-02 -3.5027709603309631e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 63 1.3998650247231126e-03</internalNodes>
          <leafValues>
            -2.0643049478530884e-01 2.4322299659252167e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 64 1.1432689614593983e-02</internalNodes>
          <leafValues>
            5.5187370628118515e-02 -3.2619899511337280e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 65 4.8775069415569305e-02</internalNodes>
          <leafValues>
            -6.8992510437965393e-02 7.1171808242797852e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 66 6.5284021198749542e-02</internalNodes>
          <leafValues>
            3.7155740428715944e-03 5.9318971633911133e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 67 6.1603228095918894e-04</internalNodes>
          <leafValues>
            -2.3272520303726196e-01 2.0441530644893646e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 68 -1.0527499951422215e-02</internalNodes>
          <leafValues>
            -3.1773790717124939e-01 1.0171309858560562e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 69 1.6231339424848557e-02</internalNodes>
          <leafValues>
            9.1734193265438080e-02 -4.7143009305000305e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 70 3.8958500954322517e-04</internalNodes>
          <leafValues>
            -1.2997549772262573e-01 1.3475489616394043e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 71 -4.4165689498186111e-02</internalNodes>
          <leafValues>
            -6.0331028699874878e-01 6.4766876399517059e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 72 -1.3663209974765778e-02</internalNodes>
          <leafValues>
            -5.2762842178344727e-01 6.3485741615295410e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 73 -8.8231859263032675e-04</internalNodes>
          <leafValues>
            1.4510250091552734e-01 -2.7845200896263123e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 74 -2.7819190174341202e-02</internalNodes>
          <leafValues>
            4.3640869855880737e-01 -8.5191860795021057e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 75 6.2560990452766418e-02</internalNodes>
          <leafValues>
            1.0027889907360077e-01 -4.2235919833183289e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 76 -4.4808178790844977e-04</internalNodes>
          <leafValues>
            1.4851489663124084e-01 -1.7731289565563202e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 77 -2.1363180130720139e-02</internalNodes>
          <leafValues>
            -6.1334460973739624e-01 6.0539398342370987e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 78 -6.9122329354286194e-02</internalNodes>
          <leafValues>
            -8.6845761537551880e-01 3.9347749203443527e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 79 -3.0542839318513870e-02</internalNodes>
          <leafValues>
            -6.4021718502044678e-01 4.9593821167945862e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 80 -1.0101160034537315e-02</internalNodes>
          <leafValues>
            -1.6199150681495667e-01 5.7256899774074554e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 81 -2.2010109387338161e-04</internalNodes>
          <leafValues>
            2.1350930631160736e-01 -2.0198999345302582e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>42</maxWeakCount>
      <stageThreshold>-1.0684469938278198e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 82 5.7967850007116795e-03</internalNodes>
          <leafValues>
            -3.3844178915023804e-01 2.5066271424293518e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 83 6.3795179128646851e-02</internalNodes>
          <leafValues>
            -4.2111620306968689e-02 3.5746571421623230e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 84 -6.4332038164138794e-02</internalNodes>
          <leafValues>
            -5.0660789012908936e-01 1.1717739701271057e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 85 -1.1574289947748184e-01</internalNodes>
          <leafValues>
            -5.6678497791290283e-01 9.5880903303623199e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 86 -3.9005130529403687e-03</internalNodes>
          <leafValues>
            -4.1498228907585144e-01 1.4858320355415344e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 87 1.2512929737567902e-02</internalNodes>
          <leafValues>
            5.3696669638156891e-02 -1.4163960516452789e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 88 1.5871099894866347e-03</internalNodes>
          <leafValues>
            -2.5962340831756592e-01 1.9418330490589142e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 89 1.6291120648384094e-01</internalNodes>
          <leafValues>
            -6.1243768781423569e-02 7.8567212820053101e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 90 -3.3258220553398132e-01</internalNodes>
          <leafValues>
            7.8020131587982178e-01 -4.4036459177732468e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 91 -1.0288899764418602e-02</internalNodes>
          <leafValues>
            -1.5289680659770966e-01 6.2096230685710907e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 92 2.8956029564142227e-02</internalNodes>
          <leafValues>
            8.4707796573638916e-02 -4.7820711135864258e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 93 -3.2221511355601251e-04</internalNodes>
          <leafValues>
            1.3951259851455688e-01 -1.8819390237331390e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 94 1.5835289657115936e-01</internalNodes>
          <leafValues>
            6.6667810082435608e-02 -5.4572361707687378e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 95 -4.2584311217069626e-02</internalNodes>
          <leafValues>
            2.7040338516235352e-01 -5.6654509156942368e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 96 2.7505140751600266e-02</internalNodes>
          <leafValues>
            4.9271158874034882e-02 -7.3157638311386108e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 97 8.6879700422286987e-02</internalNodes>
          <leafValues>
            -1.7532400786876678e-02 8.6782652139663696e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 98 -2.0130439661443233e-03</internalNodes>
          <leafValues>
            1.6593940556049347e-01 -2.5266230106353760e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 99 4.2330170981585979e-04</internalNodes>
          <leafValues>
            9.4223551452159882e-02 -2.4629700183868408e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 100 1.5194499865174294e-02</internalNodes>
          <leafValues>
            7.3695637285709381e-02 -5.0068622827529907e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 101 -6.1203669756650925e-03</internalNodes>
          <leafValues>
            2.1381899714469910e-01 -1.6738100349903107e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 102 2.0660240203142166e-02</internalNodes>
          <leafValues>
            -8.0636158585548401e-02 5.7828348875045776e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 103 -6.0398250818252563e-02</internalNodes>
          <leafValues>
            -6.3411772251129150e-01 5.0899010151624680e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 104 3.5386480391025543e-02</internalNodes>
          <leafValues>
            7.3191151022911072e-02 -5.6426662206649780e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 105 -6.5997838973999023e-02</internalNodes>
          <leafValues>
            3.2833808660507202e-01 -2.6310259476304054e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 106 1.1004590196534991e-03</internalNodes>
          <leafValues>
            -2.3114609718322754e-01 2.0206519961357117e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 107 8.4488153457641602e-02</internalNodes>
          <leafValues>
            7.4589841067790985e-02 -4.3710339069366455e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 108 -2.9235990718007088e-02</internalNodes>
          <leafValues>
            6.5064769983291626e-01 -5.4531838744878769e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 109 -3.3916950225830078e-02</internalNodes>
          <leafValues>
            -2.8804349899291992e-01 3.2172881066799164e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 110 -7.9108700156211853e-03</internalNodes>
          <leafValues>
            -3.3660379052162170e-01 1.0100690275430679e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 111 5.1930431276559830e-02</internalNodes>
          <leafValues>
            3.2920960336923599e-02 -1.3176530599594116e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 112 -6.8586103618144989e-02</internalNodes>
          <leafValues>
            5.2153557538986206e-01 -6.6718578338623047e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 113 -1.9451669650152326e-03</internalNodes>
          <leafValues>
            1.5396790206432343e-01 -1.9895760715007782e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 114 7.1366228163242340e-02</internalNodes>
          <leafValues>
            -8.2927159965038300e-02 4.5292338728904724e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 115 -2.6624239981174469e-02</internalNodes>
          <leafValues>
            -4.4009739160537720e-01 1.0267119854688644e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 116 2.5266060605645180e-02</internalNodes>
          <leafValues>
            5.5799201130867004e-02 -5.5569338798522949e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 117 5.5255689658224583e-03</internalNodes>
          <leafValues>
            -1.3640299439430237e-01 2.8255200386047363e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 118 -2.9929999727755785e-03</internalNodes>
          <leafValues>
            -3.2421571016311646e-01 1.2122060358524323e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 119 2.2192109376192093e-02</internalNodes>
          <leafValues>
            -6.0741018503904343e-02 4.3473160266876221e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 120 -9.4268741086125374e-03</internalNodes>
          <leafValues>
            -3.3458408713340759e-01 1.0029699653387070e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 121 3.4395330585539341e-03</internalNodes>
          <leafValues>
            -8.3829909563064575e-02 1.7925940454006195e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 122 -3.2996390946209431e-03</internalNodes>
          <leafValues>
            1.9990429282188416e-01 -2.1068470180034637e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 123 2.6152150705456734e-02</internalNodes>
          <leafValues>
            -8.0667406320571899e-02 3.5581269860267639e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>45</maxWeakCount>
      <stageThreshold>-1.1520069837570190e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 124 -2.2792650386691093e-02</internalNodes>
          <leafValues>
            4.0725260972976685e-01 -3.3609920740127563e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 125 -5.7334620505571365e-03</internalNodes>
          <leafValues>
            2.6882189512252808e-01 -2.2775350511074066e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 126 9.6941202878952026e-02</internalNodes>
          <leafValues>
            -8.0905012786388397e-02 7.4328738451004028e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 127 -2.8288999572396278e-02</internalNodes>
          <leafValues>
            4.5610108971595764e-01 -6.1096340417861938e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 128 3.8522849790751934e-03</internalNodes>
          <leafValues>
            -2.5241801142692566e-01 2.0907109975814819e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 129 2.3100129328668118e-03</internalNodes>
          <leafValues>
            -1.4713400602340698e-01 1.5460389852523804e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 130 1.1361920041963458e-03</internalNodes>
          <leafValues>
            1.7680479586124420e-01 -3.0537289381027222e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 131 2.4962890893220901e-02</internalNodes>
          <leafValues>
            -1.2652909755706787e-01 3.7442651391029358e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 132 -5.8984099887311459e-03</internalNodes>
          <leafValues>
            2.6738989353179932e-01 -1.7762570083141327e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 133 1.1804900132119656e-02</internalNodes>
          <leafValues>
            6.6077977418899536e-02 -3.3482131361961365e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 134 6.4400159753859043e-03</internalNodes>
          <leafValues>
            1.0994800180196762e-01 -3.6303481459617615e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 135 -8.9407369494438171e-02</internalNodes>
          <leafValues>
            -4.3580460548400879e-01 1.4944310300052166e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 136 -3.1404230743646622e-02</internalNodes>
          <leafValues>
            6.9523447751998901e-01 -5.4854288697242737e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 137 -1.4607949554920197e-01</internalNodes>
          <leafValues>
            -2.5650060176849365e-01 5.6956540793180466e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 138 2.1142649929970503e-03</internalNodes>
          <leafValues>
            -2.4987550079822540e-01 1.6792559623718262e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 139 -1.5119359828531742e-02</internalNodes>
          <leafValues>
            -3.0179870128631592e-01 1.0393589735031128e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 140 2.5620959699153900e-02</internalNodes>
          <leafValues>
            -7.4821300804615021e-02 5.3600782155990601e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 141 -1.4417800307273865e-01</internalNodes>
          <leafValues>
            -2.0490899682044983e-01 7.4457786977291107e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 142 2.5954779237508774e-02</internalNodes>
          <leafValues>
            -9.0574868023395538e-02 4.8442208766937256e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 143 -2.1130720153450966e-02</internalNodes>
          <leafValues>
            -2.2689810395240784e-01 6.4876057207584381e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 144 1.6474459320306778e-02</internalNodes>
          <leafValues>
            1.0768000036478043e-01 -3.6570599675178528e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 145 1.0922150313854218e-01</internalNodes>
          <leafValues>
            5.6827351450920105e-02 -3.4728559851646423e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 146 -7.4581061198841780e-05</internalNodes>
          <leafValues>
            1.3904270529747009e-01 -2.5942608714103699e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 147 -2.7753600850701332e-02</internalNodes>
          <leafValues>
            3.8111299276351929e-01 -4.2896129190921783e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 148 3.2721430063247681e-02</internalNodes>
          <leafValues>
            -9.0872153639793396e-02 3.9289179444313049e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 149 5.5606258101761341e-03</internalNodes>
          <leafValues>
            8.4002248942852020e-02 -1.9396039843559265e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 150 -1.0710290074348450e-01</internalNodes>
          <leafValues>
            -5.8981472253799438e-01 5.6862760335206985e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 151 -8.0517623573541641e-03</internalNodes>
          <leafValues>
            1.1790599673986435e-01 -1.1595659703016281e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 152 -1.3850019872188568e-01</internalNodes>
          <leafValues>
            -9.0805321931838989e-01 4.1411358863115311e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 153 2.8620919212698936e-02</internalNodes>
          <leafValues>
            1.9928589463233948e-02 -7.3697662353515625e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 154 2.6208970695734024e-02</internalNodes>
          <leafValues>
            -6.1577551066875458e-02 6.0899931192398071e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 155 2.6527039706707001e-02</internalNodes>
          <leafValues>
            5.7193860411643982e-02 -6.2992326915264130e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 156 -4.4622488319873810e-02</internalNodes>
          <leafValues>
            -3.3318150043487549e-01 9.3214571475982666e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 157 -1.4283119700849056e-02</internalNodes>
          <leafValues>
            1.9125230610370636e-01 -1.1530569940805435e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 158 -1.9681209232658148e-03</internalNodes>
          <leafValues>
            -3.1295120716094971e-01 9.9682807922363281e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 159 5.2851080894470215e-02</internalNodes>
          <leafValues>
            -5.8919548988342285e-02 5.7887911796569824e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 160 -6.3711861148476601e-03</internalNodes>
          <leafValues>
            1.9182190299034119e-01 -1.9094540178775787e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 161 -6.4727910794317722e-03</internalNodes>
          <leafValues>
            -2.4721039831638336e-01 1.2252929806709290e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 162 -1.6690989956259727e-02</internalNodes>
          <leafValues>
            -4.9174660444259644e-01 5.0315100699663162e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 163 -1.4882409945130348e-02</internalNodes>
          <leafValues>
            1.9646610319614410e-01 -5.8250389993190765e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 164 1.7529709264636040e-02</internalNodes>
          <leafValues>
            7.6357498764991760e-02 -3.6559268832206726e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 165 4.2221389710903168e-02</internalNodes>
          <leafValues>
            -3.1560491770505905e-02 3.6011269688606262e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 166 -6.5581746399402618e-02</internalNodes>
          <leafValues>
            3.4334710240364075e-01 -8.8556960225105286e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 167 1.6703210771083832e-02</internalNodes>
          <leafValues>
            4.8210039734840393e-02 -1.5273620188236237e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 168 -6.9328742101788521e-03</internalNodes>
          <leafValues>
            -3.0573639273643494e-01 1.1821140348911285e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>46</maxWeakCount>
      <stageThreshold>-1.0648390054702759e+00</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 169 -6.3434438779950142e-03</internalNodes>
          <leafValues>
            3.3840280771255493e-01 -3.3474850654602051e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 170 5.2472548559308052e-03</internalNodes>
          <leafValues>
            -9.3596532940864563e-02 1.6791179776191711e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 171 -3.6585088819265366e-02</internalNodes>
          <leafValues>
            5.3676098585128784e-01 -8.5433527827262878e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 172 5.3153699263930321e-03</internalNodes>
          <leafValues>
            -1.2804119288921356e-01 1.4443910121917725e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 173 -3.9569609798491001e-03</internalNodes>
          <leafValues>
            1.8605449795722961e-01 -2.2311410307884216e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 174 3.3965419977903366e-02</internalNodes>
          <leafValues>
            2.7835709974169731e-02 -5.1203387975692749e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 175 -1.4852879568934441e-02</internalNodes>
          <leafValues>
            -4.6814951300621033e-01 1.1351560056209564e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 176 -2.9641329310834408e-03</internalNodes>
          <leafValues>
            2.6591798663139343e-01 -2.8183770179748535e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 177 -1.0795590281486511e-01</internalNodes>
          <leafValues>
            -5.7527697086334229e-01 1.0991639643907547e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 178 2.1237600594758987e-02</internalNodes>
          <leafValues>
            -1.0451590269804001e-01 4.6613770723342896e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 179 -2.6189640164375305e-02</internalNodes>
          <leafValues>
            4.2544820904731750e-01 -9.2278912663459778e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 180 -3.5010561347007751e-02</internalNodes>
          <leafValues>
            -7.1801197528839111e-01 7.2877250611782074e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 181 1.5026619621494319e-05</internalNodes>
          <leafValues>
            -2.7199760079383850e-01 1.0682159662246704e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 182 -2.7760250493884087e-02</internalNodes>
          <leafValues>
            -5.0185692310333252e-01 1.0118210315704346e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 183 -3.7439178675413132e-02</internalNodes>
          <leafValues>
            -3.7141519784927368e-01 8.3709038794040680e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 184 -1.4152259565889835e-02</internalNodes>
          <leafValues>
            3.0982801318168640e-01 -7.3767662048339844e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 185 -1.2331079691648483e-02</internalNodes>
          <leafValues>
            -3.9507681131362915e-01 8.3215177059173584e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 186 2.6666349731385708e-03</internalNodes>
          <leafValues>
            -1.3776129484176636e-01 2.4245689809322357e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 187 -2.9443199746310711e-03</internalNodes>
          <leafValues>
            2.4460780620574951e-01 -1.3937890529632568e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 188 -1.5788920223712921e-01</internalNodes>
          <leafValues>
            -5.6832242012023926e-01 3.6140721291303635e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 189 2.1553030237555504e-03</internalNodes>
          <leafValues>
            8.3660557866096497e-02 -4.1380259394645691e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 190 -8.5367091000080109e-02</internalNodes>
          <leafValues>
            -5.7053291797637939e-01 5.2995659410953522e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 191 3.4761740826070309e-03</internalNodes>
          <leafValues>
            -1.2189819663763046e-01 2.6553291082382202e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 192 -2.4104220792651176e-02</internalNodes>
          <leafValues>
            -5.2315437793731689e-01 2.5505660101771355e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 193 -3.0729150399565697e-02</internalNodes>
          <leafValues>
            -4.6735408902168274e-01 7.0844426751136780e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 194 -1.1937420349568129e-03</internalNodes>
          <leafValues>
            1.4596860110759735e-01 -2.3086270689964294e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 195 3.2304100692272186e-02</internalNodes>
          <leafValues>
            -6.5350927412509918e-02 5.5091381072998047e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 196 1.4955499768257141e-01</internalNodes>
          <leafValues>
            1.5002089552581310e-02 -8.9400452375411987e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 197 -4.7254669480025768e-03</internalNodes>
          <leafValues>
            1.4857460558414459e-01 -2.1019940078258514e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 198 3.6360718309879303e-02</internalNodes>
          <leafValues>
            2.8547950088977814e-02 -6.3668930530548096e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 199 -2.7109999209642410e-02</internalNodes>
          <leafValues>
            4.9661910533905029e-01 -7.3661573231220245e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 200 -9.5398407429456711e-03</internalNodes>
          <leafValues>
            -1.9384680688381195e-01 5.8507081121206284e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 201 1.0541989654302597e-01</internalNodes>
          <leafValues>
            -7.4785731732845306e-02 4.3781110644340515e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 202 6.3801761716604233e-03</internalNodes>
          <leafValues>
            5.3971529006958008e-02 -3.3829790353775024e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 203 -2.2759849205613136e-02</internalNodes>
          <leafValues>
            -5.9374898672103882e-01 4.8046529293060303e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 204 -1.7323749139904976e-02</internalNodes>
          <leafValues>
            -1.6034699976444244e-01 1.5187160111963749e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 205 2.9854409396648407e-02</internalNodes>
          <leafValues>
            -6.5698243677616119e-02 4.5057341456413269e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 206 2.3269839584827423e-02</internalNodes>
          <leafValues>
            3.8805499672889709e-02 -3.5354879498481750e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 207 4.0833871811628342e-02</internalNodes>
          <leafValues>
            4.9404840916395187e-02 -5.6222450733184814e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 208 -1.2498889863491058e-01</internalNodes>
          <leafValues>
            6.7763668298721313e-01 -1.5484940260648727e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 209 -6.5579377114772797e-02</internalNodes>
          <leafValues>
            6.7363232374191284e-01 -4.5269690454006195e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 210 -3.7901759147644043e-01</internalNodes>
          <leafValues>
            -4.9853721261024475e-01 2.3955229669809341e-02</leafValues></_>
        <_>
          <internalNodes>
            0 -1 211 2.9792459681630135e-03</internalNodes>
          <leafValues>
            -1.8436419963836670e-01 1.6265830397605896e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 212 1.3803659938275814e-02</internalNodes>
          <leafValues>
            6.3698217272758484e-02 -4.3389800190925598e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 213 3.5606899764388800e-03</internalNodes>
          <leafValues>
            -1.1455070227384567e-01 2.3618610203266144e-01</leafValues></_>
        <_>
          <internalNodes>
            0 -1 214 8.8772783055901527e-03</internalNodes>
          <leafValues>
            8.6416840553283691e-02 -1.7590980231761932e-01</leafValues></_></weakClassifiers></_>
    <_>
      <maxWeakCount>45</maxWeakCount>
      <stageThreshold>-9.5069932937622070e-01</stageThreshold>
      <weakClassifiers>
        <_>
          <internalNodes>
            0 -1 215 -6.7344820126891136e-03</internalNodes>
          <leafValues>
            3.0758589506149292e-01 -2.9761791229248047e-0
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

