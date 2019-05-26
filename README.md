# CTF di Cyber Saiyan

## Aperitech - 27 Maggio 2019
Durante il [secondo Aperitech di Cyber Saiyan a Roma](https://www.eventbrite.it/e/biglietti-secondo-appuntamento-con-le-ctf-aperitech-di-cyber-saiyan-60310700930) giocheremo una CTF preparata da Alberto *yuntao* Caponi

Sarà possibile anche partecipare da remoto visto che l'Aperitech sarà trasmesso in live streaming

Step di preparazione alla CTF
* installare VirtualBox sul proprio PC ([download](https://www.virtualbox.org/wiki/Downloads))
* [scaricare l'applicazione virtuale OVA zippata](https://mega.nz/#!bAoBzY7T!Y0HTuOPgjzW092TfUw4fskNyxdAl4steg0n_jyyM9-M) (file: *Aperitech_May27.zip* | dimesione: 5.28GB | md5sum: *034a40a6297d5cf8825a9ef674241c80*)
* estrarre dallo zip il file *Aperitech_May27.ova* 

**!! E' NECESSARIA LA PASSWORD CHE SARA' COMUNICATA ALL'INIZIO DELL'APERITECH !!**

Import OVA in VirtualBox
* importare l'applicazione virtuale in Virtualbox (*File*->*Importa applicazione virtuale...* oppure *CTRL+I*) lasciando le impostazioni di default (tempo stimato: 5 minuti | nome: *vm*)
* verificare dal menu *File*->*Gestore di rete dell'host...* (oppure *CTRL+H*), icona *Proprietà*, la presenza di almeno una rete host con DHCP abilitato. Tale rete è necessaria a raggiungere la VM guest dalla nostra macchina host (la defaul network per la rete *vboxnet0* è 192.168.56.0/24). Se non è presente nessuna rete crearne una cliccando sul tasto destro *Crea...*, assicurarsi che il DHCP sia abilitato (la VM prenderà l'indirizzo in DHCP)
![vboxnet0](vbox-01.png)
* verificare da *Impostazioni*->*Rete" della *vm* che sia abilitata la *Scheda 1* e l'opzione *Connessa a:* sia impostata a *Scheda solo host* (*Nome*: *vboxnet0*
![vboxnet0](vbox-02.png)

Avviare *vm* (che prenderà un IP nel pool DHCP 192.168.56.0/24) ed iniziare a giocare :)

NOTA: Il writeup sarà messo on line al termine dell'Aperitech
