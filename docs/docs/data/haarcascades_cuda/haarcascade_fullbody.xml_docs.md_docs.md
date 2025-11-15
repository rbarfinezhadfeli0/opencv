# Documentation for `docs/data/haarcascades_cuda/haarcascade_fullbody.xml_docs.md`

## File Metadata

- **Full Path**: `docs/data/haarcascades_cuda/haarcascade_fullbody.xml_docs.md`
- **File Name**: `haarcascade_fullbody.xml_docs.md`
- **File Size**: 51,058 bytes
- **File Type**: .md
- **Link to Source**: [docs/data/haarcascades_cuda/haarcascade_fullbody.xml_docs.md](../../../docs/data/haarcascades_cuda/haarcascade_fullbody.xml_docs.md)

## Purpose and Role

This file is located in the `docs/data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `data/haarcascades_cuda/haarcascade_fullbody.xml`

## File Metadata

- **Full Path**: `data/haarcascades_cuda/haarcascade_fullbody.xml`
- **File Name**: `haarcascade_fullbody.xml`
- **File Size**: 636,639 bytes
- **File Type**: .xml
- **Link to Source**: [data/haarcascades_cuda/haarcascade_fullbody.xml](../../data/haarcascades_cuda/haarcascade_fullbody.xml)

## Purpose and Role

This file is located in the `data/haarcascades_cuda` directory and serves as part of the OpenCV library infrastructure.

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
<haarcascade_fullbody type_id="opencv-haar-classifier">
  <size>14 28</size>
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
                <_>1 5 12 21 -1.</_>
                <_>5 5 4 21 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0558205693960190</threshold>
            <left_val>0.5869792103767395</left_val>
            <right_val>-0.6281142234802246</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 2 3 26 -1.</_>
                <_>9 15 3 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0388611815869808</threshold>
            <left_val>-0.7091681957244873</left_val>
            <right_val>0.2682121098041534</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 4 12 23 -1.</_>
                <_>5 4 4 23 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.2674087882041931</threshold>
            <left_val>0.8308296203613281</left_val>
            <right_val>-0.2259958982467651</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 7 12 9 -1.</_>
                <_>4 7 6 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0964197367429733</threshold>
            <left_val>-0.1169784963130951</left_val>
            <right_val>0.8725455999374390</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 12 3 16 -1.</_>
                <_>3 20 3 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0107987103983760</threshold>
            <left_val>-0.5721974968910217</left_val>
            <right_val>0.2532565891742706</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 8 6 6 -1.</_>
                <_>4 11 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0113656399771571</threshold>
            <left_val>0.1965083032846451</left_val>
            <right_val>-0.7274463772773743</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 25 12 3 -1.</_>
                <_>5 25 4 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.0216919044032693e-004</threshold>
            <left_val>0.2443515956401825</left_val>
            <right_val>-0.5197358131408691</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 25 8 3 -1.</_>
                <_>6 25 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0284624807536602</threshold>
            <left_val>-0.8360729217529297</left_val>
            <right_val>0.1115804016590118</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 2 4 12 -1.</_>
                <_>4 2 2 6 2.</_>
                <_>6 8 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.3473170110955834e-003</threshold>
            <left_val>-0.3840653896331787</left_val>
            <right_val>0.2676798999309540</right_val></_></_></trees>
      <stage_threshold>-1.2288980484008789</stage_threshold>
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
                <_>3 15 8 11 -1.</_>
                <_>5 15 4 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0107432203367352</threshold>
            <left_val>0.4774732887744904</left_val>
            <right_val>-0.6239293217658997</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 9 6 6 -1.</_>
                <_>8 9 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.3188569573685527e-003</threshold>
            <left_val>0.2124266028404236</left_val>
            <right_val>-0.2416270971298218</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 9 6 6 -1.</_>
                <_>4 9 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.5571161210536957e-003</threshold>
            <left_val>0.3614785969257355</left_val>
            <right_val>-0.3725171983242035</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 0 5 28 -1.</_>
                <_>8 14 5 14 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1389341056346893</threshold>
            <left_val>-0.6790050268173218</left_val>
            <right_val>0.1128031015396118</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 24 10 4 -1.</_>
                <_>7 24 5 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0264658294618130</threshold>
            <left_val>0.1247496977448463</left_val>
            <right_val>-0.8285233974456787</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 15 8 11 -1.</_>
                <_>5 15 4 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0893868431448936</threshold>
            <left_val>0.7427176237106323</left_val>
            <right_val>-0.1701931953430176</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 25 14 3 -1.</_>
                <_>7 25 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0213354192674160</threshold>
            <left_val>-0.7175018787384033</left_val>
            <right_val>0.1556618064641953</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 11 12 13 -1.</_>
                <_>5 11 4 13 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0557091012597084</threshold>
            <left_val>-0.1531004011631012</left_val>
            <right_val>0.7180476784706116</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 2 12 21 -1.</_>
                <_>5 9 4 7 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.6970995068550110</threshold>
            <left_val>0.8115419149398804</left_val>
            <right_val>-0.1088638976216316</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 0 3 28 -1.</_>
                <_>10 14 3 14 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2020599991083145</threshold>
            <left_val>0.0763984173536301</left_val>
            <right_val>-0.7301151156425476</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 0 3 28 -1.</_>
                <_>1 14 3 14 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0718826577067375</threshold>
            <left_val>-0.7148858904838562</left_val>
            <right_val>0.1651764959096909</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 5 6 8 -1.</_>
                <_>8 5 3 4 2.</_>
                <_>5 9 3 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0192287601530552</threshold>
            <left_val>-0.3986836969852448</left_val>
            <right_val>0.0405572392046452</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 5 6 8 -1.</_>
                <_>3 5 3 4 2.</_>
                <_>6 9 3 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>1.1500229593366385e-003</threshold>
            <left_val>-0.3826077878475189</left_val>
            <right_val>0.3185507953166962</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 16 4 12 -1.</_>
                <_>12 16 2 6 2.</_>
                <_>10 22 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0232527796179056</threshold>
            <left_val>0.0543904006481171</left_val>
            <right_val>-0.7066999077796936</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 8 6 4 -1.</_>
                <_>4 10 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.2618120894767344e-004</threshold>
            <left_val>0.2261060029268265</left_val>
            <right_val>-0.4070987999439240</right_val></_></_></trees>
      <stage_threshold>-1.0969949960708618</stage_threshold>
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
                <_>3 5 8 21 -1.</_>
                <_>5 5 4 21 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1291020065546036</threshold>
            <left_val>0.7600312829017639</left_val>
            <right_val>-0.2340579032897949</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 15 12 12 -1.</_>
                <_>7 15 6 6 2.</_>
                <_>1 21 6 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0674492567777634</threshold>
            <left_val>0.1717952936887741</left_val>
            <right_val>-0.8436477780342102</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 25 12 3 -1.</_>
                <_>6 25 6 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0126632703468204</threshold>
            <left_val>0.2291321009397507</left_val>
            <right_val>-0.7307245731353760</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 14 3 8 -1.</_>
                <_>8 14 3 4 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-4.2741331271827221e-003</threshold>
            <left_val>0.0624204799532890</left_val>
            <right_val>-0.4098593890666962</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 25 8 3 -1.</_>
                <_>4 25 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0231439508497715</threshold>
            <left_val>-0.8397182822227478</left_val>
            <right_val>0.2011574953794479</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 24 12 4 -1.</_>
                <_>5 24 6 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.5371038615703583e-004</threshold>
            <left_val>0.1536941975355148</left_val>
            <right_val>-0.4403811097145081</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 18 4 6 -1.</_>
                <_>3 18 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-9.5239803194999695e-003</threshold>
            <left_val>-0.6318680047988892</left_val>
            <right_val>0.1625023037195206</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 8 4 7 -1.</_>
                <_>8 8 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0283076707273722</threshold>
            <left_val>-0.0725999698042870</left_val>
            <right_val>0.3791998922824860</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 8 4 7 -1.</_>
                <_>4 8 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0451480187475681</threshold>
            <left_val>0.7449362874031067</left_val>
            <right_val>-0.1558171063661575</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 3 12 18 -1.</_>
                <_>1 3 6 18 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1001473963260651</threshold>
            <left_val>0.1794963926076889</left_val>
            <right_val>-0.6464408040046692</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 20 4 8 -1.</_>
                <_>3 20 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.3245721869170666e-003</threshold>
            <left_val>0.1776389926671982</left_val>
            <right_val>-0.5765405893325806</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 10 7 18 -1.</_>
                <_>6 19 7 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0118756704032421</threshold>
            <left_val>-0.3112972080707550</left_val>
            <right_val>0.1632139980792999</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 8 3 13 -1.</_>
                <_>5 8 1 13 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0254790391772985</threshold>
            <left_val>0.6269248127937317</left_val>
            <right_val>-0.1133375018835068</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 22 4 6 -1.</_>
                <_>10 22 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.9196523874998093e-003</threshold>
            <left_val>-0.7762442827224731</left_val>
            <right_val>0.1542761027812958</right_val></_></_></trees>
      <stage_threshold>-1.2285970449447632</stage_threshold>
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
                <_>1 0 12 27 -1.</_>
                <_>5 9 4 9 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.8580927848815918</threshold>
            <left_val>0.7879683971405029</left_val>
            <right_val>-0.2213554978370667</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 20 12 7 -1.</_>
                <_>5 20 6 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.6491119749844074e-003</threshold>
            <left_val>0.2567340135574341</left_val>
            <right_val>-0.4319424033164978</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 25 10 3 -1.</_>
                <_>7 25 5 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0258823093026876</threshold>
            <left_val>-0.8755123019218445</left_val>
            <right_val>0.0883856266736984</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 26 14 2 -1.</_>
                <_>0 26 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.7666151076555252e-003</threshold>
            <left_val>-0.4702236950397492</left_val>
            <right_val>0.2280080020427704</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 15 8 9 -1.</_>
                <_>5 15 4 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0837296992540360</threshold>
            <left_val>0.6338573098182678</left_val>
            <right_val>-0.1488831937313080</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 23 6 5 -1.</_>
                <_>8 23 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0406857393682003</threshold>
            <left_val>-0.9393178820610046</left_val>
            <right_val>0.0105989398434758</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 26 14 2 -1.</_>
                <_>7 26 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.0759920850396156e-003</threshold>
            <left_val>-0.4555442035198212</left_val>
            <right_val>0.1786437034606934</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 10 2 18 -1.</_>
                <_>8 19 2 9 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.3427829146385193e-003</threshold>
            <left_val>-0.2143428027629852</left_val>
            <right_val>0.1553142070770264</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 4 4 12 -1.</_>
                <_>4 4 2 6 2.</_>
                <_>6 10 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.7649151161313057e-004</threshold>
            <left_val>-0.3334816098213196</left_val>
            <right_val>0.2278023958206177</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 24 9 4 -1.</_>
                <_>7 24 3 4 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0169418398290873</threshold>
            <left_val>0.0741408169269562</left_val>
            <right_val>-0.5626205205917358</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 3 12 15 -1.</_>
                <_>5 8 4 5 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.4755898118019104</threshold>
            <left_val>-0.1086113005876541</left_val>
            <right_val>0.8298525810241699</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>11 16 2 12 -1.</_>
                <_>11 16 1 12 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.8000627905130386e-003</threshold>
            <left_val>0.1324903070926666</left_val>
            <right_val>-0.5162039995193481</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 4 7 16 -1.</_>
                <_>2 12 7 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0744775608181953</threshold>
            <left_val>-0.5554556846618652</left_val>
            <right_val>0.1234432011842728</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 8 4 6 -1.</_>
                <_>8 8 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.5143009154126048e-004</threshold>
            <left_val>0.0681907534599304</left_val>
            <right_val>-0.1361685991287231</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 8 8 6 -1.</_>
                <_>3 11 8 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.3454021476209164e-003</threshold>
            <left_val>0.1367851048707962</left_val>
            <right_val>-0.5364512205123901</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>8 8 6 8 -1.</_>
                <_>10 8 2 8 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0154712796211243</threshold>
            <left_val>0.2618063986301422</left_val>
            <right_val>-0.1054581031203270</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 8 6 7 -1.</_>
                <_>2 8 2 7 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>5.6055500172078609e-003</threshold>
            <left_val>-0.2574635148048401</left_val>
            <right_val>0.2879593074321747</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 25 12 3 -1.</_>
                <_>6 25 4 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.4552858667448163e-004</threshold>
            <left_val>0.1009993031620979</left_val>
            <right_val>-0.2611967921257019</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 25 12 3 -1.</_>
                <_>4 25 4 3 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0331389009952545</threshold>
            <left_val>-0.8377956748008728</left_val>
            <right_val>0.1132768988609314</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 7 12 4 -1.</_>
                <_>1 7 6 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0355918891727924</threshold>
            <left_val>0.0823360905051231</left_val>
            <right_val>-0.6250566244125366</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 2 14 12 -1.</_>
                <_>7 2 7 12 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2083403021097183</threshold>
            <left_val>0.0695244371891022</left_val>
            <right_val>-0.8688114881515503</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 19 14 6 -1.</_>
                <_>7 19 7 3 2.</_>
                <_>0 22 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0281654000282288</threshold>
            <left_val>-0.5979984998703003</left_val>
            <right_val>0.0803299024701118</right_val></_></_></trees>
      <stage_threshold>-1.1200269460678101</stage_threshold>
      <parent>2</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 4 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 14 12 6 -1.</_>
                <_>5 14 4 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0267407093197107</threshold>
            <left_val>0.3891242146492004</left_val>
            <right_val>-0.4982767999172211</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 24 12 4 -1.</_>
                <_>5 24 6 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.2516999850049615e-003</threshold>
            <left_val>0.1312343031167984</left_val>
            <right_val>-0.3636899888515472</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 1 4 14 -1.</_>
                <_>2 1 2 7 2.</_>
                <_>4 8 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0416345112025738</threshold>
            <left_val>0.5744475126266480</left_val>
            <right_val>-0.1393287926912308</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 3 4 6 -1.</_>
                <_>10 3 2 6 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0100965797901154</threshold>
            <left_val>0.0990737974643707</left_val>
            <right_val>-0.2295698970556259</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 3 6 4 -1.</_>
                <_>4 3 6 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>-0.0190903991460800</threshold>
            <left_val>-0.5515310764312744</left_val>
            <right_val>0.1511006951332092</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 16 14 8 -1.</_>
                <_>0 16 7 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0314810685813427</threshold>
            <left_val>-0.4588426947593689</left_val>
            <right_val>0.1757954955101013</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 16 3 12 -1.</_>
                <_>6 16 1 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0176875498145819</threshold>
            <left_val>0.4471183121204376</left_val>
            <right_val>-0.1529293060302734</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 15 4 7 -1.</_>
                <_>7 15 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.3685659766197205e-003</threshold>
            <left_val>0.1218549013137817</left_val>
            <right_val>-0.1668857038021088</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 15 4 8 -1.</_>
                <_>5 15 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.9326845481991768e-003</threshold>
            <left_val>-0.1333369016647339</left_val>
            <right_val>0.6375334262847900</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 17 4 8 -1.</_>
                <_>9 17 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.0706309266388416e-003</threshold>
            <left_val>-0.1122028976678848</left_val>
            <right_val>0.0698243528604507</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 17 4 8 -1.</_>
                <_>3 17 2 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-5.9803090989589691e-003</threshold>
            <left_val>-0.5184289813041687</left_val>
            <right_val>0.1609919965267181</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 18 4 7 -1.</_>
                <_>9 18 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.9967839363962412e-003</threshold>
            <left_val>0.0410653389990330</left_val>
            <right_val>-0.1945585012435913</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 18 4 7 -1.</_>
                <_>3 18 2 7 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>3.8641549181193113e-003</threshold>
            <left_val>0.1667324006557465</left_val>
            <right_val>-0.4356977939605713</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 5 4 6 -1.</_>
                <_>7 5 2 6 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>6.8349428474903107e-003</threshold>
            <left_val>-0.1716264039278030</left_val>
            <right_val>0.1481806039810181</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>7 5 6 4 -1.</_>
                <_>7 5 6 2 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0431584902107716</threshold>
            <left_val>0.0832035094499588</left_val>
            <right_val>-0.7782185077667236</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 26 12 2 -1.</_>
                <_>2 26 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>7.6560080051422119e-003</threshold>
            <left_val>0.0847408026456833</left_val>
            <right_val>-0.4973815083503723</right_val></_></_>
        <_>
          <!-- tree 16 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>4 7 3 12 -1.</_>
                <_>5 7 1 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.1110988929867744e-003</threshold>
            <left_val>0.2582714855670929</left_val>
            <right_val>-0.2555203139781952</right_val></_></_>
        <_>
          <!-- tree 17 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 7 12 11 -1.</_>
                <_>4 7 6 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.1187030971050263</threshold>
            <left_val>-0.0909442380070686</left_val>
            <right_val>0.7228621244430542</right_val></_></_>
        <_>
          <!-- tree 18 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 13 8 4 -1.</_>
                <_>6 13 4 4 2.</_></rects>
              <tilted>1</tilted></feature>
            <threshold>0.0168759692460299</threshold>
            <left_val>0.1262917071580887</left_val>
            <right_val>-0.5520529747009277</right_val></_></_>
        <_>
          <!-- tree 19 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 22 6 4 -1.</_>
                <_>5 22 3 4 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-1.0887029930017889e-004</threshold>
            <left_val>0.0816487967967987</left_val>
            <right_val>-0.1693702042102814</right_val></_></_>
        <_>
          <!-- tree 20 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 26 14 2 -1.</_>
                <_>7 26 7 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>2.8222990222275257e-003</threshold>
            <left_val>0.1641130000352860</left_val>
            <right_val>-0.3521826863288879</right_val></_></_>
        <_>
          <!-- tree 21 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 3 12 18 -1.</_>
                <_>5 9 4 6 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.5242584943771362</threshold>
            <left_val>0.4890617132186890</left_val>
            <right_val>-0.1267475932836533</right_val></_></_>
        <_>
          <!-- tree 22 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 6 9 22 -1.</_>
                <_>0 17 9 11 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.3692750930786133</threshold>
            <left_val>0.0861159935593605</left_val>
            <right_val>-0.6718463897705078</right_val></_></_>
        <_>
          <!-- tree 23 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 1 12 24 -1.</_>
                <_>7 1 6 12 2.</_>
                <_>1 13 6 12 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1688378006219864</threshold>
            <left_val>-0.8491569161415100</left_val>
            <right_val>0.0548333488404751</right_val></_></_>
        <_>
          <!-- tree 24 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 26 12 2 -1.</_>
                <_>6 26 6 2 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0192792601883411</threshold>
            <left_val>-0.7801151275634766</left_val>
            <right_val>0.0622026808559895</right_val></_></_></trees>
      <stage_threshold>-1.0664960145950317</stage_threshold>
      <parent>3</parent>
      <next>-1</next></_>
    <_>
      <!-- stage 5 -->
      <trees>
        <_>
          <!-- tree 0 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 4 12 23 -1.</_>
                <_>5 4 4 23 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.2090135067701340</threshold>
            <left_val>0.6980816721916199</left_val>
            <right_val>-0.3457359075546265</right_val></_></_>
        <_>
          <!-- tree 1 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 22 6 5 -1.</_>
                <_>5 22 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.8061009147204459e-004</threshold>
            <left_val>0.2092390060424805</left_val>
            <right_val>-0.2414764016866684</right_val></_></_>
        <_>
          <!-- tree 2 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 22 6 5 -1.</_>
                <_>6 22 3 5 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.4844119325280190e-003</threshold>
            <left_val>0.2763600945472717</left_val>
            <right_val>-0.4199039936065674</right_val></_></_>
        <_>
          <!-- tree 3 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 1 4 6 -1.</_>
                <_>5 4 4 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.1536289714276791e-003</threshold>
            <left_val>0.2471046000719070</left_val>
            <right_val>-0.3067789971828461</right_val></_></_>
        <_>
          <!-- tree 4 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>1 8 12 8 -1.</_>
                <_>4 8 6 8 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0589119903743267</threshold>
            <left_val>-0.0708347633481026</left_val>
            <right_val>0.7113314270973206</right_val></_></_>
        <_>
          <!-- tree 5 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 8 5 12 -1.</_>
                <_>6 11 5 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-2.3095219512470067e-004</threshold>
            <left_val>0.1714860051870346</left_val>
            <right_val>-0.3616837859153748</right_val></_></_>
        <_>
          <!-- tree 6 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 20 14 6 -1.</_>
                <_>0 20 7 3 2.</_>
                <_>7 23 7 3 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0313964001834393</threshold>
            <left_val>-0.8013188242912293</left_val>
            <right_val>0.1004256010055542</right_val></_></_>
        <_>
          <!-- tree 7 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 9 6 6 -1.</_>
                <_>8 9 2 6 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-3.5601970739662647e-003</threshold>
            <left_val>0.0994327664375305</left_val>
            <right_val>-0.1484826058149338</right_val></_></_>
        <_>
          <!-- tree 8 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>5 8 4 6 -1.</_>
                <_>7 8 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-4.3389322236180305e-003</threshold>
            <left_val>-0.5662124156951904</left_val>
            <right_val>0.1409679949283600</right_val></_></_>
        <_>
          <!-- tree 9 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>2 13 12 15 -1.</_>
                <_>2 18 12 5 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.2132671028375626</threshold>
            <left_val>0.0481582097709179</left_val>
            <right_val>-0.7485890984535217</right_val></_></_>
        <_>
          <!-- tree 10 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 16 4 12 -1.</_>
                <_>0 16 2 6 2.</_>
                <_>2 22 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>0.0100425295531750</threshold>
            <left_val>0.1042840033769608</left_val>
            <right_val>-0.5538737773895264</right_val></_></_>
        <_>
          <!-- tree 11 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>9 2 2 26 -1.</_>
                <_>10 2 1 13 2.</_>
                <_>9 15 1 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.0268252808600664</threshold>
            <left_val>0.5728160738945007</left_val>
            <right_val>-0.0825379788875580</right_val></_></_>
        <_>
          <!-- tree 12 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>3 2 2 26 -1.</_>
                <_>3 2 1 13 2.</_>
                <_>4 15 1 13 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>8.3760882262140512e-004</threshold>
            <left_val>-0.2562690079212189</left_val>
            <right_val>0.2589842081069946</right_val></_></_>
        <_>
          <!-- tree 13 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>10 22 4 6 -1.</_>
                <_>10 22 2 6 2.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-7.6051978394389153e-003</threshold>
            <left_val>-0.5867735743522644</left_val>
            <right_val>0.0512107796967030</right_val></_></_>
        <_>
          <!-- tree 14 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>0 1 12 12 -1.</_>
                <_>4 5 4 4 9.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>-0.1193564012646675</threshold>
            <left_val>-0.4553082883358002</left_val>
            <right_val>0.1257033050060272</right_val></_></_>
        <_>
          <!-- tree 15 -->
          <_>
            <!-- root node -->
            <feature>
              <rects>
                <_>6 15 3 12 -1.</_>
                <_>7 15 1 12 3.</_></rects>
              <tilted>0</tilted></feature>
            <threshold>6.6083478741347790e-003</threshold>
            <left_val>-0.1631637960672379</left_val>
            <right_val>0.46
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

