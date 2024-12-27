# Linux Tuning

## Oefening rond Linux Tuning

---

## Install Dependencies

### Doel
Zorg ervoor dat de benodigde tools en libraries worden geïnstalleerd om de oefeningen in deze handleiding uit te voeren.

### Script
```bash
sudo apt update 
sudo apt install gcc g++ build-essential acct hdparm lsof iproute2 cpulimit
```

---

## 1. Bestandskopie meten met `time`

### Doel
Meet de tijd die nodig is om een groot bestand te kopiëren, zowel met als zonder cache wissen.

### Script
```bash
#!/bin/bash

# Maak een groot bestand genaamd "BigFile"
echo "Aanmaken GROOT bestand..."
mkdir -p /home/bjornclaes/Desktop/scripts/temp/
bigfile="/home/bjornclaes/Desktop/scripts/temp/BigFile"
dd if=/dev/zero of=$bigfile bs=1M count=5000 status=none

# Output directories voor de kopieën
output1="/home/bjornclaes/Desktop/scripts/temp/BigFile_copy1"
output2="/home/bjornclaes/Desktop/scripts/temp/BigFile_copy2"

# Functie om tijdsmetingen op te nemen
function get_time {
    time_output=$( (time cp "$1" "$2") 2>&1 )
    real_time=$(echo "$time_output" | grep real | awk '{print $2}')
    user_time=$(echo "$time_output" | grep user | awk '{print $2}')
    sys_time=$(echo "$time_output" | grep sys | awk '{print $2}')
    echo "$real_time $user_time $sys_time"
}

# Tijd opnemen voor kopieën zonder cache wissen
echo "Uitvoeren van kopieën zonder cache wissen..."
times1_no_cache=$(get_time "$bigfile" "$output1")
times2_no_cache=$(get_time "$bigfile" "$output2")

# Cache wissen
echo "Wissen van de cache..."
sudo sync; sudo bash -c "echo 3 > /proc/sys/vm/drop_caches"

# Tijd opnemen voor kopieën met cache wissen
echo "Uitvoeren van kopieën met cache wissen..."
times1_with_cache=$(get_time "$bigfile" "$output1")
sudo sync; sudo bash -c "echo 3 > /proc/sys/vm/drop_caches"
times2_with_cache=$(get_time "$bigfile" "$output2")

# Resultaten weergeven
echo -e "\nVergelijking van de kopietijden ZONDER cache wissen:"
echo -e "Run\tReal Time\tUser Time\tSys Time"
echo -e "1\t$(echo "$times1_no_cache" | awk '{print $1}')\t$(echo "$times1_no_cache" | awk '{print $2}')\t$(echo "$times1_no_cache" | awk '{print $3}')"
echo -e "2\t$(echo "$times2_no_cache" | awk '{print $1}')\t$(echo "$times2_no_cache" | awk '{print $2}')\t$(echo "$times2_no_cache" | awk '{print $3}')"

echo -e "\nVergelijking van de kopietijden MET cache wissen:"
echo -e "Run\tReal Time\tUser Time\tSys Time"
echo -e "1\t$(echo "$times1_with_cache" | awk '{print $1}')\t$(echo "$times1_with_cache" | awk '{print $2}')\t$(echo "$times1_with_cache" | awk '{print $3}')"
echo -e "2\t$(echo "$times2_with_cache" | awk '{print $1}')\t$(echo "$times2_with_cache" | awk '{print $2}')\t$(echo "$times2_with_cache" | awk '{print $3}')"

# Opruimen
echo "Opruimen aangemaakte bestanden..."
rm -f "$bigfile" "$output1" "$output2"
```

### Output
Voorbeeld uitvoer wordt gegenereerd door het script.

---

## 2. Schakel Process Accounting aan

### Doel
Schakel process accounting in om systeemgebruik en processen te analyseren.

