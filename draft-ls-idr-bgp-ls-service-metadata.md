---
title: "Distribution of Service Metadata in BGP-LS"
abbrev: "Service Metadata in BGP-LS"
category: std

docname: draft-ls-idr-bgp-ls-service-metadata-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Inter-Domain Routing"
keyword:
 - Internet Draft
venue:
  group: "Inter-Domain Routing"
  type: "Working Group"
  mail: "idr@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/idr/"
  github: "VMatrix1900/draft-service-metadata-in-BGP-LS"
  latest: "https://VMatrix1900.github.io/draft-service-metadata-in-BGP-LS/draft-ls-idr-bgp-ls-service-metadata.html"

author:
 -
  ins: C.Li
  name: Cheng Li
  organization: Huawei Technologies
  email: c.l@huawei.com
  role: editor
  city: Beijing
  country: China
 -
  ins: H.Shi
  name: Hang Shi
  organization: Huawei Technologies
  email: shihang9@huawei.com
  role: editor
  city: Beijing
  country: China
 -
  ins: T.He
  name: Tao He
  organization: China Unicom
  email: het21@chinaunicom.cn
  city: Beijing
  country: China
 -
  ins: R.Pang
  name: Ran Pang
  organization: China Unicom
  email: pangran@chinaunicom.cn
  city: Beijing
  country: China
 -
  ins: G.Qian
  name: Guofeng Qian
  organization: Huawei Technologies
  email: qianguofeng@huawei.com
  city: Beijing
  country: China


normative:
  RFC7752:
  RFC9012:
  RFC9085:
  RFC9252:
  I-D.ietf-idr-5g-edge-service-metadata:

informative:

...

--- abstract

In edge computing, a service may be deployed on multiple instances within one or more sites, called edge service. The edge service is associated with an ANYCAST address in IP layer, and the route of it with potential service metatdata will be distributed to the network. The Edge Service Metadata can be used by ingress routers to make path selections not only based on the routing cost but also the running environment of the edge services.

The service route with metadata can be collected by a PCE(Path Compute Element) or an analyzer for calculating the best path to the best site/instance.  This draft describes a mechanism to collect the information of the service routes and related service metadata in BGP-LS.

--- middle

# Introduction {#intro}

Many services deploy their service instances in multiple sites to get better response time and resource utilization. These sites are often geographically distributed to serve the user demand. For some services such as VR/AR and intelligent transportation, the QoE will depend on both the network metrics and the compute metrics. For example, if the nearest site is overloaded due to the demand fluctuation, then steer the user traffic to an another light-loaded sites may improve the QoE.

{{I-D.ietf-idr-5g-edge-service-metadata}} descirbes the BGP extension of distributing service route with network and computing-related metrics. The router connected to the site will receive the service routes and service metadata sent from devices inside the edge site, and then generates the corresponding routes and distributes them to ingress routers. However, the route with service metadata on the router connected to the site can be also collected by a central Controller for calculating the best path to the best site.

This document defines an extension of BGP-LS to carry the service metadata along with the service route. Using the service metadata and the service route, the controller can calculate the best site for the traffic, giving each user the best QoE.


## Terminology

## Requirements Language

{::boilerplate bcp14-tagged}

# BGP-LS Extension for Service in a Site

The goal of the BGP-LS extension is to collect information of the service prefix and metadata of the service, such as network metrics and compute metrics. A service is identified by a prefix, and this information is carried by the existing prefix NLRI TLV. Other information including service metadata is carried by attributes TLVs.


## Prefix NLRI {#Prefix}

A service is identified by a prefix, and the Prefix NLRI defined in the {{RFC7752}} is used to collect the prefix information of the service. The format of the Prefix NLRI is shown in {{fig-Prefix-NLRI}} for better understanding.

~~~

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+
     |  Protocol-ID  |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                           Identifier                          |
     |                            (64 bits)                          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     //              Local Node Descriptors (variable)              //
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     //                Prefix Descriptors (variable)                //
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-Prefix-NLRI title="The IPv4/IPv6 Topology Prefix NLRI Format"}

Specifically, the service prefix is carried by the IP Reachability Information TLV({{fig-IP-Reachability-TLV}}) inside the Prefix Descriptor field. The Prefix Length field contains the length of the prefix in bits. The IP Prefix field contains the most significant octets of the prefix.

~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |              Type             |             Length            |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | Prefix Length | IP Prefix (variable)                         //
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-IP-Reachability-TLV title="IP Reachability Information TLV Format"}


