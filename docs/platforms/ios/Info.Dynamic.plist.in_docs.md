# Documentation for `platforms/ios/Info.Dynamic.plist.in`

## File Metadata

- **Full Path**: `platforms/ios/Info.Dynamic.plist.in`
- **File Name**: `Info.Dynamic.plist.in`
- **File Size**: 1,058 bytes
- **File Type**: .in
- **Link to Source**: [platforms/ios/Info.Dynamic.plist.in](../../platforms/ios/Info.Dynamic.plist.in)

## Purpose and Role

This file is located in the `platforms/ios` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleDevelopmentRegion</key>
    <string>en</string>
    <key>CFBundleExecutable</key>
    <string>${FRAMEWORK_NAME}</string>
    <key>CFBundleName</key>
    <string>${OPENCV_APPLE_BUNDLE_NAME}</string>
    <key>CFBundleIdentifier</key>
    <string>${OPENCV_APPLE_BUNDLE_ID}</string>
    <key>CFBundleVersion</key>
    <string>${OPENCV_LIBVERSION}</string>
    <key>CFBundleShortVersionString</key>
    <string>${OPENCV_LIBVERSION}</string>
    <key>CFBundleSignature</key>
    <string>????</string>
    <key>CFBundlePackageType</key>
    <string>FMWK</string>
    <key>CFBundleSupportedPlatforms</key>
    <array>
        <string>iPhoneOS</string>
    </array>
    <key>MinimumOSVersion</key>
    <string>${IPHONEOS_DEPLOYMENT_TARGET}</string>
    <key>UIDeviceFamily</key>
    <array>
        <integer>1</integer>
        <integer>2</integer>
    </array>
</dict>
</plist>

```

## General Information

This file is part of the OpenCV repository infrastructure.