### Script
```bash
#!/bin/bash

# Controleer of het acct-pakket is geïnstalleerd
if ! dpkg -l | grep -q acct; then
  echo "Process accounting (acct) is niet geïnstalleerd. Installeren..."
  sudo apt update -y && sudo apt install -y acct
fi

# Schakel process accounting in
echo "Process accounting inschakelen..."
sudo systemctl start acct
sudo systemctl enable acct

# Aanmaken van de map voor accounting logs
mkdir -p /var/log/account

# Inschakelen van accounting
sudo accton /var/log/account/pact

# Controleer de status
if systemctl is-active --quiet acct; then
  echo "Process accounting is actief."
else
  echo "Fout bij het inschakelen van process accounting."
  exit 1
fi

# Bekijk de accountinggegevens
sa

# Toon de meest gebruikte processen
echo "De meest gebruikte processen zijn:"
sudo sa -m | head -n 10

# Uitschakelen van accounting
accton off
```

### Output
Voorbeeld output wordt gegenereerd door het script.

---

## 3. Gebruik vmstat

### Doel
Analyseer CPU- en geheugenactiviteit over een bepaalde tijdsperiode.

### Command
```bash
vmstat 2 10
```

### Output
```text
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0    524 235084 130576 6202568    0    0   806  2279  250    1  1  1 98  0
...
```

---

## 4. Bekijk Kernel Cache Gebruik

### Doel
Controleer welke kernelcache het meeste geheugen gebruikt.

### Script
```bash
#!/bin/bash

# Controleer de cachegegevens in /proc/meminfo
echo "Kernel cache informatie ophalen..."
cache_info=$(grep -E "^(Slab|PageTables|Bounce|SwapCached|Dirty|Writeback|Mapped|AnonPages):" /proc/meminfo)

# Sla de gegevens op in een bestand
output_file="/home/bjornclaes/Desktop/scripts/temp/4_kernel_cache_info.txt"
echo "$cache_info" > "$output_file"

# Print de gevonden cache-informatie
echo "Kernel cache gegevens zijn opgeslagen in $output_file:"
cat "$output_file"

# Vind de grootste cache
max_cache=$(awk '{print $1, $2, $3}' "$output_file" | sort -k2 -n -r | head -1)

echo "De cache die het meeste geheugen gebruikt is:"
echo "$max_cache"

read -p "Druk op enter om de gegevens met slabtop te bekijken"

echo "------"
echo "Met SlabTop"
sudo slabtop
```

### Output
Voorbeeld output wordt gegenereerd door het script.

---

## 5. Controleer IPC-gebruik

### Doel
Analyseer welke processen IPC-geheugen (Inter-Process Communication) gebruiken.

### Script
```bash
#!/bin/bash
echo "Controleer IPC-gebruikende processen:"
ipcs -p
```

### Output
```text
Controleer IPC-gebruikende processen:

------ Message Queues PIDs --------
msqid      owner      lspid      lrpid     

------ Shared Memory Creator/Last-op PIDs --------
shmid      owner      cpid       lpid      
```

**Conclusie**: Geen IPC-processen aanwezig.

---

## 6. Analyseer vi-gebruik

### Doel
Bekijk de afhankelijkheden (`ldd`) en het geheugenverbruik (`pmap`) van de vi-editor.

### a) Gebruik ldd
```bash
ldd $(which vi)
```

### b) Gebruik pmap
```bash
#!/bin/bash

# Controleer of vi draait
vi_pid=$(pgrep -x vi)

if [ -z "$vi_pid" ]; then
    echo "vi draait niet. Starten in een nieuwe terminal..."
    gnome-terminal -- bash -c "vi; exec bash"

    # Wacht tot vi start
    while true; do
        vi_pid=$(pgrep -x vi)
        if [ -n "$vi_pid"; then
            break
        fi
        sleep 0.1
    done
    echo "vi gestart met PID: $vi_pid"
else
    echo "vi draait al met PID: $vi_pid"
fi

# Gebruik pmap
pmap_output=$(pmap $vi_pid)
echo "$pmap_output"

# Bereken het geheugen
total_kb=$(echo "$pmap_output" | grep -i total | awk '{print $2}' | sed 's/K//')
total_mb=$(echo "scale=2; $total_kb / 1024" | bc)
total_gb=$(echo "scale=2; $total_mb / 1024" | bc)

echo "Totaal geheugen:"
echo "$total_kb KB"
echo "$total_mb MB"
echo "$total_gb GB"
```

