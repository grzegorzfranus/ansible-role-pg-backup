# Ansible Role: PostgreSQL Backup

|Source|Version|CI|License|
|------|-------|-------|-------|
|[![Source Code](https://img.shields.io/badge/source-github-blue.svg)](https://github.com/grzegorzfranus/ansible-role-pg-backup)|[![Version](https://img.shields.io/github/v/release/grzegorzfranus/ansible-role-pg-backup)](https://github.com/grzegorzfranus/ansible-role-pg-backup/releases)|[![tests](https://github.com/grzegorzfranus/ansible-role-pg-backup/actions/workflows/test-and-validation.yml/badge.svg)](https://github.com/grzegorzfranus/ansible-role-pg-backup/actions)|[![Repository License](https://img.shields.io/badge/license-apache2.0-brightgreen.svg)](LICENSE)|

Professional Ansible role for automated PostgreSQL database backups with comprehensive logging, retention management, and cron scheduling.

## ✨ Features

- 📦 **Custom format backups** (`-Fc`) - Compressed, supports parallel restore
- 📝 **Comprehensive logging** - Timestamped logs with severity levels
- ✅ **Pre-flight checks** - Connection, disk space, permissions validation
- 🔄 **Automatic retention** - Configurable cleanup of old backups
- 🔐 **Secure authentication** - Uses `.pgpass` file managed by Ansible
- ⏱️ **Lock timeout handling** - Prevents indefinite hangs on locked tables
- 🧪 **Backup verification** - Integrity check after each dump using `pg_restore -l`
- 🌐 **Global objects backup** - Roles and tablespaces included
- ⏰ **Cron scheduling** - Automated daily backups (configurable)
- 📊 **Logrotate integration** - Automatic log rotation

## 🎯 Architecture

The role sets up a complete backup solution:

```
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Backup Role                       │
├─────────────────────────────────────────────────────────────────┤
│  📦 Prerequisites     → Install postgresql-client packages      │
│  📝 Script Install    → Deploy parameterized backup script      │
│  🔐 Authentication    → Configure .pgpass file                  │
│  📂 Directories       → Create backup and log directories       │
│  📊 Logrotate         → Configure log rotation                  │
│  ⏰ Cron              → Schedule automated backups              │
└─────────────────────────────────────────────────────────────────┘
```

## 📋 Requirements

- **Ansible**: 2.14 or higher
- **PostgreSQL**: Server running and accessible
- **Privileges**: sudo/root access on target hosts for package installation and cron setup

### Supported operating systems

| OS Family | Version | Status |
|-----------|---------|--------|
| Ubuntu | 24.04 (Noble) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Ubuntu | 22.04 (Jammy) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Debian | 12 (Bookworm) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Debian | 11 (Bullseye) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |

### Ansible version

Ansible >= 2.14

### Python version

Python >= 3.8

## 🚀 Quick Start

### 1. Minimal Playbook

```yaml
---
- name: Configure PostgreSQL backup
  hosts: database_servers
  become: true
  roles:
    - role: grzegorzfranus.ansible_role_pg_backup
      vars:
        pg_backup_pg_password: "{{ vault_pg_backup_password }}"
```

### 2. Run the playbook

```bash
ansible-playbook -i inventory playbook.yml --ask-vault-pass
```

## ⚙️ Configuration

### Default Configuration

The role comes with sensible defaults for a standard setup:

```yaml
# PostgreSQL connection
pg_backup_pg_host: "localhost"
pg_backup_pg_port: 5432
pg_backup_pg_user: "postgres"
pg_backup_pg_password: ""  # Use Ansible Vault!

# Backup settings
pg_backup_dir: "/var/backups/postgresql"
pg_backup_retention_days: 7
pg_backup_compression_level: 6

# Cron schedule (daily at 2:00 AM)
pg_backup_cron_enabled: true
pg_backup_cron_minute: "0"
pg_backup_cron_hour: "2"
```

### Advanced Configuration

Customize for specific requirements:

```yaml
---
- name: Advanced PostgreSQL Backup
  hosts: database_servers
  become: true
  vars:
    # Connection
    pg_backup_pg_host: "db.example.com"
    pg_backup_pg_user: "backup_user"
    pg_backup_pg_password: "{{ vault_pg_backup_password }}"
    
    # Backup retention and storage
    pg_backup_retention_days: 30
    pg_backup_dir: "/mnt/backups/postgres"
    
    # Logging
    pg_backup_log_dir: "/var/log/pg_backups"
    pg_backup_configure_logrotate: true
    
    # Timeout safety
    pg_backup_lock_wait_timeout: 600
  roles:
    - role: grzegorzfranus.ansible_role_pg_backup
```

## 📊 Variables

### General Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `pg_backup_role_action` | Define which parts of the role to execute (Options: 'all', 'install', 'configure') | `"all"` |
| `pg_backup_cron_enabled` | Enable automated cron backups | `true` |
| `pg_backup_user` | User that will run the backup script | `"postgres"` |
| `pg_backup_group` | Group for backup files and directories | `"postgres"` |

### PostgreSQL Connection Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `pg_backup_pg_host` | PostgreSQL server hostname or IP | `"localhost"` |
| `pg_backup_pg_port` | PostgreSQL server port | `5432` |
| `pg_backup_pg_user` | PostgreSQL username for backup operations | `"postgres"` |
| `pg_backup_pg_password` | PostgreSQL password (use Ansible Vault!) | `""` |
| `pg_backup_configure_pgpass` | Automatically manage .pgpass file | `true` |
| `pg_backup_pgpass_mode` | Permissions for .pgpass file | `"0600"` |

### Backup Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `pg_backup_dir` | Directory where backups will be stored | `"/var/backups/postgresql"` |
| `pg_backup_dir_mode` | Permissions for backup directory | `"0750"` |
| `pg_backup_file_mode` | Permissions for backup files | `"0600"` |
| `pg_backup_compression_level` | Compression level for pg_dump (0-9) | `6` |
| `pg_backup_script_path` | Path where the backup script will be installed | `"/usr/local/bin/pg_backup.sh"` |
| `pg_backup_script_mode` | Permissions for backup script | `"0750"` |
| `pg_backup_exclude_databases` | List of databases to exclude from backup | `["template0", "template1"]` |

### Retention Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `pg_backup_retention_days` | Number of days to keep backups | `7` |

### Logging Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `pg_backup_enable_logging` | Enable dedicated logging for backup operations | `true` |
| `pg_backup_log_dir` | Directory for backup log files | `"/var/log/postgresql"` |
| `pg_backup_log_dir_mode` | Log directory permissions | `"0755"` |
| `pg_backup_log_file` | Main backup log file path | `{{ pg_backup_log_dir }}/pg_backup.log` |
| `pg_backup_log_file_mode` | Log file permissions | `"0640"` |

### Cron Schedule Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `pg_backup_cron_name` | Cron job name for identification | `"PostgreSQL Daily Backup"` |
| `pg_backup_cron_minute` | Cron minute (0-59) | `"0"` |
| `pg_backup_cron_hour` | Cron hour (0-23) | `"2"` |
| `pg_backup_cron_day` | Cron day of month | `"*"` |
| `pg_backup_cron_month` | Cron month | `"*"` |
| `pg_backup_cron_weekday` | Cron day of week | `"*"` |
| `pg_backup_cron_log_file` | File to redirect cron output to | `{{ pg_backup_log_dir }}/pg_backup_cron.log` |

### Script Timeout Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `pg_backup_lock_wait_timeout` | Max seconds to wait for table locks (prevents hangs) | `300` |
| `pg_backup_connection_timeout` | Connection check timeout in seconds | `30` |

### Logrotate Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `pg_backup_configure_logrotate` | Enable logrotate configuration | `true` |
| `pg_backup_logrotate_options` | Dictionary of logrotate settings | See below |
| `pg_backup_logrotate_options.frequency` | Rotation frequency | `"daily"` |
| `pg_backup_logrotate_options.count` | Number of rotated logs to keep | `14` |
| `pg_backup_logrotate_options.compress` | Compress rotated logs | `true` |
| `pg_backup_logrotate_options.archive_directory_path` | Directory for archived logs | `{{ pg_backup_log_dir }}/archived` |

## 🔍 Verification

After deployment, verify the backup system is working:

### Check Backup Script

```bash
# Test script manually (verbose mode)
sudo -u postgres /usr/local/bin/pg_backup.sh -v

# Dry run (no actual backup)
sudo -u postgres /usr/local/bin/pg_backup.sh -d
```

### Check Cron Job

```bash
# List cron jobs for postgres user
sudo crontab -u postgres -l
```

### Check Backups

```bash
# List backup directory
ls -la /var/backups/postgresql/
```

## 🔄 Restore Procedures

> ⚠️ **IMPORTANT**: Always restore global objects (roles/users) BEFORE restoring databases!
> Otherwise, database ownership and permissions will fail because the roles don't exist yet.

### Step 1: Restore Global Objects (REQUIRED FIRST)

Global objects include roles (users), tablespaces, and role memberships.

```bash
psql -h localhost -U postgres -f globals_YYYY-MM-DD_HHMMSS.sql
```

### Step 2: Restore Database(s)

#### Restore Single Database

```bash
pg_restore -h localhost -U postgres -d database_name -v backup_file.dump
```

#### Restore to New Database

```bash
createdb -U postgres new_database_name
pg_restore -h localhost -U postgres -d new_database_name -v backup_file.dump
```

#### Parallel Restore (Faster for Large DBs)

```bash
pg_restore -h localhost -U postgres -d database_name -j 4 -v backup_file.dump
```

## 🛡️ Security Features

- ✅ **Secure Password Storage**: Uses `.pgpass` with strict `0600` permissions
- ✅ **Restricted Permissions**: Backup files are readable only by the owner (`0600`)
- ✅ **Script Security**: Backup script is executable only by owner/group (`0750`)
- ✅ **Vault Integration**: Designed to work with Ansible Vault for password protection

## 🔧 Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| "Cannot connect to PostgreSQL" | Verify PostgreSQL is running, check `.pgpass` content and permissions |
| "Insufficient disk space" | Free space or change `pg_backup_dir` to a larger partition |
| "Lock wait timeout exceeded" | Increase `pg_backup_lock_wait_timeout` if you have long-running transactions |
| "Permission denied" | Ensure `pg_backup_user` has write access to `pg_backup_dir` |

### Debug Commands

```bash
# Check PostgreSQL connectivity
pg_isready -h localhost -U postgres

# Verify .pgpass
cat ~/.pgpass && ls -la ~/.pgpass

# Check logs
tail -f /var/log/postgresql/pg_backup.log
```

## 📁 File Structure

```
ansible-role-pg-backup/
├── .github/                  # GitHub Actions workflows
├── CHANGELOG.md              # Version history
├── LICENSE                   # Apache-2.0 license
├── README.md                 # This documentation
├── defaults/
│   └── main.yml              # Default variables
├── handlers/
│   └── main.yml              # Service handlers
├── meta/
│   └── main.yml              # Role metadata
├── tasks/
│   ├── main.yml              # Main orchestration
│   ├── assert.yml            # Variable validation
│   ├── prerequisites.yml     # Package installation
│   ├── install.yml           # Script deployment
│   ├── configure.yml         # Configuration setup
│   ├── logrotate.yml         # Log rotation config
│   └── cron.yml              # Cron job setup
├── templates/
│   ├── logrotate/
│   │   └── pg_backup.j2      # Logrotate template
│   ├── scripts/
│   │   └── pg_backup.sh.j2   # Backup script template
│   └── pgpass.j2             # .pgpass template
└── vars/
    └── main.yml              # Internal variables
```

## 🏷️ Tags

- `always` - Validation tasks
- `setup` - Initial setup
- `prerequisites` - Package installation
- `install` - Script installation
- `configure` - Configuration tasks
- `logrotate` - Logrotate configuration
- `cron` - Cron job setup

## Example Playbook

```yaml
---
- name: Configure PostgreSQL Backup
  hosts: database_servers
  become: true
  
  vars:
    # Security
    pg_backup_pg_password: "{{ vault_pg_backup_password }}"
    
    # Retention
    pg_backup_retention_days: 14
    
    # Schedule (3:00 AM)
    pg_backup_cron_hour: "3"
    
    # Log Rotation
    pg_backup_logrotate_options:
      frequency: "daily"
      count: 30
      compress: true
      delaycompress: true
      missingok: true
      notifempty: true
      create: true
      create_mode: "0640"
      create_owner: "postgres"
      create_group: "postgres"
      dateext: true
      archive_directory_path: "/var/log/postgresql/archived"

  roles:
    - role: grzegorzfranus.pg_backup
```

## 🧪 Testing

This role includes comprehensive Molecule tests that validate functionality across supported operating systems.

### Running Tests Locally

```bash
# Install testing dependencies
pip install molecule molecule-plugins[docker] ansible-lint

# Run all tests
molecule test
```

## 🔧 CI/CD Integration

This role includes GitHub Actions workflows for automated testing and deployment:

- **Test & Validation**: Runs Molecule tests on Ubuntu and Debian
- **Galaxy Publishing**: Automatically publishes to Ansible Galaxy on release

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

- Fork the repository and create your branch from `main`.
- Make your changes with clear, descriptive commit messages.
- Ensure your code passes all Molecule and lint tests.
- Submit a pull request describing your changes.

## 📝 License

This project is licensed under the Apache-2.0 License - see the LICENSE file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).
