# Documentation for `docs/samples/java/sbt/src/main/scala/Main.scala_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/sbt/src/main/scala/Main.scala_docs.md`
- **File Name**: `Main.scala_docs.md`
- **File Size**: 1,427 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/sbt/src/main/scala/Main.scala_docs.md](../../../../../../../docs/samples/java/sbt/src/main/scala/Main.scala_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/sbt/src/main/scala` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/sbt/src/main/scala/Main.scala`

## File Metadata

- **Full Path**: `samples/java/sbt/src/main/scala/Main.scala`
- **File Name**: `Main.scala`
- **File Size**: 808 bytes
- **File Type**: .scala
- **Link to Source**: [samples/java/sbt/src/main/scala/Main.scala](../../../../../../samples/java/sbt/src/main/scala/Main.scala)

## Purpose and Role

This file is located in the `samples/java/sbt/src/main/scala` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
/*
 * The main runner for the Java demos.
 * Demos whose name begins with "Scala" are written in the Scala language,
 * demonstrating the generic nature of the interface.
 * The other demos are in Java.
 * Currently, all demos are run, sequentially.
 *
 * You're invited to submit your own examples, in any JVM language of
 * your choosing so long as you can get them to build.
 */

import org.opencv.core.Core

object Main extends App {
  // We must load the native library before using any OpenCV functions.
  // You must load this library _exactly once_ per Java invocation.
  // If you load it more than once, you will get a java.lang.UnsatisfiedLinkError.
  System.loadLibrary(Core.NATIVE_LIBRARY_NAME)

  ScalaCorrespondenceMatchingDemo.run()
  ScalaDetectFaceDemo.run()
  new DetectFaceDemo().run()
}

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

