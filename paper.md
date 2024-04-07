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

### Novel Protocols

Recent developments and adoption of novel protocols such as QUIC, HTTP/3 and IPv6
  would also make the GFW's job harder, as the multiplexed and encrypted nature
  in the transport and network layer make it harder to track traffic states, 
  also increases the problem space exponentially for rule-based filtering.

#### QUIC+HTTP/3

**QUIC** (Quick UDP Internet Connections) is a transport layer protocol was designed
  to provide secure, consistent, and low-latency connections over the internet.
  A secure connection resembling DTLS (Datagram Transport Layer Security) is
  running atop UDP and multiplexes multiple streams over the same connection.
<x<>@54_QUIC</x>

**HTTP/3** is the 3rd major version of HTTP used to exchange information on the
  WWW. HTTP/3 is based on QUIC, and is designed to improve the performance of
  the web, especially for mobile devices. <x>@55_HTTP3</x>

The introduction of HTTP/3 (HTTP over QUIC) yields promising expectations 
  to counteract such interference, due to its novelty, build-in encryption, 
  and faster connection establishment. QUIC's methods of IP address discovery
  should also make it harder to spoof. 

Researchers in 2020 conducted a study on the effectiveness of the GFW to block
  QUIC traffic, found that for AS-es which has high failure rate on TCP connections
  (TCP-handshake timeout and connection reset), yields a significantly lower 
  failure rate on QUIC connections. <x>@53_quicBlock</x>

The same research also suggests that IP blocking is the primary causation of the
  failing QUIC connections. Since the censor does not exclusively apply IP 
  endpoint blocking, hosts that are targeted by a different form of HTTPS 
  censorship are still available over QUIC.

The adaptation of IPv6 would also make the GFW's job harder, as the IPv6 address
  space is significantly larger than IPv4, and the GFW would have higher
  operational overhead to keep adding more entries in the blacklist, and also
  meant easier allocation for individual users to have their more VPSs 
  in the IPv6 space.

## VI. Keyword Filtering, Deep Packet Inspection Stateful Traffic Analysis and Evolution of GFW

Where conventional blocking criteria are based on protocol's header information,
  GFW evolves to use novel methods that inspect and analysis the payloads as well.

When deemed that the cost of false negative for oversighting of information is 
  more emergent issue than potential loss of false positives, but more are the
  cases that different parties of a authoritarian government fails to coordinate
  with each other. Such as Iran blocks all UDP traffic. <x>@53_quicBlock</x>

Various advanced methods are applied with the SDN topology of the GFW, which
  separates the concerns for high-efficiency relaying blocking (packet drop)
  nodes (data panel), and mirrors the traffic to real-time traffic 
  analysis nodes (control panel), those nodes then signal instructions to the 
  data nodes to block the traffic or not. 

However, various traffic analysis is only been found in the case where a node
  relays the traffic 

### Fingerprinting

For some encrypted and obfuscated traffic, such as TOR, even the payload are not
  easily readable, the traffic would still have side-channel vulnerabilities,
  such as the fixed packet size, fixed packet interval, and entropy of the 
  packets. <x>@50_tor_finger</x> <x>@56_gfwTOR</x> <x>@57_blockingTor</x>

In particular, Tor (in 2012), has a TLS client `hello` in the Tor cipher list, 
  with the purpose of initiate a TLS connection.

```log
c0 0a c0 14 00 39 00 38 c0 0f c0 05 00 35 c0 07
c0 09 c0 11 c0 13 00 33 00 32 c0 0c c0 0e c0 02
c0 04 00 04 00 05 00 2f c0 08 c0 12 00 16 00 13
c0 0d c0 03 fe ff 00 0a 00 ff
```
**Listing 1.1.** Tor cipher list inside TLS client hello.

It is one of the few unique features of Tor traffic that can be used to 
  fingerprint the traffic, and the GFW uses this feature to locate Tor traffic.

Fingerprinting is also the mechanisms where the routers recognizes anomalies
  in the traffic pattern, <x>@26_gfwSS2020</x> once this is detected, probes 
  will be send to server or the client for further attack gaining more data
  to confirm the anomaly, and then block the traffic. (usually done in a
  **replay attack**)

### Active Probing and evolutions

In case of Shadowsocks, the GFW uses an active probing mechanism to detect the
  Shadowsocks traffic, and then block the source IP addresses of the VPS-es.

The study in 2020 set up sets up series of experiments which attracts
 the active probs from GFW.
They studied the behaviors and fingerprinted certain traits
 of probes sent by GFW, including IP origin, AS origin, and TTL values.