## Attributes

The following three prefix attribute TLVs are used to carry the metadata of a service instance:

1. Metadata Path Attribute TLV carries the computing metric of the service instances such as site preference, capacity index, and load measurement defined in {{I-D.ietf-idr-5g-edge-service-metadata}}.
2. Prefix SID TLV carries a Prefix SID associated with the edge site.
3. Color Attribute TLV carries the service requirement level information of the service


### Metadata Path Attribute TLV {#metadata}

The Metadata Path Attribute TLV is an optional attribute to carry the Edge Service Metadata defined in the {{I-D.ietf-idr-5g-edge-service-metadata}}. It contains multiple sub-TLVs, with each sub-TLV containing a specific metric of the Edge Service Metadata. This document defines a new TLV in BGP-LS, which reuse the name and the format of Metadata Path Attribute TLV.

~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |              Type             |            Length             |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     |              Value (multiple Metadata sub-TLVs)               |
     |                                                               |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-Metadata-Path-Attribute title="Metadata Path Attribute TLV format"}

- Type: identify the Metadata Path Attribute, to be assigned by IANA.
- Length: the total number of octets of the value field.
- Value: contains multiple sub-TLVs.

There are three types of Edge Service Metadata sub-TLVs defined in {{I-D.ietf-idr-5g-edge-service-metadata}}:

1. Site Preference Index indicates the preference to choose the site.
2. Capacity Index indicates the capability of a site. One Edge Site can be at full capacity, reduced capacity, or completely out of service.
3. Load Measurement indicates the load level of the site.

To collect this information, this document defines TLVs reusing the name and format of the TLVs defined in {{I-D.ietf-idr-5g-edge-service-metadata}}.


## Prefix SID Attribute TLV {#prefix-SID}

In some cases, there may be multiple sites connected to one Edge(egress) router through different interfaces. Generally, an overlay tunnel will be used between the ingress router and the egress for steering the traffic to the best site correctly. In SR-MPLS networks or SRv6 networks, a prefix SID is needed. For example, some SRv6 Endpoint Behaviors such as End.DX6, End.X can be encoded for each site so that the egress router can steer the traffic to the corresponding site. The Prefix SID TLV defined {{RFC9085}} can be used to collect this information.

The Prefix SID TLV is an optional TLV to carry the Prefix SID associated with the edge site. The TLV format is illustrated in {{fig-Prefix-SID}}.

~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |              Type             |            Length             |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     |                 Value (Prefix SID sub-TLV)                    |
     |                                                               |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-Prefix-SID title="Prefix-SID TLV format"}

- Type: 1158, identify the Prefix SID Attribute.
- Length: the total number of octets of the value field.
- Value: contains Prefix SID sub-TLV.


### Color Attribute TLV {#color}

Color is used to indicate the service level. For example, different sites may have different levels of service capability which is taken into account by the controller when calculating the path to the egress router. More details can be added in the future revision.

The TLV format(shown in {{fig-Color}}) is similar to the BGP Color Extended Community defined in {{RFC9012}}.

~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |              Type             |            Length             |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |             Flags             |          Color Value          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |          Color Value          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-Color title="Color Attribute TLV format"}

- Type: identify the Color Attribute, to be assigned by IANA.
- Length: 6, length of Flags + Color Value.
- Flags and Color are the same as defined in {{RFC9012}}. Color Value: 32 bit value of color.


# Security Considerations

TBD


# IANA Considerations

This document requires IANA to assign the following code points from the registry called "BGP-LS Node Descriptor, Link Descriptor, Prefix Descriptor, and Attribute TLVs":


| Value | Description | Reference |
|-------|------|-----------|
| TBD1  | Metadata Path Attribute Type | {{metadata}} |
| TBD2  | Site Preference Sub-Type | {{metadata}} |
| TBD3  | Capacity Sub-Type | {{metadata}} |
| TBD4  | Load Measurement Sub-Type1: Aggregated-Cost | {{metadata}} |
| TBD5  | Load Measurement Sub-Type2: Raw-Measurements | {{metadata}} |
| TBD6  | Color Attribute Type | {{color}} |


# Contributors

Xiangfeng Ding

email: dingxiangfeng@huawei.com


--- back
# Acknowledgements
{:numbered="false"}

The authors would like to thank Haibo Wang, LiLi Wang, Jianwei Mao for their help.
