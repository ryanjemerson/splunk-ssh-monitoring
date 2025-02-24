# splunk-ssh-monitoring

# SSH Failed Login Monitoring with Splunk

## Overview
This project sets up **Splunk** to monitor failed SSH login attempts on a Linux system. The logs are collected, visualized in a dashboard, and alerts are configured to notify excessive failed attempts.

---

## **Setup and Installation**
### 1. **Install Splunk on Ubuntu**
Download and install Splunk:
```bash
wget -O splunk.deb https://download.splunk.com/products/splunk/releases/9.4.0/linux/splunk-9.4.0-6b4ebe426ca6-linux-amd64.deb
sudo dpkg -i splunk.deb
```
Start Splunk:
```bash
sudo /opt/splunk/bin/splunk start --accept-license
```
Verify installation:
```bash
splunk --version
```
![Splunk Version](2%20-%20splunk%20version.png)

### 2. **Access Splunk Web Interface**
After starting Splunk, access it via:
```
http://<your-ip>:8000
```
Example:
![Splunk Dashboard](3%20-%20splunk%20dashboard.png)

---

## **Log Collection: Monitoring SSH Failed Logins**
### 3. **Enable Log Monitoring for `/var/log/auth.log`**
Add the log file to Splunk:
```bash
sudo /opt/splunk/bin/splunk add monitor /var/log/auth.log -sourcetype linux_secure -index main
```
Verify logs in Splunk:
```spl
index=main | head 50
```
![Index Search](4%20-%20splunk%20index%20search.png)

### 4. **Filter for Failed SSH Logins**
Run the following search query in Splunk:
```spl
index=main sourcetype=linux_secure "Failed password"
```
![Failed SSH Logins](5%20-%20Failed%20passwords.png)

---

## **Data Visualization in Splunk**
### 5. **Create a Dashboard for Failed SSH Logins**
- Save the failed login query as a **Dashboard Panel**.
- Use a **Bar Chart** to visualize failed login attempts per username.
- Configure the search query:
  ```spl
  index=main sourcetype=linux_secure "Failed password"
  | rex "Failed password for (?:invalid user )?(?<user>\S+) from"
  | stats count by user
  | sort -count
  ```

Example visualization:
![SSH Dashboard](6%20-%20Dashboard%20-%20SSH.png)
![Dashboard Visualization](7%20-%20Dashboard%20-%20Visualization.png)

---

## **Alerting for Excessive Failed Logins**
### 6. **Set Up an Alert**
- Go to **Search & Reporting** > Run the following query:
  ```spl
  index=main sourcetype=linux_secure "Failed password"
  | rex "Failed password for (?:invalid user )?(?<user>\S+) from"
  | stats count by user
  | where count > 5
  ```
- Save it as an **Alert** and configure:
  - **Trigger when results > 5**
  - **Real-time or Scheduled (every 5 min)**
  - **Log an Event in Splunk**
  
Example triggered alerts:
![Triggered Alerts](8%20-%20SSH%20Alert.png)

---

## **Generating Test Data**
To generate SSH failed login attempts:
```bash
ssh fakeuser@localhost
ssh hacker@localhost
ssh admin@localhost
```
For automated failures:
```bash
nano generate_ssh_failures.sh
```
Paste the following:
```bash
#!/bin/bash
users=("fakeuser" "hacker" "admin" "test" "root")
for user in "${users[@]}"; do
  ssh $user@localhost exit
done
```
Run the script:
```bash
chmod +x generate_ssh_failures.sh
./generate_ssh_failures.sh
```
Re-run the Splunk query to see the new logs.

---

## **Conclusion**
This project demonstrates how to:
- **Ingest SSH logs** into Splunk.
- **Analyze failed login attempts** using search queries.
- **Visualize SSH activity** in dashboards.
- **Trigger alerts** when brute-force attempts occur.

**Next Steps:**
- Extend monitoring to other system logs.
- Forward logs to a centralized SIEM.
- Automate response using Splunk alerts.

---
