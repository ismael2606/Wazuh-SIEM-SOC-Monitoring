
# Summary
A self-hosted SIEM lab where I deployed Wazuh in Docker, spun up a victim endpoint, and then ran MITRE ATT&CK techniques against it to test what the default ruleset actually detects and what it doesn't. Covers the log pipeline ingestion from the Linux endpoint(s) to the Wazuh Manager.

---

## Components


* Kali Linux VM (Attacker): Perform MITRE ATTACK techniques (e.g., SSH brute force, root escalation.

* Ubuntu VM (Endpoint): Victim device
                                 
* Wazuh Agent: Installed on the endpoint, it ships logs via Port 1514/TCP.
                             
* Wazuh Server on Docker: Indexes and visualizes the data in the Dashboard via the web portal.


##  Prerequisites

### 1 - Docker Setup 
To install: https://docs.docker.com/engine/install/ubuntu/

### 2 - Wazuh installation via Docker
Instructions: https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html
  * NOTE: Installation can be done as a single or multiple node stack, depending on your preference.

### Detection results

#### SSH Brute Force |  ATT&CK T1110.001 - Brute Force: Password Guessing

* **Victim:** account created with a weak password

```sudo useradd -m ubuntu-prod && echo 'victim:password' | sudo chpasswd```

* **Attacker:** Used Hydra to perform brute force to log in.

```hydra -l ubuntu-prod -P /usr/share/wordlists/rockyou.txt ssh://VICTIM_IP -t 4```

<img width="1097" height="203" alt="image" src="https://github.com/user-attachments/assets/f8f0bc3a-34c5-47e0-aefa-5d103d54fb46" />



2. Persistent Promiscuous Mode Configuration
Force the internal network interface card (NIC) out of standard filtering mode to allow full frame inspection. Create a persistent systemd lifecycle management service unit:

```bash
sudo nano /etc/systemd/system/promisc.service
```

3. Paste the configuration below (replace eth0 with the target network interface identifier derived from ip a):


```
Ini, TOML
[Unit]
Description=Bring up interface in promiscuous mode
After=network.target

[Service]
Type=oneshot
ExecStart=/sbin/ip link set eth0 promiscuous on
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```


4. Reload the system daemons

```bash
sudo systemctl daemon-reload && sudo systemctl enable promisc.service --now
```

5. Disable Hardware Packet Offloading

```bash
sudo apt update && sudo apt install ethtool -y
```

* To make execution persistent across reboots: `sudo ethtool -K eth0 gro off lro off`



### 💾Step 2: Install and Configure Suricata NIDS

```bash
sudo apt install suricata -y
sudo nano /etc/suricata/suricata.yaml
```

* Update these values inside the `suricata.yaml` file:
   * HOME_NET: `"[192.168.1.0/24]"` <- `replace with your local subnet`

   * af-packet interface: `eth0` <- `replace with your network interface`

* Verify eve-log is active -> `enabled:yes`

* Initialize Suricata and update signatures to pinpoint malicious vectors

```bash
sudo suricata-update
sudo suricata -T -c /etc/suricata/suricata.yaml -v
sudo systemctl enable suricata --now
```

### 🕵 Step 3: Install Wazuh

1. Run the command below to install the Wazuh Manager, Indexer and Dashboard tools
   `curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh && sudo bash wazuh-install.sh -a`
   * **NOTE**: Replace the `x` placeholder from the URL with the latest wazuh version if needed.

2. Once the installation is complete, it'll display the admin credentials log in to the Wazuh web portal. Save them in a secure location. Access the interface at `https://<ip-of-SIEM-server>` using the generated admin credentials.

### ⛓️‍💥Step 4: Link the Endpoint and NIDS together

1. Open the Wazuh config on your endpoint device: `sudo nano /var/ossec/etc/ossec.conf`
2. Add the Suricata File Monitor:
   * Scroll down to the log-monitoring section and paste this tracking block directly inside the main <ossec_config> wrapper:
  ```xml
   <localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

3. Apply changes with `sudo systemctl restart wazuh-agent`

### 💥 Step 5: Attack Emulation

1. From another VM, you can stage, initiate an aggressive network scan with `nmap -A -T4 <endpoint-ip>`
**NOTE**: Other intrusive scans and staged attacks can be used to see how the NIDS reacts.

### 📈 Step 6: Dashboard Monitoring

1. Open your browser and navigate to the Wazuh Web Console (`https://ip-of-SIEM-server`) and navigate to Threat Intelligence > Threat Hunting to see the security events.
   
