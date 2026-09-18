1. Install Snort and Configure Packet Capture + IDS Mode

Manual Experiment: Exp. No. 1
Aim: Install Snort on Linux and configure it for packet capture and intrusion detection.

The manual describes Snort as an open-source NIDS/NIPS that supports sniffer, packet logger, and network intrusion detection modes.

Step 1 — Update Ubuntu

Open Terminal:

sudo apt update

Install dependencies:

sudo apt install -y build-essential libpcap-dev libpcre3-dev libdumbnet-dev bison flex zlib1g-dev

These are the dependencies specified in the manual.

Step 2 — Install Snort

The manual allows installing a Snort 2.9.x/3.x package or compiling from source.

After installation, verify:

snort -V

You should see the installed Snort version.

Step 3 — Create Snort directories

Create:

sudo mkdir -p /etc/snort
sudo mkdir -p /var/log/snort
sudo mkdir -p /usr/local/lib/snort_dynamicrules
Step 4 — Configure snort.conf

Open:

sudo nano /etc/snort/snort.conf

Set:

HOME_NET

to your protected network.

Also configure:

RULE_PATH
SO_RULE_PATH

according to your Snort installation.

The manual identifies HOME_NET, RULE_PATH, and SO_RULE_PATH as important configuration variables.

Step 5 — Test configuration

Before running Snort:

sudo snort -T -c /etc/snort/snort.conf

If the configuration is correct, Snort should report that the configuration test succeeded.

Step 6 — Find network interface
ip a

Example:

eth0
enp0s3
ens33
wlan0

Use the interface connected to your lab network.

Step 7 — Run Snort in IDS mode

For example:

sudo snort -A console -q -c /etc/snort/snort.conf -i eth0

Replace eth0 with your actual interface.

Meaning
Option	Meaning
-A console	Display alerts on terminal
-q	Quiet mode
-c	Configuration file
-i	Network interface

The manual gives this exact IDS-mode approach.

Expected result

Snort starts monitoring live traffic and displays alerts whenever traffic matches an enabled rule.

Logs are stored under:

/var/log/snort
2. Configure Real-Time Snort Alerts + Email Notification

Manual Experiment: Exp. No. 6

The manual uses:

Snort → syslog → rsyslog → Python mailer → Gmail SMTP → administrator

The purpose is to generate a real-time alert and send it through email.

Step 1 — Configure Snort syslog output

Open:

sudo nano /etc/snort/snort.conf

Add:

output alert_syslog: LOG_AUTH LOG_ALERT

Save:

Ctrl + O
Enter
Ctrl + X

This causes Snort alerts to be forwarded to syslog.

Step 2 — Configure rsyslog

Create:

sudo nano /etc/rsyslog.d/snort-email.conf

Add:

if ($programname == 'snort') then {
    action(type="omprog" binary="/usr/local/bin/snort-mailer.py")
}

Restart rsyslog:

sudo systemctl restart rsyslog

The manual uses rsyslog as the intermediary that invokes the Python mailer.

Step 3 — Create Gmail App Password

The manual specifies using a Gmail App Password, rather than your normal Gmail password.

Go to:

Google Account Security

Then:

Security
→ 2-Step Verification
→ App Passwords
→ Mail
→ Other/Custom name
→ Snort Mailer
→ Generate

Keep the generated 16-character password private. The manual specifically instructs using an App Password for this experiment.

Step 4 — Create Python mailer
sudo nano /usr/local/bin/snort-mailer.py

Use:

#!/usr/bin/env python3

import sys
import smtplib
from email.message import EmailMessage

SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587

SMTP_USER = "yourgmail@gmail.com"
SMTP_PASS = "your_app_password_here"

TO_EMAIL = "admin@gmail.com"

alert_body = sys.stdin.read()

if alert_body.strip():
    msg = EmailMessage()
    msg.set_content(alert_body)

    msg["Subject"] = "Snort Alert on localhost"
    msg["From"] = SMTP_USER
    msg["To"] = TO_EMAIL

    try:
        with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
            server.starttls()
            server.login(SMTP_USER, SMTP_PASS)
            server.send_message(msg)

    except Exception as e:
        print(f"Failed to send email: {e}")

Make executable:

sudo chmod +x /usr/local/bin/snort-mailer.py

This follows the mailer design in the manual.

Step 5 — Run Snort

For the manual's loopback test:

