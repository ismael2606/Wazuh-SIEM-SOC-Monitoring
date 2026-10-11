
# Summary
A self-hosted SIEM lab where I deployed Wazuh in Docker, spun up a victim endpoint, and then ran MITRE ATT&CK techniques against it to test what the default ruleset actually detects and what it doesn't. Covers the log pipeline ingestion from the agent endpoints to the Wazuh Manager.

---

## Components


* Kali Linux VM (Attacker): Perform MITRE ATT&CK techniques (e.g., SSH brute force, privilege escalation).

* Ubuntu VM (Endpoint): Victim device
                                 
* Wazuh Agent: Installed on the endpoint, it ships logs via Port 1514/TCP.
                             
* Wazuh Server on Docker: Indexes and visualizes the data in the Dashboard via the web portal.


##  Prerequisites

### 1 - Docker Setup 
To install: https://docs.docker.com/engine/install/ubuntu/

### 2 - Wazuh installation via Docker
Instructions: https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html
  * NOTE: Installation can be done as a single or multiple node stack, depending on your preference.

### Dashboard Monitoring

Open your browser and navigate to the Wazuh Web Console (`https://ip-of-SIEM-server`) and navigate to Threat Intelligence > Threat Hunting to see the security events.


### Detection results

#### SSH Brute Force |  ATT&CK T1110.001 - Brute Force: Password Guessing

* **Victim:** account created with a weak password

```sudo useradd -m ubuntu-prod && echo 'ubuntu-prod:<weak-password>' | sudo chpasswd```

* **Attacker:** Used Hydra to perform brute force to log in.

```hydra -l ubuntu-prod -P /usr/share/wordlists/rockyou.txt ssh://VICTIM_IP -t 4```

#### Results:

The first 3 attempts were logged, demonstrating the failed authentication. The last attempt by Hydra on the attacker's machine was successful in retrieving the weak password
<img width="1436" height="850" alt="image" src="https://github.com/user-attachments/assets/6444c63e-16e8-4620-9f6c-8dda86963d59" />


