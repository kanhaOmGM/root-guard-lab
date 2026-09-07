# Root Guard Lab - Cisco Packet Tracer

## Overview
This lab demonstrates the Spanning Tree Protocol (STP) Root Guard feature in Cisco Packet Tracer. It shows how an unauthorized switch can hijack the root bridge role by advertising a lower bridge priority, and how Root Guard prevents this security threat.

### Initial Network State
- **S2**: Root bridge with priority **4096**
- **S1**: Backup root bridge with priority **8192**
- **S3**: Access switch
- **PC1, PC2, PC3**: Test devices for traffic verification

## Lab Objectives
- Understand how STP elects the root bridge based on bridge priority
- Demonstrate a root bridge hijacking attack using an Attacker switch
- Observe how malicious root bridge takeover can intercept traffic
- Implement Root Guard to prevent unauthorized root bridge election
- Verify Root Guard operation and port state changes

## Lab Topology
```
         [S2 - Root Bridge]-[PC2]
         Priority: 4096
              |
      +-------+-------+
      |               |
    [S1]            [S3]
Priority: 8192        |
      |            [PC3]
   [PC1] 

[Attacker Switch] - connects to S2 (G1/0/1) and S3 (G1/0/2)
Priority: 0 (attempting to become root)
```

## Prerequisites
- Cisco Packet Tracer installed
- Basic understanding of Spanning Tree Protocol (STP)
- Familiarity with Cisco IOS commands
- Network topology set up with S1, S2, S3, and three PCs

## Lab Tasks

---

## Task List 1: Demonstrating Root Bridge Attack

### Task 1.1: Configure Attacker Switch
**On Attacker Switch:**
```cisco
enable
configure terminal
spanning-tree vlan 1 priority 0
exit
```

**What this does**: Sets the bridge priority to **0** (lowest possible), making it the most preferred candidate for root bridge election.

---

### Task 1.2: Connect Attacker to Network
**Physical connections:**
- Connect **Attacker G1/0/1** → **S2 G1/0/1**
- Connect **Attacker G1/0/2** → **S3 G1/0/2**

---

### Task 1.3: Verify Root Bridge Takeover
**On any switch (S1, S2, or S3):**
```cisco
enable
show spanning-tree

! Look for:
! Root ID Priority: 0
! Root ID Address: [Attacker's MAC address]
```

**Expected Result**: The Attacker switch should now be listed as the root bridge for VLAN 1.

---

### Task 1.4: Verify Traffic Interception
**On PC1 and PC2:**
```
ping 192.168.1.3
```

**Verify traffic path:**
```cisco
! On S2 or S3
show spanning-tree

! Check that traffic flows through the Attacker switch
! Use simulation mode in Packet Tracer to visualize the path
```

**Security Impact**: With the Attacker as root bridge, all inter-VLAN traffic is rerouted through the malicious switch, allowing the hacker to:
- Capture sensitive data (man-in-the-middle attack)
- Analyze network traffic patterns
- Potentially modify or drop packets

---

### Task 1.5: Note Blocked Ports on S3
**On S3:**
```cisco
show spanning-tree

! Document which interfaces are in blocking state
! These will change after the attack
```

**Record findings**: Note which ports transitioned to blocking after the Attacker became root.

---

## Task List 2: Implementing Root Guard Protection

### Task 2.1: Remove Attacker Switch
**Physically disconnect:**
- Remove cable from Attacker G1/0/1 to S2
- Remove cable from Attacker G1/0/2 to S3

**Verify:**
```cisco
! On S2
show spanning-tree

! Confirm S2 is root bridge again (priority 4096)
```

---

### Task 2.2: Enable Root Guard on S2
**On S2:**
```cisco
enable
configure terminal
interface range GigabitEthernet1/0/1 - 24
spanning-tree guard root
exit
```

**Purpose**: Protects all access/distribution ports on S2 from receiving superior BPDUs.

---

### Task 2.3: Enable Root Guard on S3 and S1
**On S3:**
```cisco
enable
configure terminal
interface range GigabitEthernet1/0/3 - 24
spanning-tree guard root
exit
```

**On S1:**
```cisco
enable
configure terminal
interface range GigabitEthernet1/0/3 - 24
spanning-tree guard root
exit
```

**Note**: We skip G1/0/1 and G1/0/2 on S3 and S1 as these may be uplink ports to legitimate switches.

---

### Task 2.4: Reconnect Attacker Switch
**Physical connections:**
- Reconnect **Attacker G1/0/1** → **S2 G1/0/1**
- Reconnect **Attacker G1/0/2** → **S3 G1/0/2**

---

### Task 2.5: Verify Root Guard Protection
**On S2:**
```cisco
show spanning-tree inconsistentports

! You should see ports in root-inconsistent state
```

**On S2 (check specific port):**
```cisco
show spanning-tree interface GigabitEthernet1/0/1

! Status should show: Root-inconsistent (Blocking)
```

**Expected Behavior:**
- Ports receiving superior BPDUs enter **root-inconsistent** state
- Ports are effectively blocked (err-disabled)
- Network topology remains stable

