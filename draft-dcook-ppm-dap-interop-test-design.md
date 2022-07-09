---
title: "DAP Interoperation Test Design"
category: info

docname: draft-dcook-ppm-dap-interop-test-design-latest
submissiontype: IETF
number:
date:
consensus: false
v: 3
area: "Security"
workgroup: "Privacy Preserving Measurement"
venue:
  group: "Privacy Preserving Measurement"
  type: "Working Group"
  mail: "ppm@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ppm/"
  github: "divergentdave/draft-dcook-ppm-dap-interop-test-design"
  latest: "https://divergentdave.github.io/draft-dcook-ppm-dap-interop-test-design/draft-dcook-ppm-dap-interop-test-design.html"

author:
 -
    fullname: David Cook
    organization: ISRG
    email: "dcook@letsencrypt.org"

normative:

informative:


--- abstract

This document defines a common test interface for implementations of the Distributed Aggregation Protocol for Privacy Preserving Measurement (DAP-PPM) and describes how this test interface can be used to perform interoperation testing between the implementations. Tests are orchestrated with containers, and new test-only APIs are introduced to provision DAP-PPM tasks and initiate processing.


--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

Any DAP-PPM implementation that adopts this testing interface should ensure that the test-only APIs described herein are only present in software used for testing purposes, and not in production systems.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

Thanks to Brandon Pitman and Christopher Patton for feedback and contributions.
