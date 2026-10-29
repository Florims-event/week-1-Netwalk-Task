
 Week 1: Cybersecurity Internship Lab Portfolio

Hands-on work from Week 1 of my cybersecurity internship training with [NetworkWalks](https://networkwalks.com). I built a small virtual lab in VirtualBox, then used Wireshark to analyse attack and plain-text traffic like a Tier 1 SOC analyst.

Tools: VirtualBox, Kali Linux, Windows 10, Android-x86, Wireshark
Focus: virtual networking, IP addressing, connectivity troubleshooting, attack detection and traffic analysis


Tasks And Solution;

 1. Windows 10 VM and Kali Linux | Built a Windows 10 VM, connected it to Kali on a NAT Network with static IPs, and fixed a firewall problem blocking ping 
 2. Wireshark SOC labs | Completed 6 labs (60 tasks): port scan, plain-text chat, hidden data, HTTP credentials, FTP brute force, VoIP 
 3. Android-x86 in VirtualBox | Built an Android 9.0 VM on the same NAT Network and tested connectivity 


 Lab network overview

All three virtual machines share one VirtualBox NAT Network.

 Machine | IP address | Role 

 Kali Linux | 10.0.0.2 / 24 | Testing and analysis 
 Android 9.0 (Android9-Lab) | 10.0.0.9 / 24 | Test machine 
 Windows 10 | 10.0.0.10 / 24 | Test machine 
 Gateway | 10.0.0.1 | NAT Network router 
 DNS | 8.8.8.8 | Name resolution 


Task 1: Windows 10 VM and Kali Linux in VirtualBox

I set up a Windows 10 virtual machine and connected it to my existing Kali Linux machine to practise virtual networking and troubleshooting.

What I did
1. Downloaded the Windows 10 installation ISO and created a virtual machine in VirtualBox.
2. Installed and configured Windows 10.
3. Created and configured a NAT Network so the virtual machines could communicate.
4. Assigned static IPv4 addresses: Kali Linux 10.0.0.2/24 and Windows 10 10.0.0.10/24.
5. Troubleshot connectivity. The first pings failed because a Windows firewall rule was affecting incoming ICMP echo requests, so I changed it to allow them.
6. Verified communication from Kali Linux to Windows 10 with the `ping` command.

Result: Kali Linux can reach Windows 10 at 10.0.0.10, confirmed with `ping`.

What I learned: a ping can fail even when the network is set up correctly, because a firewall on the target can block it. Checking the addresses first and then the firewall is a methodical way to find the cause.



Task 2: Wireshark SOC labs

Six SOC labs from NetworkWalks, each with a packet capture and 10 tasks (60 tasks in total), worked through in Wireshark.

| Lab | Scenario | Key finding | 

 1 Identify the attacker's IP | TCP SYN port scan from 192.168.10.66 |
 2  Follow TCP Stream 1 | Plain-text office chat leaked a temporary password and a Wi-Fi password |
 3  Follow TCP Stream, Part 2 | Hidden HTML comment on /staff.html exposed an access code |
 4  HTTP plain-text credentials | Login sent over HTTP, so username and password were readable |
 5  Catch the brute-force attacker | FTP brute force: 24 failed logins, then one success |
 6  Catch the call scammer | SIP call setup traced to an unencrypted RTP voice stream |


Lab 1: Port scan detection

-Attacker: 192.168.10.66, target: 192.168.10.20
- The attacker sent one SYN to each of 40 different ports, always from source port 52311.
- Open ports answered with SYN-ACK: 22, 80, 443. The other 37 answered with RST.
- Attack type: TCP SYN scan (half-open scan)
- Filters used:
  - `tcp.flags.syn == 1 && tcp.flags.ack == 0`
  - `ip.src == 192.168.10.20 && tcp.flags.syn == 1 && tcp.flags.ack == 1`
- Also used: Statistics → Conversations → IPv4 / TCP

 Lab 2: Plain-text chat (Follow TCP Stream)

- An office chat tool ran on TCP port 5000 (stream 5, 413 bytes) between 192.168.10.45 (client) and 192.168.10.20 (server).
- The conversation between Ali (Accounts) and Sara (IT) exposed a temporary password and the guest Wi-Fi password.
-Method: Statistics → Conversations to find the stream, then Analyze → Follow → TCP Stream.

Lab 3: Hidden data in web traffic

- A web page, /staff.html, returned HTTP 200 and contained a hidden HTML comment . Browsers don't show comments, but they still travel in the packet.
- The comment exposed an access code and the IP of a backup server (192.168.10.77).
- A plain-text file, file_data contains "<!--"`

Lab 4: HTTP plain-text credentials

- A staff member signed in to portal.company.local with `POST /login` over plain HTTP.
- The form was sent twice from 192.168.10.45. The second attempt worked, and the username and password were readable in HTML Form URL Encoded.
- Filter used: `http.request.method == "POST"`

 Lab 5: FTP brute-force attack

- Statistics → Protocol Hierarchy showed FTP with an unusually large share of the traffic (117 packets, 31.7%).
- Attacker: 192.168.10.66, FTP server: 192.168.10.20
- The attacker tried 25 passwords for the username `admin`. 24 got `530 Login incorrect`, and one got `230`, which means a successful login.
- Attack type: brute force
- Filters used:
  - `ftp.response.code == 530`
  - `ftp.request.command == "PASS"

 Lab 6: VoIP call analysis

- *SIP* set up the call (INVITE, Ringing, OK). *RTP* carried the voice, using the *G.711 PCMU* codec.
- The call went from extension 101 (Ali Raza, 192.168.10.45) to extension 202 (Sara, 192.168.10.52).
- Because the call was not encrypted, the audio could be played back from the capture with Telephony → VoIP Calls.
- Filters used: `sip.Method == "INVITE"` and `rtp`



LESSONS 

- Anything sent without encryption can be read by anyone on the network: chat messages, passwords, hidden notes and voice calls.
- Port scans and brute-force attempts have clear patterns. A scan shows one source sending SYNs to many ports, and a brute force shows one source making many failed logins against one service.
- Protocol Hierarchy and Conversations are the fastest ways to find the unusual traffic in a large capture.

Recommendations

 Problem and Solutions 

 Plain-text web login | Use HTTPS (TLS), redirect HTTP to HTTPS, enable HSTS 
 Brute-force on FTP | Account lockout, strong passwords, multi-factor authentication, replace FTP with SFTP 
 Port scan | Firewall rules, close unused ports, IDS alerts on SYN scans 
 Plain-text chat and hidden notes | Use encrypted tools, and strip comments from pages before publishing 
 Unencrypted VoIP | SRTP for voice, SIP over TLS, and a separate VoIP VLAN 

Task 3: Android-x86 in VirtualBox

I built an Android 9.0 virtual machine in VirtualBox and put it on the same NAT Network as my Kali Linux VM, then tested that the two machines and the internet could all reach each other.

Network setup

Setting and Value

 Network type | VirtualBox NAT Network 
 Kali Linux | 10.0.0.2 / 24 
 Android 9.0 (Android9-Lab) | 10.0.0.9 / 24 
 Gateway | 10.0.0.1 
 DNS | 8.8.8.8 

What I did
1. Downloaded the Android-x86 9.0-r2 (64-bit) ISO from the official page.
2. Created a Linux (64-bit) VM named `Android9-Lab` with 2 GB RAM and a dynamically allocated VDI disk.
3. Installed Android to the virtual disk (ext4 partition, GRUB, read-write /system).
4. Changed the VM's network adapter to the same NAT Network as Kali.
5. Set a static IP on Android's virtual Wi-Fi (VirtWifi): IP 10.0.0.9, gateway 10.0.0.1, prefix length 24, DNS 8.8.8.8.
6. Tested connectivity with `ping`.

Results

 Test and Results

 Android to Kali (`ping 10.0.0.2`) | 13 sent, 13 received, 0% packet loss 
 Android to internet (`ping 8.8.8.8`) | 6 sent, 6 received, 0% packet loss 
 Kali to Android (`ping 10.0.0.9`) | 6 sent, 6 received, 0% packet loss 

Problems Solved
- The first boot ran Android in Live CD mode, which saves nothing to the disk. I reinstalled it with the *Installation* option so it boots from the virtual disk.
- Android stopped at a text prompt (`console:/ #`) because of the graphics settings. Changing the Graphics Controller to VBoxVGA, turning off 3D acceleration and booting with `nomodeset` fixed it.
- Android has no Ethernet entry in VirtualBox, so I used the *VirtWifi* network and set the static IP there.

What I learned: a NAT Network works like a small private router. VMs on the same one can talk to each other and reach the internet, and a static IP keeps each machine's address predictable for testing.. 


