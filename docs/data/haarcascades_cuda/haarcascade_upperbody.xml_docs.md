# Documentation for `data/haarcascades_cuda/haarcascade_upperbody.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_upperbody.xml`
- **File Name**: `haarcascade_upperbody.xml`
- **File Size**: 1,046,376 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_upperbody.xml](../../data/haarcascades_cuda/haarcascade_upperbody.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

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
hkruppa@inf.ethz.cz


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
<haarcascade_upperbody type_id="opencv-haar-classifier">
  <size>22 18</size>
  <stages>
    <_>
      <!-- stage 0 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 12 6 -1.</_>
                <_>9 5 4 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0136960297822952</threshold>
            <left_val>0.4507646858692169</left_val>
            <right_val>-0.4217903017997742</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 13 10 4 -1.</_>
                <_>7 15 10 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0124414497986436</threshold>
            <left_val>0.1649325042963028</left_val>
            <right_val>-0.7479348778724670</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 14 9 4 -1.</_>
                <_>6 14 3 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.7094660326838493e-003</threshold>
            <left_val>0.3100470006465912</left_val>
            <right_val>-0.3761714100837708</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 6 5 6 -1.</_>
                <_>15 6 5 3 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.1000801026821137</threshold>
            <left_val>0.7618219852447510</left_val>
            <right_val>-0.0745569765567780</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 1 22 14 -1.</_>
                <_>11 1 11 14 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.2511411905288696</threshold>
            <left_val>-0.6415402889251709</left_val>
            <right_val>0.1513922065496445</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 11 20 4 -1.</_>
                <_>6 11 10 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1051065027713776</threshold>
            <left_val>0.7145937085151672</left_val>
            <right_val>-0.1449857950210571</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 6 6 5 -1.</_>
                <_>7 6 3 5 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0884480178356171</threshold>
            <left_val>0.7577317953109741</left_val>
            <right_val>-0.0685868933796883</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 13 12 4 -1.</_>
                <_>11 13 6 2 2.</_>
                <_>5 15 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0108749102801085</threshold>
            <left_val>0.1461060941219330</left_val>
            <right_val>-0.5426371097564697</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 12 8 6 -1.</_>
                <_>7 12 4 3 2.</_>
                <_>11 15 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0126905702054501</threshold>
            <left_val>0.1167458966374397</left_val>
            <right_val>-0.4964945912361145</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>20 0 2 18 -1.</_>
                <_>20 9 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0321983993053436</threshold>
            <left_val>-0.3852939009666443</left_val>
            <right_val>0.0984379723668098</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 6 6 12 -1.</_>
                <_>10 6 2 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.4077179152518511e-003</threshold>
            <left_val>0.2520087063312531</left_val>
            <right_val>-0.2238254994153976</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 5 6 6 -1.</_>
                <_>10 5 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0303243901580572</threshold>
            <left_val>-0.1053444966673851</left_val>
            <right_val>0.6573541760444641</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 15 12 2 -1.</_>
                <_>5 16 12 1 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.1930507868528366e-003</threshold>
            <left_val>0.1287239938974381</left_val>
            <right_val>-0.5316066145896912</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>20 0 2 18 -1.</_>
                <_>20 9 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0805014073848724</threshold>
            <left_val>0.0416966602206230</left_val>
            <right_val>-0.7212303280830383</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 0 2 18 -1.</_>
                <_>0 9 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0348220802843571</threshold>
            <left_val>-0.4975110888481140</left_val>
            <right_val>0.1395993977785111</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 7 6 4 -1.</_>
                <_>13 7 6 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>7.5519368983805180e-003</threshold>
            <left_val>-0.0921476781368256</left_val>
            <right_val>0.1129434034228325</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 14 7 4 -1.</_>
                <_>2 16 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0175721403211355</threshold>
            <left_val>-0.5678442716598511</left_val>
            <right_val>0.0935728102922440</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 7 7 4 -1.</_>
                <_>13 7 7 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>5.2012042142450809e-003</threshold>
            <left_val>-0.0792380794882774</left_val>
            <right_val>0.0618789605796337</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 6 4 12 -1.</_>
                <_>4 10 4 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0307989194989204</threshold>
            <left_val>-0.5665851235389710</left_val>
            <right_val>0.0952714905142784</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 4 6 10 -1.</_>
                <_>11 4 3 5 2.</_>
                <_>8 9 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.3465429656207561e-003</threshold>
            <left_val>0.2401147037744522</left_val>
            <right_val>-0.2602663934230804</right_val></_></_></trees>
      <stage_threshold>-1.1264339685440063</stage_threshold>
      <parent>-1</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 1 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 8 6 10 -1.</_>
                <_>6 8 3 5 2.</_>
                <_>9 13 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.9108939450234175e-003</threshold>
            <left_val>-0.4624095857143402</left_val>
            <right_val>0.3061217069625855</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 12 6 6 -1.</_>
                <_>11 15 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>9.5464065670967102e-003</threshold>
            <left_val>0.0919561386108398</left_val>
            <right_val>-0.5350117087364197</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 15 8 3 -1.</_>
                <_>5 15 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0434028096497059</threshold>
            <left_val>0.5681784152984619</left_val>
            <right_val>-0.1128493025898933</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 9 10 4 -1.</_>
                <_>6 11 10 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0503860302269459</threshold>
            <left_val>-0.0803169310092926</left_val>
            <right_val>0.7352185845375061</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 5 8 3 -1.</_>
                <_>10 6 8 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-6.8480317713692784e-004</threshold>
            <left_val>0.2579864859580994</left_val>
            <right_val>-0.2804940938949585</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 13 22 5 -1.</_>
                <_>0 13 11 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1154804974794388</threshold>
            <left_val>0.0920655727386475</left_val>
            <right_val>-0.7555689215660095</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 13 14 3 -1.</_>
                <_>9 13 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.9348369678482413e-003</threshold>
            <left_val>0.2944079041481018</left_val>
            <right_val>-0.2410271018743515</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 5 2 10 -1.</_>
                <_>11 5 1 10 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0435288101434708</threshold>
            <left_val>0.4920296967029572</left_val>
            <right_val>-0.0396501012146473</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 5 10 2 -1.</_>
                <_>11 5 10 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0302181504666805</threshold>
            <left_val>0.7722792029380798</left_val>
            <right_val>-0.0867865234613419</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 0 8 8 -1.</_>
                <_>18 0 4 4 2.</_>
                <_>14 4 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0245365891605616</threshold>
            <left_val>0.0959448218345642</left_val>
            <right_val>-0.4864296913146973</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 0 3 10 -1.</_>
                <_>5 5 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0239589903503656</threshold>
            <left_val>0.1043784022331238</left_val>
            <right_val>-0.5121983885765076</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>16 0 3 12 -1.</_>
                <_>16 6 3 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0253708306699991</threshold>
            <left_val>-0.3198154866695404</left_val>
            <right_val>0.0914865732192993</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 3 12 4 -1.</_>
                <_>3 3 6 2 2.</_>
                <_>9 5 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.8606419907882810e-003</threshold>
            <left_val>0.2278396934270859</left_val>
            <right_val>-0.2430797070264816</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 2 20 3 -1.</_>
                <_>7 2 10 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0225508008152246</threshold>
            <left_val>0.0692075565457344</left_val>
            <right_val>-0.3005428016185761</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 7 3 8 -1.</_>
                <_>11 7 3 4 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0497520901262760</threshold>
            <left_val>-0.6107804775238037</left_val>
            <right_val>0.0944727733731270</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 9 18 3 -1.</_>
                <_>4 10 18 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0266023892909288</threshold>
            <left_val>0.5958176851272583</left_val>
            <right_val>-0.0920460522174835</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 3 16 14 -1.</_>
                <_>3 3 8 7 2.</_>
                <_>11 10 8 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1076000034809113</threshold>
            <left_val>0.1027851998806000</left_val>
            <right_val>-0.5430337190628052</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 14 8 4 -1.</_>
                <_>7 14 4 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0176906995475292</threshold>
            <left_val>0.0660571381449699</left_val>
            <right_val>-0.6321390867233276</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 7 4 7 -1.</_>
                <_>10 7 2 7 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0624099187552929</threshold>
            <left_val>0.6872419714927673</left_val>
            <right_val>-0.0670705586671829</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 9 6 5 -1.</_>
                <_>11 9 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.9801619928330183e-003</threshold>
            <left_val>0.0944115519523621</left_val>
            <right_val>-0.0878194868564606</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 6 22 4 -1.</_>
                <_>11 6 11 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0636684298515320</threshold>
            <left_val>0.1153173968195915</left_val>
            <right_val>-0.4812976121902466</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 6 6 12 -1.</_>
                <_>17 6 3 6 2.</_>
                <_>14 12 3 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0307978298515081</threshold>
            <left_val>0.3585476875305176</left_val>
            <right_val>-0.1259379982948303</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 14 6 4 -1.</_>
                <_>4 16 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.8353419727645814e-004</threshold>
            <left_val>0.1478839963674545</left_val>
            <right_val>-0.2854681015014648</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 14 6 4 -1.</_>
                <_>12 16 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.7074620118364692e-003</threshold>
            <left_val>0.0799296572804451</left_val>
            <right_val>-0.2523337006568909</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 14 6 4 -1.</_>
                <_>4 16 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0153251998126507</threshold>
            <left_val>-0.5771185755729675</left_val>
            <right_val>0.0989083275198936</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 6 6 6 -1.</_>
                <_>12 6 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0413891896605492</threshold>
            <left_val>-0.0655507966876030</left_val>
            <right_val>0.5736380219459534</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 11 3 -1.</_>
                <_>8 1 11 1 3.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-4.5577771379612386e-004</threshold>
            <left_val>0.2259308993816376</left_val>
            <right_val>-0.1910558044910431</right_val></_></_>
        <_>
          <!-- tree 27 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 0 12 4 -1.</_>
                <_>13 0 6 2 2.</_>
                <_>7 2 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0134556898847222</threshold>
            <left_val>-0.4023393094539642</left_val>
            <right_val>0.0864776223897934</right_val></_></_>
        <_>
          <!-- tree 28 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 6 6 6 -1.</_>
                <_>8 6 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0379783995449543</threshold>
            <left_val>0.5525758862495422</left_val>
            <right_val>-0.0815410166978836</right_val></_></_>
        <_>
          <!-- tree 29 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 5 3 8 -1.</_>
                <_>15 9 3 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0171975009143353</threshold>
            <left_val>-0.1836300939321518</left_val>
            <right_val>0.0519998706877232</right_val></_></_>
        <_>
          <!-- tree 30 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 2 12 7 -1.</_>
                <_>9 2 4 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.2581580085679889e-003</threshold>
            <left_val>0.1883004009723663</left_val>
            <right_val>-0.2572666108608246</right_val></_></_>
        <_>
          <!-- tree 31 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 12 4 -1.</_>
                <_>9 5 4 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0677251070737839</threshold>
            <left_val>-0.0809564515948296</left_val>
            <right_val>0.7180324196815491</right_val></_></_>
        <_>
          <!-- tree 32 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 3 4 7 -1.</_>
                <_>7 3 2 7 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0354894287884235</threshold>
            <left_val>0.1006807014346123</left_val>
            <right_val>-0.5377414226531982</right_val></_></_></trees>
      <stage_threshold>-1.1226719617843628</stage_threshold>
      <parent>0</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 2 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 14 6 4 -1.</_>
                <_>5 14 3 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.3695798851549625e-003</threshold>
            <left_val>0.2747949957847595</left_val>
            <right_val>-0.3417896032333374</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 4 6 6 -1.</_>
                <_>13 4 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.2695867381989956e-004</threshold>
            <left_val>-0.0986466333270073</left_val>
            <right_val>0.1072842031717300</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 14 12 4 -1.</_>
                <_>5 14 6 2 2.</_>
                <_>11 16 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0164842698723078</threshold>
            <left_val>-0.6497290730476379</left_val>
            <right_val>0.0960377529263496</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 12 16 6 -1.</_>
                <_>11 12 8 3 2.</_>
                <_>3 15 8 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0221040993928909</threshold>
            <left_val>-0.4598448872566223</left_val>
            <right_val>0.1630463004112244</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 11 20 4 -1.</_>
                <_>6 11 10 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1190413981676102</threshold>
            <left_val>-0.0996003970503807</left_val>
            <right_val>0.7372975945472717</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 10 10 -1.</_>
                <_>14 0 5 5 2.</_>
                <_>9 5 5 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.0222070161253214e-003</threshold>
            <left_val>0.2102926969528198</left_val>
            <right_val>-0.2457713037729263</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 8 4 6 -1.</_>
                <_>8 8 2 6 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0675003528594971</threshold>
            <left_val>-0.1246778964996338</left_val>
            <right_val>0.5765423178672791</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 7 20 11 -1.</_>
                <_>1 7 10 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1965593993663788</threshold>
            <left_val>-0.6089174747467041</left_val>
            <right_val>0.0996720567345619</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 12 3 -1.</_>
                <_>9 0 6 3 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0494311712682247</threshold>
            <left_val>0.1375274956226349</left_val>
            <right_val>-0.4558086991310120</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 0 6 6 -1.</_>
                <_>13 0 3 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0233800895512104</threshold>
            <left_val>0.0471418909728527</left_val>
            <right_val>-0.3502770960330963</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 0 12 8 -1.</_>
                <_>5 2 12 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.3998650247231126e-003</threshold>
            <left_val>-0.2064304947853088</left_val>
            <right_val>0.2432229965925217</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 0 8 6 -1.</_>
                <_>18 0 4 3 2.</_>
                <_>14 3 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0114326896145940</threshold>
            <left_val>0.0551873706281185</left_val>
            <right_val>-0.3261989951133728</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 6 8 6 -1.</_>
                <_>9 6 4 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0487750694155693</threshold>
            <left_val>-0.0689925104379654</left_val>
            <right_val>0.7117180824279785</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 3 6 6 -1.</_>
                <_>13 3 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0652840211987495</threshold>
            <left_val>3.7155740428715944e-003</left_val>
            <right_val>0.5931897163391113</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 3 6 6 -1.</_>
                <_>7 3 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.1603228095918894e-004</threshold>
            <left_val>-0.2327252030372620</left_val>
            <right_val>0.2044153064489365</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>13 0 8 6 -1.</_>
                <_>17 0 4 3 2.</_>
                <_>13 3 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0105274999514222</threshold>
            <left_val>-0.3177379071712494</left_val>
            <right_val>0.1017130985856056</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 0 8 6 -1.</_>
                <_>0 0 4 3 2.</_>
                <_>4 3 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0162313394248486</threshold>
            <left_val>0.0917341932654381</left_val>
            <right_val>-0.4714300930500031</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 0 10 6 -1.</_>
                <_>12 0 5 3 2.</_>
                <_>7 3 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.8958500954322517e-004</threshold>
            <left_val>-0.1299754977226257</left_val>
            <right_val>0.1347548961639404</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 15 22 2 -1.</_>
                <_>11 15 11 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0441656894981861</threshold>
            <left_val>-0.6033102869987488</left_val>
            <right_val>0.0647668763995171</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 14 12 4 -1.</_>
                <_>5 15 12 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0136632099747658</threshold>
            <left_val>-0.5276284217834473</left_val>
            <right_val>0.0634857416152954</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 13 6 4 -1.</_>
                <_>5 15 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-8.8231859263032675e-004</threshold>
            <left_val>0.1451025009155273</left_val>
            <right_val>-0.2784520089626312</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 9 17 3 -1.</_>
                <_>3 10 17 1 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0278191901743412</threshold>
            <left_val>0.4364086985588074</left_val>
            <right_val>-0.0851918607950211</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 8 16 10 -1.</_>
                <_>3 8 8 5 2.</_>
                <_>11 13 8 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0625609904527664</threshold>
            <left_val>0.1002788990736008</left_val>
            <right_val>-0.4223591983318329</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 0 10 6 -1.</_>
                <_>14 0 5 3 2.</_>
                <_>9 3 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.4808178790844977e-004</threshold>
            <left_val>0.1485148966312408</left_val>
            <right_val>-0.1773128956556320</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 0 12 4 -1.</_>
                <_>3 0 6 2 2.</_>
                <_>9 2 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0213631801307201</threshold>
            <left_val>-0.6133446097373962</left_val>
            <right_val>0.0605393983423710</right_val></_></_>
        <_>
          <!-- tree 25 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 10 14 3 -1.</_>
                <_>4 10 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0691223293542862</threshold>
            <left_val>-0.8684576153755188</left_val>
            <right_val>0.0393477492034435</right_val></_></_>
        <_>
          <!-- tree 26 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 14 11 4 -1.</_>
                <_>1 16 11 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0305428393185139</threshold>
            <left_val>-0.6402171850204468</left_val>
            <right_val>0.0495938211679459</right_val></_></_>
        <_>
          <!-- tree 27 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 0 12 6 -1.</_>
                <_>13 0 6 3 2.</_>
                <_>7 3 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0101011600345373</threshold>
            <left_val>-0.1619915068149567</left_val>
            <right_val>0.0572568997740746</right_val></_></_>
        <_>
          <!-- tree 28 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 0 10 6 -1.</_>
                <_>3 0 5 3 2.</_>
                <_>8 3 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.2010109387338161e-004</threshold>
            <left_val>0.2135093063116074</left_val>
            <right_val>-0.2019899934530258</right_val></_></_></trees>
      <stage_threshold>-1.0127470493316650</stage_threshold>
      <parent>1</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 3 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 0 10 3 -1.</_>
                <_>6 0 5 3 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>5.7967850007116795e-003</threshold>
            <left_val>-0.3384417891502380</left_val>
            <right_val>0.2506627142429352</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 8 6 4 -1.</_>
                <_>14 8 6 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0637951791286469</threshold>
            <left_val>-0.0421116203069687</left_val>
            <right_val>0.3574657142162323</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 2 5 16 -1.</_>
                <_>0 10 5 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0643320381641388</threshold>
            <left_val>-0.5066078901290894</left_val>
            <right_val>0.1171773970127106</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 3 22 5 -1.</_>
                <_>0 3 11 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1157428994774818</threshold>
            <left_val>-0.5667849779129028</left_val>
            <right_val>0.0958809033036232</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 15 8 3 -1.</_>
                <_>10 15 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.9005130529403687e-003</threshold>
            <left_val>-0.4149822890758514</left_val>
            <right_val>0.1485832035541534</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>15 0 2 14 -1.</_>
                <_>15 0 1 14 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0125129297375679</threshold>
            <left_val>0.0536966696381569</left_val>
            <right_val>-0.1416396051645279</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 0 14 2 -1.</_>
                <_>7 0 14 1 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>1.5871099894866347e-003</threshold>
            <left_val>-0.2596234083175659</left_val>
            <right_val>0.1941833049058914</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 11 20 5 -1.</_>
                <_>6 11 10 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1629112064838409</threshold>
            <left_val>-0.0612437687814236</left_val>
            <right_val>0.7856721282005310</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 3 12 9 -1.</_>
                <_>9 6 4 3 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.3325822055339813</threshold>
            <left_val>0.7802013158798218</left_val>
            <right_val>-0.0440364591777325</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 1 12 3 -1.</_>
                <_>14 1 4 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0102888997644186</threshold>
            <left_val>-0.1528968065977097</left_val>
            <right_val>0.0620962306857109</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 1 12 3 -1.</_>
                <_>4 1 4 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0289560295641422</threshold>
            <left_val>0.0847077965736389</left_val>
            <right_val>-0.4782071113586426</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>14 12 4 6 -1.</_>
                <_>14 12 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.2221511355601251e-004</threshold>
            <left_val>0.1395125985145569</left_val>
            <right_val>-0.1881939023733139</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 10 22 7 -1.</_>
                <_>11 10 11 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1583528965711594</threshold>
            <left_val>0.0666678100824356</left_val>
            <right_val>-0.5457236170768738</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 2 4 11 -1.</_>
                <_>11 2 2 11 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0425843112170696</threshold>
            <left_val>0.2704033851623535</left_val>
            <right_val>-0.0566545091569424</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 14 16 4 -1.</_>
                <_>3 14 8 2 2.</_>
                <_>11 16 8 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0275051407516003</threshold>
            <left_val>0.0492711588740349</left_val>
            <right_val>-0.7315763831138611</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>12 12 6 6 -1.</_>
                <_>14 12 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0868797004222870</threshold>
            <left_val>-0.0175324007868767</left_val>
            <right_val>0.8678265213966370</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 12 6 6 -1.</_>
                <_>6 12 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.0130439661443233e-003</threshold>
            <left_val>0.1659394055604935</left_val>
            <right_val>-0.2526623010635376</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 14 6 4 -1.</_>
                <_>11 16 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>4.2330170981585979e-004</threshold>
            <left_val>0.0942235514521599</left_val>
            <right_val>-0.2462970018386841</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 0 12 4 -1.</_>
                <_>0 0 6 2 2.</_>
                <_>6 2 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0151944998651743</threshold>
            <left_val>0.0736956372857094</left_val>
            <right_val>-0.500686228275299
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

