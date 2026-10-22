# ROUTING AND ACCESS CONTROL LISTS (ACLs)

Cisco Packet Tracer Project

Two Routers, Two LANs and Four PCs

## 1. Introduction

A computer network allows devices to communicate and share resources. When devices belong to different IP networks ***(Prod. Dept and Sales Dept.)***, routers are needed to forward traffic between them.

An Access Control List (ACL) is a collection of rules used on routers to permit or deny network traffic according to specified criteria, such as source IP address, destination IP address, or protocol.

In this practical, i will:

- Build a network with two routers, two switches and four PCs.
- Configure IP addresses on router interfaces and PCs.
- Configure static routes between the two LANs.
- Test communication before applying ACLs.
- Configure a standard ACL and understand its limitations.
- Configure an extended ACL to control which PCs can initiate pings.
- Verify and troubleshoot the final configuration.

## 2. Objectives

At the end of this practical, anyone following should be able to:

1. Configure router interfaces using Cisco IOS commands.
2. Assign IP addresses and default gateways to PCs.
3. Configure static routing between different networks.
4. Explain the difference between standard and extended ACLs.
5. Apply ACLs in the correct direction on router interfaces.
6. Test connectivity and troubleshoot access-control problems.

## 3. Network topology

Use the following logical topology.
### Prod. Dept.
PC1

192.168.1.10/24

Gateway: 192.168.1.1

Switch 1

Router R1

G0/1: 192.168.1.1/24

G0/0: 192.168.3.1/24

Inter-router network

192.168.3.0/24

Router R2

G0/0: 192.168.3.2/24

G0/1: 192.168.2.1/24

Switch 2

### Prod. Dept.
PC3

192.168.2.10/24

Gateway: 192.168.2.1

PC2 is also connected to Switch 1, and PC4 is also connected to Switch 2.

### Devices required
- 2 routers (Cisco 1941 or 2911)
- 2 switches (Cisco 2960)
- 4 PCs
- Copper Ethernet cables (Copper Straight-Through Cable and Copper Cross-over Cable)

Connect each PC to its local switch, each switch to its local router, and R1 to R2. 
Use Copper Straight-Through Cables to connect the PCs to the Switches while using Copper Cross-over Cable to connect R1 to R2.
The interface names in this guide assume the routers have GigabitEthernet0/0 and GigabitEthernet0/1 interfaces.
Use the actual interface names on your devices if they differ.

## 4. IP addressing table

Use this addressing plan throughout the practical.

| Device | Interface | IP address | Subnet mask |
| --- | --- | --- | --- |
| R1 | G0/1 | 192.168.1.1 | 255.255.255.0 |
| R1 | G0/0 | 192.168.3.1 | 255.255.255.0 |
| R2 | G0/0 | 192.168.3.2 | 255.255.255.0 |
| R2 | G0/1 | 192.168.2.1 | 255.255.255.0 |
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 |
| PC2 | NIC | 192.168.1.20 | 255.255.255.0 |
| PC3 | NIC | 192.168.2.10 | 255.255.255.0 |
| PC4 | NIC | 192.168.2.20 | 255.255.255.0 |

The default gateways are:

- PC1 and PC2: `192.168.1.1`
- PC3 and PC4: `192.168.2.1`

Important: R1 and R2 must have different addresses on the inter-router network. R1 uses `192.168.3.1`, while R2 uses `192.168.3.2`. Never assign the same IP address to both interfaces.

## 5. Configure Router R1

### Step 1: Enter privileged and global configuration modes

Click R1, open the CLI, and enter:

```
enable
configure terminal
hostname R1
```

### What does commands means and what they do;
- `enable` enters privileged EXEC mode.
- `configure terminal` enters global configuration mode.
- `hostname R1` sets the router's name.

### Step 2: Configure the LAN interface

```
interface gigabitEthernet 0/1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
```

This interface connects R1 to Switch 1 and the `192.168.1.0/24` LAN.

### Step 3: Configure the inter-router interface

```
interface gigabitEthernet 0/0
ip address 192.168.3.1 255.255.255.0
no shutdown
exit
```

This interface connects R1 to R2 over the `192.168.3.0/24` network.

### Step 4: Configure the static route

```
ip route 192.168.2.0 255.255.255.0 192.168.3.2
end
write memory
```

This tells R1 to forward traffic destined for `192.168.2.0/24` to R2 at `192.168.3.2`.

## 6. Configure Router R2

### Step 1: Enter configuration mode

```
enable
configure terminal
hostname R2
```

### Step 2: Configure the inter-router interface

```
interface gigabitEthernet 0/0
ip address 192.168.3.2 255.255.255.0
no shutdown
exit
```

### Step 3: Configure the LAN interface

```
interface gigabitEthernet 0/1
ip address 192.168.2.1 255.255.255.0
no shutdown
exit
```

### Step 4: Configure the return static route

```
ip route 192.168.1.0 255.255.255.0 192.168.3.1
end
write memory
```

This tells R2 how to reach the LAN behind R1.

### Step 5: Verify the interfaces

On both routers, run:

