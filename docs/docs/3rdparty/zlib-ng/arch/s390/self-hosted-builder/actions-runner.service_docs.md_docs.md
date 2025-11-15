# Documentation for `docs/3rdparty/zlib-ng/arch/s390/self-hosted-builder/actions-runner.service_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/zlib-ng/arch/s390/self-hosted-builder/actions-runner.service_docs.md`
- **File Name**: `actions-runner.service_docs.md`
- **File Size**: 1,242 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/zlib-ng/arch/s390/self-hosted-builder/actions-runner.service_docs.md](../../../../../../docs/3rdparty/zlib-ng/arch/s390/self-hosted-builder/actions-runner.service_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/zlib-ng/arch/s390/self-hosted-builder` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/zlib-ng/arch/s390/self-hosted-builder/actions-runner.service`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/arch/s390/self-hosted-builder/actions-runner.service`
- **File Name**: `actions-runner.service`
- **File Size**: 489 bytes
- **File Type**: .service
- **Link to Source**: [3rdparty/zlib-ng/arch/s390/self-hosted-builder/actions-runner.service](../../../../../3rdparty/zlib-ng/arch/s390/self-hosted-builder/actions-runner.service)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng/arch/s390/self-hosted-builder` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
[Unit]
Description=Podman container: Gaplib Github Actions Runner
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=1
RequiresMountsFor=/run/user/1001/containers

[Service]
Environment=PODMAN_SYSTEMD_UNIT=%n
Restart=always
TimeoutStopSec=61
ExecStart=/usr/bin/podman start gaplib-actions-runner
ExecStop=/usr/bin/podman stop -t 1 gaplib-actions-runner
ExecStopPost=/usr/bin/podman stop -t 1 gaplib-actions-runner
Type=forking

[Install]
WantedBy=default.target

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

