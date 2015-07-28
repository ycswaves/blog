---
layout: post
title: Use Content Delivery Network (CDN)
---

For website facing large audience, it's necessary to deploy content across **multiple** and **geographically dispersed** servers.

> CDN is a collection of web servers distributed across multiple locations to deliver content to users more efficiently. CDNs are used to delivery static content, such as images, scripts stylesheets and Flash.

Servers from the collection is selected based on a measure of network proximity (e.g. fewest networks hops, quickest response).

| Pros                               | Cons                                 | 
| ---------------------------------- | ------------------------------------ |  
| Improve reponse time               | Can be affectd by traffic from other websites from the same CDN providers | 
| Backups, extended storage capacity | Inconvenience of not having control of the content servers (e.g. modifying HTTP reponse header)|  
| Caching                            | Dependent on CDN provider's performance    | 
| Absorb spikes in traffic           |