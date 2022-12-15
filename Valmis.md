Harjoitustyössä luon salt järjestelmän, jonka avulla distroa vaihtamalla voin vain ajaa salt komennon ja kaikki tarvitsemani sovellukset saadaan asennettua.  

Virtuaalikoneet on asennet Fedora 36 Lenovo Legion Pro 5 kannatteavall tietokoneelle.


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

Ja tarkistin msterin IP-osoitteen.
```bash
master:~$ hostname -I
192.168.122.57
```

Seuraavaksi asensin ensimmäisen minionin, joka toimii Fdora 36, jonka latasin [täältä](https://getfedora.org/fi/workstation/download/). Valitsin version x86_64:lle.  Käyttäjänä minion.
Asennuksen jälkeen päivitin järjestelmän ja asensin salt-minionin.
```bash
[fedora_minion@localhost-live ~]$ sudo dnf update
[fedora_minion@localhost-live ~]$ sudo dnf install salt-minion
```
Tämän jälkeen muokkasin `/etc/salt/minion` tiedostoa.
```bash
master: 192.168.122.57
id: Fedora_minion
```
Käynnistin `salt-minion` demonin uudetsaan ja tarkistin sen tilan.
```bash
[fedora_minion@localhost-live ~]$ sudo systemctl restart salt-minion.service 
[fedora_minion@localhost-live ~]$ sudo systemctl status salt-minion.service 
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
[fedora_minion@localhost-live ~]$ ping 192.168.122.57
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
[fedora_minion@localhost-live ~]$ hostname -I
192.168.122.245
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
Pingi toimi, eli yhteys saadaan.

---
Seuraavaksi siirryin takaisin masterille ja pingasin kumpaakin minionia.
Fedoraa:
```bash
master@master:~$ ping 192.168.122.245
PING 192.168.122.245 (192.168.122.245) 56(84) bytes of data.
64 bytes from 192.168.122.245: icmp_seq=1 ttl=64 time=0.243 ms
64 bytes from 192.168.122.245: icmp_seq=2 ttl=64 time=0.410 ms
64 bytes from 192.168.122.245: icmp_seq=3 ttl=64 time=0.485 ms
^C
--- 192.168.122.245 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2057ms
rtt min/avg/max/mdev = 0.243/0.379/0.485/0.101 ms

``` 
Manjaroa:
```bash
master@master:~$ ping 192.168.122.252
PING 192.168.122.252 (192.168.122.252) 56(84) bytes of data.
64 bytes from 192.168.122.252: icmp_seq=1 ttl=64 time=0.560 ms
64 bytes from 192.168.122.252: icmp_seq=2 ttl=64 time=0.626 ms
64 bytes from 192.168.122.252: icmp_seq=3 ttl=64 time=0.568 ms
64 bytes from 192.168.122.252: icmp_seq=4 ttl=64 time=0.643 ms
^C
--- 192.168.122.252 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3067ms
rtt min/avg/max/mdev = 0.560/0.599/0.643/0.035 ms
```

Molemmat pingit toimivat, joten seuraavaksi haettiin avaimet ja hyväksyttiin ne.
```bash
master@master:~$ sudo salt-key
Accepted Keys:
Denied Keys:
Unaccepted Keys:
Fedora_minion
Rejected Keys:
```
Manjaro avain ei löytynyt.  
Hyväksyin nämä jonka jälkeen käynnistin salt-minionin manjarolla uudestaan.
```bash
master@master:~$ sudo salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
Fedora_minion
Proceed? [n/Y] y
Key for minion Fedora_minion accepted.
```  

```bash
[minion@minion ~]$ sudo systemctl restart salt-minion
```
Tämän jälkeen avain löytyi, ja hyväksyin sen
```bash
master@master:~$ sudo salt-key
Accepted Keys:
Fedora_minion
Denied Keys:
Unaccepted Keys:
Manjaro_minion
Rejected Keys:

master@master:~$ sudo salt-key -A
The following keys are going to be accepted:
Unaccepted Keys:
Manjaro_minion
Proceed? [n/Y] y
Key for minion Manjaro_minion accepted.
```
Tämän jälkeen kokeilin saltin toimivuutta:
```bash
aster@master:~$ sudo salt '*' cmd.run "echo Hei maailma"
Manjaro_minion:
    Hei maailma
Fedora_minion:
    Hei maailma
```
Toimi.  

Loin `/srv/salt/ohjelmat` kansion ja loin sinne `init-sls` tiedoston.
```bash
master@master:~$ mkdir /srv/salt
master@master:~$ mkdir /srv/salt/ohjelmat
master@master:~$ cd /srv/salt/ohjelmat/
master@master:/srv/salt/ohjelmat$ sudo nano init.sls
```
Tähän tiedostoon kokeiluna asennuksen `vim`ille:
```bash
vim:
  pkg.installed
```
Kokeilin saltilla:
```bash
master@master:/srv/salt/ohjelmat$ sudo salt '*' state.apply ohjelmat
Manjaro_minion:
----------
          ID: vim
    Function: pkg.installed
      Result: True
     Comment: The following packages were installed/updated: vim
     Started: 08:03:48.910891
    Duration: 3593.837 ms
     Changes:   
              ----------
              vim:
                  ----------
                  new:
                      9.0.0910-1
                  old:
              vim-runtime:
                  ----------
                  new:
                      9.0.0910-1
                  old:

Summary for Manjaro_minion
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   3.594 s
Fedora_minion:
----------
          ID: vim
    Function: pkg.installed
      Result: False
     Comment: The following packages failed to install/update: vim
     Started: 08:03:48.270916
    Duration: 11943.694 ms
     Changes:   
              ----------
              gpm-libs:
                  ----------
                  new:
                      1.20.7-41.fc37
                  old:
              vim-common:
                  ----------
                  new:
                      2:9.0.1006-1.fc37
                  old:
              vim-enhanced:
                  ----------
                  new:
                      2:9.0.1006-1.fc37
                  old:
              vim-filesystem:
                  ----------
                  new:
                      2:9.0.1006-1.fc37
                  old:

Summary for Fedora_minion
------------
Succeeded: 0 (changed=1)
Failed:    1
------------
Total states run:     1
Total run time:  11.944 s

 
 ```
Fedora minion antoi errorina, että asennus olisi epäonnistunut, mutta vim toimii silti Fedoralla.  
Tarkastellessani erroria, huomasin että Fedoralla vim:in package nimi ei ole vim vaan vim-enhanced.  
Antin ja [Saltstack manualin](https://docs.saltproject.io/en/latest/topics/tutorials/states_pt3.html) avulla kokeilin if-lauseketta vimin asennukseen.
Muutin siis init-sls tiedostoa näyttämään tältä:
```bash
tarvitut_sovellukset:
  pkg.installed:
    - pkgs:
      {% if grains['os_family'] == 'RedHat' %}
      - vim-enhanced
      {% if grains['os_family'] == 'Arch' %}
      - vim
      {% if grains['os_family'] == 'Debian' %}
      - vim
      {% endif %}
```
Koska Masterina on Ubuntu serveri ja minioni ovat Fedora ja Manjaro, sain kolmen isoimman os_familyn tietää seuraavanlaisesti:
```bash
master@master:~$ sudo salt '*' grains.item os_family
Manjaro_minion:
    ----------
    os_family:
        Arch
Fedora_minion:
    ----------
    os_family:
        RedHat
master@master:~$ sudo salt-call --local grains.item os_family
local:
    ----------
    os_family:
        Debian
```

Kokeilin ajaa salt komennon uudestaan:
```bash
master@master:/srv/salt/ohjelmat$ sudo salt '*' state.apply ohjelmat
Manjaro_minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 08:25:47.717441
    Duration: 36.961 ms
     Changes:   

Summary for Manjaro_minion
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
Total run time:  36.961 ms
Fedora_minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 08:25:47.284054
    Duration: 439.469 ms
     Changes:   

Summary for Fedora_minion
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
Total run time: 439.469 ms
```
Tällä kertaa ei tullut mitään erroria, ja huomasin että aikaisemmassa kohdassa vim asentui Fedoralle vaikka antoi errorin.  

---
Seuraava piti saada flatpak repository minioneille. Tätä varten löysin esimerki [täältä](https://github.com/TTimo/linux-salted/blob/master/salt/flatpak.sls)
Joten lisäsin esimerkin mukaan sen init-sls.
```bash
flatpak-packages:
  pkgrepo.managed:
    - ppa: alexlarsson/flatpak
  pkg.latest:
    - refresh: True
    - pkgs:
      - flatpak
      - flatpak-builder

flatpak-flathub:
  cmd.run:
    - name: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
    - onchanges:
      - flatpak-packages
```
Ajoin salt komennon uudestaan:
```bash
master@master:/srv/salt/ohjelmat$ sudo salt '*' state.apply ohjelmat
Manjaro_minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 08:32:09.016028
    Duration: 37.507 ms
     Changes:   
----------
          ID: flatpak-flathub
    Function: cmd.run
        Name: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
      Result: False
     Comment: The following requisites were not found:
                                 onchanges:
                                     id: flatpak-packages
     Started: 08:32:09.062203
    Duration: 0.007 ms
     Changes:   
----------
          ID: flatpakrepo
    Function: pkgrepo.managed
      Result: False
     Comment: State 'pkgrepo.managed' was not found in SLS 'ohjelmat'
              Reason: 'pkgrepo' __virtual__ returned False
     Changes:   
----------
          ID: flatpakrepo
    Function: pkg.latest
      Result: True
     Comment: All packages are up-to-date (flatpak, flatpak-builder).
     Started: 08:32:09.463641
    Duration: 608.528 ms
     Changes:   

Summary for Manjaro_minion
------------
Succeeded: 2
Failed:    2
------------
Total states run:     4
Total run time: 646.042 ms
Fedora_minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 08:32:08.602631
    Duration: 421.102 ms
     Changes:   
----------
          ID: flatpak-flathub
    Function: cmd.run
        Name: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
      Result: False
     Comment: The following requisites were not found:
                                 onchanges:
                                     id: flatpak-packages
     Started: 08:32:09.024883
    Duration: 0.003 ms
     Changes:   
----------
          ID: flatpakrepo
    Function: pkgrepo.managed
      Result: False
     Comment: Failed to configure repo 'flatpakrepo': The repo does not exist and needs to be created, but either a baseurl or a mirrorlist needs to be given
     Started: 08:32:09.025343
    Duration: 17.778 ms
     Changes:   
----------
          ID: flatpakrepo
    Function: pkg.latest
      Result: True
     Comment: The following packages were successfully installed/upgraded: flatpak-builder The following packages were already up-to-date: flatpak
     Started: 08:32:09.043238
    Duration: 12082.274 ms
     Changes:   
              ----------
              ccache:
                  ----------
                  new:
                      4.7.3-1.fc37
                  old:
              debugedit:
                  ----------
                  new:
                      5.0-5.fc37
                  old:
              dwz:
                  ----------
                  new:
                      0.14-7.fc37
                  old:
              ed:
                  ----------
                  new:
                      1.18-2.fc37
                  old:
              flatpak-builder:
                  ----------
                  new:
                      1.2.3-1.fc37
                  old:
              hiredis:
                  ----------
                  new:
                      1.0.2-3.fc37
                  old:
              patch:
                  ----------
                  new:
                      2.7.6-17.fc37
                  old:

Summary for Fedora_minion
------------
Succeeded: 2 (changed=1)
Failed:    2
------------
Total states run:     4
Total run time:  12.521 s
ERROR: Minions returned with non-zero exit code
```
Ei toiminut, tosin vim kokeilusta päätellen itselleni näyttäisi siltä, että flatpak olisi saatu Fedoralle.  
Kokeilin tätä fedoralla:
```bash
[fedora_minion@localhost-live ~]$ flatpak update
Looking for updates...

Nothing to do.
```
Näyttäisi siltä että flatpak on saatu fedoralle.

Pyörittelin koodia tiedostossa ja jostakin syystä seuraavanlainen ratkaisu toimi:
```bash
flatpak-flathub:
  cmd.run:
    - name: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
    - onchanges:
      - flatpak-packages
       
flatpak-packages:
  pkg.latest:
    - refresh: True
    - pkgs:
      - flatpak
      - flatpak-builder
```
Ajettiin salt komento
```bash
master@master:/srv/salt/ohjelmat$ sudo salt '*' state.apply ohjelmat
Manjaro_minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 08:44:17.954742
    Duration: 38.045 ms
     Changes:   
----------
          ID: flatpak-packages
    Function: pkg.latest
      Result: True
     Comment: All packages are up-to-date (flatpak, flatpak-builder).
     Started: 08:44:17.993730
    Duration: 453.183 ms
     Changes:   
----------
          ID: flatpak-flathub
    Function: cmd.run
        Name: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
      Result: True
     Comment: State was not run because none of the onchanges reqs changed
     Started: 08:44:18.447058
    Duration: 0.002 ms
     Changes:   

Summary for Manjaro_minion
------------
Succeeded: 3
Failed:    0
------------
Total states run:     3
Total run time: 491.230 ms
Fedora_minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 08:44:17.567561
    Duration: 434.925 ms
     Changes:   
----------
          ID: flatpak-packages
    Function: pkg.latest
      Result: True
     Comment: All packages are up-to-date (flatpak, flatpak-builder).
     Started: 08:44:18.003446
    Duration: 6221.634 ms
     Changes:   
----------
          ID: flatpak-flathub
    Function: cmd.run
        Name: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
      Result: True
     Comment: State was not run because none of the onchanges reqs changed
     Started: 08:44:24.225226
    Duration: 0.002 ms
     Changes:   

Summary for Fedora_minion
------------
Succeeded: 3
Failed:    0
------------
Total states run:     3
Total run time:   6.657 s
```
Flatpak saatiin kummallekkin minionille, joten seuraavaksi piti saada asennettua flatpakillä ohjelmia.

---
Flatpakillä ohjelmien asentamiseen päädyin käyttämään bash skriptiä joka ajaa flatpakin asennus komennon.  
Loin scripti tiedoston `srv/salt/ohjelmat` direktpryyn.  
Flatpak pakettien nimet sain [flathubin](https://flathub.org/home) sivuilta.
Scripti:
```bash
#!/bin/bash
flatpak install app/com.discordapp.Discord/x86_64/stable com.visualstudio.code com.microsoft.Teams -y
```
`init.sls`ään lisäsin
```bahs
skripti:
  cmd.script:
    - source: salt://ohjelmat/script
```
Seuraavaksi ajoin salt komennon:
```bash
master@master:/srv/salt/ohjelmat$ sudo salt '*' state.apply ohjelmat
Fedora_minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 08:55:07.149141
    Duration: 427.336 ms
     Changes:   
----------
          ID: flatpak-packages
    Function: pkg.latest
      Result: True
     Comment: All packages are up-to-date (flatpak, flatpak-builder).
     Started: 08:55:07.577813
    Duration: 4815.281 ms
     Changes:   
----------
          ID: flatpak-flathub
    Function: cmd.run
        Name: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
      Result: True
     Comment: State was not run because none of the onchanges reqs changed
     Started: 08:55:12.393285
    Duration: 0.002 ms
     Changes:   
----------
          ID: skripti
    Function: cmd.script
      Result: False
     Comment: Command 'skripti' run
     Started: 08:55:12.393372
    Duration: 957.111 ms
     Changes:   
              ----------
              pid:
                  20254
              retcode:
                  1
              stderr:
                  error: No remote refs found for ?app/com.discordapp.Discord/x86_64/stable?
              stdout:
                  Looking for matches?

Summary for Fedora_minion
------------
Succeeded: 3 (changed=1)
Failed:    1
------------
Total states run:     4
Total run time:   6.200 s
^[[A^[[B^[[BManjaro_minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 08:55:07.581011
    Duration: 36.852 ms
     Changes:   
----------
          ID: flatpak-packages
    Function: pkg.latest
      Result: True
     Comment: All packages are up-to-date (flatpak, flatpak-builder).
     Started: 08:55:07.619028
    Duration: 458.019 ms
     Changes:   
----------
          ID: flatpak-flathub
    Function: cmd.run
        Name: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
      Result: True
     Comment: State was not run because none of the onchanges reqs changed
     Started: 08:55:08.077289
    Duration: 0.003 ms
     Changes:   
----------
          ID: skripti
    Function: cmd.script
      Result: True
     Comment: Command 'skripti' run
     Started: 08:55:08.077400
    Duration: 39378.19 ms
     Changes:   
              ----------
              pid:
                  2117
              retcode:
                  0
              stderr:
                  Skipping: com.discordapp.Discord/x86_64/stable is already installed
              stdout:
                  Looking for matches?
                  Required runtime for com.visualstudio.code/x86_64/stable (runtime/org.freedesktop.Sdk/x86_64/22.08) found in remote flathub
                  
                  com.microsoft.Teams permissions:
                      ipc	network	pcsc	pulseaudio
                      x11	devices	file access [1]	dbus access [2]
                      tags [3]
                  
                      [1] xdg-download
                      [2] org.freedesktop.Notifications, org.freedesktop.secrets,
                          org.gnome.SessionManager, org.kde.StatusNotifierWatcher
                      [3] proprietary
                  
                  com.visualstudio.code permissions:
                      ipc	network	pulseaudio	ssh-auth
                      x11	devices	devel	file access [1]
                      dbus access [2]	system dbus access [3]	tags [4]
                  
                      [1] host, xdg-config/gtk-3.0, xdg-config/kdeglobals:ro
                      [2] com.canonical.AppMenu.Registrar, com.canonical.AppMenu.Registrar.*,
                          org.freedesktop.Flatpak, org.freedesktop.Notifications,
                          org.freedesktop.secrets
                      [3] org.freedesktop.login1
                      [4] proprietary
                  
                  
                   1.	   	com.microsoft.Teams	stable	i	flathub	< 87.0?MB
                   2.	   	org.freedesktop.Sdk.Locale	22.08	i	flathub	< 338.9?MB (partial)
                   3.	   	org.freedesktop.Sdk	22.08	i	flathub	< 512.7?MB
                   4.	   	com.visualstudio.code	stable	i	flathub	< 101.8?MB
                  
                  
                  Installing 1/4?
                  Installing 1/4?                        0%  0 bytes/s
                  Installing 1/4? ??                     9%
                  Installing 1/4? ????????????          58%  47.2?MB/s
                  Installing 1/4? ???????????????????? 100%  86.5?MB/s
                  Installing 2/4?
                  Installing 2/4?                        0%  0 bytes/s
                  Installing 2/4? ???????????????????? 100%
                  Installing 3/4?
                  Installing 3/4?                        0%  0 bytes/s
                  Installing 3/4? ??                    10%
                  Installing 3/4? ???                   13%  29.8?MB/s
                  Installing 3/4? ????                  16%  19.8?MB/s
                  Installing 3/4? ????                  18%  23.4?MB/s
                  Installing 3/4? ????                  20%  18.1?MB/s
                  Installing 3/4? ?????                 22%  21.0?MB/s
                  Installing 3/4? ?????                 24%  18.0?MB/s  00:12
                  Installing 3/4? ??????                27%  20.0?MB/s  00:10
                  Installing 3/4? ???????               31%  21.9?MB/s  00:08
                  Installing 3/4? ???????               31%  19.1?MB/s  00:11
                  Installing 3/4? ???????               31%  16.0?MB/s  00:13
                  Installing 3/4? ???????               33%  16.0?MB/s  00:12
                  Installing 3/4? ???????               33%  16.1?MB/s  00:12
                  Installing 3/4? ???????               35%  15.0?MB/s  00:13
                  Installing 3/4? ???????               35%  14.1?MB/s  00:14
                  Installing 3/4? ????????              37%  15.1?MB/s  00:13
                  Installing 3/4? ????????              39%  14.3?MB/s  00:14
                  Installing 3/4? ?????????             41%  15.2?MB/s  00:12
                  Installing 3/4? ?????????             44%  14.5?MB/s  00:12
                  Installing 3/4? ??????????            46%  15.3?MB/s  00:11
                  Installing 3/4? ??????????            48%  16.0?MB/s  00:10
                  Installing 3/4? ???????????           51%  15.2?MB/s  00:10
                  Installing 3/4? ???????????           53%  15.7?MB/s  00:09
                  Installing 3/4? ????????????          57%  17.1?MB/s  00:08
                  Installing 3/4? ?????????????         61%  17.0?MB/s  00:07
                  Installing 3/4? ?????????????         63%  17.6?MB/s  00:07
                  Installing 3/4? ?????????????         64%  17.8?MB/s  00:06
                  Installing 3/4? ?????????????         64%  18.0?MB/s  00:06
                  Installing 3/4? ??????????????        67%  17.3?MB/s  00:06
                  Installing 3/4? ??????????????        69%  18.1?MB/s  00:05
                  Installing 3/4? ??????????????        70%  18.3?MB/s  00:05
                  Installing 3/4? ???????????????       71%  17.4?MB/s  00:05
                  Installing 3/4? ???????????????       74%  18.0?MB/s  00:04
                  Installing 3/4? ????????????????      76%  19.3?MB/s  00:04
                  Installing 3/4? ????????????????      80%  18.3?MB/s  00:03
                  Installing 3/4? ?????????????????     82%  18.9?MB/s  00:03
                  Installing 3/4? ?????????????????     85%  19.7?MB/s  00:02
                  Installing 3/4? ??????????????????    88%  20.3?MB/s  00:02
                  Installing 3/4? ??????????????????    90%  19.6?MB/s  00:01
                  Installing 3/4? ???????????????????   92%  20.1?MB/s  00:01
                  Installing 3/4? ???????????????????   93%  20.3?MB/s  00:01
                  Installing 3/4? ???????????????????   95%  20.2?MB/s  00:00
                  Installing 3/4? ????????????????????  98%  20.2?MB/s  00:00
                  Installing 3/4? ????????????????????  99%  20.4?MB/s  00:00
                  Installing 3/4? ???????????????????? 100%  19.6?MB/s  00:00
                  Installing 3/4? ???????????????????? 100%  20.4?MB/s  00:00
                  Installing 3/4? ???????????????????? 100%  21.0?MB/s  00:00
                  Installing 3/4? ???????????????????? 100%  21.6?MB/s  00:00
                  Installing 3/4? ???????????????????? 100%  20.5?MB/s  00:00
                  Installing 4/4?
                  Installing 4/4?                        0%  0 bytes/s
                  Installing 4/4? ??                    10%
                  Installing 4/4? ???                   12%
                  Installing 4/4? ?????????????         65%  62.9?MB/s
                  Installing 4/4? ???????????????????? 100%  102.4?MB/s
                  Installation complete.

Summary for Manjaro_minion
------------
Succeeded: 4 (changed=1)
Failed:    0
------------
Total states run:     4
Total run time:  39.873 s
ERROR: Minions returned with non-zero exit code
```
Manjarolla kaikki toimi niin kuin pitikin, mutta fedora ei löydä `refs`.   
Kokeilin lisätä flatpak repositoryn manuaalisesti suoraan fedoralle jolloin Fedora pyytää salasanaa.  
kokeilin liittää flatpakin manuaalisesti fedoralle, jonka jälkeen salt komentokin toimi.

---
Asensin uuden fedora minionin, sillä avun kanssa sain uuden tavan toteuttaa [flatpakin asennus](https://viruta.org/automating-home-network-with-salt.html), jolloin en myöskään tarvitse erillistä scriptiä.
```bash
flatpak-flathub:
  cmd.run:
    - name: sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

{% set flatpaks = [
	'com.discordapp.Discord',
	'com.visualstudio.code',
	'com.microsoft.Teams',
] %}

install-flatpaks:
  cmd.run:
    - name: flatpak install --or-update --assumeyes {{ flatpaks | join (' ') }}
    - require:
      - 'flatpak-flathub'
```

Uuden minioni kanssa saatiin
```bash
master@master:~$ sudo salt minion state.apply ohjelmat

minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: The following packages were installed/updated: vim-enhanced
     Started: 09:56:45.952683
    Duration: 14761.746 ms
     Changes:   
              ----------
              gpm-libs:
                  ----------
                  new:
                      1.20.7-41.fc37
                  old:
              vim-common:
                  ----------
                  new:
                      2:9.0.1006-1.fc37
                  old:
              vim-enhanced:
                  ----------
                  new:
                      2:9.0.1006-1.fc37
                  old:
              vim-filesystem:
                  ----------
                  new:
                      2:9.0.1006-1.fc37
                  old:
----------
          ID: flatpak-flathub
    Function: cmd.run
        Name: sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
      Result: True
     Comment: Command "sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo" run
     Started: 09:57:00.716404
    Duration: 823.989 ms
     Changes:   
              ----------
              pid:
                  2779
              retcode:
                  0
              stderr:
              stdout:
----------
          ID: install-flatpaks
    Function: cmd.run
        Name: flatpak install --or-update --assumeyes com.discordapp.Discord com.visualstudio.code com.microsoft.Teams
      Result: False
     Comment: Command "flatpak install --or-update --assumeyes com.discordapp.Discord com.visualstudio.code com.microsoft.Teams" run
     Started: 09:57:01.540698
    Duration: 10104.647 ms
     Changes:   
              ----------
              pid:
                  2834
              retcode:
                  1
              stderr:
              stdout:
                  Looking for matches?
                  Required runtime for com.microsoft.Teams/x86_64/stable (runtime/org.freedesktop.Platform/x86_64/22.08) found in remote flathub
                  Required runtime for com.visualstudio.code/x86_64/stable (runtime/org.freedesktop.Sdk/x86_64/22.08) found in remote flathub
                  
                  com.microsoft.Teams permissions:
                      ipc	network	pcsc	pulseaudio
                      x11	devices	file access [1]	dbus access [2]
                      tags [3]
                  
                      [1] xdg-download
                      [2] org.freedesktop.Notifications, org.freedesktop.secrets,
                          org.gnome.SessionManager, org.kde.StatusNotifierWatcher
                      [3] proprietary
                  
                  com.discordapp.Discord permissions:
                      ipc	network	pulseaudio	x11
                      devices	file access [1]	dbus access [2]	tags [3]
                  
                      [1] xdg-download, xdg-pictures:ro, xdg-videos:ro
                      [2] com.canonical.AppMenu.Registrar, com.canonical.Unity.LauncherEntry,
                          com.canonical.indicator.application, org.freedesktop.Notifications,
                          org.kde.StatusNotifierWatcher
                      [3] proprietary
                  
                  com.visualstudio.code permissions:
                      ipc	network	pulseaudio	ssh-auth
                      x11	devices	devel	file access [1]
                      dbus access [2]	system dbus access [3]	tags [4]
                  
                      [1] host, xdg-config/gtk-3.0, xdg-config/kdeglobals:ro
                      [2] com.canonical.AppMenu.Registrar, com.canonical.AppMenu.Registrar.*,
                          org.freedesktop.Flatpak, org.freedesktop.Notifications,
                          org.freedesktop.secrets
                      [3] org.freedesktop.login1
                      [4] proprietary
                  
                  
                   1.	   	org.freedesktop.Platform.GL.default	22.08	i	flathub	< 132.3?MB
                   2.	   	org.freedesktop.Platform.GL.default	22.08-extra	i	flathub	< 132.3?MB
                   3.	   	org.freedesktop.Platform.Locale	22.08	i	flathub	< 333.1?MB (partial)
                   4.	   	org.freedesktop.Platform.openh264	2.2.0	i	flathub	< 944.3?kB
                   5.	   	org.freedesktop.Platform	22.08	i	flathub	< 214.4?MB
                   6.	   	com.microsoft.Teams	stable	i	flathub	< 87.0?MB
                   7.	   	com.discordapp.Discord	stable	i	flathub	< 81.9?MB
                   8.	   	org.freedesktop.Sdk.Locale	22.08	i	flathub	< 338.9?MB (partial)
                   9.	   	org.freedesktop.Sdk	22.08	i	flathub	< 512.7?MB
                  10.	   	com.visualstudio.code	stable	i	flathub	< 101.8?MB
                  
                  
                  Installing 1/10?
                  Installing 1/10?                        0%  0 bytes/s
                  Installing 1/10? ???                   15%
                  Installing 1/10? ??????                29%
                  Installing 1/10? ?????????             42%
                  Installing 1/10? ????????????          59%  74.4?MB/s
                  Installing 1/10? ???????????????       72%  96.1?MB/s
                  Installing 1/10? ??????????????????    90%  119.1?MB/s
                  Installing 1/10? ????????????????????  98%  66.1?MB/s
                  Installing 1/10? ???????????????????? 100%  66.1?MB/s
                  Installing 2/10?
                  Installing 2/10?                        0%  0 bytes/s
                  Installing 2/10? ????????????????      79%
                  Installing 2/10? ???????????????????? 100%
                  Installing 3/10?
                  Installing 3/10?                        0%  0 bytes/s
                  Installing 3/10? ???????????????????? 100%
                  Installing 4/10?
                  Installing 4/10?                        0%  0 bytes/s
                  Installing 4/10? ??????????????        70%
                  Installing 4/10? ???????????????????? 100%
                  Installing 5/10?
                  Installing 5/10?                        0%  0 bytes/s
                  Installing 5/10? ???                   12%
                  Installing 5/10? ?????                 23%
                  Installing 5/10? ?????                 25%  17.7?MB/s
                  Installing 5/10? ??????                30%  13.9?MB/s
                  Installing 5/10? ???????               32%  14.9?MB/s
                  ?[?25h

Summary for minion
------------
Succeeded: 2 (changed=3)
Failed:    1
------------
Total states run:     3
Total run time:  25.690 s
```
Discordin asennus antaa erroria, mutta se on todennäköisesti väärällä nimella, muttetaan se:
```bash
flatpak-flathub:
  cmd.run:
    - name: sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

{% set flatpaks = [
	'app/com.discordapp.Discord/x86_64/stable',
	'com.visualstudio.code',
	'com.microsoft.Teams',
] %}

install-flatpaks:
  cmd.run:
    - name: flatpak install --or-update --assumeyes {{ flatpaks | join (' ') }}
    - require:
      - 'flatpak-flathub'
```
Ajoin salt komennon uudestaan
```bash
master@master:~$ sudo salt minion state.apply ohjelmat
minion:
----------
          ID: tarvitut_sovellukset
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 10:02:34.609251
    Duration: 709.696 ms
     Changes:   
----------
          ID: flatpak-flathub
    Function: cmd.run
        Name: sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
      Result: True
     Comment: Command "sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo" run
     Started: 10:02:35.319950
    Duration: 981.446 ms
     Changes:   
              ----------
              pid:
                  3098
              retcode:
                  0
              stderr:
              stdout:
----------
          ID: install-flatpaks
    Function: cmd.run
        Name: flatpak install --or-update --assumeyes app/com.discordapp.Discord/x86_64/stable com.visualstudio.code com.microsoft.Teams
      Result: True
     Comment: Command "flatpak install --or-update --assumeyes app/com.discordapp.Discord/x86_64/stable com.visualstudio.code com.microsoft.Teams" run
     Started: 10:02:36.301880
    Duration: 57318.653 ms
     Changes:   
              ----------
              pid:
                  3148
              retcode:
                  0
              stderr:
              stdout:
                  Looking for matches?
                  Required runtime for com.microsoft.Teams/x86_64/stable (runtime/org.freedesktop.Platform/x86_64/22.08) found in remote flathub
                  Required runtime for com.visualstudio.code/x86_64/stable (runtime/org.freedesktop.Sdk/x86_64/22.08) found in remote flathub
                  
                  com.microsoft.Teams permissions:
                      ipc	network	pcsc	pulseaudio
                      x11	devices	file access [1]	dbus access [2]
                      tags [3]
                  
                      [1] xdg-download
                      [2] org.freedesktop.Notifications, org.freedesktop.secrets,
                          org.gnome.SessionManager, org.kde.StatusNotifierWatcher
                      [3] proprietary
                  
                  com.discordapp.Discord permissions:
                      ipc	network	pulseaudio	x11
                      devices	file access [1]	dbus access [2]	tags [3]
                  
                      [1] xdg-download, xdg-pictures:ro, xdg-videos:ro
                      [2] com.canonical.AppMenu.Registrar, com.canonical.Unity.LauncherEntry,
                          com.canonical.indicator.application, org.freedesktop.Notifications,
                          org.kde.StatusNotifierWatcher
                      [3] proprietary
                  
                  com.visualstudio.code permissions:
                      ipc	network	pulseaudio	ssh-auth
                      x11	devices	devel	file access [1]
                      dbus access [2]	system dbus access [3]	tags [4]
                  
                      [1] host, xdg-config/gtk-3.0, xdg-config/kdeglobals:ro
                      [2] com.canonical.AppMenu.Registrar, com.canonical.AppMenu.Registrar.*,
                          org.freedesktop.Flatpak, org.freedesktop.Notifications,
                          org.freedesktop.secrets
                      [3] org.freedesktop.login1
                      [4] proprietary
                  
                  
                   1.	   	org.freedesktop.Platform	22.08	i	flathub	< 214.4?MB
                   2.	   	com.microsoft.Teams	stable	i	flathub	< 87.0?MB
                   3.	   	com.discordapp.Discord	stable	i	flathub	< 81.9?MB
                   4.	   	org.freedesktop.Sdk.Locale	22.08	i	flathub	< 338.9?MB (partial)
                   5.	   	org.freedesktop.Sdk	22.08	i	flathub	< 512.7?MB
                   6.	   	com.visualstudio.code	stable	i	flathub	< 101.8?MB
                  
                  
                  Installing 1/6?
                  Installing 1/6?                        0%  0 bytes/s
                  Installing 1/6? ????                  16%
                  Installing 1/6? ?????                 23%  23.8?MB/s
                  Installing 1/6? ??????                30%  16.3?MB/s
                  Installing 1/6? ????????              37%  19.6?MB/s
                  Installing 1/6? ????????              37%  19.9?MB/s
                  Installing 1/6? ?????????             41%  15.1?MB/s
                  Installing 1/6? ?????????             45%  13.3?MB/s  00:04
                  Installing 1/6? ????????????          57%  17.7?MB/s  00:03
                  Installing 1/6? ???????????????       75%  23.5?MB/s  00:01
                  Installing 1/6? ???????????????????   92%  23.3?MB/s  00:00
                  Installing 1/6? ???????????????????? 100%  25.0?MB/s  00:00
                  Installing 2/6?
                  Installing 2/6?                        0%  0 bytes/s
                  Installing 2/6? ??                    10%
                  Installing 2/6? ??                    10%
                  Installing 2/6? ????????????          59%  48.6?MB/s
                  Installing 2/6? ???????????????????? 100%  87.6?MB/s
                  Installing 3/6?
                  Installing 3/6?                        0%  0 bytes/s
                  Installing 3/6? ?????                 25%
                  Installing 3/6? ????????              38%
                  Installing 3/6? ??????????????        69%
                  Installing 3/6? ???????????????????   94%  75.8?MB/s
                  Installing 3/6? ???????????????????? 100%  82.4?MB/s
                  Installing 3/6? ???????????????????? 100%  41.2?MB/s
                  Installing 4/6?
                  Installing 4/6?                        0%  0 bytes/s
                  Installing 4/6? ???????????????????? 100%
                  Installing 5/6?
                  Installing 5/6?                        0%  0 bytes/s
                  Installing 5/6? ??                    10%
                  Installing 5/6? ????                  18%  16.3?MB/s
                  Installing 5/6? ????                  20%  14.2?MB/s  00:16
                  Installing 5/6? ?????                 22%  16.1?MB/s  00:14
                  Installing 5/6? ?????                 24%  14.4?MB/s  00:15
                  Installing 5/6? ??????                26%  13.3?MB/s  00:17
                  Installing 5/6? ???????               31%  13.6?MB/s  00:15
                  Installing 5/6? ???????               34%  14.6?MB/s  00:13
                  Installing 5/6? ???????               35%  13.0?MB/s  00:14
                  Installing 5/6? ???????               35%  14.1?MB/s  00:14
                  Installing 5/6? ????????              37%  13.4?MB/s  00:15
                  Installing 5/6? ????????              39%  14.2?MB/s  00:14
                  Installing 5/6? ?????????             41%  13.5?MB/s  00:14
                  Installing 5/6? ?????????             43%  14.4?MB/s  00:13
                  Installing 5/6? ?????????             45%  13.8?MB/s  00:13
                  Installing 5/6? ??????????            49%  14.4?MB/s  00:11
                  Installing 5/6? ???????????           54%  16.1?MB/s  00:09
                  Installing 5/6? ????????????          59%  16.4?MB/s  00:08
                  Installing 5/6? ?????????????         62%  18.1?MB/s  00:07
                  Installing 5/6? ??????????????        69%  19.7?MB/s  00:05
                  Installing 5/6? ???????????????       74%  19.4?MB/s  00:04
                  Installing 5/6? ????????????????      76%  20.1?MB/s  00:04
                  Installing 5/6? ?????????????????     81%  21.4?MB/s  00:03
                  Installing 5/6? ?????????????????     84%  20.8?MB/s  00:02
                  Installing 5/6? ??????????????????    88%  21.8?MB/s  00:01
                  Installing 5/6? ???????????????????   91%  22.7?MB/s  00:01
                  Installing 5/6? ???????????????????   95%  22.2?MB/s  00:00
                  Installing 5/6? ????????????????????  97%  23.4?MB/s  00:00
                  Installing 5/6? ???????????????????? 100%  24.1?MB/s  00:00
                  Installing 5/6? ???????????????????? 100%  24.2?MB/s  00:00
                  Installing 6/6?
                  Installing 6/6?                        0%  0 bytes/s
                  Installing 6/6? ??                    10%
                  Installing 6/6? ???                   11%
                  Installing 6/6? ?????????????         65%  62.9?MB/s
                  Installing 6/6? ???????????????????? 100%  102.0?MB/s
                  Installation complete.

Summary for minion
------------
Succeeded: 3 (changed=2)
Failed:    0
------------
Total states run:     3
Total run time:  59.010 s

```
Kaikki toimii

---
Tarkistetaan, että sovellukset asentui:  
Discord:  
![Discord](https://user-images.githubusercontent.com/92360351/207268829-4f2666a4-0016-42f0-b8e4-e7cbb4d88de9.png)  
Teams:  
![Teams](https://user-images.githubusercontent.com/92360351/207268879-a2204a82-3e23-4287-a4cb-7c6fb91d4500.png)  
Visual Studio Code:  
![VSCode](https://user-images.githubusercontent.com/92360351/207268954-eac78ea5-3f54-4402-89c2-17fdcce74f9d.png)  
Sovellukset löytyvät asennettuina

---
Saltin state.apply ohjelmat init.sls näyttää tältä:
```bash
tarvitut_sovellukset:
  pkg.installed:
    - pkgs:
      {% if grains['os_family'] == 'RedHat' %}
      - vim-enhanced
      {% if grains['os_family'] == 'Arch' %}
      - vim
      {% if grains['os_family'] == 'Debian' %}
      - vim
      {% endif %}
      - micro
      - git
      - openssh

flatpak-flathub:
  cmd.run:
    - name: sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

{% set flatpaks = [
	'app/com.discordapp.Discord/x86_64/stable',
	'com.visualstudio.code',
	'com.microsoft.Teams',
] %}

install-flatpaks:
  cmd.run:
    - name: flatpak install --or-update --assumeyes {{ flatpaks | join (' ') }}
    - require:
      - 'flatpak-flathub'
```
Tämän avulla uusille minioneille voi asentaa kaikki tarvittavat ohjelmat ajamalla salt state.apply komento.

---
### Lähteet
[Tero Karvisen Palvelinhallinta kurssi](https://terokarvinen.com/2022/palvelinten-hallinta-2022p2/)  
[Automating my home network with Salt](https://viruta.org/automating-home-network-with-salt.html)  
[SaltStack](https://docs.saltproject.io/en/latest/topics/tutorials/states_pt3.html)  
[Antti](https://github.com/therealhalonen)
