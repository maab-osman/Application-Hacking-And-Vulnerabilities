# H2 Break & Unbreak - Tero
## x) Summary of Materials

### OWASP: A01 Broken Access Control
- Broken Access Control happens when authorization rules are missing or incorrectly enforced on the server side
- It allows users to access data, functions, or pages beyond their intended permissions
- Common causes include insecure direct object references (IDOR), missing role checks, URL or parameter manipulation, and privilege escalation
- This vulnerability is extremely common and ranks as the most critical risk in the OWASP Top 10
- Secure design requires deny by default, centralized authorization logic, least privilege, and thorough access control testing

 ---
 
### Fuzzing URLs with ffuf (Karvinen 2023)
- Directory fuzzing is a reconnaissance technique used to discover hidden or unlinked web paths
- ffuf is a fast web fuzzer that replaces a keyword in a URL with values from a wordlist
- Wordlists such as SecLists contain thousands of common directory and file names
- False positives are common whic is filtering by response size, words, or lines helps isolate meaningful results
- Successfully discovered directories may expose sensitive functionality and enable further attacks

Excercise
- Opening 2 terminals one running the vulnerable server and on to run `ffuf`
- We run the first command to download the sample target sample
  
 ```
$ wget https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/dirfuzt-0
$ chmod u+x dirfuzt-0
$ ./dirfuzt-0
```
- We open the our local to see somethig like this:

<img width="736" height="395" alt="Screenshot 2026-01-24 at 7 22 31 PM" src="https://github.com/user-attachments/assets/8098ea32-5d1d-4829-9221-dd679fdbcd80" />

- In the other terminal i will install ffuf
  
 ```
sudo apt update
sudo apt install fuff

```
- In the same terminal I will get a wordlist
  
 ```
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/common.txt

```
- Ran ffuf to fuzz directories:

 ``` ./ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ  ```

 <img width="973" height="519" alt="Screenshot 2026-01-24 at 9 16 44 PM" src="https://github.com/user-attachments/assets/64ca6f66-ec4f-4052-9761-fc893bfc032c" />


- Filtered out empty responses to identify real directories:

 ``` ./ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ -fs 132  ```

- I mananged to find the hidden directory: ```/admin```
- Verified it in the browser and the page actually said "You've found it!".
- This concludes that just because a directory like ```/admin``` isn't linked on the homepage doesn't mean it's safe. If it's on the server, a fuzzer will find it.

---

### Access Control & Privilege Escalation - PortSwigger
- Access control defines what an authenticated user is allowed to do within an application
- Vulnerabilities occur when these rules are missing, inconsistent, or incorrectly implemented on the server side
- Vertical privilege escalation allows a normal user to access admin-only functionality
- Horizontal privilege escalation allows a user to access another user’s data or actions at the same privilege level
- Insecure Direct Object References (IDOR) occur when identifiers are exposed and not properly validated
- PortSwigger emphasizes that hiding functionality or relying on client-side controls is not sufficient that's access control must always be enforced server-side
  
---

### Report Writing (Karvinen 2006)
- Write so another person can reproduce your results in the same environment
- Record exact commands, actions, and outcomes including failures
- Use clear structure, headings, and simple language
- Explain what was done, why it was done, and what was learned
- Cite all sources and never claim work you did not do
- In security work, the report is often the main deliverable
- A clear report builds trust and proves the work was done systematically
  
---

### a) Break: 010-staff-only

- I started by installing the Python web frameworks by running ```sudo apt-get -y install wget unzip micro python3-flask python3-flask-sqlalchemy```
- Then downloaded the lab and unzipped it then running ```staff-only.py``` on my terminal
  <img width="1008" height="307" alt="Screenshot 2026-01-24 at 11 06 42 PM" src="https://github.com/user-attachments/assets/aa39c18d-3c40-4a94-828f-2790a95fe906" />

- I opened the web interface and started testing the hint ```123```
  <img width="1558" height="790" alt="Screenshot 2026-01-24 at 11 07 34 PM" src="https://github.com/user-attachments/assets/d5eefcf0-3dbb-483c-8bf2-80474fdfb3a8" />

- Since we are supposed to find the admin, we can try a common injection ```' OR 1=1--```
- First we change the variable type from number to String and this is the result:
  
  <img width="711" height="391" alt="Screenshot 2026-01-24 at 11 10 13 PM" src="https://github.com/user-attachments/assets/3d2c7286-3def-48ab-8eb8-caf83c6fd48a" />

- To find the admin password, we could modify this injection to give us something similar to a ```SUPERADMIN```
- hence in the same way we inject ``` OR password LIKE '%SUPERADMIN%'-- ```
  
  <img width="1477" height="396" alt="Screenshot 2026-01-24 at 11 15 05 PM" src="https://github.com/user-attachments/assets/12222935-1b01-4ce1-9e19-aabb3500a223" />

- We have successfully found it!

---

### b) Fix: 010-staff-only
- I opened ```staff-only.py``` using micro and found the line where the search query was built.
  <img width="1003" height="795" alt="Screenshot 2026-01-24 at 11 19 59 PM" src="https://github.com/user-attachments/assets/1e2a4810-6060-49f3-baf5-a7d4ab87ee01" />

-  The biggest problem is on line 22. I saw that the code uses ``+ pin +`` to glue the user's input directly into the SQL string. This is exactly what allowed me to break out of the query using a single quote.
-  The code takes the pin from the form and just converts it to a string ``(str())``. It doesn't check for special characters like --, OR, or ;, so it basically trusts whatever the user types.
-  On line 26, the code calls ```db.session.commit()``` after a ```SELECT``` statement. While not a security hole, it's a performance mistake.