### Verschil tussen `pmap` en `ldd`

| **Aspect**                | **`pmap`**                                            | **`ldd`**                                       |
|-------------------------- |-----------------------------------------------------  |------------------------------------------------ |
| **Wat het toont**         | Geheugenkaarten van een proces (memory mappings).     | Afhankelijkheden (shared libraries) van een uitvoerbaar bestand. |
| **Real-time informatie**  | Ja, laat zien welke bibliotheken momenteel in gebruik zijn. | Nee, toont alleen statische afhankelijkheden.  |
| **Focus**                 | Hoe geheugen wordt gebruikt door het proces.          | Welke bibliotheken nodig zijn om te draaien.    |
| **Gebruiksscenario**      | Diagnose van geheugenverbruik en geladen onderdelen.  | Controle van vereiste bibliotheken.             |
| **Commando**              | `pmap <PID>`                                          | `ldd <executable>`                              |

### Samenvatting
- **`pmap`**: Real-time geheugenanalyse van een draaiend proces.
- **`ldd`**: Statische afhankelijkheidscontrole van een bestand.

---

## 7. Test Schijfprestaties met hdparm

### Doel
Analyseer en test de prestaties van je harde schijf, inclusief beschikbare schijfblokken, configuratieparameters en leessnelheden.

### a) Bekijk beschikbare schijfblokken

#### Script
```bash
lsblk -o NAME,TYPE,SIZE | grep -E 'disk|part'
```
#### Uitleg
De optie `-d` toont alleen de schijven met de partitielagen

#### Output
```text 
sda    disk    25G
├─sda1 part     1M
└─sda2 part    25G  
```

### b) Bekijk schijfparameters

#### Script
```bash
sudo hdparm /dev/sda
```
#### Uitleg
Toont de configuratieparameters van de schijf, zoals multiblock I/O, readahead-instellingen en geometrie.

#### Output
```text
/dev/sda:
 multcount     = 128 (on)
 IO_support    =  1 (32-bit)
 readonly      =  0 (off)
 readahead     = 256 (on)
 geometry      = 3263/255/63, sectors = 52428800, start = 0
```

### c) Meet schijfprestaties

#### Script
```bash
sudo hdparm -tT /dev/sda
```
#### Uitleg
- `-t`: Meet de directe (buffered) schijfsnelheid.
- `-T`: Meet de cache-leessnelheid.

#### Output
```text
/dev/sda:
 Timing cached reads:   34336 MB in  1.99 seconds = 17211.40 MB/sec
 Timing buffered disk reads: 2746 MB in  3.00 seconds = 914.83 MB/sec
```

## 8. Controleer netwerkgebruik met lsof

### Doel
Bekijk welke processen het netwerk gebruiken, zowel voor als na het openen van Firefox.

### a) Controleer netwerkgebruik voor het openen van Firefox


#### Uitleg
Het script opent Firefox, wacht 5 seconden en controleert opnieuw welke processen het netwerk gebruiken.
De controle op leegte (grep -q) voorkomt een lege uitvoer.

#### Script
```bash
#!/bin/bash

# Controleer netwerkgebruik
if ! lsof -i4 | grep -q '[a-zA-Z0-9]'; then
    echo "Niets om weer te geven. Geen processen gebruiken momenteel het netwerk."
else
    lsof -i4
fi
```

#### Output
```text
Niets om weer te geven. Geen processen gebruiken momenteel het netwerk.
```
OF
```text
COMMAND   PID       USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker  11292 bjornclaes   57u  IPv4 234890      0t0  TCP 172.17.0.1:2375->172.18.0.2:5000 (ESTABLISHED)
```

