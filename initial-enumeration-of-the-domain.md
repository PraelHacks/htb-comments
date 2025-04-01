# Initial Enumeration of the Domain Findings

## Issue #1: mDNS Traffic Not Initially Captured in Wireshark

In the Hack The Box [Penetration Tester learning path -> Active Directory Enumeration & Attacks module -> Initial Enumeration of the Domain section](https://academy.hackthebox.com/module/143/section/1265), learners are unable to replicate capturing mDNS packets in Wireshark as outlined in the materials, leading to confusion.

![Alt text](./images/ea-wireshark-mdns.webp?raw=true "Hack The Box Wireshark Screenshot")

If they follow the course materials step by step, they will only be able to capture NBNS and ARP packets:

![Alt text](./images/NBNS-traffic.png?raw=true "Hack The Box Wireshark Screenshot")

mDNS and LLMNR packets will only start appearing once they start Responder as outlined in a later part of the materials.

![Alt text](./images/full-traffic.png?raw=true "Hack The Box Wireshark Screenshot")

## Steps to Reproduce

1. Connect to the target system via RDP with user "htb-student" and password "HTB_@cademy_stdnt!"

   ```bash
   xfreerdp /v:10.129.216.18 /u:htb-student /p:"HTB_@cademy_stdnt\!"
   ```

2. Start Wireshark as root:

   ```bash
   sudo -E wireshark
   ```

3. Begin capturing packets with Wireshark on the interface (`ens224`).

4. Inspect captured packets in Wireshark:
   - Use display filters such as `mdns` and `!NBNS && !arp` to filter packets.
   - **Observed:** Only NBNS and ARP packets are visible.
   - **Expected (based on course screenshot):** mDNS packets should also be visible.

5. Start Responder with the following command:

   ```bash
   sudo responder -I ens224 -A
   ```

6. Observe Wireshark capture again:
   - mDNS and LLMNR packets are now visible.

## Cause Analysis

By default, the Parrot VM does not subscribe to the multicast address used by mDNS (`224.0.0.251`) and LLMNR (`224.0.0.252`).

**Before starting Responder:**

```bash
ip maddr show dev ens224
```

Output:

```bash
3: ens224
    link  01:00:5e:00:00:01
    link  33:33:00:00:00:01
    link  33:33:ff:aa:25:da
    inet  224.0.0.1
    inet6 ff02::1:ffaa:25da
    inet6 ff02::1
    inet6 ff01::1
```

Running Responder in passive mode subscribes our machines to LLMNR and mDNS protocols. This allows Wireshark to capture the traffic for these protocols.

**After starting Responder:**

```bash
ip maddr show dev ens224
```

Output:

```bash
3: ens224
    link  01:00:5e:00:00:01
    link  33:33:00:00:00:01
    link  33:33:ff:aa:25:da
    link  01:00:5e:00:00:fb
    link  01:00:5e:00:00:fc
    inet  224.0.0.252
    inet  224.0.0.251
    inet  224.0.0.1
    inet6 ff02::1:ffaa:25da
    inet6 ff02::1
    inet6 ff01::1
```

## Recommendations

Consider implementing one of the following solutions:

- Update the course materials so that Responder is used **before** running Wireshark (this might need changes to the flow of the content).
- Update the course materials so the Wireshark capture screenshot only shows NBNS at the start, and have another screenshot with mDNS and LLMNR after Responder is started.
- Adjust VM configuration to manually subscribe to multicast addresses at the beginning.

## Comments

Even if we decide not to change anything, having this information is helpful in case we get questions from learners.

---

## Issue #2: Tcpdump Output not showing packets

In the same section as Issue #1, learners are unable to replicate capturing network traffic using tcpdump. 

![Alt text](./images/tcpdump-example.png?raw=true "Hack The Box Tcpdump Screenshot")

If they run the tcpdump command, little to no packets are shown in the capture. Once they terminate the capture, the summary indicates that nearly all packets have been dropped by the kernel.

![Alt text](./images/tcpdump-actual.png?raw=true "Actual Tcpdump Screenshot")

## Steps to Reproduce

1. Connect to the target system via RDP or SSH with user **htb-student** and password **HTB_@cademy_stdnt!**  
   ```bash
   xfreerdp /v:10.129.216.18 /u:htb-student /p:"HTB_@cademy_stdnt\!"
   ```
   
2. Start tcpdump as root:
   ```bash
   sudo tcpdump -i ens224
   ```
   
3. Observe that few or no packets appear in the live capture. After terminating the command, the output shows that most packets were dropped by the kernel.

## Cause Analysis

By default, `tcpdump` tries to resolve IP addresses and ports into hostnames and service names. It appears that these lookups fail or time out for this lab. Meanwhile, packets continue to arrive on the interface, quickly filling the kernel’s capture buffer before `tcpdump` can process them. 

## Recommendations

- Disable Name Resolution  using the `-nn` flag to avoid DNS and service lookups:
  ```bash
  sudo tcpdump -i ens224 -nn
  ```
