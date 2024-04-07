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

**Keywords** -- GFW of China, IDS, IP blocking, DNS manipulation, URL filtering, Keyword filtering, RST packets, Circumvention

## I. Introduction

*Across the **Great Wall** can reach every corner in the world.* <x>@ref01</x> 
This email message sent to Karlsruhe University of Germany, 
from Beijing in September 1987, marks China’s entrance into the Internet. 
Ironically, as of December 2022, with 1.07 billion Internet users domestically,
<x>@ref02</x> it would be nearly impossible to reach to anywhere on Internet 
without break across the GFW, the Great Firewall of China. 

The GFW comprises of intricate and multifaceted mechanisms,
which ranges over technical, societal and political aspects. 
Beyond being a technical barrier, the Wall also includes legal ramifications for those attempting to bypass it, mandatory content regulations responsibilities for platform hosts, and a pervasive surveillance network that monitors digital footprints, and subtly fabricating the psychological norms, shapes thought and molds behavior of the users.

In this paper, we will talk about the technical aspects of the Wall, and briefly discusses the main strategies of circumventing it.

## II. Operation of the GFW

Before covering the mechanisms used by the GFW, 
it is worth briefly covering how and where the packet monitoring is done 
to get a complete picture of the system.

### Structure of the GFW

AS-es (autonomous systems) can be divided into internal and border AS-es 
  where border AS-es communicate to foreign AS-es. 
Most of the filtering done by the GFW is done in the border AS-es <x>@ref03</x>
  where the request and response packets enter and leave the country,
  some filtering done inside internal AS-es <x>@ref03</x> as well.

Majority of the filtering is done in border AS-es, as it is the main concern on
  the network layer. 
  GFW's main work for domestic traffic is usually done in the application layer,
  through social engineering and state surveillance on hosting platforms and 
  the users, such as requiring content censors prior to publication 
  on all platforms, real-name registration of the users, and potentially legal
  repercussions for those who post illicit content, etc. <x>@ref04</x>. 
  We would not cover these aspects in this paper.

Having filtering done in internal AS-es is also likely an efficiency measure as 
  having everything done in border AS-es could bottleneck the network so 
  mitigating some of the filtering into the interior AS-es can distribute the 
  the load more evenly.

An important note is that the GFW isn’t truly a wall in literal sense, 
  as the paths to nodes in China might experience different levels of filtering,
  and the list of conditions for a packet to be censorable would change over time
  (e.g. the term "Tiananmen Square" would rise in censorship around the
  anniversary of the 1989 protests in May and June) <x>@ref05</x>.

It is the case that the purpose of the GFW isn’t to create an impassable wall
 that prevents all foreign traffic, as it's technically infeasible and 
 economically destructive to do so, but more to instill fear and self-censorship
 and implicitly mold the online behavior with the sense of compliance.

### How Packets Are Monitored

The packets in the internet traffic tracked by the GFW are monitored mainly with
  routers in various AS-es that equipped with IDS technology <x>@ref03</x>, which
  enables the routers to inspect incoming traffic using mechanisms to be discussed
  in the next sections.

When a packet satisfies the conditions for censorship, the most common case for the
  GFW is to send a TCP RST packet to both ends to terminate the TCP connection on
  both sides <x>@ref03</x>, triggering a denial-of-service attack prevent a pair
  of endpoints from communicating. Despite this, researchers have found it is 
  possible to *ignore* the spoofed RST packet and still receive the responses
  <x>@ref06</x>.

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
of granularity. <x>@ref07</x> 
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
  or wildcard to represent the client attempting to connect. <x>@ref12</x>

![Figure 3-3: Destination-based Black Hole Filtering with Remote Triggering](res/3-3-cisco.png)

![Figure 3-4: Source-based Black Hole Filtering with Remote Triggering](res/3-4-cisco-source.png)

On the BGP level, the routers are configured to drop the packets from matched
  IP addresses, such black-holed IP addresses involves 3 steps: <x>@ref13</x> <x>@ref14</x>
- **Setup (preparation)**: A trigger is a special device that is installed at the NOC 
  (Network Operations Center) exclusively for the purpose of triggering a black hole. 
  The trigger must have an BGP peering relationship with all the edge routers, and is
  configured to redistribute static routes to its BGP peers, sends the static route by
  means of an BGP routing update.

  The Provider Edges (PEs) must have a static route for an unused IP address space. 
  For example, 192.0.2.1/32 is set to Null0 (which is not used as a deployed IP address)

  Loose URPF (Unicast Reverse Path Forwarding) is configured on all external facing 
  interfaces of PEs.

- **Triggering**: When an administrator adds a static route to the trigger, which 
  redistributes the route by sending a BGP update to all its BGP peers, setting 
  the next hop to the target destination address 
