# dhcp-snooping-dynamic-arp-inspection--dai--Security-Lab
An enterprise Layer 2 security lab demonstrating mitigation techniques against Rogue DHCP Servers, DHCP Starvation, ARP Poisoning, and Man-in-the-Middle (MitM) attacks using Cisco DHCP Snooping and Dynamic ARP Inspection (DAI).

An enterprise Layer 2 security lab demonstrating mitigation techniques against Rogue DHCP Servers, DHCP Starvation, ARP Poisoning, and Man-in-the-Middle (MitM) attacks using Cisco DHCP Snooping and Dynamic ARP Inspection (DAI).🌐 Quick Overview & Network Specs🟢 Architecture & Topology: Single-switch enterprise LAN (SW1 - Catalyst 2960) connecting a legitimate DHCP Server Router (R1), a critical infrastructure server (SRV-1), a legitimate end-user workstation (PC-A), and a simulated attacker host (PC-B).🔵 Trusted vs. Untrusted Interfaces:Trusted Ports: The uplink interface to Router R1 (DHCP Server) and the static server port for SRV-1 are explicitly configured as trusted for DHCP Snooping and DAI.Untrusted Ports: Access ports connected to PC-A (Legitimate User) and PC-B (Attacker) are untrusted by default, triggering strict packet inspection.🟣 DHCP Snooping Mechanism: Intercepts untrusted DHCP messages. Builds and dynamically updates the DHCP Snooping Binding Database mapping Client MAC address, Assigned IP address, Lease Time, VLAN, and Switch Port. Drops unauthorized DHCP Offer/Ack messages originating from untrusted ports.🔴 Dynamic ARP Inspection (DAI): Validates incoming ARP packets on untrusted ports against the dynamic DHCP Snooping Binding Database (and static ARP ACLs where applicable). Drops invalid ARP requests/responses to eliminate Man-in-the-Middle (MitM) and ARP Cache Poisoning vectors.🟡 Static & Static-Server Binding Rules: Server SRV-1 (192.168.1.100 / MAC: 0001.1111.1111) uses a statically assigned IP/MAC mapping or a static ARP inspection entry to prevent spoofing of core server infrastructure.🟠 Port-Security & Rate Limiting Guardrails: Configured DAI rate-limiting (ip arp inspection limit rate) on access interfaces to defend against ARP Storm Denial-of-Service (DoS) attacks on switch CPU processing.🗺️ Topology Diagram Summary                      +-------------------+
                      |      R1 (2911)    |
                      |   Legitimate      |
                      |   DHCP Server     |
                      +-------------------+
                                | (Trusted Port)
                                |
 +------------------+   +---------------+   +-------------------+
 |    Laptop-PT     |---|   SW1 (2960)  |---|       PC-PT       |
 | PC-B (Attacker)  |   | Switch Core   |   | PC-A (Legit User) |
 +------------------+   +---------------+   +-------------------+
  (Untrusted Port)              | (Trusted/Static Port)
                                |
                      +-------------------+
                      |     Server-PT     |
                      |       SRV-1       |
                      | IP: 192.168.1.100 |
                      | MAC: 0001.1111.1111
                      +-------------------+
📐 Addressing & Node AllocationsDevice NameRole / FunctionInterfaceIP AddressMAC AddressPort Trust StateR1DHCP Server & GatewayGi0/0192.168.1.1/24DynamicDHCP / DAI TrustedSW1Layer 2 Security SwitchVLAN 1192.168.1.2/24DynamicN/A (Switch Management)SRV-1Core Enterprise ServerFastEthernet192.168.1.100/240001.1111.1111DAI Trusted / Static BindingPC-ALegitimate WorkstationFastEthernetDHCP AssignedDynamicUntrustedPC-BAttacker / Rogue HostFastEthernetSpoofed / RogueDynamicUntrusted💡 Key Technical Insights & What This Lab Demonstrates🔍 DHCP Snooping Binding Database Auditing: Demonstrates how SW1 dynamically tracks IP-to-MAC bindings for unauthenticated endpoints as they request leases via R1.⚡ ARP Poisoning & MitM Mitigation: Validates how DAI drops malicious Gratuitous ARP (GARP) packets sent by PC-B attempting to redirect PC-A traffic through the attacker's laptop.🛠️ Real-World Verification Skills: Accompanied by full command-line audit logs showcasing:show ip dhcp snoopingshow ip dhcp snooping bindingshow ip arp inspectionshow ip arp inspection interfaces🛡️ Layer 2 Defense-in-Depth: Combines DHCP Snooping and Dynamic ARP Inspection into a unified security baseline required for Zero Trust enterprise LAN access networks.🛠️ CLI Configuration Snippets1. Configure DHCP Snooping on SW1SW1# configure terminal
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1

! Set interface connected to R1 as trusted
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit
2. Configure Dynamic ARP Inspection (DAI) on SW1SW1(config)# ip arp inspection vlan 1

! Trust uplink to router and connection to static server
SW1(config)# interface range GigabitEthernet0/1, FastEthernet0/2
SW1(config-if)# ip arp inspection trust
SW1(config-if)# exit

! Configure ARP Rate Limiting on access ports to prevent DoS
SW1(config)# interface FastEthernet0/3
SW1(config-if)# ip arp inspection limit rate 15
SW1(config-if)# exit
3. Verification Commands! Check dynamic binding database
SW1# show ip dhcp snooping binding

! Verify DAI statistics and dropped packet counts
SW1# show ip arp inspection statistics vlan 1

