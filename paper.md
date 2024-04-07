---
title: The GFW of China
bibliography: references.bib
link-citations: true
---

# 434 Paper

**Abstract** -- 
The (Great Firewall) GFW of China is a national scale firewall created by 
  the Chinese government for the purpose of controlling 
  and censoring information on the internet. 
The network layer of GFW is implemented using IDS technology in 
  filtering routers spread around the many AS-es in China.
At these filtering routers packets are inspected using four main mechanisms: 
  IP blocking, DNS poisoning, Keyword filtering and stateful traffic analysis.
The focus of this paper is to examine the mechanisms in use by the GFW 
  focusing on their implementation in network and transportation layer, 
  looking at flaws and main strategies of circumventing the GFW.

**Keywords** -- 
  GFW of China, IDS, IP blocking, DNS manipulation, URL filtering, 
  Keyword filtering, RST packets, Circumvention

## I. Introduction

*Across the **Great Wall** can reach every corner in the world.* <x>@01_hist1986</x> 
This email message sent to Karlsruhe University of Germany, 
from Beijing in September 1987, marks China’s entrance into the Internet. 
Ironically, as of December 2022, with 1.07 billion Internet users domestically,
<x>@02_stat2022</x> it would be nearly impossible to reach to anywhere on Internet 
without break across the GFW, the Great Firewall of China. 

The GFW comprises of intricate and multifaceted mechanisms,
  which ranges over technical, societal and political aspects. 
Beyond being a technical barrier, the Wall also includes legal ramifications 
  for those attempting to bypass it, 
  mandatory content regulations responsibilities for platform hosts,
  and a pervasive surveillance network that monitors digital footprints, 
    and subtly fabricating the psychological norms, 
    shapes thought and molds behavior of the users.

In this paper, we will talk about the technical aspects of the Wall, 
  and briefly discusses the main strategies of circumventing it.

## II. Operation of the GFW

Before covering the mechanisms used by the GFW, 
it is worth briefly covering how and where the packet monitoring is done 
to get a complete picture of the system.

### Structure of the GFW

AS-es (autonomous systems) can be divided into internal and border AS-es 
  where border AS-es communicate to foreign AS-es. 
Most of the filtering done by the GFW is done in the border AS-es <x>@03_whereFilter</x>
  where the request and response packets enter and leave the country,
  some filtering done inside internal AS-es <x>@03_whereFilter</x> as well.

Majority of the filtering is done in border AS-es, as it is the main concern on
  the network layer. 
  GFW's main work for domestic traffic is usually done in the application layer,
  through social engineering and state surveillance on hosting platforms and 
  the users, such as requiring content censors prior to publication 
  on all platforms, real-name registration of the users, and potentially legal
  repercussions for those who post illicit content, etc. <x>@04_digitalEco_mypaper</x>. 
  We would not cover these aspects in this paper.

Having filtering done in internal AS-es is also likely an efficiency measure as 
  having everything done in border AS-es could bottleneck the network so 
  mitigating some of the filtering into the interior AS-es can distribute the 
  the load more evenly.

An important note is that the GFW isn’t truly a wall in literal sense, 
  as the paths to nodes in China might experience different levels of filtering,
  and the list of conditions for a packet to be censorable would change over time
  (e.g. the term "Tiananmen Square" would rise in censorship around the
  anniversary of the 1989 protests in May and June) <x>@05_conceptDoppler</x>.

It is the case that the purpose of the GFW isn’t to create an impassable wall
 that prevents all foreign traffic, as it's technically infeasible and 
 economically destructive to do so, but more to instill fear and self-censorship
 and implicitly mold the online behavior with the sense of compliance.

Flaws and opportunities for exploitation exist for each level of filtering,
  if exploiting these flaws would lead to eventual policy change due to the 
  increasing cost of maintaining the GFW, or the increasing dissent from the
  citizens, or the increasing pressure from the international community; The
  circumvention tools would be the catalyst for the change. <x>07_taxonomy</x>

### How Packets Are Monitored

The packets in the internet traffic tracked by the GFW are monitored mainly with
  routers in various AS-es that equipped with IDS technology <x>@03_whereFilter</x>, which
  enables the routers to inspect incoming traffic using mechanisms to be discussed
  in the next sections.

