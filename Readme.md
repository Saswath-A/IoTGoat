# IoTGoat - Firmware

You can download the firmware from the official OWASP release:

🔗 [Download IoTGoat Firmware (v1.0)](https://github.com/OWASP/IoTGoat/releases/download/v1.0/IoTGoat-x86.img.gz)

- After downloading the firmware , **extract** the `.img` file . 
- Then use **binwalk** to view/extract the file system embedded in the firmware.

```
$ binwalk IoTGoat-x86.img

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
262144        0x40000         Linux EXT filesystem, blocks count: 4096, image size: 4194304, rev 2.0, ext2 filesystem data, UUID=57f8f4bc-abf4-655f-bf67-946fc0f9c0f9
5325930       0x51446A        xz compressed data
17301504      0x1080000       Squashfs filesystem, little endian, version 4.0, compression:xz, size: 3481630 bytes, 1352 inodes, blocksize: 262144 bytes
```
- Here **Squashfs Filesystem** is the type of compressed file system , used for read-only file-system. This is where the linux files are been held.

---

> Hardcoded Password

- Then you find some hardcoded passwds in /etc/shadow of the squashfs-root. This passwds can be cracked using some tools called `John The Ripper` and `HashCat`.

- I used some commonly used wordlist called mirabot-net. And cracked the passwd for **iotgoatuser**.

- For the root, it didn't work. I used rockyou.txt which also failed. So I tried to make a custom wordlist using some tools called crunch but it didn't as per my thought. So I am searching for some another ways to crack the password.

---
> Legacy network services listening upon start up

- This one is basically using nmap to search for open port in the router's ip.

- I got multiple services, for example:- http server , ssh and also telnetd which asks credentials when connected through nc.

---

> Persistent backdoor daemon configured to run during start up

- I found the backdoor when scanned the ip for open ports.I used nmap to scan for open ports. Some open ports were http,https which shows it is running some web.

- I used used a GUI tool of namp called zenmap in kali linux which hand in-built nmap option where i choose to scan all tcp port. This scan resulted in showing a tcp as backdoor.

- When I tried to connect to it through nc, i got the access as root. I also tried to create a file which was successful.

---

> A "secret" developer diagnostics page not directly accessible and exposes shell access to users

- When you look at the url of the web page , you can see the path of the file (i.e) */www/cgi-bin/luci* which is like a entry-point. By examining it and got to found out that it the code just gets and information and instruction from */usr/lib/lua/luci* (backend).

- Then I checked the controller directory and discovered the IoTGoat backend code for the camera and door modules, as well as a hidden command injection endpoint — **a secret developer diagnostics page**.

- Also it had a command to send through url (i.e) webcmd. I can send cmd through it for example, webcmd?cmd=ls will return me the output.

---

> Insufficient Privacy Protection

- I found out many leaked data using firmwalker which is a automated script to find the binarys, db files and more.

- It showed me many db files and keys for multiple apps. One db was in */usr/lib/lua/luci/controller/iotgoat/sensordata.db* which had email and their birthday dates.

---

> Lack of Device Management

- In the web application, only the kernal logs are avaiable in system. the Openwrt logs are not enabled.

---

		

