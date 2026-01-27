---
title: Considerations on Authoritative Information for Source Address Validation
abbrev: Authoritative Information Considerations
docname: draft-qin-savnet-authoritative-information-01
obsoletes:
updates:
date:
category: info
submissionType: IETF

ipr: trust200902
area: Routing
workgroup: SAVNET
keyword: SAV

author:
 -
  ins: L. Qin
  name: Lancheng Qin
  organization: Zhongguancun Laboratory
  email: qinlc@mail.zgclab.edu.cn
  city: Beijing
  country: China
 -
  ins: D. Li
  name: Dan Li
  organization: Tsinghua University
  email: tolidan@tsinghua.edu.cn
  city: Beijing
  country: China

normative:

informative:
  RFC2827:
  RFC3704:
  RFC8704:
  RFC9582:
  I-D.qin-savnet-toa:
  I-D.ietf-sidrops-aspa-profile:
  I-D.ietf-savnet-general-sav-capabilities: 

  
...

--- abstract

Source Address Validation (SAV) prevents source address spoofing. This document explains that SAVNET relies on authoritative information. It also describes how to handle missing or conflicting data. The guidance minimizes improper blocks and improper permits while supporting reliable SAV enforcement.

--- middle

# Introduction {#sec-intro}

Source Address Validation (SAV) prevents source address spoofing and enforces BCP38 [RFC2827], BCP84 [RFC3704], and [RFC8704]. Networks rely on authoritative information to determine which source addresses are legitimate. However, networks may encounter situations where this information is missing or conflicting.

This document provides a conceptual framework for understanding authoritative information in the context of SAVNET, including:

- What constitutes authoritative information and which sources can be trusted.

- How networks should handle missing authoritative information.

- How to reconcile conflicting authoritative sources.

- The role of non-authoritative information as a reference in contextual or policy-based decisions.

By clarifying these principles, the document aims to guide the design and operation of SAV mechanisms that are both secure and operationally reliable, while minimizing improper blocks and improper permits.

## Requirements Language

{::boilerplate bcp14-tagged}

# What Constitutes Authoritative Information

Authoritative information determines which source addresses are legitimate. To be considered authoritative, information should meet the following criteria:

- Organizational authority: The source must be maintained by an entity that has recognized authority over the relevant prefixes or networks.

- Verifiability: The source must provide a mechanism to verify its correctness and authenticity, such as cryptographic validation.

- Timeliness: The information must reflect the current operational state and be updated promptly to avoid reliance on stale or outdated data.

- Integrity and security: The source must be resistant to unauthorized modifications or tampering.

Based on these criteria, authoritative information in SAVNET typically includes:

- RPKI objects: Cryptographically verifiable objects such as ROAs (Route Origin Authorizations) [RFC9582], ASPAs (Autonomous System Provider Authorizations) {{I-D.ietf-sidrops-aspa-profile}}, and TOAs (Traffic Origin Authorizations) {{I-D.qin-savnet-toa}} that provide explicit assertions about origin authorization or transit authorization.

- Local or static configuration: Operator-defined rules specifying source address permissions for hosts, non-BGP customer networks, or external ASes.

Note on routing information:
Routing information from intra-domain or inter-domain protocols (e.g., IGP, BGP) may indicate reachable prefixes and their origins, but it cannot be considered authoritative by itself because it may include unauthorized or forged routes. If routing information is used to derive SAV rules, it should be validated—such as via RPKI-based Route Origin Validation (ROV)—before being treated as a trusted source.

By defining authoritative information in this way, SAV mechanisms can rely on sources that provide sufficient guarantees of correctness, integrity, and timeliness, reducing the risk of improper blocks or improper permits in filtering.

# When Authoritative Information Is Missing

Networks may encounter situations where authoritative information about a particular prefix or source entity is unavailable. Such situations can arise for various reasons, including:

- The relevant RPKI objects (e.g., ROAs, ASPAs, TOAs) do not exist or have not yet been published.

- Local configuration has not been defined for a newly connected host, non-BGP customer network, or external AS.

When authoritative information is missing, networks must decide how to handle traffic from the corresponding source addresses. Several approaches have been considered:

- Conservative approach: Treat the source addresses as unauthorized and block traffic. This minimizes the risk of accepting spoofed packets but may result in legitimate traffic being dropped.

- Permissive approach: Allow traffic from the source addresses by default. This avoids accidental disruption of legitimate communications but increases the risk of accepting spoofed packets.