### b) Open Firefox en controleer netwerkgebruik opnieuw
#### Script
```bash
#!/bin/bash

# Controleer netwerkgebruik
if ! lsof -i4 | grep -q '[a-zA-Z0-9]'; then
    echo "Niets om weer te geven. Geen processen gebruiken momenteel het netwerk."
else
    lsof -i4
fi
```

#### Output
```text
Niets om weer te geven. Geen processen gebruiken momenteel het netwerk.
```
OF
```text
COMMAND   PID       USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker  11292 bjornclaes   57u  IPv4 234890      0t0  TCP 172.17.0.1:2375->172.18.0.2:5000 (ESTABLISHED)
firefox 11292 bjornclaes   43u  IPv4 210563      0t0  UDP localhost:51957->_localdnsstub:domain 
firefox 11292 bjornclaes   57u  IPv4 209686      0t0  TCP bc-student:46256->209.100.149.34.bc.googleusercontent.com:https (ESTABLISHED)
firefox 11292 bjornclaes   76u  IPv4 209685      0t0  TCP bc-student:36258->216.72.190.35.bc.googleusercontent.com:https (ESTABLISHED)
firefox 11292 bjornclaes   85u  IPv4 209687      0t0  TCP bc-student:46262->209.100.149.34.bc.googleusercontent.com:https (ESTABLISHED)
```



## 9. Maak een extra netwerkinterface aan en test deze

### Doel
Creëer een dummy-netwerkinterface met het adres `1.2.3.4/24`, test of deze kan worden gepingd en verwijder de interface daarna.

#### Uitleg
`ip link add dummy1 type dummy`: Maakt een dummy-interface genaamd dummy1.
`ip addr add 1.2.3.4/24 dev dummy1`: Wijs het IP-adres 1.2.3.4/24 toe aan de dummy-interface.
`ip link set dummy1 up`: Activeert de interface.
`ping -c 4 1.2.3.4`: Stuurt vier ICMP-echoverzoeken om de verbinding te testen.
`ip link delete dummy1`: Verwijdert de interface.

### Stappen
#### 1. Maak de netwerkinterface aan
Gebruik de volgende opdracht om een dummy-interface genaamd `dummy1` aan te maken:
```bash
sudo ip link add dummy1 type dummy
```
#### 1. Wijs het IP-adres toe
Voer de volgende opdracht uit om het IP-adres 1.2.3.4/24 toe te wijzen aan de interface:
```bash
sudo ip addr add 1.2.3.4/24 dev dummy1
```
#### 3. Activeer de interface
Activeer de interface met:
```bash
sudo ip link set dummy1 up
```
#### 4. Test de verbinding
Controleer of de interface bereikbaar is door te pingen:
```bash
ping -c 4 1.2.3.4
```

#### Output
```text
PING 1.2.3.4 (1.2.3.4) 56(84) bytes of data.
64 bytes from 1.2.3.4: icmp_seq=1 ttl=64 time=0.025 ms
64 bytes from 1.2.3.4: icmp_seq=2 ttl=64 time=0.026 ms
64 bytes from 1.2.3.4: icmp_seq=3 ttl=64 time=0.042 ms
64 bytes from 1.2.3.4: icmp_seq=4 ttl=64 time=0.023 ms

--- 1.2.3.4 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3071ms
rtt min/avg/max/mdev = 0.023/0.029/0.042/0.007 ms
```
#### 5. Verwijder de interface
Als je klaar bent, verwijder je de interface met:
```bash
sudo ip link delete dummy1
```

## 10. Beperk CPU-gebruik van een Oneindige Loop met cpulimit

### Doel
Maak een Shell-script met een oneindige loop en beperk het CPU-gebruik met het `cpulimit`-hulpprogramma.

