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
  Janus:
    title: Experimental implementation of the DAP-PPM specification
    date: 2022-08-25
    target: https://github.com/divviup/janus


--- abstract

This document defines a common test interface for implementations of the Distributed Aggregation Protocol for Privacy Preserving Measurement (DAP-PPM) and describes how this test interface can be used to perform interoperation testing between the implementations. Tests are orchestrated with containers, and new test-only APIs are introduced to provision DAP-PPM tasks and initiate processing.


--- middle

# Introduction

This document defines a common test interface for implementations of the Distributed Aggregation Protocol for Privacy Preserving Measurement [DAP-PPM]. This test interface facilitates interoperation tests between different participating DAP-PPM implementations. As DAP-PPM has four distinct protocol roles, (client, leader aggregator, helper aggregator, and collector) manual interoperation testing between all combinations of even a small number of DAP-PPM implementations could be taxing. The goal of this document's common test interface is to enable automation of these interoperation tests, so that different participating implementations can be exchanged for each other, and the same test suite can be re-run on different combinations of implementations. Simplifying interoperation testing will aid in identifying errors in implementations, identifying ambiguities in the protocol specification, and reducing regressions in implementations.

Taking inspiration from QuicInteropRunner [SI2020], each participating implementation provides one or more container images adhering to a common interface. A test runner will start one container for each protocol participant, configure networking between the containers, and send various HTTP API requests. As part of this common testing interface, the HTTP servers in the containers will support some new test-only HTTP APIs, which will allow the test runner to provision shared task parameters and secrets, as well as trigger the start of different sub-protocols.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Container Interface

Each participating DAP implementation may provide one or more container images, one for each protocol role it implements. (client, leader, helper, and collector) A list of available container images will be maintained for each role. Implementations may want to submit a single aggregator image in both the leader list and helper list. The test runner will fetch each container using the given repository, image name, and tag.

When the container’s entry point executable is run, it SHALL start up an HTTP server listening on port 8080. In all cases, the container will serve the endpoints described in {{test-api}} (particularly, the subsection appropriate to its protocol role). In the case of a helper or leader container, it SHALL also serve the endpoints specified by [DAP-PPM] at some relative path & port (which MAY be the same port 8080 as used to serve the {{test-api}}). The container should run indefinitely, and the test runner will terminate the container on completion of the test case. (While DAP-PPM requires HTTPS connections, only using HTTP between containers simplifies test setup. Putting TLS client/server interop out-of-scope for these tests is acceptable, as it’s not of interest.)

Log output SHOULD be captured into the directory “/logs” inside the container. This will be copied out to the host for inspection on completion of the test case.

No environment variables or volume mounts will be provided to the containers.


# Interoperation Test API {#test-api}

Each container will have an HTTP server listening on port 8080 for commands from the test runner. All requests MUST use the HTTP method POST. Requests and responses for each endpoint listed below SHALL be encoded JSON objects {{!RFC8729}}, with media type `application/json`. All binary blobs (i.e. task IDs, HPKE configurations, and verification keys) SHALL be encoded as strings with base64url {{!RFC4648}}, inside the JSON objects. Certain integer values will be as strings in base 10 instead of as numbers, where noted, if JSON numbers cannot fully represent the range of valid values.

Each of these test APIs should return a status code of 200 OK if the command was received, recognized, and parsed successfully, regardless of whether any underlying DAP-PPM request succeeded or failed. The DAP-level success or failure will be included in the test API response body. If a request is made to an endpoint starting with “/internal/test/”, but not listed here, a status code of 404 Not Found SHOULD be returned, to simplify the introduction of new test APIs.


## Common Structures

In multiple APIs defined below, the test runner will send the name of a VDAF, along with the parameters necessary to fully specify the VDAF. These will be stored in a nested object, with the following attributes (new `type` values and new keys will be added as new VDAFs are defined).

