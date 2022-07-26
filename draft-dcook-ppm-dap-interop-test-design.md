---
title: "DAP Interoperation Test Design"
category: info

docname: draft-dcook-ppm-dap-interop-test-design-latest
submissiontype: IETF
ipr: trust200902
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
  DAP-PPM:
    title: "Distributed Aggregation Protocol for Privacy Preserving Measurement"
    date: 2022-07-11
    target: "https://datatracker.ietf.org/doc/html/draft-ietf-ppm-dap-01"
    author:
      - ins: T. Geoghegan
      - ins: C. Patton
      - ins: E. Rescorla
      - ins: C. A. Wood

informative:
  SI2020:
    title: "Automating QUIC Interoperability Testing"
    date: 2020-08-10
    target: "https://research.protocol.ai/publications/automating-quic-interoperability-testing/seemann2020.pdf"
    author:
      - ins: M. Seemann
      - ins: J. Iyengar


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