### Uitleg
- **`while true`**: Creëert een oneindige loop.
- **`chmod +x loop.sh`**: Maakt het script uitvoerbaar.
- **`cpulimit -e loop.sh -l 10`**: Beperkt het CPU-gebruik van het proces `loop.sh` tot 10%.

### Conclusie
Met `cpulimit` kun je effectief het CPU-gebruik van een proces beperken, zelfs voor een oneindige loop. Dit kan nuttig zijn in situaties waarin processen een minimaal CPU-gebruik mogen hebben.

### Stappen

#### 1. Maak een Shell Script met een Oneindige Loop
Creëer een bestand genaamd `loop.sh` en voeg de volgende code toe:
```bash
#!/bin/bash

# Oneindige loop
while true; do
    echo "Looping..."
    echo "test"
    # sleep 1  # Optioneel: Pauze van 1 seconde tussen iteraties
done
```

Maak het script uitvoerbaar:
```bash
chmod +x loop.sh
```

#### 2. Installeer cpulimit
Als `cpulimit` nog niet is geïnstalleerd, installeer het met:
```bash
sudo apt-get install cpulimit
```

#### 3. Run het Script zonder cpulimit
Start het script in een terminal:
```bash
./loop.sh
```

Open in een andere terminal `top` om het CPU-gebruik te controleren:
```bash
top
```

#### Output
```text
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                           
   2200 bjorncl+  20   0 4760484 594108 209480 R  84.8   7.3  30:17.14 gnome-shell                       
  **13072 bjorncl+  20   0   19824   3704   2176 R  60.6   0.0   1:32.09 bash**                              
  13011 bjorncl+  20   0  713064  62076  48524 R  30.8   0.8   0:49.22 gnome-terminal-                   
  12645 root      20   0       0      0      0 I   9.9   0.0   0:05.74 kworker/u7:2-events_unbound       

```
Gemiddeld CPU-gebruik van het proces is rond de **60.6%**.

#### 4. Run het Script met cpulimit
Gebruik `cpulimit` om het CPU-gebruik te beperken. Start het script en beperk het CPU-gebruik tot bijvoorbeeld 10%:
```bash
cpulimit -pid <PID> -l 10
```

Controleer opnieuw in een andere terminal met `top`.

#### Output
```text
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                           
   2200 bjorncl+  20   0 4762708 596456 211876 S  35.3   7.3  32:39.05 gnome-shell                       
  **13137 bjorncl+  20   0   19824   3712   2176 T  12.0   0.0   1:27.96 bash**                              
  13011 bjorncl+  20   0  717308  65804  50716 S   7.7   0.8   1:47.56 gnome-terminal-                   
```

CPU-gebruik van het script wordt effectief beperkt tot **10%** of minder.

## 11. Maak een extra swapfile aan en gebruik deze

### Doel
Creëer een swapbestand genaamd `pagefile.sys`, gebruik deze als swapruimte, en begrijp waarom Linux meestal een swap-partitie verkiest boven een swap-bestand.

---

### Stappen

#### script
```bash
# 1. Maak een swapbestand aan van 20 MB
sudo dd if=/dev/zero of=/pagefile bs=1M count=20

# 2. Zorg dat het bestand de juiste permissies heeft
sudo chmod 600 /pagefile

# 3. Formateer het bestand als Linux Swap
sudo mkswap /pagefile

# 4. Activeer het swapbestand
sudo swapon /pagefile

# 5. Controleer het swapgebruik
swapon --show

# 6. Uitschakelen van het swapbestand als je klaar bent
sudo swapoff /pagefile
```