sudo snort -A console -q -c /etc/snort/snort.conf -i lo

In another terminal:

ping 127.0.0.1

The manual expects Snort to detect the loopback traffic and forward the alert through syslog/rsyslog to the Python mailer.

Step 6 — Verify email

Check your Gmail inbox.

Expected subject:

Snort Alert on localhost

Flow:

Ping
  ↓
Snort
  ↓
Snort Alert
  ↓
Syslog
  ↓
rsyslog
  ↓
snort-mailer.py
  ↓
Gmail SMTP
  ↓
Administrator Email

The manual specifies this expected notification mechanism.

3. Detect Malicious HTTP GET Request to Sensitive Path

Manual Experiment: Exp. No. 3

Aim: Detect an HTTP GET request attempting to access a sensitive path such as /admin.

Step 1 — Open local Snort rules
sudo nano /etc/snort/rules/local.rules
Step 2 — Add custom rule

Use the manual's rule:

alert tcp any any -> any 80
(
    msg:"Possible Sensitive Path Access";
    flow:to_server,established;
    content:"GET";
    http_method;
    content:"/admin";
    http_uri;
    nocase;
    sid:1000001;
    rev:1;
)

In one line:

alert tcp any any -> any 80 (msg:"Possible Sensitive Path Access"; flow:to_server,established; content:"GET"; http_method; content:"/admin"; http_uri; nocase; sid:1000001; rev:1;)

The important parts are:

GET       → HTTP method
/admin    → sensitive URI
http_uri  → inspect URI
nocase    → case insensitive
sid       → rule identifier

Step 3 — Test configuration
sudo snort -T -c /etc/snort/snort.conf
Step 4 — Start Snort
sudo snort -A console -q -c /etc/snort/snort.conf -i eth0

Replace eth0 with your interface.

Step 5 — Generate test HTTP request

In another terminal:

curl -v http://localhost/secret/admin/config

The request contains:

/admin

inside the URI.

Expected Snort alert

You should see something similar to:

[**] Possible Sensitive Path Access [**]

The exact formatting depends on your Snort version/configuration.

Detection flow
curl
 ↓
HTTP GET /secret/admin/config
 ↓
Snort
 ↓
http_method = GET
 ↓
http_uri contains /admin
 ↓
Rule SID 1000001 matches
 ↓
Alert generated
4. TCP SYN Flood + Wireshark

Manual Experiment: Exp. No. 7

The manual demonstrates a Kali attacker VM, an Ubuntu target VM, and Wireshark monitoring.

Example topology
Kali
Attacker
10.0.2.10
     |
     | TCP SYN packets
     ↓
Ubuntu
Target
10.0.2.20
Step 1 — Prepare target

On Ubuntu target:

sudo apt update

Install Apache:

sudo apt install -y apache2

Start:

sudo systemctl start apache2

Check IP:

ip addr show

The Apache service gives you a listening TCP service on port 80.

Step 2 — Start Wireshark

Open:

wireshark

Select the VM's lab-network interface.

Optional capture filter:

tcp and host 10.0.2.20

Then click:

Start
Step 3 — Generate SYN flood in isolated lab

The manual provides:

sudo hping3 -c 15000 -d 120 -S -w 64 -p 80 --flood --rand-source <TARGET-IP>

Use the actual target IP in your isolated lab.

The manual explains:

Option	Meaning
-c 15000	Send 15,000 packets
-d 120	120-byte data size
-S	SYN flag
-w 64	TCP window
-p 80	Destination port 80
--flood	Send rapidly
--rand-source	Randomize source IP

Step 4 — Wireshark SYN filter

Use:

tcp.flags.syn == 1 && tcp.flags.ack == 0

This displays SYN packets without ACK.

To restrict to target:

ip.dst == <TARGET-IP> && tcp.flags.syn == 1 && tcp.flags.ack == 0

To see SYN/ACK replies:

ip.src == <TARGET-IP> && tcp.flags.syn == 1 && tcp.flags.ack == 1

These are the filters given in the manual.

Step 5 — Analyze statistics

Wireshark:

Statistics
 ├── Endpoints → IPv4
 ├── Conversations → TCP
 └── IO Graph

For IO Graph, use:

tcp.flags.syn==1 && ip.dst==<TARGET-IP>

Also:

Analyze → Expert Info

Look for high packet rates, retransmissions and other anomalies.

Step 6 — Stop attack

Press:

Ctrl + C
Step 7 — Save capture

Wireshark:

File
→ Save As
→ syn_flood_<TARGET-IP>.pcapng

The manual uses the same approach.

Expected observation
Many rapid SYN packets
        ↓
Target receives SYNs
        ↓
Target sends SYN/ACK
        ↓
Few/no final ACKs
        ↓
Many half-open connections
5. Sniff Packets on Loopback + Snort IDS

Manual Experiment: Exp. No. 8

The manual specifically uses one Linux machine with:

127.0.0.2 = attacker
127.0.0.3 = target
lo         = loopback interface

This avoids requiring two physical machines.

Step 1 — Create loopback aliases
sudo ip addr add 127.0.0.2/8 dev lo
sudo ip addr add 127.0.0.3/8 dev lo

Verify:

ip -4 addr show dev lo

You should see:

127.0.0.1
127.0.0.2
127.0.0.3

Step 2 — Test aliases
ping -c 2 127.0.0.2
ping -c 2 127.0.0.3
Step 3 — Start Snort

Terminal 1:

sudo snort -A console -q -c /etc/snort/snort.conf -i lo

Snort now monitors loopback traffic.

Step 4 — Start packet capture

Terminal 2:

sudo tcpdump -i lo -nn -w loopback_capture.pcap

-nn prevents hostname/service-name resolution.

Step 5 — Generate ICMP traffic

Terminal 3:

ping -I 127.0.0.2 -c 4 127.0.0.3

Traffic:

127.0.0.2
    ↓ ICMP
    ↓
lo
    ↓
127.0.0.3

The manual also gives HTTP and Nmap examples.

HTTP test

Start server:

python3 -m http.server 8080 --bind 127.0.0.3 &

Then:

curl --interface 127.0.0.2 http://127.0.0.3:8080/
Nmap test
sudo nmap -e lo -sS -p 22,80,8080 127.0.0.3

Step 6 — Stop capture

Press:

Ctrl + C

You now have:

loopback_capture.pcap
Step 7 — Analyze
tcpdump -r loopback_capture.pcap -nn -tttt

Filter:

ip src 127.0.0.2 and ip dst 127.0.0.3

Or open in Wireshark:

wireshark loopback_capture.pcap &

The manual instructs correlating the packet timestamp/source/destination with the Snort alert.

6. IP Spoofing with Scapy + Loopback + PCAP

Manual Experiment: Exp. No. 9

The manual intentionally performs this on loopback so that the spoofing experiment remains local to the host.

Topology:

Spoofed source
1.2.3.4
     ↓
     ICMP
     ↓
127.0.0.2
loopback target
Step 1 — Install required packages
sudo apt update
sudo apt install -y python3 python3-scapy tcpdump wireshark

Verify Scapy:

python3 -c "from scapy.all import *; print('scapy ok,', conf.version)"

The manual recommends the distro python3-scapy package.

Step 2 — Create loopback target
sudo ip addr add 127.0.0.2/8 dev lo || true
Step 3 — Start tcpdump
sudo tcpdump -i lo icmp -w /tmp/icmp_spoof_loop.pcap

Or automatically stop after five packets:

sudo tcpdump -i lo -c 5 icmp -w /tmp/icmp_spoof_loop.pcap

Leave this terminal running.

Step 4 — Create Scapy script
nano ~/send_spoof_loop.py

Use:

#!/usr/bin/env python3

from scapy.all import *

conf.verb = 0

TARGET = "127.0.0.2"
SPOOF_SRC = "1.2.3.4"
IFACE = "lo"

pkt = IP(
    src=SPOOF_SRC,
    dst=TARGET
) / ICMP(type=8) / b"loopback-spoof"

send(
    pkt,
    iface=IFACE,
    count=5,
    inter=0.5
)

print(
    "Sent 5 spoofed ICMP packets from",
    SPOOF_SRC,
    "to",
    TARGET
)

The manual uses the same source/target concept and sends five packets.

Step 5 — Run
sudo python3 ~/send_spoof_loop.py

Expected:

Sent 5 spoofed ICMP packets from 1.2.3.4 to 127.0.0.2
Step 6 — Inspect PCAP
ls -lh /tmp/icmp_spoof_loop.pcap

Read using tcpdump:

sudo tcpdump -nn -r /tmp/icmp_spoof_loop.pcap

Verbose:

