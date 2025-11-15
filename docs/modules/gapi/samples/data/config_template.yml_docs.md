# Documentation for `modules/gapi/samples/data/config_template.yml`

## File Metadata

- **Full Path**: `modules/gapi/samples/data/config_template.yml`
- **File Name**: `config_template.yml`
- **File Size**: 4,301 bytes
- **File Type**: .yml
- **Link to Source**: [modules/gapi/samples/data/config_template.yml](../../../../modules/gapi/samples/data/config_template.yml)

## Purpose and Role

This file is located in the `modules/gapi/samples/data` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
%YAML:1.0

# Application running time in milliseconds: integer.
work_time: 2000

Pipelines:
    PL1:
      source:
        name: 'Src'
        latency: 33.0
        output:
          dims: [1, 3, 1280, 720]
          precision: 'U8'

      nodes:
        - name: 'PP'
          type: 'Dummy'
          time: 0.2
          output:
            dims: [1, 3, 300, 300]
            precision: 'U8'

        - name: 'Infer'
          type: 'Infer'
          xml: 'face-detection-retail-0004.xml'
          bin: 'face-detection-retail-0004.bin'
          device: 'CPU'
          input_layers:
            - 'data'
          output_layers:
            - 'detection_out'

      edges:
        - from: 'Src'
          to: 'PP'
        - from: 'PP'
          to: 'Infer'

      # Path to the dump file (*.dot)'
      dump: 'pl1.dot'

    PL2:
      source:
        name: 'Src'
        latency: 50.0
        output:
          dims: [1, 3, 1280, 720]
          precision: 'U8'

      nodes:
        - name: 'M1_PP'
          type: 'Dummy'
          time: 0.2
          output:
            dims: [1, 3, 300, 300]
            precision: 'U8'

        - name: 'M1_Infer'
          type: 'Infer'
          xml: 'face-detection-retail-0004.xml'
          bin: 'face-detection-retail-0004.bin'
          device: 'CPU'
          input_layers:
            - 'data'
          output_layers:
            - 'detection_out'

        - name: 'M2_PP'
          type: 'Dummy'
          time: 0.2
          output:
            dims: [1, 3, 300, 300]
            precision: 'U8'

        - name: 'M2_Infer'
          type: 'Infer'
          xml: 'face-detection-retail-0004.xml'
          bin: 'face-detection-retail-0004.bin'
          device: 'CPU'
          input_layers:
            - 'data'
          output_layers:
            - 'detection_out'

        - name: 'M3_PP'
          type: 'Dummy'
          time: 0.2
          output:
            dims: [1, 3, 300, 300]
            precision: 'U8'

        - name: 'M3_Infer'
          type: 'Infer'
          xml: 'face-detection-retail-0004.xml'
          bin: 'face-detection-retail-0004.bin'
          device: 'CPU'
          input_layers:
            - 'data'
          output_layers:
            - 'detection_out'

        - name: 'M4_PP'
          type: 'Dummy'
          time: 0.2
          output:
            dims: [1, 3, 300, 300]
            precision: 'U8'

        - name: 'M4_Infer'
          type: 'Infer'
          xml: 'face-detection-retail-0004.xml'
          bin: 'face-detection-retail-0004.bin'
          device: 'CPU'
          input_layers:
            - 'data'
          output_layers:
            - 'detection_out'

        - name: 'M5_PP'
          type: 'Dummy'
          time: 0.2
          output:
            dims: [1, 3, 300, 300]
            precision: 'U8'

        - name: 'M5_Infer'
          type: 'Infer'
          xml: 'face-detection-retail-0004.xml'
          bin: 'face-detection-retail-0004.bin'
          device: 'CPU'
          input_layers:
            - 'data'
          output_layers:
            - 'detection_out'

      edges:
        - from: 'Src'
          to: 'M1_PP'
        - from: 'M1_PP'
          to: 'M1_Infer'
        - from: 'M1_Infer'
          to: 'M2_PP'
        - from: 'M2_PP'
          to: 'M2_Infer'
        - from: 'M2_Infer'
          to: 'M3_PP'
        - from: 'M3_PP'
          to: 'M3_Infer'
        - from: 'M3_Infer'
          to: 'M4_PP'
        - from: 'M4_PP'
          to: 'M4_Infer'
        - from: 'M4_Infer'
          to: 'M5_PP'
        - from: 'M5_PP'
          to: 'M5_Infer'

      dump: 'pl2.dot'

    PL3:
      source:
        name: 'Src'
        latency: 33.0
        output:
          dims: [1, 3, 1280, 720]
          precision: 'U8'

      nodes:
        - name: 'PP'
          type: 'Dummy'
          time: 0.2
          output:
            dims: [1, 3, 300, 300]
            precision: 'U8'

        - name: 'Infer'
          type: 'Infer'
          xml: 'face-detection-retail-0004.xml'
          bin: 'face-detection-retail-0004.bin'
          device: 'CPU'
          input_layers:
            - 'data'
          output_layers:
            - 'detection_out'

      edges:
        - from: 'Src'
          to: 'PP'
        - from: 'PP'
          to: 'Infer'

      dump: 'pl3.dot'

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

