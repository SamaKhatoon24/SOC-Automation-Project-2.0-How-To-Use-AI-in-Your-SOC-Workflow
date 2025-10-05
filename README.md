

# SOC-Automation-Project-2.0-How-To-Use-AI-in-Your-SOC-Workflow

<img width="1045" height="484" alt="Screenshot 2025-10-04 213747" src="https://github.com/user-attachments/assets/0a3d9502-b625-4f85-8c1f-8ad2d70d5d9a" />

## 🌍 Introduction

I wanted to create something beyond a regular SOC lab — something that reflects how cybersecurity is evolving with **AI integration**.

That’s how the idea for my **AI-Augmented SOC Project** started.

The concept is simple but powerful:

use AI (ChatGPT) to **analyze and summarize alerts from Splunk**, and automatically send that analysis to **Slack**, where a SOC analyst can see context instantly.

It’s a blend of traditional security monitoring and modern AI assistance — showing how AI can accelerate threat response and reduce analyst fatigue.

*(Note: This setup is for learning and demonstration only — in real-world environments, sharing logs with public AI APIs can raise data privacy issues.)*

---

## 🎯 Objective

My main goals for this project were to:

* Build a **local virtual environment** to simulate a SOC setup.
* Configure **Splunk** as the SIEM to collect and monitor event logs.
* Use **Windows 10** as the endpoint generating telemetry.
* Automate workflows in **N8N**, integrating **Splunk → ChatGPT → Slack**.
* Demonstrate how AI can help SOC analysts by automatically enriching and summarizing incidents.

---

## 🧩 Tools and Components

Here’s the complete ecosystem I used:

| Component                   | Purpose                                             |
| --------------------------- | --------------------------------------------------- |
| **VMware Workstation Pro**  | Virtualization platform for creating all VMs.       |
| **Windows 10**              | Log generation and endpoint telemetry.              |
| **Splunk (Ubuntu)**         | SIEM solution for log collection and monitoring.    |
| **N8N (Ubuntu)**            | Workflow automation and AI integration.             |
| **Slack**                   | Notification and alerting platform.                 |
| **ChatGPT API**             | Provides context enrichment and automated analysis. |
| **Kali Linux** *(optional)* | Used for attack simulation and testing.             |
| **Iris** *(optional)*       | Case management system for SOC workflows.           |

---

## 🧱 Setting Up the Virtual Environment

Before configuring any of the tools, I first built a **controlled lab environment** using VMware Workstation.

### 🔹 Installing VMware

I created an account on **Broadcom’s website** (since VMware now requires it), downloaded **VMware Workstation Pro**, and installed it on my main system.

Once ready, I planned out my virtual machines as follows:

* Windows 10 → Endpoint system
* Ubuntu (Splunk) → SIEM
* Ubuntu (N8N) → Automation and AI logic
* Kali Linux → Attack simulation (optional)
* Iris → Case management (optional)

This gave me a small, isolated, but realistic SOC ecosystem.

---

<img width="2537" height="1308" alt="Screenshot 2025-10-04 192744" src="https://github.com/user-attachments/assets/4c05a094-7860-4132-9716-29b9c519f3ca" />

## 🪟 Setting Up Windows 10 VM

To begin, I downloaded the **Windows 10 ISO** from Microsoft’s official site.

In VMware, I created a new VM named **Sama-Windows10** with:

* 2 processors
* 4 GB RAM
* 60 GB disk space

After installation, I:

* Enabled **Remote Desktop (RDP)** → `Settings → Remote Desktop → Turn On`.
* Opened Command Prompt and ran `ipconfig` to note the IP address (e.g., `192.168.136.141`).
* Connected via **Remote Desktop** from my host system — much easier to work with than inside VMware.

Then I took a **snapshot** named *“Base VM”* in case I needed to revert later.

---

## 🧱 Setting Up Splunk Server (Ubuntu)

Next, I set up the **Splunk Server** on Ubuntu.

