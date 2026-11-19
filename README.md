# ResilientNet: Disaster-Resilient City Network with Dual ISP Connectivity

A Cisco Packet Tracer case study for course Computer Networks (21CSC302J) at SRMIST, Kattankulathur. A segmented city emergency network (Hospital, Police, Fire, Disaster Management) that keeps external services reachable when one ISP fails.

**Packet Tracer version:** 9.0.1.0858

## What it does
- Departments are separated into VLANs 10, 20, 30, 40 (management VLAN 99) with router-on-a-stick.
- Multi-area OSPF inside the city (R-CORE is the ABR for areas 0, 1, 2).
- R-EDGE (AS 65000) connects to ISP-A (AS 65010, primary) and ISP-B (AS 65020, backup) with eBGP; R-CLOUD (AS 65030) hosts the servers.
- ISP-A is preferred with a static route. If the ISP-A link fails, the BGP route via ISP-B takes over; when it returns, traffic goes back to ISP-A.
- PAT translates all city hosts to one loopback address (198.51.100.1) that both ISPs learn, so NAT keeps working after a switch.
- Services: web portal (203.0.113.10), DNS (203.0.113.11, www.resilientnet.local).

## Topology and addressing
| Link | Subnet |
|---|---|
| R-CORE – R-AREA1 | 10.10.0.0/30 |
| R-CORE – R-AREA2 | 10.10.0.4/30 |
| R-CORE – R-EDGE | 10.10.0.8/30 |
| R-EDGE – ISP-A | 10.10.0.12/30 |
| R-EDGE – ISP-B | 10.10.0.16/30 |
| ISP-A – R-CLOUD | 10.10.0.20/30 |
| ISP-B – R-CLOUD | 10.10.0.24/30 |

| VLAN | Department | Subnet | Gateway |
|---|---|---|---|
| 10 | Hospital | 10.10.10.0/27 | 10.10.10.1 |
| 20 | Police | 10.10.10.32/27 | 10.10.10.33 |
| 30 | Fire | 10.10.20.0/27 | 10.10.20.1 |
| 40 | Disaster | 10.10.20.32/27 | 10.10.20.33 |
| 99 | Management (Area 1 / Area 2) | 10.10.99.0/28, 10.10.99.16/28 | .1 / .17 |

Server network: 203.0.113.0/28 (gateway 203.0.113.1). The 203.0.113.0/24 and 198.51.100.0/24 ranges are reserved for documentation and stand in for public addresses.

## Files
- `ResilientNet.pkt`: the Packet Tracer project (no device passwords, so it is not locked).
- `configs/`: `show running-config` output for each router.
- `security_commands.txt`: optional SSH, management ACL and port security commands (placeholders, no real passwords).
- `images/`: topology and logical architecture.

## Run the failover demo
1. On R-EDGE: `show ip route 203.0.113.0` -> `S 203.0.113.0 [10/0] via 10.10.0.14` (ISP-A).
2. From a department PC, open `www.resilientnet.local` or `ping 203.0.113.10`.
3. Fail ISP-A: on R-EDGE `interface g0/1` then `shutdown` (or shut ISP-A g0/0).
4. Wait a moment, then `show ip route 203.0.113.0` -> `B 203.0.113.0 [20/0] via 10.10.0.18` (ISP-B). The service still works.
5. `no shutdown` to restore; the route returns to ISP-A.

Packet Tracer can be slow to switch routes, so allow some time before testing.

## Results observed
ISP-A down: traffic moves to ISP-B. ISP-A restored: traffic returns to ISP-A. ISP-B down: traffic stays on ISP-A. Both down: destination host unreachable. The NAT inside-global address stayed 198.51.100.1 during failover. No timing measurements were taken.

## Limitations
The static preference detects only a link failure (production designs would use IP SLA and BGP local-preference). R-EDGE could pass routes between the two ISPs (needs prefix-lists/route-maps). Two ISPs tolerate one failure, not a disaster that affects both (see "Disaster scenario and design scope"). The internal network is a tree with single points of failure. Packet Tracer did not accept route-maps or BGP weight.

## Disaster scenario and design scope
This design protects against the failure of **one ISP or one ISP link** (for example a fibre cut, a provider outage or a faulty router). It does **not** protect against a regional disaster that takes down both providers at the same time. In that case "Both down: destination host unreachable" is the expected result, and it is a limit of the design, not a bug.

Both ISP links here are wired. Wired links are not certain to fail together, but in a real disaster their failures are strongly correlated (common-cause or shared-fate failure):
- Both providers' cables may share the same ducts, bridges or roads, so one flood, earthquake or road-works cut can break both.
- Both providers may depend on the same exchange or power grid, so a power outage can take both offline even if the cables survive.
- Providers may share upstream carriers or data centres, so redundancy on paper can be less than it appears.

The simulation shows ISP diversity only. It does not show path, media or geographic diversity, which is what a real disaster network needs.


## Security note
Security hardening (SSH v2, management ACL, port security) was configured and tested in an earlier build. The project file here is the version without device passwords so that it can be opened and run without a login. The commands are in `security_commands.txt`.

## References
- Y. Rekhter, T. Li, S. Hares, "A Border Gateway Protocol 4 (BGP-4)," RFC 4271, Jan. 2006.
- P. Srisuresh, K. Egevang, "Traditional IP Network Address Translator (Traditional NAT)," RFC 3022, Jan. 2001.

## Author
Developed as an academic networking project through which I explored computer networking concepts and gained hands-on experience with Cisco Packet Tracer, VLANs, multi-area OSPF, dual-ISP BGP routing, and NAT/PAT.

## License
This project is licensed under the MIT License.

---
Malakonda Chaitanya Bhargav
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://www.linkedin.com/in/chaitanya-bhargav-malakonda/)