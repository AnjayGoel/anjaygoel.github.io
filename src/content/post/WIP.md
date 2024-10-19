---
title: ""
publishDate: 2024-09-07 19:30:00 +0530
tags: [ til, internet, dns, cloudflare ]
description: ""
---

I recently moved this blog from GitHub to its own domain, and I was surprised to see how streamlined and fast the whole
process was. It barely took me 30 minutes to set everything up. Since then, I've been fiddling with the Cloudflare
dashboard, its online glossary and other resources to understand how it all works. Unsurprisingly, there is a lot going
on under the hood. So, I am writing this to summarize what I've learned so far.

## Who actually owns and manages the domain?

ICANN is a global non-profit organization that has authority over all the top-level domains (called TLDs). It delegates
this responsibility to registry operators (Such as Verisign for .com, .net, etc.). These
registries in-turn delegate the sales to domain registrars (like Cloudflare, GoDaddy, Namecheap, etc.). The registrars
use a protocol called EPP to communicate with the registry.

When you buy a domain, it goes through this chain something like this:
<insert image>

There are two types of TLDs: gTLDs (generic TLDs) and ccTLDs (country code TLDs). ICANN manages the former, while the
latter is managed by the respective country. So the ccTLDs have their own country-specific set of regulations.

Fun fact: All two-letter top-level domains (like .in, .ai, .io) are ccTLDs.They are the respective ISO two-letter
country codes. And the .io might disappear soon (Insert link)
REF: https://www.cloudflare.com/en-gb/learning/dns/glossary/what-is-a-domain-name-registrar/

## The Domain Name System (DNS)

### The Query Resolution Process

We all know about DNS, it's a phonebook of sorts for the internet. It maps domain names to IP addresses. I always
thought that the DNS was a single monolithic system that stored all the domain records. But it's actually not the case.
The DNS servers we are familiar with (1.1.1.1, 8.8.8.8/8.8.4.4) are called recursive resolvers. There are three other
kinds of DNS servers in the whole DNS hierarchy:

* The root servers
* The TLD servers
* The authoritative servers

Let's see how they all fit together:
You enter "example.com" in your browser.
The browser sends a query to the recursive resolver, which in turn asks the root server. The root server responds with
the TLD server for the corresponding domain (.com). ICANN primarily manages these root-servers.
The recursive resolver then asks the TLD server for the authoritative server for the domain. The TLD responds with the
authoritative server for "example.com". The authoritative is the final stop in the DNS hierarchy. It holds actual DNS
records for the domain. This is where the IP address for "example.com" is stored.

Some fun facts:

* Not all the queries go through this whole process. It will often be cached at several levels. For example, your
  browser, the operating system, some routers and at the recursive resolver, etc.
* There are 13 root servers worldwide (https://www.iana.org/domains/root/servers), well sort of. These are replicated
  across the globe, so there are actually thousands of root servers. but they all share the same IP address. This is
  possible because of Anycast routing (https://www.cloudflare.com/en-gb/learning/dns/what-is-anycast-dns/). Every
  resolver has the IP of these 13 root servers built-in.
* You can change the authoritative server for your domain via your registrar. In fact, I can go to my cloudflare
  dashboard and create DNS records for "example.com", but it won't work because its registry is pointing to a
  different nameserver.

### DNS resolution in action

You can see this whole process in action by using the `dig` command in your terminal. You can install it using
`brew install bind` on macOS and `sudo apt install dnsutils` on Linux.

```bash

### The DNS records

### DNS as load balancer

## HTTPS and SSL/TLS

https://www.cloudflare.com/en-gb/learning/dns/dns-server-types/
https://www.cloudflare.com/en-gb/learning/dns/what-is-dns/
https://www.cloudflare.com/en-gb/learning/dns/glossary/dns-root-server/