I created a new VM called **Sama-Splunk**, assigned it **8 GB RAM** and **100 GB disk space**, and used **Ubuntu Server 24.04** as the OS.

### 🔹 During installation:

* Selected English language and default configurations.
* Chose “Use entire disk” for partitioning.
* Installed **OpenSSH Server** (checked that option during setup).

Once installed, I logged in and ran:

```bash
sudo apt-get update && sudo apt-get upgrade -y

```

This ensured everything was up-to-date.

To find the server’s IP address, I ran:

```bash
ip a

```

and got something like `192.168.136.138`.

From my host system, I connected to it using PowerShell:

```bash
ssh Sama@192.168.136.138

```

That allowed me to manage the Splunk server easily from my laptop instead of typing in VMware’s console.

---

## ⚙️ Installing Splunk Enterprise

I logged into my Splunk account, navigated to **Free Trials → Splunk Enterprise**, and selected **Linux (DEB)** as the download option.

VPNs must be off since Splunk blocks them during download.

I copied the `.deb` link and ran the following in my Ubuntu terminal:

```bash
wget <splunk_download_link>

```

After verifying the file with `ls`, I installed it:

```bash
sudo dpkg -i splunk*.deb

```

Then I moved to the Splunk directory:

```bash
cd /opt/splunk

```

and started Splunk for the first time:

```bash
sudo ./splunk start

```

It prompted for a license agreement — I accepted, then set my **username and password**.

Once it finished, Splunk was up and running on port **8000**.

---

## 🧩 Setting Up N8N Server

For automation, I created another Ubuntu 24.04 VM called **Sama-N8N** with:

* 4 GB RAM
* 50 GB disk space

Just like before, I installed **OpenSSH**, ran updates, and noted its IP (e.g., `192.168.136.139`).

Later, this VM will handle automation:

* Receiving alerts from Splunk.
* Sending data to ChatGPT API.
* Forwarding enriched summaries to Slack.

---

## 🧠 Connecting All Systems

At this point, I had all my systems running:

| VM             | IP Address      | Purpose                  |
| -------------- | --------------- | ------------------------ |
| Sama-Windows10 | 192.168.136.141 | Endpoint generating logs |
| Sama-Splunk    | 192.168.136.138 | SIEM                     |
| Sama-N8N       | 192.168.136.139 | Workflow automation      |
| Kali Linux     | (optional)      | Attack simulation        |
| Defer Iris     | (optional)      | Case management          |

All VMs were configured under VMware’s **NAT network**, so they could talk to each other without exposing anything externally.

---

## ⚠️ Important Note on AI Integration

While integrating ChatGPT into the workflow, I made sure to remember:

AI models like ChatGPT are **public APIs**, so sending raw logs might expose sensitive data.

For this reason, my project uses **synthetic logs** and **anonymized data** purely for demonstration.

In real-world setups, an **on-prem LLM** or **private API** would be the correct and secure approach.

# 🖥️ Configuring Splunk and Windows 10 Telemetry + N8N Integration

---

## Splunk GUI Setup

1. **Admin Username and Password**

   * Set your admin username (e.g., `Sama`).
   * Set a password of your choice.
2. **Enable Splunk to Start on Reboot**

   ```bash
   cd bin
   sudo ./splunk enable boot-start -user splunk

   ```

   * This ensures Splunk automatically starts whenever the VM reboots.
3. **Access Splunk Web GUI**

   * Open browser on your host machine: `http://192.168.136.138:8000`

   * Log in using the admin username and password created earlier.

<img width="438" height="343" alt="Screenshot 2025-10-04 211356" src="https://github.com/user-attachments/assets/7c28608d-e8e2-40b4-aa89-0eacb105ef35" />

---

## Splunk Configuration

1. **Configure Receiving Port**

   * Go to: `Settings → Forwarding and Receiving → Configure Receiving → New Receiving Port`
   * Set port: `9997` → Save
