---
layout: post
title: SOC Automation Project
date: 2024-03-25 14:15 -0400
categories: [Labs]
tags: [Cybersecurity, SIEM, SOAR]
image:
    path: /assets/img/SOCLab.png
    alt: 
---
<link href="../../assets/css/terminal_styles.css" rel="stylesheet">

Build a SOC lab that ingests logs in a SIEM, automates workflows in a SOAR, and documents with Case Management. to enhance incident response, creates tickets using TheHive Case Management tool, and threat detection capabilities.

---

## Introduction

In this lab, we will create an automated Security Operations Center (SOC) environment. We will explore how automation enhances incident response, accelerates threat detection, and streamlines SOC workflows.

![Workflow Diagram for our Security Operations Lab](/assets/img/soc-automation-diagram.png)
_**Diagram:** Data flow through our Environment_

---

## Understanding the Concepts

### Wazuh
Wazuh is a free and open-source security platform that functions as a `SIEM` (security information and event management) and `XDR` (extended detection and response) platform for endpoint devices.

#### Wazuh Agent
The Wazuh agent installed on our Windows 10 Client endpoint serves as our first line of defense by continuously monitoring the system for security events and log data. The agent encrypts and forwards this data securely to our central management server, the Wazuh Manager.

#### Wazuh Manager
Deployed in the cloud environment, the Wazuh Manager acts as a centralized hub for receiving, processing, and analyzing security events collected by the Wazuh agents. It analyzes the incoming data in realtime, using predefined rulesets and anomaly detection algorithms to identify potential security incidents. Upon detection, the Wazuh Manager triggers alerts and performs predefined responsive actions to mitigate threats promptly.

