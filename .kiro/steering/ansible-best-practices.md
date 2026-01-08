---
inclusion: always
---

# Ansible Best Practices für Act Runner Role

## Allgemeine Prinzipien

### Idempotenz

- Alle Tasks müssen idempotent sein - mehrfache Ausführung führt zum gleichen Ergebnis
- Verwende `changed_when` und `failed_when` für präzise Zustandskontrolle
- Nutze `creates` Parameter bei command/shell Tasks wo möglich

### Variablen und Namenskonventionen

- Alle Variablen beginnen mit `act_runner_` als Präfix
- Verwende aussagekräftige Namen: `act_runner_config_file` statt `config_file`
- Dokumentiere alle Variablen mit `## @param` Kommentaren
- Setze sinnvolle Defaults in `defaults/main.yaml`

### Plattform-Unterstützung

- Verwende OS-spezifische Variablen in `vars/` Verzeichnis
- Nutze `ansible_facts` für Plattform-Erkennung
- Teste auf allen unterstützten Distributionen

## Task-Organisation

### Struktur

```yaml
- name: Beschreibender Task-Name
  ansible.builtin.module_name:
    parameter: value
  register: variable_name
  when: condition
  notify: handler_name
```

### Fehlerbehandlung

- Verwende `failed_when` für spezifische Fehlerbedingungen
- Registriere wichtige Outputs mit `register`
- Nutze `assert` für Eingabevalidierung

### Handler

- Ein Handler pro Service-Aktion
- Verwende `systemd` Modul für Service-Management
- Handler sollten idempotent sein

## Sicherheit

### Berechtigungen

- Setze explizite Dateiberechtigungen (`mode`)
- Verwende dedizierte Benutzer/Gruppen
- Minimiere privilegierte Operationen

### Secrets

- Keine Secrets in Defaults oder Variablen
- Verwende Ansible Vault für sensible Daten
- Dokumentiere erforderliche Secrets klar

## Testing und Qualität

### Lint-Regeln

- Befolge `.ansible-lint` Konfiguration für Ansible Dateien.
- Verwende FQCN (Fully Qualified Collection Names)
- Keine deprecated Module verwenden
- Befolge `.markdownliny.yaml` Konfiguration für Markdown Dateien.

### Dokumentation

- Jeder Task braucht einen aussagekräftigen Namen
- Dokumentiere komplexe Logik mit Kommentaren
- README automatisch aus Variablen generieren
