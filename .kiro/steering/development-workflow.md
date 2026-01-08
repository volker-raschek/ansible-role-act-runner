---
inclusion: always
---

# Entwicklungsworkflow für Act Runner Ansible Role

## Entwicklungsumgebung

### Voraussetzungen

- Ansible >= 2.9
- Node.js >= 16.0.0 (für Dokumentations-Tools)
- Docker (für Testing)
- Molecule (für Role-Testing)

### Setup

```bash
# Dependencies installieren
npm install

# Ansible Collections installieren
ansible-galaxy collection install -r requirements.yml

# Molecule für Testing
pip install molecule[docker]
```

## Code-Änderungen

### Workflow

1. **Branch erstellen**: `git checkout -b feature/beschreibung`
2. **Änderungen implementieren**
3. **Tests ausführen**
4. **Dokumentation aktualisieren**
5. **Pull Request erstellen**

### Task-Entwicklung

```yaml
# Neue Tasks in tasks/main.yaml hinzufügen
- name: Beschreibender Name der Aktion
  ansible.builtin.module:
    parameter: "{{ variable }}"
  register: result
  when: condition
  notify: handler_name
```

### Variablen hinzufügen

1. In `defaults/main.yaml` mit `@param` Dokumentation
2. OS-spezifische Werte in `vars/OS.yaml`
3. README automatisch regenerieren: `npm run readme:parameters`

## Testing

### Lokales Testing

```bash
# Syntax Check
ansible-playbook --syntax-check tests/test.yml

# Lint Check
ansible-lint .

# YAML Lint
yamllint .

# Markdown Lint
npm run readme:lint
```

### Molecule Testing

```bash
# Alle Tests
molecule test

# Nur Syntax
molecule syntax

# Converge (ohne Destroy)
molecule converge
```

## Qualitätssicherung

### Pre-Commit Checks

- Ansible Lint muss ohne Fehler durchlaufen
- YAML Syntax muss korrekt sein
- Markdown Dokumentation muss valide sein
- Alle neuen Variablen müssen dokumentiert sein

### Code Review Kriterien

- Idempotenz gewährleistet
- Fehlerbehandlung implementiert
- Plattform-Kompatibilität beachtet
- Sicherheits-Best-Practices befolgt
- Tests für neue Funktionalität

## Release-Prozess

### Versionierung

- Semantic Versioning (MAJOR.MINOR.PATCH)
- Tags für Releases
- Changelog pflegen

### Ansible Galaxy

```bash
# Role zu Galaxy hochladen
ansible-galaxy role import volker-raschek act-runner-ansible-role
```

## Troubleshooting

### Häufige Probleme

- **Service startet nicht**: Logs prüfen mit `journalctl -u act_runner`
- **Registrierung fehlgeschlagen**: Token und URL validieren
- **Permission Denied**: Benutzer/Gruppen-Konfiguration prüfen

### Debug-Modus

```yaml
# Mehr Ausgaben für Debugging
- debug:
    var: variable_name
  when: ansible_verbosity >= 2
```
