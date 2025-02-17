## Overview
This lab demonstrated how to analyze a network topology using GNS3, inspect routing tables, test connectivity, and analyze ARP behavior using Wireshark.

#### 1. Launch GNS3:

  * Open a terminal and enter gns3 & to launch GNS3 in the background.
  * Ignore any update check prompts.
  
#### 2. Network Topology:

  * Four hosts: Linux-1, Linux-2, Linux-3, Linux-4
  * Two switches, one router

![Image](https://github.com/user-attachments/assets/f02c2524-da4f-4e11-b2a9-6a966391a8b4)


## 1. **Identify the Networks Hosts**

In this part of the lab, we will open a terminal on each host in the network and examine its network configuration information, routing table, and network configuration file.

  * In the GNS3 topology window, right-click **Linux-1 > Console** from the context menu
  * In the terminal window, on the Linux-1 tab, type **ifconfig** and press Enter
  
![Image](https://github.com/user-attachments/assets/01e6e98c-7547-4b94-99cf-11fb0bfd1d07)

  * **eth0** (the primary network interface for this Linux system).
  * **lo** (a loopback interface, which is a virtual interface used by the operating system to communicate with itself).

Take note of the MAC address (HWaddr), IP Address (inet addr), and the Subnet Mask (Mask) for the eth0 interface.

Because the subnet mask is 255.255.255.0, we know that the first three octets are the network address, and the last octet is the host address. Thus, this is the 192.168.0.10 host on the 192.168.0.0/24 network.

**OBS:** In this virtual environment, MAC addresses change each time GNS3 restarts, unlike in the real world where they remain constant—though some devices, like smartphones, can spoof them for privacy.

IP Address and subnet mask on Linux-1, Linux-2, Linux-3 and Linux-4.

| Host   | IP Address       | Subnet Mask        |
|--------|-----------------|--------------------|
| Linux1 | 192.168.0.10/24 | 255.255.255.0     |
| Linux2 | 192.168.0.20/24 | 255.255.255.0     |
| Linux3 | 192.168.1.10/24 | 255.255.255.0     |
| Linux4 | 192.168.1.20/24 | 255.255.255.0     |

In the terminal window, on the Linux-1 tab, type **ip a** and press Enter to display the network configuration on the Linux-1 host.

![Image](https://github.com/user-attachments/assets/b4c66542-95fa-4318-add0-cc8189edb007)

The **ip a** command has replaced the **ifconfig** command on many modern Linux systems. Notice that you see similar information to the output of the ifconfig command, but that **ip a uses slash notation to express the subnet mask**.

In the terminal window, on the Linux-1 tab, type **netstat -r** and press Enter to display the kernel IP routing table.

![Image](https://github.com/user-attachments/assets/75904ee0-f25c-48d3-b989-de1ffbd5067b)

You should see that the default gateway is 192.168.0.1. Any IP traffic not destined for the local network is sent to the default gateway (router). The router then forwards the packet to the correct network or onward to another router.

In the terminal window, on the Linux-1 tab, type **cat /etc/network/interfaces** and press Enter to display the full Linux network configuration file.

The section that matters is under the comment **# Static config for eth0**. This is where the network configuration is defined.

![Image](https://github.com/user-attachments/assets/8f186271-e49b-4aad-b286-1bde53380545)

**OBS:** If the gateway is misconfigured in /etc/network/interfaces it will not appear as a default gateway using netstat -r.

I found the hidden FLAG in the /etc/network/interfaces file on Linux-2.

![Image](https://github.com/user-attachments/assets/129aa7b2-3101-4910-940d-5a1384979683)


## 2. **Test Connectivity**

The first step in troubleshooting a network issue is to ensure the default gateway is correct and to test connectivity to that gateway.

In the terminal window, on the Linux-1 tab, type **ping -c 4 192.168.0.1** and press Enter to test connectivity to the default gateway.

![Image](https://github.com/user-attachments/assets/4957dff0-ce88-4a78-9692-11886f8450ea)

  If a host has the correct IP address, subnet mask, and gateway, it should be able to reach any other host the router can. Our router sits between two networks: 192.168.0.0/24 and 192.168.1.0/24. Thus, any correctly configured host on 192.168.0.0/24 should be able to reach hosts in 192.168.1.0/24 and vice-versa.

In the GNS3 topology window, right-click the **Router > Stop** from the context menu.

You will see the connection indicators on the Router turn red. Without a gateway (router), network 192.168.0.0/24 should not be able to reach network 192.168.1.0/24.

![Image](https://github.com/user-attachments/assets/ababf836-ac8d-4faa-9029-c1937f962f5f)

In the terminal window, on the Linux-1 tab, type ping -c 4 192.168.1.10 and press Enter to test connectivity to the 192.168.1.0/24 network

  * You should find that you can no longer reach the 192.168.1.0/24 network.


![Image](https://github.com/user-attachments/assets/5e6dc894-e746-427f-a599-1e9e500481cd)

In the GNS3 topology window, right-click the **Router > Start**.

![Image](https://github.com/user-attachments/assets/69bd43d9-6401-4ad7-aaa4-03e3651f9ce7)

Connectivity should now be restored.


## **Part 3: Inspecting ARP Caches**

In this part of the lab, we will shift our focus to local subnet traffic and ARP. Recall that ARP is used to map local IP Addresses to MAC addresses.

### Step 1: View ARP Cache

In the terminal window, on the Linux-1 tab, type **arp -a** to display all cached ARP entries.

  * We have not made any networking requests yet, so there will be nothing in the ARP cache.

On the Linux-1 tab, type **ping -c 4 192.168.0.20** and press Enter to test connectivity to Linux-2 on the local subnet.

  * Linux-1 sends an ARP request to all hosts on **192.168.0.0/24**.
  * Linux-2 responds with its MAC address.
  * Linux-1 caches this mapping for future communication.

![Image](https://github.com/user-attachments/assets/51a3f8a0-a71f-46ef-a7f2-7182c377c635)

In Wireshark, you should see that Linux-1 sends a Broadcast message. All hosts and devices on the local subnet can see a Broadcast message. Next, Linux-2 replies to Linux-1 with its MAC address. Notice that Linux-2 then makes its own ARP request for Linux-1's MAC address. This request is made because Linux-2 assumes more traffic to and from Linux-1 will be forthcoming.


### Step 2: Capture ARP in Wireshark
![Image](https://github.com/user-attachments/assets/6cf150d1-5e76-4b02-a187-3f568cea0364)

  * Linux-1 sends a **Broadcast ARP request**.
  * Linux-2 responds with its MAC address.
  * Linux-2 then requests Linux-1’s MAC address for future communication.

**OBS**: Remember - because this is a virtual environment, the MAC addresses will change every time GNS3 restarts the network.

In the terminal window, on the Linux-1 tab, type ping -c 4 192.168.1.10 and press Enter to ping the Linux-3 device.

In the Wireshark window, notice that Linux-1 again sends a Broadcast message looking for the MAC address of the default gateway. The Router responds with its MAC address and then makes its own request for Linux-1's MAC address.

![Image](https://github.com/user-attachments/assets/cb401b12-1c60-472a-ba74-6dc5ac22e3ca)

  * Linux-1 sends an ARP request for the **default gateway’s MAC address**.
  * The router responds and then queries for Linux-1’s MAC address.
