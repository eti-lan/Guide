# eti LAN Guide
### Schritt für Schritt zu deinem eigenen LAN Party Server


Oft wurden wir gefragt: "Gibt es nicht eine Anleitung?" "Wie installiere ich das genau?" "Wie geht das nochmal mit diesem DNS?" Mit diesem Guide versuchen wir, einige der häufigsten Fragen zu beantworten. 

Du möchtest eine **LAN Party** veranstalten und weißt nicht, wo du anfangen sollst? Du hast schon Erfahrungen als Veranstalter und möchtest dein Setup nun optimieren und z.B. **Gameserver** bereit stellen? Dieser Guide kann dir helfen.

Bitte beachte, dass es sich bei dieser Anleitung um _einen möglichen_ Weg handelt. Es gibt mit Sicherheit noch bessere Ansätze oder bessere Software. Wir haben diese Anleitung jedoch so gestaltet, dass mit einem Minimum an Aufwand ein bestmögliches Ergebnis erreicht werden kann, ohne viele Einschränkungen zu schaffen. Wir versuchen dabei, die wichtigsten **Basics** zu vermitteln, die man für eine erfolgreiche LAN Party verinnerlicht haben sollte.

Diese Anleitung geht davon aus, dass du ein System einrichten möchtest, dass als **Server für deine LAN** dienen soll. Es ist erst einmal nicht so wichtig, ob diese Hardware schon bereit steht oder erst später beschafft werden soll. Sie muss lediglich **x86_64** -kompatibel sein. Wir nutzen im Folgenden die Software **VMware Workstation**, um virtuelle Server anzulegen und auszuführen. Es ist dir überlassen, welche Schritte du bearbeiten möchtest, welche Dienste du einrichten möchtest und wo deine **virtuellen Server** später ausgeführt werden. Im Detail erklären wir folgendes:

 
## Vorgehen
**1. Installation VMware Workstation**

> Alternativ kann auch eine andere Virtualisierungsumgebung oder VMware ESXi genutzt werden. Die Wahl fiel unter anderem auch deshalb auf VMware Workstation, da hier die Möglichkeit besteht, die virtuellen Server später zu exportieren und mit wenigen Klicks anzupassen.

**2. Konfiguration virtueller Netzwerkkarten**
> Möchtest du auch einen Internetzugang ermöglichen? Dann benötigt deine Serverhardware später 2 physische Netzwerkkarten. Anderenfalls genügt auch eine einzelne. 

**3. Installation einer pfSense Firewall**
> Aufspannen deines LANs, mit DHCP-Server, DNS-Forwarder und optionaler Internetanbindung.

**4. Installation eines Gameservers**

**5. Installation von LANPage**


## Voraussetzungen
Diese Anleitung stellt keine allzu hohen Anforderungen an deine Hardware. Es hängt letztlich von deinem Konzept ab, wie viele virtuelle Server du benötigst und ausführen möchtest.  Wir können an dieser Stelle nur Empfehlungen aussprechen. Gehen wir beispielsweise davon aus, dass das Serversytem über folgende Ausstattung verfügt:

### Beispielhardware - alter Desktop PC
|Komponente|Produkt|Typ|Werte|
|----------------|-----------------|--------------|-----------------------------|
|CPU|Intel Core i5-2400 @ 3.10GHz|LGA1155|Cores: **4** Threads: **4**|
|RAM|Samsung M378B5173DB0-CK0|PC3-12800U| 4 x **4 GB**
|Storage|Western Digital WD Red SA500l|SSD |**1 TB**
|Storage|Seagate BarraCuda ST2000DM008|HDD |**2 TB**
|NIC|Realtek RTL8111E|Onboard|10/100/**1000 Mbps**
|NIC|Broadcom BCM5751-T1 1G|PCI-e X1|10/100/**1000 Mbps**



### Mögliche Ressourcenverteilung

> Diese Hardware würde schon eine ansehnliche Ressourcenverteilung ermöglichen. Bitte beachte, dass es sich um ein individuelles Beispiel handelt und die Virtualisierungsebene bzw. das unter den VMs laufende Betriebssystem selbst auch ein paar Ressourcen benötigt.
> 
|VM|vCPUs|RAM|Storage|Typ|NIC|OS
|-------|------|---------|--------|-----|---|---|
|Firewall|1|1 GB|30 GB|SSD|2|pfSense (FreeBSD)
|Gameserver|2|8 GB|100 GB|SSD|1|Windows 10
|Sync Server|1|4 GB|2000 GB|HDD|1|Debian 12
|LANPage Webserver|1|0.5 GB|10 GB|HDD|1|Debian 12


## Vorbereitung

### Downloads
Für die nächsten Schritte benötigen wir einige Dateien, welche du schon einmal herunterladen kannst. Für die Testversionen von VMware ist unter Umständen eine Registrierung erforderlich - eine Wegwerf-Spamadresse genügt.

- VMware Workstation (Testversion): https://www.vmware.com/go/getworkstation-win
- pfSense (AMD64, DVD Image): https://www.pfsense.org/download
- Debian 12 (AMD64, Netinstaller): https://www.debian.org/download
- Windows 10 (64-Bit ISO Image): https://www.microsoft.com/de-de/software-download/windows10

Optional:
- Windows Server (64-Bit ISO Image): https://www.microsoft.com/de-DE/evalcenter/evaluate-windows-server-2022