Compares to similar study done in 2015 <x>@46_gfw-discover</x>, there are few
  more traits that indicates the wall became stronger during the 5 years, in
  terms of the active prober, which it is less likely to be recognized as a 
  probe, and more likely to be recognized as a normal request.

| Trait | 2015 | 2020 |
|-------|------|------|
| IP ID | no pattern | no pattern |
| TTL | centered around 47-50 | centered around 46-50  |
| TCP source ports | distribute across 16 bit space, including under 1024 | 90% around 32768-60999, none below 1024 |
| TCP timestamp (from SYN) | 250 Hz - 1 kHz | 250 Hz - 1 kHz |
| **Source IP** | **16464 probes with 15249 addresses** | **51837 probes with 12300 addresses** |
| **replays** | **Little variation** | **systematic variation** |

Noticibly, the two source IP has a very small overlap (167 out of 3500), suggesting
  possible more dense probing nodes, or the GFW is gradually increasing the number
  of probing nodes.

Also, in terms of **replay attacks**, where nearly no variation is found in the 2015
  study, the 2020 study found a systematic variation in the replay attacks, 
  in ways that changing certain bits as a trait group for OutlineVPN and other
  set of change for ShadowSocks and for Tor. Also, there is a noticeable **state-
  transition** in which types of the replay attacks are used.

The study also found that **packet length** as a fingerprinting trait to identify
  ShadowSocks and other tunneling traffic, as well that a high-entropy payload
  is more likely to gain attention, as a packets with a per-byte entropy of 7.2
  is 4 times more likely to be probed compared to a packet with entropy of 3.0.

By analyzing the behavior before and after an VPS is blocked by the
 wall, there is a theory of machine-scanned registry of the VPS black-list (gray?)
and human-administrated block-lists, and unblockings.

An adversary model is proposed:

1. GFW passively recognizes certain traffic pattern are likely to be 
   circumvention tools,

2. Probes are sent to the suspected VPS for confirmation, 
   primiarly uses the Replay.

3. The VPS are likely to be marked on *Black/Gray List*, 
   and faced blocking by adminstrators at any future time.

### Social Engineering 

While in the attempt of demystifying the GFW, we might be underestimating or
  overestimating the technical capabilities of GFW, it is entirely feasible 
  that the deep-learning based traffic analysis is just a rumor, as the GFW
  tends to rely on the state-controlled violence and social engineering to 
  achieve the same goal.

Around the same time where the developer and maintainer posted the possible
  solutions to redirect the replay attacks to other http sites, 
  and improving the cipher being used, and ready to deploy the new version
  of the Shadowsocks, the state police found the maintainer and the repo was
  *voluntarily* deleted by the maintainer, *clowwindy@GitHub*. <x>@58_breakwaIssue</x>

Again, as many would argue, the GFW is not a technical problem, but a social
  problem, and the social engineering is the most effective way to defeat the
  GFW, as the GFW is not a monolithic entity, but a collection of individuals
  with different goals and motivations.

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

@techreport{16_censorPerspectives,
  author =	 {Khattak, Sheharbano},
  title = 	 {{Characterization of Internet censorship from multiple
         	   perspectives}},
  year = 2017,
  month = jan,
  url = {https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-897.pdf},
  institution = {University of Cambridge, Computer Laboratory},
  doi = {10.48456/tr-897},
  number = {UCAM-CL-TR-897}
}

@book{17_earlyChina,
  title={Sanctioned violence in early China},
  author={Lewis, Mark Edward},
  year={1989},
  publisher={State University of New York Press}
}

@article{18_GFWSpaceTime,
  author = {Ensafi, Roya and Winter, Philipp and Mueen, Abdullah and Crandall, Jedidiah},
  year = {2015},
  month = {04},
  pages = {},
  title = {Analyzing the Great Firewall of China Over Space and Time},
  volume = {1},
  journal = {Proceedings on Privacy Enhancing Technologies},
  doi = {10.1515/popets-2015-0005}
}

