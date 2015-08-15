---
layout: post
title: Reduce DNS Lookups
---

The role of the Domain Name System(DNS) is to map hostnames to IP addresses. When a user type a URL into the browser, a DNS resolver will return the IP address of the server hosting the site that the user requested.

If a server is replaced by one with a different IP address, DNS allows users to use the same hostname to connect to the new server; if multiple IP addresses can be associated with a hostname, DNS provides a high degree of redundancy for a web site.

DNS typically takes 20~120 ms for the browser to look up the IP address for a given hostname. The browser can't download anything from this hostname until the DNS lookup is completed. The response time depends on the DNS resolver:

- The load of requests on it
- User's proximity to it
- User's bandwidth speed