```
show ip interface brief
```

The configured interfaces should show `up` in both the Status and Protocol columns once the cables and remote interfaces are working.

Also check the routing table:

```
show ip route
```

R1 should have a static route to `192.168.2.0/24` via `192.168.3.2`, and R2 should have a static route to `192.168.1.0/24` via `192.168.3.1`.

## 7. Configure the PCs

For each PC, select Desktop → IP Configuration in Packet Tracer.

### PC1 On Prod. Dept.

- IP address: `192.168.1.10`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.1.1`

### PC2 On Prod. Dept.

- IP address: `192.168.1.20`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.1.1`

### PC3 On Sales Dept.

- IP address: `192.168.2.10`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.2.1`

### PC4 On Sales Dept.

- IP address: `192.168.2.20`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.2.1`

### Test connectivity before applying ACLs

On PC1, open Desktop → Command Prompt:

```
ping 192.168.1.1
ping 192.168.3.2
ping 192.168.2.1
ping 192.168.2.10
```

The pings should succeed when the routing and addressing are correct. The first ping can time out while ARP resolves; repeat to make sure all 4 packects are returned completely.

Also test from PC3:

```
ping 192.168.1.10
```

Do not proceed to ACL configuration until basic connectivity works in both directions.

## 8. What is an ACL?

An ACL is a numbered or named list of rules that determines whether packets are permitted or denied.

There are two common IPv4 ACL types in this practical.

| Feature | Standard ACL | Extended ACL |
| --- | --- | --- |
| Matches | Source IP address | Source, destination, protocol and, where supported, ports |
| Numbered range | 1–99 and 1300–1999 | 100–199 and 2000–2699 |
| Typical purpose | Restrict traffic based on its source | Control specific traffic between hosts or networks |
| Example | Block a source host | Block ICMP echo requests from one host to another |

ACL rules are processed from top to bottom. The first matching rule determines the action.

Implicit deny: Every ACL has an invisible deny rule at the end. If a packet does not match a permit statement, it is denied. Therefore, add an appropriate permit statement when other traffic must remain allowed.

### ACL direction

- `in` — checks packets as they enter an interface.
- `out` — checks packets as they leave an interface.

<img width="1536" height="1024" alt="ACL Direction_ In vs Out" src="https://github.com/user-attachments/assets/a0c8e3cd-cf9a-4730-8cd7-95a1724f2d59" />


An ACL must be applied to an interface and in a direction that the relevant packets actually traverse.

## 9. Configure a standard ACL

### Objective

Block PC1 (`192.168.1.10`) based on its source IP address using a standard ACL.

A standard ACL does not check the destination address. For this project, i applied it outbound on R2 G0/1, the interface leading to the R2 LAN.

### Step 1: Create the ACL on R2

```
enable
configure terminal
access-list 11 deny host 192.168.1.10
access-list 11 permit any
```

Explanation:

- `access-list 11` creates standard ACL number 11.
- `deny host 192.168.1.10` denies traffic with that source address.
- `permit any` permits traffic from other sources that reaches this ACL.

### Step 2: Apply the ACL

```
interface gigabitEthernet 0/1
ip access-group 11 out
end
write memory
```

The ACL checks packets leaving R2 toward the `192.168.2.0/24` LAN.

Note; you can as well do it for inbound traffics, all you need to do is change the outbound to inbound and change the interface to G0/0.

### Step 3: Test the ACL

- PC1 pinging PC3 should fail.
- PC2 pinging PC3 should succeed.
- PC1 may still communicate with devices on other networks if the traffic does not pass through this ACL.

### Limitation of a standard ACL

A standard ACL cannot distinguish PC1-to-PC3 traffic from PC1-to-PC4 traffic because it checks only the source IP address. In this example, it blocks traffic from PC1 to the entire LAN behind R2.

It can also affect replies and other traffic that crosses the interface. For precise control of which host can ping which destination, use an extended ACL.

To remove ACL 11 later, first remove its interface association, then delete the ACL:

```
configure terminal
interface gigabitEthernet 0/1
no ip access-group 11 out
exit
no access-list 11
end
write memory
```

## 10. Configure an extended ACL

This can be used as the main exercise for the four-PC network.

### Objective

Configure the network so that:

- PC1 cannot initiate a ping to PC3.
- PC3 can initiate a ping to PC1.
- PC2 and PC4 are not blocked by this specific rule.
- Other IP traffic is permitted by this ACL.

I will use an extended named ACL on R1's G0/1, where traffic from PC1 and PC2 enters R1.

### Step 1: Remove any conflicting ACL

If ACL 11 is still applied on R2, remove it so it does not interfere with the test:

```
R2# configure terminal
R2(config)# interface gigabitEthernet 0/1
R2(config-if)# no ip access-group 11 out
R2(config-if)# exit
R2(config)# no access-list 11
R2(config)# end
```

If you applied ACL 11 in a different direction, remove it from that interface using the matching `no ip access-group` command.

### Step 2: Create an extended ACL on R1