Great amount of effort was put in the SDN, software defined networking, 
  which separated the control plane and the data plane in the routers, 
  this enables the routers to be more flexible and adaptive 
  to the changing conditions of the network, and to be able to update the 
  filtering rules in real-time. <x>@15_SDN</x>

When a packet satisfies the conditions for censorship, the most common case for the
  GFW is to send a TCP RST packet to both ends to terminate the TCP connection on
  both sides <x>@03_whereFilter</x>, triggering a denial-of-service attack prevent a pair
  of endpoints from communicating. Despite this, researchers have found it is 
  possible to *ignore* the spoofed RST packet and still receive the responses
  <x>@06_ignoring</x>.

```bash
iptables -A INPUT -p tcp --tcp-flags RST RST -j DROP
# or using BSD's ipfw
ipfw add 1000 drop tcp from any to me tcpflags rst in
```
[2.1 commands that ignores all RST packets]()

```log
cam(55817) → china(http) [SYN]
china(http) → cam(55817) [SYN, ACK] TTL=41
cam(55817) → china(http) [ACK]
cam(55817) → china(http) GET /?falun HTTP/1.0<cr><lf><cr><lf>
china(http) → cam(55817) [RST] TTL=49, seq=1
china(http) → cam(55817) [RST] TTL=49, seq=1
china(http) → cam(55817) [RST] TTL=49, seq=1
china(http) → cam(55817) HTTP/1.1 200 OK (text/html)<cr><lf> etc
china(http) → cam(55817) . . . more of the web page
cam(55817) → china(http) [ACK] seq=25, ack=2921
china(http) → cam(55817) . . . more of the web page
china(http) → cam(55817) [RST] TTL=49, seq=1461
china(http) → cam(55817) [RST] TTL=49, seq=2921
china(http) → cam(55817) [RST] TTL=49, seq=4381
cam(55817) → china(http) [ACK] seq=25, ack=4381
china(http) → cam(55817) [RST] TTL=49, seq=2921
china(http) → cam(55817) . . . more of the web page
china(http) → cam(55817) . . . more of the web page
cam(55817) → china(http) [ACK] seq=25, ack=7301
china(http) → cam(55817) [RST] TTL=49, seq=5841
...
```
[2.2 Example of ignoring RST packets]()

Note that the wall would also have the capability to track
  the ip addresses and the content of the packets, 
  and potentially the users who initiates the communications.

Also note that this experiment was done in 2006, and the GFW had its behavior
 since then, based on our experiment in 2024, the GFW would still send
 RST packets, but the most times, the rest of the content would be dropped.

Recent standards like HTTP/2 and QUIC would also make the GFW's job harder as 
  the encrypted and multiplexed nature of these transport layer protocols, 
  the wall would use alternative strategies to track the packets, including 
  deep packet inspection and stateful traffic analysis. The increased complexity
  increases chances of false negatives, yielding slightly lower blocking rates.
  We would discuss these findings in next sections.

## III. IP Filtering

IP filtering is the mechanism which has least operational cost and lowest level 
of granularity. <x>@07_taxonomy</x> 
If not administrated properly, 
it would result in high false positive and false negative rates. Where:

- **False Positive**: Legitimate traffic is blocked.
This mostly happens to multiple sites are hosted on a shared IP address,
  when blocking one site, all other sites on the same IP would be blocked
  disregarding the content, causes unnecessary collateral damage, including
  potential impact on the economy and innovation.
- **False Negative**: Site that is deemed to be blocked is not blocked,
This mostly happens when the site is hosted on a dynamic IP address, 
  or the site is hosted on a CDN.

The IP Filtering is applied as first layer of defense, such as sites that have
  a fixed IP address such as google.com or US Senate. 

It is also applied to be a last resort, when other mechanisms fails to block
  the traffic, this is essentially the case to deal with the use of VPNs as
  a circumvention tool, in combination with active probing.

### Implementing IP Blocking

IP blocking, including other filtering mechanisms, is introducing *black hole*
  in the network which drops the packets silently once the ip is recognized as
  a blacklisted ip.

The routers would need a list of banned IP addresses, to adapt to the dynamic
  nature of the internet, the list would be updated periodically, 
  and the routers would be able to update the list in real-time.

One implementation is have a central database with entire list of banned IPs, 
  and with the knowledge of complete network topology within the country, 
  it would distribute the entries to routers in a way that both balances the
  load and ensures consistent filtering. The per-router distribution would also
  rotate the list to avoid bottleneck and synchronization issues.

