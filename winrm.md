## Windows-Seite: WinRM Basic Authentication konfigurieren

### 1. PowerShell als Administrator öffnen

Öffnen Sie PowerShell mit Administrator-Rechten.

### 2. WinRM-Service aktivieren und konfigurieren

```powershell
# WinRM Service aktivieren
Enable-PSRemoting -Force

# WinRM für alle Netzwerkprofile aktivieren
Set-NetFirewallRule -DisplayName "Windows Remote Management (HTTP-In)" -Enabled True

# WinRM Listener konfigurieren
winrm quickconfig -q
```

### 3. Basic Authentication aktivieren

```powershell
# Basic Authentication für WinRM aktivieren
winrm set winrm/config/service/auth '@{Basic="true"}'

# Unverschlüsselten Traffic für Basic Auth erlauben (nur für HTTP)
winrm set winrm/config/service '@{AllowUnencrypted="true"}'

# Client-seitige Basic Authentication aktivieren
winrm set winrm/config/client/auth '@{Basic="true"}'
```

### 4. WinRM Service-Konfiguration anpassen

```powershell
# Maximale Verbindungen und Memory erhöhen
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'
winrm set winrm/config/winrs '@{MaxProcessesPerShell="15"}'
winrm set winrm/config/winrs '@{MaxShellsPerUser="25"}'

# Timeout-Werte anpassen
winrm set winrm/config '@{MaxTimeoutms="1800000"}'
```

### 5. Firewall-Regeln erstellen

```powershell
# Firewall-Regel für WinRM HTTP (Port 5985)
New-NetFirewallRule -DisplayName "WinRM-HTTP" -Direction Inbound -Protocol TCP -LocalPort 5985 -Action Allow

# Optional: Für HTTPS (Port 5986)
New-NetFirewallRule -DisplayName "WinRM-HTTPS" -Direction Inbound -Protocol TCP -LocalPort 5986 -Action Allow
```

### 6. Benutzer für Ansible erstellen (optional)

```powershell
# Neuen lokalen Benutzer erstellen
$Password = ConvertTo-SecureString "IhrSicheresPasswort123!" -AsPlainText -Force
New-LocalUser -Name "ansible-user" -Password $Password -Description "Ansible Remote User"

# Benutzer zur Administratoren-Gruppe hinzufügen
Add-LocalGroupMember -Group "Administrators" -Member "ansible-user"
```

### 7. Konfiguration überprüfen

```powershell
# WinRM-Konfiguration anzeigen
winrm get winrm/config

# WinRM-Listener anzeigen
winrm enumerate winrm/config/listener

# Verbindung testen
winrm identify -r:http://localhost:5985 -auth:basic -u:ansible-user -p:IhrSicheresPasswort123!
```

## Ansible-Seite: Konfiguration

### 1. Ansible Inventory erstellen

Erstellen Sie eine `hosts.yml` oder `inventory.ini` Datei:

**hosts.yml (YAML-Format):**
```yaml
all:
  children:
    windows:
      hosts:
        win-server01:
          ansible_host: 192.168.1.100
          ansible_user: ansible-user
          ansible_password: IhrSicheresPasswort123!
          ansible_connection: winrm
          ansible_winrm_transport: basic
          ansible_winrm_server_cert_validation: ignore
          ansible_port: 5985
```

**inventory.ini (INI-Format):**
```ini
[windows]
win-server01 ansible_host=192.168.1.100

[windows:vars]
ansible_user=ansible-user
ansible_password=IhrSicheresPasswort123!
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_winrm_server_cert_validation=ignore
ansible_port=5985
```

### 2. Ansible-Konfigurationsdatei (ansible.cfg)

```ini
[defaults]
host_key_checking = False
inventory = hosts.yml

[inventory]
enable_plugins = yaml, ini

[winrm]
# WinRM-spezifische Einstellungen
```

### 3. Python-Abhängigkeiten installieren

```bash
# pywinrm für WinRM-Unterstützung
pip install pywinrm

# Optional: für Kerberos-Authentifizierung
pip install pywinrm[kerberos]

# Optional: für CredSSP-Authentifizierung
pip install pywinrm[credssp]
```

### 4. Verbindung testen

```bash
# Ansible-Ping-Test
ansible windows -m win_ping

# Erweiterte Informationen abrufen
ansible windows -m setup
```

### 5. Beispiel-Playbook

**test-playbook.yml:**
```yaml
---
- name: Test Windows Connection
  hosts: windows
  gather_facts: yes
  tasks:
    - name: Get Windows version
      win_shell: |
        Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion
      register: windows_info

    - name: Display Windows information
      debug:
        var: windows_info.stdout

    - name: Create test directory
      win_file:
        path: C:\ansible-test
        state: directory

    - name: Create test file
      win_copy:
        content: "Hello from Ansible!"
        dest: C:\ansible-test\hello.txt
```

Playbook ausführen:
```bash
ansible-playbook -i hosts.yml test-playbook.yml
```

## Sicherheitsüberlegungen

### 1. HTTPS statt HTTP verwenden (empfohlen)

Für Produktionsumgebungen sollten Sie HTTPS verwenden:

**Windows-Seite:**
```powershell
# Selbstsigniertes Zertifikat erstellen
$cert = New-SelfSignedCertificate -CertStoreLocation Cert:\LocalMachine\My -DnsName "win-server01"

# HTTPS Listener erstellen
winrm create winrm/config/Listener?Address=*+Transport=HTTPS '@{Hostname="win-server01"; CertificateThumbprint="' + $cert.Thumbprint + '"}'

# Basic Auth für HTTPS erlauben
winrm set winrm/config/service/auth '@{Basic="true"}'
```

**Ansible-Seite:**
```yaml
ansible_port: 5986
ansible_winrm_scheme: https
ansible_winrm_server_cert_validation: ignore  # Nur für selbstsignierte Zertifikate
```

### 2. Benutzerrechte minimieren

```powershell
# Speziellen Benutzer nur mit notwendigen Rechten erstellen
# Nicht zur Administratoren-Gruppe hinzufügen, sondern spezifische Rechte vergeben
```

### 3. Netzwerksicherheit

- Verwenden Sie Firewalls, um WinRM nur für vertrauenswürdige Netzwerke zu öffnen
- Nutzen Sie VPN-Verbindungen für Remote-Zugriffe
- Implementieren Sie starke Passwort-Richtlinien

## Problembehandlung

### Häufige Fehler und Lösungen:

1. **"401 Unauthorized" Fehler:**
   ```powershell
   winrm set winrm/config/service/auth '@{Basic="true"}'
   ```

2. **"Connection timeout" Fehler:**
   ```powershell
   # Firewall-Regeln überprüfen
   Get-NetFirewallRule -DisplayName "*WinRM*"
   ```

3. **"SSL Certificate" Fehler:**
   ```yaml
   ansible_winrm_server_cert_validation: ignore
   ```

### Debug-Modus für Ansible:

```bash
ansible windows -m win_ping -vvv
```

Mit dieser Konfiguration sollten Sie erfolgreich Ansible mit Windows-Hosts über WinRM Basic Authentication nutzen können. Denken Sie daran, die Sicherheitsaspekte für Ihre Produktionsumgebung entsprechend anzupassen.