---

### Task 2.6: Verify S2 Remains Root Bridge
**On any switch:**
```cisco
show spanning-tree

! Verify:
! Root ID Priority: 4096
! Root ID Address: [S2's MAC address]
```

**Success Criteria:**
- S2 is still the root bridge
- Attacker's ports on S2/S3 are in root-inconsistent state
- PC1 and PC2 can still ping PC3
- Traffic does NOT flow through the Attacker switch

---

## Video Demonstrations
Check the `/videos` folder in this repository for recorded demonstrations:


## Verification Commands Summary

### Check Current Root Bridge
```cisco
show spanning-tree | include Root
show spanning-tree summary
```

### Verify Root Guard Status
```cisco
show running-config | include guard
show spanning-tree interface GigabitEthernet1/0/1
```

### Check for Inconsistent Ports
```cisco
show spanning-tree inconsistentports
```

### View Detailed Port Status
```cisco
show spanning-tree detail
```

## Expected Results Comparison

| Metric | Without Root Guard (Task 1) | With Root Guard (Task 2) |
|--------|----------------------------|-------------------------|
| Root Bridge | Attacker (Priority 0) | S2 (Priority 4096) |
| S2 G1/0/1 State | Forwarding | Root-Inconsistent (Blocked) |
| S3 G1/0/2 State | Forwarding | Root-Inconsistent (Blocked) |
| Traffic Path | Through Attacker | Direct S2 → S3 path |
| Security Risk | HIGH - Traffic exposed | LOW - Attack prevented |

## Security Analysis

### Attack Vector
1. Malicious actor connects unauthorized switch to network
2. Switch configured with priority 0 to become root bridge
3. STP reconvergence routes all traffic through attacker's switch
4. Attacker can capture, analyze, or manipulate traffic

### Root Guard Protection
1. Root Guard monitors incoming BPDUs on protected ports
2. If superior BPDU detected (lower priority), port transitions to root-inconsistent
3. Port enters blocking state, preventing topology change
4. Legitimate root bridge (S2) maintains its role
5. Network topology remains stable and secure

## Key Concepts

### Bridge Priority Values
- **Range**: 0 - 61440 (increments of 4096)
- **Default**: 32768
- **Lower = Higher Priority**: 0 is the most preferred, 61440 is least preferred
- **Lab Values**:
  - Attacker: 0 (highest priority)
  - S2: 4096 (root bridge)
  - S1: 8192 (backup root)
  - Default: 32768

### Root Guard Port States
- **Forwarding**: Normal operation, no superior BPDUs received
- **Root-Inconsistent**: Superior BPDU detected, port blocked to prevent topology change
- **Recovery**: Port returns to forwarding when superior BPDUs stop

### When to Use Root Guard
  **Enable on:**
- Access ports connecting to end devices
- Ports facing untrusted networks
- Distribution layer downlink ports

  **Do NOT enable on:**
- Uplink ports to core/distribution switches
- Ports where root bridge should legitimately exist
- Trunk ports between trusted switches

## Troubleshooting

### Issue: Port Stuck in Root-Inconsistent State
```cisco
show spanning-tree inconsistentports
show log | include ROOTGUARD
```
**Solution**: 
- Verify attacker switch is disconnected or reconfigured
- Check if priority needs adjustment on legitimate switches
- Disable Root Guard if connection is legitimate: `no spanning-tree guard root`

### Issue: Root Guard Not Triggering
```cisco
show running-config interface GigabitEthernet1/0/1
show spanning-tree interface GigabitEthernet1/0/1 detail
```
**Checklist**:
- Verify Root Guard is configured on the correct interface
- Confirm spanning-tree is enabled on VLAN 1
- Check if port is in the correct VLAN
- Verify Attacker switch priority is actually 0

### Issue: Network Connectivity Lost
- Verify S2 is still reachable and functioning as root bridge
- Check that non-guarded uplink ports are forwarding
- Confirm PCs can still communicate through legitimate paths

## Additional Resources
- [Cisco STP Root Guard Configuration](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/10588-74.html)
- [Understanding Spanning Tree Protocol](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/5234-5.html)
- [STP Security Best Practices](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/24248-147.html)

## Lab Files
- `root-guard-lab.pkt` - Packet Tracer file
- `/videos` - Task demonstration videos

## Challenge Extension Ideas
1. Configure BPDU Guard in addition to Root Guard
2. Implement Root Guard with multiple VLANs
3. Test recovery time when attacker is removed
4. Compare Root Guard with BPDU Filter
5. Set up syslog monitoring for Root Guard events

## Notes
- **Bridge Priority 0** is the lowest possible value, giving highest priority in root bridge election
- Root Guard is a preventative measure, not detective
- Always document your Root Guard deployment for network operations team
- Root Guard works per-VLAN in PVST+ environments

## Author
Om Prakash Sabu

## Contributing
Contributions welcome! Feel free to:
- Add more test scenarios
- Improve documentation
- Submit bug fixes
- Share your lab results
---
