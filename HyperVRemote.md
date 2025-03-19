# *OBSOLETE* 
## Hyper-V Remotesteuerung

Anleitung zur remote Administrierung eines Hyper-V Servers ohne Mitglied einer Domäne zu sein.

> [!IMPORTANT]
>
> Damit das Ganze funktioniert muss der "Hypervisor" eine statische IP-Adresse und einen eindeutigen Computernamen im Netzwerk haben.
>
> Der Manager-PC muss Windows 10/11 Pro installiert haben. 

## Hypervisor


Zum Verwalten von Hyper-V-Remotehosts muss auf dem lokalen Computer und auf dem Remotehost die Remoteverwaltung aktiviert sein.

Öffnen Sie dazu Windows `PowerShell` als Administrator, und führen Sie Folgendes aus:

```powershell 
Enable-PSRemoting 
```

```powershell
Enable-WSManCredSSP -Role server
```


---



## Lokaler PC

### 1. PowerShell

Auf dem "Manager-PC":

```powershell 
Enable-PSRemoting 
```

```powershell
Winrm quickconfig
```

```powershell  
Enable-WSManCredSSP -Role client -DelegateComputer "Computername des Hyper-V Servers"
```

### 2. Gruppenrichtlinien Anpassen
 
Auf dem "Manager-PC" muss der "Hypervisor" in den Gruppenrichtlinien 2x unter "Delegierung von Anmeldeinformationen" eingetragen werden.  
 
 - Run Dialog mit `Win + R` aufrufen
 - `gpedit.msc` eingeben und mit <kbd>Enter</kbd> bestätigen
 - Navigieren zu: **Computerkonfiguration** > **Administrative Vorlagen** > **System** > **Delegierung von Anmeldeinformationen**
	 -  Doppelklick auf **"Delegierung von aktuellen Anmeldeinformationen"**
		- `Aktiviert` anklicken
		- Unter Optionen &rarr; Server zur Liste hinzufügen: &rarr; `Anzeigen…` anklicken  
		- `wsman/"Computername Hypervisor"` eintragen  
	 - Doppelklick auf **"Delegierung von aktuellen Anmeldeinformationen zulassen mit reiner NTLM-Serverauthentifizierung zulassen"**
		- `aktiviert` anklicken 
		- Unter Optionen &rarr; Server zur Liste hinzufügen: &rarr; `Anzeigen…` anklicken
		- `wsman/"Computername Hypervisor"` eintragen

Nun wird der Hyper-V Server als WSMAN TrustedHost hinterlegt. 
 - Öffne PowerShell als Administrator:

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value “Computername Hypervisor”
```

### 3. Host Datei bearbeiten

Zum Schluss muss die IP-Adresse und der Computername des Hyper-V Servers in die Hosts-Datei eingetragen werden, damit der Hostname aufgelöst werden kann. 

Die Hosts-Datei befindet sich unter: `C:\Windows\System32\drivers\etc\hosts`

> [!CAUTION]
> 
> Änderungen in der Hosts Datei sollten mit Vorsicht vorgenommen werden!

### 4. Server hinzufügen

Jetzt kann der Hyper-V Server im Hyper-V Manager aufgenommen werden.

Dazu muss man folgende Schritte durchführen:
 - Hyper-V Manager öffnen
	 - Auf der linken Seite &rarr; Rechtsklick auf `Hyper-V Manager`
	 - `Verbindung mit dem Server herstellen...`
	 - Bei `Anderer Computer` den Computernamen des Hyper-V Servers eingeben
	 - Als Benutzer  **\\Administrator** und das Passwort des Servers eingeben
	 
> [!NOTE]
>
> Der Backslash vor dem Benutzer verhindert das Domänennamen an den Server übertragen werden.
