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



The rule shown in figure 1-2 shows how such a rule could be used to block the IP 100.100.100.100. In the rule it states that all processes that the IP address tries to connect to should be denied a connection making a TCP connection impossible to establish. Adding this rule to the “/etc/host.deny” file would IP block the address 100.100.100.100 from the network, and to block other IPs simply replace the IP with another.

### Flaws in IP Blocking

The main disadvantages of IP blocking are that an entity’s IP address can be changed, there is delay in getting IP addresses banned, and that it is possible to *over-block*[3]. The first issue comes from the fact that it is possible for an entity to change their IP address for example by changing where their website is being hosted. This poses an issue for IP blocking because this mechanism relies only on IP to know who to block so by changing IP addresses a banned entity can essentially un-ban themselves thereby circumventing the mechanism. The next issue is related to the previous, but differs in the scale because while changing an IP can un-ban a single user new entities appearing and disappearing makes the list of IP addresses that should be blocked at any given time a moving target. Due to needing to either manually ban IP addresses or automatically ban them based on some metric such as illicit messages in a *time-frame* it creates a drift between what is actually banned and what should be banned leading to some illicit traffic getting past this mechanism. The final issue is not about failing to block traffic, but instead blocking too much traffic. *Over-blocking* comes from multiple entities sharing an IP address. An example where this would be a problem is if a NAT is in use for a network meaning that all users on that network are essentially sharing an IP address. If one user on that network managed to get IP blocked then that would mean that all users on that network would also be blocked as well due to sharing an IP address. This is not ideal as it means that non-illicit traffic that should get through cannot.

## IV. DNS Manipulation

DNS manipulation centers around DNS servers where domain names are translated into IP addresses. At these servers DNS manipulation can be done where instead of returning the actual IP addresses being searched for the result can be changed into either a non-functional IP address or into the IP address of another website. Either option prevents the user from connecting to the host they were attempting to connect to which is how this mechanism restricts internet traffic.

### Implementing DNS Manipulation

DNS manipulation in the GFW is done via DNS cache poisoning [1]. This means that the cache in DNS servers that usually stores DNS records for recently accessed hosts is manipulated such that the cached record no longer has the correct information. Normally DNS lookups work recursively where many DNS resolvers are examined to find out which one stores the relevant records or going to the name server for the host if none of the examined DNS resolvers have the record. For efficiency DNS resolvers cache recently accessed records. In order to implement DNS cache poisoning the record for the host name needs to be changed. The records themselves have many types, but focusing only on A and AAAA type records that store IPv4 and IPv6 addresses respectively is fine for an example.

![Figure 4-1: An example of A and AAAA type records](res/4.1-DNS.png)

As seen in figure 1-1 one of these such records has fields for the type, domain name, value, and time to live stored within them. The value is the important field to change as it stores the IP address associated with the domain name. By changing this value to another address the DNS record becomes poisoned and will no longer return the correct value to the requester. An example of what this could look like is shown in figure 1-2 where the records for bannedSite.com and bannedSite2.com have been poisoned to refer to another site and to give a meaningless address respectively.

![Figure 4-2: An example of DNS cache poisoning](res/4.2-DNS-poisoned.png)

Time to live does cause some complications however as cached DNS records are regularly thrown out to ensure that information stays up to date. To remedy this some special list of records for hosts which are to be poisoned would be needed and would need to be somehow distributed to ensure that the DNS records remain poisoned. To actually poison the DNS records the Chinese government would just directly access the DNS records as unlike a common hacker they have the power to easily do that. From there they would only need to ensure that anyone on a Chinese network connects to a Chinese DNS server which given the power of the Chinese government would not be difficult.

### Flaws of DNS Manipulation

The first way to bypass this mechanism is by avoiding DNS lookups in the first place. This can be done as the purpose of a DNS server to find the IP address of a host, but if you already know the IP address somehow then this can be completely bypassed. Another method to bypass this is to change the host name being used as similar to IP blocking the DNS poisoning cannot be done to a host name that hasn’t been recognized as needing to be poisoned. This is not very practical however as not many hosts would want to change their domain name and the new domain name can also just be added to the list to be poisoned again. Another similar method is to have a domain name that resolves to the true IP of another poisoned IP. This is more practical as the website being accessed doesn’t need to change its own name instead a path around the poisoning is being added. This is also not entirely practical as the new domain can be added to the poisoned list and the new domain needs to know the IP of the actual destination website which it would likely need to use DNS to learn. The main flaw of this approach is similar to IP blocking where what should ideally be blocked is a moving target as new hosts can be created and host names can be changed meaning that there will always be websites that the system doesn’t recognize as needing to be poisoned. 

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