TCP wrapping is a common method to implement IP blocking, where the routers
  would have a list of rules to block the IP addresses, and the rules would 
  be updated periodically. The rules would be in shape of files in `/etc/hosts.deny`
  and `/etc/hosts.allow` files, constituting a blacklist and whitelist respectively.

![Figure 3-1: Format of rule used in /etc/host file.](res/3.1-etc-host.jpg)

![Figure 3-2: An example rule to show how an IP could be banned.](res/3.2-rule.png)

In this sample entry of `/etc/hosts.deny` file, the rule is to block the IP address
  that is trying to connect to the host. The rule is in shape of `daemon: client: options`
  where the daemon is the process name or wildcard that the client is trying to connect to,
  the client is the address, host name, 
  or wildcard to represent the client attempting to connect. <x>@12_hostman</x>

The rule in figure 3-2 shows the block of IP 100.100.100.100. The rule states 
  that all processes that the IP address tries to connect to should be denied,
  making TCP connection impossible to establish. Adding this rule to the 
  `/etc/host.deny` file would IP block the address 100.100.100.100 within the 
  network

![Figure 3-3: Destination-based Black Hole Filtering with Remote Triggering](res/3-3-cisco.png)

![Figure 3-4: Source-based Black Hole Filtering with Remote Triggering](res/3-4-cisco-source.png)

On the BGP level, the routers are configured to drop the packets from matched
  IP addresses, such black-holed IP addresses involves 3 steps: <x>@13_blackhole_cisco</x> <x>@14_blackhole_rfc</x>
- **Setup (preparation)**: A trigger is a special device that is installed at the NOC 
  (Network Operations Center) exclusively for the purpose of triggering a black hole. 
  The trigger must have an BGP peering relationship with all the edge routers, and is
  configured to redistribute static routes to its BGP peers, sends the static route by
  means of an BGP routing update.

  The Provider Edges (PEs) must have a static route for an unused IP address space. 
  For example, 192.0.2.1/32 is set to Null0 (which is not used as a deployed IP address)

  Loose URPF (Unicast Reverse Path Forwarding) is configured on all external facing 
  interfaces of PEs.

- **Triggering**: When an administrator adds a static route to the trigger, 
  which redistributes the route by sending a BGP update to all its BGP peers, 
  setting the next hop to the target destination address (192.0.2.1 in this case)

  Each PE receives an BGP update and sets its next hop to the source IP to the
  unused IP address space 192.0.2.1. The next hop to this address is set to Null0
  using a static routing entry in the router configuration. The next hop entry
  in the FIB (Forwarding Information Base) is updated to point to Null0.

  All traffic from the source IP will fail the loose URPF check at the PEs and 
  as a consequence will be dropped.

- **Withdrawal**: Once the trigger is in place, all traffic from the source IP
  address will be dropped at the PEs. If it removed from the black list, the
  administrator would manually remove the static route from 
  the triggering device, which sends a BGP route withdrawal to its BGP peers.
  This prompts the edge routers to remove the existing route for the source IP
  that points to 192.0.2.1 and to install a new route in the FIB based on the
   IGP (Interior Gateway Protocol) RIB (Routing Information Base) entry.
  If this new route is successful, loose URPF will pass and traffic will be
  forwarded normally.

This implementation comes with 2 flavors of black hole filtering: 
  Destination-Based and Source-Based. Above describes a source-based approach
  but the destination-based approach is similar, excepting the criteria for 
  black hole a packet is the destination IP address.

In a scenario of, when Alice, a university student, attempting to querying 
  some blacklisted terms through a search engine somewhere outside the country.
  The GFW intercepts the query and may deemed that the search engine
  or the user should be punished. <x>16_censorPerspectives</x>

When encountering the source-based black hole, all packets returned from the 
  search engine would be dropped, and Alice would not be able to receive the
  search results. The search engine would not be aware of the black hole, 
  and would continue to send the packets, which would be dropped at the border.
  Other sites hosted on the same IP address would also be blocked.

When encountering the destination-based black hole, Alice's IP (since she is in
  the university network, the BGP would not recognize Alice's IP over the 
  university's NAT, but take the university's IP address) would be black holed.
  All packets across BGP into that university would be dropped.
  All the sudden her entire dorm would not be able to connect to most of the 
  sites hosted outside of the country, for a while until the administrator 
  removes the ban. (Usually within a short period to avoid too much 
  collateral damage)

