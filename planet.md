---
layout: post
title: "Critical Vulnerabilities Discovered in PLANET IGS-4215-16T2S Industrial Switches"
date: 2024-02-15
categories: [security, research]
tags: [vulnerabilities, network-security, planet-technology, industrial-switches]
---

Industrial networking equipment plays a crucial role in modern infrastructure, connecting and controlling critical systems across manufacturing plants, energy facilities, and smart city deployments. The PLANET IGS-4215-16T2S is a managed industrial switch designed for these demanding environments, featuring 16 10/100/1000T ports and 2 100/1000X SFP ports, making it a popular choice for industrial applications requiring reliable Ethernet connectivity.

During a recent security assessment at IOActive, I had the opportunity to analyze this device from a security perspective. The IGS-4215-16T2S comes with features like VLAN support, Quality of Service (QoS), and various management interfaces, including a web-based administration panel. However, our investigation revealed several concerning security vulnerabilities that could potentially compromise not just the device itself, but the entire industrial network it's connected to.

## Unauthenticated Access to Backup Files (CVE-2024-2740)

### Description
The first and most critical vulnerability discovered carries a HIGH severity rating. While the device's administrative web interface requires authentication for most operations, we found that the session verification mechanism failed to protect several critical system resources.

The device stores various configuration and backup files in predictable paths within the `/tmp/` directory, including:
- "Critical Vulne---
layout: post
title: "Cri---
layout: post
title: "Cr
What makes this particularly concerning is that these files contain sensitive information, including device credentials, and can be accessed without any authentication.

### Impact
An unauthenticated attacker could:
- Access complete device configurations
- Extract stored usernames and passwords
- Potentially compromise the entire industrial network
- Gain persistent access to the device and its management features

## CSRF in Administrative User Creation (CVE-2024-274)

### Description
The second vulnerability, rated as MEDIUM severity, enables Cross-Site Request Forgery (CSRF) attacks that could lead to unauthorized creation of administrative users without the legitimate administrator's knowledge.

The issue stems from several security oversights:
1. No CSRF tokens implementation to protect sensitive actions
2. Acceptance of GET requests for operations that should be POST-only
3. Lack of re-authentication requirements for critical actions like user creation

### Proof of Concept
The vulnerability can be exploited using a simple HTML form:
ilities that could potentially compromise not just the device itself, but the entire industrial network it's connected to.

## Unauthenticated Access to Backup Files (CVE-2024-2740)

### Description
The first and most critical vulnerability discovered carries a HIGH severity rating. While the device's administrative web interface requires authentication for most operations, we found that the session verification mechanism failed to protect several critical system resour
### Impact
An attacker could:
- Create new administrative users without authorization
- Gain persistent access to the device
- Execute malicious actions appearing as legitimate user activity
- Maintain long-term unauthorized access even after the initial compromise is detected

## Timeline and Resolution

The vulnerability was discovered on September 29, 2022, and reported to PLANET Technology on March 29, 2023. The manufacturer released a firmware update (version 1.305b231218) on December 13, 2023, that addresses these vulnerabilities.

## Recommendations

For network administrators using these devices, we recommend:

1. Immediately upgrade to the latest available firmware (1.305b240227 or higher)
2. Implement network segmentation to isolate these devices
3. Actively monitor access to these equipments
4.Conduct regular audits of users and configurations
5. Consider implementing additional security controls at the network level

## Conclusion

These vulnerabilities highlight the ongoing challenges in industrial device security. Basic security controls like session verification and CSRF protection should be standard features in any networked device, especially those deployed in critical industrial environments.

I want es to acknowledge PLANET Technology's prompt response and collaboration in addressing these vulnerabilities. Additionally, I'd like to thank INCIBE for their assistance in managing the CVE process.

This research underscores the importance of regular security assessments for industrial network equipment, as even seemingly basic vulnerabilities can have significant implications for operational technology environments.
