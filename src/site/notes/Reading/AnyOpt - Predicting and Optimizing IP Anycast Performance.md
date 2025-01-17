---
{"dg-publish":true,"permalink":"/reading/any-opt-predicting-and-optimizing-ip-anycast-performance/"}
---

SigComm 2021
> Zhang, Xiao, Tanmoy Sen, Zheyuan Zhang, Tim April, Balakrishnan Chandrasekaran, David Choffnes, Bruce M. Maggs, Haiying Shen, Ramesh K. Sitaraman, and Xiaowei Yang. “AnyOpt: Predicting and Optimizing IP Anycast Performance,” 447–62. Virtual Event USA: ACM, 2021. [https://doi.org/10.1145/3452296.3472935](https://doi.org/10.1145/3452296.3472935).

Methodology
- Akamai DNS have 500 sites and 20 transit providers. 
- Because 500 sites are too many, the authors select only 20 sites (as source). (Representing each Transit Provider)
	- **Each site has only one provider.** ??, 
	- Destination = 11K routers, each representing many customers
- Anycast site is connected to tier-1 AS. Out of many sites that belongs in tier-1 ISP, randomly pick ONE site. (They say randomly picking is good enough? - 94.2% of the client networks on average do not change their preferences.) Measure RTT from that site to customer. 
	- Customer = Selected set of routers 
- Measure RTT from 20 site to 11k representative routers. 
- Select a pair of sources. (nC2) For all representing sites - announce same test prefix to each of **tier-1 ISP**, and then observe anycast catchment from client(just like Verfploeter does).
- Using measured anycast catchment, first calculate "consistent total order". For instance, if a router(cusomer) preferred site #1 over #2, and #2 over #3... we have consistent total order. If not, this router(client) is **excluded**.
	- For instance, We pick site that has AT&T and Spectrum -> and we announce same prefix -> and customer 1 preferred AT&T > Spectrum
	- We pick Verizon and Spectrum  -> customer 1 preferred Verizon > Spectrum
	- We pick AT&T and Verizon -> customer 1 preferred AT&T > Spectrum

AT&T > Spectrum > Verizon > AT&T … They exclude

optimize mean latency for all customers , select subset. 

- Now, this problem is "SPLPO(Simple Plant Location Problem with Preference Orderings)" problem, which is NP-hard. Since this is NP-hard, network admin can either do exhaustive search(only possible when anycast site is not too many) or make offline simulator to get approximate value. (this is not described much in the paper)

> [!def] SPLPO
> **Definition:**
> The Simple Plant Location Problem (SPLP) is an optimization problem that determines the optimal placement of facilities (e.g., factories) to minimize the total cost, which includes both facility setup costs and transportation costs.  
> *Example:* Deciding in which cities to build factories to achieve the lowest overall logistics cost.
> 
> The Simple Plant Location Problem with Preference Orderings (SPLPO) extends SPLP by incorporating client preferences for each facility location.  
> *In this case:* Clients have a "preference order" for the facilities, such as favoring closer facilities over others, and the optimization is performed based on these preferences.

- After this, now, we have to find each network’s preference orders among the anycast sites  within the same transit provider AS. Do pairwise experiment in the same AS.  (However, for actual Akamai DNS, they used latency value and not do this step.)

After deployment, we can include the beneficial peering links discovered  using the one-pass method described below.
- Peering remains. We first measure whether enabling a peer  will reduce the average latency of the baseline configuration where  only transit providers are enabled. If so, we consider the peer as a  beneficial peer. From the optimal transit-only configuration found by  AnyOpt, we enable one peer at a time, measure the peer’s catchment  after the peer is enabled, and measure how the average latency has  changed. If the average latency is reduced, we mark this peer as a  beneficial peer. We then disable this peer, measure another peer, and  so on until we measure all peering connections of an anycast network. If an anycast network has a total of M peers, this step requires M BGP measurements.

Solves 2 problem
1. Subset selection problem
    - selecting and announcing 12 sites, reduced the average RTT by 33ms compared to a greedy method with the same number of sites.
2. Catchment Prediction + Latency Estimation
    - First choose a random subset R from all sites. Use each client network’s total preference order (among this subset) under a BGP announcement order for predicting the client network’s most preferred site among R and its RTT to its catchment site. (ONLY clients that exhibit total order is predicted)
    - Actually deploy and compare the results.
    - Result: Using 15 anycast sites, predicted 15,300 client catchments, with 94.7% accuracy and an average RTT error of 4.6%.  

Limitations
- network should have "total order" (clear consistent preference ranking among all available anycast sites.)  -> this is very idealistic, not many IP blocks show that. 

![Pasted image 20250115155033.png](/img/user/Reading/attachments/Pasted%20image%2020250115155033.png)

* TAKES WAY TOO LONG!
	- Using 4 anycast prefix
	- each BGP experiment = 2 hours
	- 500 sites measurement = 1000 hours, parallel in 4 = 250 hours (10 days)
	- 380 pair-wise experiments = 760 hours, parallel in 4 = 190 hours (8 days)
	-> every month you can re-configure.

Detailed Notes
How to measure RTT?
	- Everything is same as Verfploeter, but this testbed uses individual IP prefix for different Sites. Ping sent from one site, and responded at that site.

Remaining Questions
- **WHY connect to ISPs?? who knows who is their provider? and why does provider matter so much??** How does this relate to Akamai algorithm?
"""
Using Akamai CDN's technology to select our targets for measurements, we follow an approach  used by the CDN hosting our testbed [23]. Specifically, we merge the  network paths from the end-users to a CDN’s edge server into a tree,  rooted at the edge server. We then pick a common ancestor that is  closest to the end-users.
"""
- What is this? How is this possible(using traceroute?) and how does this relate to announcing an anycast prefix to only tier-1 providers?

## Need
**I want to make an algorithm that can optimize and suggest a new site for anycast.** 

My point
- this paper seems like it can work(cause they select optimal sets of anycast sites).
- However, this paper is not representing small-sited non-tier-1 peered - not work for my problem. 

The assumption for this algorithm to work is 
1. we have a set of many anycast sites and we want to reduce the number of anycast sites and optimizes the customer of latency.
2. our sites are directly connected to tier-1 Transit providers
But, the problem we want to solve needs an algorithm that can
1. we already have a small set of few anycast sites, but we want to add a new site that optimizes the customer of latency.
2. sites may not be connected to tier-1 Transit providers

The paper did not discuss
- Where should the new site be added?
    - They already had a subset of sites, and chose the downsized set. ()
    - What if we don’t have candidate list? We want to first figure out our top-N ASes which are universities or institutions, and then reach out as they may not collaborate.
        - We can also analyze if they can handle the massive traffic, or has the ability to maintain such infrastructure. This can be part of the contribution.
            
- We have a sequential data. Every month LAX site goes on maintenance, and we can actually observe 5-site state. Time-Series Analysis is possible
    - We can actually figure out WHY our prediction was wrong, and make stronger algorithm.
        - Maybe use Reinforcement learning?

I want to solve my problem in different ways.(by using BGP messages & using anycast sites history)

this relates to - [[Modeling BGP map (Explainable Anycast)\|Modeling BGP map (Explainable Anycast)]]

Few thoughts
Difference in experiment settings

1. B-root(and other anycast services) is non-profit organizations, who might not directly peer with Tier-1 AS.
2. Akamai may not had “effective” peers, but these institutions might do. (I.e., Los Nettos is a good example)
3. We don’t have Akamai’s root detecting algorithm, but we have infra for end-user pinging. (But we need to build system for site-level latency measuring)

• 1. From "Anycast Latency: How Many Sites Are Enough?”, we know that few sites can provide performance nearly as good as many, and that geographic location and good connectivity have a far stronger eﬀect on latency than having many sites.

---

TODO read reference

"An anycast-based CDN faces a similar configuration challenge.  For a CDN service provider, the latency between a client and an edge  server can have a multiplicative effect on page-load times, given the  many round-trips typically required to download various resources.  Therefore, reducing latency by even tens of milliseconds can result  in a substantial reduction of page-load times [22]."

	[22] Thomas Koch, Ke Li, Calvin Ardi, Ethan Katz-Bassett, Matt Calder, and John Heidemann. 2021. Anycast in Context: A Tale of Two Systems. In Proceedings of the 2021 Conference of the ACM Special Interest Group on Data Communication (Virtual Event) (SIGCOMM ’21). Association for Computing Machinery, New York, NY, USA, 20. [https://doi.org/10.1145/3452296.3472891](https://doi.org/10.1145/3452296.3472891)

"IP anycast has long been used by Internet services to provide automatic load balancing and latency reduction among service replicas. Previous work focuses on measuring the performance of deployed IP anycast systems, including DNS root servers [6, 11, 17, 22, 22, 24, 26, 27, 29] and CDNs [7, 12, 22]."

	[24] Matthew Lentz, Dave Levin, Jason Castonguay, Neil Spring, and Bobby Bhattacharjee. 2013. D-Mystifying the D-Root Address Change. In Proceedings of the 2013 Conference on Internet Measurement Conference (Barcelona, Spain) (IMC ’13). Association for Computing Machinery, New York, NY, USA, 57–62. [https://doi.org/10.1145/2504730.2504772](https://doi.org/10.1145/2504730.2504772)
	
	[26] Jinjin Liang, Jian Jiang, Haixin Duan, Kang Li, and Jianping Wu. 2013. Measuring Query Latency of Top Level DNS Servers. In Passive and Active Measurement, Matthew Roughan and Rocky Chang (Eds.). Springer Berlin Heidelberg, Berlin, Heidelberg, 145–154. [https://link.springer.com/chapter/10.1007/978-3-64236516- 4_15](https://link.springer.com/chapter/10.1007/978-3-642-36516-4_15)
	
	[27] Ziqian Liu, Bradley Huffaker, Marina Fomenkov, Nevil Brownlee, and kc claffy. 2007. Two Days in the Life of the DNS Anycast Root Servers. In Passive and Active Network Measurement, Steve Uhlig, Konstantina Papagiannaki, and Olivier Bonaventure (Eds.). Springer Berlin Heidelberg, Berlin, Heidelberg, 125–134. [https://link.springer.com/chapter/10.1007/978- 3- 540- 71617- 4_13](https://link.springer.com/chapter/10.1007/978-3-540-71617-4_13)

This result  may not hold for other anycast networks where peering connections  attract larger amounts of traffic, but Schlinker et al. also observed  that peer routes and provider routes had similar performance in  terms of latency for the Facebook network [35].
[35] Brandon Schlinker, Italo Cunha, Yi-Ching Chiu, Srikanth Sundaresan, and Ethan Katz-Bassett. 2019. Internet Performance from Facebook’s Edge. In Proceedings of the Internet Measurement Conference (Amsterdam, Netherlands) (IMC ’19). Association for Computing Machinery, New York, NY, USA, 179–194. [https://doi.org/10.1145/3355369.3355567](https://doi.org/10.1145/3355369.3355567)