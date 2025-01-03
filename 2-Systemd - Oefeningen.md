

# Lab 2: Linux Tuning Systemd

---

## 1. chkservice

### Doel
Beheer en controleer de status van systemd-diensten, inclusief het in- of uitschakelen van automatisch opstarten bij het opstarten van het systeem.

### Installatie
Installeer `chkservice` met:
```bash
sudo apt update
sudo apt install chkservice
```

Controleer de geïnstalleerde bestanden van het pakket:
```bash
dpkg -L chkservice
```

Voorbeeld output:
```text
/.
/usr
/usr/bin
/usr/bin/chkservice
/usr/share
/usr/share/doc/chkservice
/usr/share/doc/chkservice/changelog.Debian.gz
/usr/share/doc/chkservice/copyright
/usr/share/man/man8/chkservice.8.gz
```

### Oefening
- Start de `nginx` service en zorg ervoor dat deze niet automatisch opstart bij het opstarten van het systeem:
  ```bash
  sudo systemctl start nginx
  sudo systemctl disable nginx
  chkservice
  ```
- **Controleer** met `chkservice`:
  - Een vinkje betekent dat de service automatisch opstart.
  - Geen vinkje betekent dat de service niet automatisch opstart.

---

## 2. Opstarttijd van Services

### Doel
Gebruik `systemd` om de services met de langste opstarttijd te identificeren.

### Oefening
Toon de opstarttijden met:
```bash
systemd-analyze blame
```

Beperk de output tot de vier langzaamste services:
```bash
systemd-analyze blame | head -n 4
```

Voorbeeld output:
```text
44.996s plymouth-quit-wait.service
23.775s vboxadd.service
1.482s snapd.seeded.service
1.369s snapd.service
```

---

## 3. Beheer de nginx Service

### Oefening
- Stop en start de `nginx` service met:
  ```bash
  systemctl stop nginx
  systemctl start nginx
  ```
- Bekijk de logs van `nginx` met:
  ```bash
  sudo journalctl -u nginx
  ```

---

## 4. Maak de `helloworld` Service

### Doel
Schrijf een script dat elke 10 seconden "Hello World" met de datum en tijd naar een logbestand schrijft.

### Script
Plaats het script in `/usr/local/sbin/helloworld`:
```bash
#!/bin/bash
LOG_FILE="/var/log/helloworld.log"
PID_FILE="/tmp/helloworld.pid"

if [ "$1" == "start" ]; then
    if [ -f "$PID_FILE" ]; then
        echo "Service is al gestart (PID: $(cat $PID_FILE))"
        exit 1
    fi
    echo "Service starten..."
    while true; do
        echo "Hello World $(date)" >> "$LOG_FILE"
        sleep 10
    done &
    echo $! > "$PID_FILE"
    echo "Service gestart met PID $(cat $PID_FILE)"
elif [ "$1" == "stop" ]; then
    if [ ! -f "$PID_FILE" ]; then
        echo "Service draait niet."
        exit 1
    fi
    echo "Service stoppen..."
    kill "$(cat $PID_FILE)" && rm -f "$PID_FILE"
    echo "Service gestopt."
else
    echo "Gebruik: $0 {start|stop}"
    exit 1
fi
```

Maak het script uitvoerbaar:
```bash
sudo chmod +x /usr/local/sbin/helloworld
```

Start en stop de service:
```bash
sudo /usr/local/sbin/helloworld start
sudo /usr/local/sbin/helloworld stop
```

Controleer de log met:
```bash
tail -f /var/log/helloworld.log
```

### Systemd-configuratie
Maak een servicebestand `/etc/systemd/system/helloworld.service`:
```ini
[Unit]
Description=Service dat het script helloworld uitvoert.

[Service]
Type=simple
ExecStart=/usr/local/sbin/helloworld start
ExecStop=/usr/local/sbin/helloworld stop

[Install]
WantedBy=multi-user.target
```

Herlaad de systemd-configuratie:
```bash
sudo systemctl daemon-reload
sudo systemctl start helloworld
sudo systemctl status helloworld
sudo systemctl stop helloworld
```

---

## 5. Maak de `firewalld` Service

### Doel
Schrijf een firewall-script en maak een systemd-service om deze te beheren.

### Script
Plaats het script in `/usr/local/sbin/firewalld`:
```bash
#!/bin/bash
firewalld_stop() {
    iptables -F
    iptables -X
    iptables -P INPUT ACCEPT
    iptables -P FORWARD ACCEPT
    iptables -P OUTPUT ACCEPT
}

firewalld_start() {
    iptables -F
    iptables -X
    iptables -P INPUT ACCEPT
    iptables -P FORWARD ACCEPT
    iptables -P OUTPUT ACCEPT
    iptables -A INPUT -p icmp -j DROP
    iptables -A OUTPUT -p icmp -j DROP
}

firewalld_restart() {
    firewalld_stop
    firewalld_start
}

firewalld_reload() {
    firewalld_stop
    firewalld_start
}

case "$1" in
    start) firewalld_start ;;
    stop) firewalld_stop ;;
    restart) firewalld_restart ;;
    reload) firewalld_reload ;;
    *) echo "Gebruik: $0 {start|stop|restart|reload}" ;;
esac
```

Maak het script uitvoerbaar:
```bash
sudo chmod +x /usr/local/sbin/firewalld
```

Start en stop de service:
```bash
sudo /usr/local/sbin/firewalld start
sudo /usr/local/sbin/firewalld stop
```

### Systemd-configuratie
Maak een servicebestand `/etc/systemd/system/firewalld.service`:
```ini
[Unit]
Description=Service dat het script firewalld uitvoert.

[Service]
Type=simple
ExecStart=/usr/local/sbin/firewalld start
ExecStop=/usr/local/sbin/firewalld stop
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

Herlaad de systemd-configuratie:
```bash
sudo systemctl daemon-reload
sudo systemctl start firewalld.service
sudo systemctl status firewalld.service
sudo systemctl stop firewalld.service
```

Test door te pingen:
```bash
ping -c 4 google.com
```
```bash
sudo /usr/local/sbin/firewalld start
ping -c 4 google.com
sudo /usr/local/sbin/firewalld stop
```

---

**Einde Lab 2**