|Key|Value|
|`type`|One of `"Prio3Aes128Count"`, `"Prio3Aes128Sum"`, or `"Prio3Aes128Histogram"`|
|`bits` (only present if `type` is `"Prio3Aes128Sum"`)|The bit width of the integers being summed, (as a number) used to parameterize the Prio3Aes128Sum VDAF.|
|`buckets` (only present if `type` is `"Prio3Aes128Histogram"`)|An array of histogram bucket boundaries, (encoded in base 10 as strings) used to parameterize the Prio3Aes128Histogram VDAF.|
{: title="VDAF JSON object structure" #vdaf-object}


## Client

### `/internal/test/ready` {#client-ready}

The test runner will POST an empty object (i.e. `{}`) to this endpoint to check if the client container is ready to serve requests. If it is ready, it MUST return a status code of 200 OK.


### `/internal/test/upload` {#upload}

Upon receipt of this command, the client container will construct a DAP-PPM report with the given configuration and measurement, and submit it. The client container will send its response to the test runner once report submission has either succeeded or permanently failed.

|Key|Value|
|`taskId`|A base64url-encoded DAP-PPM `TaskId`.|
|`leader`|The leader's endpoint URL.|
|`helper`|The helper's endpoint URL.|
|`vdaf`|An object, with the layout given in {vdaf-object}. This determines the VDAF to be used when constructing a report.|
|`measurement`|If the VDAF's `type` is `"Prio3Aes128Count"`: 0 or 1. If the VDAF's `type` is `"Prio3Aes128Sum"`: a string (representing an integer in base 10). If the VDAF's `type` is `"Prio3Aes128Histogram"`: a string (representing an integer in base 10).|
|`nonceTime` (optional)|If present, this provides a substitute time value that should be used when constructing the report. If not present, the current system time should be used, as per normal. The time is represented as a number, with a value of the number of seconds since the UNIX epoch.|
|`minBatchDuration`|A number, providing the minimum number of seconds that can be in a batch’s interval. The batch interval will always be a multiple of this value.|
{: title="Request JSON object structure"}

|Key|Value|
|`status`|`"success"` if the report was submitted to the leader successfully, or `"error"` otherwise.|
|`error` (optional)|An optional error message, to assist in troubleshooting. This will be included in the test runner logs.|
{: title="Response JSON object structure"}


## Aggregator (Leader or Helper)

### `/internal/test/ready` {#aggregator-ready}

The test runner will POST an empty object (i.e. `{}`) to this endpoint to check if the aggregator container is ready to serve requests. If it is ready, it MUST return a status code of 200 OK.


### `/internal/test/endpoint_for_task` {#endpoint-for-task}

Request the base URL for DAP-PPM endpoints for a new task. This API will be invoked immediately before `/internal/test/add_task` (see {{aggregator-add-task}}), to determine the endpoint URLs of the aggregators. If the aggregator uses a common set of DAP-PPM endpoints for all tasks, it could always return the same value, such as the relative URL `/`. Alternately, implementations may wish to generate new endpoints for each task, derive the endpoint based on the `TaskId`, etc.

The test runner will provide the hostname at which the aggregator is externally reachable. If the aggregator returns a relative URL, the test runner will combine it with the hostname into an absolute URL, assuming that the port is 8080. Otherwise, the aggregator can incorporate the hostname into an absolute URL and return that.

|Key|Value|
|`taskId`|A base64url-encoded DAP-PPM `TaskId`|
|`aggregatorId`|0 if this aggregator is the leader, or 1 if this aggregator is the helper.|
|`hostname`|This aggregator's hostname in the interoperation test environment. This may optionally be used in constructing the endpoint URL as an absolute URL.|
{: title="Request JSON object structure"}

|Key|Value|
|`status`|`"success"` if the endpoint was successfully selected or set up, or `"error"` otherwise.|
|`error` (optional)|An optional error message, to assist in troubleshooting. This will be included in the test runner logs.|
|`endpoint`|A relative or absolute URL, specifying the DAP-PPM aggregator endpoint that should be used for this task. If the test runner receives a relative URL, it will transform it into an absolute URL before performing the next phase of task setup.|
{: title="Response JSON object structure"}


### `/internal/test/add_task` {#aggregator-add-task}

Register a task with the aggregator, with the given configuration and secrets.

The HPKE keypair generated for this task should use the mandatory-to-implement algorithms in section 6 of [DAP-PPM], for broad compatibility.

|Key|Value|
|`taskId`|A base64url-encoded DAP-PPM `TaskId`.|
|`leader`|The leader's endpoint URL. The test runner will ensure this is an absolute URL.|
|`helper`|The helper's endpoint URL. The test runner will ensure this is an absolute URL.|
|`vdaf`|An object, with the layout given in {vdaf-object}. This determines the task's VDAF.|
|`leaderAuthenticationToken`|The authentication bearer token that is shared with the other aggregator, as a string. This string must be safe for use as an HTTP header value.|
|`collectorAuthenticationToken` (only present if `aggregatorId` is 0)|The authentication bearer token that is shared between the leader and collector, as a string. This string must be safe for use as an HTTP header value.|
|`aggregatorId`|0 if this aggregator is the leader, or 1 if this aggregator is the helper.|
|`verifyKey`|The verification key shared by the two aggregators, encoded with base64url.|
|`maxBatchLifetime`|A number, providing the maximum number of times any report can be included in a collect request.|
|`minBatchSize`|A number, providing the minimum number of reports that must be in a batch for it to be collected.|
|`minBatchDuration`|A number, providing the minimum number of seconds that can be in a batch’s interval. The batch interval will always be a multiple of this value.|
|`collectorHpkeConfig`|The collector’s HPKE configuration, encoded in base64url, for encryption of aggregate shares.|
{: title="Request JSON object structure"}

|Key|Value|
|`status`|`"success"` if the task was successfully set up, or `"error"` otherwise. (for example, if the VDAF was not supported)|
|`error` (optional)|An optional error message, to assist in troubleshooting. This will be included in the test runner logs.|
{: title="Response JSON object structure"}


## Collector

### `/internal/test/ready` {#collector-ready}

The test runner will POST an empty object (i.e. `{}`) to this endpoint to check if the collector container is ready to serve requests. If it is ready, it MUST return a status code of 200 OK.


### `/internal/test/add_task` {#collector-add-task}

Register a task with the collector, with the given configuration. Returns the collector’s HPKE configuration for this task.

|Key|Value|
|`taskId`|A base64url-encoded DAP-PPM `TaskId`.|
|`leader`|The leader's endpoint URL.|
|`vdaf`|An object, with the layout given in {vdaf-object}. This determines the task's VDAF.|
|`collectorAuthenticationToken`|The authentication bearer token that is shared between the leader and collector, as a string. This string must be safe for use as an HTTP header value.|
{: title="Request JSON object structure"}

|Key|Value|
|`status`|`"success"` if the task was successfully set up, or `"error"` otherwise. (for example, if the VDAF was not supported)|
|`error` (optional)|An optional error message, to assist in troubleshooting. This will be included in the test runner logs.|
|`collectorHpkeConfig` (if successful)|The collector's HPKE configuration, encoded in base64url, for encryption of aggregate shares.|
{: title="Response JSON object structure"}


### `/internal/test/collect_start` {#collect-start}

Send a collect request to the leader with the provided parameters, and return a handle to the test runner identifying this collect request. The test runner will provide this handle to the collector in subsequent `/internal/test/collect_poll` requests (see {{collect-poll}}).

|Key|Value|
|`taskId`|A base64url-encoded DAP-PPM `TaskId`.|
|`aggParam`|A base64url-encoded aggregation parameter.|
|`batchIntervalStart`|The start of the batch interval, represented as a number equal to the number of seconds since the UNIX epoch.|
|`batchIntervalDuration`|The duration of the batch interval in seconds, as a number.|
{: title="Request JSON object structure"}

|Key|Value|
|`status`|`"success"` if the collect request succeeded, or `"error"` otherwise.|
|`error` (optional)|An optional error message, to assist in troubleshooting. This will be included in the test runner logs.|
|`handle` (if successful)|A handle produced by the collector to refer to this collect request. This must be a string.|
{: title="Response JSON object structure"}


### `/internal/test/collect_poll` {#collect-poll}

Upon receiving this command, the collector will poll the leader’s collect URL for the collect job associated with the provided handle, and provide the status and result to the test runner.

|Key|Value|
|`handle`|The handle for a collect request from a previous invocation of `/internal/test/collect_start`. (see {{collect-start}})|
{: title="Request JSON object structure"}

|Key|Value|
|`status`|Either `"complete"` if the result was returned, `"in progress"` if the result was not yet ready, or `"error"` if an error occurred.|
|`error` (optional)|An optional error message, to assist in troubleshooting. This will be included in the test runner logs.|
|`result` (if complete)|The result of the aggregation. If the VDAF is of type Prio3Aes128Count, this will be a number. If the VDAF is of type Prio3Aes128Sum, this will be a string, representing an integer in base 10. If the VDAF is of type Prio3Aes128Histogram, this will be an array of numbers.|
{: title="Response JSON object structure"}


### Heavy Hitters

Once Poplar1 reaches a future draft of [DAP-PPM], additional test APIs for collector containers should be introduced to perform an entire Heavy Hitters computation on a given Poplar1 task and collection interval, encompassing multiple collect flows automatically initiated by the collector.


## Test Cases

Test cases could be written to cover the following scenarios.

* Test successful aggregations with each VDAF.
* Test an aggregation over a few hundred or thousand reports, to exercise the aggregators' division of reports into aggregation jobs.
* Test that uploading a report with a time far in the future is rejected.
* Confirm that leaders and helpers reject requests with respective authentication tokens that are incorrect.
* Test enforcement of `max_batch_lifetime` by making overlapping collect requests.
* Perform an entire aggregation and collect flow, attempt to upload a late report that falls into the same collect interval, and test that performing the collect request a second time yields the same result.
* Attempt to upload a canned report from the test runner more than once, and confirm that anti-replay measures were effective by inspecting the aggregation result.


## Other Test Considerations

All test cases should automatically fail after a generous timeout.

It is the responsibility of the test runner to wait for all containers to start up and listen on their ports before sending any commands.

Aggregator URLs will be constructed by the test runner with hostnames that resolve to the respective containers within the container network.

Once a future [DAP-PPM] draft solves the issue of retries in the aggregate flow, a reverse proxy could be introduced in front of each aggregator to inject failures when sending requests or responses, to test the protocol's resilience. (It is known such a test would fail based on the current protocol.)


## Test Runner Operation

The following sequence outlines how the test runner will use the above APIs on port 8080 of each container to perform a typical integration test, executing a successful aggregation.

1. Create and start containers.
1. Set up networking between containers.
1. Try sending `/internal/test/ready` requests to each container, and retry until they succeed.
1. Generate a random `TaskId`, random authentication tokens, and a VDAF verification key.
1. Send a `/internal/test/endpoint_for_task` request ({{endpoint-for-task}}) to the leader.
1. Send a `/internal/test/endpoint_for_task` request to the helper.
1. Construct aggregator URLs using the above responses.
1. Send a `/internal/test/add_task` request ({{collector-add-task}}) to the collector. (the collector generates an HPKE key pair as a side-effect)
1. Send a `/internal/test/add_task` request ({{aggregator-add-task}}) to the leader.
1. Send a `/internal/test/add_task` request ({{aggregator-add-task}}) to the helper.
1. Send one or more `/internal/test/upload` requests ({{upload}}) to the client.
1. Send a `/internal/test/collect_start` request ({{collect-start}}) to the collector. (this provides a handle for use in the next step)
1. Send `/internal/test/collect_poll` requests ({{collect-poll}}) to the collector, polling until it is completed. (the collector will provide the calculated aggregate result)
1. Stop containers.
1. Copy logs out of each container.
1. Delete containers, and clean up container networking resources.


# Implementation Status

[Janus] currently implements a version of this test interface.

Additional DAP-PPM implementations would be warmly welcomed.


# Security Considerations

Any DAP-PPM implementation that adopts this testing interface should ensure that the test-only APIs described herein are only present in software used for testing purposes, and not in production systems.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

Thanks to Brandon Pitman, Christopher Patton, and Tim Geoghegan for feedback and contributions.
