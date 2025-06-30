# ClamAV Integration with Wazuh SIEM 
This project outlines how to integrate **ClamAV antivirus** with the Wazuh SIEM for real-time malware detection by forwarding ClamAV logs into Wazuh for alerting, monitoring, and dashboard visualization.

---

## 🎯 Objective  
To install ClamAV on the Wazuh server, configure it to forward logs to syslog, and validate that Wazuh detects and alerts on malware activity logged by ClamAV.

---

## 🔍 Why Integrate ClamAV with Wazuh?  
Integrating ClamAV with Wazuh allows for:
- **Real-time virus and malware detection** at the server level  
- **Visibility of antivirus events** inside the Wazuh dashboard  
- **Automated alerting** for infected files or suspicious activities  
- **Enhanced endpoint protection** without the need for expensive solutions

---

## 📚 Skills Learned  
- Installing and configuring ClamAV on RHEL-based systems  
- Enabling syslog forwarding for ClamAV logs  
- Verifying Wazuh’s ability to decode and alert on antivirus events  
- Testing and validating malware detection in a SIEM platform

---

## 🛠️ Tools Used  
<div>
  <img src="https://img.shields.io/badge/-Wazuh-0078D4?&style=for-the-badge&logo=Wazuh&logoColor=white" />
  <img src="https://img.shields.io/badge/-ClamAV-005571?&style=for-the-badge&logo=VirusTotal&logoColor=white" />
  <img src="https://img.shields.io/badge/-RHEL-EE0000?&style=for-the-badge&logo=Red-Hat&logoColor=white" />
</div>

---

## 📝 Deployment Steps

### 1. Install ClamAV on the Wazuh server
- Check if ClamAV is Already Installed
```bash
rpm -qa | grep clamav
```
- If theres no outputs then your good to install.
Enable the EPEL repository:
```bash
sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum repolist
```
- Install ClamAV and its daemon:
```bash
sudo dnf install -y clamav clamd clamav-update
clamscan --version
```
- Update the virus definitions:
```bash
sudo freshclam
```
- Start the clamav-freshclam service:
```bash
sudo systemctl start clamav-freshclam
sudo systemctl status clamav-freshclam
```

### 2. Configure ClamAV Logging
- Configure ClamAV Logging. Wazuh can monitor ClamAV logs via syslog to detect malware activity.
Forward ClamAV logs to Syslog
Edit the ClamAV daemon configuration file:
```bash
sudo nano /etc/clamd.d/scan.conf
```
Uncomment the line LogSyslog true.
Wazuh reads /var/log/syslog by default, so no further configuration on the Wazuh side is needed for basic log collection.
- Also uncomment/update the TCP address and socket configuration as below:
```bash
 TCPAddr 127.0.0.1
 TCPSocket 3310
 LocalSocket /run/clamd.scan/clamd.sock
 LogFile /var/log/clamd.scan
```

### 3. Start clamd deamon service
```bash
sudo systemctl enable clamd@scan
sudo systemctl start clamd@scan
sudo systemctl status clamd@scan
```

### 4. You can check the PID and IP configurations using:
```bash
sudo netstat -anp | grep -E "(Active|State|clam|3310)"
```

### 5. Scan a directory/file using clamscan:
```bash
clamscan -i /home
```

### 6. Verify Wazuh Configuration
- Decoders and Rules: Wazuh has built-in decoders and rules for ClamAV logs. You can find the rules file at /var/ossec/ruleset/rules/0320-clam_av_rules.xml on your Wazuh server. You shouldn't need to create custom decoders for standard ClamAV logs.
- Testing the Configuration:
Generate a ClamAV log event (e.g., by scanning a file containing a known virus signature - be cautious when doing this).
Check the Wazuh alerts log (/var/ossec/logs/alerts/alerts.log or alerts.json) on your Wazuh server for ClamAV-related alerts.
You can also use the wazuh-logtest tool on the Wazuh server to simulate ClamAV log messages and see if they are decoded and matched by the rules.
- Wazuh Dashboard
ClamAV alerts will be visible in the Wazuh dashboard under the "Security Events" or a similar section, often categorized under "Malware Detection."

---

### 👨‍💻 Author
Mario Tagaras | Florida State University Alum




















