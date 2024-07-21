---
layout: post
title:  DNS & GoLinks
date:   2021-03-09 22:00:00 +0800
categories: dns 
excerpt: "A records, huh?"
---

### Why DNS, suddenly? 
GoLinks, a URL shortener service where if you were to input a short link, e.g. `go/docs` in the URL of your web browser, you would be brought to the long link, e.g. `your-company.intranet.org/very-long-documentation-link` that was hidden behind the short link. 

After coming across a self-hosted GoLinks service, I wondered how `go/docs` actually worked. The url didn't even contain any of the domain suffixes that I am used to like `.com`, `.sg` or `.org`. 

I guessed that DNS had a part to play, and confirmed it after looking at the setup guide of [GoLinks](https://github.com/GoLinks/golinks). 

> You have to add an A record

Hmmm. That sounds like DNS alright, and I know what DNS does! It translates easy to remember names like `www.example.com` to its IP address. Our web browser contacts a DNS server whenever we try to access something like `www.example.com` to fetch its corresponding IP address.

> What are A records?

A records? It's a DNS thing... What my reply would be, if I were to be asked about it. Using this little run-in with DNS, I decided to stretch my understanding for a bit.

### DNS lookup
Although we refer to them usually as just DNS servers, there are actually 4 types. 

1. Recursive resolvers
1. Root nameservers
1. TLD nameservers
1. Authoritative nameservers

When a web client wants to resolve `www.example.com`, it queries a DNS recursive resolver (e.g. Google's `8.8.8.8`). The DNS recursive resolver could respond with a cached result if it has recently answered another `www.example.com` query. If not, it will: 

1. Query a root nameserver (there are 13 in the world) for the .com top-level domain (TLD) nameserver
2. Query the .com TLD nameserver, for the authoritative nameserver responsible for the domain example.com 
3. Query the example nameserver for the IP address of `www.example.com` and returns it to the web client

### Types of DNS records
NS records - Records that indicate which DNS nameserver will know IP address of the hostname. 

A records - Records that map a hostname and its corresponding IP address. `www.example.com` to `93.184.216.34` 

There are more (i.e. CNAME, MX, etc) but we don't need them here.

With the DNS lookup flow and the two types of DNS records in mind, what do the various touchpoints (e.g. the root nameserver, .com TLD nameserver, ..) have to contain?

### DNS Records
The root nameserver should contain NS records of the .com TLD nameservers

The .com TLD nameserver should contain NS records of the example.com authoritative nameservers

The example.com nameserver should an A record of `www.example.com` 

### Using dig
`dig +trace ww.example.com @8.8.8.8` allows us to follow how a DNS query to 8.8.8.8 for www.example.com gets resolved from the start to the end. The trace can be seen in the eyes of the DNS recursive resolver 8.8.8.8.

Let's verify this trace with our understanding from before.

{% highlight shell %}
$ dig +trace www.example.com @8.8.8.8
; <<>> DiG 9.16.6-Ubuntu <<>> +trace www.example.com @8.8.8.8
;; global options: +cmd
.			33781	IN	NS	a.root-servers.net.
.			33781	IN	NS	b.root-servers.net.
.			33781	IN	NS	c.root-servers.net.
.			33781	IN	NS	d.root-servers.net.
.			33781	IN	NS	e.root-servers.net.
.			33781	IN	NS	f.root-servers.net.
.			33781	IN	NS	g.root-servers.net.
.			33781	IN	NS	h.root-servers.net.
.			33781	IN	NS	i.root-servers.net.
.			33781	IN	NS	j.root-servers.net.
.			33781	IN	NS	k.root-servers.net.
.			33781	IN	NS	l.root-servers.net.
.			33781	IN	NS	m.root-servers.net.
.			33781	IN	RRSIG	NS 8 0 518400 20210319170000 20210306160000 42351 . iqsCUuF9/rC1oJuFGJnO9HjxYmXhxeRwRKPTndTgVIDiTdtV5lRw+ege 0zalNDI02MUZeSSEoUVohOg4SnMpVKGg/RCQt94ZnLnm/NVnYcpqykHF bubq+Kln2ITVrTGdYjrkTVidbm25DH1m8w2P0yrhtqJsPU7aNp5bKpi+ ZpOUBV+xmt10RT1DRakJEH+Z2ysY/n0FV/zOhZKZptqGOXI03K0EKevo +NxOy69djacvT/8kQEzOs87IqGRTkOKZrwHHLe/U8qW9fkYC+iMjcd+g trgFsWn5T4NA9Zdp7tgcs+1xsOtsrPXKSiG8MvkQx7P2XXoHClQiHFdf udjDDg==
;; Received 525 bytes from 8.8.8.8#53(8.8.8.8) in 4 ms
{% endhighlight %}

We see that we get first a list of root nameservers. This are stored in a roots hint file in the DNS recursive resolver `8.8.8.8`.

{% highlight shell %}
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			86400	IN	DS	30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
com.			86400	IN	RRSIG	DS 8 1 86400 20210320050000 20210307040000 42351 . aQXxSbd2K+tk/YuLKO8QebROjVDwIvFHqCAKoudJA2m5kBCsR7XNethY W2LM0vENSe0oOhKV1+u+xJDA4pPo4y2H+n4GIxx4Ibx/JK4YKr4grFrG JUW1e5EL+m3BE0Ocw5C1C39Si0TvLintnfdXTKCc93I7aRQ18cjrnXmx UCXEyTXB8lZRow0gWqifQL1uXbSDbUvWJo1c+tXJIuwU75JR1/XDgc/3 5LHQWJaCOYfqgZw8yNB5mEOG+71ecoyA6gs5KbcNno2YWYz59vw/mU+g jWH4IafhPAKj1oRecbKM8vXr1sAR1E7nJVFhNT1HYJcpY+P6t9HUS93e WbGCIQ==
;; Received 1175 bytes from 2001:500:1::53#53(h.root-servers.net) in 40 ms
{% endhighlight %}

We then query the root nameserver `h.root-servers.net` for the .com TLD nameservers

{% highlight shell %}
example.com.		172800	IN	NS	a.iana-servers.net.
example.com.		172800	IN	NS	b.iana-servers.net.
example.com.		86400	IN	DS	31589 8 1 3490A6806D47F17A34C29E2CE80E8A999FFBE4BE
example.com.		86400	IN	DS	31589 8 2 CDE0D742D6998AA554A92D890F8184C698CFAC8A26FA59875A990C03 E576343C
example.com.		86400	IN	DS	43547 8 1 B6225AB2CC613E0DCA7962BDC2342EA4F1B56083
example.com.		86400	IN	DS	43547 8 2 615A64233543F66F44D68933625B17497C89A70E858ED76A2145997E DF96A918
example.com.		86400	IN	DS	31406 8 1 189968811E6EBA862DD6C209F75623D8D9ED9142
example.com.		86400	IN	DS	31406 8 2 F78CF3344F72137235098ECBBD08947C2C9001C7F6A085A17F518B5D 8F6B916D
example.com.		86400	IN	RRSIG	DS 8 2 86400 20210312052107 20210305041107 58540 com. oO2xUltbwHywM7BCfDy+izBNuV8qHD/fWOvWnpV0XWObn3ZqSz+mrybJ rYIwOJnxFk2B/Q3teMcB1hrpjNV2AedaWJ49/Ha909figkZOsgAQr0H1 00+fe4J9S/IZSEUj28K5e1NLHSoBUn5GU1SXNacS0Xxb1BFg48vhh+3Z zvXhtPMgRSRr1QPi4IpGBKK5iC7gEMtRxFlazO86pwy9Rw==
;; Received 539 bytes from 2001:503:d414::30#53(f.gtld-servers.net) in 248 ms
{% endhighlight %}

Next, we query a .com TLD nameserver `f.gtld-servers.net` for the NS record of the example.com authoritative nameserver

{% highlight shell %}
www.example.com.	86400	IN	A	93.184.216.34
www.example.com.	86400	IN	RRSIG	A 8 3 86400 20210316085034 20210223165712 45150 example.com. UyyNiGG0WDAsberOUza21vYos8vDc6aLq8FV9lvJT4YRBn6V8CTd3cdo ljXV5uETcD54tuv1kLZWg7YZxSQDGFeNC3luZFkbrWAqPbHXy4D7Tdey LBK0R3xywGxgZIEfp9HMjpZpikFQuKC/iFvd14uJhoquMqFPFvTfJB/s XJ8=
;; Received 231 bytes from 199.43.135.53#53(a.iana-servers.net) in 180 ms
{% endhighlight %}

Finally, we query the authoritative nameserver a.iana-servers.net for the A record of `www.example.com`, getting `93.184.216.34` back.

### Why is there an extra . at the back?
Adding a . at the end makes the domain name absolute. I think it is another rabbit hole that I would want to avoid for now, but here is my way of thinking about it.

If we were to look at a domain name wwww.example.com, we could say that the domain name (from the right to the left) represents the domain name records you have to traverse to resolve it. However, the query does not start with the records in the .com TLD nameserver, but the root nameserver. Hence, by including the trailing . as in `www.example.com.`, it accurately represents the traversal of domain name records to resolve it.

## GoLinks
Back to GoLinks. This is why you need an A record in your authoritative DNS server for the `go` in `go/docs` to resolve to the GoLinks server so that the GoLinks server can redirect users to the long url given the short url.

## References
<https://www.cloudflare.com/en-ca/learning/dns/dns-server-types/>
<https://superuser.com/questions/715632/how-does-dig-trace-actually-work>
<https://github.com/GoLinks/golinks>
