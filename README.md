# Lab 02 -- VPN IPsec & GRE Tunnels

> **NetworkThinkTank Labs** -- Hands-on networking labs for engineers, by engineers. Blog: [networkthinktank.blog](https://networkthinktank.blog)

---

## Blog Article Summary

This lab accompanies our blog series on **VPN technologies using IPsec and GRE tunnels**. Virtual Private Networks (VPNs) are the backbone of secure site-to-site and remote-access connectivity across untrusted networks like the Internet. IPsec provides encryption, authentication, and integrity, while GRE tunnels enable the encapsulation of multicast and routing protocols over point-to-point links.

In this lab, you will build a multi-site VPN topology from scratch, configure both GRE tunnels and IPsec encryption, implement GRE-over-IPsec for secure tunnel transport, and troubleshoot common VPN issues that you will encounter in production networks.

---

## Lab Objectives

By completing this lab, you will be able to:

1. Configure GRE tunnels between remote sites
2. Implement IPsec site-to-site VPN using IKEv1 and IKEv2
3. Combine GRE with IPsec (GRE-over-IPsec) for secure multicast-capable tunnels
4. Verify tunnel status, ISAKMP/IKE SAs, and IPsec SAs
5. Run dynamic routing protocols (OSPF/EIGRP) over GRE tunnels
6. Troubleshoot common VPN establishment and traffic flow issues

---

## Lab Topology

```
         Site A (HQ)                                    Site B (Branch)
    +-----------------+                            +-----------------+
    |                 |                            |                 |
    |   R1 (HQ-RTR)  |                            |  R3 (BR-RTR)   |
    |   10.1.1.0/24  |                            |  10.3.3.0/24   |
    |   Tunnel0:      |                            |  Tunnel0:       |
    |   172.16.0.1/30 |                            |  172.16.0.2/30  |
    +--------+--------+                            +--------+--------+
             |                                              |
             | GigE0/0                                      | GigE0/0
             | 203.0.113.1                                  | 198.51.100.1
             |                                              |
    +--------+----------------------------------------------+--------+
    |                                                                |
    |                  R2 (ISP / Internet)                           |
    |              203.0.113.2  <-->  198.51.100.2                   |
    |                                                                |
    +----------------------------------------------------------------+
```

> Full topology diagrams are available in the [`/diagrams`](./diagrams) folder.

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Platform** | GNS3, EVE-NG, or Cisco CML |
| **Router Images** | Cisco IOSv 15.x+, CSR1000v, or equivalent with crypto support |
| **Knowledge** | CCNA-level routing & switching, basic encryption concepts |
| **Tools** | Terminal emulator (SecureCRT, PuTTY, or built-in console) |

---

## Step-by-Step Instructions

### Phase 1 -- Base Connectivity

1. Deploy the topology using your preferred platform (see `/lab-files`).
2. Assign IP addresses on all interfaces per the addressing table.
3. Configure default routes on R1 and R3 pointing to R2 (simulated ISP).
4. Verify Layer 3 reachability between R1 and R3 public interfaces with `ping`.

### Phase 2 -- GRE Tunnel Configuration

5. Configure a GRE tunnel (Tunnel0) between R1 and R3.
6. Assign tunnel IP addresses: R1 = 172.16.0.1/30, R3 = 172.16.0.2/30.
7. Set tunnel source and destination to the public-facing interfaces.
8. Verify tunnel is up: `show interface Tunnel0` and `ping 172.16.0.2`.

### Phase 3 -- Dynamic Routing Over GRE

9. Enable OSPF (or EIGRP) on R1 and R3, advertising LAN subnets and tunnel interfaces.
10. Verify routing tables show remote LAN subnets learned via tunnel.
11. Test end-to-end connectivity: `ping` from R1 LAN to R3 LAN.

### Phase 4 -- IPsec Encryption (GRE-over-IPsec)

12. Configure ISAKMP (IKE Phase 1) policy on both R1 and R3.
13. Define pre-shared keys for authentication.
14. Create IPsec transform-set with ESP encryption and hashing.
15. Build a crypto map or tunnel protection profile and apply to the tunnel interface.
16. Verify ISAKMP SA: `show crypto isakmp sa`.
17. Verify IPsec SA: `show crypto ipsec sa`.

### Phase 5 -- Verification & Troubleshooting

18. Generate traffic across the tunnel and verify encryption counters increment.
19. Use `debug crypto isakmp` and `debug crypto ipsec` for detailed troubleshooting.
20. Test failover scenarios by shutting down interfaces.

---

## Real Configurations

Production-ready configuration snippets are provided in the [`/configs`](./configs) folder:

| File | Description |
|---|---|
| `R1-hq-config.txt` | HQ router -- GRE tunnel + IPsec config |
| `R2-isp-config.txt` | ISP router -- basic transit routing |
| `R3-branch-config.txt` | Branch router -- GRE tunnel + IPsec config |

---

## What Can Go Wrong -- Common Issues & Fixes

### 1. GRE Tunnel Interface Shows "Up/Down"
- **Cause:** No route to the tunnel destination IP address.
- **Fix:** Verify a route exists to the remote tunnel endpoint (public IP). Check default routes.

### 2. ISAKMP SA Stuck in MM_NO_STATE
- **Cause:** Mismatched IKE Phase 1 parameters (encryption, hash, DH group, or lifetime).
- **Fix:** Ensure both peers have identical ISAKMP policy settings. Use `show crypto isakmp policy`.

### 3. IPsec SA Not Establishing (QM_IDLE with 0 packets)
- **Cause:** Mismatched transform-set or interesting traffic ACL not matching.
- **Fix:** Verify transform-set parameters match on both sides. Check crypto map ACLs.

### 4. Tunnel Is Up but Traffic Not Encrypted
- **Cause:** Crypto map not applied to the correct interface, or traffic not matching the ACL.
- **Fix:** Verify `crypto map` is applied on the physical interface (not the tunnel). For tunnel protection, apply the IPsec profile directly to the tunnel interface.

### 5. Routing Protocol Neighbors Not Forming Over GRE
- **Cause:** Tunnel MTU issues causing fragmentation, or multicast not traversing the tunnel.
- **Fix:** Adjust `ip mtu` on tunnel interfaces (typically 1400). Verify `tunnel mode gre ip`.

### 6. Intermittent Connectivity / Packet Drops
- **Cause:** MTU/MSS mismatch causing fragmentation and drops.
- **Fix:** Set `ip tcp adjust-mss 1360` on tunnel interfaces. Consider PMTUD settings.

---

## References

- [RFC 2784 -- Generic Routing Encapsulation (GRE)](https://datatracker.ietf.org/doc/html/rfc2784)
- [RFC 7296 -- IKEv2 Protocol](https://datatracker.ietf.org/doc/html/rfc7296)
- [Cisco IPsec Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_conn_ike/configuration/xe-16/sec-ike-xe-16-book.html)
- [Cisco GRE Tunnel Configuration](https://www.cisco.com/c/en/us/support/docs/ip/ip-multicast/43584-gre-tunnel.html)
- [NetworkThinkTank Blog](https://networkthinktank.blog)

---

## Repository Structure

```
lab-02-vpn-ipsec-gre/
|-- README.md
|-- configs/
|   +-- .gitkeep
|-- diagrams/
|   +-- .gitkeep
+-- lab-files/
    +-- .gitkeep
```

---

> **Pro Tip:** Always verify IKE Phase 1 completes before troubleshooting Phase 2. Use `debug crypto isakmp` to watch the negotiation in real-time.

**Happy Labbing!**