2. **Create an Index**

   * Go to: `Settings → Indexes → New Index`
   * Index name: `myDFIRproject` → Save
3. **Install Windows Event Add-On**

   * Go to: `Apps → Find More Apps → Search "Splunk Add-on for Microsoft Windows"`
   * Install using your Splunk account credentials (not the internal Splunk user).
4. **Take Snapshot**

   * In VMware: `VM → Snapshot → Take Snapshot`
   * Name: `Splunk-Installed`

---

## Sending Windows 10 Telemetry to Splunk

1. **Verify Connection**

   * Open Command Prompt on Windows VM:

     ```bash
     ping 192.168.136.138

     ```

   * Ensure successful ping to Splunk VM.
2. **Download & Install Splunk Universal Forwarder**

   * Download from: `splunk.com → Free Trials → Universal Forwarder → 64-bit Windows 10`
   * Install with the following configurations:

     * Username: `Sama`
     * Password: your choice
     * Deployment server: None
     * Receiving indexer: `192.168.136.138:9997`
3. **Configure Inputs**

   * Path: `C:\Program Files\SplunkUniversalForwarder\etc\system\local`
   * Create `inputs.conf` file (can download pre-configured from free resources).
4. **Restart Splunk Forwarder Service**

   * Go to `Services → SplunkForwarder → Log On → Local System Account → Apply → Restart`
5. **Verify Logs in Splunk**

   * In Splunk GUI: `Apps → Search and Reporting`

   * Query:

     ```
     index="myDFIRproject" | stats count by time, computer_name, user, src_ip

     ```

   * You should see logs from your Windows 10 VM (Security, Application, System, RDP).

---

<img width="927" height="590" alt="Screenshot 2025-10-04 211610" src="https://github.com/user-attachments/assets/0ba644ee-ef40-4ff7-aad1-f42dc6268933" />

## Generating Test Events

1. **Manual Failed Logins**

   * Open Remote Desktop to Windows VM and enter incorrect password multiple times.
   * Check Splunk for new events (`event_code=4625`). <img width="425" height="409" alt="Screenshot 2025-10-04 211910" src="https://github.com/user-attachments/assets/d2b9a5d1-cbb1-4287-9104-35f3ec23903d" />

2. **Verify Field Names**

   * Event ID field is `event_code` (case-sensitive).
   * Successful login: `4624`
   * Failed login: `4625`

3. **Optional**: Use Kali Linux for advanced telemetry simulations.

   * Not required for basic testing, but useful for more advanced attack simulations.

---

## N8N Installation on Ubuntu VM

1. **SSH into N8N VM**

   ```bash
   ssh Sama@192.168.136.139

   ```

2. **Install Docker & Docker-Compose**

   ```bash
   sudo apt install docker.io
   sudo apt install docker-compose

   ```

3. **Create N8N Directory & Docker Compose File**

   ```bash
   mkdir n8n-compose
   cd n8n-compose
   sudo nano docker-compose.yaml

   ```

   Example YAML:

   ```yaml
   services:
     n8n:
       image: n8nio/n8n:latest
       restart: always
       ports:
         - "5678:5678"
       environment:
         - N8N_HOST=192.168.136.139
         - N8N_PORT=5678
         - N8N_PROTOCOL=http
       volumes:
         - ./n8n-data:/home/node/.n8n

   ```

4. **Fix Permissions**

   ```bash
   sudo chown -R 1000:1000 n8n-data
   sudo docker-compose down
   sudo docker-compose up -d

   ```

5. **Access N8N Web GUI**

   * Browser: `http://192.168.136.139:5678`
   * Create a test account (email and password can be fake).

6. **Take Snapshot in VMware**

   * Name: `N8N-Installed`

     <img width="797" height="268" alt="Screenshot 2025-10-04 193545" src="https://github.com/user-attachments/assets/1195b5ca-e772-483b-95a8-fe4bc862ac44" />

---

## ⚙️ Setting Up the Splunk Alert

