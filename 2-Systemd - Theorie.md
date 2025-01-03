
# Lab 2: Theorie Linux Tuning

---

## `/etc/issue`

### Beschrijving
`/etc/issue` bevat een tekstbericht (banner) dat aan gebruikers wordt getoond voordat ze inloggen. Dit bericht is zichtbaar op het toestel, bijvoorbeeld in een lokale console of terminal.

- **`/etc/issue`**: Voor consoles op het toestel.
- **`/etc/issue.net`**: Gebruikt bij telnet of SSH.

---

## `/etc/motd`

### Beschrijving
Het Message of the Day-bestand (`/etc/motd`) toont een bericht aan de gebruiker direct na inloggen.

---

## Shell Configuratiebestanden

### `/etc/profile` en `/etc/bash.bashrc`
- **`/etc/profile`**:
  - Wordt uitgevoerd bij het openen van een login shell.
  - Geldt globaal voor alle gebruikers.
- **`/etc/bash.bashrc`**:
  - Wordt uitgevoerd bij een niet-login interactieve shell.
  - Geldt ook voor alle gebruikers.

### `/etc/profile.d/`
Scripts in deze map kunnen specifieke instellingen toevoegen zonder `/etc/profile` direct te bewerken.

### `~/.bashrc`
Individueel configuratiebestand voor een gebruiker, uitgevoerd bij niet-login interactieve shells.

---

## Umask

### Beschrijving
`umask` bepaalt welke rechten **niet** zijn toegestaan bij het aanmaken van bestanden of directories.

### Formule
- **Standaard permissies**: `666` (voor bestanden) en `777` (voor directories).
- **Umask**: Een waarde zoals `0022`.
- **Resultaat**:
  - Bestanden: `644`.
  - Directories: `755`.

---

## Gebruikersbeheer

### Commands
- **`useradd`**: Nieuwe gebruikers aanmaken.
- **`/etc/login.defs`**: Standaardinstellingen zoals UID/GID-ranges en wachtwoordbeleid.
- **`/etc/skel`**: Voorbeeldbestanden die naar een nieuwe home-directory worden gekopieerd.

---

## Systeemanalyse Tools

### Overzicht
- **`ps`**: Toont actieve processen.
- **`top`**: Dynamisch overzicht van processen.
- **`slabtop`**: Kernel caching analyseren.
- **`ipcs`**: Deelbare geheugenobjecten tonen.
- **`ldd`**: Afhankelijkheden van een uitvoerbaar bestand tonen.
- **`hdparm`**: Schijfprestaties testen.
- **`lsof`**: Open bestanden/netwerkverbindingen tonen.
- **`ip r`**: Routetabellen bekijken.
- **`w`** en **`last`**: Gebruikersactiviteit en systeemhistorie tonen.

---

## Resources en Limieten

### Beschrijving
- **`ulimit`**: Beperkingen instellen zoals geheugen, bestandsgrootte, en processen.
- **`syslog`**: Logbestanden beheren (nu vervangen door `journalctl`).

---

## Swap en RAMdisk

### Beschrijving
- **Swapspace**: Back-up voor RAM, essentieel voor functies zoals hibernation.
- **RAMdisk**: Tijdelijke opslag in RAM, sneller maar vluchtig.

---

## Filesystem Configuratie

### Belangrijke bestanden
- **`/etc/fstab`**: Filesystem tabellen configureren, inclusief swap en RAMdisk.

---