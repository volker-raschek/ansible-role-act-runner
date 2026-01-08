---
inclusion: fileMatch
fileMatchPattern: 'tests/**/*'
---

# Testing-Strategie für Act Runner Role

## Test-Pyramide

### Unit Tests (Syntax/Lint)

```bash
# Ansible Syntax
ansible-playbook --syntax-check tests/test.yml

# Ansible Lint
ansible-lint .

# YAML Lint
yamllint .
```

### Integration Tests (Molecule)

```bash
# Vollständiger Test-Zyklus
molecule test

# Einzelne Schritte
molecule create    # Test-Umgebung erstellen
molecule converge  # Role ausführen
molecule verify    # Tests ausführen
molecule destroy   # Aufräumen
```

### End-to-End Tests

- Runner registriert sich erfolgreich
- Service startet und läuft stabil
- Jobs können ausgeführt werden

## Molecule-Konfiguration

### molecule.yml

```yaml
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: ubuntu-20.04
    image: ubuntu:20.04
    pre_build_image: true
  - name: archlinux
    image: archlinux:latest
    pre_build_image: true
provisioner:
  name: ansible
  config_options:
    defaults:
      callbacks_enabled: profile_tasks
verifier:
  name: ansible
```

### Test Scenarios

```yaml
# molecule/default/converge.yml
- name: Converge
  hosts: all
  become: true
  vars:
    act_runner_gitea_url: "https://gitea.example.com"
    act_runner_token: "test-token"
  roles:
    - role: act_runner
```

## Testfälle

### Positive Tests

- Installation auf unterstützten OS
- Konfiguration mit Standard-Werten
- Service-Start und -Status
- Registrierung (gemockt)

### Negative Tests

- Fehlende Required Variables
- Ungültige Gitea-URL
- Ungültiger Token
- Unzureichende Berechtigungen

### Edge Cases

- Bereits registrierter Runner
- Service bereits gestartet
- Konfigurationsdatei existiert bereits

## Testdaten

### Mock-Konfigurationen

```yaml
# Minimal
act_runner_gitea_url: "https://test.gitea.com"
act_runner_token: "test-token-123"

# Erweitert
act_runner_config:
  runner:
    capacity: 2
    labels:
      - "test:docker://alpine:latest"
```

### Test-Inventories

```ini
[runners]
test-ubuntu ansible_host=ubuntu-container
test-arch ansible_host=arch-container

[runners:vars]
ansible_connection=docker
```

## Verifikation

### Service Tests

```yaml
# In molecule/default/verify.yml
- name: Verify service is running
  service_facts:
  
- name: Check act_runner service
  assert:
    that:
      - ansible_facts.services['act_runner.service'].state == 'running'
      - ansible_facts.services['act_runner.service'].status == 'enabled'
```

### Configuration Tests

```yaml
- name: Verify config file exists
  stat:
    path: /etc/act_runner/config.yaml
  register: config_file

- name: Check config file
  assert:
    that:
      - config_file.stat.exists
      - config_file.stat.mode == '0644'
```

### Process Tests

```yaml
- name: Check act_runner process
  command: pgrep -f act_runner
  register: runner_process
  failed_when: runner_process.rc != 0
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Test Role
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Molecule
        run: molecule test
```

### Gitea Actions

```yaml
name: Ansible Role Test
on: [push]
jobs:
  molecule:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Test with Molecule
        run: |
          pip install molecule[docker]
          molecule test
```

## Performance Tests

### Resource Usage

- Memory consumption des Runners
- CPU-Last während Job-Ausführung
- Disk I/O für Container-Images

### Scalability

- Mehrere Runner auf einem Host
- Concurrent Job-Ausführung
- Container-Startup-Zeit

## Regression Tests

### Upgrade-Tests

- Update von vorheriger Version
- Konfigurationsmigration
- Service-Kontinuität

### Compatibility Tests

- Verschiedene Ansible-Versionen
- Verschiedene OS-Versionen
- Verschiedene Docker-Versionen