1. **Create Splunk Alert**

   * Example: Brute-force login attempt (`event_code=4625`)

   * Query:

     ```
     index="myDFIRproject" event_code=4625 | stats count by time, computer_name, user, src_ip

     ```

   * Save as alert: `test-brute-force`

I started by creating a new alert in **Splunk** called `test-brute-force`.

For the **description**, I didn’t overthink it — I just left a placeholder.

To test it frequently, I configured it to run on a **cron schedule** with all asterisks (`* * * * *`) so it triggers **every minute** — just for testing.

<img width="659" height="404" alt="Screenshot 2025-10-04 195341" src="https://github.com/user-attachments/assets/2021b348-99bf-409d-816c-9bbb10907ac1" />

Then under **Trigger Actions**, I added two things:

* **Add to Trigger Alerts**
* **Webhook**

The webhook will send my alert data to **N8N**, so I can automate the response flow from there.

---

<img width="1598" height="902" alt="Screenshot 2025-10-04 143807" src="https://github.com/user-attachments/assets/6c7a3f24-b6de-4c29-8851-73332a4bc0b1" />

## 🧩 Connecting Splunk to N8N via Webhook

In **N8N**, I started a new workflow “from scratch.”

* Added the first node: **Webhook**
* Changed the HTTP method from `GET` to `POST`
* Copied the **Webhook URL**

Then I went back to **Splunk** and pasted that webhook URL there.

Once saved, I enabled the alert and went back to N8N to **listen for test events**.

After waiting a minute, N8N caught the alert 🎉

It showed me the alert data including the **time, computer name, user, source IP, and count.**

That confirmed my connection between Splunk and N8N was working!

---<img width="842" height="537" alt="Screenshot 2025-10-04 213005" src="https://github.com/user-attachments/assets/c26ca3c5-d529-435a-bab1-ce67e23826ed" />

## 🧠 Integrating ChatGPT (OpenAI) into N8N

Next, I started building the automation in N8N by adding an **OpenAI node**.

But to use that, I first needed an **API Key** from OpenAI.

Here’s what I did:

* Went to [openai.com](https://openai.com/)
* Logged into the **API platform**
* Clicked on **Start Building**
* Created my organization named **Sama-Project**
* Generated my API key and copied it
* Pasted it inside N8N under new credentials

Then, I purchased $5 worth of credits to ensure my workflow wouldn’t fail due to lack of API balance.

Now, I was ready to use ChatGPT in my automation 🎯

---<img width="641" height="631" alt="Screenshot 2025-10-04 213100" src="https://github.com/user-attachments/assets/ebd9bdea-b57f-4731-ba2d-37394c008a9b" />

## 🧩 Configuring ChatGPT Node

In the OpenAI node inside N8N:

* I selected model **GPT-4.1-mini**
* Under **Messages**, I set up roles (Assistant, System, and User)

For the **Assistant role**, I gave ChatGPT this prompt:

> “Act as a Tier 1 SOC Analyst Assistant.
>
> When provided with alert details, summarize the alert, enrich with threat intelligence, assess severity based on MITRE ATT&CK, and recommend next actions.”
>
> <img width="498" height="871" alt="Screenshot 2025-10-04 213217" src="https://github.com/user-attachments/assets/6a779c21-33d9-48e7-b708-ff0b31267b62" />

For the **System role**, I wrote:

> “Format the output clearly in a structured format such as:
>
> Summary → IOC Enrichment → Severity Assessment → Recommended Actions.”

This told ChatGPT how to structure its output.

Then I connected the **Webhook node** (which receives the alert data) to this ChatGPT node.

---

## 🧱 Passing Data from Splunk to ChatGPT

Now I wanted ChatGPT to analyze the alert data automatically.

So, I dragged relevant alert info (like the alert name and details) from the webhook output to the ChatGPT input.

But when I tried to drag “Results,” N8N just showed `[Object object]` — meaning it couldn’t directly interpret JSON objects.

So here’s what I did:

I used a **JSON.stringify** method to convert the results into a readable string format:

```
{{ JSON.stringify($json["results"], null, 2) }}

```

<img width="1198" height="385" alt="Screenshot 2025-10-04 213336" src="[https://github.com/user-attachments/assets/2517958d-08a5-4052-](https://github.com/user-attachments/assets/2517958d-08a5-4052-)


90dc-4e114e2660d3" />

* The `null` part means “don’t filter anything”
* The `2` means “use 2 spaces for indentation”

That way, ChatGPT could actually *see* and understand all the alert data.

---

<img width="822" height="323" alt="Screenshot 2025-10-04 213404" src="https://github.com/user-attachments/assets/b9b167ca-045f-4292-8a62-bac1b942ac65" />

## 💬 Sending AI-Generated Alerts to Slack

Next, I wanted the AI-generated analysis to show up automatically in **Slack**.

I opened Slack, created a workspace called **Sama**, and then created a channel called `#alerts`.

Then I went back to N8N:

* Added a new **Slack node**
* Selected “Send a message”
* Connected Slack to my workspace via OAuth
* Installed my Slack app and gave it the necessary permissions
* Pasted the **OAuth token** into N8N

After a quick permission fix (I had to add the app to the `#alerts` channel manually), it finally worked!

When I clicked **Execute Step**, I saw my first “test” message appear in Slack 🎉

It felt great seeing the automation connect end-to-end.

---

## 💥 End-to-End Test

Now it was time for the real test.

I connected all the nodes — Webhook → ChatGPT → Slack — and ran the workflow. <img width="1072" height="354" alt="image" src="https://github.com/user-attachments/assets/88046173-c12b-47e4-8339-2e00728b5c76" />

When I triggered the alert, the Slack channel received a message like this:

> “A brute-force alert named test-brute-force was triggered on the host involving the user Sama.
>
> The source IP address is x.x.x.x with a single recorded attempt.”

It even summarized the severity and recommended next actions automatically.

Everything worked perfectly!

---

## 🌍 External IP Enrichment using AbuseIPDB

Then, I wanted to test **Threat Intelligence Enrichment**.

Because my VM is local (not exposed to the internet), I simulated an external IP address.

I got a real suspicious IP from AbuseIPDB.

In N8N:

* I added a new node called **HTTP Request**
* Used the **AbuseIPDB API** to check the IP’s reputation
* Copied my API key from their website and configured it

N8N has a cool trick — I could import a `curl` command directly, and it filled everything in automatically.

Then I set it up so that ChatGPT could automatically call this enrichment tool during its analysis using this line:

> “For any IP enrichment, use the tool named abuse-ipdb-enrichment.”

After running the test again, it worked!

The output in Slack said something like:

> “The IP address is publicly routable, resides in Romania, has an abuse confidence score of 100/100, and was reported for SSH brute-force attacks.” <img width="1223" height="883" alt="Screenshot 2025-10-04 213846" src="https://github.com/user-attachments/assets/a2609a57-5cf1-430d-a1ec-c9c49af9ffd5" />

That was so satisfying to see. <img width="1045" height="484" alt="Screenshot 2025-10-04 213747" src="https://github.com/user-attachments/assets/017ee6b9-d297-442b-962e-81426364c3c8" />

---

## 🧩 What’s Next (Advanced Add-Ons)

In the future, I can extend this system by:

* Integrating **VirusTotal** for deeper file or domain analysis
* Adding a **Ticketing System** to automatically log and track alerts
* Connecting to **MCP Server** so AI can directly query Splunk for data

These would take my SOC automation to a full enterprise-grade setup.

---

## 🎯 Final Thoughts

This whole project really made me understand how AI and automation can work together in cybersecurity.

I learned about how to chain tools together to build something that thinks for you.

If anyone’s reading this and wants to replicate it — trust me, it’s so worth it.