- Contextual or policy-based approach: Apply local policies or heuristics to make decisions when authoritative information is missing. Contextual information may include trusted routing information, historical traffic patterns, or operational context. Given the current limited deployment of authoritative information, this approach is often the only practical way to make incremental progress in enforcing source address validation. It allows operators to mitigate spoofing risk while accommodating the realities of incomplete authoritative data.

Even when contextual or trusted information is used, uncertainty remains. Trusted data may be missing, stale, or inconsistent for specific prefixes or ASes. Operators must therefore decide how enforcement should behave under these conditions. Options include strict blocking, permissive acceptance, or intermediate strategies, such as permitting traffic while logging and investigating. [I-D.ietf-savnet-general-sav-capabilities] defines a broader set of traffic-handling actions, including rate limiting, traffic redirection, counting, sampling, and monitoring, which can help operators manage uncertainty while maintaining operational continuity.

Operators should document their chosen strategy and apply it consistently across interfaces. While authoritative information provides the strongest basis for high-confidence filtering decisions, fallback strategies based on contextual information offer a practical path for incremental deployment and operational learning.


# When Authoritative Sources Conflict

Networks may encounter situations where multiple sources of authoritative information provide overlapping or apparently conflicting statements about the legitimacy of a source address or prefix. Such conflicts can arise, for example, when:

- Different RPKI objects (ROAs, ASPAs, TOAs) provide overlapping assertions for the same prefix.

- Local or static configurations overlap with information from other authoritative sources.

Networks should treat all authoritative sources as equally credible and merge information from all sources. Any address or prefix authorized by at least one source should be considered legitimate. This approach ensures that no legitimate source address is incorrectly blocked, reducing improper blocks while maintaining reliable SAV enforcement.

Fallback and reference to non-authoritative information:
When authoritative information is incomplete or missing, non-authoritative information—such as routing data—may be used as a reference within a contextual or policy-based approach (see Section 3) to help inform decisions, but it cannot be relied upon as definitive.

# Discussion and Next Steps

This document provides a conceptual framework for understanding authoritative information in SAVNET, addressing scenarios where information is missing or conflicting. The following points highlight key considerations for SAV design and operations:

- Definition of authoritative information: Networks must rely on sources that are verifiable, secure, timely, and maintained by recognized authorities, such as RPKI objects (ROAs, ASPAs, TOAs), SAV-specific signaling with security guarantees, or operator-defined local/static configuration.

- Handling missing information: When authoritative information is unavailable, fallback strategies—conservative, permissive, or contextual/policy-based using non-authoritative information as reference—should be defined.

- Conflict resolution: Conflicting authoritative sources should be merged to ensure all authorized prefixes and source addresses are preserved.

- Open questions: Additional work may include defining authoritative attributes in greater detail, coordinating with other working groups (e.g., GROW, OSPAWG) for operational guidance, and specifying mechanisms to securely exchange SAV-specific signaling information.

This framework provides a foundation for discussion and standardization of SAV mechanisms that rely on authoritative information, ensuring both security and operational reliability.

# Security and Operational Considerations

Reliable SAV enforcement depends on correct identification of legitimate source addresses. Inaccurate, missing, or conflicting authoritative information can lead to operational and security risks, including:

- Improper blocks: Legitimate traffic is blocked, potentially disrupting services.

- Improper permits: Spoofed or unauthorized traffic is accepted, increasing vulnerability to attacks.

Mitigation strategies include:

- Validation and cross-checking: Ensure authoritative sources are consistent and verifiable.

- Timely updates and monitoring: Maintain freshness of authoritative information to avoid reliance on stale data.

- Documentation and operational procedures: Clearly define policies for missing, inaccurate, or conflicting authoritative information, including fallback handling.

- Fallback and reference mechanisms: Non-authoritative information (e.g., routing data) may be used as a reference in contextual or policy-based approaches but must never be treated as definitive.

- Merge strategy for conflicts: All authoritative sources should be combined, ensuring that any prefix or source address authorized by at least one source is accepted, minimizing improper blocks.

By implementing these practices, networks can maintain reliable SAV enforcement while reducing both security and operational risks. This approach emphasizes using authoritative information wherever possible and leveraging non-authoritative data only as a reference when necessary.

# IANA Considerations {#sec-iana}

This document does not request any IANA allocations.


--- back



