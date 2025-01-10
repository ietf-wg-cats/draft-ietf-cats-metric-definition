---

title: "CATS Metrics Definition"
abbrev: "CATS Metrics"
category: info

docname: draft-ietf-cats-metric-definition-latest
submissiontype: IETF
number:
date:
consensus:
v: 3
area: "Routing"
workgroup: "Computing-Aware Traffic Steering"
keyword: CATS, metric

author:
 -
    ins: Y. Kehan
    fullname: Kehan Yao
    organization: China Mobile
    email: yaokehan@chinamobile.com
    country: China
 -
    ins: H. Shi
    fullname: Hang Shi
    organization: Huawei Technologies
    email: shihang9@huawei.com
    country: China
 -
    ins: C. Li
    fullname: Cheng Li
    organization: Huawei Technologies
    email: c.l@huawei.com
    country: China
 -
    ins: L.M. Contreras
    fullname: L. M. Contreras
    organization: Telefonica
    email: luismiguel.contrerasmurillo@telefonica.com
 -
    ins: J. Ros-Giralt
    fullname: Jordi Ros-Giralt
    organization: Qualcomm Europe, Inc.
    email: jros@qti.qualcomm.com


normative:

informative:
  performance-metrics:
    title: performance-metrics
    author:
    org:
    date:
    target: https://www.iana.org/assignments/performance-metrics/performance-metrics.xhtml

  DMTF:
    title: DMTF
    author:
    org:
    date:
    target: https://www.dmtf.org/

--- abstract

This document defines a set of computing metrics used for Computing-Aware Traffic Steering (CATS).

--- middle

# Introduction

Service providers are deploying computing capabilities across the network for hosting applications such as distributed AI workloads, AR/VR and driverless vehicles, among others. In these deployments, multiple service instances are replicated across various sites to ensure sufficient capacity for maintaining the required Quality of Experience (QoE) expected by the application.  To support the selection of these instances, a framework called Computing-Aware Traffic Steering (CATS) is introduced in {{!I-D.ietf-cats-framework}}.

CATS is a traffic engineering approach that optimizes the steering of traffic to a given service instance by considering the dynamic nature of computing and network resources. To achieve this, CATS components (C-PS, C-Forwarders, etc.) require performance metrics for both communication and compute resources. Since these resources are deployed by multiple providers, standardized metrics are essential to ensure interoperability and enable precise traffic steering decisions, thereby optimizing resource utilization and enhancing overall system performance.

