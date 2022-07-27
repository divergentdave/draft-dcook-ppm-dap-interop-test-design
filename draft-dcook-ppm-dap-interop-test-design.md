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

This document defines a common test interface for implementations of the Distributed Aggregation Protocol for Privacy Preserving Measurement [DAP-PPM]. This test interface facilitates interoperation tests between different participating DAP-PPM implementations. As DAP-PPM has four distinct protocol roles, (client, leader aggregator, helper aggregator, and collector) manual interoperation testing between all combinations of even a small number of DAP-PPM implementations could be taxing. The goal of this document's common test interface is to enable automation of these interoperation tests, so that different participating implementations can be exchanged for each other, and the same test suite can be re-run on different combinations of implementations. Simplifying interoperation testing will aid in identifying errors in implementations, identifying ambiguities in the protocol specification, and reducing regressions in implementations.

Taking inspiration from QuicInteropRunner [SI2020], each participating implementation provides one or more container images adhering to a common interface. A test runner will start one container for each protocol participant, configure networking between the containers, and send various HTTP API requests. As part of this common testing interface, the HTTP servers in the containers will support some new test-only HTTP APIs, which will allow the test runner to provision shared task parameters and secrets, as well as trigger the start of different sub-protocols.


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