@inproceedings {19_detect,
  author = {Mingshi Wu and Jackson Sippe and Danesh Sivakumar and Jack Burg and Peter Anderson and Xiaokang Wang and Kevin Bock and Amir Houmansadr and Dave Levin and Eric Wustrow},
  title = {How the Great Firewall of China Detects and Blocks Fully Encrypted Traffic},
  booktitle = {32nd USENIX Security Symposium (USENIX Security 23)},
  year = {2023},
  isbn = {978-1-939133-37-3},
  address = {Anaheim, CA},
  pages = {2653--2670},
  url = {https://www.usenix.org/conference/usenixsecurity23/presentation/wu-mingshi},
  publisher = {USENIX Association},
  month = aug
}

@misc{20_dnsUDP,
    series =    {Request for Comments},
    number =    1035,
    howpublished =  {RFC 1035},
    publisher = {RFC Editor},
    doi =       {10.17487/RFC1035},
    url =       {https://www.rfc-editor.org/info/rfc1035},
    author =    {},
    title =     {{Domain names - implementation and specification}},
    pagetotal = 55,
    year =      1987,
    month =     nov,
    abstract =  {This RFC is the revised specification of the protocol and format used in the implementation of the Domain Name System. It obsoletes RFC-883. This memo documents the details of the domain name client - server communication.},
}

@inproceedings{21_dnsCompreh,
  title={Towards a Comprehensive Picture of the Great Firewall's DNS Censorship},
  author={},
  booktitle={IEEE Symposium on Foundations of Computational Intelligence},
  year={2014},
  url={https://api.semanticscholar.org/CorpusID:8688873}
}

@misc{22_dnsConcepts,
    series =    {Request for Comments},
    number =    1034,
    howpublished =  {RFC 1034},
    publisher = {RFC Editor},
    doi =       {10.17487/RFC1034},
    url =       {https://www.rfc-editor.org/info/rfc1034},
    author =    {},
    title =     {{Domain names - concepts and facilities}},
    pagetotal = 55,
    year =      1987,
    month =     nov,
    abstract =  {This RFC is the revised basic definition of The Domain Name System. It obsoletes RFC-882. This memo describes the domain style names and their used for host address look up and electronic mail forwarding. It discusses the clients and servers in the domain name system and the protocol used between them.},
}

@misc{23_dnsMeasure,
      title={How Great is the Great Firewall? Measuring China's DNS Censorship}, 
      author={Nguyen Phong Hoang and Arian Akhavan Niaki and Jakub Dalek and Jeffrey Knockel and Pellaeon Lin and Bill Marczak and Masashi Crete-Nishihata and Phillipa Gill and Michalis Polychronakis},
      year={2021},
      eprint={2106.02167},
      archivePrefix={arXiv},
      primaryClass={cs.CR}
}

@article{24_poisoning2012,
  author = {Anonymous},
  title = {The collateral damage of internet censorship by DNS injection},
  year = {2012},
  issue_date = {July 2012},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  volume = {42},
  number = {3},
  issn = {0146-4833},
  url = {https://doi.org/10.1145/2317307.2317311},
  doi = {10.1145/2317307.2317311},
  abstract = {Some ISPs and governments (most notably the Great Firewall of China) use DNS injection to block access to "unwanted" websites. The censorship tools inspect DNS queries near the ISP's boundary routers for sensitive domain keywords and inject forged DNS responses, blocking the users from accessing censored sites, such as twitter and facebook. Unfortunately this causes collateral damage, affecting communication beyond the censored networks when outside DNS traffic traverses censored links. In this paper, we analyze the causes of the collateral damages and measure the Internet to identify the injecting activities and their effect. We find 39 ASes in China injecting forged DNS replies. Furthermore, 26 of 43,000 measured open resolvers outside China, distributed in 109 countries, may suffer some collateral damage from these forged replies. Different from previous work that considers the collateral damage being limited to queries to root servers (F, I, J) located in China, we find that most collateral damage arises when the paths between resolvers and some TLD name servers transit through ISPs in China.},
  journal = {SIGCOMM Comput. Commun. Rev.},
  month = {jun},
  pages = {21–27},
  numpages = {7},
  keywords = {collateral damage, dns, great firewall of china, internet censorship, internet measurement, packet injection}
}

@inproceedings{25_evolCensor,
  author = {Waheed, Asim and Qunaibi, Sara and Barradas, Diogo and Weinberg, Zachary},
  title = {Darwin's Theory of Censorship: Analysing the Evolution of Censored Topics with Dynamic Topic Models},
  year = {2022},
  isbn = {9781450398732},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://doi.org/10.1145/3559613.3563206},
  doi = {10.1145/3559613.3563206},
  abstract = {We present a statistical analysis of changes in the Internet censorship policy of the government of India from 2016 to 2020. Using longitudinal observations of censorship collected by the ICLab censorship measurement project, together with historical records of web page contents collected by the Internet Archive, we find that machine classification techniques can detect censors' reactions to events without prior knowledge of what those events are. However, gaps in ICLab's observations can cause the classifier to fail to detect censored topics, and gaps in the Internet Archive's records can cause it to misidentify them.},
  booktitle = {Proceedings of the 21st Workshop on Privacy in the Electronic Society},
  pages = {103–108},
  numpages = {6},
  keywords = {censorship, internet measurement, topic modelling},
  location = {Los Angeles, CA, USA},
  series = {WPES'22}
}

@inproceedings{26_gfwSS2020,
  author = {Alice and Bob and Carol and Beznazwy, Jan and Houmansadr, Amir},
  title = {How China Detects and Blocks Shadowsocks},
  year = {2020},
  isbn = {9781450381383},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://doi.org/10.1145/3419394.3423644},
  doi = {10.1145/3419394.3423644},
  abstract = {Shadowsocks is one of the most popular circumvention tools in China. Since May 2019, there have been numerous anecdotal reports of the blocking of Shadowsocks from Chinese users. In this study, we reveal how the Great Firewall of China (GFW) detects and blocks Shadowsocks and its variants. Using measurement experiments, we find that the GFW uses the length and entropy of the first data packet in each connection to identify probable Shadowsocks traffic, then sends seven different types of active probes, in different stages, to the corresponding servers to test whether its guess is correct.We developed a prober simulator to analyze the effect of different types of probes on various Shadowsocks implementations, and used it to infer what vulnerabilities are exploited by the censor. We fingerprinted the probers and found differences relative to previous work on active probing. A network-level side channel reveals that the probers, which use thousands of IP addresses, are likely controlled by a set of centralized structures.Based on our gained understanding, we present a temporary workaround that successfully mitigates the traffic analysis attack by the GFW. We further discuss essential strategies to defend against active probing. We responsibly disclosed our findings and suggestions to Shadowsocks developers, which has led to more censorship-resistant tools.},
  booktitle = {Proceedings of the ACM Internet Measurement Conference},
  pages = {111–124},
  numpages = {14},
  keywords = {Great Firewall of China, Shadowsocks, active probing, censorship circumvention},
  location = {Virtual Event, USA},
  series = {IMC '20}
}

@INPROCEEDINGS{27_censorWeapon,
  author={Bock, Kevin and Bharadwaj, Pranav and Singh, Jasraj and Levin, Dave},
  booktitle={2021 IEEE Security and Privacy Workshops (SPW)}, 
  title={Your Censor is My Censor: Weaponizing Censorship Infrastructure for Availability Attacks}, 
  year={2021},
  volume={},
  number={},
  pages={398-409},
  keywords={Privacy;Open Access;Weapons;Conferences;Censorship;Internet;Security;censorship;denial of service;weaponizing;availability attacks},
  doi={10.1109/SPW53761.2021.00059}
}

@inproceedings {28_GFWEncrypt2023,
  author = {Mingshi Wu and Jackson Sippe and Danesh Sivakumar and Jack Burg and Peter Anderson and Xiaokang Wang and Kevin Bock and Amir Houmansadr and Dave Levin and Eric Wustrow},
  title = {How the Great Firewall of China Detects and Blocks Fully Encrypted Traffic},
  booktitle = {32nd USENIX Security Symposium (USENIX Security 23)},
  year = {2023},
  isbn = {978-1-939133-37-3},
  address = {Anaheim, CA},
  pages = {2653--2670},
  url = {https://www.usenix.org/conference/usenixsecurity23/presentation/wu-mingshi},
  publisher = {USENIX Association},
  month = aug
}

@online{29_SS2020,
  author = {Anonymous and Anonymous and  Anonymous and David Fifield and Amir Houmansadr},
  title={{How China Detects and Blocks Shadowsocks}},
  year = 2019,
  month = dec,
  url = {https://gfw.report/blog/gfw_shadowsocks},
  lastaccessed = {September 2020},
}

@article{30_SSActiveProb,
  title={{ACER}: detecting {Shadowsocks} server based on active probe technology},
  author={Jiaxing Cheng and Ying Li and Cheng Huang and Ailing Yu and Tao Zhang},
  journal={Journal of Computer Virology and Hacking Techniques},
  volume={16},
  number={3},
  pages={217--227},
  year={2020},
  publisher={Springer},
  url = {https://link.springer.com/article/10.1007/s11416-020-00353-z},
}

@online{31_SSRedirectStream,
  author = {Zhiniang Peng},
  title = {Redirect attack on {Shadowsocks} stream ciphers},
  year = {2020},
  publisher = {GitHub},
  journal = {GitHub repository},
  url = {https://github.com/edwardz246003/shadowsocks},
  lastaccessed = {June 2020},
}

@online{32_obfs2Probe,
  title = {{GFW} actively probes obfs2bridges},
  author = {Philipp Winter},
  year = {2013},
  month = mar,
  url = {https://bugs.torproject.org/8591},
  lastaccessed = {February 2020},
}


@inproceedings{33_VPNGate,
  author = {Daiyuu Nobori and Yasushi Shinjo},
  booktitle = {Networked Systems Design and Implementation},
  publisher = {USENIX},
  title = {{VPN Gate}: A Volunteer-Organized Public {VPN} Relay System with Blocking Resistance for Bypassing Government Censorship Firewalls},
  year = {2014},
  url = {https://www.usenix.org/system/files/conference/nsdi14/nsdi14-paper-nobori.pdf},
}


@inproceedings{34_yourState,
  author = {Wang, Zhongjie and Cao, Yue and Qian, Zhiyun and Song, Chengyu and Krishnamurthy, Srikanth V.},
  title = {Your State is Not Mine: A Closer Look at Evading Stateful Internet Censorship},
  year = {2017},
  isbn = {9781450351188},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://doi.org/10.1145/3131365.3131374},
  doi = {10.1145/3131365.3131374},
  booktitle = {Proceedings of the 2017 Internet Measurement Conference},
  pages = {114–127},
  numpages = {14},
  keywords = {the great firewall of china, INTANG, TCP, traffic manipulation, censorship circumvention},
  location = {London, United Kingdom},
  series = {IMC ’17}
}

@online{35_SS-rustn,
  author = {zonyitoo},
  title = {Shadowsocks-rust v1.8.5},
  url = {https://github.com/shadowsocks/shadowsocks-rust/releases/tag/v1.8.5},
}

@online{36_SS-py,
  author = {{Shadowsocks developers}},
  title = {{Shadowsocks}},
  url = {https://github.com/shadowsocks/shadowsocks/tree/master},
}

@online{37_SS-rust,
  author = {Shadowsocks-rust developers},
  title = {Shadowsocks-rust},
  url = {https://github.com/shadowsocks/shadowsocks-rust},
}

@online{38_SS-go,
  author = {go-Shadowsocks2 developers},
  title = {go-Shadowsocks2},
  url = {https://github.com/shadowsocks/go-shadowsocks2},
}

@online{39_SS-cs,
  author = {shadowsocksr-csharp developers},
  title = {shadowsocksr-csharp},
  url = {https://github.com/shadowsocksrr/shadowsocksr-csharp},
}

@online{40_v2ray,
  author = {{V2Ray developers}},
  title = {{V2Ray}},
  url = {https://github.com/v2fly/v2ray-core},
}

@online{41_lshadowsocks,
  author = {{Shadowsocks developers}},
  title = {{Shadowsocks-libev}},
  url = {https://github.com/shadowsocks/shadowsocks-libev},
}


@inproceedings{42_torBridgeBlock,
  author = {Arun Dunna and Ciar\'{a}n O'Brien and Phillipa Gill},
  title = {Analyzing {China}'s Blocking of Unpublished {Tor} Bridges},
  booktitle = {Free and Open Communications on the Internet},
  publisher = {USENIX},
  year = {2018},
  url = {https://www.usenix.org/system/files/conference/foci18/foci18-paper-dunna.pdf},
}

@mastersthesis{43_MPTCP,
  title = {Multipath {TCP} with real Smartphone applications},
  author = {Baerts, Matthieu and De Coninck, Quentin},
  school = {Université catholique de Louvain},
  year = 2015,
  url = {https://dial.uclouvain.be/memoire/ucl/object/thesis:366},
}

@ARTICLE{44_DPI-SS,
  author = {Xuemei Zeng and Xingshu Chen and Guolin Shao and Tao He and Zhenhui Han and Yi Wen and Qixu Wang},
  journal = {IEEE Access},
  title = {Flow Context and Host Behavior Based {Shadowsocks}'s Traffic Identification},
  year = {2019},
  volume = {7},
  number = {},
  pages = {41017-41032},
  keywords = {Big Data;cloud computing;computer network security;telecommunication traffic;large-scale network environment;host behavior;SS flows;cloud virtual private server services;cloud service providers;public cloud platforms;novel SS detection method;Data Transparent Transmission;Invisible Negotiation Process;local network administrators;darknet transactions;cyber-attacks;illegal network activities;illegal network activists;anonymous proxy solutions;anonymous proxy services;flow context;SS traffic;experimental data sets;big data platform;NTCI-BDP data platform;VPSs;campus network;SS running environment;association techniques;unassociated domain names;flow burst;detection model;hosts;12-dimensional features;Protocols;Feature extraction;Cloud computing;Servers;Encryption;Big Data;Big Data association;cloud-based anonymous proxy;flow burst;flow context;host behavior;traffic identification;shadowsocks},
  doi = {10.1109/ACCESS.2019.2907149},
  ISSN = {2169-3536},
}

@inproceedings{45_seeObfs,
  author = {Liang Wang and Kevin P. Dyer and Aditya Akella and Thomas Ristenpart and Thomas Shrimpton},
  title = {Seeing through Network-Protocol Obfuscation},
  booktitle = {Computer and Communications Security},
  year = {2015},
  publisher = {ACM},
  url = {http://pages.cs.wisc.edu/~liangw/pub/ccsfp653-wangA.pdf},
}

@inproceedings{46_gfw-discover,
  author = {Roya Ensafi and David Fifield and Philipp Winter and Nick Feamster and Nicholas Weaver and Vern Paxson},
  title = {Examining How the {Great Firewall} Discovers Hidden Circumvention Servers},
  booktitle = {Internet Measurement Conference},
  publisher = {ACM},
  year = {2015},
  url = {http://conferences2.sigcomm.org/imc/2015/papers/p445.pdf},
}

@article{47_circumvention,
  author = {Yi Mou and Kevin Wu and David Atkin},
  title ={Understanding the use of circumvention tools to bypass online censorship},
  journal = {New Media \& Society},
  volume = {18},
  number = {5},
  pages = {837-856},
  year = {2016},
  doi = {10.1177/1461444814548994},
  URL = { https://doi.org/10.1177/1461444814548994},
  eprint = { https://doi.org/10.1177/1461444814548994},
  abstract = { Circumvention tools designed to bypass online censorship—such as simple web proxies, virtual private network service, and so on—are frequently used in countries whose governments impose heavy Internet censorship. Around 18 million Internet users in China are currently using those tools to bypass the Great Firewall and access unblocked online content. In a pioneering empirical investigation of unblocked information seeking in China’s censored online environment, the present study systematically examines a wide range of macro-social and micro-individual factors which affect the use of circumvention tools to bypass Internet censorship under the guidance of the interactive communication technology adoption model. The results reveal that, with the exception of social trust, macro-social factors have only a modest influence on the use of circumvention tools. In contrast, micro-individual-level variables—including perceived technology fluidity, gratifications, and selected demographic variables—play a much larger role in our multivariate model. Theoretical and practical implications are discussed. }
}

@article{48_globalDefeat,
  title={Defeat Internet censorship: overview of advanced technologies and products},
  author={Global Internet Freedom Consortium and others},
  journal={GIFC, White Paper},
  year={2007}
}

@article{49_ICTAdopt,
  author = {Lin, Carolyn A.},
  title = {An Interactive Communication Technology Adoption Model},
  journal = {Communication Theory},
  volume = {13},
  number = {4},
  pages = {345-365},
  doi = {https://doi.org/10.1111/j.1468-2885.2003.tb00296.x},
  url = {https://onlinelibrary.wiley.com/doi/abs/10.1111/j.1468-2885.2003.tb00296.x},
  eprint = {https://onlinelibrary.wiley.com/doi/pdf/10.1111/j.1468-2885.2003.tb00296.x},
  abstract = {In the evolving research arena of mediated communication technology adoption and uses, one of the most valuable developments involves the increased integration of distinct communication research traditions. This emerging fusion presents an unprecedented opportunity for communication researchers to share, confer, and challenge the “native” tradition that each has followed. This article proposes an integrated research model and explains how it can serve as the basis for mediated communication technology adoption research. In particular, this proposed model is intended to provide a research framework for studying the factors that help shape adoption decisions of various communication technologies and the potential impact of technology adoption on the social system, audiences, and use patterns.},
  year = {2003}
}

@article{50_tor_finger,
  title = {The rise of website fingerprinting on Tor: Analysis on techniques and assumptions},
  journal = {Journal of Network and Computer Applications},
  volume = {212},
  pages = {103582},
  year = {2023},
  issn = {1084-8045},
  doi = {https://doi.org/10.1016/j.jnca.2023.103582},
  url = {https://www.sciencedirect.com/science/article/pii/S1084804523000012},
  author = {Mohamad Amar Irsyad {Mohd Aminuddin} and Zarul Fitri Zaaba and Azman Samsudin and Faiz Zaki and Nor Badrul Anuar},
  keywords = {Security, Privacy, Anonymity, Tor, Traffic analysis, Website fingerprinting},
  abstract = {Tor is one of the most popular anonymity networks that allows Internet users to hide their browsing activity. Hiding browsing activity is essential for Internet users to increase their privacy. Only Tor users should know the website they are browsing. However, an attacker can utilise the Website Fingerprinting (WF) attack to identify webpages browsed by Tor users. WF is a significant threat to Internet users' privacy as Tor should conceal the browsed webpages' information. Existing WF studies focused on the investigation to improve the identification capabilities, overlooking the systematic discussion and assessment of existing techniques. In addition, existing surveys and analyses reviewed insufficient variation of WF on Tor techniques. Therefore, this survey paper aims to provide a systematic and thorough review of various WF on Tor techniques. First, we discuss WF on Tor techniques in five primary aspects: threat model, victim target, website realm, traffic feature, and traffic classifier. We analyse and classify the reviewed studies on each WF aspect. The classification facilitates in-depth understanding and comparison between WF on Tor techniques. Furthermore, this paper investigates nine assumptions exercised in WF on Tor: closed-world, sequential browsing, isolated traffic, replicability, traffic parsing, passive webpage, disabled cache, static content, and single webpage. These assumptions limit the WF on Tor's practicality in real-world scenarios. Our analysis and classification indicate that most WF on Tor studies apply these assumptions despite being only suitable in controlled environments or laboratory experiments. In addition, most reviewed studies often lack transparency on the assumptions exercised in their studies, risking misunderstanding the WF on Tor techniques' actual practicality. At the end of this survey, we present WF on Tor taxonomy and highlight 21 WF on Tor research's limitations and gaps with plausible recommendations. We also discuss the WF on Tor studies' contribution category and development phase.}
}

@inproceedings{51_CDN403,
  author = {McDonald, Allison and Bernhard, Matthew and Valenta, Luke and VanderSloot, Benjamin and Scott, Will and Sullivan, Nick and Halderman, J. Alex and Ensafi, Roya},
  title = {403 Forbidden: A Global View of CDN Geoblocking},
  year = {2018},
  isbn = {9781450356190},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://doi.org/10.1145/3278532.3278552},
  doi = {10.1145/3278532.3278552},
  abstract = {We report the first wide-scale measurement study of server-side geographic restriction, or geoblocking, a phenomenon in which server operators intentionally deny access to users from particular countries or regions. Many sites practice geoblocking due to legal requirements or other business reasons, but excessive blocking can needlessly deny valuable content and services to entire national populations.To help researchers and policymakers understand this phenomenon, we develop a semi-automated system to detect instances where whole websites were rendered inaccessible due to geoblocking. By focusing on detecting geoblocking capabilities offered by large CDNs and cloud providers, we can reliably distinguish the practice from dynamic anti-abuse mechanisms and network-based censorship. We apply our techniques to test for geoblocking across the Alexa Top 10K sites from thousands of vantage points in 177 countries. We then expand our measurement to a sample of CDN customers in the Alexa Top 1M.We find that geoblocking occurs across a broad set of countries and sites. We observe geoblocking in nearly all countries we study, with Iran, Syria, Sudan, Cuba, and Russia experiencing the highest rates. These countries experience particularly high rates of geoblocking for finance and banking sites, likely as a result of U.S. economic sanctions. We also verify our measurements with data provided by Cloudflare, and find our observations to be accurate.},
  booktitle = {Proceedings of the Internet Measurement Conference 2018},
  pages = {218–230},
  numpages = {13},
  location = {Boston, MA, USA},
  series = {IMC '18}
}

@inproceedings{52_googleScholar,
  author = {Lu, Zhen and Li, Zhenhua and Yang, Jian and Xu, Tianyin and Zhai, Ennan and Liu, Yao and Wilson, Christo},
  year = {2017},
  month = {12},
  pages = {8-14},
  title = {Accessing google scholar under extreme internet censorship: a legal avenue},
  doi = {10.1145/3154448.3154450}
}

@inproceedings{53_quicCensor,
  author = {Elmenhorst, Kathrin and Sch\"{u}tz, Bertram and Aschenbruck, Nils and Basso, Simone},
  title = {Web censorship measurements of HTTP/3 over QUIC},
  year = {2021},
  isbn = {9781450391290},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://doi.org/10.1145/3487552.3487836},
  doi = {10.1145/3487552.3487836},
  abstract = {Web traffic censorship limits the free access to information, making it a global human rights issue. The introduction of HTTP/3 (HTTP over QUIC) yields promising expectations to counteract such interference, due to its novelty, build-in encryption, and faster connection establishment. To evaluate this hypothesis and analyze the current state of HTTP/3 blocking, we extended the open-source censorship measurement-tool OONI with an HTTP/3 module. Using an input list of possibly-blocked websites, real-world measurements with HTTPS and HTTP/3 were conducted in selected Autonomous Systems in China, Iran, India, and Kazakhstan. The presented evaluation assesses the different blocking methodologies employed for TCP/TLS versus the ones employed for QUIC. The results reveal dedicated UDP blocking in Iran and major IP blocklisting affecting QUIC in China and India.},
  booktitle = {Proceedings of the 21st ACM Internet Measurement Conference},
  pages = {276–282},
  numpages = {7},
  location = {Virtual Event},
  series = {IMC '21}
}

@inproceedings{54_QUIC,
  author = {Langley, Adam and Riddoch, Alistair and Wilk, Alyssa and Vicente, Antonio and Krasic, Charles and Zhang, Dan and Yang, Fan and Kouranov, Fedor and Swett, Ian and Iyengar, Janardhan and Bailey, Jeff and Dorfman, Jeremy and Roskind, Jim and Kulik, Joanna and Westin, Patrik and Tenneti, Raman and Shade, Robbie and Hamilton, Ryan and Vasiliev, Victor and Chang, Wan-Teh and Shi, Zhongyi},
  title = {The QUIC Transport Protocol: Design and Internet-Scale Deployment},
  year = {2017},
  isbn = {9781450346535},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://doi.org/10.1145/3098822.3098842},
  doi = {10.1145/3098822.3098842},
  abstract = {We present our experience with QUIC, an encrypted, multiplexed, and low-latency transport protocol designed from the ground up to improve transport performance for HTTPS traffic and to enable rapid deployment and continued evolution of transport mechanisms. QUIC has been globally deployed at Google on thousands of servers and is used to serve traffic to a range of clients including a widely-used web browser (Chrome) and a popular mobile video streaming app (YouTube). We estimate that 7\% of Internet traffic is now QUIC. We describe our motivations for developing a new transport, the principles that guided our design, the Internet-scale process that we used to perform iterative experiments on QUIC, performance improvements seen by our various services, and our experience deploying QUIC globally. We also share lessons about transport design and the Internet ecosystem that we learned from our deployment.},
  booktitle = {Proceedings of the Conference of the ACM Special Interest Group on Data Communication},
  pages = {183–196},
  numpages = {14},
  location = {Los Angeles, CA, USA},
  series = {SIGCOMM '17}
}

@misc{55_HTTP3,
    series =    {Request for Comments},
    number =    9114,
    howpublished =  {RFC 9114},
    publisher = {RFC Editor},
    doi =       {10.17487/RFC9114},
    url =       {https://www.rfc-editor.org/info/rfc9114},
    author =    {Mike Bishop},
    title =     {{HTTP/3}},
    pagetotal = 57,
    year =      2022,
    month =     jun,
    abstract =  {The QUIC transport protocol has several features that are desirable in a transport for HTTP, such as stream multiplexing, per-stream flow control, and low-latency connection establishment. This document describes a mapping of HTTP semantics over QUIC. This document also identifies HTTP/2 features that are subsumed by QUIC and describes how HTTP/2 extensions can be ported to HTTP/3.},
}

@inproceedings{56_gfwTOR,
  title = {How the Great Firewall of China is Blocking Tor},
  booktitle = {2nd USENIX Workshop on Free and Open Communications on the Internet (FOCI 12)},
  year = {2012},
  address = {Bellevue, WA},
  url = {https://www.usenix.org/conference/foci12/workshop-program/presentation/Winter},
  publisher = {USENIX Association},
  month = aug
}

@misc{57_blockingTor,
      title={How China Is Blocking Tor}, 
      author={Philipp Winter and Stefan Lindskog},
      year={2012},
      eprint={1204.0447},
      archivePrefix={arXiv},
      primaryClass={cs.CR}
}

@online{58_breakwaIssue,
        title= {Analysis and Detection of Issues in Shadowsocks}
        author={BreakWa11@GitHub},
        type={GitHub Issue},
        url={https://web.archive.org/web/20160829052958/https://github.com/breakwa11/shadowsocks-rss/issues/38}
}
```
