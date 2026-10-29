## 1. CyberSecurity Lab Setup
This project documents the setup, configuration, testing, and troubleshooting of a cybersecurity testing lab using Oracle VirtualBox and Kali Linux.

The lab was created as part of the WK1-PM1 assignment at Networkwalks to establish a working environment for cybersecurity and ethical hacking exercises. Kali Linux serves as the attacking machine, while VirtualBox provides the virtualisation environment.

The lab uses a NAT Network with the IPv4 subnet `10.0.0.0/24`. Kali Linux is configured with the required IP address `10.0.0.2/24`, a shared folder connects the Windows host to the virtual machine, and network connectivity is tested using ICMP ping and DNS resolution.

## 2. Project Objectives

The objectives of this project is to:

1. Use Oracle VirtualBox as the virtualisation platform.
2. Set up Kali Linux as the attacking/testing machine.
3. Configure the network using the `10.0.0.0/24` subnet.
4. Create and use a NAT Network with the required subnet.
5. Enable bidirectional clipboard sharing and drag-and-drop.
6. Configure a shared folder between the Windows host and Kali Linux.
7. Assign Kali Linux the IP address `10.0.0.2/24`.
8. Ensure that Kali Linux has full internet access.

## 3. Lab Architecture

The lab consists of a Windows host computer running Oracle VirtualBox, with Kali Linux running as a guest virtual machine.

The virtual machine connects to a dedicated NAT Network, which provides a path to external networks through the host computer's internet connection.

**Network flow:**

`Internet <-> Windows Host <-> VirtualBox NAT Network <-> Kali Linux`

The Windows Downloads folder is also shared with Kali Linux to allow file access between the host and guest operating systems.

## 4. Lab Environment and Configuration

