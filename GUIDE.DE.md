# eti LAN Guide
### Schritt für Schritt zu deinem eigenen LAN Party Server


Oft wurden wir gefragt: "Gibt es nicht eine Anleitung?" "Wie installiere ich das genau?" "Wie geht das nochmal mit diesem DNS?" Mit diesem Guide versuchen wir, einige der häufigsten Fragen zu beantworten. 

Du möchtest eine **LAN Party** veranstalten und weißt nicht, wo du anfangen sollst? Du hast schon Erfahrungen als Veranstalter und möchtest dein Setup nun optimieren und z.B. **Gameserver** bereit stellen? Dieser Guide kann dir helfen.

Bitte beachte, dass es sich bei dieser Anleitung um _einen möglichen_ Weg handelt. Es gibt mit Sicherheit noch bessere Ansätze oder bessere Software. Wir haben diese Anleitung jedoch so gestaltet, dass mit einem Minimum an Aufwand ein bestmögliches Ergebnis erreicht werden kann, ohne viele Einschränkungen zu schaffen. Wir versuchen dabei, die wichtigsten **Basics** zu vermitteln, die man für eine erfolgreiche LAN Party verinnerlicht haben sollte.

Diese Anleitung geht davon aus, dass du ein System einrichten möchtest, dass als **Server für deine LAN** dienen soll. Es ist erst einmal nicht so wichtig, ob diese Hardware schon bereit steht oder erst später beschafft werden soll. Sie muss lediglich **x86_64** -kompatibel sein. Wir nutzen im Folgenden die Software **VMware Workstation**, um virtuelle Server anzulegen und auszuführen. Es ist dir überlassen, welche Schritte du bearbeiten möchtest, welche Dienste du einrichten möchtest und wo deine **virtuellen Server** später ausgeführt werden. Im Detail erklären wir folgendes:

 
## Vorgehen
**1. Installation VMware Workstation**

Wir zeigen in diesem Tutorial, wie VMware Workstation unter Windows konfiguriert wird. Auf deinem Server muss also irgendeine Windows-Version installiert sein, damit du die folgenden Schritte abarbeiten kannst.

> Alternativ kann auch eine andere Virtualisierungsumgebung oder VMware ESXi genutzt werden. Die Wahl fiel unter anderem auch deshalb auf VMware Workstation, da hier die Möglichkeit besteht, die virtuellen Server später zu exportieren und mit wenigen Klicks anzupassen.

**2. Konfiguration virtueller Netzwerkkarten**
> Möchtest du auch einen Internetzugang ermöglichen? Dann benötigt deine Serverhardware später 2 physische Netzwerkkarten. Anderenfalls genügt auch eine einzelne. 

**3. Installation einer pfSense Firewall**
> Aufspannen deines LANs, mit DHCP-Server, DNS-Forwarder und optionaler Internetanbindung.

**4. Installation eines Gameservers**

**5. Installation von LANPage**


## Voraussetzungen
Diese Anleitung stellt keine allzu hohen Anforderungen an deine Hardware. Es hängt letztlich von deinem Konzept ab, wie viele virtuelle Server du benötigst und ausführen möchtest.  Wir können an dieser Stelle nur Empfehlungen aussprechen. Gehen wir beispielsweise davon aus, dass das Serversystem über folgende Ausstattung verfügt:

### Beispielhardware - alter Desktop PC
|Komponente|Produkt|Werte  
|-------|------|---------|
|CPU|Intel Core i5-2400|Cores: **4** Threads: **4**
|RAM|Samsung PC3-12800U|4 x **4 GB**
|Storage|Western Digital WD Red SSD|**1 TB**
|Storage|Seagate BarraCuda HDD|**2 TB**
|NIC|Realtek RTL8111E|10/100/**1000 Mbps**
|NIC|Broadcom BCM5751-T1|10/100/**1000 Mbps**
|Switch|D-Link DGS-108|8 Port



### Mögliche Ressourcenverteilung

> Diese Hardware würde schon eine halbwegs ansehnliche Ressourcenverteilung ermöglichen. Bitte beachte, dass es sich um ein individuelles Beispiel handelt und die Virtualisierungsebene bzw. das unter den VMs laufende Betriebssystem selbst auch ein paar Ressourcen benötigt.
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


### Installation VMware Workstation
Dieser Schritt gestaltet sich einfach: starte das soeben heruntergeladene VMware Workstation -Installationsprogramm und folge den Anweisungen.

Wenn du einen anderen Hypervisor verwenden möchtest (VMware ESXi, Hyper-V, VirtualBox, ...) kannst du die Installation überspringen. Achte darauf, dass deine Hardware mit dem Hypervisor kompatibel ist.

![](images/VMware-workstation-17.0.2-21581411_POjKgqo6DV.png)


## Vorbereitung der Firewall-VM

Los geht's! In diesem Schritt erstellst du deinen ersten virtuellen Server: Eine Firewall. Die Firewall-VM kümmert sich um die Dienste DHCP sowie DNS und fungiert als Gateway für den Internetzugriff auf deiner LAN-Party (optional). Wir demonstrieren die Einrichtung anhand der Firewall-Distribution PFsense.

### Virtuellen Server erstellen ###

![](images/vmware_5aiEANiwKz.png)

Öffne VMware Workstation und wähle über das Menü den Punkt _New Virtual Machine_ oder drücke _STRG + N_.

Konfiguriere die VM nun, wie auf den folgenden Screenshots zu sehen:

![](images/vmware_79g31C5P3b.png)

![](images/vmware_u7Uu5RH7xj.png)

#### Hinweis ####
Es empfiehlt sich, das Kompatiblitätslevel möglichst niedrig anzusetzen, da sich die VM damit später leichter auf einen ESXi-Server oder einen anderen Hypervisor übertragen lässt.

Wähle im nächsten Schritt das bereits heruntergeladene _pfSense ISO-Image_ aus.

![](images/vmware_JcH2WdhfFL.png)

Gib der Firewall-VM einen passenden Namen. Die _Location_ stellt den Speicherort der VM auf deinem Server dar. In unserem Beispiel verwenden wir eine der SSDs.

![](images/vmware_XBeCo2Xbn5.png)

Die CPU-Konfiguration ergibt sich aus den zur Verfügung stehenden Ressourcen des Servers. Wir folgen auf dem Screenshot nicht ganz der Berechnung zu unserer _Beispielhardware_.

![](images/vmware_b9We9x8E1b.png)

![](images/vmware_KICpEjCBQr.png)

### Netzwerkkonfiguration der VM ###

Belasse den Netzwerktyp zunächst bei der Standardeinstellung, in einem der nächsten Schritte konfigurieren wir zuerst die Netzwerke des Hypervisors.

![](images/vmware_W1huAqroHd.png)

### Datenträgerkonfiguration der VM ###

Die Voreinstellungen können unserer Ansicht nach beibehalten werden.

![](images/vmware_WKkPxnfWM3.png)

![](images/vmware_KA0d5NCVbu.png)

Erstelle eine virtuelle Festplatte entsprechend der Screenshots. Bei der Angabe von 30 GB handelt es um eine Empfehlung, sie darf selbstverständlich auch größer sein. Für die Firewall ist ein höherer Speicherplatzbedarf aber normalerweise nicht zu erwarten.

![](images/vmware_nZ74wDA7C8.png)

![](images/vmware_TaVj5OregI.png)

Wähle einen Speicherort für die virtuelle Festplatte. Normalerweise entspricht der Dateiname der Bezeichnung der VM und die VMDK-Datei wird im selben Verzeichnis abgelegt. Du solltest es hierbei belassen.

![](images/vmware_qkCFpidBEy.png)

![](images/vmware_5yQ1sB8e4o.png)

### Hardwareanpassungen ###

Fertig! Die VM ist nun vorkonfiguriert und du kannst nun weitere Anpassungen vornehmen. Eine Soundkarte wird für eine Firewall vermutlich nicht erforderlich sein. Benötigen werden wir jedoch eine weitere Netzwerkkarte.

![](images/vmware_7gLvfCVGdA.png)

Wähle hierzu _Add_ und _Network Adapter_.

![](images/vmware_uSiRurnNSD.png)

Die VM ist nun bereit für den ersten Start und die Installation der Firewall-Software. Bevor wir damit weiter machen, passen wir aber zunächst die Netzwerkeinstellungen der VMware Workstation (des Hypervisors) an. 

Solltest du einen anderen Hypervisor wie KVM, Hyper-V oder Virtualbox verwenden, musst du die Adapterkonfiguration analog zu den folgenden Anweisungen dort nachbilden.

> Selbstverständlich ist die Netzwerkkonfiguration auch abhängig vom gewünschten Setup. Dieses Tutorial behandelt ein gut funktionierendes Basissetup mit Internetzugang. Soll überhaupt kein Internet verteilt werden? Dann genügt für ein Basissetup auch ein Netzwerkadapter. Soll eine Lastverteilung oder ein Failover-Setup realisiert werden? Gibt es VLANs? Das weißt du sicher am besten. 


![](images/vmware_DHVKal9JPb.png)

### Netzwerkkonfiguration (Hypervisor) ###

Öffne den _Virtual Network Editor_ über das Menü _Edit_. Im Editor siehst du nun alle physischen und gegebenenfalls virtuelle Netzwerkkarten, die in deinem Server-Betriebssystem eingerichtet sind.

Wir gehen davon aus, dass eine Netzwerkkarte für _LAN_ konfiguriert wird und eine für _WAN_, also den Internetzugang. Hier kann beispielsweise eine Fritzbox, ein Kabelmodem oder ein LTE-Router angeschlossen werden.


> In VMware Workstation gibt es zwei grundlegende Netzwerkmodi: NAT (Network Address Translation) und Bridge (Brücke). Hier eine kurze Erklärung, was die beiden Modi unterscheidet:
> 
> ### NAT-Modus (Network Address Translation) ###
> 
> Hier erstellst du ein privates Netzwerk für die VMs.
> Die VMs teilen sich _eine_ IP-Adresse des Hosts, um Verbindungen nach Außen aufzubauen. Nach Außen heißt in dem Fall: zum Rest des LAN oder WAN.
> Gut, wenn von einander isolierte VMs oder Internetzugriff für VMs gewünscht sind und keine IP-Adressen im LAN benutzt werden sollen. Die Performance ist jedoch geringer und NAT bringt im Regelfall weitere Probleme mit sich. Der Netzwerkmodus ist ungeeignet für eine Firewall.


> ### Bridge-Modus (Brückenmodus) ###
> 
> Im Bridge-Modus benimmt sich die VM wie ein eigener, physischer Rechner im Netzwerk. 
> Die VM erhält über die zufallsgenerierte MAC-Adresse ihrer virtuellen Netzwerkkarte eine eigene IP-Adresse und kann direkt mit anderen Geräten sprechen. Nützlich, wenn eine nahtlose Integration der VMs ins Netzwerk gewünscht ist, sehr gute Performance.


### Virtual Network Editor ###

![](images/vmware_r0fWB5hhn9.png)

Wähle für die beiden zu verwendenden Netzwerkadapter den _Bridged_-Modus aus. Sollte eine deiner Netzwerkkarten nicht erkannt beziehungsweise aufgeführt werden, musst du sie im _ Windows Geräte-Manager_ überprüfen. Beende anschließend den _Virtuel Network Editor_ mit einem Klick auf _OK_.

![](images/vmnetcfg_y2fQ1y2mP1.png)

### Netzwerkkonfiguration (Firewall VM) ###

Öffne mit einem Rechtsklick auf die VM unter dem Punkt _Settings_ erneut das Konfigurationsmenü der VM.

![](images/vmware_OGeChzNSZx.png)

Wähle den oder die Netzwerkadapter aus und konfiguriere auch hier den _Bridged_-Modus.

![](images/vmware_4lNTHSa0X0.png)


### Zusammenfassung ###

Die Hardwareausstattung der VM sollte nun wie folgt aussehen:

![](images/vmware_m2NIH8NS8d.png)

Das sieht gut aus! Zeit für den ersten Boot! Wähle _Power on this virtual machine_.
 
![](images/vmware_R5djnctguj.png)

Wenn alles richtig konfiguriert ist, solltest du nach kurzer Zeit den _pfSense Installer_ sehen können, der von dem in die VM eingehängten _ISO-Image_ geladen wurde.

## Installation Firewall ##

Die nächsten Schritte sind recht unspektakulär. Folge den Anweisungen des _pfSense Installers_ indem du die nächsten Punkte anhand der folgenden Screenshots abarbeitest. 

> Sollte sich heraus stellen, dass für die nächsten Teilschritte ausführliche Erklärungen erforderlich sind, werden wir sie nachpflegen.

Wähle _Accept_ durch einfaches Drücken der _Eingabetaste_ und im nächsten Menü anschließend _Install_ durch Bestätigung von _OK_.

![](images/vmware_NdNPdul6Kt.png)

### Installation starten ###

![](images/vmware_WzUt4steHT.png)

### Partitionierung ###

![](images/vmware_9IMK4ex6Fl.png)

![](images/vmware_uaIZiwbJWt.png)

![](images/vmware_1bdBAeakYf.png)

#### Bestätigung ####

In diesem Schritt bestätigst du noch einmal, dass die virtuelle Festplatte der Firewall-VM formatiert werden darf.

![](images/vmware_VC7zU1OR8w.png)

### Kopiervorgang ###

Keine Sorge, du merkst schon, wenn es brennt 😛 Abwarten und Tee trinken.

![](images/vmware_HMVsfmXDxs.png)

Wenn der Kopiervorgang abgeschlossen ist, wähle _Reboot_ um die VM neu zu starten.

![](images/vmware_WquW2D2PqO.png)


## Konfiguration der Firewall ##

Wenn der Neustart der VM abgeschlossen ist, solltest du das _pfSense Wartungsmenü_ sehen können. Hier können und müssen wir zunächst eine grundlegende Konfiguration der Netzwerkkarten vornehmen.

Bisher spielte es noch keine Rolle, an welche der beiden Netzwerkkarten in deinem Server, die Netzwerkkabel zum LAN oder WAN angeschlossen sind. Das wird sich nun ändern.

### LAN-Interface ###
Wähle die Option _1_, um den virtuellen Netzwerkkarten IP-Adressen zuzuweisen.

![](images/vmware_E2PDsozprz.png)

Fangen wir mit dem _LAN-Interface_ an. In unserer Übersicht ist das NIC _em1_. Demnach muss _2_ ausgewählt werden, um _em1_ zu konfigurieren. 

Bei der Frage ob die Netzwerkkarte mit DHCP konfiguriert werden soll, wählen wir _n_, denn stattdessen wollen wir unsere Firewall IP-Adressen verteilen lassen.

### IP-Bereich festlegen ###

![](images/vmware_R6nC7TSL8P.png)

pfSense fragt daraufhin nach einer IP-Adresse für sich selbst und dem dazugehörigen IP-Bereich.

Theoretisch funktionieren etliche private Adressbereiche:

|Typ|Bereich von|bis|
|-------|------|------|
|Klasse A|10.0.0.0|10.255.255.255|
|Klasse B|172.16.0.0|172.31.255.255|
|Klasse C|192.168.0.0|192.168.255.255|


In der Praxis hat sich jedoch gezeigt, dass manche Spiele mit IP-Bereichen aus _Klasse A_ oder _B_ nicht zurecht kommen oder diese im LAN-Modus gar blockieren. Wir empfehlen daher, einen IP-Bereich aus _Klasse C_ zu verwenden.

Der Bereich sollte außerdem _nicht_ identisch mit dem privaten Netzbereich deines Routers beziehungsweise Internetmodems sein.


### Adresse der Firewall ###
Wir verwenden in unserem Beispiel die IP _192.168.168.1_ für die Firewall, woraus sich der IP-Bereich _192.168.168.0/24_ ergibt.

![](images/vmware_875Cjk78ue.png)

### IPv6 ###

In deinem Netz wirst du _IPv6_ vermutlich nicht benötigen. Du kannst _IPv6_ konfigurieren, darauf gehen wir hier aber nicht weiter ein.

![](images/vmware_qg8FHhALjc.png)


### DHCP-Server aktivieren ###

Bei der Nachfrage, ob wir den DHCP-Server im LAN einschalten wollen, bestätigen wir mit _y_, damit unsere Firewall IPs an die Clients verteilt.

![](images/vmware_L1EzHmaxRU.png)

Du kannst nun einen Bereich innerhalb des IP-Netzes angeben, aus dem Adressen an die Clients verteilt werden sollen.

![](images/vmware_XSh9fY3wQX.png)



### WAN-Interface ###

Die Netzwerkkarte _em0_ konfigurieren wir zunächst nicht. Auf unserem Beispiel-Screenshot ist zu sehen, dass diese eine IP-Adresse 192.168.178.x erhalten hat. Diese stammt von einer angeschlossenen Fritzbox, welche für den Internetzugang benutzt werden soll. Die Konfiguration von _em0_ kann später im Webinterface der pfSense vorgenommen werden. Zunächst werden wir daher eine _Test-VM_ einrichten, womit wir diese Konfigurationsoberfläche erreichen können. So kann außerdem geprüft werden ob der DHCP-Server und die anderen Einstellungen korrekt funktionieren.




## Installation Gameserver ##

Die _Test-VM_ kann nach der Konfiguration der Firewall als Gameserver fungieren. Daher werden wir sie in den folgenden Schritten auch so bezeichnen. Du kannst natürlich auch ein anderes Testsystem benutzen, wenn du keinen Gameserver betreiben möchtest. Das Vorgehen bleibt zu einem Großteil identisch.


Erstelle nun eine neue VM und wähle das _Windows Server ISO-Image_ für die Installation aus. Du kannst auch ein Windows 10 verwenden.

![](images/vmware_xvy76r0qcy.png)

Folge den Anweisungen und passe die Vorgaben gegebenenfalls an. 

![](images/vmware_ykYnPc0IRy.png)

![](images/vmware_l2IeHuGkwu.png)

![](images/vmware_teSejCS7j8.png)

Plane etwas mehr Speicherplatz ein, wenn du mehrere Gameserver in einer VM betreiben möchtest. Die virtuelle Festplatte lässt sich aber auch nachträglich vergrößern.

![](images/vmware_HxjxFMtltJ.png)

![](images/vmware_TNMBCFrnNB.png)

Prüfe die Konfiguration noch einmal in der Zusammenfassung.

![](images/vmware_UqK5upCZBV.png)

Wähle anschließend auch bei dieser VM den _Bridged_-Modus für die Netzwerkkarte.

![](images/vmware_dy4tly1Idr.png)

Klicke auf _Edit virtual machine settings_, wenn du die VM nachträglich anpassen willst. Wenn alles passt, starte die VM.

![](images/vmware_ReC39JQnSG.png)

Das Windows-Logo sollte bald erscheinen, gefolgt von der _Windows Server Installationsroutine_.

![](images/vmware_2ZCUVz7t5o.png)

Folge den Anweisungen und wähle eine passende Edition aus, die von deinen Lizenzen abhängt.

![](images/vmware_VrUonVU2PE.png)

![](images/vmware_4EORrRttsd.png)

Wähle die gesamte virtuelle Festplatte zur automatisch Partitionierung.

![](images/vmware_Yh29oEAXDy.png)

Bitte warten...

![](images/vmware_Sr6oMTtK9C.png)


>Langweilig oder? Dann lass uns doch gleich noch eine weitere VM erstellen!  Denn das dauert hier noch ein wenig und wir haben ja gerade so viel Übung darin.



## Installation Webserver ##

Ein _Webserver_ benötigt normalerweise nicht sehr viele Ressourcen und ist schnell eingerichtet. Mittels der _Webserver-VM_ kannst du zum Beispiel _LANPage_ betreiben und deinen Mitspielern ein Informationsportal anbieten. Du kannst diesen Schritt überspringen, wenn du keine solche VM betreiben möchtest.

Anderenfalls erstelle eine weitere VM und wähle das 'Debian ISO-Image' aus.

![](images/vmware_Wt24I9Q2lG.png)

![](images/vmware_JXACqtMVNN.png)

Vergib auch hier wieder einen aussagekräftigen Namen.

![](images/vmware_ID9y2W92ff.png)

Es genügt eine Minimalkonfiguration mit einem oder zwei CPU-Kernen.

![](images/vmware_yqcieIPG7f.png)

256 MB RAM würden wahrscheinlich genügen, aber um Komplikationen zu vermeiden, vergebe mindestens 1 GB.

![](images/vmware_l4VmrAzFgI.png)

![](images/vmware_CgECiqlznb.png)

![](images/vmware_iDdEO7Qfhz.png)

![](images/vmware_wP1UeNKGx5.png)

![](images/vmware_H5emm94klt.png)

Überlege, ob der Webserver künftig weitere Aufgaben übernehmen soll. 20 GB sind aber mehr als ausreichend.

![](images/vmware_QOGmah6iwq.png)

![](images/vmware_Cd1Lmhsgqj.png)

![](images/vmware_goZWA4rI8g.png)

Wähle auch hier wieder den _Bridged_-Modus für die Netzwerkkarte. Starte die VM aber zunächst noch nicht. Kehre stattdessen zurück zu deiner _Gameserver-VM_ und prüfe den Stand der Installation.

Wähle ein Passwort, wenn du bei der Maske angekommen bist.

![](images/vmware_AXOZkpRDXe.png)

Beende den Server Manager, der sich daraufhin automatisch öffnet. Du kannst ihn auch gleich so konfigurieren, dass er nicht jedes Mal startet (Klick auf _Manage_).

![](images/vmware_FdKcxc5Mop.png)

Dir wird sicherlich schon vor einer Weile der kleine Hinweis aufgefallen sein, denn VMware Workstation am unteren Ende des VM-Fensters anzeigt. 

Es wird empfohlen, die _VMware Tools_ zu installieren, um das Gastsystem mit virtuellen Treibern zu beschleunigen. Klicke dazu auf _I finished Installing_.

![](images/vmware_PwoYr04i2o.png)

Die _VMware Tools_ werden daraufhin von der Workstation heruntergeladen und automatisch installiert. Wenn nicht, kannst du diesen Schritt auch wiederholen.

![](images/vmware_jVItTvTG6m.png)

![](images/vmware_0WMY3jz2Rm.png)

Öffne dazu einfach das Menü _VM_ und wähle _Install VMware Tools_.

![](images/vmware_cQMrEVIasd.png)

Belasse es bei Installationstyp _Typical_.

![](images/vmware_2wT9ajbmXy.png)

![](images/vmware_YATNMMicSU.png)

Sobald die Installation der Treiber abgeschlossen ist, öffne die Übersicht der Netzwerkverbindungen durch _Rechtsklick_ auf das Icon neben der Lautstärkeregelung oder durch _Start-> Ausführen:_

>control netconnections ncpa.cpl


Wenn bis hierhin alles geklappt hat, sollte die _Gameserver VM_ eine IP-Adresse von der _Firewall_ erhalten haben. Ist das nicht der Fall, stimmt vermutlich die Reihenfolge der Netzwerkkabel am Server nicht mit der Auswahl von _LAN_ und _WAN_ überein. Das würde sich aber auch dadurch bemerkbar machen, dass auf dem Server auf dem _VMware Workstation_ läuft, die Internetverbindung nun nicht mehr funktioniert. Tausche in diesem Fall einmal die beiden Netzwerkkabel und starte die _Firewall-VM_ neu, gefolgt von der _Gameserver-VM_.

![](images/vmware_YO4O1EfHLJ.png)

Prüfe nun erneut die IP der _Gameserver-VM_. Hat sie eine Verbindung? Super! Dann öffne nun einen Browser. Der integrierte _Edge_ Browser dürfte für unsere Zwecke genügen.

Nun ist es Zeit, die Konfigurationsoberfläche der _pfSense Firewall_ zu öffnen. Rufe dazu die IP-Adresse mittels _https://_ auf, die du im Abschnitt _LAN-Interface_ festgelegt hast. Also beispielsweise:

>https://192.168.168.1

und bestätige die Zertifikatsfehlermeldung aufgrund der selbsterstellen _Certification Authority (CA)_ der pfSense mit einem Klick auf _Continue to 192.168.168.1 (unsafe).

![](images/vmware_57aSaRwMu5.png)

Die Standardzugangsdaten lauten:

>Benutzer:	admin

>Password:	pfsense


![](images/vmware_PBlJVuHJVy.png)

Du solltest nun den Installationsassistenten durchlaufen, um eine Grundkonfiguration der _pfSense_ vorzunehmen. 

Wähle einen _Hostnamen_ und eine _Domain_. Der _DHCP-Server_ der pfSense wird diese Informationen an die Clients verteilen und die Weboberfläche unter der Kombination aus beidem erreichbar machen, also zum Beispiel:

> https://firewall.mylan

![](images/vmware_5S57b2Uvbf.png)

Bestätige im nächsten Schritt den vorgegebenen Zeitserver oder wähle einen anderen, zum Beispiel _fritz.box_, wenn du einen entsprechenden Router hast oder _time.google.com_ für einen öffentlichen NTP-Server. 

> Ein Zeitserver ist relevanter als du vielleicht denkst. Eine zu stark von anderen Systemen abweichende Systemzeit kann in der Kommunikation der Teilnehmer zu Servern oder untereinander zu verschiedensten Problemen führen.


![](images/vmware_BUknn5BgdI.png)

Prüfe nun bei _Schritt 4_ noch einmal die Konfiguration des DHCP-Servers.

Und anschließend bei _Schritt 5_...

![](images/vmware_oJq1Fubp86.png)

...die Konfiguration der Firewall-IP...

![](images/vmware_AKdfApvEXx.png)

...sowie das von dir gewählte Passwort.

![](images/vmware_DAMv2ZsTrf.png)

Klicke auf _Reload_ um die Konfiguration zu speichern und die _Firewall_ neu zu starten.

![](images/vmware_9IPKB0QbMS.png)

Nach dem Neustart der _Firewall-VM_ solltest du nun das _Dashboard_ sehen können.

![](images/vmware_yK0ZRjzUcS.png)

Überprüfe, ob die IP-Adressen für _LAN_ und _WAN_ korrekt sind und ob das _Gateway_ einer IP-Adresse aus dem IP-Bereich deines Routers entspricht.

![](images/vmware_1c7wi487IW.png)

![](images/vmware_YCRRI7VzMU.png)

![](images/vmware_YZIK4kTwK2.png)

![](images/vmware_fL3DxEqbfO.png)

![](images/vmware_j6yI0knhgm.png)

![](images/vmware_Hx7mFqm3qz.png)

![](images/vmware_N0C5SmLpD9.png)

![](images/vmware_T84WZYZx3O.png)

![](images/vmware_k22J4eHPbo.png)

![](images/vmware_zaDibs7VEf.png)

![](images/vmware_vH0UU7bgn5.png)

![](images/vmware_B2Gsme9FNO.png)

![](images/vmware_bdelhaDKdm.png)

![](images/vmware_eYDFUyrX80.png)

![](images/vmware_k0CyRPxl0B.png)

![](images/vmware_0HQBmQ9uTk.png)

![](images/vmware_9r3RLLJY8B.png)

![](images/vmware_25fs4gRUar.png)

![](images/vmware_wy0X1NpDvA.png)

![](images/vmware_O2lQrzn1xj.png)

![](images/vmware_qLQ1ftLWm9.png)

![](images/vmware_p3ulkZ5mp6.png)

![](images/vmware_3pG30xxQGy.png)

![](images/vmware_kkbWmciamM.png)

![](images/vmware_w0b0rIECiX.png)

![](images/vmware_mKOaPEhnsA.png)

![](images/vmware_QjXH4q2s9S.png)

![](images/vmware_H6ap1j5k8Y.png)

![](images/vmware_91jCGPh8qr.png)

![](images/vmware_umrYTnTSFK.png)

![](images/vmware_Bab3eYv0oC.png)

![](images/vmware_fT612Be2MV.png)

![](images/vmware_HUuf28TKVv.png)

![](images/vmware_iz1KQmWnhT.png)

![](images/vmware_NgYJhD5WCF.png)

![](images/vmware_RrIKUzIcdh.png)

![](images/vmware_R8buyt7LMy.png)

![](images/vmware_DzqRtDS9jy.png)

![](images/vmware_k6I1dmIbgx.png)

![](images/vmware_M5TtLUmhoq.png)

![](images/vmware_eHo7TK5RFL.png)

![](images/vmware_gUVrsGcAjj.png)

![](images/vmware_tHEG51pO0q.png)

![](images/vmware_8qpX4Xyc85.png)

![](images/vmware_2WZ5cUozGd.png)

![](images/vmware_YzqMNkLqqT.png)

![](images/vmware_f84U7b7YbB.png)

![](images/vmware_7yYv4AO9fU.png)

![](images/vmware_lW4wjE1e3a.png)

![](images/vmware_tXNYukJbD2.png)

![](images/vmware_SpW3KohCW9.png)

![](images/vmware_8Q18KZAcoe.png)

![](images/vmware_T17F8ocVSv.png)

![](images/vmware_d5bjbwRexO.png)

![](images/vmware_jMgn9pZmZp.png)

![](images/vmware_4KzXrdqCXU.png)

![](images/vmware_8VAwFw36X2.png)

![](images/vmware_9Cn413TQ2t.png)

![](images/vmware_LpdAVOYktw.png)

![](images/vmware_0pakStdmYB.png)

![](images/vmware_Kcb2IFkTUc.png)

![](images/vmware_MdG5oDUVeR.png)

![](images/vmware_grYmZ60b2w.png)

![](images/vmware_FDcqZAquKG.png)