The later is often been regarded as a punitive measure, and shifts the blames 
  of entire university's network inaccessibility to Alice for *running into the
  wall*, and would be a warning to other students to avoid similar behavior.
  The practice of *collective punishiment* is not uncommon in China since very
  ancient times to extend the legal actions beyond the individual, and thus 
  strengthen the social control. <x>@17_earlyChina</x>

The blacklist strategy is more common in the GFW, although whitelist strategy is 
  possible and has been used in the past, such that after 2009 Urumqi riots,
  internet access was blocked in Xinjiang in a way that only less than 100 
  local sites, such as banks and local government sites were accessible. <x>@10_missingLink</x>
It lasted for nearly a year, <x>@11_xinjiang</x>
  damages to the economy, society and people's social life were immense.

### Flaws in IP Blocking

In the case of IP blocking, the flaw would lies in their false-positive and 
  false-negative rates, being effective and finale itself, certain oversites
  would defeat its purpose.

Since the IP blocking is limited only to a specific IP address, 
  the earliest circumventing attempts are proxied access, this does not
  necessarily mean using a VPN. Some early circumvention strategies features
  using a proxy over HTTP which is generally supported by most browsers and
  applications. <x>@18_GFWSpaceTime</x>

Also, even for a single site, it is likely to host on CDN and the IP address
  would be dynamic, the GFW would have to update the list frequently to keep
  the site blocked. (Or, block the entire CDN, which the collateral damage 
  would be substantial. As of 2023, many CDN are affected by the GFW. <x>@19_detect</x>)

Being the cheapest option on the list, it essentially is not scalable to 
  block all the sites, to keep up with the dynamic nature of the internet, 
  maintenance of such list would mean a large amount of human resources and 
  technical overhead. Let alone of the indirect economic impacts.

Tools with minimal configuration and cost would be enabled to bypass the IP
  blocking, and the GFW would have to rely on other mechanisms to keep up with
  the circumvention tools.

The mechanism of over-blocking can be abused by groups or individuals 
  to vandalize the network, such as compromising competitor's network via 
  weaponizing censorship infrastructure, constitutes availibility attack;
  or to disrupt the network constantly that the cooperating parties would be
  annoyed, increase dissent and therefore defeat the purpose of the GFW.
  (Although the existence of GFW is self-defeating in the first place)

## IV. DNS Manipulation

Due to the majority of DNS request is over UDP, which tends to lack verification
  and encryption, DNS request are very vulnerable to hijacking and spoofing. 
  <x>@20_dnsUDP</x>

Part of the GFW is consisted of liar DNS servers and DNS hijackers returning 
  incorrect IP addresses. 

At these server DNS, manipulation can be done where instead of returning the 
  actual IP address for a result or relaying it to another DNS server, the 
  servers would reply with either a non-functional IP address or with IP address
  of another website (e.g., leads to the site of local police department).

Evidences suggests that the DNS filtering is keyword based <x>@21_dnsCompreh</x>
, this would implicitly
  solve the problem of mirror url for the same content, and the DNS manipulation
  would be able to block the entire domain, including all subdomains and the 
  content hosted on the domain. 

Because those domestic DNS servers are topologically
  and geographically closer to the users, the uses would be directed to the 
  compromised addresses before the upstream can respond to the query. 

### Implementing DNS Manipulation

Usually, DNS servers stores DNS records for recently accessed hosts in cache.
  If a record is in the cache, then the server would return the record directly
  as there is no need to query the record from the authoritative server again.
<x>@22_dnsConcpets</x> Unless the record is expired, the server would query the
  upstream servers for the record of actual IP address.
  Most general DNS records are in the form of A and AAAA records, which stores
 IPv4 and IPv6 addresses respectively. 

![Figure 4-1: An example of A and AAAA type records](res/4.1-DNS.png)

DNS poisoning happens when the cache inside a server has been modified so that
  it returns the wrong IP address without validating the record with the 
  authoritative server. This is a common attack vector for redirecting users to
  malicious sites, and the GFW uses this mechanism to block the access to 
  certain sites.

![Figure 4-2: An example of DNS cache poisoning](res/4.2-DNS-poisoned.png)

