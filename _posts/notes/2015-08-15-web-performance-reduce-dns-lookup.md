---
layout: post
title: Reduce DNS Lookups
category: notes
---

The role of the Domain Name System(DNS) is to map hostnames to IP addresses. When a user type a URL into the browser, a DNS resolver will return the IP address of the server hosting the site that the user requested.

If a server is replaced by one with a different IP address, DNS allows users to use the same hostname to connect to the new server; if multiple IP addresses can be associated with a hostname, DNS provides a high degree of redundancy for a web site.

DNS typically takes 20~120 ms for the browser to look up the IP address for a given hostname. The browser can't download anything from this hostname until the DNS lookup is completed. The response time depends on the DNS resolver:

- The load of requests on it
- User's proximity to it
- User's bandwidth speed

### DNS Caching and TTLs
DNS lookups are cached for bettern performance. There are multiple occasions of DNS being cached. One can occur on a special caching server maintained by the user's ISP or LAN. Another two are at user's side, one is in OS and the other in browser. If browser keeps a DNS record in its cache, then no need to bother OS. Otherwise, OS will either retrieve from its own cache or sends a request to a remote server (where potential slowdowns occur). In addtion, IP addresses could change and DNS caches consume memory. Therefore, the DNS records have to be periodically flushed from the cache, and several different configuration settings determine how often they are discarded.

#### Factors Affecting DNS Caching
The DNS record returned from a lookup contains a time-to-live (TTL) value. This tells the client how long the record can be cached.

While DNS cache in the OS respect the TTL, browsers often ignore it and set their own time limit. Furthermore, the `Keep-Alive` feature of the HTTP can override both the TTL and the browser's time limit.

Browsers put a limit on the number of DNS records cached, regardless of the time the records have been in the cache. Once reaching limit, early DNS records are discarded. However, the OS cache might still be there and that saves the effort to query for the record over the network.

#### TTL Values
The TTL can range from 1 minute to 1 hour. If failover (when old server goes offline and new ones deployed) is critical, short TTL is chosen. Otherwise a longer TTL makes more sense because it reduces the number of DNS lookups.

### Reducing DNS Lookups
The number of DNS lookups is equal to the number of unique hostnames in the web page. Reducing the number of unique hostnames reduces the number of DNS lookups. However, it has the potential to reduce the amount of parallel downloading. So the general guideline is to use at least 2 but no more than 4 hostnames. This results in a good compromise between reducing DNS lookups and allowing a high degree of parallel downloads. Also, make sure the server supports `Keep-Alive`, that helps reducing DNS lookups too.