#### Output
```text
bjornclaes@bc-student:~$ sudo dd if=/dev/zero of=/pagefile bs=1024 count=100000
100000+0 records in
100000+0 records out
102400000 bytes (102 MB, 98 MiB) copied, 0.199072 s, 514 MB/s
bjornclaes@bc-student:~$ sudo chmod 600 /pagefile
bjornclaes@bc-student:~$ sudo mkswap /pagefile
Setting up swapspace version 1, size = 97.7 MiB (102395904 bytes)
no label, UUID=2c75ca16-7c61-4a57-8a64-4031f1bd76b1
bjornclaes@bc-student:~$ sudo swapon /pagefile
bjornclaes@bc-student:~$ swapon --show
NAME      TYPE  SIZE USED PRIO
/swap.img file    4G 116K   -2
/pagefile file 97.7M   0B   -3
bjornclaes@bc-student:~$ sudo swapoff /pagefile
```

---

### Waarom Gebruikt Linux Meestal een Swap-partitie in Plaats van een Swap-bestand?

Linux gebruikt meestal een swap-partitie in plaats van een swap-bestand vanwege de volgende redenen:

#### 1. Prestaties
- **Directe toegang**:
  - Swap-partities bieden directe toegang tot de schijf zonder tussenkomst van het bestandssysteem.
  - Dit resulteert in lagere overhead en hogere prestaties.
- **Bestandssysteem-overhead**:
  - Bij swap-bestanden moet de kernel via het bestandssysteem werken om bij de gegevens te komen, wat extra stappen toevoegt en de snelheid vermindert.

#### 2. Fragmentatie
- **Swap-bestanden**:
  - Kunnen na verloop van tijd gefragmenteerd raken, vooral als ze dynamisch worden toegevoegd of verwijderd.
  - Dit kan leiden tot tragere prestaties en inconsistente toegangstijden.
- **Swap-partities**:
  - Zijn vooraf gealloceerd en gebruiken een vaste ruimte op de schijf.
  - Fragmentatie is hierbij geen probleem, waardoor consistent hogere prestaties mogelijk zijn.

#### 3. Beheer en Stabiliteit
- **Eenvoudig beheer**:
  - Swap-partities zijn eenvoudiger te beheren en minder gevoelig voor fouten, vooral op productie- en serversystemen.
- **Robuustheid**:
  - Als het bestandssysteem beschadigd raakt, kan een swap-bestand onbruikbaar worden.
  - Een swap-partitie blijft vaak bruikbaar omdat deze onafhankelijk van het bestandssysteem functioneert.

#### 4. Betrouwbaarheid
- **Historische beperkingen**:
  - In oudere Linux-versies waren swap-bestanden minder betrouwbaar vanwege beperkingen in de implementatie.
  - Moderne Linux-versies hebben deze beperkingen grotendeels opgelost, maar de voorkeur voor partities blijft.

---

### Conclusie
- Swap-partities zijn vaak de beste keuze vanwege hun betere prestaties, stabiliteit, en eenvoud in beheer.
- Swap-bestanden kunnen echter nuttig zijn in situaties waar:
  - Het maken van een nieuwe partitie niet mogelijk is.
  - Tijdelijk extra swapruimte nodig is.

Met de bovenstaande stappen kun je eenvoudig een swap-bestand configureren en beheren. Begrijp echter dat swap-partities doorgaans de voorkeur hebben op systemen waar prestaties en stabiliteit van groot belang zijn.


## 12. Verander een kernelparameter zodat deze zoveel mogelijk swapt

### Doel
Wijzig de kernelparameter `vm.swappiness` om het systeem aan te moedigen zo veel mogelijk te swappen. Controleer vervolgens of de swapfile daadwerkelijk gebruikt wordt door enkele programma's te starten.

---

### Stappen

#### 1. Controleer de huidige swappiness-waarde
Bekijk de huidige waarde van `vm.swappiness` (standaard meestal 60):
```bash
cat /proc/sys/vm/swappiness
```

#### 2. Verhoog de swappiness-waarde
Stel de swappiness in op een hoge waarde, bijvoorbeeld 100, zodat het systeem zoveel mogelijk zal swappen:
```bash
sudo sysctl vm.swappiness=100
```
> **Opmerking**: Deze wijziging is tijdelijk en wordt gereset na een herstart. Voor een permanente wijziging moet je `/etc/sysctl.conf` bewerken en de volgende regel toevoegen:
> ```bash
> vm.swappiness=100
> ```

