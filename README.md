# Ansible Role: pg_autofailover

An Ansible role to install and configure PostgreSQL auto-failover on Ubuntu/Debian systems.

## Description

This role installs the pg_autofailover packages and can configure either a monitor node or keeper nodes for PostgreSQL high-availability clustering.

## Requirements

- Ansible 2.9 or higher
- Ubuntu 20.04+ or Debian 10+
- PostgreSQL 17 (configurable)

## Role Variables

### Package Configuration

```yaml
# PostgreSQL version
pg_autofailover_psql_version: "17"

# Package names
pg_autofailover_packages:
  - pg-auto-failover-cli
  - postgresql-{{ pg_autofailover_psql_version }}-auto-failover
```

### Monitor Configuration

```yaml
# Enable monitor setup
pg_autofailover_monitor_enabled: false

# Monitor data directory
pg_autofailover_monitor_pgdata: "/var/lib/postgresql/monitor"

# Monitor port
pg_autofailover_monitor_pgport: 5500

# Authentication method
pg_autofailover_monitor_auth: "trust"

# Enable self-signed SSL
pg_autofailover_monitor_ssl_self_signed: true
```

### Node Configuration

```yaml
# Enable node setup
pg_autofailover_node_enabled: true

# Node data directory
pg_autofailover_node_pgdata: "/var/lib/postgresql/{{ pg_autofailover_psql_version }}/main"

# Node PostgreSQL port
pg_autofailover_node_pgport: 5432

# Node hostname (defaults to ansible_default_ipv4.address)
pg_autofailover_node_hostname: "{{ ansible_default_ipv4.address }}"

# Monitor URI (REQUIRED when pg_autofailover_node_enabled is true)
pg_autofailover_node_monitor_uri: "postgres://autoctl_node@monitor-host:5500/pg_auto_failover?sslmode=require"

# Node authentication method
pg_autofailover_node_auth: "trust"

# Enable self-signed SSL for node
pg_autofailover_node_ssl_self_signed: true
```

### Service Configuration

```yaml
# Service user and group
pg_autofailover_service_user: "postgres"
pg_autofailover_service_group: "postgres"

# Service restart interval (seconds)
pg_autofailover_service_restart_sec: 2

# Service stop timeout (seconds)
pg_autofailover_service_timeout_stop_sec: 30
```

## Dependencies

None.

## Example Playbook

### Setting up a Monitor Node

```yaml
- hosts: monitor
  become: yes
  roles:
    - role: pg_autofailover
      pg_autofailover_monitor_enabled: true
      pg_autofailover_node_enabled: false
```

### Setting up a Keeper Node

```yaml
- hosts: keepers
  become: yes
  roles:
    - role: pg_autofailover
      pg_autofailover_monitor_enabled: false
      pg_autofailover_node_enabled: true
      pg_autofailover_node_monitor_uri: "postgres://autoctl_node@monitor.example.com:5500/pg_auto_failover"
```

### Complete Cluster Setup

```yaml
# Monitor node
- hosts: monitor
  become: yes
  roles:
    - role: pg_autofailover
      pg_autofailover_monitor_enabled: true
      pg_autofailover_node_enabled: false

# Keeper nodes
- hosts: keepers
  become: yes
  roles:
    - role: pg_autofailover
      pg_autofailover_monitor_enabled: false
      pg_autofailover_node_enabled: true
      pg_autofailover_node_monitor_uri: "postgres://autoctl_node@{{ hostvars[groups['monitor'][0]]['ansible_host'] }}:5500/pg_auto_failover"
```

## Installation Commands Reference

This role automates the following manual installation steps:

```bash
# Install packages
apt install pg-auto-failover-cli postgresql-17-auto-failover

# Create monitor (handled by role)
sudo -u postgres env XDG_RUNTIME_DIR=/run/postgresql pg_autoctl create monitor \
  --pgdata /var/lib/postgresql/monitor \
  --pgport 5500 \
  --auth trust \
  --ssl-self-signed

# Create postgres node (handled by role)
sudo -u postgres env XDG_RUNTIME_DIR=/run/postgresql pg_autoctl create postgres \
  --pgdata /var/lib/postgresql/17/main \
  --pgport 5432 \
  --hostname 10.0.0.10 \
  --monitor "postgres://autoctl_node@10.0.0.10:5500/pg_auto_failover?sslmode=require" \
  --auth trust \
  --ssl-self-signed

# Systemd service setup (handled by role)
systemctl daemon-reload
systemctl enable --now pg_autoctl-node.service
```

## Daily Operations

The role deploys a wrapper script with simplified commands for common operations:

### Simplified Commands (via aliases)

After logging in (or `source /etc/profile.d/pg_autoctl_aliases.sh`), use these aliases:

```bash
# Show cluster state
pgctl-state

# Perform coordinated switchover
pgctl-switchover

# Perform failover
pgctl-failover

# Show current node status
pgctl-status

# Show help
pgctl-help
```

### Direct Wrapper Functions

Alternatively, source the wrapper and use function names:

```bash
source /usr/local/bin/pg_autoctl_wrapper.sh

pg_autoctl_show_state          # Show cluster state from monitor
pg_autoctl_switchover          # Perform a coordinated switchover
pg_autoctl_failover            # Perform a failover
pg_autoctl_status              # Show current node status
pg_autoctl_config [setting]    # Show configuration setting(s)
pg_autoctl_enable_maintenance  # Enable maintenance mode
pg_autoctl_disable_maintenance # Disable maintenance mode
```

### Original Commands (if needed)

The wrapper encapsulates these full commands:

```bash
# Show cluster state
sudo -u postgres pg_autoctl show state \
  --pgdata /var/lib/postgresql/17/main \
  --monitor "postgres://autoctl_node@10.0.0.10:5500/pg_auto_failover?sslmode=require"

# Perform switchover
sudo -u postgres pg_autoctl perform switchover \
  --monitor "postgres://autoctl_node@10.0.0.10:5500/pg_auto_failover?sslmode=require"
```

## Directory Structure

```
pg_autofailover/
├── defaults/
│   └── main.yml          # Default variables
├── handlers/
│   └── main.yml          # Service handlers
├── meta/
│   └── main.yml          # Role metadata
├── tasks/
│   ├── main.yml          # Main task orchestration
│   ├── install.yml       # Package installation
│   ├── monitor.yml       # Monitor setup
│   └── node.yml          # Node setup
├── templates/
│   ├── pg_autoctl-monitor.service.j2  # Monitor systemd unit
│   ├── pg_autoctl-node.service.j2     # Node systemd unit
│   └── pg_autoctl_wrapper.sh.j2       # Wrapper script for daily ops
├── vars/
│   └── default.yml       # Fall back for OS-specific defaults
└── README.md
```

## License

MIT

## Author Information

Created for PostgreSQL auto-failover deployment automation.
