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

### DNS Caching and TTLs
DNS lookups are cached for bettern performance. There are multiple occasions of DNS being cached. One can occur on a special caching server maintained by the user's ISP or LAN. Another two are at user's side, one is in OS and the other in browser. If browser keeps a DNS record in its cache, then no need to bother OS. Otherwise, OS will either retrieve from its own cache or sends a request to a remote server (where potential slowdowns occur). In addtion, IP addresses could change and DNS caches consume memory. Therefore, the DNS records have to be periodically flushed from the cache, and several different configuration settings determine how often they are discarded.