The blacklist strategy is more common in the GFW, although whitelist strategy is 
  possible and has been used in the past, such that after 2009 Urumqi riots,
  internet access was blocked in Xinjiang in a way that only less than 100 
  local sites, such as banks and local government sites were accessible. <x>@ref10</x>
It lasted for nearly a year, <x>@ref11</x>
  damages to the economy, society and people's social life were immense.


The daemon list contains process names or wildcards that represent what the client is trying to connect to. The client list contains the address, host name, or wildcard to represent the client attempting to connect. The options specify various things to do whenever the rule is triggered [1].


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

---
refs:
    
    - id: ref01
        title: The Internet Timeline of China 1986-2003
        pubinfo: China Internet Network Information Center (CNNIC)
        URL: https://web.archive.org/web/20240119091818/https://www.cnnic.com.cn/IDR/hlwfzdsj/201306/t20130628_40563.htm
        accessed: 2024-01-19
    
    - id: ref02
        title: The 50th Statistical Report on China's Internet Development
        pubinfo: China Internet Network Information Center (CNNIC)
        URL: https://web.archive.org/web/20231117230017/https://www.cnnic.com.cn/IDR/ReportDownloads/202212/P020230829504457839260.pdf
        accessed: 2023-11-17

    - id: ref03
        title: Internet Censorship in China: Where Does the Filtering Occur?
        author: Xu, X., Mao, Z. M., & Halderman, J. A.
        pubinfo: Spring, N., Riley, G.F. (eds) Passive and Active Measurement. PAM 2011. Lecture Notes in Computer Science, vol 6579. Springer, Berlin, Heidelberg.
        URL: https://doi.org/10.1007/978-3-642-19260-9_14
        accessed: 2024-03-08
    
    - id: ref04
        title: Analysis on China's Digital Ecosystem and its Implications for the Modern World
        author: Jia, M., Vassileva, J.
        pubinfo: (this is my paper for CMPT412)
        accessed: 2024-03-08
    
    - id: ref05
        title: ConceptDoppler, A Weather Tracker for Internet Censorship
        author: Crandall, J. R., Zinn, D., Byrd, M., Barr, E. T., & East, R.
        pubinfo: CCS. 2007 Oct 29;7:352-65.
        URL: https://doi.org/10.1145/1315245.1315290
        accessed: 2024-04-03

    - id: ref06
        title: Ignoring the Great Firewall of China
        author: Clayton, R., Murdoch, S.J., Watson, R.N.M
        pubinfo: Danezis, G., Golle, P. (eds) Privacy Enhancing Technologies. PET 2006. Lecture Notes in Computer Science, vol 4258. Springer, Berlin, Heidelberg. 
        URL: https://doi.org/10.1007/11957454_2
        accessed: 2024-04-01        
    
    - id: ref07
        title: A Taxonomy of Internet Censorship and Anti-Censorship
        author: Leberknight, Christopher S., et al.
        pubinfo: Fifth International Conference on Fun with Algorithms (pp. 52-64).
        URL: https://www.princeton.edu/~chiangm/anticensorship.pdf
        accessed: 2024-04-01

    - id: ref08
        title: Poisoning the Well
        author: Farnan, O., Darer, A., Wright, J.
        pubinfo: Proceedings of the 2016 ACM on Workshop on Privacy in the Electronic Society - WPES'16. pp. 95–98. 
        URL: https://doi.org/10.1145/2994620.2994636
        accessed: 2024-04-01

    - id: ref09
        title: IT Security in the USA, Japan and China, A Study of Initiatives and Trends within.
        author: Ahlgren, M., Breidne, M., Hektor, A.
        accessed: 2024-04-01
    
    - id: ref10
        title: The Missing Link
        author: Jia, Cui (2009 Nov.)
        pubinfo: China Daily
        accessed: 2024-04-03
    
    - id: ref11
        title: Xinjiang Internet Restored Starting Today 
        pubinfo: News.china.com.cn
        URL: web.archive.org/web/20100517023849/http://news.china.com.cn/txt/2010-05/14/content_20038848.htm
        accessed: 2024-04-03
    
    - id: ref12
        title: hosts.den(5) - Linux Man Page
        author: Wietse Venema
        pubinfo: Free Software Foundation, Inc.
        URL: https://linux.die.net/man/5/hosts.deny

    - id: ref13
        title: Remotely Triggered Black Hole Filtering - Destination-Based and Source-Based
        author: Cisco Systems, Inc. (2005)
        pubinfo: Cisco Systems, Inc.
        URL: https://www.cisco.com/c/dam/en_us/about/security/intelligence/blackhole.pdf
        accessed: 2024-04-03
    
    - id: ref14
        title: RFC 5635 - Remote Triggered Black Hole filtering with uRPF
        author: Kumari, W., et al. (2009)
    

    
---