In example with the Figure 4-2, the DNS cache poisoning would be done by 
  directly implanting fake DNS records into the cache of the DNS server.
  With the state-owned ISPs, DNS relay is compromised and the DNS records are
  all poisoned. Existing verification means such as using TTL on the DNS records
  are not effective, compares to a 3rd party DNS spoofing attack, where an 
  individual attacker usually have temporary access to few servers;
  the affected DNS server, and all downstream and domestic servers 
  are state-controlled. Conventional verification is futile. <x>23_dnsMeasure</x>

### Flaws of DNS Manipulation

Avoiding the compromised DNS servers in first place would be the most effective
  way to bypass the DNS manipulation.

For example, systems using systemd would be able to use `systemd-resolved` to
  edit `/etc/systemd/resolved.conf` to add entries that override the default
  DNS. This would allow the system to use a different DNS server that is trusted.
  Note that due to the recursive lookup feature, the DNS server need to ensure
  that all upstream are from clean sources too.

Directly typing the IP addresses in browsers would also bypass DNS manipulation.

Recent moves towards DNS over HTTPS (DoH) and DNS over TLS (DoT) would also 
  make the DNS manipulation harder, as the DNS request and response would be
  encrypted and the upstream DNS server would be able to verify the authenticity
  of the DNS records.

The problems still ensue as
1. The ISP can just deny providing the service to the user who uses DoH/DoT.
2. Opportunities of sidechannel attacks still exist to detect anomalies in the
  encrypted traffic.

Note that the *poison* in the tampered servers infers that, those compromises 
  would have propagating effects due to the recursive nature of DNS lookups.
  In 2012, internet groups has found out that with 39 AS-es in China injecting
  forged DNS replies, open resolvers outside China distributed in 109 countries, 
  suffer some collateral damage from those forged replies,
  3 TLDs affected almost completely (99.53%, domains from within GFW, expected),
  and 11573 resolvers affected (26.4%) or 1 out of 16 unexpected TLDs. 
  (`de xn--3e0b707e kr kp co travel pl no iq hk fi uk jp nz ca`)
  As some paths between resolvers and some TLD (Top Level Domain) name servers 
  transit through (affected) ISPs in China. <x>24_dnsGlobal</x>

![Figure 4-3: The Path where a .de TLD is poisoned](res/4.3-de-poison.png)

Figure 4-3 shows a path where a `.de` TLD is poisoned, via its transit in 
  CN (China) AS 7497 and AS 24151

## V. Survey of Circumvention Tools and Strategies

Various circumvention tools have been developed to bypass the GFW, 
  gradually increasing in complexity, with trade-offs among dimensions of: 
  <x>@07_taxonomy</x>

 1. *Availability*: 
 there is no use of an anti-censorship technology if 
 the target users cannot acces it.

 2. *User-friendliness*: 
 an often under-explored dimension, 
 given the large population of users who are not technology-savvy.
 
 3. *Verifiability*: 
 how can a user verify that the software is not a monitoring tool 
 from the government.
 
 4. *Scope*: how many modes of communication can be covered.

 5. *Security*: this is the most obvious dimension of an anti-censor.

 6. *Deniability*: if caught, how can a user deny her involvement.

 7. *Performance*: how much will throughput and delay be 
degraded by using the anti-censor. 

Those aspects often has trade-offs against each other, 
e.g., scope-deniability vs performance-availablity, userfriendliness-denialibilty.

Just like the censorship system, the anti-censorship tools also comes in
  multiple components targeting different layers of the network stack that
  works together to bypass the censored network, most times corresponding to
  a specific censorship mechanism. 
e.g., secured alternative messaging application to ones raised security concerns
  (e.g., Signal, Telegram, etc), alternative DNS server for DNS manipulation, 
  VPN or HTTP proxy for IP blocking, and encrypted tunnel for DPI and 
  obfuscation for stateful traffic analysis. <x>@48_globalDefeat</x>

![Figure 5-1: Anatomy of anti-censorship system](./res/5.1-anatomy.png)

Note that supporting the circumvention tools might involve out-of-band 
  communication channels such as hidden part of CD/DVD, floppy disks, USB drives, 
  or QR codes, URL and barcodes as printed materials, also includes word-of-mouth.

