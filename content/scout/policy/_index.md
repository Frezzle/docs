---
title: Get started with Policy Evaluation in Docker Scout
keywords: scout, supply chain, vulnerabilities, packages, cves, policy
description: |
  Policies in Docker Scout let you define supply chain rules and thresholds
  for your artifacts, and track how your artifacts perform against those
  requirements over time
---

> **Early Access**
>
> Policy Evaluation is an [Early Access](/release-lifecycle/#early-access-ea)
> feature of Docker Scout.
{ .restricted }

In software supply chain management, maintaining the security and reliability
of artifacts is a top priority. Policy Evaluation in Docker Scout introduces a
layer of control, on top of existing analysis capabilities. It lets you define
supply chain rules for your artifacts, and helps you track how your artifacts
perform, relative to your rules and thresholds, over time.

Learn how you can use Policy Evaluation to ensure that your artifacts align
with established best practices.

## How it works

When you activate Docker Scout for a repository, images that you push are
[automatically analyzed](../image-analysis.md). The analysis gives you insights
about the composition of your images, including what packages they contain and
what vulnerabilities they're exposed to. Policy Evaluation builds on top of the
image analysis feature, interpreting the analysis results against the rules
defined by policies.

A policy defines one or more criteria that your artifacts should fulfill. For
example, one of the default policies in Docker Scout is the **Critical
vulnerabilities** policy, which proclaims that your artifacts must not contain
any critical vulnerabilities. If an artifact contains one or more
vulnerabilities with a critical severity, that artifact fails the evaluation.

In Docker Scout, policies are designed to help you ratchet forward your
security and supply chain stature. Where other tools focus on providing a pass
or fail status, Docker Scout policies visualizes how small, incremental changes
affect policy status, even when your artifacts don't meet the policy
requirements (yet). By tracking how the fail gap changes over time, you more
easily see whether your artifact is improving or deteriorating relative to
policy.

Policies don't necessarily have to be related to application security and
vulnerabilities. You can use policies to measure and track other aspects of
supply chain management as well, such as base image dependencies and
open-source licenses.

## Default policies

Docker Scout ships the following out-of-the-box policies:

- [Fixable critical and high vulnerabilities](#fixable-critical-and-high-vulnerabilities)
- [Critical vulnerabilities](#critical-vulnerabilities)
- [Copyleft licenses](#copyleft-licenses)
- [Outdated base images](#outdated-base-images)
- [High-profile vulnerabilities](#high-profile-vulnerabilities)
- [Supply chain attestations](#supply-chain-attestations)

These policies are turned on by default for Scout-enabled repositories. There's
currently no way to turn off or configure these policies.

### Fixable critical and high vulnerabilities

This policy requires that your artifacts aren't exposed to known
vulnerabilities with a critical or high severity, and where there's a fix
version available. Essentially, this means that there's an easy fix that you
can deploy for images that fail this policy: upgrade the vulnerable package to
a version containing a fix for the vulnerability.

This policy only flags vulnerabilities that were published more than 30
days ago, with the rationale that newly discovered vulnerabilities
shouldn't cause your evaluations to fail until you've had a chance to
address them.

This policy is unfulfilled if an artifact is affected by one or more critical-
or high-severity vulnerability, where a fix version is available.

### Critical vulnerabilities

This policy requires that your artifacts contain no known critical
vulnerabilities. The policy is unfulfilled if your artifact contains one or
more critical vulnerabilities.

This policy flags all critical vulnerabilities, whether or not there's a fix
version available.

### Copyleft licenses

This policy requires that your artifacts don't contain packages distributed
under an AGPLv3 or GPLv3 license. These licenses are protective
[copyleft](https://en.wikipedia.org/wiki/Copyleft), and may be unsuitable for
use in your software because of the restrictions they enforce.

This policy is unfulfilled if your artifacts contain one or more packages with
a violating license.

### Outdated base images

This policy requires that the base images you use are up-to-date.

It's unfulfilled when the tag you used to build your image points to a
different digest than what you're using. If there's a mismatch in digests, that
means the base image you're using is out of date.

#### No base image data

There are cases when it's not possible to determine whether or not the base
image is up-to-date. In such cases, the **Outdated base images** policy
gets flagged as having **No data**.

This occurs when:

- Docker Scout doesn't know what base image tag you used
- The base image version you used has multiple tags, but not all tags are out
  of date

To make sure that Docker Scout always knows about your base image, you can
attach [provenance attestations](../../build/attestations/slsa-provenance.md)
at build-time. Docker Scout uses provenance attestations to find out the base
image version.

### High-profile vulnerabilities

This policy requires that your artifacts don't contain vulnerabilities from
Docker Scout’s curated list. This list is kept up-to-date with newly disclosed
vulnerabilities that are widely recognized to be risky.

The list includes the following vulnerabilities:

- [CVE-2014-0160 (OpenSSL Heartbleed)](https://scout.docker.com/v/CVE-2014-0160)
- [CVE-2021-44228 (Log4Shell)](https://scout.docker.com/v/CVE-2021-44228)
- [CVE-2023-38545 (cURL SOCKS5 heap buffer overflow)](https://scout.docker.com/v/CVE-2023-38545)
- [CVE-2023-44487 (HTTP/2 Rapid Reset)](https://scout.docker.com/v/CVE-2023-44487)

### Supply chain attestations

The Supply chain attestations policy requires that your artifacts have
[SBOM](../../build/attestations/sbom.md) and
[provenance](../../build/attestations/slsa-provenance.md) attestations.

This policy is unfulfilled if an artifact lacks either an SBOM attestation or a
provenance attestation, or if the provenance attestation lacks information
about the Git repository and base images being used. To ensure compliance,
update your build command to attach these attestations at build-time:

```console
$ docker buildx build --provenance=true --sbom=true -t <IMAGE> --push .
```

BuildKit automatically detects the Git repository and base images when this
information is available in the build context. For more information about
building with attestations, see
[Attestations](../../build/attestations/_index.md).
