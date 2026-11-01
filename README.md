# Cyber Threat Intelligence and Analytics Lab (23AI2007) - Commands

**Name:** FEVIN.K | **Register No:** URK24CS5067

Run each step in the order given. Values in `<angle brackets>` are yours to replace. Lines starting with `#` are notes, and steps without a code block are done in the web interface.

## Contents

1. [Ex 1 - OSINT using Recon-ng](#ex-1---osint-using-recon-ng-kali-linux)
2. [Ex 2 - Nmap scan traffic analysis with Wireshark](#ex-2---nmap-scan-traffic-analysis-with-wireshark-2-kali-vms)
3. [Ex 3 - Malware analysis with ThreatFox and VirusTotal](#ex-3---malware-analysis-with-threatfox-and-virustotal)
4. [Ex 4 - Splunk Enterprise installation and configuration](#ex-4---splunk-enterprise-installation-and-configuration-windows-cmd-as-administrator)
5. [Ex 5 - Splunk: upload data, search, chart, dashboard](#ex-5---splunk-upload-data-search-chart-dashboard)
6. [Ex 6 - Collecting information for security monitoring](#ex-6---collecting-information-for-security-monitoring)
7. [Ex 7 - Knowledge objects: event type and tag](#ex-7---threat-detection-with-knowledge-objects-event-type-and-tag)
8. [Ex 8 - Visualizations with SPL](#ex-8---visualizations-with-spl)
9. [Ex 9 - Scheduled reporting in Splunk](#ex-9---scheduled-reporting-in-splunk-windows-event-logs)
10. [Ex 10 - Alerts in Splunk (Event ID 4648)](#ex-10---monitoring-threats-with-alerts-in-splunk-event-id-4648)

---

## Ex 1 - OSINT using Recon-ng (Kali Linux)

**Step 1: Update packages**
```bash
sudo apt update
```

**Step 2: Install Recon-ng**
```bash
sudo apt install recon-ng -y
```

**Step 3: Launch Recon-ng**
```bash
recon-ng
```

**Step 4: Refresh and view the marketplace**
```
marketplace refresh
marketplace search
```

**Step 5: Install and load the HackerTarget module**
```
marketplace install recon/domains-hosts/hackertarget
modules load recon/domains-hosts/hackertarget
```

**Step 6: View module info and set the target**
```
info
options set SOURCE karunya.edu
```

**Step 7: Run and show discovered hosts**
```
run
show hosts
```

**Step 8: Leave the module and find reporting modules**
```
back
marketplace search reporting
```

**Step 9: Install the reporting modules**
```
marketplace install reporting/html
marketplace install reporting/csv
marketplace install reporting/json
marketplace install reporting/xlsx
marketplace install reporting/xml
marketplace install reporting/list
```

**Step 10: Load the HTML report module and check its options**
```
modules load reporting/html
info
```

**Step 11: Set report details and output file**
```
options set CREATOR "Fevin.k"
options set CUSTOMER "Karunya Institute of Technology and Sciences"
options set FILENAME /home/kali/.recon-ng/workspaces/default/Experiment1_OSINT_Report.html
```

**Step 12: Generate the report, then exit**
```
run
exit
```

**Step 13: Open the report in the browser**
```bash
xdg-open /home/kali/.recon-ng/workspaces/default/Experiment1_OSINT_Report.html
```

---

## Ex 2 - Nmap scan traffic analysis with Wireshark (2 Kali VMs)

**Step 1: HOST machine - create one NAT network and attach both VMs**
```bash
VBoxManage natnetwork add --netname LabNet --network "10.0.2.0/24" --enable --dhcp on
VBoxManage modifyvm "Kali-Attacker" --nic1 natnetwork --nat-network1 LabNet
VBoxManage modifyvm "Kali-Victim" --nic1 natnetwork --nat-network1 LabNet
```

**Step 2: HOST - start both VMs**
```bash
VBoxManage startvm "Kali-Attacker"
VBoxManage startvm "Kali-Victim"
```

**Step 3: ATTACKER and VICTIM - note each IP address** (here attacker `10.0.2.15`, victim `10.0.2.3`)
```bash
ip -br addr
```

**Step 4: ATTACKER - test connectivity to the victim**
```bash
ping -c 4 10.0.2.3
```

**Step 5: VICTIM - (optional) open one port, check listening ports**
```bash
sudo systemctl start ssh
ss -tuln
```

**Step 6: VICTIM - start Wireshark, select eth0, start capture**
```bash
sudo wireshark
```

**Step 7: ATTACKER - run the TCP SYN scan on all ports**
```bash
sudo nmap -sS -p- -T5 10.0.2.3
```

**Step 8: VICTIM - stop capture, then apply these display filters one by one**
```
tcp.flags.syn == 1 && tcp.flags.ack == 0
tcp.flags.reset == 1 && tcp.flags.ack == 1
tcp.flags.syn == 1 && tcp.flags.ack == 1
ip.addr == 10.0.2.15 && ip.addr == 10.0.2.3
```

**Step 9: VICTIM - (optional) command-line capture instead of the GUI**
```bash
sudo tshark -i eth0 -w nmap_scan.pcap
tshark -r nmap_scan.pcap -Y "tcp.flags.syn==1 && tcp.flags.ack==0"
```

---

## Ex 3 - Malware analysis with ThreatFox and VirusTotal

**Step 1:** Open https://threatfox.abuse.ch, search a malware name (WannaCry, LockBit, Akira, CryptoLocker, 8Base), open an entry and copy its IOC.

**Step 2:** Open https://www.virustotal.com, paste the IOC and check the Detection, Details, Relations and Behavior tabs.

**Step 3: (Optional) same lookup from the command line - API keys needed**
```bash
curl -s -X POST https://threatfox-api.abuse.ch/api/v1/ -H "Auth-Key: <KEY>" \
     -d '{"query":"search_ioc","search_term":"<IOC>"}'
curl -s https://www.virustotal.com/api/v3/files/<SHA256> -H "x-apikey: <VT_KEY>"
curl -s https://www.virustotal.com/api/v3/domains/<DOMAIN> -H "x-apikey: <VT_KEY>"
```

---

## Ex 4 - Splunk Enterprise installation and configuration (Windows CMD as Administrator)

**Step 1: Install Splunk silently from the downloaded MSI**
```bat
msiexec /i splunk-10.4.3-x64-release.msi AGREETOLICENSE=Yes SPLUNKUSERNAME=splunkadmin SPLUNKPASSWORD=12345678 /quiet
```

**Step 2: Start Splunk and check status**
```bat
cd "C:\Program Files\Splunk\bin"
splunk start --accept-license
splunk status
sc query Splunkd
```

**Step 3: Confirm Splunk Web is listening; allow port 8000**
```bat
netstat -an | findstr 8000
netsh advfirewall firewall add rule name="Splunk Web" dir=in action=allow protocol=TCP localport=8000
```

**Step 4: Get the hostname**, then browse to `http://<HOSTNAME>:8000` and log in (`splunkadmin` / `12345678`)
```bat
hostname
```

**Step 5: Get the IPv4 address**, then browse to `http://<IPv4>:8000`
```bat
ipconfig
```

---

## Ex 5 - Splunk: upload data, search, chart, dashboard

**Step 1: Create `splunk_cti_sample_events.log`** (one event per line, format shown)
```
2026-09-01 15:30:18 LOGIN_SUCCESS user=student04 src_ip=192.168.1.34
2026-09-01 15:00:27 LOGIN_FAILED user=guest src_ip=198.51.100.50
2026-09-01 14:11:17 FIREWALL_DENY src_ip=203.0.113.88 dest_ip=192.168.1.20 dest_port=445
2026-09-01 13:05:10 PORT_SCAN src_ip=198.51.100.24 target=192.168.1.12 ports=22,80,443
2026-09-01 14:02:30 MALWARE_ALERT src_ip=192.168.1.45 host=LAB-PC-12 malware=DemoRansomware
2026-09-01 14:02:41 FILE_QUARANTINED host=LAB-PC-12 file=document.scr
```

**Step 2: Upload the file.** Settings > Add Data > Upload > Source type `cti_security_events` > Index `main` > Submit

```bat
:: or from CMD
splunk add oneshot "C:\data\splunk_cti_sample_events.log" -index main -sourcetype cti_security_events -auth splunkadmin:12345678
```

**Step 3: Search & Reporting - show failed logins**
```
index=main sourcetype=cti_security_events "LOGIN_FAILED"
```

**Step 4: Count failed logins per source IP**
```
index=main sourcetype=cti_security_events "LOGIN_FAILED"
| rex "src_ip=(?<src_ip>\S+)"
| stats count by src_ip
| sort - count
```

**Step 5:** Visualization > Column Chart > Save As > Dashboard

---

## Ex 6 - Collecting information for security monitoring

Set the time range so it covers 01/09/2026.

**Step 1: Collect all security events**
```
index=main sourcetype=cti_security_events
```

**Step 2: Filter failed logins**
```
index=main sourcetype=cti_security_events "LOGIN_FAILED"
```

**Step 3: Failed-login source IPs**
```
index=main sourcetype=cti_security_events "LOGIN_FAILED"
| rex "src_ip=(?<src_ip>\S+)"
| stats count by src_ip
| sort - count
```

**Step 4: Firewall-denied events**
```
index=main sourcetype=cti_security_events "FIREWALL_DENY"
```

**Step 5: Firewall source IPs and destination ports**
```
index=main sourcetype=cti_security_events "FIREWALL_DENY"
| rex "src_ip=(?<src_ip>[0-9.]+)"
| rex "dest_port=(?<dest_port>[0-9]+)"
| stats count by src_ip dest_port
| sort - count
```

**Step 6: Malware alerts, quarantined files, port scans** (run each separately)
```
index=main sourcetype=cti_security_events "MALWARE_ALERT"
index=main sourcetype=cti_security_events "FILE_QUARANTINED"
index=main sourcetype=cti_security_events "PORT_SCAN"
```

**Step 7: Summarize events by type**
```
index=main sourcetype=cti_security_events
| rex "^\S+\s+\S+\s+(?<event_type>\S+)"
| stats count by event_type
| sort - count
```

**Step 8: Top five event types**
```
index=main sourcetype=cti_security_events
| rex "^\S+\s+\S+\s+(?<event_type>\S+)"
| stats count by event_type
| sort - count
| head 5
```

**Step 9: Events over time, then total count** (run each separately)
```
index=main sourcetype=cti_security_events
| timechart count

index=main sourcetype=cti_security_events
| stats count
```

**Step 10: Visualize by type.** Run the Step 7 search > Visualization > Column Chart > Save As > Existing Dashboard.
- Dashboard: `Cyber Threat Intelligence Dashboard`
- Panel title: `Security Events by Event Type`

**Step 11: Total count.** Run the `stats count` search > Save As > Report, then add it to the same dashboard.
- Title: `Total Security Events`
- Content: Statistics Table
- Time Range Picker: No

---

## Ex 7 - Threat detection with knowledge objects (event type and tag)

**Step 1: Detect port scans**
```
index=main sourcetype=cti_security_events "PORT_SCAN"
```

**Step 2: Find the scanning source IP**
```
index=main sourcetype=cti_security_events "PORT_SCAN"
| rex "src_ip=(?<src_ip>\S+)"
| stats count by src_ip
| sort - count
```

**Step 3: Create the event type.** Settings > Event types > New Event Type
- Name: `Port_Scan_Activity`
- Search string: `index=main sourcetype=cti_security_events "PORT_SCAN"`

**Step 4: Verify the event type**
```
index=main eventtype=Port_Scan_Activity
```

**Step 5: Add the tag `port_scan` to the event type.** Verify in Settings > Tags > List by tag name.

Optional config-file version in `...\Splunk\etc\apps\search\local\`:
```
# eventtypes.conf
[Port_Scan_Activity]
search = index=main sourcetype=cti_security_events "PORT_SCAN"

# tags.conf
[eventtype=Port_Scan_Activity]
port_scan = enabled
```

**Step 6: Verify the tag**
```
index=main eventtype=Port_Scan_Activity tag::eventtype=port_scan
```

---

## Ex 8 - Visualizations with SPL

Run each search, open the Visualization tab and choose the chart type.

**Step 1: Pie chart - event types**
```
index=main sourcetype=cti_security_events
| rex "^\S+\s+\S+\s+(?<event_type>\S+)"
| stats count by event_type
| sort - count
```

**Step 2: Line chart - events over time**
```
index=main sourcetype=cti_security_events
| timechart span=1h count
```

**Step 3: Bar chart - firewall activity**
```
index=main sourcetype=cti_security_events
| rex "^\S+\s+\S+\s+(?<event_type>\S+)"
| search event_type=FIREWALL_DENY OR event_type=FIREWALL_ALLOW
| stats count by event_type
```

**Step 4: Column chart - DNS activity**
```
index=main sourcetype=cti_security_events
| rex "^\S+\s+\S+\s+(?<event_type>\S+)"
| search event_type=DNS_QUERY OR event_type=DNS_BLOCK
| stats count by event_type
```

**Step 5: Column chart - login activity**
```
index=main sourcetype=cti_security_events
| rex "^\S+\s+\S+\s+(?<event_type>\S+)"
| search event_type=LOGIN_SUCCESS OR event_type=LOGIN_FAILED
| stats count by event_type
```

---

## Ex 9 - Scheduled reporting in Splunk (Windows Event Logs)

**Step 1: Enable logon auditing** (CMD as Administrator)
```bat
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

**Step 2: Collect Windows logs.** Settings > Data Inputs > Local Event Logs > localhost > select Security and System > Save

Optional config-file version in `C:\Program Files\Splunk\etc\system\local\inputs.conf`:
```
[WinEventLog://Security]
disabled = 0
[WinEventLog://System]
disabled = 0
```

**Step 3: Restart Splunk**
```bat
cd "C:\Program Files\Splunk\bin"
splunk restart
```

**Step 4: Verify incoming events** (time range: Last 15 minutes)
```
index=* earliest=-15m latest=now
| stats count by index, sourcetype, source
```

**Step 5: Search Security events**
```
index=main sourcetype="WinEventLog:Security" earliest=-15m latest=now
```

**Step 6: Summarize by host for 24 hours**
```
index=main sourcetype="WinEventLog:Security" earliest=-24h latest=now
| stats count by host
| sort - count
```

**Step 7:** Save As > Report: `Scheduled Windows Security Monitoring`

**Step 8: Schedule it.** Edit > Edit Schedule: Schedule Report on, Run every day at 9:00, Time Range Last 24 hours > Save

Optional `savedsearches.conf` version:
```
[Scheduled Windows Security Monitoring]
enableSched = 1
cron_schedule = 0 9 * * *
dispatch.earliest_time = -24h
dispatch.latest_time = now
```

**Step 9: Verify.** Settings > Searches, reports, and alerts > run the report and view results.

---

## Ex 10 - Monitoring threats with alerts in Splunk (Event ID 4648)

**Step 1: Collect Windows logs** as in Ex 9 steps 2-3 (inputs, restart), then verify:
```
index=* earliest=-15m latest=now
| stats count by index, sourcetype, source
```

**Step 2: (Test) generate 4648 events from CMD**
```bat
runas /user:<other_user> cmd
```

**Step 3: List Event IDs for 24 hours**
```
index=main sourcetype="WinEventLog:Security" earliest=-24h latest=now
| stats count by EventCode
| sort - count
```

**Step 4: Search Event ID 4648**
```
index=main sourcetype="WinEventLog:Security" earliest=-24h latest=now EventCode=4648
```

**Step 5: Create the alert.** Save As > Alert
- Title: `Windows Explicit Credential Logon Alert`
- Permissions: Shared in App
- Alert type: Scheduled, hourly at 0 minutes, Expires 24 hours
- Trigger when: Number of Results is greater than 0, Trigger Once, Throttle off

**Step 6: Add the action.** + Add Actions > Add to Triggered Alerts > Save > View Alert (Enabled: Yes)

Optional `savedsearches.conf` version:
```
[Windows Explicit Credential Logon Alert]
search = index=main sourcetype="WinEventLog:Security" EventCode=4648
enableSched = 1
cron_schedule = 0 * * * *
dispatch.earliest_time = -24h
dispatch.latest_time = now
alert_type = number of events
alert_comparator = greater than
alert_threshold = 0
alert.digest_mode = 1
alert.expires = 24h
alert.track = 1
```
