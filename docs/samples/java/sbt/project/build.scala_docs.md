# Documentation for `samples/java/sbt/project/build.scala`

## File Metadata

- **Full Path**: `samples/java/sbt/project/build.scala`
- **File Name**: `build.scala`
- **File Size**: 464 bytes
- **File Type**: .scala
- **Link to Source**: [samples/java/sbt/project/build.scala](../../../../samples/java/sbt/project/build.scala)

## Purpose and Role

This file is located in the `samples/java/sbt/project` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
import sbt._
import Keys._

object OpenCVJavaDemoBuild extends Build {
  def scalaSettings = Seq(
    scalaVersion := "2.10.0",
    scalacOptions ++= Seq(
      "-optimize",
      "-unchecked",
      "-deprecation"
    )
  )

  def buildSettings =
    Project.defaultSettings ++
    scalaSettings

  lazy val root = {
    val settings = buildSettings ++ Seq(name := "OpenCVJavaDemo")
    Project(id = "OpenCVJavaDemo", base = file("."), settings = settings)
  }
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

