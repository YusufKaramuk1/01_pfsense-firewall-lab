# Internal Network Firewall Lab – pfSense, Windows & Kali

> This repository documents an isolated VirtualBox internal network lab created for internship training and internal network behavior analysis.

---

![Lab Topology](evidence/00_lab_topology.png)

---

## Objective

This lab evaluates how a Windows host behaves when inbound ports are filtered by host firewall rules, and whether internal interaction and file retrieval remain possible from an attacker located in the same LAN segment.

The goal is to observe:

- Host discovery feasibility  
- Firewall filtering behavior at Layer 3/4  
- Transition to Layer 2 validation when L3/L4 access is blocked  
- Internal access attempt and file proof  

---

## Lab Environment

| Role | OS / Hostname | IP Address |
|------|---------------|------------|
| Firewall Gateway | pfSense | `192.168.50.1` |
| Attacker | Kali Linux | `192.168.50.10` |
| Victim | Windows 11 (EVAL) | `192.168.50.11` |
| Network | VirtualBox Internal | `192.168.50.0/24` |

---

# Step 1 – Attacker Network Verification (`ip a`)

First, the attacker machine’s IP configuration was verified to confirm it is in the same subnet as the victim.

Purpose:
- Confirm correct subnet placement  
- Ensure attacker is in the same LAN as the target  

![Kali IP Configuration](evidence/10_kali_ip_config_ip_a.png)

---

# Step 2 – Network Discovery (`nmap -sn`)

A subnet discovery scan was performed to identify active hosts in the internal segment.

Purpose:
- Identify live systems  
- Confirm victim and gateway presence  

![Network Discovery](evidence/01_network_discovery_nmap_sn.png)

---

# Step 3 – Default Target Scan (`nmap <target>`)

A default scan was executed against the Windows victim.

Purpose:
- Identify exposed services  
- Observe firewall filtering behavior  

![Default Target Scan](evidence/02_target_port_scan_nmap_default.png)

**Observation**
- Results returned as **filtered**, indicating inbound filtering.

---

# Step 4 – SYN Scan Validation (`nmap -sS <target>`)

A stealth SYN scan was performed as an additional verification step.

Purpose:
- Try different scan methods  
- Confirm whether filtering behavior is consistent  

![SYN Scan](evidence/05_syn_scan_nmap_sS.png)

**Observation**
- Results remained **filtered** across scan types.

At this stage, it was concluded that Layer 3/4 access through traditional inbound scanning was not viable.

---

# Step 5 – Layer 2 Validation (`arp -a` / `ip neigh`)

Since Layer 3/4 access appeared blocked, Layer 2 visibility was validated next.

Purpose:
- Confirm same broadcast domain  
- Verify MAC resolution of victim and gateway  
- Validate LAN-level presence despite filtered ports  

![ARP and Neighbor Table](evidence/06_arp_and_ip_neigh.png)

---

# Step 6 – IP Forwarding Enablement

IP forwarding was enabled on the attacker machine to prepare for potential relay / routing scenarios during LAN testing.

Purpose:
- Enable forwarding capability on attacker machine  
- Prepare for LAN-level traffic manipulation scenarios  

![IP Forwarding](evidence/08_enable_ip_forwarding.png)

---

# Step 7 – SMB Connection Attempt (Failed)

An SMB connection attempt was made against the administrative share.

Purpose:
- Test SMB reachability under filtered port conditions  
- Observe authentication / access behavior  

![SMB Timeout](evidence/03_smb_attempt_timeout.png)

Result:
- Initial attempt failed (timeout / denied behavior observed).

---

# Step 8 – SMB Interaction (Successful)

A subsequent SMB attempt succeeded and allowed directory listing / navigation.

Purpose:
- Validate successful internal interaction  
- Confirm SMB access becomes possible under the lab conditions  

![SMB Access](evidence/04_smb_access_success.png)

---

# Step 9 – File Proof (Read + Victim-Side Confirmation)

After access was obtained, a known file was read on the attacker system and verified on the victim system for proof.

Purpose:
- Confirm file retrieval / access  
- Provide integrity evidence (victim-side confirmation)

![File Read on Kali](evidence/07_file_read_cat_nbr.png)

![Windows File Proof](evidence/09_windows_file_proof.png)

---

# Key Observations

- Different Nmap scan methods consistently returned **filtered** results  
- L3/L4 inbound access appeared blocked by firewall filtering  
- Layer-2 presence was confirmed via ARP/MAC resolution  
- SMB attempts showed both failure and success behavior  
- File access was proven with attacker-side read and victim-side proof  

---

# Conclusion

This lab demonstrates that:

Even when inbound scanning indicates fully filtered ports, internal network positioning and Layer-2 visibility can still enable meaningful interaction paths, including SMB-based access and file validation under certain conditions.

A layered defensive approach is required, including:

- Segmentation and internal ACLs  
- Strict share and credential controls  
- Monitoring for lateral movement  
- Endpoint hardening and logging  

---