![Figure 5-2: An interactive communication technology adoption channel](./res/5.2-adoption.png)

As figure 5-2 <x>@49_ICTAdopt</x> addressed the people's adoption of most ICTs,
  it is consisted on the spectrum from micro-individual to macro-social poles:
  with factors in aspects of *system, technology, audience, social, use, adoption*

![Figure 5-3: A roadmap for understanding the use of circumvention tools](./res/5.3-circumventAdopt.png)

Similar pattern (figure 5-3) <x>@47_circumvention</x> would apply to 
  circumvention tool use, with more weights on
  how each other factors would affect the usage, then usage would also have
  a reciprocal effect on the other factors. 

Noting that each of those factors are important in their own right,
  the scope of rest of this paper would focus on the technical aspects of the 
  circumvention tools, and how they interact with the GFW.

There are three commonly used circumvention methods that works together to 
  bypass the GFW: <x>@07_taxonomy</x>
- HTTP/CGI proxy
- VPN (IP tunneling)
- Re-routing
- Distributed Hosting

### HTTP/CGI Proxy

HTTP proxying sends HTTP requests through an intermediate proxying server, the
  client would send exact same HTTP request to the proxy as if it is send to 
  the target server. The proxy server may re-parse the HTTP request, and sends
  its own HTTP request to the target server, then returns the response back to
  the proxy client.

CGI proxy, on the other hand, is a bit obfuscated, it uses a server-side script
  to perform proxying function. A CGI proxy client sends the requested URL as 
  payload embedded in the data portion of an HTTP request to CGI server. 

The CGI server then pull out the intended target information from the payload,
  and sends the request to the target server, then returns the response back to
  the CGI client.

Those are seen more primarily in the older systems, such as *Freenet* (1999),
  *UltraSurf* (2002), *DynaWeb* (2002), which mainly uses HTTP proxy, and 
  *Psiphon* (2006) which uses CGI proxy.

Each has its own issues such as limitations and dependencies on the client 
  software, and it was designed when GFW was at its infancy, later added features
  (such as addition of DNS hijacking) would render these tools ineffective.

The ideas of HTTP/CGI still leaves a legacy in more modern circumvention tools.

### Re-routing and Distributed Hosting

Re-routing systems route data through a **series** of proxying servers,
  encrypting the data again at each proxy, so that a given proxy knows at most
  either where the traffic came from or where it is going to, but not both.

*Tor* (2002) is the most famous and long-lived re-routing system, 
  which routes the data through a series of volunteer-run nodes.

However, *Tor* is not a accessible in China since around 2015, it is lately
  known that TOR's obfuscation leaves a fingerprint that is possible to be 
  detected. (e.g., fixed packet size, fixed packet interval, etc) Even it is
  hard to decipher the content of the packets, the fact that it is a TOR packet
  is enough for such traffic to be blocked. <x>@50_tor_finger</x> The SDN 
  topology of the GFW has separate nodes that is each specialized at high-efficency
  relaying and traffic analysis. Similar to the remotely triggered black hole
  model, the analysing node triggers the blocking node to block the TOR traffic.

Usability and QoS issues also makes the TOR less attractive to the general
  public, combined with social engineering and legal repercussions as deterrents,
  users are less likely to participate in the TOR network. 

On the other hand, the GFW is scaling up by with increasing control 
  and higher computation power, with the decreasing number of participants 
  in P2P networks, it is not surprising that the peers in such networks are
  likely to be state-controlled.

Distributed hosting is more of a server effort than client effort, and usually
  for the purpose of mitigating DDoS and single-point-of-failure issues.
  It is effective against ip blocking.

However, depending on the CDN, there are cases that entire CDN is blocked 
  by the GFW, such as github was not blocked but had connection issues in China, 
  as its CDN, *Fastly*, was blocked. <x>@19_detect</x> 

At another case, CDNs might chose to practice geo-block due to economical
  pressures. In 2018 researchers found that many CDNs had server-side blocking
  with China, Iran, Syria, Sudan, Cuba, Russia and North Korea. <x>@51_CDN403</x>

### Tunneling

As of today, the most common circumvention tool is use of a VPN tunneling with
  self-hosted VPS or commercial shared VPN services.

The user usually purchases a (virtual) machine in a place without censorship
that can be accessed remotely, and installs tunneling software 
(e.g., openSSH, etc.) on the machine. 

