# Ansible Role: pg_autofailover

This is an Ansible role for installing and configuring PostgreSQL auto-failover.

## Project Structure
- Standard Ansible role structure with tasks, handlers, templates, defaults, and meta directories
- Installs pg-auto-failover-cli and postgresql-17-auto-failover packages
- Configures pg_autoctl monitor
- Sets up systemd service for pg_autoctl node

## Development Guidelines
- Follow Ansible best practices
- Use YAML for all configuration files
- Ensure idempotent task execution
- Test on Ubuntu/Debian systems
