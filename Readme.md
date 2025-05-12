# IoTGoat - Firmware

You can download the firmware from the official OWASP release:

🔗 [Download IoTGoat Firmware (v1.0)](https://github.com/OWASP/IoTGoat/releases/download/v1.0/IoTGoat-x86.img.gz)

1. After downloading the firmware , **extract** the `.img` file . 
2. Then use **binwalk** to view/extract the file system embedded in the firmware.

```
$ binwalk IoTGoat-x86.img

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
262144        0x40000         Linux EXT filesystem, blocks count: 4096, image size: 4194304, rev 2.0, ext2 filesystem data, UUID=57f8f4bc-abf4-655f-bf67-946fc0f9c0f9
5325930       0x51446A        xz compressed data
17301504      0x1080000       Squashfs filesystem, little endian, version 4.0, compression:xz, size: 3481630 bytes, 1352 inodes, blocksize: 262144 bytes
```
3. Here **Squashfs Filesystem** is the type of compressed file system , used for read-only file-system. This is where the linux files are been held.

> Hardcoded Password

4. Then you find some hardcoded passwds in /etc/shadow of the squashfs-root. This passwds can be cracked using some tools called `John The Ripper` and `HashCat`.

5. I used some commonly used wordlist called mirabot-net. And cracked the passwd for **iotgoatuser**.

6. For the root, it didn't work. I used rockyou.txt which also failed. So I tried to make a custom wordlist using some tools called crunch but it didn't as per my thought. So I am searching for some another ways to crack the password.

> Persistent backdoor daemon configured to run during start up

7. I found the backdoor when scanned the ip for open ports.I used nmap to scan for open ports. Some open ports were http,https which shows it is running some web.

8. I used used a GUI tool of namp called zenmap in kali linux which hand in-built nmap option where i choose to scan all tcp port. This scan resulted in showing a tcp as backdoor.

9. When I tried to connect to it through nc, i got the access as root. I also tried to create a file which was successful.

