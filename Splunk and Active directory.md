
# 🛡️ SOC Lab Project: Splunk + Active Directory + Sysmon + Attack Simulation

This repository documents the steps I followed to build a mini SOC (Security Operations Center) lab using **Splunk**, **Windows Active Directory**, **Sysmon**, and **Kali Linux** for testing detection capabilities.

---

## 🧪 Lab Environment

| System             | Role                             |
|--------------------|----------------------------------|
| Ubuntu Server      | Splunk Enterprise (Receiver)     |
| Windows Server 2022| Domain Controller (AD + Forwarder)|
| Windows 10         | Domain-joined Client + Forwarder |
| Kali Linux         | Attacker                         |

---

## 🔧 Setup Steps

### 1️⃣ Install Operating Systems

- **Ubuntu Server**  
  Download and install Ubuntu Server (22.04 LTS or similar):  
  https://ubuntu.com/download/server

- **Windows Server 2022**  
  Obtain through Microsoft evaluation center or VM image:  
  https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022

- **Windows 10 (Client)**  
  Download from Microsoft’s evaluation page or via Insider Preview:  
  https://www.microsoft.com/software-download/windows10

---

### 1. Install Splunk Enterprise on Ubuntu

```bash
wget -O splunk.tgz 'https://download.splunk.com/products/splunk/releases/10.0.0/linux/splunk-10.0.0-xxxxxxx.tgz'
tar -xvzf splunk.tgz
sudo mv splunk /opt/
sudo /opt/splunk/bin/splunk start --accept-license
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:changeme
```

> ✅ Make sure port `9997` is open on the Ubuntu server.
> ## ✅ Post-Installation Tips

- Add firewall rules if using `ufw`:
  ```bash
  sudo ufw allow 8000/tcp
  ```

---

### 2. Install Splunk Universal Forwarder on Windows (Both Server & 10)

Download link: https://www.splunk.com/en_us/download/universal-forwarder.html

Example `outputs.conf`:

```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.10.131:9997
```

Example `inputs.conf`:

```ini
[WinEventLog://System]
disabled = false
index = endpoint

[WinEventLog://Application]
disabled = false
index = endpoint

[WinEventLog://Security]
disabled = false
index = endpoint

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = false
renderXml = false
index = endpoint
```

---

### 3. Configure Windows Firewall Manually

- Run `wf.msc`
- Inbound Rule → New Rule → Port → TCP 9997 → Allow
- Apply to all profiles

---

### 4. Setup Active Directory (on Windows Server)

- Promote to Domain Controller via `Server Manager`
- Create a domain (e.g., `blue.local`)
- Add Organizational Units, Users
- Join Windows 10 to the domain

---

### 5. Verify Splunk Forwarding

On Ubuntu (Splunk Enterprise):

```bash
sudo /opt/splunk/bin/splunk list forward-server
netstat -plnt | grep 9997
```

On Windows PowerShell:

```powershell
Test-NetConnection 192.168.10.131 -Port 9997
```

✅ **If connection succeeds but logs aren't appearing in Splunk:**

1. **Test a custom file input:**

Add to `inputs.conf` on the forwarder:

```ini
[monitor://C:\testlog.txt]
disabled = false
index = endpoint
```

Then test it by running:

```powershell
echo test >> C:\testlog.txt
```

2. Restart the Splunk Forwarder service:
```powershell
Restart-Service splunkforwarder
```

---

### 6. Install and Configure Sysmon

Sysmon GitHub: https://github.com/Sysinternals/Sysmon

```powershell
sysmon.exe -accepteula -i sysmonconfig.xml
```

Use community config: https://github.com/SwiftOnSecurity/sysmon-config

---

### 7. Simulate Attack from Kali Linux

Use Hydra to test brute-force:

```bash
hydra -l user -P passwords.txt rdp://192.168.10.130
```

Then search failed logins in Splunk:

```spl
index=endpoint EventCode=4625
```

---

## 🧠 Key Learnings

- Fixed `no active forwarder` issue by ensuring ports and config files were correct
- Understood how domain join events show up in Splunk
- Captured Sysmon events: process creation (EventCode 1), network connection (EventCode 3)
- Hands-on troubleshooting made the concepts real

---

## 🔗 My GitHub: [github.com/Anith58](https://github.com/Anith58)

Full config files and screenshots coming soon!

---

## 📌 To Do

- [ ] Add correlation rules
- [ ] Build dashboards in Splunk
- [ ] Upload PCAPs and logs for community learning

---

## 📎 Tags

`#Splunk #SOC #Sysmon #SIEM #BlueTeam #ActiveDirectory #Homelab #DFIR #WindowsServer #CyberSecurity`
