---
tags: [notes, networking, nmap, port-scanning, tcp-ip, cybersecurity]
source: "Personal Networking & Nmap Notes"
created: 2026-06-20
status: in-progress
---

# 🌐 TCP/UDP Fundamentals & Nmap Port Scanning

> A practical walkthrough of TCP/UDP basics, the TCP header & flags, the 3-way handshake, and the core Nmap port-scanning techniques every network/security learner should know.

---

## 📋 Table of Contents

- [TCP vs UDP](#-tcp-vs-udp)
- [The TCP Header](#-the-tcp-header)
- [TCP Flags](#-tcp-flags)
- [The TCP 3-Way Handshake](#-the-tcp-3-way-handshake)
- [Understanding Ports](#-understanding-ports)
- [Nmap Port Scanning Techniques](#-nmap-port-scanning-techniques)
- [Specifying IP Ranges](#-specifying-ip-ranges)
- [Customizing Scans](#-customizing-scans)
- [Nmap Scripting Engine (NSE)](#-nmap-scripting-engine-nse)
- [Key Takeaways](#-key-takeaways)

---

## 🧠 TCP vs UDP

| Feature     | TCP                            | UDP                       |
|-------------|---------------------------------|----------------------------|
| Connection  | Connection-oriented            | Connectionless             |
| Reliability | Reliable (guaranteed delivery) | Unreliable (no guarantee)  |
| Speed       | Slower                         | Faster                     |
| Use cases   | Web, email, file transfer      | Gaming, streaming, DNS     |

---

## 🧠 The TCP Header

The **TCP header** is a set of information attached to every TCP packet. It tells the receiver how to handle the data and how the connection should behave.

**Main fields worth knowing:**

- **Source Port** → where the packet came from
- **Destination Port** → where it is going
- **Sequence Number** → order of the data
- **Acknowledgment Number** → confirms received data
- **Flags** → control the connection's behavior
- **Window Size** → controls the flow of data
- **Checksum** → error checking

![TCP Header structure](Screenshot%20From%202026-06-19%2019-50-34.png)

---

## 🧠 TCP Flags

**TCP flags** are small bits inside the TCP header that tell the receiver what the packet wants to do. There are 6 main flags:

| Flag | Meaning |
|------|---------|
| **SYN** | Start a connection |
| **ACK** | Acknowledge received data ("I have your message") |
| **FIN** | Finish / close the connection ("I have no more data to send") |
| **RST** | Reset / force-close the connection ("something's wrong, reset it") |
| **PSH** | Push data to the application immediately ("send this to me fast") |
| **URG** | Mark data as urgent |

---

## 🤝 The TCP 3-Way Handshake

When a client wants to establish a TCP connection with a server, they go through the **3-way handshake** process to start the connection:

1. The client sends a packet with the **SYN** flag → *"Hey server, I want to connect."*
2. The server replies with a **SYN, ACK** packet → *"I got your request, ready to connect."*
3. The client responds with an **ACK** packet → *"Great, connection established — ready to send data."*

```
Client                          Server
  |  ------ SYN --------------->  |
  |  <----- SYN, ACK -----------  |
  |  ------ ACK --------------->  |
  |        Connection open        |
```

> 🧠 **In my own words (Arabic):**
> لما العميل عايز يعمل اتصال TCP مع السيرفر بيحصل عملية اسمها **3-way handshaking** عشان يبدأوا اتصال بينهم.
>
> الرسالة بتبدأ من العميل لما يبعت Packet عليها علم **SYN** ويقول فيها: "يا سيرفر أنا عايز أعمل معاك اتصال."
>
> لما الباكت توصل للسيرفر وهو يقرر إنه يوافق على الاتصال، بيرد بباكت فيها الـ flag **SYN, ACK** ويقول: "يا عميل أنا شوفت طلبك وموافق."
>
> لما العميل يستقبل الرسالة دي، بيرد بباكت فيها علم **ACK** ويقول: "تمام كده، الاتصال اتعمل، استلم مني بقى."

![3-way handshake example](Screenshot%20From%202026-06-19%2020-12-15.png)

---

## 🔌 Understanding Ports

A **port** is a logical number used to identify a specific service or application running on a device in a network. There are **65,535** ports in total.

> 🧠 **Analogy (Arabic):** العمارة هي الجهاز بتاعك، والعمارة دي لازم يكون ليها عنوان يحددلي هي فين — دي وظيفة الـ `IP`. بعد كده، العمارة دي فيها شقق كتير: كل شقة فيها واحد، ده نجار وده سباك وده محامي وده مبرمج — وهي دي الـ **port**.

### Port Status (in Nmap)

| Status | Meaning |
|--------|---------|
| `open` | Port is open and ready to receive packets |
| `closed` | Port has a service but it's closed to external traffic |
| `filtered` | A firewall is protecting/blocking this port |
| `unfiltered` | Nmap can't determine whether it's open or closed |
| `open\|filtered` | Nmap can't determine whether it's open or filtered |
| `closed\|filtered` | Nmap can't determine whether it's closed or filtered |

---

## 🛠️ Nmap Port Scanning Techniques

The most common Nmap scan types:

| # | Scan Type | Command | What It Does |
|---|-----------|---------|---------------|
| 1 | TCP Connect Scan | `nmap -sT <target>` | Completes a full TCP connection on each port |
| 2 | TCP SYN Scan | `sudo nmap -sS <target>` | "Half-open" stealth scan — never completes the handshake |
| 3 | UDP Scan | `sudo nmap -sU <target>` | Sends UDP packets; relies on ICMP responses to infer port state |
| 4 | TCP NULL Scan | `sudo nmap -sN <target>` | Sends a packet with **no flags** set |
| 5 | TCP FIN Scan | `sudo nmap -sF <target>` | Sends a packet with only the **FIN** flag |
| 6 | TCP Xmas Scan | `sudo nmap -sX <target>` | Sends a packet with **FIN, URG, PSH** flags |
| 7 | Ping Sweep | `nmap -sn <ip-range>` | Discovers live hosts via ICMP — doesn't scan ports |

### 1. TCP Connect Scan

Nmap completes a full TCP connection with each port to determine whether it's open or closed.

```bash
nmap -sT <target-ip>
```

![TCP connect scan example](Screenshot%20From%202026-06-20%2006-22-07.png)

### 2. TCP SYN Scan

Also known as a **"Half-open"** or **"Stealth"** scan. It never completes the full TCP connection: Nmap sends a **SYN** packet, and once the server replies with **SYN, ACK**, Nmap responds with **RST** to tear the connection down before it's ever established.

```bash
sudo nmap -sS <target-ip>
```

> 🧠 **In my own words (Arabic):** شوف، أنا دلوقتي جاي أطلب منك إني أعمل اتصال معاك، روحت بعتلك رسالة SYN، روحت انت قلتلي "معاك يا باشا" وبعتلي SYN, ACK — أقولك بقى: "ماني هرد عليك وهقولك خلاص مش عايزك" وأبعتلك RST، أقولك بيها: "خلاص، مش عايزك تاني."

**Pros:**
- Faster than a TCP connect scan since it never completes the connection.

**Cons:**
- Requires root privileges, since Nmap needs to craft and modify raw packets.

![SYN scan example](Screenshot%20From%202026-06-20%2006-31-52%201.png)

### 3. UDP Scan

**UDP (User Datagram Protocol)** is connectionless and fast — when a client sends UDP data it doesn't wait for an ACK, which is what makes it quick.

Because of this, a UDP scan can't always tell if a port is open: Nmap sends UDP packets and, if there's no response, it can't distinguish between *"open"* and *"filtered by a firewall."*

> ⚠️ However, if the port is **closed**, the target responds with an **ICMP Type 3, Code 3** message ("Destination/Port Unreachable") — that's how Nmap confirms a closed port.

```bash
sudo nmap -sU <target-ip>
```

![UDP scan ICMP unreachable response](Screenshot%20From%202026-06-20%2006-43-36%201.png)

### 4. TCP NULL Scan

Nmap sends a packet with **no flags set**. If the port is closed, the target responds with an **RST, ACK** packet.

```bash
sudo nmap -sN <target-ip>
```

### 5. TCP FIN Scan

Nmap sends a packet with only the **FIN** flag. If the port is closed, the target responds with **RST, ACK**.

```bash
sudo nmap -sF <target-ip>
```

### 6. TCP Xmas Scan

Nmap sends a packet with the **FIN, URG, PSH** flags set ("lit up like a Christmas tree"). If the port is closed, the target responds with **RST, ACK**.

```bash
sudo nmap -sX <target-ip>
```

> ⚠️ **Gotcha:** This logic only tells you a response means *closed*. If a port is **open**, there's usually no response at all — which looks identical to a port being **filtered** by a firewall. NULL, FIN, and Xmas scans all share this ambiguity.
>
> 🧠 **In my own words (Arabic):** تعالى بقى، انت لسه مشفتش الغلط فين — الرد بييجي بس لما تكون الـ port في حالة الـ closed. طيب لو هي في حالة open مفيش رسالة هتيجي، فبالتالي مش هتعرف انها كده مفتوحة... لأ، طبعًا افرض فيه firewall بيمنع الـ packets — هي دي مشكلة التلاتة .
>
> طيب ليه اتعملوا أصلًا؟ زمان كان الـ firewall بيحظر أي packet عندها  SYN flag، فعملوا الطريقة دي عشان يتخطوها (**firewall evasion**) — بس في التقنيات الحديثة بقت مش بتنفع كتير.

### 7. Ping Sweep

When you're dealing with a "black box" network and want to discover which hosts are live and how they're connected, you use a **ping sweep**. Nmap sends ICMP packets to each IP — a response means the host is live; no response means it's likely dead.

```bash
nmap -sn <ip-range>
```

> 🧠 **In my own words (Arabic):** طيب تعالى، رايح فين؟ افرض فيه firewall حاظر الـ packet، هتعرف منين إن الهوست بقى dead؟ ده هو عيبها بقى. وعشان كده، لو فيه firewall بيمنعك توصل للنتورك، نستخدم `-Pn` لما يديك error وقت العمل.

> 💡 **Tip:**
> - `-sn` → Nmap doesn't scan ports, it only checks if hosts are alive.
> - `-Pn` → tells Nmap to scan directly without sending an ICMP probe first. The downside: it takes much longer, since it scans **all** ports on every live *and* dead host.

---

## 🌍 Specifying IP Ranges

| Format | Example | Meaning |
|--------|---------|---------|
| `ip/subnet` | `172.0.0.1/16` | CIDR notation |
| `ip-n` | `172.0.0.1-255` | Range using the last octet |

---

## ⚙️ Customizing Scans

### Port Selection

| Option | Purpose |
|--------|---------|
| `-p 22,445` | Scan specific ports (comma-separated) |
| `-p 1-999` | Scan a port range |
| `-p-` | Scan all 65,535 ports |

### Timing & Performance

| Option                                    | Purpose                                                                                                                |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `-T<0-5>`                                 | Timing template: `0` = slowest, `5` = fastest (default is `T3`)                                                        |
| `-r`                                      | Scan ports in consecutive (non-randomized) order                                                                       |
| `--max-rate` / `--min-rate`               | Set packets-per-second limits                                                                                          |
| `--min-parallelism` / `--max-parallelism` | Set how many packets are handled in parallel                                                                           |
| `--scan-delay`                            | Add a delay between packets                                                                                            |
| -A                                        | Enables **OS detection**, **service/version detection**, **default NSE scripts**, and **traceroute** in a single scan. |

> 💡 **Tip:** Going from `T0` to `T5` trades accuracy for speed. In CTFs, `-T4` is a solid middle ground.

### Firewall / IDS Evasion

| Option | Purpose |
|--------|---------|
| `-f` | Fragment packets into smaller pieces |
| `--mtu` | Like `-f`, but gives finer control over fragment size |
| `--badsum` | Send packets with an invalid checksum to test/deceive the firewall |

---

## 📜 Nmap Scripting Engine (NSE)

Nmap includes scripts written in **Lua** that help with tasks like brute-forcing, reconnaissance, and vulnerability detection. Scripts fall into 7 categories:

| Category | Purpose |
|----------|---------|
| `safe` | Won't disrupt or crash the target |
| `intrusive` | May affect the target negatively |
| `vuln` | Checks for known vulnerabilities |
| `exploit` | Attempts to exploit a vulnerability |
| `auth` | Tests authentication mechanisms |
| `brute` | Performs brute-force attacks |
| `discovery` | Gathers more information about the target/network |

### Running Scripts

| Option | Purpose |
|--------|---------|
| `--script <category>` | Run all scripts in a given category |
| `--script <script-name>` | Run a specific script |
| `--script <script-name> --script-args scriptname.arg=value` | Pass an argument to a script |
| `--script-help <script-name>` | View help for a specific script |

### Finding Scripts

- **Online** — search the official Nmap website for scripts and documentation.
- **Locally** — script files live in `/usr/share/nmap/scripts`, and `script.db` in that directory lists every script along with its category. To search locally:
  1. `ls` the scripts directory piped through `grep`
  2. `grep` directly inside `script.db`

---

## 📌 Key Takeaways

- TCP is reliable and connection-oriented; UDP is fast but connectionless and unreliable.
- The 3-way handshake (**SYN → SYN/ACK → ACK**) is how every TCP connection begins.
- A **SYN scan** is faster and stealthier than a full **connect scan**, but needs root privileges.
- **UDP, NULL, FIN, and Xmas scans** all share the same blind spot: no response can mean *open* **or** *filtered* — firewalls are the wildcard.
- Timing (`-T`), fragmentation (`-f`), and checksum tricks (`--badsum`) exist mainly to evade firewalls and IDS — modern security tools are largely wise to them.
- NSE scripts extend Nmap far beyond port scanning, into recon, vulnerability detection, and brute-forcing.

---

> 📌 *Source: Personal Networking & Nmap Notes. Last updated: 2026-06-20.*