#### 3. Start enkele programma's om geheugen te gebruiken
Open geheugenintensieve programma's, zoals:
- Een webbrowser met veel tabbladen.
- Een teksteditor met grote bestanden.
- Virtuele machines (indien mogelijk).

#### 4. Controleer het swapgebruik
Gebruik de volgende commando's om te zien of het systeem swap gebruikt:

- **Met `free -h`**:
  Dit geeft een overzicht van het gebruikte en beschikbare RAM- en swapgeheugen:
  ```bash
  free -h
  ```

- **Met `swapon --show`**:
  Dit toont details van de actieve swapruimte:
  ```bash
  swapon --show
  ```

#### 5. Outputvoorbeelden
**Voorbeeld output van `free -h`:**
```text
              total        used        free      shared  buff/cache   available
Mem:           15Gi        14Gi       100Mi       500Mi        1Gi       200Mi
Swap:          2Gi         1Gi        1Gi
```

**Voorbeeld output van `swapon --show`:**
```text
NAME        TYPE      SIZE USED PRIO
/pagefile.sys file      20M  10M   -2
/dev/sda2   partition   2G   1G   -1
```

---

### Conclusie
Door de `vm.swappiness`-parameter te verhogen naar 100, dwing je het systeem om actief geheugen naar swapruimte te verplaatsen, zelfs als er nog beschikbaar RAM-geheugen is. Dit kan nuttig zijn voor het testen van swapconfiguraties en het evalueren van systeemprestaties onder hoge belasting.


## 13. Compileer de voorziene C- en C++-source bestanden en start de programma's

### Doel
Leer hoe je C- en C++-programma's compileert, een Apache-server lokaal draait voor `b.c`, en de programma's uitvoert. Analyseer en los eventuele fouten op.

---

### Stappen

#### 1. Bestanden Compileren
Installeer de benodigde tools voor het compileren van de bestanden:
```bash
sudo apt-get install gcc g++ build-essential
```

Compileer de C-bestanden met `gcc`:
```bash
gcc a.c -o a
gcc b.c -o b
gcc c.c -o c
gcc d.c -o d
```

Compileer het C++-bestand met `g++`:
```bash
g++ b.cpp -o b_cpp
```

> **Opmerking**: Controleer of er foutmeldingen zijn tijdens het compileren. Fix eventuele fouten voordat je verdergaat.

---

#### 2. Lokaal een Apache Server Draaien
Voor `b.c` is een lokale Apache-server nodig. Volg deze stappen:

1. Installeer Apache:
   ```bash
   sudo apt update
   sudo apt install apache2
   ```

2. Start de Apache-server:
   ```bash
   sudo systemctl start apache2
   sudo systemctl status apache2
   ```

3. Controleer of de server draait:
   - Open je webbrowser en ga naar: [http://localhost](http://localhost).
   - Je zou een standaard Apache-pagina moeten zien die aangeeft dat de server werkt.

---

#### 3. Programma's Starten
Voer de gecompileerde programma's uit in de terminal:
```bash
./a
./b
./c
./d
./b_cpp
```

---

### Problemen Identificeren
- **Bij `a`**: Controleer of er tijdens het compileren een waarschuwing (`warning`) wordt weergegeven. Noteer deze waarschuwing voor verdere analyse.
- **Bij `b.c`**: Zorg ervoor dat de Apache-server draait. Zonder een actieve server zal het programma niet correct werken.
- **Algemeen**: Controleer de uitvoer van elk programma en identificeer foutmeldingen of onverwachte resultaten.

---

### Conclusie
- Controleer de compilatie-output zorgvuldig op fouten of waarschuwingen.
- Zorg ervoor dat de lokale Apache-server correct draait voor `b.c`.
- Voer de programma's uit en analyseer de output om problemen op te lossen.