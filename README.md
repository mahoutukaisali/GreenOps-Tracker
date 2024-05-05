# GreenOps Tracker

## Overview
**GreenOps Tracker** is a sophisticated real-time monitoring system designed to enhance server efficiency and reduce CO2 emissions by optimizing the use of computing resources. This system utilizes Performance Co-Pilot (PCP) and Ansible Event-Driven on Linux servers to monitor and respond to server performance metrics in real time.

## Business Driver
The initiative behind **GreenOps Tracker** is to empower server administrators with precise, real-time data to make informed decisions about resource allocation, thereby minimizing unnecessary power consumption and reducing the overall carbon footprint of data centers.

## Why It Matters
As digital infrastructures expand, the energy consumption of data centers has become a critical issue. **GreenOps Tracker** addresses this by providing tools that not only help in reducing operational costs but also support environmental sustainability goals.

## Solution
**GreenOps Tracker** incorporates a dynamic system architecture featuring:
1. **Linux Servers with PCP**: These servers are configured to monitor various performance metrics. Upon detecting specific conditions, they trigger and send webhooks.
2. **Ansible Event-Driven Server**: This server receives webhooks from the Linux servers. In response, it automatically launches Ansible playbooks.
3. **Dynamic Data Collection**: The triggered playbooks are designed to collect detailed performance data such as CPU usage, memory usage, and filesystem status. This ensures that server administrators have access to the most relevant and current data.

## Challenges Overcome
Developing **GreenOps Tracker** involved navigating several significant challenges:
- **Real-time Data Handling**: Ensuring the system could handle and process real-time data efficiently without delays or errors.
- **Integration of Diverse Technologies**: Seamlessly integrating PCP with Ansible and ensuring reliable communication between various components.
- **User-Centric Design**: Creating a system that is easy for server administrators to use and interpret in high-pressure environments.

## Future Extensions
To further enhance **GreenOps Tracker**, potential future features might include:
- **Enhanced Predictive Capabilities**: Using AI to predict potential issues before they occur based on historical data trends.
- **Expanded Metric Support**: Including additional metrics such as network performance and advanced thermal metrics.
- **Greater Automation**: Developing more sophisticated response mechanisms that automatically adjust configurations based on the data received.

## Conclusion
**GreenOps Tracker** is not just a monitoring tool; it is a strategic asset for sustainable IT management. By providing real-time insights and automated responses, it plays a pivotal role in optimizing server performance and advancing environmental sustainability in the tech industry.

# Installation
## Prerequisites
- This project needs 3 Linux servers assuming CentOS/Fedora/Red Hat Enterprise Linux as an OS type. This project has been tested on Fedora servers.
- An OS user called `ansible` exists on all Fedora nodes and this user can become `sudo` user.
- The below steps will be operated by `ansible` user.
- `ansible` user can ssh key login to client servers from the ansible server.
- Make sure you have `Cisco Webex` account and a room that can receive notifications from the Ansible server.
  
### Step 1: Install Ansible and Ansible Event-Driven
The installation process follows the official document:
https://ansible.readthedocs.io/projects/rulebook/en/stable/installation.html

Run the below commands on one of the fedora node:
```
$ sudo dnf --assumeyes install java-17-openjdk python3-pip
$ export JAVA_HOME=/usr/lib/jvm/jre-17-openjdk
$ pip3 install ansible ansible-rulebook ansible-runner
```
Install ansible.eda collection:
```
$ ansible-galaxy collection install ansible.eda
```
### Step 2: Clone this "GreenOps Tracker" repository on the Ansible node
```
$ git clone
```

### Step 3: Modify the cloned source code to suit your environments
Open `inventory.yml` file and edit IP addresses and `sudo password`. Below is the current file contents. You can replace `192.168.57.10/192.168.57.11` with your pcp server's IP and replace `ansible_become_pass: changeme` with your `sudo password`.
```
---
all:
  children:
    pcp_servers:
      hosts:
        192.168.57.10:
          ansible_become_pass: changeme
        192.168.57.11:
          ansible_become_pass: changeme
```

Next, open `pre_setup.yml` file and replace `192.168.57.3` part with your ansible server's IP address.
```
---
- hosts: pcp_servers
  become: true
  gather_facts: no
  vars:
    webhook_endpoint: "http://192.168.57.3:5000/endpoint"
```

Next, open `playbooks/vars/webex.yml` file and replace `changeme` with the actual values. `room_id` is your Cisco Webex room ID, `token` is your Cisco Webex account's token.
```
---
room_id: changeme
token: changeme
```
For further information about Cisco Webex, please refer to the official documentation: https://developer.webex.com/docs

Now you can launch `pre_setup.yml` playbook on the Ansible server. This will install all the necessary packages on pcp client servers to try this project.
```
$ ansible-playbook pre_setup.yml
```

# Launch GreenOps Tracker project
Now you can try `GreenOps Tracker` project. 

### Step 1: Run "ansible-rulebook" on the Ansible server. This will listen to 0.0.0.0:5000 on the Ansible server and wait to receive the payloads from pcp servers.
```
$  ansible-rulebook --rulebook pcpEventRules.yml -i inventory.yml --verbose
```

### Step 2: Put a load on the pcp server.　
We have already installed `stress-ng` package in pcp servers using `pre_setup.yml` playbook at the `Installation` step, so in this example, we can use `stress-ng` to put a load on the pcp server.
example command:
```
$ stress-ng --cpu $(nproc) --vm 2 --fork 8 --switch 4 --timeout 120s -v
```
**Note: If the above command does not sufficiently load the server, try increasing the numbers to match the specifications of your server.**

### Step 3: Monitoring logs from Ansible Event-Driven. 
