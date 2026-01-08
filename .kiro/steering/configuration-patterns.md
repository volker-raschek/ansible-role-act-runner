---
inclusion: fileMatch
fileMatchPattern: 'defaults/**/*.yaml'
---

# Konfigurationsmuster für Act Runner

## Variablen-Struktur

### Hierarchische Konfiguration

```yaml
act_runner_config:
  log:
    level: info
  runner:
    capacity: 1
    labels: []
  container:
    privileged: false
    options: {}
```

### Naming Conventions

- Präfix: `act_runner_` für alle Variablen
- Verschachtelte Struktur für zusammengehörige Optionen
- Klare, selbsterklärende Namen

## Konfigurationsmuster

### Standard-Setup (Minimal)

```yaml
act_runner_gitea_url: "https://gitea.example.com"
act_runner_token: "{{ vault_act_runner_token }}"
```

### Erweiterte Konfiguration

```yaml
act_runner_config:
  runner:
    capacity: 3
    labels:
      - "ubuntu-latest:docker://docker.gitea.com/runner-images:ubuntu-latest"
      - "node-18:docker://node:18-alpine"
      - "python-3.11:docker://python:3.11-slim"
  container:
    privileged: true
    network: "runner-network"
    valid_volumes:
      - "/tmp"
      - "/var/run/docker.sock:/var/run/docker.sock"
```

### High-Performance Setup

```yaml
act_runner_config:
  runner:
    capacity: 5
    timeout: "6h"
    fetch_interval: "1s"
  cache:
    enabled: true
    dir: "/var/cache/act_runner"
  container:
    force_pull: false  # Für bessere Performance
    docker_timeout: "30s"
```

## Template-Patterns

### Bedingte Konfiguration

```yaml
# In defaults/main.yaml
act_runner_config:
  container:
    privileged: "{{ act_runner_privileged_mode | default(false) }}"
    docker_host: "{{ act_runner_docker_host | default('') }}"
```

### Umgebungsspezifische Overrides

```yaml
# Für Development
act_runner_config:
  runner:
    insecure: true  # Nur für Dev-Umgebungen
    
# Für Production
act_runner_config:
  runner:
    insecure: false
    timeout: "1h"  # Kürzere Timeouts in Prod
```

## Validierungsmuster

### Required Variables

```yaml
# In tasks/verify_vars.yaml
- name: Verify required variables
  assert:
    that:
      - act_runner_gitea_url is defined
      - act_runner_gitea_url | length > 0
      - act_runner_token is defined
      - act_runner_token | length > 0
```

### Configuration Validation

```yaml
- name: Validate runner capacity
  assert:
    that:
      - act_runner_config.runner.capacity is number
      - act_runner_config.runner.capacity > 0
      - act_runner_config.runner.capacity <= 10
    fail_msg: "Runner capacity must be between 1 and 10"
```

## Dokumentationsmuster

### Variable Documentation

```yaml
## @param act_runner_config.runner.capacity Execute how many tasks concurrently
## @param act_runner_config.runner.labels The labels determine which jobs can run
## @param act_runner_config.container.privileged Use privileged mode for containers
```

### Beispiel-Konfigurationen

```yaml
# Beispiele in defaults/main.yaml als Kommentare
act_runner_config:
  runner:
    envs: {}
    # Beispiel:
    # envs:
    #   NODE_ENV: production
    #   CUSTOM_VAR: value
```