Component Configuration 
Host operating system: Windows 
Virtualisation platform: Oracle VirtualBox
Guest operating system: Kali Linux (64-bit)
Network attachment: NAT Network 
NAT Network name: Kali-NATNetwork` 
IPv4 subnet: 10.0.0.0/24
Required Kali IPv4 address: 10.0.0.2/24
Default gateway used: 10.0.0.1
Shared host folder: Windows Downloads 
Shared folder name: Downloads
Shared folder path in Kali: /media/sf_Downloads
Clipboard: Bidirectional 
Drag-and-drop: Bidirectional 

## 5. Implementation Procedure

### Step 1: Set Up Oracle VirtualBox and Kali Linux

Oracle VirtualBox was used as the virtualisation platform. Kali Linux was configured as a 64-bit virtual machine to provide a dedicated environment for cybersecurity testing.

The virtual machine was started, and the Kali Linux terminal was used to perform network configuration and verification.

**Verification:** Kali Linux started successfully and provided access to a Linux terminal.

### Step 2: Creating NAT Network

A NAT Network was configured in VirtualBox using the following settings:

- **Network name:** `Kali-NATNetwork`
- **IPv4 prefix:** `10.0.0.0/24`
- **DHCP:** Enabled

The network was configured through VirtualBox's Network settings. The Kali virtual machine's Adapter 1 was then configured with:

- **Attached to:** NAT Network
- **Name:** `Kali-NATNetwork`
- **Virtual Cable Connected:** Enabled

This configuration connects Kali Linux to the designated NAT Network.

**Verification:** VirtualBox displayed the correct NAT Network name and IPv4 prefix, and the virtual machine was attached to that network.

### Step 3: Configuring the Kali Linux IP Address

Theis project required Kali Linux to use the IPv4 address `10.0.0.2/24`.

The interface was inspected using:

```bash
ip addr show eth0
```

The required address was confirmed in the output:

```text
inet 10.0.0.2/24 scope global eth0
```

The `/24` prefix corresponds to the subnet mask `255.255.255.0`, giving the network address `10.0.0.0` and broadcast address `10.0.0.255`.

During configuration, an additional address (`10.0.0.3/24`) was initially present. The extra address was removed, and the final verification showed `10.0.0.2/24` on `eth0`.

**Verification command:**

```bash
ip addr show eth0
```

**Expected result:** The interface `eth0` displays `10.0.0.2/24`.


### Step 4: Enable Clipboard Sharing and Drag-and-Drop

The virtual machine's General > Features settings were configured as follows:

Shared Clipboard: Bidirectional
Drag-and-Drop: Bidirectional

Bidirectional clipboard sharing allows copying and pasting between Windows and Kali Linux. Bidirectional drag-and-drop is intended to allow file transfers in either direction, subject to VirtualBox Guest Additions support and compatibility.

**Verification:** Both options were set to Bidirectional in the virtual machine settings.

### Step 5: Configuring the Shared Folder

The Windows Downloads folder was added as a shared folder in VirtualBox.

The configuration was:

Host folder: Windows Downloads
Shared folder name: `Downloads` 
Auto-mount: Enabled
Make Machine-permanent: Enabled
Read-only: Disabled 

The folder was configured to be automatically mounted and remain available in the virtual machine configuration.

After saving the settings, the Kali Linux terminal was used to verify the mount.

First, the available directories were listed:

```bash
ls /media
```

The output included:

```text
sf_Downloads
```

The contents were then inspected:

```bash
ls /media/sf_Downloads
```

The command displayed files from the Windows Downloads folder, confirming that Kali Linux could access the shared directory.



### Step 6: Diagnosing and Configuring the Default Route

During the internet connectivity test, Kali Linux could not initially reach an external IP address.

The routing table was inspected using:

```bash
ip route
```

The initial output contained only the directly connected subnet route:

```text
10.0.0.0/24 dev eth0 proto kernel scope link src 10.0.0.2
```

There was no default route. Without a default route, packets destined for external networks had no configured next hop.

The default route was added using:

```bash
sudo ip route add default via 10.0.0.1 dev eth0
```

Here:

- `sudo` runs the command with administrative privileges.
- `ip route add` adds a routing-table entry.
- `default` identifies the route used when no more specific route matches.
- `via 10.0.0.1` specifies the next-hop gateway.
- `dev eth0` identifies the network interface.

The routing table can be checked again using:

```bash
ip route
```

**Expected result:** A default route through `10.0.0.1` on `eth0`, alongside the directly connected `10.0.0.0/24` route.


### Step 7: Testing IP Connectivity

After adding the default route, connectivity was tested using Google's public DNS IP address:

```bash
ping -c 4 8.8.8.8
```

The test returned four replies, with zero packet loss.

This demonstrated that Kali Linux could reach an external IP address through the configured network path.

**Observed result:**

- Packets transmitted: 4
- Packets received: 4
- Packet loss: 0%

### Step 8: Troubleshooting DNS Resolution

Although IP connectivity was working, the following command initially failed:

```bash
ping -c 4 google.com
```

The error was:

```text
Temporary failure in name resolution
```

This indicated a DNS resolution problem rather than a complete loss of IP connectivity. The resolver configuration was inspected using:

```bash
cat /etc/resolv.conf
```

The file contained:

```text
# Generated by NetworkManager
```

No nameserver was listed. To test the DNS diagnosis, the following temporary resolver configuration was applied:

```bash
sudo sh -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
```

The domain connectivity test was then repeated:

```bash
ping -c 4 google.com
```

The test succeeded. The hostname resolved to an IP address, four replies were received, and packet loss was zero.

**Observed result:**

- Hostname resolution: Successful
- Packets transmitted: 4
- Packets received: 4
- Packet loss: 0%

This confirmed that the immediate DNS resolution issue had been resolved.


## 6. Final Verification Checklist


## 7. Challenges Encountered and Solutions

### Challenge 1: Missing Default Route

**Problem:** Kali Linux could not reach `8.8.8.8` and reported that the network was unreachable.

**Cause:** The routing table did not contain a default route.

**Solution:** Added a default route through `10.0.0.1` using `ip route`.

### Challenge 2: DNS Resolution Failure

**Problem:** The IP connectivity test succeeded, but `google.com` could not be resolved.

**Cause:** The resolver configuration did not contain a nameserver.

**Solution:** Added `8.8.8.8` as a nameserver for testing. The subsequent hostname ping succeeded.

**Lesson learned:** IP connectivity and DNS resolution are separate functions. A machine may reach an IP address but still fail to access services by hostname if DNS is not configured correctly.

## 8. Skills Demonstrated

This assignment provided practical experience in:

- Virtualisation using Oracle VirtualBox
- Kali Linux administration
- IPv4 addressing and CIDR notation
- NAT Network configuration
- Routing tables and default gateways
- DNS resolution and troubleshooting
- Shared-folder configuration
- Network connectivity testing using `ping`
- Linux terminal commands and administrative privileges
- Technical documentation and evidence collection


## 10. Conclusion

The WK1-PM1 cybersecurity lab was configured using Oracle VirtualBox and Kali Linux. The required NAT Network, IP address, bidirectional clipboard and drag-and-drop settings, and host shared folder were configured and verified.

Initial connectivity problems were investigated using Linux networking commands. A missing default route was added, and a DNS resolution issue was addressed by configuring a nameserver for testing. Subsequent IP and hostname ping tests succeeded with zero packet loss.

The project established a foundation for further hands-on cybersecurity and ethical hacking practice. Persistent network configuration and reboot verification remain follow-up tasks.

## 11. Responsible Use

This lab is intended for educational purposes and authorised security testing only. Any future scanning or security testing will be limited to systems owned by me or systems for which I have explicit permission to test.

## All evidences (screenshots have been attached at the files session)
