# PalvelinhallintaTyo
Tyon tarkoituksena on saada master toimimaan niin, että jos asennan uuden Linux distributionin niin laittamalla salt-minion ja hyväksymällä avaimen, uudelle minionille asentuu kaikki tarvitsemani sovellukset. 
Osa sovelluksista tulisi asentaa flatpakin avulla, mikä tuo omat haasteensa.

---
Asensin qemulle ubuntu serverin, joka toimii master koneena. Tämä siksi, että qemu ja virtual machine managerilla on olemassa NAT verkko jolla ssh yhteydet toimii virtuaalikoneiden välillä. 
Latasin Ubuntu serverin (22.04.1 LTS)[täältä](https://ubuntu.com/download/server) ja asensin sen.  
Asetoin serverin nimeksi ja käyttäjäksi `master`  

Päivitn serverin ja sensin serverille `salt-msater`.
```bash
master:~$ sudo apt update && sudo apt upgrade
master:~$ sudo apt install salt-master
```
Tarkistin salt-masterin olevan käynnissä:
![kuva1](https://github.com/Veliquu/Palvelintenhallinta/blob/main/Screenshot_2022-12-07_04-00-51.png)

Ja tarkistin msterin IP-osoitteen.
```bash
master:~$ hostname -I
192.168.122.57
```

Seuraavaksi asensin ensimmäisen minionin, joka toimii Fdora 36, jonka latasin [täätlä](https://getfedora.org/fi/workstation/download/). Valitsin version x86_64:lle.  Käyttäjänä minion.
Asennuksen jälkeen päivitin järjestelmän ja asensin salt-minionin.
```bash
[minion@localhost-live ~]$ sudo dnf update
[minion@localhost-live ~]$ sudo dnf install salt-minion
```
Tämän jälkeen muokkasin `/etc/salt/minion` tiedostoa.
```bash
master: 192.168.122.57
id: fedora_minion
```
Käynnistin `salt-minion` demonin uudetsaan ja tarkistin sen tilan.
```bash
[minion@localhost-live ~]$ sudo systemctl restart salt-minion.service 
[minion@localhost-live ~]$ sudo systemctl status salt-minion.service 
● salt-minion.service - The Salt Minion
     Loaded: loaded (/usr/lib/systemd/system/salt-minion.service; disabled; pre>
     Active: active (running) since Wed 2022-12-07 04:18:26 EET; 5s ago
       Docs: man:salt-minion(1)
             file:///usr/share/doc/salt/html/contents.html
             https://docs.saltproject.io/en/latest/contents.html
   Main PID: 16815 (salt-minion)
      Tasks: 7 (limit: 4651)
     Memory: 90.4M
        CPU: 959ms
     CGroup: /system.slice/salt-minion.service
             ├─16815 /usr/bin/python3 -P /usr/bin/salt-minion
             └─16821 /usr/bin/python3 -P /usr/bin/salt-minion

Dec 07 04:18:26 localhost-live systemd[1]: Starting salt-minion.service - The S>
Dec 07 04:18:26 localhost-live salt-minion[16815]: /usr/lib/python3.11/site-pac>
Dec 07 04:18:26 localhost-live salt-minion[16815]:   warnings.warn("Setuptools >
Dec 07 04:18:26 localhost-live systemd[1]: Started salt-minion.service - The Sa>
Dec 07 04:18:29 localhost-live salt-minion[16821]: [ERROR   ] The Salt Master h>
```
Pingasin myös masteria.
```bash
[minion@localhost-live ~]$ ping 192.168.122.57
PING 192.168.122.57 (192.168.122.57) 56(84) bytes of data.
64 bytes from 192.168.122.57: icmp_seq=1 ttl=64 time=0.195 ms
64 bytes from 192.168.122.57: icmp_seq=2 ttl=64 time=0.448 ms
64 bytes from 192.168.122.57: icmp_seq=3 ttl=64 time=0.300 ms
^C
--- 192.168.122.57 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2062ms
rtt min/avg/max/mdev = 0.195/0.314/0.448/0.103 ms
```
Pingi toimi, eli yhteys saadaan.  
Tarkistin myös Fedoran IP-osoitteen:
```bash
[minion@localhost-live ~]$ hostname -I
192.168.122.93 
```

Seuraavaksi asensin minion2, joka toimii Manjarolla. Latasin sen [täältä](https://manjaro.org/download/).
Asennuksen jälkeen päivitn koneen ja asensin `salt-minio`nin.
```bash
[minion@minion ~]$ sudo pacman -Syu
[minion@minion ~]$ sudo pacman -S salt
```
Muokkkasin `/etc/salt/minion` tiedostoa seuraavanlaiseksi.
```bash
msater: 192.168.122.57
id: Manjaro_minion
```
Ja käynnistin `salt-minion` demonin uudestaan ja tarkistin, että se on käynnissä.
```bash
[minion@minion ~]$ sudo systemctl restart salt-minion
[minion@minion ~]$ sudo systemctl status salt-minion
● salt-minion.service - The Salt Minion
     Loaded: loaded (/usr/lib/systemd/system/salt-minion.service; disabled; pre>
     Active: active (running) since Wed 2022-12-07 04:36:18 EET; 6s ago
       Docs: man:salt-minion(1)
             file:///usr/share/doc/salt/html/contents.html
             https://docs.saltproject.io/en/latest/contents.html
   Main PID: 12238 (salt-minion)
      Tasks: 7 (limit: 4688)
     Memory: 91.3M
        CPU: 770ms
     CGroup: /system.slice/salt-minion.service
             ├─12238 /usr/bin/python /usr/bin/salt-minion
             └─12240 /usr/bin/python /usr/bin/salt-minion

Dec 07 04:36:18 minion systemd[1]: Starting The Salt Minion...
Dec 07 04:36:18 minion salt-minion[12238]: /usr/lib/python3.10/site-packages/_d>
Dec 07 04:36:18 minion salt-minion[12238]:   warnings.warn("Setuptools is repla>
Dec 07 04:36:18 minion systemd[1]: Started The Salt Minion.
Dec 07 04:36:24 minion salt-minion[12240]: [ERROR   ] Failed to resolve address>
Dec 07 04:36:24 minion salt-minion[12240]: [ERROR   ] The Salt Master has cache>
```
Tarkistin Manjaro minionin IP osoitteen ja pingasin masteria.
```bash
[minion@minion ~]$ ip -br a
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             192.168.122.252/24 fe80::993c:6177:fb3f:4689/64

[minion@minion ~]$ ping 192.168.122.57
PING 192.168.122.57 (192.168.122.57) 56(84) bytes of data.
64 bytes from 192.168.122.57: icmp_seq=1 ttl=64 time=0.265 ms
64 bytes from 192.168.122.57: icmp_seq=2 ttl=64 time=0.369 ms
^C
--- 192.168.122.57 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 0.265/0.317/0.369/0.052 ms
```

##### Seuraavaksi siirryin takaisin masterille ja pingasin kumpaakin minionia.  
![Kuva2](https://github.com/Veliquu/Palvelintenhallinta/blob/main/Screenshot_2022-12-07_04-46-11.png)

Pingit toimii, joten seuraavaksi haettiin avaimet ja hyväksyttiin ne.  
![Kuva3](https://github.com/Veliquu/Palvelintenhallinta/blob/main/Screenshot_2022-12-07_04-48-50.png)

Tämän jälkeen kokeilin saltin toimivuutta.  
![Kuva4](https://github.com/Veliquu/Palvelintenhallinta/blob/main/Screenshot_2022-12-07_04-50-09.png)


Seuraavaksi loin `/srv/salt/ohjelmat` kansion ja loin sinne `init-sls` tiedoston.
```bash
master@master:~$ mkdir /srv/salt
master@master:~$ mkdir /srv/salt/ohjelmat
master@master:~$ cd /srv/salt/ohjelmat/
master@master:/srv/salt/ohjelmat$ sudo nano init.sls
```
Tähän tiedostoon laitoin kokeiluna `vim`in asennuksen.
```bash
vim:
  pkg.installed
```
Toiselle minionille asennus onnistui ja toiselle ei.  

![Kuva5](https://github.com/Veliquu/Palvelintenhallinta/blob/main/Screenshot_2022-12-07_05-10-35.png)

Salt versiot näyttäisi olevan eri, joten täytyy selvittää kuinka saan ubuntu serverille uusimman version, tai vaihtaa masteria.
Fedora
```bash
[minion@localhost-live ~]$ sudo salt-minion --version
[sudo] password for minion: 
/usr/lib/python3.11/site-packages/_distutils_hack/__init__.py:33: UserWarning: Setuptools is replacing distutils.
  warnings.warn("Setuptools is replacing distutils.")
salt-minion 3005.1
```
Manjaro
```bash
[minion@minion ~]$ sudo salt-minion --version
[sudo] password for minion: 
/usr/lib/python3.10/site-packages/_distutils_hack/__init__.py:33: UserWarning: Setuptools is replacing distutils.
  warnings.warn("Setuptools is replacing distutils.")
salt-minion 3005.1
```
Ubuntu server  
![Kuva6](https://github.com/Veliquu/Palvelintenhallinta/blob/main/Screenshot_2022-12-07_05-21-56.png)