Fix
-  I removed the ```+ pin +``` part of the SQL string. Instead of putting the data directly in the string, I put a colon placeholder ```(:pin)```.
-  Also, When calling ```db.session.execute()```, I passed the actual pin variable as a second argument.

```
# Secure: input is treated as a separate value
sql = "SELECT password FROM pins WHERE pin = :pin"
res = db.session.execute(text(sql), {"pin": pin})

```
- Then I ran the python file again to see if the vulnerability still exits.
  
<img width="1144" height="634" alt="Screenshot 2026-01-24 at 11 26 33 PM" src="https://github.com/user-attachments/assets/a9f1ccc5-d189-4db6-825d-674f167dac84" />

- It's fixed!
  
---

### c) Fuzzing: dirfuzt-1
- Started by installing ```dirfuzt-1``` on the terminal
<img width="1020" height="617" alt="Screenshot 2026-01-25 at 12 03 49 AM" src="https://github.com/user-attachments/assets/4a928e27-87dd-4321-9a18-f154ed0f3e6c" />

- Then I ran ``ffuf`` and noticed that most sizes were 154 bytes.
  
<img width="1134" height="618" alt="Screenshot 2026-01-25 at 12 05 28 AM" src="https://github.com/user-attachments/assets/3b5001cf-9efe-44f5-b3ef-95deb9d172eb" />

- Therefore I have decided to filter by that running:

```
$ ./ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ -fs 154

```
<img width="1124" height="633" alt="Screenshot 2026-01-25 at 12 06 58 AM" src="https://github.com/user-attachments/assets/e9153adb-8baf-481c-988a-aaa47dc50411" />

- I found 4 matching responses, so lets try ``` /.git ``` and here is the response:

  <img width="1098" height="628" alt="Screenshot 2026-01-25 at 12 09 11 AM" src="https://github.com/user-attachments/assets/c5240778-47bc-4286-a15f-af6b8ad65c58" />

---

### d) Break: 020-your-eyes-only

- I navigated to the second challenge directory, created a Python ```virtualenv``` , and installed the requirements.
-  I ran the migrations to set up the database and started the Django server.

<img width="786" height="95" alt="Screenshot 2026-01-25 at 9 31 28 AM" src="https://github.com/user-attachments/assets/af0ac1f4-9f1c-4940-88b6-bbb590e81aa1" />

-  I registered and logged in. But when I pressed the admin dashboard I got an error
<img width="805" height="231" alt="Screenshot 2026-01-25 at 9 32 04 AM" src="https://github.com/user-attachments/assets/d661c7da-1514-4091-acf0-bf3f9eb56af7" />


-  So I used ```ffuf``` to scan the site. I found a hidden directory called ```/admin-console/```

  <img width="822" height="105" alt="Screenshot 2026-01-25 at 9 32 44 AM" src="https://github.com/user-attachments/assets/843f4ab7-05ff-4322-a8ad-e7d1efa5f383" />

- I went straight to ```http://127.0.0.1:8000/admin-console/```. It worked!
<img width="1025" height="415" alt="Screenshot 2026-01-25 at 9 34 26 AM" src="https://github.com/user-attachments/assets/734926f4-5603-41e9-b239-348de43e2e81" />

- This is an Insecure Direct Object Reference (IDOR) or Broken Access Control. The developer assumed only admins would know the URL, or they forgot to verify user roles.
---

### e) Fix: 020-your-eyes-only
- First I need to find the source file using the grep command, but it did not directly lead me to it since it was scanning binary files. So I started narrowing down to files containing the "admin-console"

  <img width="1029" height="126" alt="Screenshot 2026-01-25 at 10 19 44 AM" src="https://github.com/user-attachments/assets/b5ec4d28-7ada-4031-a618-abb028c4d4ea" />

- After finally finding it in the ```urls.py```, it turns out the the paths are linked to anoother file called ```views.py```. So I navigated to that file.
- The issue is clear here, the ```AdminShowAllView``` currently allows any authenticated user to access it.
- This means that any one logged it can access the admin console which is a dangerous.
- In order to fix this, I have to modify it to restrict access.
- the fix was simple, it was to modidy the ```test_func``` in ```AdminShowAll``` to restrict only to staff members.
  <img width="1060" height="156" alt="Screenshot 2026-01-25 at 10 20 43 AM" src="https://github.com/user-attachments/assets/b13753a1-cffa-484c-a0da-499b597ec999" />
  
- After modifying and running again, the vulnerability seems to be fixed.
<img width="1090" height="146" alt="Screenshot 2026-01-25 at 10 21 41 AM" src="https://github.com/user-attachments/assets/93471dc7-643a-4946-9776-a114bfbb7f94" />

---
### Refrences 
- Karvinen, T. (2006). Raportin kirjoittaminen. [online] Terokarvinen.com. Available at: https://terokarvinen.com/2006/raportin-kirjoittaminen-4/ [Accessed 25 Jan. 2026].
- Karvinen, T. (2023). Fuzz URLs - Find Hidden Directories. [online] Terokarvinen.com. Available at: https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/ [Accessed 25 Jan. 2026].
- Karvinen, T. (2024). Hack’n Fix. [online] Terokarvinen.com. Available at: https://terokarvinen.com/hack-n-fix/ [Accessed 25 Jan. 2026].
- Open Worldwide Application Security Project (OWASP) (2021). A01:2021 – Broken Access Control. [online] Owasp.org. Available at: https://owasp.org/Top10/2021/A01_2021-Broken_Access_Control/index.html [Accessed 25 Jan. 2026].
- OpenAI(2024). Large Language Model. [online] Assisted in structuring the Markdown (MD) report template and troubleshooting environment setup.
- PortSwigger (2024). Access Control. [online] Portswigger.net. Available at: https://portswigger.net/web-security/access-control [Accessed 25 Jan. 2026].


