---
inclusion: always
---

# Sicherheitsrichtlinien für Act Runner

## Grundprinzipien

### Least Privilege

- Runner läuft unter dediziertem Benutzer (`act_runner`)
- Minimale Berechtigungen für Dateien und Verzeichnisse
- Keine Root-Rechte für Runner-Prozess

### Isolation

- Container-basierte Job-Ausführung
- Netzwerk-Isolation wo möglich
- Begrenzte Volume-Mounts

## Konfigurationssicherheit

### Secrets Management

```yaml
# NIEMALS Secrets in Klartext
act_runner_token: "{{ vault_act_runner_token }}"

# Ansible Vault verwenden
ansible-vault encrypt_string 'secret-token' --name 'vault_act_runner_token'
```

### Sichere Defaults

```yaml
act_runner_config:
  runner:
    insecure: false  # TLS-Verifikation aktiviert
  container:
    privileged: false  # Kein privilegierter Modus
    require_docker: true  # Docker-Daemon erforderlich
```

## Container-Sicherheit

### Privileged Mode

```yaml
# Nur wenn unbedingt erforderlich
act_runner_config:
  container:
    privileged: true
    # Dokumentiere WARUM privileged erforderlich ist
```

### Volume Restrictions

```yaml
act_runner_config:
  container:
    valid_volumes:
      - "/tmp"
      - "/var/cache"
      # NIEMALS: "/", "/etc", "/var/lib"
```

### Docker Socket

```yaml
# Vorsicht bei Docker Socket Mount
valid_volumes:
  - "/var/run/docker.sock:/var/run/docker.sock"
# Ermöglicht Container-Escape!
```

## Netzwerk-Sicherheit

### TLS-Konfiguration

```yaml
act_runner_config:
  runner:
    insecure: false  # Immer TLS verwenden
    fetch_timeout: "5s"  # Kurze Timeouts
```

### Firewall-Regeln

- Nur ausgehende Verbindungen zu Gitea-Server
- Keine eingehenden Verbindungen erforderlich
- Docker-Netzwerk isolieren

## Dateisystem-Sicherheit

### Berechtigungen

```yaml
# In tasks
- name: Create secure directory
  file:
    path: "{{ act_runner_lib_dir }}"
    owner: "{{ act_runner_unix_user }}"
    group: "{{ act_runner_unix_group }}"
    mode: "0750"  # Keine world-readable
```

### Sensitive Files

```yaml
- name: Template config with restricted permissions
  template:
    src: config.yaml.j2
    dest: "{{ act_runner_config_file }}"
    mode: "0640"  # Nur Owner und Group
```

## Monitoring und Logging

### Log-Sicherheit

```yaml
act_runner_config:
  log:
    level: info  # Nicht debug in Production
```

### Audit-Trail

- Alle Runner-Registrierungen loggen
- Service-Status überwachen
- Fehlgeschlagene Jobs protokollieren

## Incident Response

### Kompromittierung

1. Runner-Service stoppen
2. Token widerrufen in Gitea
3. Logs analysieren
4. System neu aufsetzen
5. Neuen Token generieren

### Verdächtige Aktivitäten

- Unerwartete Container-Images
- Excessive Resource Usage
- Netzwerk-Verbindungen zu unbekannten Hosts

## Compliance

### Datenverarbeitung

- Keine persistente Speicherung von Job-Daten
- Temporäre Dateien nach Job-Ende löschen
- Logs rotieren und archivieren

### Zugriffskontrolle

- Dokumentiere wer Zugriff auf Runner-Token hat
- Regelmäßige Token-Rotation
- Prinzip der minimalen Berechtigung