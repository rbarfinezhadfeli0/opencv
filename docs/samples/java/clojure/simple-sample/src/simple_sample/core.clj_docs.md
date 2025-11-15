# Documentation for `samples/java/clojure/simple-sample/src/simple_sample/core.clj`

## File Metadata

- **Full Path**: `samples/java/clojure/simple-sample/src/simple_sample/core.clj`
- **File Name**: `core.clj`
- **File Size**: 627 bytes
- **File Type**: .clj
- **Link to Source**: [samples/java/clojure/simple-sample/src/simple_sample/core.clj](../../../../../../samples/java/clojure/simple-sample/src/simple_sample/core.clj)

## Purpose and Role

This file is located in the `samples/java/clojure/simple-sample/src/simple_sample` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
;;; to run this code from the terminal: "$ lein run". It will save a
;;; blurred image version of resources/images/lena.png as
;;; resources/images/blurred.png

(ns simple-sample.core
  (:import [org.opencv.core Point Rect Mat CvType Size Scalar]
           org.opencv.imgcodecs.Imgcodecs
           org.opencv.imgproc.Imgproc))

(defn -main [& args]
  (let [lena (Imgcodecs/imread "resources/images/lena.png")
        blurred (Mat. 512 512 CvType/CV_8UC3)]
    (print "Blurring...")
    (Imgproc/GaussianBlur lena blurred (Size. 5 5) 3 3)
    (Imgcodecs/imwrite "resources/images/blurred.png" blurred)
    (println "done!")))

```

## General Information

This file is part of the OpenCV repository infrastructure.

