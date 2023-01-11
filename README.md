# Project Copacetic: Directly patch container image vulnerabilities

`copa` is a CLI tool written in [Go](https://golang.org) and based on [buildkit](https://github.com/moby/buildkit) that can be used to directly patch container images given the vulnerability scanning results from popular tools like [Trivy](https://github.com/aquasecurity/trivy).

## Why?

We needed the ability to patch containers quickly without going upstream for a full rebuild. As the window between [vulnerability disclosure and active exploitation continues to narrow](https://www.bleepingcomputer.com/news/security/hackers-scan-for-vulnerabilities-within-15-minutes-of-disclosure/), there is a growing operational need to patch critical security vulnerabilities in container images so they can be quickly redeployed into production. The need is especially acute when those vulnerabilities are:

- inherited from base images several levels deep and waiting on updated releases to percolate through the supply chain is not an option
- found in 3rd party app images you don't maintain with update cadences that don't meet your security SLAs.

![direct image patching](./docs/imgs/direct-image-patching.png)

In addition to filling the operational gap not met by left-shift security practices and tools, the ability of `copa` to patch a container without requiring a rebuild of the container image provides other benefits:

- Allows users other than the image publishers to also patch container images, such as DevSecOps engineers.
- Reduces the storage and transmission costs of redistributing patched images by only creating an additional patch layer, instead of rebuilding the entire image which usually results in different layer hashes that break layer caching.
- Reduces the turnaround time for patching a container image by not having to wait for base image updates and being a faster operation than a full image rebuild.
- Reduces the complexity of patching the image from running a rebuild pipeline to running a single tool on the image.

## How?

The `copa` tool is an extensible engine that:

1. Parses the needed update packages from the container image’s vulnerability report produced by a scanner like Trivy. New adapters can be written to accommodate more report formats.
2. Obtains and processes the needed update packages using the appropriate package manager tools such as apt, apk, etc. New adapters can be written to support more package managers.
3. Applies the resulting update binaries to the container image using buildkit.

![report-driven vulnerability patching](./docs/imgs/vulnerability-patch.png)

This approach is motivated by the core principles of making direct container patching broadly applicable and accessible:

- **Copa supports patching _existing_ container images**.
  - Devs don't need to build their images using specific tools or modify them in some way just to support container patching.
- **Copa works with the existing vulnerability scanning and mitigation ecosystems**.
  - Image publishers don't need to create new workflows for container patching since Copa supports patching container images using the security update packages already being published today.
  - Consumers do not need to migrate to a new and potentially more limited support ecosystem for custom distros or change their container vulnerability scanning pipelines to include remediation, since Copa can be integrated seamlessly as an extra step to patch containers based on those scanning reports.
- **Copa reduces the technical expertise needed and waiting on dependencies needed to patch an image**.
  - For OS package vulnerabilities, no specialized knowledge about a specific image is needed to be patch it as Copa relies on the vulnerability remediation knowledge already embedded in the reports produced by popular container scanning tools today.

For more details refer to the [copa design](./docs/vulnerability-driven-patching.md) document.

## Getting Started

1. [Setup and build copa](./docs/tutorials/dev-setup.md).
2. Run through the sample for [using copa with container scanning reports](./docs/tutorials/patch.md).

### Features

`copa` currently supports:

- Parsing container scanning reports from:
  - [trivy](https://github.com/aquasecurity/trivy)
- Patching OS vulnerabilities through the most commonly used Linux OS package managers:
  - dpkg/apt (Ubuntu and Debian)
  - apk (Alpine)
  - rpm (RHEL)
- Patching amd64 and arm64 images.

In addition, `copa` takes advantage of buildkit's layer graph to support patching images where the package tools may not be present in the target image, for example, [Google distroless images](https://github.com/GoogleContainerTools/distroless/blob/main/base/README.md).

### Future possibilities

As an extensible tool, `copa` is expected to continue to grow in its capabilities. Additions being considered include:

- Support for more formats including:
  - [grype](https://github.com/anchore/grype)
  - custom `copa` json schema for manually specified fixes.
- Support for broader range of patching functionality including:
  - Patching more types of images without package manager tools present.
  - Patching based on app framework package managers such as pypi, npm, and maven.
  - Additional options for configuring custom patch sources, vulnerability types and criticality.
- Better integration buildkit options.
