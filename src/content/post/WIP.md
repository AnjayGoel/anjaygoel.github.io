---
title: "WIP"
publishDate: 2024-11-01 19:30:00 +0530
tags: [ til, internet, dns, cloudflare ]
description: "WIP               "
draft: true
---

I recently moved this blog from GitHub to its own domain, and I was surprised to see how streamlined and fast the whole
process was. It barely took me 30 minutes to set up everything. Cloudflare did most of the heavy-lifting for me. Since
then, I've been fiddling with the Cloudflare dashboard, its online glossary and other resources to understand how it all
works. Unsurprisingly, there is a lot going on under the hood. So, I am writing this to summarize what I've learned so
far.

## Who am I getting the domain from?

I go to a domain registrar, click a few buttons and voila!, I have got my own domain. But whats happening under the
hood?

Lets talk about ICANN; ICANN is a global non-profit organization that has authority over all the top-level
domains (called TLDs). It delegates this responsibility to registry operators (Such as Verisign for .com, .net, etc.).
These registries in-turn delegate the sales to domain registrars (like Cloudflare, GoDaddy, Namecheap, etc.). These
registrars use a protocol called [EPP](https://en.wikipedia.org/wiki/Extensible_Provisioning_Protocol) to communicate
with the registry and perform operations like registering a domain, renewals etc.
Note that there are two types of TLDs:
gTLDs (generic TLDs) and ccTLDs (country code TLDs). The latter being managed by the respective country. So the ccTLDs
have their own country-specific set of regulations.

The hierarchy looks something like this:
<figure>
<img src="https://www.cloudflare.com/img/learning/dns/glossary/what-is-a-domain-name-registrar/registrar-flow.png">
<figcaption>Source: <a href="https://www.cloudflare.com/en-gb/learning/dns/glossary/what-is-a-domain-name-registrar/">Cloudflare:What is a domain name registrar?
</a> </figcaption>
</figure>


**Fun fact:** All two-letter top-level domains (like .in, .ai, .io) are ccTLDs. They are the respective ISO two-letter
country codes. And the [.io domain mightdisappear soon](https://every.to/p/the-disappearance-of-an-internet-domain)

## How does the domain point to my server?

I have a domain of my own now, but how do I get it to point to my server? This is where the Domain Name System (DNS)
comes in. We all know about DNS, it's a phonebook of sorts for the internet. It maps domain names to IP addresses. I
always thought that the DNS was a single monolithic system that stored all the domain records. But it's actually not the
case. The DNS servers we are familiar with (1.1.1.1, 8.8.8.8/8.8.4.4) are called recursive resolvers. There are three
other kinds of DNS servers in the whole DNS hierarchy:

* The root servers
* The TLD servers
* The authoritative servers

Let's see how they all fit together:

1. You enter "example.com" in your browser.
2. The browser sends a query to the recursive resolver, which in turn asks the root server.
3. The root server responds with the TLD server for the corresponding domain (.com).
4. The recursive resolver then asks the TLD server for the authoritative server for the domain.
5. The TLD responds with the authoritative server for "example.com."
6. The authoritative is the final stop in the DNS hierarchy. It holds actual DNS records for the domain. This is where
   the IP address for "example.com" is stored.

<br>

**Some fun facts:**

* Not all the queries go through this whole process. It will often be cached at several levels. For example, your
  browser, the operating system, some routers and at the recursive resolver, etc.
* There are [13 root servers worldwide](https://www.iana.org/domains/root/servers), well sort of. These are replicated
  across the globe, so there are actually thousands of root servers. but they all share the same IP address. This is
  possible because of [Anycast routing](https://www.cloudflare.com/en-gb/learning/dns/what-is-anycast-dns/). Every
  resolver has the IP of these 13 root servers built-in.
* You can change the authoritative server for your domain via your registrar. In fact, I can go to my cloudflare
  dashboard and create DNS records for "example.com", but it won't work because its registry is pointing to a
  different nameserver.

### DNS resolution in action

Let's see the query resolution in action by using the `dig` command in your terminal.
You can install it using `brew install bind` on macOS and `sudo apt install dnsutils` on Linux.
Now let's run a trace for "google.co.in":

```bash
dig google.co.in +trace
```

The output will look something like this:
![dig trace output](../../assets/images/wip/dig-trace-output.png)

Let's break down what's happening here:

* The recursive resolver (1.1.1.1, in this case) sends a query to the root servers.
  Note how there are 13 of them, as we discussed earlier.
* The root server responds with the TLD servers for ".in" (ccTLD for India, the registry is registry.in).
* The recursive resolver then asks the TLD server for the authoritative server for "google.co.in".
* The TLD server responds with the authoritative servers for "google.co.in".
* The recursive resolver then asks the authoritative server for the IP address of "google.co.in".
* The authoritative server responds with the IP address of "google.co.in", 142.250.195.195 in this case.

There are some other details in the output, like the type of the record (`NS`, `DS`, `RRSIG`), TTL (Time To Live)
for the records, query time, etc.

### The DNS records

The authoritative server holds the actual DNS records for the domain. There are tens of different types of DNS records,
but the ones you will most commonly use and encounter are given below:

* A: Maps a domain/subdomain to an IPv4 address. This is what's needed to get a website working. For example,
  `dig anjaygoel.com A` will return the IPv4 address of my server: `anjaygoel.com.		115	IN	A	172.67.168.31`
* AAAA : Maps a domain/subdomain to an IPv6 address.
* CNAME: Forwards to another domain/subdomain. Its like a reference to another domain.
* NS: Specifies the authoritative server for the domain. Like in the output above.
* MX: Redirects email traffic to a mail server.
* TXT: Holds text data. Often used for verification purposes. For example, Google uses it for domain verification in
  Google Search Console. `dig anjaygoel.com TXT` will the following output:
  ```
  anjaygoel.com.		300	IN	TXT	"v=spf1 -all"
  anjaygoel.com.		300	IN	TXT	"google-site-verification=aTRw2UBBr_jg-Z_WM0kMf6mgXnzrCIgd54xcZtXohIk"
  ```
* PTR: Maps an IP address to a domain/subdomain. This is useful for reverse DNS lookups. The are stored as <reverse-ip>
  .in-addr.arpa under the .arpa TLD. You can use `dig -x <ip>` to do a reverse lookup.

### DNS as load balancer

https://www.cloudflare.com/en-gb/learning/performance/what-is-dns-load-balancing/
Going a little off-topic, But I found this super cool. You can actually use DNS to load balance traffic across multiple
servers! Based on the load balancing algorithm, the DNS server will return different A/AAAA records for the same domain.
Lets see this in action:

`dig facebook.com` gives me `57.144.124.1` on my PC. If I do a reverse lookup using `dig -x 57.144.124.1`, I get the
pointer record as `edge-star-mini-shv-03-bom2.facebook.com.` (Bom as in Bombay or mumbai). But if I do the same on a
colab notebook, I get a different IP address, `31.13.67.35` and the reverse lookup gives me
`edge-star-mini-shv-01-mia3.facebook.com`. (mia as in Miami) which is closer to where my colab notebook is running(
South Carolina).

## How Do I secure It?: HTTPS and SSL/TLS

Now that I have got my domain pointing to my server, I need to secure the connection between the client and the server.

https://www.cloudflare.com/en-gb/learning/dns/glossary/what-is-a-domain-name-registrar/
https://www.cloudflare.com/en-gb/learning/dns/dns-server-types/
https://www.cloudflare.com/en-gb/learning/dns/what-is-dns/
https://www.cloudflare.com/en-gb/learning/dns/glossary/dns-root-server/
https://www.cloudflare.com/en-gb/learning/dns/dns-records/