![Our consolidated logs shown on the Wazuh Dashboard](/assets/img/soc-automation-wazuh.png)
_**Source:** [Wazuh Documentation](https://documentation.wazuh.com/current/release-notes/release-4-3-0.html)_

### Shuffle
<p style="margin-bottom:0">Shuffle serves as our Security Orchestration, Automation, and Response (SOAR) platform, and plays a crucial role in automating incident response processes. Upon receiving alerts from the Wazuh Manager, Shuffle orchestrates various tasks:</p>

- **Enrichment with OSINT**: Shuffle uses open-source intelligence (OSINT) to enhance its understanding of detected indicators of compromise (IOCs).
- **Case Management with TheHive**: Shuffle integrates with TheHive to automatically create cases for each detected security incident. This allows for structured investigation, coordination, and documentation of response efforts.
- **Alert Notifications via Email**: Shuffle notifies the SOC Analysts about newly generated alerts. This ensures timely awareness and responsive actions.

### TheHive
Operating as a cloud-based incident response and case management tool, TheHive streamlines the management of security incidents. It provides a centralized platform for SOC teams to collaborate, analyze, and respond to threats effectively.

![Our case management tool, TheHive, is used to track and document incidents](/assets/img/soc-automation-thehive.png)
_**Source:** [TheHive](https://github.com/TheHive-Project/TheHive/blob/main/images/Current_cases.png)_

### Response Workflow
Upon receiving notification emails, SOC analysts interact with Shuffle to execute response actions. These actions encompass **performing remediation steps**, **communicating with stakeholders**, and these alerts as a **feedback loop** to use for continuously improving our environment.

![An image of our project's workflow: our Windows VM's logs go to Wazuh, which uses Shuffle to create alerts in TheHive and gather online Virus data, and then emails to the SOC analyst](/assets/img/soc-automation-workflow.png)
_**Diagram:** SOC Automation Project Workflow_

---

## Setting up the Environment

<p style="margin-bottom:0">This lab makes use of three devices:</p>
- Windows 10 client with Sysmon
- Wazuh Server (in the cloud)
- TheHive Server (in the cloud)

### Windows 10 Client with Sysmon

<p style="margin-bottom:0">Create the Windows 10 VM using a <a href="https://www.microsoft.com/en-us/software-download/windows10">Windows 10 ISO</a>.</p>
- Download [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- Download [sysmonconfig.xml](https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml) from the sysmon-modular Github repository.

<p style="margin-bottom:0">Setting up Sysmon:</p>
- Extract Sysmon.zip and move sysmonconfig.xml into the folder.
- Run PowerShell as Administrator, navigate to the Sysmon folder, and use the command:<br/>`.\Sysmon64.exe -i .\sysmonconfig.xml`

![In Powershell, use 'cd ~\Sysmon\' and '.\Sysmon64.exe -i .\sysmonconfig.xml'](/assets/img/soc-automation-sysmon-download.png)

<p style="margin-bottom:0">You can verify that Sysmon was installed successfully with:</p>
- **Services.msc**: Confirm the presence of Sysmon64
- **Event Manager**: Application and Services Logs > Microsoft > Windows > Sysmon

![In Event Viewer, you should now have a Sysmon folder under Application and Services logs](/assets/img/soc-automation-sysmon-verify.png)

---

### Wazuh Server
You can choose to either set this up in the cloud (recommended), or in a virtual machine (manual, but you won't need to sign up for an account). 

If you want to set it up in a VM, you can use Ubuntu 22.04 and follow the [Wazuh Quickstart Guide](https://documentation.wazuh.com/current/quickstart.html).

#### In the Cloud (Digital Ocean): 
<p style="margin-bottom:0">Create a new Droplet (cloud server)</p>
- OS: Ubuntu 22.04 (LTS), Basic CPU, 8 GB RAM
- Set a password, and change your Hostname to 'Wazuh'

We will be accessing this VM using SSH over the internet, so let's configure a firewall to mitigate potential security threats.

<p style="margin-bottom:0">Create a Firewall:</p>
- Navigate to: Networking > Firewalls
- Create Inbound rules to limit Incoming TCP and UDP traffic to your IP address.

![Setting these firewall rules should help prevent outsiders from connecting](/assets/img/soc-automation-wazuh-firewall.png)

<p style="margin-bottom:0">Attach the Firewall:</p>
- Navigate to Droplets > Wazuh > Networking > Firewalls, and click Edit.
- Select the Firewall we created: Droplets > Add Droplets > "Wazuh".

<p style="margin-bottom:0">SSH into the VM:</p>
- Update packages:
    - `apt-get update && apt-get upgrade -y`
- Install Wazuh with the curl command located on the [Wazuh Quickstart Guide](https://documentation.wazuh.com/current/quickstart.html).
    - `curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a`

![A terminal showing the installation completed](/assets/img/soc-automation-wazuh-install-finish.png)

We can now access the Wazuh Dashboard at https\://\[Wazuh-Droplet-IP\]/ and use the credentials to sign in.

![The sign in page is now accessible by entering our Wazuh's IP address in a browser](/assets/img/soc-automation-wazuh-web-app.png)

---

### TheHive Server
You can choose to either set this up in the cloud or on a virtual machine as well.

If you want to set up TheHive in a VM, use Ubuntu 22.04 and follow this [guide](https://github.com/MyDFIR/SOC-Automation-Project/blob/main/TheHive-Install-Instructions) to download TheHive & prerequisites.

#### In the Cloud (Digital Ocean): 
<p style="margin-bottom:0">Create a new Droplet (cloud server)</p>
- OS: Ubuntu 22.04 (LTS), Basic CPU, 8 GB RAM
- Set a password, and change your Hostname to 'TheHive'

<p style="margin-bottom:0">Attach the Firewall:</p>
- Navigate to Droplets > TheHive > Networking > Firewalls, and click Edit.
- Select the Firewall we created: Droplets > Add Droplets > "TheHive".

<p style="margin-bottom:0">SSH into the VM:</p>
- Follow this [guide](https://github.com/MyDFIR/SOC-Automation-Project/blob/main/TheHive-Install-Instructions) for the commands to use to download TheHive and its prerequisites. (Java, Cassandra, ElasticSearch, TheHive)

---

## Installing and Configuring Files
#### Configure TheHive Dependencies
<p style="margin-bottom:0">Editing Cassandra config files:</p>
- nano /etc/cassandra/cassandra.yaml
	- Change `cluster_name` (optional)
	- Change `listen_address:`, `rpc_address:`, and `seed_provider: seeds:` to TheHive droplet's public IP
- systemctl stop cassandra.service
- rm -rf /var/lib/cassandra/*
- systemctl start cassandra.service

<p style="margin-bottom:0">Editing ElasticSearch config files:</p>
- nano /etc/elasticsearch/elasticsearch.yml
	- Uncomment and edit the following:
		- `cluster.name: thehive` 
		- `node.name: node-1`
		- `network.host: [TheHive droplet's public IP]
		- `http.port: 9200`
		- `cluster.initial_master_nodes: ["node-1"]
- systemctl start elasticsearch
- systemctl enable elasticsearch

<p style="margin-bottom:0">Limit ElasticSearch RAM usage (so it doesn't crash!):</p>
- nano /etc/elasticsearch/jvm.options.d/jvm.options
- Paste the following:

<div class="language-console">
    <div class="highlight">
        <pre class="highlight"><code>-Dlog4j2.formatMsgNoLookups=true<br/>-Xms2g<br/>-Xmx2g</code></pre>
    </div>
</div>

### Configure TheHive
<p style="margin-bottom:0">Set the owner of <code class="language-plaintext highlighter-rouge">/opt/thp</code> to <code class="language-plaintext highlighter-rouge">thehive</code></p>
- chown -R thehive:thehive /opt/thp
- *Check using `ls -la /opt/thp`*
<p style="margin-bottom:0">nano /etc/thehive/application.conf</p>
- Set `hostname = ["TheHive droplet's public IP"]` for all hostname variables.
- Set `cluster-name` to the same as `cluster_name` in Cassandra earlier.
- Change `application.baseURL = "http://[TheHive droplet's public IP]:9000"`
<p style="margin-bottom:0">Start TheHive:</p>
- systemctl start thehive
- systemctl enable thehive

At this point, all 3 services should be running, which can be checked using `systemctl status [service]`.

You can also now access the TheHive dashboard by navigating to `[TheHive droplet's public IP]:9000` and signing in using the credentials: `admin@thehive.local:secret`

### Add a Wazuh Agent
<p style="margin-bottom:0">Within the Windows 10 VM, sign in to the Wazuh dashboard at https://&lt;Wazuh-Droplet-IP&gt;/</p>
- If you forgot the credentials, you can go onto the Wazuh server, and do the following to unzip the wazuh-install-files.tar and view your passwords:
	- `tar -xvf wazuh-install-files.tar`
	- `cat wazuh-install-files/wazuh-passwords.txt`

A message at the top of the page says "No agents were added to this manager. Add agent", click on it to go to the page to deploy a new Wazuh agent.

Select the Windows Package, set the Server address to the Wazuh droplet IP address, set an agent name, and copy the given command into a PowerShell window with Administrator privileges.

![You can download the Wazuh agent using a curl command in the Wazuh Quickstart Guide](/assets/img/soc-automation-wazuh-agent-install.png)

After a little bit of time, refreshing the Wazuh dashboard will show that our agent was installed successfully!

![Our Wazuh dashboard will now show 1 new active agent](/assets/img/soc-automation-agent-added.png)

---

### Configure Wazuh to ingest Sysmon data

Wazuh acts a bit differently than other SIEMs - only logs that trigger a rule will appear on the Dashboard, rather than all logs being forwarded.

We will have to configure our Wazuh agent so that specific data, in our case Sysmon, will be ingested.<br/>
To set this up, [follow these instructions](https://www.youtube.com/watch?v=amTtlN3uvFU).

### Create Rules for Wazuh
On the Wazuh dashboard, under Management > Administration > Rules, we can create rules that will trigger when a specific event occurs. 

<p style="margin-bottom:0">Using a rule from <code class="language-plaintext highlighter-rouge">0800-sysmon_id_1.xml</code> as a template:</p>
- Select Custom rules > edit `local_rules.xml`
- Add a new rule that will detect if an executed file's original name was Mimikatz.exe:

![Our custom rule will alert if an executed files original filename is mimikatz.exe](/assets/img/soc-automaton-custom-rule.png)

Now, if we run Mimikatz on our system, a security alert will be created on our dashboard.

![A new security alert appeared once we ran Mimikatz.exe](/assets/img/soc-automation-security-alert.png)

---

## Automation through Shuffle
Create a Shuffle account at [shuffler.io](https://shuffler.io/), go to the Workflows page, and create a new workflow called "SOC Automation Project".

Select 'Triggers', and drag the 'Webhook' workflow starter into the project. 

![In Shuffle, you can drag workflow starters into the project and connect them together](/assets/img/soc-automation-shuffle-webhook.png)

Give it a name such as 'Wazuh-Alerts' and copy the Webhook URI for later.
<p style="margin-bottom:0">Select the 'Change Me' icon and set the following:</p>
- Find Actions: Repeat back to me
- Call: $exec

<p style="margin-bottom:0">We will need to SSH into the Wazuh droplet, and add our Webhook URI to Wazuh:</p>
- nano /var/ossec/etc/ossec.conf
- Add the following to the ossec.conf file:

![The .conf file will now contain information about our rule](/assets/img/soc-automation-shuffle-added.png)

After the change we must restart the services: `systemctl restart wazuh-manager.service`

Back on Shuffle, select the Wazuh-Alerts Webhook, and press Start. Now running Mimikatz on our Windows VM should make a new webhook appear.

![In the Show Executions tab, we should now see a new run with today's date and time and data from our Webhook](/assets/img/soc-automation-shuffle-webhook_2.png)

### Creating a Workflow
<p style="margin-bottom:0">Let's create an example workflow to show off the power of Shuffle:</p>
1. A Mimikatz alert is sent to Shuffle
2. Shuffle receives the Mimikatz Alert and extracts the SHA256 Hash of the executed file.
3. Check the file's reputation score with VirusTotal
4. Send details to TheHive to create an alert
5. Send an email to the SOC Analyst to begin an investigation

Shuffle is currently sending back the entire log through the webhook, we can have Shuffle parse out just the SHA256 hash within the log (`"hashes": "SHA1=D34DB33F,MD5=ABCD,SHA256=12345..."`). 

<p style="margin-bottom:0">We can do this by editing the 'Change Me' object:</p>
- Name: `Get_SHA256_Hash`
- Find Actions: `Regex capture group`
- Input data: `$exec.text.win.eventdata.hashes`
- Regex: `SHA256=([a-fA-F0-9]{64})`

Save the workflow, then select 'Show Executions' at the bottom of the screen to rerun the workflow and view if our changes were successful.

![The workflow information shows the SHA256 hash was successfully pulled](/assets/img/soc-automation-shuffle-regex.png)

### Adding VirusTotal to Shuffle
To use VirusTotal's API, register an account on [VirusTotal](https://www.virustotal.com/gui/home/upload), select the menu, and copy your API Key.

In Shuffle, add the Virustotal Active App to the project:

![Once the Virustotal app is in our workflow, we can connect it to our Get_SHA256_Hash app](/assets/img/soc-automation-shuffle-virustotal.png)

<p style="margin-bottom:0">Configure the VirusTotal app in Shuffle with the following settings:</p>
- Name: `VirusTotal`
- Find Actions: `Get a hash report`
- Parameters:
	- Apikey: `[YOUR-API-KEY]`
	- Url: `https://www.virustotal.com`
	- Hash: `$get_sha256_hash.group_0.#`

![These configuration settings will allow us to pull exactly the data we need](/assets/img/soc-automation-virustotal-configs.png)

Save the changes, press 'Show executions', select the workflow runs, and press 'Rerun workflows'. The reran workflow should say **Status** SUCCESS > "Results for Virustotal".

![We should now able to view information about the successful VirusTotal workflow](/assets/img/soc-automation-virustotal-success.png)

### Adding TheHive to Shuffle
In Shuffle, search and add TheHive to our project.

![Search "TheHive" into the searchbar in Shuffle](/assets/img/soc-automation-shuffle-thehive.png)

Log in to TheHive dashboard at `http://[TheHive-Droplet-IP]:9000/`<br/>
Create a new Organization for our project.

![We can click the plus button and fill out the fields to create a new org in TheHive](/assets/img/soc-automation-thehive-new-org.png)

Click on the new organization that we created and create new users:<br/>
<p style="margin-bottom:0">User 1:</p>
- Type: `Normal`
- Login: `austin@test.com` (use your name)
- Name: `Austin`
- Profile: `analyst`
- **Give this account a password: Select 'Preview' > 'Set a new password'.**

<p style="margin-bottom:0">User 2:</p>
- Type: `Service`
- Login: `shuffle@test.com`
- Name: `SOAR`
- Profile: `analyst`

Generate an API key for this account: Select 'Preview' > API key.

<p style="margin-bottom:0">Now go back onto Shuffle and select TheHive:</p>
- Press the + button beside Authentication:
	- Add the API key from our Service account, and the URL of our TheHive server. (`http://[TheHive-Droplet-IP]:9000/`)
- Find Actions: `Create Alert`

Now let's give some details to this Alert. First, add a line connecting VirusTotal to TheHive in the workflow, allowing us to pass through details.

<p style="margin-bottom:0">Click the checkmark next to Hide Body, and fill in the following information:</p>
 - Title: `$exec.title`
 - Tags: `["T1003"]`
 - Summary: `Mimikatz activity detection on host: $exec.text.win.system.computer and the process ID is: $exec.text.win.system.processID and the command line is: $exec.text.win.eventdata.commandLine`
 - Severity: `2`
 - Type: `Internal`
 - Tlp: `2`
 - Status: `New`
 - Sourceref: `*Rule: 100002*`
- Source: `Wazuh`
- Pap: `2`
- Flag: `false`
- Description: `Mimikatz Detected on host: $exec.text.win.system.computer from user: $exec.text.win.eventdata.user`
- Date: `$exec.text.win.eventdata.utcTime`

In Digital Ocean, we need to allow all incoming traffic to Port 9000<br/>
<p style="margin-bottom:0">In Digital Ocean > Networking > Firewalls > create a new Inbound Rule:</p>
- Custom, TCP, 9000, All IPv4

Now within Shuffle, select 'Show Executions', and rerun the workflow. There should be a new alert created in TheHive:

![We should now able to view information about the successful TheHive workflow](/assets/img/soc-automation-shuffle-thehive-success.png)

![And an alert in its dashboard showing that Mimikatz was detected on our Windows VM!](/assets/img/soc-automation-thehive-new-alert.png)

### Adding Email to Shuffle
In Shuffle, search for and add the "Email" Active App. Add a line connecting VirusTotal to the Email in the workflow.

<p style="margin-bottom:0">Setting up the email app is relatively straight-forward:</p>
- Find Actions: `Send email shuffle`
- Recipients: `[Email-Address]`
- Subject: `[Alert] Mimikatz Detected!`
- Body: 

<div class="language-console">
    <div class="highlight">
        <pre class="highlight"><code>Time: $exec.text.win.eventdata.utcTime<br/>Title: $exec.title<br/>Host: $exec.text.win.system.computer<br/>PID: $exec.text.win.system.processID<br/>Command Line: $exec.text.win.eventdata.commandLine</code></pre>
    </div>
</div>

If you are running into errors, you can change the Find Action to `Send email smtp` and set up a temporary email address to use as the sender of these emails.

![We should now also recieve emails when an alert occurs in Wazuh or when rerunning a workflow](/assets/img/soc-automation-email.png)

## Conclusion

Security Information and Event Management Systems (SIEMs) are very powerful tools for detecting threats on a network and providing constant monitoring and visibility of what is occurring within your network. The ability to ingest and normalize large quantities and varieties of logs. 

Gaining proficiency by getting hands-on experience with tools like `SIEMs` (Splunk, Wazuh), `SOARs` (Shuffle), and `Case Management` (TheHive) can only work to help you a become a effective security practitioner. 

Shoutout to MyDFIR on YouTube for the original guide to this lab!<br/>
I highly suggest his video series: [Intro](https://www.youtube.com/watch?v=Lb_ukgtYK_U), [Part 1](https://www.youtube.com/watch?v=XR3eamn8ydQ), [Part 2](https://www.youtube.com/watch?v=YxpUx0czgx4), [Part 3](https://www.youtube.com/watch?v=VuSKMPRXN1M), [Part 4](https://www.youtube.com/watch?v=amTtlN3uvFU), [Part 5](https://www.youtube.com/watch?v=GNXK00QapjQ). This homelab helped me get a better understanding of how these security tools can work together to create efficient workflows and create SIEM rules to automatically detect and alert when an attack is made on our network.