For each session where the user needs to access uncensored content, the user 
  would establish a SSH connection to the machine, and set up a SOCKS proxy 
  on the client side, then redirect partial or all of the traffic through the
  tunnel to their VPS, then the VPS would relay the request and response between
  the client and the target server as a bridge.

It is by far the most effective and secure way to bypass the GFW, since the 
  VPS tends to be privately owned, and the traffic is encrypted end-to-end.
  Taking down one VPS would only affect the users on that VPS, but it is 
  relatively cheap to switch to another VPS, compares to the increasing overhead
  of the GFW to block all the VPS-es.

The only known way for the GFW to confronting it is to perform deep packet 
  inspection and traffic analysis that recognizes the traffic pattern within 
  the tunnel, and block the source IP address of the VPS. 
  (we are excluding the social-engineering and legal repercussions in this paper)

**Shadowsocks** (2012) is an open-sourced SOCKS protocols framework that is 
  popular in China. It has many open-source forks in various languages 
  <x>@36_SS-py</x> <x>@37_SS-rust</x> <x>38_SS-go</x> <x>39_SS-cs</x>.  
According to a research survey in July 2015, of 371 faculty members and students 
  from Tsinghua Univer-sity, 21% used Shadowsocks to bypass censorship in China.
<x>@50_googleScholar</x>

The popularity of Shadowsocks stems from its simplicity. 
Its lightweight design imposes minimal overhead on proxied traffic and 
  makes it easy to implement on a variety of platforms. 
A large, profit-incentivized proxy reseller market, 
  as well as numerous tutorials and one-click installation scripts, 
  have reduced the difficulty of installing and using Shadowsocks, 
  and made it popular even among non-technical users.

Researchers found that the GFW utilizes an *active probing* mechanism to 
  detect the Shadowsocks traffic, and then block the source IP addresses of 
  the VPS-es. <x>@40_SSProbe</x> Still, the blocks are only anecdotally reported,
  and continuing development of Shadowsocks to improve its encryption and 
  obfuscation with community efforts. 

refs:
    
