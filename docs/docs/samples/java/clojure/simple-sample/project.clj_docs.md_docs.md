# Documentation for `docs/samples/java/clojure/simple-sample/project.clj_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/clojure/simple-sample/project.clj_docs.md`
- **File Name**: `project.clj_docs.md`
- **File Size**: 1,311 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/clojure/simple-sample/project.clj_docs.md](../../../../../docs/samples/java/clojure/simple-sample/project.clj_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/clojure/simple-sample` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/clojure/simple-sample/project.clj`

## File Metadata

- **Full Path**: `samples/java/clojure/simple-sample/project.clj`
- **File Name**: `project.clj`
- **File Size**: 680 bytes
- **File Type**: .clj
- **Link to Source**: [samples/java/clojure/simple-sample/project.clj](../../../../samples/java/clojure/simple-sample/project.clj)

## Purpose and Role

This file is located in the `samples/java/clojure/simple-sample` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
(defproject simple-sample "0.1.0-SNAPSHOT"
  :pom-addition [:developers [:developer {:id "magomimmo"}
                              [:name "Mimmo Cosenza"]
                              [:url "https://github.com/magomimmoo"]]]

  :description "A simple project to start REPLing with OpenCV"
  :url "http://example.com/FIXME"
  :license {:name "Apache 2.0 License"
            :url "https://www.apache.org/licenses/LICENSE-2.0"}
  :dependencies [[org.clojure/clojure "1.5.1"]
                 [opencv/opencv "2.4.7"]
                 [opencv/opencv-native "2.4.7"]]
  :main simple-sample.core
  :injections [(clojure.lang.RT/loadLibrary org.opencv.core.Core/NATIVE_LIBRARY_NAME)])

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

