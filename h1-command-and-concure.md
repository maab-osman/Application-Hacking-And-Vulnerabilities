# h1 Command and Concure

## a) Baseline

### 1. In-scope
The purpose of this scope is to keep my study environment reliable and secure so I can complete coursework without data loss, account compromise, or accidental disruption to the household network.
**Scope boundary:** I cover what I own and can configure. Anything beyond that is treated as an external dependency.

**What I include**
- Home network that consists of Zyxel router, my Wi-Fi network, and basic router settings (the stuff I can actually configure myself).
- Study devices I use for the course including a ThinkPad laptop (Linux), VirtualBox for lab VMs, my phone for MFA, and iPad for taking notes.
- Storage I use for schoolwork icluding Haaga-Helia OneDrive, local storage on my laptop, and my personal iCloud (mainly for syncing notes/files).
- Credentials are stored either in my password manager and/or as SSH keys on my laptop.
- Information handled for the course like course materials, my own notes, GitHub repositories, and any credentials used in labs

### 2. Out-of-scope

**What I exclude**
- **Family members’ devices:** I don’t control their settings or updates, so I can’t realistically include them in my scope.
- **PlayStation:** it’s on the network, but it’s not part of my study or lab work. I’m accepting that risk as “not relevant for this course scope.”
- **Smart TV:** same reason as the PlayStation it is not used for coursework and I don’t want the scope to grow into everything in the house.
- **Anything on the ISP side of the connection:** once traffic leaves my router, it’s outside my control. That part is handled by the ISP and is not in my scope.


### 3. Key interfaces 

**Main boundaries and connections**
- **Home network and Internet:** The main boundary is my Zyxel router. It’s where my home Wi-Fi connects to the internet. If something weird happens on the network, this is the first place I would check.
- **Course cloud services:** I use GitHub for course submission and OneDrive for course files. These are outside my home network, so the security depends on my account settings (especially MFA) and the service provider.
- **Accounts and login:** My phone is part of the setup because I use it for MFA. If I lose access to my phone, it affects my ability to access coursework.
- **Local lab environment:** My VirtualBox VMs run locally on my ThinkPad. They’re “separate” from my main system, but still inside my control because they live on my laptop.
- **External parties:** The ISP provides the connection, Zyxel is the router vendor, and GitHub + Microsoft are cloud providers. If there’s an outage or a platform issue, I can’t fix that myself.

## Network & Interface Diagram

<img width="759" height="645" alt="Screenshot 2026-01-18 at 9 02 09 AM" src="https://github.com/user-attachments/assets/73d4a535-f39c-4eaa-9341-a62cab62b207" />

---
### Evidence

- OneDrive course folder view
Shows where I store course materials and notes (cloud storage in use).

<img width="1389" height="509" alt="Screenshot 2026-01-18 at 9 28 20 AM" src="https://github.com/user-attachments/assets/d5e435a9-4f7e-4725-bc2e-09c9a795dfaa" />


- Github commit history
Shows my continuous work and version history for this submission.

<img width="1327" height="464" alt="Screenshot 2026-01-18 at 9 26 12 AM" src="https://github.com/user-attachments/assets/2f5bb52b-28c5-4915-911c-f8f0ee64344f" />

- VirtualBox VM list
Shows the lab VM environment that will be used for this course.

<img width="442" height="269" alt="Screenshot 2026-01-18 at 9 37 36 AM" src="https://github.com/user-attachments/assets/ac51a397-e93a-4fed-9293-7aaed3087914" />

## b) Tie it to the Standard

Even in a home setup, there are other people and services that are affected by how I use my network and lab environment. Below are the main interested parties that matter for my home network + study lab, what they need from me, and how I could show that I meet those expectations.

| Interested party | Need / expectation / requirement | ISO 27001 requirement area | How I demonstrate |
|---|---|---|---|
| Me | I need my coursework environment to stay usable and not break right before deadlines. I also need my files not to disappear  and my accounts to stay secure. | Planning, Operation, Performance evaluation | Repo commit history, basic backup snd sync settings, OS update status, VM list |
| Family members | They expect privacy and normal internet use. My lab work shouldn’t slow down the Wi-Fi or cause risky activity on the home network. | Context, Operation | WPA2/WPA3 Wi-Fi security and strong password|
| Cloud providers (GitHub & OneDrive) | They expect my account to be protected and that I follow their terms | Operation, Improvement | MFA enabled on accounts, no secrets committed to repos, security alerts, password manager |
| Haaga Helia and courses | They expect academic integrity and that course exercises are done safely. | Leadership, Operation, Improvement | Clear scope statement + exclusions, documentation of boundaries, repo shows my own work and progress, no prohibited tooling in the home environment |

---

## References
ISO/IEC (2022) *ISO/IEC 27001:2022 Information security, cybersecurity and privacy protection — Information security management systems — Requirements*. Geneva: International Organization for Standardization.