Various considerations for metric definition are proposed in {{?I-D.du-cats-computing-modeling-description}}, which are useful for defining computing metrics. This document categorizes the relevant compute and network metrics for CATS into three levels based on their complexity and granularity, following the considerations outlined in {{?I-D.du-cats-computing-modeling-description}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the following terms defined in {{!I-D.ietf-cats-framework}}:

- Computing-Aware Traffic Steering (CATS)

- Service

- Service contact instance

# Definition of Metrics

Introducing a definition of metrics requires balancing the following trade-off: if the metrics are too fine-grained, they become unscalable due to the excessive number of metrics that must be communicated through the metrics distribution protocol. (See {{?I-D.rcr-opsawg-operational-compute-metrics}} for a discussion of metrics distribution protocols.) Conversely, if the metrics are too coarse-grained, they may lack the necessary information to make informed decisions. To ensure scalability while providing sufficient detail for effective decision-making, we propose a definition of metrics that incorporates three levels of abstraction:

- **Level 0 (L0): Raw metrics.** These metrics are presented without abstraction, with each metric using its own unit and format as defined by the underlying resource.

- **Level 1 (L1): Normalized metrics in categories.** These metrics are derived by aggregating L0 metrics into multiple categories, such as network, computing, and storage. Each category is summarized with a single L1 metric by normalizing it into a value within a defined range of scores.

- **Level 2 (L2): Fully normalized metric.** These metrics are derived by aggregating lower level metrics (L0 or L1) into a single L2 metric, which is then normalized into a value within a defined range of scores.

## Level 0: Raw Metrics

Level 0 metrics encompass detailed, raw metrics, including but not limit to:

- CPU: Base Frequency, boosted frequency, number of cores, core utilization, memory bandwidth, memory size, memory utilization, power consumption.
- GPU: Frequency, number of render units, memory bandwidth, memory size, memory utilization, core utilization, power consumption.
- NPU: Computing power, utilization, power consumption.
- Network: Bandwidth, capacity, throughput, transmit bytes, receive bytes, host bus utilization.
- Storage: Available space, read speed, write speed.
- Delay: Time taken to process a request.

L0 metrics can be encoded into an Application Programming Interface (API), such as a RESTful API, and can be solution-specific. Different resources can have their own metrics, each conveying unique information about their status. These metrics can generally have units, such as bits per second (bps) or floating point instructions per second (flops).

Regarding network-related information, the IPPM WG has defined various types of metrics in {{performance-metrics}}. Additionally, in {{?RFC9439}}, the ALTO WG has introduced an extended set of metrics related to packet performance and throughput/bandwidth. For compute metrics, {{?I-D.rcr-opsawg-operational-compute-metrics}} lists a set of cloud resource metrics.

## Level 1: Normalized Metrics in Categories

L1 metrics are organized into distinct categories, such as computing, networking, storage, and delay. Each L0 metric is classified into one of these categories. Within each category, a single L1 metric is computed using an *aggregation function* and normalized to a unitless score that represents the performance of the underlying resources according to that category. Potential categories include:

- **Computing:** A normalized value derived from computing-related L0 metrics, such as CPU, GPU, and NPU metrics.

- **Networking:** A normalized value derived from network-related L0 metrics.

- **Storage:** A normalized value derived from storage-related L0 metrics.

- **Delay:** A normalized value derived from computing, networking, and storage metrics, reflecting the end-to-end processing delay of a request.

Editor note: detailed categories can be updated according to the CATS WG discussion.

The L0 metrics, such as those defined in {{performance-metrics}}, {{?RFC9439}}, and {{?I-D.rcr-opsawg-operational-compute-metrics}}, can be categorized into the aforementioned categories. Each category will employ its own aggregation function (e.g., weighted summary) to generate the normalized value. This approach allows the protocol to focus solely on the metric categories and their normalized values, thereby avoiding the need to process solution-specific detailed metrics.

## Level 2: Fully Normalized Metric.

The L2 metric is a single score value derived from the lower level metrics (L0 or L1) using an aggregation function. Different implementations may employ different aggregation functions to characterize the overall performance of the underlying compute and communication resources. The definition of the L2 metric simplifies the complexity of collecting and distributing numerous lower-level metrics by consolidating them into a single, unified score.

TODO: Some implementations may support configuration of Ingress CATS-Forwarders with the metric normalizing method so that it can decode the affection from the L1 or L0 metrics.

Figure 1 shows the logic of metrics in Level 0, Level 1, and Level 2.

<!-- Comment JRG: I would like to suggest some changes to this diagram. Seems like M1 is repeated at level 0 and level 1, same for M2, M3, so this can be confusing. Also the words Raw are repeated for all boxes, which seems redudant. Same for the word Category. Also the edges seem unidirectional, from bottom to top, so we can add an arrow to reflect the direction. I am suggesting the following adjustments: -->


~~~
                                       +--------+
                            L2 Metric: |   M2   |
                                       +---^----+
                                           |
                         +-----------------+---------------+
                         |                 |               |
                     +---+----+        +---+----+      +---+----+
         L1 Metrics: |  M1-1  |        |  M1-2  |      |  M1-3  | (...)
                     +---^----+        +---^----+      +----^---+
                         |                 |                |
              +--------+-+-------+       +-+-------+        |
              |        |         |       |         |        |
           +--+---+ +--+---+ +---+--+ +--+---+ +---+--+  +--+---+
L0 Metrics:| M0-1 | | M0-2 | | M0-3 | | M0-4 | | M0-5 |  | M0-6 | (...)
           +------+ +------+ +------+ +------+ +------+  +------+

~~~
{: #fig-cats-metric title="Logic of CATS Metrics in levels"}



# Representation of Metrics

This section includes the detailed representation of metrics. [RFC9439] gives a good way to show the representation of some network metrics which is used for network capabilities exposure to applications. This document further describes the representation of CATS metrics.

Basically, in each metric level and for each metric, there will be some common fields for representation, including metric type, unit, and precision.  Metric type is a label for network devices to recognize what the metric is. "unit" and "precision" are usually associated with the metric.  How many bits a metric occupies in protocols is also required.

Beyond these basic representations, the source of the metrics must also be declared, since there are multiple levels of metrics and their sources are different.  As defined in [RFC9439], there are three cost-sources, nominal, sla, and estimation.  This document further divide the estimation type into three sub-types, direct measurement, aggregation, and normalization, since different levels of metrics require different sources to acquire CATS metrics.  Directly measured metrics have physical meanings and units without any processing. Aggregated metrics can be either physically meaningful or not, and they maintain their meanings compared to the directly measured metrics.  Normalized metrics can have physical meanings or not, but they do not have units, and they are just numbers that used for routing decision making.

To be more fine-grained, this document refers to the definition of [RFC9439]  on the metrics statistics.

## Level 0 Metric Representation

Raw metrics have exact physical meanings and units.  They are directly measured from the underlying computing resources providers.
   Lots of definition on this level of metrics have been defined in IT industry and other standardizations[DMTF], and this document only show some examples for different categories of metrics for reference.

### Compute Raw Metrics

The metric type of compute resources are named as “compute_type:
   CPU” or “compute_type: GPU”. Their frequency unit is GHZ, the compute
   capabilities unit is FLOPS.  Format should support integer and FP8.
   It will occupy 4 octets. Example:

~~~
Basic fields:
      Metric type: “compute type_CPU”
      Format: integer, FP8
      Bits occupation: 4 octets
Special fields:
      Frequency unit: GHZ
      Compute capabilities unit: FLOPs
Source:
      Direct measurement
Statistics:
      Mean
~~~
{: #fig-compute-raw-metric title="An Example for Compute Raw Metrics"}



### Storage Raw Metrics

   The metric type of storage resources like SSD are named as
   “storage_type: SSD”. The storage space unit is megaBytes(MBs).
   Format is integer.  It will occupy 2 octets.  The unit of read or
   write speed is denoted as MB per second. Example:

~~~
Basic fields:
      Metric type: “storage type_SSD”
      Format: integer
      Unit: GB
      Bits occupation: 2 octets
Source:
      nominal
Statistics:
      cur
~~~
{: #fig-storage-raw-metric title="An Example for Storage Raw Metrics"}

###  Network Raw Metrics

   The metric type of network resources like bandwidth are named as
   "network_type: Bandwidth”. The unit is gigabits per second(Gb/s).
   Format is integer.  It will occupy 2 octets.  The unit of TXBytes and
   RXBytes is denoted as MB per second. Example:

~~~
Basic fields:
      Metric type: “network type_Bandwidth”
      Format: integer
      Unit: Gb/s
      Bits occupation: 2 octets
Source:
      nominal
Statistics:
      cur
~~~
{: #fig-network-raw-metric title="An Example for Network Raw Metrics"}


###  Delay Raw Metrics

   Delay is a kind of synthesized metric which is influenced by
   computing, storage access, and network transmission.  It is named as
   “delay_raw”. Format should support integer and FP8.  Its unit is
   microsecond.  It will occupy 4 octets. Example:

~~~
Basic fields:
      Metric type: “delay_raw”
      Format: integer, FP8
      Unit: Microsecond(us)
      Bits occupation: 4 octets
Source:
      aggregation
Statistics:
      max
~~~
{: #fig-delay-raw-metric title="An Example for Delay Raw Metrics"}

### Considerations on the Sources of Metrics and the Statistics

The sources of L0 metrics can be nominal, directly measured, or aggregated.  Nominal L0 metrics are provided initially by resource
   providers.  Dynamic L0 metrics are measured and updated during service stage.  L0 metrics also support aggregation, in case that
   there are multiple service instances.

The statistics of L0 metrics will follow the definition of Section 3.2 of [RFC9439].

## Level 1 Metric Representation

Normalized metrics in categories have physical meanings but they do not have unit.  They are numbers after some ways of abstraction, but they can represent their type, in case that in some use cases, some specific types of metrics require more attention.

### Normalized Compute Metrics

The metric type of normalized compute metrics is “compute_norm”,
   and its format is integer.  It has no unit.  It will occupy an octet. Example:

~~~
Basic fields:
      Metric type: “compute_norm”
      Format: integer
      Bits occupation: an octet
      Score: 1
Source:
      normalization
~~~
{: #fig-normalized-compute-metric title="An Example for Normalized Compute Metrics"}

### Normalized Storage Metrics

The metric type of normalized compute metrics is “storage_norm”, and its format is integer.  It has no unit.  It will occupy a octet. Example:

~~~
Basic fields:
      Metric type: “storage_norm”
      Format: integer
      Bits occupation: an octet
      Score: 1
Source:
      normalization
~~~
{: #fig-normalized-storage-metric title="An Example for Normalized Storage Metrics"}

### Normalized Network Metrics

The metric type of normalized compute metrics is “network_norm”, and its format is integer.  It has no unit.  It will occupy a octet. Example:

~~~
Basic fields:
      Metric type: “network_norm”
      Format: integer
      Bits occupation: an octet
      Score: 1
Source:
      normalization
~~~
{: #fig-normalized-network-metric title="An Example for Normalized Network Metrics"}

### Normalized Delay

The metric type of normalized compute metrics is “delay_norm”, and its format is integer.  It has no unit.  It will occupy a octet. Example:

~~~
Basic fields:
      Metric type: “delay_norm”
      Format: integer
      Bits occupation: an octet
      Score: 1
Source:
      normalization
~~~
{: #fig-normalized-metric title="An Example for Normalized Delay Metrics"}

### Considerations on the Sources of Metrics and the Statistics

The sources of L1 metrics is normalized. Based on L0 metrics, service providers design their own algorithms to normalize metrics.  For example, assigning different cost values to each raw metric and do summation.  L1 metric do not need further statistical values.

## Level 2 Metric Representation

A fully normalized metric is a single value which does not have any physical meaning or unit.  Each provider may have its own methods to derive the value, but all providers must follow the definition in this section to represent the fully normalized value.

Metric type is “norm_fi”. The format of the value is non-negative integer.  It has no unit.  It will occupy a octet. Example:

~~~
Basic fields:
      Metric type: “norm_fi”
      Format: non-negative integer
      Bits occupation: an octet
      Score: 1
Source:
      normalization
~~~
{: #fig-level-2-metric title="An Example for Fully Normalized Metric"}

The fully normalized value also supports aggregation when there are multiple service instances providing these fully normalized values. When providing fully normalized values, service instances do not need to do further statistics.

# Comparison of three layers of metric

From L0 to L1 to L2, the computing metric is consolidated. Different level of abstraction can meet the requirements from different services. Table 1 shows the comparison among metric levels.


|  Level   | Encoding Complexity| Extensibility| Stability |Accuracy|
|:--------:+:-------------------+--------------+-----------|--------|
| Level 0  | Complicated        | Bad          | Bad       |Good    |
| Level 1  | Medium             | Medium       | Medium    |Medium  |
| Level 2  | Simple             | Good         | Good      |Medium  |
{: #comparison title="Comparison among Metrics Levels"}


Since Level 0 metrics are raw metrics, therefore, different services may have their own metrics, resulting in hundreds or thousands of metrics in total, this brings huge complexity in protocol encoding and standardization. Therefore, this kind of metrics are always used in customized IT systems case by case. In Level 1 metrics, metrics are categorized into several categories and each category is normalized into a value, therefore they can be encoded into the protocol and standardized. Regarding the Level 2 metrics, all the metrics are normalized into one single metric, it is easier to be encoded in protocol and standardized. Therefore, from the encoding complexity aspect, Level 2 and Level 1 metrics are suggested.

Similarly, when considering extensibility, new services can define their own new L0 metrics, which requires protocol to be extended as needed. Too many metrics type can create a lot of overhead to the protocol resulting in a bad extensibility of the protocol. Level 1 introduce only several metrics categories, which is acceptable for protocol extension. Level 2 metric only need one single metric, so it brings least burden to the protocol. Therefore, from the extensibility aspect, Level 2 and Level 1 metrics are suggested.

Regarding Stability, new Level 0 raw metrics may require new extension in protocol, which brings unstable format for protocol, therefore, this document does not recommend to standardize Level 0 metrics in protocol. Level 1 metrics request only few categories, and Level 2 Metric only introduce one metric to the protocol, so they are preferred from the stability aspect.


In conclusion, for computing-aware traffic steering, it is recommended to use the L2 metric due to its simplicity. If advanced scheduling is needed, L1 metric can be used. L2 metrics are the most comprehensive and dynamic, therefore transferring them to network devices is discouraged due to their high overhead.

Editor notes: this draft can be updated according to the discussion of metric definition in CATS WG.


# Security Considerations

TBD

# IANA Considerations

TBD


--- back
