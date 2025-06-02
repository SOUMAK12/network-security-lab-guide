# 🛡️ Network Security Lab Guide

**Cybersecurity Lab Project – 2024/2025**
*Prepared by: Soumaya Elkanfoud
*Supervisor: Mrs. Saiida Lazaar

---

## 📋 Introduction

This guide documents the steps to build a virtual lab to test and implement essential network security techniques using:

* **iptables** (Linux firewall)
* **Snort** (Intrusion Detection System)
* **Wireshark** (traffic analyzer)
* **OpenSSL + Apache2** (HTTPS server setup)

We use **VirtualBox** with 3 VMs:

![image](https://github.com/user-attachments/assets/57711009-e578-4387-8018-cbc38232d313)

* 🔒 **Firewall** (Kali Linux)
* 🧑‍💻 **Client** (Ubuntu)
* 🖥️ **Server** (Ubuntu Server or any machine you have)

---

## 🖥️ 1. Virtual Machines Setup

### Network Topology

```
Client <--> Firewall <--> Server
```

* Client communicates *only* with Firewall
* Server communicates *only* with Firewall

### Interface Configuration

Use **Internal Network** mode in VirtualBox for full isolation:

| Machine  | Interface | IP Address     | To            |
| -------- | --------- | -------------- | ------------- |
| Client   | `enp0s3`  | `192.168.10.2` | Firewall eth0 |
| Firewall | `eth0`    | `192.168.10.1` | Client        |
| Firewall | `eth1`    | `192.168.20.1` | Server        |
| Server   | `eth0`    | `192.168.20.2` | Firewall eth1 |

Assign **static IPs** manually in `/etc/network/interfaces` or via Netplan.

---

## 🔥 2. Configure iptables on Firewall

### Block HTTPS traffic:

```bash
sudo iptables -A INPUT -p tcp --dport 443 -j DROP
```

✅ Result: HTTPS blocked.

### Unblock HTTPS:

```bash
sudo iptables -D INPUT -p tcp --dport 443 -j DROP
```

✅ Result: HTTPS allowed again.

You can verify with:

```bash
sudo iptables -L -n -v
```

---

## 🌐 3. Set Up Apache2 Web Server (on Server VM)

### Install Apache2

```bash
sudo apt update
sudo apt install apache2
```

### Create a login page `login.html`

Place it in:

```
/var/www/html/login.html
```

Test by accessing `http://192.168.20.2/login.html` from the Client.

---

## 🔐 4. Configure HTTPS with OpenSSL

### Generate a Self-Signed Certificate:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
 -keyout /etc/ssl/private/apache-selfsigned.key \
 -out /etc/ssl/certs/apache-selfsigned.crt
```

### Enable SSL in Apache:

```bash
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo systemctl restart apache2
```

### Update `ports.conf`:

Make sure it includes:

```
Listen 443
```

### Common Warning:

> ⚠️ Browser may show "Potential Security Risk Ahead" due to self-signed cert. ✅ Accept the risk and continue.

---

## 🕵️ 5. Capture Traffic with Wireshark

### On the Firewall:

* Open Wireshark
* Select the interface (eth0 or eth1)
* Apply filter:

```bash
http || tls
```

### Example:

You’ll observe:

* HTTP login request: `username=soumaya&password=kanfoud` in plaintext
* HTTPS request: encrypted

---

## 🚨 6. Snort: Intrusion Detection Setup

### Install Snort (on Firewall):

```bash
sudo apt update
sudo apt install snort
```

### Test Snort in Packet Sniffer Mode:

```bash
sudo snort -i eth0 -A console
```

### Example Rules:

```bash
alert icmp any any -> any any (msg:"ICMP detected"; sid:1000001;)
```

Put custom rules in `/etc/snort/rules/local.rules`.

### Restart Snort:

```bash
sudo systemctl restart snort
```

✅ Snort now monitors suspicious traffic.

---

## ✅ Conclusion

You have successfully:

* Configured isolated networks in VirtualBox
* Deployed a secure HTTPS server
* Analyzed unencrypted/encrypted traffic
* Filtered traffic with `iptables`
* Detected intrusions with Snort

This lab forms a solid foundation in **hands-on network security** for beginners and students.

---

## 📎 Bonus

Feel free to fork this repo, add your own test cases, rules, or share suggestions via Pull Requests!

📧 Contact: [Soumaya Elkanfoud](mailto:youremail@example.com)
🔗 LinkedIn: [linkedin.com/in/yourprofile](https://linkedin.com/in/yourprofile)