```bib
@online{01_hist1986,
  title = "The Internet Timeline of China 1986-2003",
  author = "China Internet Network Information Center (CNNIC)",
  URL = "https://web.archive.org/web/20240119091818/https://www.cnnic.com.cn/IDR/hlwfzdsj/201306/t20130628_40563.htm",
  date = {2012-06-28},
  accessed = {2024-01-19}
}

@online{02_stat2022,
  title = "The 50th Statistical Report on China's Internet Development",
  author = "China Internet Network Information Center (CNNIC)",
  date = {2022-12},
  URL = "https://web.archive.org/web/20231117230017/https://www.cnnic.com.cn/IDR/ReportDownloads/202212/P020230829504457839260.pdf",
  accessed = "2023-11"
}

@InProceedings{03_whereFilter,
  author="Xu, Xueyang
  and Mao, Z. Morley
  and Halderman, J. Alex",
  editor="Spring, Neil
  and Riley, George F.",
  title="Internet Censorship in China: Where Does the Filtering Occur?",
  booktitle="Passive and Active Measurement",
  year="2011",
  publisher="Springer Berlin Heidelberg",
  address="Berlin, Heidelberg",
  pages="133--142",
  abstract="China filters Internet traffic in and out of the country. In order to circumvent the firewall, it is helpful to know where the filtering occurs. In this work, we explore the AS-level topology of China's network, and probe the firewall to find the locations of filtering devices. We find that even though most filtering occurs in border ASes, choke points also exist in many provincial networks. The result suggests that two major ISPs in China have different approaches placing filtering devices.",
  isbn="978-3-642-19260-9",
  URL="https://doi.org/10.1007/978-3-642-19260-9_14"
}

@report{04_digitalEco_mypaper,
  title = "Analysis on China's Digital Ecosystem and its Implications for the Modern World",
  author = "Jia, M., Vassileva, J.",
  institution = "University of Saskatchewan",
  type = "CMPT412 report",
  date = "2023-12",
  note = "This is my paper for CMPT412 (unpublished)"
}
    
@article{05_conceptDoppler,
  title={ConceptDoppler: a weather tracker for internet censorship.},
  author={Crandall, Jedidiah R and Zinn, Daniel and Byrd, Michael and Barr, Earl T and East, Rich},
  journal={CCS},
  volume={7},
  pages={352--365},
  year={2007},
  URL="https://doi.org/10.1145/1315245.1315290"
}

@inproceedings{06_ignoring,
  title={Ignoring the great firewall of china},
  author={Clayton, Richard and Murdoch, Steven J and Watson, Robert NM},
  booktitle={International Workshop on Privacy Enhancing Technologies},
  pages={20--35},
  year={2006},
  organization={Springer},
  URL="https://doi.org/10.1007/11957454_2"
}
    
@inproceedings{07_taxonomy,
  title={A taxonomy of Internet censorship and anti-censorship},
  author={Leberknight, Christopher S and Chiang, Mung and Poor, Harold Vincent and Wong, Felix},
  booktitle={Fifth International Conference on Fun with Algorithms},
  pages={52--64},
  year={2010},
  URL="https://www.princeton.edu/~chiangm/anticensorship.pdf"
}


@inproceedings{08_poisoning,
  title={Poisoning the well: Exploring the great firewall's poisoned dns responses},
  author={Farnan, Oliver and Darer, Alexander and Wright, Joss},
  booktitle={Proceedings of the 2016 ACM on Workshop on Privacy in the Electronic Society},
  pages={95--98},
  year={2016},
  URL="https://doi.org/10.1145/2994620.2994636"
}

@article{09_securityTrend,
  title={IT Security in the USA, Japan and China},
  author={Ahlgren, Martin and Breidne, Magnus and Hektor, A},
  journal={A Study of Initiatives and Trends within},
  year={2005},
  publisher={Citeseer},
  URL="https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=dd87273d88927ca989937673e210a94a8186b6a4"
}

    
@article{10_missingLink,
  title={The Missing Link},
  author={Jia, Cui},
  journal={China Daily},
  year={2009},
}
    
@online{11_xinjiang,
  title={Xinjiang Internet Restored Starting Today},
  URL="web.archive.org/web/20100517023849/http://news.china.com.cn/txt/2010-05/14/content_20038848.htm",
  date={2010-05-14}
}
    
@commentary{12_hostman,
  title={hosts.den(5) - Linux},
  author={Wietse Venema},
  URL="https://linux.die.net/man/5/hosts.deny"
}

@manual{13_blackhole_cisco,
  title={remotely triggered black hole filtering - Destination-Based and Source-Based},
  author={Cisco Systems, Inc.},
  organization={Cisco Systems, Inc.},
  year={2005},
  URL="https://www.cisco.com/c/dam/en_us/about/security/intelligence/blackhole.pdf"
}
    
@standard{14_blackhole_rfc,
  title={RFC 5635 - Remote Triggered Black Hole filtering with uRPF},
  author={Kumari, W},
  year={2009},
}

@article{15_SDN,
  author = {Benzekki, Kamal and El Fergougui, Abdeslam and Elbelrhiti Elalaoui, Abdelbaki},
  title = {Software-defined networking (SDN): a survey},
  journal = {Security and Communication Networks},
  volume = {9},
  number = {18},
  pages = {5803-5833},
  keywords = {software-defined networking, SDN challenges, SDN issues, SDN architecture, OpenFlow},
  doi = {https://doi.org/10.1002/sec.1737},
  url = {https://onlinelibrary.wiley.com/doi/abs/10.1002/sec.1737},
  eprint = {https://onlinelibrary.wiley.com/doi/pdf/10.1002/sec.1737},
  abstract = {Abstract With the advent of cloud computing, many new networking concepts have been introduced to simplify network management and bring innovation through network programmability. The emergence of the software-defined networking (SDN) paradigm is one of these adopted concepts in the cloud model so as to eliminate the network infrastructure maintenance processes and guarantee easy management. In this fashion, SDN offers real-time performance and responds to high availability requirements. However, this new emerging paradigm has been facing many technological hurdles; some of them are inherent, while others are inherited from existing adopted technologies. In this paper, our purpose is to shed light on SDN related issues and give insight into the challenges facing the future of this revolutionary network model, from both protocol and architecture perspectives. Additionally, we aim to present different existing solutions and mitigation techniques that address SDN scalability, elasticity, dependability, reliability, high availability, resiliency, security, and performance concerns. Copyright © 2017 John Wiley \& Sons, Ltd.},
  year = {2016}
}
```
