# h1 Command and Concure

## a) Baseline

### 1. In-scope

**What I include**
- **Home network (things I manage):** Zyxel router, my Wi-Fi network, and basic router settings (the stuff I can actually configure myself).
- **Study devices I use for the course:** ThinkPad laptop (Linux), VirtualBox for lab VMs, my phone for MFA, and iPad for taking notes.
- **Storage I use for schoolwork:** Haaga-Helia OneDrive, local storage on my laptop, and my personal iCloud (mainly for syncing notes/files).
- **Information handled for the course:** course materials, my own notes, GitHub repositories, and any credentials used in labs

### 2. Out-of-scope (and why)

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