sudo tcpdump -nn -v -r /tmp/icmp_spoof_loop.pcap

Or:

wireshark /tmp/icmp_spoof_loop.pcap &

Expected packet:

Source:      1.2.3.4
Destination: 127.0.0.2
Protocol:    ICMP
Type:        Echo Request

The manual's expected result is five ICMP echo-request packets showing the forged source.

7. Configure Linux Firewall Using iptables

Manual Experiment: Exp. No. 10

The manual explains the hierarchy as:

iptables
   ↓
tables
   ↓
chains
   ↓
rules

It focuses on the filter table.

Step 1 — Check installation
iptables -V

If required:

sudo apt-get install iptables

Help:

iptables -h
Step 2 — View current rules
sudo iptables -L

Verbose:

sudo iptables -L -v

With numbers:

sudo iptables -L --line-numbers
Step 3 — Understand the three chains
INPUT

Controls packets coming into the local machine.

FORWARD

Controls packets being routed through the machine.

OUTPUT

Controls packets originating from the local machine.

The manual identifies these three filter-table chains.

Drop all traffic demonstration

The manual demonstrates changing the default behavior to DROP.

For a controlled lab VM:

sudo iptables -P INPUT DROP

Similarly:

sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP

Be careful: doing this on a remotely accessed machine can disconnect you.

Allow ping

The manual demonstrates adding rules to permit required traffic.

General format:

sudo iptables -A INPUT -p icmp -j ACCEPT

And if output needs to be permitted:

sudo iptables -A OUTPUT -p icmp -j ACCEPT

Then check:

sudo iptables -L -v

The manual emphasizes that many protocols require two-way communication.

Allow SSH from a particular IP

The manual uses:

10.0.2.7

Add:

sudo iptables -A INPUT -p tcp --dport 22 -s 10.0.2.7 -m state --state NEW,ESTABLISHED -j ACCEPT

Then:

sudo iptables -A OUTPUT -p tcp --sport 22 -d 10.0.2.7 -m state --state ESTABLISHED -j ACCEPT

This permits SSH traffic associated with that source while using connection state tracking.

Delete a specific rule

First:

sudo iptables -L --line-numbers

Example:

num  target
1    ACCEPT
2    DROP
3    ACCEPT

Delete rule 2:

sudo iptables -D INPUT 2
Flush all rules
sudo iptables -F

This removes the current rules.

Save rules

The manual gives:

sudo /sbin/iptables-save > /etc/iptables/rules.v4

Persistence mechanisms can vary by Linux distribution.

8. Configure VPN Using Cisco Packet Tracer

Manual Experiment: Exp. No. 11

The manual demonstrates a site-to-site VPN-style topology using:

PC0 ─ Router0 ─ Router2 ─ Router1 ─ PC1

with Router2 acting as the intermediate/public network. The manual discusses tunneling, IPsec, IKE and ACL concepts.

Step 1 — Open Cisco Packet Tracer

Add:

PC0
PC1
Router0
Router1
Router2

The manual specifies three 1841 routers.

Step 2 — Connect devices

Connect:

PC0 → Router0
PC1 → Router1
Router0 → Router2
Router1 → Router2

Use the FastEthernet interfaces specified by the manual.

IP Address Table

Use this as your reference while configuring.

Device	Interface	IP
PC0	FastEthernet	192.168.1.2
Router0	Fa0/0	192.168.1.1
Router0	Fa0/1	1.0.0.2
Router2	Fa0/0	1.0.0.1
Router2	Fa0/1	2.0.0.1
Router1	Fa0/1	2.0.0.2
Router1	Fa0/0	192.168.2.1
PC1	FastEthernet	192.168.2.2

These are the addresses specified throughout the VPN procedure.

The manual contains a couple of typographical inconsistencies in gateway text; use the intended subnet gateway values shown by the router interfaces, e.g. PC0 → 192.168.1.1 and PC1 → 192.168.2.1.

Step 3 — Configure PC0

PC0:

Desktop
→ IP Configuration

Set:

IP Address:       192.168.1.2
Default Gateway:  192.168.1.1
Step 4 — Configure Router0

Router0:

Config
→ FastEthernet0/0

Set:

IP: 192.168.1.1

Turn interface On.

For Fa0/1:

IP: 1.0.0.2

Turn On.

The manual assigns these addresses to Router0.

Step 5 — Configure Router2

Fa0/0:

1.0.0.1

Fa0/1:

2.0.0.1

Turn both interfaces On.

Step 6 — Configure Router1

Fa0/0:

192.168.2.1

Fa0/1:

2.0.0.2

Turn both On.

Step 7 — Configure PC1

PC1:

Desktop
→ IP Configuration

Set:

IP Address:       192.168.2.2
Default Gateway:  192.168.2.1
Step 8 — Configure static routes

The manual uses the Config → Static routing section on Router0 and Router1.

The purpose is to tell each branch router how to reach the remote LAN through Router2.

Conceptually:

192.168.1.0/24
      |
   Router0
      |
   1.0.0.x
      |
   Router2
      |
   2.0.0.x
      |
   Router1
      |
192.168.2.0/24

The manual's steps 19–20 configure these static routes.

Step 9 — Create tunnel on Router0

Go to:

Router0
→ CLI

Enter:

exit
ping 2.0.0.2
config t
interface tunnel 1
ip address 172.16.1.1 255.255.0.0
tunnel source FastEthernet0/1
tunnel destination 2.0.0.2
no shut

The manual uses exactly this tunnel configuration.

Step 10 — Create tunnel on Router1

Router1 CLI:

interface tunnel 2
ip address 172.16.1.2 255.255.0.0
tunnel source FastEthernet0/1
tunnel destination 1.0.0.2
no shut

Step 11 — Configure required static routes

The manual then adds the required static routing entries through the Config tab on the routers.

Make sure:

PC0 → Remote LAN

and

PC1 → Remote LAN

have a valid route.

Step 12 — Test from PC0

PC0:

Desktop
→ Command Prompt

Run:

ping 192.168.1.1

Then:

tracert 192.168.2.2
Step 13 — Test from PC1

PC1:

ping 192.168.2.1

Then:

tracert 192.168.1.2

These are the final connectivity tests specified in the manual.

Basic VPN concept
Branch A
192.168.1.0/24
      |
    R0
      |
      | Tunnel
      |
    R1
      |
192.168.2.0/24
Branch B

The manual explains that VPNs provide secure communication using tunneling/encryption and discusses IPsec, IKE and ACLs.

9. Snort NIDS + Custom Rules for All Scans

Manual Experiment: Exp. No. 2

This is the most important Snort experiment for your list because it combines:

SYN scan/flood
UDP scan
Ping scan
FIN scan
NULL scan
XMAS scan
Generic TCP scan

The manual explicitly defines this experiment and provides the custom rules and Nmap tests.

Step 1 — Verify Snort
snort -V
Step 2 — Find interface
ip a

Example:

eth0
enp0s3
Step 3 — Open local rules
sudo nano /etc/snort/rules/local.rules
Step 4 — Add all seven rules

Paste:

# SYN scan / possible SYN flood
alert tcp any any -> $HOME_NET any (msg:"[IDS] Possible TCP SYN scan/flood"; flags:S; detection_filter: track by_src, count 20, seconds 5; sid:1000001; rev:1;)

# UDP scan
alert udp any any -> $HOME_NET any (msg:"[IDS] Possible UDP scan"; detection_filter: track by_src, count 40, seconds 60; sid:1000002; rev:1;)

# ICMP Ping sweep
alert icmp any any -> $HOME_NET any (msg:"[IDS] ICMP echo request sweep"; itype:8; detection_filter: track by_src, count 10, seconds 60; sid:1000003; rev:1;)

# FIN scan
alert tcp any any -> $HOME_NET any (msg:"[IDS] Possible FIN scan"; flags:F; detection_filter: track by_src, count 5, seconds 60; sid:1000004; rev:1;)

# NULL scan
alert tcp any any -> $HOME_NET any (msg:"[IDS] Possible NULL scan"; flags:0; detection_filter: track by_src, count 5, seconds 60; sid:1000005; rev:1;)

# XMAS scan
alert tcp any any -> $HOME_NET any (msg:"[IDS] Possible XMAS scan"; flags:FPU; detection_filter: track by_src, count 5, seconds 60; sid:1000006; rev:1;)

# Generic TCP scan
alert tcp any any -> $HOME_NET any (msg:"[IDS] Generic TCP portscan (SYN probes)"; flags:S; detection_filter: track by_src, count 15, seconds 60; sid:1000007; rev:1;)

These are the rules supplied by your manual.

