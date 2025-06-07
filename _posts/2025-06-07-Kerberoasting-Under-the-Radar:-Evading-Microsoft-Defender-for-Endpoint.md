---
layout: default
title: "Kerberoasting Under the Radar: Evading Microsoft Defender for Endpoint"
date: 2025-06-07 000:00:00 +0800
categories: [evasion, redteam] 
---

I often encounter Microsoft Defender for Endpoint (MDE) during engagements. MDE relies heavily on the Microsoft-Windows-Threat-Intelligence (ETWTi) provider, which grants access to a rich set of low-level system events. These events are essential for detecting advanced threats that traditional EDR solutions might overlook. During one such engagement, I was exploring ways to perform Kerberoasting attacks, so I began experimenting with MDE’s detection capabilities around Kerberoasting.  

## Understanding the detection logic
I observed that when performing Kerberoasting attacks using Rubeus, MDE triggered two distinct alerts:
- ***Process loaded suspicious .NET assembly*** 
- ***Suspicious LDAP query***


The first alert was consistently triggered when using an unmodified Rubeus binary, indicating that MDE flags assemblies associated with known offensive tooling, particularly when recognizable namespaces are used.

The second alert is self-explanatory and results from Rubeus’s default LDAP query used to perform Kerberoasting.

![]({{ site.url }}/images/2025-05-29_11-15-kerberoast-detection.png)


## Breaking the detection flow

The loading of suspicious .NET assembly can easily be bypassed by simply renaming the default namespace to something not Rubeus in Rubeus.csproj file. 

![]({{ site.url }}/images/2025-05-29_14-24-rubeus-namespace.png)


The Kerberoasting logic in Rubeus is implemented in the file [Roast.cs](https://github.com/GhostPack/Rubeus/blob/master/Rubeus/lib/Roast.cs). One straightforward way to evade detection is by obfuscating the default LDAP query used during the attack. 

*For LDAP obfuscation techniques, I highly recommend this excellent [talk](https://www.youtube.com/watch?v=mKRS5Iyy7Qo) by Daniel Bohannon and Sabajete Eleza.*  


So, we can apply simple obfuscation to the `samAccountType` and `servicePrincipalName` attributes.

![]({{ site.url }}/images/2025-05-29_14-51-obfuscated-ldap-query.png)

## Outcome
With these simple modifications, MDE no longer triggered the previous two alerts. 

![]({{ site.url }}/images/2025-05-31_11-43-kerberoast.png)

While telemetry related to the obfuscated LDAP query and modified .NET assembly was still generated, it did not result in high-fidelity detections.

![]({{ site.url }}/images/2025-05-31_11-41-mde-telemetry.png)

This highlights how minor adjustments can significantly reduce detection rates, emphasizing the importance of telemetry analysis beyond just alerting.