```
enable
configure terminal
ip access-list extended BLOCK-PC1-TO-PC3
deny icmp host 192.168.1.10 host 192.168.2.10 echo
permit ip any any
exit
```

Explanation:

- `ip access-list extended BLOCK-PC1-TO-PC3` creates a named extended ACL.
- `deny icmp host 192.168.1.10 host 192.168.2.10 echo` blocks ICMP echo requests from PC1 to PC3.
- `permit ip any any` permits all other IPv4 traffic that reaches this ACL.

### Step 3: Apply the ACL inbound on R1 G0/1

```
interface gigabitEthernet 0/1
ip access-group BLOCK-PC1-TO-PC3 in
end
write memory
```

Why G0/1? Packets from PC1 enter R1 through G0/1. The ACL blocks the specified ping request before it can be routed toward R2.

### Step 4: Test the extended ACL

On PC1, run:

```
ping 192.168.2.10
```

Expected: the ping fails because the echo requests are denied.

On PC3, run:

```
ping 192.168.1.10
```

Expected: the ping succeeds, assuming routing is working. The echo request from PC3 is permitted, and the echo reply from PC1 is also permitted because it does not match the deny rule.

Test the other PCs as well:

| Source | Destination | Expected result |
| --- | --- | --- |
| PC1 | PC3 | Blocked |
| PC3 | PC1 | Allowed |
| PC2 | PC3 | Allowed |
| PC4 | PC1 | Allowed |

This ACL controls only the specified ICMP echo request. It does not block all traffic between PC1 and PC3, and it does not prevent PC1 from pinging PC4.

## 11. Verification commands

Use these commands to inspect your configuration.

| Command | Purpose |
| --- | --- |
| `show ip interface brief` | Displays interface addresses and status |
| `show ip route` | Displays the routing table |
| `show access-lists` | Displays ACL rules and match counters |
| `show ip interface g0/1` | Shows ACLs and other settings on G0/1 |
| `show running-config` | Displays the active configuration |
| `show startup-config` | Displays the saved configuration |

On R1:

```
show access-lists
```

You should see a rule similar to:

```
Extended IP access list BLOCK-PC1-TO-PC3
    deny icmp host 192.168.1.10 host 192.168.2.10 echo
    permit ip any any
```

The deny counter should increase when PC1 tries to ping PC3. The exact output formatting may vary by Packet Tracer version.

To verify interface placement:

```
show ip interface gigabitEthernet 0/1
```

Confirm that `BLOCK-PC1-TO-PC3` is listed as an inbound access list.

## 12. Common mistakes and their solutions

| Mistake | Explanation | Solution |
| --- | --- | --- |
| Duplicate IP addresses | Two interfaces share an address | Assign unique IPs in the correct subnet |
| Incorrect next-hop address | Static route points to the wrong router address | R1 uses `192.168.3.2`; R2 uses `192.168.3.1` |
| Interface is down | Port is shut down or cable is disconnected | Use `no shutdown` and check cabling |
| Wrong default gateway | PC cannot route traffic outside its LAN | Use the IP address of its local router interface |
| Standard ACL blocks too much | It matches only the source IP | Use an extended ACL to match source and destination |
| ACL applied in the wrong direction | Packets do not cross the ACL as expected | Trace the packet path and choose the correct interface and direction |
| Missing permit statement | Unmatched traffic hits implicit deny | Add the required permit rules |
| Ping fails in one direction | Replies may be blocked or routing may be incomplete | Inspect both routes and ACLs along the forward and return paths |
| `show access-listt` gives an error | The command is misspelled | Use `show access-lists` |

## 13. Summary of the final configurations

The following is the intended configuration for the final ACL exercise, assuming you have completed the base router configuration and PC addressing.

## R1-Final configuration additions

```
enable
configure terminal
ip route 192.168.2.0 255.255.255.0 192.168.3.2

ip access-list extended BLOCK-PC1-TO-PC3
 deny icmp host 192.168.1.10 host 192.168.2.10 echo
 permit ip any any
exit

interface gigabitEthernet 0/1
 ip access-group BLOCK-PC1-TO-PC3 in
exit

end
write memory
```

## R2-Final routing configuration

```
enable
configure terminal
ip route 192.168.1.0 255.255.255.0 192.168.3.1
end
write memory
```

No ACL is required on R2 for the final extended-ACL exercise. Remove any previous ACL 11 association that would interfere with the tests.

<img width="1919" height="1079" alt="Static- -Extended-ACL" src="https://github.com/user-attachments/assets/6326e78f-9cc0-4006-b2da-0863eb708d8b" />

## 14. Conclusion

In this project, two LANs were connected through R1 and R2 using static routing. PCs were assigned unique IP addresses and configured with their respective default gateways. Connectivity was verified before access restrictions were introduced.

A standard ACL was used to demonstrate source-based filtering, while an extended ACL was used to block a specific ICMP echo request between PC1 and PC3. Correct interface placement, traffic direction, permit rules, and return-path routing are essential for ACLs to behave as expected.

The key lesson is that routing determines where traffic goes, while ACLs determine whether matching traffic is allowed to pass.
