
# Linux Tuning

Meer de admin kant van Linux, wat zou er kunnen mislopen/wat loopt er mis?

## `/etc/issue`

`/etc/issue` is een tekstbestand waarin de Message of the Day (MOTD) of een banner staat die aan gebruikers wordt getoond voordat ze inloggen. Dit bericht wordt direct op het toestel weergegeven, bijvoorbeeld op een lokale console of terminal.

- **`/etc/issue`**: Rechtstreeks in de console op het toestel.
- **`/etc/issue.net`**: Gebruikt bij telnet of SSH.

## `/etc/motd`

Dit is een bericht dat in de console/terminal wordt weergegeven vlak nadat een gebruiker heeft ingelogd.

## `/etc/profile` en `/etc/bash.bashrc`

- **`/etc/profile`**: 
  - Een opstartbestand dat wordt uitgevoerd voor elke gebruiker wanneer ze een login shell openen. 
  - Geldt zowel voor interactieve (console/terminal) als niet-interactieve (SSH/telnet) sessies.
  - Het bestand bevindt zich in de `/etc`-directory, dus het is een globaal configuratiebestand.
  - De instellingen gelden voor alle gebruikers van het systeem.

- **`/etc/bash.bashrc`**: 
  - Een globaal configuratiebestand dat wordt uitgevoerd wanneer een gebruiker een interactieve, niet-login shell opent.
  - Dit bestand wordt dus uitgevoerd bij het starten van een nieuwe terminalsessie, maar niet bij inloggen.
  - Geldt ook voor alle gebruikers (`/etc`).

### `/etc/profile.d/`

Dit is een directory die verschillende scriptbestanden (.sh) bevat. Deze scripts bevatten instellingen die van toepassing zijn op specifieke software of services. 

- Je kunt een script toevoegen om instellingen toe te voegen voor alle gebruikers zonder dat je `/etc/profile` hoeft aan te passen. 
- Dit maakt het eenvoudiger om instellingen te beheren en software te configureren.

---

## `~/.bashrc`

Dit is een configuratiebestand in de home-directory van elke gebruiker en is specifiek voor elke gebruiker. 

- Het wordt geladen wanneer een nieuwe niet-login interactieve shell wordt gestart.
- Het kan configuraties bevatten voor een shellsessie zoals:
  - Aliassen
  - Environment-variabelen
  - Functies of kleine scriptjes
  - Terminal-instellingen
  - ...

---

## Umask

De `umask` (User file creation mask) bepaalt de standaardbestandsrechten wanneer een bestand of directory wordt aangemaakt. Het stelt in welke rechten een gebruiker **niet** mag hebben in plaats van welke rechten ze moeten hebben.

### Bestanden en directories hebben drie typen gebruikersrechten:
1. **Owner**: De gebruiker die de file heeft aangemaakt.
2. **Group**: De groep waartoe de gebruiker behoort.
3. **Others**: Alle andere gebruikers op het systeem.

### Rechten
- **Read (r)**: 4
- **Write (w)**: 2
- **Execute (x)**: 1

De som van deze getallen bepaalt de rechten per categorie (owner, group, others). 

### Werking
De `umask` werkt omgekeerd aan `chmod`.

#### Voorbeeld:
- **Standaard permissies**: `666` (rw-rw-rw-).
- **Umask**: `0022`.
- **Resultaat**: `644` (rw-r--r--).

```bash
vi test.txt
# Standaard permissies: 666
umask 0022
# Resultaat: 644
```

---

## Gebruikersbeheer

- **`useradd -D`**: Standaardopties voor het aanmaken van gebruikers.
- **`useradd`**: Nieuwe gebruikers toevoegen.
- **Standaardwaarden:**
  - Locatie van de home-directory.
  - Standaard shell: `/bin/bash`.
  - Standaard gebruiker-ID (UID).
  - Standaard groeps-ID (GID).

### Belangrijke bestanden
- **`/etc/login.defs`**: Globale instellingen die worden toegepast bij het aanmaken van nieuwe gebruikers en het inloggen op het systeem. Belangrijke standaardwaarden:
  - UID- en GID-ranges.
  - Home-directories.
  - Wachtwoordbeleid.
  - Shell.
- **`/etc/pam.d/`**: Beveiligingsinstellingen.
- **`/etc/skel`**: Skeleton-directory. Bij het aanmaken van een nieuwe gebruiker worden standaardbestanden uit deze directory gekopieerd naar de home-directory van de gebruiker.

---

## Bekijken van een systeem

- **`ps`**: Procesoverzicht.
  - **`ps auxww`**: Gedetailleerd procesoverzicht.
- **`top`**: Processen met het meeste geheugengebruik bekijken.
- **`slabtop`**: Kernel caching bekijken, inclusief informatie over:
  - **Inodes**: Geeft aan waar een bestand op de schijf staat.
- **`ipcs`**: Inter-proces communicatie (processen die data delen).
- **`ldd`**: Bibliotheken tonen die door een programma worden gebruikt.
- **`hdparm`**: Schijfprestaties testen.
- **`fuser`**: Bekijken welke processen bestanden blokkeren.
- **`lsof`**: Open bestanden en netwerkverbindingen tonen.
  - **`lsof -i4`**: Controleert alle IP-adressen.
- **`ip r`**: Routetabel van netwerken bekijken.
- **`w`**: Systeembelasting en actieve gebruikers tonen.
- **`last`**: Laatste ingelogde gebruikers tonen.

---

## Services en Limieten

- **`ulimit`**: Beperkingen instellen zoals:
  - Bestandsgrootte.
  - Aantal open bestanden.
  - Aantal processen.
  - Geheugengebruik.
- **`syslog`**: Logbestanden beheren (vervangen door `journalctl` van `systemd`).

---

## Swapspace en RAMdisk

- **Swapspace**: Gebruikt als back-up voor RAM. Niet nodig als er genoeg RAM is, maar vereist voor functies zoals hibernation (swapgrootte minimaal gelijk aan RAM).
- **RAMdisk**: Tijdelijke opslag in RAM. Voordelen:
  - Snelheid.
  - Ideaal voor tijdelijke gegevens.
  - Gegevens gaan verloren bij stroomuitval.

### Belangrijk bestand
- **`/etc/fstab`**: Filesystem-tabellen configureren.