Understanding the Seven Rules
SID	Detection	Important condition
1000001	SYN scan/flood	TCP SYN
1000002	UDP scan	Many UDP packets
1000003	Ping sweep	ICMP type 8
1000004	FIN scan	FIN flag
1000005	NULL scan	No TCP flags
1000006	XMAS scan	FIN + PSH + URG
1000007	Generic TCP scan	Many SYN probes
Step 5 — Save
Ctrl + O
Enter
Ctrl + X

The manual uses this exact save procedure.

Step 6 — Test Snort configuration
sudo snort -T -c /etc/snort/snort.conf

If there are rule/configuration errors, fix them before continuing.

Step 7 — Start Snort

Terminal 1:

sudo snort -A console -q -c /etc/snort/snort.conf -i eth0

Replace eth0.

Step 8 — Install Nmap

Terminal 2:

sudo apt install nmap
Step 9 — Ping scan test

The manual uses:

ping -c 12 127.0.0.1

This should produce enough ICMP echo requests to trigger the ICMP rule.

Expected:

[IDS] ICMP echo request sweep
Step 10 — SYN scan
sudo nmap -sS -p 1-1000 --min-rate 200 --max-retries 0 127.0.0.1

Expected:

[IDS] Possible TCP SYN scan/flood
Step 11 — TCP Connect scan
sudo nmap -sT -p 1-1000 --min-rate 100 127.0.0.1

Step 12 — FIN scan
sudo nmap -sF -p 1-200 --min-rate 100 127.0.0.1

Expected:

[IDS] Possible FIN scan

Step 13 — NULL scan
sudo nmap -sN -p 1-200 --min-rate 100 127.0.0.1

Expected:

[IDS] Possible NULL scan

Step 14 — XMAS scan
sudo nmap -sX -p 1-200 --min-rate 100 127.0.0.1

Expected:

[IDS] Possible XMAS scan

Step 15 — Generic TCP port scan
sudo nmap -sS -p- --min-rate 200 --max-retries 0 127.0.0.1

Expected:

[IDS] Generic TCP portscan (SYN probes)

Complete Lab Flow for Experiment 9

You can remember the entire experiment like this:

                 ┌──────────────────────┐
                 │      Snort NIDS       │
                 │  snort.conf           │
                 │  local.rules          │
                 └──────────┬───────────┘
                            │
                            ↓
                     Monitor interface
                            │
            ┌───────────────┼───────────────┐
            ↓               ↓               ↓
          Ping            Nmap           Hping3
            │               │               │
            ↓               ↓               ↓
         ICMP          TCP/UDP scans      SYNs
            │               │               │
            └───────────────┼───────────────┘
                            ↓
                     Snort Detection
                            ↓
                         Alert
                            ↓
                     /var/log/snort

The manual's expected result is that the different scan types are detected and logged as alerts.

Quick Revision Sheet — All 9 Experiments
No.	Experiment	Main Tool	Main Command/Concept
1	Install Snort + IDS	Snort	snort -A console ...
2	Advanced alert + email	Snort + rsyslog + Python	alert_syslog
3	Sensitive HTTP GET	Snort + curl	content:"/admin"
4	TCP SYN flood	hping3 + Wireshark	SYN-only filter
5	Loopback sniffing	tcpdump + Snort	-i lo
6	IP spoofing	Scapy + tcpdump	forged src
7	Linux firewall	iptables	INPUT/FORWARD/OUTPUT
8	VPN	Cisco Packet Tracer	Router/tunnel configuration
9	NIDS scans	Snort + Nmap	7 custom rules
Most important commands to memorize
# Snort version
snort -V

# Test configuration
sudo snort -T -c /etc/snort/snort.conf

# Snort IDS
sudo snort -A console -q -c /etc/snort/snort.conf -i eth0

# Loopback Snort
sudo snort -A console -q -c /etc/snort/snort.conf -i lo

# Packet capture
sudo tcpdump -i lo -nn -w loopback_capture.pcap

# Read PCAP
tcpdump -r loopback_capture.pcap -nn -tttt

# Loopback aliases
sudo ip addr add 127.0.0.2/8 dev lo
sudo ip addr add 127.0.0.3/8 dev lo

# Firewall
sudo iptables -L -v
sudo iptables -L --line-numbers
sudo iptables -F

# Nmap
sudo nmap -sS ...
sudo nmap -sF ...
sudo nmap -sN ...
sudo nmap -sX ...
sudo nmap -sU ...
