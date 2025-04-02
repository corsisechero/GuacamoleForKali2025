# GuacamoleForKali2025

# Guida: Installazione di Apache Guacamole su Ubuntu 22.04 con Proxmox

## Introduzione
Apache Guacamole è un gateway desktop remoto che consente l'accesso ai desktop tramite un browser web. In questa guida, configureremo Guacamole su Ubuntu 22.04 e imposteremo un ambiente RDP funzionante tramite Proxmox.

> **Link alla guida originale:** [How to Create Remote Desktop Gateway via Apache Guacamole on Ubuntu 22.04](https://www.atlantic.net/dedicated-server-hosting/how-to-create-remote-desktop-gateway-via-apache-guacamole-on-ubuntu-22-04/)

---

## Prerequisiti
- Un server Ubuntu 22.04
- Accesso root o utente con privilegi sudo
- Proxmox installato e configurato

---

## 1. Installazione di Apache Guacamole

### Aggiornamento dei pacchetti
```bash
sudo apt update
```

### Installazione delle dipendenze
```bash
sudo apt install -y build-essential libcairo2-dev libjpeg62-turbo-dev libpng-dev libtool-bin libossp-uuid-dev libavcodec-dev libavformat-dev libswscale-dev freerdp2-dev libpango1.0-dev libssh2-1-dev libtelnet-dev libvncserver-dev libpulse-dev libssl-dev libvorbis-dev libwebp-dev tomcat9
```

---

## 2. Configurazione delle regole IP su Proxmox

### Visualizzare le regole NAT esistenti
```bash
sudo iptables -t nat -L -n -v
```

### Configurare il NAT per il traffico HTTP verso la macchina Guacamole
```bash
iptables -t nat -A POSTROUTING -p tcp -d 10.10.10.30 --dport 80 -j MASQUERADE
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.10.10.30:80
```

### Rendere persistenti le regole
```bash
apt install iptables-persistent
```

---

## 3. Risoluzione dei problemi di RDP con XFCE

### Errore Comune
Se nei log compare il messaggio:
```
[WARN ] Window manager (pid xxxx, display 10) exited quickly (0 secs). This could indicate a window manager config problem
```
Questo errore indica che il comando per avviare XFCE non viene eseguito correttamente.

#### Soluzione
Modificare il file di avvio di XRDP:
```bash
sudo nano /etc/xrdp/startwm.sh
```

Sostituire tutto il contenuto con:
```bash
#!/bin/sh
if [ -r /etc/profile ]; then
    . /etc/profile
fi
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR

# Avvia XFCE4 correttamente
if [ -x /usr/bin/startxfce4 ]; then
    exec startxfce4
fi

# Fallback se non trova XFCE
exec /bin/sh /etc/X11/xinit/xinitrc
```

---

## 4. Assegnare i permessi corretti
Rendere il file eseguibile:
```bash
sudo chmod +x /etc/xrdp/startwm.sh
```

---

## 5. Verifica del file .xsession
Impostare il comando di avvio per l'utente:
```bash
echo "startxfce4" > ~/.xsession
```
Se si utilizza l'utente root:
```bash
sudo echo "startxfce4" > /root/.xsession
```

---

## 6. Impostare XFCE come desktop predefinito
```bash
sudo update-alternatives --set x-session-manager /usr/bin/startxfce4
```

---

## 7. Riavviare il servizio XRDP
```bash
sudo systemctl restart xrdp
sudo systemctl restart xrdp-sesman
```

## Conclusione
Apache Guacamole è ora configurato e funzionante su Ubuntu 22.04 con RDP abilitato tramite Proxmox.  
Per ulteriori informazioni, consulta la [documentazione ufficiale di Guacamole](https://guacamole.apache.org/).
```
