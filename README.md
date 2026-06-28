# Enterprise Network Architecture, Cryptographic Tunneling, and Infrastructure Hardening

This repository houses the comprehensive core network configurations, topology files, and security implementations developed across two structural milestones for an enterprise infrastructure expansion project. The architecture transitions from a hardened single-site setup into a multi-branch infrastructure utilizing advanced security frameworks, server-blind cryptographic tunnels, and robust Layer 2/Layer 3 defense-in-depth principles.

---

## Technical Architecture Overview

The system architecture encapsulates two evolutionary design phases:

### 1. Milestone 1: Corporate Headquarters & Core Services Baseline

* **Internal Segmentation**: Divided into functional subnet boundaries including the Server Room (`192.168.1.0/24`), Technical Support (`192.168.2.0/24`), and Employee Offices (`192.168.3.0/24`).


* **Hardened Perimeter DMZ**: Isolated public-facing application nodes (`172.16.1.0/24`) managing dedicated Web, FTP, and Email service architectures.


* **Dynamic Core Routing**: Complete dynamic internal accessibility convergence driven by Open Shortest Path First (OSPF Process 1, Area 0) across edge frameworks.



### 2. Milestone 2: Multi-Branch Expansion & Secure Cryptographic Tunneling

* **Branch Office Deployment**: Provisioned an autonomous remote office environment running on the private `192.168.5.0/24` subnet space.


* **WAN Site-to-Site IPsec VPN**: Constructed an encrypted network barrier traversing an unsecure public ISP simulator (Router R3) to seamlessly protect inter-branch internal transit streams.


* **Infrastructure Rectification**: Resolved a structural subnet overlap conflict by migrating the R1-R2 core interconnection link from `172.16.3.0/24` to a unique `172.16.4.0/24` block, preserving `172.16.3.0/24` exclusively for the new branch edge WAN route.



---

## Complete Topology Visuals

### Inter-Branch Network Expansion Diagram

> The complete visual blueprint showcasing the corporate headquarters, DMZ segment, public internet routing path, and the extended branch network architecture.


<img width="1745" height="836" alt="image" src="https://github.com/user-attachments/assets/8ef5a11d-56ad-458b-939d-b40e4eb1c132" />



---

## Core Security & Technical Phases

### Phase 1: Dynamic Interior Routing Architecture (OSPF)

To maintain routing reliability and eliminate clean-text static routes, an interior dynamic routing environment was formed via OSPF. Symmetrical routing boundaries automatically map corporate networks across provider links.

```cisco
! New Branch Border Router OSPF Deployment
router ospf 1
 log-adjacency-changes
 network 192.168.5.0 0.0.0.255 area 0
 network 172.16.3.0 0.0.0.255 area 0

```


<img width="1162" height="595" alt="image" src="https://github.com/user-attachments/assets/5102a7b2-2c45-440c-a1bc-43e6c982ce46" />


---

### Phase 2: Cryptographic Infrastructure (Site-to-Site IPsec VPN)

Because corporate data crosses public intermediary nodes (R3 ISP), a site-to-site IPsec architecture was constructed between Headquarters Router R2 and the NewBranchStub router.

#### Cryptographic Matrix Specifications:

* **Phase 1 ISAKMP Policy**: Advanced Encryption Standard 256-bit keys (AES-256), Secure Hash Algorithm (SHA) message authentication code integrity check, Diffie-Hellman Group 2 key exchange, and an explicit Pre-Shared Key sequence.


* **Phase 2 Transform-Set**: Encapsulating Security Payload parameters combining AES data encryption (`esp-aes`) alongside SHA hash verification protocols (`esp-sha-hmac`).



```cisco
! Symmetrical Crypto Parameters Implementation
crypto isakmp policy 10
 encr aes 256
 hash sha
 authentication pre-share
 group 2
crypto isakmp key vpn_pass address 172.16.2.1
!
crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac
!
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 172.16.2.1
 set transform-set VPN-SET
 match address 110

```


<img width="1067" height="221" alt="image" src="https://github.com/user-attachments/assets/512df0ed-efd3-4d87-bea7-730650dcf135" />



<img width="1257" height="713" alt="image" src="https://github.com/user-attachments/assets/2be780bf-fcc1-4891-be88-be9436d312ca" />


---

### Phase 3: Identity Management & Centralized Logging (AAA Services)

To replace high-risk independent local accounts, a centralized Authentication, Authorization, and Accounting (AAA) server environment was introduced utilizing **TACACS+** protocols.

```cisco
! Hardening Control Configuration for Centralized Identity Control
aaa new-model
aaa authentication login default group tacacs+ local
tacacs-server host 192.168.1.3 key tacacsPa55

```

#### Engineering Remediation & Troubleshooting Note:

> During early staging environments, terminal authentications triggered an invalid login error loop. Comprehensive diagnostic trace analysis discovered an authorization source IP validation fault: the TACACS+ registry expected traffic sourced from the external serial gateway interface (`172.16.3.2`), whereas the router naturally originated packets via its localized internal interface (`192.168.5.1`). Updating client targets on the AAA host interface resolved credential verification issues across the network matrix.
> 
> 


<img width="1158" height="656" alt="image" src="https://github.com/user-attachments/assets/e2fd1161-954e-40a8-a405-01b5b829c2d3" />


#### Central Syslog Event Auditing

All active enterprise hardware units securely forward event logging information, real-time administrative changes, and state tracking structures directly to an isolated target storage server (`192.168.1.2`).


<img width="1190" height="250" alt="image" src="https://github.com/user-attachments/assets/5c3a1f5a-2525-4b5c-8ec0-f3402b111036" />


---

### Phase 4: Perimeter Defense & Control Plane Hardening

#### Brute-Force Protective Shields

To mitigate automated password-guessing tools and unauthorized administrative entry vectors, automated block thresholds were applied at interface lines. If any authentication stream receives 3 failed credential entries inside a sliding 60-second window, the device engine enforces a 300-second (5-minute) Quiet Mode lockout.

```cisco
login block-for 300 attempts 3 within 60
login on failure log
login on-success log

```

#### Management Plane Encryption

* Cleartext terminal accessibility protocols (like Telnet) are strictly banned globally across all appliances.


* All remote infrastructure administration lines are restricted to encrypted **Secure Shell (SSH v2)** pipelines generated using custom domain identities tied to an asymmetric 1024-bit RSA modulus parameter.


* Global device configuration variables, local administrative passphrases, and privileged EXEC level access structures are obfuscated using standard password encryption and cryptographic MD5 hashing algorithms.




<img width="1117" height="242" alt="image" src="https://github.com/user-attachments/assets/4fc60ceb-0afd-47b6-8411-3da10ae6e862" />


---

### Phase 5: Layer 2 Infrastructure Hardening (Port Security)

Access layer edge connectivity points are hardened against physical rogue hardware drops, MAC spoofing attempts, and MAC table overflow floods using strict MAC-layer security.

```cisco
! Standard Interface Sticky Port Hardening Example
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

```

#### Inter-Switch Trunk Interface Rectification:

> During Layer 2 staging, AccessSwitch1 port `FastEthernet0/2` crashed into an automated `%PM-4-ERR DISABLE` link state. Port-security restrictions had been mistakenly applied to a multi-MAC distribution backbone inter-switch Trunk link port. Stripping port-security parameters from the Trunk line completely normalized communications across the internal switch matrix.



<img width="1075" height="776" alt="image" src="https://github.com/user-attachments/assets/2f4dbe1c-04b6-40e2-aa98-696d07443f60" />


---

### Phase 6: Traffic Filtering & Network Optimization (ACL & IPS)

* **Cisco IOS IPS Ingress Rule**: Configured a signature monitoring profile named `iosips` targeting the employee segment gateway (`GigabitEthernet0/2` on R1) to block unauthorized ICMP ping traffic trying to scan core server coordinates.


* **Extended Access Control Lists (ACL)**: Deployed the `DEPT_POLICY` filter mechanism on corporate gateways to enforce the Principle of Least Privilege:
* Restricts specific Meeting Room terminals (`192.168.3.0/24` network bounds / host `192.168.3.9`) from establishing unauthorized connection states with the DMZ FTP host (`172.16.1.3`).


* Permits broad, verified corporate access pipelines to the public corporate Web Server (`172.16.1.4` on port 80).





```cisco
ip access-list extended DEPT_POLICY
 deny tcp host 192.168.3.9 host 172.16.1.3 eq ftp
 permit tcp 192.168.3.0 0.0.0.255 host 172.16.1.4 eq www
 permit ip any any

```

---

## Technical Audit & Verification Playbook

To audit and verify the security settings in the Cisco Packet Tracer ecosystem:

1. **Verify Volatile Memory Commitment**: Ensure all active runtime states are written directly into non-volatile storage to maintain system stability across power cycles.


```cisco
Router# copy running-config startup-config

```




2. **Audit Crypto Tunnel Operations**: Run `show crypto ipsec sa` on perimeter nodes during active cross-branch data transfers to confirm structural data encryption metrics.
3. **Analyze AAA Connectivity Status**: Execute management login operations from an internal administration workstation terminal to verify real-time log tracking streams inside the central Syslog environment.
4. **Trigger Port Security Violations**: Map an unauthorized client address against an interface protected by a sticky MAC binding profile to verify automatic shutdown responses.
