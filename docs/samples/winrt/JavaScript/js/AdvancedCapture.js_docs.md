# Documentation for `samples/winrt/JavaScript/js/AdvancedCapture.js`

## File Metadata

- **Full Path**: `samples/winrt/JavaScript/js/AdvancedCapture.js`
- **File Name**: `AdvancedCapture.js`
- **File Size**: 5,548 bytes
- **File Type**: .js
- **Link to Source**: [samples/winrt/JavaScript/js/AdvancedCapture.js](../../../../samples/winrt/JavaScript/js/AdvancedCapture.js)

## Purpose and Role

This file is located in the `samples/winrt/JavaScript/js` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
﻿//// THIS CODE AND INFORMATION IS PROVIDED "AS IS" WITHOUT WARRANTY OF
//// ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING BUT NOT LIMITED TO
//// THE IMPLIED WARRANTIES OF MERCHANTABILITY AND/OR FITNESS FOR A
//// PARTICULAR PURPOSE.
////
//// Copyright (c) Microsoft Corporation. All rights reserved

(function () {
    "use strict";

    var cameraList = null;
    var mediaCaptureMgr = null;
    var captureInitSettings = null;

    var page = WinJS.UI.Pages.define("/html/AdvancedCapture.html", {

        ready: function (element, options) {
            scenarioInitialize();
        },

        unload: function (element, options) {
            // release resources
            releaseMediaCapture();
        }
    });

    function scenarioInitialize() {
        // Initialize the UI elements
        id("btnStartDevice").disabled = false;
        id("btnStartDevice").addEventListener("click", startDevice, false);
        id("btnStartPreview").disabled = true;
        id("videoEffect").disabled = true;
        id("btnStartPreview").addEventListener("click", startPreview, false);
        id("cameraSelect").addEventListener("change", onDeviceChange, false);

        id("videoEffect").addEventListener('change', addEffectToImageStream, false);

        enumerateCameras();
    }

    function initCameraSettings() {
        captureInitSettings = new Windows.Media.Capture.MediaCaptureInitializationSettings();
        captureInitSettings.streamingCaptureMode = Windows.Media.Capture.StreamingCaptureMode.video

        // If the user chose another capture device, use it by default
        var selectedIndex = id("cameraSelect").selectedIndex;
        var deviceInfo = cameraList[selectedIndex];
        captureInitSettings.videoDeviceId = deviceInfo.id;
    }

    // this function takes care of releasing the resources associated with media capturing
    function releaseMediaCapture() {
        if (mediaCaptureMgr) {
            mediaCaptureMgr.close();
            mediaCaptureMgr = null;
        }
    }

    //Initialize media capture with the current settings
    function startDevice() {
        displayStatus("Starting device");
        releaseMediaCapture();
        initCameraSettings();

        mediaCaptureMgr = new Windows.Media.Capture.MediaCapture();
        mediaCaptureMgr.initializeAsync(captureInitSettings).done(function (result) {
            // Update the UI
            id("btnStartPreview").disabled = false;
            id("btnStartDevice").disabled = true;
            displayStatus("Device started");
        });
    }

    function startPreview() {
        displayStatus("Starting preview");
        id("btnStartPreview").disabled = true;
        id("videoEffect").disabled = false;
        var video = id("previewVideo");
        video.src = URL.createObjectURL(mediaCaptureMgr, { oneTimeOnly: true });
        video.play();
        displayStatus("Preview started");
    }

    function addEffectToImageStream() {
        var effectId = id("videoEffect").selectedIndex;
        var props = new Windows.Foundation.Collections.PropertySet();
        props.insert("{698649BE-8EAE-4551-A4CB-3EC98FBD3D86}", effectId);

        mediaCaptureMgr.clearEffectsAsync(Windows.Media.Capture.MediaStreamType.videoPreview).then(function () {
            return mediaCaptureMgr.addEffectAsync(Windows.Media.Capture.MediaStreamType.videoPreview, 'OcvTransform.OcvImageManipulations', props);
        }).then(function () {
            displayStatus('Effect has been successfully added');
        }, errorHandler);
    }

    function enumerateCameras() {
        displayStatus("Enumerating capture devices");
        var cameraSelect = id("cameraSelect");
        cameraList = null;
        cameraList = new Array();

        // Clear the previous list of capture devices if any
        while (cameraSelect.length > 0) {
            cameraSelect.remove(0);
        }

        // Enumerate cameras and add them to the list
        var deviceInfo = Windows.Devices.Enumeration.DeviceInformation;
        deviceInfo.findAllAsync(Windows.Devices.Enumeration.DeviceClass.videoCapture).done(function (cameras) {
            if (cameras.length === 0) {
                cameraSelect.disabled = true;
                displayError("No camera was found");
                id("btnStartDevice").disabled = true;
                cameraSelect.add(new Option("No cameras available"));
            } else {
                cameras.forEach(function (camera) {
                    cameraList.push(camera);
                    cameraSelect.add(new Option(camera.name));
                });
            }
        }, errorHandler);
    }

    function onDeviceChange() {
        releaseMediaCapture();
        id("btnStartDevice").disabled = false;
        id("btnStartPreview").disabled = true;
        id("videoEffect").disabled = true;
        displayStatus("");
    }

    function suspendingHandler(suspendArg) {
        displayStatus("Suspended");
        releaseMediaCapture();
    }

    function resumingHandler(resumeArg) {
        displayStatus("Resumed");
        scenarioInitialize();
    }

    function errorHandler(err) {
        displayError(err.message);
    }

    function failedEventHandler(e) {
        displayError("Fatal error", e.message);
    }

    function displayStatus(statusText) {
        SdkSample.displayStatus(statusText);
    }

    function displayError(error) {
        SdkSample.displayError(error);
    }

    function id(elementId) {
        return document.getElementById(elementId);
    }
})();
```

## High-Level Overview

This is a .js source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **id()**: A function/method defined in this file
- **displayStatus()**: A function/method defined in this file
- **enumerateCameras()**: A function/method defined in this file
- **addEffectToImageStream()**: A function/method defined in this file
- **failedEventHandler()**: A function/method defined in this file
- **takes()**: A function/method defined in this file
- **scenarioInitialize()**: A function/method defined in this file
- **startDevice()**: A function/method defined in this file
- **resumingHandler()**: A function/method defined in this file
- **suspendingHandler()**: A function/method defined in this file
- **releaseMediaCapture()**: A function/method defined in this file
- **initCameraSettings()**: A function/method defined in this file
- **errorHandler()**: A function/method defined in this file
- **onDeviceChange()**: A function/method defined in this file
- **startPreview()**: A function/method defined in this file
- **displayError()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.

