# 🛡️ Network Security Lab Guide

## 📋 Introduction
This guide documents the steps to build a virtual lab to test and implement essential network security techniques using:

* **iptables** (Linux firewall)
* **Snort** (Intrusion Detection System)
* **Wireshark** (traffic analyzer)
* **OpenSSL + Apache2** (HTTPS server setup)

We use **VirtualBox** with 3 VMs:
  <p align="center">
<img src="https://github.com/user-attachments/assets/57711009-e578-4387-8018-cbc38232d313" width="50%"></p>

* 🔒 **Firewall** (Kali Linux)
* 🧑‍💻 **Client** (Ubuntu or any machine you have)
* 🖥️ **Server** (kali Server or any machine you have)

---

## 🖥️ 1. Virtual Machines Setup

### Network Topology

```
Client <--> Firewall <--> Server
```
Use **Internal Network** mode in VirtualBox for full isolation in each machine:
  <p align="center">
 <img src="https://github.com/user-attachments/assets/cd05367b-54a9-4c89-956e-4056abe7de12" width></p>
   
* Client communicates *only* with Firewall
   
* Server communicates *only* with Firewall
  
  <p align="center">
  <img src="https://github.com/user-attachments/assets/063312da-ce1d-42cf-9a73-7a87a5a23cb1" width="45%" style="margin-right: 10px;"/>
  <img src="https://github.com/user-attachments/assets/7ff5435d-a9f7-4b06-9413-d46353c6b6f2" width="45%"/>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/387e071c-84db-4f57-bbea-6b8338abe9ae" width="45%" style="margin-right: 10px;"/>
  <img src="https://github.com/user-attachments/assets/34b10fa1-76ed-4191-8ba4-bb2dcab943fe" width="45%"/>
</p>

### Interface Configuration
For each **machine** (Client, Firewall, and Server), you have manually assign the following IP addresses to their respective network interfaces:

<div align="center">

  
<table>
  <tr>
    <th>Machine</th>
    <th>Interface</th>
    <th>IP Address</th>
    <th>To</th>
  </tr>
  <tr>
    <td>Client</td>
    <td><code>enp0s3</code></td>
    <td><code>192.168.10.2</code></td>
    <td>Firewall eth0</td>
  </tr>
  <tr>
    <td>Firewall</td>
    <td><code>eth0</code></td>
    <td><code>192.168.10.1</code></td>
    <td>Client</td>
  </tr>
  <tr>
    <td>Firewall</td>
    <td><code>eth1</code></td>
    <td><code>192.168.20.1</code></td>
    <td>Server</td>
  </tr>
  <tr>
    <td>Server</td>
    <td><code>eth0</code></td>
    <td><code>192.168.20.2</code></td>
    <td>Firewall eth1</td>
  </tr>
</table>

</div>


Assign **static IPs** manually in `/etc/network/interfaces` 

### ⚙️ Automating IP Configuration with a Script

When working with VirtualBox in Internal Network mode, machines may lose their IP addresses upon reboot or interface changes. To avoid manually reconfiguring each time, it's good practice to create a dedicated script for each machine that sets its static IP addresses.

In the screenshot below, I demonstrate the creation and execution of such a script (firewall.sh) on the Firewall machine (Kali Linux).

```bash
sudo nano firewall.sh          # Create the script file
sudo chmod +x firewall.sh     # Make it executable
sudo ./firewall.sh            # Run the script
```
This script typically contains ip or ifconfig commands to assign static IPs to interfaces like eth0 and eth1.

<p align="center"> <img src="https://github.com/user-attachments/assets/6e6700ec-d227-48bf-a719-837ad45e074c" width="47%" style="margin-right:10px;" /> <img src="https://github.com/user-attachments/assets/3a63eaab-b030-487a-ae68-ee31b323739b" width="47%" /> </p>
For each machine `server.sh` and `client.sh`follow the same steps by creating a script and add the ip addresses as showing in the table.
✅ This method saves time, avoids misconfigurations, and ensures consistent network behavior in your lab setup .

To make sure that you asign to each interface the right ip address run this command in the terminal 

<p align="center"><img src="https://github.com/user-attachments/assets/dd0251f5-1325-45ab-8950-0ac970cb1d81" width="50%"></p>

---
## 🌐 2. Set Up Apache2 Web Server (on Server VM)

### Install Apache2

```bash
sudo apt update
sudo apt install apache2
```

### Create a login page `login.html`

```bash
cd /var/www/html
sudo nano login.html
```
Paste the following basic HTML code inside login.html:
```bash <!DOCTYPE html>
<html>
<head>
  <title>Login Page</title>
</head>
<body>
  <h2>Login</h2>
  <form method="POST" action="/login">
    <label for="username">Username:</label><br>
    <input type="text" id="username" name="username"><br><br>
    
    <label for="password">Password:</label><br>
    <input type="password" id="password" name="password"><br><br>
    
    <input type="submit" value="Login">
  </form>
</body>
</html>
```
Save and exit nano:

Press `Ctrl + O` → then `Enter` to save.

Then press `Ctrl + X` to `exit`.

After configuring the server and website, test the setup by accessing the server’s IP address from both server and the client machine. 

Test by accessing `http://192.168.20.2/login.html` use firefox or any browser :

<p align="center"><img src="https://github.com/user-attachments/assets/bf03806d-8bf2-47b8-97bc-82147ed6e25d" width="50%"></p>

Test the access also from the client :

<p align="center"><img src="https://github.com/user-attachments/assets/4b9f574f-3362-4eae-bb1d-0b0d90f717c5" width="%50"></p>

---
## 🔎📡 3.Filtering Traffic with Wireshark
In this section, we’ll use Wireshark to capture and analyze traffic between the Client and the Server.
The goal is to compare HTTP vs HTTPS and understand how unencrypted traffic can expose sensitive data.

🧭 Step-by-step Instructions
* Open Wireshark on the Firewall machine.
* Select the interface connected to either the Client (eth0) or Server (eth1).
* Start capturing packets by clicking the blue shark icon.
* Apply a filter to only see HTTP traffic:
  
```bash
http 
```
These screenshots show the live packet capture window and the login page used to trigger HTTP requests.
<p align="center">
  <img src="https://github.com/user-attachments/assets/b534371c-d5a7-4d36-99e3-7681feb0b7e0" width="30%" style="margin-right: 10px;" />
  <img src="https://github.com/user-attachments/assets/c69fbd5d-464e-4458-872c-389126b0d654" width="30%" style="margin-right: 10px;" />
  <img src="https://github.com/user-attachments/assets/af891363-db39-4471-a82a-1cb855196391" width="30%" />
</p>

### 📬 Filtering for HTTP Traffic

Captured HTTP request with sensitive data in plaintext:

![image](https://github.com/user-attachments/assets/e66de6b3-805e-4dad-bb65-5f2877ecabcd)

This line confirms that username and password are transmitted without encryption over HTTP.
```bash
GET /test1/?username=salut&password=kan HTTP/1.1
```
### ⚠️ Security Risk

 HTTP transmits sensitive data in plaintext, making it vulnerable to interception. To protect this data, HTTPS should be used, as it encrypts communication, ensuring the confidentiality and integrity of sensitive information.
 
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
<p align="center"><img src="https://github.com/user-attachments/assets/48afee02-ae91-4ecc-97b1-c48cba5708c0" width="50%"></p>

### Update `ports.conf`:

Make sure it includes:
```
Listen 443
```
<p align="center">
  <img src="https://github.com/user-attachments/assets/0087d1e0-6170-43a8-9135-a32bf280a9b0" width="48%" style="margin-right: 2%;" />
  <img src="https://github.com/user-attachments/assets/3f7f848f-d9bf-4240-9216-702929f28efd" width="48%" />
</p>

### Common Warning:
![image](https://github.com/user-attachments/assets/543b6b70-f0d9-4082-8df4-80ed08768e8e)

> ⚠️ Browser may show "Potential Security Risk Ahead" due to self-signed cert. ✅ Accept the risk and continue.

### Capture Traffic with Wireshark:
follow the same steps of using wireshark ,now filter with tls
```bash
tls
```
You’ll observe:
* HTTPS request: encrypted
![image](https://github.com/user-attachments/assets/e5d3a227-41f4-460f-b70a-bda513129fde)

---
## 🔥 5. Configure iptables on Firewall

### Block HTTPS traffic:

```bash
sudo iptables -A INPUT -p tcp --dport 443 -j DROP
```
✅ Result: HTTPS blocked.

<p align="center"><img src="https://github.com/user-attachments/assets/6bcc56ec-758f-469a-93cb-599471e27fe7" width="50%"></p>

### Unblock HTTPS:
Steps to Stop iptables from Filtering Traffic:

1.	Flush all current iptables rules:
This will remove all the current filtering rules.
```bash
sudo iptables -F
```
2.	Set the default policies to ACCEPT:
This will allow all incoming, outgoing, and forwarded traffic by default.
```bash
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```
4.	Verify that the rules have been cleared:
After applying the changes, check the iptables configuration to ensure there are no active rules:
```bash
sudo iptables -L -n
```
You should see:
```bash
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
You can verify with:

```bash
sudo iptables -L -n -v
```
<p align="center"><img src="https://github.com/user-attachments/assets/6712bc61-f5d6-437d-8caf-9a51aca25daa" width="50%"></p>

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
🔗 LinkedIn: 
